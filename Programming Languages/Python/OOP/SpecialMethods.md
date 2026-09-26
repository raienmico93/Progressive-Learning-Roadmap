# Special Methods (Dunder Methods) — Comprehensive Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Special methods (also called dunder methods or magic methods) are predefined method names surrounded by double underscores that allow user-defined classes to integrate with Python's built-in syntax and protocols.

**Technical Definition:** Special methods are hooks that the Python interpreter calls implicitly when certain syntax is used. Rather than calling `obj.__len__()` directly, you write `len(obj)` and Python dispatches to the special method. They enable user-defined types to behave like built-in types, supporting operations such as `+`, `in`, `[]`, iteration, and context management.

**Beginner-Friendly Explanation:** Dunder methods let your own classes "act like" Python's built-in types. If you define the right dunder methods, your object can be used with operators like `+`, work in a `for` loop, or be managed by a `with` statement.

### Key Characteristics

- **Interpreter-Driven:** Called by Python, not by the programmer directly.
- **Double Underscore Convention:** Names like `__init__`, `__str__`, `__add__`.
- **Protocol Integration:** Each dunder method corresponds to a specific language feature or operation.
- **Optional Implementation:** Most are optional; only define what your class needs.

### Prerequisites

- Basic class definition and instantiation.
- Understanding of `self` and method binding.
- Familiarity with built-in functions (`len()`, `str()`, `iter()`).

### Related Programming Areas

- Operator overloading.
- Collection design and custom containers.
- Resource management via context managers.
- Serialization and string representation.

---

## 1. String Representation (`__str__`, `__repr__`)

### Definitions

**Core Definition:** `__str__` provides a human-readable string representation of an object, while `__repr__` provides an unambiguous, developer-focused representation.

**Technical Definition:** `__str__` is called by `str()`, `format()`, and `print()`. `__repr__` is called by `repr()`, the interactive interpreter, and as a fallback when `__str__` is not defined. The goal of `__repr__` is to produce a string that could recreate the object.

**Beginner-Friendly Explanation:** `__repr__` is for programmers (debugging), `__str__` is for users (display). If you only define one, define `__repr__`.

### Purposes

- To provide readable output when objects are printed.
- To aid debugging with unambiguous representations.
- To support string formatting and logging.

### Syntax Rules and Structure

```python
class ClassName:
    def __str__(self):
        return "human-readable string"
    
    def __repr__(self):
        return "ClassName(args)"
```

**Component Breakdown:**
| Component | Description |
|-----------|-------------|
| `__str__` | Called by `str()` and `print()` |
| `__repr__` | Called by `repr()` and interactive display |
| Return type | Must return a string |

**Syntax Rules:**
- Both must return strings; raising `TypeError` otherwise.
- If `__str__` is undefined, Python falls back to `__repr__`.
- `__repr__` should ideally return valid Python code.

**Constraints and Limitations:**
- Cannot return non-string types.
- `__str__` should not be relied upon for serialization.

### Annotated Code Examples

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        """Unambiguous representation for developers."""
        return f"Point(x={self.x}, y={self.y})"
    
    def __str__(self):
        """Readable representation for users."""
        return f"({self.x}, {self.y})"

p = Point(3, 4)

# __repr__ called by repr() and interactive interpreter
print(f"repr: {repr(p)}")

# __str__ called by str() and print()
print(f"str: {str(p)}")
print(f"print: {p}")

# Fallback demonstration
class NoStr:
    def __repr__(self):
        return "NoStr()"

