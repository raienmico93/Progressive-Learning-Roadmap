# Raising Exceptions: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Raising an exception is the act of explicitly triggering an error condition in Python using the `raise` statement, which interrupts normal program flow and signals that something has gone wrong.

**Technical Definition**
The `raise` statement instantiates an exception object (or uses an existing one) and transfers control to the nearest enclosing `try`/`except` handler. Python 3 consolidates the `raise` statement into two forms: `raise` (bare, re-raising the active exception) and `raise EXCEPTION` (raising a new exception, where `EXCEPTION` may be a class or an instance). When raising a new exception during handling of another, Python automatically sets `__context__` to the active exception; the `from` clause explicitly sets `__cause__` to establish a deliberate chain.

**Beginner-Friendly Explanation**
Raising an exception is like pulling a fire alarm. You've detected a problem, and instead of trying to fix it yourself, you alert the rest of the program so that whoever is responsible for handling that kind of problem can deal with it. The `raise` statement is how you pull the alarm.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Explicit Error Signaling** | `raise` makes error conditions visible and intentional |
| **Two Forms** | Bare `raise` re-raises; `raise ExceptionType(...)` raises new |
| **Class or Instance** | You can raise either an exception class or an instance |
| **Automatic Context** | Raising inside an `except` block sets `__context__` implicitly |
| **Explicit Chaining** | `raise ... from original` sets `__cause__` for deliberate chains |
| **Custom Messages** | Exception constructors accept arguments that become the error message |

### Prerequisites

- **Basic exception handling**: `try`/`except`/`finally` semantics
- **Exception hierarchy**: Understanding `Exception` and its subclasses
- **Function definitions**: Where and why to raise exceptions

### Related Programming Areas

- **Exception Handling**: Catching and recovering from raised exceptions
- **Custom Exceptions**: Defining application-specific error types
- **Debugging**: Reading tracebacks to locate where exceptions were raised
- **API Design**: Raising meaningful exceptions as part of a function's contract

### Core Concepts / Features

1. **`raise`** (the statement)
2. **Re-raising** (bare `raise` and `raise e`)
3. **Exception Chaining** (`from` clause, `__cause__`, `__context__`)
4. **Custom Error Messages** (exception arguments and messages)

---

## Core Concept 1: `raise`

### Definitions

**Core Definition**
The `raise` statement explicitly triggers an exception, interrupting normal execution and transferring control to an exception handler.

**Technical Definition**
In Python 3, the `raise` statement has two forms: `raise` (bare) and `raise EXCEPTION`. When `EXCEPTION` is a class, Python instantiates it with no arguments; when it is an instance, that instance is raised directly. The raised exception propagates up the call stack until caught by an `except` clause or reaching the top level, where the interpreter prints a traceback and exits.

**Beginner-Friendly Explanation**
`raise` is how you say "this is a problem, and I'm not going to handle it here." You create an exception object (or use an existing one) and throw it, and Python starts looking for someone who can deal with it.

### Purposes

- **To signal that an error condition has occurred** that the current code cannot handle
- **To enforce preconditions** by raising `ValueError` or `TypeError` when inputs are invalid
- **To implement custom error handling** by raising application-specific exceptions
- **To abort an operation** when continuing would produce incorrect results
- **To communicate failure** to callers through a well-defined interface

### Syntax Rules and Structure

**Complete General Syntaxes**

```python
# Raise a new exception instance
raise ExceptionType("message")

# Raise an exception class (instantiated automatically)
raise ExceptionType

# Raise with no arguments (re-raise active exception)
raise
```

**Breakdown:**
- `ExceptionType`: A class that is a subclass of `BaseException` (typically `Exception`).
- `"message"`: Optional arguments passed to the exception constructor; become the exception's message.
- Bare `raise`: Re-raises the currently active exception; only valid when an exception is being handled.

**Syntax Rules**

| Rule | Description |
|---|---|
| Exception must be class or instance | `raise` requires a `BaseException` subclass or instance |
| Class is auto-instantiated | `raise ValueError` calls `ValueError()` with no arguments |
| Instance is raised directly | `raise ValueError("msg")` raises the created instance |
| Bare `raise` requires active exception | Raises `RuntimeError` if no exception is being handled |
| Raising non-exception | Raises `TypeError` |

**Constraints and Limitations**

- **Raising a non-exception**: `raise 42` raises `TypeError: exceptions must derive from BaseException`.
- **Bare `raise` outside `except`**: In Python 3, bare `raise` outside an `except` block raises `RuntimeError: No active exception to reraise`.
- **Class vs. instance**: Raising a class creates a new instance each time; raising an instance reuses the same object.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Raising an Exception Instance**

