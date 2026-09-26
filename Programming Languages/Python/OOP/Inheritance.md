# Python Inheritance — Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Inheritance is an object-oriented mechanism in which a new class (the subclass or derived class) acquires the attributes and methods of an existing class (the superclass or base class), enabling code reuse and the creation of hierarchical type relationships.

**Technical Definition:** In Python, inheritance is established by listing base classes in the class header. The derived class receives a reference to its base classes and resolves attribute lookups through the Method Resolution Order (MRO), a linearization computed by the C3 algorithm. Python supports single, multiple, and multilevel inheritance, method overriding, and cooperative multiple inheritance through `super()`. Classes may be prevented from being subclassed using the `@final` decorator from the `typing` module.

**Beginner-Friendly Explanation:** Inheritance lets you build a new class based on an existing one. The new class automatically gets everything the old class has (its attributes and methods), and you can add more or change what you need. It's like inheriting traits from a parent — you get their features, but you're still your own person.

### Key Characteristics

- **Code Reuse:** Subclasses inherit methods and attributes from base classes.
- **Is-a Relationship:** Inheritance models "A is a B" relationships (a Dog is an Animal).
- **MRO-Driven Resolution:** Attribute and method lookup follows the Method Resolution Order.
- **Dynamic Linearization:** MRO is computed using the C3 algorithm, ensuring consistency.
- **Cooperative Design:** `super()` enables classes to work together in multiple inheritance.
- **Optional Restriction:** `@final` prevents inheritance and overriding (enforced by type checkers).

### Prerequisites

- Basic class definition and instantiation.
- Understanding of instance and class attributes.
- Familiarity with `self` and method binding.
- Knowledge of the `typing` module basics.

### Related Programming Areas

- Framework design (Django, Flask) and plugin architectures.
- Mixin patterns and composition.
- Abstract base classes and interface design.
- Method Resolution Order and Python's data model.

---

## 1. Single Inheritance

### Definitions

**Core Definition:** Single inheritance is the derivation of a class from exactly one base class.

**Technical Definition:** In single inheritance, the derived class's MRO is a simple linear chain from the derived class to its base and ultimately to `object`. Attribute lookup proceeds up this chain until the attribute is found or `AttributeError` is raised.

**Beginner-Friendly Explanation:** A child class inherits from one parent class. The child gets all the parent's features and can add or change things.

### Purposes

- To reuse code from an existing class.
- To extend or specialize the behavior of a base class.
- To establish an "is-a" relationship.

### Syntax Rules and Structure

```python
class BaseClass:
    """Base class definition."""
    def base_method(self):
        return "base behavior"

class DerivedClass(BaseClass):
    """Derived class inheriting from BaseClass."""
    def derived_method(self):
        return "derived behavior"
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `BaseClass` | The class being inherited from |
| `(BaseClass)` | Parent class specification in the header |
| `base_method` | Inherited by the derived class |
| `derived_method` | New method specific to the derived class |

**Syntax Rules:**
- The base class must be defined (or imported) before the derived class.
- The base class name appears in parentheses after the derived class name.
- If no base class is specified, `object` is the implicit base.

**Constraints and Limitations:**
- A derived class can only have one direct base class in single inheritance.
- Modifying the base class affects all derived classes.

### Annotated Code Examples

```python
class Animal:
    """Base class for animals."""
    
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        """Default behavior for all animals."""
        return "Some generic sound"
    
    def describe(self):
        """Shared description method."""
        return f"I am {self.name}"

class Dog(Animal):
    """Dog inherits from Animal."""
    
    def __init__(self, name, breed):
        # Call the parent's __init__ to set up name
        super().__init__(name)
        self.breed = breed
    
    def speak(self):
        """Override the parent's speak method."""
        return "Woof!"
    
    def fetch(self):
        """Dog-specific method."""
        return f"{self.name} fetches the ball"

# Create an instance
dog = Dog("Buddy", "Golden Retriever")

# Inherited method
print(dog.describe())

# Overridden method
print(dog.speak())

# New method
print(dog.fetch())

