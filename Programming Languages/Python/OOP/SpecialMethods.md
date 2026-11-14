# Python Magic Methods: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Magic methods—also called **dunder methods** (double underscore methods)—are special methods with fixed names that Python invokes automatically to implement built-in behavior for user-defined classes. They form the bridge between user-defined objects and Python's syntax, protocols, and built-in functions.

**Technical Definition**

Magic methods are methods whose names begin and end with double underscores, such as `__init__`, `__str__`, and `__add__`. They are not called directly by the programmer; instead, the interpreter invokes them in response to specific syntax or built-in operations. For example, `x + y` invokes `x.__add__(y)`, `len(x)` invokes `x.__len__()`, and `print(x)` invokes `x.__str__()`. This mechanism is Python's approach to operator overloading and protocol customization, allowing user-defined types to participate in the same operations as built-in types. Special methods are looked up on the object's **type**, not on the instance dictionary, which ensures consistent behavior across all instances of a class.

**Beginner-Friendly Explanation**

Magic methods are the secret handshakes that let your objects work with Python's built-in features. When you write `print(my_object)`, Python calls `my_object.__str__()`. When you write `len(my_object)`, it calls `__len__()`. When you write `my_object + other`, it calls `__add__()`. You do not call these methods yourself—Python calls them for you. By defining them, you make your objects behave like built-in types, using familiar syntax instead of awkward method calls.

### Key Characteristics

- **Implicit invocation**: Magic methods are called automatically by the interpreter in response to syntax or built-in functions.
- **Type-level lookup**: Special methods are looked up on the type, not the instance, ensuring consistent behavior.
- **Comprehensive coverage**: Magic methods exist for object lifecycle, representation, comparison, attribute access, containers, callables, context management, numeric operations, class customization, and pickling.
- **Operator overloading**: They enable user-defined types to define how operators like `+`, `==`, `<`, and `in` behave.
- **Protocol integration**: They connect user classes to Python's iteration, context management, and pickling protocols.
- **Returning `NotImplemented`**: Binary special methods should return `NotImplemented` when the operation is unsupported, allowing the interpreter to try the other operand.

### Prerequisites

- Basic Python syntax: classes, objects, methods, and inheritance.
- Understanding of the `self` parameter and attribute lookup.
- Familiarity with Python's built-in functions and operators.
- Awareness of Python's dynamic typing model.

### Related Programming Areas

- **Object-oriented programming**: Magic methods underpin Python's object model and operator overloading.
- **Protocols**: Iteration, context management, and pickling are all protocol-driven and rely on magic methods.
- **Design patterns**: Many patterns (e.g., iterator, context manager, visitor) are implemented via magic methods.
- **Data modeling**: Dataclasses, enums, and named tuples generate magic methods automatically.

---

## Core Concepts / Features

---

### 1. Magic Methods and Dunder Methods

#### Definitions

**Core Definition**

Magic methods, also called dunder methods, are special methods with names surrounded by double underscores that Python calls implicitly to implement built-in operations for user-defined objects.

**Technical Definition**

Magic methods are the mechanism by which Python supports operator overloading and protocol customization. Their names follow the pattern `__name__`, and they are invoked by the interpreter in specific contexts: construction (`__new__`, `__init__`), representation (`__str__`, `__repr__`), comparison (`__eq__`, `__lt__`), arithmetic (`__add__`, `__mul__`), containers (`__len__`, `__getitem__`), context management (`__enter__`, `__exit__`), and more. Special method lookup bypasses the instance dictionary and searches the type's MRO, ensuring that implicit invocations behave consistently across all instances. Setting a special method to `None` indicates that the corresponding operation is not available.

**Beginner-Friendly Explanation**

Think of magic methods as the "special reactions" your objects have to Python's syntax. When you use `+` on two objects, Python looks for the `__add__` method. When you use `len()`, it looks for `__len__`. You define these methods to tell Python how your objects should behave in these situations. The double underscores are like a secret code that tells Python, "This is a special method—use it automatically".

#### Purposes

- To enable user-defined objects to integrate with Python's built-in syntax and protocols.
- To provide operator overloading without special syntax.
- To customize object lifecycle, representation, comparison, and interaction with built-in functions.
- To make custom types interchangeable with built-in types in polymorphic code.
- To implement protocols such as iteration, context management, and pickling.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __magic_method__(self, ...):
        # Implementation
        ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__name__` | The special method name with double leading and trailing underscores. |
| `self` | The instance (for instance methods) or class (for class-level hooks). |
| Implicit call | The interpreter invokes the method automatically in response to syntax. |

**Syntax Rules**

- Magic methods are defined like ordinary methods but with special names.
- They are invoked implicitly; direct calls are possible but discouraged.
- Special method lookup uses the type, not the instance dictionary.
- Binary special methods should return `NotImplemented` for unsupported operand types.
- Setting a special method to `None` disables the corresponding operation.

**Constraints and Limitations**

- Magic methods cannot be arbitrarily named; only documented special names are recognized.
- Overusing magic methods can make code harder to understand.
- Special methods must be defined on the class, not the instance, to be reliably invoked.
- Not all operators can be overloaded (e.g., `and`, `or`, `not`).

#### Annotated Code Examples

**Example 1: Basic Magic Method Demonstration**

```python
class Demo:
    def __init__(self, value):
        self.value = value

    def __str__(self):
        return f"Demo({self.value})"

    def __add__(self, other):
        if not isinstance(other, Demo):
            return NotImplemented
        return Demo(self.value + other.value)

