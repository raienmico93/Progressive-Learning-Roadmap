# Python Dictionaries: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A dictionary is a built-in Python data type that stores an unordered collection of key-value pairs, where each unique key maps to a corresponding value.

**Technical Definition**
A dictionary (`dict`) is a mutable mapping type that implements a hash table, providing O(1) average-case time complexity for key lookups, insertions, and deletions. Keys must be hashable objects (numbers, strings, tuples of hashables), while values can be any Python object. Since Python 3.7, dictionaries are guaranteed to preserve insertion order as a language specification, and this behaviour was an implementation detail of CPython 3.6. Dictionaries support a rich set of methods for accessing, updating, and manipulating mappings, as well as operators for merging (PEP 584) and comprehensive views for iteration.

**Beginner-Friendly Explanation**
A dictionary is like a real-world dictionary or phone book: you look up a word (the key) to find its definition (the value). In Python, you create a dictionary with curly braces, like `{"name": "Alice", "age": 30}`. You can quickly find, add, or change values using their keys. Dictionaries are one of the most powerful and commonly used data structures in Python because they make data retrieval extremely fast and are perfect for storing structured information.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Key-Value Mapping** | Each key maps to exactly one value; keys must be unique |
| **Hashable Keys** | Keys must be hashable (strings, numbers, tuples); values can be anything |
| **Mutable** | Key-value pairs can be added, modified, or removed after creation |
| **Insertion-Order Preserved** | As of Python 3.7, dictionaries maintain the order in which keys were inserted |
| **O(1) Lookup** | Average constant-time access, insertion, and deletion via hashing |
| **No Indexing** | Access is by key, not by numerical index |
| **Iterable** | Dictionaries support iteration over keys, values, or items |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and statements
- **Understanding of data types**: integers, floats, strings, booleans, tuples
- **Familiarity with the Python interpreter**: running scripts or using the REPL
- **Basic concepts of hashability**: understanding which objects can be dictionary keys
- **Knowledge of lists and tuples**: since dictionaries share some operations and can contain them as values

### Related Programming Areas

- **Sets**: Both use hash tables; dictionary keys share hashability requirements with set elements
- **JSON**: Dictionaries map directly to JSON objects for data interchange
- **Configuration Management**: Dictionaries store application settings
- **Namespaces**: Python's `globals()` and `locals()` return dictionaries
- **Object Attributes**: `__dict__` holds an object's writable attributes as a dictionary
- **Caching**: Dictionaries implement memoization and fast lookups
- **Graph Algorithms**: Adjacency lists are often represented as dictionaries

### Core Concepts / Features

1. **Key-Value Structures and Hashability Requirements**
2. **Creating Dictionaries**
3. **Accessing Values** (`get()`, `setdefault()`)
4. **Updating Mappings** (Including `|` and `|=` operators)
5. **Removing Entries**
6. **Dictionary Methods**
7. **Nested Dictionaries**
8. **Dictionary Views** (`keys()`, `values()`, `items()`)
9. **Dictionary Comprehensions**
10. **Insertion-Order Preservation Guarantee**

---

## Core Concept 1: Key-Value Structures and Hashability Requirements

### Definitions

**Core Definition**
A dictionary's keys must be hashable objects, meaning they have a hash value that never changes during their lifetime, enabling efficient lookup.

**Technical Definition**
An object is hashable if it has a hash value which never changes during its lifetime (it needs a `__hash__()` method) and can be compared to other objects (it needs an `__eq__()` method). Hashable objects which compare equal must have the same hash value. Hashability makes an object usable as a dictionary key and a set member, because these data structures use the hash value internally. All of Python's immutable built-in objects are hashable; mutable containers (such as lists or dictionaries) are not. Objects which are instances of user-defined classes are hashable by default. They all compare unequal (except with themselves), and their hash value is derived from their `id()`.

**Beginner-Friendly Explanation**
For something to be a dictionary key, it must be "hashable"—which basically means it can't change. Strings, numbers, and tuples can be keys. Lists and other dictionaries cannot, because they can change after you create them. Think of it like a locker key: if the key could change shape, the locker would never open again. That's why mutable objects can't be keys—their "shape" (hash) could change.

### Purposes

- **To ensure fast, reliable key lookups** via hashing
- **To prevent keys from being modified** while they are in use
- **To guarantee that equal keys map to the same value**
- **To support immutable data structures** as identifiers (e.g., coordinates, composite keys)
- **To enable dictionary keys to be used in sets** (since both require hashability)

### Syntax Rules and Structure

**Hashability Rules**

| Type | Hashable? | Can Be Dict Key? |
|---|---|---|
| `int`, `float`, `bool` | Yes | Yes |
| `str` | Yes | Yes |
| `tuple` (of hashables) | Yes | Yes |
| `frozenset` | Yes | Yes |
| `list` | No | No |
| `dict` | No | No |
| `set` | No | No |
| Custom objects (default) | Yes (by id) | Yes |

**Constraints and Limitations**

- **Mutable objects cannot be keys**: Lists, dicts, and sets raise `TypeError: unhashable type`.
- **Tuples with mutable elements are unhashable**: `(1, [2, 3])` cannot be a key.
- **Hash collisions**: Different keys may have the same hash, but Python handles this via equality checks.
- **Custom `__hash__` must be consistent**: If two objects are equal, they must have the same hash.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Valid vs. Invalid Keys**

```python
# Step 1: Valid keys
valid = {
    42: "integer key",
    "name": "string key",
    (1, 2): "tuple key",
    frozenset({1, 2}): "frozenset key",
}
print(f"Valid dict: {valid}")

# Step 2: Invalid key (list)
try:
    bad = {[1, 2]: "value"}
except TypeError as e:
    print(f"Error: {e}")

# Step 3: Invalid key (tuple containing a list)
try:
    bad = {(1, [2, 3]): "value"}
except TypeError as e:
    print(f"Error: {e}")
```

**Expected Output:**
```
Valid dict: {42: 'integer key', 'name': 'string key', (1, 2): 'tuple key', frozenset({1, 2}): 'frozenset key'}
Error: unhashable type: 'list'
Error: unhashable type: 'list'
```

**Breakdown:** Integers, strings, tuples, and frozensets are hashable and can be keys. Lists and tuples containing lists are unhashable.