# Check inheritance relationship
print(f"Is dog an Animal? {isinstance(dog, Animal)}")
```

**Expected Output:**
```
I am Buddy
Woof!
Buddy fetches the ball
Is dog an Animal? True
```

**Why This Output:** `Dog` inherits `describe()` from `Animal` unchanged. It overrides `speak()` with its own implementation. It adds `fetch()`. `isinstance(dog, Animal)` returns `True` because `Dog` is a subclass of `Animal`.

### Real-World Cases

Django's `View` class is subclassed for specific view types (`TemplateView`, `ListView`). Each subclass inherits HTTP handling logic while customizing behavior. Python's exception hierarchy (`ValueError`, `TypeError` inheriting from `Exception`) is another classic example.

### References

- Python Tutorial: Inheritance — https://docs.python.org/3/tutorial/classes.html#inheritance
- Python Data Model — https://docs.python.org/3/reference/datamodel.html

---

## 2. Multiple Inheritance (The Diamond Problem)

### Definitions

**Core Definition:** Multiple inheritance allows a class to inherit from more than one base class, combining their attributes and methods.

**Technical Definition:** In Python, a class can list multiple base classes: `class D(B, C)`. The C3 linearization algorithm computes the MRO, which determines the order of attribute lookup. The "diamond problem" occurs when a class inherits from two classes that share a common ancestor; Python resolves this by ensuring each class appears only once in the MRO, with a consistent ordering.

**Beginner-Friendly Explanation:** Multiple inheritance is like having two parents instead of one. If both parents can do something differently, Python needs a rule to decide which one to use. The rule is: check in order from left to right, and never visit the same ancestor twice.

### Purposes

- To combine functionality from multiple sources (mixins).
- To create complex type hierarchies.
- To reuse code across unrelated class families.

### Syntax Rules and Structure

```python
class Base1:
    pass

class Base2:
    pass

class Derived(Base1, Base2):
    pass
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `Base1, Base2` | Base classes listed left to right |
| `Derived` | Class inheriting from both |

**Syntax Rules:**
- Base classes are listed in parentheses, separated by commas.
- Left-to-right order matters for MRO.
- If the C3 algorithm cannot produce a consistent linearization, Python raises `TypeError`.

**Constraints and Limitations:**
- Multiple inheritance can lead to fragile hierarchies if not designed carefully.
- The diamond problem requires cooperative design for `super()` to work correctly.
- Python 2.3+ uses C3 linearization; older Python versions had inconsistent MRO behavior .

### Annotated Code Examples

```python
class A:
    def method(self):
        return "A.method"

class B(A):
    def method(self):
        return "B.method"

class C(A):
    def method(self):
        return "C.method"

class D(B, C):
    """D inherits from B and C, both inheriting from A."""
    pass

# Check the MRO
print("MRO:", [cls.__name__ for cls in D.mro()])

# D doesn't define method, so MRO determines which is used
d = D()
print(f"D.method(): {d.method()}")

# Explicit calls to specific parents
print(f"B.method(d): {B.method(d)}")
print(f"C.method(d): {C.method(d)}")
```

**Expected Output:**
```
MRO: ['D', 'B', 'C', 'A', 'object']
D.method(): B.method
B.method(d): B.method
C.method(d): C.method
```

**Why This Output:** The C3 algorithm produces MRO `[D, B, C, A, object]`. Since `D` doesn't define `method`, lookup proceeds to `B`, which does. The diamond is resolved: `A` appears only once, and `B` takes precedence over `C` because it is listed first.

### Real-World Cases

**Mixins:** A `LoggingMixin` and a `SerializationMixin` can both be inherited by a `Model` class to add independent features without deep hierarchies .

**Framework Classes:** Django's `CreateView` inherits from `SingleObjectTemplateResponseMixin` and `BaseCreateView`, combining template rendering with form handling.

### References

- Python Tutorial: Multiple Inheritance — https://docs.python.org/3/tutorial/classes.html#multiple-inheritance
- The Python 2.3 Method Resolution Order — https://docs.python.org/3/howto/mro.html
- Raymond Hettinger: Super Considered Super — https://rhettinger.wordpress.com/2011/05/26/super-considered-super/

---

## 3. Multilevel Inheritance

