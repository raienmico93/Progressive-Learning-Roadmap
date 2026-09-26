# Type Hint Fundamentals in Python: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Type hint fundamentals are the foundational syntax and semantics for attaching type information to variables, function parameters, and function return values in Python. They constitute the entry point to Python's gradual type system, enabling static analysis tools to verify program correctness without affecting runtime behavior.

### Technical Definition

Type hints are expressions placed in designated annotation slots within Python syntax — variable assignments, function parameter lists, and function return positions — that are evaluated at compile time (or deferred, in modern Python versions) and stored in `__annotations__` dictionaries. They are specified by PEP 3107 (function annotations, Python 3.0), PEP 484 (type hints, Python 3.5), and PEP 526 (variable annotations, Python 3.6), and are consumed by static type checkers, IDEs, and linters. The Python runtime does not enforce type annotations.

### Beginner-Friendly Explanation

Type hints are a way to tell Python (and other programmers) what kind of data you expect in your variables and functions. When you write `name: str = "Alice"`, you're saying "this variable should be a string." When you write `def greet(name: str) -> str:`, you're saying "this function takes a string and returns a string." Python itself doesn't enforce these hints — they're for tools like type checkers and your code editor, which use them to catch mistakes before you run the code.

### Key Characteristics

- **Optional and non-invasive**: Type hints are completely optional; unannotated code is treated as dynamically typed.
- **Runtime-accessible**: Annotations are stored in `__annotations__` dictionaries on functions, classes, and modules.
- **Gradually adoptable**: You can add type hints to one variable or function at a time.
- **Tool-consumed**: Type checkers (mypy, Pyright) and IDEs (Pylance, PyCharm) read annotations; Python runtime does not enforce them.
- **Version-dependent**: Syntax and semantics vary by Python version (e.g., built-in generics require 3.9+, `from __future__ import annotations` requires 3.7+).

### Prerequisites

- Python 3.6 or newer (variable annotations require 3.6+; function annotations require 3.0+).
- A code editor with type-checking integration (optional but recommended).
- Basic familiarity with Python functions, variables, and classes.

### Related Programming Areas

- **Static type checking**: mypy, Pyright, Pylance.
- **Data validation**: Pydantic, attrs.
- **API frameworks**: FastAPI, Django REST Framework.
- **Documentation**: Sphinx, MkDocs.

### Core Concepts / Features

The following sections cover each fundamental type hint feature using a uniform structure.

---

## 1. Variable Annotations

### Definitions

**Core Definition**: Variable annotations are a syntax for declaring the expected type of a variable (module-level, class-level, or local) directly in the source code, using a colon followed by a type expression.

**Technical Definition**: Introduced by PEP 526 (Python 3.6), variable annotations use the syntax `target: expression [= value]` for annotated assignments, and `target: expression` for bare annotations without assignment. Annotations are stored in the `__annotations__` dictionary of the enclosing module or class; local variable annotations are not evaluated or stored. The assignment statement is optional — a variable can be annotated without being assigned a value.

**Beginner-Friendly Explanation**: Instead of writing a comment like `# type: int`, you can write `count: int = 0` to say "this variable should be an integer." You can also annotate without assigning: `count: int` just says "I'll assign an integer later."

### Purposes

- To replace type comments (`# type: int`) with first-class syntax.
- To enable static type checking of module-level and class-level variables.
- To allow annotation of variables without initial values.
- To support class variable and instance variable annotations in class bodies.
- To provide runtime-accessible type metadata via `__annotations__`.

### Syntax Rules and Structure

#### Complete General Syntax

```python
# Annotated assignment (with initializer)
name: str = "Alice"

# Bare annotation (no initializer)
count: int

# Multiple annotations in one line (not recommended)
x: int; y: str

# Class variable annotation
class Starship:
    captain: str = "Picard"
    damage: int
    stats: ClassVar[dict[str, int]] = {}

# Local variable annotation
def process() -> None:
    local: int = 42  # Evaluated and stored in local scope
```

**Component Breakdown**:
- `target` — the variable name being annotated (any valid single assignment target).
- `:` — the annotation separator.
- `expression` — any valid Python expression (typically a type).
- `= value` — optional initializer expression.

The syntax rule for annotated assignments is: `annotated_assignment_stmt ::= augtarget ":" expression ["=" expression]`.

#### Syntax Rules

