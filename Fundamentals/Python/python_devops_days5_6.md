# Python for DevOps — Days 5 & 6
## Testing, Dockerizing, and File System Operations

---

## DAY 5 — Testing and Dockerizing Python Applications

### Why Testing Matters

In DevOps, software is deployed frequently, sometimes multiple times per day. Without automated tests, every deployment is a gamble. Testing provides a **safety net**: it catches regressions (previously working features broken by new code), enforces contracts between components, enables confident refactoring, and serves as living documentation of intended behavior.

**Key properties of good tests**:
- **Maintainability** — tests should be easy to read and update when requirements change. Brittle tests that break on unrelated changes become a burden.
- **Speed** — slow tests don't get run. Aim for a fast feedback loop.
- **Isolation** — each test should be independent. Order shouldn't matter.
- **Determinism** — same input always produces same result (no flakiness).

---

### Test Pyramid

The **test pyramid** describes the ideal proportion of different test types:

```
         /\
        /  \   System Tests (few, slow, expensive)
       /----\
      /      \ Integration Tests (some)
     /--------\
    /          \ Component Tests (more)
   /------------\
  /              \ Unit Tests (many, fast, cheap)
 /________________\
```

**Unit Tests** — test a single function or class in complete isolation. No network, no database, no filesystem. Fast (milliseconds). Should make up the majority of your test suite.

**Component Tests** — test a complete module or service boundary. May use a real database or file system but still isolated from external services.

**Integration Tests** — test how multiple components work together. May involve real external services (database, cache, message queue). Slower, more complex setup.

**System Tests (E2E)** — test the entire deployed application from a user's perspective. Slow, expensive, fragile. Used sparingly for critical user journeys.

**Manual Tests** — human-performed exploratory testing. Catches issues automated tests miss (UX problems, visual regressions) but doesn't scale.

---

### Unit Testing with `unittest`

`unittest` is Python's built-in testing framework, modeled after JUnit. Test cases are organized as methods in classes that inherit from `unittest.TestCase`.

```python
# calculator.py — the code under test
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def is_healthy_cpu(usage_percent):
    return 0 <= usage_percent <= 85
```

```python
# test_calculator.py
import unittest
from calculator import add, divide, is_healthy_cpu

class TestAdd(unittest.TestCase):

    def test_positive_numbers(self):
        result = add(2, 3)
        self.assertEqual(result, 5)   # most common assertion

    def test_negative_numbers(self):
        self.assertEqual(add(-1, -1), -2)

    def test_zero(self):
        self.assertEqual(add(0, 5), 5)


class TestDivide(unittest.TestCase):

    def test_normal_division(self):
        self.assertAlmostEqual(divide(10, 3), 3.333, places=2)

    def test_divide_by_zero_raises(self):
        with self.assertRaises(ValueError):
            divide(10, 0)

    def test_integer_result(self):
        self.assertEqual(divide(10, 2), 5)


class TestCPUHealth(unittest.TestCase):

    def test_normal_cpu(self):
        self.assertTrue(is_healthy_cpu(50))

    def test_high_but_ok_cpu(self):
        self.assertTrue(is_healthy_cpu(85))

    def test_over_threshold(self):
        self.assertFalse(is_healthy_cpu(90))

    def test_negative_cpu(self):
        self.assertFalse(is_healthy_cpu(-5))


if __name__ == "__main__":
    unittest.main()
```

**Key assertion methods**:

| Method | Checks |
|--------|--------|
| `assertEqual(a, b)` | `a == b` |
| `assertNotEqual(a, b)` | `a != b` |
| `assertTrue(x)` | `bool(x) is True` |
| `assertFalse(x)` | `bool(x) is False` |
| `assertIs(a, b)` | `a is b` |
| `assertIsNone(x)` | `x is None` |
| `assertIn(a, b)` | `a in b` |
| `assertRaises(Exc)` | context manager for expected exceptions |
| `assertAlmostEqual(a, b)` | floats equal to N decimal places |