### Definitions

**Core Definition:** Multilevel inheritance is a chain of inheritance where a class inherits from a derived class, forming a hierarchy of more than two levels.

**Technical Definition:** In multilevel inheritance, class C inherits from B, which inherits from A. The MRO is a simple linear chain: C → B → A → object. Attributes propagate down the chain, and overriding at any level affects all subsequent levels.

**Beginner-Friendly Explanation:** It's like a family tree with grandparents, parents, and children. A grandchild inherits from their parent, who inherited from their parent.

### Purposes

- To create deep specialization hierarchies.
- To model increasingly specific types.
- To layer behavior across multiple levels.

### Syntax Rules and Structure

```python
class GrandParent:
    def method(self):
        return "GrandParent"

class Parent(GrandParent):
    def method(self):
        return "Parent"

class Child(Parent):
    def method(self):
        return "Child"
```

**Component Breakdown:**
| Class | Relationship |
|-------|--------------|
| `GrandParent` | Base of the hierarchy |
| `Parent` | Inherits from `GrandParent` |
| `Child` | Inherits from `Parent` |

**Syntax Rules:**
- Each class inherits from exactly one parent in the chain.
- MRO is linear: `[Child, Parent, GrandParent, object]`.
- `super()` in `Child` calls `Parent`, then `GrandParent`.

**Constraints and Limitations:**
- Deep hierarchies can become hard to maintain.
- Changes at higher levels affect all lower levels.
- Prefer composition over deep inheritance where possible.

### Annotated Code Examples

```python
class Vehicle:
    """Base class for all vehicles."""
    
    def __init__(self, name):
        self.name = name
    
    def move(self):
        return f"{self.name} moves"
    
    def describe(self):
        return f"A vehicle named {self.name}"

class Car(Vehicle):
    """Car inherits from Vehicle."""
    
    def __init__(self, name, wheels=4):
        super().__init__(name)
        self.wheels = wheels
    
    def move(self):
        return f"{self.name} drives on {self.wheels} wheels"

class SportsCar(Car):
    """SportsCar inherits from Car (multilevel)."""
    
    def __init__(self, name, top_speed):
        super().__init__(name, wheels=4)
        self.top_speed = top_speed
    
    def move(self):
        return f"{self.name} zooms at {self.top_speed} mph"

# Create instance
sc = SportsCar("Ferrari", 200)

# Methods resolved through the chain
print(sc.move())          # SportsCar's override
print(sc.describe())      # Inherited from Vehicle
print(f"Wheels: {sc.wheels}")  # Inherited from Car

# MRO inspection
print("MRO:", [cls.__name__ for cls in SportsCar.mro()])
```

**Expected Output:**
```
Ferrari zooms at 200 mph
A vehicle named Ferrari
Wheels: 4
MRO: ['SportsCar', 'Car', 'Vehicle', 'object']
```

**Why This Output:** `SportsCar` overrides `move()`. `describe()` is inherited from `Vehicle` (two levels up). `wheels` is set by `Car.__init__` via `super()` chain. The MRO is a clean linear chain.

### Real-World Cases

Python's exception hierarchy is a multilevel inheritance chain: `object` → `BaseException` → `Exception` → `OSError` → `FileNotFoundError`. Each level adds specificity and behavior.

### References

- Python Tutorial: Inheritance — https://docs.python.org/3/tutorial/classes.html#inheritance
- Python Data Model: MRO — https://docs.python.org/3/reference/datamodel.html

---

## 4. Method Overriding

### Definitions

**Core Definition:** Method overriding occurs when a subclass provides a new implementation for a method that is already defined in its superclass.

**Technical Definition:** When a subclass defines a method with the same name as a method in a base class, the subclass's version takes precedence in the MRO. The base class method is not automatically called; explicit invocation requires `super()` or direct class reference.

**Beginner-Friendly Explanation:** Overriding means replacing a parent's method with your own version. The child's version is used instead.

### Purposes

- To customize inherited behavior.
- To specialize a method for a specific subclass.
- To implement polymorphic behavior.

### Syntax Rules and Structure

