# Python Descriptors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

A **descriptor** is an object attribute with “binding behavior”—an object whose class defines one or more of the special methods `__get__()`, `__set__()`, or `__delete__()`, allowing it to customize what happens when an attribute is accessed, assigned, or deleted on instances of another class.

**Technical Definition**

The descriptor protocol is a set of three methods—`__get__()`, `__set__()`, and `__delete__()`—plus the optional `__set_name__()` hook. An object that implements any of these methods is a descriptor. When a descriptor is stored as a class variable, Python’s attribute lookup machinery invokes the descriptor’s methods instead of performing a normal dictionary lookup. Descriptors are the underlying mechanism behind `property()`, functions (which become bound methods), `classmethod()`, `staticmethod()`, and `__slots__`. The distinction between **data descriptors** (which define `__set__()` and/or `__delete__()`) and **non-data descriptors** (which define only `__get__()`) determines their precedence relative to the instance dictionary during attribute lookup.

**Beginner-Friendly Explanation**

A descriptor is a “smart attribute” that lives on a class and intercepts every attempt to read, write, or delete it on an instance. Instead of simply storing a value in the instance’s dictionary, a descriptor runs your custom code each time the attribute is used. Think of it as a programmable property that you can reuse across multiple attributes and multiple classes. Many of Python’s most familiar features—`@property`, methods, `@classmethod`, and `@staticmethod`—are built on descriptors.

### Key Characteristics

- **Protocol-based**: Descriptors are defined by the methods they implement, not by inheritance from a specific base class.
- **Class-level objects**: A descriptor must be stored as a class attribute to be activated; storing it only on an instance does not invoke the protocol.
- **Lookup precedence**: Data descriptors take priority over the instance `__dict__`; non-data descriptors do not.
- **Reusability**: A single descriptor instance can manage multiple attributes across multiple classes.
- **Implicit invocation**: The descriptor methods are called automatically by the dot operator; the programmer does not call them directly.
- **Foundation of core features**: `property`, methods, `classmethod`, `staticmethod`, and `__slots__` are all implemented as descriptors.

### Prerequisites

- Solid understanding of Python classes, instances, and attribute lookup.
- Familiarity with `__dict__` and the difference between class attributes and instance attributes.
- Knowledge of special methods and the `self` parameter.
- Awareness of the `property()` built-in as a higher-level abstraction.

### Related Programming Areas

- **Properties**: `property()` is a built-in data descriptor; descriptors generalize its behavior.
- **Methods**: Functions are non-data descriptors that produce bound methods when accessed on instances.
- **Metaclasses**: Descriptors operate at the attribute level; metaclasses operate at the class-creation level.
- **ORMs and validation frameworks**: Django ORM fields and SQLAlchemy columns are descriptors that manage database-backed attributes.
- **Attribute access hooks**: `__getattr__`, `__setattr__`, and `__getattribute__` operate at the object level; descriptors operate at the attribute level.

---

## Core Concepts / Features

---

### 1. Descriptors

#### Definitions

**Core Definition**

A descriptor is an object that customizes attribute access when it is stored as a class variable in another class.

**Technical Definition**

A descriptor is an object whose class defines `__get__()`, `__set__()`, or `__delete__()`. When Python evaluates `obj.attr` and `attr` is a class variable that is a descriptor, the interpreter calls the descriptor’s `__get__()` method with the instance and owner class. For assignment (`obj.attr = value`), it calls `__set__()`; for deletion (`del obj.attr`), it calls `__delete__()`. The `__set_name__()` hook, if defined, is called at class creation time to inform the descriptor of the attribute name it was assigned to.

**Beginner-Friendly Explanation**

A descriptor is a reusable, programmable attribute. You define a small class that says “when someone reads this attribute, do X; when they write it, do Y.” Then you attach an instance of that class as a class variable. Every object of the enclosing class gets the smart behavior automatically.

#### Purposes

- To customize attribute lookup, storage, and deletion in a reusable way.
- To enforce validation, type checking, or transformation on attribute access.
- To implement lazy loading, caching, or computed attributes.
- To provide the underlying machinery for `property`, methods, `classmethod`, `staticmethod`, and `__slots__`.
- To enable frameworks (e.g., ORMs) to map class attributes to external resources.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        ...
    def __set__(self, obj, value):
        ...
    def __delete__(self, obj):
        ...
    def __set_name__(self, owner, name):
        ...