#### `setUp` and `tearDown`

`setUp()` runs before each test method. `tearDown()` runs after. Use them to create and clean up shared test state:

```python
class TestServerManager(unittest.TestCase):

    def setUp(self):
        """Called before each test method."""
        self.manager = ServerManager()
        self.manager.add("web-01", "10.0.0.1")

    def tearDown(self):
        """Called after each test method."""
        self.manager.clear_all()

    def test_server_exists(self):
        self.assertIn("web-01", self.manager.list())

    def test_remove_server(self):
        self.manager.remove("web-01")
        self.assertNotIn("web-01", self.manager.list())
```

For class-level setup (runs once before/after ALL tests in class), use `setUpClass` / `tearDownClass` decorated with `@classmethod`.

---

### Mocking

**Mocking** is the practice of replacing real dependencies (network calls, database queries, file I/O, external APIs) with controlled fakes during testing. This ensures:
- Tests run fast (no real network round-trips)
- Tests are deterministic (real APIs can return different data or fail)
- Tests remain isolated (no side effects on real systems)

Python's `unittest.mock` module (also `from unittest.mock import ...`) is the standard tool.

```python
from unittest.mock import Mock, patch, MagicMock, call
import unittest

# Suppose we have this function that calls an external API
import requests

def get_server_status(hostname):
    """Fetch server status from monitoring API."""
    response = requests.get(f"https://monitor.internal/{hostname}/status")
    response.raise_for_status()
    data = response.json()
    return data["status"]
```

```python
class TestGetServerStatus(unittest.TestCase):

    @patch("requests.get")   # Replace requests.get with a Mock
    def test_returns_status(self, mock_get):
        # Configure what the mock returns
        mock_response = Mock()
        mock_response.json.return_value = {"status": "healthy", "cpu": 45}
        mock_response.raise_for_status.return_value = None
        mock_get.return_value = mock_response

        result = get_server_status("web-01")

        self.assertEqual(result, "healthy")
        # Verify the mock was called correctly
        mock_get.assert_called_once_with("https://monitor.internal/web-01/status")

    @patch("requests.get")
    def test_handles_http_error(self, mock_get):
        mock_response = Mock()
        mock_response.raise_for_status.side_effect = requests.HTTPError("404")
        mock_get.return_value = mock_response

        with self.assertRaises(requests.HTTPError):
            get_server_status("nonexistent-host")
```

#### `@patch` as Context Manager

```python
def test_with_context_manager():
    with patch("requests.get") as mock_get:
        mock_get.return_value.json.return_value = {"status": "ok"}
        result = get_server_status("web-01")
        assert result == "ok"
```

#### `MagicMock`

`MagicMock` is like `Mock` but also supports magic/dunder methods (`__len__`, `__iter__`, `__enter__`, `__exit__`, etc.):

```python
mock_file = MagicMock()
mock_file.__enter__.return_value = mock_file
mock_file.read.return_value = "server-01\nserver-02\n"

with patch("builtins.open", return_value=mock_file):
    # Test code that opens files
    ...
```

#### Mock Verification

```python
mock = Mock()
mock(1, 2, key="value")
mock(3, 4)

mock.assert_called()                        # Was called at least once
mock.assert_called_once()                   # Was called exactly once — FAILS here
mock.assert_called_with(3, 4)              # Last call had these args
mock.assert_any_call(1, 2, key="value")    # Any call had these args
mock.call_count                            # 2
mock.call_args_list                        # [call(1, 2, key='value'), call(3, 4)]
```

---

### `nose` + Coverage

**`nose2`** (successor to `nose`) is a test runner that discovers and runs tests automatically:

```bash
pip install nose2 coverage
```

```bash
# Run all tests (discovers test_*.py files automatically)
nose2

# Run with verbosity
nose2 -v

# Run specific test
nose2 test_calculator.TestDivide.test_divide_by_zero_raises
```

