# Python Advanced Typing: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Advanced typing in Python refers to the use of the `typing` module and related syntax to express static type information for variables, functions, classes, and data structures beyond basic type annotations. It encompasses features such as unions, generics, protocols, literal types, and metadata attachment, enabling static type checkers to detect errors before runtime.

### Technical Definition

Advanced typing comprises a set of special forms, type constructors, and syntactic extensions standardized through Python Enhancement Proposals (PEPs) that extend Python's gradual type system beyond simple nominal annotations. These features allow programmers to express parametric polymorphism, structural subtyping, literal value constraints, variance, finality, and arbitrary metadata on types, all of which are consumed by static type checkers such as mypy, pyright, and pyre.

### Beginner-Friendly Explanation

Think of basic type hints as telling Python "this variable is a number" or "this function returns text." Advanced typing goes further: it lets you say "this function returns *either* a number *or* nothing," "this container holds items of whatever type you specify," "this dictionary must have exactly these keys," "this argument is only ever the string `'left'` or `'right'`," or "this method returns an instance of whatever subclass calls it." These richer descriptions help your editor catch mistakes before you run the code, and they make your code easier for others to understand.

### Key Characteristics

- **Gradual typing**: Type hints are optional and do not affect runtime behavior; they are checked by external tools.
- **Standardized via PEPs**: Each feature is specified in a Python Enhancement Proposal, ensuring broad consistency across type checkers.
- **Backward-compatible**: New features are typically introduced in the `typing` module and later backported via `typing_extensions`.
- **Runtime accessible**: Many typing constructs can be inspected at runtime (e.g., `get_origin()`, `get_args()`).
- **Version-dependent**: Syntax and features vary by Python version (e.g., `X | Y` requires Python 3.10+; `Self` requires 3.11+; `type` statement requires 3.12+).

### Prerequisites

- Basic understanding of Python functions, classes, and variables.
- Familiarity with basic type annotations (`int`, `str`, `list[int]`, `Optional[str]`).
- A static type checker installed (e.g., `mypy`, `pyright`).
- Python 3.8 or newer (some features require 3.10+, 3.11+, or 3.12+).

### Related Programming Areas

- **Static analysis and type checking**: mypy, pyright, pyre, pytype.
- **Data validation and serialization**: Pydantic, attrs, dataclasses.
- **Web frameworks**: FastAPI, Django (with type stubs), Litestar.
- **API design**: Flask, Starlette.
- **Generics and design patterns**: Repository pattern, dependency injection.
- **Runtime type inspection**: `typing.get_type_hints()`, `typing.get_origin()`.

### Core Concepts / Features

The following sections cover each advanced typing feature using a uniform structure.

---

## 1. Optional and Union Types (and the Modern `|` Operator)

### Definitions

**Core Definition**: `Optional` and `Union` express that a value may be one of several types, or may be absent (`None`).

**Technical Definition**: `typing.Union[X, Y]` denotes the untagged union of types `X` and `Y`; a value of type `Union[X, Y]` is either a value of type `X` or a value of type `Y`. `typing.Optional[X]` is equivalent to `Union[X, None]`. PEP 604 introduced the `|` operator as syntactic sugar for `Union`, i.e., `X | Y` is equivalent to `Union[X, Y]`. The `|` operator also works with `isinstance()` and `issubclass()`.

**Beginner-Friendly Explanation**: "This thing can be one of these types." For example, `Optional[str]` means "a string or `None`." `Union[int, str]` means "an integer or a string." The modern `|` syntax lets you write `int | str` instead of `Union[int, str]`.

### Purposes

- To express that a value may be absent (`None`) or one of several alternatives.
- To simplify function signatures that accept multiple input types.
- To improve error messages from type checkers by narrowing types at runtime.
- To enable pattern matching and exhaustiveness checking over union members.
- To replace verbose `Union[X, Y]` with the concise `X | Y` syntax.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: `Union` from `typing`**
```python
from typing import Union, Optional

# Union of two or more types
value: Union[int, str]
value: Union[int, str, float]

# Optional (equivalent to Union[X, None])
value: Optional[int]  # int | None
```

**Component Breakdown**:
- `Union[` — special form from `typing` indicating a union type.
- `Type1, Type2, ...` — one or more type expressions separated by commas.
- `]` — closes the union.

- `Optional[` — special form equivalent to `Union[X, None]`.
- `Type` — the type that may be absent.
- `]` — closes `Optional`.

**Syntax 2: PEP 604 `|` operator (Python 3.10+)**
```python
# Union of two or more types using |
value: int | str
value: int | str | float

# Optional using |
value: int | None
```

**Component Breakdown**:
- `Type1 | Type2` — overloads `type.__or__()` to create a union type.
- `|` — the pipe operator, used here to combine types.
- `None` — the type of `None` (equivalent to `type(None)`).

**Syntax 3: `isinstance` and `issubclass` with `|`**
```python
isinstance(5, int | str)        # True
issubclass(bool, int | float)    # True
```

**Component Breakdown**:
- `isinstance(obj, type_expr)` — checks if `obj` is an instance of `type_expr`.
- `issubclass(cls, type_expr)` — checks if `cls` is a subclass of `type_expr`.

#### Syntax Rules

1. **Order does not matter**: `Union[int, str]` is equivalent to `Union[str, int]`.
2. **Duplicates are collapsed**: `Union[int, int]` is equivalent to `int`; `int | int` is equivalent to `int`.
3. **`Optional[X]` is exactly `Union[X, None]`**: `Optional[int]` ≡ `int | None` ≡ `Union[int, None]`.
4. **`|` requires Python 3.10+** for type annotations; for older versions, use `from __future__ import annotations` to defer evaluation, but runtime uses of `|` (e.g., in `isinstance`) still require 3.10+.
5. **Parameterized generics cannot be used with `isinstance`**: `isinstance(x, list[int])` raises `TypeError`; use `list` or `int | list` if needed.
6. **Metaclass `__or__` can override**: If a metaclass defines `__or__`, it takes precedence.

#### Constraints and Limitations

- `Union` cannot be used with `isinstance` or `issubclass` at runtime in a parameterized form (e.g., `list[int]`).
- `|` is not available in Python versions before 3.10 for runtime evaluation.
- `Optional` does not mean "optional keyword argument"; it means the value may be `None`. An argument with a default value is still required to have its type annotated separately.
- Type narrowing requires explicit `is not None` checks or `assert` statements.
- Nested unions are flattened by type checkers.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: `Union` and `Optional` with `isinstance` Narrowing

```python
# step1: Import typing constructs
from typing import Union, Optional

# step2: Define a function that accepts a union
def process(value: Union[int, str]) -> str:
    # step3: Narrow the type using isinstance
    if isinstance(value, int):
        return f"Integer: {value * 2}"
    else:
        return f"String: {value.upper()}"

# step4: Call with different types
print(process(10))        # Integer: 20
print(process("hello"))   # String: HELLO

# step5: Optional example
def greet(name: Optional[str] = None) -> str:
    if name is None:
        return "Hello, stranger!"
    return f"Hello, {name}!"

print(greet())            # Hello, stranger!
print(greet("Alice"))     # Hello, Alice!
```

**Expected Output**:
```
Integer: 20
String: HELLO
Hello, stranger!
Hello, Alice!
```

**Why**: `process` accepts either an `int` or a `str`. The `isinstance` check narrows the type for each branch. `greet` accepts an optional string; `None` is handled explicitly.

#### Example 2: PEP 604 `|` Operator (Python 3.10+)

```python
# step1: Use | directly in annotations (Python 3.10+)
def format_id(id: int | str) -> str:
    if isinstance(id, int):
        return f"ID-{id:04d}"
    return f"ID-{id.upper()}"

print(format_id(42))        # ID-0042
print(format_id("abc"))     # ID-ABC

# step2: Runtime isinstance with |
print(isinstance(5, int | str))     # True
print(isinstance(5.0, int | str))   # False

# step3: Optional with |
def get_user(user_id: int) -> dict | None:
    if user_id == 1:
        return {"name": "Alice"}
    return None

print(get_user(1))   # {'name': 'Alice'}
print(get_user(2))   # None
```

**Expected Output**:
```
ID-0042
ID-ABC
True
False
{'name': 'Alice'}
None
```

