# Python Code Formatting and Linting: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Code formatting and linting encompass the automated tools and practices used to enforce consistent code style, detect structural errors, and maintain code quality across a Python project. Formatting tools (Black, isort) rewrite code to conform to a style standard; linters (Ruff, Flake8) analyse code for errors, style violations, and complexity issues without modifying it.

### Technical Definition

Code formatting is the automated transformation of source code into a canonical style layout, eliminating subjective formatting decisions. Linting is the static analysis of source code for programming errors, bugs, stylistic errors, and suspicious constructs. In the Python ecosystem, these practices are implemented through a toolchain comprising **Black** (opinionated formatter), **Ruff** (Rust-based linter and formatter), **Flake8** (legacy linting wrapper), **isort** (import sorter), and **pre-commit** (Git hook framework), complemented by static type checkers (**mypy**, **Pyright**, **Pyre**).

### Beginner-Friendly Explanation

Writing Python code is easier when everyone follows the same style rules. Formatting tools like Black automatically reformat your code so it looks consistent — you don't have to think about where to put spaces or line breaks. Linters like Ruff and Flake8 check your code for mistakes and style problems, like unused imports or variables that don't follow naming conventions. Import sorters like isort organise your imports alphabetically. And pre-commit hooks run all these tools automatically every time you try to commit code, so you never accidentally commit messy or broken code.

### Key Characteristics

- **Automated**: Tools run automatically, reducing manual effort and human error.
- **Deterministic**: Formatting tools produce the same output every time for the same input.
- **Configurable**: Most tools support configuration via `pyproject.toml`, `setup.cfg`, or dedicated config files.
- **Composable**: Tools work together (e.g., Black + isort + Ruff) and integrate with pre-commit and CI/CD.
- **Fast**: Modern tools like Ruff (written in Rust) are orders of magnitude faster than legacy alternatives.

### Prerequisites

- Python 3.8+ installed (Ruff requires 3.8+; Black requires 3.8+).
- A text editor or IDE with Python support.
- Basic familiarity with Python syntax, imports, and project structure.
- Git (for pre-commit hook integration).

### Related Programming Areas

- **Continuous integration**: Running linters and formatters in CI pipelines.
- **Static type checking**: mypy, Pyright, and Pyre complement linting.
- **Code review**: Automated tools reduce review burden by handling style.
- **Developer experience**: IDE integrations (format-on-save, inline linting).

### Core Concepts / Features

The following sections cover each tool using a uniform structure.

---

## 1. Black — The Uncompromising Code Formatter

### Definitions

**Core Definition**: Black is an opinionated, deterministic Python code formatter that reformats entire files in place according to a strict, PEP 8-compliant style.

**Technical Definition**: Black is a PEP 8-compliant opinionated formatter. It reformats entire files in place, and style configuration options are deliberately limited and rarely added. By using it, developers agree to cede control over hand-formatting in return for speed, determinism, and freedom from pycodestyle nagging about formatting. Black can be installed via `pip install black` and requires Python 3.6.0+ to run, though it can reformat Python 2 code as well.

**Beginner-Friendly Explanation**: Black is a tool that automatically fixes the formatting of your Python code. Instead of arguing about where to put commas or how many blank lines to use, you just run Black and it makes all the decisions for you. It's called "uncompromising" because it doesn't have many options — you either accept its style or you don't use it.

### Purposes