```python
class Base:
    def method(self):
        return "base"

class Derived(Base):
    def method(self):
        return "derived"  # Overrides Base.method
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `Base.method` | Original implementation |
| `Derived.method` | Overriding implementation |

**Syntax Rules:**
- Same method name in subclass overrides the parent's.
- `super().method()` calls the parent's version.
- Overriding works across all inheritance types.

**Constraints and Limitations:**
- Cannot override `__init__` without calling `super().__init__()` if parent initialization is needed.
- Overriding can break Liskov Substitution Principle if not careful.

### Annotated Code Examples

```python
class Shape:
    def area(self):
        raise NotImplementedError("Subclass must implement area")
    
    def describe(self):
        return f"Shape with area {self.area()}"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        """Override Shape.area."""
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        """Override Shape.area."""
        return 3.14159 * self.radius ** 2

# Polymorphic behavior
shapes = [Rectangle(3, 4), Circle(5)]
for shape in shapes:
    print(shape.describe())
```

**Expected Output:**
```
Shape with area 12
Shape with area 78.53975
```

**Why This Output:** `Rectangle` and `Circle` override `area()`. `describe()` calls `self.area()`, which dispatches to the appropriate override based on the instance type.

### Real-World Cases

Django's `save()` method is overridden in models to add custom pre-save logic. Python's `__str__` and `__repr__` are commonly overridden to provide custom string representations.

### References

- Python Tutorial: Inheritance — https://docs.python.org/3/tutorial/classes.html#inheritance
- Python Data Model — https://docs.python.org/3/reference/datamodel.html

---

## 5. The `super()` Function (Cooperative Multiple Inheritance)

### Definitions

**Core Definition:** `super()` returns a proxy object that delegates method calls to the next class in the MRO, enabling cooperative multiple inheritance.

**Technical Definition:** `super()` without arguments (Python 3) uses the current class and instance to determine the next class in the MRO. In cooperative multiple inheritance, each class's method calls `super().method()` to pass control along the chain, ensuring all base classes are invoked exactly once.

**Beginner-Friendly Explanation:** `super()` doesn't mean "call my parent." It means "call the next class in the chain." This lets multiple classes work together smoothly.

### Purposes

- To call parent class methods without hardcoding class names.
- To enable cooperative multiple inheritance.
- To support mixins that work with arbitrary base classes.

### Syntax Rules and Structure

```python
class Base:
    def method(self):
        return "base"

class Derived(Base):
    def method(self):
        # Calls the next method in MRO
        result = super().method()
        return f"derived + {result}"
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `super()` | Returns a proxy for the next MRO class |
| `super().method()` | Calls `method` on the next class |

**Syntax Rules:**
- Python 3: `super()` is equivalent to `super(CurrentClass, self)`.
- In cooperative MI, every class should call `super().method()`.
- Use `**kwargs` for cooperative inheritance to pass arguments cleanly .

**Constraints and Limitations:**
- `super()` requires careful design; not calling it breaks the chain.
- Argument signatures must be compatible across cooperating classes .
- `super()` is unnecessary in single inheritance if you don't need cooperative behavior.

### Annotated Code Examples

```python
class LoggingMixin:
    """Mixin that logs method calls."""
    
    def process(self, data, **kwargs):
        print(f"[LOG] Processing: {data}")
        # Cooperative call to next class
        result = super().process(data, **kwargs)
        print(f"[LOG] Result: {result}")
        return result

class ValidationMixin:
    """Mixin that validates data."""
    
    def process(self, data, **kwargs):
        if not data:
            raise ValueError("Data cannot be empty")
        print(f"[VALID] {data} is valid")
        # Cooperative call
        return super().process(data, **kwargs)

class BaseProcessor:
    """Base processor."""
    
    def process(self, data, **kwargs):
        return f"Processed: {data}"

class DataProcessor(ValidationMixin, LoggingMixin, BaseProcessor):
    """Combines validation, logging, and base processing."""
    pass

# Create and use
processor = DataProcessor()
result = processor.process("Hello World")
print(f"Final: {result}")

# Check MRO
print("MRO:", [cls.__name__ for cls in DataProcessor.mro()])
```

