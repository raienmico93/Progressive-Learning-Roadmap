# Python TOML: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

TOML (Tom’s Obvious, Minimal Language) is a configuration file format designed to be minimal, unambiguous, and easy to read, mapping directly to hash tables. Python provides built-in TOML parsing through the `tomllib` module (Python 3.11+), with the `tomli` package serving as a backport for older versions, and `tomli-w` providing the complementary write capability.

**Technical Definition**

TOML is specified by TOML v1.0.0 (released January 11, 2021) and is designed to map unambiguously to a hash table. It is case-sensitive, must be a valid UTF-8 encoded Unicode document, and uses a line-oriented syntax of key–value pairs, tables, arrays of tables, and inline tables. Python’s standard library `tomllib` module (added in Python 3.11 via PEP 680) provides a read-only interface for parsing TOML 1.0.0, returning `dict` objects. The `tomli` package is the reference implementation from which `tomllib` is derived; it remains available as a backport for Python versions without `tomllib` and offers optional mypyc-compiled wheels for improved performance. The `tomli-w` package provides the write counterpart, serializing Python mappings to TOML with a `tomllib`-compatible API.

**Beginner-Friendly Explanation**

TOML is a configuration format that looks like a clean, readable INI file—but with proper support for nested tables, arrays, dates, and numbers. Python 3.11 and later can read TOML files out of the box with `tomllib`. If you are on an older Python version, you use `tomli` instead (it works exactly the same). To write TOML files, you use `tomli-w`. Together, these tools let you read and write configuration files in a format that is both human-friendly and machine-precise.

### Key Characteristics

- **Read-only standard library**: `tomllib` parses TOML 1.0.0 and returns `dict`; it does not write TOML.
- **Backport compatibility**: `tomli` provides the same API as `tomllib` for Python < 3.11 and is also faster on Python 3.11+ when compiled with mypyc.
- **Write support via `tomli-w`**: The `tomli-w` package complements `tomllib`/`tomli` by serializing Python mappings to TOML.
- **Type mapping**: TOML types map cleanly to Python types (`str`, `int`, `float`, `bool`, `datetime`, `date`, `time`, `list`, `dict`).
- **Configurable float parsing**: Both `tomllib` and `tomli` accept `parse_float` for precision-sensitive numeric data (e.g., `decimal.Decimal`).
- **Security awareness**: Parsing untrusted TOML can consume significant CPU and memory; input size should be limited.
- **Dominant in Python packaging**: `pyproject.toml` is the standard configuration file for modern Python projects.

### Prerequisites

- Basic Python syntax: dictionaries, lists, functions, and file I/O.
- Familiarity with the `with` statement and exception handling.
- Awareness of Python version differences (3.11+ vs. older versions).
- Understanding of configuration file concepts.

### Related Programming Areas

- **Python packaging**: `pyproject.toml` is the standard for build systems, metadata, and tool configuration.
- **Configuration management**: TOML is used by tools like Black, Ruff, mypy, pytest, and Hatch.
- **Serialization alternatives**: JSON, YAML, and INI serve overlapping roles.
- **Rust and Go ecosystems**: Cargo and Go modules also use TOML for configuration.

---

## Core Concepts / Features

---

### 1. TOML (Tom’s Obvious, Minimal Language)

#### Definitions

**Core Definition**

TOML is a minimal configuration file format with clear semantics, designed to map unambiguously to a hash table and be easily parsed into data structures in any programming language.

**Technical Definition**

TOML v1.0.0 is defined by a specification authored by Tom Preston-Werner, Pradyun Gedam, and others. A TOML document consists of key–value pairs, tables (sections introduced by `[table]`), arrays of tables (introduced by `[[table]]`), and inline tables (`{ key = "value" }`). Keys can be bare (ASCII letters, digits, underscores, hyphens), quoted, or dotted (for nesting). Values include strings (basic, literal, and multi-line variants), integers, floats, booleans, offset date-times, local date-times, local dates, local times, arrays, and inline tables. TOML is case-sensitive, must be UTF-8, and uses `#` for comments. Whitespace means tab or space; newlines are LF or CRLF.

**Beginner-Friendly Explanation**

TOML looks like a clean INI file with superpowers. You write `key = value` pairs, group related settings under `[section]` headers, and use `[[section]]` for lists of sections. It supports strings, numbers, booleans, dates, and nested structures—all in a format that is easy to read and edit by hand. It is the format behind `pyproject.toml`, which almost every modern Python tool uses.