**`coverage`** measures which lines of code are executed during tests — **code coverage**. It helps identify untested code paths. 100% coverage doesn't guarantee correctness, but low coverage is a red flag.

```bash
# Run tests with coverage
coverage run -m nose2

# Or with unittest directly
coverage run -m unittest discover

# Generate a text report
coverage report -m

# Generate interactive HTML report
coverage html
# Open htmlcov/index.html in browser

# Fail if coverage drops below a threshold (useful in CI)
coverage report --fail-under=80
```

```
Name                  Stmts   Miss  Cover   Missing
------------------------------------------------------
calculator.py            12      2    83%   15-16
test_calculator.py       30      0   100%
------------------------------------------------------
TOTAL                    42      2    95%
```

Configure in `.coveragerc`:
```ini
[run]
source = .
omit = tests/*, venv/*, setup.py

[report]
fail_under = 80
show_missing = True
```

---

### `tox`

**`tox`** automates testing across multiple Python versions and environments. It creates isolated virtualenvs for each environment defined in `tox.ini`, installs dependencies, and runs tests. Essential for library authors and for ensuring compatibility.

```bash
pip install tox
```

**`tox.ini`**:
```ini
[tox]
envlist = py39, py310, py311, lint

[testenv]
deps =
    pytest
    requests
    coverage
commands =
    coverage run -m pytest {posargs}
    coverage report --fail-under=80

[testenv:lint]
deps =
    flake8
    black
    mypy
commands =
    flake8 src/
    black --check src/
    mypy src/
```

```bash
tox          # Run all environments
tox -e py311 # Run only py311 environment
tox -e lint  # Run only linting
```

In a CI/CD pipeline, `tox` ensures the codebase works across all supported Python versions before merging.

---

### Dockerizing Python Applications

Containerizing your Python application ensures it runs consistently across every environment — developer laptop, CI server, staging, and production.

#### Dockerfile for a Flask Application

```dockerfile
# Use an official Python base image
# "slim" variants are smaller — no build tools, no documentation
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1  # Don't write .pyc files
ENV PYTHONUNBUFFERED=1         # Don't buffer stdout/stderr (important for logs)

# Set working directory inside container
WORKDIR /app

# Install dependencies FIRST (before copying source code)
# This layer is cached as long as requirements.txt doesn't change
# Changing source code won't invalidate this layer
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application source code
COPY . .

# Create a non-root user for security
RUN adduser --disabled-password --gecos "" appuser
USER appuser

# Document that the app runs on port 5000
EXPOSE 5000

# Use Gunicorn (production WSGI server) instead of Flask dev server
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

#### `.dockerignore`

Prevent unnecessary files from being copied into the build context:

```
.git/
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
htmlcov/
*.egg-info/
.env
.env.*
tests/
*.md
```

#### `docker-compose.yml` for Development

```yaml
version: "3.9"

services:
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/appdb
    volumes:
      - .:/app   # Mount source for live reload in development
    depends_on:
      - db
    command: flask run --host=0.0.0.0 --debug

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

```bash
# Build and start all services
docker-compose up --build

# Run in background
docker-compose up -d

# Run tests inside the container
docker-compose run app python -m pytest

# Stop and remove containers
docker-compose down

# Also remove volumes (WARNING: deletes database data)
docker-compose down -v
```

#### Multi-stage Builds (for smaller production images)

```dockerfile
# Stage 1: Build stage — install build dependencies
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime stage — only copy what's needed
FROM python:3.11-slim AS runtime
WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
COPY . .

ENV PATH=/root/.local/bin:$PATH
USER nobody
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

The final image contains no build tools or intermediate layers — just the runtime essentials.

---

## DAY 6 — File System Operations

Python's `os`, `os.path`, `pathlib`, `shutil`, `glob`, `fnmatch`, and `tempfile` modules provide comprehensive file system manipulation capabilities — essential for automation, log processing, backups, and deployment scripts.

### The Modern Way: `pathlib`

`pathlib` (Python 3.4+) provides an object-oriented interface to the file system. It is generally preferred over the older `os.path` string-based approach because it is more readable and cross-platform:

```python
from pathlib import Path

