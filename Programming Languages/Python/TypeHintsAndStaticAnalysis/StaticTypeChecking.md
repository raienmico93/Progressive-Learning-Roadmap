# Static Type Checking in Python: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Static type checking is the process of analyzing Python source code, without executing it, to verify that values and expressions conform to their declared type annotations. It is performed by external tools called type checkers, which operate on the principle of gradual typing — allowing programmers to opt in to static checking at any level of granularity.

### Technical Definition

Static type checking is the static analysis of a program's type correctness according to a formal type system. In Python's gradual type system, the special type `Any` represents statically unknown types, and operations on `Any` values are deferred to runtime dynamic checking. A statically typed program is one in which a type checker can verify that all expressions are well typed before execution; a gradually typed program mixes statically checked and dynamically checked regions, with `Any` serving as the boundary.

### Beginner-Friendly Explanation

Static type checking is like having a very careful assistant read through your code before you run it, checking that you're using variables and functions the way you said you would. If you told Python "this function returns a number" but then return a string, the type checker will tell you about the mistake before your program ever runs. You don't have to annotate everything — you can add type hints gradually, one function at a time.

### Key Characteristics

- **Gradual**: Type checking is optional and granular; unannotated code is treated as dynamically typed (implicitly `Any`).
- **Static**: Analysis happens before runtime; no execution is required.
- **Tool-dependent**: Python itself does not enforce type annotations at runtime; type checkers (mypy, Pyright, Pyrefly) perform the checking.
- **Non-invasive**: Type hints do not affect runtime behavior.
- **Ecosystem-driven**: Type checkers, IDE integrations (Pylance), and runtime validators (Pydantic, Typeguard) form a cohesive toolchain.

### Prerequisites

- Python 3.8 or newer (some features require 3.10+, 3.11+, or 3.12+).
- Basic familiarity with Python type annotations (`int`, `str`, `list[int]`, `Optional`).
- A type checker installed (`mypy`, `pyright`, or equivalent).
- A code editor with type-checking integration (optional but recommended).

### Related Programming Areas

- **Static analysis and linting**: Ruff, Flake8, Pylint.
- **Data validation and serialization**: Pydantic, attrs, dataclasses.
- **Web frameworks**: FastAPI, Django (with django-stubs), Litestar.
- **CI/CD pipelines**: GitHub Actions, GitLab CI, pre-commit.
- **IDE tooling**: VS Code (Pylance), PyCharm, Neovim (with LSP).

### Core Concepts / Features

The following sections cover each core concept using a uniform structure.

---

## 1. mypy (Features, Configuration via `pyproject.toml`, and Strict Mode)

### Definitions

**Core Definition**: mypy is the original static type checker for Python, created by Jukka Lehtosalo in 2012 and maintained by the Python Software Foundation. It is the reference implementation against which PEP 484 was defined.

**Technical Definition**: mypy is a Python program that performs static type analysis on annotated Python code. It uses the `typing` module's constructs and a system of type stubs (`.pyi` files) to verify type correctness. It supports incremental checking via a daemon (`dmypy`), plugin systems for third-party libraries, and per-module configuration overrides.

**Beginner-Friendly Explanation**: mypy is a tool you run on your Python code that tells you if you've made type mistakes. It's written in Python, has been around the longest, and has a large ecosystem of plugins. You configure it in a file called `pyproject.toml`.

### Purposes

- To catch type errors before runtime, reducing bugs in production.
- To serve as the reference implementation for Python's type system specification.
- To provide a plugin ecosystem for libraries with dynamic typing (Django, SQLAlchemy, Pydantic).
- To support incremental type checking for large codebases.
- To enforce strict type discipline in new projects via `--strict` mode.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: `pyproject.toml` Configuration (Recommended)**

```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_ignores = true
show_error_codes = true
show_error_context = true
files = ["src", "tests"]
exclude = ["migrations/", "venv/", "__pycache__/"]

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false

[[tool.mypy.overrides]]
module = ["requests.*", "boto3.*", "botocore.*"]
ignore_missing_imports = true
```

**Component Breakdown**:
- `[tool.mypy]` — TOML table for mypy configuration.
- `python_version` — target Python version for type checking.
- `strict = true` — enables all strict-mode checks (see below).
- `files` — list of paths to type-check.
- `exclude` — paths to skip.
- `[[tool.mypy.overrides]]` — per-module configuration overrides.

**Syntax 2: `mypy.ini` (Alternative)**

```ini
[mypy]
python_version = 3.11
strict = True
warn_return_any = True
warn_unused_ignores = True
show_error_codes = True

[mypy-tests.*]
disallow_untyped_defs = False

[mypy-requests.*]
ignore_missing_imports = True
```

**Syntax 3: Command-Line Usage**

```bash
mypy src/
mypy --strict src/
mypy --python-version 3.11 src/
```

#### Syntax Rules

1. **Configuration precedence**: Command-line flags override `pyproject.toml` settings, which override `mypy.ini` settings.
2. **`strict = true` is a meta-flag**: It enables a set of individual strict checks (listed below).
3. **Per-module overrides**: Use `[[tool.mypy.overrides]]` in TOML or `[mypy-module.*]` in INI.
4. **File inclusion**: mypy checks only files that are annotated or explicitly included; unannotated files are checked for syntax but not type errors.
5. **Incremental mode**: Use `dmypy` for daemon-based incremental checking.

