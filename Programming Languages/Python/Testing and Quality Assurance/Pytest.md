# Pytest: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Pytest is a mature, feature-rich testing framework for Python that allows writing tests as simple functions rather than classes, provides powerful fixtures for setup/teardown, supports parametrization for data-driven testing, and offers a rich plugin ecosystem. It is the most widely used testing framework in the Python community.

### Technical Definition

Pytest is a testing framework that implements a discovery-based architecture: it automatically finds tests following standard naming conventions (`test_*.py` files, `test_*` functions, `Test*` classes), collects them into a test session, and executes them with configurable reporting. Its core features include the `@pytest.fixture` decorator for dependency injection, `@pytest.mark.parametrize` for data-driven tests, `pytest.mark` for metadata and filtering, and assertion introspection for detailed failure reports. The framework is extensible through a plugin system that hooks into its execution lifecycle.

### Beginner-Friendly Explanation

Pytest is a tool that makes writing and running tests in Python easy. Instead of creating classes and inheriting from `unittest.TestCase`, you just write simple functions that start with `test_`. Pytest automatically finds them, runs them, and tells you which ones passed and which ones failed — with helpful error messages that show exactly what went wrong.

### Key Characteristics

- **Function-based**: Tests are plain functions, not classes; no boilerplate inheritance required.
- **Auto-discovery**: Automatically finds tests based on naming conventions.
- **Fixture system**: Powerful, modular dependency injection for setup and teardown.
- **Parametrization**: Run the same test logic with multiple input/output pairs.
- **Assertion introspection**: Detailed failure reports from plain `assert` statements.
- **Plugin ecosystem**: Extensible via community plugins like `pytest-cov`, `pytest-xdist`, and `pytest-asyncio`.
- **Backward compatible**: Can run `unittest.TestCase` tests alongside pytest-style tests.

### Prerequisites

- Python 3.8+ installed (pytest 8.x requires Python 3.8+).
- Basic understanding of Python functions, assertions, and exceptions.
- A text editor or IDE.
- `pip install pytest` to install the framework.

### Related Programming Areas

- **Test-Driven Development (TDD)**: Writing tests before implementation.
- **Continuous Integration**: Running pytest automatically on every commit.
- **Mocking**: Using `pytest-mock` or `unittest.mock` for test doubles.
- **Coverage**: Measuring test coverage with `pytest-cov`.
- **Asynchronous testing**: Testing `async def` functions with `pytest-asyncio`.

### Core Concepts / Features

---

## 1. Test Functions

### Definitions

**Core Definition**: A test function is a Python function whose name begins with `test_`, which pytest automatically discovers and executes as a test case.

**Technical Definition**: Pytest allows writing tests as simple Python functions prefixed with `test_`. Unlike `unittest`, which requires subclassing `unittest.TestCase`, pytest test functions require no inheritance and no special base class. The `test_` prefix ensures that pytest collects and executes the function. Test functions can accept fixture arguments, use plain `assert` statements, and be organized into classes (named `Test*`) for grouping related tests.

**Beginner-Friendly Explanation**: A test function is just a regular Python function that starts with `test_`. You write your test logic inside it, and pytest finds it and runs it automatically. No need to create classes or inherit from anything.

### Purposes

- To write tests as simple, readable Python functions without boilerplate.
- To enable automatic test discovery based on naming conventions.
- To allow tests to be grouped into classes when logical organization is helpful.
- To support fixture injection through function arguments.
- To keep tests concise and focused on assertions.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# test_sample.py

def test_simple_assertion():
    assert 1 + 1 == 2

def test_with_fixture(tmp_path):
    file = tmp_path / "test.txt"
    file.write_text("hello")
    assert file.read_text() == "hello"

class TestGroup:
    def test_one(self):
        assert "h" in "this"

    def test_two(self):
        assert hasattr("hello", "check") is False
```

**Component Breakdown**:
- `def test_*():` — a test function; the `test_` prefix is mandatory for discovery.
- `class Test*:` — a test class for grouping related tests.
- Fixture arguments — the function can declare fixtures as parameters.

#### Syntax Rules

1. **Function naming**: Test functions must start with `test_`.
2. **No inheritance required**: Unlike `unittest`, pytest test functions do not need to inherit from any base class.
3. **Class naming**: Test classes must start with `Test` and cannot have an `__init__` method.
4. **Method naming**: Test methods within classes must start with `test_`.
5. **No `__init__` in test classes**: Pytest does not support `__init__` in test classes; use fixtures instead.
6. **Assertions**: Use plain `assert` statements; pytest rewrites them for detailed failure reports.
7. **Async support**: With `pytest-asyncio`, async test functions can be written as `async def test_*()`.

#### Constraints and Limitations

- Test classes cannot have `__init__` methods.
- Test functions must be at module level or inside `Test*` classes.
- Test functions cannot return non-`None` values (pytest ignores return values).
- Mixing `unittest.TestCase` and pytest-style functions in the same module is supported but discouraged.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Test Functions

```python
# step1: Source code (calculator.py)
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# step2: Test file (test_calculator.py)
from calculator import add, divide