class Owner:
    attribute = Descriptor()
```

**Component Breakdown**

| Method | Signature | Invoked By |
|--------|-----------|------------|
| `__get__` | `(self, obj, objtype=None)` | Reading `obj.attribute` |
| `__set__` | `(self, obj, value)` | Assigning `obj.attribute = value` |
| `__delete__` | `(self, obj)` | `del obj.attribute` |
| `__set_name__` | `(self, owner, name)` | Class creation (automatic) |

**Syntax Rules**

- A descriptor must be a class variable of another class to be activated.
- `__get__` is called with `obj=None` when accessed on the class rather than an instance.
- `__set__` and `__delete__` make the descriptor a **data descriptor**, granting it priority over the instance dictionary.
- `__set_name__` is optional and receives the attribute name and owning class.

**Constraints and Limitations**

- Storing a descriptor only on an instance does not invoke the protocol; it must be a class attribute.
- Descriptors that define `__set__` or `__delete__` override instance dictionary entries.
- Non-data descriptors can be shadowed by instance attributes.
- `__get__` should handle `obj=None` gracefully (typically by returning the descriptor itself).

#### Annotated Code Examples

**Example 1: A Minimal Constant Descriptor**

```python
class Ten:
    def __get__(self, obj, objtype=None):
        return 10

class A:
    x = 5          # regular class attribute
    y = Ten()      # descriptor instance

a = A()
print(a.x)   # 5 — normal lookup
print(a.y)   # 10 — descriptor lookup
```

**Expected Output:**

```
5
10
```

**Step-by-Step Breakdown:**

1. `Ten` implements `__get__`, making it a descriptor.
2. `A.y = Ten()` stores a descriptor as a class variable.
3. `a.x` is a normal class attribute lookup, returning `5`.
4. `a.y` triggers `Ten.__get__(a, A)`, which returns `10`.

**Why the Output is Produced:**

The dot operator recognizes `y` as a descriptor and calls its `__get__` method instead of returning the descriptor object itself.

#### Real-World Cases

- **Django ORM fields**: `models.CharField()` is a descriptor that manages database-backed attributes.
- **SQLAlchemy columns**: `Column()` descriptors translate attribute access into SQL operations.
- **Validation frameworks**: Descriptors enforce type and range constraints on assignment.
- **Lazy loading**: Descriptors compute expensive values only on first access.

---

### 2. `__get__`

#### Definitions

**Core Definition**

`__get__(self, obj, objtype=None)` defines what happens when a descriptor-managed attribute is read.

**Technical Definition**

`__get__` is invoked when the attribute is accessed on an instance (`obj` is the instance) or on the class (`obj` is `None`). It should return the computed or stored value. For descriptors that manage instance data, `__get__` typically retrieves the value from the instance dictionary using a private attribute name. When accessed on the class, `__get__` commonly returns the descriptor itself so that introspection and help tools work correctly.

**Beginner-Friendly Explanation**

`__get__` is the “reader” method. Whenever someone asks for the attribute’s value, this method runs and decides what to give back. It can look up stored data, compute something fresh, or return a constant.

#### Purposes

- To return a stored value from the instance dictionary.
- To compute a value dynamically on each access.
- To implement lazy loading or caching.
- To return the descriptor itself when accessed on the class.
- To support read-only attributes when no `__set__` is defined.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
def __get__(self, obj, objtype=None):
    if obj is None:
        return self
    return obj.__dict__[self._private_name]
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `obj` | The instance (or `None` when accessed on the class). |
| `objtype` | The owner class (optional). |

**Syntax Rules**

- Return the descriptor itself when `obj is None` for class-level access.
- For instance access, retrieve or compute the value.
- Use `__set_name__` to learn the attribute name for private storage.

**Constraints and Limitations**

- `__get__` should not raise unexpected exceptions for missing attributes; return a default or raise `AttributeError` as appropriate.
- For non-data descriptors, instance `__dict__` entries can shadow the descriptor.

#### Annotated Code Examples

**Example 1: Dynamic Lookup**

```python
import os

class DirectorySize:
    def __get__(self, obj, objtype=None):
        return len(os.listdir(obj.dirname))

class Directory:
    size = DirectorySize()

    def __init__(self, dirname):
        self.dirname = dirname

