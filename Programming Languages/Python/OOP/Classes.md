# Defining Classes — Python OOP Deep Dive

## Topic Overview

### Definitions

**Core Definition:** Defining classes in Python is the process of creating new user-defined types using the `class` keyword, which bundles data attributes and methods into a single reusable unit.

**Technical Definition:** A class definition creates a new namespace and a class object upon execution. The `class` statement binds a class name to a class object that serves as a template for creating instances. Python distinguishes between object creation (`__new__`) and initialization (`__init__`), stores instance attributes in a per-instance `__dict__` (unless `__slots__` is used), and passes the instance implicitly to methods as the `self` argument.

**Beginner-Friendly Explanation:** Defining a class is like creating a blueprint. You write down what every object of that type should have (attributes) and what it can do (methods). Python then uses that blueprint to build as many objects as you need.

### Key Characteristics

- **Dynamic Creation:** Class definitions are executed at runtime; classes are objects.
- **Namespace Management:** Each class has its own namespace for attributes and methods.
- **Implicit Binding:** Instance methods receive `self` automatically.
- **Memory Flexibility:** Classes use `__dict__` by default; `__slots__` can optimize memory.
- **Dual-Phase Construction:** `__new__` creates; `__init__` initializes.

### Prerequisites

- Basic Python syntax (variables, functions, indentation).
- Understanding of modules and imports.
- Familiarity with dictionaries (for `__dict__`).

### Related Programming Areas

- Object instantiation and lifecycle management.
- Memory optimization for large-scale object creation.
- API design and encapsulation.

---

## 1. The `class` Keyword

### Definitions

**Core Definition:** The `class` keyword is Python's syntax for defining a new class, introducing a new type into the program.

**Technical Definition:** The `class` statement creates a new namespace, executes the class body, and produces a class object bound to the given name. The class object is an instance of `type` (or a metaclass).

**Beginner-Friendly Explanation:** `class` is the word you type to say "I'm about to describe a new kind of thing."

### Purposes

- To declare a new user-defined type.
- To group related data and behavior.
- To serve as a template for creating instances.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName(BaseClass1, BaseClass2, ...):
    """Optional docstring."""
    class_attribute = value
    
    def __init__(self, parameters):
        self.instance_attribute = value
    
    def method_name(self, parameters):
        return value
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `class` | Keyword beginning the definition |
| `ClassName` | Name of the new class (CapWords convention) |
| `(BaseClass...)` | Optional base classes for inheritance |
| `"""docstring"""` | Optional class documentation |
| `class_attribute` | Shared variable in class namespace |
| `def __init__` | Initializer method |
| `def method_name` | Regular instance method |

**Syntax Rules:**
- Class names use `CapWords` convention.
- The class body is indented.
- Class definition must be executed before the class can be used.

**Constraints and Limitations:**
- Class name must be a valid identifier.
- Empty class bodies require `pass`.

### Annotated Code Examples

**Example: Complete Class Definition**

```python
class Student:
    """A class representing a university student."""
    
    # Class attribute (shared by all instances)
    university = "State University"
    student_count = 0
    
    def __init__(self, name, student_id):
        """Initialize a new Student instance."""
        self.name = name          # Instance attribute
        self.student_id = student_id
        self.courses = []         # Mutable instance attribute
        Student.student_count += 1  # Modify class attribute
    
    def enroll(self, course):
        """Add a course to the student's schedule."""
        self.courses.append(course)
        return f"{self.name} enrolled in {course}"
    
    def describe(self):
        """Return a description of the student."""
        return f"{self.name} ({self.student_id}) at {self.university}"

# Create instances
s1 = Student("Alice", "S001")
s2 = Student("Bob", "S002")

print(s1.describe())
print(s2.describe())
print(f"Total students: {Student.student_count}")

s1.enroll("CS 101")
print(s1.courses)
print(s2.courses)  # Empty - separate list
```

**Expected Output:**
```
Alice (S001) at State University
Bob (S002) at State University
Total students: 2
['CS 101']
[]
```

**Why This Output:** Each instance has its own `name`, `student_id`, and `courses`. The `university` and `student_count` are shared at class level. Modifying `s1.courses` does not affect `s2.courses`.

### Real-World Cases

Django models use the `class` keyword to define database tables. Each class maps to a table, class attributes define columns, and methods define behavior.

### References

- Python Tutorial: Class Definition Syntax — https://docs.python.org/3.14/tutorial/classes.html

---

## 2. Constructor vs. Initializer (`__new__` vs `__init__`)

### Definitions

**Core Definition:** `__new__` is the constructor that creates a new object; `__init__` is the initializer that customizes an already-created object.

