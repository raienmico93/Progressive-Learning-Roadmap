# Python Inheritance: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Inheritance is a fundamental object-oriented programming mechanism in Python that allows a new class (the **derived class**) to acquire the attributes and methods of an existing class (the **base class**). It models an **“is-a”** relationship and enables code reuse, specialization, and polymorphism.

**Technical Definition**

In Python, inheritance is implemented by listing base classes in parentheses in the class definition header: `class Derived(Base):`. The derived class inherits the namespace of its bases and can override methods, extend behavior by calling `super()`, and add new attributes. Python supports single and multiple inheritance, with method resolution governed by the **C3 linearization algorithm** that produces a deterministic Method Resolution Order (MRO). The `abc` module provides **Abstract Base Classes (ABCs)** to enforce interface contracts, and built-in functions `isinstance()` and `issubclass()` allow runtime type checking against inheritance hierarchies.

**Beginner-Friendly Explanation**

Think of inheritance as a family tree for classes. A **base class** (parent) defines common traits and behaviors. A **derived class** (child) inherits all of them and can add its own special traits or change inherited ones. For example, a `Dog` class can inherit from an `Animal` class: every dog **is an** animal, so it automatically gets the `eat()` and `sleep()` methods, and it can add its own `bark()` method. This saves you from rewriting the same code for every type of animal.

### Key Characteristics

- **“Is-a” relationship**: Inheritance models specialization, not composition.
- **Code reuse**: Derived classes automatically receive base class attributes and methods.
- **Method overriding**: A derived class can replace an inherited method with its own implementation.
- **Cooperative multiple inheritance**: `super()` delegates method calls along the MRO, enabling clean diamond-shaped hierarchies.
- **C3 linearization**: Python computes a deterministic MRO for every class, ensuring each ancestor appears exactly once.
- **Abstract contracts**: The `abc` module allows defining interfaces that subclasses must implement.
- **Runtime introspection**: `isinstance()` and `issubclass()` support polymorphic dispatch and validation.

### Prerequisites

- Basic Python syntax: classes, objects, and methods.
- Understanding of attributes and instance state.
- Familiarity with the `self` parameter and method definitions.
- Awareness of Python’s dynamic typing model.

### Related Programming Areas

- **Polymorphism**: Inheritance enables a single interface to represent different underlying types.
- **Design patterns**: Template method, factory, and strategy patterns rely on inheritance.
- **Abstract base classes**: `collections.abc` and custom ABCs define formal interfaces.
- **Mixins**: Small classes that add specific behavior through multiple inheritance.
- **Frameworks**: Django models, pytest fixtures, and GUI toolkits use inheritance for extension hooks.

---

## Core Concepts / Features

---

### 1. Inheritance, Base Classes, and Derived Classes

#### Definitions

**Core Definition**

**Inheritance** is the mechanism by which a **derived class** (subclass) acquires the attributes and methods of a **base class** (superclass). The derived class **is a** specialized version of the base class.

**Technical Definition**

A base class is any class defined with `class Base:`. A derived class is defined as `class Derived(Base):`, which places `Base` in the derived class’s MRO and makes all of `Base`’s attributes accessible on `Derived` instances. The derived class can override methods, call the base implementation via `super()`, and add new members. In Python, classes that inherit from another are called **derived classes**, **subclasses**, or **subtypes**; classes from which others are derived are **base classes** or **superclasses**.

**Beginner-Friendly Explanation**

A base class is a general template. A derived class is a specialized version of that template. If `Animal` is the base class, then `Dog` and `Cat` are derived classes. They automatically know how to `eat()` and `sleep()` because `Animal` defined those, but they can also have their own `bark()` or `meow()` methods.

#### Purposes

- To reuse common code across related classes without duplication.
- To model hierarchical “is-a” relationships in a domain.
- To enable polymorphism: code written against a base class works with any derived class.
- To support specialization: derived classes tailor or extend base behavior.
- To organize code into clear, extensible hierarchies.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Base:
    """Base class with common behavior."""
    def common_method(self):
        return "common"