1. **Assignment is optional**: `x: int` is a valid bare annotation; the variable need not be initialized.
2. **Target must be a single assignment target**: `x: int` is valid; `x, y: int` is invalid.
3. **Annotations are evaluated at definition time** (unless deferred): `x: int = 0` evaluates `int` and stores it in `__annotations__`.
4. **Local annotations are not evaluated**: As per PEP 526, local variable annotations are not evaluated at all since they are not accessible outside of the function's closure.
5. **`ClassVar` marks class variables**: Use `ClassVar[<type>]` to indicate an attribute should not be set on instances.
6. **`Final` marks constants**: Use `Final` to indicate a variable cannot be reassigned.
7. **`__annotations__` storage**: Module-level and class-level annotations are stored in the respective `__annotations__` dictionary.

#### Constraints and Limitations

- Local variable annotations are not evaluated and cannot be retrieved at runtime.
- Annotations do not prevent reassignment to a different type at runtime.
- Type comments (`# type: int`) are deprecated but still supported for backward compatibility.
- `ClassVar` cannot include type variables.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Variable Annotations

```python
# step1: Annotated assignment
name: str = "Alice"
age: int = 30

# step2: Bare annotation (no initializer)
score: float

# step3: Assign later
score = 95.5

# step4: Access annotations at runtime
print(__annotations__)  # {'name': <class 'str'>, 'age': <class 'int'>, 'score': <class 'float'>}

print(name)   # Alice
print(age)    # 30
print(score)  # 95.5
```

**Expected Output**:
```
{'name': <class 'str'>, 'age': <class 'int'>, 'score': <class 'float'>}
Alice
30
95.5
```

**Why**: Module-level annotations are stored in `__annotations__` and can be inspected at runtime. The bare annotation for `score` is evaluated and stored even without an initializer.

#### Example 2: Class and Instance Variable Annotations

```python
from typing import ClassVar

class Starship:
    # step1: Instance variable with default
    captain: str = "Picard"

    # step2: Instance variable without default
    damage: int

    # step3: Class variable (shared across instances)
    stats: ClassVar[dict[str, int]] = {}

    def __init__(self, damage: int) -> None:
        self.damage = damage

# step4: Inspect class annotations
print(Starship.__annotations__)
# {'captain': <class 'str'>, 'damage': <class 'int'>, 'stats': typing.ClassVar[dict[str, int]]}

ship = Starship(50)
print(ship.captain)  # Picard
print(ship.damage)   # 50
print(Starship.stats)  # {}
```

**Expected Output**:
```
{'captain': <class 'str'>, 'damage': <class 'int'>, 'stats': typing.ClassVar[dict[str, int]]}
Picard
50
{}
```

**Why**: `captain` and `damage` are instance variables; `stats` is a class variable marked with `ClassVar`. The type checker distinguishes between them.

#### Example 3: Local Variable Annotations (Not Evaluated)

```python
def process() -> None:
    # step1: Local annotation with initializer
    result: int = 42

    # step2: Local annotation without initializer
    pending: str

    # step3: Local annotations are NOT stored in __annotations__
    print(result)  # 42

    # step4: Assign pending later
    pending = "done"
    print(pending)  # done

process()

# step5: Module __annotations__ does NOT contain local annotations
print(__annotations__)  # {}
```

**Expected Output**:
```
42
done
{}
```

**Why**: Local variable annotations are evaluated syntactically but not stored at runtime. The module-level `__annotations__` remains empty because no module-level variables were annotated.

### Real-World Cases

- **Configuration modules**: `DEBUG: bool = False`, `DATABASE_URL: str`.
- **Class attributes**: `timeout: int = 30`, `retries: ClassVar[int] = 3`.
- **Dataclasses**: `@dataclass` uses `__annotations__` to generate `__init__`.
- **Module constants**: `MAX_SIZE: Final[int] = 100`.

### References

- PEP 526 – Syntax for Variable Annotations - https://peps.python.org/pep-0526/
- Variable Annotations - https://docs.python.org/3/tutorial/controlflow.html#variable-annotations
- Class Type Assignability (ClassVar) - https://typing.python.org/en/latest/spec/class-compat.html
- `typing.ClassVar` - https://docs.python.org/3/library/typing.html#typing.ClassVar

---

## 2. Function Annotations and Parameter Hints

### Definitions

**Core Definition**: Function annotations are expressions attached to function parameters (and return values) using a colon after the parameter name, providing metadata that is typically used for type hints.