def test_add_positive():
    """Test addition with positive integers."""
    assert add(2, 3) == 5

def test_add_negative():
    """Test addition with negative integers."""
    assert add(-1, -1) == -2

def test_divide_normal():
    """Test normal division."""
    assert divide(10, 2) == 5.0

def test_divide_by_zero_raises():
    """Test that division by zero raises ValueError."""
    import pytest
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

# step3: Run
# $ pytest test_calculator.py -v
```

**Expected Output**:
```
test_calculator.py::test_add_positive PASSED
test_calculator.py::test_add_negative PASSED
test_calculator.py::test_divide_normal PASSED
test_calculator.py::test_divide_by_zero_raises PASSED
```

**Why**: Each test function verifies one behaviour; `pytest.raises` checks that the expected exception is raised.

#### Example 2: Grouping Tests in a Class

```python
# step1: Test class (test_strings.py)
class TestStringMethods:
    def test_upper(self):
        assert "foo".upper() == "FOO"

    def test_isupper(self):
        assert "FOO".isupper() is True
        assert "Foo".isupper() is False

    def test_split(self):
        s = "hello world"
        assert s.split() == ["hello", "world"]
        with __import__("pytest").raises(TypeError):
            s.split(2)
```

**Expected Output**:
```
test_strings.py::TestStringMethods::test_upper PASSED
test_strings.py::TestStringMethods::test_isupper PASSED
test_strings.py::TestStringMethods::test_split PASSED
```

**Why**: The `TestStringMethods` class groups related string tests; pytest discovers methods prefixed with `test_`.

#### Example 3: Tests with Fixture Arguments

```python
# step1: Test using the built-in tmp_path fixture
def test_create_file(tmp_path):
    file = tmp_path / "data.txt"
    file.write_text("content")
    assert file.read_text() == "content"
    assert file.exists()
```

**Expected Output**:
```
test_file.py::test_create_file PASSED
```

**Why**: The `tmp_path` fixture provides a temporary directory unique to the test; no manual setup or teardown is needed.

### Real-World Cases

- **Library functions**: Testing utility functions and helpers.
- **Business logic**: Testing pricing rules, validators, and calculators.
- **Data transformations**: Testing parsers and serializers.
- **API endpoints**: Testing request/response behaviour with `pytest-flask` or `httpx`.

### References

- Get Started — pytest documentation - https://docs.pytest.org/en/stable/getting-started.html
- Good Integration Practices — pytest documentation - https://docs.pytest.org/en/stable/explanation/goodpractices.html

---

## 2. Fixtures

### Definitions

**Core Definition**: A fixture is a function that provides a defined, reliable, and consistent context for tests, such as test data, environment variables, database connections, or files needed before the test runs.

**Technical Definition**: In pytest, fixtures are functions decorated with `@pytest.fixture`. They define the steps and data that constitute the arrange phase of a test. Fixtures are requested by test functions through arguments: for each fixture used, there is a parameter in the test function's definition. Fixtures can request other fixtures, are modular and reusable, and support scoped lifecycles (function, class, module, session) and teardown via `yield`.

**Beginner-Friendly Explanation**: A fixture is a reusable setup function. If multiple tests need the same data or environment, you write a fixture once and use it in all those tests by adding it as an argument. Fixtures can also clean up after themselves using `yield`.

### Purposes

- To provide a defined, reliable context for tests (arrange phase).
- To enable dependency injection: tests declare what they need as arguments.
- To make setup code modular, reusable, and explicit.
- To support scoped lifecycles: create once per function, class, module, or session.
- To manage teardown/cleanup safely via `yield`.
- To allow fixtures to depend on other fixtures, creating complex setups from simple building blocks.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import pytest

@pytest.fixture
def simple_fixture():
    return "value"

@pytest.fixture(scope="module")
def module_fixture():
    resource = create_resource()
    yield resource
    resource.close()

@pytest.fixture(autouse=True)
def auto_fixture():
    setup()
    yield
    teardown()
```

**Component Breakdown**:
- `@pytest.fixture` — decorator that marks a function as a fixture.
- `scope` — controls lifecycle: `"function"` (default), `"class"`, `"module"`, `"package"`, `"session"`.
- `autouse=True` — fixture runs automatically for all tests in its scope.
- `yield` — separates setup from teardown; code after `yield` runs during teardown.

#### Fixture Scopes

| Scope | Lifecycle |
|-------|-----------|
| `function` | Created and destroyed for each test (default) |
| `class` | Created once per test class |
| `module` | Created once per module |
| `package` | Created once per package |
| `session` | Created once per pytest session |

#### Syntax Rules

1. **Decorator required**: Fixtures must be decorated with `@pytest.fixture`.
2. **Requesting**: Tests and fixtures request fixtures by declaring them as arguments.
3. **Yield for teardown**: Use `yield` to provide a value and register cleanup code.
4. **Teardown runs even on failure**: Code after `yield` runs even if the test fails.
5. **Fixture dependencies**: Fixtures can request other fixtures, enabling modular composition.
6. **Autouse**: `@pytest.fixture(autouse=True)` runs the fixture for every test in its scope without being explicitly requested.
7. **`conftest.py`**: Fixtures defined in `conftest.py` are available to all tests in that directory and its subdirectories.
8. **Scope**: Broader scopes improve performance but share state for longer.

