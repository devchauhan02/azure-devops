# Python for DevOps — Days 3 & 4
## Advanced Python, Web, and HTTP

---

## DAY 3 — Advanced Python Concepts

### Scopes and the LEGB Rule

**Scope** defines where in your code a variable name is visible and accessible. Python resolves names using the **LEGB rule**, searching in this order:

1. **L — Local**: Names defined inside the current function.
2. **E — Enclosing**: Names in the local scope of any enclosing (outer) functions (relevant for closures and nested functions).
3. **G — Global**: Names defined at the top level of the module.
4. **B — Built-in**: Python's own built-in names (`len`, `print`, `range`, `True`, etc.).

Python searches these scopes in order and uses the first match found.

```python
# Global scope
server_count = 10       # Global variable

def configure():
    # Local scope
    timeout = 30        # Local variable — invisible outside this function
    print(server_count) # Finds it in Global scope (G in LEGB)
    print(timeout)

def modify_global():
    global server_count         # Declare intent to modify the global
    server_count = 20           # Now modifies the global, not a new local

configure()
# print(timeout)  # NameError — timeout is local to configure()
```

#### `global` and `nonlocal`

- `global` — allows a function to read AND write a module-level variable.
- `nonlocal` — used in nested functions to write to a variable in the enclosing (outer) function's scope.

```python
def outer():
    count = 0               # Enclosing scope

    def inner():
        nonlocal count      # Access outer's count
        count += 1
        print(count)

    inner()   # 1
    inner()   # 2
    inner()   # 3

outer()
```

**Best practice for DevOps**: Avoid mutable global state. Pass configuration through function parameters or class attributes instead. Global state makes code harder to test and reason about in complex automation scripts.

---

### Lambda Expressions (Anonymous Functions)

A `lambda` is a small, **anonymous function** defined in a single expression. It can have any number of arguments but only one expression (no statements, no assignments, no loops). The expression is evaluated and returned automatically.

```python
# Syntax: lambda arguments: expression
square = lambda x: x ** 2
add = lambda x, y: x + y

square(5)   # 25
add(3, 4)   # 7
```

Lambdas shine as **inline arguments** to higher-order functions (`sorted`, `map`, `filter`):

```python
servers = [
    {"name": "web-01", "cpu": 72},
    {"name": "db-01",  "cpu": 45},
    {"name": "api-01", "cpu": 88},
]

# Sort by CPU usage
sorted_servers = sorted(servers, key=lambda s: s["cpu"])
# [{'name': 'db-01', 'cpu': 45}, ...]

# Filter servers with high CPU
high_cpu = list(filter(lambda s: s["cpu"] > 70, servers))

# Map — transform each element
names = list(map(lambda s: s["name"].upper(), servers))
# ["WEB-01", "DB-01", "API-01"]
```

`map()` applies a function to every element of an iterable.  
`filter()` keeps only elements where the function returns True.  
Both are lazy (return iterators); wrap with `list()` to materialize.

A note on style: for complex logic, always prefer a named `def` over a multi-argument lambda. Lambdas should be short and obvious.

---

### Classes and Objects

**Object-Oriented Programming (OOP)** models the real world as objects that have state (attributes) and behavior (methods). A **class** is a blueprint; an **object** is a concrete instance of that blueprint.

When a class is defined, no memory is allocated. Memory is only allocated when an object is **instantiated** (created) using the class.

```python
class Server:
    # Class attribute — shared by ALL instances
    provider = "AWS"

    # __init__ is the initializer (not quite a constructor)
    # self refers to the instance being created
    def __init__(self, hostname, ip, role="web"):
        # Instance attributes — unique to each object
        self.hostname = hostname
        self.ip = ip
        self.role = role
        self.status = "stopped"

    def start(self):
        self.status = "running"
        print(f"{self.hostname} is now running")

    def stop(self):
        self.status = "stopped"

    def __repr__(self):
        return f"Server(hostname={self.hostname!r}, role={self.role!r})"


# Instantiation
web = Server("web-01", "10.0.0.1")
db  = Server("db-01",  "10.0.0.5", role="database")

web.start()              # web-01 is now running
print(web.status)        # running
print(db.provider)       # AWS — class attribute
print(web)               # Server(hostname='web-01', role='web')
```