**Example 2: Using Tuples as Composite Keys**

```python
# Step 1: Create a dictionary with tuple keys
coordinates = {
    (40.7128, -74.0060): "New York",
    (51.5074, -0.1278): "London",
    (35.6762, 139.6503): "Tokyo",
}

# Step 2: Look up by tuple key
city = coordinates[(40.7128, -74.0060)]
print(f"City: {city}")

# Step 3: Iterate over keys
for coord, name in coordinates.items():
    print(f"{name}: {coord}")
```

**Expected Output:**
```
City: New York
New York: (40.7128, -74.006)
London: (51.5074, -0.1278)
Tokyo: (35.6762, 139.6503)
```

**Breakdown:** Tuples of hashable elements make excellent composite keys for multi-dimensional lookups.

**Example 3: Custom Objects as Keys**

```python
# Step 1: Define a class (hashable by default)
class UserId:
    def __init__(self, value):
        self.value = value
    def __repr__(self):
        return f"UserId({self.value})"

# Step 2: Use custom objects as keys
users = {
    UserId(1): "Alice",
    UserId(2): "Bob",
}
print(users)

# Step 3: Look up by object (must be the SAME object)
uid = UserId(1)
users[uid] = "Alice (new)"
print(f"Lookup: {users[uid]}")
```

**Expected Output:**
```
{UserId(1): 'Alice', UserId(2): 'Bob'}
Lookup: Alice (new)
```

**Breakdown:** Custom objects are hashable by default (based on `id()`). Two objects with the same `value` are different keys unless `__eq__` and `__hash__` are defined.

### Real-World Cases with Explanation

**Case 1: Caching with Composite Keys**

```python
cache = {}
def compute(a, b):
    key = (a, b)
    if key not in cache:
        cache[key] = expensive_operation(a, b)
    return cache[key]
```

**Why it matters:** Tuples as keys enable caching of results indexed by multiple parameters.

**Case 2: Graph Adjacency Lists**

```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C"],
}
```

**Why it matters:** Dictionaries with string keys naturally represent graph adjacency lists.

---

## Core Concept 2: Creating Dictionaries

### Definitions

**Core Definition**
Creating a dictionary means instantiating a new `dict` object, either by writing a dictionary literal or by calling the `dict()` constructor.

**Technical Definition**
Dictionaries can be created by placing a comma-separated list of `key: value` pairs within braces, `{key1: value1, key2: value2, ...}`, or by using the `dict()` constructor. The `dict()` constructor accepts an optional positional argument (an iterable of key-value pairs) and arbitrary keyword arguments. The `fromkeys()` class method creates a dictionary with specified keys and a common value. Dictionary comprehensions provide a concise way to build dictionaries from existing iterables.

**Beginner-Friendly Explanation**
You create a dictionary by putting key-value pairs inside curly braces. Keys and values are separated by a colon, and pairs are separated by commas. You can also use `dict()` to build one from a list of pairs, or use a comprehension to build one dynamically.

### Purposes

- **To store structured data** as key-value mappings
- **To group related information** under descriptive keys
- **To initialise configuration settings** or application state
- **To convert other data structures** into dictionaries for fast lookup
- **To build mappings dynamically** from computations or input

### Syntax Rules and Structure

**Complete General Syntax**

```
# Empty dictionary
{}
dict()

# Dictionary literal
{key1: value1, key2: value2, ...}

# From iterable of pairs
dict([(key1, value1), (key2, value2)])
dict(zip(keys, values))

# From keyword arguments
dict(key1=value1, key2=value2)

# From keys with common value
dict.fromkeys(keys, value)

# Dictionary comprehension
{key_expr: value_expr for item in iterable}
```

**Breakdown:**
- `{}`: Empty dictionary literal.
- `{key: value, ...}`: Dictionary literal with key-value pairs.
- `dict(iterable)`: Constructs from an iterable of `(key, value)` pairs.
- `dict(**kwargs)`: Constructs from keyword arguments.
- `dict.fromkeys(keys, value)`: Creates a dict with all keys set to the same value.
- `{k: v for ...}`: Dictionary comprehension.

**Syntax Rules**

| Rule | Description |
|---|---|
| Curly braces | Dictionary literals use `{}` |
| Colon separates | Key and value are separated by `:` |
| Comma separates | Key-value pairs are separated by commas |
| Keys must be hashable | Strings, numbers, tuples |
| Duplicate keys | Last value wins in a literal |

**Constraints and Limitations**

- **`{}` creates a dict, not a set**: This is the opposite of what some beginners expect.
- **Duplicate keys in literals**: Later values silently overwrite earlier ones.
- **`fromkeys()` shares the same value**: All keys reference the same object for mutable values.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Creating Dictionaries in Various Ways**

```python
# Step 1: Empty dictionary
empty = {}
print(f"Empty: {empty}")

# Step 2: Dictionary literal
person = {"name": "Alice", "age": 30, "city": "New York"}
print(f"Literal: {person}")

# Step 3: From list of tuples
pairs = [("a", 1), ("b", 2), ("c", 3)]
from_pairs = dict(pairs)
print(f"From pairs: {from_pairs}")

# Step 4: From zip
keys = ["x", "y", "z"]
values = [10, 20, 30]
from_zip = dict(zip(keys, values))
print(f"From zip: {from_zip}")

# Step 5: From keyword arguments
from_kwargs = dict(name="Bob", age=25)
print(f"From kwargs: {from_kwargs}")

# Step 6: From keys with common value
from_keys = dict.fromkeys(["a", "b", "c"], 0)
print(f"From keys: {from_keys}")
```

**Expected Output:**
```
Empty: {}
Literal: {'name': 'Alice', 'age': 30, 'city': 'New York'}
From pairs: {'a': 1, 'b': 2, 'c': 3}
From zip: {'x': 10, 'y': 20, 'z': 30}
From kwargs: {'name': 'Bob', 'age': 25}
From keys: {'a': 0, 'b': 0, 'c': 0}
```

**Breakdown:** Each method produces a dictionary from different input formats. `fromkeys()` is convenient for initialising all keys to the same value.

**Example 2: Duplicate Keys in Literals**

```python
# Step 1: Duplicate keys in a literal
data = {"a": 1, "b": 2, "a": 3}
print(f"Duplicates: {data}")

# Step 2: The last value wins
print(f"Value of 'a': {data['a']}")
```