#### Constraints and Limitations

- If a yield fixture raises before reaching `yield`, the teardown code after `yield` does not run.
- Fixtures cannot be called directly (must be requested as arguments).
- Scope mismatches: a function-scoped fixture cannot request a session-scoped fixture in certain configurations.
- `autouse` fixtures hide test dependencies and should be used sparingly.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Fixture

```python
import pytest

# step1: Define a fixture
@pytest.fixture
def sample_data():
    return [1, 2, 3, 4, 5]

# step2: Request the fixture in a test
def test_sum(sample_data):
    assert sum(sample_data) == 15

def test_length(sample_data):
    assert len(sample_data) == 5
```

**Expected Output**:
```
test_fixture.py::test_sum PASSED
test_fixture.py::test_length PASSED
```

**Why**: The `sample_data` fixture provides a list to both tests; each test receives a fresh copy because the default scope is `function`.

#### Example 2: Fixture with Yield (Teardown)

```python
import pytest

@pytest.fixture
def database():
    # Setup
    db = create_test_database()
    db.connect()
    yield db
    # Teardown
    db.disconnect()
    db.destroy()

def test_query(database):
    result = database.execute("SELECT 1")
    assert result == 1

def test_insert(database):
    database.execute("INSERT INTO users VALUES (1, 'Alice')")
    assert database.count("users") == 1
```

**Expected Output**:
```
test_database.py::test_query PASSED
test_database.py::test_insert PASSED
```

**Why**: The `database` fixture sets up a connection before each test and tears it down after; the teardown code runs even if a test fails.

#### Example 3: Scoped Fixtures

```python
import pytest

@pytest.fixture(scope="module")
def shared_resource():
    print("\n[setup] creating shared resource")
    resource = {"data": []}
    yield resource
    print("[teardown] destroying shared resource")

def test_add_to_shared(shared_resource):
    shared_resource["data"].append("a")
    assert len(shared_resource["data"]) == 1

def test_shared_persists(shared_resource):
    assert len(shared_resource["data"]) == 1  # Same resource
```

**Expected Output**:
```
[setup] creating shared resource
test_fixture.py::test_add_to_shared PASSED
test_fixture.py::test_shared_persists PASSED
[teardown] destroying shared resource
```

**Why**: The `scope="module"` fixture is created once for the entire module and shared across all tests; teardown runs after the last test.

### Real-World Cases

- **Database connections**: Creating connections once per session and cleaning up.
- **Temporary files**: Using `tmp_path` for isolated file operations.
- **API clients**: Setting up authenticated clients for tests.
- **Configuration**: Loading test configuration from environment variables.

### References

- About fixtures — pytest documentation - https://docs.pytest.org/en/stable/explanation/fixtures.html
- How to use fixtures — pytest documentation - https://docs.pytest.org/en/stable/how-to/fixtures.html
- Pytest fixtures — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/python-advanced-pytest/4-fixtures

---

## 3. Parameterization

### Definitions

**Core Definition**: Parameterization is the technique of running the same test function multiple times with different input values and expected outputs, using the `@pytest.mark.parametrize` decorator.

**Technical Definition**: The `@pytest.mark.parametrize` decorator defines multiple sets of arguments and fixtures at the test function or class level. Each set of arguments is passed to the test function in turn, and pytest reports each as a separate test item. The decorator takes two required arguments: `argnames` (a string of comma-separated parameter names) and `argvalues` (a list of tuples or values). Parameter values are passed as-is to tests without copying.

**Beginner-Friendly Explanation**: Parametrization lets you run the same test with different inputs without writing duplicate tests. You write the test once, provide a list of input/expected pairs, and pytest runs the test once for each pair.

### Purposes

- To eliminate redundant test loops by feeding multiple data inputs into a single test.
- To reduce code duplication when the test logic is the same but inputs differ.
- To enable pytest to report each input as a separate test item, making failures easier to identify.
- To support data-driven testing where test cases are defined as data.
- To combine with fixtures for complex parametrized setups.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import pytest

@pytest.mark.parametrize("input, expected", [
    (1, 1),
    (2, 4),
    (3, 9),
])
def test_square(input, expected):
    assert input ** 2 == expected
```

**Component Breakdown**:
- `@pytest.mark.parametrize` — the decorator.
- `"input, expected"` — comma-separated parameter names as a string.
- `[(1, 1), (2, 4), (3, 9)]` — list of tuples, one per test case.

#### Syntax Rules

1. **Two required arguments**: `argnames` (string) and `argvalues` (iterable).
2. **Single parameter**: Use a string name and a list of values: `@pytest.mark.parametrize("x", [1, 2, 3])`.
3. **Multiple parameters**: Use comma-separated names and a list of tuples.
4. **Parameter values passed as-is**: No copying; mutations affect subsequent calls.
5. **IDs**: Pytest generates test IDs from parameter values; custom IDs can be provided with the `ids` parameter.
6. **Stacking**: Multiple parametrize decorators can be stacked for Cartesian products.
7. **Indirect parametrization**: Use `indirect=True` to parametrize fixtures.

#### Constraints and Limitations

- Parameter values are not copied; mutable values mutated by tests affect subsequent test cases.
- Pytest escapes non-ASCII characters in parametrization IDs by default.
- Stacked decorators produce a Cartesian product of all parameter sets.
- Large parameter sets can significantly increase test suite runtime.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Parametrization

```python
import pytest