s = Directory("songs")
print(s.size)  # number of files in "songs"
```

**Expected Output:**

```
(Number of files in the songs directory)
```

**Step-by-Step Breakdown:**

1. `DirectorySize.__get__` computes the directory size each time.
2. `s.size` triggers the computation, which reflects the current filesystem state.

**Why the Output is Produced:**

`__get__` runs a computation on every access, enabling dynamic, up-to-date values.

#### Real-World Cases

- **Computed attributes**: File sizes, counts, and derived metrics.
- **Lazy loading**: Deferring expensive computation until first access.
- **Read-only properties**: Defining `__get__` without `__set__`.

---

### 3. `__set__`

#### Definitions

**Core Definition**

`__set__(self, obj, value)` defines what happens when a descriptor-managed attribute is assigned.

**Technical Definition**

`__set__` is invoked on attribute assignment (`obj.attribute = value`). It allows the descriptor to validate, transform, or redirect the assignment. Defining `__set__` makes the descriptor a **data descriptor**, which takes precedence over the instance dictionary during attribute lookup. This means the descriptor’s `__set__` is always called, even if an instance dictionary entry exists.

**Beginner-Friendly Explanation**

`__set__` is the “writer” method. Whenever someone assigns a value to the attribute, this method runs. You can use it to check the value, convert it, or store it under a different name.

#### Purposes

- To validate values before storing them.
- To transform or normalize values on assignment.
- To redirect storage to a private attribute or external resource.
- To implement write-once or read-only attributes (by raising `AttributeError`).
- To enforce invariants in data models.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
def __set__(self, obj, value):
    if not isinstance(value, int):
        raise TypeError("Value must be an integer")
    obj.__dict__[self._private_name] = value
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `obj` | The instance whose attribute is being set. |
| `value` | The value being assigned. |

**Syntax Rules**

- `__set__` returns `None`.
- Store the actual data in the instance dictionary under a private name.
- Raise `ValueError` or `TypeError` for invalid values.

**Constraints and Limitations**

- Defining `__set__` (or `__delete__`) makes the descriptor a data descriptor, which overrides instance dictionary entries.
- Shared state pitfalls: descriptor instances are shared across all instances of the owner class; store per-instance data on the instance, not the descriptor.

#### Annotated Code Examples

**Example 1: Validated Attribute**

```python
class Positive:
    def __set_name__(self, owner, name):
        self._name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self._name, 0)

    def __set__(self, obj, value):
        if value < 0:
            raise ValueError("Value must be non-negative")
        setattr(obj, self._name, value)

class Product:
    price = Positive()

    def __init__(self, price):
        self.price = price

p = Product(10)
print(p.price)
p.price = 20
print(p.price)
# p.price = -5  # raises ValueError
```

**Expected Output:**

```
10
20
```

**Step-by-Step Breakdown:**

1. `Positive.__set_name__` records the private storage name.
2. `p.price = 10` calls `__set__`, which validates and stores under `_price`.
3. `p.price` calls `__get__`, retrieving `_price`.

**Why the Output is Produced:**

`__set__` intercepts assignment and enforces the non-negative constraint.

#### Real-World Cases

- **Form validation**: Ensuring email addresses contain `@`.
- **Numeric constraints**: Prices, ages, and quantities must be within bounds.
- **Type coercion**: Converting strings to integers or floats on assignment.

---

### 4. `__delete__`

#### Definitions

**Core Definition**

`__delete__(self, obj)` defines what happens when a descriptor-managed attribute is deleted with `del`.

**Technical Definition**

`__delete__` is invoked by `del obj.attribute`. It allows the descriptor to clean up associated state, remove private attributes, or raise an error to prevent deletion. Defining `__delete__` (with or without `__set__`) makes the descriptor a **data descriptor**. If `__delete__` is not defined, attempting to delete the attribute raises `AttributeError` (unless a non-data descriptor permits shadowing).

**Beginner-Friendly Explanation**

`__delete__` is the “cleanup” method. When someone uses `del` on the attribute, this method runs, letting you remove the stored value or refuse the deletion.

#### Purposes

- To clean up private storage when an attribute is deleted.
- To prevent deletion of critical attributes.
- To release external resources associated with the attribute.
- To maintain consistency between descriptor-managed state and instance state.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
def __delete__(self, obj):
    delattr(obj, self._private_name)
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `obj` | The instance from which the attribute is being deleted. |

**Syntax Rules**

- `__delete__` returns `None`.
- Use `delattr(obj, self._private_name)` to remove the stored value.
- Raise `AttributeError` to prevent deletion.

**Constraints and Limitations**

- Defining `__delete__` makes the descriptor a data descriptor.
- Failing to clean up can leave orphaned state.

#### Annotated Code Examples

**Example 1: Deleting a Managed Attribute**

```python
class Managed:
    def __set_name__(self, owner, name):
        self._name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self._name, None)

    def __set__(self, obj, value):
        setattr(obj, self._name, value)

    def __delete__(self, obj):
        print(f"Deleting {self._name}")
        delattr(obj, self._name)

