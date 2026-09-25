## Custom Exceptions: A Comprehensive Programming Cheat Sheet

### Topic Overview

#### Definitions

**Core Definition**
A custom exception is a user-defined error class that subclasses Python's built-in `Exception` (or a subclass thereof) to represent application-specific error conditions.

**Technical Definition**
Custom exceptions are classes that inherit from `Exception` or a more specific built-in exception type. They participate fully in Python's exception hierarchy: they can be raised with `raise`, caught with `except`, and their instances carry `args`, `__cause__`, and `__context__` attributes like any exception. By overriding `__init__` and `__str__`, they can carry structured metadata (error codes, transaction IDs, HTTP status codes) and produce customized error messages that make debugging and programmatic handling more precise.

**Beginner-Friendly Explanation**
Built-in exceptions like `ValueError` and `TypeError` describe generic problems. A custom exception is your own error type that says exactly what went wrong in your application—like `BalanceInsufficientError` instead of just `ValueError`. This makes code easier to read, debug, and handle correctly.

#### Key Characteristics

| Characteristic | Description |
|---|---|
| **Subclass of `Exception`** | Custom exceptions should inherit from `Exception`, not `BaseException`, to avoid interfering with system-exiting signals |
| **Descriptive Naming** | Names should end with `Error` (e.g., `PaymentError`, `ValidationError`) to follow convention |
| **Structured Metadata** | Can store attributes like `status_code`, `transaction_id`, or `retry_after` |
| **Hierarchical Design** | Can form trees where catching a parent catches all children |
| **Domain-Specific** | Represent application failures, not low-level programming errors |

#### Prerequisites

- Basic exception handling (`try`/`except`/`finally`)
- Understanding of class inheritance
- Familiarity with the built-in exception hierarchy

#### Related Programming Areas

- **Error Handling**: Catching and responding to exceptions
- **API Design**: Exceptions as part of a function's contract
- **Architecture**: Separating domain errors from infrastructure errors
- **Logging and Monitoring**: Structured errors for observability

#### Core Concepts / Features

1. **Defining Exception Classes**
2. **Overriding `__init__` and `__str__`**
3. **Domain-Specific Errors**
4. **Exception Hierarchy Design**


### Core Concept 1: Defining Exception Classes

#### Definitions

**Core Definition**
Defining an exception class means creating a new class that inherits from `Exception` (or a subclass) to represent a specific error condition.

**Technical Definition**
A custom exception class is created with `class MyError(Exception): pass`. It inherits `__init__`, `__str__`, `args`, and all other exception machinery from `Exception`. When raised, it propagates through the call stack like any built-in exception and can be caught with `except MyError`.

**Beginner-Friendly Explanation**
You create a custom exception by writing a class that extends `Exception`. The simplest version needs no body—just `pass`. Python gives it everything it needs to work as an exception.

#### Purposes

- **To give a specific, descriptive name** to an error condition that built-in exceptions don't capture
- **To make stack traces self-documenting**, so developers immediately understand what failed
- **To enable precise exception handling**, allowing callers to catch exactly the errors they can handle
- **To create a stable interface** that decouples error handling from implementation details
- **To group related errors** under a common base class for package-wide catching

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class CustomError(Exception):
    """Optional docstring describing when this error is raised."""
    pass
```

**Breakdown:**
- `class`: Defines a new class
- `CustomError`: The name, conventionally ending in `Error`
- `(Exception)`: Inherits from the built-in `Exception` base class 
- `pass`: The body can be empty; `Exception` provides `__init__`, `__str__`, and `args`

**Syntax Rules**

| Rule | Description |
|---|---|
| Inherit from `Exception` | Not `BaseException`; `BaseException` is reserved for system-exiting exceptions  |
| Name ends with `Error` | Convention; e.g., `PaymentError`, `ValidationError`  |
| Empty body allowed | `pass` is sufficient for simple exceptions |
| Docstring recommended | Describe when the exception is raised  |

**Constraints and Limitations**

- **No automatic formatting**: A bare `CustomError("msg")` prints exactly `"msg"` when uncaught
- **No structured data**: The base class stores arguments in `args` as a tuple; accessing specific fields requires `args[0]` unless you override `__init__`

#### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Simplest Custom Exception**

```python
# Define the exception class
class PaymentError(Exception):
    """Raised when a payment operation fails."""
    pass

