# Python for DevOps — Days 1 & 2
## Foundations, Data Types, and Core Structures

---

## DAY 1 — Python Fundamentals

### What is Python?

Python is a **high-level, interpreted, general-purpose programming language** created by Guido van Rossum and first released in 1991. "High-level" means it abstracts away machine-level details like memory addresses and CPU registers, letting you think in terms of logic and data rather than hardware. "Interpreted" means Python source code is executed line-by-line at runtime by the Python interpreter — there is no separate compilation step that produces a binary.

Python's design philosophy is captured in the **Zen of Python** (`import this`): readability counts, explicit is better than implicit, simple is better than complex. Code indentation is not stylistic — it is syntactically meaningful. Indentation defines blocks (functions, loops, conditionals), replacing the braces `{}` used in languages like C or Java.

Python is:
- **Dynamically typed** — variable types are inferred at runtime; you don't declare `int x = 5`, just `x = 5`.
- **Garbage collected** — the interpreter automatically manages memory. Objects that are no longer referenced are freed, so you don't manually call `free()` as in C.
- **Multi-paradigm** — it supports procedural (step-by-step scripts), object-oriented (classes and objects), and functional (lambdas, `map`, `filter`, `reduce`) styles.

---

### Virtual Environments

#### Why Virtual Environments?

When working on multiple projects, each project may need different versions of the same library (e.g., Project A needs `requests==2.26`, Project B needs `requests==2.28`). Installing everything globally causes conflicts. A virtual environment creates an **isolated Python installation** for each project — its own interpreter, its own `site-packages` folder, entirely separate from the system Python.

#### `venv` — Built-in Virtual Environment Tool

`venv` is part of Python's standard library (Python 3.3+). It is the simplest and most common approach.

```bash
# Create a virtual environment in a folder called .venv
python3 -m venv .venv

# Activate it (Linux/macOS)
source .venv/bin/activate

# Activate it (Windows)
.venv\Scripts\activate

# Deactivate when done
deactivate
```

Once activated, the shell prompt shows `(.venv)` and `python`/`pip` commands point to the isolated environment.

#### `pyenv` — Python Version Manager

`pyenv` solves a different but related problem: managing **multiple Python versions** on the same machine (e.g., Python 3.9, 3.10, 3.11 simultaneously). It intercepts the `python` command using shims (small wrapper scripts) placed at the front of `$PATH`.

```bash
# Install a specific Python version
pyenv install 3.11.4

# Set it globally
pyenv global 3.11.4

# Set it for a specific project directory
pyenv local 3.10.12   # writes .python-version file

# List installed versions
pyenv versions
```

`pyenv` and `venv` are complementary: use `pyenv` to install and switch Python versions, then use `venv` (or `pyenv-virtualenv`) to create isolated environments for each project.

---

### `pip` — Package Installer for Python

`pip` is Python's standard package manager. It downloads packages from **PyPI** (Python Package Index, `pypi.org`).

```bash
# Install a package
pip install requests

# Install a specific version
pip install requests==2.28.0

# Install from a requirements file
pip install -r requirements.txt

# Upgrade a package
pip install --upgrade requests

# Uninstall
pip uninstall requests

# List installed packages
pip list

# Show details of a package
pip show requests

# Freeze current environment to a file
pip freeze > requirements.txt
```

`requirements.txt` is a plain text file listing packages and their pinned versions. It ensures reproducible environments:
```
requests==2.28.0
flask==2.3.2
boto3==1.26.0
```

---

### Scripts

A Python script is a `.py` file executed directly by the interpreter. The **shebang line** at the top tells the OS which interpreter to use when the file is run as an executable:

```python
#!/usr/bin/env python3
```

The **`if __name__ == "__main__"`** guard is a crucial pattern. When Python imports a file, it sets `__name__` to the module name. When it runs the file directly, it sets `__name__` to `"__main__"`. This guard ensures certain code only runs when the script is the entry point, not when it is imported:

```python
def main():
    print("Running as a script")

if __name__ == "__main__":
    main()
```

```bash
# Make executable and run
chmod +x script.py
./script.py

# Or run directly
python3 script.py
```

---

### Operators

#### Arithmetic Operators
```python
x = 10
y = 3

x + y    # 13  — addition
x - y    # 7   — subtraction
x * y    # 30  — multiplication
x / y    # 3.333... — true division (always float)
x // y   # 3   — floor division (integer result, rounded down)
x % y    # 1   — modulo (remainder)
x ** y   # 1000 — exponentiation
```

