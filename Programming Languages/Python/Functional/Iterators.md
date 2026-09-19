# Python Iterators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
An iterator is an object that represents a stream of data and produces one element at a time, while an iterable is an object that can produce an iterator.

**Technical Definition**
In Python, an **iterator** is an object that implements the iterator protocol: it has a `__iter__()` method that returns the iterator object itself, and a `__next__()` method that returns the next item or raises `StopIteration` when exhausted. An **iterable** is an object that implements `__iter__()`, returning an iterator, or implements `__getitem__()` with sequential integer indices (the sequence protocol). The built-in `iter()` function converts an iterable to an iterator, and `next()` retrieves the next item from an iterator. Iterators enable lazy evaluation, producing values on demand rather than computing all values upfront, which makes them memory-efficient for large or infinite data streams.

**Beginner-Friendly Explanation**
Imagine a deck of cards. The deck is an **iterable** — you can ask it for a way to go through the cards one by one. The **iterator** is like your hand that deals the next card each time you ask. Once all cards are dealt, the deck (iterator) is empty and cannot be reused. This is fundamentally different from a list, which you can loop over as many times as you like. Iterators are like a one-way stream: once you've consumed the data, it's gone.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Iterator Protocol** | Objects with `__iter__()` and `__next__()` methods |
| **Iterable Protocol** | Objects with `__iter__()` method returning an iterator |
| **Lazy Evaluation** | Values produced on demand, not all at once |
| **Single-Pass** | Iterators are exhausted after one full traversal |
| **Memory Efficient** | Only one item in memory at a time |
| **`iter()` and `next()`** | Built-in functions to obtain and advance iterators |
| **Customizable** | Classes can implement the protocol for custom iteration |

### Prerequisites

- **Basic Python syntax**: variables, functions, classes
- **Understanding of `for` loops**: how iteration works
- **Familiarity with lists and tuples**: as iterable examples
- **Basic concepts of object-oriented programming**: classes and methods

### Related Programming Areas

- **Generators**: Functions using `yield` that create iterators
- **Comprehensions**: List/set/dict comprehensions that use iteration internally
- **`itertools` Module**: Tools for efficient iterator-based operations
- **Asynchronous Iterators**: `__aiter__` and `__anext__` for async iteration
- **Functional Programming**: `map()`, `filter()`, and lazy pipelines

### Core Concepts / Features

1. **Iterable vs. Iterator** (The fundamental distinction)
2. **`iter()`** (Obtaining iterators, including the two-argument sentinel form)
3. **`next()`** (Advancing iterators, with default fallback values)
4. **Iterator Protocol** (`__iter__` and `__next__`)
5. **Custom Iterators** (Building your own iterable and iterator classes)
6. **Lazy Evaluation and Single-Pass Limitation** (A common beginner trap)


## Core Concept 1: Iterable vs. Iterator

### Definitions

**Core Definition**
An iterable is an object you can loop over; an iterator is the object that actually produces the values during iteration.

**Technical Definition**
An **iterable** is an object which can be passed to the built-in `iter()` function, which returns an iterator. An **iterator** is an object with a `__next__()` method (or `.next()` in Python 2), which is used to invoke the iteration. An iterator should also be an iterable, and will nearly always return itself as its own iterator . The key distinction is state: iterators store the state needed to iterate over a given iterable, while iterables themselves do not track iteration progress . A list is iterable, but not an iterator; `iter(list)` returns a list iterator.

**Beginner-Friendly Explanation**
Think of a list as a book and an iterator as a bookmark. You can read the book (iterate) from start to finish, but the bookmark (iterator) keeps track of where you are. If you want to read the book again, you need a new bookmark. The book (iterable) is always the same; the bookmark (iterator) is what changes as you read.

### Purposes

- **To distinguish between data and the process of accessing data**
- **To enable multiple independent iterations** over the same data (each `iter()` call creates a fresh iterator)
- **To support lazy evaluation** where data is produced only when needed
- **To provide a consistent interface** for looping over any collection

