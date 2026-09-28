# Test Coverage in Python: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Test coverage is the measurement of how much of a program's source code is executed during a test run. It quantifies the extent to which the test suite exercises the codebase, providing a metric for identifying untested code paths and assessing test completeness.

### Technical Definition

Test coverage in Python is typically measured using **coverage.py**, a mature and widely used tool that uses the code analysis tools and tracing hooks provided in the Python standard library to determine which lines are executable and which have been executed. Coverage.py measures statement coverage by default (the percentage of statements in your modules that the interpreter executed during tests) and can optionally track branch coverage (which paths through conditional logic were taken). Coverage data can be reported in terminal output, HTML dashboards, XML (Cobertura-compatible), and JSON formats.

### Beginner-Friendly Explanation

Test coverage tells you how much of your code your tests actually run. If your program has 100 lines and your tests execute 80 of them, you have 80% line coverage. Coverage tools like `coverage.py` track which lines ran, which didn't, and (with branch coverage) whether both sides of every `if` statement were tested. It helps you find "blind spots" in your test suite.

### Key Characteristics

- **Line/statement coverage**: Measures which executable lines were run during tests.
- **Branch coverage**: Measures whether both paths of conditional logic (if/else, loops) were taken.
- **Multiple report formats**: Terminal, HTML (with annotated source), XML (Cobertura), and JSON.
- **Configurable**: Via `.coveragerc`, `pyproject.toml`, `setup.cfg`, or `tox.ini`.
- **CI-ready**: `--fail-under` threshold causes non-zero exit when coverage drops.
- **Integration**: Works with `pytest` via `pytest-cov`, with `unittest`, and as a standalone tool.

### Prerequisites

- Python 3.8+ installed.
- `coverage` or `pytest-cov` installed (`pip install coverage` or `pip install pytest-cov`).
- A test suite (pytest or unittest) to measure.
- Basic understanding of Python control flow (if/else, loops, try/except).

### Related Programming Areas

- **Continuous Integration**: Enforcing coverage thresholds in CI/CD pipelines.
- **Test-Driven Development**: Using coverage maps to guide test writing.
- **Code review**: Identifying untested risk surfaces.
- **Quality assurance**: Measuring test suite effectiveness.

### Core Concepts / Features

---

## 1. Code Coverage

### Definitions

**Core Definition**: Code coverage is the measurement of which lines of source code are executed during a test run, expressed as a percentage of total executable lines.

**Technical Definition**: Coverage.py executes a Python program, measures which of its statements are executed and which are not, and reports these coverage measurements. By default, it measures statement coverage — the percentage of statements in your modules that the interpreter executed during the tests. It uses the code analysis tools and tracing hooks provided in the Python standard library to determine which lines are executable and which have been executed.

**Beginner-Friendly Explanation**: Code coverage tells you which lines of your program ran when you ran your tests. If a line never ran, it means no test touched that code — which could be a bug waiting to happen.

### Purposes

- To measure the percentage of source code executed by the test suite.
- To identify lines, branches, and functions that are not covered by any test.
- To provide a quantitative metric for test suite completeness.
- To guide developers toward untested areas of the codebase.
- To enforce minimum coverage standards in CI pipelines.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Install coverage.py
pip install coverage

# Run tests under coverage
coverage run -m pytest tests/

# Run tests with branch coverage
coverage run --branch -m pytest tests/

# Generate terminal report
coverage report -m

# Generate HTML report
coverage html

# Generate XML report
coverage xml
```

#### Configuration via `pyproject.toml`

```toml
[tool.coverage.run]
source = ["src"]
branch = true
omit = ["*/tests/*", "*/__init__.py"]

[tool.coverage.report]
show_missing = true
fail_under = 80
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
]
```

#### Configuration via `.coveragerc`

```ini
[run]
branch = True
source = src

[report]
show_missing = True
fail_under = 80
exclude_lines =
    pragma: no cover
    def __repr__
    if self\.debug
    raise AssertionError
    raise NotImplementedError
