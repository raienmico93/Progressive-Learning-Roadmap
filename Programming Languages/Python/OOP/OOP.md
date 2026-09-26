# Python OOP Fundamentals — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Object-Oriented Programming (OOP) in Python is a programming paradigm that organizes software design around data (objects) rather than functions and logic, bundling state and behavior into discrete, reusable units called classes and instances.

**Technical Definition:** Python's object model treats every entity as an object with an identity, a type, and a value. Classes serve as user-defined types that define a namespace for attributes and methods, while instances are concrete realizations of those types allocated on the heap. Python supports encapsulation through naming conventions and name mangling, abstraction through the `abc` module, single and multiple inheritance with a well-defined Method Resolution Order (MRO), and both duck typing and formal polymorphism.

**Beginner-Friendly Explanation:** Think of a class as a blueprint for a house. The blueprint itself isn't a house—it just describes what every house built from it will have: doors, windows, a roof. An object (or instance) is an actual house built from that blueprint. Each house can have different paint colors and furniture, but they all share the same structural design. OOP lets you create many houses from one blueprint without rebuilding the design each time.

### Key Characteristics

- **Encapsulation:** Bundling data and methods together; Python uses `_` and `__` conventions rather than enforced access modifiers.
- **Abstraction:** Hiding complexity behind simplified interfaces using abstract base classes.
- **Inheritance:** Creating new classes that inherit attributes and methods from existing ones.
- **Polymorphism:** The ability of different classes to respond to the same method call in different ways.
- **Dynamic Nature:** Attributes can be added to instances at runtime (for user-defined classes).
- **Everything is an Object:** Integers, strings, functions, and classes themselves are objects in Python.

### Prerequisites

Before diving into Python OOP, you should understand:

- **Variables and Data Types:** `int`, `str`, `list`, `dict`, `bool`, `None`.
- **Functions:** Definition, parameters, return values, scope.
- **Modules and Imports:** How to import standard library modules like `abc`.
- **Basic Control Flow:** `if`/`else`, `for`/`while` loops, exception handling (`try`/`except`).

### Related Programming Areas

- **Functional Programming:** Python supports both paradigms; functions are first-class objects.
- **Design Patterns:** OOP is the foundation for patterns like Factory, Singleton, Observer, and Strategy.
- **Data Modeling:** Classes are used extensively in frameworks like Django (models) and SQLAlchemy (ORM).
- **API Design:** Abstract base classes define interfaces for plugins and extensible systems.

### Core Concepts / Features

The following core concepts are covered in this cheat sheet:

1. **Objects**
2. **Classes**
3. **Attributes (State)**
4. **Methods (Behavior)**
5. **Instances**
6. **Encapsulation**
7. **Abstraction**
8. **Inheritance**
9. **Polymorphism**

---

## 1. Objects

### Definitions

**Core Definition:** An object is a self-contained entity that combines data (attributes) and behavior (methods) into a single unit.

**Technical Definition:** In Python, an object is an instance of a class, possessing a unique identity (`id()`), a type (`type()`), and a value. Every object in Python is allocated memory on the heap and is referenced by a variable name. Objects can be mutable or immutable depending on their type.

**Beginner-Friendly Explanation:** An object is like a specific car. It has properties (color, model, speed) and things it can do (accelerate, brake, honk). Every car is an object, and although they all share the same general concept, each one is distinct.

### Purposes

- To bundle related data and behavior into a single, cohesive unit.
- To provide a unique identity for each entity in a program.
- To enable state management through attribute mutations.
- To allow interaction between different parts of a program through well-defined interfaces.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
# Creating an object (instantiation)
variable_name = ClassName(arguments)

# Accessing attributes
variable_name.attribute_name

# Calling methods
variable_name.method_name(arguments)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `variable_name` | A valid Python identifier that references the object |
| `ClassName` | The name of the class being instantiated (CapWords convention) |
| `arguments` | Optional arguments passed to the constructor (`__init__`) |
| `.attribute_name` | Dot notation to access an object's attribute |
| `.method_name()` | Dot notation to call an object's method |

**Syntax Rules:**
- Objects are created by calling the class name as if it were a function.
- Every object has a type accessible via `type(obj)` and an identity via `id(obj)`.
- Objects are referenced by variables, not stored directly in variables.

**Constraints and Limitations:**
- Built-in types like `int` and `str` are immutable; user-defined objects are typically mutable.
- Not all objects support all operations; calling an undefined method raises `AttributeError`.
- Objects must be instantiated before their instance attributes can be accessed.

### Annotated Code Examples

**Example 1: Basic Object Creation and Inspection**

```python
# Define a simple class (blueprint for objects)
class Dog:
    """A class representing a dog."""
    
    def __init__(self, name, age):
        """Initialize a new Dog object with a name and age."""
        self.name = name  # Instance attribute
        self.age = age    # Instance attribute

# Create an object (instance) of the Dog class
my_dog = Dog("Buddy", 3)

# Inspect the object
print(f"Object: {my_dog}")           # Default representation
print(f"Type: {type(my_dog)}")       # The class of the object
print(f"ID: {id(my_dog)}")           # Memory address identifier
print(f"Name attribute: {my_dog.name}")  # Access instance attribute
print(f"Age attribute: {my_dog.age}")    # Access instance attribute
```

**Step-by-Step Setup Guide:**
1. Define the `Dog` class with an `__init__` method.
2. Call `Dog("Buddy", 3)` to create an instance.
3. The variable `my_dog` references the new object.
4. Access attributes using dot notation.

**Expected Output:**
```
Object: <__main__.Dog object at 0x7f...>
Type: <class '__main__.Dog'>
ID: 140234567890123
Name attribute: Buddy
Age attribute: 3
```