# Create path objects
p = Path("/var/log/nginx")
home = Path.home()       # /home/username
cwd = Path.cwd()         # current working directory

# Path arithmetic with / operator
log_file = p / "access.log"           # /var/log/nginx/access.log
config = home / ".config" / "app.yaml"

# Properties
log_file.name        # "access.log"
log_file.stem        # "access"
log_file.suffix      # ".log"
log_file.parent      # Path('/var/log/nginx')
log_file.parts       # ('/', 'var', 'log', 'nginx', 'access.log')

# Checks
log_file.exists()    # True/False
log_file.is_file()   # True
log_file.is_dir()    # False
```

---

### Directory Listing

```python
import os
from pathlib import Path

# os approach
entries = os.listdir("/var/log")              # list of strings, no order guaranteed
entries_with_path = os.scandir("/var/log")    # iterator of DirEntry objects (faster)

for entry in os.scandir("/var/log"):
    if entry.is_file():
        print(f"{entry.name}: {entry.stat().st_size} bytes")

# pathlib approach (preferred)
log_dir = Path("/var/log")

# List all entries
for item in log_dir.iterdir():
    print(item)

# Filter: only files
files = [f for f in log_dir.iterdir() if f.is_file()]

# Filter: only directories
dirs = [d for d in log_dir.iterdir() if d.is_dir()]

# Sorted listing
sorted_files = sorted(log_dir.iterdir(), key=lambda p: p.stat().st_mtime)
```

---

### Making Directories

```python
import os
from pathlib import Path

# os approach
os.mkdir("/tmp/new_dir")                 # fails if parent doesn't exist
os.makedirs("/tmp/a/b/c")               # creates all parents
os.makedirs("/tmp/a/b/c", exist_ok=True) # doesn't fail if already exists

# pathlib approach
p = Path("/tmp/deployments/2024/01")
p.mkdir(parents=True, exist_ok=True)   # creates full path, no error if exists

# Create with specific permissions
p.mkdir(mode=0o755, parents=True, exist_ok=True)
```

---

### Filename Pattern Matching

#### `glob` — Shell-style wildcards

```python
import glob
from pathlib import Path

# os.glob — returns list of strings
log_files = glob.glob("/var/log/*.log")
all_python = glob.glob("/app/**/*.py", recursive=True)  # ** = any depth

# pathlib glob — returns Path objects (preferred)
log_dir = Path("/var/log")

# Match in current directory only
for f in log_dir.glob("*.log"):
    print(f)

# Match recursively at any depth
for f in log_dir.rglob("*.log"):     # equivalent to glob("**/*.log")
    print(f)

# Pattern examples
Path(".").glob("test_*.py")         # test_calculator.py, test_server.py
Path(".").glob("**/*.json")         # all JSON files, any depth
Path(".").glob("config.[yj]*")      # config.yaml, config.json
```

#### `fnmatch` — Filename matching without globbing

```python
import fnmatch

filenames = ["access.log", "error.log", "deploy.sh", "config.yaml", "README.md"]

# Filter matching names
log_files = fnmatch.filter(filenames, "*.log")   # ["access.log", "error.log"]

# Test a single name
fnmatch.fnmatch("access.log", "*.log")   # True
fnmatch.fnmatch("deploy.sh", "*.log")    # False

# Case-insensitive on case-sensitive systems
fnmatch.fnmatchcase("ACCESS.LOG", "*.log")  # False (exact case)
```

---

### Traversing Directory Trees

```python
import os
from pathlib import Path

# os.walk — yields (dirpath, dirnames, filenames) tuples
# Visits every subdirectory depth-first
for dirpath, dirnames, filenames in os.walk("/app"):
    # Modify dirnames in-place to prune traversal
    dirnames[:] = [d for d in dirnames if d != "__pycache__"]

    level = dirpath.replace("/app", "").count(os.sep)
    indent = "  " * level
    print(f"{indent}{os.path.basename(dirpath)}/")

    for filename in filenames:
        file_path = os.path.join(dirpath, filename)
        size = os.path.getsize(file_path)
        print(f"{indent}  {filename} ({size} bytes)")

