# Method Types in Python — Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Methods in Python are functions defined within a class that define the behavior of instances, classes, or utility operations, distinguished by how they receive their first argument and when they are bound.

**Technical Definition:** Python methods are descriptors implementing the descriptor protocol (`__get__`). Instance methods bind `self`, class methods bind `cls` via `@classmethod`, static methods return the underlying function unchanged via `@staticmethod`, properties are data descriptors managing attribute access, and custom descriptors implement `__get__`/`__set__`/`__delete__` for full attribute control.

**Beginner-Friendly Explanation:** Methods are the "actions" an object or class can perform. Some need a specific object (instance methods), some need the class itself (class methods), some need neither (static methods), and properties let you treat methods like simple attributes.

### Key Characteristics

- **Binding Behavior:** Determines what the first argument receives (`self`, `cls`, or nothing).
- **Descriptor Protocol:** All method types are built on descriptors.
- **Property Magic:** `@property` creates data descriptors for managed attribute access.
- **Custom Descriptors:** The foundation for properties, methods, and `super()`.

### Prerequisites

- Basic class syntax and instance creation.
- Understanding of `self` and instance attributes.
- Familiarity with decorators (`@` syntax).

### Related Programming Areas

- API design and factory patterns.
- Data validation and ORM implementation.
- Attribute management and encapsulation.

---

## 1. Instance Methods (Operating on the Instance)

### Definitions

**Core Definition:** Instance methods are functions defined in a class that receive the instance as their first argument (`self`), operating on instance-specific state.

**Technical Definition:** Instance methods are non-data descriptors stored in the class `__dict__`. When accessed via an instance, the function's `__get__` method returns a bound method object that prepends the instance to the argument list. When accessed via the class, it returns the plain function requiring explicit instance passing.

**Beginner-Friendly Explanation:** Instance methods are the default kind of method. They work on a specific object, like `my_dog.bark()` — the method knows it's `my_dog` because Python passes it automatically.

### Purposes

- To access and modify instance-specific state.
- To provide behavior unique to each object.
- To call other methods on the same instance.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    def method_name(self, arg1, arg2):
        """Method operating on the instance."""
        self.attribute = value
        return result
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `self` | First parameter; reference to the instance |
| `arg1, arg2` | Additional explicit arguments |
| `self.attribute` | Access or modify instance attribute |

**Syntax Rules:**
- `self` must be the first parameter (convention, not enforced keyword).
- Called as `instance.method(args)` — Python passes `instance` as `self`.
- Equivalent explicit call: `ClassName.method(instance, args)`.

**Constraints and Limitations:**
- Without `self`, the method cannot access instance state.
- Forgetting `self` in the definition causes `TypeError` when called.

### Annotated Code Examples

```python
class BankAccount:
    """A bank account with instance methods."""
    
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
    
    def deposit(self, amount):
        """Add money to this account."""
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self.balance += amount
        return f"Deposited ${amount}. New balance: ${self.balance}"
    
    def withdraw(self, amount):
        """Remove money from this account."""
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        return f"Withdrew ${amount}. New balance: ${self.balance}"

# Create instance
account = BankAccount("Alice", 1000)

# Instance methods operate on the specific instance
print(account.deposit(500))    # Modifies account.balance
print(account.withdraw(200))   # Modifies account.balance
print(f"Final balance: {account.balance}")

# Equivalent explicit call
print(BankAccount.deposit(account, 100))
```

**Expected Output:**
```
Deposited $500. New balance: $1500
Withdrew $200. New balance: $1300
Final balance: $1300
Deposited $100. New balance: $1400
```

**Why This Output:** Each method receives `account` implicitly as `self`. The methods modify `self.balance` and return descriptive strings. The explicit call demonstrates the same binding.

### Real-World Cases

Django views, Flask routes, and custom data models all use instance methods extensively. For example, a `User` model might have an instance method `get_full_name()` that combines `self.first_name` and `self.last_name`.

### References

- Python Tutorial: Method Objects — https://docs.python.org/3/tutorial/classes.html#method-objects
- Python Data Model: Methods — https://docs.python.org/3/reference/datamodel.html

---

## 2. Class Methods (`@classmethod` and the `cls` Argument)

### Definitions

**Core Definition:** Class methods are methods that receive the class itself as their first argument (`cls`), rather than an instance, allowing them to operate on class-level state or create instances.