**Technical Definition:** When a class is called, Python first invokes `__new__(cls, ...)` to allocate and return a new instance. If `__new__` returns an instance of `cls`, Python then calls `__init__(self, ...)` to initialize it. `__new__` is a static method receiving the class; `__init__` is an instance method receiving the new object.

**Beginner-Friendly Explanation:** `__new__` builds the raw object (like manufacturing a blank car). `__init__` paints and customizes it (like adding paint, seats, and radio). Most of the time you only need `__init__`.

### Purposes

- To customize object creation for immutable types (`__new__`).
- To initialize instance attributes after creation (`__init__`).
- To control what object is actually returned from construction.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        instance = super().__new__(cls)
        # Customize creation
        return instance
    
    def __init__(self, *args, **kwargs):
        # Initialize attributes
        pass
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `cls` | The class object (first arg to `__new__`) |
| `self` | The new instance (first arg to `__init__`) |
| `super().__new__(cls)` | Calls parent `__new__` to allocate |
| `return instance` | Must return the new object |

**Syntax Rules:**
- `__new__` must return an instance; if it doesn't return an instance of `cls`, `__init__` is skipped.
- `__new__` is implicitly a static method.
- `__init__` must return `None`.

**Constraints and Limitations:**
- `__new__` is mainly needed for immutable types (int, str, tuple) or metaclasses.
- For mutable objects, use `__init__` only.

### Annotated Code Examples

**Example 1: `__init__` for Mutable Objects**

```python
class MutablePoint:
    def __init__(self, x, y):
        print("__init__ called")
        self.x = x
        self.y = y

p = MutablePoint(3, 4)
print(f"Point: ({p.x}, {p.y})")
```

**Expected Output:**
```
__init__ called
Point: (3, 4)
```

**Why This Output:** Python creates the empty object, then calls `__init__` to set attributes.

**Example 2: `__new__` for Immutable Subclass**

```python
class PositiveInt(int):
    """An int that must be positive."""
    
    def __new__(cls, value):
        if value < 0:
            raise ValueError("Value must be positive")
        return super().__new__(cls, value)

try:
    n = PositiveInt(5)
    print(f"Created: {n}")
    bad = PositiveInt(-3)
except ValueError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Created: 5
Error: Value must be positive
```

**Why This Output:** Since `int` is immutable, validation must happen in `__new__` before the object is created. `__init__` cannot modify the value afterward.

### Real-World Cases

- **Enums:** Custom `__new__` to enforce singleton patterns.
- **Cached instances:** `__new__` can return a cached object instead of creating a new one.
- **Immutable value objects:** Validation in `__new__` ensures the object is never in an invalid state.

### References

- Python Data Model: object.__new__ — https://docs.python.org/3/reference/datamodel.html#object.__new__
- Tutor Mailing List: __new__ vs __init__ — https://mail.python.org/archives/list/tutor@python.org/message/SI4GYJQSMKUZIPIZELYUAEAW6G7DTUXQ/

---

## 3. Instance Attributes and the `__dict__` Namespace

### Definitions

**Core Definition:** Instance attributes are variables bound to a specific instance, stored in the instance's `__dict__` dictionary.

**Technical Definition:** Each instance of a user-defined class has a `__dict__` attribute (unless `__slots__` is defined) that maps attribute names to values. Attribute lookup checks the instance `__dict__` first, then the class and its bases.

**Beginner-Friendly Explanation:** The `__dict__` is like a personal locker for each object, where it stores its own data.

### Purposes

- To store per-instance state.
- To allow dynamic attribute addition.
- To enable attribute introspection.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class MyClass:
    def __init__(self):
        self.attr1 = value1  # Creates entry in __dict__
        self.attr2 = value2

# Access __dict__
obj.__dict__  # Returns {'attr1': value1, 'attr2': value2}
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `self.attr` | Assignment creates `__dict__` entry |
| `obj.__dict__` | The instance's attribute dictionary |
| `vars(obj)` | Equivalent to `obj.__dict__` |

**Syntax Rules:**
- Attributes spring into existence on first assignment.
- Instance attributes shadow class attributes of the same name.
- `__dict__` is writable for user-defined classes.

**Constraints and Limitations:**
- Classes with `__slots__` have no `__dict__`.
- Built-in types (int, str) have no instance `__dict__`.

### Annotated Code Examples

```python
class Config:
    default_timeout = 30  # Class attribute
    
    def __init__(self, host):
        self.host = host  # Instance attribute
    
    def set_port(self, port):
        self.port = port  # Created on first call

cfg = Config("api.example.com")
print(f"Initial __dict__: {cfg.__dict__}")

cfg.set_port(8080)
print(f"After set_port: {cfg.__dict__}")

# Class attribute not in instance __dict__
print(f"timeout from instance: {cfg.default_timeout}")
print(f"timeout in __dict__: {'default_timeout' in cfg.__dict__}")
```