# pathlib equivalent with rglob
for path in Path("/app").rglob("*"):
    if path.is_file() and "__pycache__" not in path.parts:
        print(path.relative_to("/app"))

# Find large log files
for log in Path("/var/log").rglob("*.log"):
    size_mb = log.stat().st_size / 1024 / 1024
    if size_mb > 100:
        print(f"Large file: {log} ({size_mb:.1f} MB)")
```

---

### Temporary Directories and Files

Temporary files and directories are automatically cleaned up and avoid name collisions — essential in concurrent scripts and test fixtures:

```python
import tempfile
from pathlib import Path

# Temporary file — deleted when closed (default)
with tempfile.NamedTemporaryFile(suffix=".json", mode="w", delete=False) as tf:
    tf.write('{"key": "value"}')
    temp_path = tf.name     # e.g., /tmp/tmpXXXXXX.json
# File still exists here (delete=False)
Path(temp_path).unlink()   # Now delete manually

# Temporary file — auto-deleted on close
with tempfile.TemporaryFile(mode="w+") as tf:
    tf.write("temporary data")
    tf.seek(0)
    print(tf.read())
# File is deleted automatically when the with block exits

# Temporary directory — all contents deleted when context exits
with tempfile.TemporaryDirectory() as tmpdir:
    tmp = Path(tmpdir)
    config = tmp / "config.yaml"
    config.write_text("host: localhost\nport: 5432\n")
    # Use the temporary directory for processing
    print(list(tmp.iterdir()))
# Entire directory tree deleted here — even if exceptions occurred

# Create temp directory in a specific location
with tempfile.TemporaryDirectory(dir="/data/processing", prefix="job_") as tmpdir:
    print(tmpdir)   # e.g., /data/processing/job_XXXXXXXX
```

---

### Deleting Files and Directories

```python
import os
import shutil
from pathlib import Path

# Delete a single file
os.remove("/tmp/old_file.log")       # raises FileNotFoundError if missing
Path("/tmp/old_file.log").unlink()   # same, pathlib style
Path("/tmp/old_file.log").unlink(missing_ok=True)  # Python 3.8+, no error if missing

# Delete an empty directory
os.rmdir("/tmp/empty_dir")
Path("/tmp/empty_dir").rmdir()

# Delete a non-empty directory tree (DESTRUCTIVE — no recycle bin!)
shutil.rmtree("/tmp/deploy_artifacts")

# Safe delete with error handling
def safe_rmtree(path):
    path = Path(path)
    if not path.exists():
        return
    if path.is_file():
        path.unlink()
    elif path.is_dir():
        shutil.rmtree(path)

# Delete files matching a pattern (e.g., rotate logs)
log_dir = Path("/var/log/app")
for old_log in log_dir.glob("*.log.2024*"):
    old_log.unlink()
    print(f"Deleted: {old_log.name}")
```

**Warning**: `shutil.rmtree` is permanent and immediate. Always confirm the path is correct, especially when working with variables. Consider implementing a dry-run mode in automation scripts.

---

### Copying and Moving Files

```python
import shutil
from pathlib import Path

# Copy a file (metadata not preserved by default)
shutil.copy("config.yaml", "/backup/config.yaml")             # copies content + permissions
shutil.copy2("config.yaml", "/backup/config.yaml")            # copies content + all metadata

# Copy to a directory (filename preserved)
shutil.copy("config.yaml", "/backup/")    # creates /backup/config.yaml

# Copy an entire directory tree
shutil.copytree("./app", "/backup/app")                       # fails if dest exists
shutil.copytree("./app", "/backup/app", dirs_exist_ok=True)  # Python 3.8+ OK if exists