**Technical Definition**: Introduced by PEP 3107 (Python 3.0), function annotations associate arbitrary expressions with function parameters via the syntax `identifier [: expression] [= expression]`. Annotations are evaluated at function definition time and stored in the function's `__annotations__` dictionary, keyed by parameter name. PEP 484 later gave these annotations a standard type-hinting semantics.

**Beginner-Friendly Explanation**: When you write `def greet(name: str) -> str:`, the `: str` after `name` is a parameter annotation. It tells anyone reading the code (and type checkers) that `name` should be a string. Annotations can go on regular parameters, `*args`, and `**kwargs`.

### Purposes

- To associate type information with function parameters.
- To provide machine-readable documentation of parameter expectations.
- To enable static type checkers to verify argument types at call sites.
- To support arbitrary metadata (not just types) for third-party libraries.
- To improve IDE autocompletion and error detection.

### Syntax Rules and Structure

#### Complete General Syntax

```python
def function(
    positional: int,
    with_default: str = "default",
    *args: int,
    keyword_only: float = 0.0,
    **kwargs: bool,
) -> None:
    ...
```

**Component Breakdown**:
- `positional: int` — annotation for a positional parameter.
- `with_default: str = "default"` — annotation precedes the default value.
- `*args: int` — annotation for excess positional arguments (each is an `int`).
- `keyword_only: float` — annotation for keyword-only parameters.
- `**kwargs: bool` — annotation for excess keyword arguments (each value is a `bool`).

Annotations for parameters take the form of optional expressions that follow the parameter name. Annotations always precede a parameter's default value; both annotations and default values are optional.

#### Syntax Rules

1. **Annotations are optional**: Functions can be fully annotated, partially annotated, or unannotated.
2. **Colon syntax**: `param: expression` attaches an annotation to `param`.
3. **Default values**: `param: expression = value` — annotation comes before the default.
4. **`*args` annotation**: The annotation applies to each element of `args`, not to the tuple itself.
5. **`**kwargs` annotation**: The annotation applies to each value in `kwargs`, not to the dict itself.
6. **Annotations are evaluated at definition time** (unless deferred via `from __future__ import annotations`).
7. **`__annotations__` storage**: Access via `function.__annotations__`, keyed by parameter name.
8. **Return annotation is stored under `'return'` key**: `function.__annotations__['return']`.

#### Constraints and Limitations

- Annotations are expressions; they can be any valid Python expression, though types are conventional.
- Python does not enforce annotations at runtime.
- Annotations are evaluated at definition time (unless deferred), which can cause `NameError` for forward references.
- Lambda functions cannot have parameter annotations.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Parameter Annotations

```python
# step1: Annotate parameters
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

# step2: Call with correct types
print(greet("Alice"))              # Hello, Alice!
print(greet("Bob", "Hi"))          # Hi, Bob!

# step3: Inspect annotations
print(greet.__annotations__)
# {'name': <class 'str'>, 'greeting': <class 'str'>, 'return': <class 'str'>}
```

**Expected Output**:
```
Hello, Alice!
Hi, Bob!
{'name': <class 'str'>, 'greeting': <class 'str'>, 'return': <class 'str'>}
```

**Why**: Annotations are stored in `__annotations__` with parameter names as keys and the return annotation under `'return'`.

#### Example 2: Annotating `*args` and `**kwargs`

```python
# step1: Annotate variadic parameters
def sum_all(*args: int, **flags: bool) -> int:
    total = sum(args)
    if flags.get("double"):
        total *= 2
    return total

# step2: Call with correct types
print(sum_all(1, 2, 3))                # 6
print(sum_all(1, 2, 3, double=True))   # 12

# step3: Inspect annotations
print(sum_all.__annotations__)
# {'args': <class 'int'>, 'flags': <class 'bool'>, 'return': <class 'int'>}
```

**Expected Output**:
```
6
12
{'args': <class 'int'>, 'flags': <class 'bool'>, 'return': <class 'int'>}
```

**Why**: `*args: int` means each argument in `args` should be an `int`; `**flags: bool` means each value in `flags` should be a `bool`.

#### Example 3: Positional-Only and Keyword-Only Parameters