### Syntax Rules and Structure

**Key Differences**

| Aspect | Iterable | Iterator |
|---|---|---|
| Has `__iter__` | Yes | Yes (returns self) |
| Has `__next__` | No | Yes |
| Stateful | No (usually) | Yes |
| Can be reused | Yes | No (single-pass) |
| Example | `[1, 2, 3]` | `iter([1, 2, 3])` |
| `for` loop usage | `for x in iterable` | `for x in iterator` |

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Iterable vs. Iterator Behavior**

```python
# Step 1: A list is iterable
my_list = [1, 2, 3]
print(f"Has __iter__: {hasattr(my_list, '__iter__')}")
print(f"Has __next__: {hasattr(my_list, '__next__')} \n")

# Step 2: Get an iterator from the list
my_iter = iter(my_list)
print(f"Has __iter__: {hasattr(my_iter, '__iter__')}")
print(f"Has __next__: {hasattr(my_iter, '__next__')} \n")

# Step 3: The iterator is exhausted after one pass
print(f"First pass  : {list(my_iter)}")
print(f"Second pass : {list(my_iter)}")  # Empty!
```

**Expected Output:**
```
Has __iter__: True
Has __next__: False

Has __iter__: True
Has __next__: True

First pass  : [1, 2, 3]
Second pass : []
```

**Breakdown:** The list has `__iter__` but not `__next__`; the iterator has both. After one full traversal, the iterator is exhausted and returns an empty list on the second pass.

**Example 2: Multiple Iterators from One Iterable**

```python
# Step 1: Create a list (iterable)
data = ["a", "b", "c"]

# Step 2: Create two independent iterators
iter1 = iter(data)
iter2 = iter(data)

# Step 3: They advance independently
print(next(iter1))  # a
print(next(iter2))  # a
print(next(iter1))  # b
print(next(iter2))  # b
```

**Expected Output:**
```
a
a
b
b
```

**Breakdown:** Each `iter()` call produces a fresh, independent iterator. They do not share state, so iterating one does not affect the other.

**Example 3: Iterators Are Also Iterable**

```python
# Step 1: Create an iterator
my_iter = iter([1, 2, 3])

# Step 2: `iter()` on an iterator returns itself
same_iter = iter(my_iter)
print(f"Same object: {my_iter is same_iter}")

# Step 3: A for loop works on an iterator
for x in my_iter:
    print(x)
```

**Expected Output:**
```
Same object: True
1
2
3
```

**Breakdown:** `iter()` on an iterator returns the same object (because iterators implement `__iter__` to return `self`). This makes iterators usable in `for` loops directly.

### Real-World Cases with Explanation

**Case 1: File Objects Are Iterators**

```python
with open("data.txt") as f:
    for line in f:  # f is an iterator
        process(line)
```

**Why it matters:** File objects are iterators that yield lines one at a time, making them memory-efficient for large files.

**Case 2: Generators Are Iterators**

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for x in countdown(5):
    print(x)