**Expected Output:**
```
Duplicates: {'a': 3, 'b': 2}
Value of 'a': 3
```

**Breakdown:** When the same key appears multiple times in a literal, the last assignment overwrites the previous ones.

**Example 3: `fromkeys()` with Mutable Values (Pitfall)**

```python
# Step 1: fromkeys with a mutable value
shared = dict.fromkeys(["a", "b", "c"], [])
print(f"Initial: {shared}")

# Step 2: Modify one value
shared["a"].append(1)
print(f"After modification: {shared}")
```

**Expected Output:**
```
Initial: {'a': [], 'b': [], 'c': []}
After modification: {'a': [1], 'b': [1], 'c': [1]}
```

**Breakdown:** `fromkeys()` assigns the *same* mutable object to all keys. Modifying it through one key affects all keys. Use a comprehension for independent mutable values.

### Real-World Cases with Explanation

**Case 1: Configuration Dictionaries**

```python
config = {
    "host": "localhost",
    "port": 8080,
    "debug": False,
}
```

**Why it matters:** Dictionaries are the natural Python representation for configuration settings, easily serialised to/from JSON.

**Case 2: Building a Lookup Table**

```python
status_codes = dict(zip(range(200, 206), ["OK", "Created", "Accepted", "Non-Authoritative", "No Content", "Reset Content"]))
```

**Why it matters:** `zip()` combined with `dict()` quickly builds lookup tables from parallel sequences.

---

## Core Concept 3: Accessing Values

### Definitions

**Core Definition**
Accessing values retrieves the value associated with a key using either direct indexing (`dict[key]`) or safer methods (`get()`, `setdefault()`).

**Technical Definition**
`dict[key]` returns the value for `key`, raising `KeyError` if the key is absent. `dict.get(key, default=None)` returns the value for `key` if present, otherwise returns `default`. `dict.setdefault(key, default=None)` returns the value for `key` if present; if not, it inserts `key` with `default` and returns `default`. These methods provide graceful handling of missing keys without exceptions.

**Beginner-Friendly Explanation**
You can look up a value by its key using square brackets, but if the key doesn't exist, Python raises an error. The `get()` method is safer—it returns `None` (or a default you specify) instead of crashing. `setdefault()` is like `get()` but also adds the key if it's missing.

### Purposes

- **To retrieve values** associated with known keys
- **To handle missing keys gracefully** with defaults (`get()`)
- **To initialise dictionary entries** lazily (`setdefault()`)
- **To avoid `KeyError` exceptions** in uncertain lookups
- **To build nested structures** without pre-checking keys

### Syntax Rules and Structure

**Complete General Syntax**

```
# Direct access (raises KeyError if missing)
dict[key]

# Safe access with default
dict.get(key, default=None)

# Get or insert with default
dict.setdefault(key, default=None)
```

**Breakdown:**
- `dict[key]`: Direct indexing; raises `KeyError` if key is absent.
- `dict.get(key, default)`: Returns value or `default` (defaults to `None`).
- `dict.setdefault(key, default)`: Returns value if present; otherwise inserts `key: default` and returns `default`.

**Access Rules**

| Method | Returns | On Missing Key |
|---|---|---|
| `dict[key]` | Value | Raises `KeyError` |
| `dict.get(key)` | Value or `None` | Returns `None` |
| `dict.get(key, default)` | Value or `default` | Returns `default` |
| `dict.setdefault(key)` | Value or `None` | Inserts `key: None` |
| `dict.setdefault(key, default)` | Value or `default` | Inserts `key: default` |

**Constraints and Limitations**

- **`get()` does not modify the dictionary**: It returns a default but doesn't insert the key.
- **`setdefault()` modifies the dictionary**: It inserts the key if absent.
- **`setdefault()` evaluates the default every time**: Even if the key exists, the default expression is evaluated.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Direct Access vs. `get()`**

```python
# Step 1: Create a dictionary
person = {"name": "Alice", "age": 30}

# Step 2: Direct access (works for existing keys)
print(person["name"])

# Step 3: Direct access (raises KeyError for missing keys)
try:
    print(person["email"])
except KeyError as e:
    print(f"Error: {e}")

# Step 4: Safe access with get()
print(person.get("name"))           # Alice
print(person.get("email"))          # None
print(person.get("email", "N/A"))   # N/A
```

**Expected Output:**
```
Alice
Error: 'email'
Alice
None
N/A
```

**Breakdown:** Direct indexing raises `KeyError` for missing keys. `get()` returns `None` or a specified default instead.

**Example 2: `setdefault()` for Lazy Initialisation**

```python
# Step 1: Create an empty dictionary
word_counts = {}

# Step 2: Count words using setdefault
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
for word in words:
    word_counts.setdefault(word, 0)
    word_counts[word] += 1

print(word_counts)
```

**Expected Output:**
```
{'apple': 3, 'banana': 2, 'cherry': 1}
```

**Breakdown:** `setdefault(word, 0)` initialises the count to 0 if the word is new, then the increment works safely.

**Example 3: `setdefault()` for Nested Structures**

```python
# Step 1: Build a nested dictionary without pre-checking keys
data = {}
data.setdefault("fruits", []).append("apple")
data.setdefault("fruits", []).append("banana")
data.setdefault("vegetables", []).append("carrot")

print(data)
```

**Expected Output:**
```
{'fruits': ['apple', 'banana'], 'vegetables': ['carrot']}
```

**Breakdown:** `setdefault("fruits", [])` returns the existing list (or creates and returns a new one), allowing immediate `.append()` calls.

### Real-World Cases with Explanation

**Case 1: Configuration with Defaults**

```python
config = {"host": "localhost"}
port = config.get("port", 8080)  # Default port if not specified
```

**Why it matters:** `get()` provides a clean way to supply default values for missing configuration keys.

**Case 2: Grouping Data**

```python
groups = {}
for item in items:
    groups.setdefault(item.category, []).append(item)
```

**Why it matters:** `setdefault()` is the idiomatic way to group items by a category without pre-initialising each group.

---

## Core Concept 4: Updating Mappings

### Definitions

**Core Definition**
Updating a dictionary means adding new key-value pairs, modifying existing values, or merging another dictionary into it.