```

#### Syntax Rules

1. **Run command**: `coverage run -m pytest` runs the test suite under coverage measurement.
2. **Source specification**: Use `source = ["src"]` or `--source=src` to limit measurement to project code.
3. **Omit patterns**: Use `omit` to exclude test files, virtual environments, and boilerplate.
4. **`--branch`**: Enables branch coverage in addition to line coverage.
5. **Configuration precedence**: `.coveragerc` > `setup.cfg` > `tox.ini` > `pyproject.toml`.
6. **`exclude_lines`**: Regex patterns for lines to exclude from coverage (e.g., `pragma: no cover`).
7. **`fail_under`**: Minimum coverage percentage; exits with status 2 if not met.

#### Constraints and Limitations

- Coverage.py measures execution, not correctness — 100% coverage does not mean bug-free code.
- Excluding lines with `pragma: no cover` should be used judiciously.
- Coverage.py cannot measure code in compiled C extensions.
- Multi-process coverage requires `--parallel-mode` and `combine`.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Coverage Measurement

```py
# step1: Source code (calculator.py)
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def subtract(a, b):
    return a - b
```
```py
# step2: Test file (test_calculator.py)
from calculator import add, divide

def test_add():
    assert add(2, 3) == 5

def test_divide():
    assert divide(10, 2) == 5.0

# step3: Run coverage
# $ coverage run -m pytest test_calculator.py
# $ coverage report -m
```

**Expected Output**:
```
Name            Stmts   Miss  Cover   Missing
---------------------------------------------
calculator.py      10      3    70%   7-9, 14
test_calculator.py  6      0   100%
---------------------------------------------
TOTAL              16      3    81%
```

**Why**: `subtract` (line 14) and the `ValueError` branch (lines 7–9) are never executed by the tests; coverage.py identifies these as missing lines.

#### Example 2: Branch Coverage

```py
# step1: Source code (auth.py)
def validate_role(role):
    if role == "admin":
        return "full_access"
    else:
        return "limited_access"

def validate_age(age):
    if age >= 18:
        return "adult"
    return "minor"
```
```py
# step2: Test file — only tests one branch each
from auth import validate_role, validate_age

def test_admin_role():
    assert validate_role("admin") == "full_access"

def test_adult_age():
    assert validate_age(25) == "adult"

# step3: Run with branch coverage
# $ coverage run --branch -m pytest test_auth.py
# $ coverage report -m
```

**Expected Output**:
```
Name       Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------
auth.py       10      0      4      2    86%   5->7, 11->13
test_auth.py   6      0      0      0   100%
------------------------------------------------------
TOTAL         16      0      4      2    86%
```

**Why**: Both functions have 100% line coverage (all lines ran), but branch coverage is 86% because the `else` path of `validate_role` and the implicit `return "minor"` path of `validate_age` were never taken.

#### Example 3: Excluding Lines with Pragma

```py
# step1: Source code with exclusions (api.py)
def process_request(request):
    if request is None:
        raise ValueError("Request cannot be None")  # pragma: no cover
    data = request.get("data", {})
    if data.get("debug"):
        print("DEBUG MODE")  # pragma: no cover
    return {"status": "ok", "data": data}
```
```py
# step2: Test file
from api import process_request

def test_process_request():
    result = process_request({"data": {"key": "value"}})
    assert result["status"] == "ok"

# step3: Run coverage with exclude_lines
# $ coverage run -m pytest test_api.py
# $ coverage report -m
```

**Expected Output**:
```
Name     Stmts   Miss  Cover   Missing
---------------------------------------
api.py       8      0   100%
test_api.py  5      0   100%
---------------------------------------
TOTAL       13      0   100%
```

**Why**: Lines marked with `# pragma: no cover` are excluded from the coverage calculation, so the "missing" branches do not reduce the coverage percentage.

### Real-World Cases

- **CI pipelines**: Enforcing minimum coverage thresholds on pull requests.
- **Code review**: Identifying untested functions and branches before merging.
- **Legacy code**: Measuring coverage to prioritise testing efforts.
- **Library development**: Ensuring public APIs are thoroughly tested.

### References

- Coverage.py Documentation - https://coverage.readthedocs.io/
- python3.12-coverage manpage - https://manpages.debian.org/unstable/python3-coverage/python3.12-coverage.1
- Real Python: Coverage.py - https://realpython.com/python-code-coverage/
- Coverage.py on PyPI - https://pypi.org/project/coverage/

---

## 2. Branch Coverage

### Definitions

**Core Definition**: Branch coverage measures whether both the `True` and `False` paths of every conditional statement (if/else, loops, try/except) are executed during tests.