@pytest.mark.parametrize("test_input, expected", [
    ("3+5", 8),
    ("2+4", 6),
    ("6*9", 42),
])
def test_eval(test_input, expected):
    assert eval(test_input) == expected
```

**Expected Output**:
```
test_expectation.py::test_eval[3+5-8] PASSED
test_expectation.py::test_eval[2+4-6] PASSED
test_expectation.py::test_eval[6*9-42] FAILED
```

**Why**: The test runs three times with different inputs; the third case fails because `6*9` evaluates to 54, not 42.

#### Example 2: Single Parameter

```python
import pytest

@pytest.mark.parametrize("number", [0, 2, 4, 6, 8])
def test_even(number):
    assert number % 2 == 0

@pytest.mark.parametrize("text", ["hello", "world", "pytest"])
def test_non_empty(text):
    assert len(text) > 0
```

**Expected Output**:
```
test_param.py::test_even[0] PASSED
test_param.py::test_even[2] PASSED
test_param.py::test_even[4] PASSED
test_param.py::test_even[6] PASSED
test_param.py::test_even[8] PASSED
test_param.py::test_non_empty[hello] PASSED
test_param.py::test_non_empty[world] PASSED
test_param.py::test_non_empty[pytest] PASSED
```

**Why**: A single parameter uses a list of values; each value generates a separate test case.

#### Example 3: Stacked Parametrization

```python
import pytest

@pytest.mark.parametrize("x", [0, 1])
@pytest.mark.parametrize("y", [2, 3])
def test_cartesian(x, y):
    assert x < y
```

**Expected Output**:
```
test_cartesian.py::test_cartesian[2-0] PASSED
test_cartesian.py::test_cartesian[2-1] PASSED
test_cartesian.py::test_cartesian[3-0] PASSED
test_cartesian.py::test_cartesian[3-1] PASSED
```

**Why**: Stacked decorators produce a Cartesian product: 2 values of `x` × 2 values of `y` = 4 test cases.

### Real-World Cases

- **Input validation**: Testing validators with valid and invalid inputs.
- **Mathematical functions**: Testing with boundary values and edge cases.
- **API responses**: Testing different HTTP status codes and payloads.
- **String processing**: Testing parsers with various input formats.

### References

- How to parametrize fixtures and test functions — pytest documentation - https://docs.pytest.org/en/stable/how-to/parametrize.html
- Parametrizing tests — pytest documentation - https://docs.pytest.org/en/stable/example/parametrize.html

---

## 4. Markers

### Definitions

**Core Definition**: Markers are metadata attributes that can be applied to test functions and classes using the `@pytest.mark` decorator to categorize, filter, and control test execution.

**Technical Definition**: The `pytest.mark` helper allows setting metadata on test functions. Built-in markers include `skip`, `skipif`, `xfail`, `parametrize`, `usefixtures`, and `filterwarnings`. Custom markers can be created and registered for project-specific categorization. Markers are used by plugins and for test selection on the command line with the `-m` option.

**Beginner-Friendly Explanation**: Markers are labels you can put on tests. You can use built-in markers like `skip` to skip a test, `xfail` to mark a test as expected to fail, or create custom markers like `@pytest.mark.slow` to categorize tests. Then you can run only tests with specific markers.

### Purposes

- To categorize tests (e.g., `@pytest.mark.slow`, `@pytest.mark.regression`).
- To skip tests conditionally (`@pytest.mark.skip`, `@pytest.mark.skipif`).
- To mark tests as expected to fail (`@pytest.mark.xfail`).
- To parametrize tests (`@pytest.mark.parametrize`).
- To filter test execution from the command line (`pytest -m "not slow"`).
- To register custom markers and prevent warnings for unknown marks.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import pytest

# Built-in markers
@pytest.mark.skip(reason="Not implemented yet")
def test_skipped():
    pass

@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_conditional_skip():
    pass

@pytest.mark.xfail(reason="Known bug #123")
def test_expected_failure():
    assert False

@pytest.mark.parametrize("x", [1, 2, 3])
def test_parametrized(x):
    pass

# Custom markers (must be registered)
@pytest.mark.slow
def test_slow_operation():
    pass

@pytest.mark.regression
def test_bug_fix():
    pass
```

#### Built-in Markers

| Marker | Purpose |
|--------|---------|
| `skip` | Always skip the test |
| `skipif` | Skip the test if a condition is met |
| `xfail` | Produce an "expected failure" outcome |
| `parametrize` | Perform multiple calls with different arguments |
| `usefixtures` | Use fixtures on a test function or class |
| `filterwarnings` | Filter warnings for a test function |

#### Syntax Rules