**Expected Output:**
```
[VALID] Hello World is valid
[LOG] Processing: Hello World
[LOG] Result: Processed: Hello World
Final: Processed: Hello World
MRO: ['DataProcessor', 'ValidationMixin', 'LoggingMixin', 'BaseProcessor', 'object']
```

**Why This Output:** The MRO is `DataProcessor → ValidationMixin → LoggingMixin → BaseProcessor`. `ValidationMixin.process` runs first, validates, then calls `super().process()`, which invokes `LoggingMixin.process`. That logs, then calls `BaseProcessor.process`. The result bubbles back up.

### Real-World Cases

Django's class-based views use cooperative multiple inheritance extensively. Mixins like `LoginRequiredMixin` and `PermissionRequiredMixin` call `super()` to ensure the view's `dispatch()` method runs properly.

### References

- Raymond Hettinger: Super Considered Super — https://rhettinger.wordpress.com/2011/05/26/super-considered-super/
- Python-Dev: Things to Know About Super — https://mail.python.org/archives/list/python-dev@python.org/message/CES5T6Q66WU2IPSISGUTLCDA5WLRJ2UK/
- Python Documentation: super() — https://docs.python.org/3/library/functions.html#super

---

## 6. Method Resolution Order (MRO) and the C3 Linearization Algorithm

### Definitions

**Core Definition:** The Method Resolution Order (MRO) is the linear order in which Python searches base classes for attributes and methods.

