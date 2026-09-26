# Type-Oriented Design in Python: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Core Definition

Type-oriented design is a software design methodology in which the type system is used as a primary tool for expressing program structure, contracts, and invariants. Rather than treating types as mere documentation, type-oriented design places types at the center of the design process: interfaces are defined by structural contracts, algorithms are parameterized over types, and data structures are made generic to preserve type information throughout transformations.

### Technical Definition

Type-oriented design leverages Python's gradual type system — including nominal and structural subtyping, parametric polymorphism (generics), variance, type narrowing, and protocol classes — to construct programs whose correctness properties are statically verifiable by type checkers. It draws on principles from type theory, including the Liskov substitution principle, parametric polymorphism, and structural type equivalence, while remaining compatible with Python's dynamic runtime semantics.

### Beginner-Friendly Explanation

Type-oriented design means thinking about your program in terms of types: what kinds of values flow through your code, what operations are valid on them, and how they relate to one another. Instead of just writing functions and hoping they work, you design the types first — defining interfaces, making algorithms work with any type, and using the type checker to catch mistakes before you run the code.

### Key Characteristics

- **Interfaces first**: Define contracts (via `Protocol` or ABCs) before implementations.
- **Genericity**: Write algorithms and data structures once, parameterized over types.
- **Structural flexibility**: Use `Protocol` for static duck typing, avoiding unnecessary inheritance.
- **Type preservation**: Generic containers and functions preserve type information through transformations.
- **Static verifiability**: Type checkers (mypy, Pyright) verify design correctness.
- **Variance awareness**: Understand when subtyping relationships are preserved, reversed, or blocked.

### Prerequisites

- Python 3.9+ (some features require 3.10+, 3.11+, 3.12+, or 3.13+).
- Familiarity with basic type annotations (`int`, `str`, `list[int]`).
- A static type checker installed (`mypy`, `pyright`).
- Understanding of classes, inheritance, and abstract base classes.

### Related Programming Areas

- **Domain-driven design**: Using types to model business domains.
- **Functional programming**: Generic algorithms, higher-order functions.
- **Design patterns**: Strategy, repository, visitor, and factory patterns.
- **API design**: Type-safe interfaces for libraries and frameworks.

### Core Concepts / Features

The following sections cover each type-oriented design concept using a uniform structure.

---

## 1. Interfaces (Nominal vs. Structural)

### Definitions

**Core Definition**: An interface is a contract specifying the methods and attributes that a type must provide. Python supports two forms of interface-based subtyping: nominal (based on explicit inheritance) and structural (based on the presence of required members).

**Technical Definition**: Nominal subtyping determines type compatibility strictly from the class hierarchy: if class `Dog` inherits from class `Animal`, then `Dog` is a subtype of `Animal`. Structural subtyping determines compatibility from the operations an object supports: class `Dog` is a structural subtype of class `Animal` if the former has all attributes and methods of the latter, with compatible types. Python's type system predominantly uses nominal subtyping, while `typing.Protocol` (PEP 544) enables structural subtyping.

**Beginner-Friendly Explanation**: Nominal typing says "you are what you inherit from." Structural typing says "you are what you can do." If a class has a `close()` method, it can be used wherever something with a `close()` method is expected — even if it never inherited from a "Closeable" base class.

### Purposes

- To define explicit contracts for classes and modules.
- To enable polymorphism without forcing inheritance hierarchies.
- To decouple components and improve testability.
- To annotate third-party objects that cannot be modified.
- To provide clear error messages when contracts are violated.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Nominal Interface via Abstract Base Class**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        ...

class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius ** 2
```

**Component Breakdown**:
- `ABC` — base class for abstract base classes; enforces nominal subtyping.
- `@abstractmethod` — marks a method that must be implemented by subclasses.
- Subclasses must explicitly inherit from `Shape` to be considered subtypes.

**Syntax 2: Structural Interface via `Protocol`**

```python
from typing import Protocol

class SupportsArea(Protocol):
    def area(self) -> float:
        ...

class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius ** 2

class Square:
    def __init__(self, side: float) -> None:
        self.side = side

    def area(self) -> float:
        return self.side ** 2

def print_area(shape: SupportsArea) -> None:
    print(shape.area())