**Expected Output:**
```
Initial __dict__: {'host': 'api.example.com'}
After set_port: {'host': 'api.example.com', 'port': 8080}
timeout from instance: 30
timeout in __dict__: False
```

**Why This Output:** `host` and `port` are instance-specific, so they appear in `__dict__`. `default_timeout` is a class attribute, found via class lookup, not stored in the instance dictionary.

### Real-World Cases

ORMs like SQLAlchemy use `__dict__` to track loaded column values vs. defaults. Serialization libraries inspect `__dict__` to convert objects to JSON.

### References

- Python Tutorial: Instance Objects — https://docs.python.org/3.14/tutorial/classes.html
- Python Reference: Attribute Access — New Mexico Tech PDF

---

## 4. Instance Methods and the Implicit `self` Argument

### Definitions

**Core Definition:** Instance methods are functions defined in a class that receive the instance as their first argument (`self`), allowing access to instance state.

**Technical Definition:** A method is a function object stored in the class namespace. When accessed via an instance, Python creates a bound method object that automatically prepends the instance to the argument list. `instance.method(args)` is equivalent to `Class.method(instance, args)`.

**Beginner-Friendly Explanation:** `self` is how a method knows which object it's working on. When you call `dog.bark()`, Python automatically passes `dog` as `self`.

### Purposes

- To access and modify instance attributes.
- To call other methods on the same instance.
- To provide behavior specific to each object.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class MyClass:
    def method_name(self, arg1, arg2):
        # self is the instance
        self.attr = arg1
        return arg2
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `self` | First parameter; reference to the instance |
| `arg1, arg2` | Additional explicit arguments |
| `self.attr` | Access instance attribute via self |

**Syntax Rules:**
- `self` must be the first parameter (convention, not keyword).
- The name `self` is a strong convention but not enforced.
- Calling `instance.method(a, b)` passes `instance` as `self` automatically.
- Calling `Class.method(instance, a, b)` is equivalent but explicit.

**Constraints and Limitations:**
- Forgetting `self` in the definition causes `TypeError` when called.
- `self` cannot be omitted in instance methods.

### Annotated Code Examples

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1
        return self.count
    
    def add(self, amount):
        self.count += amount
        return self.count

c = Counter()
print(c.increment())      # Implicit self
print(c.add(5))           # Implicit self with argument

# Explicit call (equivalent to above)
print(Counter.increment(c))
```

**Expected Output:**
```
1
6
7
```

**Why This Output:** `c.increment()` becomes `Counter.increment(c)`. The instance is passed automatically as `self`, allowing the method to modify `c.count`.

### Real-World Cases

All instance methods in Django views, Flask routes, and custom classes rely on `self` to access request data, database connections, or configuration.

### References

- Python Tutorial: Method Objects — https://docs.python.org/3.14/tutorial/classes.html
- Python List: Why 'self' — https://mail.python.org/pipermail/python-list/2003-September/207843.html

---

## 5. Class Attributes (Shared State)

### Definitions

**Core Definition:** Class attributes are variables defined in the class body that are shared by all instances of the class.

**Technical Definition:** Class attributes are stored in the class's `__dict__`. They are accessed via the class or any instance. If an instance assigns to the same name, an instance attribute is created that shadows the class attribute for that instance only.

**Beginner-Friendly Explanation:** Class attributes are like shared community property. Everyone in the class can see and use them, but if one instance changes its own copy, others aren't affected.

### Purposes

- To store constants shared by all instances.
- To track class-level state (e.g., instance count).
- To provide default values.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class MyClass:
    shared_attr = value  # Class attribute
    
    def __init__(self):
        self.instance_attr = value  # Instance attribute
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `shared_attr` | Defined at class level, shared by all |
| `MyClass.shared_attr` | Access via class |
| `instance.shared_attr` | Access via instance (falls back to class) |

**Syntax Rules:**
- Class attributes are defined in the class body, outside methods.
- Modifying `ClassName.attr` changes it for all instances.
- Assigning `instance.attr = value` creates an instance attribute that shadows the class attribute.

**Constraints and Limitations:**
- Mutable class attributes (lists, dicts) are shared; modifying them affects all instances.
- Use class attributes for constants, not mutable state.

### Annotated Code Examples

```python
class Circle:
    pi = 3.14159  # Class attribute (constant)
    count = 0     # Class attribute (mutable tracker)
    
    def __init__(self, radius):
        self.radius = radius
        Circle.count += 1
    
    def area(self):
        return Circle.pi * self.radius ** 2

c1 = Circle(5)
c2 = Circle(10)

print(f"Count: {Circle.count}")
print(f"c1 area: {c1.area():.2f}")
print(f"c2 area: {c2.area():.2f}")