1. **Built-in markers**: `skip`, `skipif`, `xfail`, `parametrize`, `usefixtures`, `filterwarnings`.
2. **Custom markers must be registered**: Use `pytest.ini` or `pyproject.toml` to register custom markers and avoid warnings.
3. **`strict_markers`**: Set `strict_markers = true` to treat unknown marks as errors.
4. **Test selection**: Use `-m` to select or deselect tests by marker: `pytest -m "slow"`, `pytest -m "not slow"`.
5. **Marks apply to tests only**: Marks have no effect on fixtures.
6. **`pytest --markers`**: List all available markers (built-in and custom).

#### Constraints and Limitations

- Unregistered custom markers emit warnings.
- Marks cannot be applied to fixtures.
- Marker names should be descriptive and consistent.
- Stacking markers is allowed; all conditions apply.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Skip and Skipif

```python
import sys
import pytest

@pytest.mark.skip(reason="Feature not implemented")
def test_unimplemented():
    assert False

@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_requires_python_310():
    assert True

def test_always_runs():
    assert True
```

**Expected Output**:
```
test_skip.py::test_unimplemented SKIPPED
test_skip.py::test_requires_python_310 SKIPPED
test_skip.py::test_always_runs PASSED
```

**Why**: `skip` always skips; `skipif` skips only when the condition is true.

#### Example 2: Expected Failure (xfail)

```python
import pytest

@pytest.mark.xfail(reason="Known bug #123")
def test_known_bug():
    assert 1 == 2  # Expected to fail

@pytest.mark.xfail
def test_expected_to_fail():
    raise RuntimeError("Something went wrong")
```

**Expected Output**:
```
test_xfail.py::test_known_bug XFAIL
test_xfail.py::test_expected_to_fail XFAIL
```

**Why**: `xfail` marks tests as expected to fail; if they pass, they are reported as XPASS.

#### Example 3: Custom Markers

```python
# step1: Register markers in pytest.ini
# [pytest]
# markers =
#     slow: marks tests as slow
#     regression: marks tests as regression tests

import pytest

@pytest.mark.slow
def test_slow_operation():
    import time
    time.sleep(0.1)
    assert True

@pytest.mark.regression
def test_old_bug_fix():
    assert True

@pytest.mark.slow
@pytest.mark.regression
def test_slow_regression():
    assert True
```

**Expected Output** (running `pytest -m "slow"`):
```
test_markers.py::test_slow_operation PASSED
test_markers.py::test_slow_regression PASSED
```

**Why**: Custom markers categorize tests; `-m "slow"` selects only tests marked as slow.

### Real-World Cases

- **CI pipelines**: Running fast tests on every commit and slow tests nightly.
- **Platform-specific tests**: Skipping Windows-only tests on Linux.
- **Known bugs**: Marking tests as `xfail` until the bug is fixed.
- **Test categorization**: Separating unit, integration, and end-to-end tests.

### References

- How to mark test functions with attributes — pytest documentation - https://docs.pytest.org/en/stable/how-to/mark.html
- Skip and xfail: dealing with tests that cannot succeed — pytest documentation - https://docs.pytest.org/en/stable/how-to/skipping.html

---

## 5. Assertions

### Definitions

**Core Definition**: Pytest uses Python's standard `assert` statement for verifying expectations in tests, enhanced by intelligent assertion introspection that reports intermediate values on failure.

**Technical Definition**: Pytest allows using the standard Python `assert` statement for verifying expectations and values. When an assertion fails, pytest's advanced assertion introspection intelligently reports intermediate values of the assert expression, including calls, attributes, comparisons, and binary/unary operators. For exceptions, `pytest.raises()` is used as a context manager. For approximate equality, `pytest.approx()` handles floating-point comparisons.

**Beginner-Friendly Explanation**: Instead of using special assertion methods like `assertEqual()`, you just write `assert result == expected`. If the assertion fails, pytest shows you exactly what the values were, making it easy to debug.

### Purposes

- To verify test expectations using idiomatic Python `assert` statements.
- To get detailed failure reports with intermediate values without boilerplate.
- To assert that specific exceptions are raised using `pytest.raises`.
- To compare floating-point values with `pytest.approx`.
- To avoid the many specialized assertion methods required by `unittest`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
import pytest

# Basic assertion
assert expression, "optional message"

# Exception assertion
with pytest.raises(ExceptionType, match="pattern"):
    code_that_raises()

# Approximate equality
assert (0.1 + 0.2) == pytest.approx(0.3)

# Warning assertion
with pytest.warns(UserWarning):
    warn("message")
```

**Component Breakdown**:
- `assert expression` — verifies that the expression is truthy.
- `pytest.raises(ExceptionType)` — context manager that asserts the exception is raised.
- `match` — optional regex pattern to match the exception message.
- `pytest.approx()` — compares floating-point values with tolerance.

#### Syntax Rules

1. **Plain `assert`**: Use `assert` for all non-exception assertions.
2. **Failure message**: An optional message can be provided after a comma.
3. **`pytest.raises`**: Use as a context manager to assert exceptions.
4. **`match` parameter**: Use a regex to verify the exception message.
5. **`pytest.approx`**: For floating-point comparisons; works with scalars, lists, dicts, and NumPy arrays.
6. **`pytest.warns`**: For asserting that warnings are emitted.
7. **Assertion introspection**: Pytest rewrites assertions to provide detailed failure output.

#### Constraints and Limitations

- `assert` statements can be disabled with the `-O` optimization flag; pytest's assertion rewriting prevents this for test files.
- `pytest.approx` requires the `pytest` package (not available in plain Python).
- Assertion rewriting only applies to test files and plugins, not to application code.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Assertions

```python
def test_basic_assertions():
    assert 1 + 1 == 2
    assert "hello".upper() == "HELLO"
    assert [1, 2, 3] == [1, 2, 3]
    assert {"a": 1} == {"a": 1}