class Derived(Base):
    """Derived class inheriting from Base."""
    def specialized_method(self):
        return "specialized"
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class Base:` | Defines the base class. |
| `class Derived(Base):` | Defines a derived class; `Base` is the base class. |
| Inherited members | Available on derived instances without redefinition. |
| Overridden members | Redefined in the derived class to replace the base version. |

**Syntax Rules**

- Base classes are listed in parentheses in the class header.
- Multiple base classes are separated by commas.
- The derived class inherits all public and “protected” (underscore-prefixed) members.
- Name-mangled (`__private`) members are not directly accessible in subclasses.
- Method overriding occurs when a derived class defines a method with the same name as a base class method.

**Constraints and Limitations**

- Deep inheritance hierarchies become hard to understand and maintain.
- Overriding methods without calling `super()` can break base-class invariants.
- Python does not enforce interface contracts unless ABCs are used.
- Inheritance is inappropriate for “has-a” relationships; composition should be preferred there.

#### Annotated Code Examples

**Example 1: Basic Inheritance**

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

class Dog(Animal):
    def speak(self):
        return f"{self.name} says woof!"

d = Dog("Rex")
print(d.name)       # Rex (inherited attribute)
print(d.speak())    # Rex says woof! (overridden method)
```

**Expected Output:**

```
Rex
Rex says woof!
```

**Step-by-Step Breakdown:**

1. `Animal` defines `__init__` and `speak`.
2. `Dog` inherits both and overrides `speak`.
3. `Dog("Rex")` calls the inherited `__init__` implicitly.
4. `d.speak()` resolves to `Dog.speak` because of the MRO.

**Why the Output is Produced:**

The derived class inherits the base class’s `__init__` and can override its methods. Attribute lookup checks the instance, then the derived class, then the base class.

#### Real-World Cases

- **Django models**: `models.Model` is a base class; every model inherits database behavior.
- **GUI toolkits**: Custom widgets inherit from `tkinter.Frame` or `PyQt.QWidget`.
- **Machine learning**: Custom estimators inherit from `sklearn.base.BaseEstimator`.
- **Exceptions**: Custom exceptions inherit from `Exception` or more specific built-ins.

---

### 2. `super()`

#### Definitions

**Core Definition**

`super()` returns a proxy object that delegates method calls to the **next class in the Method Resolution Order (MRO)** , enabling cooperative method invocation in inheritance hierarchies.

**Technical Definition**

`super()` returns a proxy object bound to the current class and instance, configured to delegate method lookups to whatever class comes **next** in the MRO. In single inheritance, this is simply the parent class. In multiple inheritance, it may be a **sibling** class that does not appear in the direct parent list. The no-argument form `super()` is preferred in Python 3; the explicit form `super(ClassName, self)` is considered error-prone. Cooperative multiple inheritance requires that every class in the hierarchy calls `super()` so that each ancestor’s method runs exactly once.

**Beginner-Friendly Explanation**

`super()` is your way of saying “now run the version of this method from the next class in line.” It is most commonly used in `__init__` to let the parent class set up its attributes before the child adds its own. In multiple inheritance, `super()` follows a precise order (the MRO) so that every ancestor gets its turn, and no ancestor runs twice.

#### Purposes

- To call a parent class’s method without hard-coding the parent’s name.
- To ensure that base-class initialization runs in derived classes.
- To enable cooperative multiple inheritance where each ancestor’s method executes exactly once.
- To make code resilient to changes in the class hierarchy.
- To support the cooperative method-calling pattern popularized by Raymond Hettinger.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Base:
    def method(self):
        ...

class Derived(Base):
    def method(self):
        super().method()  # calls Base.method
        ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `super()` | Returns a proxy bound to the current class and instance. |
| `super().method()` | Delegates to the next implementation in the MRO. |
| `super().__init__()` | Calls the next `__init__` in the MRO. |

**Syntax Rules**

- Use the no-argument form `super()` inside instance methods.
- `super()` can be used in class methods with `super().classmethod_name()`.
- The proxy respects the MRO computed for the **instance’s actual type**, not the lexical class.
- In cooperative inheritance, every class in the hierarchy should call `super()` consistently.

**Constraints and Limitations**