ns = NoStr()
print(f"Fallback: {ns}")  # Uses __repr__
```

**Expected Output:**
```
repr: Point(x=3, y=4)
str: (3, 4)
print: (3, 4)
Fallback: NoStr()
```

**Why This Output:** `repr()` explicitly calls `__repr__`. `print()` and `str()` call `__str__` when defined, otherwise falling back to `__repr__`.

### Real-World Cases

- **Dataclasses:** Automatically generate both methods.
- **Debugging:** `__repr__` helps inspect objects in logs.
- **User interfaces:** `__str__` controls how objects appear to end users.

### References

- Python Data Model: object.__str__ — https://docs.python.org/3/reference/datamodel.html#object.__str__
- Python Data Model: object.__repr__ — https://docs.python.org/3/reference/datamodel.html#object.__repr__

---

## 2. Collection Emulation (`__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`)

### Definitions

**Core Definition:** Collection emulation methods allow user-defined objects to behave like containers, supporting operations such as length, indexing, membership testing, and item assignment.

**Technical Definition:** Implementing `__len__` enables `len(obj)`. `__getitem__` enables `obj[key]` for reading. `__setitem__` enables `obj[key] = value`. `__delitem__` enables `del obj[key]`. `__contains__` enables `item in obj`. These methods integrate with Python's sequence and mapping protocols.

**Beginner-Friendly Explanation:** These methods make your object work like a list or dictionary. You can check its length, access items with square brackets, and test membership.

### Purposes

- To create custom container types.
- To enable iteration and slicing on custom objects.
- To support `in` operator and `len()` function.

### Syntax Rules and Structure

```python
class Container:
    def __len__(self):
        return count
    
    def __getitem__(self, key):
        return value
    
    def __setitem__(self, key, value):
        pass
    
    def __delitem__(self, key):
        pass
    
    def __contains__(self, item):
        return bool
```

**Component Breakdown:**
| Method | Trigger | Description |
|--------|---------|-------------|
| `__len__` | `len(obj)` | Returns integer length |
| `__getitem__` | `obj[key]` | Returns item or raises KeyError/IndexError |
| `__setitem__` | `obj[key] = value` | Sets item |
| `__delitem__` | `del obj[key]` | Deletes item |
| `__contains__` | `item in obj` | Returns True/False |

**Syntax Rules:**
- `__len__` must return a non-negative integer.
- `__getitem__` should raise `IndexError` for sequences or `KeyError` for mappings.
- `__contains__` should return a boolean.
- If `__contains__` is not defined, `in` falls back to iterating via `__getitem__` or `__iter__`.

**Constraints and Limitations:**
- `__len__` cannot return negative values.
- `__getitem__` with slices must handle `slice` objects.

### Annotated Code Examples

```python
class Playlist:
    """A music playlist behaving like a sequence."""
    
    def __init__(self, songs=None):
        self._songs = list(songs) if songs else []
    
    def __len__(self):
        """Called by len(playlist)."""
        return len(self._songs)
    
    def __getitem__(self, index):
        """Called by playlist[index] and slicing."""
        return self._songs[index]
    
    def __setitem__(self, index, song):
        """Called by playlist[index] = song."""
        self._songs[index] = song
    
    def __delitem__(self, index):
        """Called by del playlist[index]."""
        del self._songs[index]
    
    def __contains__(self, song):
        """Called by song in playlist."""
        return song in self._songs

# Usage
pl = Playlist(["Song A", "Song B", "Song C"])

print(f"Length: {len(pl)}")           # __len__
print(f"First: {pl[0]}")              # __getitem__
print(f"Slice: {pl[0:2]}")            # __getitem__ with slice

pl[1] = "Song B Remix"                # __setitem__
print(f"After set: {pl[1]}")

print(f"'Song A' in playlist: {'Song A' in pl}")  # __contains__