**Technical Definition**: Branch coverage measures the fraction of the branches (if/else paths) that were exercised during test execution. Coverage.py's branch coverage facility tracks which destinations are actually visited from each branch point and flags lines that haven't visited all of their possible destinations. When branch coverage is enabled, the report includes Branch (total branches), BrPart (partial branches), and Cover columns, with the Missing column detailing the missed branches.

**Beginner-Friendly Explanation**: Line coverage tells you a line ran. Branch coverage tells you whether both the "yes" and "no" answers to every `if` question were tested. If your code has `if x > 0: return "positive" else: return "negative"`, branch coverage checks that both the positive and negative paths were tested.

### Purposes

- To ensure both branches of every conditional are exercised by tests.
- To catch "coverage theater" — tests that run lines without testing all logical paths.
- To provide a more reliable metric of test completeness than line coverage alone, as it isn't biased by large blocks of simple code.
- To identify partial branches where only one side of a condition was tested.
- To guide the creation of tests for edge cases and error paths.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Enable branch coverage via CLI
coverage run --branch -m pytest

# Enable branch coverage via configuration
# [run]
# branch = True
```

#### Configuration via `pyproject.toml`

```toml
[tool.coverage.run]
branch = true
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 85
```

#### Branch Coverage in Reports

```bash
$ coverage report -m
Name            Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------------
my_module.py       15      2      4      1    84%   8, 12, 15->17
```

**Component Breakdown**:
- `Branch` — total number of branches detected.
- `BrPart` — branches that were partially executed (only one path taken).
- `Missing` — details of missed branches (e.g., `15->17` means the jump from line 15 to line 17 was never taken).

#### Syntax Rules

1. **`branch = true`**: Enables branch coverage in `.coveragerc`, `pyproject.toml`, or via `--branch`.
2. **Branch counting**: Coverage.py counts possible jumps from branch points using line transitions.
3. **Partial branches**: Reported as `BrPart`; a branch is partial when only one path is taken.
4. **Missing column**: Shows the line-to-line transitions that were not executed.
5. **Combined coverage**: The overall coverage percentage includes both line and branch coverage.
6. **Exclusions**: `pragma: no cover` excludes branches from consideration.

#### Constraints and Limitations

- Branch coverage numbers are generally lower than line coverage numbers for the same test suite.
- Coverage.py uses line jumps to count branches, which may not perfectly represent all logical branches.
- Complex expressions with `and`/`or` may not be fully tracked as branches.
- Loop branches (zero iterations vs. one or more) may not be fully captured.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Line vs Branch Coverage

```python
# step1: Source code (discount.py)
def apply_discount(price, is_member):
    if is_member:
        price = price * 0.9
    if price > 100:
        price = price - 10
    return price

# step2: Test file — line coverage satisfied, branch not
from discount import apply_discount

def test_member_discount():
    assert apply_discount(200, True) == 170.0  # Both ifs true

def test_no_discount():
    assert apply_discount(50, False) == 50.0   # Both ifs false
```

**Expected Output** (branch coverage report):
```
Name          Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------
discount.py       6      0      4      2    83%   3->5, 5->7
```

**Why**: All lines ran (line coverage 100%), but the branch `3->5` (member=True, price≤100) and `5->7` (price>100, member=False) were never taken, resulting in 83% branch coverage.

#### Example 2: Loop Branch Coverage

```python
# step1: Source code (sum_positive.py)
def sum_positive(numbers):
    total = 0
    for n in numbers:
        if n > 0:
            total += n
    return total

# step2: Test file — no negative numbers, no empty list
from sum_positive import sum_positive

def test_sum_with_positives():
    assert sum_positive([1, 2, 3]) == 6

def test_sum_with_mixed():
    assert sum_positive([-1, 5, -2, 3]) == 8
```

**Expected Output** (branch coverage report):
```
Name             Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------------
sum_positive.py      6      0      4      1    90%   3->6
```

**Why**: The branch `3->6` (loop skipped entirely when the list is empty) was never taken; the test never calls `sum_positive([])`.

#### Example 3: Full Branch Coverage

```python
# step1: Source code (grade.py)
def get_grade(score):
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    else:
        return "F"

# step2: Test file — covers all branches
from grade import get_grade

