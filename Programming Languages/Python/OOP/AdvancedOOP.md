# Advanced Object-Oriented Design — Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Advanced Object-Oriented Design encompasses the patterns, protocols, and mechanisms that enable robust, maintainable, and extensible object-oriented architectures in Python, including abstract interfaces, structural typing, composition strategies, and declarative class construction.

**Technical Definition:** These advanced techniques leverage Python's dynamic features—metaclasses, descriptors, the typing module, and the dataclasses module—to enforce contracts, decouple components, and reduce boilerplate. Key mechanisms include abstract base classes for nominal subtyping, `typing.Protocol` for structural subtyping, composition and dependency injection for loose coupling, mixins for behavioral reuse, and `@dataclass` for automatic method generation.

**Beginner-Friendly Explanation:** Basic OOP teaches you how to create classes and objects. Advanced OOP design teaches you the best ways to organize them: how to define clear interfaces, how to avoid rigid inheritance hierarchies, and how to write less repetitive code.

### Key Characteristics

- **Contract Enforcement:** ABCs and Protocols define required interfaces.
- **Loose Coupling:** Composition and dependency injection reduce dependencies between classes.
- **Code Reuse Without Inheritance:** Mixins and composition provide alternatives to deep hierarchies.
- **Declarative Construction:** `@dataclass` generates boilerplate methods automatically.
- **Immutability Support:** `frozen=True` creates hashable, read-only data objects.

### Prerequisites

- Solid understanding of classes, inheritance, and the MRO.
- Familiarity with decorators and the `typing` module.
- Knowledge of special methods (`__init__`, `__eq__`, `__hash__`).

### Related Programming Areas

- Framework and library design.
- API interface definition.
- Testing and dependency management.
- Domain modeling and data transfer objects.

---

## 1. Abstract Base Classes (ABCs via the `abc` Module and `@abstractmethod`)

### Definitions

**Core Definition:** Abstract Base Classes (ABCs) define interfaces that cannot be instantiated directly and require subclasses to implement specific abstract methods.

**Technical Definition:** An ABC is a class created using the `abc.ABC` base class or the `ABCMeta` metaclass. Methods decorated with `@abstractmethod` must be overridden in concrete subclasses; otherwise, instantiation raises `TypeError`. ABCs can also provide concrete implementations and register virtual subclasses .

**Beginner-Friendly Explanation:** An ABC is like a contract. It says "any class that wants to be this type must provide these methods." You can't create an object from the contract itself—only from classes that fulfill it.

### Purposes

- To define a common interface for a group of related classes.
- To enforce that certain methods must be implemented by subclasses.
- To support `isinstance()` and `issubclass()` checks for interface conformance.

### Syntax Rules and Structure

```python
from abc import ABC, abstractmethod

class AbstractClassName(ABC):
    @abstractmethod
    def required_method(self, args):
        """Must be implemented by subclasses."""
        pass
    
    def concrete_method(self):
        """Optional: default implementation."""
        return "default behavior"
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `ABC` | Base class from `abc` module |
| `@abstractmethod` | Marks a method as required |
| Concrete subclass | Must implement all abstract methods |

**Syntax Rules:**
- ABCs with unimplemented abstract methods cannot be instantiated.
- Subclasses must implement all abstract methods to become concrete.
- `register()` allows virtual subclasses without inheritance .

**Constraints and Limitations:**
- If an ABC defines no `@abstractmethod`, it can be instantiated (it's not truly abstract) .
- `__subclasshook__` can customize `issubclass()` behavior .

### Annotated Code Examples

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    """Abstract base class for shapes."""
    
    @abstractmethod
    def area(self):
        """Calculate area. Must be implemented."""
        pass
    
    @abstractmethod
    def perimeter(self):
        """Calculate perimeter. Must be implemented."""
        pass
    
    def description(self):
        """Concrete method: shared by all shapes."""
        return f"A shape with area {self.area():.2f}"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# Shape() would raise TypeError
rect = Rectangle(5, 3)
print(rect.description())

# Virtual subclass registration
class CustomShape:
    def area(self): return 100
    def perimeter(self): return 40

Shape.register(CustomShape)
print(f"CustomShape is Shape: {issubclass(CustomShape, Shape)}")
```