```python
# step1: Annotate positional-only and keyword-only parameters
def configure(name: str, /, timeout: int = 30, *, verbose: bool = False) -> None:
    print(f"name={name}, timeout={timeout}, verbose={verbose}")

# step2: Call with correct syntax
configure("server")                      # name=server, timeout=30, verbose=False
configure("server", 60, verbose=True)    # name=server, timeout=60, verbose=True

# step3: Inspect annotations
print(configure.__annotations__)
# {'name': <class 'str'>, 'timeout': <class 'int'>, 'verbose': <class 'bool'>}
```

**Expected Output**:
```
name=server, timeout=30, verbose=False
name=server, timeout=60, verbose=True
{'name': <class 'str'>, 'timeout': <class 'int'>, 'verbose': <class 'bool'>}
```

**Why**: The `/` separator marks positional-only parameters; `*` marks the start of keyword-only parameters. Annotations are attached to each parameter regardless of its kind.

### Real-World Cases

- **API endpoints**: `def get_user(user_id: int) -> dict:`.
- **Data processing**: `def transform(data: list[float], scale: float = 1.0) -> list[float]:`.
- **Callbacks**: `def on_event(event: str, **payload: Any) -> None:`.
- **Configuration**: `def configure(host: str, port: int = 8080) -> None:`.

### References

- PEP 3107 – Function Annotations - https://peps.python.org/pep-3107/
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- Function Annotations - https://docs.python.org/3/tutorial/controlflow.html#function-annotations
- Annotations Best Practices - https://docs.python.org/3/howto/annotations.html

---

## 3. Return Annotations

### Definitions

**Core Definition**: A return annotation specifies the expected type of a function's return value, using the arrow (`->`) syntax between the parameter list and the colon that begins the function body.

**Technical Definition**: Return annotations use the syntax `def function(...) -> expression:` where `expression` is any valid Python expression (typically a type). The return annotation is evaluated at function definition time and stored in the function's `__annotations__` dictionary under the special key `'return'`. PEP 3107 introduced the syntax; PEP 484 gave it type-hinting semantics.

**Beginner-Friendly Explanation**: The return annotation tells you what type of value the function produces. If you write `def add(a: int, b: int) -> int:`, you're saying "this function returns an integer." If the function returns nothing meaningful, use `-> None`.

### Purposes

- To specify the expected return type of a function.
- To enable type checkers to verify that return statements match the declared type.
- To improve IDE autocompletion for function call results.
- To document the function's output for human readers.
- To support generic return types (e.g., `-> T`).

### Syntax Rules and Structure

#### Complete General Syntax

```python
def function(param: type) -> return_type:
    return value

# Examples
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}"

def log(message: str) -> None:
    print(message)
```

**Component Breakdown**:
- `->` — the arrow operator, placed between the parameter list and the colon.
- `return_type` — any valid Python expression (typically a type).
- The annotation is stored in `__annotations__['return']`.

The syntax is: the parameter list is followed by a literal `->` and a Python expression.

#### Syntax Rules

1. **Arrow syntax**: `->` comes after the closing parenthesis of the parameter list and before the colon.
2. **`-> None` for functions that return nothing**: Functions without an explicit `return` statement implicitly return `None`.
3. **Return annotations are evaluated at definition time** (unless deferred).
4. **`__annotations__['return']`**: The return annotation is stored under the key `'return'`.
5. **Generic return types**: Use type variables (`-> T`) for generic functions.
6. **`Self` return type**: Use `-> Self` for methods that return `self` or an instance of the same class.
7. **`Never`/`NoReturn`**: Use `-> Never` (or `-> NoReturn`) for functions that never return (always raise).
8. **`NoReturn` is equivalent to `Never`**: Both indicate the function never returns normally.

#### Constraints and Limitations

- The return annotation describes the declared return type, but Python does not enforce it at runtime.
- Functions with branches returning different types must be annotated with a union (e.g., `-> int | None`).
- `async def` functions return coroutines; annotate with `-> Coroutine[Any, Any, ReturnType]` or use `async def func() -> ReturnType:` which the type checker interprets correctly.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Return Annotations

```python
# step1: Function returning int
def add(a: int, b: int) -> int:
    return a + b

# step2: Function returning str
def greet(name: str) -> str:
    return f"Hello, {name}"

# step3: Function returning None
def log(message: str) -> None:
    print(message)

# step4: Inspect return annotations
print(add.__annotations__['return'])     # <class 'int'>
print(greet.__annotations__['return'])   # <class 'str'>
print(log.__annotations__['return'])     # <class 'NoneType'>

print(add(1, 2))          # 3
print(greet("Alice"))     # Hello, Alice
print(log("test"))        # test
```

