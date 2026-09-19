# Python Asynchronous Iteration: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Asynchronous iteration is Python's mechanism for traversing sequences of values where each step may involve waiting for I/O or other asynchronous operations without blocking the event loop.

**Technical Definition**
Asynchronous iteration extends Python's iterator protocol to the `async`/`await` paradigm. An **asynchronous iterable** implements `__aiter__()`, which returns an **asynchronous iterator**. The asynchronous iterator implements `__anext__()`, which returns an awaitable that eventually yields the next value or raises `StopAsyncIteration` to signal exhaustion. This protocol is consumed by the `async for` statement and enables memory-efficient, non-blocking data streams.

**Beginner-Friendly Explanation**
Imagine reading a large file from a slow network drive. With a regular `for` loop, your program would freeze at each read until the data arrives. Asynchronous iteration lets your program say, "Start reading the next chunk, and while I'm waiting, go do something else." When the data is ready, Python resumes where it left off. This is essential for programs that handle many concurrent I/O operations—like web servers, chat applications, or data pipelines.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Non-Blocking** | Each iteration step may await I/O without freezing the entire program |
| **Lazy Evaluation** | Values are produced on demand, not pre-computed into a list |
| **Protocol-Based** | Implemented through `__aiter__()` and `__anext__()` special methods |
| **Event-Loop Driven** | Requires an active `asyncio` event loop to run |
| **Introduced in Python 3.5+** | PEP 492 introduced `async for`; PEP 525 added async generators in 3.6; PEP 530 added async comprehensions in 3.6 |

### Prerequisites

- **Basic `async`/`await` syntax**: How to define and call coroutines with `async def` and `await`
- **Python iterators**: Understanding `__iter__()` and `__next__()` (the synchronous counterpart)
- **The `asyncio` event loop**: How to run asynchronous code with `asyncio.run()` or `loop.run_until_complete()`
- **Basic I/O concepts**: Why network and file operations benefit from non-blocking execution
- **Python 3.6+** (most examples use async generators and comprehensions; `aiter()`/`anext()` require 3.10+)

### Related Programming Areas

- **Asyncio**: Python's standard library for asynchronous programming, providing event loops, tasks, and queues
- **Concurrency vs. Parallelism**: Async iteration enables concurrency (overlapping I/O) without threads
- **Generators (Synchronous)**: The conceptual predecessor; async generators mirror `yield` in `async def` functions
- **Stream Processing**: Async iteration is the foundation for processing data streams (network packets, log lines, database cursors)
- **Backpressure**: Async generators and queues help manage producer–consumer flow control

### Core Concepts / Features

1. **The Asynchronous Iterator Protocol** (`__aiter__` and `__anext__`)
2. **Asynchronous Comprehensions** (`[await x for x in async_iterable]`)
3. **Asynchronous Generators** (`async def` with `yield`)

---

## Core Concept 1: The Asynchronous Iterator Protocol

### Definitions

**Core Definition**
The asynchronous iterator protocol defines two special methods—`__aiter__()` and `__anext__()`—that an object must implement to be usable in an `async for` loop.

**Technical Definition**
An **asynchronous iterable** implements `__aiter__()`, which must return an asynchronous iterator. An **asynchronous iterator** implements both `__aiter__()` (returning `self`) and `__anext__()`, which returns an awaitable. When awaited, `__anext__()` produces the next value or raises `StopAsyncIteration` to signal that iteration is complete. The built-in `aiter()` and `anext()` functions provide convenient access to these methods, analogous to `iter()` and `next()` for synchronous iterators.

**Beginner-Friendly Explanation**
Think of an asynchronous iterator as a "ticket dispenser" that works over a slow connection. You press the button (`await anext(iterator)`), wait for the ticket to arrive (the awaitable completes), and receive either a ticket (a value) or a "sold out" signal (`StopAsyncIteration`). The `__aiter__()` method is like the sign on the machine saying "I am a ticket dispenser."

### Purposes

- **To enable `async for` loops** over data sources that require awaiting (network sockets, database cursors, async queues)
- **To provide a standard interface** for asynchronous iteration, mirroring the synchronous iterator protocol
- **To support lazy, non-blocking data production** where values arrive over time
- **To integrate with `asyncio` constructs** such as queues and streams
- **To allow custom objects** to participate in asynchronous iteration without writing a full async generator

### Syntax Rules and Structure

**Complete General Syntaxes**