#### Purposes

- To provide a minimal, unambiguous, and human-readable configuration format.
- To map cleanly to hash tables and native data structures in any language.
- To serve as the standard configuration format for Python packaging (`pyproject.toml`) and tooling.
- To support a rich but simple type system including dates and times.
- To avoid the ambiguities and complexity of older formats like INI and YAML.

#### Syntax Rules and Structure

**Complete General Syntax**

```toml
# Comment
title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00

[database]
enabled = true
ports = [ 8000, 8001, 8002 ]
data = [ ["delta", "phi"], [3.14] ]
temp_targets = { cpu = 79.5, case = 72.0 }

[[products]]
name = "Hammer"
sku = 738594937

[[products]]
name = "Nail"
sku = 284758393
color = "gray"
```

**Component Breakdown**

| TOML Construct | Description | Python Equivalent |
|----------------|-------------|-------------------|
| Key–value pair | `key = value` | `dict` entry |
| Table | `[table]` section | nested `dict` |
| Array of tables | `[[table]]` repeated sections | `list` of `dict` |
| Inline table | `{ key = "value" }` | `dict` |
| Array | `[1, 2, 3]` | `list` |
| String | `"basic"`, `'literal'`, `"""multi-line"""` | `str` |
| Integer | `42` | `int` |
| Float | `3.14` | `float` |
| Boolean | `true`, `false` | `bool` |
| Date-time | `1979-05-27T07:32:00Z` | `datetime.datetime` |

**Syntax Rules**

- Keys and values must be on the same line (except multi-line strings and arrays).
- Bare keys may contain ASCII letters, digits, underscores, and hyphens.
- Dotted keys (`physical.color`) create nested tables.
- Tables cannot be redefined once defined.
- Arrays of tables (`[[products]]`) append new tables to a list.
- Comments begin with `#` and extend to the end of the line.
- Strings can be basic (`"..."`), literal (`'...'`), or multi-line (`"""..."""` or `'''...'''`).

**Constraints and Limitations**

- TOML does not support `null` or `None` values.
- Table definitions cannot be duplicated or redefined.
- TOML is case-sensitive.
- The format is intentionally minimal; complex nested structures can become verbose.
- `tomllib` does not preserve comments or formatting when parsing.

#### Annotated Code Examples