**Expected Output:**
```
A shape with area 15.00
CustomShape is Shape: True
```

**Why This Output:** `Shape` cannot be instantiated because it has abstract methods. `Rectangle` implements both, so it works. `register()` makes `CustomShape` a virtual subclass without inheritance .

### Real-World Cases

`collections.abc` provides ABCs like `Sequence`, `Mapping`, and `Iterable` that define standard container interfaces . Django's class-based views use ABCs for template views.

### References

- abc — Abstract Base Classes — https://docs.python.org/3/library/abc.html
- collections.abc — Abstract Base Classes for Containers — https://docs.python.org/3.10/library/collections.abc.html

---

## 2. Interfaces Through Protocols (Structural Subtyping via `typing.Protocol`)

### Definitions

**Core Definition:** Protocols define interfaces through structural subtyping, where a class conforms to a protocol by implementing the required methods, without explicit inheritance.

**Technical Definition:** `typing.Protocol` (PEP 544) enables static duck typing. A class is considered compatible with a protocol if it provides all protocol members with compatible signatures. At runtime, `@runtime_checkable` allows `isinstance()` checks (verifying only method names, not signatures) .

**Beginner-Friendly Explanation:** A protocol is like a job description: "anyone who can do these tasks qualifies." You don't need to officially register—just having the right skills is enough.

### Purposes

- To define interfaces without forcing inheritance.
- To enable static type checking with duck typing.
- To work with third-party classes that already have the right methods.

### Syntax Rules and Structure

```python
from typing import Protocol, runtime_checkable

class MyProtocol(Protocol):
    def required_method(self, arg: int) -> str:
        ...

@runtime_checkable
class CheckableProtocol(Protocol):
    def close(self) -> None:
        ...
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `Protocol` | Base class for structural interfaces |
| Method definitions | Signatures only, no implementation required |
| `@runtime_checkable` | Enables `isinstance()` checks (name-only) |

**Syntax Rules:**
- Classes don't inherit from the protocol; they just implement its methods.
- Protocol members are declared in the class body.
- Use `ClassVar` for class attributes in protocols .

**Constraints and Limitations:**
- `@runtime_checkable` `isinstance()` only checks method names, not signatures .
- Runtime checks can produce false positives.

### Annotated Code Examples

```python
from typing import Protocol, runtime_checkable

class SupportsClose(Protocol):
    """Anything with a close() method qualifies."""
    def close(self) -> None:
        ...

class FileResource:
    def close(self) -> None:
        print("File closed")

class LockResource:
    def close(self) -> None:
        print("Lock released")

def close_all(resources: list[SupportsClose]) -> None:
    for r in resources:
        r.close()

# Both work without inheritance
close_all([FileResource(), LockResource()])
```

**Expected Output:**
```
File closed
Lock released
```

**Why This Output:** `FileResource` and `LockResource` both have `close()` methods, so they satisfy `SupportsClose` structurally. No inheritance is required .

### Real-World Cases

Protocols are ideal for defining plugin interfaces, callback signatures, and repository patterns where external code should be able to conform without modifying its class hierarchy .

### References

- PEP 544 — Protocols: Structural Subtyping — https://peps.python.org/pep-0544/
- typing.Protocol Documentation — https://docs.python.org/3/library/typing.html#typing.Protocol

---

## 3. Composition vs. Inheritance

### Definitions

**Core Definition:** Composition builds classes by combining objects (has-a relationships), while inheritance builds classes by extending existing classes (is-a relationships).

**Technical Definition:** Composition delegates behavior by holding references to other objects and calling their methods. Inheritance acquires attributes and methods through the MRO. Composition provides greater flexibility and avoids the fragility of deep inheritance hierarchies .

**Beginner-Friendly Explanation:** Inheritance says "I am a kind of X." Composition says "I have an X that does this for me." Composition is often more flexible.

### Purposes

- To reuse behavior without creating rigid hierarchies.
- To enable changing behavior at runtime by swapping components.
- To avoid the diamond problem and MRO complications.

### Syntax Rules and Structure

```python
# Composition
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()  # Has-a relationship
    
    def start(self):
        return self.engine.start()  # Delegation