**Technical Definition**
The `update()` method merges key-value pairs from another dictionary or iterable into the current dictionary, overwriting existing keys. The `|` operator (introduced in Python 3.9 via PEP 584) returns a new merged dictionary, while `|=` updates in place. Both resolve key conflicts by keeping the rightmost value. The `update()` method modifies in place and returns `None`.

**Beginner-Friendly Explanation**
You can update a dictionary in several ways: assign to a key (`d["key"] = value`), use `update()` to merge another dictionary, or use the `|` operator (Python 3.9+) to combine dictionaries into a new one. The `|=` operator updates in place like `update()` but with cleaner syntax.

### Purposes

- **To add new key-value pairs** to an existing dictionary
- **To modify existing values** by key
- **To merge dictionaries** from multiple sources
- **To apply defaults** from a base configuration
- **To combine data** from multiple sources into one mapping

### Syntax Rules and Structure

**Complete General Syntax**

```
# Assign or modify
dict[key] = value

# In-place update
dict.update(other)
dict.update(iterable)
dict.update(**kwargs)

# Merge operators (Python 3.9+)
merged = dict1 | dict2
dict1 |= dict2
```

**Breakdown:**
- `dict[key] = value`: Adds or modifies a single entry.
- `dict.update(other)`: Merges `other` into `dict` in place.
- `dict1 | dict2`: Returns a new dictionary merging `dict1` and `dict2`.
- `dict1 |= dict2`: Updates `dict1` in place with `dict2`.

**Update Rules**

| Operation | Modifies Original | Returns |
|---|---|---|
| `dict[key] = value` | Yes | — |
| `dict.update(other)` | Yes | `None` |
| `dict1 \| dict2` | No | New dict |
| `dict1 \|= dict2` | Yes | — |

**Constraints and Limitations**

- **`update()` returns `None`**: Do not assign the result.
- **`|` requires both operands to be dicts**: Cannot mix with lists or other types.
- **`|=` accepts any mapping or iterable of pairs**: More flexible than `|`.
- **Rightmost value wins**: When keys conflict, the later value overwrites the earlier.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Update Operations**

```python
# Step 1: Create a dictionary
person = {"name": "Alice", "age": 30}

# Step 2: Add a new key
person["city"] = "New York"
print(f"After add: {person}")

# Step 3: Modify existing key
person["age"] = 31
print(f"After modify: {person}")

# Step 4: Update with another dict
person.update({"email": "alice@example.com", "age": 32})
print(f"After update: {person}")
```

**Expected Output:**
```
After add: {'name': 'Alice', 'age': 30, 'city': 'New York'}
After modify: {'name': 'Alice', 'age': 31, 'city': 'New York'}
After update: {'name': 'Alice', 'age': 32, 'city': 'New York', 'email': 'alice@example.com'}
```

**Breakdown:** Direct assignment adds or modifies single keys. `update()` merges multiple pairs, overwriting existing keys.

**Example 2: Merge Operators (`|` and `|=`)**

```python
# Step 1: Define two dictionaries
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}

# Step 2: Merge with | (returns new dict)
merged = d1 | d2
print(f"Merged: {merged}")
print(f"d1 unchanged: {d1}")

# Step 3: Update with |= (in place)
d1 |= d2
print(f"d1 after |=: {d1}")

# Step 4: Key conflict — rightmost wins
conflict = {"a": 1} | {"a": 99}
print(f"Conflict: {conflict}")
```

**Expected Output:**
```
Merged: {'a': 1, 'b': 3, 'c': 4}
d1 unchanged: {'a': 1, 'b': 2}
d1 after |=: {'a': 1, 'b': 3, 'c': 4}
Conflict: {'a': 99}
```

**Breakdown:** `|` returns a new dictionary without modifying either operand. `|=` updates in place. In conflicts, the right operand's value wins.

**Example 3: Updating from Various Sources**

```python
# Step 1: From a list of pairs
d = {"a": 1}
d.update([("b", 2), ("c", 3)])
print(f"From pairs: {d}")

# Step 2: From keyword arguments
d.update(d=4, e=5)
print(f"From kwargs: {d}")

# Step 3: From zip
d.update(zip(["f", "g"], [6, 7]))
print(f"From zip: {d}")
```

**Expected Output:**
```
From pairs: {'a': 1, 'b': 2, 'c': 3}
From kwargs: {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5}
From zip: {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5, 'f': 6, 'g': 7}
```

**Breakdown:** `update()` accepts iterables of pairs, keyword arguments, and other mappings, making it highly flexible.

### Real-World Cases with Explanation

**Case 1: Merging Configuration Files**

```python
defaults = {"host": "localhost", "port": 8080, "debug": False}
user_config = {"port": 9090, "debug": True}
final = defaults | user_config
# {'host': 'localhost', 'port': 9090, 'debug': True}
```

**Why it matters:** The `|` operator cleanly merges defaults with user overrides, keeping the user's values.

**Case 2: Accumulating Data**

```python
totals = {}
for transaction in transactions:
    totals[transaction.category] = totals.get(transaction.category, 0) + transaction.amount
```

**Why it matters:** `update()` and `get()` patterns are common for accumulating values in dictionaries.

---

## Core Concept 5: Removing Entries

### Definitions

**Core Definition**
Removing entries deletes key-value pairs from a dictionary using `del`, `pop()`, `popitem()`, or `clear()`.

**Technical Definition**
`del d[key]` removes the entry with the specified key, raising `KeyError` if absent. `d.pop(key, default)` removes and returns the value for `key`; if absent, returns `default` or raises `KeyError`. `d.popitem()` removes and returns the last inserted key-value pair as a tuple. `d.clear()` removes all entries.

**Beginner-Friendly Explanation**
You can remove items from a dictionary in several ways. `del` removes by key. `pop()` removes by key and gives you the value back. `popitem()` removes the most recently added item. `clear()` empties the whole dictionary.

### Purposes

- **To delete specific key-value pairs** by key
- **To remove and retrieve** values for further processing (`pop()`)
- **To remove items in LIFO order** (`popitem()`)
- **To empty a dictionary** completely (`clear()`)
- **To filter dictionaries** by removing unwanted entries

### Syntax Rules and Structure

**Complete General Syntax**