def test_with_message():
    result = 5
    assert result == 5, f"Expected 5, got {result}"
```

**Expected Output**:
```
test_assertions.py::test_basic_assertions PASSED
test_assertions.py::test_with_message PASSED
```

**Why**: Plain `assert` statements work as expected; pytest provides introspection on failure.

#### Example 2: Exception Assertions

```python
import pytest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def test_divide_by_zero():
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

def test_divide_negative():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)  # Wrong exception type
```

**Expected Output**:
```
test_assertions.py::test_divide_by_zero PASSED
test_assertions.py::test_divide_negative FAILED
```

**Why**: `pytest.raises` verifies the exception type and optionally the message; the second test fails because the wrong exception is expected.

#### Example 3: Approximate Equality

```python
import pytest

def test_floats():
    assert (0.1 + 0.2) == pytest.approx(0.3)

def test_arrays():
    import numpy as np
    a = np.array([1.0, 2.0, 3.0])
    b = np.array([0.9999, 2.0001, 3.0])
    assert a == pytest.approx(b, abs=1e-3)

def test_lists():
    assert [1.0, 2.0] == pytest.approx([1.0001, 2.0002], rel=1e-3)
```

**Expected Output**:
```
test_assertions.py::test_floats PASSED
test_assertions.py::test_arrays PASSED
test_assertions.py::test_lists PASSED
```

**Why**: `pytest.approx` handles floating-point rounding errors with configurable tolerance.

### Real-World Cases

- **Numeric computations**: Testing floating-point results with `pytest.approx`.
- **Input validation**: Testing that validators raise appropriate exceptions.
- **API responses**: Testing status codes and error messages.
- **Data processing**: Testing transformations with approximate equality.

### References

- How to write and report assertions in tests — pytest documentation - https://docs.pytest.org/en/stable/how-to/assert.html
- Get Started (Assertions) — pytest documentation - https://docs.pytest.org/en/stable/getting-started.html

---

## 6. Plugins

### Definitions

**Core Definition**: Pytest plugins are third-party packages that extend pytest's functionality, adding features such as coverage reporting, parallel execution, asynchronous test support, and more.

**Technical Definition**: Pytest's plugin system allows third-party packages to hook into pytest's execution lifecycle. Plugins are installed via `pip` and are automatically discovered and integrated — no explicit activation is required. Popular plugins include `pytest-cov` (coverage reporting), `pytest-xdist` (parallel/distributed testing), `pytest-asyncio` (asynchronous test support), `pytest-django` (Django integration), `pytest-bdd` (Behaviour-Driven Development), and `pytest-timeout` (test timeouts).

**Beginner-Friendly Explanation**: Plugins are add-ons that give pytest extra abilities. If you want to measure test coverage, install `pytest-cov`. If you want to run tests faster by using multiple CPU cores, install `pytest-xdist`. If you're testing async functions, install `pytest-asyncio`. Just `pip install` the plugin and pytest automatically finds it.

### Purposes

- To add coverage reporting (`pytest-cov`).
- To enable parallel and distributed test execution (`pytest-xdist`).
- To support asynchronous test functions (`pytest-asyncio`).
- To integrate with web frameworks (`pytest-django`, `pytest-flask`).
- To write BDD-style tests (`pytest-bdd`).
- To enforce test timeouts (`pytest-timeout`).
- To report failures immediately (`pytest-instafail`).

### Syntax Rules and Structure

#### Complete General Syntax

```bash
# Install a plugin
pip install pytest-cov
pip install pytest-xdist
pip install pytest-asyncio

# Use plugin features
pytest --cov=myproj tests/
pytest -n auto
pytest --asyncio-mode=auto
```

#### Common Plugins

| Plugin | Purpose | Usage |
|--------|---------|-------|
| `pytest-cov` | Coverage reporting | `pytest --cov=myproj` |
| `pytest-xdist` | Parallel execution | `pytest -n auto` |
| `pytest-asyncio` | Async test support | `@pytest.mark.asyncio` |
| `pytest-django` | Django integration | `pytest --ds=settings` |
| `pytest-bdd` | BDD/Gherkin | Feature files |
| `pytest-timeout` | Test timeouts | `@pytest.mark.timeout(10)` |
| `pytest-instafail` | Immediate failure reporting | `pytest --instafail` |

#### Syntax Rules

1. **Installation**: `pip install pytest-NAME`.
2. **Automatic discovery**: Installed plugins are automatically found; no activation needed.
3. **Configuration**: Plugin options are set in `pytest.ini`, `pyproject.toml`, or via CLI.
4. **`pytest_plugins`**: Can require plugins in a test module or `conftest.py`.
5. **Finding active plugins**: Use `pytest --trace-config` or `pytest --version` to see installed plugins.

#### Constraints and Limitations

- Plugin conflicts may occur; check compatibility.
- Some plugins require additional configuration.
- `pytest-asyncio` requires either `@pytest.mark.asyncio` or `asyncio_mode = auto`.
- `pytest-cov` with `pytest-xdist` requires special handling for coverage combination.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: pytest-cov (Coverage)

```bash
# step1: Install
pip install pytest-cov