- To eliminate formatting bike-shedding (time wasted debating style) by enforcing a single canonical style.
- To produce the smallest possible diffs, making code reviews faster and more focused on substance.
- To provide deterministic formatting so that the same code always looks the same regardless of who wrote it.
- To free developers from pycodestyle nagging by automatically fixing style violations.
- To save time and mental energy for more important matters than hand-formatting.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
black [OPTIONS] [SRC]...
```

**Component Breakdown**:
- `SRC` — one or more files or directories to format.
- `OPTIONS` — command-line flags controlling Black's behaviour.

#### Common Options

| Option | Description | Default |
|--------|-------------|---------|
| `-l, --line-length INTEGER` | Characters per line | 88 |
| `-t, --target-version` | Python versions to support | Auto-detected |
| `--check` | Don't write files; return status only | — |
| `--diff` | Output diff without writing | — |
| `-S, --skip-string-normalization` | Don't normalize string quotes | — |
| `-C, --skip-magic-trailing-comma` | Don't use trailing commas to split lines | — |

#### Configuration via `pyproject.toml`

```toml
[tool.black]
line-length = 88
target-version = ["py311"]
include = '\.pyi?$'
extend-exclude = '''
/(
    \.git
  | \.mypy_cache
  | \.tox
  | \.venv
  | _build
  | buck-out
  | build
  | dist
)/
'''
```

#### Syntax Rules

1. **Black reformats entire files in place**: It does not partially format; the whole file is rewritten.
2. **Style options are deliberately limited**: Only a small set of configuration options are available; the vast majority of style decisions are made by Black.
3. **Line length defaults to 88**: This is the only widely adjusted option.
4. **Target version auto-detection**: Black infers the target Python version from `pyproject.toml` project metadata unless overridden.
5. **`--check` mode**: Returns 0 if nothing would change, 1 if files would be reformatted, and 123 for internal errors.
6. **String normalization**: Black normalizes string quotes and prefixes unless `-S` is used.

#### Constraints and Limitations

- Black is opinionated; it does not allow extensive customisation.
- Formatting may occasionally produce unexpected line breaks for very long strings or complex expressions.
- Black does not perform linting; it only handles formatting.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Formatting

```python
# step1: Unformatted code (input.py)
def calculate( a,b ):
    return a+b

# step2: Run Black
# $ black input.py
# reformatted input.py
# All done! ✨ 🍰 ✨
# 1 file reformatted.

# step3: Formatted code (output)
def calculate(a, b):
    return a + b
```

**Expected Output**:
```
reformatted input.py
All done! ✨ 🍰 ✨
1 file reformatted.
```

**Why**: Black normalizes whitespace around operators, removes spaces inside parentheses, and ensures consistent indentation.

#### Example 2: Configuration with `pyproject.toml`

```toml
# pyproject.toml
[tool.black]
line-length = 100
target-version = ["py311"]
```

```python
# step1: Code with a long line (before formatting)
def process_data(input_file, output_file, encoding="utf-8", errors="strict", buffer_size=8192):
    pass

# step2: After Black with line-length = 100
def process_data(
    input_file,
    output_file,
    encoding="utf-8",
    errors="strict",
    buffer_size=8192,
):
    pass
```

**Expected Output**: Black splits the long function signature across multiple lines, with one parameter per line and a trailing comma.

**Why**: Black uses the configured line length to decide when to split lines; the trailing comma triggers the "magic trailing comma" behaviour, forcing the split.

#### Example 3: `--check` and `--diff` in CI

```bash
# step1: Check formatting without writing
black --check src/

# step2: Output if files would be reformatted
# would reformat src/module.py
# Oh no! 💥 💔 💥
# 1 file would be reformatted.
```

**Expected CI Output**:
```
would reformat src/module.py
Oh no! 💥 💔 💥
1 file would be reformatted.
```

**Why**: `--check` is used in CI to fail the build if any files are not formatted; `--diff` shows what would change.

### Real-World Cases

- **Open-source projects**: Black is widely adopted; its deterministic output makes it ideal for shared codebases.
- **CI pipelines**: `black --check` ensures all committed code is formatted.
- **Pre-commit hooks**: Black is commonly used as a pre-commit hook to format code before committing.

### References

- Black Documentation - https://black.readthedocs.io/
- Black — Debian Manpage - https://manpages.debian.org/trixie/black/black.1.en.html

---

## 2. Ruff — Ultra-Fast Linter and Formatter

### Definitions

**Core Definition**: Ruff is an extremely fast Python linter and code formatter written in Rust, designed to replace multiple legacy tools (Flake8, isort, pydocstyle, pyupgrade, etc.) with a single, unified binary.

**Technical Definition**: Ruff is an extremely fast Python linter and code formatter, written in Rust. It can be configured through a `pyproject.toml`, `ruff.toml`, or `.ruff.toml` file and supports hierarchical configuration, where the "closest" config file in the directory hierarchy is used for each file. Ruff provides both `ruff check` (linting) and `ruff format` (formatting) commands, along with quick-fix capabilities for many lint rules.

**Beginner-Friendly Explanation**: Ruff is a tool that both checks your code for problems (linting) and reformats it (formatting). It's written in Rust, which makes it incredibly fast — much faster than older Python-based tools. It can do the job of several different tools at once, so you only need to install and configure one thing.

### Purposes

- To replace multiple legacy linting tools (Flake8, isort, pydocstyle, pyupgrade, etc.) with a single fast binary.
- To provide linting and formatting in one tool, reducing configuration complexity.
- To offer quick-fixes for many common lint violations via `--fix`.
- To be fast enough to run on every save in an IDE without noticeable delay.
- To support hierarchical configuration, allowing different rules for different subdirectories.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
ruff check [OPTIONS] [FILES]...
ruff format [OPTIONS] [FILES]...
```