**Expected Output**:
```
<class 'int'>
<class 'str'>
<class 'NoneType'>
3
Hello, Alice
test
```

**Why**: The return annotation is stored under the `'return'` key in `__annotations__`.

#### Example 2: Union Return Types

```python
# step1: Function that may return int or None
def find_index(items: list[int], target: int) -> int | None:
    try:
        return items.index(target)
    except ValueError:
        return None

# step2: Call and handle both cases
print(find_index([1, 2, 3], 2))   # 1
print(find_index([1, 2, 3], 9))   # None

# step3: Inspect annotation
print(find_index.__annotations__['return'])  # int | None
```

**Expected Output**:
```
1
None
int | None
```

**Why**: The union return type `int | None` correctly expresses that the function may return an integer or `None`.

#### Example 3: Generic and `Self` Return Types

```python
from typing import TypeVar, Self

T = TypeVar('T')

# step1: Generic return type
def first(items: list[T]) -> T:
    return items[0]

# step2: Self return type for method chaining
class Builder:
    def __init__(self) -> None:
        self.parts: list[str] = []

    def add(self, part: str) -> Self:
        self.parts.append(part)
        return self

# step3: Use generic function
print(first([10, 20, 30]))       # 10
print(first(["a", "b", "c"]))    # a

# step4: Use Self for chaining
builder = Builder().add("x").add("y")
print(builder.parts)  # ['x', 'y']
```

**Expected Output**:
```
10
a
['x', 'y']
```

**Why**: `-> T` preserves the element type; `-> Self` enables method chaining while preserving the subclass type.

### Real-World Cases

- **Factory functions**: `def create_model(name: str) -> Model:`.
- **Validation**: `def validate(data: dict) -> bool:`.
- **Data retrieval**: `def get_user(id: int) -> User | None:`.
- **Fluent interfaces**: `def where(self, condition: str) -> Self:`.

### References

- PEP 3107 – Function Annotations (Return Values) - https://peps.python.org/pep-3107/#return-values
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- `typing.Self` - https://docs.python.org/3/library/typing.html#typing.Self
- `typing.Never` - https://docs.python.org/3/library/typing.html#typing.Never

---

## 4. Built-in Generic Types (e.g., `list[int]`, `dict[str, int]`)

### Definitions

**Core Definition**: Built-in generic types allow standard Python collections (`list`, `dict`, `set`, `tuple`, etc.) to be parameterized with element types directly, without importing from the `typing` module.

**Technical Definition**: Introduced by PEP 585 (Python 3.9), standard collections implement `__class_getitem__()` to support generic subscription. This enables `list[int]`, `dict[str, float]`, `set[bytes]`, and `tuple[int, ...]` as valid type annotations. The `typing.List`, `typing.Dict`, etc. are deprecated aliases for backward compatibility. Generic subscription also works with `collections.abc` abstract base classes such as `Sequence`, `Mapping`, and `Iterable`.

**Beginner-Friendly Explanation**: Instead of writing `from typing import List` and then `List[int]`, you can just write `list[int]` — it means "a list of integers." This works for `dict`, `set`, `tuple`, and many other standard collections.

### Purposes

- To simplify type annotations by removing the need for `typing` imports.
- To unify the type hierarchy, eliminating duplicated collection types in `typing`.
- To make type hints more readable and teachable.
- To enable generic annotations for all standard collections.
- To support abstract collection types via `collections.abc`.

### Syntax Rules and Structure

#### Complete General Syntaxes

```python
# List of integers
numbers: list[int] = [1, 2, 3]

# Dictionary mapping strings to floats
scores: dict[str, float] = {"alice": 95.5}

# Set of strings
tags: set[str] = {"python", "typing"}

# Fixed-length tuple
point: tuple[int, int] = (10, 20)

# Variable-length tuple
values: tuple[int, ...] = (1, 2, 3, 4)

# Abstract collections
from collections.abc import Sequence, Mapping, Iterable
def process(items: Sequence[str]) -> None: ...
def lookup(data: Mapping[str, int]) -> None: ...
def iterate(items: Iterable[float]) -> None: ...
```