**Why**: The `|` operator creates union types at runtime, enabling `isinstance` checks. `dict | None` is equivalent to `Optional[dict]`.

#### Example 3: `Union` with Complex Types

```python
from typing import Union

# A union of a list and a tuple
def total(items: Union[list[int], tuple[int, ...]]) -> int:
    return sum(items)

print(total([1, 2, 3]))       # 6
print(total((4, 5, 6)))       # 15

# Union with None and a custom class
class Config:
    def __init__(self, debug: bool = False):
        self.debug = debug

def get_config(config: Union[Config, None]) -> bool:
    if config is None:
        return False
    return config.debug

print(get_config(Config(debug=True)))  # True
print(get_config(None))                # False
```

**Expected Output**:
```
6
15
True
False
```

**Why**: `Union` works with any type expressions, including generic collections and user-defined classes.

### Real-World Cases

- **API responses**: A function may return a `dict` on success or `None` on failure: `dict | None`.
- **Configuration parsing**: A setting may be an `int`, `str`, or `None`.
- **Event handling**: A callback may receive an `Event` object or a `str` error message.
- **Pattern matching**: `Union` types enable exhaustive `match` statements.

### References

- Union Type - https://docs.python.org/3/library/typing.html#typing.Union
- Optional Type - https://docs.python.org/3/library/typing.html#typing.Optional
- PEP 604 – Allow writing union types as X | Y - https://peps.python.org/pep-0604/
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/

---

## 2. Literal Types

### Definitions

**Core Definition**: `Literal` specifies that a value must be exactly one of a fixed set of literal values.

**Technical Definition**: `typing.Literal[v1, v2, ...]` denotes a type whose members are exactly the literal values `v1, v2, ...`. Literal types are subtypes of their corresponding base types (e.g., `Literal[3]` is a subtype of `int`). They are used to constrain arguments, return types, and variables to specific constant values.

**Beginner-Friendly Explanation**: "This value must be exactly `'left'` or `'right'` — nothing else." It's like an enum but for literal values like strings, integers, and booleans.

### Purposes

- To constrain a value to a specific set of literal constants (strings, ints, bools, `None`).
- To enable type checkers to catch invalid constant values at compile time.
- To improve overloading based on literal values (e.g., different return types for different modes).
- To document the exact allowed values of a parameter in the type signature.
- To support exhaustiveness checking with `assert_never`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Literal

# Single literal
mode: Literal["r"]

# Multiple literals
mode: Literal["r", "w", "a", "x"]

# Mixed types
value: Literal[42, "foo", True]

# Function parameter
def open_file(path: str, mode: Literal["r", "w", "a"]) -> None:
    ...
```

**Component Breakdown**:
- `Literal[` — special form from `typing`.
- `value1, value2, ...` — one or more literal values (int, str, bool, None, bytes).
- `]` — closes the literal.

#### Syntax Rules

1. **At least one value**: `Literal[]` and bare `Literal` are illegal.
2. **Values must be literals**: ints, strings, booleans, `None`, and bytes. Enums and other expressions are not allowed.
3. **Equivalent literals**: `Literal[20]` and `Literal[0x14]` are equivalent because `20 == 0x14` and both are `int`.
4. **`Literal[0]` ≠ `Literal[False]`**: Although `0 == False` at runtime, they have different types (`int` vs `bool`).
5. **Shortening unions**: `Literal[v1, v2, v3]` is equivalent to `Literal[v1] | Literal[v2] | Literal[v3]`.
6. **Type checkers do not enforce at runtime**: `Literal` is a static-only construct.

#### Constraints and Limitations

- Only immutable literal values are allowed.
- `Literal` cannot contain arbitrary expressions.
- Type narrowing with `Literal` requires `match` statements or explicit checks.
- Some type checkers may not fully support all literal operations.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Literal Usage

```python
from typing import Literal

# step1: Define a function that only accepts specific string literals
def set_mode(mode: Literal["r", "w", "a"]) -> str:
    return f"Mode set to {mode}"

# step2: Valid calls
print(set_mode("r"))   # Mode set to r
print(set_mode("w"))   # Mode set to w

# step3: Invalid call (type checker would flag this)
# set_mode("x")  # Error: Argument 1 has incompatible type "Literal['x']"

# step4: Literal with integers
def set_volume(level: Literal[0, 1, 2, 3]) -> str:
    return f"Volume: {level}"

print(set_volume(2))   # Volume: 2
```

**Expected Output**:
```
Mode set to r
Mode set to w
Volume: 2
```

**Why**: `set_mode` is constrained to only `"r"`, `"w"`, or `"a"`. `set_volume` is constrained to integers 0–3.

#### Example 2: Literal with `assert_never` for Exhaustiveness

```python
from typing import Literal, assert_never

# step1: Define a Literal type alias
Direction = Literal["north", "south", "east", "west"]

# step2: Function that handles all directions
def move(direction: Direction) -> str:
    if direction == "north":
        return "Moving up"
    elif direction == "south":
        return "Moving down"
    elif direction == "east":
        return "Moving right"
    elif direction == "west":
        return "Moving left"
    else:
        # step3: assert_never ensures all cases are handled
        assert_never(direction)

print(move("north"))   # Moving up
print(move("west"))    # Moving left
```

**Expected Output**:
```
Moving up
Moving left
```

**Why**: If a new direction were added to the `Direction` type without updating `move`, the type checker would report an error at `assert_never(direction)`, ensuring exhaustiveness.

#### Example 3: Literal in Overloaded Functions

```python
from typing import Literal, overload

# step1: Define overloads based on literal mode
@overload
def open_file(path: str, mode: Literal["r"]) -> str: ...
@overload
def open_file(path: str, mode: Literal["w"]) -> None: ...
def open_file(path: str, mode: Literal["r", "w"]) -> str | None:
    if mode == "r":
        return f"Reading {path}"
    return None

# step2: Type checker infers return type from mode
result = open_file("data.txt", "r")   # str
print(result)                         # Reading data.txt

result2 = open_file("data.txt", "w")  # None
print(result2)                        # None
```

**Expected Output**:
```
Reading data.txt
None
```

**Why**: The overload signatures allow the type checker to infer that `mode="r"` returns `str` and `mode="w"` returns `None`.

### Real-World Cases

- **File modes**: `open()` uses `Literal` to constrain mode strings.
- **HTTP methods**: `Literal["GET", "POST", "PUT", "DELETE"]`.
- **State machines**: `Literal["idle", "running", "stopped"]`.
- **API versions**: `Literal["v1", "v2"]`.

### References

- Literal Types - https://typing.python.org/en/latest/spec/literal.html
- PEP 586 – Literal Types - https://peps.python.org/pep-0586/
- `typing.Literal` - https://docs.python.org/3/library/typing.html#typing.Literal

---

## 3. `Any` vs. `object` vs. `Never`/`NoReturn`

### Definitions

**Core Definition**: `Any` means "any type, no restrictions"; `object` means "any object, but only safe operations"; `Never`/`NoReturn` means "this value cannot exist" (the bottom type).

**Technical Definition**:
- `typing.Any` is a special form that is consistent with every type. Assigning a value of type `Any` to any other type is allowed, and vice versa. It effectively disables type checking for that value.
- `object` is the base class of all Python types. A value of type `object` can only be used in ways that are valid for every object (e.g., `str()`, `repr()`, `isinstance()`). It is the safe top type.
- `typing.Never` (Python 3.11+) is the bottom type: a type with no values. It is equivalent to `typing.NoReturn`, which was originally intended for functions that never return. Both indicate that a function never returns normally (it either raises an exception or loops forever).

**Beginner-Friendly Explanation**:
- `Any` says: "I don't care what type this is — turn off checking."
- `object` says: "This is some object, but I don't know what kind, so don't assume it has any special methods."
- `Never`/`NoReturn` says: "This function never returns — it always raises an error or runs forever."

### Purposes

- To represent dynamically typed values when migrating untyped code (`Any`).
- To safely accept any object without assuming its interface (`object`).
- To annotate functions that never return, such as `sys.exit()` or `raise` helpers (`Never`/`NoReturn`).
- To enable exhaustiveness checking with `assert_never`.
- To distinguish between "I don't know the type" and "this is any object."

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Any, Never, NoReturn

# Any
def process(data: Any) -> Any:
    return data

# object
def stringify(obj: object) -> str:
    return str(obj)

# Never / NoReturn
def fail(message: str) -> Never:
    raise RuntimeError(message)

def loop_forever() -> NoReturn:
    while True:
        pass
```