**Why This Output:** The default `print` shows the class name and memory address because no `__str__` or `__repr__` method was defined. `type()` returns the class object, `id()` returns the unique memory address, and dot notation retrieves the attribute values stored in the object's `__dict__`.

**Example 2: Objects Are Independent**

```python
class Counter:
    def __init__(self, start=0):
        self.count = start

# Create two independent objects
c1 = Counter(10)
c2 = Counter(20)

print(f"c1 count: {c1.count}")  # 10
print(f"c2 count: {c2.count}")  # 20

# Modify one object's attribute
c1.count = 15

print(f"After modification:")
print(f"c1 count: {c1.count}")  # 15
print(f"c2 count: {c2.count}")  # 20 (unchanged)
```

**Expected Output:**
```
c1 count: 10
c2 count: 20
After modification:
c1 count: 15
c2 count: 20
```

**Why This Output:** Each instance maintains its own attribute dictionary. Modifying `c1.count` does not affect `c2.count` because they are separate objects with separate state.

### Real-World Cases

**Case 1: GUI Elements**
In frameworks like Tkinter, every button, label, and window is an object. Each widget object maintains its own state (text, color, position) and behavior (click handlers, redraw methods).

**Case 2: Web Request Handling**
In Django, an incoming HTTP request is represented as an `HttpRequest` object with attributes like `method`, `path`, and `GET`, and methods like `is_ajax()`. This encapsulation keeps request data and logic together.

### References

- Python Documentation: Data Model — https://docs.python.org/3/reference/datamodel.html
- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html

---

## 2. Classes

### Definitions

**Core Definition:** A class is a blueprint or template that defines the attributes and methods that objects of that type will have.

**Technical Definition:** A class is a user-defined type created using the `class` statement. It defines a namespace containing attributes (class variables), methods (functions defined within the class), and special methods (dunder methods like `__init__`). The class object itself is an instance of `type` (or a metaclass).

**Beginner-Friendly Explanation:** A class is like a cookie cutter. It defines the shape of the cookie, but it isn't a cookie itself. You can use the same cutter to make many cookies (objects), each of which can be decorated differently.

### Purposes

- To define a new type that bundles data and behavior.
- To provide a template for creating multiple objects with the same structure.
- To organize code into logical, reusable units.
- To enable inheritance hierarchies and polymorphism.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    """Class specification (optional docstring)."""
    
    # Class attributes (shared by all instances)
    class_attribute = value
    
    def __init__(self, parameters):
        """Initialize instance attributes."""
        self.instance_attribute = value
    
    def method_name(self, parameters):
        """Method definition."""
        # method body
        return value
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class` | Keyword that begins the class definition |
| `ClassName` | Class name, conventionally `CapWords` |
| `"""docstring"""` | Optional class specification describing the class |
| `class_attribute` | Variable defined in class body, shared by all instances |
| `def __init__(self, ...)` | Initializer method called during instantiation |
| `self` | Reference to the instance being created/used |
| `method_name` | Method defined in class body, conventionally `snake_case` |

**Syntax Rules:**
- Class names use `CapWords` convention; internal classes may have a leading underscore.
- The class body is indented relative to the `class` statement.
- `__init__` is not a constructor but an initializer; the actual object is created before `__init__` is called.
- Methods must include `self` as the first parameter (unless using `@staticmethod`).

**Constraints and Limitations:**
- A class can be empty (using `pass`), but then it has no useful behavior.
- Class names must be valid Python identifiers.
- The class statement creates a class object at runtime; it is not a compile-time declaration.

### Annotated Code Examples

**Example 1: Complete Class Definition**

```python
class Student:
    """A class representing a student.
    
    Instance Attributes:
        name (str): The student's full name
        net_id (str): The student's network ID
        courses (list): List of course names
    """
    
    # Class attribute (shared by all instances)
    enrollment_count = 0
    
    def __init__(self, name, net_id):
        """Initialize a new Student instance.
        
        Args:
            name: Student's full name
            net_id: Student's network ID
        """
        # Instance attributes (unique per instance)
        self.name = name
        self.net_id = net_id
        self.courses = []
        
        # Increment class attribute
        Student.enrollment_count += 1
    
    def enroll(self, course):
        """Add a course to the student's schedule."""
        self.courses.append(course)
        return f"{self.name} enrolled in {course}"
    
    def describe(self):
        """Return a description of the student."""
        return f"Student: {self.name} ({self.net_id}), Courses: {self.courses}"

# Usage
s1 = Student("Alice Smith", "as123")
s2 = Student("Bob Jones", "bj456")

print(s1.describe())
print(s2.describe())
print(f"Total students: {Student.enrollment_count}")

s1.enroll("CS 1110")
s1.enroll("MATH 1920")
print(s1.describe())
```

**Step-by-Step Setup Guide:**
1. Define `Student` class with a docstring.
2. Add class attribute `enrollment_count`.
3. Define `__init__` to set instance attributes.
4. Define `enroll()` and `describe()` methods.
5. Create two instances.
6. Call methods and inspect class attribute.

**Expected Output:**
```
Student: Alice Smith (as123), Courses: []
Student: Bob Jones (bj456), Courses: []
Total students: 2
Student: Alice Smith (as123), Courses: ['CS 1110', 'MATH 1920']
```

**Why This Output:** Each instance has its own `name`, `net_id`, and `courses`. The `enrollment_count` is shared and incremented each time `__init__` runs. `enroll()` modifies only the specific instance's `courses` list.

**Example 2: Class with Default Arguments**

```python
class Course:
    """A university course."""
    
    def __init__(self, name, credits=3, instructor=None):
        """Initialize a Course.
        
        Args:
            name: Course name (required)
            credits: Number of credits (default: 3)
            instructor: Instructor name (default: None)
        """
        self.name = name
        self.credits = credits
        self.instructor = instructor
    
    def __str__(self):
        """String representation of the course."""
        inst = self.instructor or "TBA"
        return f"{self.name} ({self.credits} credits) - {inst}"