class Example:
    attr = Managed()

e = Example()
e.attr = "value"
print(e.attr)
del e.attr
print(e.attr)
```

**Expected Output:**

```
value
Deleting _attr
None
```

**Step-by-Step Breakdown:**

1. `__set__` stores the value under `_attr`.
2. `__get__` retrieves it.
3. `del e.attr` calls `__delete__`, which prints a message and removes `_attr`.

**Why the Output is Produced:**

`__delete__` intercepts deletion and performs custom cleanup.

#### Real-World Cases

- **Session management**: Deleting session attributes cleans up associated resources.
- **Cache invalidation**: Removing cached values when the underlying data changes.
- **Protected attributes**: Preventing deletion of required fields.

---

### 5. `__set_name__`

#### Definitions

**Core Definition**

`__set_name__(self, owner, name)` is a hook called automatically at class creation time, informing the descriptor of the attribute name it was assigned to and the class that owns it.

**Technical Definition**

Introduced in Python 3.6 (PEP 487), `__set_name__` is called on every descriptor instance in a class body when the class is created. It receives the owning class (`owner`) and the attribute name (`name`) as written in the class definition. This allows descriptors to learn their own public name and derive a private storage name, eliminating the need to pass the name explicitly to the descriptor constructor.

**Beginner-Friendly Explanation**

`__set_name__` is how a descriptor learns its own name. Instead of hard-coding the attribute name in the descriptor, Python tells the descriptor “you were assigned to the name `price` in class `Product`.” The descriptor can then store data under a private name like `_price`.

#### Purposes

- To let descriptors learn their attribute name automatically.
- To derive private storage names without manual configuration.
- To support multiple descriptors of the same class without naming conflicts.
- To enable reusable descriptors that work for any attribute name.
- To eliminate boilerplate constructor arguments for the attribute name.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
def __set_name__(self, owner, name):
    self.public_name = name
    self.private_name = "_" + name
```

**Component Breakdown**

| Parameter | Description |
|-----------|-------------|
| `owner` | The class that owns the descriptor. |
| `name` | The attribute name as written in the class body. |

**Syntax Rules**

- `__set_name__` is called once per descriptor instance per class.
- It is called automatically; no explicit invocation is needed.
- Store the name for later use in `__get__`, `__set__`, and `__delete__`.

**Constraints and Limitations**

- `__set_name__` is available only in Python 3.6+.
- If a class has multiple descriptors, each receives its own name.
- Inheritance: `__set_name__` is called for descriptors in each class body where they appear.

#### Annotated Code Examples

**Example 1: Automatic Name Discovery**

```python
class Field:
    def __set_name__(self, owner, name):
        self.name = name
        self.private = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private, None)

    def __set__(self, obj, value):
        setattr(obj, self.private, value)

class User:
    username = Field()
    email = Field()

u = User()
u.username = "alice"
u.email = "alice@example.com"
print(u.username, u.email)
```

**Expected Output:**

```
alice alice@example.com
```

**Step-by-Step Breakdown:**

1. `__set_name__` is called twice: once for `username`, once for `email`.
2. Each `Field` instance stores its own public and private names.
3. `__get__` and `__set__` use the correct private name for each attribute.

**Why the Output is Produced:**

`__set_name__` eliminates the need to pass attribute names to the descriptor constructor, enabling a single descriptor class to manage multiple attributes.

#### Real-World Cases

- **Reusable validators**: One descriptor class used for `title`, `director`, and `year` attributes.
- **ORM fields**: Automatically mapping attribute names to column names.
- **Caching descriptors**: Storing cached values under names derived from the attribute name.