**Technical Definition:** `@classmethod` is a descriptor that wraps a function. When accessed via the class or an instance, it returns a bound method with the class (owner) as the first argument. This enables factory methods and operations that need class context without requiring an instance.

**Beginner-Friendly Explanation:** Class methods are like factory workers who work for the company (class), not for a specific product (instance). They can create new products or access company-wide information without needing an individual product.

### Purposes

- To create alternative constructors (factory methods).
- To access or modify class-level state.
- To provide methods that need class context but not instance context.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    @classmethod
    def method_name(cls, arg1, arg2):
        """Method receiving the class."""
        return cls(...)  # Can create instances
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `@classmethod` | Decorator marking the method as a class method |
| `cls` | First parameter; reference to the class |
| `cls(...)` | Can instantiate new objects |

**Syntax Rules:**
- `cls` is passed automatically; do not pass it explicitly when calling.
- Can be called on the class (`ClassName.method()`) or an instance (`instance.method()`).
- Commonly used for alternative constructors.

**Constraints and Limitations:**
- Cannot access instance-specific state (no `self`).
- When called via an instance, `cls` is still the class, not the instance.

### Annotated Code Examples

**Example 1: Alternative Constructor**

```python
class Date:
    """A date with flexible construction."""
    
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_string):
        """Create a Date from 'YYYY-MM-DD' format."""
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)  # Uses cls to create instance
    
    @classmethod
    def today(cls):
        """Create a Date for today."""
        import datetime
        today = datetime.date.today()
        return cls(today.year, today.month, today.day)
    
    def __str__(self):
        return f"{self.year}-{self.month:02d}-{self.day:02d}"

# Standard constructor
d1 = Date(2024, 1, 15)
print(f"Standard: {d1}")

# Alternative constructor via class method
d2 = Date.from_string("2024-03-20")
print(f"From string: {d2}")

# Works with subclasses too
class DateTime(Date):
    def __init__(self, year, month, day, hour=0):
        super().__init__(year, month, day)
        self.hour = hour
    
    def __str__(self):
        return f"{super().__str__()} {self.hour:02d}:00"

dt = DateTime.from_string("2024-03-20")
print(f"Subclass: {dt}")  # Uses DateTime's __init__
```

**Expected Output:**
```
Standard: 2024-01-15
From string: 2024-03-20
Subclass: 2024-03-20 00:00
```

**Why This Output:** `from_string` uses `cls` to call the appropriate constructor. When called on `DateTime`, `cls` is `DateTime`, so it creates a `DateTime` instance, not a `Date`.

**Example 2: Class-Level State Management**

```python
class Employee:
    """Tracks total employees at class level."""
    
    total_employees = 0
    
    def __init__(self, name):
        self.name = name
        Employee.total_employees += 1
    
    @classmethod
    def get_total(cls):
        """Access class-level state."""
        return cls.total_employees
    
    @classmethod
    def create_anonymous(cls):
        """Factory method for anonymous employees."""
        return cls("Anonymous")

# Create instances
emp1 = Employee("Alice")
emp2 = Employee("Bob")

print(f"Total employees: {Employee.get_total()}")

# Create via class method
emp3 = Employee.create_anonymous()
print(f"Total after anonymous: {Employee.get_total()}")
print(f"Anonymous name: {emp3.name}")
```

**Expected Output:**
```
Total employees: 2
Total after anonymous: 3
Anonymous name: Anonymous
```

**Why This Output:** `get_total` accesses `cls.total_employees`, which is the class attribute. `create_anonymous` uses `cls("Anonymous")` to create a new instance.

### Real-World Cases

- **Factory Methods:** `datetime.fromtimestamp()`, `dict.fromkeys()` are class methods.
- **ORM Queries:** Django's `Model.objects.get()` is a class method.
- **Plugin Registration:** `Plugin.create_from_config()` factory patterns.

### References

- Python Documentation: classmethod — https://docs.python.org/3/library/functions.html#classmethod
- PEP 318: Decorators for Functions and Methods — https://peps.python.org/pep-0318/

---

## 3. Static Methods (`@staticmethod` for Utility Functions)

### Definitions

**Core Definition:** Static methods are methods that neither receive the instance (`self`) nor the class (`cls`), functioning as regular functions logically grouped within the class namespace.