```python
def set_age(age):
    """Set an age, raising ValueError if invalid."""
    if age < 0:
        raise ValueError(f"Age cannot be negative: {age}")
    return age

try:
    set_age(-5)
except ValueError as e:
    print(f"Caught: {e}")
```

**Expected Output:**
```
Caught: Age cannot be negative: -5
```

**Breakdown:**
1. `set_age(-5)` detects the invalid value and raises `ValueError` with a descriptive message.
2. The exception propagates out of `set_age` and is caught by the `except` clause.
3. The message is accessible via the exception object `e`.

**Example 2: Raising an Exception Class**

```python
def divide(a, b):
    """Divide a by b, raising ZeroDivisionError if b is zero."""
    if b == 0:
        raise ZeroDivisionError  # Class is auto-instantiated
    return a / b

try:
    divide(10, 0)
except ZeroDivisionError as e:
    print(f"Caught: {e}")
```

**Expected Output:**
```
Caught: 
```

**Breakdown:**
1. `raise ZeroDivisionError` instantiates `ZeroDivisionError()` with no arguments.
2. The exception is caught, but its message is empty (no arguments were passed).
3. To provide a message, use `raise ZeroDivisionError("division by zero")`.

### Real-World Cases with Explanation

**Case 1: Input Validation**

```python
def create_user(username, email):
    if not username:
        raise ValueError("Username cannot be empty")
    if "@" not in email:
        raise ValueError(f"Invalid email: {email}")
    return {"username": username, "email": email}

try:
    create_user("", "invalid")
except ValueError as e:
    print(f"Validation failed: {e}")
# Output: Validation failed: Username cannot be empty
```

**Why it matters:** Raising exceptions with clear messages enforces preconditions and makes debugging easier for callers.

**Case 2: Abstract Methods**

```python
class Shape:
    def area(self):
        raise NotImplementedError("Subclasses must implement area()")

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

shape = Shape()
try:
    shape.area()
except NotImplementedError as e:
    print(f"Error: {e}")
```

**Why it matters:** `NotImplementedError` signals that a method is abstract and must be overridden, enforcing interface contracts.

---

## Core Concept 2: Re-raising

### Definitions

**Core Definition**
Re-raising is the act of raising the currently active exception again, typically after some handling or logging, to allow it to propagate to an outer handler.

**Technical Definition**
Bare `raise` (with no arguments) re-raises the active exception, preserving its original traceback. `raise e` (where `e` is the caught exception) also re-raises, but may reset the traceback depending on context. Bare `raise` is preferred because it preserves the full traceback and does not create a new exception instance.

**Beginner-Friendly Explanation**
Sometimes you catch an exception to do something (like log it) but still want the program to know the error happened. Re-raising is how you say "I handled this a little, but someone else should handle it too."

### Purposes

- **To log an exception** without suppressing it
- **To perform cleanup** before letting the exception propagate
- **To add context** while preserving the original exception
- **To implement "handle and re-raise" patterns** in middleware and decorators

### Syntax Rules and Structure

```python
try:
    risky_operation()
except SomeError:
    # Do something (log, cleanup)
    raise  # Re-raise the same exception
```

**Syntax Rules**

| Form | Behavior |
|---|---|
| `raise` | Re-raises active exception; preserves traceback |
| `raise e` | Raises the exception object `e`; may lose traceback context |
| `raise` outside except | Raises `RuntimeError` |

**Constraints and Limitations**

- **Bare `raise` outside `except`**: Invalid; raises `RuntimeError`.
- **`raise e` vs. bare `raise`**: Bare `raise` is preferred; `raise e` can truncate the traceback in some contexts.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Log and Re-raise**

```python
import logging

def process_data(data):
    try:
        return int(data)
    except ValueError:
        logging.error(f"Failed to process: {data}")
        raise  # Re-raise the original exception

try:
    process_data("abc")
except ValueError as e:
    print(f"Outer handler: {e}")
```

**Expected Output:**
```
ERROR:root:Failed to process: abc
Outer handler: invalid literal for int() with base 10: 'abc'
```

**Breakdown:**
1. The inner `except` logs the error.
2. Bare `raise` re-raises the same `ValueError`.
3. The outer `try` catches it, demonstrating the exception propagated.

**Example 2: Cleanup and Re-raise**

```python
def use_resource():
    resource = acquire()
    try:
        resource.use()
    except Exception:
        resource.cleanup()
        raise  # Re-raise after cleanup
```

**Breakdown:** The `except` block performs cleanup, then bare `raise` lets the original exception propagate, preserving its traceback.

### Real-World Cases with Explanation

**Case 1: Middleware Logging**

```python
def logging_middleware(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            log.error(f"Error in {func.__name__}: {e}")
            raise  # Let the caller handle it
    return wrapper
```