#### Strict Mode Components

When `strict = true` is set, mypy enables the following individual checks:

| Flag | Effect |
|------|--------|
| `warn_unused_configs` | Warns about unused configuration sections |
| `disallow_any_generics` | Disallows `list` without type parameters |
| `disallow_subclassing_any` | Disallows subclassing `Any` |
| `disallow_untyped_calls` | Disallows calling untyped functions |
| `disallow_untyped_defs` | Disallows functions without type annotations |
| `disallow_incomplete_defs` | Disallows partially annotated functions |
| `check_untyped_defs` | Type-checks the bodies of unannotated functions |
| `disallow_untyped_decorators` | Disallows untyped decorators |
| `warn_redundant_casts` | Warns about unnecessary casts |
| `warn_unused_ignores` | Warns about unused `# type: ignore` comments |
| `warn_return_any` | Warns when returning `Any` from a typed function |
| `no_implicit_reexport` | Disallows implicit re-export of imported names |
| `strict_equality` | Enables strict equality checks |
| `extra_checks` | Enables additional checks |

#### Commonly Adjusted Flags

- `disallow_untyped_defs = false` for test files (`[[tool.mypy.overrides]] module = "tests.*"`).
- `ignore_missing_imports = true` for third-party libraries without stubs.
- `show_error_context = true` to display the source line where the error occurred.
- `show_error_codes = true` to display error codes (useful for `# type: ignore[code]`).
- `pretty = true` for more readable error output.

#### Constraints and Limitations

- mypy is slower than Pyright on large codebases because it is implemented in Python.
- It is permissive by default; `--strict` is required to catch many type errors.
- Some advanced typing features (e.g., PEP 695) require mypy 1.10+.
- Plugin support is required for libraries that use dynamic typing patterns.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic mypy Check

```python
# file: example.py
def add(a: int, b: int) -> int:
    return a + b

result: str = add(1, 2)  # type error: int is not str
print(result)
```

**Expected mypy Output**:
```
example.py:4: error: Incompatible types in assignment (expression has type "int", variable has type "str")
Found 1 error in 1 file (checked 1 source file)
```

**Why**: `add` returns an `int`, but `result` is annotated as `str`. mypy catches this before runtime.

#### Example 2: Strict Mode Catches Unannotated Functions

```python
# file: strict_example.py
def greet(name):
    return f"Hello, {name}"

print(greet("Alice"))
```

**Expected mypy Output (with `strict = true`)**:
```
strict_example.py:1: error: Function is missing a type annotation
Found 1 error in 1 file (checked 1 source file)
```

**Why**: In strict mode, `disallow_untyped_defs` requires all functions to have type annotations.

#### Example 3: Per-Module Override for Tests

```toml
# pyproject.toml
[tool.mypy]
strict = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

```python
# tests/test_example.py
def test_something():
    assert 1 + 1 == 2  # No type annotation required for tests