```
del dict[key]            # remove by key
dict.pop(key[, default]) # remove and return value
dict.popitem()           # remove and return last item
dict.clear()             # remove all items
```

**Breakdown:**
- `del dict[key]`: Removes the entry; raises `KeyError` if missing.
- `dict.pop(key, default)`: Removes and returns value; returns `default` if missing.
- `dict.popitem()`: Removes and returns the last inserted pair.
- `dict.clear()`: Removes all items.

**Removal Rules**

| Method | Removes | Returns | On Missing Key |
|---|---|---|---|
| `del d[key]` | By key | — | `KeyError` |
| `d.pop(key)` | By key | Value | `KeyError` |
| `d.pop(key, default)` | By key | Value or default | Returns default |
| `d.popitem()` | Last item | `(key, value)` | `KeyError` if empty |
| `d.clear()` | All items | `None` | — |

**Constraints and Limitations**

- **`popitem()` is LIFO**: Removes the most recently inserted item (since Python 3.7).
- **`clear()` returns `None`**: Do not assign the result.
- **Modifying during iteration**: Removing items while iterating over a dictionary raises `RuntimeError`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: `del` and `pop()`**

```python
# Step 1: Create a dictionary
data = {"a": 1, "b": 2, "c": 3}

# Step 2: Delete by key
del data["a"]
print(f"After del: {data}")

# Step 3: Pop by key (returns value)
value = data.pop("b")
print(f"Popped: {value}")
print(f"After pop: {data}")

# Step 4: Pop with default (no KeyError)
result = data.pop("z", "not found")
print(f"Pop default: {result}")
```

**Expected Output:**
```
After del: {'b': 2, 'c': 3}
Popped: 2
After pop: {'c': 3}
Pop default: not found
```

**Breakdown:** `del` removes without returning. `pop()` removes and returns the value. The default argument prevents `KeyError`.

**Example 2: `popitem()` and `clear()`**

```python
# Step 1: Create a dictionary
data = {"a": 1, "b": 2, "c": 3}

# Step 2: Pop items (LIFO)
last = data.popitem()
print(f"Popped last: {last}")
print(f"Remaining: {data}")

# Step 3: Clear all
data.clear()
print(f"After clear: {data}")
```

**Expected Output:**
```
Popped last: ('c', 3)
Remaining: {'a': 1, 'b': 2}
After clear: {}
```

**Breakdown:** `popitem()` removes the most recently inserted item. `clear()` empties the dictionary.

**Example 3: Safe Removal with `get()` and `pop()`**

```python
# Step 1: Remove all keys with value 0
data = {"a": 1, "b": 0, "c": 3, "d": 0}
keys_to_remove = [k for k, v in data.items() if v == 0]
for key in keys_to_remove:
    data.pop(key)
print(data)
```

**Expected Output:**
```
{'a': 1, 'c': 3}
```

**Breakdown:** Collecting keys to remove first avoids modifying the dictionary during iteration.

### Real-World Cases with Explanation

**Case 1: Cleaning Configuration**

```python
config = {"host": "localhost", "port": 8080, "temp": None}
config.pop("temp", None)  # Remove temporary key safely
```

**Why it matters:** `pop()` with a default safely removes optional keys without raising errors.

**Case 2: Processing a Queue**

```python
tasks = {"task1": "pending", "task2": "pending"}
while tasks:
    key, value = tasks.popitem()
    process(key, value)
```

**Why it matters:** `popitem()` removes and processes items one by one until the dictionary is empty.

---

## Core Concept 6: Dictionary Methods

### Complete Dictionary Method Reference

| Method | Syntax | Description | Returns |
|---|---|---|---|
| `get` | `d.get(key[, default])` | Value for key, or default | Value or default |
| `setdefault` | `d.setdefault(key[, default])` | Value or insert default | Value or default |
| `pop` | `d.pop(key[, default])` | Remove and return value | Value |
| `popitem` | `d.popitem()` | Remove and return last item | `(key, value)` |
| `update` | `d.update(other)` | Merge other into d | `None` |
| `clear` | `d.clear()` | Remove all items | `None` |
| `copy` | `d.copy()` | Shallow copy | New dict |
| `fromkeys` | `dict.fromkeys(keys[, value])` | New dict with keys | New dict |
| `keys` | `d.keys()` | View of keys | `dict_keys` view |
| `values` | `d.values()` | View of values | `dict_values` view |
| `items` | `d.items()` | View of key-value pairs | `dict_items` view |

**Method Details**

**`get(key, default=None)`**: Return the value for `key` if `key` is in the dictionary, else `default`.

**`setdefault(key, default=None)`**: If `key` is in the dictionary, return its value. If not, insert `key` with a value of `default` and return `default`.

**`pop(key[, default])`**: If `key` is in the dictionary, remove it and return its value. If not, and `default` is given, return `default`. Otherwise, raise `KeyError`.

**`popitem()`**: Remove and return a `(key, value)` pair from the dictionary. Pairs are returned in LIFO order.

**`update([other])`**: Update the dictionary with the key/value pairs from `other`, overwriting existing keys.

**`clear()`**: Remove all items from the dictionary.

**`copy()`**: Return a shallow copy of the dictionary.

**`fromkeys(iterable, value=None)`**: Class method that creates a new dictionary with keys from `iterable` and values set to `value`.

**`keys()`**: Return a new view of the dictionary's keys.

**`values()`**: Return a new view of the dictionary's values.

**`items()`**: Return a new view of the dictionary's items (`(key, value)` pairs).

---

## Core Concept 7: Nested Dictionaries

### Definitions

**Core Definition**
A nested dictionary is a dictionary that contains other dictionaries as its values, enabling the representation of hierarchical or multi-level data structures.

**Technical Definition**
Dictionaries can contain other dictionaries as values, creating nested structures of arbitrary depth. Access requires multiple key lookups: `nested[key1][key2]` retrieves the value for `key2` within the inner dictionary stored at `key1`. Nested dictionaries are commonly used to model JSON-like data, configuration hierarchies, and relational data.

**Beginner-Friendly Explanation**
A nested dictionary is a dictionary inside another dictionary. Think of it like a filing cabinet: the outer dictionary is the cabinet, each drawer is an inner dictionary, and each folder inside the drawer is a key-value pair.

### Purposes