def test_grade_a(): assert get_grade(95) == "A"
def test_grade_b(): assert get_grade(85) == "B"
def test_grade_c(): assert get_grade(75) == "C"
def test_grade_f(): assert get_grade(65) == "F"
```

**Expected Output**:
```
Name       Stmts   Miss Branch BrPart  Cover   Missing
------------------------------------------------------
grade.py      10      0      6      0   100%
```

**Why**: All four branches (A, B, C, F) are tested, achieving 100% branch coverage.

### Real-World Cases

- **Business logic**: Ensuring all pricing tiers and discount paths are tested.
- **Validation**: Testing both valid and invalid inputs.
- **Error handling**: Testing both success and exception paths.
- **State machines**: Testing all state transitions.

### References

- Branch Coverage — Coverage.py Documentation - https://coverage.readthedocs.io/en/stable/branch.html
- Line or Branch Coverage — Codecov - https://about.codecov.io/blog/line-or-branch-coverage-which-type-of-coverage-is-right-for-you/

---

## 3. Coverage Reports

### Definitions

**Core Definition**: Coverage reports are structured outputs that present coverage measurements in human-readable or machine-readable formats, including terminal summaries, annotated HTML dashboards, and XML for platform integrations.

**Technical Definition**: Coverage.py provides several reporting styles: **report** (terminal summary), **html** (annotated HTML pages with source code highlighting), **annotate** (annotated source files), **json** (structured JSON), and **xml** (Cobertura-compatible XML). The HTML report creates an HTML file for each module, with source code decorated to show the status of each line — green for executed, red for missing, and gray for excluded; partial branches are yellow. The XML report writes coverage data to a `coverage.xml` file in a format compatible with Cobertura.

**Beginner-Friendly Explanation**: Coverage reports are the different ways coverage.py shows you the results. You can see a quick summary in the terminal, explore a visual HTML page with colour-coded lines, or export XML that platforms like Codecov or GitHub Actions can read.

### Purposes

- To provide a quick terminal summary of coverage percentages.
- To generate visual HTML dashboards with line-by-line annotations.
- To export structured XML for integration with coverage platforms.
- To produce JSON for programmatic consumption.
- To annotate source files with execution information.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Terminal report with missing lines
coverage report -m

# HTML report in htmlcov/ directory
coverage html

# HTML with custom output directory
coverage html -d coverage_html

# XML report (Cobertura format)
coverage xml

# JSON report
coverage json

# Skip 100% covered files
coverage report --skip-covered
```

#### Report Format Comparison

| Format | Command | Output | Best For |
|--------|---------|--------|----------|
| Terminal | `coverage report` | Text table | Quick checks, CI logs |
| HTML | `coverage html` | `htmlcov/` directory | Local exploration, debugging |
| XML | `coverage xml` | `coverage.xml` | CI platforms, Codecov |
| JSON | `coverage json` | `coverage.json` | Programmatic analysis |
| Annotate | `coverage annotate` | `,cover` files | Source-annotated output |

#### Syntax Rules

1. **`-m` / `--show-missing`**: Shows line numbers of missing statements in the terminal report.
2. **`--skip-covered`**: Omits files with 100% coverage from the report, focusing attention on problem areas.
3. **`--fail-under=MIN`**: Exits with status 2 if total coverage is less than MIN.
4. **`-d DIR`**: Specifies the output directory for HTML reports (default: `htmlcov`).
5. **`--title`**: Sets the title of the HTML report.
6. **`--precision=N`**: Controls decimal places in coverage percentages.
7. **XML format**: Cobertura-compatible; can be consumed by Codecov, Coveralls, and Jenkins.
8. **JSON format**: Structured for programmatic analysis.

#### Constraints and Limitations

- HTML reports can be time-consuming to generate for large projects; coverage.py caches data to speed up subsequent reports.
- XML output is Cobertura-compatible but not identical to all Cobertura implementations.
- Terminal reports do not show branch details unless branch coverage is enabled.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Terminal Report

```bash
# step1: Run tests with coverage
coverage run -m pytest tests/

# step2: Generate terminal report with missing lines
coverage report -m

# step3: Output
# Name                 Stmts   Miss  Cover   Missing
# ---------------------------------------------------
# src/calculator.py       20      4    80%   33-35, 39
# src/validator.py        15      2    86%   8, 12
# src/api.py              56      6    89%   17-23
# ---------------------------------------------------
# TOTAL                   91     12    87%
```

**Expected Output**:
```
Name                 Stmts   Miss  Cover   Missing
---------------------------------------------------
src/calculator.py       20      4    80%   33-35, 39
src/validator.py        15      2    86%   8, 12
src/api.py              56      6    89%   17-23
---------------------------------------------------
TOTAL                   91     12    87%
```

