# Python Mocking: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Mocking is the practice of replacing parts of a system under test with mock objects — simulated objects that record how they are used and return pre-configured values — to isolate the code under test from external dependencies and make assertions about interactions.

### Technical Definition

`unittest.mock` is a library for testing in Python. It allows you to replace parts of your system under test with mock objects and make assertions about how they have been used. The library provides a core `Mock` class, removing the need to create a host of stubs throughout your test suite. After performing an action, you can make assertions about which methods and attributes were used and the arguments they were called with. Mock provides a `patch()` decorator that handles patching module and class-level attributes within the scope of a test, along with `sentinel` for creating unique objects. Mock is designed for use with `unittest` and is based on the **action → assertion** pattern instead of the **record → replay** pattern used by many mocking frameworks [6†L4-L21].

### Beginner-Friendly Explanation

When you write tests for your code, sometimes your code depends on things that are slow, unreliable, or hard to control — like a database, a web API, or the current time. Mocking lets you replace those real dependencies with fake stand-ins called "mocks." You tell the mock what to return, and it records how it was used. Then you can check that your code called the right methods with the right arguments — all without touching the real database or API.

### Key Characteristics

- **Standard library**: Ships with Python 3.3+ as `unittest.mock`; a backport (`mock`) is available on PyPI for earlier versions [6†L21-L23].
- **Action → assertion pattern**: Configure a mock, perform an action, then assert how the mock was used.
- **Auto-creating attributes**: Mock and MagicMock create all attributes and methods as you access them [6†L25-L27].
- **Patching**: `patch()`, `patch.object()`, and `patch.dict()` temporarily replace objects during tests.
- **Side effects**: `.side_effect` allows raising exceptions, cycling through iterables, or calling functions.
- **Async support**: `AsyncMock` handles asynchronous functions and coroutines.

### Prerequisites

- Python 3.3+ installed (for `AsyncMock`, Python 3.8+).
- Basic understanding of Python functions, classes, and modules.
- A testing framework (pytest or unittest).
- `pip install pytest` (recommended) or use built-in `unittest`.

### Related Programming Areas

- **Unit testing**: Mocking is primarily used to isolate units of code.
- **Integration testing**: Mock external boundaries (APIs, databases) while testing real internal components.
- **Test-Driven Development (TDD)**: Mocks are used to test code before dependencies are available.
- **Dependency injection**: Mocking complements DI by providing test doubles.

### Core Concepts / Features

---

## 1. unittest.mock — Core Mocking Infrastructure

### Definitions

**Core Definition**: `unittest.mock` is Python's standard library module for creating mock objects and patching system components during tests.

**Technical Definition**: `unittest.mock` provides `Mock`, `MagicMock`, `AsyncMock`, `patch`, `sentinel`, `DEFAULT`, and `call`. It allows replacing parts of the system under test with mock objects and making assertions about how they were used. The module is designed around the "action → assertion" pattern: you set up a mock, execute the code under test, then assert that the mock was used correctly [6†L4-L6].

**Beginner-Friendly Explanation**: `unittest.mock` is the built-in Python library that gives you mock objects and the tools to put them in place of real objects during tests.

### Purposes

- To replace parts of the system under test with mock objects that simulate real behaviour.
- To make assertions about how methods and attributes were used.
- To avoid creating a host of stubs manually throughout the test suite.
- To provide a standard, consistent mocking interface across all Python projects.
- To integrate seamlessly with `unittest` and `pytest`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from unittest.mock import Mock, MagicMock, AsyncMock, patch, sentinel, call, DEFAULT

# Create a mock
mock = Mock()
mock.method()
mock.method.assert_called_once_with()

# Patch an object
with patch("module.ClassName") as MockClass:
    ...