# Various ways to create instances
c1 = Course("CS 1110")                    # Uses defaults
c2 = Course("MATH 1920", 4)               # Overrides credits
c3 = Course("PHYS 1112", instructor="Dr. Einstein")  # Uses keyword arg

print(c1)
print(c2)
print(c3)
```

**Expected Output:**
```
CS 1110 (3 credits) - TBA
MATH 1920 (4 credits) - TBA
PHYS 1112 (3 credits) - Dr. Einstein
```

**Why This Output:** Default arguments allow flexible instantiation. `__str__` defines the string representation used by `print()`.

### Real-World Cases

**Case 1: Data Models in Web Frameworks**
Django models are classes that map to database tables. Each class defines fields (attributes) and methods for querying and manipulating data.

**Case 2: Game Development**
A `Character` class defines health, position, and methods like `move()` and `attack()`. Different character types inherit from this base class.

### References

- Python Documentation: Classes — https://docs.python.org/3/tutorial/classes.html
- PEP 8: Style Guide for Python Code — https://peps.python.org/pep-0008/#class-names

---

## 3. Attributes (State)

### Definitions

**Core Definition:** Attributes are variables that store data associated with a class or an instance, representing the object's state.

**Technical Definition:** Attributes are name-value bindings stored in either the class's `__dict__` (class attributes) or the instance's `__dict__` (instance attributes). Python uses a lookup order: instance `__dict__` first, then class `__dict__`, then base classes following the MRO.

**Beginner-Friendly Explanation:** Attributes are like the properties of an object. For a car object, attributes might be `color`, `model`, and `speed`. Each car can have different values for these properties.

### Purposes

- To store the state of an object.
- To share data across all instances of a class.
- To provide named access to an object's internal data.
- To enable attribute-based polymorphism through dynamic lookup.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
# Class attribute (defined in class body)
class ClassName:
    class_attr = value

# Instance attribute (defined in __init__ or any method)
def __init__(self, param):
    self.instance_attr = param

# Accessing attributes
obj.instance_attr           # Access instance attribute
ClassName.class_attr        # Access class attribute
obj.class_attr              # Also accesses class attribute (if no instance override)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `class_attr` | Variable in class body; shared by all instances |
| `self.instance_attr` | Variable bound to specific instance; unique per object |
| `self` | The instance reference used to access/modify instance attributes |
| `.__dict__` | Dictionary storing an object's or class's attributes |

**Syntax Rules:**
- Instance attributes are created by assignment to `self.attribute`.
- Class attributes are accessed via `ClassName.attribute` or `instance.attribute`.
- If an instance attribute shadows a class attribute, the instance value takes precedence.
- Attributes can be added dynamically to user-defined class instances.

**Constraints and Limitations:**
- Accessing an undefined attribute raises `AttributeError`.
- Mutable class attributes (like lists) can lead to unexpected sharing between instances.
- Instance attributes cannot be accessed before assignment.

### Annotated Code Examples

**Example 1: Instance vs. Class Attributes**

```python
class Circle:
    """A circle with a class-level pi constant."""
    
    pi = 3.14159  # Class attribute (shared)
    
    def __init__(self, radius):
        self.radius = radius  # Instance attribute (unique)
    
    def area(self):
        return Circle.pi * self.radius ** 2

# Create instances
c1 = Circle(5)
c2 = Circle(10)

print(f"c1 radius: {c1.radius}, area: {c1.area():.2f}")
print(f"c2 radius: {c2.radius}, area: {c2.area():.2f}")
print(f"Class pi: {Circle.pi}")

# Accessing class attribute through instance
print(f"c1.pi: {c1.pi}")

# Changing instance attribute doesn't affect others
c1.radius = 7
print(f"After change - c1 radius: {c1.radius}, c2 radius: {c2.radius}")
```

**Expected Output:**
```
c1 radius: 5, area: 78.54
c2 radius: 10, area: 314.16
Class pi: 3.14159
c1.pi: 3.14159
After change - c1 radius: 7, c2 radius: 10
```

**Why This Output:** `pi` is shared across all instances. `radius` is unique per instance. Changing `c1.radius` does not affect `c2.radius`.

**Example 2: Dynamic Attribute Assignment**

```python
class Point:
    """A point with optional metadata."""
    pass

# Create instance
p = Point()

# Add attributes dynamically
p.x = 10
p.y = 20
p.label = "Origin"

print(f"Point: ({p.x}, {p.y}) - {p.label}")