```
# Asynchronous iterable
class AsyncIterable:
    def __aiter__(self):
        return AsyncIterator(...)

# Asynchronous iterator
class AsyncIterator:
    def __aiter__(self):
        return self

    async def __anext__(self):
        if <exhausted>:
            raise StopAsyncIteration
        return <next_value>
```

**Breakdown:**
- `__aiter__(self)`: Called by `aiter()` or by `async for` to obtain an asynchronous iterator. Must return an object that implements `__anext__()`.
- `__anext__(self)`: An `async def` method (a coroutine) that returns the next value or raises `StopAsyncIteration`. It is called by `anext()` or implicitly by `async for`.
- `StopAsyncIteration`: The exception that signals the end of iteration. Unlike synchronous iteration (which uses `StopIteration`), this is a distinct exception type introduced in PEP 492.

**Syntax Rules**

| Rule | Description |
|---|---|
| `__aiter__` returns an iterator | Must return an object implementing `__anext__()`; returning an async generator is common |
| `__anext__` is a coroutine | Must be defined with `async def` and return an awaitable |
| Termination | Raise `StopAsyncIteration` to end iteration |
| `aiter()` / `anext()` built-ins | Available in Python 3.10+; `aiter()` calls `__aiter__()`, `anext()` awaits `__anext__()` |
| No `__anext__` on iterables | An object with only `__aiter__()` is an iterable but not an iterator |

**Constraints and Limitations**

- **Requires an event loop**: Asynchronous iterators only function inside an active `asyncio` event loop.
- **`StopAsyncIteration` propagation**: Raising `StopIteration` inside `__anext__()` is transformed into a `RuntimeError` to prevent accidental termination of coroutines.
- **Manual protocol implementation**: Writing `__aiter__`/`__anext__` by hand is more verbose than using async generators; prefer `async def` + `yield` when possible.
- **Python 3.5 only**: `__aiter__` was originally required to return an awaitable; this was changed in Python 3.5.2 to return an iterator directly. Modern code should return the iterator directly.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: A Minimal Custom Async Iterator**

```python
import asyncio

class Countdown:
    """Async iterator that counts down from a given number."""
    def __init__(self, start):
        self.current = start

    def __aiter__(self):
        # Return self because this object is both iterable and iterator
        return self

    async def __anext__(self):
        # Simulate a small delay (e.g., waiting for I/O)
        await asyncio.sleep(0.1)
        if self.current <= 0:
            raise StopAsyncIteration  # Signal end of iteration
        value = self.current
        self.current -= 1
        return value  # Return the next value

async def main():
    # Step 1: Create the async iterator
    counter = Countdown(3)

    # Step 2: Consume it with 'async for'
    async for number in counter:
        print(number)

# Run the async program
asyncio.run(main())
```

**Expected Output:**
```
3
2
1
```

**Breakdown:**
1. `Countdown.__aiter__()` returns `self`, making `Countdown` both an iterable and an iterator.
2. `__anext__()` awaits `asyncio.sleep(0.1)` to simulate an I/O wait, then returns the current value and decrements the counter.
3. When `current` reaches 0, `StopAsyncIteration` is raised, terminating the `async for` loop.
4. `asyncio.run(main())` starts the event loop and executes the coroutine.

**Example 2: Using `aiter()` and `anext()` Built-ins (Python 3.10+)**

```python
import asyncio

async def main():
    # Step 1: Create an async iterable
    async def async_range(n):
        for i in range(n):
            await asyncio.sleep(0.1)
            yield i

    # Step 2: Get an async iterator using aiter()
    iterator = aiter(async_range(3))

    # Step 3: Manually consume using anext()
    try:
        while True:
            value = await anext(iterator)
            print(value)
    except StopAsyncIteration:
        print("Iteration complete")

asyncio.run(main())
```

**Expected Output:**
```
0
1
2
Iteration complete
```

**Breakdown:**
1. `async_range` is an async generator (see Core Concept 3).
2. `aiter()` calls `async_range(3).__aiter__()`, returning the async generator object itself.
3. `anext(iterator)` awaits the next value. When exhausted, it raises `StopAsyncIteration`, which the `try`/`except` block catches.
4. This pattern is useful when you need manual control over iteration rather than using `async for`.

**Example 3: A Custom Async Iterator with a Shared Queue**