#### Attribute Modification

Attributes can be read and modified directly, or via methods. Direct modification works but using methods is better practice because methods can enforce validation:

```python
# Direct modification
web.ip = "10.0.0.2"

# Using a property for controlled access
class ManagedServer:
    def __init__(self, hostname):
        self.hostname = hostname
        self._port = 80       # Convention: _ means "private-ish"

    @property
    def port(self):
        return self._port

    @port.setter
    def port(self, value):
        if not (1 <= value <= 65535):
            raise ValueError("Port must be 1–65535")
        self._port = value

s = ManagedServer("web-01")
s.port = 8080     # calls the setter
print(s.port)     # calls the getter → 8080
s.port = 99999    # raises ValueError
```

#### Inheritance

A **child class** inherits all attributes and methods of its **parent class** and can override or extend them:

```python
class CloudServer(Server):
    def __init__(self, hostname, ip, region):
        super().__init__(hostname, ip)   # call parent's __init__
        self.region = region

    def start(self):
        print(f"Provisioning in {self.region}...")
        super().start()   # also call parent's start()

ec2 = CloudServer("ec2-web-01", "52.0.0.1", "us-east-1")
ec2.start()
# Provisioning in us-east-1...
# ec2-web-01 is now running
```

---

### Class MRO — Method Resolution Order

When using **multiple inheritance**, Python needs a deterministic order to search for methods. This is computed using the **C3 linearization algorithm** and stored in the class's `__mro__` attribute.

```python
class A:
    def hello(self):
        return "A"

class B(A):
    def hello(self):
        return "B"

class C(A):
    def hello(self):
        return "C"

class D(B, C):   # Multiple inheritance
    pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

d = D()
d.hello()   # "B" — Python searches D → B → C → A → object, stops at B
```

`super()` respects MRO, so calling `super()` in a chain of classes traverses the full MRO in order — this is the key mechanism behind cooperative multiple inheritance.

---

### Magic Methods (Dunder Methods)

Magic methods (also called **dunder methods** from "double underscore") allow your classes to integrate with Python's built-in operations and syntax. They are how Python's operator overloading works.

```python
class ResourcePool:
    def __init__(self, name, capacity):
        self.name = name
        self.capacity = capacity
        self._resources = []

    # Called by repr() — for developers/debugging
    def __repr__(self):
        return f"ResourcePool(name={self.name!r}, capacity={self.capacity})"

    # Called by str() / print() — for end users
    def __str__(self):
        return f"{self.name} ({len(self._resources)}/{self.capacity})"

    # Called by len()
    def __len__(self):
        return len(self._resources)

    # Called by 'item in pool'
    def __contains__(self, item):
        return item in self._resources

    # Called by pool[index]
    def __getitem__(self, index):
        return self._resources[index]

    # Called by pool + other_pool
    def __add__(self, other):
        combined = ResourcePool(f"{self.name}+{other.name}", self.capacity + other.capacity)
        combined._resources = self._resources + other._resources
        return combined

    # Context manager protocol (__enter__ and __exit__)
    def __enter__(self):
        print(f"Acquiring pool: {self.name}")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Releasing pool: {self.name}")
        return False  # Don't suppress exceptions

pool = ResourcePool("workers", 10)
print(len(pool))          # 0 — calls __len__
print(str(pool))          # workers (0/10) — calls __str__
print(repr(pool))         # ResourcePool(name='workers', capacity=10)

with ResourcePool("db-connections", 5) as p:
    print(f"Using {p}")   # Context manager protocol
```

Other useful dunder methods: `__eq__` (==), `__lt__` (<), `__hash__`, `__call__` (make instance callable), `__iter__`, `__next__`.

---

### Modules

A **module** is any `.py` file. Modules let you organize code into logical namespaces and reuse them across projects.

```python
# utils.py
def format_size(bytes):
    """Convert bytes to human-readable size."""
    for unit in ["B", "KB", "MB", "GB", "TB"]:
        if bytes < 1024:
            return f"{bytes:.2f} {unit}"
        bytes /= 1024

# In another file
import utils
print(utils.format_size(1073741824))   # 1.00 GB

# Import specific names
from utils import format_size
print(format_size(2048))               # 2.00 KB

# Import with alias
import utils as u
u.format_size(512)
```