```

**Expected mypy Output**:
```
Success: no issues found in 1 source file
```

**Why**: The override disables `disallow_untyped_defs` for test files, allowing unannotated test functions.

### Real-World Cases

- **Large codebases**: mypy's incremental daemon (`dmypy`) is used in CI to check only changed files.
- **Library development**: mypy is the reference checker for PEP 484 conformance.
- **Django projects**: `django-stubs` provides type stubs and a mypy plugin.
- **SQLAlchemy projects**: `sqlalchemy[mypy]` provides a plugin for ORM type inference.

### References

- mypy Documentation - https://mypy.readthedocs.io/
- mypy Configuration File - https://mypy.readthedocs.io/en/stable/config_file.html
- mypy Strict Mode - https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-strict
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/

---

## 2. Pyright / Pylance (Features and Performance Comparison)

### Definitions

**Core Definition**: Pyright is a static type checker for Python developed by Microsoft, written in TypeScript. It is the engine behind Pylance, the default Python language server in Visual Studio Code.

**Technical Definition**: Pyright performs static type analysis using a TypeScript-based implementation, leveraging the V8 JavaScript engine for speed. It supports the full Python typing specification, including PEP 484, 526, 544, 586, 589, 591, 593, 604, 612, 613, 647, 655, 673, and 695. Pylance is the VS Code extension that wraps Pyright and adds additional features such as auto-imports, semantic highlighting, and Jupyter notebook support.

**Beginner-Friendly Explanation**: Pyright is a very fast type checker made by Microsoft. It powers the type checking you see in VS Code when you use the Python extension. It's written in TypeScript (a faster language than Python), so it runs much faster than mypy on large projects.

### Purposes

- To provide fast, accurate static type checking for Python code.
- To power IDE features such as autocompletion, go-to-definition, and error highlighting.
- To lead the Python typing-spec conformance suite, ensuring correctness on advanced typing features.
- To offer strict-by-default checking without requiring extensive configuration.
- To integrate seamlessly with VS Code via the Pylance extension.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: `pyrightconfig.json`**

```json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__", "venv"],
  "pythonVersion": "3.11",
  "pythonPlatform": "All",
  "typeCheckingMode": "strict",
  "reportMissingImports": true,
  "reportMissingTypeStubs": false,
  "reportUnusedImport": true,
  "reportUnusedClass": true,
  "reportUnusedFunction": true,
  "reportUnusedVariable": true,
  "reportDuplicateImport": true,
  "reportPrivateUsage": true,
  "reportConstantRedefinition": true,
  "reportIncompatibleMethodOverride": true,
  "reportIncompatibleVariableOverride": true,
  "reportInconsistentConstructor": true,
  "reportOverlappingOverload": true,
  "reportUninitializedInstanceVariable": true
}
```

**Syntax 2: `pyproject.toml` (Pyright)**

```toml
[tool.pyright]
include = ["src"]
exclude = ["**/node_modules", "**/__pycache__", "venv"]
pythonVersion = "3.11"
typeCheckingMode = "strict"
reportMissingTypeStubs = false
```

**Syntax 3: Command-Line Usage**

```bash
pyright src/
pyright --pythonversion 3.11 src/
npm install -g pyright
```

#### Type Checking Modes

Pyright supports four type checking modes:

| Mode | Description |
|------|-------------|
| `off` | No checking |
| `basic` | Basic checks (default) |
| `standard` | Standard checks |
| `strict` | Strict checks (all rules enabled) |

#### Syntax Rules

1. **Configuration precedence**: `pyrightconfig.json` takes precedence over `pyproject.toml`.
2. **`typeCheckingMode`**: Controls the overall strictness level.
3. **Per-rule configuration**: Individual rules can be overridden with `"reportRuleName": true/false`.
4. **Node.js required**: Command-line usage requires Node.js installed.
5. **Pylance is a VS Code extension**: It wraps Pyright and adds IDE-specific features.

#### Performance Comparison: mypy vs. Pyright

Based on official typing-spec conformance benchmarks and community reports:

| Dimension | mypy | Pyright |
|-----------|------|---------|
| **Speed (cold)** | Moderate (Python) | 3–5× faster than mypy |
| **Speed (incremental)** | Fast with `dmypy` | Fast (built-in) |
| **Typing-spec conformance** | ~75% | ~95% |
| **IDE integration** | Via LSP plugins | Native Pylance in VS Code |
| **Configuration** | `pyproject.toml`, `mypy.ini` | `pyrightconfig.json`, `pyproject.toml` |
| **Implementation language** | Python | TypeScript (Node.js) |
| **Strict by default** | No (`--strict` required) | Yes (in `strict` mode) |
| **Plugin ecosystem** | Rich (Django, SQLAlchemy, Pydantic) | Limited |

**Key insight**: Pyright is significantly faster than mypy on large codebases because it uses incremental analysis and is written in TypeScript with a focus on performance. Pyright currently leads the typing-spec conformance suite at approximately 95%, while mypy is at approximately 75%. The common pattern in 2026 is to use Pyright via Pylance in the IDE and mypy or Pyrefly in CI for the authoritative gate.

#### Constraints and Limitations

- Pyright requires Node.js for command-line use.
- Error messages are occasionally cryptic compared to mypy.
- Not Python-native; some developers prefer pure-Python tooling.
- Plugin ecosystem is less mature than mypy's.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Pyright Strict Mode

```python
# file: example.py
def add(a: int, b: int) -> int:
    return a + b

result: str = add(1, 2)
```

**Expected Pyright Output**:
```
example.py:4:9 - error: Type "int" is not assignable to declared type "str"
    int is not assignable to str
```

**Why**: Pyright in strict mode catches the same type error as mypy, but with a different message format.

#### Example 2: Pyright with `pyproject.toml`

```toml
[tool.pyright]
include = ["src"]
typeCheckingMode = "strict"
reportMissingTypeStubs = false
```

```bash
pyright
```

**Expected Output**:
```
0 errors, 0 warnings, 0 informations
```

**Why**: Pyright reads configuration from `pyproject.toml` and checks only the `src` directory.

#### Example 3: Pyright vs. mypy on Advanced Typing

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Circle")

def render(item: Drawable) -> None:
    item.draw()

render(Circle())
```

**Expected Pyright Output**: No errors (protocol structurally satisfied).

**Expected mypy Output**: No errors (with appropriate configuration).

**Why**: Both checkers support protocols, but Pyright's spec conformance may catch edge cases that mypy misses.

### Real-World Cases

- **VS Code users**: Pylance provides real-time type checking and IntelliSense.
- **Large monorepos**: Pyright's speed makes it suitable for very large codebases.
- **Open-source projects**: Pyright is used in CI alongside mypy for cross-checking.
- **Jupyter notebooks**: Pylance provides type checking in notebooks.

### References

- Pyright Documentation - https://microsoft.github.io/pyright/
- Pyright Configuration - https://microsoft.github.io/pyright/#/configuration
- Pylance - https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance
- Python Typing Spec Conformance - https://github.com/python/typing/tree/main/conformance

---