```python
import asyncio

class QueueIterator:
    """Async iterator that yields items from an asyncio.Queue until a sentinel."""
    def __init__(self, queue, sentinel):
        self.queue = queue
        self.sentinel = sentinel

    def __aiter__(self):
        return self

    async def __anext__(self):
        item = await self.queue.get()  # Blocks (awaits) until an item is available
        if item is self.sentinel:
            raise StopAsyncIteration
        return item

async def producer(queue):
    for i in range(3):
        await asyncio.sleep(0.2)
        await queue.put(f"item-{i}")
    await queue.put(None)  # Sentinel to stop iteration

async def main():
    queue = asyncio.Queue()
    # Start producer concurrently
    asyncio.create_task(producer(queue))

    # Consume queue items with async for
    async for item in QueueIterator(queue, None):
        print(f"Consumed: {item}")

asyncio.run(main())
```

**Expected Output:**
```
Consumed: item-0
Consumed: item-1
Consumed: item-2
```

**Breakdown:**
1. `QueueIterator.__anext__()` awaits `queue.get()`, which suspends the coroutine until an item is available.
2. The producer task runs concurrently, putting items into the queue with delays.
3. The sentinel `None` signals the end of iteration by raising `StopAsyncIteration`.
4. This pattern is common in producer–consumer architectures where items arrive asynchronously.

### Real-World Cases with Explanation

**Case 1: Streaming API Responses**

```python
import asyncio
import aiohttp

class PaginatedAPI:
    """Async iterator that fetches pages from a paginated REST API."""
    def __init__(self, session, base_url):
        self.session = session
        self.base_url = base_url
        self.page = 1
        self.has_more = True

    def __aiter__(self):
        return self

    async def __anext__(self):
        if not self.has_more:
            raise StopAsyncIteration
        url = f"{self.base_url}?page={self.page}"
        async with self.session.get(url) as response:
            data = await response.json()
        if not data.get("items"):
            raise StopAsyncIteration
        self.page += 1
        self.has_more = data.get("has_next", False)
        return data["items"]
```

**Why it matters:** REST APIs often paginate results. An async iterator lets you process each page as it arrives without blocking, and the `async for` loop handles the pagination logic transparently.

**Case 2: Database Cursor Iteration**

```python
class AsyncCursorIterator:
    """Wraps an async database cursor as an async iterator."""
    def __init__(self, cursor):
        self.cursor = cursor

    def __aiter__(self):
        return self

    async def __anext__(self):
        row = await self.cursor.fetchone()
        if row is None:
            raise StopAsyncIteration
        return row
```

**Why it matters:** Database drivers like `asyncpg` or `aiomysql` provide cursors that require `await` for each fetch. Wrapping them in an async iterator enables clean `async for row in cursor:` syntax.

---

## Core Concept 2: Asynchronous Comprehensions

### Definitions

**Core Definition**
Asynchronous comprehensions allow the use of `async for` and `await` inside list, set, dict comprehensions, and generator expressions.

**Technical Definition**
Introduced in PEP 530, asynchronous comprehensions extend Python's comprehension syntax with the `async for` clause and allow `await` expressions in the output and filter clauses. The grammar adds an optional `async` keyword to the `comp_for` production: `comp_for: [ASYNC] 'for' exprlist 'in' or_test [comp_iter]`. Asynchronous comprehensions are only valid inside an `async def` function (in Python 3.6; the restriction was lifted in 3.7 once `async`/`await` became reserved keywords).

**Beginner-Friendly Explanation**
A regular list comprehension like `[x * 2 for x in data]` builds a list from a synchronous iterable. An asynchronous comprehension does the same but for an async iterable: `[await process(x) async for x in async_data]`. It's a compact way to write "for each item in this async stream, do something (possibly await) and collect the results."

### Purposes

- **To concisely build collections** from asynchronous iterables
- **To apply awaitable transformations** to each element of an async stream
- **To filter asynchronous streams** using `if` clauses with `await`
- **To reduce boilerplate** compared to manual `async for` loops with `append()`
- **To create async generator expressions** that can be lazily consumed

### Syntax Rules and Structure

**Complete General Syntaxes**

```
[EXPRESSION async for TARGET in ASYNC_ITERABLE if CONDITION]
{EXPRESSION async for TARGET in ASYNC_ITERABLE}
{KEY: VALUE async for TARGET in ASYNC_ITERABLE}
(EXPRESSION async for TARGET in ASYNC_ITERABLE)   # async generator expression
```

**Breakdown:**
- `EXPRESSION`: The value to collect. May contain `await` (e.g., `await process(x)`).
- `async for TARGET in ASYNC_ITERABLE`: Iterates over an asynchronous iterable.
- `if CONDITION`: Optional filter. May contain `await`.
- Enclosing brackets determine the result type: `[]` list, `{}` set or dict, `()` generator.