#### Comparison Operators
Return `True` or `False`.
```python
x == y   # equal to
x != y   # not equal to
x > y    # greater than
x < y    # less than
x >= y   # greater than or equal to
x <= y   # less than or equal to
```

#### Logical Operators
```python
True and False   # False — both must be True
True or False    # True  — at least one must be True
not True         # False — negation
```

Python evaluates logical expressions with **short-circuit evaluation**: in `A and B`, if `A` is False, `B` is never evaluated. In `A or B`, if `A` is True, `B` is never evaluated.

#### Assignment Operators
```python
x = 5
x += 3    # x = x + 3 → 8
x -= 2    # x = x - 2 → 6
x *= 4    # x = x * 4 → 24
x //= 5   # x = x // 5 → 4
```

#### Identity and Membership Operators
```python
a = [1, 2, 3]
b = a

b is a       # True  — same object in memory
b is not a   # False

1 in a       # True  — membership test
4 not in a   # True
```

`is` compares **object identity** (same memory address via `id()`), while `==` compares **value**. Never use `is` to compare strings or integers in general code.

#### Bitwise Operators
```python
5 & 3    # 1  — AND (0101 & 0011 = 0001)
5 | 3    # 7  — OR
5 ^ 3    # 6  — XOR
~5       # -6 — NOT (bitwise complement)
5 << 1   # 10 — left shift
5 >> 1   # 2  — right shift
```

---

### Conditions

The `if / elif / else` structure controls execution flow based on boolean expressions:

```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

print(f"Grade: {grade}")  # Grade: B
```

**Ternary (conditional) expression** — a one-line if/else:
```python
status = "pass" if score >= 50 else "fail"
```

**Truthy and Falsy values** — Python evaluates many things as True or False in a boolean context:
- Falsy: `None`, `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `False`
- Truthy: everything else

```python
name = ""
if name:
    print(f"Hello, {name}")
else:
    print("No name provided")  # This runs because "" is falsy
```

---

### `input()`

`input()` reads a line of text from standard input (keyboard) and returns it as a **string**. You must explicitly convert to other types:

```python
name = input("Enter your name: ")          # Always a string
age = int(input("Enter your age: "))       # Convert to int
price = float(input("Enter price: "))      # Convert to float

print(f"Hello {name}, you are {age} years old.")
```

In DevOps scripts, `input()` is used for interactive prompts (confirmation before destructive operations, etc.), though automated scripts often read from environment variables or config files instead.

---

## DAY 2 — Data Structures and Functions

### Loops

#### `for` Loop

Iterates over any **iterable** (list, string, range, dict, file, etc.):

```python
# Iterate over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# range(start, stop, step) — generates numbers
for i in range(0, 10, 2):  # 0, 2, 4, 6, 8
    print(i)

# enumerate — get index and value together
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}. {fruit}")

# zip — iterate over multiple iterables simultaneously
names = ["Alice", "Bob"]
scores = [95, 87]
for name, score in zip(names, scores):
    print(f"{name}: {score}")
```

#### `while` Loop

Repeats as long as a condition is True:

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

**`break`** — immediately exit the loop.  
**`continue`** — skip the rest of the current iteration, jump to the next.  
**`pass`** — a no-op placeholder (does nothing, used syntactically where a statement is required).

```python
for i in range(10):
    if i == 3:
        continue   # skip 3
    if i == 7:
        break      # stop at 7
    print(i)       # prints 0,1,2,4,5,6
```

#### `for/while` with `else`

Python loops can have an `else` clause. The `else` block runs **only if the loop completed normally** (not terminated by `break`). This is useful for search patterns:

```python
# Search for a value; else runs if not found
target = 7
for i in [1, 3, 5, 9]:
    if i == target:
        print("Found!")
        break
else:
    print("Not found")   # Runs because break was never hit

# while with else
n = 10
while n > 0:
    n -= 1
else:
    print("Loop finished normally")  # Runs
```

---

### Numbers

Python has three built-in numeric types:

- **`int`** — arbitrary precision integers. There's no overflow; Python handles big numbers automatically.
- **`float`** — 64-bit double-precision floating-point. Subject to rounding errors (`0.1 + 0.2 != 0.3`).
- **`complex`** — complex numbers (`3 + 4j`).

```python
x = 1_000_000    # underscores for readability
y = 0xFF         # hexadecimal → 255
z = 0b1010       # binary → 10
w = 0o17         # octal → 15

import math
math.sqrt(16)    # 4.0
math.ceil(4.2)   # 5
math.floor(4.9)  # 4
math.pi          # 3.14159...
abs(-5)          # 5
round(3.14159, 2)  # 3.14