**Component Breakdown**:
- `check` — run the linter.
- `format` — run the formatter.
- `FILES` — files or directories to process.

#### Configuration via `pyproject.toml`

```toml
[tool.ruff]
line-length = 88
target-version = "py311"
exclude = [
    ".bzr", ".direnv", ".eggs", ".git", ".hg",
    ".mypy_cache", ".nox", ".pants.d", ".pytype",
    ".ruff_cache", ".svn", ".tox", ".venv", "__pypackages__",
    "_build", "buck-out", "build", "dist", "node_modules", "venv",
]

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM", "RUF"]
ignore = ["E501"]

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["S101", "PLR2004"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "auto"
```

#### Rule Categories

| Prefix | Source | Description |
|--------|--------|-------------|
| `E` | pycodestyle | Style errors |
| `W` | pycodestyle | Style warnings |
| `F` | PyFlakes | Logical errors |
| `I` | isort | Import sorting |
| `N` | pep8-naming | Naming conventions |
| `UP` | pyupgrade | Modern Python idioms |
| `B` | flake8-bugbear | Bug detection |
| `C4` | flake8-comprehensions | Comprehension improvements |
| `SIM` | flake8-simplify | Code simplification |
| `RUF` | Ruff-specific | Ruff-only rules |

#### Syntax Rules

1. **Configuration file resolution**: Ruff respects `pyproject.toml`, `ruff.toml`, and `.ruff.toml` files; hierarchical config means the closest file wins.
2. **`select` and `ignore`**: Control which rule categories are enabled or disabled.
3. **`per-file-ignores`**: Apply different rules to specific file patterns.
4. **`--fix`**: Automatically fixes fixable violations.
5. **Formatter compatibility**: Ruff's formatter is 99.9% compatible with Black.
6. **Editor integration**: Ruff provides a language server for IDE integration with configurable settings.

#### Constraints and Limitations

- Some Flake8 plugins are not fully implemented in Ruff.
- Ruff's formatter, while Black-compatible, may have minor differences.
- Certain complex lint rules may produce false positives.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Linting and Formatting

```python
# step1: Code with issues
import os
import sys
import json

def  process( data ):
    x=1
    if x==1:
        return True
    unused_var = 42

# step2: Run Ruff check
# $ ruff check example.py
# example.py:1:1: I001 [*] Import block is un-sorted or un-formatted
# example.py:5:1: E302 Expected 2 blank lines, found 1
# example.py:5:7: E211 Whitespace before '('
# example.py:6:5: E225 Missing whitespace around operator
# example.py:7:8: E225 Missing whitespace around operator
# example.py:9:5: F841 Local variable `unused_var` is assigned to but never used
# Found 6 errors.
# [*] 3 fixable with the `--fix` option.

# step3: Run Ruff with --fix
# $ ruff check --fix example.py
# Found 3 errors (3 fixed, 0 remaining).

# step4: Formatted and fixed code
import json
import os
import sys


def process(data):
    x = 1
    if x == 1:
        return True
    unused_var = 42
```

**Expected Output**:
```
Found 6 errors.
[*] 3 fixable with the `--fix` option.
Found 3 errors (3 fixed, 0 remaining).
```

**Why**: Ruff detects import sorting, spacing, and unused variable issues; `--fix` automatically resolves the fixable ones.