# Adding attributes after creation is allowed for user-defined classes
p.color = "red"
print(f"Color: {p.color}")
```

**Expected Output:**
```
Point: (10, 20) - Origin
Color: red
```

**Why This Output:** Python allows dynamic attribute creation for user-defined classes. The instance's `__dict__` is populated at runtime.

### Real-World Cases

**Case 1: Configuration Objects**
A `Config` class with instance attributes like `host`, `port`, and `timeout` represents application settings. Class attributes can hold defaults.

**Case 2: ORM Entities**
In SQLAlchemy, each column in a database table is an instance attribute of the mapped class. Class attributes define column metadata.

### References

- Python Documentation: Class and Instance Variables — https://docs.python.org/3/tutorial/classes.html#class-and-instance-variables
- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html

---

## 4. Methods (Behavior)

### Definitions

**Core Definition:** Methods are functions defined within a class that describe the behavior of instances of that class.

**Technical Definition:** A method is a function object stored as an attribute of a class. When accessed through an instance, Python binds the instance as the first argument (typically named `self`), creating a bound method object. When accessed through the class, it remains a plain function requiring explicit instance passing.

**Beginner-Friendly Explanation:** Methods are the actions an object can perform. If a dog is an object, `bark()`, `sit()`, and `fetch()` are its methods. They define what the dog can do.

### Purposes

- To define operations that objects can perform.
- To encapsulate logic that manipulates an object's state.
- To provide a clear interface for interacting with objects.
- To enable polymorphic behavior through method overriding.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    def method_name(self, param1, param2):
        """Method docstring."""
        # method body
        return value
    
    @staticmethod
    def static_method(param):
        """Does not receive self or cls."""
        return value
    
    @classmethod
    def class_method(cls, param):
        """Receives the class as first argument."""
        return value
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `def method_name` | Function definition inside class body |
| `self` | First parameter; reference to the instance |
| `@staticmethod` | Decorator; method doesn't receive implicit first arg |
| `@classmethod` | Decorator; method receives class as first arg (`cls`) |
| `return` | Optional return value |

**Syntax Rules:**
- Instance methods must have `self` as the first parameter.
- Method names use `snake_case` convention.
- Methods are called using `object.method(args)`; `self` is passed automatically.
- `@staticmethod` methods are called on the class or instance without implicit arguments.
- `@classmethod` methods receive the class (`cls`) as the first argument.

**Constraints and Limitations:**
- Calling an instance method on the class requires explicitly passing an instance: `ClassName.method(instance, args)`.
- Static methods cannot access instance or class state.
- Method names must be valid Python identifiers.

### Annotated Code Examples

**Example 1: Instance Methods**

```python
class BankAccount:
    """A simple bank account."""
    
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
    
    def deposit(self, amount):
        """Add money to the account."""
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.balance += amount
        return f"Deposited ${amount}. New balance: ${self.balance}"
    
    def withdraw(self, amount):
        """Remove money from the account."""
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        return f"Withdrew ${amount}. New balance: ${self.balance}"
    
    def get_balance(self):
        """Return the current balance."""
        return self.balance

# Usage
account = BankAccount("Alice", 1000)
print(account.get_balance())        # 1000
print(account.deposit(500))         # Deposited $500. New balance: $1500
print(account.withdraw(200))        # Withdrew $200. New balance: $1300
```

**Expected Output:**
```
1000
Deposited $500. New balance: $1500
Withdrew $200. New balance: $1300
```

**Why This Output:** Each method receives `self` (the `account` instance) automatically. The methods modify `self.balance` and return descriptive strings.

**Example 2: Static and Class Methods**

```python
class Temperature:
    """Temperature conversion utility."""
    
    def __init__(self, celsius):
        self.celsius = celsius
    
    def to_fahrenheit(self):
        """Instance method: uses self."""
        return (self.celsius * 9/5) + 32
    
    @staticmethod
    def celsius_to_fahrenheit(c):
        """Static method: no self or cls."""
        return (c * 9/5) + 32
    
    @classmethod
    def from_fahrenheit(cls, f):
        """Class method: receives the class."""
        celsius = (f - 32) * 5/9
        return cls(celsius)

# Instance method
t = Temperature(25)
print(f"{t.celsius}°C = {t.to_fahrenheit()}°F")

# Static method (called on class, no instance needed)
print(f"Static: 0°C = {Temperature.celsius_to_fahrenheit(0)}°F")

# Class method (alternative constructor)
t2 = Temperature.from_fahrenheit(77)
print(f"77°F = {t2.celsius:.1f}°C")
```

**Expected Output:**
```
25°C = 77.0°F
Static: 0°C = 32.0°F
77°F = 25.0°C
```

**Why This Output:** `to_fahrenheit` needs `self.celsius`. `celsius_to_fahrenheit` is a utility that doesn't need instance state. `from_fahrenheit` creates a new instance using the class reference.

### Real-World Cases

**Case 1: String Methods**
Python strings have methods like `.upper()`, `.split()`, and `.replace()`. These operate on the string object's data.

**Case 2: File Objects**
File objects have methods like `.read()`, `.write()`, and `.close()` that perform operations on the underlying file.

### References

- Python Documentation: Classes — https://docs.python.org/3/tutorial/classes.html
- Python Documentation: Built-in Types — https://docs.python.org/3/library/stdtypes.html

---

## 5. Instances

### Definitions

**Core Definition:** An instance is a specific object created from a class; it is a concrete realization of the class blueprint.

**Technical Definition:** An instance is an object whose type is a particular class. It is created by calling the class object (the constructor), which allocates memory, calls `__init__` to initialize attributes, and returns the instance identifier.

**Beginner-Friendly Explanation:** If a class is a cookie cutter, an instance is an actual cookie made from that cutter. You can make many cookies (instances) from one cutter (class), and each can have different icing (attribute values).

### Purposes

- To create concrete entities that can hold state and perform behavior.
- To allow multiple independent objects with the same structure.
- To enable object identity and reference semantics.
- To provide the runtime representation of class-defined types.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
# Creating an instance
instance_name = ClassName(argument1, argument2, ...)

# Checking instance type
isinstance(instance_name, ClassName)  # True

# Checking exact type
type(instance_name) is ClassName  # True
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `instance_name` | Variable referencing the created instance |
| `ClassName` | The class being instantiated |
| `arguments` | Values passed to `__init__` |
| `isinstance()` | Built-in function; checks inheritance relationship |
| `type()` | Built-in function; returns exact class |

**Syntax Rules:**
- Instances are created by calling the class like a function.
- Each instance has its own `__dict__` for instance attributes.
- Instances share methods defined in the class.
- The constructor returns the instance identifier (memory reference).

**Constraints and Limitations:**
- Abstract classes cannot be instantiated directly (raises `TypeError`).
- Instances of built-in immutable types cannot be modified in place.
- Instance attributes shadow class attributes with the same name.

### Annotated Code Examples

**Example 1: Instance Creation and Identity**

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

# Create two instances
r1 = Rectangle(10, 5)
r2 = Rectangle(10, 5)

# Same class, different identity
print(f"r1 is r2: {r1 is r2}")           # False (different objects)
print(f"r1 == r2: {r1 == r2}")           # False (no __eq__ defined)
print(f"type(r1): {type(r1)}")           # <class '__main__.Rectangle'>
print(f"isinstance(r1, Rectangle): {isinstance(r1, Rectangle)}")  # True

# Independent state
r1.width = 20
print(f"r1 area: {r1.area()}")  # 100
print(f"r2 area: {r2.area()}")  # 50
```