# step2: Run tests with coverage
pytest --cov=myproj tests/

# step3: Output
# ---------- coverage: platform linux, python 3.11 ----------
# Name                    Stmts   Miss  Cover
# -------------------------------------------
# myproj/__init__.py          2      0   100%
# myproj/core.py             45      5    89%
# myproj/utils.py            23      0   100%
# -------------------------------------------
# TOTAL                      70      5    93%
```

**Expected Output**:
```
---------- coverage: platform linux, python 3.11 ----------
Name                    Stmts   Miss  Cover
-------------------------------------------
myproj/__init__.py          2      0   100%
myproj/core.py             45      5    89%
myproj/utils.py            23      0   100%
-------------------------------------------
TOTAL                      70      5    93%
```

**Why**: `pytest-cov` measures which lines of code are executed during tests and reports coverage percentages.

#### Example 2: pytest-xdist (Parallel Execution)

```bash
# step1: Install
pip install pytest-xdist

# step2: Run tests in parallel
pytest -n auto

# step3: Output
# =========================== test session starts ===========================
# platform linux -- Python 3.11.x, pytest-8.x.x, pluggy-1.x.x
# gw0 [100] / gw1 [100] / gw2 [100] / gw3 [100]
# =========================== 400 passed in 2.34s ============================
```

**Expected Output**:
```
gw0 [100] / gw1 [100] / gw2 [100] / gw3 [100]
400 passed in 2.34s
```

**Why**: `-n auto` distributes tests across all available CPU cores, reducing total execution time.

#### Example 3: pytest-asyncio

```python
# step1: Install
# pip install pytest-asyncio

# step2: Configure in pytest.ini
# [pytest]
# asyncio_mode = auto

# step3: Write async tests
import pytest

async def fetch_data():
    return {"status": "ok"}

@pytest.mark.asyncio
async def test_fetch_data():
    result = await fetch_data()
    assert result["status"] == "ok"

async def test_async_operation():
    assert True
```

**Expected Output**:
```
test_async.py::test_fetch_data PASSED
test_async.py::test_async_operation PASSED
```

**Why**: With `asyncio_mode = auto`, pytest-asyncio automatically handles `async def` test functions.

### Real-World Cases

- **CI pipelines**: `pytest-cov` for coverage gates; `pytest-xdist` for faster CI runs.
- **Async applications**: `pytest-asyncio` for FastAPI, aiohttp, and asyncio-based code.
- **Web frameworks**: `pytest-django`, `pytest-flask` for framework-specific testing.
- **BDD teams**: `pytest-bdd` for Gherkin-style specifications.

### References

- How to install and use plugins — pytest documentation - https://docs.pytest.org/en/stable/how-to/plugins.html
- pytest-cov Documentation - https://pytest-cov.readthedocs.io/
- pytest-xdist Documentation - https://pytest-xdist.readthedocs.io/
- pytest-asyncio Documentation - https://pytest-asyncio.readthedocs.io/

---

## 7. Test Discovery

### Definitions

**Core Definition**: Test discovery is the process by which pytest automatically finds test files, test functions, test classes, and test methods in a project based on standard naming conventions.

**Technical Definition**: Pytest implements standard test discovery: if no arguments are specified, collection starts from `testpaths` (if configured) or the current directory. It recurses into directories unless they match `norecursedirs`, searching for `test_*.py` or `*_test.py` files. From those files, it collects test-prefixed functions and methods, including those inside `Test`-prefixed classes (without an `__init__` method). Naming conventions can be customized via `python_files`, `python_classes`, and `python_functions` configuration options.

**Beginner-Friendly Explanation**: Pytest automatically finds your tests. It looks for files named `test_*.py` or `*_test.py`, and within those files, it looks for functions and methods that start with `test_`. You don't have to tell pytest where your tests are — it finds them for you.

### Purposes

- To automatically find and collect tests without manual registration.
- To enforce consistent naming conventions across a project.
- To allow customization of discovery rules for non-standard layouts.
- To exclude directories (e.g., `.git`, `.venv`) from test collection.
- To enable running specific tests via file paths, node IDs, or directory arguments.

### Syntax Rules and Structure

#### Complete General Syntax

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py *_test.py
python_classes = Test*
python_functions = test_*
norecursedirs = .git .venv build dist
```

**Component Breakdown**:
- `testpaths` — directories to search for tests.
- `python_files` — file name patterns for test discovery.
- `python_classes` — class name patterns.
- `python_functions` — function/method name patterns.
- `norecursedirs` — directories to exclude from collection.