# Raise it
try:
    raise PaymentError("Transaction declined by issuer")
except PaymentError as e:
    print(f"Caught: {e}")
    print(f"Type: {type(e).__name__}")
    print(f"Args: {e.args}")
```

**Expected Output:**
```
Caught: Transaction declined by issuer
Type: PaymentError
Args: ('Transaction declined by issuer',)
```

**Breakdown:**
1. `PaymentError` inherits all behavior from `Exception`.
2. `raise PaymentError("...")` creates an instance and raises it.
3. `str(e)` returns the message because `Exception.__str__` returns `args[0]` when there's one argument.

**Example 2: Custom Exception in a Function**

```python
class InsufficientFundsError(Exception):
    """Raised when an account has insufficient funds."""
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(
            f"Insufficient funds: balance={balance}, requested={amount}"
        )
    return balance - amount

try:
    withdraw(100, 150)
except InsufficientFundsError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Error: Insufficient funds: balance=100, requested=150
```

**Breakdown:**
1. The function checks the condition and raises the custom exception with a descriptive message.
2. The caller catches the specific type, making the handling intent explicit.

#### Real-World Cases with Explanation

**Case 1: Web Application Authentication**

```python
class InvalidCredentialsError(Exception):
    """Raised when username/password combination is invalid."""
    pass

class PermissionDeniedError(Exception):
    """Raised when a user lacks required permissions."""
    pass
```

**Why it matters:** Distinguishing authentication failure from authorization failure allows different handling: one redirects to login, the other returns 403 .

**Case 2: Data Processing Pipeline**

```python
class InvalidFileFormatError(Exception):
    """Raised when a file does not match the expected format."""
    pass

class MissingDataError(Exception):
    """Raised when required data is absent."""
    pass
```

**Why it matters:** Pipeline stages can catch format errors differently from missing-data errors, enabling targeted recovery strategies .


### Core Concept 2: Overriding `__init__` and `__str__`

#### Definitions

**Core Definition**
Overriding `__init__` allows a custom exception to accept and store structured metadata; overriding `__str__` controls how the exception is displayed when printed or converted to a string.

**Technical Definition**
The base `Exception.__init__(*args)` stores all arguments in `self.args`. Overriding `__init__` lets you extract named parameters, store them as attributes, and pass a formatted message to `super().__init__()`. Overriding `__str__` returns a custom string representation, which is used by `print()`, `str()`, and traceback formatting.

**Beginner-Friendly Explanation**
Instead of just carrying a message string, your exception can carry extra data—like an error code, a transaction ID, or an HTTP status. You write a custom `__init__` to store this data, and a custom `__str__` to make the error message include it.

#### Purposes

- **To carry structured metadata** (error codes, IDs, retry hints) that handlers can access programmatically 
- **To produce informative, formatted messages** that include context beyond a simple string
- **To distinguish between different failure modes** of the same exception type
- **To enable machine-readable error handling** without parsing message strings

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class CustomError(Exception):
    def __init__(self, message, *, metadata=None):
        super().__init__(message)  # Store the message in args
        self.metadata = metadata   # Store extra attributes

    def __str__(self):
        return f"{self.__class__.__name__}: {self.args[0]}"
```

**Breakdown:**
- `__init__(self, message, ...)`: Accepts custom parameters
- `super().__init__(message)`: **Must be called** to populate `args` and enable standard exception behavior 
- `self.metadata = metadata`: Stores additional structured data
- `__str__(self)`: Returns the string shown in tracebacks and by `print()` 

**Syntax Rules**

| Rule | Description |
|---|---|
| Always call `super().__init__()` | Omitting it leaves `args` empty and breaks `str()`  |
| Store extra data as attributes | `self.code = code`, `self.status = status` |
| `__str__` overrides display | Used by `print()`, `str()`, and tracebacks  |
| `__repr__` for debugging | Optional; `__str__` takes precedence in most contexts  |

**Constraints and Limitations**

- **`args` still exists**: Even with custom `__init__`, `args` contains what you pass to `super().__init__()`
- **Tracebacks use `str()`**: The formatted message appears in uncaught exception output
- **Keyword-only parameters**: Useful for metadata to avoid positional confusion

#### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Exception with Structured Metadata**