**Component Breakdown**:
- `Any` — special form; can be used as a type annotation.
- `object` — built-in type; the root of the class hierarchy.
- `Never` — special form introduced in Python 3.11; the bottom type.
- `NoReturn` — special form introduced in Python 3.6.2; equivalent to `Never`.

#### Syntax Rules

1. **`Any` is compatible with everything**: `Any` can be assigned to `int`, `int` can be assigned to `Any`, and operations on `Any` are unchecked.
2. **`object` is the safe top type**: Every type is a subtype of `object`, but `object` is only a subtype of `Any`.
3. **Operations on `object` are limited**: You cannot call arbitrary methods on an `object` without narrowing.
4. **`Never`/`NoReturn` only valid as return type**: Using them as parameter types is allowed but unusual (only for functions that are never called).
5. **`Never` and `NoReturn` are equivalent**: Type checkers treat them identically.
6. **`Never` requires Python 3.11+ or `typing_extensions`**: For earlier versions, use `NoReturn`.

#### Constraints and Limitations

- `Any` disables type checking and should be used sparingly.
- `object` cannot be used to call methods without a type narrowing check (`isinstance`).
- `Never`/`NoReturn` cannot be used as a type annotation for variables that are expected to hold values.
- Some type checkers may not fully support `Never` in all positions.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: `Any` vs. `object`

```python
from typing import Any

# step1: Any allows any operation (unchecked)
def process_any(x: Any) -> Any:
    return x.upper()  # No type error, but runtime may fail

print(process_any("hello"))  # HELLO

# step2: object restricts operations
def process_object(x: object) -> str:
    # return x.upper()  # Type error: object has no attribute 'upper'
    if isinstance(x, str):
        return x.upper()
    return str(x)

print(process_object("hello"))  # HELLO
print(process_object(42))       # 42
```

**Expected Output**:
```
HELLO
HELLO
42
```

**Why**: `Any` disables checking, so `x.upper()` is allowed even though `x` might not have that method. `object` forces an `isinstance` check before calling `upper()`.

#### Example 2: `Never` / `NoReturn` with Exhaustiveness

```python
from typing import Never, assert_never
import sys

# step1: Function that never returns
def fatal_error(message: str) -> Never:
    print(f"FATAL: {message}")
    sys.exit(1)

# step2: Exhaustiveness checking
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

def describe(color: Color) -> str:
    if color == Color.RED:
        return "Red"
    elif color == Color.GREEN:
        return "Green"
    elif color == Color.BLUE:
        return "Blue"
    else:
        assert_never(color)

print(describe(Color.RED))    # Red
print(describe(Color.BLUE))   # Blue
```

**Expected Output**:
```
Red
Blue
```

**Why**: `fatal_error` never returns, so its return type is `Never`. `assert_never` uses `Never` to verify that all enum members are handled.

#### Example 3: `NoReturn` for Legacy Code

```python
from typing import NoReturn

def raise_error(msg: str) -> NoReturn:
    raise ValueError(msg)

# Type checker knows code after this is unreachable
def check(value: int) -> int:
    if value < 0:
        raise_error("Negative value")
    return value

print(check(5))   # 5
# check(-1)       # Raises ValueError
```

**Expected Output**:
```
5
```

**Why**: `NoReturn` tells the type checker that `raise_error` never returns, so the code after the `if` is reachable only when `value >= 0`.

### Real-World Cases

- **Plugin systems**: `Any` for arbitrary plugin data; `object` for safe generic containers.
- **Error handling**: `Never` for functions that always raise.
- **Legacy migration**: `Any` to gradually add types to untyped code.
- **Library APIs**: `object` as a safe default for parameters that accept anything.

### References

- `typing.Any` - https://docs.python.org/3/library/typing.html#typing.Any
- `typing.Never` - https://docs.python.org/3/library/typing.html#typing.Never
- `typing.NoReturn` - https://docs.python.org/3/library/typing.html#typing.NoReturn
- Unreachable Code and Exhaustiveness Checking - https://typing.python.org/en/latest/guides/unreachable.html
- PEP 484 – Type Hints (NoReturn) - https://peps.python.org/pep-0484/

---

## 4. `Callable` (Annotating Functions and Callback Signatures)

### Definitions

**Core Definition**: `Callable` is the type used to annotate functions, lambdas, and other callable objects, specifying their parameter types and return type.

**Technical Definition**: `typing.Callable[[ArgType1, ArgType2], ReturnType]` (or `collections.abc.Callable` in Python 3.9+) denotes a callable that accepts arguments of types `ArgType1, ArgType2` and returns a value of type `ReturnType`. `Callable[..., ReturnType]` denotes a callable with arbitrary arguments and a specific return type. `Callable` alone is equivalent to `Callable[..., Any]`.

**Beginner-Friendly Explanation**: "This is a function that takes these kinds of arguments and returns this kind of value." It's like a contract for functions you pass around.

### Purposes

- To annotate callbacks and higher-order functions.
- To specify the expected signature of a function parameter.
- To document the return type of a function that returns another function.
- To enable type-safe dependency injection and event handling.
- To distinguish between callables with different signatures.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Callable

# Callable with specific parameters and return type
handler: Callable[[int, str], bool]

# Callable with any arguments
handler: Callable[..., bool]

# Callable with no arguments
handler: Callable[[], None]