d1 = Demo(1)
d2 = Demo(2)
print(d1)           # Calls __str__
print(d1 + d2)      # Calls __add__, then __str__
```

**Expected Output:**

```
Demo(1)
Demo(3)
```

**Step-by-Step Breakdown:**

1. `Demo(1)` calls `__init__` to set `value`.
2. `print(d1)` calls `__str__`, which returns the formatted string.
3. `d1 + d2` calls `__add__`, which returns a new `Demo` with `value=3`.
4. The result is printed via `__str__`.

**Why the Output is Produced:**

Python translates `print(d1)` into `d1.__str__()` and `d1 + d2` into `d1.__add__(d2)`, invoking the defined magic methods.

#### Real-World Cases

- **Numeric types**: `decimal.Decimal` and `fractions.Fraction` define arithmetic magic methods.
- **Path objects**: `pathlib.Path` overloads `/` for path joining.
- **Custom containers**: Classes that implement `__len__`, `__getitem__`, and `__iter__` behave like built-in collections.
- **ORM models**: `__repr__` and `__eq__` provide useful debugging and comparison behavior.

---

### 2. Object Lifecycle: `__init__`, `__new__`, `__del__`

#### Definitions

**Core Definition**

The object lifecycle magic methods control how objects are created, initialized, and destroyed: `__new__` allocates the instance, `__init__` initializes it, and `__del__` finalizes it.

**Technical Definition**

`__new__(cls, ...)` is a static method that creates and returns a new instance of `cls`. If `__new__` returns an instance of `cls`, Python then calls `__init__(self, ...)` to initialize it. `__init__` must return `None`. `__del__(self)` is called when the instance is about to be destroyed, but its timing is not guaranteed (especially at interpreter shutdown or in reference cycles). `__new__` is rarely overridden except for immutable type subclassing, singletons, and object pools.

**Beginner-Friendly Explanation**

When you create an object, Python first **builds** the empty shell (`__new__`), then **fills it with data** (`__init__`). You almost always write `__init__`; you rarely need `__new__`. When the object is no longer needed, Python may call `__del__` before cleaning it up, but you should not rely on `__del__` for critical cleanup—use context managers instead.

#### Purposes

- To allocate and initialize new objects with the correct state.
- To customize object creation when subclassing immutable built-in types.
- To implement patterns like singletons and object pools via `__new__`.
- To release external resources when an object is destroyed.
- To understand the distinction between construction and initialization.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        instance = super().__new__(cls)
        return instance

    def __init__(self, value):
        self.value = value

    def __del__(self):
        # cleanup
        pass
```

**Component Breakdown**

| Method | Responsibility | Return Value |
|--------|----------------|--------------|
| `__new__(cls, ...)` | Create the instance | An instance (usually of `cls`) |
| `__init__(self, ...)` | Initialize the instance | `None` |
| `__del__(self)` | Finalize the instance | `None` |

**Syntax Rules**

- `__new__` is called first and returns the instance.
- If `__new__` returns an instance of `cls`, `__init__` is called automatically.
- If `__new__` returns something else, `__init__` is skipped.
- `__init__` must not return a non-`None` value.
- `__del__` should not raise exceptions.

**Constraints and Limitations**

- `__del__` is not guaranteed to be called.
- Exceptions in `__del__` are ignored with a warning.
- `del x` does not directly call `__del__`; it decrements the reference count.
- Using `__del__` for cleanup is fragile; context managers are preferred.

#### Annotated Code Examples

**Example 1: `__init__` Initialization**

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)
print(p.name, p.age)
```

**Expected Output:**

```
Alice 30
```

**Step-by-Step Breakdown:**

1. `Person("Alice", 30)` calls `__new__` to create the instance.
2. `__init__` sets `name` and `age` on the instance.

**Why the Output is Produced:**

`__init__` initializes the instance's attributes after creation.

---

**Example 2: `__new__` for a Singleton**

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

a = Singleton()
b = Singleton()
print(a is b)
```

**Expected Output:**

```
True
```

**Step-by-Step Breakdown:**

1. `__new__` checks whether an instance already exists.
2. If not, it creates one; otherwise, it returns the existing instance.
3. Both `a` and `b` refer to the same object.

**Why the Output is Produced:**

`__new__` controls allocation, enabling the singleton pattern.

#### Real-World Cases

- **Singleton pattern**: `__new__` ensures only one instance exists.
- **Immutable subclassing**: Subclassing `int` or `str` requires `__new__`.
- **Resource management**: `__del__` as a last-resort cleanup for files or sockets.
- **Object pooling**: `__new__` can reuse instances.

---

### 3. Object Representation: `__str__`, `__repr__`, `__format__`, `__bytes__`

#### Definitions

**Core Definition**

Object representation magic methods control how objects are converted to strings and bytes: `__str__` for human-friendly output, `__repr__` for developer-friendly output, `__format__` for f-strings and `format()`, and `__bytes__` for byte conversion.

**Technical Definition**

`__str__(self)` is called by `str()` and `print()` and should return a human-readable string. `__repr__(self)` is called by `repr()` and should ideally return a string that could recreate the object; it is also used for debugging and container display. `__format__(self, format_spec)` is called by `format()` and f-strings to apply format specifications. `__bytes__(self)` is called by `bytes()` and should return a `bytes` representation. If `__str__` is not defined, it falls back to `__repr__`.

**Beginner-Friendly Explanation**

`__str__` is how your object introduces itself to users. `__repr__` is how it introduces itself to developers—ideally something you could copy-paste to recreate it. `__format__` lets your object participate in f-strings with custom formatting. `__bytes__` lets you convert your object to raw bytes.