print_area(Circle(5))   # 78.53975
print_area(Square(4))   # 16
```

**Component Breakdown**:
- `Protocol` — base class for structural interfaces.
- `SupportsArea` — protocol defining the required `area()` method.
- `Circle` and `Square` satisfy the protocol structurally without inheriting from it.

#### Syntax Rules

1. **Nominal subtyping is the default**: Python's `isinstance()` checks are based on class hierarchy.
2. **Structural subtyping requires `Protocol`**: Only protocol classes enable structural compatibility.
3. **Protocol members must have empty bodies**: Use `...` or `pass`.
4. **Explicit subclassing of protocols is allowed**: A class can inherit from a protocol, but it is not required.
5. **Protocols can be generic**: `class Container[T](Protocol): ...`.

#### Constraints and Limitations

- Protocol classes cannot be instantiated.
- Runtime `isinstance()` checks with `@runtime_checkable` only verify method presence, not signatures.
- Nominal and structural subtyping coexist; type checkers use both depending on the context.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Nominal Interface with ABC

```python
from abc import ABC, abstractmethod

# step1: Define the interface
class Logger(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        ...

# step2: Implement the interface
class ConsoleLogger(Logger):
    def log(self, message: str) -> None:
        print(f"[CONSOLE] {message}")

# step3: Use the interface
def process(logger: Logger, data: str) -> None:
    logger.log(f"Processing: {data}")

process(ConsoleLogger(), "test")
```

**Expected Output**:
```
[CONSOLE] Processing: test
```

**Why**: `ConsoleLogger` inherits from `Logger` and implements `log()`. The type checker enforces this nominal relationship.

#### Example 2: Structural Interface with `Protocol`

```python
from typing import Protocol

# step1: Define the protocol
class Loggable(Protocol):
    def log(self, message: str) -> None:
        ...

# step2: Classes that satisfy the protocol (no inheritance)
class ConsoleLogger:
    def log(self, message: str) -> None:
        print(f"[CONSOLE] {message}")

class FileLogger:
    def log(self, message: str) -> None:
        print(f"[FILE] {message}")

# step3: Use the protocol
def process(logger: Loggable, data: str) -> None:
    logger.log(f"Processing: {data}")

process(ConsoleLogger(), "test")  # [CONSOLE] Processing: test
process(FileLogger(), "test")     # [FILE] Processing: test
```

**Expected Output**:
```
[CONSOLE] Processing: test
[FILE] Processing: test
```

**Why**: Both loggers have a `log()` method, so they satisfy `Loggable` structurally without inheriting from it.

#### Example 3: Combined Nominal and Structural Interfaces

```python
from abc import ABC, abstractmethod
from typing import Protocol

# Nominal interface
class Serializable(ABC):
    @abstractmethod
    def serialize(self) -> str:
        ...

# Structural interface
class Comparable(Protocol):
    def compare_to(self, other: object) -> int:
        ...

# Class implementing both
class Record(Serializable):
    def __init__(self, id: int) -> None:
        self.id = id

    def serialize(self) -> str:
        return f"Record({self.id})"

    def compare_to(self, other: object) -> int:
        if not isinstance(other, Record):
            return NotImplemented
        return self.id - other.id

# Use both interfaces
def save_and_sort(items: list[Serializable]) -> None:
    for item in sorted(items, key=lambda x: x.compare_to):
        print(item.serialize())

save_and_sort([Record(3), Record(1), Record(2)])
```

**Expected Output**:
```
Record(1)
Record(2)
Record(3)
```

**Why**: `Record` satisfies the nominal `Serializable` interface (via inheritance) and the structural `Comparable` protocol (via method presence).

### Real-World Cases

- **Plugin systems**: Protocols define plugin interfaces without forcing plugin authors to inherit from a specific base class.
- **Testing**: Mock objects satisfy protocols without inheriting from production classes.
- **Standard library**: `Iterable`, `Iterator`, and `Sized` are predefined protocols.
- **Web frameworks**: Request/response objects are often defined as protocols.

### References

- Protocols and Structural Subtyping - https://typing.python.org/en/latest/reference/protocols.html
- PEP 544 – Protocols: Structural subtyping - https://peps.python.org/pep-0544/
- Abstract Base Classes - https://docs.python.org/3/library/abc.html
- Real Python: Python Protocols - https://realpython.com/python-protocol/

---

## 2. Generic Algorithms and Data Structures

### Definitions

**Core Definition**: Generic algorithms and data structures are parameterized over types, allowing them to operate uniformly on values of different types while preserving type information through transformations.

**Technical Definition**: A generic function uses type variables (`TypeVar`) in its parameter and return annotations, with the type variables inferred at call time. A generic class inherits from `typing.Generic[T]` (or uses PEP 695 syntax `class Stack[T]:`) and uses type variables in its method signatures and attribute annotations. Generic algorithms are parametric polymorphic — they work for any type that satisfies their constraints.

**Beginner-Friendly Explanation**: A generic algorithm is one that works with any type of data. A generic stack can hold integers, strings, or any other type — and when you pop an item, the type checker knows exactly what type you get back.

### Purposes

- To write algorithms that work with any type while preserving type safety.
- To create reusable data structures (stacks, queues, trees, repositories).
- To avoid code duplication across similar types.
- To preserve type information through transformations.
- To enable type-safe abstractions like `Result[T, E]` and `Repository[T]`.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Generic Function**

```python
from typing import TypeVar

T = TypeVar('T')

def first(items: list[T]) -> T:
    return items[0]
```

**Component Breakdown**:
- `T = TypeVar('T')` — declares a type variable.
- `items: list[T]` — parameter using the type variable.
- `-> T` — return type preserving the type variable.

**Syntax 2: Generic Class (Traditional)**

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
```

**Syntax 3: Generic Class (PEP 695, Python 3.12+)**

```python
class Stack[T]:
    def __init__(self) -> None:
        self.items: list[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()
```

**Component Breakdown**:
- `[T]` — type parameter list.
- `self.items: list[T]` — generic attribute.

#### Syntax Rules

1. **Type variables must be declared**: Use `TypeVar('T')` or PEP 695 syntax.
2. **Generic base class must be parameterized**: `class Stack(Generic[T])`, not `class Stack(Generic)`.
3. **Type parameters are scoped**: In PEP 695 syntax, `[T]` is scoped to the class/function.
4. **Generic functions infer type parameters**: Type checkers infer `T` from call arguments.
5. **Variance rules apply**: For generic classes, type variables can be covariant, contravariant, or invariant (see Section 6).

#### Constraints and Limitations

- Generic classes cannot be used without type parameters unless `Any` is implied.
- Type variables must be distinct within a generic class.
- PEP 695 syntax requires Python 3.12+.
- Runtime enforcement of generic types is limited to container element checks (e.g., `list[int]` is not enforced at runtime).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Generic Stack

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

# step1: Stack of integers
s = Stack[int]()
s.push(1)
s.push(2)
print(s.pop())  # 2

# step2: Stack of strings
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

**Why**: `Stack[int]` only accepts integers; `Stack[str]` only accepts strings. The type checker enforces this.

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

#### Example 3: Generic Queue with Bounded Type Parameter

```python
from typing import TypeVar, Generic
from collections import deque

T = TypeVar('T')

class Queue(Generic[T]):
    def __init__(self) -> None:
        self._items: deque[T] = deque()

    def enqueue(self, item: T) -> None:
        self._items.append(item)

    def dequeue(self) -> T:
        return self._items.popleft()

    def __len__(self) -> int:
        return len(self._items)

q = Queue[int]()
q.enqueue(10)
q.enqueue(20)
print(q.dequeue())  # 10
print(len(q))       # 1
```

**Expected Output**:
```
10
1
```

**Why**: `Queue[int]` is a queue of integers. The generic type parameter `T` is preserved through enqueue and dequeue operations.

### Real-World Cases

- **Collections**: `list[T]`, `dict[K, V]`, `set[T]` are built-in generics.
- **Repositories**: `Repository[Model]` with generic CRUD operations.
- **Result types**: `Result[T, E]` for success/failure handling.
- **Trees and graphs**: Generic node and edge types.

### References

- Generics - https://typing.python.org/en/latest/spec/generics.html
- PEP 484 – Type Hints (Generics) - https://peps.python.org/pep-0484/#generics
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/
- mypy Generics Documentation - https://mypy.readthedocs.io/en/stable/generics.html

---

## 3. Protocol-Based Programming

### Definitions

**Core Definition**: Protocol-based programming is a design approach that uses `typing.Protocol` to define structural interfaces, enabling static duck typing and decoupling components without inheritance.

**Technical Definition**: A protocol class is a class that inherits from `typing.Protocol` and defines a set of methods and attributes that constitute a structural type. Any class that provides those members (with compatible types) is considered a structural subtype of the protocol, regardless of its inheritance chain. This implements structural subtyping in a statically typed context, matching Python's runtime duck typing semantics.

**Beginner-Friendly Explanation**: Protocol-based programming means defining what an object can do, not what it is. Instead of requiring a class to inherit from a specific base class, you say "I need something with a `read()` method and a `close()` method" — and any object that has those methods works.

### Purposes

- To define interfaces without forcing inheritance.
- To enable static duck typing in type-checked code.
- To decouple components and improve testability.
- To annotate third-party objects that cannot be modified.
- To provide structural type safety for callbacks and handlers.

### Syntax Rules and Structure

#### Complete General Syntax

```python
from typing import Protocol, runtime_checkable

class SupportsClose(Protocol):
    def close(self) -> None:
        ...
```

**Component Breakdown**:
- `Protocol` — base class from `typing` (or `typing_extensions`).
- `close(self) -> None` — protocol member (method signature).
- `...` — empty body.

#### Syntax Rules

1. **Protocol members must have empty bodies**: Use `...` or `pass`.
2. **Subclassing a protocol does not make it a protocol**: A subclass must also inherit from `Protocol` to remain a protocol.
3. **Structural subtyping is automatic**: No explicit inheritance required for implementers.
4. **`@runtime_checkable`**: Allows `isinstance()` checks against protocols at runtime (methods only).
5. **Protocols can be generic**: `class Container[T](Protocol): ...`.

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
class Drawable(Protocol):
    def draw(self) -> None:
        ...

# step2: Implementations (no inheritance)
class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

# step3: Function accepts the protocol
def render(items: list[Drawable]) -> None:
    for item in items:
        item.draw()

render([Circle(), Square()])
```

**Expected Output**:
```
Drawing circle
Drawing square
```

**Why**: `Circle` and `Square` have a `draw()` method, so they satisfy `Drawable` structurally.

#### Example 2: Runtime Checkable Protocol

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None:
        ...

class Resource:
    def close(self) -> None:
        print("Resource closed")

class NotCloseable:
    pass

print(isinstance(Resource(), Closeable))      # True
print(isinstance(NotCloseable(), Closeable))  # False
```

**Expected Output**:
```
True
False
```

**Why**: `@runtime_checkable` allows `isinstance` to check for the presence of `close()` at runtime.

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

- **File-like objects**: `SupportsRead`, `SupportsWrite` protocols.
- **Database connections**: `SupportsExecute` protocol.
- **Web frameworks**: Request/response protocols.
- **Testing**: Mock objects that satisfy protocols without inheritance.

### References

- Protocols - https://typing.python.org/en/latest/spec/protocol.html
- PEP 544 – Protocols: Structural subtyping - https://peps.python.org/pep-0544/
- `typing.Protocol` - https://docs.python.org/3/library/typing.html#typing.Protocol
- Real Python: Python Protocols - https://realpython.com/python-protocol/

---

## 4. Type Narrowing (isinstance(), Structural Checks, TypeGuard, TypeIs)

### Definitions

**Core Definition**: Type narrowing is the process by which a type checker refines the type of a variable within a conditional branch, based on runtime checks.

**Technical Definition**: Type narrowing occurs when a type checker uses information from conditional checks (e.g., `isinstance()`, `is not None`, `callable()`) to infer a more specific type for a variable within a branch. Python supports built-in narrowing constructs and user-defined narrowing functions via `typing.TypeGuard` (PEP 647) and `typing.TypeIs` (PEP 742).

**Beginner-Friendly Explanation**: Type narrowing is how the type checker figures out "inside this `if` block, this variable must be a string." You write checks like `isinstance(x, str)`, and the type checker uses them to give you more precise type information.

### Purposes

- To enable safe operations on values of union types.
- To leverage runtime type checks for static type information.
- To create reusable type-narrowing functions.
- To support exhaustiveness checking with discriminated unions.
- To reduce the need for `cast()` and `# type: ignore`.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Built-in Narrowing with `isinstance()`**

```python
def process(value: int | str) -> None:
    if isinstance(value, int):
        print(value + 1)      # value is int
    else:
        print(value.upper())  # value is str
```

**Syntax 2: `TypeIs` (Python 3.13+)**

```python
from typing import TypeIs

def is_str(x: object) -> TypeIs[str]:
    return isinstance(x, str)

def process(x: object) -> None:
    if is_str(x):
        print(x.upper())  # x is str
    else:
        print("not a string")
```

**Component Breakdown**:
- `TypeIs[str]` — the return annotation indicating the narrowed type.
- The function must return `True` if the argument is of type `T`, and `False` otherwise.

**Syntax 3: `TypeGuard` (Python 3.10+)**

```python
from typing import TypeGuard

def is_str(x: object) -> TypeGuard[str]:
    return isinstance(x, str)

def process(x: object) -> None:
    if is_str(x):
        print(x.upper())  # x is str
```

#### Syntax Rules

1. **`isinstance()` narrows in both branches**: The `if` branch gets the narrowed type; the `else` branch gets the remaining type.
2. **`TypeIs` narrows in both branches**: If the function returns `True`, the type is narrowed to `T`; if `False`, the type checker narrows to the remaining type.
3. **`TypeGuard` narrows only in the `True` branch**: When `TypeGuard` returns `False`, the type is not narrowed.
4. **`TypeIs` requires the narrowed type to be a subtype of the input type**: `TypeIs[B]` requires `B` to be a subtype of the parameter type.
5. **`TypeGuard` does not have this restriction**: The narrowed type can be any type.
6. **`TypeIs` and `TypeGuard` are not covariant**: `TypeIs[B]` is not a subtype of `TypeIs[A]` even if `B` is a subtype of `A`.
7. **`TypeGuard` is recommended for rare cases**: `TypeIs` is recommended for most use cases because it has more intuitive behavior.

#### Constraints and Limitations

- `TypeIs` requires Python 3.13+ (or `typing_extensions` 4.10.0+).
- `TypeGuard` requires Python 3.10+ (or `typing_extensions`).
- The set of narrowing constructs understood by type checkers varies; consult your checker's documentation.
- Narrowing does not apply to all expressions (e.g., tuple expressions may not narrow).

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: `isinstance()` Narrowing

```python
def process(value: int | str | list) -> None:
    if isinstance(value, int):
        print(f"Integer: {value + 1}")
    elif isinstance(value, str):
        print(f"String: {value.upper()}")
    else:
        print(f"List of {len(value)} items")

process(42)        # Integer: 43
process("hello")   # String: HELLO
process([1, 2, 3]) # List of 3 items
```

**Expected Output**:
```
Integer: 43
String: HELLO
List of 3 items
```

**Why**: Each `isinstance()` check narrows `value` to the corresponding type within each branch.

#### Example 2: `TypeIs` vs. `TypeGuard`

```python
from typing import TypeIs, TypeGuard

# TypeIs: narrows in both branches
def is_int_typeis(x: object) -> TypeIs[int]:
    return isinstance(x, int)

def process_typeis(x: int | str) -> None:
    if is_int_typeis(x):
        print(x + 1)      # x is int
    else:
        print(x.upper())  # x is str (narrowed in else branch)

# TypeGuard: narrows only in True branch
def is_int_typeguard(x: object) -> TypeGuard[int]:
    return isinstance(x, int)

def process_typeguard(x: int | str) -> None:
    if is_int_typeguard(x):
        print(x + 1)      # x is int
    else:
        print(x)          # x is int | str (NOT narrowed)

process_typeis(42)        # 43
process_typeis("hello")   # HELLO
process_typeguard(42)     # 43
process_typeguard("hello")  # hello
```

**Expected Output**:
```
43
HELLO
43
hello
```

**Why**: `TypeIs` narrows in both branches; `TypeGuard` only narrows in the `True` branch.

#### Example 3: Structural Narrowing with `hasattr()`

```python
class Person:
    def __init__(self, name: str) -> None:
        self.name = name

class Animal:
    def __init__(self, species: str) -> None:
        self.species = species

def identify(obj: Person | Animal | None) -> str:
    if obj is None:
        return "Nothing"
    if hasattr(obj, "name"):
        return f"Person: {obj.name}"  # obj narrowed to Person
    return f"Animal: {obj.species}"   # obj narrowed to Animal

print(identify(Person("Alice")))    # Person: Alice
print(identify(Animal("Dog")))      # Animal: Dog
print(identify(None))               # Nothing
```

**Expected Output**:
```
Person: Alice
Animal: Dog
Nothing
```

**Why**: The `hasattr()` check narrows `obj` to `Person` in the `if` branch and `Animal` in the `else` branch. Note that `hasattr` narrowing support varies across type checkers.

### Real-World Cases

- **JSON parsing**: Narrow `JsonValue` to specific types after validation.
- **Event handling**: Narrow event objects to specific event types.
- **AST processing**: Narrow node types in compilers and interpreters.
- **API responses**: Narrow success/error responses.

### References

- Type Narrowing - https://typing.python.org/en/latest/guides/type_narrowing.html
- PEP 647 – User-Defined Type Guards - https://peps.python.org/pep-0647/
- PEP 742 – Narrowing types with TypeIs - https://peps.python.org/pep-0742/
- `typing.TypeIs` - https://docs.python.org/3/library/typing.html#typing.TypeIs
- `typing.TypeGuard` - https://docs.python.org/3/library/typing.html#typing.TypeGuard

---

## 5. Type-Safe Collections

### Definitions

**Core Definition**: Type-safe collections are Python collections (lists, dictionaries, sets, tuples, and abstract collection types) annotated with the types of their elements, enabling static verification of element access and mutation.

**Technical Definition**: Since PEP 585 (Python 3.9), standard collections support generic subscription: `list[int]`, `dict[str, float]`, `set[bytes]`, and `tuple[int, ...]`. The `collections.abc` module provides abstract collection types such as `Sequence`, `Mapping`, `MutableSequence`, and `MutableMapping`, which are preferred for function parameter annotations because they describe the required interface rather than the concrete implementation.

**Beginner-Friendly Explanation**: A type-safe collection is a list or dictionary that knows what kind of items it contains. A `list[int]` is a list of integers — if you try to put a string in it, the type checker will complain. Using abstract types like `Sequence` makes your functions more flexible because they accept any sequence, not just lists.

### Purposes

- To catch element type errors at type-check time.
- To document the expected contents of collections.
- To enable safe iteration and element access.
- To write functions that accept any compatible collection type (via abstract base classes).
- To preserve element type information through transformations.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Concrete Collections (Python 3.9+)**

```python
# List of integers
numbers: list[int] = [1, 2, 3]

# Dictionary mapping strings to floats
scores: dict[str, float] = {"alice": 95.5, "bob": 87.0}

# Set of strings
tags: set[str] = {"python", "typing"}

# Tuple of fixed types
point: tuple[int, int] = (10, 20)

# Variable-length tuple
values: tuple[int, ...] = (1, 2, 3, 4)
```

**Component Breakdown**:
- `list[int]` — a list whose elements are integers.
- `dict[str, float]` — a dictionary with string keys and float values.
- `tuple[int, ...]` — a tuple of any number of integers.

**Syntax 2: Abstract Collection Types**

```python
from collections.abc import Sequence, Mapping, MutableSequence, MutableMapping, Iterable, Set

def process_items(items: Sequence[str]) -> None:
    for item in items:
        print(item)

def lookup(mapping: Mapping[str, int], key: str) -> int | None:
    return mapping.get(key)
```

**Component Breakdown**:
- `Sequence[str]` — any read-only sequence of strings (list, tuple, etc.).
- `Mapping[str, int]` — any read-only mapping from strings to integers.
- `MutableSequence[str]` — any mutable sequence of strings.
- `Iterable[str]` — anything that can be iterated over.

#### Syntax Rules

1. **Standard collections are generic since Python 3.9**: Use `list[int]` instead of `typing.List[int]`.
2. **`typing.List`, `typing.Dict`, etc. are deprecated**: Use built-in generics instead.
3. **Prefer abstract types for parameters**: Use `Sequence`, `Mapping`, `Iterable` instead of `list`, `dict`.
4. **Mutable collections are invariant**: `MutableSequence[int]` is not a subtype of `MutableSequence[object]`.
5. **Read-only collections are covariant**: `Sequence[int]` is a subtype of `Sequence[object]`.
6. **Tuples are covariant**: `tuple[int, str]` is a subtype of `tuple[object, object]`.

#### Constraints and Limitations

- Runtime enforcement of element types is limited; `list[int]` does not prevent adding a string at runtime.
- `typing.List` and friends are deprecated but still work for backward compatibility.
- Some abstract collection types have complex inheritance hierarchies.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Concrete vs. Abstract Collections

```python
from collections.abc import Sequence, Mapping

# step1: Concrete list annotation
def sum_list(numbers: list[int]) -> int:
    return sum(numbers)

# step2: Abstract sequence annotation (more flexible)
def sum_sequence(numbers: Sequence[int]) -> int:
    return sum(numbers)

# Both work with lists
print(sum_list([1, 2, 3]))       # 6
print(sum_sequence([1, 2, 3]))   # 6

# Abstract sequence also works with tuples
print(sum_sequence((4, 5, 6)))   # 15
```

**Expected Output**:
```
6
6
15
```

**Why**: `sum_sequence` accepts any `Sequence[int]`, including tuples and lists. `sum_list` only accepts `list[int]`.

#### Example 2: Type-Safe Dictionary Operations

```python
from collections.abc import MutableMapping

def update_scores(scores: MutableMapping[str, int], name: str, score: int) -> None:
    scores[name] = score

scores: dict[str, int] = {"alice": 90}
update_scores(scores, "bob", 85)
print(scores)  # {'alice': 90, 'bob': 85}
```

**Expected Output**:
```
{'alice': 90, 'bob': 85}
```

**Why**: `MutableMapping[str, int]` describes any mutable mapping with string keys and integer values. `dict` satisfies this interface.

#### Example 3: Type-Safe Collection with Abstract Types

```python
from collections.abc import Sequence

def find_first(items: Sequence[str], target: str) -> int | None:
    for i, item in enumerate(items):
        if item == target:
            return i
    return None

# Works with lists
print(find_first(["a", "b", "c"], "b"))  # 1

# Works with tuples
print(find_first(("x", "y", "z"), "z"))  # 2

# Works with strings (strings are sequences of characters)
print(find_first("hello", "l"))  # 2
```

**Expected Output**:
```
1
2
2
```

**Why**: `Sequence[str]` accepts any sequence of strings, including lists, tuples, and even strings (which are sequences of characters).

### Real-World Cases

- **Data processing**: `Sequence[float]` for numerical pipelines.
- **Configuration**: `Mapping[str, Any]` for configuration dictionaries.
- **API responses**: `dict[str, JsonValue]` for JSON objects.
- **Caching**: `MutableMapping[str, CacheEntry]` for cache implementations.

### References

- PEP 585 – Type Hinting Generics In Standard Collections - https://peps.python.org/pep-0585/
- `collections.abc` — Abstract Base Classes for Containers - https://docs.python.org/3/library/collections.abc.html
- `typing` — Support for type hints - https://docs.python.org/3/library/typing.html
- mypy Generics Documentation - https://mypy.readthedocs.io/en/stable/generics.html

---

## 6. Variance (Covariance, Contravariance, and Invariance)

### Definitions

**Core Definition**: Variance describes how subtyping relationships between component types affect subtyping relationships between the generic types that contain them.

**Technical Definition**: Given types `A` and `B` where `B` is a subtype of `A`:
- A generic class `MyCovGen[T]` is **covariant** in `T` if `MyCovGen[B]` is a subtype of `MyCovGen[A]`.
- A generic class `MyContraGen[T]` is **contravariant** in `T` if `MyContraGen[A]` is a subtype of `MyContraGen[B]`.
- A generic class `MyInvGen[T]` is **invariant** in `T` if neither of the above is true.

**Beginner-Friendly Explanation**: Variance tells you whether you can substitute a more specific type for a more general one in a generic container. If `Dog` is a subtype of `Animal`:
- **Covariant**: `list[Dog]` can be used where `list[Animal]` is expected (read-only containers).
- **Contravariant**: `Callable[[Animal], int]` can be used where `Callable[[Dog], int]` is expected (function arguments).
- **Invariant**: `MutableList[Dog]` cannot be used where `MutableList[Animal]` is expected (mutable containers).

### Purposes

- To understand when generic types can be safely substituted.
- To declare type variables with the correct variance for generic classes.
- To avoid type safety violations in mutable containers.
- To design generic APIs with correct subtyping behavior.
- To leverage covariant read-only collections and contravariant callables.

### Syntax Rules and Structure

#### Complete General Syntaxes

**Syntax 1: Declaring Variance with `TypeVar`**

```python
from typing import TypeVar, Generic

T_co = TypeVar('T_co', covariant=True)
T_contra = TypeVar('T_contra', contravariant=True)
T = TypeVar('T')  # Invariant (default)
```

**Component Breakdown**:
- `covariant=True` — declares covariance.
- `contravariant=True` — declares contravariance.
- No flag — invariant (default).

**Syntax 2: Using Variance in a Generic Class**

```python
from typing import TypeVar, Generic

T_co = TypeVar('T_co', covariant=True)

class ImmutableBox(Generic[T_co]):
    def __init__(self, value: T_co) -> None:
        self._value = value

    def get(self) -> T_co:
        return self._value
```

**Syntax 3: PEP 695 Variance Inference (Python 3.12+)**

```python
class ImmutableBox[T]:
    def __init__(self, value: T) -> None:
        self._value = value

    def get(self) -> T:
        return self._value
```

**Component Breakdown**:
- In PEP 695 syntax, variance is inferred automatically based on how the type parameter is used. Type checkers infer invariance, covariance, or contravariance depending on usage.

#### Syntax Rules

1. **Invariant is the default**: Manually created `TypeVar` instances are invariant unless `covariant=True` or `contravariant=True` is specified.
2. **Covariant type variables cannot be used as parameter types**: A covariant `T_co` can only appear in return positions, not parameter positions.
3. **Contravariant type variables cannot be used as return types**: A contravariant `T_contra` can only appear in parameter positions, not return positions.
4. **Mutable containers are invariant**: `list[T]`, `dict[K, V]`, `set[T]` are invariant.
5. **Read-only containers are covariant**: `Sequence[T_co]`, `Mapping[K, V_co]`, `frozenset[T_co]` are covariant.
6. **Callables are contravariant in parameter types and covariant in return types**: `Callable[[T_contra], T_co]`.
7. **PEP 695 infers variance**: In the new syntax, variance is inferred from usage, so explicit variance flags are not needed.
8. **Naming convention**: Covariant type variables are conventionally suffixed with `_co`; contravariant with `_contra`.

#### Constraints and Limitations

- Variance is only meaningful for generic classes; standalone generic functions do not have variance.
- PEP 695 variance inference requires Python 3.12+.
- Type checkers may not fully support all variance scenarios.
- Incorrect variance declarations can lead to type checker errors or unsoundness.

### Multiple Annotated Complete Step by Step Code Examples and Their Expected Outputs

#### Example 1: Covariance with Read-Only Sequences

```python
from typing import TypeVar, Generic
from collections.abc import Sequence

class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

class Dog(Animal):
    pass

def print_names(animals: Sequence[Animal]) -> None:
    for animal in animals:
        print(animal.name)

dogs: list[Dog] = [Dog("Rex"), Dog("Buddy")]

# Covariance: list[Dog] is a Sequence[Animal]
print_names(dogs)
```

**Expected Output**:
```
Rex
Buddy
```

**Why**: `Sequence` is covariant, so `Sequence[Dog]` is a subtype of `Sequence[Animal]`. A list of dogs can be used where a sequence of animals is expected.

#### Example 2: Contravariance with Callables

```python
from typing import Callable

class Animal:
    pass

class Dog(Animal):
    pass

def handle_animal(animal: Animal) -> None:
    print("Handling animal")

def handle_dog(dog: Dog) -> None:
    print("Handling dog")

# Contravariance: Callable[[Animal], None] is a subtype of Callable[[Dog], None]
handler: Callable[[Dog], None] = handle_animal  # OK: accepts a broader type
handler(Dog())

# The reverse would be a type error
# handler2: Callable[[Animal], None] = handle_dog  # Type error
```

**Expected Output**:
```
Handling animal
```

**Why**: A function that accepts any `Animal` can be used where a function that accepts only `Dog` is expected, because `Dog` is a subtype of `Animal`.

#### Example 3: Invariance with Mutable Containers

```python
from typing import TypeVar, Generic

T = TypeVar('T')  # Invariant by default

class MutableBox(Generic[T]):
    def __init__(self, value: T) -> None:
        self.value = value

    def set(self, value: T) -> None:
        self.value = value

    def get(self) -> T:
        return self.value

# Invariance: MutableBox[int] is NOT a subtype of MutableBox[object]
box_int: MutableBox[int] = MutableBox(42)
# box_obj: MutableBox[object] = box_int  # Type error
```

**Expected Output**: No output (type checker error at the commented line).

**Why**: Mutable containers are invariant because allowing `MutableBox[int]` to be used as `MutableBox[object]` would permit setting a string, violating the original `int` constraint.

### Real-World Cases

- **Read-only collections**: `Sequence`, `Mapping`, `frozenset` are covariant for safe read operations.
- **Callables**: Function signatures use contravariance for parameters and covariance for return types.
- **Mutable containers**: `list`, `dict`, `set` are invariant to prevent type safety violations.
- **Protocol design**: Protocols can be declared with covariant or contravariant type variables.

### References

- Variance of Generic Types - https://mypy.readthedocs.io/en/stable/generics.html#variance-of-generic-types
- PEP 484 – Type Hints (Variance) - https://peps.python.org/pep-0484/#covariance-and-contravariance
- PEP 695 – Type Parameter Syntax (Variance Inference) - https://peps.python.org/pep-0695/
- `typing.TypeVar` - https://docs.python.org/3/library/typing.html#typing.TypeVar

---

## References

- Protocols and Structural Subtyping - https://typing.python.org/en/latest/reference/protocols.html
- PEP 544 – Protocols: Structural subtyping - https://peps.python.org/pep-0544/
- Generics - https://typing.python.org/en/latest/spec/generics.html
- PEP 484 – Type Hints - https://peps.python.org/pep-0484/
- PEP 585 – Type Hinting Generics In Standard Collections - https://peps.python.org/pep-0585/
- PEP 647 – User-Defined Type Guards - https://peps.python.org/pep-0647/
- PEP 742 – Narrowing types with TypeIs - https://peps.python.org/pep-0742/
- PEP 695 – Type Parameter Syntax - https://peps.python.org/pep-0695/
- Type Narrowing - https://typing.python.org/en/latest/guides/type_narrowing.html
- `collections.abc` — Abstract Base Classes for Containers - https://docs.python.org/3/library/collections.abc.html
- mypy Generics Documentation - https://mypy.readthedocs.io/en/stable/generics.html
- Real Python: Python Protocols - https://realpython.com/python-protocol/
- `typing` — Support for type hints - https://docs.python.org/3/library/typing.html