**Expected Output:**
```
r1 is r2: False
r1 == r2: False
type(r1): <class '__main__.Rectangle'>
isinstance(r1, Rectangle): True
r1 area: 100
r2 area: 50
```

**Why This Output:** `r1` and `r2` are distinct objects with separate memory locations. Modifying `r1.width` doesn't affect `r2.width`.

**Example 2: Instance Attribute Lookup**

```python
class Config:
    default_timeout = 30  # Class attribute
    
    def __init__(self, host):
        self.host = host  # Instance attribute

# Create instance
cfg = Config("api.example.com")

# Instance attributes vs class attributes
print(f"host: {cfg.host}")                    # Instance
print(f"timeout: {cfg.default_timeout}")      # Falls through to class

# Override class attribute on instance
cfg.default_timeout = 60
print(f"After override: {cfg.default_timeout}")  # Instance shadows class
print(f"Class still: {Config.default_timeout}")   # Class unchanged
```

**Expected Output:**
```
host: api.example.com
timeout: 30
After override: 60
Class still: 30
```

**Why This Output:** Attribute lookup checks the instance `__dict__` first. If not found, it checks the class and its bases. Setting `cfg.default_timeout` creates an instance attribute that shadows the class attribute.

### Real-World Cases

**Case 1: Database Connections**
Each database connection is an instance of a `Connection` class, with its own state (transaction status, cursor position) and methods (`execute()`, `commit()`).

**Case 2: Game Entities**
In a game, each enemy is an instance of an `Enemy` class, with independent health, position, and behavior.

### References

- Python Documentation: Classes — https://docs.python.org/3/tutorial/classes.html
- Python Tutorial: A Word About Names and Objects — https://docs.python.org/3/tutorial/classes.html#a-word-about-names-and-objects

---

## 6. Encapsulation (Public, Protected, and Private Access Modifiers via Name Mangling)

### Definitions

**Core Definition:** Encapsulation is the bundling of data and methods within a class, along with controlling access to the internal state.

**Technical Definition:** Python does not have true access modifiers like Java or C++. Instead, it uses naming conventions: single underscore (`_`) for protected (non-public API), and double underscore (`__`) for private, which triggers name mangling (the attribute `__name` becomes `_ClassName__name`). This is a mechanism for avoiding name clashes in inheritance, not a security measure.

**Beginner-Friendly Explanation:** Encapsulation is like a capsule that holds medicine. The capsule protects the medicine inside and controls how it's released. In Python, you can mark parts of a class as "internal use only" with underscores, signaling to other programmers "don't touch this directly" even though they technically could.

### Purposes

- To hide internal implementation details from external code.
- To prevent accidental interference with an object's internal state.
- To provide a clean, stable public interface.
- To avoid name clashes in inheritance hierarchies through name mangling.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    def __init__(self):
        self.public = "Anyone can access"
        self._protected = "Internal use; accessible but discouraged"
        self.__private = "Name-mangled to _ClassName__private"
    
    def _protected_method(self):
        """Non-public method."""
        pass
    
    def __private_method(self):
        """Name-mangled method."""
        pass
```

**Component Breakdown:**

| Convention | Example | Meaning |
|------------|---------|---------|
| No underscore | `name` | Public; part of the official API |
| Single underscore | `_name` | Protected; non-public, subject to change |
| Double underscore | `__name` | Private; name-mangled to `_ClassName__name` |
| Double underscore both sides | `__name__` | Special method (dunder); reserved by Python |

**Syntax Rules:**
- Name mangling applies to identifiers with two leading underscores and at most one trailing underscore.
- `__name` inside class `Foo` becomes `_Foo__name`.
- The mangling prevents accidental overriding in subclasses.
- Python does not enforce access restrictions; these are conventions.

**Constraints and Limitations:**
- Name mangling does not make attributes truly private; they can still be accessed via the mangled name.
- Single underscore has no syntactic effect; it's purely a convention.
- Deprecated/unsafe: Relying on name mangling for security is discouraged; Python trusts the programmer.

### Annotated Code Examples

**Example 1: Public, Protected, and Private Attributes**

```python
class Employee:
    def __init__(self, name, salary, ssn):
        self.name = name          # Public
        self._salary = salary     # Protected (convention)
        self.__ssn = ssn          # Private (mangled)
    
    def get_ssn_last_four(self):
        """Public method accessing private attribute."""
        return self.__ssn[-4:]
    
    def _calculate_bonus(self):
        """Protected method."""
        return self._salary * 0.1
    
    def __validate(self):
        """Private method."""
        return self.__ssn is not None

# Create instance
emp = Employee("Alice", 75000, "123-45-6789")

# Public access
print(f"Name: {emp.name}")           # OK
print(f"Bonus: {emp._calculate_bonus()}")  # OK but discouraged