```

**Why it matters:** Generator functions return iterators, which can be used directly in loops. They produce values lazily.


## Core Concept 2: `iter()`

### Definitions

**Core Definition**
`iter()` is a built-in function that returns an iterator from an iterable or, in its two-argument form, creates an iterator from a callable and a sentinel value.

**Technical Definition**
The built-in `iter()` function has two forms. `iter(iterable)` returns an iterator object for the given iterable by calling its `__iter__()` method or, if absent, using the sequence protocol (`__getitem__` with integer indices starting at 0). `iter(callable, sentinel)` creates an iterator that calls `callable` with no arguments for each `__next__()` call; if the returned value equals `sentinel`, `StopIteration` is raised, otherwise the value is returned . The two-argument form is useful for block readers and I/O loops, such as reading fixed-width blocks until EOF.

**Beginner-Friendly Explanation**
`iter()` has two jobs. Normally, you give it a list, tuple, or other collection, and it gives you an iterator to go through it. The second form is less common: you give it a function and a "stop" value. Each time you ask for the next item, it calls the function. When the function returns the stop value, iteration ends. This is useful for things like reading chunks of a file until the end.

### Purposes

- **To obtain an iterator** from any iterable
- **To create an iterator from a callable** that produces values until a sentinel is reached
- **To enable `for` loops** to work with any iterable
- **To implement block readers** that read chunks of data until EOF
- **To support custom iteration protocols** in user-defined classes

### Syntax Rules and Structure

**Complete General Syntax**

```
iter(iterable)
iter(callable, sentinel)
```

**Breakdown:**
- `iterable`: Any object supporting `__iter__()` or the sequence protocol.
- `callable`: A callable object (function, method, etc.) that takes no arguments.
- `sentinel`: The value that, when returned by `callable`, terminates iteration.

**Syntax Rules**

| Form | First Argument | Second Argument | Returns |
|---|---|---|---|
| `iter(iterable)` | Iterable | Not provided | Iterator |
| `iter(callable, sentinel)` | Callable | Sentinel value | Iterator |

**Constraints and Limitations**

- **`TypeError` if not iterable**: `iter(42)` raises `TypeError`.
- **Callable must take no arguments**: `iter(lambda x: x, 0)` raises `TypeError`.
- **Sentinel comparison uses `==`**: The callable's return value is compared to the sentinel using `==`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `iter()`**

```python
# Step 1: Get an iterator from a list
my_iter = iter([10, 20, 30])
print(type(my_iter))

# Step 2: Advance the iterator
print(next(my_iter))  # 10
print(next(my_iter))  # 20
print(next(my_iter))  # 30
```

**Expected Output:**
```
<class 'list_iterator'>
10
20
30
```

**Breakdown:** `iter()` returns a list iterator. `next()` retrieves each value in sequence.

**Example 2: Two-Argument `iter()` (Sentinel Form)**

```python
import random

# Step 1: Create an iterator that calls a function until sentinel
def roll_dice():
    return random.randint(1, 6)

# Stop when we roll a 6
dice_iter = iter(roll_dice, 6)

# Step 2: Consume the iterator
for roll in dice_iter:
    print(f"Rolled: {roll}")
print("Got a 6! Stopping.")
```

**Expected Output (example):**
```
Rolled: 3
Rolled: 1
Rolled: 5
Rolled: 2
Got a 6! Stopping.
```

**Breakdown:** `iter(roll_dice, 6)` creates an iterator that calls `roll_dice()` repeatedly. When the function returns `6`, iteration stops.

**Example 3: Block Reader with `iter()`**

```python
from functools import partial

# Step 1: Read a file in 64-byte blocks until EOF
with open('data.bin', 'rb') as f:
    # partial(f.read, 64) creates a callable that reads 64 bytes
    for block in iter(partial(f.read, 64), b''):
        process_block(block)
```

**Expected Output:** (Depends on file contents; processes each 64-byte block)

**Breakdown:** `partial(f.read, 64)` creates a callable that reads 64 bytes. `iter(callable, b'')` stops when the callable returns an empty bytes object (`b''`), which signals EOF. This pattern is documented in the Python standard library as a useful application of the two-argument form .

### Real-World Cases with Explanation

**Case 1: Reading Until a Sentinel Value**

```python
# Read user input until an empty line
lines = []
for line in iter(input, ''):
    lines.append(line)
```

**Why it matters:** The two-argument `iter()` provides a clean way to read input until a sentinel (empty string) is entered.

**Case 2: Consuming a Queue Until Empty**

```python
import queue

q = queue.Queue()
# ... populate queue ...

for item in iter(q.get_nowait, queue.Empty):
    process(item)