#### Purposes

- To provide readable output for end users with `__str__`.
- To provide unambiguous, debuggable output with `__repr__`.
- To support f-strings and `format()` with `__format__`.
- To enable byte-level serialization with `__bytes__`.
- To integrate custom objects with Python's string and formatting protocols.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __str__(self):
        return "human-readable"

    def __repr__(self):
        return f"MyClass(...)"

    def __format__(self, format_spec):
        return format(str(self), format_spec)

    def __bytes__(self):
        return b"..."
```

**Component Breakdown**

| Method | Triggered By | Purpose |
|--------|--------------|---------|
| `__str__` | `str(obj)`, `print(obj)` | Human-readable string |
| `__repr__` | `repr(obj)`, containers | Developer-friendly string |
| `__format__` | `format(obj, spec)`, f-strings | Custom formatting |
| `__bytes__` | `bytes(obj)` | Byte representation |

**Syntax Rules**

- `__str__` should return a human-readable string.
- `__repr__` should return a string that could ideally recreate the object.
- `__format__` receives the format specification string.
- `__bytes__` must return a `bytes` object.
- If `__str__` is not defined, `__repr__` is used as a fallback.

**Constraints and Limitations**

- `__repr__` should not raise exceptions.
- `__format__` should handle empty format specifications gracefully.
- `__bytes__` is rarely used; prefer `encode()` on string representations for text-based data.

#### Annotated Code Examples

**Example 1: `__str__` and `__repr__`**

```python
class Engineer:
    def __init__(self, name, rate):
        self.name = name
        self.rate = rate

    def __str__(self):
        return f"{self.name} (€{self.rate}/hour)"

    def __repr__(self):
        return f"Engineer(name='{self.name}', rate={self.rate})"

eng = Engineer("Willem", 116)
print(str(eng))
print(repr(eng))
print([eng])
```

**Expected Output:**

```
Willem (€116/hour)
Engineer(name='Willem', rate=116)
[Engineer(name='Willem', rate=116)]
```

**Step-by-Step Breakdown:**

1. `str(eng)` calls `__str__` for human-friendly output.
2. `repr(eng)` calls `__repr__` for developer-friendly output.
3. Printing a list uses `__repr__` on its elements.

**Why the Output is Produced:**

`__str__` is used by `print()` and `str()`, while `__repr__` is used by `repr()` and container display.

#### Real-World Cases

- **Debugging**: Informative `__repr__` makes logs and interactive sessions readable.
- **User interfaces**: `__str__` provides clean display strings.
- **Logging**: `__repr__` is preferred in logs for unambiguous context.
- **Container elements**: Lists, dicts, and sets use `__repr__` for their contents.

---

### 4. Object Comparison: `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__`, `__hash__`, `__bool__`

#### Definitions

**Core Definition**

Comparison magic methods define how objects compare for equality, ordering, hashing, and truthiness: `__eq__` and `__ne__` for equality, `__lt__`, `__le__`, `__gt__`, `__ge__` for ordering, `__hash__` for hash-based collections, and `__bool__` for truth value testing.

**Technical Definition**

Rich comparison methods (`__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__`) implement the comparison operators. If `__eq__` is defined without `__hash__`, the class becomes unhashable. `__hash__` must be consistent with `__eq__`: equal objects must have equal hashes. `__bool__` is called by `bool()` and in truth-value contexts; if not defined, Python falls back to `__len__`. These methods enable value-based comparison and use in dictionaries, sets, and conditional expressions.

**Beginner-Friendly Explanation**

These methods let your objects answer questions like "Are you equal to this other object?" (`__eq__`), "Are you less than that one?" (`__lt__`), "What is your hash code?" (`__hash__`), and "Are you truthy or falsy?" (`__bool__`). By default, objects compare by identity, but you can customize them to compare by value.

#### Purposes

- To define meaningful value-based equality and ordering.
- To make objects usable as dictionary keys or set members via consistent hashing.
- To support sorting and comparison operations.
- To control truthiness in conditional contexts.
- To integrate user-defined objects with Python's comparison protocols.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __eq__(self, other):
        if not isinstance(other, MyClass):
            return NotImplemented
        return self.value == other.value

    def __hash__(self):
        return hash(self.value)

    def __bool__(self):
        return bool(self.value)
```

**Component Breakdown**

| Method | Operator | Purpose |
|--------|----------|---------|
| `__eq__` | `==` | Equality |
| `__ne__` | `!=` | Inequality (auto-derived) |
| `__lt__` | `<` | Less than |
| `__le__` | `<=` | Less than or equal |
| `__gt__` | `>` | Greater than |
| `__ge__` | `>=` | Greater than or equal |
| `__hash__` | `hash()` | Hashing |
| `__bool__` | `bool()` | Truthiness |

**Syntax Rules**

- `__eq__` should return `NotImplemented` for unsupported types.
- Defining `__eq__` without `__hash__` sets `__hash__` to `None`.
- `__hash__` must be consistent with `__eq__`.
- `__bool__` must return a boolean.
- `__ne__` is auto-derived from `__eq__` in Python 3.

**Constraints and Limitations**

- Mutable objects generally should not be hashable.
- The `__eq__`/`__hash__` contract must be respected.
- Ordering methods should be consistent with `__eq__`.
- `__bool__` should not have side effects.

#### Annotated Code Examples

**Example 1: Equality, Ordering, and Hashing**