```

#### Key Imports

| Import | Purpose |
|--------|---------|
| `Mock` | Basic mock object |
| `MagicMock` | Mock with dunder method support |
| `AsyncMock` | Mock for async functions |
| `patch` | Temporarily replace objects |
| `sentinel` | Create unique sentinel objects |
| `call` | Assert multiple calls |
| `DEFAULT` | Fallback for side_effect |

#### Syntax Rules

1. **Import from `unittest.mock`**: All mock utilities are in this module [6†L4-L5].
2. **Action → assertion**: Configure the mock, perform the action, then assert.
3. **Mock auto-creates attributes**: Accessing any attribute creates it [6†L25-L27].
4. **Patching scope**: `patch()` restores the original object when the test ends [6†L44-L46].
5. **Compatible with unittest and pytest**: Works with both testing frameworks.

#### Constraints and Limitations

- Mocking is not a substitute for good design; over-mocking indicates tight coupling.
- Patching the wrong namespace is the most common mistake.
- `Mock` does not support dunder methods; use `MagicMock` for those.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Mock Usage

```python
from unittest.mock import Mock

# step1: Create a mock
mock = Mock()

# step2: Configure return value
mock.get_data.return_value = {"status": "ok"}

# step3: Use the mock
result = mock.get_data("key")
print(result)  # {'status': 'ok'}

# step4: Assert how it was used
mock.get_data.assert_called_once_with("key")
print("Assertion passed")
```

**Expected Output**:
```
{'status': 'ok'}
Assertion passed
```

**Why**: `Mock()` creates a mock object; `return_value` configures what it returns; `assert_called_once_with` verifies the call arguments [6†L29-L31].

#### Example 2: Sentinel Objects

```python
from unittest.mock import sentinel

# step1: Use sentinel for unique values
def process(data=sentinel.DEFAULT):
    if data is sentinel.DEFAULT:
        return "no data"
    return data

print(process())              # no data
print(process("hello"))       # hello
print(sentinel.DEFAULT)       # sentinel.DEFAULT
```

**Expected Output**:
```
no data
hello
sentinel.DEFAULT
```

**Why**: `sentinel` creates unique objects for default values; no two sentinel attributes are equal [6†L14-L15].

#### Example 3: Multiple Call Assertions

```python
from unittest.mock import Mock, call

mock = Mock()
mock("foo")
mock("bar")

# step1: Assert multiple calls
mock.assert_has_calls([call("foo"), call("bar")])
print("All calls asserted")

# step2: Check call count
print(mock.call_count)  # 2

# step3: Access call arguments
print(mock.call_args_list)  # [call('foo'), call('bar')]
```

**Expected Output**:
```
All calls asserted
2
[call('foo'), call('bar')]
```

**Why**: `call` constructs expected calls; `assert_has_calls` verifies multiple calls in order.

### Real-World Cases

- **API clients**: Mocking HTTP calls to avoid network dependency.
- **Database access**: Mocking database queries to test logic in isolation.
- **File I/O**: Mocking file reads/writes to avoid filesystem dependency.
- **Time-dependent code**: Mocking `datetime.now()` for deterministic tests.

### References

- unittest.mock — mock object library - https://docs.python.org/3/library/unittest.mock.html
- unittest.mock — getting started - https://docs.python.org/3/library/unittest.mock-examples.html

---

## 2. Mock Objects

### Definitions

**Core Definition**: A mock object is a flexible, auto-creating stand-in object that records its usage and can be configured to return specific values or perform side effects.

**Technical Definition**: Mock objects create all attributes and methods as you access them and store details of how they have been used. You can configure them to specify return values or limit what attributes are available. The `Mock` class is the base; `MagicMock` adds support for Python's magic (dunder) methods; `AsyncMock` supports asynchronous functions [6†L25-L28]. The `spec` argument configures the mock to take its specification from another object; accessing attributes that don't exist on the spec raises `AttributeError` [6†L39-L42].

**Beginner-Friendly Explanation**: A mock object is a fake object that pretends to be the real thing. It automatically creates methods and attributes as you use them, and it remembers how it was called so you can check later.

### Purposes

- To replace real objects with controllable stand-ins during tests.
- To record method calls and arguments for later assertions.
- To configure return values and side effects for simulating different scenarios.
- To enforce interface contracts via `spec` and `autospec`.
- To support both synchronous and asynchronous code.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from unittest.mock import Mock, MagicMock, AsyncMock

# Basic Mock
mock = Mock()
mock.method.return_value = 42

# MagicMock (supports dunder methods)
mm = MagicMock()
len(mm)  # 0
mm.__getitem__.return_value = "item"

# AsyncMock
async_mock = AsyncMock()
await async_mock()  # Works with await

# With spec
spec_mock = Mock(spec=SomeClass)
```