```

**Why it matters:** `iter(callable, sentinel)` elegantly consumes a queue until it raises a specific exception (treated as the sentinel).


## Core Concept 3: `next()`

### Definitions

**Core Definition**
`next()` is a built-in function that retrieves the next item from an iterator, optionally returning a default value instead of raising `StopIteration` when the iterator is exhausted.

**Technical Definition**
`next(iterator)` calls the iterator's `__next__()` method, returning the next item or raising `StopIteration` if exhausted. `next(iterator, default)` returns `default` instead of raising `StopIteration` when the iterator is exhausted . This is particularly useful for preventing exceptions in loops that manually consume iterators or when providing fallback values. The default argument is evaluated lazily—only when the iterator is exhausted.

**Beginner-Friendly Explanation**
`next()` asks an iterator for its next value. If there are no more values, it normally raises an error (`StopIteration`). But you can give it a default value, and then it returns that default instead of crashing. It's like asking "What's the next item? If there isn't one, just give me this instead."

### Purposes

- **To manually advance an iterator** one item at a time
- **To prevent `StopIteration` exceptions** by providing a default value
- **To skip the first item** of an iterator (e.g., CSV header)
- **To implement manual iteration loops** where `for` is not suitable
- **To provide fallback values** when an iterator may be empty

### Syntax Rules and Structure

**Complete General Syntax**

```
next(iterator)
next(iterator, default)
```

**Breakdown:**
- `iterator`: An iterator object.
- `default`: Optional value to return when the iterator is exhausted.

**Return Values**

| Situation | `next(iterator)` | `next(iterator, default)` |
|---|---|---|
| Items remain | Next item | Next item |
| Exhausted | Raises `StopIteration` | Returns `default` |

**Constraints and Limitations**

- **`TypeError` if not an iterator**: `next([1, 2, 3])` raises `TypeError`.
- **Default is evaluated eagerly**: The default expression is evaluated even if not needed (in some contexts).
- **`StopIteration` in generators**: Raising `StopIteration` inside a generator is treated as `RuntimeError` in Python 3.7+.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `next()`**

```python
# Step 1: Create an iterator
my_iter = iter([1, 2, 3])

# Step 2: Retrieve items one by one
print(next(my_iter))  # 1
print(next(my_iter))  # 2
print(next(my_iter))  # 3

# Step 3: Exhausted — raises StopIteration
try:
    print(next(my_iter))
except StopIteration:
    print("Iterator exhausted")
```

**Expected Output:**
```
1
2
3
Iterator exhausted
```

**Breakdown:** `next()` returns each item in sequence. When exhausted, it raises `StopIteration`.

**Example 2: `next()` with Default Value**

```python
# Step 1: Create an empty iterator
empty_iter = iter([])

# Step 2: Without default — raises StopIteration
try:
    next(empty_iter)
except StopIteration:
    print("No items (exception)")

# Step 3: With default — returns default
result = next(empty_iter, "No items left")
print(result)
```

**Expected Output:**
```
No items (exception)
No items left
```

**Breakdown:** The default value prevents `StopIteration` and provides a fallback.

**Example 3: Skipping a Header Line**

```python
# Step 1: Read a CSV file, skipping the header
with open("data.csv") as f:
    header = next(f)  # Read and discard the header line
    print(f"Header: {header.strip()}")
    for line in f:
        print(line.strip())
```

**Expected Output:**
```
Header: name,age,city
Alice,30,NYC
Bob,25,LA
```

**Breakdown:** `next(f)` reads the first line (header) and advances the file iterator. The `for` loop then processes the remaining lines.

### Real-World Cases with Explanation

**Case 1: Safely Getting the First Item**

```python
def get_first(iterable, default=None):
    return next(iter(iterable), default)
```

**Why it matters:** `next()` with a default provides a safe way to get the first item without checking if the iterable is empty.

**Case 2: Manual Iterator Control in Loops**

```python
data = iter([1, 2, 3, 4, 5])
while True:
    try:
        value = next(data)
    except StopIteration:
        break
    if value % 2 == 0:
        print(f"First even: {value}")
        break