**Why**: The `-m` flag shows the specific line numbers that were not executed, making it easy to identify untested code.

#### Example 2: HTML Report

```bash
# step1: Generate HTML report
coverage html

# step2: Open in browser
# htmlcov/index.html

# step3: Report structure
# htmlcov/
# ├── index.html          # Summary page
# ├── src_calculator.py.html
# ├── src_validator.py.html
# └── src_api.py.html
```

**Expected Output**:
```
Writing HTML report to: htmlcov/index.html
```

**Why**: The HTML report creates a dashboard with clickable module names; each module page shows source code with green (executed), red (missing), and yellow (partial branch) highlighting.

#### Example 3: XML Report for CI

```bash
# step1: Generate XML report
coverage xml

# step2: Output
# Wrote XML report to coverage.xml

# step3: Upload to Codecov in CI
# - uses: codecov/codecov-action@v4
#   with:
#     files: ./coverage.xml
#     fail_ci_if_error: true
```

**Expected Output**:
```
Wrote XML report to coverage.xml
```

**Why**: The XML report is Cobertura-compatible and can be uploaded to Codecov, Coveralls, or other coverage platforms for tracking and visualisation.

### Real-World Cases

- **Local development**: HTML reports for exploring untested code interactively.
- **CI pipelines**: XML reports uploaded to Codecov or Coveralls.
- **Pull request checks**: Terminal reports with `--fail-under` to enforce thresholds.
- **Documentation**: JSON reports for custom dashboards and metrics.

### References

- Reporting — Coverage.py Documentation - https://coverage.readthedocs.io/en/stable/cmd.html#reporting
- HTML Reporting — Coverage.py Documentation - https://coverage.readthedocs.io/en/stable/cmd.html#html-reporting
- XML Reporting — Coverage.py Documentation - https://coverage.readthedocs.io/en/stable/cmd.html#xml-reporting

---

## 4. Coverage-Driven Improvement

### Definitions

**Core Definition**: Coverage-driven improvement is the practice of using coverage metrics to guide testing efforts, enforce quality gates in CI pipelines, and identify untested risk surfaces in a codebase.

**Technical Definition**: Coverage-driven improvement integrates coverage thresholds into CI/CD pipelines, typically via `fail_under` in configuration or `--cov-fail-under` in pytest-cov. The `--fail-under=MIN` switch causes coverage.py to exit with a status of 2 if the total coverage is less than MIN, enabling CI systems to reject pull requests that reduce coverage below the threshold. Coverage maps (HTML reports) reveal untested risk surfaces — specific lines and branches that no test exercises — allowing teams to prioritise test writing for critical paths.

**Beginner-Friendly Explanation**: Coverage-driven improvement means using coverage numbers to make decisions: "We won't merge this pull request because it drops coverage below 80%," or "Let's write tests for the functions that coverage shows are completely untested."

### Purposes

- To enforce minimum coverage thresholds in continuous integration pipelines.
- To prevent coverage regressions when new code is added.
- To identify untested risk surfaces using coverage maps.
- To prioritise testing efforts on critical, uncovered paths.
- To maintain a baseline of test completeness across the project's lifetime.

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Fail if coverage is below 80%
coverage report --fail-under=80

# pytest-cov equivalent
pytest --cov=src --cov-fail-under=80

# pytest-cov with branch coverage
pytest --cov=src --cov-branch --cov-fail-under=85
```

#### CI Integration (GitHub Actions)

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install pytest pytest-cov
      - run: pytest --cov=src --cov-branch --cov-fail-under=80 --cov-report=xml
      - uses: codecov/codecov-action@v4
        with:
          files: ./coverage.xml
```

#### Configuration via `pyproject.toml`

```toml
[tool.coverage.report]
fail_under = 80
show_missing = true
skip_covered = true
```

#### Syntax Rules

1. **`fail_under` in configuration**: Set a global minimum coverage threshold.
2. **`--fail-under=MIN`**: CLI equivalent; exits with status 2 if below MIN.
3. **`--cov-fail-under=MIN`**: pytest-cov equivalent.
4. **`--cov-branch`**: Enables branch coverage in pytest-cov.
5. **`--cov-report=xml`**: Generates XML for CI platform upload.
6. **`skip_covered = true`**: Excludes 100%-covered files from reports, focusing on gaps.
7. **Per-file thresholds**: Not natively supported; `fail_under` is aggregate only.