#### Example 2: Ruff Formatter

```bash
# step1: Run Ruff formatter
ruff format src/

# step2: Output
# 3 files reformatted, 2 files left unchanged
```

**Expected Output**:
```
3 files reformatted, 2 files left unchanged
```

**Why**: `ruff format` applies Black-compatible formatting to all Python files in `src/`.

#### Example 3: Editor Integration Configuration

```json
{
  "ruff.configuration": {
    "lint": {
      "select": ["E", "F", "I", "N", "W"]
    },
    "format": {
      "quote-style": "single"
    }
  }
}
```

**Expected Behaviour**: The editor uses the inline configuration, overriding any project-level config, and formats strings with single quotes.

### Real-World Cases

- **Large codebases**: Ruff's Rust implementation provides massive speed improvements over Python-based linters.
- **Unified tooling**: Using Ruff alone simplifies the toolchain by replacing Flake8, isort, and Black.
- **IDE integration**: Ruff's language server provides real-time linting and formatting.

### References

- Ruff Documentation - https://docs.astral.sh/ruff/
- Ruff Configuration - https://docs.astral.sh/ruff/configuration/
- Ruff Editor Settings - https://docs.astral.sh/ruff/editors/settings/

---

## 3. Flake8 — Legacy Static Linting Wrapper

### Definitions

**Core Definition**: Flake8 is a command-line utility that wraps three tools — PyFlakes, pycodestyle, and Ned Batchelder's McCabe script — into a single interface for enforcing style consistency and detecting logical errors.

**Technical Definition**: Flake8 is a wrapper around PyFlakes (logical error detection), pycodestyle (PEP 8 style checks), and the McCabe complexity checker. It runs all tools by launching a single `flake8` command. Flake8 supports configuration in `setup.cfg`, `tox.ini`, or `.flake8` files under a `[flake8]` section; configuration in `pyproject.toml` requires a plugin like `flake8-toml-config`.

**Beginner-Friendly Explanation**: Flake8 is a tool that checks your Python code for style problems and common mistakes. It combines three older tools into one, so you don't have to run them separately. It tells you about things like lines that are too long, unused imports, and code that's too complex.

### Purposes

- To enforce PEP 8 style consistency across a Python project.
- To detect logical errors (unused imports, undefined names) via PyFlakes.
- To measure code complexity via the McCabe complexity checker.
- To provide a single command that runs multiple linting tools.
- To support plugins that extend its functionality.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
flake8 [OPTIONS] [FILES]...
```

#### Configuration via `setup.cfg`

```ini
[flake8]
max-line-length = 88
extend-ignore = E203, E501
exclude =
    .git,
    __pycache__,
    build,
    dist,
    .venv