#### Mock vs MagicMock vs AsyncMock

| Class | Dunder Support | Async Support | Use When |
|-------|---------------|---------------|----------|
| `Mock` | No | No | You want errors for unexpected dunder usage |
| `MagicMock` | Yes | No | The mock is used as a context manager, iterable, etc. |
| `AsyncMock` | Yes | Yes | Testing `async def` functions |

#### Syntax Rules

1. **Auto-creation**: Accessing any attribute on a mock creates it [6†L25-L27].
2. **`return_value`**: Sets what the mock returns when called.
3. **`side_effect`**: Overrides `return_value`; can be a callable, iterable, or exception [6†L31-L38].
4. **`spec`**: Restricts the mock to the interface of the specified object [6†L39-L42].
5. **`assert_called_with`**: Verifies the most recent call's arguments [7†L20-L24].
6. **`call_count`**: Tracks how many times the mock was called.
7. **`called`**: `True` if the mock has been called.

#### Constraints and Limitations

- `Mock` does not support dunder methods; use `MagicMock` for those [9†L14-L21].
- `spec` only restricts attributes; it does not enforce method signatures (use `autospec` for that).
- Mocks do not enforce type checking.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Mock vs MagicMock

```python
from unittest.mock import Mock, MagicMock

# step1: Mock does not support dunder methods
m = Mock()
try:
    len(m)
except TypeError as e:
    print(f"Mock: {e}")  # object of type 'Mock' has no len()

# step2: MagicMock supports dunder methods
mm = MagicMock()
print(len(mm))              # 0
mm.__getitem__.return_value = "item"
print(mm["key"])            # item
```

**Expected Output**:
```
Mock: object of type 'Mock' has no len()
0
item
```

**Why**: `MagicMock` implements magic methods like `__len__`, `__getitem__`, and `__enter__`, making it suitable for mocks used as containers or context managers [9†L14-L21].

#### Example 2: Configuring Return Values and Attributes

```python
from unittest.mock import Mock

mock = Mock()

# step1: Set return value
mock.fetch.return_value = {"status": "ok", "count": 3}
print(mock.fetch("/health"))  # {'status': 'ok', 'count': 3}

# step2: Set attributes
mock.config.timeout = 30
print(mock.config.timeout)    # 30

# step3: Different values per call with side_effect
mock.fetch.side_effect = [{"count": 1}, {"count": 2}]
print(mock.fetch())           # {'count': 1}
print(mock.fetch())           # {'count': 2}
```

**Expected Output**:
```
{'status': 'ok', 'count': 3}
30
{'count': 1}
{'count': 2}
```

**Why**: `return_value` sets a fixed return; `side_effect` with an iterable returns a different value per call [9†L30-L39].

#### Example 3: Using spec to Enforce Interface

```python
from unittest.mock import Mock

class UserService:
    def get_user(self, user_id):
        pass

    def save_user(self, user):
        pass

# step1: Create a mock with spec
mock = Mock(spec=UserService)

# step2: Valid attribute access works
mock.get_user(1)
print("get_user called")

# step3: Invalid attribute raises AttributeError
try:
    mock.delete_user(1)
except AttributeError as e:
    print(f"AttributeError: {e}")
```