```python
class Version:
    def __init__(self, major, minor, patch):
        self.major = major
        self.minor = minor
        self.patch = patch

    def _key(self):
        return (self.major, self.minor, self.patch)

    def __eq__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return self._key() == other._key()

    def __lt__(self, other):
        if not isinstance(other, Version):
            return NotImplemented
        return self._key() < other._key()

    def __hash__(self):
        return hash(self._key())

v1 = Version(1, 2, 3)
v2 = Version(1, 3, 0)
print(v1 < v2)
print(v1 == v2)
print(len({v1, Version(1, 2, 3)}))
```

**Expected Output:**

```
True
False
1
```

**Step-by-Step Breakdown:**

1. `__eq__` compares the version tuples.
2. `__lt__` compares lexicographically.
3. `__hash__` hashes the tuple, so equal versions produce equal hashes and deduplicate in a set.

**Why the Output is Produced:**

Value-based comparison and consistent hashing make `Version` behave like a value type.

#### Real-World Cases

- **Value objects**: Money, dates, and coordinates need value equality and hashing.
- **Sorting**: Custom objects can be sorted with `sorted()`.
- **Dictionaries and sets**: Hashable objects can be used as keys or members.
- **Conditional logic**: `__bool__` controls `if obj:` behavior.

---

### 5. Attribute Access: `__getattr__`, `__setattr__`, `__delattr__`, `__getattribute__`, `__dir__`

#### Definitions

**Core Definition**

Attribute access magic methods control how attributes are retrieved, set, deleted, and listed: `__getattribute__` is called for every attribute access, `__getattr__` is called only when normal lookup fails, `__setattr__` for assignment, `__delattr__` for deletion, and `__dir__` for `dir()`.

**Technical Definition**

`__getattribute__(self, name)` is called unconditionally for every attribute access and is the primary hook for attribute retrieval. `__getattr__(self, name)` is called only when the attribute is not found through normal means (instance dictionary, class, base classes). `__setattr__(self, name, value)` is called for every attribute assignment. `__delattr__(self, name)` is called for attribute deletion. `__dir__(self)` is called by `dir()` and should return a list of attribute names. Overriding `__getattribute__` requires careful handling to avoid infinite recursion.

**Beginner-Friendly Explanation**

`__getattribute__` is the "always called" hook for attribute access. `__getattr__` is the "fallback" hook—called only when Python cannot find the attribute normally. `__setattr__` intercepts assignment, `__delattr__` intercepts deletion, and `__dir__` controls what `dir()` shows. These methods let you create objects with dynamic attributes, lazy loading, or proxy behavior.

#### Purposes

- To implement dynamic attribute access, lazy loading, or proxies.
- To intercept attribute assignment for validation or side effects.
- To provide fallback behavior for missing attributes with `__getattr__`.
- To customize the output of `dir()` for introspection.
- To implement descriptors and frameworks that rely on attribute hooks.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __getattribute__(self, name):
        return object.__getattribute__(self, name)

    def __getattr__(self, name):
        return f"default_{name}"

    def __setattr__(self, name, value):
        object.__setattr__(self, name, value)

    def __delattr__(self, name):
        object.__delattr__(self, name)

    def __dir__(self):
        return list(super().__dir__()) + ["extra"]
```

**Component Breakdown**

| Method | Trigger | Purpose |
|--------|---------|---------|
| `__getattribute__` | Every attribute access | Primary retrieval hook |
| `__getattr__` | Failed attribute lookup | Fallback retrieval |
| `__setattr__` | Attribute assignment | Intercept setting |
| `__delattr__` | Attribute deletion | Intercept deletion |
| `__dir__` | `dir(obj)` | Custom attribute listing |

**Syntax Rules**

- `__getattribute__` must call `object.__getattribute__` to avoid infinite recursion.
- `__getattr__` should raise `AttributeError` for genuinely missing attributes.
- `__setattr__` and `__delattr__` should delegate to `object` methods for normal behavior.
- `__dir__` should return a list of strings.

**Constraints and Limitations**

- Overriding `__getattribute__` is error-prone; use `__getattr__` when possible.
- `__getattr__` is not called for attributes found through normal lookup.
- `__setattr__` is called for every assignment, including those in `__init__`.

#### Annotated Code Examples

**Example 1: `__getattr__` Fallback**

```python
class LazyLoader:
    def __init__(self):
        self._loaded = {"name": "Alice"}

    def __getattr__(self, name):
        if name in self._loaded:
            return self._loaded[name]
        raise AttributeError(f"No attribute {name}")

obj = LazyLoader()
print(obj.name)          # Found in _loaded via __getattr__
try:
    obj.missing
except AttributeError as e:
    print("Error:", e)