**Technical Definition:** `@staticmethod` is a descriptor whose `__get__` method returns the underlying function unchanged. It does not bind any implicit first argument, making it callable via the class or an instance without argument injection.

**Beginner-Friendly Explanation:** Static methods are like utility tools stored in the class's toolbox. They don't need to know about any specific object or the class itself — they just do a job that's related to the class conceptually.

### Purposes

- To group utility functions with related classes.
- To provide helper functions that don't need instance or class state.
- To improve code organization without creating module-level functions.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    @staticmethod
    def utility_name(arg1, arg2):
        """No self or cls."""
        return result
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `@staticmethod` | Decorator marking the method as static |
| `arg1, arg2` | Explicit arguments only (no implicit first arg) |

**Syntax Rules:**
- No `self` or `cls` parameter.
- Called as `ClassName.method(args)` or `instance.method(args)`.
- Cannot access instance or class state.

**Constraints and Limitations:**
- `staticmethod` doesn't "wrap" in the traditional sense; it's a descriptor that returns the original callable.
- Should be used sparingly; module-level functions are often clearer.

### Annotated Code Examples

```python
class MathUtils:
    """Utility functions for math operations."""
    
    @staticmethod
    def is_even(n):
        """Check if a number is even."""
        return n % 2 == 0
    
    @staticmethod
    def celsius_to_fahrenheit(celsius):
        """Convert Celsius to Fahrenheit."""
        return (celsius * 9/5) + 32
    
    @staticmethod
    def factorial(n):
        """Calculate factorial iteratively."""
        if n < 0:
            raise ValueError("Factorial not defined for negative numbers")
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

# Call via class
print(f"Is 4 even? {MathUtils.is_even(4)}")
print(f"25°C = {MathUtils.celsius_to_fahrenheit(25)}°F")
print(f"5! = {MathUtils.factorial(5)}")

# Call via instance (also works)
utils = MathUtils()
print(f"Is 7 even? {utils.is_even(7)}")
```

**Expected Output:**
```
Is 4 even? True
25°C = 77.0°F
5! = 120
Is 7 even? False
```

**Why This Output:** Static methods don't receive implicit arguments. They work identically whether called on the class or an instance because `__get__` returns the plain function.

### Real-World Cases

- **Validation helpers:** A `User` class might have `@staticmethod def validate_email(email)`.
- **Conversion utilities:** Temperature, currency, or unit conversions related to a domain class.
- **Factory helpers:** Functions that create objects but don't need class context.

### References

- Python Documentation: staticmethod — https://docs.python.org/3/library/functions.html#staticmethod
- Python-Dev Discussion: staticmethod behavior — https://mail.python.org/pipermail/python-dev/2008-June/080252.html

---

## 4. Properties (`@property`, `@<name>.setter`, `@<name>.deleter`)

### Definitions

**Core Definition:** Properties are managed attributes that allow methods to be accessed like attributes, providing controlled access to instance data with getter, setter, and deleter logic.

**Technical Definition:** `property()` is a built-in that creates a data descriptor implementing `__get__`, `__set__`, and `__delete__`. The `@property` decorator transforms a method into a getter; `.setter` and `.deleter` create new property objects with the respective methods attached.

**Beginner-Friendly Explanation:** Properties let you use `obj.attribute` syntax while running validation or computation behind the scenes. It's like having a smart door that looks like a normal door but checks your ID before letting you in.

### Purposes

- To provide attribute-style access with method-level control.
- To validate data before assignment.
- To compute values dynamically on access.
- To maintain backward compatibility when adding validation to existing attributes.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class ClassName:
    def __init__(self, value):
        self._value = value  # Private storage
    
    @property
    def value(self):
        """Getter: accessed as obj.value."""
        return self._value
    
    @value.setter
    def value(self, new_value):
        """Setter: called on obj.value = new_value."""
        self._value = new_value
    
    @value.deleter
    def value(self):
        """Deleter: called on del obj.value."""
        del self._value
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `@property` | Decorator creating the getter |
| `@value.setter` | Decorator creating the setter (same name) |
| `@value.deleter` | Decorator creating the deleter (same name) |
| `_value` | Private storage attribute (convention) |

**Syntax Rules:**
- The getter method name becomes the property name.
- Setter and deleter must use the same method name.
- `@property` must be defined before `.setter` or `.deleter`.
- The `@property` decorator returns a new property object each time; it doesn't modify in place.