**Expected Output**:
```
get_user called
AttributeError: Mock object has no attribute 'delete_user'
```

**Why**: `spec=UserService` restricts the mock to only the attributes and methods of `UserService` [6†L39-L42].

### Real-World Cases

- **API clients**: Mocking `requests.get` or `httpx.get` calls.
- **Database sessions**: Mocking SQLAlchemy sessions.
- **File handles**: Mocking `open()` for file I/O tests.
- **Async functions**: Mocking `async` functions with `AsyncMock`.

### References

- Mock objects - https://docs.python.org/3/library/unittest.mock.html#the-mock-class
- MagicMock - https://docs.python.org/3/library/unittest.mock.html#magicmock-and-magic-method-support
- AsyncMock - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.AsyncMock

---

## 3. Patching

### Definitions

**Core Definition**: Patching is the technique of temporarily replacing an object in a module or class with a mock during a test, using `patch()`, `patch.object()`, or `patch.dict()`.

**Technical Definition**: `patch()` acts as a function decorator, class decorator, or context manager. Inside the body of the function or `with` statement, the target (specified as `'package.module.ClassName'`) is patched with a new object. When the function or `with` statement exits, the patch is undone. The target is imported and the specified object replaced, so it must be importable from the environment you are calling `patch()` from. The target is imported when the decorated function is executed, not at decoration time. The key rule: **patch where the name is looked up, not where it is defined** [2†L7-L8][9†L41-L42].

**Beginner-Friendly Explanation**: Patching means temporarily swapping out a real object with a mock during a test. The tricky part is that you need to patch it where it's used, not where it was originally defined.

### Purposes

- To replace module-level and class-level attributes for the duration of a test.
- To avoid modifying production code for testability.
- To isolate the unit under test from external dependencies.
- To verify that the code under test calls the patched object correctly.
- To mock dictionaries via `patch.dict()`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from unittest.mock import patch

# As a decorator
@patch("module.ClassName")
def test_function(MockClass):
    ...

# As a context manager
with patch("module.ClassName") as MockClass:
    ...

# patch.object — patch a specific attribute
@patch.object(SomeClass, "method")
def test_method(MockMethod):
    ...

# patch.dict — patch a dictionary
with patch.dict("module.DICT", {"key": "value"}):
    ...
```

#### The Where-to-Patch Rule

```python
# app/service.py
from app.clients import PaymentClient

def charge(amount):
    return PaymentClient().submit(amount)

# app/tests/test_service.py
from unittest.mock import patch

@patch("app.service.PaymentClient")  # CORRECT — where it is used
def test_charge(MockClient):
    MockClient.return_value.submit.return_value = "txn_123"
    assert charge(100) == "txn_123"
```

**Why**: The name `PaymentClient` is imported into `app.service`, so patching `app.clients.PaymentClient` would not affect `app.service` [9†L41-L48].

#### Syntax Rules

1. **Target format**: A string in the form `'package.module.ClassName'` [2†L21-L24].
2. **Patch where looked up**: Patch the name in the namespace where it is used, not where it is defined [9†L41-L42].
3. **As decorator**: The mock is passed as an argument to the decorated function.
4. **As context manager**: The mock is returned by the `with` statement.
5. **`patch.object`**: Patches an attribute of a specific object.
6. **`patch.dict`**: Patches a dictionary with new values.
7. **`autospec=True`**: Creates a mock with the same spec as the original object; the highest-value habit in mocking Python code [5†L16].

#### Constraints and Limitations

- Patching the wrong namespace is the most common error.
- `patch()` imports the target when the decorated function runs, not at decoration time.
- Patching built-in functions can affect other tests if not properly scoped.
- `autospec` is slower but more accurate.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: patch() as Decorator

```python
# step1: Source code (weather.py)
import requests