```

**Component Breakdown:**
| Pattern | Relationship | Mechanism |
|---------|--------------|-----------|
| Inheritance | is-a | `class Child(Parent)` |
| Composition | has-a | `self.component = Component()` |

**Syntax Rules:**
- Composition uses instance attributes to hold component objects.
- Delegation calls methods on the component.
- Components can be swapped at runtime.

**Constraints and Limitations:**
- Composition requires more explicit delegation code.
- Inheritance is simpler for clear "is-a" relationships.

### Annotated Code Examples

```python
# Inheritance approach (rigid)
class Animal:
    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):
        return "Woof"

# Composition approach (flexible)
class Speaker:
    def __init__(self, sound):
        self.sound = sound
    
    def speak(self):
        return self.sound

class AnimalComposed:
    def __init__(self, speaker):
        self.speaker = speaker  # Has-a Speaker
    
    def speak(self):
        return self.speaker.speak()

# Can change behavior at runtime
dog = AnimalComposed(Speaker("Woof"))
print(dog.speak())

dog.speaker = Speaker("Meow")  # Swap behavior
print(dog.speak())
```

**Expected Output:**
```
Woof
Meow
```

**Why This Output:** The composed `Animal` delegates to its `speaker` attribute. Changing the `speaker` changes behavior without creating a new class .

### Real-World Cases

A `Car` class composes `Engine`, `Transmission`, and `Wheel` objects rather than inheriting from all of them. Django models use composition extensively with field objects.

### References

- Python-ideas: Composition over Inheritance — https://mail.python.org/archives/list/python-ideas@python.org/message/LFLUFUOEBJQIYFWC222X7GFDC4IKAYUW/
- Tutor List: Class Relationships — https://mail.python.org/pipermail/tutor/2011-November/086572.html

---

## 4. Aggregation

### Definitions

**Core Definition:** Aggregation is a specialized form of association where one object contains references to other objects, but the contained objects can exist independently.

**Technical Definition:** In aggregation, the container object holds references to components, but does not control their lifecycle. The components can outlive the container or be shared with other containers. This differs from composition, where the container owns and manages the component's lifecycle .

**Beginner-Friendly Explanation:** Aggregation is like a car and its tires. The car has tires, but the tires can be removed and used elsewhere. The car doesn't "own" the tires in a lifecycle sense.

### Purposes

- To model "has-a" relationships where components are shared or independent.
- To enable flexible lifecycle management.
- To represent real-world relationships where components can exist independently.

### Syntax Rules and Structure

```python
class Tire:
    def __init__(self, brand):
        self.brand = brand

class Car:
    def __init__(self, tires):
        self.tires = tires  # Aggregation: references, no ownership
```

**Component Breakdown:**
| Aspect | Aggregation | Composition |
|--------|-------------|-------------|
| Lifecycle | Independent | Controlled by container |
| Sharing | Possible | Typically exclusive |
| Reference | External | Created internally |

**Syntax Rules:**
- Aggregation is implemented by accepting component objects as parameters.
- The container does not create the components internally.
- Components can be passed to multiple containers.

**Constraints and Limitations:**
- Python doesn't distinguish aggregation from composition syntactically—it's about intent .
- Both are implemented as object references.

### Annotated Code Examples

```python
class Department:
    def __init__(self, name):
        self.name = name

class University:
    def __init__(self, departments):
        # Aggregation: departments passed in, can exist independently
        self.departments = departments
    
    def list_departments(self):
        return [d.name for d in self.departments]

# Departments created independently
dept1 = Department("Computer Science")
dept2 = Department("Mathematics")

# University aggregates them
uni = University([dept1, dept2])
print(uni.list_departments())