# Using collections.abc (Python 3.9+)
from collections.abc import Callable
handler: Callable[[int, str], bool]
```

**Component Breakdown**:
- `Callable[` — special form (or `collections.abc.Callable`).
- `[ArgType1, ArgType2, ...]` — a list of parameter types (not parameter names).
- `ReturnType` — the return type of the callable.
- `...` — ellipsis, indicating any arguments.

#### Syntax Rules

1. **Parameter list must be a list**: Use `[]` even for a single parameter: `Callable[[int], str]`.
2. **No argument names**: Only types are specified: `Callable[[int, str], bool]`, not `Callable[[x: int, y: str], bool]`.
3. **No keyword-only or optional arguments**: `Callable` cannot express default values or keyword-only parameters.
4. **`Callable[..., R]`**: Accepts any arguments and returns `R`.
5. **`Callable` alone**: Equivalent to `Callable[..., Any]`.
6. **Deprecation**: `typing.Callable` is deprecated in favor of `collections.abc.Callable` since Python 3.9, but both work.

#### Constraints and Limitations

- Cannot express parameter names or default values.
- Cannot express keyword-only arguments.
- Variance rules apply: a callable returning a subtype is compatible where a supertype return is expected; parameters are contravariant.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Callable Annotation

```python
from typing import Callable

# step1: Define a function that takes a callback
def apply_twice(func: Callable[[int], int], value: int) -> int:
    return func(func(value))

# step2: Define a callback
def double(x: int) -> int:
    return x * 2

# step3: Call with the callback
print(apply_twice(double, 5))   # 20

# step4: Use a lambda
print(apply_twice(lambda x: x + 1, 10))  # 12
```

**Expected Output**:
```
20
12
```

**Why**: `apply_twice` expects a callable that takes an `int` and returns an `int`. `double` matches this signature.

#### Example 2: Callable with Arbitrary Arguments

```python
from typing import Callable, Any

# step1: Define a function that accepts any callable
def call_with_logging(func: Callable[..., Any], *args: Any, **kwargs: Any) -> Any:
    print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
    result = func(*args, **kwargs)
    print(f"Result: {result}")
    return result

# step2: Call with different functions
def add(a: int, b: int) -> int:
    return a + b

call_with_logging(add, 3, 4)
# Calling add with args=(3, 4), kwargs={}
# Result: 7

call_with_logging(print, "hello")
# Calling print with args=('hello',), kwargs={}
# hello
# Result: None
```

**Expected Output**:
```
Calling add with args=(3, 4), kwargs={}
Result: 7
Calling print with args=('hello',), kwargs={}
hello
Result: None
```

**Why**: `Callable[..., Any]` accepts any callable regardless of its signature.

#### Example 3: Callable in a Class (Event Handler)

```python
from typing import Callable, list

# step1: Define an event system
class EventEmitter:
    def __init__(self) -> None:
        self._handlers: list[Callable[[str], None]] = []

    def on(self, handler: Callable[[str], None]) -> None:
        self._handlers.append(handler)

    def emit(self, event: str) -> None:
        for handler in self._handlers:
            handler(event)

# step2: Use the event system
emitter = EventEmitter()

def log_event(event: str) -> None:
    print(f"Logging: {event}")

emitter.on(log_event)
emitter.on(lambda e: print(f"Lambda: {e}"))

emitter.emit("user_login")
# Logging: user_login
# Lambda: user_login
```

**Expected Output**:
```
Logging: user_login
Lambda: user_login
```

**Why**: `on` accepts any callable that takes a `str` and returns `None`. Both `log_event` and the lambda match this signature.

### Real-World Cases

- **GUI frameworks**: Button click handlers.
- **Web frameworks**: Route decorators and middleware.
- **Asynchronous programming**: `asyncio` callbacks.
- **Data pipelines**: Transformation functions.

### References

- `typing.Callable` - https://docs.python.org/3/library/typing.html#typing.Callable
- `collections.abc.Callable` - https://docs.python.org/3/library/collections.abc.html#collections.abc.Callable
- PEP 484 – Type Hints (Callable) - https://peps.python.org/pep-0484/#callable

---

## 5. Type Variables (`TypeVar`) and Modern Generic Syntax (`X[T]`)

### Definitions

**Core Definition**: A `TypeVar` is a placeholder for a type that will be specified later, enabling generic functions and classes that work with multiple types.

**Technical Definition**: `typing.TypeVar('T')` creates a type variable `T` that can be used in generic function signatures, class definitions, and type aliases. A type variable can be constrained to a set of types (`TypeVar('T', int, str)`), bounded by a supertype (`TypeVar('T', bound=Animal)`), or marked as covariant or contravariant. PEP 695 introduced a new inline syntax for type parameters: `def func[T](x: T) -> T` and `class Box[T]`.

**Beginner-Friendly Explanation**: A type variable is like a blank in a sentence: "This function takes a value of type `T` and returns a value of type `T`." When you call the function with an `int`, `T` becomes `int`; when you call it with a `str`, `T` becomes `str`.

### Purposes

- To write functions and classes that work generically over many types.
- To preserve type information through transformations (e.g., identity function).
- To constrain type parameters to specific types or subtypes.
- To express variance relationships for generic types.
- To avoid code duplication across similar types.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Traditional `TypeVar`**
```python
from typing import TypeVar

T = TypeVar('T')                    # Unconstrained
AnyStr = TypeVar('AnyStr', str, bytes)  # Constrained
T_bound = TypeVar('T_bound', bound=Animal)  # Bounded
T_co = TypeVar('T_co', covariant=True)      # Covariant
T_contra = TypeVar('T_contra', contravariant=True)  # Contravariant
```

**Component Breakdown**:
- `TypeVar(` — factory function.
- `'T'` — the name of the type variable (must match the variable name).
- `str, bytes` — optional constraints.
- `bound=Animal` — optional upper bound.
- `covariant=True` / `contravariant=True` — optional variance.

**Syntax 2: PEP 695 Inline Syntax (Python 3.12+)**
```python
# Generic function
def first[T](items: list[T]) -> T:
    return items[0]

# Generic class
class Box[T]:
    def __init__(self, value: T) -> None:
        self.value = value

# Generic type alias
type ListOrSet[T] = list[T] | set[T]

# Constrained type parameter
def concat[T: (str, bytes)](x: T, y: T) -> T:
    return x + y

# Bounded type parameter
def process[T: Animal](animal: T) -> T:
    return animal
```

**Component Breakdown**:
- `[T]` — type parameter list.
- `T` — type parameter name.
- `: (str, bytes)` — constraint tuple.
- `: Animal` — upper bound.
- `type Alias[T] = ...` — generic type alias.

#### Syntax Rules

1. **TypeVar name must match variable name**: `T = TypeVar('T')` is correct; `T = TypeVar('U')` is invalid.
2. **TypeVar must be assigned directly**: `T = TypeVar('T')` is valid; `T = TypeVar('T') + 1` is not.
3. **At least two constraints**: If using constraints, at least two types must be specified.
4. **Covariant/contravariant only in generic classes**: Variance is only meaningful when the type variable is bound to a generic class.
5. **PEP 695 syntax requires Python 3.12+**: For earlier versions, use traditional `TypeVar`.
6. **Variance is inferred with PEP 695**: In the new syntax, type checkers infer variance automatically.

#### Constraints and Limitations

- Type variables cannot be reused across different generic contexts in the new syntax (each `[T]` is scoped).
- `TypeVar` with constraints cannot be parameterized by type variables.
- Variance is meaningless for standalone generic functions.
- `TypeVar` is not enforced at runtime.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Traditional TypeVar Generic Function

```python
from typing import TypeVar

# step1: Declare a type variable
T = TypeVar('T')

# step2: Generic identity function
def identity(value: T) -> T:
    return value

# step3: Type checker infers T from the argument
x: int = identity(42)        # T = int
y: str = identity("hello")   # T = str
print(x, y)                  # 42 hello

# step4: Generic function with multiple type variables
K = TypeVar('K')
V = TypeVar('V')

def swap(pair: tuple[K, V]) -> tuple[V, K]:
    a, b = pair
    return (b, a)

print(swap((1, "a")))   # ('a', 1)
```

**Expected Output**:
```
42 hello
('a', 1)
```

**Why**: `T` is inferred from the argument type. `identity(42)` returns `int`; `identity("hello")` returns `str`.

#### Example 2: PEP 695 Inline Generic Syntax (Python 3.12+)

```python
# step1: Generic function with inline type parameter
def first[T](items: list[T]) -> T:
    return items[0]

# step2: Type inference
print(first([1, 2, 3]))        # 1
print(first(["a", "b", "c"]))  # a

# step3: Generic class
class Stack[T]:
    def __init__(self) -> None:
        self.items: list[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()

# step4: Use the generic class
stack = Stack[int]()
stack.push(10)
stack.push(20)
print(stack.pop())  # 20
```

**Expected Output**:
```
1
a
20
```

**Why**: The inline `[T]` syntax declares a type parameter scoped to the function or class. `Stack[int]` instantiates a stack of integers.

#### Example 3: Constrained and Bounded TypeVars

```python
from typing import TypeVar

# step1: Constrained TypeVar
AnyStr = TypeVar('AnyStr', str, bytes)

def concat(x: AnyStr, y: AnyStr) -> AnyStr:
    return x + y

print(concat("a", "b"))      # ab
print(concat(b"a", b"b"))    # b'ab'
# concat("a", b"b")          # Type error: cannot mix str and bytes

# step2: Bounded TypeVar
class Animal:
    def speak(self) -> str:
        return "..."

class Dog(Animal):
    def speak(self) -> str:
        return "Woof"

T_Animal = TypeVar('T_Animal', bound=Animal)

def make_speak(animal: T_Animal) -> T_Animal:
    print(animal.speak())
    return animal

dog = make_speak(Dog())  # Woof
```

**Expected Output**:
```
ab
b'ab'
Woof
```

**Why**: `AnyStr` is constrained to `str` or `bytes`, so `concat` accepts either but not a mix. `T_Animal` is bounded by `Animal`, so any subtype of `Animal` is accepted, and the return type preserves the subtype.

### Real-World Cases

- **Container classes**: `List[T]`, `Dict[K, V]`.
- **Repository pattern**: `Repository[T]` with CRUD operations.
- **Functional programming**: `map`, `filter`, `reduce`.
- **Data validation**: Generic validators that preserve input type.

### References

- Generics - https://typing.python.org/en/latest/spec/generics.html
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/
- `typing.TypeVar` - https://docs.python.org/3/library/typing.html#typing.TypeVar
- `typing.Generic` - https://docs.python.org/3/library/typing.html#typing.Generic

---

## 6. Generic Classes and Generic Functions

### Definitions

**Core Definition**: A generic class or function is one that is parameterized by one or more type variables, allowing it to operate uniformly on values of different types.

**Technical Definition**: A generic class is defined by inheriting from `typing.Generic[T]` (or using the PEP 695 `class ClassName[T]:` syntax) and using type variables in its method signatures and attribute annotations. A generic function is a function that uses type variables in its parameter and return type annotations, with the type variables inferred at call time.

**Beginner-Friendly Explanation**: A generic class is like a template: "I can hold any type of item, and you tell me which type when you use me." A generic function is a function that works with any type while preserving that type information.

### Purposes

- To create reusable containers and data structures that work with any type.
- To preserve type information through transformations.
- To avoid code duplication across similar types.
- To enable type-safe abstractions like repositories, stacks, and queues.
- To support functional programming patterns.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Traditional `Generic` Base Class**
```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Box(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value
```

**Component Breakdown**:
- `Generic[T]` — base class parameterized by type variable `T`.
- `T` — type variable declared with `TypeVar`.
- `value: T` — attribute annotated with the type variable.

**Syntax 2: PEP 695 Inline Generic Class (Python 3.12+)**
```python
class Box[T]:
    def __init__(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value
```

**Component Breakdown**:
- `[T]` — type parameter list after class name.
- `T` — type parameter scoped to the class.

**Syntax 3: Generic Function**
```python
from typing import TypeVar

T = TypeVar('T')

def first(items: list[T]) -> T:
    return items[0]
```

**Component Breakdown**:
- `T` — type variable declared before the function.
- `items: list[T]` — parameter using the type variable.
- `-> T` — return type using the type variable.

#### Syntax Rules

1. **Type variables must be declared**: Before using `T` in a generic class/function, it must be created with `TypeVar('T')`.
2. **Generic base class must be parameterized**: `class Box(Generic[T])`, not `class Box(Generic)`.
3. **Type parameters are scoped**: In PEP 695 syntax, `[T]` is scoped to the class/function.
4. **Variance rules apply**: For generic classes, type variables can be covariant, contravariant, or invariant.
5. **Generic functions infer type parameters**: Type checkers infer `T` from the call arguments.

#### Constraints and Limitations

- Generic classes cannot be used without type parameters unless `Any` is implied.
- Type variables must be distinct within a generic class.
- PEP 695 syntax requires Python 3.12+.
- Variance must be explicitly declared in traditional syntax; inferred in PEP 695.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Generic Stack Class

```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: list[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()

    def is_empty(self) -> bool:
        return len(self.items) == 0

# step1: Create a stack of integers
s = Stack[int]()
s.push(1)
s.push(2)
print(s.pop())  # 2

# step2: Create a stack of strings
s2 = Stack[str]()
s2.push("a")
s2.push("b")
print(s2.pop())  # b
```

**Expected Output**:
```
2
b
```

**Why**: `Stack[int]` is a stack that only accepts integers; `Stack[str]` only accepts strings. The type checker enforces this.

#### Example 2: Generic Function with Multiple Type Variables

```python
from typing import TypeVar

K = TypeVar('K')
V = TypeVar('V')

def zip_dict(keys: list[K], values: list[V]) -> dict[K, V]:
    return dict(zip(keys, values))

print(zip_dict(["a", "b"], [1, 2]))  # {'a': 1, 'b': 2}
print(zip_dict([1, 2], ["x", "y"]))  # {1: 'x', 2: 'y'}
```

**Expected Output**:
```
{'a': 1, 'b': 2}
{1: 'x', 2: 'y'}
```

**Why**: `K` is inferred as `str` and `V` as `int` in the first call; `K` as `int` and `V` as `str` in the second.

#### Example 3: PEP 695 Generic Class with Bounded Type Parameter

```python
# Python 3.12+
class Animal:
    def speak(self) -> str:
        return "..."

class Dog(Animal):
    def speak(self) -> str:
        return "Woof"

class Cage[T: Animal]:
    def __init__(self, animal: T) -> None:
        self.animal = animal

    def get(self) -> T:
        return self.animal

cage = Cage(Dog())
print(cage.get().speak())  # Woof
```

**Expected Output**:
```
Woof
```

**Why**: `T: Animal` bounds the type parameter to `Animal` and its subclasses. `Cage(Dog())` infers `T = Dog`, so `get()` returns `Dog`.

### Real-World Cases

- **Collections**: `list[T]`, `dict[K, V]`, `set[T]`.
- **Repositories**: `Repository[Model]` with `get(id) -> Model`.
- **Result types**: `Result[T, E]` for success/failure.
- **Pipelines**: Generic transformers.

### References

- Generics - https://typing.python.org/en/latest/spec/generics.html
- `typing.Generic` - https://docs.python.org/3/library/typing.html#typing.Generic
- PEP 484 – Type Hints (Generics) - https://peps.python.org/pep-0484/#generics
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/

---

## 7. Protocols (Structural Subtyping via `typing.Protocol`)

### Definitions

**Core Definition**: A `Protocol` defines an interface based on the methods and attributes an object must have, without requiring explicit inheritance. This is structural subtyping, or "static duck typing."

**Technical Definition**: `typing.Protocol` is a base class for defining protocol classes. A class that defines the required methods and attributes of a protocol is considered a structural subtype of that protocol, even if it does not inherit from it. Protocols can be used as type annotations, and type checkers verify structural compatibility.

**Beginner-Friendly Explanation**: Instead of saying "this class must inherit from that class," protocols say "this class must have these methods." If it walks like a duck and quacks like a duck, it's a duck — even if it's not a `Duck` subclass.

### Purposes

- To define interfaces without requiring inheritance.
- To enable static duck typing, matching Python's dynamic nature.
- To decouple components and improve testability.
- To annotate third-party objects that cannot be modified.
- To define callback and handler interfaces.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Protocol

class SupportsClose(Protocol):
    def close(self) -> None:
        ...
```

**Component Breakdown**:
- `Protocol` — base class from `typing`.
- `def close(self) -> None:` — protocol member (method signature).
- `...` — empty body (protocol members have no implementation by default).

#### Syntax Rules

1. **Protocol members must have empty bodies**: Use `...` or `pass`.
2. **Subclassing a protocol does not make it a protocol**: A subclass must also inherit from `Protocol` to remain a protocol.
3. **Structural subtyping is automatic**: No explicit inheritance required for implementers.
4. **`@runtime_checkable`**: Allows `isinstance()` checks against protocols at runtime (methods only).
5. **Protocols can be generic**: `class SupportsClose[T](Protocol): ...`.

#### Constraints and Limitations

- Runtime `isinstance()` checks with `@runtime_checkable` only check method presence, not signatures.
- Protocols cannot be instantiated.
- Protocol members cannot have default implementations unless using `@abstractmethod`.
- Some type checkers may have incomplete protocol support.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Simple Protocol

```python
from typing import Protocol

# step1: Define a protocol
class SupportsClose(Protocol):
    def close(self) -> None:
        ...

# step2: A class that implements the protocol (no inheritance)
class Resource:
    def close(self) -> None:
        print("Resource closed")

# step3: A function that accepts the protocol
def close_all(items: list[SupportsClose]) -> None:
    for item in items:
        item.close()

# step4: Use the function
close_all([Resource(), Resource()])
```

**Expected Output**:
```
Resource closed
Resource closed
```

**Why**: `Resource` has a `close()` method, so it satisfies the `SupportsClose` protocol structurally, even without inheriting from it.

#### Example 2: Runtime Checkable Protocol

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None:
        ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

print(isinstance(Circle(), Drawable))  # True
print(isinstance(Square(), Drawable))  # True
print(isinstance("not drawable", Drawable))  # False
```

**Expected Output**:
```
True
True
False
```

**Why**: `@runtime_checkable` allows `isinstance` to check for the presence of `draw()` at runtime.

#### Example 3: Generic Protocol

```python
from typing import Protocol, TypeVar

T = TypeVar('T')

class Container(Protocol[T]):
    def get(self) -> T:
        ...

class Box:
    def __init__(self, value: int) -> None:
        self.value = value

    def get(self) -> int:
        return self.value

def extract(container: Container[int]) -> int:
    return container.get()

print(extract(Box(42)))  # 42
```

**Expected Output**:
```
42
```

**Why**: `Box` has a `get()` method returning `int`, so it satisfies `Container[int]` structurally.

### Real-World Cases

- **File-like objects**: `SupportsRead`, `SupportsWrite`.
- **Database connections**: `SupportsExecute`.
- **Web frameworks**: Request/response protocols.
- **Testing**: Mock objects that satisfy protocols without inheritance.

### References

- Protocols and Structural Subtyping - https://typing.python.org/en/latest/reference/protocols.html
- PEP 544 – Protocols: Structural subtyping - https://peps.python.org/pep-0544/
- `typing.Protocol` - https://docs.python.org/3/library/typing.html#typing.Protocol

---

## 8. Type Aliases (and the `TypeAlias` / `type` Keywords)

### Definitions

**Core Definition**: A type alias is a name that refers to another type, simplifying complex type expressions and improving readability.

**Technical Definition**: Type aliases are created by simple assignment (`Url = str`), by explicit annotation with `typing.TypeAlias` (`Url: TypeAlias = str`), or by the `type` statement (Python 3.12+): `type Url = str`. The `type` statement creates a `TypeAliasType` instance and supports generic type aliases.

**Beginner-Friendly Explanation**: A type alias is a nickname for a type. Instead of writing `dict[str, list[int]]` everywhere, you can write `IntListMap = dict[str, list[int]]` and use `IntListMap` in annotations.

### Purposes

- To simplify complex type expressions.
- To improve code readability and maintainability.
- To create reusable type definitions.
- To document the meaning of complex types.
- To support generic type aliases with type parameters.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Implicit Type Alias**
```python
Url = str

def retry(url: Url, retry_count: int) -> None:
    ...
```

**Component Breakdown**:
- `Url` — alias name (conventionally capitalized).
- `= str` — the aliased type.

**Syntax 2: `TypeAlias` Explicit**
```python
from typing import TypeAlias

Url: TypeAlias = str
```

**Component Breakdown**:
- `Url: TypeAlias = str` — explicitly declares `Url` as a type alias, not a variable.

**Syntax 3: `type` Statement (Python 3.12+)**
```python
type Url = str

type ListOrSet[T] = list[T] | set[T]
```

**Component Breakdown**:
- `type Url = str` — creates a `TypeAliasType`.
- `type ListOrSet[T] = ...` — generic type alias.

#### Syntax Rules

1. **Capitalization convention**: Alias names should be capitalized (`Url`, `Vector`).
2. **Any type hint allowed**: Type aliases can be as complex as any type hint.
3. **`TypeAlias` requires Python 3.10+**: For earlier versions, use `typing_extensions`.
4. **`type` statement requires Python 3.12+**: For earlier versions, use implicit aliases or `TypeAlias`.
5. **Generic aliases**: The `type` statement supports type parameters (`type ListOrSet[T] = ...`).

#### Constraints and Limitations

- Implicit aliases may be confused with variable assignments.
- `TypeAlias` is deprecated in favor of the `type` statement.
- Type aliases are not enforced at runtime.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Implicit Type Alias

```python
from typing import Union

# step1: Define a complex type alias
JsonValue = Union[str, int, float, bool, None, dict[str, "JsonValue"], list["JsonValue"]]

# step2: Use the alias
def parse_json(data: JsonValue) -> JsonValue:
    return data

print(parse_json({"key": [1, 2, 3]}))  # {'key': [1, 2, 3]}
```

**Expected Output**:
```
{'key': [1, 2, 3]}
```

**Why**: `JsonValue` is a recursive type alias that simplifies a complex union.

#### Example 2: `TypeAlias` Explicit

```python
from typing import TypeAlias

# step1: Explicit type alias
UserId: TypeAlias = int
UserName: TypeAlias = str

# step2: Use aliases
def get_user(user_id: UserId) -> UserName:
    return f"User{user_id}"

print(get_user(42))  # User42
```

**Expected Output**:
```
User42
```

**Why**: `TypeAlias` makes it clear that `UserId` and `UserName` are type aliases, not variables.

#### Example 3: `type` Statement with Generics (Python 3.12+)

```python
# Python 3.12+
type Vector[T] = list[T]

def scale[T: (int, float)](v: Vector[T], factor: T) -> Vector[T]:
    return [x * factor for x in v]

print(scale([1, 2, 3], 2))      # [2, 4, 6]
print(scale([1.0, 2.0], 1.5))   # [1.5, 3.0]
```

**Expected Output**:
```
[2, 4, 6]
[1.5, 3.0]
```

**Why**: The `type` statement creates a generic type alias `Vector[T]` that can be used with type parameters.

### Real-World Cases

- **JSON types**: `JsonValue` recursive alias.
- **Database rows**: `Row = dict[str, Any]`.
- **API responses**: `ApiResponse[T] = dict[str, T]`.
- **Configuration**: `Config = dict[str, Union[str, int, bool]]`.

### References

- Type Aliases - https://typing.python.org/en/latest/spec/aliases.html
- PEP 613 – Explicit Type Aliases - https://peps.python.org/pep-0613/
- PEP 695 – Type Parameter Syntax (type statement) - https://peps.python.org/pep-0695/
- `typing.TypeAlias` - https://docs.python.org/3/library/typing.html#typing.TypeAlias

---

## 9. Typed Dictionaries (`TypedDict`)

### Definitions

**Core Definition**: `TypedDict` is a way to annotate dictionaries with a fixed set of string keys and specific value types for each key.

**Technical Definition**: `typing.TypedDict` creates a dictionary type where each key is a string literal and each value has a declared type. By default, all keys are required, but `NotRequired` and `Required` (PEP 655) allow marking keys as optional or required. TypedDicts are structural types and can be generic.

**Beginner-Friendly Explanation**: A `TypedDict` is like a struct or record: it says "this dictionary must have a `'name'` key with a string value and an `'age'` key with an integer value."

### Purposes

- To annotate dictionaries with known keys and value types.
- To represent JSON objects and API payloads.
- To improve type safety when working with dictionary-based data.
- To document the expected structure of configuration dictionaries.
- To support structural subtyping for dictionaries.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import TypedDict

class Movie(TypedDict):
    name: str
    year: int
```

**Component Breakdown**:
- `class Movie(TypedDict):` — class-based syntax.
- `name: str` — key `'name'` with value type `str`.
- `year: int` — key `'year'` with value type `int`.

**Alternative Functional Syntax**:
```python
Movie = TypedDict('Movie', {'name': str, 'year': int})
```

#### Syntax Rules

1. **Keys must be strings**: TypedDict keys are always strings.
2. **All keys required by default**: Use `NotRequired` or `total=False` for optional keys.
3. **No extra keys by default**: TypedDicts are "open" by default (extra keys allowed at runtime but type checkers may warn).
4. **Structural subtyping**: Two TypedDicts with the same structure are equivalent.
5. **Generic TypedDicts**: Can be parameterized with type variables.
6. **`Required` and `NotRequired`** (PEP 655): Allow per-key control.

#### Constraints and Limitations

- Values must be instances of `dict`, not subclasses.
- Runtime enforcement is limited; type checkers only.
- Extra keys are not allowed in literal construction with closed TypedDicts.
- Inheritance is supported but must be consistent.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic TypedDict

```python
from typing import TypedDict

# step1: Define a TypedDict
class Movie(TypedDict):
    name: str
    year: int

# step2: Create a dictionary
movie: Movie = {"name": "Blade Runner", "year": 1982}

# step3: Access values
print(movie["name"])  # Blade Runner
print(movie["year"])  # 1982

# step4: Invalid key (type checker error)
# movie["director"] = "Ridley Scott"
```

**Expected Output**:
```
Blade Runner
1982
```

**Why**: `Movie` requires exactly `'name'` (str) and `'year'` (int). Extra keys are not allowed.

#### Example 2: Optional Keys with `NotRequired`

```python
from typing import TypedDict, NotRequired

class Movie(TypedDict):
    name: str
    year: int
    rating: NotRequired[float]

# step1: Valid with and without optional key
m1: Movie = {"name": "Alien", "year": 1979}
m2: Movie = {"name": "Aliens", "year": 1986, "rating": 8.5}

print(m1)  # {'name': 'Alien', 'year': 1979}
print(m2)  # {'name': 'Aliens', 'year': 1986, 'rating': 8.5}
```

**Expected Output**:
```
{'name': 'Alien', 'year': 1979}
{'name': 'Aliens', 'year': 1986, 'rating': 8.5}
```

**Why**: `rating` is `NotRequired`, so it can be omitted.

#### Example 3: Generic TypedDict

```python
from typing import TypedDict, TypeVar

T = TypeVar('T')

class ApiResponse(TypedDict, Generic[T]):
    data: T
    status: int

# step1: Response with string data
resp: ApiResponse[str] = {"data": "success", "status": 200}
print(resp)  # {'data': 'success', 'status': 200}

# step2: Response with list data
resp2: ApiResponse[list[int]] = {"data": [1, 2, 3], "status": 200}
print(resp2)  # {'data': [1, 2, 3], 'status': 200}
```

**Expected Output**:
```
{'data': 'success', 'status': 200}
{'data': [1, 2, 3], 'status': 200}
```

**Why**: `ApiResponse[T]` is generic over the `data` value type.

### Real-World Cases

- **API responses**: `{"data": ..., "status": ...}`.
- **Configuration**: `{"host": str, "port": int}`.
- **JSON objects**: Any JSON object with known schema.
- **Database rows**: `{"id": int, "name": str}`.

### References

- Typed Dictionaries - https://typing.python.org/en/latest/spec/typeddict.html
- PEP 589 – TypedDict - https://peps.python.org/pep-0589/
- PEP 655 – Required and NotRequired - https://peps.python.org/pep-0655/
- `typing.TypedDict` - https://docs.python.org/3/library/typing.html#typing.TypedDict

---

## 10. `Final` and `ClassVar`

### Definitions

**Core Definition**: `Final` indicates that a variable or attribute cannot be reassigned or overridden; `ClassVar` indicates that an attribute is a class variable, not an instance variable.

**Technical Definition**:
- `typing.Final` is a qualifier that marks a name as final: it cannot be reassigned, and a method cannot be overridden. It can be used in variable annotations, class attributes, and method signatures.
- `typing.ClassVar` is a special type construct that marks an attribute as a class variable, meaning it should not be set on instances.

**Beginner-Friendly Explanation**: `Final` says "this value is constant — don't change it." `ClassVar` says "this attribute belongs to the class, not to individual objects."

### Purposes

- To declare constants that should not be reassigned (`Final`).
- To prevent method overriding in subclasses (`Final`).
- To indicate that an attribute is shared across all instances (`ClassVar`).
- To improve type checking of class-level vs. instance-level attributes.
- To document intent in dataclasses and attrs classes.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Final, ClassVar

# Final variable
MAX_SIZE: Final = 100

# Final class attribute
class Config:
    VERSION: Final = "1.0"

# Final method
class Base:
    def method(self) -> None:
        ...

class Derived(Base):
    # def method(self) -> None: ...  # Type error: cannot override final method
    pass

# ClassVar
class Counter:
    count: ClassVar[int] = 0

    def __init__(self) -> None:
        self.count += 1  # Type error: cannot assign to ClassVar via instance
```

**Component Breakdown**:
- `Final` — qualifier from `typing`.
- `MAX_SIZE: Final = 100` — final variable annotation.
- `ClassVar[int]` — class variable type.
- `count: ClassVar[int] = 0` — class variable annotation.

#### Syntax Rules

1. **`Final` must be used in annotations**: `x: Final = 1` is valid; `Final x = 1` is not.
2. **`Final` cannot be combined with `ClassVar`**: Type checkers should infer a final class attribute as a class variable.
3. **`ClassVar` can only be used in class bodies**: Cannot be used in instance method annotations.
4. **`Final` methods cannot be overridden**: Type checkers enforce this.
5. **`Final` attributes cannot be reassigned**: Type checkers enforce this.

#### Constraints and Limitations

- `Final` and `ClassVar` are static-only; not enforced at runtime.
- `Final` cannot be combined with `ClassVar` in the same annotation.
- Dataclasses have special behavior with `Final` and `ClassVar`.
- Some type checkers may have incomplete support for `Final` methods.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: `Final` Variable and Method

```python
from typing import Final

# step1: Final variable
PI: Final = 3.14159
# PI = 3.14  # Type error: Cannot assign to final name

# step2: Final method
class Base:
    def method(self) -> str:
        return "base"

class Derived(Base):
    # def method(self) -> str:  # Type error: Cannot override final method
    #     return "derived"
    pass

print(PI)  # 3.14159
print(Derived().method())  # base
```

**Expected Output**:
```
3.14159
base
```

**Why**: `PI` cannot be reassigned; `method` cannot be overridden.

#### Example 2: `ClassVar` in a Class

```python
from typing import ClassVar

class Counter:
    count: ClassVar[int] = 0

    def __init__(self) -> None:
        # self.count += 1  # Type error: Cannot assign to ClassVar via instance
        Counter.count += 1

c1 = Counter()
c2 = Counter()
print(Counter.count)  # 2
```

**Expected Output**:
```
2
```

**Why**: `count` is a class variable shared across all instances. `Counter.count += 1` updates the class variable.

#### Example 3: `Final` in Dataclass

```python
from dataclasses import dataclass
from typing import Final

@dataclass
class Config:
    name: str
    version: Final[str] = "1.0"

config = Config("app")
print(config)  # Config(name='app', version='1.0')
# config.version = "2.0"  # Type error: Cannot assign to final attribute
```

**Expected Output**:
```
Config(name='app', version='1.0')
```

**Why**: `version` is a final instance attribute with a default value; it cannot be reassigned after initialization.

### Real-World Cases

- **Configuration constants**: `MAX_CONNECTIONS: Final = 100`.
- **Class-level counters**: `ClassVar[int]`.
- **Immutable dataclasses**: `Final` fields.
- **API versioning**: `API_VERSION: Final = "v2"`.

### References

- `typing.Final` - https://docs.python.org/3/library/typing.html#typing.Final
- `typing.ClassVar` - https://docs.python.org/3/library/typing.html#typing.ClassVar
- PEP 591 – Adding a final qualifier to typing - https://peps.python.org/pep-0591/
- PEP 526 – Syntax for Variable Annotations - https://peps.python.org/pep-0526/

---

## 11. `Self` Type (For Fluid Interfaces and Method Chaining)

### Definitions

**Core Definition**: `Self` is a special type that refers to the type of the enclosing class, enabling methods to return the correct subtype when called on subclasses.

**Technical Definition**: `typing.Self` (Python 3.11+) is a special form that stands for a type variable bound to the encapsulating class. It is used in method return annotations to indicate that the method returns an instance of the same class (or subclass) it was called on. It replaces the more verbose `TypeVar` bound approach.

**Beginner-Friendly Explanation**: `Self` means "this method returns an object of the same type as the one it was called on." If you call a method on a `Dog`, it returns a `Dog`, not just an `Animal`.

### Purposes

- To annotate methods that return `self` or a new instance of the same class.
- To enable fluent interfaces and method chaining.
- To preserve the subtype in method return types.
- To replace the verbose `TypeVar` bound workaround.
- To improve type safety in builder patterns.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Self

class Shape:
    def set_scale(self, scale: float) -> Self:
        self.scale = scale
        return self
```

**Component Breakdown**:
- `Self` — special form from `typing`.
- `-> Self` — return type annotation indicating the enclosing class type.

#### Syntax Rules

1. **`Self` requires Python 3.11+**: For earlier versions, use `typing_extensions.Self`.
2. **`Self` can be used in return types and parameter types**: Typically in return types.
3. **`Self` is bound to the encapsulating class**: It cannot be used outside a class definition.
4. **`Self` preserves subtyping**: `Circle().set_scale()` returns `Circle`, not `Shape`.

#### Constraints and Limitations

- `Self` cannot be used in standalone functions.
- `Self` in class methods (`@classmethod`) refers to the class itself.
- Some type checkers may have incomplete support for `Self` in complex inheritance.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Method Chaining with `Self`

```python
from typing import Self

class Shape:
    def __init__(self) -> None:
        self.scale = 1.0

    def set_scale(self, scale: float) -> Self:
        self.scale = scale
        return self

class Circle(Shape):
    def set_radius(self, radius: float) -> Self:
        self.radius = radius
        return self

# step1: Method chaining on subclass
circle = Circle().set_scale(2.0).set_radius(5.0)
print(type(circle).__name__)  # Circle
print(circle.scale)            # 2.0
print(circle.radius)           # 5.0
```

**Expected Output**:
```
Circle
2.0
5.0
```

**Why**: `set_scale` returns `Self`, which is `Circle` when called on a `Circle` instance. This enables chaining without losing type information.

#### Example 2: `Self` in Class Methods

```python
from typing import Self

class Builder:
    def __init__(self) -> None:
        self.parts: list[str] = []

    @classmethod
    def create(cls) -> Self:
        return cls()

    def add(self, part: str) -> Self:
        self.parts.append(part)
        return self

builder = Builder.create().add("a").add("b")
print(builder.parts)  # ['a', 'b']
```

**Expected Output**:
```
['a', 'b']
```

**Why**: `create` returns `Self`, which is `Builder` (or a subclass). `add` returns `Self` for chaining.

#### Example 3: `Self` with Inheritance

```python
from typing import Self

class QueryBuilder:
    def where(self, condition: str) -> Self:
        self.condition = condition
        return self

class SQLQueryBuilder(QueryBuilder):
    def select(self, columns: str) -> Self:
        self.columns = columns
        return self

query = SQLQueryBuilder().where("id = 1").select("*")
print(type(query).__name__)  # SQLQueryBuilder
print(query.condition)        # id = 1
print(query.columns)          # *
```

**Expected Output**:
```
SQLQueryBuilder
id = 1
*
```

**Why**: `Self` ensures that `where` returns `SQLQueryBuilder` when called on a `SQLQueryBuilder`, enabling further chaining.

### Real-World Cases

- **Query builders**: SQL, GraphQL.
- **Fluent APIs**: Configuration builders.
- **Data pipelines**: Transformation chains.
- **UI builders**: Widget configuration.

### References

- PEP 673 – Self Type - https://peps.python.org/pep-0673/
- `typing.Self` - https://docs.python.org/3/library/typing.html#typing.Self

---

## 12. `Annotated` (Attaching Metadata to Types)

### Definitions

**Core Definition**: `Annotated` allows attaching arbitrary metadata to a type without changing the type itself.

**Technical Definition**: `typing.Annotated[T, x, y, ...]` creates a type that is equivalent to `T` but carries additional metadata `x, y, ...`. Type checkers ignore the metadata for type compatibility; it can be inspected at runtime via `__metadata__` and `get_args()`.

**Beginner-Friendly Explanation**: `Annotated` lets you say "this is a `str`, and here's some extra information about it" — like "this string must be at least 3 characters" or "this field should be validated as an email."

### Purposes

- To attach validation constraints to types (e.g., `Annotated[int, Gt(0)]`).
- To provide documentation or unit information (`Annotated[float, "meters"]`).
- To enable framework-specific metadata (Pydantic, FastAPI, SQLAlchemy).
- To distinguish semantically different uses of the same type.
- To support runtime introspection of type metadata.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Annotated

# Single metadata
x: Annotated[int, "positive"]

# Multiple metadata
y: Annotated[str, "email", MaxLen(100)]

# Nested Annotated (flattened)
z: Annotated[Annotated[int, "a"], "b"]  # Equivalent to Annotated[int, "a", "b"]
```

**Component Breakdown**:
- `Annotated[` — special form from `typing`.
- `T` — the base type.
- `metadata` — arbitrary objects (strings, instances, etc.).
- `]` — closes `Annotated`.

#### Syntax Rules

1. **First argument must be a valid type**: `Annotated[int, ...]` is valid; `Annotated[42, ...]` is not.
2. **Metadata is arbitrary**: Any object can be metadata.
3. **`Annotated` is not a type**: `isinstance(Annotated[int, "x"], type)` is `False`.
4. **Nested `Annotated` flattens**: `Annotated[Annotated[T, a], b]` ≡ `Annotated[T, a, b]`.
5. **Aliases block flattening**: PEP 695 type aliases are lazy, so nesting may not flatten.
6. **Runtime access**: `__metadata__` and `get_args()` retrieve metadata.

#### Constraints and Limitations

- `Annotated` cannot be used as a type in `type[T]` (requires `TypeForm`, PEP 747).
- Metadata is not enforced by type checkers; only by tools that inspect it.
- Some type checkers may not fully support nested `Annotated` with aliases.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic `Annotated` with Metadata

```python
from typing import Annotated, get_args, get_origin

# step1: Define Annotated type
PositiveInt = Annotated[int, "positive"]

# step2: Use in function
def process(value: PositiveInt) -> PositiveInt:
    return value

print(process(5))  # 5

# step3: Inspect metadata at runtime
print(PositiveInt.__metadata__)  # ('positive',)
print(get_origin(PositiveInt))   # <class 'int'>? Actually get_origin(Annotated) returns Annotated
```

**Expected Output**:
```
5
('positive',)
typing.Annotated
```

**Why**: `PositiveInt` is equivalent to `int` for type checking, but carries `'positive'` as metadata.

#### Example 2: `Annotated` with Validation (Pydantic-style)

```python
from typing import Annotated
from pydantic import BaseModel, Field

# step1: Define a model with Annotated constraints
class User(BaseModel):
    name: Annotated[str, Field(min_length=1, max_length=50)]
    age: Annotated[int, Field(ge=0, le=150)]

# step2: Validate data
user = User(name="Alice", age=30)
print(user)  # name='Alice' age=30

# step3: Invalid data raises validation error
# User(name="", age=200)  # ValidationError
```

**Expected Output**:
```
name='Alice' age=30
```

**Why**: Pydantic reads the `Annotated` metadata to enforce validation rules at runtime.

#### Example 3: `Annotated` with Units

```python
from typing import Annotated

# step1: Semantic type aliases
Meters = Annotated[float, "meters"]
Seconds = Annotated[float, "seconds"]

def speed(distance: Meters, time: Seconds) -> float:
    return distance / time

print(speed(100.0, 10.0))  # 10.0

# step2: Metadata inspection
print(Meters.__metadata__)  # ('meters',)
```

**Expected Output**:
```
10.0
('meters',)
```

**Why**: `Annotated` distinguishes `Meters` from `Seconds` semantically, even though both are `float`.

### Real-World Cases

- **Pydantic/FastAPI**: Validation constraints.
- **SQLAlchemy**: Column type metadata.
- **Scientific computing**: Units of measurement.
- **Documentation**: Semantic type aliases.

### References

- `typing.Annotated` - https://docs.python.org/3/library/typing.html#typing.Annotated
- PEP 593 – Flexible function and variable annotations - https://peps.python.org/pep-0593/
- Annotated Types - https://typing.python.org/en/latest/spec/annotated.html

---

## References

- typing — Support for type hints - https://docs.python.org/3/library/typing.html
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- PEP 483 – The Theory of Type Hints - https://peps.python.org/pep-0483/
- PEP 585 – Type Hinting Generics In Standard Collections - https://peps.python.org/pep-0585/
- PEP 604 – Allow writing union types as X | Y - https://peps.python.org/pep-0604/
- PEP 586 – Literal Types - https://peps.python.org/pep-0586/
- PEP 589 – TypedDict - https://peps.python.org/pep-0589/
- PEP 591 – Adding a final qualifier to typing - https://peps.python.org/pep-0591/
- PEP 593 – Flexible function and variable annotations - https://peps.python.org/pep-0593/
- PEP 544 – Protocols: Structural subtyping - https://peps.python.org/pep-0544/
- PEP 673 – Self Type - https://peps.python.org/pep-0673/
- PEP 613 – Explicit Type Aliases - https://peps.python.org/pep-0613/
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/
- PEP 655 – Required and NotRequired - https://peps.python.org/pep-0655/
- PEP 612 – Parameter Specification Variables - https://peps.python.org/pep-0612/
- typing — Type System Reference - https://typing.readthedocs.io/en/latest/
- mypy Type Hints Cheat Sheet - https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html
- Python Type System Specification - https://typing.python.org/en/latest/spec/
- typing_extensions (backports) - https://pypi.org/project/typing-extensions/