---

### 6. Data Descriptors vs. Non-Data Descriptors

#### Definitions

**Core Definition**

A **data descriptor** defines `__set__()` and/or `__delete__()` (in addition to `__get__()`), while a **non-data descriptor** defines only `__get__()`. Data descriptors take precedence over the instance dictionary; non-data descriptors do not.

**Technical Definition**

The classification is determined solely by which protocol methods the descriptor’s class defines. If `__set__()` or `__delete__()` is present, the descriptor is a data descriptor. If only `__get__()` is present, it is a non-data descriptor. During attribute lookup, Python searches the type’s MRO for data descriptors first, then checks the instance dictionary, and finally checks for non-data descriptors or class variables. This ordering explains why `property()` (a data descriptor) always intercepts assignment, while a method (a non-data descriptor) can be shadowed by an instance attribute.

**Beginner-Friendly Explanation**

Data descriptors are “stronger”: they control both reading and writing, and they always win over values stored directly on the instance. Non-data descriptors only control reading; an instance attribute with the same name can override them. Properties are data descriptors; methods are non-data descriptors.

#### Purposes

- To choose the appropriate level of control for an attribute.
- To understand why properties override instance attributes.
- To design descriptors that can or cannot be shadowed intentionally.
- To implement read-only, validated, or cached attributes with the correct precedence.
- To reason about Python’s attribute lookup order.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
# Data descriptor
class DataDesc:
    def __get__(self, obj, objtype=None): ...
    def __set__(self, obj, value): ...

# Non-data descriptor
class NonDataDesc:
    def __get__(self, obj, objtype=None): ...
```

**Component Breakdown**

| Descriptor Type | Methods Defined | Priority |
|-----------------|-----------------|----------|
| Data descriptor | `__get__` + (`__set__` or `__delete__`) | Over instance `__dict__` |
| Non-data descriptor | `__get__` only | Below instance `__dict__` |

**Syntax Rules**

- Data descriptors always intercept assignment, even if an instance dictionary entry exists.
- Non-data descriptors are shadowed by instance dictionary entries.
- The lookup order is: data descriptor → instance `__dict__` → non-data descriptor / class variable → `__getattr__`.

**Constraints and Limitations**

- Non-data descriptors cannot enforce validation on assignment.
- Data descriptors cannot be easily overridden by instance attributes, which may be desirable or undesirable depending on the design.

#### Annotated Code Examples

**Example 1: Precedence in Action**

```python
class DataDescriptor:
    def __get__(self, obj, objtype=None):
        return "data descriptor"
    def __set__(self, obj, value):
        print("DataDescriptor.__set__ called")

class NonDataDescriptor:
    def __get__(self, obj, objtype=None):
        return "non-data descriptor"

class Example:
    data = DataDescriptor()
    nondata = NonDataDescriptor()

e = Example()
print(e.data)          # data descriptor
print(e.nondata)       # non-data descriptor