```

**Why it matters:** Manual `next()` calls allow fine-grained control over iteration, such as breaking based on a condition.


## Core Concept 4: Iterator Protocol

### Definitions

**Core Definition**
The iterator protocol is the set of two methods—`__iter__()` and `__next__()`—that an object must implement to be an iterator.

**Technical Definition**
The iterator protocol defines how objects provide iteration. An iterator must implement `__iter__()` (which returns the iterator object itself) and `__next__()` (which returns the next item or raises `StopIteration`). An iterable must implement `__iter__()` (which returns an iterator) or, alternatively, `__getitem__()` with integer indices starting at 0 (the sequence protocol). The `for` loop calls `iter()` on the iterable, then repeatedly calls `next()` until `StopIteration` is raised . This protocol is the foundation of all iteration in Python, including comprehensions, generators, and `for` loops.

**Beginner-Friendly Explanation**
The iterator protocol is like a contract: "If you want to be iterable in Python, you must provide a way to get an iterator (`__iter__`). If you want to be an iterator, you must provide a way to get the next item (`__next__`) and know when to stop (raise `StopIteration`)." Python's `for` loop follows this contract automatically.

### Purposes

- **To define a standard interface** for iteration
- **To enable `for` loops** to work with any object implementing the protocol
- **To support lazy evaluation** through `__next__()` producing one item at a time
- **To allow custom objects** to be iterable
- **To integrate with comprehensions, generators, and built-in functions**

### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyIterator:
    def __iter__(self):
        return self

    def __next__(self):
        # Return next value or raise StopIteration
        ...

class MyIterable:
    def __iter__(self):
        # Return an iterator
        return MyIterator(...)
```

**Breakdown:**
- `__iter__()`: Returns an iterator object (usually `self` for iterators).
- `__next__()`: Returns the next item; raises `StopIteration` when exhausted.

**Protocol Rules**

| Method | Required For | Returns |
|---|---|---|
| `__iter__` | Iterable and iterator | Iterator object |
| `__next__` | Iterator only | Next item or raises `StopIteration` |
| `__getitem__` | Alternative iterable | Item at integer index |

**Constraints and Limitations**

- **`StopIteration` signals exhaustion**: Must be raised to end iteration.
- **`__iter__` on iterators returns self**: This makes iterators iterable.
- **No `__next__` on iterables**: A list is iterable but not an iterator.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: How `for` Uses the Protocol**

```python
# Step 1: A for loop internally does this:
my_list = [1, 2, 3]

# Equivalent to:
iterator = iter(my_list)
while True:
    try:
        item = next(iterator)
    except StopIteration:
        break
    print(item)

# Step 2: The for loop does it automatically
for item in my_list:
    print(item)
```

**Expected Output:**
```
1
2
3
1
2
3
```

**Breakdown:** The `for` loop calls `iter()` once, then `next()` repeatedly until `StopIteration`. The manual version demonstrates the protocol explicitly.

**Example 2: Sequence Protocol (Alternative)**

```python
# Step 1: A class with __getitem__ but no __iter__
class MySequence:
    def __getitem__(self, index):
        if index >= 3:
            raise IndexError
        return index * 10

# Step 2: It's still iterable via the sequence protocol
for x in MySequence():
    print(x)
```

**Expected Output:**
```
0
10
20
```

**Breakdown:** The sequence protocol (`__getitem__` with integer indices) provides fallback iteration. Python's `iter()` uses it if `__iter__` is absent.

**Example 3: Custom Iterator Implementing Both Methods**

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

# Use it
for x in Countdown(3):
    print(x)
```

**Expected Output:**
```
3
2
1
```

**Breakdown:** `Countdown` implements both `__iter__` (returns self) and `__next__` (returns values, raises `StopIteration` when done).

### Real-World Cases with Explanation

**Case 1: Custom Data Structures**

```python
class LinkedList:
    def __iter__(self):
        current = self.head
        while current:
            yield current.value
            current = current.next
```

**Why it matters:** Custom data structures implement `__iter__` to support `for` loops, making them behave like built-in collections.

**Case 2: Lazy Data Loading**

```python
class DatabaseIterator:
    def __init__(self, query):
        self.cursor = execute(query)

    def __iter__(self):
        return self

    def __next__(self):
        row = self.cursor.fetchone()
        if row is None:
            raise StopIteration
        return row