```python
class CardDeclinedError(Exception):
    """Raised when a card is declined, carrying transaction context."""

    def __init__(self, message, *, transaction_id=None, amount=None, http_status=402):
        super().__init__(message)  # Critical: pass message to base
        self.transaction_id = transaction_id
        self.amount = amount
        self.http_status = http_status

    def __str__(self):
        return (
            f"{self.__class__.__name__}: {self.args[0]} "
            f"[txn={self.transaction_id}, amount={self.amount}]"
        )

# Raise with structured data
try:
    raise CardDeclinedError(
        "Card declined by issuer",
        transaction_id="txn_abc123",
        amount=99.99
    )
except CardDeclinedError as e:
    print(f"Message: {e}")                # Uses __str__
    print(f"Transaction: {e.transaction_id}")
    print(f"Amount: {e.amount}")
    print(f"HTTP Status: {e.http_status}")
    print(f"Args: {e.args}")              # Base class still works
```

**Expected Output:**
```
Message: CardDeclinedError: Card declined by issuer [txn=txn_abc123, amount=99.99]
Transaction: txn_abc123
Amount: 99.99
HTTP Status: 402
Args: ('Card declined by issuer',)
```

**Breakdown:**
1. `super().__init__(message)` populates `args` so `str(e)` works even without `__str__` .
2. Extra attributes (`transaction_id`, `amount`, `http_status`) are accessible to handlers.
3. `__str__` uses `self.__class__.__name__` so subclasses automatically get their own name in the message .

**Example 2: Exception with Error Code and Retry Hint**

```python
class RateLimitError(Exception):
    """Raised when an API rate limit is exceeded."""

    def __init__(self, code, *, retry_after=None):
        super().__init__(f"Rate limit exceeded ({code})")
        self.code = code
        self.retry_after = retry_after

    def __str__(self):
        base = super().__str__()
        if self.retry_after is not None:
            return f"{base} — retry after {self.retry_after}s"
        return base

try:
    raise RateLimitError("TOO_MANY_REQUESTS", retry_after=30)
except RateLimitError as e:
    print(e)
    print(f"Code: {e.code}")
    print(f"Retry: {e.retry_after}")
```

**Expected Output:**
```
Rate limit exceeded (TOO_MANY_REQUESTS) — retry after 30s
Code: TOO_MANY_REQUESTS
Retry: 30
```

**Breakdown:**
1. `super().__init__(f"Rate limit exceeded ({code})")` builds the base message.
2. `__str__` augments the base message with retry information when present.
3. The handler can read `e.code` and `e.retry_after` to decide how long to wait .

#### Real-World Cases with Explanation

**Case 1: HTTP API Error Handling**

```python
class APIError(Exception):
    def __init__(self, message, status_code, endpoint):
        super().__init__(message)
        self.status_code = status_code
        self.endpoint = endpoint

    def __str__(self):
        return f"{self.status_code} at {self.endpoint}: {self.args[0]}"

# Handler can switch on status_code without parsing strings
try:
    raise APIError("Not found", 404, "/api/users/42")
except APIError as e:
    if e.status_code == 404:
        print("Resource missing")
```

**Why it matters:** Structured metadata lets handlers branch on error codes programmatically, while `__str__` provides a human-readable message for logs .

**Case 2: Payment Processing**

```python
class PaymentError(Exception):
    def __init__(self, message, transaction_id=None, decline_code=None):
        super().__init__(message)
        self.transaction_id = transaction_id
        self.decline_code = decline_code

    def __str__(self):
        return f"{self.args[0]} [txn={self.transaction_id}, decline={self.decline_code}]"
```

**Why it matters:** Payment errors carry transaction IDs for support tickets and decline codes for mapping to user-facing messages .


### Core Concept 3: Domain-Specific Errors

#### Definitions

**Core Definition**
Domain-specific errors are custom exceptions that represent failures within a specific architectural layer or business domain, such as database operations, HTTP routing, or payment processing.

**Technical Definition**
Domain-specific errors delineate boundaries between architectural layers. A database layer might raise `DatabaseError` subclasses, while an HTTP layer raises `HTTPError` subclasses. This separation prevents low-level implementation details (e.g., `sqlite3.OperationalError`) from leaking into higher layers, and allows each layer to handle its own error semantics without coupling to other layers' internals .

**Beginner-Friendly Explanation**
Different parts of your application should have their own error types. The database part shouldn't raise HTTP errors, and the HTTP part shouldn't catch SQLite errors directly. Domain-specific errors create clean boundaries between layers.