# Protected access (allowed but signals "internal")
print(f"Salary: {emp._salary}")      # OK but discouraged

# Direct private access fails
try:
    print(emp.__ssn)
except AttributeError as e:
    print(f"Error: {e}")

# Access via mangled name (possible but strongly discouraged)
print(f"Mangled: {emp._Employee__ssn}")

# Public method accessing private data
print(f"Last 4 SSN: {emp.get_ssn_last_four()}")
```

**Expected Output:**
```
Name: Alice
Bonus: 7500.0
Salary: 75000
Error: 'Employee' object has no attribute '__ssn'
Mangled: 123-45-6789
Last 4 SSN: 6789
```

**Why This Output:** `__ssn` is mangled to `_Employee__ssn`. The direct `emp.__ssn` raises `AttributeError` because no such attribute exists in the instance dictionary. Accessing the mangled name works but is a strong signal that you're violating encapsulation.

**Example 2: Name Mangling in Inheritance**

```python
class Base:
    def __init__(self):
        self.__data = "Base data"
    
    def show(self):
        return f"Base: {self.__data}"  # Accesses _Base__data

class Derived(Base):
    def __init__(self):
        super().__init__()
        self.__data = "Derived data"  # Accesses _Derived__data
    
    def show_derived(self):
        return f"Derived: {self.__data}"

# Test
d = Derived()
print(d.show())           # Accesses Base's mangled attribute
print(d.show_derived())   # Accesses Derived's mangled attribute

# Both exist separately
print(f"Base data: {d._Base__data}")
print(f"Derived data: {d._Derived__data}")
```

**Expected Output:**
```
Base: Base data
Derived: Derived data
Base data: Base data
Derived data: Derived data
```

**Why This Output:** Name mangling prevents `Derived` from accidentally overwriting `Base`'s private attribute. Each class has its own mangled version.

### Real-World Cases

**Case 1: Library Design**
Libraries use `_` prefixes to indicate internal functions that users shouldn't call directly. The public API remains stable while internals can change.

**Case 2: Avoiding Name Conflicts**
When a class injects attributes into unrelated objects, double underscores prevent name collisions.

### References

- PEP 8: Naming Conventions — https://peps.python.org/pep-0008/#naming-conventions
- Python Tutorial: Private Variables — https://docs.python.org/3/tutorial/classes.html#private-variables

---

## 7. Abstraction

### Definitions

**Core Definition:** Abstraction is the process of hiding complex implementation details and exposing only the essential features of an object.

**Technical Definition:** In Python, abstraction is achieved through Abstract Base Classes (ABCs) defined in the `abc` module. An ABC cannot be instantiated and may contain abstract methods (decorated with `@abstractmethod`) that subclasses must implement. This enforces a contract or interface.

**Beginner-Friendly Explanation:** Abstraction is like driving a car. You use the steering wheel, pedals, and gear shift without needing to understand how the engine, transmission, and fuel injection work. The car provides a simplified interface (drive, brake, turn) while hiding the complexity underneath.

### Purposes

- To define a common interface for a group of related classes.
- To enforce that certain methods must be implemented by subclasses.
- To hide complex implementation details from users.
- To enable programming to an interface rather than an implementation.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
from abc import ABC, abstractmethod

class AbstractClassName(ABC):
    """Abstract base class."""
    
    @abstractmethod
    def required_method(self, args):
        """Must be implemented by subclasses."""
        pass
    
    def concrete_method(self):
        """Optional: provide default implementation."""
        return "default behavior"

class ConcreteClass(AbstractClassName):
    def required_method(self, args):
        """Concrete implementation."""
        return "implemented"
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `ABC` | Base class from `abc` module for defining ABCs |
| `@abstractmethod` | Decorator marking a method as abstract |
| `AbstractClassName` | Cannot be instantiated directly |
| `ConcreteClass` | Must implement all abstract methods |

**Syntax Rules:**
- Abstract classes inherit from `ABC` or use `metaclass=ABCMeta`.
- Abstract methods are decorated with `@abstractmethod`.
- Subclasses that don't implement all abstract methods remain abstract.
- Abstract classes can have concrete methods with implementations.

**Constraints and Limitations:**
- Instantiating an ABC with unimplemented abstract methods raises `TypeError`.
- ABCs add complexity; use only when you need to enforce an interface.
- Multiple inheritance with ABCs may cause metaclass conflicts.

### Annotated Code Examples

**Example 1: Shape Abstraction**

```python
from abc import ABC, abstractmethod
import math

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
        return f"A shape with area {self.area():.2f} and perimeter {self.perimeter():.2f}"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        return 2 * math.pi * self.radius

# Usage
# Shape() would raise TypeError
rect = Rectangle(5, 3)
circ = Circle(4)

print(rect.description())
print(circ.description())
```

**Expected Output:**
```
A shape with area 15.00 and perimeter 16.00
A shape with area 50.27 and perimeter 25.13
```

**Why This Output:** `Shape` defines the interface. `Rectangle` and `Circle` provide concrete implementations. The `description()` method works polymorphically with any `Shape` subclass.

**Example 2: Abstract Class Cannot Be Instantiated**

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def connect(self):
        pass
    
    @abstractmethod
    def execute(self, query):
        pass

# Attempt to instantiate abstract class
try:
    db = Database()
except TypeError as e:
    print(f"Error: {e}")

# Concrete implementation
class SQLiteDB(Database):
    def connect(self):
        return "Connected to SQLite"
    
    def execute(self, query):
        return f"Executed: {query}"

# This works
db = SQLiteDB()
print(db.connect())
print(db.execute("SELECT * FROM users"))
```