- `super()` cannot be used outside a method that has access to `self` or `cls`.
- In multiple inheritance, `super()` may delegate to a sibling class, not a direct parent.
- Signatures must be compatible across cooperative classes; using `**kwargs` is a common pattern.
- Forgetting to call `super().__init__()` can leave base attributes uninitialized.

#### Annotated Code Examples

**Example 1: `super()` in Single Inheritance**

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)  # delegate name setup to Animal
        self.breed = breed

d = Dog("Rex", "Labrador")
print(d.name, d.breed)
```

**Expected Output:**

```
Rex Labrador
```

**Step-by-Step Breakdown:**

1. `Dog.__init__` calls `super().__init__(name)`, which runs `Animal.__init__`.
2. `self.name` is set by the base class.
3. `Dog` then adds `self.breed`.

**Why the Output is Produced:**

`super()` delegates to `Animal.__init__` according to the MRO, ensuring base initialization runs before derived specialization.

---

**Example 2: Cooperative Multiple Inheritance with `super()`**

```python
class A:
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        print("A initialized")

class B(A):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        print("B initialized")

class C(A):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        print("C initialized")

class D(B, C):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        print("D initialized")

d = D()
```

**Expected Output:**

```
A initialized
C initialized
B initialized
D initialized
```

**Step-by-Step Breakdown:**

1. `D` calls `super().__init__()`, which goes to `B` (first in MRO).
2. `B` calls `super().__init__()`, which goes to `C` (next in MRO).
3. `C` calls `super().__init__()`, which goes to `A` (next in MRO).
4. `A` calls `super().__init__()`, which reaches `object` and stops.
5. The print statements execute in reverse order of the call stack.

**Why the Output is Produced:**

Cooperative `super()` calls traverse the MRO, ensuring each class initializes exactly once, even in a diamond hierarchy.

#### Real-World Cases

- **Framework base classes**: Calling `super().__init__()` in Django models, forms, and views.
- **Mixins**: Cooperative mixins that add behavior via `super()` in the MRO chain.
- **Exception hierarchies**: Custom exceptions that delegate to `Exception.__init__`.

---

### 3. Method Resolution Order (MRO)

#### Definitions

**Core Definition**

The **Method Resolution Order (MRO)** is the linear list of classes that Python computes for every class, determining the order in which base classes are searched when resolving an attribute or method.

**Technical Definition**

Python computes the MRO using the **C3 linearization algorithm**, which produces a deterministic ordering that satisfies three constraints: (1) every class appears exactly once; (2) parent classes appear after their children; and (3) the order respects the left-to-right order of base classes. The MRO is accessible via `ClassName.__mro__` or `ClassName.mro()`. It is central to understanding `super()` behavior and multiple inheritance.

**Beginner-Friendly Explanation**

The MRO is the “search order” Python uses when you access an attribute. If you have a class `D(B, C)`, Python checks `D` first, then `B`, then `C`, then `A` (their common base), and so on. This order is calculated automatically and guarantees that the same method is not accidentally called twice in a diamond hierarchy.

#### Purposes

- To provide a deterministic attribute lookup order in single and multiple inheritance.
- To define the delegation path for `super()`.
- To prevent ambiguity and conflicts in diamond-shaped hierarchies.
- To allow introspection of a class’s inheritance structure at runtime.
- To support the C3 linearization guarantees that make cooperative multiple inheritance possible.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class D(B, C):
    ...

print(D.__mro__)      # tuple of classes
print(D.mro())        # list of classes
```

**Component Breakdown**

| Access Method | Description |
|---------------|-------------|
| `D.__mro__` | Tuple of classes in resolution order. |
| `D.mro()` | Method returning the MRO as a list. |
| `super()` | Delegates to the next class in the MRO. |

**Syntax Rules**

- The MRO always begins with the class itself and ends with `object`.
- C3 linearization ensures that a class appears before its parents and that the left-to-right order of bases is preserved.
- An inconsistent hierarchy (where C3 cannot produce a valid MRO) raises `TypeError` at class definition time.
- `__mro__` is a read-only tuple.

**Constraints and Limitations**

- The MRO can be surprising in complex multiple-inheritance graphs; always inspect it when debugging.
- Cooperative `super()` requires that all classes in the MRO cooperate; a class that does not call `super()` breaks the chain.
- Deep hierarchies make the MRO long and hard to reason about.