#### Purposes

- **To separate architectural layers** (database vs. HTTP vs. business logic) 
- **To prevent implementation details from leaking** across layer boundaries
- **To allow each layer to handle its own errors** without knowing about other layers
- **To make error handling intent explicit** in each part of the codebase
- **To enable independent evolution** of layers without breaking error handling

#### Syntax Rules and Structure

**Complete General Syntax**

```python
# Domain 1: Database layer
class DatabaseError(Exception):
    """Base for all database-related errors."""
    pass

class ConnectionError(DatabaseError):
    pass

class QueryError(DatabaseError):
    pass

# Domain 2: HTTP layer
class HTTPError(Exception):
    """Base for all HTTP-related errors."""
    pass

class NotFoundError(HTTPError):
    pass

class UnauthorizedError(HTTPError):
    pass
```

**Breakdown:**
- Each domain has its own base exception class
- Subclasses represent specific failures within that domain
- Layers catch only their own domain's errors

**Syntax Rules**

| Rule | Description |
|---|---|
| One base per domain | `DatabaseError`, `HTTPError`, `PaymentError`  |
| No cross-domain inheritance | `HTTPError` should not inherit from `DatabaseError` |
| Translate at boundaries | Catch low-level errors and raise domain errors with `from`  |
| Document which layer raises what | Clarify the contract between layers |

**Constraints and Limitations**

- **More classes to maintain**: Each domain needs its own hierarchy
- **Translation overhead**: Low-level errors must be caught and re-raised as domain errors
- **Avoid over-fragmentation**: Too many tiny domains can become unwieldy

#### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Separating Database and HTTP Errors**

```python
# --- Database domain ---
class DatabaseError(Exception):
    """Base for all database-related failures."""
    pass

class RecordNotFoundError(DatabaseError):
    """Raised when a database query returns no rows."""
    pass

# --- HTTP domain ---
class HTTPError(Exception):
    """Base for all HTTP-related failures."""
    def __init__(self, message, status_code=500):
        super().__init__(message)
        self.status_code = status_code

class NotFoundError(HTTPError):
    def __init__(self, message="Resource not found"):
        super().__init__(message, status_code=404)

# --- Service layer: translates database errors into HTTP errors ---
def get_user(user_id):
    try:
        # Simulate a database query that finds nothing
        raise RecordNotFoundError(f"User {user_id} not in database")
    except RecordNotFoundError as e:
        # Translate to HTTP layer error, preserving the cause
        raise NotFoundError(f"User {user_id} not found") from e

# --- Handler catches HTTP errors, not database errors ---
try:
    get_user(42)
except NotFoundError as e:
    print(f"HTTP {e.status_code}: {e}")
    print(f"Cause: {e.__cause__}")
except DatabaseError as e:
    print(f"Database problem: {e}")  # Not reached in this case
```

**Expected Output:**
```
HTTP 404: User 42 not found
Cause: User 42 not in database
```

**Breakdown:**
1. The database layer raises `RecordNotFoundError`.
2. The service layer catches it and translates to `NotFoundError` (HTTP domain) using `raise ... from e` .
3. The handler catches `NotFoundError` and can read `status_code` without knowing about the database .

**Example 2: Payment vs. Validation Domains**

```python
# Validation domain
class ValidationError(Exception):
    """Base for input validation errors."""
    pass

class MissingFieldError(ValidationError):
    def __init__(self, field):
        super().__init__(f"Required field missing: {field}")
        self.field = field

# Payment domain
class PaymentError(Exception):
    """Base for payment processing errors."""
    pass

class InsufficientFundsError(PaymentError):
    def __init__(self, balance, amount):
        super().__init__(f"Insufficient funds: balance={balance}, amount={amount}")
        self.balance = balance
        self.amount = amount

# A checkout function that uses both domains
def checkout(cart, account):
    if "payment_method" not in cart:
        raise MissingFieldError("payment_method")  # Validation domain
    if cart["total"] > account["balance"]:
        raise InsufficientFundsError(account["balance"], cart["total"])  # Payment domain
    return "Success"

# Handler can catch domains separately
try:
    checkout({"total": 100}, {"balance": 50})
except ValidationError as e:
    print(f"Validation: {e}")
except PaymentError as e:
    print(f"Payment: {e}")
```