**Why it matters:** Middleware should log but not suppress exceptions; re-raising preserves the original error for the caller.

---

## Core Concept 3: Exception Chaining (`from`)

### Definitions

**Core Definition**
Exception chaining is the practice of linking a new exception to an original one, preserving the causal relationship between them.

**Technical Definition**
When an exception is raised inside an `except` block, Python automatically sets the new exception's `__context__` attribute to the active exception. The `raise ... from original` syntax explicitly sets `__cause__` to the original exception, establishing a deliberate chain. The traceback distinguishes these: `__cause__` produces "The above exception was the direct cause of the following exception," while implicit `__context__` produces "During handling of the above exception, another exception occurred".

**Beginner-Friendly Explanation**
Exception chaining is like telling a story: "I got error A, and because of that, error B happened." The `from` clause says "B was directly caused by A," while the automatic context says "B happened while I was dealing with A."

### Purposes

- **To preserve the original error** when translating between exception types
- **To provide a clear causal chain** in tracebacks for debugging
- **To suppress irrelevant internal errors** with `from None`
- **To make error handling explicit** and intentional

### Syntax Rules and Structure

```python
# Explicit chain: sets __cause__
raise NewError("message") from original_exception

# Implicit chain: sets __context__ (automatic inside except)
try:
    ...
except OriginalError:
    raise NewError("message")  # __context__ is set automatically

# Suppress context: sets __cause__ to None, __suppress_context__ to True
raise NewError("message") from None
```

**Syntax Rules**

| Form | `__cause__` | `__context__` | Traceback Message |
|---|---|---|---|
| `raise New from orig` | `orig` | `orig` | "direct cause" |
| `raise New` in except | `None` | `orig` | "During handling" |
| `raise New from None` | `None` | `orig` (suppressed) | Only New shown |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Explicit Chaining with `from`**

```python
def load_config(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError as e:
        raise ConfigError(f"Config missing: {path}") from e

class ConfigError(Exception):
    pass

try:
    load_config("missing.toml")
except ConfigError as e:
    print(f"Caught: {e}")
    print(f"Cause: {e.__cause__}")
```

**Expected Output:**
```
Caught: Config missing: missing.toml
Cause: [Errno 2] No such file or directory: 'missing.toml'
```

**Breakdown:**
1. `FileNotFoundError` is caught and used as the cause for `ConfigError`.
2. The `from e` clause sets `__cause__` to the original exception.
3. The traceback will show "The above exception was the direct cause...".

**Example 2: Implicit Context (No `from`)**

```python
def parse_int(s):
    try:
        return int(s)
    except ValueError:
        raise TypeError("Invalid input")  # __context__ set automatically

try:
    parse_int("abc")
except TypeError as e:
    print(f"Caught: {e}")
    print(f"Context: {e.__context__}")
```

**Expected Output:**
```
Caught: Invalid input
Context: invalid literal for int() with base 10: 'abc'
```

**Breakdown:**
1. `ValueError` is raised by `int()`.
2. `TypeError` is raised in the `except` block; Python sets `__context__` to `ValueError`.
3. The traceback shows "During handling of the above exception...".

**Example 3: Suppressing Context with `from None`**

```python
def parse_timestamp(s):
    try:
        return datetime.fromisoformat(s)
    except ValueError:
        raise ValueError(f"Invalid timestamp: {s!r}") from None

try:
    parse_timestamp("not-a-date")
except ValueError as e:
    print(f"Caught: {e}")
```

**Expected Output:**
```
Caught: Invalid timestamp: 'not-a-date'
```

**Breakdown:**
1. `from None` suppresses the original `ValueError`'s context.
2. Only the new `ValueError` appears in the traceback.
3. Use this when the internal error is noise to the caller.

### Real-World Cases with Explanation

**Case 1: Translating Low-Level Errors**

```python
class DatabaseError(Exception):
    pass

def query(sql):
    try:
        return db.execute(sql)
    except sqlite3.OperationalError as e:
        raise DatabaseError(f"Query failed: {sql}") from e
```

**Why it matters:** `from e` preserves the original database error as the cause, making debugging easier while presenting a clean interface to callers.

**Case 2: Clean User-Facing Errors**

```python
def parse_config(data):
    try:
        return json.loads(data)
    except json.JSONDecodeError:
        raise ValueError("Configuration is not valid JSON") from None
```

**Why it matters:** `from None` hides the internal JSON parsing details, giving users a clean, actionable message.

---

## Core Concept 4: Custom Error Messages

### Definitions

**Core Definition**
Custom error messages are descriptive strings passed to exception constructors that explain what went wrong and provide context for debugging.