```

**Expected Output:**

```
Alice
Error: No attribute missing
```

**Step-by-Step Breakdown:**

1. `obj.name` is not found normally, so `__getattr__` is called.
2. It checks `_loaded` and returns the value.
3. `obj.missing` is not in `_loaded`, so `AttributeError` is raised.

**Why the Output is Produced:**

`__getattr__` is invoked only when normal attribute lookup fails, providing a fallback mechanism.

#### Real-World Cases

- **Lazy loading**: Loading attributes only when accessed.
- **Proxies**: Forwarding attribute access to wrapped objects.
- **ORM models**: Dynamic attribute access for database columns.
- **Configuration objects**: Providing defaults for missing settings.

---

### 6. Container Methods: `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`, `__iter__`, `__next__`, `__reversed__`, `__missing__`

#### Definitions

**Core Definition**

Container magic methods allow user-defined classes to behave like built-in containers: `__len__` for length, `__getitem__`/`__setitem__`/`__delitem__` for indexing, `__contains__` for membership, `__iter__`/`__next__` for iteration, `__reversed__` for reversed iteration, and `__missing__` for dictionary-like missing-key behavior.

**Technical Definition**

`__len__(self)` is called by `len()` and should return a non-negative integer. `__getitem__(self, key)` supports subscript access and slicing. `__setitem__(self, key, value)` supports subscript assignment. `__delitem__(self, key)` supports subscript deletion. `__contains__(self, item)` supports the `in` operator. `__iter__(self)` returns an iterator and is called by `iter()`. `__next__(self)` returns the next item from an iterator and raises `StopIteration` when exhausted. `__reversed__(self)` returns a reverse iterator. `__missing__(self, key)` is called by `dict` subclasses when a key is missing. These methods enable custom objects to participate in Python's container protocols.

**Beginner-Friendly Explanation**

These methods make your objects act like lists, dictionaries, or other containers. `len(obj)` calls `__len__`, `obj[key]` calls `__getitem__`, `for x in obj` calls `__iter__`, and `x in obj` calls `__contains__`. By implementing them, your custom class can be used anywhere a built-in container is expected.

#### Purposes

- To create custom container types with list-like or dictionary-like behavior.
- To support indexing, slicing, and membership testing.
- To enable iteration with `for` loops and `iter()`.
- To support `reversed()` with `__reversed__`.
- To provide custom missing-key behavior in dictionary subclasses.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyContainer:
    def __len__(self): ...
    def __getitem__(self, key): ...
    def __setitem__(self, key, value): ...
    def __delitem__(self, key): ...
    def __contains__(self, item): ...
    def __iter__(self): ...
    def __next__(self): ...
    def __reversed__(self): ...
    def __missing__(self, key): ...
```

**Component Breakdown**

| Method | Trigger | Purpose |
|--------|---------|---------|
| `__len__` | `len(obj)` | Length |
| `__getitem__` | `obj[key]` | Subscript access |
| `__setitem__` | `obj[key] = value` | Subscript assignment |
| `__delitem__` | `del obj[key]` | Subscript deletion |
| `__contains__` | `item in obj` | Membership test |
| `__iter__` | `iter(obj)`, `for x in obj` | Iteration |
| `__next__` | `next(iterator)` | Next item |
| `__reversed__` | `reversed(obj)` | Reverse iteration |
| `__missing__` | Missing key in `dict` subclass | Default value |

**Syntax Rules**

- `__len__` must return a non-negative integer.
- `__getitem__` should support both integer indices and slices where appropriate.
- `__iter__` must return an iterator.
- `__next__` must raise `StopIteration` when exhausted.
- `__missing__` is only called by `dict` subclasses.

**Constraints and Limitations**

- `__len__` is limited to `sys.maxsize` by CPython.
- `__getitem__` without `__iter__` can still support iteration via the legacy sequence protocol, but `__iter__` is preferred.
- `__contains__` should be efficient; otherwise, iteration is used as a fallback.

#### Annotated Code Examples

**Example 1: Custom Container with `__len__`, `__getitem__`, and `__contains__`**

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs

    def __len__(self):
        return len(self.songs)

    def __getitem__(self, index):
        return self.songs[index]

    def __contains__(self, item):
        return item in self.songs

p = Playlist(["Song A", "Song B"])
print(len(p))
print(p[0])
print("Song B" in p)
```

**Expected Output:**

```
2
Song A
True
```

**Step-by-Step Breakdown:**

1. `len(p)` calls `__len__`, returning `2`.
2. `p[0]` calls `__getitem__(0)`, returning `"Song A"`.
3. `"Song B" in p` calls `__contains__`, returning `True`.

**Why the Output is Produced:**

These magic methods enable the object to behave like a read-only sequence.

#### Real-World Cases

- **Custom collections**: Ordered sets, trees, and linked lists.
- **Database result sets**: Lazy, iterable result objects.
- **Configuration mappings**: Dictionaries with default values via `__missing__`.
- **Matrix and array types**: Custom indexing and slicing.

---

### 7. Callable Objects: `__call__`

#### Definitions

**Core Definition**

`__call__` makes an object callable, allowing it to be invoked like a function.

**Technical Definition**

`__call__(self, *args, **kwargs)` is invoked when an instance is called with function-call syntax: `obj(*args, **kwargs)`. It can accept any arguments and return any value, making the object behave like a callable. Callable objects are useful for stateful functions, decorators with state, and class-based function factories.

**Beginner-Friendly Explanation**

`__call__` lets your objects act like functions. If you define `__call__`, you can write `obj(args)` instead of `obj.method(args)`. This is useful when an object needs to maintain state between calls or when you want to pass it where a function is expected.

#### Purposes

- To create stateful callables that remember information between calls.
- To implement function-like objects for use as callbacks.
- To build class-based decorators and function factories.
- To provide a natural interface for objects that represent operations.
- To integrate with APIs that expect callable objects.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyCallable:
    def __call__(self, *args, **kwargs):
        return ...
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `__call__(self, ...)` | The method invoked when the instance is called. |
| `*args`, `**kwargs` | Arbitrary arguments, matching function signatures. |

**Syntax Rules**

- `__call__` can accept any parameters.
- The return value becomes the result of the call expression.
- The object is callable if `__call__` is defined on its type.
- `callable(obj)` returns `True` if the type defines `__call__`.

**Constraints and Limitations**

- `__call__` should not have surprising side effects unless documented.
- Callable objects add indirection; simple functions may be clearer for stateless behavior.

#### Annotated Code Examples

**Example 1: Stateful Counter**

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self):
        self.count += 1
        return self.count

c = Counter()
print(c())
print(c())
print(c())
```