**Technical Definition:** Python 3 uses the C3 linearization algorithm to compute a consistent MRO. The algorithm ensures three properties: (1) local precedence ordering (left-to-right order of bases is preserved), (2) monotonicity (a class's MRO is consistent with its base classes), and (3) each class appears exactly once. If no valid linearization exists, Python raises `TypeError` .

**Beginner-Friendly Explanation:** The MRO is Python's rulebook for deciding which parent's method to use when there are conflicts. C3 linearization makes sure the rulebook is fair and consistent.

### Purposes

- To determine attribute and method lookup order.
- To resolve diamond inheritance conflicts.
- To support `super()` in cooperative multiple inheritance.

### Syntax Rules and Structure

```python
# Inspect MRO
ClassName.mro()
ClassName.__mro__

# MRO is computed automatically for every class
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `mro()` | Method returning list of classes in MRO order |
| `__mro__` | Tuple attribute with same information |

**Syntax Rules:**
- MRO is computed at class creation time.
- C3 linearization merges MROs of base classes.
- Inconsistent hierarchies raise `TypeError` at class definition.

**Constraints and Limitations:**
- Not all multiple inheritance graphs are linearizable.
- Ambiguous hierarchies (e.g., `class C(A, B)` where `B` inherits from `A`) can fail.
- MRO is deterministic and depends only on the inheritance graph .

### Annotated Code Examples

```python
class O:
    pass

class X(O):
    pass

class Y(O):
    pass

class A(X, Y):
    pass

class B(Y, X):
    pass

# Valid MRO for A
print("A MRO:", [cls.__name__ for cls in A.mro()])

# Attempt to create C(A, B) - this will fail
try:
    class C(A, B):
        pass
except TypeError as e:
    print(f"Error creating C: {e}")

# Demonstrate a working diamond
class D(X, Y):
    pass

print("D MRO:", [cls.__name__ for cls in D.mro()])

# Show that MRO is consistent
print(f"D.mro()[0]: {D.mro()[0].__name__}")
print(f"D.mro()[-1]: {D.mro()[-1].__name__}")
```

**Expected Output:**
```
A MRO: ['A', 'X', 'Y', 'O', 'object']
Error creating C: Cannot create a consistent method resolution order (MRO) for bases X, Y
D MRO: ['D', 'X', 'Y', 'O', 'object']
D.mro()[0]: D
D.mro()[-1]: object
```

**Why This Output:** `A(X, Y)` has a valid MRO because `X` and `Y` are independent. `C(A, B)` fails because `A` requires `X` before `Y`, while `B` requires `Y` before `X` — a contradiction C3 cannot resolve. `D(X, Y)` works fine because there's no conflict.

### Real-World Cases

Django's mixin classes must be designed with MRO in mind. Placing `LoginRequiredMixin` before `View` ensures its `dispatch()` method runs first, calling `super().dispatch()` to continue the chain.

### References

- The Python 2.3 Method Resolution Order — https://docs.python.org/3/howto/mro.html
- Python-ideas: Conditions for a coherent MI relationship — https://mail.python.org/archives/list/python-ideas@python.org/message/FUO5LZILFKOOJOPUJRG5LMRB6PIMSILA/

---

## 7. Preventing Inheritance Using `@final`

### Definitions

**Core Definition:** `@final` is a decorator from the `typing` module that signals to type checkers that a class cannot be subclassed or a method cannot be overridden.

**Technical Definition:** The `@final` decorator (PEP 591) is a type-checking hint, not a runtime enforcement. Static type checkers (mypy, pyright) report errors when attempting to subclass a `@final` class or override a `@final` method. At runtime, Python does not prevent inheritance from `@final` classes .

**Beginner-Friendly Explanation:** `@final` is like a "do not inherit from this" sign. Python itself won't stop you, but good tools will warn you, and other programmers will know it's not meant to be subclassed.

### Purposes

- To communicate design intent that a class should not be subclassed.
- To prevent accidental overriding of critical methods.
- To enable type checkers to catch inheritance errors.

### Syntax Rules and Structure

```python
from typing import final

@final
class FinalClass:
    pass

class Base:
    @final
    def critical_method(self):
        return "cannot be overridden"
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `@final` | Decorator on class or method |
| `typing.final` | Import source |

**Syntax Rules:**
- `@final` can decorate classes, instance methods, class methods, static methods, and properties.
- On overloaded methods, place `@final` on the implementation.
- Type checkers report errors; runtime does not enforce.

**Constraints and Limitations:**
- No runtime enforcement: `class Derived(FinalClass)` still works at runtime.
- Requires a type checker to be useful.
- `@final` on a non-method function is an error .

### Annotated Code Examples

```python
from typing import final

@final
class SecureConfig:
    """This class cannot be subclassed (type checker will warn)."""
    
    def __init__(self, secret):
        self.secret = secret

class BaseService:
    @final
    def authenticate(self, token):
        """This method cannot be overridden."""
        return token == "valid"

# Type checker would flag this:
# class DerivedConfig(SecureConfig):  # Error: Cannot inherit from final class
#     pass

# Type checker would flag this:
# class DerivedService(BaseService):
#     def authenticate(self, token):  # Error: Cannot override final method
#         return True

# Runtime does NOT enforce
class RuntimeDerived(SecureConfig):
    pass

print("Runtime inheritance succeeded (no enforcement)")
```

**Expected Output:**
```
Runtime inheritance succeeded (no enforcement)
```

**Why This Output:** `@final` is a static analysis tool. At runtime, Python allows inheritance. A type checker like mypy would report errors on the commented-out lines.

### Real-World Cases

Python's standard library uses `@final` in `asyncio` and other modules to signal that certain classes should not be subclassed. Libraries like `attrs` and `pydantic` use `@final` on generated methods.

### References

- PEP 591: Adding a final qualifier to typing — https://peps.python.org/pep-0591/
- typing.final Documentation — https://typing.python.org/en/latest/spec/qualifiers.html#final
- Python typing.Final — https://docs.python.org/3/library/typing.html#typing.final

---

## Master Reference List

- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html
- The Python 2.3 Method Resolution Order — https://docs.python.org/3/howto/mro.html
- PEP 591: Adding a final qualifier to typing — https://peps.python.org/pep-0591/
- typing.final Specification — https://typing.python.org/en/latest/spec/qualifiers.html#final
- Raymond Hettinger: Super Considered Super — https://rhettinger.wordpress.com/2011/05/26/super-considered-super/
- Python-Dev: Things to Know About Super — https://mail.python.org/archives/list/python-dev@python.org/message/CES5T6Q66WU2IPSISGUTLCDA5WLRJ2UK/
- Python-ideas: Conditions for a coherent MI relationship — https://mail.python.org/archives/list/python-ideas@python.org/message/FUO5LZILFKOOJOPUJRG5LMRB6PIMSILA/