e.__dict__["data"] = "instance value"
e.__dict__["nondata"] = "instance value"
print(e.data)          # still data descriptor
print(e.nondata)       # instance value (shadows non-data descriptor)
```

**Expected Output:**

```
data descriptor
non-data descriptor
data descriptor
instance value
```

**Step-by-Step Breakdown:**

1. `e.data` uses the data descriptor because data descriptors have priority.
2. `e.nondata` uses the non-data descriptor initially.
3. After adding `nondata` to `e.__dict__`, it shadows the non-data descriptor.
4. `data` remains controlled by the data descriptor.

**Why the Output is Produced:**

Data descriptors override the instance dictionary; non-data descriptors are overridden by it.

#### Real-World Cases

- **Properties**: Data descriptors that enforce validation and cannot be shadowed.
- **Methods**: Non-data descriptors that can be shadowed by instance attributes (rare but possible).
- **Caching**: Non-data descriptors that store computed values in the instance dictionary after first access.

---

### 7. Descriptor Use Cases

#### Definitions

**Core Definition**

Descriptor use cases are the practical applications where the descriptor protocol provides cleaner, more reusable, or more powerful solutions than alternative mechanisms such as properties or `__getattr__`.

**Technical Definition**

Descriptors are appropriate when the same attribute management logic must be reused across multiple attributes or multiple classes, when attribute access must be intercepted at the class level, or when implementing frameworks that map attributes to external resources. Common use cases include validation, lazy loading, caching, logging, type enforcement, and ORM field mapping. The official Python documentation notes that a popular use for descriptors is managing access to instance data, with the descriptor assigned to a public attribute in the class dictionary and the actual data stored as a private attribute in the instance dictionary.

**Beginner-Friendly Explanation**

Use descriptors when you find yourself writing the same property logic over and over, or when you need an attribute to behave in a special way across many classes. If you only need one or two properties, plain `@property` is simpler. Descriptors shine when the logic is repeated.

#### Purposes

- To reuse attribute management logic across many attributes and classes.
- To enforce consistent validation or transformation rules.
- To implement lazy loading and caching without duplicating code.
- To map attributes to external resources (databases, APIs, files).
- To provide framework-level infrastructure for declarative APIs.

#### Syntax Rules and Structure

A decision framework for choosing descriptors:

| Scenario | Recommended Approach |
|----------|----------------------|
| One-off validation on one attribute | `@property` |
| Same validation on many attributes | Descriptor |
| Lazy loading on many attributes | Descriptor (non-data) |
| ORM field mapping | Descriptor (data) |
| Attribute logging/profiling | Descriptor |
| Simple computed attribute | `@property` |

**Syntax Rules**

- Use descriptors when the logic is genuinely reusable.
- Prefer `@property` when the logic applies to a single attribute.
- Use `__set_name__` to make descriptors work for any attribute name.
- Store per-instance data on the instance, not on the descriptor.

**Constraints and Limitations**

- Descriptors add indirection; for simple cases, properties are clearer.
- Shared descriptor instances require careful state management.
- Debugging descriptor behavior can be harder than debugging properties.

#### Annotated Code Examples

**Example 1: Reusable Validation Descriptor**

```python
class Typed:
    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name
        self.private = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private, None)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"{self.name} must be {self.expected_type.__name__}")
        setattr(obj, self.private, value)

class Person:
    name = Typed(str)
    age = Typed(int)

p = Person()
p.name = "Alice"
p.age = 30
print(p.name, p.age)
```

**Expected Output:**

```
Alice 30
```

**Step-by-Step Breakdown:**

1. `Typed` is a reusable descriptor that enforces a type.
2. `__set_name__` records each attribute’s name.
3. `name` and `age` each get their own type enforcement.

**Why the Output is Produced:**

The same descriptor class enforces type checking across multiple attributes without duplicating validation logic.

#### Real-World Cases

- **Django ORM**: `CharField`, `IntegerField`, and `ForeignKey` are descriptors.
- **SQLAlchemy**: `Column` descriptors manage database-mapped attributes.
- **Caching decorators**: Descriptors that cache computed values per instance.
- **Audit logging**: Descriptors that log attribute reads and writes.

---

### 8. Properties as Descriptors

#### Definitions

**Core Definition**

`property()` is a built-in **data descriptor** that bundles a getter, setter, and deleter into a single attribute, providing the most common and convenient form of descriptor-based attribute management.

**Technical Definition**

The `property` class implements `__get__`, `__set__`, and `__delete__`, making every property instance a data descriptor. When you define `@property` in a class, Python creates a property object that intercepts attribute access. Because properties are data descriptors, they take precedence over the instance dictionary. The official Python documentation describes `property()` as one of the built-in tools implemented via descriptors.

**Beginner-Friendly Explanation**

A property is a ready-made descriptor. Instead of writing a descriptor class yourself, you use the `@property` decorator to turn methods into managed attributes. It is the “everyday” descriptor that most Python programmers use.

#### Purposes

- To provide getter, setter, and deleter logic for a single attribute.
- To validate or transform values on access.
- To create read-only attributes (getter only).
- To avoid boilerplate descriptor classes for simple cases.
- To serve as the standard, readable tool for attribute management.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, v):
        self._value = v

    @value.deleter
    def value(self):
        del self._value
```

**Component Breakdown**

| Decorator | Descriptor Method |
|-----------|-------------------|
| `@property` | Defines `__get__` |
| `@value.setter` | Defines `__set__` |
| `@value.deleter` | Defines `__delete__` |

**Syntax Rules**

- `property` is a data descriptor; it overrides instance attributes.
- The getter is defined first with `@property`.
- The setter and deleter are optional.
- Omitting the setter makes the property read-only.