**Expected Output:**

```
1
2
3
```

**Step-by-Step Breakdown:**

1. `Counter()` creates a callable object with `count = 0`.
2. `c()` invokes `__call__`, incrementing and returning `count`.
3. The state persists across calls.

**Why the Output is Produced:**

`__call__` turns the instance into a callable that maintains internal state.

#### Real-World Cases

- **Decorators with state**: Class-based decorators that track calls.
- **Function factories**: Objects that generate specialized functions.
- **Callbacks**: Objects passed where functions are expected.
- **ML models**: Model objects that are callable for prediction.

---

### 8. Context Managers: `__enter__`, `__exit__`

#### Definitions

**Core Definition**

Context manager magic methods enable objects to be used with the `with` statement: `__enter__` runs on entry and `__exit__` runs on exit, including when exceptions occur.

**Technical Definition**

`__enter__(self)` is called when the `with` block is entered; its return value is bound to the `as` variable. `__exit__(self, exc_type, exc_value, traceback)` is called when the block exits, whether normally or via an exception. If `__exit__` returns a truthy value, the exception is suppressed. If the context exits without an exception, all three arguments are `None`. Context managers are defined by the context manager protocol (PEP 343).

**Beginner-Friendly Explanation**

A context manager is an object that knows how to set things up when you enter a `with` block and clean things up when you leave. `__enter__` does the setup, `__exit__` does the cleanup. Even if an error occurs inside the block, `__exit__` is still called, so cleanup always happens.

#### Purposes

- To ensure resources are properly acquired and released.
- To guarantee cleanup even when exceptions occur.
- To encapsulate setup and teardown logic.
- To support the `with` statement for custom resources.
- To suppress or handle exceptions in a controlled way.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyContext:
    def __enter__(self):
        return resource

    def __exit__(self, exc_type, exc_value, traceback):
        # cleanup
        return False  # or True to suppress
```

**Component Breakdown**

| Method | Trigger | Purpose |
|--------|---------|---------|
| `__enter__` | `with` block entry | Setup; returns resource |
| `__exit__` | `with` block exit | Cleanup; may suppress exceptions |

**Syntax Rules**

- `__enter__` returns the value bound to `as`.
- `__exit__` receives exception information or `None`s.
- Returning `True` from `__exit__` suppresses the exception.
- `__exit__` should not re-raise the passed exception.

**Constraints and Limitations**

- `__exit__` is not called if `__enter__` raises.
- Suppressing exceptions can hide bugs if used carelessly.
- Nested context managers require careful ordering.

#### Annotated Code Examples

**Example 1: File Context Manager**

```python
class ManagedFile:
    def __init__(self, path):
        self.path = path
        self.file = None

    def __enter__(self):
        self.file = open(self.path, "r")
        return self.file

    def __exit__(self, exc_type, exc_value, traceback):
        if self.file:
            self.file.close()
        return False

with ManagedFile("data.txt") as f:
    content = f.read()
```

**Expected Output:**

```
(File contents read; file closed on exit.)
```

**Step-by-Step Breakdown:**

1. `__enter__` opens the file and returns it.
2. The `with` block reads the content.
3. `__exit__` closes the file, even if an exception occurs.

**Why the Output is Produced:**

The context manager protocol guarantees cleanup via `__exit__`.

#### Real-World Cases

- **File handling**: Ensuring files are closed.
- **Database connections**: Committing or rolling back transactions.
- **Locks**: Acquiring and releasing threading locks.
- **Temporary state**: Saving and restoring global state.

---

### 9. Numeric Operations: Arithmetic, Bitwise, In-Place, Conversion, `__index__`, `__int__`, `__float__`, `__complex__`, `__round__`, `__trunc__`, `__floor__`, `__ceil__`

#### Definitions

**Core Definition**

Numeric magic methods define how objects behave with arithmetic operators (`+`, `-`, `*`, `/`, etc.), bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`), in-place operations (`+=`, `-=`, etc.), and numeric conversions (`int()`, `float()`, `complex()`, `round()`, `math.trunc()`, `math.floor()`, `math.ceil()`).

**Technical Definition**

Arithmetic methods include `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__floordiv__`, `__mod__`, `__pow__`, and their reflected (`__radd__`, etc.) and in-place (`__iadd__`, etc.) variants. Bitwise methods include `__and__`, `__or__`, `__xor__`, `__lshift__`, `__rshift__`, `__invert__`, and in-place variants. Conversion methods include `__int__`, `__float__`, `__complex__`, `__index__` (for integer-like objects used as indices), `__round__`, `__trunc__`, `__floor__`, and `__ceil__`. These methods enable user-defined numeric types to participate in Python's numeric protocols.

**Beginner-Friendly Explanation**

These methods let your objects do math. If you have a `Money` class, you can define `__add__` so `money1 + money2` works. If you have a `Vector`, you can define `__mul__` for scalar multiplication. Conversion methods let your object be converted to `int`, `float`, or `complex` when needed.

#### Purposes

- To implement arithmetic and bitwise operations for custom numeric types.
- To support in-place operations with augmented assignment.
- To define how objects convert to built-in numeric types.
- To integrate with Python's math functions and indexing.
- To enable numeric polymorphism.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyNumber:
    def __add__(self, other): ...
    def __radd__(self, other): ...
    def __iadd__(self, other): ...
    def __int__(self): ...
    def __float__(self): ...
    def __index__(self): ...
    def __round__(self, ndigits=None): ...