del pl[2]                             # __delitem__
print(f"After delete length: {len(pl)}")
```

**Expected Output:**
```
Length: 3
First: Song A
Slice: ['Song A', 'Song B']
After set: Song B Remix
'Song A' in playlist: True
After delete length: 2
```

**Why This Output:** Each method is triggered by the corresponding syntax. `__getitem__` handles both integers and slices. `__len__` returns the count. `__contains__` checks membership.

### Real-World Cases

- **Custom data structures:** Linked lists, trees, graphs.
- **ORM collections:** Query results that behave like lists.
- **Configuration mappings:** Settings accessed with `config['key']`.

### References

- Python Data Model: Emulating container types — https://docs.python.org/3/reference/datamodel.html#emulating-container-types
- collections.abc: Container ABC — https://docs.python.org/3/library/collections.abc.html#collections.abc.Container

---

## 3. Iteration Protocol (`__iter__`, `__next__`)

### Definitions

**Core Definition:** The iteration protocol allows objects to be iterated in `for` loops and other contexts, using `__iter__` to return an iterator and `__next__` to produce successive values.

**Technical Definition:** `__iter__` must return an iterator object. An iterator implements `__next__`, which returns the next value or raises `StopIteration` to signal exhaustion. Python's `for` loop calls `iter(obj)` then repeatedly calls `next()` on the result.

**Beginner-Friendly Explanation:** These methods make your object work in a `for` loop. `__iter__` says "start here" and `__next__` says "give me the next item."

### Purposes

- To enable `for` loops over custom objects.
- To support comprehensions, `sum()`, `max()`, and other iteration-based functions.
- To create custom iterators and generators.

### Syntax Rules and Structure

```python
class Iterable:
    def __iter__(self):
        return self  # or a separate iterator
    
    def __next__(self):
        if no_more_items:
            raise StopIteration
        return next_item
```

**Component Breakdown:**
| Method | Description |
|--------|-------------|
| `__iter__` | Returns an iterator object (often `self`) |
| `__next__` | Returns next value or raises `StopIteration` |

**Syntax Rules:**
- `__iter__` must return an object with `__next__`.
- `__next__` must raise `StopIteration` when exhausted.
- A class can be its own iterator (implement both methods).

**Constraints and Limitations:**
- Iterators are single-use; re-iterating requires a new iterator.
- `__iter__` returning `self` means the object is its own iterator.

### Annotated Code Examples

```python
class Countdown:
    """A countdown iterator."""
    
    def __init__(self, start):
        self.start = start
        self.current = start
    
    def __iter__(self):
        """Return self as the iterator."""
        return self
    
    def __next__(self):
        """Return next value or raise StopIteration."""
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

# Usage in for loop
print("Countdown:")
for num in Countdown(5):
    print(num)

# Works with other iteration contexts
print(f"Sum: {sum(Countdown(4))}")  # 4+3+2+1 = 10

# Iterators are single-use
c = Countdown(3)
print(f"First pass: {list(c)}")
print(f"Second pass: {list(c)}")  # Empty - exhausted
```

**Expected Output:**
```
Countdown:
5
4
3
2
1
Sum: 10
First pass: [3, 2, 1]
Second pass: []
```

**Why This Output:** `__iter__` returns `self`. `__next__` decrements and returns values until `current <= 0`, then raises `StopIteration`. The iterator is exhausted after the first pass.

### Real-World Cases

- **File readers:** Iterating over lines in a file.
- **Database cursors:** Streaming query results.
- **Generators:** Functions using `yield` create iterators automatically.

### References

- Python Data Model: Iterator protocol — https://docs.python.org/3/reference/datamodel.html#object.__iter__
- collections.abc: Iterator ABC — https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator

---

## 4. Object Lifecycle (`__new__`, `__init__`, `__del__`)

### Definitions

**Core Definition:** Lifecycle methods control object creation, initialization, and destruction: `__new__` creates, `__init__` initializes, and `__del__` finalizes.

**Technical Definition:** When a class is called, `__new__` is invoked first to allocate and return the object. If the returned object is an instance of the class, `__init__` is called to initialize it. `__del__` is called when the object is about to be destroyed (before garbage collection), though it is not guaranteed to be called.

**Beginner-Friendly Explanation:** `__new__` builds the object, `__init__` decorates it, and `__del__` is called when the object is about to disappear.

### Purposes

- To customize object creation for immutable types.
- To initialize instance attributes.
- To release external resources (with caution).

### Syntax Rules and Structure

```python
class ClassName:
    def __new__(cls, *args):
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, *args):
        pass
    
    def __del__(self):
        pass