# Modifying class attribute via class
Circle.pi = 3.14
print(f"c1 area after pi change: {c1.area():.2f}")

# Instance shadowing
c1.pi = 3.0
print(f"c1.pi (instance): {c1.pi}")
print(f"c2.pi (class): {c2.pi}")
```

**Expected Output:**
```
Count: 2
c1 area: 78.54
c2 area: 314.16
c1 area after pi change: 78.50
c1.pi (instance): 3.0
c2.pi (class): 3.14
```

**Why This Output:** `Circle.pi` change affects both instances. `c1.pi = 3.0` creates an instance attribute only on `c1`, shadowing the class attribute for that instance.

### Real-World Cases

Configuration classes use class attributes for defaults. Game entities use class attributes for shared constants (gravity, max health).

### References

- Python List: Class Attributes — https://mail.python.org/pipermail/python-list/2012-August/attachments/20120830/a16c4d8d/attachment.html
- Python List: OOP in Python — https://mail.python.org/pipermail/python-list/2001-June/108505.html

---

## 6. Memory Optimization Using `__slots__`

### Definitions

**Core Definition:** `__slots__` is a class-level declaration that pre-defines allowed instance attributes, eliminating the per-instance `__dict__` to save memory.

**Technical Definition:** When a class defines `__slots__`, instances use a fixed-size array for attributes instead of a dictionary. This reduces memory overhead and prevents adding attributes not listed in `__slots__`.

**Beginner-Friendly Explanation:** Normally each object carries a dictionary for its attributes. `__slots__` replaces that dictionary with a fixed list, like assigning numbered seats instead of a flexible but bulky address book.

### Purposes

- To reduce memory usage for classes with many instances.
- To prevent accidental attribute creation.
- To improve attribute access speed.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class MyClass:
    __slots__ = ('attr1', 'attr2', 'attr3')
    
    def __init__(self):
        self.attr1 = value1
        self.attr2 = value2
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `__slots__` | Tuple of allowed attribute names |
| `self.attr1` | Must be in `__slots__` |

**Syntax Rules:**
- `__slots__` must be defined at class level.
- Attributes not in `__slots__` raise `AttributeError`.
- Instances of `__slots__` classes have no `__dict__`.

**Constraints and Limitations:**
- Inheritance: if a parent has `__dict__`, child instances still have it unless child also defines `__slots__` and parent doesn't have `__dict__`.
- Cannot use `__slots__` with variable-length built-in types directly.
- Weak references require `__weakref__` in `__slots__`.

### Annotated Code Examples

```python
import sys

class RegularPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class SlottedPoint:
    __slots__ = ('x', 'y')
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Compare memory usage
r = RegularPoint(1, 2)
s = SlottedPoint(1, 2)

print(f"Regular has __dict__: {hasattr(r, '__dict__')}")
print(f"Slotted has __dict__: {hasattr(s, '__dict__')}")

# Size comparison (approximate)
print(f"Regular size: {sys.getsizeof(r)} bytes")
print(f"Slotted size: {sys.getsizeof(s)} bytes")

# Cannot add new attribute to slotted
try:
    s.z = 3
except AttributeError as e:
    print(f"Error: {e}")

# Regular allows dynamic attributes
r.z = 3
print(f"Regular z: {r.z}")
```

**Expected Output:**
```
Regular has __dict__: True
Slotted has __dict__: False
Regular size: 48 bytes
Slotted size: 40 bytes
Error: 'SlottedPoint' object has no attribute 'z'
Regular z: 3
```

**Why This Output:** `__slots__` eliminates the `__dict__`, saving memory. Attempting to set an attribute not in `__slots__` raises `AttributeError`. The regular instance can accept new attributes because it has a `__dict__`.

### Real-World Cases

- **Particle simulations:** Millions of objects each storing x, y, z coordinates.
- **Network packets:** Many small objects with fixed fields.
- **Game entities:** Large numbers of NPCs with position, health, and state.

### References

- Python Glossary: __slots__ — https://docs.python.org/3/glossary.html#term-__slots__
- Python Reference: Conserving Memory with __slots__ — New Mexico Tech PDF

---

## Master Reference List

- Python Tutorial: Classes — https://docs.python.org/3.14/tutorial/classes.html
- Python Data Model — https://docs.python.org/3/reference/datamodel.html
- Python Glossary — https://docs.python.org/3/glossary.html
- Object Life Cycle (C API) — https://docs.python.org/3.14/c-api/lifecycle.html
- Tutor Mailing List: __new__ vs __init__ — https://mail.python.org/archives/list/tutor@python.org/message/SI4GYJQSMKUZIPIZELYUAEAW6G7DTUXQ/