# Copy with filtering (exclude __pycache__, .pyc files)
def ignore_compiled(dir, files):
    return [f for f in files if f.endswith(".pyc") or f == "__pycache__"]

shutil.copytree("./app", "/backup/app", ignore=ignore_compiled)

# pathlib read/write for simple file copies
src = Path("config.yaml")
dst = Path("/backup/config.yaml")
dst.write_bytes(src.read_bytes())   # binary copy
dst.write_text(src.read_text())     # text copy (respects encoding)
```

---

### Moving and Renaming

```python
import os
import shutil
from pathlib import Path

# Move a file or directory (works across filesystems)
shutil.move("old_location/file.log", "new_location/file.log")
shutil.move("old_dir/", "archive/old_dir/")

# Rename within the same filesystem (atomic operation)
os.rename("deploy.log", "deploy.log.bak")
Path("deploy.log").rename("deploy.log.bak")

# replace() — like rename() but overwrites destination if it exists
Path("new_config.yaml").replace("config.yaml")

# Batch rename — add timestamp prefix to log files
import time
log_dir = Path("/var/log/app")
timestamp = time.strftime("%Y%m%d")

for log in log_dir.glob("*.log"):
    new_name = log.with_name(f"{timestamp}_{log.name}")
    log.rename(new_name)
    print(f"Renamed: {log.name} → {new_name.name}")
```

`os.rename()` is atomic on most filesystems (no partial state if it fails), but only works within the same filesystem/device. `shutil.move()` works across filesystems by copying then deleting, so it's not atomic.

---

### Archiving and Compression

**`shutil` — high-level archiving**:

```python
import shutil

# Create archives
shutil.make_archive(
    base_name="backup_2024",    # output filename without extension
    format="gztar",             # formats: zip, tar, gztar, bztar, xztar
    root_dir="/app",            # directory to archive
    base_dir="."                # base directory inside archive
)
# Creates: backup_2024.tar.gz

# Extract archives
shutil.unpack_archive("backup_2024.tar.gz", "/restore/")
shutil.unpack_archive("release.zip", "/opt/app/")

# List supported formats
shutil.get_archive_formats()    # [('bztar', ...), ('gztar', ...), ('tar', ...), ('zip', ...)]
```

**`zipfile` — fine-grained ZIP control**:

```python
import zipfile
from pathlib import Path

# Create a ZIP with selective content
with zipfile.ZipFile("deployment.zip", "w", compression=zipfile.ZIP_DEFLATED) as zf:
    app_dir = Path("./app")
    for file in app_dir.rglob("*"):
        if file.is_file() and "__pycache__" not in file.parts:
            # arcname controls path inside the archive
            zf.write(file, arcname=file.relative_to(app_dir.parent))

# Read and inspect a ZIP
with zipfile.ZipFile("deployment.zip", "r") as zf:
    print(zf.namelist())                   # list all files inside
    info = zf.getinfo("app/config.yaml")   # metadata about a file
    print(f"Compressed: {info.compress_size} bytes")
    content = zf.read("app/config.yaml")  # read without extracting
    zf.extractall("/opt/release/")         # extract everything

# Check if a file is a valid ZIP
zipfile.is_zipfile("deployment.zip")       # True/False
```

**`tarfile` — fine-grained TAR control**:

```python
import tarfile

# Create .tar.gz
with tarfile.open("backup.tar.gz", "w:gz") as tar:
    tar.add("/var/log/app", arcname="logs")   # arcname = path inside archive
    tar.add("config.yaml", arcname="config/config.yaml")

# Modes: "w" (tar), "w:gz" (gzip), "w:bz2" (bzip2), "w:xz" (xz)
#        "r" (auto), "r:gz", "r:bz2", "r:xz"
#        "a" (append, only for uncompressed tar)

# Read a tar archive
with tarfile.open("backup.tar.gz", "r:gz") as tar:
    print(tar.getnames())              # list all file paths
    tar.extractall("/restore/")        # extract all
    tar.extract("logs/access.log", "/tmp/")  # extract single file