**Example 1: A Typical `pyproject.toml` Snippet**

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-package"
version = "0.1.0"
dependencies = [
    "requests>=2.28",
    "click>=8.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.pytest.ini_options]
testpaths = ["tests"]
```

**Expected Output (parsed by `tomllib`):**

```
{'build-system': {'requires': ['hatchling'], 'build-backend': 'hatchling.build'}, 'project': {'name': 'my-package', 'version': '0.1.0', 'dependencies': ['requests>=2.28', 'click>=8.0']}, 'tool': {'ruff': {'line-length': 88, 'target-version': 'py311'}, 'pytest': {'ini_options': {'testpaths': ['tests']}}}}
```

**Step-by-Step Breakdown:**

1. `[build-system]` defines the build backend and its requirements.
2. `[project]` contains package metadata and dependencies.
3. `[tool.ruff]` and `[tool.pytest.ini_options]` are tool-specific configuration tables.

**Why the Output is Produced:**

`tomllib` maps TOML tables to nested dictionaries, arrays to lists, and scalars to Python types according to the standard conversion table.

#### Real-World Cases

- **Python packaging**: `pyproject.toml` centralizes build configuration, metadata, and tool settings.
- **Tool configuration**: Ruff, Black, mypy, pytest, and Hatch all read TOML configuration.
- **Rust projects**: `Cargo.toml` is the standard manifest for Rust packages.
- **Static site generators**: Tools like Hugo and Zola use TOML for configuration.

---

### 2. `tomllib`

#### Definitions

**Core Definition**

`tomllib` is Python’s standard-library module, added in Python 3.11, for parsing TOML 1.0.0 documents into Python dictionaries.

**Technical Definition**

`tomllib` provides a read-only interface for parsing TOML 1.0.0. It defines two functions: `tomllib.load(fp, /, *, parse_float=float)`, which reads from a binary file object and returns a `dict`, and `tomllib.loads(s, /, *, parse_float=float)`, which parses a string and returns a `dict`. The module converts TOML types to Python types according to a standard conversion table: TOML strings become `str`, integers become `int`, floats become `float` (configurable via `parse_float`), booleans become `bool`, and date-time types become `datetime.datetime`, `datetime.date`, or `datetime.time`. Invalid TOML raises `tomllib.TOMLDecodeError`. The module does not support writing TOML.

**Beginner-Friendly Explanation**

`tomllib` is the built-in TOML reader that comes with Python 3.11 and later. You don’t need to install anything—just `import tomllib`. It has two functions: `load()` for files and `loads()` for strings. It gives you back a regular Python dictionary that you can use like any other. It is read-only; if you need to write TOML, you use `tomli-w`.

#### Purposes

- To parse TOML 1.0.0 documents using the Python standard library without third-party dependencies.
- To read `pyproject.toml` and other configuration files into Python dictionaries.
- To provide a secure, well-tested TOML parser as part of CPython.
- To support precision-sensitive float parsing via `parse_float`.
- To serve as the reference API that `tomli` mirrors for backward compatibility.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import tomllib

# Parse a file (binary mode required)
with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)

# Parse a string
data = tomllib.loads('key = "value"')

# Custom float parsing
from decimal import Decimal
data = tomllib.loads("price = 19.99", parse_float=Decimal)
```

**Component Breakdown**

| Function / Exception | Description |
|----------------------|-------------|
| `tomllib.load(fp)` | Reads TOML from a binary file object; returns `dict`. |
| `tomllib.loads(s)` | Parses TOML from a string; returns `dict`. |
| `parse_float` | Callable applied to every TOML float string. |
| `tomllib.TOMLDecodeError` | Raised on invalid TOML; subclass of `ValueError`. |

**Syntax Rules**

- `load()` requires a binary file object (opened with `"rb"`).
- `loads()` accepts a `str`.
- `parse_float` must not return a `dict` or `list`; doing so raises `ValueError`.
- Invalid TOML raises `TOMLDecodeError`.
- The module is read-only; no `dump` or `dumps` exists.

**Constraints and Limitations**

- `tomllib` does not write TOML; use `tomli-w` for serialization.
- It does not preserve comments or original formatting.
- Parsing untrusted TOML can consume significant CPU and memory; limit input size.
- Available only in Python 3.11+; older versions require `tomli`.

#### Annotated Code Examples

**Example 1: Parsing `pyproject.toml`**

```python
import tomllib

with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)

print(config["project"]["name"])
print(config["build-system"]["build-backend"])
```

**Expected Output:**

```
my-package
hatchling.build
```

**Step-by-Step Breakdown:**

1. The file is opened in binary mode, as required by `tomllib.load()`.
2. `tomllib.load()` parses the TOML and returns a nested dictionary.
3. Keys and values are accessed using standard dictionary syntax.

**Why the Output is Produced:**

`tomllib` maps TOML tables to dictionaries and scalars to Python types, making configuration data immediately accessible.

---

**Example 2: Precision-Sensitive Parsing**

```python
import tomllib
from decimal import Decimal

toml_str = "price = 19.99\nquantity = 3"
data = tomllib.loads(toml_str, parse_float=Decimal)

print(data["price"], type(data["price"]))
print(data["quantity"], type(data["quantity"]))
```

**Expected Output:**

```
19.99 <class 'decimal.Decimal'>
3 <class 'int'>
```

**Step-by-Step Breakdown:**

1. `parse_float=Decimal` converts all TOML floats to `Decimal`.
2. Integers remain `int` because `parse_float` applies only to floats.

**Why the Output is Produced:**

The `parse_float` parameter overrides the default `float` constructor, enabling precise decimal representation.

#### Real-World Cases

- **Reading `pyproject.toml`**: Parsing build configuration and tool settings without third-party dependencies.
- **Application configuration**: Loading `.toml` settings files in Python applications.
- **CI/CD tooling**: Reading TOML configuration in scripts running on Python 3.11+.

---

### 3. `tomli`

#### Definitions

**Core Definition**

`tomli` is a Python library for parsing TOML, fully compatible with TOML v1.0.0 (and v1.1.0 from version 2.4.0), serving as the reference implementation from which `tomllib` was derived and as a backport for Python versions without `tomllib`.

**Technical Definition**

Tomli is a pure-Python TOML parser with zero dependencies, fully compliant with TOML v1.0.0 and passing the entire `toml-lang/toml-test` suite with 100% branch coverage. A version of Tomli was added to the standard library as `tomllib` in Python 3.11 via PEP 680. Tomli remains available on PyPI for Python versions that lack `tomllib` and have not yet reached end-of-life. It uses mypyc to generate binary wheels for most platforms, making it faster than `tomllib` on Python 3.11+ when the compiled wheels are used. Its API mirrors `tomllib`: `load()`, `loads()`, `parse_float`, and `TOMLDecodeError`.

**Beginner-Friendly Explanation**

`tomli` is the library you use to read TOML if you are on Python 3.10 or older. It is also the library that Python’s built-in `tomllib` is based on. On Python 3.11+, you can use `tomli` instead of `tomllib` if you want a speed boost from compiled wheels, or you can stick with the standard library. The API is exactly the same, so switching between them is easy.

#### Purposes

- To provide TOML parsing on Python versions earlier than 3.11.
- To offer a faster alternative to `tomllib` via mypyc-compiled wheels on Python 3.11+.
- To serve as the reference implementation of TOML parsing in Python.
- To guarantee 100% spec compliance and thorough test coverage.
- To provide a seamless fallback for code that supports multiple Python versions.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import tomli

with open("config.toml", "rb") as f:
    data = tomli.load(f)

data = tomli.loads('key = "value"')

from decimal import Decimal
data = tomli.loads("price = 19.99", parse_float=Decimal)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `tomli.load(fp)` | Reads TOML from a binary file object. |
| `tomli.loads(s)` | Parses TOML from a string. |
| `parse_float` | Custom float constructor. |
| `tomli.TOMLDecodeError` | Invalid TOML exception. |

**Syntax Rules**

- The API is identical to `tomllib`.
- The file must be opened in binary mode (`"rb"`).
- `parse_float` behaves identically to `tomllib`.
- Version 2.4.0+ supports TOML v1.1.0; earlier versions support v1.0.0.

**Constraints and Limitations**

- `tomli` is read-only; it does not write TOML.
- On Python 3.11+, using `tomllib` is simpler unless the performance benefit of compiled `tomli` is needed.
- Tomli’s error messages are informational and may change between versions.

#### Annotated Code Examples

**Example 1: Version-Compatible Import Pattern**

```python
import sys

if sys.version_info >= (3, 11):
    import tomllib
else:
    import tomli as tomllib

with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)
```

**Expected Output:**

```
(Parsed configuration dictionary.)
```

**Step-by-Step Breakdown:**

1. The code checks the Python version.
2. On 3.11+, it imports `tomllib` from the standard library.
3. On older versions, it imports `tomli` as `tomllib`, creating a unified API.

**Why the Output is Produced:**

This pattern ensures the code works on all supported Python versions without changing the calling code.

#### Real-World Cases

- **Backport compatibility**: Libraries supporting Python 3.8–3.10 use `tomli` alongside `tomllib`.
- **Performance-sensitive tools**: Linters and build tools on Python 3.11+ can use compiled `tomli` wheels for speed.
- **Reference implementation**: `tomli` is used as a test oracle for TOML parsing behavior.

---

### 4. `tomli-w`

#### Definitions

**Core Definition**

`tomli-w` is a Python library for writing TOML, providing the write-only counterpart to `tomli`/`tomllib` with full compatibility with TOML v1.0.0.

**Technical Definition**

Tomli-W is a write-only TOML serializer that produces standards-compliant TOML 1.0.0 output. It exposes two public functions: `tomli_w.dumps(obj, *, multiline_strings=False, indent=4) -> str` and `tomli_w.dump(obj, fp, *, multiline_strings=False, indent=4) -> None`. The `dumps()` function returns a TOML string; `dump()` writes encoded TOML bytes to a binary file object. Tomli-W does not read TOML, does not sort the document, and does not preserve comments or insignificant whitespace. It supports `Decimal` values in round trips (since version 1.2.0) and can emit TOML multiline strings when `multiline_strings=True`.

**Beginner-Friendly Explanation**

`tomli-w` is how you write TOML from Python. You give it a dictionary, and it gives you TOML text or writes it to a file. It is the exact opposite of `tomli`/`tomllib`: they read TOML, and `tomli-w` writes it. It does not try to be clever—it does not sort your keys, and it does not preserve comments. It just produces clean, valid TOML that any TOML parser can read.

#### Purposes

- To serialize Python dictionaries to TOML 1.0.0 text or files.
- To provide a write API that mirrors the `marshal`/`pickle` style familiar from the standard library.
- To complement `tomllib`/`tomli` in read-write workflows.
- To support `Decimal` and date/datetime values in output.
- To generate configuration files programmatically.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import tomli_w

toml_text = tomli_w.dumps({"key": "value"})

with open("output.toml", "wb") as f:
    tomli_w.dump({"key": "value"}, f)
```

**Component Breakdown**

| Function | Description |
|----------|-------------|
| `dumps(obj)` | Returns a TOML string. |
| `dump(obj, fp)` | Writes TOML bytes to a binary file object. |
| `multiline_strings` | If `True`, emit triple-quoted strings for values with newlines. |
| `indent` | Number of spaces for array indentation (default 4). |

**Syntax Rules**

- `dump()` requires a binary file object (`"wb"`); it writes bytes.
- `dumps()` returns a `str`.
- Tomli-W does not sort keys; insertion order is preserved.
- Tomli-W does not preserve comments; it writes only data.
- `Decimal` values are written as TOML floats.

**Constraints and Limitations**

- Tomli-W does not read TOML; use `tomllib`/`tomli` for parsing.
- It does not support writing comments or custom formatting.
- It does not guarantee that output is byte-for-byte stable across versions.
- `multiline_strings=True` changes newline handling; leave it off when exact newline preservation matters.

#### Annotated Code Examples

**Example 1: Writing to a String**

```python
import tomli_w

doc = {
    "title": "example",
    "enabled": True,
    "ports": [8000, 8001],
    "database": {"host": "db.internal", "port": 5432},
}

toml_text = tomli_w.dumps(doc)
print(toml_text)
```

**Expected Output:**

```
title = "example"
enabled = true
ports = [
    8000,
    8001,
]

[database]
host = "db.internal"
port = 5432
```

**Step-by-Step Breakdown:**

1. `tomli_w.dumps()` serializes the dictionary to TOML.
2. Nested dictionaries become TOML tables.
3. Lists become TOML arrays with indentation.

**Why the Output is Produced:**

Tomli-W maps Python types to TOML types according to the TOML 1.0.0 specification, producing valid, readable output.

---

**Example 2: Writing to a File**

```python
import tomli_w

pyproject = {
    "project": {
        "name": "demo-app",
        "version": "0.1.0",
    }
}

with open("pyproject.generated.toml", "wb") as f:
    tomli_w.dump(pyproject, f)
```

**Expected Output:**

```
(A valid TOML file is created.)
```

**Step-by-Step Breakdown:**

1. The file is opened in binary mode (`"wb"`), as required by `dump()`.
2. `tomli_w.dump()` writes the TOML representation to the file.

**Why the Output is Produced:**

`dump()` encodes the TOML document as bytes and writes them to the binary file object.

#### Real-World Cases

- **Generating `pyproject.toml`**: Programmatically creating or updating project configuration.
- **Configuration file generation**: Writing application settings from Python dictionaries.
- **Test fixtures**: Producing TOML inputs for tests that use `tomllib`/`tomli`.
- **Data export**: Serializing structured data to TOML for other tools.

---

### 5. TOML Best Practices

#### Definitions

**Core Definition**

TOML best practices in Python are guidelines for securely parsing, correctly writing, and maintainably organizing TOML configuration, with emphasis on version compatibility, input validation, and the central role of `pyproject.toml`.

**Technical Definition**

Best practices encompass: (1) using `tomllib` on Python 3.11+ and `tomli` as a backport on older versions, with a version-checked import pattern; (2) using `parse_float=Decimal` when numeric precision matters; (3) limiting the size of untrusted TOML input to prevent resource exhaustion; (4) using `tomli-w` for writing with `multiline_strings` and `indent` set deliberately; (5) centralizing Python project configuration in `pyproject.toml` across `[build-system]`, `[project]`, `[dependency-groups]`, and `[tool.*]` tables; and (6) avoiding `null` values (TOML has no `null`) and duplicate table definitions. These practices align with the `pyproject.toml` specification maintained by the Python Packaging Authority.

**Beginner-Friendly Explanation**

Best practices are the “good habits” of TOML in Python. Use the built-in `tomllib` if you can, and fall back to `tomli` if you can’t. Use `Decimal` for money or other precise numbers. Don’t parse giant TOML files from strangers without a size limit. When writing, use `tomli-w` and think about whether you want multiline strings. And if you are setting up a Python project, put all your configuration in `pyproject.toml`—it is the modern standard.

#### Purposes

- To write TOML-handling code that works across Python versions.
- To ensure numeric precision and prevent silent data loss.
- To protect against resource-exhaustion attacks from malicious TOML.
- To produce clean, standards-compliant TOML output.
- To adopt the `pyproject.toml` convention for Python project configuration.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
import sys

if sys.version_info >= (3, 11):
    import tomllib
else:
    import tomli as tomllib

with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f, parse_float=Decimal)
```

**Component Breakdown**

| Practice | Description |
|----------|-------------|
| Version-checked import | Use `tomllib` on 3.11+, `tomli` otherwise. |
| `parse_float=Decimal` | Preserve precision for monetary or scientific values. |
| Input size limiting | Reject TOML larger than a reasonable threshold. |
| `tomli-w` for writing | Use `dumps()`/`dump()` with deliberate formatting options. |
| `pyproject.toml` centralization | Put build, project, dependency-group, and tool config in one file. |

**Syntax Rules**

- Always open TOML files in binary mode (`"rb"` for reading, `"wb"` for writing with `tomli-w`).
- Use `parse_float` when precision matters.
- Limit input size when parsing untrusted data.
- Use `multiline_strings=True` only when multiline output is desired.
- Keep `pyproject.toml` tables organized: `[build-system]`, `[project]`, `[dependency-groups]`, `[tool.*]`.

**Constraints and Limitations**

- `tomllib` is unavailable before Python 3.11; the fallback pattern is required for compatibility.
- TOML has no `null`; use empty strings or omit keys instead.
- `tomli-w` does not preserve comments; round-trip editing requires a style-preserving library like `tomlkit`.
- `pyproject.toml` is not universal; some legacy tools still expect `setup.cfg` or `setup.py`.

#### Annotated Code Examples

**Example 1: Cross-Version TOML Loading with Precision**

```python
import sys
from decimal import Decimal

if sys.version_info >= (3, 11):
    import tomllib
else:
    import tomli as tomllib

with open("config.toml", "rb") as f:
    config = tomllib.load(f, parse_float=Decimal)

print(config["price"], type(config["price"]))
```

**Expected Output:**

```
19.99 <class 'decimal.Decimal'>
```

**Step-by-Step Breakdown:**

1. The version check selects the appropriate TOML parser.
2. The file is opened in binary mode.
3. `parse_float=Decimal` ensures floats are decoded as `Decimal`.

**Why the Output is Produced:**

The unified API and explicit float parser make the code portable and precision-safe.

---

**Example 2: Generating `pyproject.toml` with `tomli-w`**

```python
import tomli_w

pyproject = {
    "build-system": {
        "requires": ["hatchling"],
        "build-backend": "hatchling.build",
    },
    "project": {
        "name": "demo",
        "version": "0.1.0",
        "dependencies": ["requests>=2.28"],
    },
}

with open("pyproject.toml", "wb") as f:
    tomli_w.dump(pyproject, f)
```

**Expected Output:**

```
(A valid pyproject.toml file.)
```

**Step-by-Step Breakdown:**

1. The dictionary mirrors the standard `pyproject.toml` structure.
2. `tomli_w.dump()` writes the file in binary mode.

**Why the Output is Produced:**

`tomli-w` serializes the nested dictionary into the corresponding TOML tables and key–value pairs.

#### Real-World Cases

- **Python packaging**: `pyproject.toml` centralizes build, metadata, and tool configuration.
- **Multi-version libraries**: Supporting Python 3.8–3.12 with a single TOML-reading codebase.
- **Financial applications**: Using `Decimal` for monetary values in TOML configuration.
- **Configuration generation**: Producing TOML for other tools from Python dictionaries.

---

## References

- Python Documentation – `tomllib` — Parse TOML files (Python 3.14) - https://docs.python.org/3.14/library/tomllib.html
- Python Documentation – `tomllib` (Python 3.11+) - https://docs.python.org/3/library/tomllib.html
- PyPI – Tomli: A lil’ TOML parser - https://pypi.org/project/tomli/
- PyPI – Tomli-W: A lil’ TOML writer - https://pypi.org/project/tomli-w/
- GitHub – Tomli-W Python Package Guide - https://github.com/andrewyng/context-hub/blob/67dcbeb2eb42c808549f08397920ad58be7c2206/content/tomli-w/docs/package/python/DOC.md
- TOML v1.0.0 Specification - https://toml.io/en/v1.0.0
- Python Packaging Authority – Writing your `pyproject.toml` - https://packaging.python.org/en/latest/guides/writing-pyproject-toml/
- Python Packaging Authority – `pyproject.toml` specification - https://packaging.python.org/en/latest/specifications/pyproject-toml/
- PEP 680 – `tomllib`: Support for parsing TOML in the Standard Library - https://peps.python.org/pep-0680/