```

**Component Breakdown**

| Category | Methods |
|----------|---------|
| Arithmetic | `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__floordiv__`, `__mod__`, `__pow__` |
| Reflected | `__radd__`, `__rsub__`, `__rmul__`, etc. |
| In-place | `__iadd__`, `__isub__`, `__imul__`, etc. |
| Bitwise | `__and__`, `__or__`, `__xor__`, `__lshift__`, `__rshift__`, `__invert__` |
| Conversion | `__int__`, `__float__`, `__complex__`, `__index__`, `__round__`, `__trunc__`, `__floor__`, `__ceil__` |

**Syntax Rules**

- Binary methods should return `NotImplemented` for unsupported operands.
- Reflected methods handle the case where the left operand does not support the operation.
- In-place methods should return the modified object.
- `__index__` must return an integer.
- `__round__` receives an optional `ndigits` argument.

**Constraints and Limitations**

- Not all operators can be overloaded (e.g., `and`, `or`, `not`).
- In-place methods that return a new object should be named with the `i` prefix.
- Conversion methods should raise appropriate exceptions for invalid conversions.

#### Annotated Code Examples

**Example 1: Arithmetic and Conversion**

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        return Money(self.amount + other.amount)

    def __int__(self):
        return int(self.amount)

    def __repr__(self):
        return f"Money({self.amount})"

m1 = Money(10)
m2 = Money(5)
print(m1 + m2)
print(int(m1 + m2))
```

**Expected Output:**

```
Money(15)
15
```

**Step-by-Step Breakdown:**

1. `m1 + m2` calls `__add__`, returning a new `Money` with `amount=15`.
2. `int(m1 + m2)` calls `__int__`, returning `15`.

**Why the Output is Produced:**

`__add__` enables addition, and `__int__` enables conversion to `int`.

#### Real-World Cases

- **Financial types**: Money, currency, and decimals.
- **Geometry**: Vectors, matrices, and points.
- **Scientific computing**: Units and quantities.
- **Custom numeric types**: Fixed-point numbers, fractions, and complex types.

---

### 10. Class Methods: `__init_subclass__`, `__class_getitem__`, `__subclasshook__`

#### Definitions

**Core Definition**

Class-level magic methods customize class creation and behavior: `__init_subclass__` runs when a subclass is created, `__class_getitem__` supports generic type syntax (`MyClass[int]`), and `__subclasshook__` customizes `issubclass()` checks for abstract base classes.

**Technical Definition**

`__init_subclass__(cls, **kwargs)` is called on the parent class whenever a subclass is created, allowing automatic initialization or modification of subclasses (PEP 487). `__class_getitem__(cls, item)` is a class method that enables subscript syntax on the class itself, used for generic type hints (PEP 585). `__subclasshook__(cls, C)` is an abstract base class hook that determines whether `C` is considered a subclass of `cls`, enabling structural subtyping checks.

**Beginner-Friendly Explanation**

`__init_subclass__` lets a parent class automatically set up its children when they are defined. `__class_getitem__` makes `MyClass[int]` work, which is useful for type hints. `__subclasshook__` lets an abstract class say "I consider this other class a subclass" without requiring inheritance.

#### Purposes

- To automatically initialize or register subclasses without metaclasses.
- To support generic type syntax with square brackets.
- To customize subclass checks for abstract base classes.
- To implement class-level validation and registration.
- To provide structural subtyping without explicit inheritance.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        # initialize subclass

    def __class_getitem__(cls, item):
        return f"{cls.__name__}[{item}]"

    @classmethod
    def __subclasshook__(cls, C):
        if cls is MyABC:
            return hasattr(C, "required_method")
        return NotImplemented
```

**Component Breakdown**

| Method | Trigger | Purpose |
|--------|---------|---------|
| `__init_subclass__` | Subclass creation | Initialize or register subclasses |
| `__class_getitem__` | `Class[item]` | Generic type syntax |
| `__subclasshook__` | `issubclass()` check | Structural subclass checks |

**Syntax Rules**

- `__init_subclass__` is implicitly a class method and receives the subclass as `cls`.
- `__class_getitem__` receives the subscript item.
- `__subclasshook__` is a class method that returns `True`, `False`, or `NotImplemented`.
- `__init_subclass__` should call `super().__init_subclass__(**kwargs)` for cooperation.

**Constraints and Limitations**

- `__init_subclass__` cannot modify the class after creation; it runs during class definition.
- `__class_getitem__` is automatically a class method.
- `__subclasshook__` is used only by ABCs; it is not called for ordinary classes.

#### Annotated Code Examples

**Example 1: `__init_subclass__` Registration**

```python
class PluginBase:
    plugins = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        PluginBase.plugins.append(cls.__name__)

class PluginA(PluginBase):
    pass

class PluginB(PluginBase):
    pass