# Departments still exist independently
print(f"Dept1 still exists: {dept1.name}")
```

**Expected Output:**
```
['Computer Science', 'Mathematics']
Dept1 still exists: Computer Science
```

**Why This Output:** The departments are created outside the university and passed in. They are not destroyed when the university is destroyed. This is aggregation .

### Real-World Cases

A `Playlist` aggregates `Song` objects that can also exist in other playlists. A `Team` aggregates `Player` objects that exist independently.

### References

- Python List: Composition and Aggregation — https://mail.python.org/pipermail/python-list/2011-November/765157.html
- Tutor List: Object Lifecycle — https://mail.python.org/pipermail/tutor/2011-November/086572.html

---

## 5. Dependency Injection

### Definitions

**Core Definition:** Dependency injection is a design pattern where an object's dependencies are provided externally rather than created internally, reducing coupling and improving testability.

**Technical Definition:** In Python, dependency injection is typically implemented via constructor parameters, setter methods, or dedicated DI containers/frameworks. The object declares what it needs, and the caller (or container) supplies the actual instances .

**Beginner-Friendly Explanation:** Instead of a class creating its own tools, you hand it the tools it needs. This makes it easy to swap tools for testing or different configurations.

### Purposes

- To decouple classes from concrete implementations.
- To enable easy testing with mock dependencies.
- To centralize configuration and object creation.

### Syntax Rules and Structure

```python
# Constructor injection (most common)
class Service:
    def __init__(self, dependency):
        self.dependency = dependency

# The caller provides the dependency
dep = SomeDependency()
service = Service(dep)
```

**Component Breakdown:**
| Injection Type | Mechanism |
|----------------|-----------|
| Constructor | Passed to `__init__` |
| Setter | Assigned via method |
| Container | Framework resolves and injects |

**Syntax Rules:**
- Dependencies are passed as parameters, not created internally.
- Use abstract types (ABCs or Protocols) for the dependency parameter.
- DI containers can automate resolution for complex graphs .

**Constraints and Limitations:**
- Adds indirection; simpler cases may not need it.
- Overuse can make code harder to trace.

### Annotated Code Examples

```python
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message):
        pass

class ConsoleLogger(Logger):
    def log(self, message):
        print(f"[CONSOLE] {message}")

class FileLogger(Logger):
    def log(self, message):
        print(f"[FILE] {message}")

class UserService:
    def __init__(self, logger: Logger):
        self.logger = logger  # Injected dependency
    
    def create_user(self, name):
        self.logger.log(f"Creating user: {name}")
        return {"name": name}

# Inject different loggers
service1 = UserService(ConsoleLogger())
service1.create_user("Alice")

service2 = UserService(FileLogger())
service2.create_user("Bob")
```

**Expected Output:**
```
[CONSOLE] Creating user: Alice
[FILE] Creating user: Bob
```

**Why This Output:** `UserService` doesn't create its logger—it receives one. Different logger implementations can be injected without changing `UserService` .

### Real-World Cases

Django's class-based views inject `request` and `args` into methods. Flask uses dependency injection for database sessions and configuration. Frameworks like `classic-container` and `pybeandi` provide DI containers .

### References

- classic-container — https://pypi.org/project/classic-container/
- pybeandi — https://pypi.org/project/pybeandi/0.2.1/

---

## 6. Mixins (Design Patterns and Limitations)

### Definitions

**Core Definition:** A mixin is a class that provides methods for use by other classes without being intended for instantiation, typically used to add functionality via multiple inheritance.

**Technical Definition:** Mixins are small classes that encapsulate reusable behavior. They rely on cooperative multiple inheritance using `super()` to chain method calls. They should not have their own `__init__` or instance state to avoid conflicts .

**Beginner-Friendly Explanation:** A mixin is like a "feature pack" you can add to a class. It's not meant to stand alone—just to give extra abilities to other classes.

### Purposes

- To add reusable behavior to multiple unrelated classes.
- To avoid code duplication across class hierarchies.
- To compose features flexibly without deep inheritance.

### Syntax Rules and Structure

```python
class MyMixin:
    def extra_method(self):
        # Behavior to mix in
        super().extra_method()  # Cooperative call
```

**Component Breakdown:**
| Rule | Description |
|------|-------------|
| No `__init__` | Avoid state conflicts |
| Use `super()` | Cooperative MRO chaining |
| Place before base | Mixins usually go first in inheritance list |

**Syntax Rules:**
- Mixins should call `super()` to continue the MRO chain.
- Mixin order matters: place mixins before concrete classes .
- Avoid instance attributes in mixins unless carefully designed.

**Constraints and Limitations:**
- Mixins break if the next class in MRO doesn't call `super()` .
- Mixin methods can be shadowed if placed incorrectly in the MRO.
- Name clashes between mixins can cause issues.

### Annotated Code Examples

```python
class LoggingMixin:
    """Adds logging to method calls."""
    def process(self, data, **kwargs):
        print(f"[LOG] Processing: {data}")
        result = super().process(data, **kwargs)
        print(f"[LOG] Result: {result}")
        return result