**Syntax Rules**

| Rule | Description |
|---|---|
| `async for` clause | Required for asynchronous comprehensions |
| `await` in expression | Allowed in both async and sync comprehensions inside `async def` |
| Multiple clauses | Can mix `for`, `async for`, and `if` clauses |
| Context restriction | In Python 3.6, only inside `async def`; lifted in 3.7 |
| Generator expression | `(x async for x in agen())` produces an async generator |

**Constraints and Limitations**

- **Event loop required**: Like all async code, comprehensions must run inside an event loop.
- **No `await` outside `async def`**: In Python 3.6, async comprehensions were restricted to `async def` bodies; in 3.7+ they can appear in any async context.
- **Eager consumption**: List, set, and dict comprehensions consume the entire async iterable immediately; use generator expressions for lazy evaluation.
- **Readability trade-off**: Deeply nested async comprehensions can be harder to read than explicit loops; prefer clarity when complexity grows.

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Async List Comprehension with Await**

```python
import asyncio

async def fetch_data(item):
    """Simulate fetching data for an item."""
    await asyncio.sleep(0.1)
    return f"processed-{item}"

async def main():
    items = [1, 2, 3]

    # Async comprehension: await fetch_data for each item
    results = [await fetch_data(item) for item in items]
    # This is a SYNCHRONOUS comprehension with await inside.
    # It runs sequentially, awaiting each fetch before the next.

    print(results)

asyncio.run(main())
```

**Expected Output:**
```
['processed-1', 'processed-2', 'processed-3']
```

**Breakdown:**
1. `[await fetch_data(item) for item in items]` is a regular list comprehension containing an `await` expression.
2. Because it is inside `async def main()`, the `await` is valid.
3. Each `fetch_data` call is awaited sequentially; the total time is approximately 0.3 seconds (3 × 0.1s).
4. This pattern is useful when you need results in order and each fetch depends on the previous one (or when concurrency is not desired).

**Example 2: Async Comprehension over an Async Iterable**

```python
import asyncio

async def async_range(n):
    """Async generator yielding 0..n-1 with a delay."""
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def main():
    # Async comprehension: async for over an async iterable
    squares = [i * i async for i in async_range(5)]
    # Iterates asynchronously and collects squares.

    print(squares)

asyncio.run(main())
```

**Expected Output:**
```
[0, 1, 4, 9, 16]
```

**Breakdown:**
1. `async_range(5)` is an async generator (see Core Concept 3).
2. The comprehension `[i * i async for i in async_range(5)]` awaits each value from the async generator.
3. The result is a list of squares: 0², 1², 2², 3², 4².
4. This is equivalent to writing a manual `async for` loop with `append()`, but more concise.

**Example 3: Async Comprehension with Filter and Await**

```python
import asyncio

async def is_valid(x):
    await asyncio.sleep(0.05)
    return x % 2 == 0

async def async_range(n):
    for i in range(n):
        await asyncio.sleep(0.05)
        yield i

async def main():
    # Async comprehension with async for and async filter
    evens = [x async for x in async_range(10) if await is_valid(x)]
    # For each x from 0..9, await is_valid(x); keep only even numbers.

    print(evens)

asyncio.run(main())
```

**Expected Output:**
```
[0, 2, 4, 6, 8]
```

**Breakdown:**
1. `async_range(10)` yields 0 through 9 asynchronously.
2. For each value, `await is_valid(x)` checks if it is even.
3. The comprehension collects only the even numbers into a list.
4. This pattern is useful for filtering streams where the filter itself requires asynchronous work (e.g., checking a remote validation service).

### Real-World Cases with Explanation

**Case 1: Fetching Multiple URLs Concurrently**

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ["https://example.com", "https://httpbin.org/get"]
    async with aiohttp.ClientSession() as session:
        # Sequential await in a comprehension
        pages = [await fetch_url(session, url) for url in urls]
        print(f"Fetched {len(pages)} pages")
```

**Why it matters:** While this example is sequential, the comprehension pattern extends naturally to concurrent fetching with `asyncio.gather()` when combined with async generators. The comprehension provides a clean syntax for transforming each response.

**Case 2: Processing Log Lines from an Async Stream**

```python
async def process_logs(stream):
    errors = [line async for line in stream if "ERROR" in line]
    return errors