**Technical Definition**
Exception constructors accept arbitrary arguments, which are stored in the `args` attribute. The first argument is typically the message string, accessible via `str(exception)`. When the exception is printed, the message is displayed in the traceback. Custom exception classes can override `__init__` or `__str__` to customize message formatting.

**Beginner-Friendly Explanation**
A custom error message is the note you attach to an exception that says "here's what I was trying to do and here's what went wrong." Good messages make debugging faster because they tell you exactly what failed.

### Purposes

- **To provide context** about what operation failed and why
- **To include relevant values** (file paths, user inputs, parameters) in the error
- **To make tracebacks self-explanatory** without needing to read source code
- **To distinguish between different failure modes** of the same exception type

### Syntax Rules and Structure

```python
# Basic message
raise ValueError("Age must be positive")

# Formatted message with values
raise ValueError(f"Invalid age: {age}. Expected positive integer.")

# Multiple arguments
raise OSError(errno.ENOENT, "File not found", path)

# Custom exception with custom message
class ValidationError(Exception):
    def __init__(self, field, message):
        self.field = field
        super().__init__(f"{field}: {message}")
```

**Syntax Rules**

| Rule | Description |
|---|---|
| Arguments stored in `args` | `e.args` is a tuple of constructor arguments |
| `str(e)` returns first arg | Typically the message string |
| `repr(e)` shows class and args | Useful for debugging |
| Custom `__init__` | Can add attributes and format messages |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Descriptive Message**

```python
def withdraw(account, amount):
    if amount > account["balance"]:
        raise ValueError(
            f"Insufficient funds: balance={account['balance']}, "
            f"requested={amount}, shortfall={amount - account['balance']}"
        )
    account["balance"] -= amount
    return account["balance"]

account = {"balance": 100}
try:
    withdraw(account, 150)
except ValueError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Error: Insufficient funds: balance=100, requested=150, shortfall=50
```

**Breakdown:**
1. The message includes all relevant values: balance, requested amount, and shortfall.
2. This makes the error self-explanatory without needing to inspect variables.

**Example 2: Custom Exception Class with Structured Message**

```python
class ValidationError(Exception):
    """Raised when input validation fails."""
    def __init__(self, field, value, reason):
        self.field = field
        self.value = value
        self.reason = reason
        super().__init__(f"Invalid {field}: {reason} (got {value!r})")

try:
    raise ValidationError("email", "not-an-email", "missing @ symbol")
except ValidationError as e:
    print(f"Caught: {e}")
    print(f"Field: {e.field}")
    print(f"Value: {e.value}")
```

**Expected Output:**
```
Caught: Invalid email: missing @ symbol (got 'not-an-email')
Field: email
Value: not-an-email
```

**Breakdown:**
1. The custom class stores structured data (field, value, reason).
2. The message is formatted in `__init__` and passed to `super().__init__`.
3. Callers can access both the message and the structured attributes.

### Real-World Cases with Explanation

**Case 1: API Error Responses**

```python
class APIError(Exception):
    def __init__(self, status_code, message, endpoint):
        self.status_code = status_code
        self.endpoint = endpoint
        super().__init__(f"{status_code} at {endpoint}: {message}")

try:
    raise APIError(404, "User not found", "/api/users/42")
except APIError as e:
    print(f"Error: {e}")
    print(f"Status: {e.status_code}")
```

**Why it matters:** Structured error messages allow programmatic handling (checking `status_code`) while providing human-readable text.

---

## References

- Python Software Foundation. *Built-in Exceptions*. https://docs.python.org/3.10/library/exceptions.html 
- Python Software Foundation. *PEP 3109 – Raising Exceptions in Python 3000*. https://hugovk-peps.readthedocs.io/en/latest/pep-3109/ 
- Python.org Python-list. *Catching exceptions from an except block*. https://mail.python.org/pipermail/python-list/2007-March/445031.html 
- Python.org Python-list. *Re: The Most Diabolical Python Antipattern*. https://mail.python.org/archives/list/python-list@python.org/thread/WPZED6IUVMKKODPUC4PM5GR7FIZ6PNDF/ 
- Nathan Gage. *Use raise ... from to Preserve Exception Causality*. https://raw.githubusercontent.com/nathan-gage/python-skills/refs/heads/main/skills/python-best-practices/rules/error-raise-from-for-chains.md 
- Pluralsight. *Creating Custom Python Exception Types*. https://www.pluralsight.com/labs/aws/creating-custom-python-exception-types 
- Alibaba Cloud Developer. *Classes & OOP—Defining Your Own Exception Classes*. https://developer.aliyun.com/article/1618772 
- Python Software Foundation. *traceback — Print or retrieve a stack traceback*. https://docs.python.org/3.10/library/traceback.html 