print(PluginBase.plugins)
```

**Expected Output:**

```
['PluginA', 'PluginB']
```

**Step-by-Step Breakdown:**

1. `__init_subclass__` is called when `PluginA` and `PluginB` are created.
2. It appends the subclass name to `PluginBase.plugins`.

**Why the Output is Produced:**

`__init_subclass__` provides a clean hook for subclass registration.

#### Real-World Cases

- **Plugin systems**: Auto-registering plugin subclasses.
- **Generic types**: Supporting `MyList[int]` syntax.
- **ABCs**: Customizing structural subclass checks.
- **Framework configuration**: Applying class-level settings to subclasses.

---

### 11. Pickling: `__reduce__`, `__reduce_ex__`, `__getstate__`, `__setstate__`

#### Definitions

**Core Definition**

Pickling magic methods customize how objects are serialized and deserialized: `__reduce__` and `__reduce_ex__` control the reduction process, while `__getstate__` and `__setstate__` control state extraction and restoration.

**Technical Definition**

`__reduce__()` is called by the pickler and must return either a string or a tuple describing how to reconstruct the object. The tuple typically contains a callable, arguments, and optional state, list items, and dict items. `__reduce_ex__(protocol)` is a protocol-aware variant. `__getstate__()` returns the object's state for pickling; if absent, `__dict__` is used. `__setstate__(state)` restores the object from the state returned by `__getstate__`. If `__setstate__` is absent, the state must be a dictionary and is assigned to `__dict__`.

**Beginner-Friendly Explanation**

Pickling is Python's way of saving objects to bytes. `__getstate__` tells Python what data to save; `__setstate__` tells it how to restore that data. `__reduce__` is the low-level control for specifying exactly how to rebuild the object. Most classes do not need these methods, but they are essential for objects with non-trivial state or resources that cannot be pickled directly.

#### Purposes

- To customize what data is pickled and how it is restored.
- To handle objects with resources that cannot be pickled (e.g., file handles).
- To implement custom serialization formats.
- To support versioning and migration of pickled data.
- To enable pickling of objects that would otherwise fail.

#### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyClass:
    def __getstate__(self):
        return {"key": self.key}

    def __setstate__(self, state):
        self.key = state["key"]

    def __reduce__(self):
        return (self.__class__, (self.arg,), self.__getstate__())
```

**Component Breakdown**

| Method | Trigger | Purpose |
|--------|---------|---------|
| `__getstate__` | Pickling | Return state for serialization |
| `__setstate__` | Unpickling | Restore state |
| `__reduce__` | Pickling | Full reconstruction control |
| `__reduce_ex__` | Pickling | Protocol-aware reconstruction |

**Syntax Rules**

- `__getstate__` can return any picklable object.
- `__setstate__` receives the state from `__getstate__`.
- `__reduce__` must return a string or a tuple of 2–5 (or 6) elements.
- `__reduce_ex__` receives the protocol version.

**Constraints and Limitations**

- `__reduce__` and `__reduce_ex__` are low-level and easy to misuse.
- If `__getstate__` returns a falsy value, `__setstate__` is not called for new-style classes.
- Pickle is not secure against untrusted data.

#### Annotated Code Examples

**Example 1: `__getstate__` and `__setstate__`**

```python
import pickle

class Resource:
    def __init__(self, name):
        self.name = name
        self.handle = None  # non-picklable resource

    def __getstate__(self):
        state = self.__dict__.copy()
        state.pop("handle", None)
        return state

    def __setstate__(self, state):
        self.__dict__.update(state)
        self.handle = None  # reinitialize

r = Resource("file.txt")
blob = pickle.dumps(r)
restored = pickle.loads(blob)
print(restored.name, restored.handle)
```

**Expected Output:**

```
file.txt None
```

**Step-by-Step Breakdown:**

1. `__getstate__` removes the non-picklable `handle`.
2. `__setstate__` restores the state and reinitializes `handle` to `None`.

**Why the Output is Produced:**

`__getstate__` excludes the problematic attribute, and `__setstate__` handles its reinitialization.

#### Real-World Cases

- **Database connections**: Excluding connection objects from pickled models.
- **ML models**: Custom state saving for model weights and configuration.
- **Caching**: Serializing complex objects with resources.
- **Version migration**: Handling state format changes across versions.

---

## References

- Python Documentation – Data Model: Special Method Names - https://docs.python.org/3/reference/datamodel.html#special-method-names
- Python Documentation – `object.__new__` - https://docs.python.org/3/reference/datamodel.html#object.__new__
- PEP 487 – Simpler customisation of class creation (`__init_subclass__`) - https://peps.python.org/pep-0487/
- PEP 343 – The "with" statement (context managers) - https://peps.python.org/pep-0343/
- Python Documentation – Context Managers (`__enter__`, `__exit__`) - https://docs.python.org/3/reference/datamodel.html#context-managers
- Python Documentation – Pickling (`__reduce__`, `__getstate__`, `__setstate__`) - https://docs.python.org/3/library/pickle.html
- Python Documentation – `__getattr__` vs `__getattribute__` - https://docs.python.org/3/reference/datamodel.html#object.__getattr__
- Python-list – Explanation of `__getattr__` and `__getattribute__` - https://mail.python.org/pipermail/python-list/2007-May/463785.html
- GitHub – Magic Methods (vanHeemstraSystems) - https://github.com/vanHeemstraSystems/learning-python-object-oriented/tree/main/700
- GitHub – Magic Methods Examples (choudaryhussainali) - https://raw.githubusercontent.com/choudaryhussainali/Mastering_PYTHON_in30Days/refs/heads/main/06-OOP/magic_methods.py
- Hydro-Informatics – Magic Methods Overview - https://raw.githubusercontent.com/hydro-informatics/hydro-informatics.github.io/main/jupyter/classes.ipynb
- Python Documentation – `__class_getitem__` and PEP 585 - https://peps.python.org/pep-0585/
- Python Documentation – `__subclasshook__` (ABCs) - https://docs.python.org/3/library/abc.html#abc.ABCMeta.__subclasshook__