```

**Component Breakdown:**
| Method | Description |
|--------|-------------|
| `__new__(cls, ...)` | Static method; creates and returns instance |
| `__init__(self, ...)` | Instance method; initializes attributes |
| `__del__(self)` | Called before destruction; no guarantees |

**Syntax Rules:**
- `__new__` must return an instance; if not an instance of `cls`, `__init__` is skipped.
- `__init__` must return `None`.
- `__del__` should not raise exceptions.

**Constraints and Limitations:**
- `__del__` timing is unpredictable; avoid relying on it for cleanup. Use context managers instead.
- `__new__` is rarely needed except for immutable subclasses.

### Annotated Code Examples

```python
class LifecycleDemo:
    def __new__(cls, value):
        print(f"__new__ called with {value}")
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, value):
        print(f"__init__ called with {value}")
        self.value = value
    
    def __del__(self):
        print(f"__del__ called for value={self.value}")

# Creation
print("Creating object:")
obj = LifecycleDemo(42)
print(f"Object value: {obj.value}")

# Deletion (may be delayed)
print("Deleting object:")
del obj
print("Done")
```

**Expected Output:**
```
Creating object:
__new__ called with 42
__init__ called with 42
Object value: 42
Deleting object:
__del__ called for value=42
Done
```

**Why This Output:** `__new__` runs first to create the object, then `__init__` initializes it. `del obj` triggers `__del__` before the object is freed.

### Real-World Cases

- **Singleton pattern:** `__new__` controls instance creation.
- **Resource management:** `__del__` as a fallback for cleanup (but context managers are preferred).
- **Immutable subclasses:** Validation in `__new__` for types like `int` or `str`.

### References

- Python Data Model: object.__new__ — https://docs.python.org/3/reference/datamodel.html#object.__new__
- Python C API: Object Life Cycle — https://docs.python.org/3/c-api/lifecycle.html

---

## 5. Rich Comparisons (`__eq__`, `__lt__`, `__le__`, `__gt__`, `__ge__`, `__ne__)

### Definitions

**Core Definition:** Rich comparison methods allow user-defined objects to be compared using operators like `==`, `<`, `>`, `<=`, `>=`, and `!=`.

**Technical Definition:** Python calls `__eq__` for `==`, `__lt__` for `<`, `__le__` for `<=`, `__gt__` for `>`, `__ge__` for `>=`, and `__ne__` for `!=`. If a method returns `NotImplemented`, Python tries the reflected operation on the other operand.

**Beginner-Friendly Explanation:** These methods define what it means for your objects to be equal, less than, or greater than each other.

### Purposes

- To define custom ordering for objects.
- To enable sorting and comparison.
- To make objects work with `min()`, `max()`, and `sorted()`.

### Syntax Rules and Structure

```python
class ClassName:
    def __eq__(self, other):
        return bool
    
    def __lt__(self, other):
        return bool
    
    # etc.
```

**Component Breakdown:**
| Method | Operator |
|--------|----------|
| `__eq__` | `==` |
| `__ne__` | `!=` |
| `__lt__` | `<` |
| `__le__` | `<=` |
| `__gt__` | `>` |
| `__ge__` | `>=` |

**Syntax Rules:**
- Return `NotImplemented` (not `False`) for unsupported types.
- If `__eq__` is defined, `__hash__` is set to `None` unless explicitly defined.
- `__ne__` defaults to the negation of `__eq__` if not defined.

**Constraints and Limitations:**
- Defining `__eq__` removes default hashability.
- Comparisons should be consistent (e.g., `a < b` and `b > a` should agree).

### Annotated Code Examples

```python
class Version:
    """A semantic version number."""
    
    def __init__(self, major, minor, patch):
        self.major = major
        self.minor = minor
        self.patch = patch
    
    def __repr__(self):
        return f"Version({self.major}.{self.minor}.{self.patch})"
    
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
    
    def __le__(self, other):
        return self == other or self < other
    
    def __gt__(self, other):
        return not (self <= other)
    
    def __ge__(self, other):
        return not (self < other)

# Usage
v1 = Version(1, 2, 3)
v2 = Version(1, 2, 4)

print(f"{v1} == {v2}: {v1 == v2}")
print(f"{v1} < {v2}: {v1 < v2}")
print(f"{v1} > {v2}: {v1 > v2}")

# Sorting works
versions = [Version(2, 0, 0), Version(1, 5, 0), Version(1, 2, 3)]
print(f"Sorted: {sorted(versions)}")
```