```

**Why it matters:** Reading logs from a network stream or async file requires `async for`. The comprehension filters error lines into a list for later analysis, all without blocking the event loop.

---

## Core Concept 3: Asynchronous Generators

### Definitions

**Core Definition**
An asynchronous generator is a function defined with `async def` that contains one or more `yield` expressions, producing an asynchronous iterator.

**Technical Definition**
Introduced in PEP 525, an asynchronous generator function uses `async def` and `yield` together. When called, it returns an **asynchronous generator object** that implements the asynchronous iteration protocol (`__aiter__` and `__anext__`). Each `__anext__()` call awaits until the generator produces a value via `yield`. When the function returns or falls off the end, `StopAsyncIteration` is raised. It is a `SyntaxError` to have a non-empty `return` statement in an asynchronous generator. Asynchronous generators are approximately 2× faster than equivalent hand-written async iterator classes and require far less boilerplate.

**Beginner-Friendly Explanation**
A regular generator (using `def` and `yield`) lets you produce values one at a time without storing them all in memory. An async generator does the same, but each step can `await`—for example, waiting for the next line from a network connection. Instead of writing a class with `__aiter__` and `__anext__`, you just write an `async def` function with `yield`, and Python handles the protocol for you.

### Purposes

- **To create asynchronous iterators with minimal boilerplate** compared to manual protocol implementation
- **To produce values lazily** from asynchronous data sources (streams, queues, APIs)
- **To enable `async for` consumption** of custom data pipelines
- **To support `async with` cleanup** through `aclose()` and `athrow()`
- **To model producer–consumer patterns** where values become available over time

### Syntax Rules and Structure

**Complete General Syntax**

```
async def async_generator(args):
    <setup>
    while <condition>:
        <await some_async_operation>
        yield value
    <cleanup>
```

**Breakdown:**
- `async def`: Declares an asynchronous generator function.
- `yield value`: Suspends the coroutine and produces a value to the consumer.
- When the function returns (implicitly or via empty `return`), `StopAsyncIteration` is raised.
- The returned object supports `__aiter__()` (returns self) and `__anext__()` (awaits until the next `yield`).

**Syntax Rules**

| Rule | Description |
|---|---|
| `async def` + `yield` | Both are required; either alone defines a coroutine or sync generator, not an async generator |
| No `return value` | Only empty `return` is allowed; `return value` is a `SyntaxError` |
| Supports `await` | Can await inside the generator body before yielding |
| `async for` consumption | The primary way to consume values |
| `aclose()` method | Manually close the generator; triggers `GeneratorExit` inside |
| `athrow()` method | Throw an exception into the generator at the suspension point |

**Constraints and Limitations**

- **Event loop required**: Async generators only work inside an active `asyncio` event loop.
- **No `yield from`**: Asynchronous generators do not support `yield from` (the async equivalent is `async for` inside the generator).
- **Finalization complexity**: Closing async generators involves scheduling a coroutine, which can be tricky if the event loop is already closed. Python's `asyncio` handles this in most cases.
- **`StopAsyncIteration` semantics**: Raising `StopIteration` inside an async generator is transformed into a `RuntimeError` to prevent accidental coroutine termination.
- **Version**: Added in Python 3.6 (PEP 525).

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Async Generator**

```python
import asyncio

async def ticker(delay, to):
    """Yield numbers from 0 to 'to' every 'delay' seconds."""
    for i in range(to):
        yield i                       # Produce the value
        await asyncio.sleep(delay)    # Wait before next iteration

async def main():
    # Consume the async generator with async for
    async for number in ticker(0.1, 3):
        print(number)

asyncio.run(main())
```

**Expected Output:**
```
0
1
2
```

**Breakdown:**
1. `ticker` is an async generator because it uses both `async def` and `yield`.
2. Each iteration yields `i`, then awaits `asyncio.sleep(0.1)`.
3. The `async for` loop awaits each value as it becomes available.
4. This is significantly simpler than writing a class with `__aiter__` and `__anext__` (compare with Example 1 in Core Concept 1).

**Example 2: Async Generator with `aclose()`**

```python
import asyncio

async def resource_stream():
    """Async generator that acquires and releases a resource."""
    print("Resource acquired")
    try:
        for i in range(10):
            yield i
            await asyncio.sleep(0.1)
    finally:
        print("Resource released")  # Cleanup runs when generator is closed

async def main():
    stream = resource_stream()

    # Consume only the first 3 values
    count = 0
    async for value in stream:
        print(f"Value: {value}")
        count += 1
        if count >= 3:
            break

    # Explicitly close the generator to trigger cleanup
    await stream.aclose()