# Decimal for precise financial arithmetic
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")  # 0.3 exactly
```

---

### Strings

Strings are **immutable** sequences of Unicode characters. They can be created with single, double, or triple quotes. Immutability means string operations always return a new string — the original is never modified.

```python
s = "Hello, DevOps World"

# Slicing: s[start:stop:step]
s[0:5]      # "Hello"
s[-5:]      # "World"
s[::2]      # every other character

# Common methods
s.upper()           # "HELLO, DEVOPS WORLD"
s.lower()           # "hello, devops world"
s.strip()           # remove leading/trailing whitespace
s.lstrip()          # remove left whitespace
s.rstrip()          # remove right whitespace
s.split(", ")       # ["Hello", "DevOps World"]
s.replace("World", "Engineer")  # new string with replacement
s.startswith("Hello")  # True
s.endswith("World")    # True
```

#### `find()` and `count()`

```python
s = "error: disk error: disk full"

# find() — returns index of first occurrence, or -1 if not found
s.find("error")      # 0
s.find("warning")    # -1
s.find("error", 5)   # 13 — start searching from index 5

# index() — like find() but raises ValueError if not found
s.index("error")     # 0

# count() — number of non-overlapping occurrences
s.count("error")     # 2
s.count("disk")      # 2
```

#### f-Strings (Formatted String Literals)

Introduced in Python 3.6, f-strings are the modern, preferred way to embed expressions in strings. They are faster and more readable than `%` formatting or `.format()`:

```python
name = "Alice"
cpu = 87.6
host = "prod-server-01"

# Basic interpolation
print(f"Hello, {name}!")

# Expressions inside {}
print(f"CPU usage: {cpu:.1f}%")     # one decimal place → 87.6%
print(f"Double: {cpu * 2}")
print(f"Upper: {name.upper()}")

# Alignment and padding
print(f"{'Server':<15} {'CPU':>10}")  # left-align, right-align
print(f"{host:<15} {cpu:>10.1f}")

# Debugging with = (Python 3.8+)
print(f"{cpu=}")   # prints: cpu=87.6
```

---

### Lists

Lists are **ordered, mutable** sequences. They can hold items of mixed types and grow/shrink dynamically.

```python
servers = ["web-01", "web-02", "db-01"]

# Indexing and slicing
servers[0]        # "web-01"
servers[-1]       # "db-01"
servers[0:2]      # ["web-01", "web-02"]

# Modifying
servers.append("cache-01")         # add to end
servers.insert(1, "web-03")        # insert at index
servers.extend(["api-01","api-02"])# append all from iterable
servers.remove("db-01")            # remove first matching value
popped = servers.pop()             # remove and return last element
popped = servers.pop(0)            # remove and return element at index

# Searching
servers.index("web-02")    # index of first match
"web-01" in servers        # True — membership test
servers.count("web-01")    # number of occurrences

# Sorting
servers.sort()                     # sort in-place (modifies list)
servers.sort(reverse=True)         # descending
sorted_copy = sorted(servers)      # returns new sorted list

# Other operations
servers.reverse()           # reverse in-place
len(servers)                # number of elements
servers.clear()             # remove all elements
copy = servers.copy()       # shallow copy

# List comprehension — concise way to create lists
ports = [80, 443, 22, 3306, 5432]
open_ports = [p for p in ports if p < 1024]   # [80, 443, 22]
squared = [x**2 for x in range(5)]            # [0, 1, 4, 9, 16]
```

---

### Tuples

Tuples are **ordered, immutable** sequences. Once created, elements cannot be added, removed, or changed. They are faster than lists and conventionally used for data that should not change (coordinates, RGB values, database records, function return values).

```python
server_info = ("web-01", "192.168.1.10", 80)

# Indexing works like lists
server_info[0]    # "web-01"
server_info[1:]   # ("192.168.1.10", 80)

# Unpacking — assign multiple variables at once
hostname, ip, port = server_info
print(f"Connecting to {hostname} at {ip}:{port}")

# Swap values elegantly (tuple unpacking)
a, b = 1, 2
a, b = b, a   # a=2, b=1

# Single-element tuple requires trailing comma
single = ("only",)    # without comma it's just a string in parens

# Named tuples — tuples with named fields (more readable)
from collections import namedtuple
Server = namedtuple("Server", ["hostname", "ip", "port"])
s = Server("web-01", "192.168.1.10", 80)
print(s.hostname)   # "web-01"
print(s.port)       # 80
```

Tuples are **hashable** (as long as their contents are hashable), so they can be used as dictionary keys or set members — lists cannot.

---

### Sets

Sets are **unordered collections of unique elements**. They automatically discard duplicates and are optimized for membership testing, intersection, union, and difference operations. Sets are mutable; `frozenset` is the immutable variant.

```python
active = {"web-01", "web-02", "db-01", "web-01"}  # duplicate removed
# active = {"web-01", "web-02", "db-01"}