## 3. Type-Checking Workflows and CI/CD Integration

### Definitions

**Core Definition**: A type-checking workflow is a systematic process for running type checkers as part of the software development lifecycle, including local development, pre-commit hooks, and continuous integration/continuous deployment (CI/CD) pipelines.

**Technical Definition**: Type-checking workflows integrate static type analysis into automated pipelines using tools such as pre-commit, GitHub Actions, GitLab CI, and other CI/CD platforms. The workflow typically involves: (1) local checking during development, (2) pre-commit hook validation before commits, and (3) CI pipeline enforcement on pull requests and merges.

**Beginner-Friendly Explanation**: A type-checking workflow is how you make sure type checking actually happens — not just when you remember to run it manually, but automatically every time you commit code or open a pull request.

### Purposes

- To catch type errors early in the development cycle, before they reach production.
- To enforce consistent type-checking standards across a team.
- To prevent type regressions from being introduced into the codebase.
- To automate quality gates in CI/CD pipelines.
- To provide fast feedback to developers via pre-commit hooks.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Pre-Commit Hook (`.pre-commit-config.yaml`)**

```yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.17.1
    hooks:
      - id: mypy
        additional_dependencies:
          - pydantic>=2.0.0
          - pytest>=7.0.0
```

**Component Breakdown**:
- `repo` — the repository containing the hook.
- `rev` — the version of the hook.
- `hooks` — list of hooks to run.
- `id: mypy` — the hook identifier.
- `additional_dependencies` — extra packages required for type checking.

**Syntax 2: GitHub Actions Workflow (`.github/workflows/type-check.yml`)**

```yaml
name: Type Check
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -e ".[dev]"
      - run: mypy src/
```

**Component Breakdown**:
- `on` — triggers for the workflow (push, pull request).
- `jobs` — the type-checking job.
- `steps` — sequential steps: checkout, setup Python, install dependencies, run mypy.

**Syntax 3: Combined Workflow with Ruff and mypy**

```yaml
name: Code Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -e ".[dev]"
      - run: ruff check src/
      - run: mypy src/
      - run: pytest
```

#### Syntax Rules

1. **Pre-commit hooks run on `git commit`**: They check only staged files by default.
2. **CI workflows run on push and pull request**: They check the entire codebase.
3. **`additional_dependencies`**: Required for type checkers that need to analyze third-party libraries.
4. **Caching**: Use `actions/cache` to cache mypy's incremental cache for faster CI.
5. **Fail fast**: Configure CI to fail on type errors to prevent regressions.

#### Constraints and Limitations

- Pre-commit hooks need dependencies installed; this can be slow.
- CI workflows need a Python environment with all dependencies.
- mypy is slow on large codebases; consider using `dmypy` or Pyright in CI.
- Pre-commit hooks may not catch type errors in files not staged for commit.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Pre-Commit Hook with mypy

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies:
          - pytest>=7.0.0
          - pydantic>=2.0.0
```

```bash
pre-commit install
pre-commit run --all-files
```

**Expected Output**:
```
mypy.....................................................................Passed
```

**Why**: The pre-commit hook runs mypy on all files before they are committed.

#### Example 2: GitHub Actions Type Check Workflow

```yaml
# .github/workflows/type-check.yml
name: Type Check
on: [push, pull_request]

jobs:
  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install mypy
      - run: mypy src/
```

**Expected CI Output**:
```
Success: no issues found in 42 source files
```

**Why**: The workflow runs mypy on every push and pull request, ensuring type safety in CI.

#### Example 3: Full Quality Gate with Ruff, mypy, and Pytest

```yaml
name: Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -e ".[dev]"
      - run: ruff check src/
      - run: mypy src/
      - run: pytest --cov=src tests/
```

**Expected CI Output**:
```
All checks passed!
Success: no issues found in 42 source files
42 passed in 3.14s
```

**Why**: This workflow combines linting, type checking, and testing into a single quality gate.

### Real-World Cases

- **Open-source projects**: Most Python projects with type hints run mypy or Pyright in CI.
- **Enterprise teams**: Pre-commit hooks ensure all team members follow the same standards.
- **Monorepos**: Incremental type checking is essential for CI performance.
- **Library development**: Type checking is part of the release process.

### References

- pre-commit - https://pre-commit.com/
- pre-commit mypy mirror - https://github.com/pre-commit/mirrors-mypy
- GitHub Actions for Python - https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python
- mypy Continuous Integration - https://mypy.readthedocs.io/en/stable/continuous_integration.html

---

## 4. Gradual Typing (Balancing Dynamic Flexibility with Static Safety)

### Definitions

**Core Definition**: Gradual typing is a type system that allows programmers to combine static and dynamic typing within the same program, opting in to static checking at a fine level of granularity.

**Technical Definition**: A gradual type system is one in which a special "unknown" or "dynamic" type (in Python, `Any`) is used to describe names or expressions whose types are not known statically. The type checker cannot check type correctness of operations on `Any`-typed expressions; these operations are deferred to the Python runtime's dynamic checking. This allows a program to be partially statically checked and partially dynamically checked, with `Any` serving as the boundary between the two regions.

**Beginner-Friendly Explanation**: Gradual typing means you don't have to annotate your entire codebase at once. You can add type hints to the parts that benefit most, leave other parts unannotated, and the type checker will only complain about the annotated parts. You can migrate to static typing gradually, at your own pace.

### Purposes

- To lower the barrier to entry for static typing by allowing incremental adoption.
- To balance the convenience of dynamic typing with the safety of static checking.
- To support migration of legacy codebases to type-annotated code.
- To allow rapid prototyping in dynamic style while enabling later refinement.
- To provide a spectrum of type safety rather than an all-or-nothing choice.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Any

# Fully static function
def add(a: int, b: int) -> int:
    return a + b

# Gradual function: parameter is Any (unchecked)
def process(data: Any) -> None:
    data.unknown_method()  # No static error

# Mixed: static and dynamic in the same function
def handle(value: int | Any) -> None:
    if isinstance(value, int):
        print(value + 1)  # Statically checked
    else:
        value.unknown()  # Dynamically checked
```