```

**Why it matters:** Database cursors are naturally iterators, fetching rows one at a time without loading the entire result set into memory.


## Core Concept 5: Custom Iterators

### Definitions

**Core Definition**
A custom iterator is a user-defined class that implements the iterator protocol to provide custom iteration behaviour.

**Technical Definition**
Custom iterators are classes that implement `__iter__()` and `__next__()` to control how iteration proceeds. They maintain internal state (e.g., a current index or position) and produce values on demand. Custom iterables are classes that implement `__iter__()` to return an iterator (often a separate class or a generator). The distinction allows separating the data (iterable) from the iteration state (iterator), enabling multiple independent iterations over the same data.

**Beginner-Friendly Explanation**
Sometimes you need to loop over something that isn't a built-in collection—like a range with custom steps, a tree structure, or data from a database. You can create your own iterator by writing a class with `__iter__` and `__next__` methods. This gives you complete control over what values are produced and when iteration stops.

### Purposes

- **To iterate over custom data structures** (trees, graphs, linked lists)
- **To generate sequences** with custom logic (e.g., Fibonacci numbers)
- **To read data from external sources** lazily (files, databases, APIs)
- **To separate iteration state** from the underlying data
- **To support multiple independent iterations** over the same data

### Syntax Rules and Structure

**Complete General Syntax**

```python
class MyIterator:
    def __init__(self, ...):
        # Initialize state
        ...

    def __iter__(self):
        return self

    def __next__(self):
        # Compute next value
        if exhausted:
            raise StopIteration
        return value

class MyIterable:
    def __init__(self, data):
        self.data = data

    def __iter__(self):
        return MyIterator(self.data)
```

**Breakdown:**
- The iterator class maintains state (e.g., index, current node).
- `__iter__()` returns `self` (for iterators) or a new iterator (for iterables).
- `__next__()` returns the next value or raises `StopIteration`.

**Design Patterns**

| Pattern | Description | Example |
|---|---|---|
| Self-iterator | `__iter__` returns self | `Countdown` |
| Separate iterator | `__iter__` returns a new iterator class | `LinkedList` |
| Generator-based | `__iter__` uses `yield` | `Tree` traversal |

**Constraints and Limitations**

- **Iterators are single-pass**: Once exhausted, they cannot be reset.
- **State management**: Complex state requires careful initialization.
- **Not thread-safe**: Concurrent iteration can cause race conditions.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Custom Range Iterator**

```python
class MyRange:
    def __init__(self, start, end, step=1):
        self.start = start
        self.end = end
        self.step = step

    def __iter__(self):
        self.current = self.start
        return self

    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += self.step
        return value

# Use it
for x in MyRange(0, 10, 2):
    print(x, end=" ")
```

**Expected Output:**
```
0 2 4 6 8 
```

**Breakdown:** `MyRange` implements the iterator protocol directly. `__next__` checks bounds, returns the current value, and increments by the step.

**Example 2: Fibonacci Iterator**

```python
class Fibonacci:
    def __init__(self, limit):
        self.limit = limit
        self.a, self.b = 0, 1
        self.count = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.count >= self.limit:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return value

# Use it
for num in Fibonacci(10):
    print(num, end=" ")
```

**Expected Output:**
```
0 1 1 2 3 5 8 13 21 34 
```

**Breakdown:** The Fibonacci iterator maintains state (`a`, `b`, `count`) and produces the next number in the sequence on each call.

**Example 3: Iterable vs. Iterator Separation**

```python
class NumberLine:
    """Iterable that returns a fresh iterator each time."""
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __iter__(self):
        return NumberLineIterator(self.start, self.end)