**Expected Output:**
```
Version(1.2.3) == Version(1.2.4): False
Version(1.2.3) < Version(1.2.4): True
Version(1.2.3) > Version(1.2.4): False
Sorted: [Version(1.2.3), Version(1.5.0), Version(2.0.0)]
```

**Why This Output:** `_key()` provides a tuple for comparison. `__eq__` and `__lt__` use it directly. `__le__`, `__gt__`, and `__ge__` derive from the first two. Sorting uses `__lt__`.

### Real-World Cases

- **Sorting domain objects:** Products by price, users by age.
- **Priority queues:** Tasks with priority values.
- **Version comparison:** Package managers comparing versions.

### References

- Python Data Model: Rich comparison methods — https://docs.python.org/3/reference/datamodel.html#object.__lt__
- functools.total_ordering — https://docs.python.org/3/library/functools.html#functools.total_ordering

---

## 6. Hashing and Truthiness (`__hash__`, `__bool__`)

### Definitions

**Core Definition:** `__hash__` defines an object's hash value for use in dictionaries and sets, while `__bool__` defines truthiness for use in `if` statements and boolean contexts.

**Technical Definition:** `__hash__` must return an integer. Objects that compare equal must have the same hash. `__bool__` is called by `bool()` and in conditional contexts; it must return `True` or `False`. If `__bool__` is not defined, Python falls back to `__len__`.

**Beginner-Friendly Explanation:** `__hash__` lets your objects be dictionary keys. `__bool__` determines whether your object is considered True or False.

### Purposes

- To make objects usable as dictionary keys or set members.
- To define custom truthiness for conditional logic.
- To ensure consistency between equality and hashing.

### Syntax Rules and Structure

```python
class ClassName:
    def __hash__(self):
        return int
    
    def __bool__(self):
        return bool
```

**Component Breakdown:**
| Method | Description |
|--------|-------------|
| `__hash__` | Returns integer hash; must be consistent with `__eq__` |
| `__bool__` | Returns True or False |

**Syntax Rules:**
- If `__eq__` is defined and `__hash__` is not, `__hash__` is set to `None` (unhashable).
- If `__bool__` is not defined, Python falls back to `__len__`.
- Hash values should be consistent for equal objects.

**Constraints and Limitations:**
- Mutable objects with `__eq__` should generally not be hashable.
- `__hash__` must return an integer, not a float or other type.

### Annotated Code Examples

```python
class User:
    """A user identified by ID."""
    
    def __init__(self, user_id, name):
        self.user_id = user_id
        self.name = name
    
    def __eq__(self, other):
        if not isinstance(other, User):
            return NotImplemented
        return self.user_id == other.user_id
    
    def __hash__(self):
        return hash(self.user_id)
    
    def __bool__(self):
        """Active if user_id is positive."""
        return self.user_id > 0

# Usage
u1 = User(1, "Alice")
u2 = User(1, "Alice Clone")
u3 = User(2, "Bob")

# Hash and equality
print(f"u1 == u2: {u1 == u2}")
print(f"hash(u1) == hash(u2): {hash(u1) == hash(u2)}")

# Works as dictionary key
users = {u1: "First", u3: "Second"}
print(f"Lookup by u2: {users[u2]}")  # u2 equals u1

# Truthiness
print(f"bool(u1): {bool(u1)}")
inactive = User(0, "Inactive")
print(f"bool(inactive): {bool(inactive)}")
```

**Expected Output:**
```
u1 == u2: True
hash(u1) == hash(u2): True
Lookup by u2: First
bool(u1): True
bool(inactive): False
```