class ValidationMixin:
    """Validates data before processing."""
    def process(self, data, **kwargs):
        if not data:
            raise ValueError("Data cannot be empty")
        print(f"[VALID] {data} is valid")
        return super().process(data, **kwargs)

class BaseProcessor:
    def process(self, data, **kwargs):
        return f"Processed: {data}"

class DataProcessor(ValidationMixin, LoggingMixin, BaseProcessor):
    """Combines validation, logging, and processing."""
    pass

processor = DataProcessor()
result = processor.process("Hello")
print(f"Final: {result}")
```

**Expected Output:**
```
[VALID] Hello is valid
[LOG] Processing: Hello
[LOG] Result: Processed: Hello
Final: Processed: Hello
```

**Why This Output:** The MRO is `DataProcessor → ValidationMixin → LoggingMixin → BaseProcessor`. Each mixin calls `super().process()`, passing control along the chain until `BaseProcessor` returns the result .

### Real-World Cases

Django's `LoginRequiredMixin` and `PermissionRequiredMixin` add authentication checks to views. Python's `socketserver` uses mixins for threading and forking behaviors.

### References

- Real Python: What Are Mixin Classes? — https://realpython.com/python-mixin/
- Educative: Multiple Inheritance and Mixins — https://www.educative.io/courses/building-robust-object-oriented-python-apps-and-libraries/lta/multiple-inheritance

---

## 7. Data Classes (`@dataclass`, Field Customization, and `__post_init__`)

### Definitions

**Core Definition:** `@dataclass` is a decorator that automatically generates boilerplate methods (`__init__`, `__repr__`, `__eq__`) for classes primarily used to store data.

**Technical Definition:** The `dataclasses.dataclass` decorator inspects class-level annotated attributes and generates special methods based on configuration options (`init`, `repr`, `eq`, `order`, `frozen`, `slots`). The `field()` function customizes individual fields, and `__post_init__` provides a hook for post-initialization processing .

**Beginner-Friendly Explanation:** `@dataclass` writes your `__init__` and other boring methods for you. You just declare the fields and their types.

### Purposes

- To reduce boilerplate for data-holding classes.
- To provide automatic equality, representation, and ordering.
- To support immutable and hashable value objects.

### Syntax Rules and Structure

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int = 0  # Default value
    z: list = field(default_factory=list)  # Mutable default
    
    def __post_init__(self):
        """Called after __init__."""
        if self.x < 0:
            raise ValueError("x must be non-negative")
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `@dataclass` | Generates `__init__`, `__repr__`, `__eq__` |
| `field()` | Customizes individual fields |
| `default_factory` | Provides mutable defaults safely |
| `__post_init__` | Hook after initialization |

**Syntax Rules:**
- Fields are declared as annotated class attributes.
- Mutable defaults must use `field(default_factory=...)`.
- `__post_init__` runs after the generated `__init__` .

**Constraints and Limitations:**
- Generated methods can be overridden or disabled.
- `order=True` requires `eq=True`.
- Inheritance with dataclasses requires careful field ordering.

### Annotated Code Examples

```python
from dataclasses import dataclass, field

@dataclass
class Employee:
    name: str
    department: str
    salary: float = 50000.0
    skills: list[str] = field(default_factory=list)
    
    def __post_init__(self):
        """Validate after initialization."""
        if self.salary < 0:
            raise ValueError("Salary cannot be negative")
        # Normalize name
        self.name = self.name.strip().title()

# Usage
emp = Employee("  alice smith  ", "Engineering")
print(emp)
print(f"Name normalized: {emp.name}")

# Equality works automatically
emp2 = Employee("Alice Smith", "Engineering")
print(f"Equal? {emp == emp2}")