max-complexity = 10
per-file-ignores =
    tests/*:S101
```

#### Configuration via `tox.ini`

```ini
[flake8]
max-line-length = 88
extend-ignore = E203
max-complexity = 10
```

#### Syntax Rules

1. **Configuration files**: Flake8 reads from `setup.cfg`, `tox.ini`, or `.flake8`; only the first file found is used.
2. **INI syntax**: Configuration uses `[flake8]` section with `key = value` pairs.
3. **`max-line-length`**: Sets the maximum allowed line length.
4. **`extend-ignore`**: Adds error codes to ignore.
5. **`max-complexity`**: Sets the McCabe complexity threshold.
6. **`per-file-ignores`**: Applies different ignores to specific file patterns.

#### Constraints and Limitations

- Flake8 does not natively support `pyproject.toml`; a plugin is required.
- It is slower than Ruff because it is written in Python.
- It does not auto-fix violations; it only reports them.
- The plugin ecosystem is mature but requires separate installation.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Linting

```python
# step1: Code with issues
import os
import sys

def  process( data ):
    x=1
    if x==1:
        return True

# step2: Run Flake8
# $ flake8 example.py
# example.py:1:1: F401 'os' imported but unused
# example.py:2:1: F401 'sys' imported but unused
# example.py:4:1: E302 expected 2 blank lines, found 1
# example.py:4:7: E211 whitespace before '('
# example.py:5:5: E225 missing whitespace around operator
# example.py:6:8: E225 missing whitespace around operator
```

**Expected Output**:
```
example.py:1:1: F401 'os' imported but unused
example.py:2:1: F401 'sys' imported but unused
example.py:4:1: E302 expected 2 blank lines, found 1
example.py:4:7: E211 whitespace before '('
example.py:5:5: E225 missing whitespace around operator
example.py:6:8: E225 missing whitespace around operator
```

**Why**: Flake8 reports unused imports (F401), style violations (E302, E211, E225), and missing blank lines.

#### Example 2: Configuration

```ini
# setup.cfg
[flake8]
max-line-length = 100
extend-ignore = E203, W503
max-complexity = 12
exclude = .git,__pycache__,build,dist,.venv
```

```bash
# step1: Run with configuration
flake8 src/

# step2: Output (only errors above complexity 12 and beyond line 100)
# src/module.py:45:1: C901 'process' is too complex (15)
```

**Expected Output**:
```
src/module.py:45:1: C901 'process' is too complex (15)
```

**Why**: The configuration raises the line-length limit, ignores specific style rules, and enforces a complexity threshold of 12.

#### Example 3: Plugin Usage

```bash
# step1: Install plugins
pip install flake8-docstrings flake8-bugbear

# step2: Run Flake8 with plugins
flake8 --select=D,B src/

# step3: Output
# src/module.py:1:1: D100 Missing docstring in public module
# src/module.py:5:1: B006 Do not use mutable data structures for argument defaults
```

**Expected Output**:
```
src/module.py:1:1: D100 Missing docstring in public module
src/module.py:5:1: B006 Do not use mutable data structures for argument defaults
```

**Why**: Plugins extend Flake8's capabilities: `flake8-docstrings` adds docstring checks (D codes), and `flake8-bugbear` adds bug detection (B codes).

### Real-World Cases

- **Legacy projects**: Flake8 remains common in projects that have not migrated to Ruff.
- **Plugin ecosystem**: Projects relying on specific Flake8 plugins (e.g., `flake8-docstrings`).
- **CI pipelines**: Flake8 is often run alongside or instead of Ruff in CI.

### References

- Flake8 Documentation - https://flake8.pycqa.org/
- Flake8 Configuration - https://flake8.pycqa.org/en/latest/user/configuration.html

---

## 4. isort — Import Sorting and Organisation

### Definitions

**Core Definition**: isort is a Python utility that sorts imports alphabetically and automatically separates them into sections and by type.

**Technical Definition**: isort is a Python utility/library to sort imports alphabetically and automatically separate them into sections and by type. It provides a command-line utility, a Python library, and plugins for various editors to quickly sort all imports. isort supports configuration via `pyproject.toml`, `.isort.cfg`, `setup.cfg`, and `tox.ini`, and includes a `profile = "black"` option for compatibility with Black.

**Beginner-Friendly Explanation**: isort organises your import statements. It puts standard library imports first, then third-party imports, then your own local imports — all sorted alphabetically. It's like a filing system for your imports.

### Purposes

- To sort imports alphabetically within groups.
- To separate imports into standard library, third-party, and local sections.
- To ensure consistent import ordering across a project.
- To be compatible with Black via the `profile = "black"` setting.
- To verify import correctness in CI pipelines.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
isort [OPTIONS] [FILES]...
```

#### Configuration via `pyproject.toml`

```toml
[tool.isort]
profile = "black"
line_length = 88
known_first_party = ["mypackage"]
src_paths = ["src", "tests"]
skip_gitignore = true
```

#### Syntax Rules

1. **Section separation**: isort groups imports into FUTURE, STDLIB, THIRDPARTY, FIRSTPARTY, and LOCALFOLDER sections.
2. **Alphabetical sorting**: Within each section, imports are sorted alphabetically.
3. **`profile = "black"`**: Ensures isort's output is compatible with Black.
4. **`--check` mode**: Verifies that imports are sorted without making changes.
5. **`--diff` mode**: Shows what would change.
6. **`known_first_party`**: Specifies which packages are considered first-party (local).

#### Constraints and Limitations

- isort does not lint; it only sorts imports.
- Conflicting configurations between isort and Black must be resolved via the `black` profile.
- Large projects may require careful configuration of `known_first_party` and `known_third_party`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Import Sorting

```python
# step1: Unsorted imports (before.py)
import sys
import os
import json
from collections import Counter
import requests
from mypackage import utils

# step2: Run isort
# $ isort before.py
# Fixing /path/to/before.py

# step3: Sorted imports (after)
import json
import os
import sys
from collections import Counter

import requests

from mypackage import utils
```

**Expected Output**:
```
Fixing /path/to/before.py
```

**Why**: isort groups standard library imports (json, os, sys, collections) first, then third-party (requests), then local (mypackage).

#### Example 2: Configuration with Black

```toml
# pyproject.toml
[tool.isort]
profile = "black"
line_length = 88
known_first_party = ["mypackage"]
```

```python
# step1: Before formatting
from mypackage import utils
import requests
import os

# step2: After isort with profile="black"
import os

import requests

from mypackage import utils
```

**Expected Output**: Imports are grouped and sorted; the `profile = "black"` setting ensures compatibility with Black's formatting.

#### Example 3: CI Verification

```bash
# step1: Check imports in CI
isort --check --diff src/

# step2: Output if unsorted
# ERROR: /path/to/src/module.py Imports are incorrectly sorted.
# --- /path/to/src/module.py:before
# +++ /path/to/src/module.py:after
# @@ -1,3 +1,3 @@
# -import sys
#  import os
# +import sys
```

**Expected CI Output**:
```
ERROR: /path/to/src/module.py Imports are incorrectly sorted.
```

**Why**: `--check --diff` is used in CI to fail if imports are not sorted, showing a diff of what should change.

### Real-World Cases

- **Pre-commit hooks**: isort is commonly used as a pre-commit hook to sort imports before committing.
- **CI pipelines**: `isort --check` ensures all committed code has sorted imports.
- **IDE integration**: isort plugins sort imports on save in editors.

### References

- isort Documentation - https://isort.readthedocs.io/
- isort Compatibility with Black - https://isort.readthedocs.io/en/latest/configuration/black_compatibility.html

---

## 5. Automated Formatting (CI/CD, IDE, Pre-commit)

### Definitions

**Core Definition**: Automated formatting is the practice of running code formatters and linters automatically at key points in the development workflow — on save in the IDE, before committing via Git hooks, and in CI/CD pipelines.

**Technical Definition**: Automated formatting integrates tools like Black, Ruff, and isort into development workflows through three mechanisms: (1) IDE format-on-save extensions, (2) Git pre-commit hooks managed by the pre-commit framework, and (3) CI/CD pipeline steps that verify formatting. The pre-commit framework is a multi-language tool for managing and running code-quality checks as Git hooks, catching problems before changes are committed.

**Beginner-Friendly Explanation**: Automated formatting means you don't have to remember to run the formatter — it runs for you. When you save a file in your editor, it formats automatically. When you try to commit code, a hook checks the formatting and blocks the commit if something's wrong. And in CI, the pipeline verifies everything is formatted correctly before merging.

### Purposes

- To prevent unformatted code from being committed or merged.
- To provide immediate feedback on formatting and linting issues.
- To reduce manual effort by automating style enforcement.
- To ensure consistency across the entire team without relying on individual discipline.
- To integrate code quality checks into the CI/CD pipeline as quality gates.

### Syntax Rules and Structure

#### Pre-Commit Configuration (`.pre-commit-config.yaml`)

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/psf/black
    rev: 24.8.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/PyCQA/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ["--profile", "black"]

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.11.0
    hooks:
      - id: mypy
        additional_dependencies: [pydantic>=2.0]
```

#### GitHub Actions Workflow

```yaml
name: Lint and Format
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install black ruff isort mypy
      - run: black --check src/
      - run: isort --check src/
      - run: ruff check src/
      - run: mypy src/
```

#### Syntax Rules

1. **Pre-commit hooks run on `git commit`**: They check only staged files by default.
2. **`.pre-commit-config.yaml`**: Defines hooks, their versions, and arguments.
3. **CI workflows run on push/pull request**: They check the entire codebase.
4. **`--check` flags**: Used in CI to fail if files would be reformatted.
5. **`--fix` flags**: Used locally to automatically fix issues.
6. **Hook ordering**: Formatters should run before linters so linters see formatted code.

#### Constraints and Limitations

- Pre-commit hooks add time to each commit; large projects may need optimisation.
- CI pipelines add latency to pull requests.
- Conflicting tool configurations must be resolved (e.g., isort `profile = "black"`).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Pre-Commit Setup

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.8.0
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
        args: [--fix]
```

```bash
# step1: Install pre-commit
pip install pre-commit
pre-commit install

# step2: Run on all files
pre-commit run --all-files

# step3: Output
# black....................................................................Passed
# ruff.....................................................................Passed
```

**Expected Output**:
```
black....................................................................Passed
ruff.....................................................................Passed
```

**Why**: Pre-commit runs Black and Ruff on all files; both pass because the code is already formatted and linted.

#### Example 2: CI Integration

```yaml
# .github/workflows/lint.yml
name: Lint
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install black ruff isort mypy
      - run: black --check --diff src/
      - run: isort --check --diff src/
      - run: ruff check src/
      - run: mypy src/
```

**Expected CI Output** (on failure):
```
would reformat src/module.py
Oh no! 💥 💔 💥
1 file would be reformatted.
Error: Process completed with exit code 1.
```

**Why**: The CI pipeline fails if any file is not formatted, has unsorted imports, or has lint or type errors.

#### Example 3: IDE Format-on-Save

```json
// VS Code settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "ms-python.black-formatter",
  "[python]": {
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit",
      "source.fixAll": "explicit"
    }
  },
  "ruff.lint.enable": true,
  "ruff.format.enable": false
}
```

**Expected Behaviour**: VS Code formats the file with Black on save, organises imports with isort, and applies Ruff quick-fixes.

**Why**: IDE integration provides immediate feedback and automatic fixing without leaving the editor.

### Real-World Cases

- **Team development**: Pre-commit hooks ensure all team members commit formatted code.
- **Open-source projects**: CI checks enforce formatting standards for contributions.
- **Enterprise**: Pre-commit and CI quality gates prevent unformatted code from reaching production.

### References

- pre-commit Documentation - https://pre-commit.com/
- pre-commit Hooks Reference - https://pre-commit.com/hooks.html

---

## 6. Static Linting (Type Checking)

### Definitions

**Core Definition**: Static linting, in the context of type checking, is the use of dedicated tools (mypy, Pyright, Pyre) to analyse Python code for type errors without executing it.

**Technical Definition**: Static type checkers analyse Python source code against type annotations to detect type errors, inconsistent annotations, and potential bugs before runtime. The principal tools are **mypy** (the original, PEP 484 co-developed checker), **Pyright** (Microsoft's checker, powers Pylance in VS Code), and **Pyre** (Meta's checker, designed for large codebases). A 2022 study found that Pyright was superior to other checking tools with the highest F1 score, while mypy focused on explicitly annotated code, leading to relatively few bug reports.

**Beginner-Friendly Explanation**: Type checkers look at your Python code and tell you if you're using types incorrectly — like passing a string where an integer is expected. They catch mistakes before you run the code, saving you from runtime errors.

### Purposes

- To catch type errors before runtime.
- To verify that type annotations are consistent and correct.
- To improve IDE autocompletion and refactoring support.
- To document the expected types of function parameters and return values.
- To enforce type discipline in large codebases.

### Syntax Rules and Structure

#### mypy Configuration (`pyproject.toml`)

```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
check_untyped_defs = true
no_implicit_reexport = true
```

#### Pyright Configuration (`pyproject.toml`)

```toml
[tool.pyright]
pythonVersion = "3.11"
typeCheckingMode = "strict"
include = ["src"]
exclude = ["**/node_modules", "**/__pycache__"]
reportMissingTypeStubs = false
```

#### Syntax Rules

1. **mypy `strict = true`**: Enables all strict-mode checks, including `disallow_untyped_defs`, `warn_return_any`, and `no_implicit_reexport`.
2. **Pyright `typeCheckingMode = "strict"`**: Enables all strict checks.
3. **Type stubs**: Both tools use `.pyi` stub files for libraries without inline annotations.
4. **Gradual typing**: Unannotated code is treated as dynamically typed (implicitly `Any`).
5. **Pyright leads conformance**: Pyright currently leads the typing-spec conformance suite at ~95%, while mypy is at ~75%.

#### Constraints and Limitations

- mypy is slower than Pyright on large codebases.
- Pyright requires Node.js for command-line use.
- Pyre is primarily designed for Meta's internal workflows.
- Not all type checkers agree on edge cases.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: mypy Basic Check

```python
# step1: Code with type error
def add(a: int, b: int) -> int:
    return a + b