**Why This Output:** `u1` and `u2` are equal and have the same hash, so `u2` retrieves the value stored with `u1`. `__bool__` returns `True` for positive IDs.

### Real-World Cases

- **Dictionary keys:** Custom objects used as keys.
- **Set membership:** Deduplicating custom objects.
- **Conditional logic:** Objects that should be treated as "empty" or "inactive."

### References

- Python Data Model: object.__hash__ — https://docs.python.org/3/reference/datamodel.html#object.__hash__
- Python Data Model: object.__bool__ — https://docs.python.org/3/reference/datamodel.html#object.__bool__

---

## 7. Arithmetic & Type Casting (`__add__`, `__sub__`, `__mul__`, `__truediv__`, `__int__`, `__float__`)

### Definitions

**Core Definition:** Arithmetic dunder methods enable operator overloading for `+`, `-`, `*`, `/`, and others, while `__int__` and `__float__` enable type casting to numeric types.

**Technical Definition:** Python calls `__add__` for `+`, `__sub__` for `-`, `__mul__` for `*`, `__truediv__` for `/`. `__int__` is called by `int()`, `__float__` by `float()`. Reflected operations (`__radd__`, etc.) handle cases where the left operand doesn't support the operation.

**Beginner-Friendly Explanation:** These methods let your objects use math operators and be converted to numbers.

### Purposes

- To define arithmetic behavior for custom types.
- To enable numeric conversion.
- To support mathematical operations and aggregations.

### Syntax Rules and Structure

```python
class ClassName:
    def __add__(self, other):
        return result
    
    def __radd__(self, other):
        return result
    
    def __int__(self):
        return int
```

**Component Breakdown:**
| Method | Operator / Function |
|--------|---------------------|
| `__add__` | `+` |
| `__sub__` | `-` |
| `__mul__` | `*` |
| `__truediv__` | `/` |
| `__int__` | `int()` |
| `__float__` | `float()` |

**Syntax Rules:**
- Return `NotImplemented` for unsupported operand types.
- Define `__radd__` for reversed operands (e.g., `5 + obj`).
- Type casting methods must return the appropriate numeric type.

**Constraints and Limitations:**
- `__int__` must return an `int`; `__float__` must return a `float`.
- Operators should behave consistently (commutativity where appropriate).

### Annotated Code Examples

```python
class Vector:
    """A 2D vector supporting arithmetic."""
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"
    
    def __add__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x + other.x, self.y + other.y)
        return NotImplemented
    
    def __sub__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x - other.x, self.y - other.y)
        return NotImplemented
    
    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented
    
    def __rmul__(self, scalar):
        return self * scalar
    
    def __truediv__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x / scalar, self.y / scalar)
        return NotImplemented
    
    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

# Usage
v1 = Vector(3, 4)
v2 = Vector(1, 2)

print(f"v1 + v2 = {v1 + v2}")
print(f"v1 - v2 = {v1 - v2}")
print(f"v1 * 2 = {v1 * 2}")
print(f"2 * v1 = {2 * v1}")  # Uses __rmul__
print(f"v1 / 2 = {v1 / 2}")
print(f"abs(v1) = {abs(v1)}")
```

**Expected Output:**
```
v1 + v2 = Vector(4, 6)
v1 - v2 = Vector(2, 2)
v1 * 2 = Vector(6, 8)
2 * v1 = Vector(6, 8)
v1 / 2 = Vector(1.5, 2.0)
abs(v1) = 5.0
```

**Why This Output:** `__add__` and `__sub__` handle Vector operands. `__mul__` and `__truediv__` handle scalars. `__rmul__` handles `2 * v1`. `__abs__` computes magnitude.

### Real-World Cases

- **Mathematics libraries:** NumPy arrays, sympy expressions.
- **Money/currency types:** Adding, subtracting, multiplying amounts.
- **Physics simulations:** Vector and matrix operations.

### References

- Python Data Model: Emulating numeric types — https://docs.python.org/3/reference/datamodel.html#emulating-numeric-types
- Python Data Model: object.__int__ — https://docs.python.org/3/reference/datamodel.html#object.__int__