def get_temperature(city):
    response = requests.get(f"https://api.weather.com/v1/{city}")
    return response.json()["temperature"]

# step2: Test with patch (test_weather.py)
from unittest.mock import patch
from weather import get_temperature

@patch("weather.requests.get")
def test_get_temperature(mock_get):
    mock_get.return_value.json.return_value = {"temperature": 22}
    result = get_temperature("London")
    assert result == 22
    mock_get.assert_called_once_with("https://api.weather.com/v1/London")
    print("Test passed")

test_get_temperature()
```

**Expected Output**:
```
Test passed
```

**Why**: `patch("weather.requests.get")` replaces the `requests.get` call in the `weather` module; the mock is passed as an argument to the test function [9†L41-L48].

#### Example 2: patch() as Context Manager

```python
from unittest.mock import patch
import os

# step1: Patch os.getcwd in a context manager
with patch("os.getcwd") as mock_getcwd:
    mock_getcwd.return_value = "/mocked/path"
    print(os.getcwd())  # /mocked/path

# step2: After the with block, the original is restored
print(os.getcwd())  # /actual/path (real value)
```

**Expected Output**:
```
/mocked/path
/actual/path
```

**Why**: The patch applies only inside the `with` block; the original `os.getcwd` is restored after the block exits [2†L9-L10].

#### Example 3: patch.object and patch.dict

```python
from unittest.mock import patch

class Calculator:
    def add(self, a, b):
        return a + b

# step1: patch.object — patch a method
with patch.object(Calculator, "add", return_value=99) as mock_add:
    calc = Calculator()
    print(calc.add(1, 2))  # 99
    mock_add.assert_called_once_with(1, 2)

# step2: patch.dict — patch a dictionary
config = {"debug": False, "version": "1.0"}
with patch.dict(config, {"debug": True}):
    print(config)  # {'debug': True, 'version': '1.0'}

# step3: Original restored
print(config)  # {'debug': False, 'version': '1.0'}
```

**Expected Output**:
```
99
{'debug': True, 'version': '1.0'}
{'debug': False, 'version': '1.0'}
```

**Why**: `patch.object` patches a specific method; `patch.dict` temporarily updates a dictionary and restores it afterward.

### Real-World Cases

- **HTTP clients**: Patching `requests.get` or `httpx.get`.
- **Database connections**: Patching `psycopg2.connect` or SQLAlchemy sessions.
- **Time-dependent code**: Patching `datetime.now` or `time.time`.
- **Environment variables**: Patching `os.environ` via `patch.dict`.

### References

- patch() - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch
- Where to patch - https://docs.python.org/3/library/unittest.mock.html#where-to-patch
- patch.object - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch.object
- patch.dict - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch.dict
- autospec - https://docs.python.org/3/library/unittest.mock.html#autospeccing

---

## 4. Mock Side Effects

### Definitions

**Core Definition**: `side_effect` is a mock attribute that allows you to perform dynamic behaviours when the mock is called, including raising exceptions, returning values from an iterable, or calling a function.

**Technical Definition**: `side_effect` can be a function to be called when the mock is called, an iterable, or an exception (class or instance) to be raised. If you pass in an iterable, it is used to retrieve an iterator which must yield a value on every call — that value can either be an exception instance or a value to be returned [3†L4-L10]. If `side_effect` is a function, whatever that function returns is used as the mock's return value. If it returns `DEFAULT`, the mock falls back to `return_value` [9†L36-L39].

**Beginner-Friendly Explanation**: `side_effect` lets you make a mock do different things each time it's called — like raising an error, returning different values, or running a custom function.

### Purposes

- To simulate exceptions that the real dependency might raise.
- To return different values on successive calls (e.g., modelling retries).
- To compute return values dynamically based on call arguments.
- To trigger additional side effects when the mock is called.
- To test error-handling and retry logic.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from unittest.mock import Mock

mock = Mock()

# Raise an exception
mock.side_effect = ValueError("Invalid input")

# Iterable of return values
mock.side_effect = [1, 2, 3]

# Callable for dynamic behaviour
def dynamic_side_effect(arg):
    return arg * 2
mock.side_effect = dynamic_side_effect

# Mixed: values and exceptions
mock.side_effect = [1, TimeoutError("Timed out"), 3]
```