asyncio.run(main())
```

**Expected Output:**
```
Resource acquired
Value: 0
Value: 1
Value: 2
Resource released
```

**Breakdown:**
1. The `try`/`finally` block ensures cleanup code runs when the generator is closed.
2. The `async for` loop breaks after 3 values, leaving the generator suspended.
3. `await stream.aclose()` closes the generator, triggering the `finally` block.
4. This pattern is essential for releasing resources (file handles, network connections) deterministically.

**Example 3: Async Generator as a Data Pipeline**

```python
import asyncio

async def source(n):
    """Produce raw numbers."""
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async def transform(agen):
    """Transform: square each value."""
    async for value in agen:
        yield value * value

async def filter_even(agen):
    """Filter: keep only even values."""
    async for value in agen:
        if value % 2 == 0:
            yield value

async def main():
    # Compose the pipeline: source -> transform -> filter
    pipeline = filter_even(transform(source(6)))

    async for result in pipeline:
        print(result)

asyncio.run(main())
```

**Expected Output:**
```
0
4
16
```

**Breakdown:**
1. `source(6)` yields 0, 1, 2, 3, 4, 5.
2. `transform` squares each value: 0, 1, 4, 9, 16, 25.
3. `filter_even` keeps only even squares: 0, 4, 16.
4. Each stage is an async generator, composed into a pipeline. Values flow lazily through the pipeline as they become available.
5. This mirrors the synchronous generator pipeline pattern but works with asynchronous data sources.

### Real-World Cases with Explanation

**Case 1: Reading Lines from a Network Stream**

```python
async def read_lines(reader):
    """Async generator yielding lines from an asyncio StreamReader."""
    while True:
        line = await reader.readline()
        if not line:
            break
        yield line.decode().rstrip()

# Usage
async def main():
    reader, _ = await asyncio.open_connection('example.com', 80)
    async for line in read_lines(reader):
        print(line)
```

**Why it matters:** Network streams deliver data incrementally. An async generator naturally models "keep reading until the stream ends," yielding each line as it arrives without blocking.

**Case 2: Consuming an Async Queue**

```python
async def consume_queue(queue):
    """Async generator that yields items from a queue until sentinel."""
    while True:
        item = await queue.get()
        if item is None:
            break
        yield item

# Producer
async def producer(queue):
    for i in range(5):
        await queue.put(i)
        await asyncio.sleep(0.1)
    await queue.put(None)  # Sentinel
```

**Why it matters:** Async queues are a core `asyncio` primitive for producer–consumer patterns. Wrapping a queue in an async generator provides a clean interface for consuming items as they arrive.

**Case 3: Paginated API Fetching**

```python
async def fetch_all_pages(session, base_url):
    """Async generator yielding items from all pages of a REST API."""
    page = 1
    while True:
        async with session.get(f"{base_url}?page={page}") as resp:
            data = await resp.json()
        if not data["items"]:
            break
        for item in data["items"]:
            yield item
        page += 1

# Consume all items across all pages
async def main():
    async with aiohttp.ClientSession() as session:
        async for item in fetch_all_pages(session, "https://api.example.com/items"):
            process(item)
```

**Why it matters:** Pagination logic is encapsulated in the async generator. The consumer simply writes `async for item in fetch_all_pages(...)` and doesn't need to manage page numbers or termination conditions.

---

## References

- Python Software Foundation. *Built-in Functions — aiter() and anext()*. https://docs.python.org/3/library/functions.html#aiter
- Python Software Foundation. *Data Model — Asynchronous Generator Functions*. https://docs.python.org/3/reference/datamodel.html#asynchronous-generator-functions
- Yury Selivanov. *PEP 525 – Asynchronous Generators*. https://peps.python.org/pep-0525/
- Yury Selivanov. *PEP 530 – Asynchronous Comprehensions*. https://peps.python.org/pep-0530/
- Yury Selivanov. *PEP 492 – Coroutines with async and await syntax*. https://peps.python.org/pep-0492/
- Real Python. *Asynchronous Iterators and Iterables in Python*. https://realpython.com/python-async-iterators/
- Python Software Foundation. *collections.abc — AsyncIterator and AsyncIterable*. https://docs.python.org/3/library/collections.abc.html#collections.abc.AsyncIterator
- Python Software Foundation. *asyncio — Asynchronous I/O*. https://docs.python.org/3/library/asyncio.html