**Constraints and Limitations:**
- Property names can shadow instance attribute names.
- Overusing properties can hurt performance for frequently accessed attributes.
- Cannot have a setter without a getter.

### Annotated Code Examples

**Example 1: Basic Getter/Setter with Validation**

```python
class Person:
    """A person with validated age."""
    
    def __init__(self, name, age):
        self.name = name
        self.age = age  # Calls the setter
    
    @property
    def age(self):
        """Getter for age."""
        print("Getting age")
        return self._age
    
    @age.setter
    def age(self, value):
        """Setter for age with validation."""
        print(f"Setting age to {value}")
        if not isinstance(value, int):
            raise TypeError("Age must be an integer")
        if value < 0:
            raise ValueError("Age cannot be negative")
        if value > 150:
            raise ValueError("Age seems unrealistic")
        self._age = value
    
    @age.deleter
    def age(self):
        """Deleter for age."""
        print("Deleting age")
        del self._age

# Usage
p = Person("Alice", 30)
print(f"Person: {p.name}, {p.age}")

# Setting with validation
try:
    p.age = -5
except ValueError as e:
    print(f"Error: {e}")

# Valid update
p.age = 31
print(f"Updated age: {p.age}")

# Deleting
del p.age
try:
    print(p.age)
except AttributeError as e:
    print(f"After delete: {e}")
```

**Expected Output:**
```
Setting age to 30
Getting age
Person: Alice, 30
Setting age to -5
Error: Age cannot be negative
Setting age to 31
Getting age
Updated age: 31
Deleting age
Getting age
After delete: 'Person' object has no attribute '_age'
```

**Why This Output:** The setter validates before storing in `_age`. The getter retrieves `_age`. The deleter removes `_age`, causing `AttributeError` on subsequent access.

**Example 2: Computed Property (No Setter)**

```python
class Circle:
    """A circle with computed properties."""
    
    def __init__(self, radius):
        self.radius = radius
    
    @property
    def area(self):
        """Computed property — no setter."""
        return 3.14159 * self.radius ** 2
    
    @property
    def diameter(self):
        """Computed property."""
        return 2 * self.radius

c = Circle(5)
print(f"Radius: {c.radius}")
print(f"Area: {c.area:.2f}")
print(f"Diameter: {c.diameter}")

# Attempting to set raises AttributeError
try:
    c.area = 100
except AttributeError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Radius: 5
Area: 78.54
Diameter: 10
Error: property 'area' of 'Circle' object has no setter
```

**Why This Output:** `area` and `diameter` are read-only computed properties. Attempting to assign raises `AttributeError` because no setter is defined.

### Real-World Cases

- **ORM Fields:** Django models use properties for computed fields.
- **Configuration Classes:** Validated settings with type checking.
- **API Wrappers:** Properties that make network calls or lazy-load data.

### References

- Python Documentation: property — https://docs.python.org/3/library/functions.html#property
- Descriptor HowTo Guide: Properties — https://docs.python.org/3/howto/descriptor.html#properties

---

## 5. Attribute Descriptors (`__get__`, `__set__`, `__delete__`)

### Definitions

**Core Definition:** Descriptors are objects that implement `__get__`, `__set__`, or `__delete__` to customize attribute access, storage, and deletion.

**Technical Definition:** The descriptor protocol allows an object stored as a class attribute to intercept attribute operations. Data descriptors (defining `__set__` or `__delete__`) take priority over instance dictionaries; non-data descriptors (only `__get__`) can be overridden by instance attributes.

**Beginner-Friendly Explanation:** Descriptors are the low-level machinery behind properties, methods, and class methods. They let you hook into what happens when someone reads, writes, or deletes an attribute.

### Purposes

- To implement reusable attribute management logic.
- To create validated attributes across multiple classes.
- To build the foundation for properties and ORMs.
- To customize attribute lookup for advanced use cases.

### Syntax Rules and Structure