#### Side Effect Types

| Type | Behaviour |
|------|-----------|
| Exception class/instance | Raised when the mock is called |
| Iterable | Yields next value on each call |
| Callable | Called with the mock's arguments; return value is used |
| `DEFAULT` | Falls back to `return_value` |

#### Syntax Rules

1. **Exception**: Setting `side_effect` to an exception class or instance makes the mock raise it [6†L33-L35].
2. **Iterable**: Each call returns the next item; when exhausted, `StopIteration` is raised [9†L31-L33].
3. **Callable**: The callable receives the same arguments as the mock; its return value becomes the mock's return value [6†L35-L37].
4. **DEFAULT**: If a callable returns `DEFAULT`, the mock falls back to `return_value` [9†L36-L38].
5. **Precedence**: `side_effect` takes precedence over `return_value`.
6. **Reset**: Setting `side_effect = None` restores normal `return_value` behaviour.

#### Constraints and Limitations

- When `side_effect` is an iterable, after exhaustion the mock raises `StopIteration`.
- `side_effect` and `return_value` interact: `side_effect` wins unless it returns `DEFAULT`.
- Dynamic callables must accept the same arguments as the mock.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Raising Exceptions

```python
from unittest.mock import Mock

# step1: Configure mock to raise an exception
mock = Mock(side_effect=ConnectionError("Connection refused"))

# step2: Call the mock
try:
    mock()
except ConnectionError as e:
    print(f"Caught: {e}")  # Caught: Connection refused
```

**Expected Output**:
```
Caught: Connection refused
```

**Why**: `side_effect` set to an exception instance causes the mock to raise that exception when called [6†L33-L35].

#### Example 2: Cycling Through Values

```python
from unittest.mock import Mock

# step1: Configure side_effect with a list
mock = Mock(side_effect=[5, 4, 3, 2, 1])

# step2: Each call returns the next value
print(mock())  # 5
print(mock())  # 4
print(mock())  # 3

# step3: Exhausted list raises StopIteration
try:
    mock()
    mock()
    mock()
except StopIteration:
    print("Iterable exhausted")
```

**Expected Output**:
```
5
4
3
Iterable exhausted
```

**Why**: When `side_effect` is an iterable, each call returns the next value; after exhaustion, `StopIteration` is raised [9†L31-L33].

#### Example 3: Dynamic Callable Side Effect

```python
from unittest.mock import Mock, DEFAULT

# step1: Define a dynamic side effect
def fake_lookup(key):
    values = {"a": 1, "b": 2, "c": 3}
    return values.get(key, DEFAULT)

mock = Mock(side_effect=fake_lookup)
mock.return_value = -1  # Fallback for unknown keys

# step2: Known keys return computed values
print(mock("a"))  # 1
print(mock("b"))  # 2

# step3: Unknown key falls back to return_value
print(mock("z"))  # -1
```

**Expected Output**:
```
1
2
-1
```

**Why**: The callable computes the return value dynamically; returning `DEFAULT` falls back to `return_value` [9†L36-L38].

### Real-World Cases

- **Retry logic**: Modelling a function that fails twice then succeeds.
- **API rate limiting**: Simulating a rate-limit exception after N calls.
- **Error handling**: Testing that code correctly handles exceptions from dependencies.
- **Dynamic responses**: Computing return values based on input arguments.

### References

- side_effect - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect
- Configuring Mock return values and side effects - https://docs.python.org/3/library/unittest.mock.html#configuring-mock-return-values-and-side-effects

---