Python searches for modules in `sys.path`, which includes the current directory, PYTHONPATH, and standard library paths.

---

### File Handling

Python's `open()` built-in provides file I/O. Always use it as a **context manager** (`with` statement) — it guarantees the file is closed even if an exception occurs.

```python
# Writing a file
with open("servers.txt", "w") as f:    # "w" = write (creates/overwrites)
    f.write("web-01\n")
    f.write("web-02\n")

# Appending to a file
with open("servers.txt", "a") as f:    # "a" = append
    f.write("db-01\n")

# Reading entire file
with open("servers.txt", "r") as f:    # "r" = read (default)
    content = f.read()                 # one big string

# Reading line by line (memory efficient for large files)
with open("servers.txt") as f:
    for line in f:
        print(line.strip())   # strip() removes the trailing newline

# Reading all lines into a list
with open("servers.txt") as f:
    lines = f.readlines()   # ["web-01\n", "web-02\n", ...]

# Working with JSON (very common in DevOps)
import json

config = {"host": "localhost", "port": 5432}
with open("config.json", "w") as f:
    json.dump(config, f, indent=2)

with open("config.json") as f:
    loaded = json.load(f)

# Working with CSV
import csv

with open("inventory.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["host", "role", "ip"])
    writer.writeheader()
    writer.writerow({"host": "web-01", "role": "web", "ip": "10.0.0.1"})
```

**File modes**: `"r"` (read), `"w"` (write, truncate), `"a"` (append), `"x"` (exclusive create, fail if exists), `"b"` (binary mode, combine as `"rb"`, `"wb"`), `"+"` (read+write).

---

### Packages

A **package** is a directory containing an `__init__.py` file (which can be empty) and other `.py` modules. Packages let you build hierarchical namespaces.

```
devops_tools/
├── __init__.py
├── aws/
│   ├── __init__.py
│   ├── ec2.py
│   └── s3.py
└── monitoring/
    ├── __init__.py
    └── metrics.py
```

```python
# Importing from a package
from devops_tools.aws import ec2
from devops_tools.monitoring.metrics import collect_cpu

# __init__.py can expose a clean public API
# devops_tools/__init__.py
from .aws.ec2 import launch_instance
from .monitoring.metrics import collect_cpu
__all__ = ["launch_instance", "collect_cpu"]

# Now users can do:
from devops_tools import launch_instance
```

When you `pip install` a package, it installs into `site-packages` and becomes importable anywhere in that environment.

---

## DAY 4 — Exceptions, Debugging, Web & HTTP

### Exceptions

**Exceptions** are runtime errors that disrupt normal program flow. Python uses a class hierarchy for exceptions — `BaseException` at the root, `Exception` as the base for most user-visible exceptions, then specific subclasses like `ValueError`, `FileNotFoundError`, `KeyError`, etc.

```python
# Basic try/except
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Cannot divide by zero: {e}")

# Multiple except clauses
try:
    data = json.loads(user_input)
    value = data["key"]
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
except KeyError as e:
    print(f"Missing key: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
    raise   # re-raise the exception

# else — runs only if NO exception was raised
# finally — ALWAYS runs (cleanup: close files, release locks)
try:
    conn = open_connection("db-01")
    result = conn.query("SELECT 1")
except ConnectionError as e:
    print(f"Connection failed: {e}")
else:
    print(f"Query result: {result}")
finally:
    conn.close()   # Always executed, even if exception occurred
```

#### Raising Exceptions

```python
def connect(host, port):
    if not isinstance(port, int):
        raise TypeError(f"Port must be int, got {type(port).__name__}")
    if not (1 <= port <= 65535):
        raise ValueError(f"Port {port} out of valid range")
```

#### Custom Exceptions

Define custom exceptions by subclassing `Exception`. This lets callers catch your specific error type and not accidentally catch unrelated errors:

```python
class DeploymentError(Exception):
    """Raised when a deployment fails."""
    def __init__(self, service, reason):
        self.service = service
        self.reason = reason
        super().__init__(f"Deployment of '{service}' failed: {reason}")

class RollbackError(DeploymentError):
    """Raised when rollback also fails."""
    pass

try:
    raise DeploymentError("payment-service", "health check timeout")
except DeploymentError as e:
    print(f"Service affected: {e.service}")
    print(e)
```

#### Context Managers and `contextlib`

```python
from contextlib import contextmanager

@contextmanager
def managed_deployment(service):
    print(f"Starting deployment of {service}")
    try:
        yield service    # code inside 'with' block runs here
    except Exception as e:
        print(f"Deployment failed, rolling back: {e}")
        raise
    finally:
        print(f"Cleanup for {service}")

with managed_deployment("api-gateway") as svc:
    print(f"Deploying {svc}...")
    # do actual deployment work here
```

---

### Debugging

#### Python Debugger (PDB)

`pdb` is Python's built-in command-line debugger. It lets you pause execution, inspect variables, step through code line-by-line, and set breakpoints — all without an IDE.

```python
import pdb

def process_config(config):
    pdb.set_trace()   # Execution pauses here and drops into interactive debugger
    host = config["host"]
    port = config["port"]
    return f"{host}:{port}"
```

**Python 3.7+**: Use `breakpoint()` instead — it's cleaner and hooks into `PYTHONBREAKPOINT` env var:

```python
def process_config(config):
    breakpoint()   # Same as pdb.set_trace() but more flexible
    ...
```

**PDB commands**:

| Command | Action |
|---------|--------|
| `n` | Next line (step over function calls) |
| `s` | Step into function calls |
| `c` | Continue until next breakpoint |
| `l` | List source code around current line |
| `p expression` | Print the value of an expression |
| `pp expression` | Pretty-print |
| `b 42` | Set breakpoint at line 42 |
| `b func` | Set breakpoint at start of function |
| `w` | Print stack trace (where am I?) |
| `q` | Quit the debugger |
| `h` | Help |

Set `PYTHONBREAKPOINT=0` environment variable to disable all `breakpoint()` calls without changing code (useful in production).

#### Logging

`print()` is fine for quick debugging but insufficient for production systems. The `logging` module provides structured, configurable, leveled output that can be sent to files, remote servers, or filtered by severity.

**Log levels** (in order of severity): `DEBUG < INFO < WARNING < ERROR < CRITICAL`

```python
import logging

# Basic configuration — sets up a handler to stderr
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logger = logging.getLogger(__name__)   # Use module name as logger name

logger.debug("Connecting to host: %s", host)        # Development details
logger.info("Deployment started for %s", service)   # Normal operation
logger.warning("Retry %d of %d", attempt, max_tries)
logger.error("Connection failed: %s", error)        # Problem, but continuing
logger.critical("Database unreachable, shutting down")
```

**Configuring handlers** — send logs to different destinations:

```python
import logging

logger = logging.getLogger("devops")
logger.setLevel(logging.DEBUG)

# Console handler — shows INFO and above
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

# File handler — logs everything
file_handler = logging.FileHandler("deploy.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(
    logging.Formatter("%(asctime)s %(name)s %(levelname)s: %(message)s")
)

logger.addHandler(console)
logger.addHandler(file_handler)

# Now logger has two outputs
logger.debug("Detailed info goes to file only")
logger.info("This goes to both console and file")
```

**Best practices**:
- Always use `logging.getLogger(__name__)` — not the root logger.
- Use `%s` style formatting in log calls (lazy evaluation; skips string formatting if level is filtered out).
- Use `logging.exception()` inside `except` blocks to automatically include the traceback.

```python
try:
    risky_operation()
except Exception:
    logger.exception("risky_operation failed")  # logs ERROR + full traceback
```

---

### Jinja2

**Jinja2** is a templating engine for Python, widely used to generate dynamic configuration files, HTML, Kubernetes manifests, Ansible playbooks, and more. Ansible uses Jinja2 natively.

```bash
pip install jinja2
```

**Core syntax**:
- `{{ variable }}` — output/interpolation
- `{% statement %}` — control flow (if, for, block)
- `{# comment #}` — template comments (not in output)