**Expected Output:**
```
Payment: Insufficient funds: balance=50, amount=100
```

**Breakdown:**
1. `MissingFieldError` belongs to the validation domain; `InsufficientFundsError` belongs to the payment domain.
2. The handler catches each domain with its own `except` clause.
3. If both could be raised, order and specificity matter.

#### Real-World Cases with Explanation

**Case 1: Web Application Layers**

```python
# Repository layer
class RepositoryError(Exception): pass
class EntityNotFoundError(RepositoryError): pass

# Service layer
class ServiceError(Exception): pass
class UserNotFoundError(ServiceError): pass

# Controller layer
class APIError(Exception): pass
class UserEndpointError(APIError): pass
```

**Why it matters:** Each layer can evolve independently. The controller doesn't import `sqlite3`; it catches `ServiceError` and returns appropriate HTTP responses .

**Case 2: Data Pipeline Stages**

```python
class ExtractionError(Exception): pass
class TransformationError(Exception): pass
class LoadingError(Exception): pass
```

**Why it matters:** Each pipeline stage raises its own error type, allowing operators to monitor and alert on specific stage failures .


### Core Concept 4: Exception Hierarchy Design

#### Definitions

**Core Definition**
Exception hierarchy design is the practice of organizing custom exceptions into a tree structure with a common base class, enabling callers to catch errors at different levels of specificity.

**Technical Definition**
A well-designed exception hierarchy has a root base class (e.g., `MyLibraryError`) that inherits from `Exception`. All other exceptions in the library or application inherit from this base, directly or indirectly. This allows callers to catch `MyLibraryError` for all library errors, or a specific subclass for targeted handling. The hierarchy mirrors the domain's natural error categories .

**Beginner-Friendly Explanation**
Think of your exceptions as a family tree. The root is `MyLibraryError`. Its children are `ValidationError`, `PaymentError`, and `NetworkError`. Each child may have its own children. A caller can catch the whole family, one branch, or a single leaf.

#### Purposes

- **To allow callers to choose their specificity**: catch everything (`LibraryError`), a category (`PaymentError`), or a specific case (`CardDeclinedError`) 
- **To provide a stable public API**: even as internal exceptions change, the base class remains catchable
- **To enable layered catching**: different layers can catch at different levels
- **To make the error taxonomy explicit** and discoverable in code
- **To support package-wide error handling** with a single `except` target 

#### Syntax Rules and Structure

**Complete General Syntax**

```python
# Root base class
class LibraryError(Exception):
    """Base for all library errors."""
    pass

# Category: Configuration
class ConfigurationError(LibraryError):
    pass

class MissingConfigError(ConfigurationError):
    pass

class InvalidConfigError(ConfigurationError):
    pass

# Category: Validation
class ValidationError(LibraryError):
    pass

class MissingFieldError(ValidationError):
    pass

# Category: API/Network
class APIError(LibraryError):
    pass

class TimeoutError(APIError):
    pass
```

**Breakdown:**
- `LibraryError(Exception)`: The root; all library exceptions inherit from it 
- `ConfigurationError(LibraryError)`: A category of errors
- `MissingConfigError(ConfigurationError)`: A specific case
- Callers can catch at any level: `except LibraryError`, `except ConfigurationError`, or `except MissingConfigError`

**Syntax Rules**

| Rule | Description |
|---|---|
| Single root base | One base class for the whole package/library  |
| Categories inherit from root | `ConfigurationError(LibraryError)` |
| Specifics inherit from categories | `MissingConfigError(ConfigurationError)` |
| Names end with `Error` | `ConfigurationError`, not `ConfigurationException`  |
| Document the hierarchy | Docstrings on each class explaining when it's raised |

**Constraints and Limitations**

- **Order matters in `except`**: Specific exceptions must come before their parents, or the specific handler never runs 
- **Avoid deep nesting**: 3–4 levels is usually sufficient; deeper hierarchies become hard to navigate
- **Don't over-engineer**: Only create categories you'll actually catch separately

#### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Three-Level Hierarchy**