## 5. Dependency Isolation

### Definitions

**Core Definition**: Dependency isolation is the practice of substituting external constraints — slow network calls, live databases, third-party APIs — with mocks to ensure fast, predictable, local test runs.

**Technical Definition**: Dependency isolation involves identifying the boundaries of the system under test (network, filesystem, database, time) and replacing them with mock objects. The goal is to make unit tests fast, deterministic, and independent of external systems. A mock is a fake object that mimics the behaviour of a real dependency, allowing you to isolate the specific unit of code you want to test [10†L14-L16]. The best approach is to refactor code to separate I/O and side effects from pure calculation logic, then write clean, mock-free unit tests for the calculation logic [10†L41-L47].

**Beginner-Friendly Explanation**: Dependency isolation means replacing real external systems with fakes so your tests run quickly and don't fail because of network issues or database downtime.

### Purposes

- To make tests fast by avoiding network calls and database queries.
- To make tests deterministic by removing external variability.
- To test code in isolation without relying on external systems.
- To simulate edge cases that are hard to trigger with real dependencies.
- To decouple tests from external service availability.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from unittest.mock import patch, Mock

# Isolate a network dependency
@patch("module.requests.get")
def test_api_call(mock_get):
    mock_get.return_value.json.return_value = {"data": "mocked"}
    ...

# Isolate a database dependency
@patch("module.database.connect")
def test_db_query(mock_connect):
    mock_conn = Mock()
    mock_connect.return_value = mock_conn
    mock_conn.execute.return_value = [{"id": 1}]
    ...

# Isolate time
@patch("module.datetime")
def test_timestamp(mock_datetime):
    mock_datetime.now.return_value = datetime(2026, 9, 28)
    ...
```

#### When to Mock (Good Targets)

| Boundary | Example |
|----------|---------|
| Network | API requests, database queries, web sockets |
| Filesystem | File creation, read/write loops, system status |
| Non-deterministic inputs | `datetime.now()`, `random.random()` |
| External services | Payment gateways, email senders |

#### When NOT to Mock

| Target | Why |
|--------|-----|
| Your own helper functions | Indicates tight coupling |
| Mathematical subroutines | Pure logic should be tested directly |
| Internal class structures | Mocking internal implementation is brittle |

#### Syntax Rules

1. **Mock at boundaries**: Mock the network, filesystem, and external services [10†L27-L31].
2. **Don't mock your own code**: If you need to mock internal functions, refactor [10†L32-L37].
3. **Separate I/O from logic**: Write pure functions for calculation and thin I/O wrappers [10†L41-L47].
4. **Use `autospec=True`**: Ensures the mock has the same interface as the real object [5†L16].
5. **Reset mocks between tests**: Use `mock.reset_mock()` or fixture teardown.

#### Constraints and Limitations

- Over-mocking leads to brittle tests tied to implementation details [10†L35-L40].
- Mocking too many layers indicates poor design.
- Mocks do not catch integration issues between real components.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Isolating a Web API

```python
# step1: Source code (weather.py)
import requests

def get_weather(city):
    response = requests.get(f"https://api.weather.com/v1/{city}")
    return response.json()

# step2: Test with mocked API (test_weather.py)
from unittest.mock import patch
from weather import get_weather

@patch("weather.requests.get")
def test_get_weather(mock_get):
    mock_get.return_value.json.return_value = {"temperature": 22, "condition": "sunny"}
    result = get_weather("London")
    assert result["temperature"] == 22
    assert result["condition"] == "sunny"
    print("Test passed in < 1ms")
```

**Expected Output**:
```
Test passed in < 1ms
```

**Why**: The real API call is replaced with a mock, making the test fast and independent of network availability [10†L17-L23].

#### Example 2: Isolating a Database

```python
# step1: Source code (user_repo.py)
import sqlite3