**Complete General Syntax:**
```python
class Descriptor:
    def __init__(self, default=None):
        self.default = default
    
    def __get__(self, obj, objtype=None):
        """Called on attribute access."""
        if obj is None:
            return self  # Accessed via class
        return self.default
    
    def __set__(self, obj, value):
        """Called on attribute assignment."""
        self.default = value
    
    def __delete__(self, obj):
        """Called on attribute deletion."""
        del self.default

class Owner:
    attr = Descriptor(10)
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `__get__(self, obj, objtype)` | Called when attribute is read; `obj` is instance, `objtype` is class |
| `__set__(self, obj, value)` | Called when attribute is assigned |
| `__delete__(self, obj)` | Called when attribute is deleted |
| Data descriptor | Defines `__set__` or `__delete__` |
| Non-data descriptor | Defines only `__get__` |

**Syntax Rules:**
- Descriptor instances are class attributes, not instance attributes.
- `__get__` receives `None` for `obj` when accessed via the class.
- Data descriptors override instance `__dict__`; non-data descriptors do not.

**Constraints and Limitations:**
- Descriptors must be class-level attributes to function.
- Overriding `__getattribute__` bypasses descriptor protocol.
- Non-data descriptors can be shadowed by instance attributes.

### Annotated Code Examples

**Example 1: Logging Descriptor**

```python
class LoggedAccess:
    """A descriptor that logs attribute access."""
    
    def __init__(self, name, default=None):
        self.name = name
        self.default = default
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        print(f"Accessing {self.name}")
        return obj.__dict__.get(self.name, self.default)
    
    def __set__(self, obj, value):
        print(f"Setting {self.name} to {value}")
        obj.__dict__[self.name] = value
    
    def __delete__(self, obj):
        print(f"Deleting {self.name}")
        del obj.__dict__[self.name]

class Person:
    name = LoggedAccess("name")
    age = LoggedAccess("age", 0)

# Usage
p = Person()
p.name = "Alice"      # Calls __set__
print(p.name)         # Calls __get__
p.age = 30
print(p.age)
del p.name            # Calls __delete__
```

**Expected Output:**
```
Setting name to Alice
Accessing name
Alice
Setting age to 30
Accessing age
30
Deleting name
```

**Why This Output:** Each attribute access triggers the descriptor methods. The descriptor stores values in the instance `__dict__` but intercepts every operation.

**Example 2: Validated Descriptor**

```python
class ValidatedNumber:
    """Descriptor that validates numeric values within a range."""
    
    def __init__(self, min_value=0, max_value=100):
        self.min_value = min_value
        self.max_value = max_value
        self.data = {}  # Store per-instance values
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return self.data.get(id(obj), self.min_value)
    
    def __set__(self, obj, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"Expected number, got {type(value).__name__}")
        if not (self.min_value <= value <= self.max_value):
            raise ValueError(f"Value must be between {self.min_value} and {self.max_value}")
        self.data[id(obj)] = value

class Temperature:
    celsius = ValidatedNumber(-273, 100)
    
    def __init__(self, celsius):
        self.celsius = celsius
    
    def __str__(self):
        return f"{self.celsius}°C"

# Usage
t1 = Temperature(25)
t2 = Temperature(30)
print(f"t1: {t1}")
print(f"t2: {t2}")

# Validation works
try:
    t1.celsius = -500
except ValueError as e:
    print(f"Error: {e}")

# Values are independent per instance
t1.celsius = 50
print(f"t1 after update: {t1}")
print(f"t2 unchanged: {t2}")
```

**Expected Output:**
```
t1: 25°C
t2: 30°C
Error: Value must be between -273 and 100
t1 after update: 50°C
t2 unchanged: 30°C
```

**Why This Output:** The descriptor validates before storing. The `data` dictionary keyed by `id(obj)` maintains independent values per instance.

### Real-World Cases

- **ORMs:** SQLAlchemy uses descriptors to map database columns to Python attributes.
- **Type Checking:** Libraries like `attrs` and `pydantic` use descriptors for validation.
- **Lazy Loading:** Descriptors that load data from disk or network on first access.

### References

- Descriptor HowTo Guide — https://docs.python.org/3/howto/descriptor.html
- Python Data Model: Implementing Descriptors — https://docs.python.org/3/reference/datamodel.html#implementing-descriptors

---

## Master Reference List

- Python Documentation: Functions — https://docs.python.org/3/library/functions.html
- Descriptor HowTo Guide — https://docs.python.org/3/howto/descriptor.html
- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html
- PEP 318: Decorators — https://peps.python.org/pep-0318/
- Python-Dev: staticmethod behavior — https://mail.python.org/pipermail/python-dev/2008-June/080252.html
- Stack Overflow: @property confusion — https://stackoverflow.com/questions/79119460/need-help-on-property-in-python