# Add skills
emp.skills.append("Python")
print(f"Skills: {emp.skills}")
```

**Expected Output:**
```
Employee(name='Alice Smith', department='Engineering', salary=50000.0, skills=[])
Name normalized: Alice Smith
Equal? True
Skills: ['Python']
```

**Why This Output:** `@dataclass` generates `__init__` and `__repr__`. `__post_init__` normalizes the name. `__eq__` compares all fields, so `emp` and `emp2` are equal .

### Real-World Cases

Dataclasses are used for DTOs, configuration objects, and domain entities. They are common in FastAPI for request/response models and in testing for expected values.

### References

- dataclasses — Data Classes — https://docs.python.org/3/library/dataclasses.html
- Fluent Python: Data Class Builders — https://archive.org/download/traitlmentaired00serrgoog_202205/Fluent-Python-Clear_-Concise-and-Effective-Programming_-2nd-Edition-_Luciano-Ramalho_-_z-lib.org_.pdf

---

## 8. Immutable Data Classes (`frozen=True`)

### Definitions

**Core Definition:** A frozen dataclass is an immutable dataclass where attribute assignment after creation raises `FrozenInstanceError`.

**Technical Definition:** `@dataclass(frozen=True)` generates `__setattr__` and `__delattr__` methods that raise exceptions on modification. It also enables `__hash__` generation when `eq=True`, making instances hashable and usable in sets and as dictionary keys .

**Beginner-Friendly Explanation:** A frozen dataclass is like a read-only record. Once created, you can't change it. This makes it safe to share and use as dictionary keys.

### Purposes

- To create immutable value objects.
- To enable hashability for use in sets and dictionaries.
- To prevent accidental modification of shared data.

### Syntax Rules and Structure

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
```

**Component Breakdown:**
| Aspect | Behavior |
|--------|----------|
| `frozen=True` | Blocks attribute assignment |
| `__hash__` | Auto-generated when `eq=True` |
| `__setattr__` | Raises `FrozenInstanceError` |

**Syntax Rules:**
- Frozen dataclasses are hashable if `eq=True` (default).
- Use `object.__setattr__` to bypass in `__post_init__` if needed.
- Inheritance from non-frozen to frozen is allowed; reverse is not.

**Constraints and Limitations:**
- Cannot modify fields after creation.
- Mutable field values (like lists) can still be mutated internally .

### Annotated Code Examples

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Color:
    red: int
    green: int
    blue: int
    
    def __post_init__(self):
        # Can use object.__setattr__ if absolutely needed
        if not all(0 <= c <= 255 for c in (self.red, self.green, self.blue)):
            raise ValueError("Color values must be 0-255")

# Create instances
red = Color(255, 0, 0)
blue = Color(0, 0, 255)

print(red)
print(f"Hashable: {hash(red)}")

# Can be used in sets
colors = {red, blue, Color(255, 0, 0)}
print(f"Set size: {len(colors)}")  # 2 (red appears twice)

# Cannot modify
try:
    red.red = 128
except Exception as e:
    print(f"Error: {type(e).__name__}")
```

**Expected Output:**
```
Color(red=255, green=0, blue=0)
Hashable: 8665453637427
Set size: 2
Error: FrozenInstanceError
```

**Why This Output:** `frozen=True` makes the dataclass hashable and immutable. Equal instances hash the same, so the set contains only unique colors. Assignment raises `FrozenInstanceError` .

### Real-World Cases

Frozen dataclasses are ideal for configuration objects, coordinate points, API tokens, and any value object that should be immutable. They are frequently used as dictionary keys and set members.

### References

- dataclasses — Frozen Instances — https://docs.python.org/3/library/dataclasses.html#frozen-instances
- Python-Dev: frozen=True discussion — https://mail.python.org/pipermail/python-dev/attachments/20180206/3b1133a7/attachment.html

---

## Master Reference List

- abc — Abstract Base Classes — https://docs.python.org/3/library/abc.html
- collections.abc — Container ABCs — https://docs.python.org/3.10/library/collections.abc.html
- PEP 544 — Protocols: Structural Subtyping — https://peps.python.org/pep-0544/
- typing.Protocol Documentation — https://docs.python.org/3/library/typing.html#typing.Protocol
- Real Python: What Are Mixin Classes? — https://realpython.com/python-mixin/
- Real Python: Python Protocols — https://realpython.com/python-protocol/
- dataclasses — Data Classes — https://docs.python.org/3/library/dataclasses.html
- Prefer Protocol Over ABC — https://github.com/ludo-technologies/python-best-practices
- classic-container — https://pypi.org/project/classic-container/
- pybeandi — https://pypi.org/project/pybeandi/0.2.1/