- **To represent hierarchical data** (e.g., JSON, XML)
- **To store multi-dimensional records** (e.g., user profiles with nested addresses)
- **To model relationships** between entities
- **To organise configuration** into sections
- **To implement tree-like structures**

### Syntax Rules and Structure

**Complete General Syntax**

```
nested = {
    "outer_key1": {
        "inner_key1": "value1",
        "inner_key2": "value2",
    },
    "outer_key2": {
        "inner_key3": "value3",
    },
}

# Access
nested["outer_key1"]["inner_key1"]
```

**Breakdown:**
- Outer dictionary contains keys whose values are inner dictionaries.
- Access requires chained indexing.
- Nesting can be arbitrarily deep.

**Nested Dictionary Rules**

| Rule | Description |
|---|---|
| Multi-level access | `d[k1][k2]` for 2 levels, `d[k1][k2][k3]` for 3 levels |
| Inner dicts are independent | Each inner dict is a separate object |
| Values can be any type | Inner values can be dicts, lists, or scalars |
| `setdefault()` for creation | Safely initialise nested levels |

**Constraints and Limitations**

- **KeyError on missing outer key**: Accessing a non-existent outer key raises `KeyError`.
- **Verbose access**: Deep nesting requires long chains of indexing.
- **`setdefault()` or `defaultdict` for safe creation**: Avoid manual checking for each level.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Creating and Accessing Nested Dictionaries**

```python
# Step 1: Create a nested dictionary
users = {
    "alice": {
        "age": 30,
        "email": "alice@example.com",
        "address": {
            "city": "New York",
            "zip": "10001",
        },
    },
    "bob": {
        "age": 25,
        "email": "bob@example.com",
    },
}

# Step 2: Access nested values
print(users["alice"]["email"])
print(users["alice"]["address"]["city"])

# Step 3: Access missing key
print(users["bob"].get("address", {}).get("city", "Unknown"))
```

**Expected Output:**
```
alice@example.com
New York
Unknown
```

**Breakdown:** Nested access chains key lookups. The `get()` chain safely handles missing keys.

**Example 2: Building Nested Dictionaries with `setdefault()`**

```python
# Step 1: Build a nested structure dynamically
data = {}
data.setdefault("fruits", {})["apple"] = 5
data.setdefault("fruits", {})["banana"] = 3
data.setdefault("vegetables", {})["carrot"] = 10

print(data)
```

**Expected Output:**
```
{'fruits': {'apple': 5, 'banana': 3}, 'vegetables': {'carrot': 10}}
```

**Breakdown:** `setdefault()` creates the inner dictionary if it doesn't exist, then the assignment adds the key-value pair.

**Example 3: Using `defaultdict` for Deep Nesting**

```python
from collections import defaultdict

# Step 1: Create a nested defaultdict
nested = defaultdict(lambda: defaultdict(dict))

# Step 2: Assign values without pre-creating levels
nested["level1"]["level2"]["level3"] = "deep value"
print(nested["level1"]["level2"]["level3"])
```

**Expected Output:**
```
deep value
```

**Breakdown:** `defaultdict` automatically creates missing levels, making deep assignment straightforward.

### Real-World Cases with Explanation

**Case 1: JSON Data Representation**

```python
response = {
    "status": "success",
    "data": {
        "users": [
            {"name": "Alice", "age": 30},
            {"name": "Bob", "age": 25},
        ]
    }
}
```

**Why it matters:** JSON objects map directly to Python dictionaries, and nested dictionaries model the hierarchical structure of API responses.

**Case 2: Application Configuration**

```python
config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "credentials": {
            "username": "admin",
            "password": "secret",
        }
    },
    "logging": {
        "level": "INFO",
        "file": "app.log",
    }
}
```

**Why it matters:** Nested dictionaries organise configuration settings into logical sections for clarity.

---

## Core Concept 8: Dictionary Views

### Definitions

**Core Definition**
Dictionary views are dynamic, read-only objects returned by the `keys()`, `values()`, and `items()` methods that provide a live view of the dictionary's contents.

**Technical Definition**
The `keys()`, `values()`, and `items()` methods return view objects (`dict_keys`, `dict_values`, `dict_items`) that provide a dynamic view of the dictionary's entries. Views are iterable, support membership tests, and reflect changes to the underlying dictionary. Keys and items views are set-like (supporting `&`, `|`, `-`, `^`), while values views are not, since values are not required to be unique or hashable.

**Beginner-Friendly Explanation**
Instead of giving you a static list of keys, values, or pairs, Python gives you a "window" into the dictionary. If you add or remove items from the dictionary, the view automatically reflects those changes. Views are memory-efficient and support fast membership testing.

### Purposes

- **To iterate over keys, values, or items** without copying
- **To check membership** in keys or items quickly
- **To perform set operations** on keys (intersection, union, etc.)
- **To reflect dictionary changes** dynamically
- **To save memory** compared to creating lists

### Syntax Rules and Structure

**Complete General Syntax**

```
dict.keys()      # view of all keys
dict.values()    # view of all values
dict.items()     # view of all (key, value) pairs
```

**Breakdown:**
- `keys()`: Returns a `dict_keys` view; set-like.
- `values()`: Returns a `dict_values` view; not set-like.
- `items()`: Returns a `dict_items` view of `(key, value)` tuples; set-like if all values are hashable.

**View Rules**

| Feature | `keys()` | `values()` | `items()` |
|---|---|---|---|
| Iterable | Yes | Yes | Yes |
| Membership test | Yes | Yes | Yes |
| Set-like operations | Yes | No | Yes (if hashable) |
| Dynamic | Yes | Yes | Yes |
| Reversible (Python 3.8+) | Yes | Yes | Yes |

**Constraints and Limitations**

- **Views are not lists**: They don't support indexing or slicing.
- **Values views are not set-like**: Values may not be unique or hashable.
- **Views reflect changes**: Modifying the dict while iterating a view can raise `RuntimeError`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic View Usage**

```python
# Step 1: Create a dictionary
dishes = {"eggs": 2, "sausage": 1, "bacon": 1, "spam": 500}

# Step 2: Get views
keys = dishes.keys()
values = dishes.values()
items = dishes.items()

print(f"Keys: {list(keys)}")
print(f"Values: {list(values)}")
print(f"Items: {list(items)}")

# Step 3: Views are dynamic
dishes["new"] = 99
print(f"Updated keys: {list(keys)}")
```