---

## 8. Context-Manager Protocol (`__enter__`, `__exit__`, `__aenter__`, `__aexit__`)

### Definitions

**Core Definition:** The context-manager protocol enables objects to be used with the `with` statement for automatic setup and teardown of resources, with async variants for `async with`.

**Technical Definition:** `__enter__` is called when entering the `with` block and its return value is bound to the `as` variable. `__exit__` is called when leaving, receiving exception information if one occurred. Returning `True` from `__exit__` suppresses the exception. `__aenter__` and `__aexit__` are the async equivalents.

**Beginner-Friendly Explanation:** These methods let your object automatically handle setup and cleanup, like opening and closing a file.

### Purposes

- To manage resources (files, connections, locks) safely.
- To ensure cleanup happens even if errors occur.
- To provide async resource management.

### Syntax Rules and Structure

```python
class ContextManager:
    def __enter__(self):
        return resource
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        return False  # or True to suppress
    
    async def __aenter__(self):
        return resource
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        return False
```

**Component Breakdown:**
| Method | Description |
|--------|-------------|
| `__enter__` | Returns value bound to `as` |
| `__exit__` | Handles cleanup; return True suppresses exceptions |
| `__aenter__` | Async enter |
| `__aexit__` | Async exit |

**Syntax Rules:**
- `__exit__` receives `(exc_type, exc_val, exc_tb)` or three `None`s.
- Return `True` from `__exit__` to suppress exceptions; `False` or `None` to propagate.
- Async variants must be used with `async with`.

**Constraints and Limitations:**
- Suppressing exceptions can hide errors; use with caution.
- Async context managers cannot be used with sync `with`.

### Annotated Code Examples

```python
class ManagedFile:
    """A context manager for file handling."""
    
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        print(f"Opening {self.filename}")
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Closing {self.filename}")
        if self.file:
            self.file.close()
        if exc_type is not None:
            print(f"Exception occurred: {exc_type.__name__}: {exc_val}")
        return False  # Do not suppress exceptions

# Usage
with ManagedFile('test.txt', 'w') as f:
    f.write("Hello, World!")

print("File operation complete")

# Exception handling demonstration
try:
    with ManagedFile('nonexistent.txt', 'r') as f:
        content = f.read()
except FileNotFoundError as e:
    print(f"Caught: {e}")
```

**Expected Output:**
```
Opening test.txt
Closing test.txt
File operation complete
Opening nonexistent.txt
Closing nonexistent.txt
Exception occurred: FileNotFoundError: [Errno 2] No such file or directory: 'nonexistent.txt'
Caught: [Errno 2] No such file or directory: 'nonexistent.txt'
```

**Why This Output:** `__enter__` opens the file and returns it. `__exit__` closes it and reports exceptions. The exception propagates because `__exit__` returns `False`.

### Real-World Cases

- **File I/O:** `open()` returns a context manager.
- **Database connections:** Connection pools with automatic release.
- **Thread locks:** `threading.Lock()` supports `with`.
- **Async HTTP clients:** `aiohttp` sessions use `async with`.

### References

- Python Data Model: With Statement Context Managers — https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers
- contextlib: Utilities for with-statement contexts — https://docs.python.org/3/library/contextlib.html

---

## 9. Dynamic Attribute Access (`__getattr__`, `__getattribute__`, `__setattr__`)

### Definitions

**Core Definition:** Dynamic attribute access methods allow customization of how attributes are retrieved, set, or created on an object.

**Technical Definition:** `__getattribute__` is called for every attribute access and is responsible for the entire lookup. `__getattr__` is called only when normal lookup fails (AttributeError). `__setattr__` is called for every attribute assignment. `__getattr__` is the safest hook for dynamic behavior; `__getattribute__` requires extreme care to avoid infinite recursion.

**Beginner-Friendly Explanation:** These methods let you control what happens when someone reads or writes an attribute, including creating attributes on the fly.