class UserRepository:
    def __init__(self, db_path):
        self.conn = sqlite3.connect(db_path)

    def get_user(self, user_id):
        cursor = self.conn.execute("SELECT * FROM users WHERE id = ?", (user_id,))
        return cursor.fetchone()

# step2: Test with mocked database
from unittest.mock import patch, MagicMock
from user_repo import UserRepository

@patch("user_repo.sqlite3.connect")
def test_get_user(mock_connect):
    mock_conn = MagicMock()
    mock_conn.execute.return_value.fetchone.return_value = (1, "Alice", "alice@test.com")
    mock_connect.return_value = mock_conn

    repo = UserRepository(":memory:")
    result = repo.get_user(1)
    assert result == (1, "Alice", "alice@test.com")
    print("Database test passed")
```

**Expected Output**:
```
Database test passed
```

**Why**: The database connection and query are fully mocked; the test verifies the repository logic without a real database.

#### Example 3: Separating I/O from Logic

```python
# step1: Refactored code — pure function + I/O wrapper
def parse_temperature(raw_data):
    """Pure function: no I/O, easy to test."""
    return raw_data["temperature"]

def fetch_and_parse(city):
    """I/O wrapper: mock this in tests."""
    import requests
    response = requests.get(f"https://api.weather.com/v1/{city}")
    return parse_temperature(response.json())

# step2: Test pure function — no mocks needed
def test_parse_temperature():
    assert parse_temperature({"temperature": 22}) == 22
    print("Pure function test passed")

# step3: Test I/O wrapper with mock
from unittest.mock import patch

@patch("__main__.requests.get")
def test_fetch_and_parse(mock_get):
    mock_get.return_value.json.return_value = {"temperature": 22}
    assert fetch_and_parse("London") == 22
    print("I/O wrapper test passed")

test_parse_temperature()
test_fetch_and_parse()
```

**Expected Output**:
```
Pure function test passed
I/O wrapper test passed
```

**Why**: Separating pure logic from I/O allows testing the calculation without mocks and mocking only the boundary [10†L41-L47].

### Real-World Cases

- **Microservices**: Mocking downstream services while testing one service.
- **Data pipelines**: Mocking external data sources while testing transformations.
- **Web applications**: Mocking database and API calls in unit tests.
- **CLI tools**: Mocking file system access for portable tests.

### References

- Testing: Mocking and Isolation - https://intersect-training.org/testing/instructor/05-mocking-and-isolation.html
- Mocking Anti-Patterns - https://github.com/tachyon-beep/skillpacks/blob/main/plugins/axiom-python-engineering/skills/using-python-engineering/testing-and-quality.md
- Pytest Monkeypatch - https://docs.pytest.org/en/stable/how-to/monkeypatch.html

---

## References

- unittest.mock — mock object library - https://docs.python.org/3/library/unittest.mock.html
- unittest.mock — getting started - https://docs.python.org/3/library/unittest.mock-examples.html
- Mock objects - https://docs.python.org/3/library/unittest.mock.html#the-mock-class
- MagicMock - https://docs.python.org/3/library/unittest.mock.html#magicmock-and-magic-method-support
- AsyncMock - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.AsyncMock
- patch() - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch
- Where to patch - https://docs.python.org/3/library/unittest.mock.html#where-to-patch
- patch.object - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch.object
- patch.dict - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch.dict
- autospec - https://docs.python.org/3/library/unittest.mock.html#autospeccing
- side_effect - https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect
- Configuring Mock return values and side effects - https://docs.python.org/3/library/unittest.mock.html#configuring-mock-return-values-and-side-effects
- Testing: Mocking and Isolation - https://intersect-training.org/testing/instructor/05-mocking-and-isolation.html
- Python Unit Test Mock: unittest.mock and MagicMock Guide - https://safeguard.sh/resources/blog/python-mocking-complete-guide
- Pytest Monkeypatch - https://docs.pytest.org/en/stable/how-to/monkeypatch.html