# Safe extraction — prevent path traversal attacks (Python 3.12+)
with tarfile.open("untrusted.tar.gz", "r:gz") as tar:
    tar.extractall("/safe/dir/", filter="data")  # "data" filter prevents escaping
```

**`gzip` — compress individual files**:

```python
import gzip
import shutil
from pathlib import Path

# Compress a file
with open("access.log", "rb") as f_in:
    with gzip.open("access.log.gz", "wb") as f_out:
        shutil.copyfileobj(f_in, f_out)

# Decompress
with gzip.open("access.log.gz", "rt") as f:  # "rt" = read text
    for line in f:
        print(line.strip())

# Read gzipped file directly
with gzip.open("data.json.gz", "rt", encoding="utf-8") as f:
    import json
    data = json.load(f)
```

---

### File Metadata and Permissions

```python
import os
import stat
from pathlib import Path

p = Path("/etc/nginx/nginx.conf")

# File stats
st = p.stat()
st.st_size          # size in bytes
st.st_mtime         # last modified time (Unix timestamp)
st.st_atime         # last accessed time
st.st_ctime         # last changed (metadata) time

import datetime
modified = datetime.datetime.fromtimestamp(st.st_mtime)
print(f"Last modified: {modified:%Y-%m-%d %H:%M:%S}")

# Permissions
print(oct(st.st_mode))   # e.g., 0o100644
p.chmod(0o644)           # rw-r--r--
p.chmod(0o755)           # rwxr-xr-x

# Check specific permissions using stat constants
is_executable = bool(st.st_mode & stat.S_IXUSR)

# Change owner (requires root)
os.chown(str(p), uid=1000, gid=1000)

# Check file type
stat.S_ISREG(st.st_mode)   # regular file?
stat.S_ISDIR(st.st_mode)   # directory?
stat.S_ISLNK(st.st_mode)   # symlink?
```

---

### Practical DevOps File Operations — Putting It Together

```python
#!/usr/bin/env python3
"""
Log rotation script — archives logs older than N days.
"""

import shutil
import logging
from pathlib import Path
from datetime import datetime, timedelta

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s: %(message)s")
logger = logging.getLogger(__name__)

def rotate_logs(log_dir: str, archive_dir: str, days_old: int = 7, dry_run: bool = False):
    """
    Move log files older than `days_old` to `archive_dir` and compress them.
    """
    log_path = Path(log_dir)
    archive_path = Path(archive_dir)
    cutoff = datetime.now() - timedelta(days=days_old)

    archive_path.mkdir(parents=True, exist_ok=True)

    moved = 0
    total_size = 0

    for log_file in log_path.glob("*.log"):
        modified_time = datetime.fromtimestamp(log_file.stat().st_mtime)

        if modified_time < cutoff:
            size_mb = log_file.stat().st_size / 1024 / 1024
            dest = archive_path / f"{log_file.stem}_{modified_time:%Y%m%d}.log.gz"

            logger.info("Archiving: %s (%.1f MB, modified %s)",
                        log_file.name, size_mb, modified_time.date())

            if not dry_run:
                import gzip
                with open(log_file, "rb") as f_in:
                    with gzip.open(dest, "wb") as f_out:
                        shutil.copyfileobj(f_in, f_out)
                log_file.unlink()

            moved += 1
            total_size += log_file.stat().st_size

    logger.info("Done. Archived %d files (%.1f MB total). dry_run=%s",
                moved, total_size / 1024 / 1024, dry_run)

if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser(description="Log rotation utility")
    parser.add_argument("--log-dir", default="/var/log/app")
    parser.add_argument("--archive-dir", default="/archive/logs")
    parser.add_argument("--days", type=int, default=7)
    parser.add_argument("--dry-run", action="store_true")
    args = parser.parse_args()

    rotate_logs(args.log_dir, args.archive_dir, args.days, args.dry_run)
```