### Purposes

- To delegate attribute access to another object.
- To provide computed or lazy attributes.
- To intercept and validate attribute assignments.

### Syntax Rules and Structure

```python
class ClassName:
    def __getattr__(self, name):
        # Called only when normal lookup fails
        return value
    
    def __getattribute__(self, name):
        # Called for EVERY attribute access
        return object.__getattribute__(self, name)
    
    def __setattr__(self, name, value):
        # Called for EVERY attribute assignment
        object.__setattr__(self, name, value)
```

**Component Breakdown:**
| Method | When Called |
|--------|-------------|
| `__getattr__` | Only when normal lookup raises `AttributeError` |
| `__getattribute__` | Every attribute access (use with caution) |
| `__setattr__` | Every attribute assignment |

**Syntax Rules:**
- `__getattr__` should raise `AttributeError` for unknown attributes.
- `__getattribute__` must use `object.__getattribute__` to avoid infinite recursion.
- `__setattr__` must use `object.__setattr__` or `self.__dict__[name] = value` to avoid recursion.

**Constraints and Limitations:**
- Overriding `__getattribute__` can break `self` access and built-in behavior.
- `__getattr__` is not called for attributes found via normal lookup.
- Infinite recursion is a common pitfall.

### Annotated Code Examples

```python
class Proxy:
    """Delegates attribute access to a wrapped object."""
    
    def __init__(self, target):
        # Use object.__setattr__ to avoid recursion
        object.__setattr__(self, '_target', target)
        object.__setattr__(self, '_access_log', [])
    
    def __getattr__(self, name):
        """Called only when normal lookup fails."""
        self._access_log.append(f"get: {name}")
        return getattr(self._target, name)
    
    def __setattr__(self, name, value):
        """Called for every assignment."""
        if name.startswith('_'):
            object.__setattr__(self, name, value)
        else:
            self._access_log.append(f"set: {name} = {value}")
            setattr(self._target, name, value)

# Target object
class User:
    def __init__(self):
        self.name = "Alice"
        self.age = 30
    
    def greet(self):
        return f"Hello, I'm {self.name}"

# Usage
user = User()
proxy = Proxy(user)

# Access proxied attributes
print(f"Name: {proxy.name}")      # __getattr__
print(f"Greet: {proxy.greet()}")  # __getattr__

# Set through proxy
proxy.age = 31                     # __setattr__
print(f"Age: {proxy.age}")

# View access log
print(f"Access log: {proxy._access_log}")
```

**Expected Output:**
```
Name: Alice
Greet: Hello, I'm Alice
Age: 31
Access log: ['get: name', 'get: greet', 'set: age = 31', 'get: age']
```

**Why This Output:** `proxy.name` triggers `__getattr__` because `name` is not in `proxy.__dict__`. It delegates to `user.name`. `proxy.age = 31` triggers `__setattr__`, which delegates to `user`. The log tracks all dynamic accesses.

### Real-World Cases

- **Proxy objects:** Remote procedure calls, ORMs.
- **Lazy loading:** Attributes loaded on first access.
- **Attribute validation:** Intercepting assignments for type checking.
- **Mock objects:** Testing frameworks use `__getattr__` to return mocks for any attribute.

### References

- Python Data Model: Customizing attribute access — https://docs.python.org/3/reference/datamodel.html#customizing-attribute-access
- Fluent Python: Attribute Access (Chapter 19)

---

## Master Reference List

- Python Data Model — https://docs.python.org/3/reference/datamodel.html
- collections.abc — Container ABCs — https://docs.python.org/3/library/collections.abc.html
- contextlib — Context Manager Utilities — https://docs.python.org/3/library/contextlib.html
- Python C API: Object Life Cycle — https://docs.python.org/3/c-api/lifecycle.html
- Fluent Python (Ramalho) — Chapter 1: The Python Data Model
- Cornell CS 1110: Special Methods — https://www.cs.cornell.edu/courses/cs1110/2018sp/lectures/lecture18/handout-18.pdf