result: str = add(1, 2)  # Type error: int is not str

# step2: Run mypy
# $ mypy example.py
# example.py:4: error: Incompatible types in assignment
#     (expression has type "int", variable has type "str")
# Found 1 error in 1 file (checked 1 source file)
```

**Expected Output**:
```
example.py:4: error: Incompatible types in assignment
    (expression has type "int", variable has type "str")
Found 1 error in 1 file (checked 1 source file)
```

**Why**: mypy detects that the return value of `add` (an `int`) is assigned to a variable annotated as `str`.

#### Example 2: Pyright Strict Mode

```python
# step1: Code with type error
def process(data):
    return data.upper()

# step2: Run pyright
# $ pyright example.py
# example.py:2:12 - error: Type of parameter "data" is unknown
# example.py:3:12 - error: Cannot access member "upper" for type "Unknown"
# 2 errors, 0 warnings, 0 informations
```

**Expected Output**:
```
example.py:2:12 - error: Type of parameter "data" is unknown
example.py:3:12 - error: Cannot access member "upper" for type "Unknown"
2 errors, 0 warnings, 0 informations
```

**Why**: Pyright in strict mode requires type annotations and flags untyped parameters and unknown member access.

#### Example 3: CI Integration

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

**Why**: The CI pipeline runs mypy on every push and pull request, ensuring type safety in the codebase.

### Real-World Cases

- **Large codebases**: Pyright's speed makes it suitable for very large projects.
- **Open-source projects**: mypy is the ecosystem standard and what most CI systems expect.
- **Meta-scale projects**: Pyre is designed for Instagram's 20-million-line codebase.
- **IDE integration**: Pyright powers Pylance in VS Code.

### References

- mypy Documentation - https://mypy.readthedocs.io/
- Pyright Documentation - https://microsoft.github.io/pyright/
- Pyre Documentation - https://pyre-check.org/
- mypy vs Pyright vs Pyrefly: Choosing a Python Type Checker in 2026 - https://codegym.cc/groups/posts/mypy-vs-pyright-vs-pyrefly
- Aalto University Type Checker Comparison - https://aaltodoc.aalto.fi/

---

## References

- Black Documentation - https://black.readthedocs.io/
- Black — Debian Manpage - https://manpages.debian.org/trixie/black/black.1.en.html
- Ruff Documentation - https://docs.astral.sh/ruff/
- Ruff Configuration - https://docs.astral.sh/ruff/configuration/
- Ruff Editor Settings - https://docs.astral.sh/ruff/editors/settings/
- Flake8 Documentation - https://flake8.pycqa.org/
- Flake8 Configuration - https://flake8.pycqa.org/en/latest/user/configuration.html
- isort Documentation - https://isort.readthedocs.io/
- isort Compatibility with Black - https://isort.readthedocs.io/en/latest/configuration/black_compatibility.html
- pre-commit Documentation - https://pre-commit.com/
- pre-commit Hooks Reference - https://pre-commit.com/hooks.html
- mypy Documentation - https://mypy.readthedocs.io/
- Pyright Documentation - https://microsoft.github.io/pyright/
- Pyre Documentation - https://pyre-check.org/
- mypy vs Pyright vs Pyrefly: Choosing a Python Type Checker in 2026 - https://codegym.cc/groups/posts/mypy-vs-pyright-vs-pyrefly
- Aalto University Type Checker Comparison - https://aaltodoc.aalto.fi/
- PyScan: Type Annotation Defects in Python - https://csslab-ustc.github.io/publications/2022/py3-types.pdf