**Expected Output:**
```
Keys: ['eggs', 'sausage', 'bacon', 'spam']
Values: [2, 1, 1, 500]
Items: [('eggs', 2), ('sausage', 1), ('bacon', 1), ('spam', 500)]
Updated keys: ['eggs', 'sausage', 'bacon', 'spam', 'new']
```

**Breakdown:** Views reflect changes to the dictionary dynamically. The `keys` view automatically includes the newly added key.

**Example 2: Set Operations on Views**

```python
# Step 1: Create dictionaries
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 20, "c": 30, "d": 40}

# Step 2: Set operations on keys
print(f"Intersection: {d1.keys() & d2.keys()}")
print(f"Union: {d1.keys() | d2.keys()}")
print(f"Difference: {d1.keys() - d2.keys()}")
print(f"Symmetric difference: {d1.keys() ^ d2.keys()}")
```

**Expected Output:**
```
Intersection: {'b', 'c'}
Union: {'a', 'b', 'c', 'd'}
Difference: {'a'}
Symmetric difference: {'a', 'd'}
```

**Breakdown:** Keys views support set operations, enabling powerful comparisons between dictionaries.

**Example 3: Membership Testing on Views**

```python
# Step 1: Create a dictionary
data = {"name": "Alice", "age": 30}

# Step 2: Membership tests
print("name" in data.keys())          # True
print(30 in data.values())            # True
print(("name", "Alice") in data.items())  # True
```

**Expected Output:**
```
True
True
True
```

**Breakdown:** Views support efficient membership testing for keys, values, and items.

### Real-World Cases with Explanation

**Case 1: Comparing Dictionary Keys**

```python
required = {"name", "email", "age"}
provided = user_data.keys()
missing = required - provided
```

**Why it matters:** Set operations on keys views quickly identify missing required fields.

**Case 2: Iterating Efficiently**

```python
for key, value in config.items():
    print(f"{key} = {value}")
```

**Why it matters:** `items()` provides a memory-efficient way to iterate over key-value pairs without creating intermediate lists.

---

## Core Concept 9: Dictionary Comprehensions

### Definitions

**Core Definition**
A dictionary comprehension is a concise syntactic construct for creating a dictionary by applying an expression to each item in an iterable, optionally filtering with a condition.

**Technical Definition**
Dict comprehensions use curly braces with the syntax `{key_expr: value_expr for item in iterable if condition}`. The left part before the `for` keyword expresses both a key and a value, separated by a colon. The resulting dictionary is built by evaluating the key and value expressions for each item that satisfies the condition.

**Beginner-Friendly Explanation**
A dictionary comprehension is a shortcut for building a dictionary from another sequence. It looks like a list comprehension but uses curly braces and has both a key and a value expression. It's a clean, one-line way to create or transform dictionaries.

### Purposes

- **To create dictionaries concisely** from existing iterables
- **To transform existing dictionaries** (e.g., invert keys and values)
- **To filter dictionary entries** while building
- **To replace explicit loops** with a more readable expression
- **To build lookup tables** from parallel sequences

### Syntax Rules and Structure

**Complete General Syntax**

```
{key_expr: value_expr for item in iterable}
{key_expr: value_expr for item in iterable if condition}
```

**Breakdown:**
- `key_expr`: Expression for the key (can use `item`).
- `value_expr`: Expression for the value (can use `item`).
- `for item in iterable`: The iteration clause.
- `if condition`: Optional filter.

**Comprehension Rules**

| Rule | Description |
|---|---|
| Curly braces | Dict comprehensions use `{}` |
| Colon separates | Key and value expressions separated by `:` |
| Unique keys | Later keys overwrite earlier ones |
| Scope | Loop variables are local |
| Any iterable | Can iterate over lists, tuples, strings, ranges, or `items()` |

**Constraints and Limitations**

- **Memory**: Creates the entire dictionary in memory at once.
- **Readability**: Overly complex comprehensions should be replaced with explicit loops.
- **No `break` or `continue`**: Comprehensions do not support loop control statements.
- **Only expressions**: No statements allowed.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Dictionary Comprehension**

```python
# Step 1: Squares using a loop
squares_loop = {}
for x in range(5):
    squares_loop[x] = x**2
print(f"Loop: {squares_loop}")

# Step 2: Squares using a dict comprehension
squares_comp = {x: x**2 for x in range(5)}
print(f"Comp: {squares_comp}")
```

**Expected Output:**
```
Loop: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
Comp: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

**Breakdown:** The comprehension produces the same result as the loop in a single expression.

**Example 2: Inverting a Dictionary**

```python
# Step 1: Original dictionary
original = {"a": 1, "b": 2, "c": 3}

# Step 2: Invert keys and values
inverted = {v: k for k, v in original.items()}
print(f"Inverted: {inverted}")
```

**Expected Output:**
```
Inverted: {1: 'a', 2: 'b', 3: 'c'}
```

**Breakdown:** The comprehension iterates over `items()` and swaps keys and values. This works only if values are unique and hashable.

**Example 3: Filtering with a Comprehension**

```python
# Step 1: Filter even values
data = {"a": 1, "b": 2, "c": 3, "d": 4}
evens = {k: v for k, v in data.items() if v % 2 == 0}
print(f"Evens: {evens}")