class NumberLineIterator:
    """Iterator for NumberLine."""
    def __init__(self, current, end):
        self.current = current
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# Use it
line = NumberLine(1, 5)
print("First pass:", list(line))
print("Second pass:", list(line))  # Works! Fresh iterator each time
```

**Expected Output:**
```
First pass: [1, 2, 3, 4, 5]
Second pass: [1, 2, 3, 4, 5]
```

**Breakdown:** `NumberLine` is iterable (returns a new iterator each time). `NumberLineIterator` is the iterator (maintains state). This separation allows multiple independent iterations.

### Real-World Cases with Explanation

**Case 1: Tree Traversal**

```python
class TreeNode:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

class DepthFirstIterator:
    def __init__(self, root):
        self.stack = [root]

    def __iter__(self):
        return self

    def __next__(self):
        if not self.stack:
            raise StopIteration
        node = self.stack.pop()
        self.stack.extend(reversed(node.children))
        return node.value
```

**Why it matters:** Custom iterators enable elegant traversal of tree structures, which are common in file systems, XML parsing, and algorithms.

**Case 2: Paginated API Results**

```python
class PaginatedAPI:
    def __init__(self, endpoint):
        self.endpoint = endpoint
        self.page = 1
        self.exhausted = False

    def __iter__(self):
        return self

    def __next__(self):
        if self.exhausted:
            raise StopIteration
        response = fetch(self.endpoint, page=self.page)
        if not response["items"]:
            self.exhausted = True
            raise StopIteration
        self.page += 1
        return response["items"]
```

**Why it matters:** APIs often return paginated results. A custom iterator hides the pagination logic, providing a simple stream of items.


## Core Concept 6: Lazy Evaluation and Single-Pass Limitation

### Definitions

**Core Definition**
Lazy evaluation means values are produced only when requested, and the single-pass limitation means iterators can only be traversed once.

**Technical Definition**
Iterators implement lazy evaluation: they compute and return values on demand rather than precomputing all values. This makes them memory-efficient for large or infinite data streams. However, iterators are single-pass: once exhausted (all values consumed), they cannot be reset or reused. Calling `iter()` on the same iterator returns the same exhausted iterator. To iterate again, you must create a new iterator from the original iterable (if it still exists) or materialise the data into a list. Generator expressions and generator functions exhibit the same behaviour .

**Beginner-Friendly Explanation**
Lazy evaluation is like a water tap: water flows only when you open it. You don't store all the water upfront. The single-pass limitation is like a one-way street: once you've driven down it, you can't go back. Iterators are consumed as you use them. If you need to loop over the same data twice, you need either two iterators or a reusable iterable (like a list).

### Purposes

- **To handle large datasets** without loading everything into memory
- **To support infinite sequences** (e.g., prime numbers, sensor data)
- **To improve performance** by avoiding unnecessary computation
- **To enable pipeline processing** where data flows through stages
- **To understand a common source of bugs** (reusing exhausted iterators)

### Syntax Rules and Structure

**Lazy vs. Eager**

| Aspect | Lazy (Iterator) | Eager (List) |
|---|---|---|
| Memory | Low (one item at a time) | High (all items stored) |
| Evaluation | On demand | Immediate |
| Reusable | No | Yes |
| Infinite possible | Yes | No |
| Example | `(x**2 for x in range(10))` | `[x**2 for x in range(10)]` |

**Single-Pass Rules**

| Operation | Effect |
|---|---|
| `next(iterator)` | Advances iterator; consumed item is gone |
| `list(iterator)` | Consumes all remaining items |
| `for x in iterator` | Consumes items as loop runs |
| `iter(iterator)` | Returns the same (possibly exhausted) iterator |

**Constraints and Limitations**

- **No reset**: Iterators cannot be rewound.
- **No `len()`**: Iterators do not support length queries.
- **No indexing**: Iterators do not support `iterator[0]`.
- **Shared state**: Multiple references to the same iterator share consumption.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Lazy Evaluation with Generators**

```python
import sys

# Step 1: Generator expression (lazy)
gen = (x ** 2 for x in range(1000000))
print(f"Generator size: {sys.getsizeof(gen)} bytes")