**Component Breakdown**:
- `list[int]` — a list whose elements are integers.
- `dict[str, float]` — a dictionary with string keys and float values.
- `set[str]` — a set of strings.
- `tuple[int, int]` — a fixed-length tuple of two integers.
- `tuple[int, ...]` — a variable-length tuple of integers.
- `Sequence[str]` — any read-only sequence of strings (list, tuple, etc.).

#### Syntax Rules

1. **Built-in generics require Python 3.9+**: For earlier versions, use `typing.List`, `typing.Dict`, etc.
2. **`typing.List` and friends are deprecated**: Use built-in generics instead.
3. **PEP 585 applies to standard collections only**: Custom classes need `Generic[T]` or `__class_getitem__`.
4. **`tuple[int, ...]` for variable-length tuples**: The ellipsis indicates any number of integers.
5. **`tuple[int, str]` for fixed-length tuples**: Each position has a specific type.
6. **Abstract collections from `collections.abc`**: `Sequence`, `Mapping`, `Iterable`, etc. support generic subscription.
7. **Variance rules apply**: `Sequence` is covariant; `MutableSequence` is invariant.

#### Constraints and Limitations

- Built-in generics require Python 3.9+ for runtime subscription.
- `from __future__ import annotations` allows syntax before 3.9 but type checkers must support it.
- Runtime enforcement of element types is limited; `list[int]` does not prevent adding a string at runtime.
- `typing.List` and friends are deprecated but still functional.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Basic Built-in Generics

```python
# step1: List of integers
numbers: list[int] = [1, 2, 3]

# step2: Dictionary mapping strings to integers
counts: dict[str, int] = {"a": 1, "b": 2}

# step3: Set of strings
tags: set[str] = {"python", "typing"}

# step4: Fixed-length tuple
point: tuple[int, int] = (10, 20)

# step5: Variable-length tuple
values: tuple[int, ...] = (1, 2, 3, 4, 5)

print(numbers)  # [1, 2, 3]
print(counts)   # {'a': 1, 'b': 2}
print(tags)     # {'python', 'typing'}
print(point)    # (10, 20)
print(values)   # (1, 2, 3, 4, 5)
```

**Expected Output**:
```
[1, 2, 3]
{'a': 1, 'b': 2}
{'python', 'typing'}
(10, 20)
(1, 2, 3, 4, 5)
```

**Why**: Built-in generics annotate the element types of standard collections without importing from `typing`.

#### Example 2: Abstract Collection Types

```python
from collections.abc import Sequence, Mapping, Iterable

# step1: Function accepting any sequence
def sum_sequence(numbers: Sequence[int]) -> int:
    return sum(numbers)

# step2: Works with lists and tuples
print(sum_sequence([1, 2, 3]))    # 6
print(sum_sequence((4, 5, 6)))    # 15

# step3: Function accepting any mapping
def get_value(data: Mapping[str, int], key: str) -> int | None:
    return data.get(key)

print(get_value({"a": 1}, "a"))   # 1
print(get_value({"a": 1}, "b"))   # None

# step4: Function accepting any iterable
def total(items: Iterable[float]) -> float:
    return sum(items)

print(total([1.5, 2.5]))          # 4.0
print(total({3.0, 4.0}))          # 7.0
```

**Expected Output**:
```
6
15
1
None
4.0
7.0
```

**Why**: `Sequence[int]` accepts lists and tuples; `Mapping[str, int]` accepts dictionaries; `Iterable[float]` accepts lists and sets.

#### Example 3: Nested Built-in Generics

```python
# step1: List of lists
matrix: list[list[int]] = [[1, 2], [3, 4]]

# step2: Dictionary with list values
groups: dict[str, list[str]] = {"fruits": ["apple", "banana"]}

# step3: Tuple of different types
record: tuple[int, str, float] = (1, "hello", 3.14)

# step4: Function with nested generics
def flatten(matrix: list[list[int]]) -> list[int]:
    return [item for row in matrix for item in row]

print(flatten(matrix))  # [1, 2, 3, 4]
print(groups)           # {'fruits': ['apple', 'banana']}
print(record)           # (1, 'hello', 3.14)
```

**Expected Output**:
```
[1, 2, 3, 4]
{'fruits': ['apple', 'banana']}
(1, 'hello', 3.14)
```

**Why**: Built-in generics can be nested arbitrarily: `list[list[int]]`, `dict[str, list[str]]`, etc.

### Real-World Cases