#### Constraints and Limitations

- `fail_under` checks aggregate coverage, not per-file coverage.
- A single high-coverage file can mask a low-coverage critical file.
- Coverage thresholds can incentivise writing trivial tests to hit numbers.
- Coverage does not guarantee correctness — 100% coverage ≠ bug-free.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: CI Coverage Gate

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -e ".[dev]"
      - run: pytest --cov=src --cov-branch --cov-fail-under=80
```

**Expected Output** (when coverage is 75%):
```
---------- coverage: platform linux, python 3.11 ----------
Name                 Stmts   Miss Branch BrPart  Cover
------------------------------------------------------
src/calculator.py       20      4      4      1    75%
------------------------------------------------------
TOTAL                   20      4      4      1    75%
FAIL Required test coverage of 80% not reached. Total coverage: 75.00%
Error: Process completed with exit code 1.
```

**Why**: The CI pipeline fails because coverage is below the 80% threshold, preventing the merge.

#### Example 2: pytest-cov with Branch Coverage

```bash
# step1: Run pytest with coverage
pytest --cov=src --cov-branch --cov-report=term-missing --cov-fail-under=85

# step2: Output
# ---------- coverage: platform linux, python 3.11 ----------
# Name                 Stmts   Miss Branch BrPart  Cover   Missing
# ---------------------------------------------------------------
# src/calculator.py       20      2      4      1    87%   33-34, 15->17
# src/validator.py        15      0      2      0   100%
# ---------------------------------------------------------------
# TOTAL                   35      2      6      1    91%
#
# Required test coverage of 85% reached. Total coverage: 91.43%
```

**Expected Output**:
```
Required test coverage of 85% reached. Total coverage: 91.43%
```

**Why**: pytest-cov reports both line and branch coverage, shows missing lines and partial branches, and enforces the 85% threshold.

#### Example 3: Coverage Map for Improvement

```bash
# step1: Generate HTML coverage map
coverage html --skip-covered

# step2: Open htmlcov/index.html
# Files with <100% coverage are shown; 100% files are hidden

# step3: Navigate to a file to see:
#   - Green lines: executed
#   - Red lines: not executed
#   - Yellow lines: partial branch (one path taken)
#   - Gray lines: excluded
```

**Expected Output**:
```
Writing HTML report to: htmlcov/index.html
```

**Why**: The HTML coverage map provides a visual guide for identifying which specific lines and branches need tests, enabling targeted coverage improvement.

### Real-World Cases

- **Open-source projects**: Enforcing coverage thresholds on pull requests.
- **Enterprise CI**: Using coverage gates to maintain quality standards.
- **Legacy modernisation**: Using coverage maps to prioritise testing of critical modules.
- **Team workflows**: Setting a project-wide baseline (e.g., 80%) and preventing regressions.

### References

- Coverage.py fail_under - https://coverage.readthedocs.io/en/stable/config.html#report
- pytest-cov Documentation - https://pytest-cov.readthedocs.io/
- Codecov — Coverage Thresholds - https://docs.codecov.com/docs/commit-status

---

## References

- Coverage.py Documentation - https://coverage.readthedocs.io/
- python3.12-coverage manpage — Debian - https://manpages.debian.org/unstable/python3-coverage/python3.12-coverage.1
- Real Python: Coverage.py - https://realpython.com/python-code-coverage/
- Coverage.py on PyPI - https://pypi.org/project/coverage/
- Branch Coverage — Coverage.py - https://coverage.readthedocs.io/en/stable/branch.html
- Reporting — Coverage.py - https://coverage.readthedocs.io/en/stable/cmd.html#reporting
- HTML Reporting — Coverage.py - https://coverage.readthedocs.io/en/stable/cmd.html#html-reporting
- XML Reporting — Coverage.py - https://coverage.readthedocs.io/en/stable/cmd.html#xml-reporting
- Configuration — Coverage.py - https://coverage.readthedocs.io/en/stable/config.html
- pytest-cov Documentation - https://pytest-cov.readthedocs.io/
- Codecov — Line or Branch Coverage - https://about.codecov.io/blog/line-or-branch-coverage-which-type-of-coverage-is-right-for-you/
- Codecov — Coverage Thresholds - https://docs.codecov.com/docs/commit-status