```python
from jinja2 import Environment, FileSystemLoader, Template

# Inline template
template = Template("""
server {
    listen {{ port }};
    server_name {{ hostname }};
    {% for location in locations %}
    location {{ location.path }} {
        proxy_pass {{ location.upstream }};
    }
    {% endfor %}
}
""")

config = template.render(
    port=80,
    hostname="api.example.com",
    locations=[
        {"path": "/api", "upstream": "http://backend:8000"},
        {"path": "/health", "upstream": "http://backend:8000/health"},
    ]
)
print(config)
```

**Loading from files** (recommended for real projects):

```python
env = Environment(
    loader=FileSystemLoader("templates/"),
    trim_blocks=True,     # Remove first newline after block tags
    lstrip_blocks=True    # Strip leading whitespace from block tags
)

template = env.get_template("nginx.conf.j2")
output = template.render(context)

with open("/etc/nginx/sites-enabled/app.conf", "w") as f:
    f.write(output)
```

**Jinja2 filters** (pipe `|` syntax):
```
{{ hostname | upper }}          → "WEB-01"
{{ items | length }}            → 3
{{ cpu | round(1) }}            → 87.6
{{ servers | join(', ') }}      → "web-01, web-02"
{{ value | default('N/A') }}    → "N/A" if value is undefined
```

---

### Python Application Architecture for Web

A Python web application involves three layers:

1. **Web Server Layer** — Handles raw HTTP/HTTPS connections, TLS termination, static files, and acts as a reverse proxy. Examples: **nginx**, **Apache**. These are production-grade, battle-tested servers written in C.

2. **WSGI Server Layer** — Bridges the web server and the Python application. **WSGI** (Web Server Gateway Interface, PEP 3333) is a standard interface — a Python callable that accepts `(environ, start_response)` and returns a response body iterable. WSGI servers run the Python code. Examples: **Gunicorn**, **uWSGI**, **Waitress**.

3. **Web Application Framework Layer** — The Python code you write. Handles routing, request parsing, business logic, and response generation. Examples: **Flask**, **Django**, **FastAPI**.

```
Browser → nginx (port 443, TLS)
        → Gunicorn (port 8000, WSGI)
        → Flask app (Python code)
```

**WSGI in its simplest form:**

```python
# A valid WSGI application — no framework needed
def application(environ, start_response):
    status = "200 OK"
    headers = [("Content-Type", "text/plain")]
    start_response(status, headers)
    return [b"Hello from WSGI!"]
```

---

### Flask

**Flask** is a micro web framework — it gives you routing and request handling with minimal overhead, leaving architecture decisions to you.

```bash
pip install flask
```

```python
from flask import Flask, request, jsonify, abort
import logging

app = Flask(__name__)
logger = logging.getLogger(__name__)

# In-memory data store (use a real DB in production)
servers = {
    "web-01": {"ip": "10.0.0.1", "status": "running"},
    "db-01":  {"ip": "10.0.0.5", "status": "stopped"},
}

# Route with URL parameter
@app.route("/servers/<hostname>", methods=["GET"])
def get_server(hostname):
    server = servers.get(hostname)
    if not server:
        abort(404)
    return jsonify({"hostname": hostname, **server})

# POST endpoint
@app.route("/servers", methods=["POST"])
def add_server():
    data = request.get_json()
    if not data or "hostname" not in data:
        abort(400)
    hostname = data["hostname"]
    servers[hostname] = {"ip": data.get("ip", ""), "status": "stopped"}
    logger.info("Added server: %s", hostname)
    return jsonify({"message": "created", "hostname": hostname}), 201

# List all servers
@app.route("/servers", methods=["GET"])
def list_servers():
    return jsonify(servers)

# Error handlers
@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "Not found"}), 404

@app.errorhandler(400)
def bad_request(e):
    return jsonify({"error": "Bad request"}), 400

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)
```

Running: `flask run` (uses `FLASK_APP` env var) or `python app.py`.  
**Never use `debug=True` in production** — it enables an interactive debugger that executes arbitrary code.

#### Flask Blueprints (for larger apps)

Blueprints let you split routes across multiple files:

```python
# blueprints/servers.py
from flask import Blueprint, jsonify
servers_bp = Blueprint("servers", __name__, url_prefix="/servers")

@servers_bp.route("/")
def list_servers():
    return jsonify({"servers": []})

# app.py
from flask import Flask
from blueprints.servers import servers_bp
app = Flask(__name__)
app.register_blueprint(servers_bp)
```

---

### HTTP Fundamentals

**HTTP (HyperText Transfer Protocol)** is the application-layer protocol for transferring data on the web. It is stateless — each request is independent.

#### Request Structure
```
METHOD /path?query=value HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{"key": "value"}  ← body (for POST/PUT/PATCH)
```

**HTTP Methods**:
- `GET` — retrieve a resource (idempotent, no body)
- `POST` — create a resource
- `PUT` — replace a resource (idempotent)
- `PATCH` — partially update a resource
- `DELETE` — remove a resource (idempotent)
- `HEAD` — like GET but returns only headers
- `OPTIONS` — ask what methods are supported (used in CORS)

**HTTP Status Codes**:
- `2xx` Success: `200 OK`, `201 Created`, `204 No Content`
- `3xx` Redirect: `301 Moved Permanently`, `302 Found`, `304 Not Modified`
- `4xx` Client Error: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`
- `5xx` Server Error: `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

---

### `requests` Library

`requests` is the de facto standard for making HTTP requests in Python. It is much friendlier than the built-in `urllib`.

```bash
pip install requests
```

```python
import requests

# GET request
response = requests.get(
    "https://api.github.com/repos/python/cpython",
    headers={"Accept": "application/vnd.github.v3+json"},
    timeout=10   # Always set a timeout!
)

print(response.status_code)     # 200
print(response.headers["Content-Type"])
data = response.json()           # Parse JSON body
print(data["stargazers_count"])

# POST with JSON body
payload = {"hostname": "web-03", "ip": "10.0.0.3"}
resp = requests.post(
    "http://inventory-api/servers",
    json=payload,   # sets Content-Type: application/json automatically
    headers={"Authorization": "Bearer my-token"},
    timeout=10
)
resp.raise_for_status()   # Raises HTTPError for 4xx/5xx responses

# Session — reuses connections and persists headers/cookies
session = requests.Session()
session.headers.update({"Authorization": "Bearer my-token"})
session.timeout = 10

r1 = session.get("https://api.example.com/servers")
r2 = session.get("https://api.example.com/metrics")

# Retry with exponential backoff
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry_strategy = Retry(
    total=3,
    backoff_factor=2,   # waits 2, 4, 8 seconds between retries
    status_forcelist=[429, 500, 502, 503, 504]
)
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("https://", adapter)

# Exception handling
try:
    resp = session.get("https://api.example.com/health", timeout=5)
    resp.raise_for_status()
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e.response.status_code}")
except requests.exceptions.ConnectionError:
    print("Network unreachable")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

---

### `urllib` — Standard Library HTTP

`urllib` is Python's built-in HTTP library. It's more verbose than `requests` but requires no installation — useful in minimal environments (e.g., Lambda functions without layers, Docker images where you want zero dependencies):

```python
import urllib.request
import urllib.parse
import json

# Simple GET
with urllib.request.urlopen("https://httpbin.org/get", timeout=10) as response:
    body = response.read().decode("utf-8")
    data = json.loads(body)
    print(data["url"])

# POST with JSON
payload = json.dumps({"key": "value"}).encode("utf-8")
req = urllib.request.Request(
    "https://httpbin.org/post",
    data=payload,
    headers={"Content-Type": "application/json"},
    method="POST"
)
with urllib.request.urlopen(req, timeout=10) as resp:
    result = json.loads(resp.read().decode())
    print(result["json"])

# URL encoding for query parameters
params = {"search": "python devops", "page": 1}
query_string = urllib.parse.urlencode(params)
url = f"https://api.example.com/search?{query_string}"
# https://api.example.com/search?search=python+devops&page=1

# Error handling
from urllib.error import URLError, HTTPError
try:
    urllib.request.urlopen("https://nonexistent.example.com", timeout=5)
except HTTPError as e:
    print(f"HTTP {e.code}: {e.reason}")
except URLError as e:
    print(f"URL error: {e.reason}")
```

In general, use `requests` for application code and `urllib` only when you need zero external dependencies.