- **Data processing**: `list[float]` for numerical arrays.
- **Configuration**: `dict[str, str]` for key-value settings.
- **API responses**: `dict[str, Any]` for JSON-like objects.
- **Type-safe collections**: `set[int]` for unique identifiers.

### References

- PEP 585 – Type Hinting Generics In Standard Collections - https://peps.python.org/pep-0585/
- Built-in Types (Generic Subscription) - https://docs.python.org/3/library/stdtypes.html
- `collections.abc` — Abstract Base Classes for Containers - https://docs.python.org/3/library/collections.abc.html
- `typing` — Support for type hints - https://docs.python.org/3/library/typing.html

---

## 5. Forward References (String Literals and `from __future__ import annotations`)

### Definitions

**Core Definition**: Forward references allow a type annotation to refer to a name that has not yet been defined in the current scope, using either a string literal or postponed evaluation via `from __future__ import annotations`.

**Technical Definition**: When a type annotation refers to a class or type that is defined later in the module (or in a circular import), Python's eager evaluation of annotations at definition time causes a `NameError`. Forward references solve this by (a) expressing the type as a string literal (e.g., `"Tree"`), which is not evaluated at definition time, or (b) using `from __future__ import annotations` (PEP 563) to defer all annotation evaluation to string form. Python 3.14 introduces deferred evaluation by default via PEP 649/749.

**Beginner-Friendly Explanation**: Sometimes you need to refer to a class in a type hint before that class is defined. Python would normally complain because it evaluates annotations immediately. Forward references let you say "I'll tell you about this type later" — either by putting the type in quotes (`"MyClass"`) or by telling Python to postpone all annotation evaluation.

### Purposes

- To annotate self-referential classes (e.g., tree nodes, linked lists).
- To annotate classes defined later in the module.
- To resolve circular import issues in type annotations.
- To avoid `NameError` when annotations reference undefined names.
- To improve module load time by deferring annotation evaluation.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: String Literal Forward Reference**

```python
class TreeNode:
    def __init__(self, value: int, left: "TreeNode | None" = None) -> None:
        self.value = value
        self.left = left
```

**Syntax 2: `from __future__ import annotations`**

```python
from __future__ import annotations

class TreeNode:
    def __init__(self, value: int, left: TreeNode | None = None) -> None:
        self.value = value
        self.left = left
```

**Syntax 3: Combining with Type Aliases**

```python
from __future__ import annotations

type JsonValue = str | int | float | bool | None | dict[str, JsonValue] | list[JsonValue]
```

**Component Breakdown**:
- `"TreeNode | None"` — string literal; not evaluated at definition time.
- `from __future__ import annotations` — enables PEP 563 postponed evaluation.
- `type JsonValue = ...` — PEP 695 type alias statement (Python 3.12+).

#### Syntax Rules

1. **String literals are not evaluated**: `"TreeNode"` is stored as the string `'TreeNode'` in `__annotations__`.
2. **`from __future__ import annotations` must be the first statement**: It must appear before any other imports or code (after the module docstring).
3. **PEP 563 stores annotations as strings**: All annotations are preserved in string form in `__annotations__`.
4. **Type checkers resolve forward references**: Static type checkers (mypy, Pyright) parse string annotations and resolve them against the module scope.
5. **`typing.get_type_hints()` resolves forward references**: Use this function to evaluate string annotations at runtime.
6. **Python 3.14 defers annotations by default**: PEP 649/749 makes deferred evaluation the default, eliminating the need for `from __future__ import annotations`.
7. **`TYPE_CHECKING` for runtime-only imports**: Use `if TYPE_CHECKING:` to import types only for type checking.

#### Constraints and Limitations

- String annotations are not resolved automatically at runtime; use `get_type_hints()`.
- `from __future__ import annotations` changes runtime behavior of `__annotations__`.
- PEP 563 is superseded by PEP 649 (Python 3.14+), which uses a different mechanism.
- Forward references in non-annotation contexts (e.g., `TypeVar` bounds) still require string literals.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Self-Referential Class with String Literal

```python
# step1: Define a tree node with a string forward reference
class TreeNode:
    def __init__(self, value: int, left: "TreeNode | None" = None, right: "TreeNode | None" = None) -> None:
        self.value = value
        self.left = left
        self.right = right

# step2: Create a tree
root = TreeNode(1, TreeNode(2), TreeNode(3))
print(root.value)         # 1
print(root.left.value)    # 2
print(root.right.value)   # 3

# step3: Inspect annotations (stored as strings)
print(TreeNode.__init__.__annotations__)
# {'value': <class 'int'>, 'left': 'TreeNode | None', 'right': 'TreeNode | None', 'return': None}
```