#### Annotated Code Examples

**Example 1: Inspecting the MRO**

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

for cls in D.__mro__:
    print(cls.__name__)
```

**Expected Output:**

```
D
B
C
A
object
```

**Step-by-Step Breakdown:**

1. `D` is the class itself.
2. `B` comes before `C` because `D(B, C)` lists `B` first.
3. `A` appears after both `B` and `C` because it is their common parent.
4. `object` is the ultimate base.

**Why the Output is Produced:**

C3 linearization merges the MROs of `B` and `C` while preserving the local precedence order of `D`’s bases.

---

**Example 2: Diamond Hierarchy and MRO**

```python
class A:
    def method(self): return "A"
class B(A):
    def method(self): return "B" + super().method()
class C(A):
    def method(self): return "C" + super().method()
class D(B, C):
    def method(self): return "D" + super().method()

print(D().method())
print(D.__mro__)
```

**Expected Output:**

```
DCBA
(<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

**Step-by-Step Breakdown:**

1. `D.method` calls `super().method()`, which resolves to `B.method` next in the MRO.
2. `B.method` calls `super().method()`, which resolves to `C.method`.
3. `C.method` calls `super().method()`, which resolves to `A.method`.
4. `A.method` returns `"A"`, and the results concatenate back up the chain.

**Why the Output is Produced:**

The MRO `D → B → C → A` ensures each class’s `method` runs exactly once, in the correct order, despite the diamond shape.

#### Real-World Cases

- **Mixins in frameworks**: Understanding MRO is essential when combining multiple mixins.
- **Debugging attribute errors**: Inspecting `__mro__` reveals why a method resolves the way it does.
- **ABC registration**: Virtual subclasses are not inserted into the MRO, which affects `super()` behavior.

---

### 4. Multiple Inheritance and the Diamond Problem

#### Definitions

**Core Definition**

**Multiple inheritance** allows a class to inherit from more than one base class. The **diamond problem** is the ambiguity that arises when two base classes share a common ancestor, and a derived class inherits from both—Python solves this via C3 linearization.

**Technical Definition**

In Python, `class D(B, C):` creates a class with two direct bases. If `B` and `C` both inherit from `A`, the hierarchy forms a diamond. C3 linearization ensures that `A` appears exactly once in the MRO, after both `B` and `C`, and that `super()` calls traverse the MRO cooperatively so that each ancestor’s method runs exactly once. Python’s MRO solves the diamond problem by guaranteeing a consistent, deterministic order.

**Beginner-Friendly Explanation**

Multiple inheritance means a class can have more than one parent. The diamond problem is what happens when two parents share the same grandparent: which version of the grandparent’s method should run, and how many times? Python answers this with the MRO, which makes sure the grandparent’s method runs exactly once, in a predictable order.

#### Purposes

- To combine behaviors from multiple independent base classes (mixins).
- To model complex domain relationships that genuinely involve multiple “is-a” roles.
- To enable cooperative mixins that add orthogonal features.
- To leverage C3 linearization for unambiguous method resolution.
- To understand and avoid the pitfalls of overly complex hierarchies.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class A:
    ...
class B(A):
    ...
class C(A):
    ...
class D(B, C):
    ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `class D(B, C):` | Multiple inheritance; MRO determines lookup order. |
| Diamond shape | `B` and `C` share a common ancestor `A`. |
| C3 linearization | Produces an MRO where `A` appears once, after `B` and `C`. |

**Syntax Rules**

- Base classes are listed left-to-right; earlier bases take precedence.
- All classes in the hierarchy should cooperate with `super()` for predictable behavior.
- The MRO is computed at class definition time; inconsistent hierarchies raise `TypeError`.
- `super()` in a diamond hierarchy delegates to the next class in the MRO, which may be a sibling.

**Constraints and Limitations**

- Multiple inheritance increases complexity and should be used sparingly.
- Non-cooperative classes that do not call `super()` can break the chain.
- Deep or wide hierarchies make the MRO difficult to reason about.
- The diamond problem is solved for method resolution, but state initialization still requires careful cooperative design.

#### Annotated Code Examples

**Example 1: Diamond with Cooperative `super()`**

```python
class A:
    def __init__(self):
        print("A")
        super().__init__()

class B(A):
    def __init__(self):
        print("B")
        super().__init__()

class C(A):
    def __init__(self):
        print("C")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D")
        super().__init__()

D()
```

**Expected Output:**

```
D
B
C
A
```

**Step-by-Step Breakdown:**

1. `D.__init__` prints “D” and calls `super().__init__()`, which goes to `B`.
2. `B.__init__` prints “B” and calls `super().__init__()`, which goes to `C`.
3. `C.__init__` prints “C” and calls `super().__init__()`, which goes to `A`.
4. `A.__init__` prints “A” and calls `super().__init__()`, which reaches `object`.

**Why the Output is Produced:**

Cooperative `super()` traverses the MRO `D → B → C → A`, ensuring each class’s `__init__` runs exactly once, even though `B` and `C` share `A` as a parent.

#### Real-World Cases

- **Mixins**: Combining `LoggingMixin`, `SerializationMixin`, and `ValidationMixin` into a single class.
- **Framework extension**: Django’s class-based views combine multiple mixins.
- **Game development**: Combining movement, weapon, and armor behaviors without a combinatorial explosion of subclasses.

---

### 5. `isinstance()` and `issubclass()`

#### Definitions

**Core Definition**

`isinstance(obj, cls)` checks whether an **object** is an instance of a class or a tuple of classes, respecting inheritance. `issubclass(sub, cls)` checks whether a **class** is a subclass of another class.

**Technical Definition**

`isinstance(obj, cls)` returns `True` if `obj` is an instance of `cls` or any subclass thereof, or if `cls` is an ABC and `obj` is registered as a virtual subclass. `issubclass(sub, cls)` returns `True` if `sub` is a subclass of `cls`, including indirect inheritance and ABC registration. A common mistake is passing a class as the first argument to `isinstance`; a class is an instance of `type`, not of its own class.

**Beginner-Friendly Explanation**

`isinstance()` asks “is this object a kind of this class?” It returns `True` for subclasses too. `issubclass()` asks “is this class a kind of that class?” It is used for runtime type checking and polymorphic dispatch. Think of `isinstance(dog, Animal)` as “is this dog an animal?” and `issubclass(Dog, Animal)` as “is the Dog class a kind of Animal?”

#### Purposes

- To perform runtime type checking for validation or dispatch.
- To support polymorphic behavior based on object type.
- To check whether a class implements an interface (especially with ABCs).
- To handle multiple acceptable types by passing a tuple to `isinstance`.
- To distinguish between objects and classes in generic code.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
isinstance(object, classinfo)
issubclass(class, classinfo)
```

**Component Breakdown**

| Function | First Argument | Second Argument | Returns |
|----------|----------------|-----------------|---------|
| `isinstance` | An object | A class or tuple of classes | `True` if the object is an instance of any class |
| `issubclass` | A class | A class or tuple of classes | `True` if the class is a subclass of any class |

**Syntax Rules**

- `isinstance` accepts a tuple of classes: `isinstance(x, (int, float))`.
- `issubclass` accepts a tuple of classes as the second argument.
- Both respect inheritance and ABC registration.
- `isinstance` returns `True` for virtual subclasses registered with ABCs.
- `issubclass` raises `TypeError` if the first argument is not a class.

**Constraints and Limitations**

- Overusing `isinstance` can indicate a design that should use polymorphism instead.
- `isinstance` with `type()` is not equivalent; `type(obj) is cls` does not respect inheritance.
- ABC virtual subclasses pass `issubclass`/`isinstance` but do not appear in the MRO.

#### Annotated Code Examples

**Example 1: `isinstance` vs. `issubclass`**

```python
class Animal: pass
class Dog(Animal): pass

d = Dog()
print(isinstance(d, Animal))      # True
print(isinstance(Dog, Animal))    # False (Dog is a class, not an instance)
print(issubclass(Dog, Animal))    # True
```

**Expected Output:**

```
True
False
True
```

**Step-by-Step Breakdown:**

1. `d` is an instance of `Dog`, which is a subclass of `Animal`, so `isinstance(d, Animal)` is `True`.
2. `Dog` is a class object, not an instance of `Animal`, so `isinstance(Dog, Animal)` is `False`.
3. `Dog` is a subclass of `Animal`, so `issubclass(Dog, Animal)` is `True`.

**Why the Output is Produced:**

`isinstance` checks object-to-class relationships; `issubclass` checks class-to-class relationships.

#### Real-World Cases

- **Polymorphic dispatch**: `if isinstance(event, ClickEvent): ...`.
- **Validation**: Checking that a parameter is an instance of an expected type.
- **Interface checks**: `isinstance(obj, collections.abc.Iterable)` to test for iteration support.
- **Factory functions**: Selecting behavior based on the class hierarchy.

---

### 6. Abstract Base Classes and the `abc` Module

#### Definitions

**Core Definition**

**Abstract Base Classes (ABCs)** are classes that cannot be instantiated directly and define one or more **abstract methods** that subclasses must implement. The `abc` module provides the infrastructure: `ABC`, `ABCMeta`, `abstractmethod`, and (legacy) `abstractproperty`.

**Technical Definition**

The `abc` module implements ABCs per PEP 3119. `ABC` is a helper class whose metaclass is `ABCMeta`. The `@abstractmethod` decorator marks methods that must be overridden; a class with any unimplemented abstract methods cannot be instantiated. `ABCMeta` computes the set of abstract methods at class creation time and enforces instantiation rules. ABCs can also register **virtual subclasses** via `register()`, making `issubclass()` and `isinstance()` recognize classes that do not inherit from the ABC.

**Beginner-Friendly Explanation**

An ABC is a contract. It says “any class that inherits from me must implement these methods.” You cannot create an object directly from an ABC—it is incomplete by design. Subclasses fill in the missing methods, and only then can they be instantiated. This is useful when you want to guarantee that a group of classes all provide a certain interface.

#### Purposes

- To define formal interfaces that subclasses must implement.
- To prevent instantiation of incomplete classes.
- To document the required methods of a class hierarchy.
- To enable `isinstance()` and `issubclass()` checks against interfaces.
- To support virtual subclass registration for classes that cannot inherit directly.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
from abc import ABC, abstractmethod

class MyABC(ABC):
    @abstractmethod
    def required_method(self):
        ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ABC` | Base class for defining ABCs. |
| `@abstractmethod` | Marks a method as abstract; must be overridden. |
| `ABCMeta` | Metaclass that enforces abstractness (usually implicit via `ABC`). |
| `register()` | Registers a virtual subclass. |

**Syntax Rules**

- Inherit from `ABC` (or set `metaclass=ABCMeta`).
- Decorate required methods with `@abstractmethod`.
- A subclass must override all abstract methods to be instantiable.
- Abstract methods can have implementations that subclasses call via `super()`.
- `abstractproperty` is deprecated; use `@property` above `@abstractmethod`.

**Constraints and Limitations**

- ABCs add metaclass complexity; use them when the contract justifies it.
- Virtual subclasses do not inherit implementations from the ABC and do not appear in the MRO.
- Overusing ABCs can lead to over-engineering.

#### Annotated Code Examples

**Example 1: Defining and Implementing an ABC**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """Return the area of the shape."""

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

# Shape()  # TypeError: Can't instantiate abstract class
c = Circle(5)
print(c.area())
```

**Expected Output:**

```
78.5
```

**Step-by-Step Breakdown:**

1. `Shape` inherits from `ABC` and declares `area` as abstract.
2. `Shape()` would raise `TypeError` because `area` is not implemented.
3. `Circle` overrides `area`, so it can be instantiated.

**Why the Output is Produced:**

`ABCMeta` checks at instantiation time that all abstract methods have been overridden. `Circle` satisfies the contract, so it is instantiable.

#### Real-World Cases

- **`collections.abc`**: `Sequence`, `Mapping`, and `Iterable` are ABCs that define container interfaces.
- **Framework interfaces**: Plugins must implement required methods.
- **API contracts**: Ensuring that adapters implement the expected methods.

---

### 7. Abstract Properties

#### Definitions

**Core Definition**

An **abstract property** is a property declared with `@property` and `@abstractmethod` (or the legacy `@abstractproperty`) that subclasses must implement.

**Technical Definition**

Since Python 3.3, `@property` combined with `@abstractmethod` correctly marks the property as abstract. The decorator order is significant: `@property` must appear above `@abstractmethod`. The legacy `@abstractproperty` decorator is deprecated; the combined decorator form is preferred. Subclasses must override the property (getter, and setter if required) to become instantiable.

**Beginner-Friendly Explanation**

An abstract property is like an abstract method, but it defines an attribute that must be provided by subclasses. You declare it with `@property` and `@abstractmethod` stacked together. Any concrete subclass must implement the property’s getter (and setter, if defined).

#### Purposes

- To require subclasses to provide a specific attribute.
- To combine the interface enforcement of ABCs with the attribute syntax of properties.
- To define read-only or read-write attribute contracts.
- To document expected attributes in an interface.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
from abc import ABC, abstractmethod

class MyABC(ABC):
    @property
    @abstractmethod
    def my_property(self):
        """Must be implemented by subclasses."""
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@property` | Defines an attribute-like accessor. |
| `@abstractmethod` | Marks the property as abstract. |
| Decorator order | `@property` must be above `@abstractmethod`. |

**Syntax Rules**

- Use `@property` above `@abstractmethod`.
- Subclasses must override the property to be instantiable.
- For read-write abstract properties, define both getter and setter with `@abstractmethod`.
- The legacy `@abstractproperty` is deprecated.

**Constraints and Limitations**

- Decorator order is critical; reversing it disables abstractness.
- Abstract properties still require subclasses to provide implementations.
- The `@abstractproperty` form should not be used in new code.

#### Annotated Code Examples

**Example 1: Abstract Property**

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    @property
    @abstractmethod
    def wheels(self):
        """Number of wheels."""

class Car(Vehicle):
    @property
    def wheels(self):
        return 4

c = Car()
print(c.wheels)
```

**Expected Output:**

```
4
```

**Step-by-Step Breakdown:**

1. `Vehicle` declares `wheels` as an abstract property.
2. `Car` overrides it with a concrete property returning `4`.
3. `Car()` is instantiable because the abstract property is implemented.

**Why the Output is Produced:**

The abstract property contract is satisfied by `Car`’s override, so instantiation is allowed.

#### Real-World Cases

- **Model interfaces**: Requiring subclasses to expose a `name` or `id` property.
- **Plugin systems**: Ensuring plugins provide expected attributes.
- **Data adapters**: Requiring a `data` property for downstream consumers.

---

### 8. Inheritance Best Practices and Composition vs. Inheritance

#### Definitions

**Core Definition**

Inheritance best practices guide when and how to use inheritance effectively. The central principle is to **favor composition over inheritance** when the relationship is not a genuine “is-a,” and to keep inheritance hierarchies shallow, cooperative, and well-documented.

**Technical Definition**

Best practices include: using inheritance only for “is-a” relationships; preferring composition (“has-a”) for code reuse and swappable behavior; keeping hierarchies shallow (ideally ≤2 levels) to avoid the drawbacks of deep hierarchies; using `super()` consistently in cooperative hierarchies; documenting the MRO for complex multiple inheritance; and using ABCs to define contracts. Composition handles multiple axes of variation with linear growth, whereas inheritance causes combinatorial explosion of subclasses.

**Beginner-Friendly Explanation**

Use inheritance when one thing genuinely **is a** kind of another: a `Dog` **is an** `Animal`. Use composition when one thing **has a** part: a `Car` **has an** `Engine`. If you find yourself creating dozens of subclasses to cover every combination of features, switch to composition. Keep inheritance trees shallow—deep hierarchies are hard to understand and maintain. And always call `super()` so every ancestor gets its turn.

#### Purposes

- To avoid the combinatorial explosion of subclasses.
- To keep code flexible, testable, and maintainable.
- To prevent the fragility of deep, tightly coupled hierarchies.
- To use inheritance only where it genuinely models specialization.
- To leverage modern tools (Protocols, dataclasses) for cleaner composition.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
# Composition: "has-a"
class Engine:
    def start(self): ...

class Car:
    def __init__(self):
        self.engine = Engine()

    def start(self):
        self.engine.start()

# Inheritance: "is-a"
class Animal:
    def speak(self): ...

class Dog(Animal):
    def speak(self):
        return "Woof"
```

**Component Breakdown**

| Approach | Relationship | Growth Pattern | Use When |
|----------|--------------|----------------|----------|
| Inheritance | “is-a” | Combinatorial (product) | Genuine specialization; framework hooks |
| Composition | “has-a” | Linear (sum) | Swappable behavior; multiple axes of variation |

**Syntax Rules**

- Apply the “is-a” vs. “has-a” test before choosing inheritance.
- Keep inheritance hierarchies shallow (ideally ≤2 levels).
- Use `super()` consistently in cooperative hierarchies.
- Prefer composition when behavior needs to be swapped at runtime.
- Use Protocols (PEP 544) for structural typing without inheritance.

**Constraints and Limitations**

- Composition can add indirection and delegation boilerplate.
- Inheritance is still the right choice for framework extension points (e.g., Django models) and genuine specialization.
- Deep hierarchies are a code smell; refactor toward composition when subclasses multiply.

#### Annotated Code Examples

**Example 1: Composition over Inheritance**

```python
# BAD: Combinatorial explosion with inheritance
class Walker: pass
class Flyer: pass
class WalkerWithSword(Walker): pass
class FlyerWithSword(Flyer): pass
# ... explodes with more axes

# GOOD: Composition
class WalkMover:
    def move(self): return "walking"

class FlyMover:
    def move(self): return "flying"

class Sword:
    def attack(self): return "slashing"

class Character:
    def __init__(self, mover, weapon):
        self.mover = mover
        self.weapon = weapon

c = Character(WalkMover(), Sword())
print(c.mover.move(), c.weapon.attack())
```

**Expected Output:**

```
walking slashing
```

**Step-by-Step Breakdown:**

1. The inheritance version would require a subclass for every combination of movement and weapon.
2. The composition version creates independent components and assembles them at construction time.
3. Adding a new movement style requires only one new class, not a new subclass for every weapon.

**Why the Output is Produced:**

Composition lets `Character` delegate movement and attack to separate objects, keeping the class count linear instead of combinatorial.

#### Real-World Cases

- **Game development**: Characters composed of movement, weapon, and armor components.
- **Web frameworks**: Django models use inheritance for framework hooks but composition for services and helpers.
- **Data pipelines**: Composing transformers and validators rather than building deep inheritance chains.
- **Plugin systems**: Using Protocols and composition for loosely coupled extensibility.

---

## References

- Python Documentation – Classes (Tutorial) - https://docs.python.org/3/tutorial/classes.html
- Python Documentation – Built-in Functions: `super()`, `isinstance()`, `issubclass()` - https://docs.python.org/3/library/functions.html
- Python Documentation – `abc` — Abstract Base Classes - https://docs.python.org/3/library/abc.html
- Python Documentation – `collections.abc` — Abstract Base Classes for Containers - https://docs.python.org/3/library/collections.abc.html
- PEP 3119 – Introducing Abstract Base Classes - https://peps.python.org/pep-3119/
- PEP 544 – Protocols: Structural subtyping (static duck typing) - https://peps.python.org/pep-0544/
- Real Python – Inheritance and Composition: A Python OOP Guide - https://realpython.com/inheritance-composition-python/
- Real Python – Method Resolution Order (MRO) - https://realpython.com/ref/glossary/mro/
- CodeGym – `super().__init__()` and MRO Explained: Inheritance Without the Diamond Problem - https://codegym.cc/groups/posts/python-super-init-and-mro
- CodeGym – Composition vs Inheritance in Python: When to Favor Each - https://codegym.cc/groups/posts/python-composition-vs-inheritance
- GeeksforGeeks – C3 Linearization Algorithm in Python - https://www.geeksforgeeks.org/c3-linearization-algorithm-in-python/
- Raymond Hettinger – “Super Considered Super!” - https://rhettinger.wordpress.com/2011/05/26/super-considered-super/
- Python `abc` source code (Lib/abc.py) - https://chromium.googlesource.com/external/github.com/python/cpython/+/d69a0105c0a068e57e95a61a6f096a4207535625/Lib/abc.py