**Constraints and Limitations**

- Properties are per-attribute; they do not easily generalize across many attributes.
- Property logic is embedded in the class, reducing reusability.
- For repeated logic, a custom descriptor is more appropriate.

#### Annotated Code Examples

**Example 1: Property as a Data Descriptor**

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Below absolute zero")
        self._celsius = value

t = Temperature(25)
print(t.celsius)
t.celsius = 30
print(t.celsius)
```

**Expected Output:**

```
25
30
```

**Step-by-Step Breakdown:**

1. `@property` creates a getter.
2. `@celsius.setter` creates a setter with validation.
3. `t.celsius = 30` calls the setter, which validates and stores.
4. `t.celsius` calls the getter.

**Why the Output is Produced:**

The property object is a data descriptor that intercepts both reads and writes, enforcing the validation rule.

#### Real-World Cases

- **Data validation**: Ensuring values meet domain rules.
- **Computed attributes**: `area`, `full_name`, `total`.
- **Read-only fields**: `id`, `created_at`.
- **Lazy properties**: Computing values on first access and caching them.

---

### 9. Methods as Descriptors

#### Definitions

**Core Definition**

Python functions are **non-data descriptors**. When a function is stored as a class attribute, accessing it on an instance produces a **bound method**—a callable that automatically passes the instance as the first argument (`self`).

**Technical Definition**

A function object implements `__get__`, making it a non-data descriptor. When accessed on an instance (`obj.method`), the function’s `__get__` returns a bound method object that wraps the function and the instance. When accessed on the class (`Class.method`), it returns the plain function. This is how Python implements the `self` binding mechanism. The same descriptor protocol powers `classmethod()` and `staticmethod()`, which use custom descriptor classes to alter binding behavior.

**Beginner-Friendly Explanation**

Methods are descriptors too! When you define `def method(self):` inside a class, Python turns it into a descriptor. Accessing it on an instance automatically binds `self`, which is why you don’t have to pass the instance explicitly.

#### Purposes

- To explain how `self` is automatically bound to methods.
- To understand why methods can be shadowed by instance attributes (non-data descriptors).
- To explain the behavior of `classmethod` and `staticmethod`.
- To provide the conceptual foundation for understanding all descriptor-based tools.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def method(self):
        return self
```

At the class level, `MyClass.method` is a plain function. At the instance level, `obj.method` is a bound method with `self` already bound.

**Component Breakdown**

| Access | Result |
|--------|--------|
| `MyClass.method` | Function object |
| `obj.method` | Bound method object |
| `obj.method()` | Calls function with `obj` as `self` |

**Syntax Rules**

- Functions are non-data descriptors; they define only `__get__`.
- Bound methods carry a reference to the instance.
- `classmethod` and `staticmethod` are descriptors that alter binding.

**Constraints and Limitations**

- Because methods are non-data descriptors, an instance attribute with the same name can shadow them.
- Bound methods hold a reference to the instance, which can create reference cycles.

#### Annotated Code Examples

**Example 1: Method Binding via Descriptor Protocol**

```python
class Example:
    def method(self):
        return f"Called on {self}"

e = Example()
print(Example.method)     # plain function
print(e.method)           # bound method
print(e.method())         # calls with e as self
```

**Expected Output:**

```
<function Example.method at ...>
<bound method Example.method of <__main__.Example object at ...>>
Called on <__main__.Example object at ...>
```

**Step-by-Step Breakdown:**

1. `Example.method` returns the function object.
2. `e.method` triggers the function’s `__get__`, which returns a bound method.
3. `e.method()` invokes the bound method, passing `e` as `self`.

**Why the Output is Produced:**

Functions are non-data descriptors; `__get__` returns a bound method when accessed on an instance.

#### Real-World Cases

- **All Python methods**: Every instance method relies on this descriptor mechanism.
- **Class methods**: `@classmethod` uses a custom descriptor to bind `cls`.
- **Static methods**: `@staticmethod` uses a descriptor that returns the plain function.

---

### 10. Descriptor Best Practices

#### Definitions

**Core Definition**

Descriptor best practices are guidelines for designing and using descriptors safely, efficiently, and maintainably, with emphasis on state management, reuse, and choosing the right tool for the job.

**Technical Definition**