**Expected Output:**
```
Error: Can't instantiate abstract class Database with abstract methods connect, execute
Connected to SQLite
Executed: SELECT * FROM users
```

**Why This Output:** The ABC enforces that `connect` and `execute` are implemented before instantiation. `SQLiteDB` provides both, so it can be instantiated.

### Real-World Cases

**Case 1: Plugin Systems**
A plugin architecture defines an abstract `Plugin` class with `initialize()` and `execute()` methods. Each plugin implements these methods.

**Case 2: Payment Processing**
An abstract `PaymentProcessor` defines `process_payment()` and `refund()`. Concrete classes (`StripeProcessor`, `PayPalProcessor`) implement these for different providers.

### References

- Python Documentation: abc — Abstract Base Classes — https://docs.python.org/3/library/abc.html
- PEP 3119: Introducing Abstract Base Classes — https://peps.python.org/pep-3119/

---

## 8. Inheritance

### Definitions

**Core Definition:** Inheritance is a mechanism where a new class (subclass/derived class) acquires the attributes and methods of an existing class (superclass/base class).

**Technical Definition:** Python supports both single and multiple inheritance. A subclass inherits all accessible attributes and methods from its base classes. Method Resolution Order (MRO) determines the lookup order for methods and attributes, using the C3 linearization algorithm. The `super()` function provides access to methods in the MRO chain.

**Beginner-Friendly Explanation:** Inheritance is like a family tree. A child inherits traits from their parents. In programming, a `SportsCar` class can inherit from a `Car` class, getting all of Car's features (wheels, engine) while adding its own (turbo boost).

### Purposes

- To reuse code by inheriting from existing classes.
- To create a hierarchy of related types.
- To enable polymorphism through method overriding.
- To model "is-a" relationships (a SportsCar is a Car).

### Syntax Rules and Structure

**Complete General Syntax:**
```python
# Single inheritance
class BaseClass:
    pass

class SubClass(BaseClass):
    pass

# Multiple inheritance
class SubClass(Base1, Base2, Base3):
    pass

# Using super() to call parent methods
class SubClass(BaseClass):
    def method(self):
        super().method()  # Calls BaseClass.method
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `BaseClass` | Parent class whose members are inherited |
| `SubClass` | Child class that inherits |
| `super()` | Function returning a proxy to call parent methods |
| `mro()` | Class method returning the method resolution order |

**Syntax Rules:**
- A subclass inherits all non-private attributes and methods.
- Overriding replaces the parent's implementation in the subclass.
- `super()` should be used for cooperative multiple inheritance.
- MRO is computed using C3 linearization; accessible via `ClassName.mro()`.

**Constraints and Limitations:**
- The diamond problem in multiple inheritance is resolved by MRO.
- Deep inheritance hierarchies can become hard to maintain.
- `super()` requires careful use in multiple inheritance to avoid calling parent methods multiple times.

### Annotated Code Examples

**Example 1: Single Inheritance**

```python
class Animal:
    """Base class for animals."""
    
    def __init__(self, name, species):
        self.name = name
        self.species = species
    
    def speak(self):
        return "Some generic sound"
    
    def describe(self):
        return f"{self.name} is a {self.species}"

class Dog(Animal):
    """Dog inherits from Animal."""
    
    def __init__(self, name, breed):
        super().__init__(name, "Dog")  # Call parent __init__
        self.breed = breed
    
    def speak(self):
        """Override parent method."""
        return "Woof!"
    
    def fetch(self):
        """New method specific to Dog."""
        return f"{self.name} fetches the ball"

class Cat(Animal):
    def __init__(self, name):
        super().__init__(name, "Cat")
    
    def speak(self):
        return "Meow!"

# Usage
dog = Dog("Buddy", "Golden Retriever")
cat = Cat("Whiskers")

print(dog.describe())      # Inherited method
print(dog.speak())         # Overridden method
print(dog.fetch())         # Dog-specific method
print(cat.describe())      # Inherited
print(cat.speak())         # Cat's override
```

**Expected Output:**
```
Buddy is a Dog
Woof!
Buddy fetches the ball
Whiskers is a Cat
Meow!
```

**Why This Output:** `Dog` and `Cat` inherit `describe()` from `Animal` but override `speak()`. `Dog` adds `fetch()`. `super().__init__()` properly initializes the parent's attributes.

**Example 2: Method Resolution Order (MRO)**

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
    pass

# Check MRO
print("MRO:", [cls.__name__ for cls in D.mro()])

# D doesn't define method, so MRO determines which is used
d = D()
print(d.method())

# Call specific parent method
print(B.method(d))
print(C.method(d))
```

**Expected Output:**
```
MRO: ['D', 'B', 'C', 'A', 'object']
B.method
B.method
C.method
```

**Why This Output:** Python uses C3 linearization to determine MRO. `D` looks for `method` first in itself, then `B`, then `C`, then `A`. Since `B` is first in MRO, `B.method` is called.

### Real-World Cases

**Case 1: Web Framework Views**
Django's `View` class is subclassed for different view types (`TemplateView`, `ListView`). Each subclass inherits common HTTP handling while customizing behavior.

**Case 2: Exception Hierarchies**
Python's built-in exceptions form an inheritance hierarchy. `ValueError` inherits from `Exception`, allowing catch blocks to be as specific or general as needed.

### References

- Python Documentation: Inheritance — https://docs.python.org/3/tutorial/classes.html#inheritance
- Python 2.3 MRO — https://www.python.org/download/releases/2.3/mro/

---

## 9. Polymorphism (Duck Typing and Formal Polymorphism)

### Definitions

**Core Definition:** Polymorphism is the ability of different classes to respond to the same method call in different ways.