# Step 2: Transform keys and values
transformed = {k.upper(): v * 10 for k, v in data.items()}
print(f"Transformed: {transformed}")
```

**Expected Output:**
```
Evens: {'b': 2, 'd': 4}
Transformed: {'A': 10, 'B': 20, 'C': 30, 'D': 40}
```

**Breakdown:** The `if` clause filters items. The key and value expressions can transform data independently.

### Real-World Cases with Explanation

**Case 1: Building a Lookup Table**

```python
users = [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]
user_by_id = {u["id"]: u["name"] for u in users}
```

**Why it matters:** Dict comprehensions quickly build lookup tables from lists of records.

**Case 2: Normalising Data**

```python
raw = {"Name": "Alice", "Age": "30", "City": "NYC"}
normalized = {k.lower(): v for k, v in raw.items()}
```

**Why it matters:** Comprehensions cleanly normalise keys and values in a single expression.

---

## Core Concept 10: Insertion-Order Preservation Guarantee

### Definitions

**Core Definition**
Since Python 3.7, dictionaries are guaranteed to preserve the order in which keys were inserted, making them ordered collections for all practical purposes.

**Technical Definition**
As of Python 3.7, the insertion-order preservation nature of `dict` objects has been declared an official part of the Python language spec. This behaviour was an implementation detail of CPython 3.6, but it became a language guarantee in 3.7. Updating a key does not affect its position in the order; however, keys added after deletion are inserted at the end. Order comparisons (`<`, `<=`, `>=`, `>`) raise `TypeError`.

**Beginner-Friendly Explanation**
Dictionaries remember the order in which you add items. If you add "a", then "b", then "c", iterating over the dictionary gives you "a", "b", "c" in that order. Before Python 3.7, you couldn't rely on this—the order could be anything. Now you can. However, dictionaries are still not sequences; they don't support indexing or slicing like lists.

### Purposes

- **To rely on consistent iteration order** across Python versions
- **To simplify code** that previously required `OrderedDict`
- **To preserve user-defined ordering** of configuration or data
- **To make dictionary output predictable** for debugging and logging
- **To support ordered serialisation** (JSON, YAML)

### Syntax Rules and Structure

**Order Preservation Rules**

| Operation | Effect on Order |
|---|---|
| Insert new key | Added at the end |
| Update existing key | Position unchanged |
| Delete key and re-insert | Added at the end |
| Iteration | Follows insertion order |
| `popitem()` | Removes last inserted item |

**Constraints and Limitations**

- **Not a sequence**: Dictionaries do not support indexing (`d[0]`) or slicing.
- **Order comparisons not supported**: `<`, `<=`, `>=`, `>` raise `TypeError`.
- **Deletion and re-insertion moves to end**: The key is treated as new.
- **`OrderedDict` still useful**: For order-sensitive equality and additional methods.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Order Preservation in Action**

```python
# Step 1: Create a dictionary
d = {"one": 1, "two": 2, "three": 3, "four": 4}
print(f"Initial             : {list(d)}")

# Step 2: Update an existing key (order unchanged)
d["one"] = 42
print(f"After update        : {list(d)}")

# Step 3: Delete and re-insert a key (moves to end)
del d["two"]
d["two"] = None
print(f"After delete+insert : {list(d)}")
```

**Expected Output:**
```
Initial             : ['one', 'two', 'three', 'four']
After update        : ['one', 'two', 'three', 'four']
After delete+insert : ['one', 'three', 'four', 'two']
```

**Breakdown:** Updating a key does not change its position. Deleting and re-inserting moves the key to the end. This behaviour is documented in the Python language specification.

**Example 2: Iteration Order**

```python
# Step 1: Create a dictionary
config = {"host": "localhost", "port": 8080, "debug": False}

# Step 2: Iterate over keys
for key in config:
    print(key)

# Step 3: Iterate over items
for key, value in config.items():
    print(f"{key} = {value}")
```

**Expected Output:**
```
host
port
debug
host = localhost
port = 8080
debug = False
```

**Breakdown:** Iteration follows insertion order. This makes output predictable and reproducible.

**Example 3: Comparison with `OrderedDict`**

```python
from collections import OrderedDict

# Step 1: Regular dict (ordered since 3.7)
regular = {"a": 1, "b": 2}

# Step 2: OrderedDict (still available)
ordered = OrderedDict([("a", 1), ("b", 2)])

# Step 3: Both preserve order
print(f"Regular: {list(regular)}")
print(f"Ordered: {list(ordered)}")

# Step 4: Equality ignores order for both
print(f"Equal: {regular == ordered}")
```

**Expected Output:**
```
Regular: ['a', 'b']
Ordered: ['a', 'b']
Equal: True
```

**Breakdown:** Regular dictionaries now provide order preservation, reducing the need for `OrderedDict` in most cases. However, `OrderedDict` still supports order-sensitive equality and additional methods.

### Real-World Cases with Explanation

**Case 1: Configuration Files**

```python
config = {}
config["host"] = "localhost"
config["port"] = 8080
config["debug"] = False
# Iteration order matches insertion order
```

**Why it matters:** Configuration files often need to preserve the order of settings for readability and reproducibility.

**Case 2: JSON Serialisation**

```python
import json
data = {"name": "Alice", "age": 30, "city": "NYC"}
json_str = json.dumps(data)
# Order in JSON matches insertion order
```

**Why it matters:** JSON serialisation preserves dictionary order, producing consistent output.

---

## References

- Python Software Foundation. *Built-in Types — Mapping Types — dict*. https://docs.python.org/3/library/stdtypes.html#mapping-types-dict
- Python Software Foundation. *5. Data Structures — Dictionaries*. https://docs.python.org/3/tutorial/datastructures.html#dictionaries
- Python Software Foundation. *PEP 584 – Add Union Operators To dict*. https://peps.python.org/pep-0584/
- Python Software Foundation. *PEP 274 – Dict Comprehensions*. https://peps.python.org/pep-0274/
- Python Software Foundation. *Python 3.7 Release Notes — Insertion-order preservation*. https://docs.python.org/3/whatsnew/3.7.html
- Real Python. *Dictionaries in Python*. https://realpython.com/python-dicts/
- Real Python. *Python Dictionary Comprehension*. https://realpython.com/python-dictionary-comprehension/
- Real Python. *Python Dictionary Views*. https://realpython.com/python-dictionary-views/
- Python Software Foundation. *collections — Container datatypes — OrderedDict*. https://docs.python.org/3/library/collections.html#collections.OrderedDict
- Python Software Foundation. *Dictionary view objects — CPython source documentation*. https://chromium.googlesource.com/external/github.com/python/cpython/
- GeeksforGeeks. *Define a 3 Level Nested Dictionary in Python*. https://origin.geeksforgeeks.org/python/define-a-3-level-nested-dictionary-in-python/
- Python Mailing List. *Dictionary Nesting — Tutor mailing list*. https://mail.python.org/pipermail/tutor/2008-January/060312.html
- Python Software Foundation. *bpo-33609: Document dict insertion order guarantee as of 3.7*. https://mail.python.org/archives/list/python-checkins@python.org/thread/GRSP77INSOBTY2VPUOK7AHRYZAIL4OCB/