```python
# Root
class AppError(Exception):
    """Base for all application errors."""
    pass

# Category: Validation
class ValidationError(AppError):
    """Input validation failures."""
    pass

class MissingFieldError(ValidationError):
    def __init__(self, field):
        super().__init__(f"Missing required field: {field}")
        self.field = field

class InvalidFormatError(ValidationError):
    def __init__(self, field, value):
        super().__init__(f"Invalid format for {field}: {value!r}")
        self.field = field
        self.value = value

# Category: Payment
class PaymentError(AppError):
    """Payment processing failures."""
    pass

class InsufficientFundsError(PaymentError):
    def __init__(self, balance, amount):
        super().__init__(f"Insufficient funds: {balance} < {amount}")
        self.balance = balance
        self.amount = amount

# --- Catch at different levels ---
def demo_catch(exc):
    try:
        raise exc
    except MissingFieldError as e:
        print(f"Specific: {e}")
    except ValidationError as e:
        print(f"Category: {e}")
    except AppError as e:
        print(f"Root: {e}")

demo_catch(MissingFieldError("email"))
demo_catch(InvalidFormatError("age", "abc"))
demo_catch(InsufficientFundsError(50, 100))
```

**Expected Output:**
```
Specific: Missing required field: email
Category: Invalid format for age: 'abc'
Root: Insufficient funds: 50 < 100
```

**Breakdown:**
1. `MissingFieldError` matches the first `except` clause (most specific).
2. `InvalidFormatError` doesn't match `MissingFieldError`, but matches `ValidationError`.
3. `InsufficientFundsError` matches only `AppError` because it's a `PaymentError`, not a `ValidationError` .

**Example 2: Order Matters**

```python
class AppError(Exception):
    pass

class ValidationError(AppError):
    pass

class MissingFieldError(ValidationError):
    pass

# WRONG ORDER: AppError first
try:
    raise MissingFieldError("email")
except AppError as e:
    print("Caught by AppError (WRONG — too broad)")
except MissingFieldError as e:
    print("Caught by MissingFieldError (never reached)")
```

**Expected Output:**
```
Caught by AppError (WRONG — too broad)
```

**Breakdown:**
1. Python tries `except` clauses top to bottom and stops at the first match .
2. Since `MissingFieldError` is a subclass of `AppError`, `AppError` matches first.
3. The specific handler is dead code. **Always put specific exceptions above their parents** .

#### Real-World Cases with Explanation

**Case 1: Enterprise Module with Package-Wide Catching**

```python
# mylib/errors.py
class MyLibError(Exception):
    """Base for all mylib errors."""
    pass

class ConfigurationError(MyLibError): pass
class ValidationError(MyLibError): pass
class APIError(MyLibError): pass

# Consumer code
try:
    mylib.do_something()
except MyLibError as e:
    log.error(f"Library failure: {e}")
    # Handles all mylib errors with one clause
```

**Why it matters:** The consumer doesn't need to know about every internal exception type. `except MyLibError` catches everything the library raises .

**Case 2: Web Framework Error Handling**

```python
class WebFrameworkError(Exception): pass

class RoutingError(WebFrameworkError): pass
class MethodNotAllowedError(RoutingError): pass

class DatabaseError(WebFrameworkError): pass
class RecordNotFoundError(DatabaseError): pass

# Middleware catches all framework errors
try:
    handle_request(request)
except WebFrameworkError as e:
    return error_response(e)
```

**Why it matters:** The framework's middleware can catch all framework errors with a single clause, while individual handlers can catch specific types for targeted responses .


### References

- Python Software Foundation. *Built-in Exceptions*. https://docs.python.org/3.12/library/exceptions.html 
- Python Software Foundation. *PEP 352 – Required Superclass for Exceptions*. https://peps.python.org/pep-0352/ 
- Python.org Tutor Mailing List. *Why should modules or packages define their own domain-specific base exception class?* https://mail.python.org/archives/list/tutor@python.org/message/OREH26OHT36QX7ZOSCXN6DIDGSTMEXJI/ 
- Real Python. *exception handling | Python Best Practices*. https://realpython.com/ref/best-practices/exception-handling/ 
- Alibaba Cloud Developer. *Classes & OOP—Defining Your Own Exception Classes*. https://developer.aliyun.com/article/1618772 
- Ruff TRY/BLE/EM/LOG/N818. *Python exception design, raising, handling, and logging*. https://raw.githubusercontent.com/leynos/episodic/690236e8e58b4134b3729c5bc4a4736c46f71d3f/.rules/python-exception-design-raising-handling-and-logging.md 