#### Discovery Rules

| Element | Default Pattern |
|---------|-----------------|
| Test files | `test_*.py` or `*_test.py` |
| Test functions | `test_*` |
| Test classes | `Test*` (no `__init__`) |
| Test methods | `test_*` |

#### Syntax Rules

1. **File patterns**: `test_*.py` or `*_test.py`.
2. **Function patterns**: Functions prefixed with `test_`.
3. **Class patterns**: Classes prefixed with `Test` (no `__init__` method).
4. **Method patterns**: Methods prefixed with `test_`.
5. **Recursion**: Pytest recurses into directories unless they match `norecursedirs`.
6. **Customization**: Override defaults with `python_files`, `python_classes`, `python_functions`.
7. **`conftest.py`**: Fixtures and hooks in `conftest.py` are available to tests in that directory and below.
8. **unittest compatibility**: `unittest.TestCase` subclasses are also discovered.

#### Constraints and Limitations

- Test classes cannot have `__init__` methods.
- Files not matching the patterns are ignored.
- Directories matching `norecursedirs` are skipped.
- Custom patterns require configuration.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Standard Test Discovery

```bash
# Project structure:
# myproject/
# ├── src/
# │   └── mypkg/
# │       └── __init__.py
# └── tests/
#     ├── test_module_a.py
#     └── test_module_b.py

# step1: Run pytest from project root
$ pytest

# step2: Output
# =========================== test session starts ===========================
# collected 10 items
# tests/test_module_a.py ......
# tests/test_module_b.py ....
# =========================== 10 passed in 0.12s ============================
```

**Expected Output**:
```
collected 10 items
tests/test_module_a.py ......
tests/test_module_b.py ....
10 passed in 0.12s
```

**Why**: Pytest discovers all `test_*.py` files in the `tests` directory and collects all `test_*` functions.

#### Example 2: Custom Discovery Patterns

```ini
# pytest.ini
[pytest]
testpaths = specs
python_files = check_*.py
python_classes = Check*
python_functions = check_*
```

```python
# specs/check_math.py
class CheckArithmetic:
    def check_addition(self):
        assert 1 + 1 == 2

    def check_subtraction(self):
        assert 5 - 3 == 2

def check_multiplication():
    assert 2 * 3 == 6
```

**Expected Output**:
```
specs/check_math.py::CheckArithmetic::check_addition PASSED
specs/check_math.py::CheckArithmetic::check_subtraction PASSED
specs/check_math.py::check_multiplication PASSED
```

**Why**: Custom patterns override defaults; pytest discovers `check_*.py` files, `Check*` classes, and `check_*` functions.

#### Example 3: Running Specific Tests

```bash
# step1: Run a specific file
pytest tests/test_module_a.py

# step2: Run a specific test function
pytest tests/test_module_a.py::test_specific_function

# step3: Run tests matching a keyword
pytest -k "addition or subtraction"

# step4: Run tests by marker
pytest -m "slow"
```

**Expected Output** (for `-k "addition"`):
```
tests/test_module_a.py::test_addition PASSED
tests/test_module_a.py::test_addition_negative PASSED
```

**Why**: Pytest supports selective execution by file, node ID, keyword expression, or marker.

### Real-World Cases

- **Large projects**: Customizing `testpaths` and `norecursedirs` for efficient discovery.
- **Non-standard layouts**: Using `python_files` and `python_classes` for custom naming.
- **CI pipelines**: Running specific test subsets based on markers or keywords.
- **Monorepos**: Using `conftest.py` for shared fixtures across packages.

### References

- Good Integration Practices — pytest documentation - https://docs.pytest.org/en/stable/explanation/goodpractices.html
- Changing standard (Python) test discovery — pytest documentation - https://docs.pytest.org/en/stable/example/pythoncollection.html

---

## References

- Get Started — pytest documentation - https://docs.pytest.org/en/stable/getting-started.html
- About fixtures — pytest documentation - https://docs.pytest.org/en/stable/explanation/fixtures.html
- How to use fixtures — pytest documentation - https://docs.pytest.org/en/stable/how-to/fixtures.html
- How to parametrize fixtures and test functions — pytest documentation - https://docs.pytest.org/en/stable/how-to/parametrize.html
- How to mark test functions with attributes — pytest documentation - https://docs.pytest.org/en/stable/how-to/mark.html
- Skip and xfail: dealing with tests that cannot succeed — pytest documentation - https://docs.pytest.org/en/stable/how-to/skipping.html
- How to write and report assertions in tests — pytest documentation - https://docs.pytest.org/en/stable/how-to/assert.html
- How to install and use plugins — pytest documentation - https://docs.pytest.org/en/stable/how-to/plugins.html
- Good Integration Practices — pytest documentation - https://docs.pytest.org/en/stable/explanation/goodpractices.html
- Pytest fixtures — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/python-advanced-pytest/4-fixtures
- pytest-cov Documentation - https://pytest-cov.readthedocs.io/
- pytest-xdist Documentation - https://pytest-xdist.readthedocs.io/
- pytest-asyncio Documentation - https://pytest-asyncio.readthedocs.io/