**Technical Definition:** Python supports several forms of polymorphism: (1) duck typing—if an object has the required method, it can be used regardless of its type; (2) formal polymorphism through inheritance and method overriding; (3) operator overloading via dunder methods. Unlike statically typed languages, Python does not support function overloading by signature.

**Beginner-Friendly Explanation:** Polymorphism means "many forms." If you ask different animals to "speak," a dog barks and a cat meows. The same command (`speak()`) produces different results depending on the object. You don't need to know the exact type—just that it can speak.

### Purposes

- To write flexible code that works with multiple types.
- To enable a single interface for different underlying implementations.
- To support extensibility without modifying existing code.
- To leverage Python's dynamic nature through duck typing.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
# Duck typing: no inheritance required
class Duck:
    def quack(self):
        return "Quack!"

class Person:
    def quack(self):
        return "I'm imitating a duck!"

def make_it_quack(thing):
    """Works with any object that has a quack method."""
    return thing.quack()

# Formal polymorphism through inheritance
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

# Operator overloading
class Vector:
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
```

**Component Breakdown:**

| Form | Mechanism | Example |
|------|-----------|---------|
| Duck typing | Method existence | `obj.method()` works if method exists |
| Inheritance-based | Method overriding | Subclass redefines parent method |
| Operator overloading | Dunder methods | `__add__`, `__str__`, `__eq__` |

**Syntax Rules:**
- Duck typing requires no inheritance; only method presence matters.
- Method overriding requires inheritance.
- Operator overloading uses special methods like `__add__`, `__sub__`, etc..

**Constraints and Limitations:**
- Python does not support traditional function overloading (same name, different signatures).
- Duck typing can lead to runtime errors if methods are missing.
- Operator overloading should follow conventions (e.g., `+` should be commutative where appropriate).

### Annotated Code Examples

**Example 1: Duck Typing**

```python
class FileWriter:
    def write(self, data):
        return f"Writing to file: {data}"

class ConsoleWriter:
    def write(self, data):
        return f"Console: {data}"

class NetworkWriter:
    def write(self, data):
        return f"Sending over network: {data}"

def output_data(writer, data):
    """Works with ANY object that has a write method."""
    return writer.write(data)

# All three work without sharing a common base class
fw = FileWriter()
cw = ConsoleWriter()
nw = NetworkWriter()

print(output_data(fw, "Hello"))
print(output_data(cw, "Hello"))
print(output_data(nw, "Hello"))

# Even built-in objects with write methods work
import sys
# sys.stdout has a write method
```

**Expected Output:**
```
Writing to file: Hello
Console: Hello
Sending over network: Hello
```

**Why This Output:** Python doesn't care about the type, only whether the object has the required method. This is duck typing: "If it walks like a duck and quacks like a duck, it's a duck".

**Example 2: Formal Polymorphism with Inheritance**

```python
class PaymentProcessor:
    def process(self, amount):
        raise NotImplementedError("Subclass must implement")

class CreditCardProcessor(PaymentProcessor):
    def process(self, amount):
        return f"Charged ${amount} to credit card"

class PayPalProcessor(PaymentProcessor):
    def process(self, amount):
        return f"Sent ${amount} via PayPal"

class CryptoProcessor(PaymentProcessor):
    def process(self, amount):
        return f"Transferred ${amount} in Bitcoin"

def checkout(processor, amount):
    """Works with any PaymentProcessor subclass."""
    return processor.process(amount)

# Polymorphic behavior
processors = [CreditCardProcessor(), PayPalProcessor(), CryptoProcessor()]
for p in processors:
    print(checkout(p, 100))
```

**Expected Output:**
```
Charged $100 to credit card
Sent $100 via PayPal
Transferred $100 in Bitcoin
```

**Why This Output:** Each subclass overrides `process()` with its own implementation. The `checkout()` function works with any `PaymentProcessor` without knowing the specific type.

**Example 3: Operator Overloading**

```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency
    
    def __add__(self, other):
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
    
    def __str__(self):
        return f"{self.amount} {self.currency}"
    
    def __eq__(self, other):
        return self.amount == other.amount and self.currency == other.currency

# Usage
m1 = Money(100, "USD")
m2 = Money(50, "USD")
m3 = Money(50, "USD")

total = m1 + m2  # Calls __add__
print(total)      # Calls __str__

print(m2 == m3)   # Calls __eq__
print(m1 == m3)   # Calls __eq__
```

**Expected Output:**
```
150 USD
True
False
```

**Why This Output:** `__add__` defines how `+` works for `Money` objects. `__str__` defines how `print()` displays them. `__eq__` defines equality comparison.

### Real-World Cases

**Case 1: File-like Objects**
Python's `open()` returns a file object, but functions like `json.dump()` accept any object with `write()` and `read()` methods, enabling testing with `io.StringIO`.

**Case 2: Iteration Protocol**
Any object implementing `__iter__` and `__next__` can be used in a `for` loop, enabling custom data structures to be iterable.

### References

- Python Documentation: Data Model — Special Method Names — https://docs.python.org/3/reference/datamodel.html#special-method-names
- Python Glossary: Duck Typing — https://docs.python.org/3/glossary.html#term-duck-typing

---

## Master Reference List

- Python Documentation: Data Model — https://docs.python.org/3/reference/datamodel.html
- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html
- PEP 8: Style Guide for Python Code — https://peps.python.org/pep-0008/
- PEP 3119: Introducing Abstract Base Classes — https://peps.python.org/pep-3119/
- Python Documentation: abc — Abstract Base Classes — https://docs.python.org/3/library/abc.html
- Python 2.3 MRO — https://www.python.org/download/releases/2.3/mro/
- Python Glossary — https://docs.python.org/3/glossary.html