Best practices include: storing per-instance data on the instance (not on the descriptor) to avoid shared-state bugs; using `__set_name__` to eliminate manual name configuration; preferring `@property` for one-off attribute logic and descriptors for genuinely reusable logic; documenting descriptor behavior clearly; being mindful of the data/non-data distinction and its effect on lookup precedence; and testing descriptors with both instance and class-level access. The official documentation emphasizes that descriptors should manage access to instance data by storing the actual data in the instance dictionary under a private name.

**Beginner-Friendly Explanation**

Best practices for descriptors are mostly about avoiding shared-state bugs and choosing the right level of complexity. Store data on the instance, not on the descriptor. Use `__set_name__` instead of hard-coding names. Use properties for simple cases and descriptors for repeated patterns. Document what the descriptor does.

#### Purposes

- To avoid shared-state bugs caused by storing data on the descriptor.
- To make descriptors reusable across multiple attributes and classes.
- To choose the appropriate tool (property vs. descriptor) for each scenario.
- To ensure descriptor behavior is clear and testable.
- To integrate descriptors cleanly with inheritance and class-level access.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class WellDesignedDescriptor:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value):
        # validate and store on obj, not on self
        setattr(obj, self.private_name, value)

    def __delete__(self, obj):
        delattr(obj, self.private_name)
```

**Component Breakdown**

| Practice | Recommendation |
|----------|----------------|
| State storage | Store per-instance data on `obj`, not `self` |
| Name discovery | Use `__set_name__` |
| Reuse | Prefer descriptors for repeated logic |
| Simplicity | Prefer `@property` for one-off logic |
| Testing | Test instance and class access |
| Documentation | Document validation and storage behavior |

**Syntax Rules**

- Never store mutable per-instance state on the descriptor instance.
- Use `__set_name__` to derive private names.
- Handle `obj is None` in `__get__`.
- Document whether the descriptor is data or non-data.

**Constraints and Limitations**

- Descriptors add conceptual overhead; do not overuse them.
- Shared state bugs are the most common descriptor pitfall.
- Non-data descriptors can be shadowed, which may or may not be intended.

#### Annotated Code Examples

**Example 1: Correct State Management**

```python
class Validated:
    def __set_name__(self, owner, name):
        self.public = name
        self.private = "_" + name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private, None)

    def __set__(self, obj, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.public} must be a string")
        setattr(obj, self.private, value)

class User:
    name = Validated()
    email = Validated()

u = User()
u.name = "Alice"
u.email = "alice@example.com"
print(u.name, u.email)
```

**Expected Output:**

```
Alice alice@example.com
```

**Step-by-Step Breakdown:**

1. `__set_name__` gives each descriptor its own public and private names.
2. `__set__` stores data on the instance (`u`), not on the descriptor.
3. Multiple instances remain independent.

**Why the Output is Produced:**

Per-instance storage prevents the shared-state bug that would occur if data were stored on the descriptor itself.

#### Real-World Cases

- **Framework field descriptors**: Django and SQLAlchemy store model data on instances, not on field descriptors.
- **Validation libraries**: Descriptors that enforce invariants without leaking state.
- **Caching descriptors**: Storing cached values on the instance to avoid cross-instance contamination.

---

## References

- Python Documentation – Descriptor HowTo Guide (Official) - https://docs.python.org/3/howto/descriptor.html
- Python Documentation – Data Model: Implementing Descriptors - https://docs.python.org/3/reference/datamodel.html#implementing-descriptors
- Real Python – Descriptor (Glossary) - https://realpython.com/ref/glossary/descriptor/
- Real Python – Python Descriptors: An Introduction - https://realpython.com/python-descriptors/
- Real Python – Exercise: Reusing a Descriptor Across Movie Attributes (`__set_name__`) - https://realpython.com/ref/glossary/descriptor/
- Descriptors — Deep Dive (ReddyBytes) - https://raw.githubusercontent.com/ReddyBytes/Python-DSA-API-Mastery/3512e8b11e4ac9d02463cf6ae48aec43ac129265/01_Python_Mastery/15_advanced_python/02_descriptors/theory.md
- Descriptor HowTo Guide (Python 3.11, official) - https://docs.python.org/3.11/howto/descriptor.html
- PEP 487 – Simpler customisation of class creation (`__set_name__`) - https://peps.python.org/pep-0487/