**Expected Output**:
```
1
2
3
{'value': <class 'int'>, 'left': 'TreeNode | None', 'right': 'TreeNode | None', 'return': None}
```

**Why**: The string literal `"TreeNode | None"` avoids `NameError` because `TreeNode` is not yet defined when the annotation is evaluated.

#### Example 2: `from __future__ import annotations`

```python
from __future__ import annotations

# step1: Define a class that refers to itself without quotes
class TreeNode:
    def __init__(self, value: int, left: TreeNode | None = None) -> None:
        self.value = value
        self.left = left

# step2: Create a tree
root = TreeNode(1, TreeNode(2))
print(root.value)         # 1
print(root.left.value)    # 2

# step3: All annotations are stored as strings
print(TreeNode.__init__.__annotations__)
# {'value': 'int', 'left': 'TreeNode | None', 'return': 'None'}
```

**Expected Output**:
```
1
2
{'value': 'int', 'left': 'TreeNode | None', 'return': 'None'}
```

**Why**: `from __future__ import annotations` defers all annotation evaluation, so `TreeNode` can be referenced directly without quotes.

#### Example 3: Resolving Forward References with `get_type_hints()`

```python
from __future__ import annotations
from typing import get_type_hints

class TreeNode:
    def __init__(self, value: int, left: TreeNode | None = None) -> None:
        self.value = value
        self.left = left

# step1: Raw annotations are strings
print(TreeNode.__init__.__annotations__)
# {'value': 'int', 'left': 'TreeNode | None', 'return': 'None'}

# step2: Resolve annotations to actual types
hints = get_type_hints(TreeNode.__init__)
print(hints)
# {'value': <class 'int'>, 'left': TreeNode | None, 'return': NoneType}

# step3: Use resolved hints
print(hints['value'])  # <class 'int'>
```

**Expected Output**:
```
{'value': 'int', 'left': 'TreeNode | None', 'return': 'None'}
{'value': <class 'int'>, 'left': TreeNode | None, 'return': NoneType}
<class 'int'>
```

**Why**: `get_type_hints()` evaluates string annotations against the module's namespace, resolving forward references to actual types.

### Real-World Cases

- **Tree and linked list nodes**: Self-referential data structures.
- **Circular imports**: Modules that import each other for type annotations.
- **Recursive JSON types**: `type JsonValue = ... | list[JsonValue]`.
- **Graph structures**: `class Node: neighbors: list["Node"]`.

### References

- PEP 563 – Postponed Evaluation of Annotations - https://peps.python.org/pep-0563/
- PEP 649 – Deferred Evaluation Of Annotations - https://peps.python.org/pep-0649/
- PEP 749 – Deferred Evaluation Of Annotations Using Descriptors - https://peps.python.org/pep-0749/
- Forward References - https://typing.python.org/en/latest/spec/annotations.html#forward-references
- `typing.get_type_hints()` - https://docs.python.org/3/library/typing.html#typing.get_type_hints
- Annotations Best Practices - https://docs.python.org/3/howto/annotations.html

---

## References

- PEP 3107 – Function Annotations - https://peps.python.org/pep-3107/
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- PEP 483 – The Theory of Type Hints - https://peps.python.org/pep-0483/
- PEP 526 – Syntax for Variable Annotations - https://peps.python.org/pep-0526/
- PEP 563 – Postponed Evaluation of Annotations - https://peps.python.org/pep-0563/
- PEP 585 – Type Hinting Generics In Standard Collections - https://peps.python.org/pep-0585/
- PEP 649 – Deferred Evaluation Of Annotations - https://peps.python.org/pep-0649/
- PEP 749 – Deferred Evaluation Of Annotations Using Descriptors - https://peps.python.org/pep-0749/
- `typing` — Support for type hints - https://docs.python.org/3/library/typing.html
- Annotations Best Practices - https://docs.python.org/3/howto/annotations.html
- `collections.abc` — Abstract Base Classes for Containers - https://docs.python.org/3/library/collections.abc.html
- mypy Type Hints Cheat Sheet - https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html
- Python Type System Specification - https://typing.python.org/en/latest/spec/