# Step 2: List comprehension (eager)
lst = [x ** 2 for x in range(1000000)]
print(f"List size: {sys.getsizeof(lst)} bytes")
```

**Expected Output:**
```
Generator size: 200 bytes
List size: 8448728 bytes
```

**Breakdown:** The generator uses minimal memory (200 bytes) because it produces values lazily. The list uses 8.4 MB because it stores all values.

**Example 2: Single-Pass Trap**

```python
# Step 1: Create a generator
gen = (x for x in range(3))

# Step 2: First consumption
print("First pass:", list(gen))

# Step 3: Second consumption — EMPTY!
print("Second pass:", list(gen))
```

**Expected Output:**
```
First pass: [0, 1, 2]
Second pass: []
```

**Breakdown:** The generator is exhausted after the first `list()` call. The second call returns an empty list. This is a common source of bugs.

**Example 3: Fixing the Single-Pass Trap**

```python
# Step 1: Use a reusable iterable (list)
data = [x for x in range(3)]

print("First pass:", list(data))
print("Second pass:", list(data))  # Works!

# Step 2: Or create a function that returns a fresh iterator
def get_gen():
    return (x for x in range(3))

print("First pass:", list(get_gen()))
print("Second pass:", list(get_gen()))  # Works!
```

**Expected Output:**
```
First pass: [0, 1, 2]
Second pass: [0, 1, 2]
First pass: [0, 1, 2]
Second pass: [0, 1, 2]
```

**Breakdown:** Using a list (reusable iterable) or a function that creates a fresh generator each time solves the single-pass problem.

### Real-World Cases with Explanation

**Case 1: Large File Processing**

```python
# Lazy: process lines one at a time
with open("huge_file.txt") as f:
    for line in f:  # File iterator is lazy
        process(line)
```

**Why it matters:** Lazy evaluation allows processing files larger than available memory.

**Case 2: Pipeline Processing**

```python
# Chain generators for a lazy pipeline
numbers = (x for x in range(1000))
evens = (x for x in numbers if x % 2 == 0)
squares = (x ** 2 for x in evens)
result = sum(squares)  # Only now is anything computed
```

**Why it matters:** Generator pipelines process data one item at a time, avoiding intermediate lists and saving memory.

**Case 3: Infinite Sequences**

```python
def natural_numbers():
    n = 1
    while True:
        yield n
        n += 1

# Take the first 5 natural numbers
gen = natural_numbers()
for _ in range(5):
    print(next(gen))
```

**Why it matters:** Iterators can represent infinite sequences because they produce values on demand. Lists cannot.


## References

- Python Software Foundation. *Iterator Protocol — Python Language Reference*. https://docs.python.org/3/c-api/iter.html
- Python Software Foundation. *Built-in Functions — iter()*. https://docs.python.org/3/library/functions.html#iter
- Python Software Foundation. *Built-in Functions — next()*. https://docs.python.org/3/library/functions.html#next
- Python Software Foundation. *Iterator Types — Python Standard Library*. https://docs.python.org/3/library/stdtypes.html#iterator-types
- Python Software Foundation. *PEP 234 – Iterators*. https://peps.python.org/pep-0234/
- Python Software Foundation. *PEP 255 – Simple Generators*. https://peps.python.org/pep-0255/
- Python Software Foundation. *PEP 479 – Change StopIteration handling inside generators*. https://peps.python.org/pep-0479/
- Real Python. *Iterators and Iterables in Python: Run Efficient Iterations*. https://realpython.com/python-iterators-iterables/
- Real Python. *next() | Python's Built-in Functions*. https://realpython.com/ref/builtin-functions/next/
- Python-Dev Mailing List. *Terminology of "Iterable" and "Iterator"*. https://mail.python.org/archives/list/python-dev@python.org/thread/UZO5WFMONSVC4A5UR2S6M6FM6U6FQRIK/
- Python-List Mailing List. *classes and __iter__*. https://mail.python.org/pipermail/python-list/2012-January/618126.html
- Python-List Mailing List. *Lazy evaluation and generators*. https://mail.python.org/pipermail/tutor/2013-January/092407.html