maintenance = {"db-01", "cache-01"}

# Membership testing — O(1) average time
"web-01" in active     # True (much faster than list for large sets)

# Set operations
active | maintenance    # union: all unique elements from both
active & maintenance    # intersection: elements in both
active - maintenance    # difference: in active but not in maintenance
active ^ maintenance    # symmetric difference: in one but not both

# Methods
active.add("api-01")        # add one element
active.update(["a", "b"])   # add multiple elements
active.discard("web-99")    # remove if present, no error if missing
active.remove("web-01")     # remove; raises KeyError if missing
active.pop()                # remove and return an arbitrary element
len(active)                 # number of elements

# Set comprehension
unique_ports = {port % 100 for port in [80, 180, 443, 543, 22]}
```

---

### Dictionaries

Dictionaries are **unordered (Python 3.7+ maintains insertion order) key-value stores**. Keys must be hashable (strings, numbers, tuples); values can be anything. They are the most heavily used data structure in Python for configuration, JSON data, and caching.

```python
config = {
    "host": "prod-db.example.com",
    "port": 5432,
    "database": "app_db",
    "ssl": True
}

# Accessing values
config["host"]                  # "prod-db.example.com"
config.get("timeout", 30)       # returns 30 if "timeout" missing (safe)
config["missing"]               # KeyError — crashes

# Adding / updating
config["timeout"] = 60
config.update({"retries": 3, "pool_size": 10})

# Deleting
del config["ssl"]
removed = config.pop("timeout", None)   # returns value or None if missing

# Checking membership (checks keys)
"host" in config    # True
"ssl" in config     # False (we deleted it)

# Iterating
for key in config:              # iterates over keys
    print(key)

for key, value in config.items():  # key-value pairs
    print(f"{key}: {value}")

config.keys()       # dict_keys view
config.values()     # dict_values view
config.items()      # dict_items view (key, value tuples)

# Dictionary comprehension
env_vars = {"DB_HOST": "localhost", "DB_PORT": "5432", "APP_ENV": "prod"}
db_vars = {k: v for k, v in env_vars.items() if k.startswith("DB_")}
# {"DB_HOST": "localhost", "DB_PORT": "5432"}

# Nested dictionaries
servers = {
    "web-01": {"ip": "10.0.0.1", "role": "web"},
    "db-01":  {"ip": "10.0.0.5", "role": "database"},
}
servers["web-01"]["ip"]   # "10.0.0.1"

# Merging dicts (Python 3.9+)
defaults = {"timeout": 30, "retries": 3}
overrides = {"timeout": 60}
merged = defaults | overrides   # {"timeout": 60, "retries": 3}
```

---

### Functions

Functions are **named, reusable blocks of code** defined with `def`. They promote the DRY principle (Don't Repeat Yourself) and make code testable and modular.

```python
def greet(name, greeting="Hello"):
    """
    Greet a person by name.
    This is a docstring — it documents the function.
    """
    return f"{greeting}, {name}!"

greet("Alice")           # "Hello, Alice!"
greet("Bob", "Hi")       # "Hi, Bob!"
greet(greeting="Hey", name="Carol")  # keyword arguments — order doesn't matter
```

#### *args and **kwargs

`*args` collects extra positional arguments into a tuple.  
`**kwargs` collects extra keyword arguments into a dictionary.

```python
def deploy(*services, **options):
    print(f"Deploying: {services}")      # tuple
    print(f"Options: {options}")         # dict

deploy("web", "api", "worker", env="prod", rolling=True)
# Deploying: ('web', 'api', 'worker')
# Options: {'env': 'prod', 'rolling': True}
```

#### Return Values

Functions implicitly return `None` if no `return` statement is hit. They can return multiple values as a tuple:

```python
def get_server_stats(server):
    cpu = 72.5
    memory = 45.1
    return cpu, memory   # returns tuple (72.5, 45.1)

cpu, mem = get_server_stats("web-01")  # tuple unpacking
```

#### First-Class Functions

In Python, functions are objects. They can be assigned to variables, stored in lists, and passed as arguments:

```python
def square(x):
    return x ** 2

operations = [square, abs, str]
for op in operations:
    print(op(-4))   # 16, 4, "-4"
```