**Component Breakdown**:
- `Any` — the gradual type; disables static checking for that value.
- `int | Any` — a union with `Any`; the `Any` part is unchecked.
- Unannotated code — implicitly typed as `Any`.

#### Syntax Rules

1. **`Any` is consistent with all types**: A value of type `Any` can be assigned to any variable and vice versa.
2. **Unannotated code is implicitly `Any`**: Functions without annotations are unchecked.
3. **Type narrowing works across gradual boundaries**: `isinstance()` checks narrow `Any` to a specific type within a branch.
4. **Gradual types can be nested**: `list[Any]`, `dict[str, Any]`, etc.
5. **`Any` is not the same as `object`**: `object` is the safe top type; `Any` disables checking.

#### Constraints and Limitations

- `Any` disables type checking for that value; overuse defeats the purpose of static typing.
- Type errors in dynamic regions are only caught at runtime.
- Gradual typing can lead to inconsistencies between statically checked and dynamically checked regions.
- Performance overhead in sound gradual typing implementations (not applicable to Python's erasure semantics).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Gradual Migration from Dynamic to Static

```python
from typing import Any

# Step 1: Start with unannotated code
def calculate(x, y):
    return x + y

# Step 2: Add type annotations incrementally
def calculate_v2(x: int, y: int) -> int:
    return x + y

# Step 3: Leave some parts dynamic
def process(data: Any) -> Any:
    return data

print(calculate(1, 2))        # 3
print(calculate_v2(1, 2))     # 3
print(process({"key": "val"}))  # {'key': 'val'}
```

**Expected Output**:
```
3
3
{'key': 'val'}
```

**Why**: `calculate` is unannotated (dynamic), `calculate_v2` is fully annotated (static), and `process` uses `Any` to remain dynamic. All three work at runtime.

#### Example 2: Type Narrowing Across Gradual Boundaries

```python
from typing import Any

def process(data: Any) -> None:
    if isinstance(data, int):
        print(data + 1)  # data is narrowed to int
    elif isinstance(data, str):
        print(data.upper())  # data is narrowed to str
    else:
        print("Unknown type")

process(42)       # 43
process("hello")  # HELLO
process([1, 2])   # Unknown type
```

**Expected Output**:
```
43
HELLO
Unknown type
```

**Why**: Even though `data` is `Any`, `isinstance` checks narrow the type within each branch.

#### Example 3: `Any` vs. `object` in Gradual Typing

```python
from typing import Any

def process_any(x: Any) -> None:
    x.unknown_method()  # No type error

def process_object(x: object) -> None:
    # x.unknown_method()  # Type error: object has no attribute 'unknown_method'
    if hasattr(x, 'unknown_method'):
        x.unknown_method()  # Allowed after hasattr check

process_any("hello")  # Runtime AttributeError
```

**Expected Output**:
```
AttributeError: 'str' object has no attribute 'unknown_method'
```

**Why**: `Any` disables checking, so the error only appears at runtime. `object` forces a narrowing check.

### Real-World Cases

- **Legacy code migration**: Add type hints to one module at a time.
- **Prototyping**: Start with dynamic code and add types later.
- **Third-party integration**: Use `Any` for untyped library boundaries.
- **Data science**: Dynamic exploration in notebooks, static analysis for production pipelines.

### References

- Gradual Typing - https://typing.python.org/en/latest/spec/concepts.html#static-dynamic-and-gradual-typing
- PEP 484 – Type Hints (Gradual Typing) - https://peps.python.org/pep-0484/#the-meaning-of-annotations
- Siek, J. & Taha, W. (2006). Gradual Typing for Functional Languages - http://scheme2006.cs.uchicago.edu/13-siek.pdf
- Type System Concepts - https://typing.python.org/en/latest/spec/concepts.html

---

## 5. Type-Safe API Design

### Definitions

**Core Definition**: Type-safe API design is the practice of using Python's type system to create function and class interfaces that are self-documenting, machine-verifiable, and resistant to misuse.

**Technical Definition**: Type-safe API design leverages `TypedDict`, `Protocol`, `Generic`, `@overload`, `Literal`, and other typing constructs to encode API contracts directly in function signatures. These signatures become machine-checkable specifications that type checkers verify at compile time, catching bugs, revealing design smells, and guiding API evolution.

**Beginner-Friendly Explanation**: A type-safe API is one where the type hints tell you exactly how to use it — what arguments to pass, what you'll get back, and what mistakes are impossible. If you try to use it wrong, the type checker tells you before you run the code.

### Purposes

- To turn function signatures into machine-checkable contracts.
- To catch API misuse at type-check time rather than runtime.
- To document the expected structure of data (via `TypedDict`).
- To define interfaces without forcing inheritance (via `Protocol`).
- To support multiple call signatures with different return types (via `@overload`).
- To guide API evolution by making breaking changes visible to type checkers.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: `Protocol` for Structural Interfaces**

```python
from typing import Protocol

class Repository(Protocol):
    def get(self, id: int) -> dict | None: ...
    def save(self, data: dict) -> None: ...
```

**Syntax 2: `TypedDict` for Structured Data**

```python
from typing import TypedDict, NotRequired

class User(TypedDict):
    name: str
    age: int
    email: NotRequired[str]
```

**Syntax 3: `@overload` for Multiple Signatures**

```python
from typing import overload, Literal

@overload
def fetch(url: str, mode: Literal["text"]) -> str: ...
@overload
def fetch(url: str, mode: Literal["json"]) -> dict: ...
def fetch(url: str, mode: Literal["text", "json"]) -> str | dict:
    if mode == "text":
        return "response"
    return {}
```

**Syntax 4: Generics for Reusable Abstractions**

```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Result(Generic[T]):
    def __init__(self, value: T, error: Exception | None = None):
        self.value = value
        self.error = error
```

#### Syntax Rules

1. **`Protocol` defines structural interfaces**: Classes need not inherit; they just need to have the required methods.
2. **`TypedDict` defines fixed-key dictionaries**: All keys required by default; use `NotRequired` for optional keys.
3. **`@overload` requires a concrete implementation**: The final (non-overloaded) function is the actual implementation.
4. **Generics preserve type information**: `Result[T]` preserves the type of `T` through transformations.
5. **`Literal` constrains values**: Use `Literal` for parameters that must be one of a fixed set of constants.

#### Constraints and Limitations

- `Protocol` cannot be instantiated.
- `TypedDict` values must be `dict` instances, not subclasses.
- `@overload` signatures must be compatible with the implementation.
- Generic type variables must be declared with `TypeVar`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: `Protocol` for Dependency Injection

```python
from typing import Protocol

# step1: Define a protocol
class Logger(Protocol):
    def log(self, message: str) -> None: ...

# step2: Implementations (no inheritance)
class ConsoleLogger:
    def log(self, message: str) -> None:
        print(f"[CONSOLE] {message}")

class FileLogger:
    def log(self, message: str) -> None:
        print(f"[FILE] {message}")

# step3: Function accepts the protocol
def process(logger: Logger, data: str) -> None:
    logger.log(f"Processing: {data}")

process(ConsoleLogger(), "test")  # [CONSOLE] Processing: test
process(FileLogger(), "test")     # [FILE] Processing: test
```

**Expected Output**:
```
[CONSOLE] Processing: test
[FILE] Processing: test
```

**Why**: `ConsoleLogger` and `FileLogger` structurally satisfy `Logger` without inheriting from it.

#### Example 2: `TypedDict` for API Payloads

```python
from typing import TypedDict, NotRequired

# step1: Define the schema
class ApiResponse(TypedDict):
    status: int
    data: dict
    error: NotRequired[str]

# step2: Use the schema
def handle_response(response: ApiResponse) -> None:
    if "error" in response:
        print(f"Error: {response['error']}")
    else:
        print(f"Data: {response['data']}")

handle_response({"status": 200, "data": {"key": "value"}})
handle_response({"status": 500, "data": {}, "error": "Server error"})
```

**Expected Output**:
```
Data: {'key': 'value'}
Error: Server error
```

**Why**: `ApiResponse` enforces the presence of `status` and `data`, while `error` is optional.

#### Example 3: `@overload` for Type-Safe API

```python
from typing import overload, Literal

# step1: Define overloads
@overload
def open_file(path: str, mode: Literal["r"]) -> str: ...
@overload
def open_file(path: str, mode: Literal["w"]) -> None: ...
def open_file(path: str, mode: Literal["r", "w"]) -> str | None:
    if mode == "r":
        return f"Reading {path}"
    return None

# step2: Type checker infers return type
content = open_file("data.txt", "r")   # str
print(content)                         # Reading data.txt

result = open_file("data.txt", "w")    # None
print(result)                          # None
```

**Expected Output**:
```
Reading data.txt
None
```

**Why**: The overloads tell the type checker that `mode="r"` returns `str` and `mode="w"` returns `None`.

### Real-World Cases

- **FastAPI**: Uses `TypedDict`, `Protocol`, and `Annotated` for request/response schemas.
- **SQLAlchemy**: Uses generics for `Session`, `Query`, and `Result`.
- **Pydantic**: Uses `TypedDict` and `Protocol` for model definitions.
- **aiida-core**: Uses `Protocol`, `TypedDict`, and `@overload` for plugin interfaces.

### References

- Protocols and Structural Subtyping - https://typing.python.org/en/latest/spec/protocol.html
- TypedDict - https://typing.python.org/en/latest/spec/typeddict.html
- Generics - https://typing.python.org/en/latest/spec/generics.html
- PEP 544 – Protocols - https://peps.python.org/pep-0544/
- PEP 589 – TypedDict - https://peps.python.org/pep-0589/
- PEP 484 – Type Hints (@overload) - https://peps.python.org/pep-0484/#function-method-overloading

---

## 6. Runtime Type Enforcement (Pydantic and Typeguard)

### Definitions

**Core Definition**: Runtime type enforcement is the practice of validating that values conform to their type annotations at program execution time, rather than only statically.

**Technical Definition**: Runtime type enforcement tools such as Pydantic and Typeguard inspect type annotations at runtime and validate input data against them. Pydantic uses type hints to define data models and enforces data schemas when validating input, raising `ValidationError` if the data does not match. Typeguard provides a `@typechecked` decorator and `check_type()` function that raise `TypeError` when runtime values do not match their annotations.

**Beginner-Friendly Explanation**: Static type checkers catch mistakes before you run the code. Runtime type enforcers catch mistakes while the code is running — they check that the data you actually receive (from a user, an API, a file) matches the types you said you expected. Pydantic is the most popular tool for this; it's used by FastAPI and many other frameworks.

### Purposes

- To validate external data (API requests, JSON payloads, user input) against type schemas.
- To enforce type correctness at the boundaries of a system where data is untrusted.
- To catch type errors that static analysis cannot detect (e.g., data from dynamic sources).
- To provide detailed error messages when data does not match the expected schema.
- To enable type-safe serialization and deserialization.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Pydantic BaseModel**

```python
from pydantic import BaseModel, Field
from typing import Annotated

class User(BaseModel):
    name: Annotated[str, Field(min_length=1, max_length=50)]
    age: Annotated[int, Field(ge=0, le=150)]
    email: str | None = None
```

**Component Breakdown**:
- `BaseModel` — Pydantic's base class for data models.
- `name: Annotated[str, Field(min_length=1, max_length=50)]` — validated string field with constraints.
- `age: Annotated[int, Field(ge=0, le=150)]` — validated integer field with bounds.
- `email: str | None = None` — optional field.

**Syntax 2: Pydantic Strict Mode**

```python
class MyModel(BaseModel):
    x: int

# Lax mode (default): coerces "123" to 123
MyModel.model_validate({"x": "123"})  # OK

# Strict mode: rejects "123"
MyModel.model_validate({"x": "123"}, strict=True)  # ValidationError
```

**Syntax 3: Typeguard `@typechecked` Decorator**

```python
from typeguard import typechecked

@typechecked
def add(a: int, b: int) -> int:
    return a + b

add(1, 2)      # OK
add("1", 2)    # TypeError
```

**Syntax 4: Typeguard `check_type()`**

```python
from typeguard import check_type

check_type(42, int)          # OK
check_type("hello", int)     # TypeError
```

#### Syntax Rules

1. **Pydantic models inherit from `BaseModel`**: Fields are defined as class attributes with type annotations.
2. **Pydantic validates on instantiation**: `User(name="Alice", age=30)` triggers validation.
3. **Pydantic strict mode**: Enabled per-call (`strict=True`), per-field (`Field(strict=True)`), or per-model (`ConfigDict(strict=True)`).
4. **Typeguard `@typechecked`**: Decorates functions and classes; checks argument and return types on every call.
5. **Typeguard `check_type()`**: Can be used as a standalone function or with `assert`.
6. **Typeguard import hook**: `install_import_hook()` automatically decorates all functions in a module.

#### Pydantic Strict Mode Details

By default, Pydantic attempts to coerce values to the desired type when possible. For example, passing `"123"` to an `int` field converts it to `123`. When strict mode is enabled, Pydantic errors instead of coercing. Strict mode can be enabled:

- **Per validation call**: `MyModel.model_validate(data, strict=True)`
- **Per field**: `Field(strict=True)`
- **Per model**: `ConfigDict(strict=True)`
- **As a type annotation**: `pydantic.types.StrictInt`

In strict mode, for most types, only instances of the exact type are accepted. However, Pydantic is looser when validating data from JSON in strict mode (e.g., strings are accepted for `date` fields when validating from JSON).

#### Typeguard Methods

Typeguard provides four principal ways to do runtime type checking:

1. **`check_argument_types()` and `check_return_type()` functions**: Debugger-friendly; can be used with `assert` (removed in optimized mode).
2. **`@typechecked` decorator**: Automatically checks arguments and return values; works with generators (checks `yield` and `send`).
3. **Stack profiler hook (deprecated)**: Emits warnings instead of raising errors.
4. **Import hook (`install_import_hook()`)**: Automatically decorates all functions in a module on import.

#### Constraints and Limitations

- Pydantic is a runtime dependency; it adds overhead to model instantiation.
- Typeguard's `@typechecked` adds a stack frame per call, which can affect performance.
- Runtime type checking does not replace static type checking; they are complementary.
- Pydantic strict mode may not be appropriate for all use cases (e.g., when receiving data from HTML forms or URL parameters).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Pydantic Validation (Lax vs. Strict)

```python
from pydantic import BaseModel, ValidationError

class MyModel(BaseModel):
    x: int

# step1: Lax mode (default)
print(MyModel.model_validate({'x': '123'}))  # x=123

# step2: Strict mode
try:
    MyModel.model_validate({'x': '123'}, strict=True)
except ValidationError as exc:
    print(exc)
    # 1 validation error for MyModel
    # x
    #   Input should be a valid integer [type=int_type, input_value='123', input_type=str]
```

**Expected Output**:
```
x=123
1 validation error for MyModel
x
  Input should be a valid integer [type=int_type, input_value='123', input_type=str]
```

**Why**: Lax mode coerces `"123"` to `123`; strict mode rejects it because `"123"` is not an `int`.

#### Example 2: Typeguard `@typechecked`

```python
from typeguard import typechecked

@typechecked
def add(a: int, b: int) -> int:
    return a + b

print(add(1, 2))  # 3

try:
    add("1", 2)
except TypeError as e:
    print(e)  # argument of type "str" cannot be assigned to parameter "a" of type "int"
```

**Expected Output**:
```
3
argument of type "str" cannot be assigned to parameter "a" of type "int"
```

**Why**: `@typechecked` validates arguments at runtime, raising `TypeError` for mismatched types.

#### Example 3: Pydantic with `Annotated` Constraints

```python
from pydantic import BaseModel, Field
from typing import Annotated

class User(BaseModel):
    name: Annotated[str, Field(min_length=1, max_length=50)]
    age: Annotated[int, Field(ge=0, le=150)]

# step1: Valid data
user = User(name="Alice", age=30)
print(user)  # name='Alice' age=30

# step2: Invalid data
try:
    User(name="", age=200)
except ValidationError as e:
    print(e)
    # 2 validation errors for User
    # name: String should have at least 1 character
    # age: Input should be less than or equal to 150
```

**Expected Output**:
```
name='Alice' age=30
2 validation errors for User
name
  String should have at least 1 character [type=string_too_short, ...]
age
  Input should be less than or equal to 150 [type=less_than_equal, ...]
```

**Why**: `Annotated` with `Field` attaches validation constraints that Pydantic enforces at runtime.

### Real-World Cases

- **FastAPI**: Uses Pydantic for request/response validation.
- **Django REST Framework**: Can use Pydantic for serializer validation.
- **CLI tools**: Use Pydantic to validate configuration files.
- **Data pipelines**: Use Pydantic to validate incoming data.
- **Testing**: Use Typeguard to catch type errors in tests.

### References

- Pydantic Documentation - https://docs.pydantic.dev/
- Pydantic Strict Mode - https://docs.pydantic.dev/latest/concepts/strict_mode/
- Typeguard Documentation - https://typeguard.readthedocs.io/
- Typeguard GitHub - https://github.com/agronholm/typeguard
- PEP 593 – Flexible function and variable annotations - https://peps.python.org/pep-0593/

---

## References

- mypy Documentation - https://mypy.readthedocs.io/
- mypy Configuration - https://mypy.readthedocs.io/en/stable/config_file.html
- Pyright Documentation - https://microsoft.github.io/pyright/
- Pylance - https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance
- Python Typing Spec - https://typing.python.org/en/latest/spec/
- Gradual Typing Concepts - https://typing.python.org/en/latest/spec/concepts.html
- Protocols and Structural Subtyping - https://typing.python.org/en/latest/spec/protocol.html
- Pydantic Documentation - https://docs.pydantic.dev/
- Pydantic Strict Mode - https://docs.pydantic.dev/latest/concepts/strict_mode/
- Typeguard Documentation - https://typeguard.readthedocs.io/
- pre-commit - https://pre-commit.com/
- pre-commit mypy mirror - https://github.com/pre-commit/mirrors-mypy
- GitHub Actions for Python - https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-python
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- PEP 544 – Protocols - https://peps.python.org/pep-0544/
- PEP 589 – TypedDict - https://peps.python.org/pep-0589/
- PEP 591 – Final qualifier - https://peps.python.org/pep-0591/
- PEP 593 – Flexible annotations - https://peps.python.org/pep-0593/
- PEP 604 – Union syntax - https://peps.python.org/pep-0604/
- PEP 612 – Parameter Specification Variables - https://peps.python.org/pep-0612/
- PEP 613 – Explicit Type Aliases - https://peps.python.org/pep-0613/
- PEP 647 – User-Defined Type Guards - https://peps.python.org/pep-0647/
- PEP 655 – Required and NotRequired - https://peps.python.org/pep-0655/
- PEP 673 – Self Type - https://peps.python.org/pep-0673/
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/
- Python Typing Conformance Suite - https://github.com/python/typing/tree/main/conformance
- Siek, J. & Taha, W. (2006). Gradual Typing for Functional Languages - http://scheme2006.cs.uchicago.edu/13-siek.pdf