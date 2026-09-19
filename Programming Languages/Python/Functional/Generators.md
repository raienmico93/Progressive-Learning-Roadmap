# Python Generators: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
A generator is a special kind of function that produces a sequence of values lazily, yielding one value at a time and suspending its state between yields.

**Technical Definition**
A generator function is a function that contains the `yield` expression. When called, it returns an iterator known as a generator. The generator controls the execution of the generator function: execution starts when one of the generator's methods is called, proceeds to the first `yield` expression, where it is suspended, returning the value of the `yield` expression to the caller. By suspended, all local state is retained, including the current bindings of local variables, the instruction pointer, the internal evaluation stack, and the state of any exception handling. When resumed, the function can proceed exactly as if the `yield` expression were just another external call. Generator functions are quite similar to coroutines: they yield multiple times, have more than one entry point, and their execution can be suspended.

**Beginner-Friendly Explanation**
A generator is like a function that can pause and resume. Instead of computing all its values at once (like a list), it produces one value, then waits until you ask for the next. This makes generators perfect for working with large datasets or infinite sequences — you only compute what you need, when you need it. Think of it like reading a book one page at a time instead of memorising the entire book before telling anyone what's on page 1.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Lazy Evaluation** | Values are produced on demand, not all at once |
| **Stateful** | Local variables and execution position are preserved between yields |
| **Memory Efficient** | Only one item in memory at a time (O(1) memory) |
| **Single-Pass** | Generators are exhausted after one full traversal |
| **Coroutine-Like** | Generators can send, receive, and throw values (via advanced methods) |
| **Iterator Protocol** | Generators implement `__iter__()` and `__next__()` |

### Prerequisites

- **Basic Python syntax**: variables, functions, and loops
- **Understanding of iterators**: how `iter()` and `next()` work
- **Familiarity with `for` loops**: the primary way to consume generators
- **Basic concepts of memory**: why lazy evaluation matters

### Related Programming Areas

- **Iterators**: Generators are a convenient way to create iterators
- **Coroutines**: Advanced generator methods enable coroutine-style programming
- **Asynchronous Programming**: `async` generators (`async def` with `yield`) for async iteration
- **`itertools` Module**: Tools for efficient iterator-based operations
- **Functional Programming**: Lazy pipelines and data streams

### Core Concepts / Features

1. **`yield`** (The keyword that creates generators)
2. **Generator Functions** (Functions defined with `yield`)
3. **Generator Expressions** (Concise inline generators, memory comparison with list comprehensions)
4. **Generator State and Life Cycle** (Suspension, resumption, and exhaustion)
5. **Lazy Computation & Memory-Efficient Data Streaming**
6. **`yield from`** (Delegating to sub-generators)
7. **Advanced Generator Methods** (`.send()`, `.throw()`, `.close()`)


## Core Concept 1: `yield`

### Definitions

**Core Definition**
`yield` is an expression that suspends the execution of a generator function, returning a value to the caller and saving the function's state for later resumption.

**Technical Definition**
The `yield` expression is used when defining a generator function. When a generator function is called, it returns a generator (an iterator). Execution starts when one of the generator's methods is called. At that time, execution proceeds to the first `yield` expression, where it is suspended again, returning the value of `yield` to the generator's caller. If `yield` is used as an expression (e.g., `x = yield value`), the value sent to the generator via `.send()` becomes the result of the `yield` expression. If `__next__()` is used, the result is `None`.

**Beginner-Friendly Explanation**
`yield` is like a pause button for functions. When Python hits `yield`, it sends a value back to whoever called the function and freezes the function's state. The next time you ask for a value, the function wakes up exactly where it left off and continues until it hits another `yield` (or finishes). It's `return` that doesn't end the function — it just takes a break.

### Purposes

- **To create generator functions** that produce sequences lazily
- **To suspend and resume function execution** while preserving local state
- **To produce values on demand** without computing all of them upfront
- **To enable memory-efficient iteration** over large or infinite sequences
- **To implement coroutine-style programming** when combined with `.send()`

### Syntax Rules and Structure

**Complete General Syntax**

```python
def generator_function():
    # ... code ...
    yield value
    # ... code ...
    yield another_value
```

**Breakdown:**
- `def`: Standard function definition.
- `yield`: Suspends execution and returns a value.
- The function becomes a generator function when `yield` appears anywhere in its body.

**Syntax Rules**

| Rule | Description |
|---|---|
| `yield` in function body | Makes the function a generator function |
| `yield` as expression | Can be used as `x = yield value` to receive sent values |
| Multiple yields | A generator can yield many times |
| `yield` outside function | Not allowed (except in generator expressions) |
| `return` in generator | Ends the generator, raising `StopIteration` |

**Constraints and Limitations**

- **`yield` makes the function lazy**: Nothing runs until the generator is iterated.
- **Single-pass**: Generators cannot be reset; once exhausted, they're done.
- **No `return` value initially**: A bare `return` in a generator raises `StopIteration`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `yield`**

```python
# Step 1: Define a generator function
def simple_generator():
    print("Starting")
    yield 1
    print("Resuming")
    yield 2
    print("Resuming again")
    yield 3

# Step 2: Create the generator (no code runs yet)
gen = simple_generator()
print("Generator created")

# Step 3: Consume values one by one
print(next(gen))   # Runs up to first yield
print(next(gen))   # Resumes and runs to second yield
print(next(gen))   # Resumes and runs to third yield
```

**Expected Output:**
```
Generator created
Starting
1
Resuming
2
Resuming again
3
```

**Breakdown:** The generator function does not execute until `next()` is called. Each `next()` call runs the function until the next `yield`, then suspends it. The print statements between yields demonstrate resumption.

**Example 2: `yield` as an Expression**

```python
def echo_generator():
    while True:
        received = yield
        print(f"Received: {received}")

gen = echo_generator()
next(gen)              # Prime the generator (run to first yield)
gen.send("hello")      # Sends value into the generator
gen.send("world")      # Sends another value
```

**Expected Output:**
```
Received: hello
Received: world
```

**Breakdown:** `yield` without a value returns `None`. When `.send(value)` is called, the `yield` expression evaluates to that value, which is assigned to `received`.

**Example 3: Generator with a Return Statement**

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1
    return "Done!"

gen = countdown(3)
print(next(gen))   # 3
print(next(gen))   # 2
print(next(gen))   # 1
try:
    next(gen)
except StopIteration as e:
    print(f"Return value: {e.value}")
```

**Expected Output:**
```
3
2
1
Return value: Done!
```

**Breakdown:** A `return` statement in a generator ends iteration and raises `StopIteration`. The return value is available in the `StopIteration` exception's `value` attribute.

### Real-World Cases with Explanation

**Case 1: Reading Large Files Lazily**

```python
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()
```

**Why it matters:** Instead of loading the entire file into memory, the generator yields one line at a time, making it possible to process files larger than available RAM.

**Case 2: Generating Infinite Sequences**

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Take the first 10 Fibonacci numbers
gen = fibonacci()
for _ in range(10):
    print(next(gen))
```

**Why it matters:** Generators can represent infinite sequences because they produce values on demand, never needing to store the entire sequence.


## Core Concept 2: Generator Functions

### Definitions

**Core Definition**
A generator function is any function that contains a `yield` expression; when called, it returns a generator object.

**Technical Definition**
Generator functions are described as functions that contain the `yield` expression. When a generator function is called, it returns an iterator known as a generator. The generator then controls the execution of the generator function. Execution starts when one of the generator's methods is called. At that time, execution proceeds to the first `yield` expression, where it is suspended again, returning the value of `yield` to the generator's caller. Any function containing `yield` is a generator function, and calling it returns a generator object without executing the function body.

**Beginner-Friendly Explanation**
A generator function looks like a normal function, but it uses `yield` instead of (or in addition to) `return`. The key difference: calling a generator function doesn't run the code inside it. Instead, it returns a generator object. The code only runs when you start iterating over the generator. It's like a recipe that you can follow step by step, pausing after each step.

### Purposes

- **To create generators** conveniently without writing iterator classes
- **To encapsulate complex iteration logic** in a readable function
- **To produce values lazily** and save memory
- **To maintain state** across multiple yields
- **To support coroutine-style programming** with `.send()`

### Syntax Rules and Structure

**Complete General Syntax**

```python
def generator_name(parameters):
    # ... code ...
    yield value
    # ... more code ...
    yield another_value
```

**Breakdown:**
- Defined with `def`, but contains at least one `yield`.
- Calling the function returns a generator object.
- The function body does not run until iteration begins.

**Syntax Rules**

| Rule | Description |
|---|---|
| Any function with `yield` | Automatically becomes a generator function |
| Calling returns generator | The function body is not executed immediately |
| State preserved | Local variables retain values between yields |
| `return` ends generator | Raises `StopIteration` |

**Constraints and Limitations**

- **Cannot be restarted**: Once exhausted, a generator is done.
- **Single-pass**: Cannot iterate twice over the same generator.
- **No `len()`**: Generators do not support length queries.
- **Not reentrant**: Calling a generator method while it's already executing raises `ValueError`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Simple Generator Function**

```python
def squares(n):
    """Generate squares of numbers from 0 to n-1."""
    for i in range(n):
        yield i ** 2

# Use the generator
for square in squares(5):
    print(square, end=" ")
```

**Expected Output:**
```
0 1 4 9 16 
```

**Breakdown:** The `squares` function is a generator function. It yields each square one at a time. The `for` loop consumes the generator lazily.

**Example 2: Generator with State**

```python
def running_total():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

gen = running_total()
next(gen)           # Prime: run to first yield (total = 0)
print(gen.send(10)) # total = 10
print(gen.send(20)) # total = 30
print(gen.send(5))  # total = 35
```

**Expected Output:**
```
10
30
35
```

**Breakdown:** The local variable `total` persists across `send()` calls. Each call adds to the running total and yields the new value.

**Example 3: Generator with Cleanup**

```python
def managed_resource():
    print("Acquiring resource")
    try:
        yield "resource"
    finally:
        print("Releasing resource")

gen = managed_resource()
print(next(gen))     # Acquire and yield
gen.close()          # Trigger cleanup
```

**Expected Output:**
```
Acquiring resource
resource
Releasing resource
```

**Breakdown:** The `finally` block runs when `close()` is called, allowing cleanup code to execute even when the generator is terminated externally.

### Real-World Cases with Explanation

**Case 1: Database Query Streaming**

```python
def stream_users(cursor):
    while True:
        row = cursor.fetchone()
        if row is None:
            break
        yield row
```

**Why it matters:** Streaming database results one row at a time avoids loading the entire result set into memory.

**Case 2: Data Pipeline**

```python
def read_numbers(file):
    for line in file:
        yield int(line)

def filter_even(numbers):
    for n in numbers:
        if n % 2 == 0:
            yield n

# Pipeline: read → filter
for even in filter_even(read_numbers(open("numbers.txt"))):
    print(even)
```

**Why it matters:** Generators compose into lazy pipelines, processing data one item at a time with minimal memory.


## Core Concept 3: Generator Expressions

### Definitions

**Core Definition**
A generator expression is a concise, inline syntax for creating a generator, similar to a list comprehension but with parentheses instead of square brackets.

**Technical Definition**
Generator expressions are a high-performance, memory-efficient generalization of list comprehensions and generators. They use the syntax `(expression for item in iterable [if condition])`. Unlike list comprehensions, which build the entire list in memory, generator expressions return an iterator that produces values lazily. A list comprehension builds a whole list at one time; a generator expression builds a function that acts like a list but doesn't build the values until you ask for them. Generator expressions are ideal for large datasets where only a subset of values is needed or where memory is constrained.

**Beginner-Friendly Explanation**
A generator expression is like a list comprehension's lazy cousin. `[x**2 for x in range(1000000)]` creates a list with a million squares in memory. `(x**2 for x in range(1000000))` creates a generator that produces each square only when you ask for it. Same syntax, different brackets — but vastly different memory behaviour.

### Purposes

- **To create generators concisely** without a full function definition
- **To save memory** compared to list comprehensions
- **To process large datasets** without loading everything into memory
- **To enable lazy pipelines** with minimal syntax
- **To replace `map()` and `filter()`** with more readable inline expressions

### Syntax Rules and Structure

**Complete General Syntax**

```
(expression for item in iterable)
(expression for item in iterable if condition)
```

**Breakdown:**
- `expression`: The value to yield (can use `item`).
- `for item in iterable`: The iteration clause.
- `if condition`: Optional filter.

**Memory Comparison: Generator vs. List Comprehension**

| Aspect | Generator Expression | List Comprehension |
|---|---|---|
| Syntax | `(x for x in ...)` | `[x for x in ...]` |
| Memory | O(1) — one item at a time | O(n) — all items stored |
| Evaluation | Lazy (on demand) | Eager (immediate) |
| Reusable | No (single-pass) | Yes |
| Best for | Large/infinite data | Small data, repeated access |

**Constraints and Limitations**

- **Single-pass**: Cannot iterate twice.
- **No indexing**: Generators do not support indexing or slicing.
- **No `len()`**: Cannot query the length.
- **Debugging**: Harder to inspect intermediate state.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Generator Expression**

```python
import sys

# Step 1: List comprehension (eager)
list_comp = [x ** 2 for x in range(1000000)]
print(f"List size: {sys.getsizeof(list_comp)} bytes")

# Step 2: Generator expression (lazy)
gen_expr = (x ** 2 for x in range(1000000))
print(f"Generator size: {sys.getsizeof(gen_expr)} bytes")
```

**Expected Output:**
```
List size: 8448728 bytes
Generator size: 200 bytes
```

**Breakdown:** The list comprehension allocates ~8.4 MB for a million squares. The generator expression uses ~200 bytes regardless of the range size, because it stores only the computation metadata.

**Example 2: Using Generator Expressions in Functions**

```python
# Step 1: Sum of squares using a generator expression
total = sum(x ** 2 for x in range(1000000))
print(total)

# Step 2: Any/all with generators
numbers = [1, 2, 3, 4, 5]
has_even = any(x % 2 == 0 for x in numbers)
print(has_even)
```

**Expected Output:**
```
333332833333500000
True
```

**Breakdown:** Passing a generator expression to `sum()`, `any()`, or `all()` avoids creating an intermediate list. The generator is consumed directly by the built-in function.

**Example 3: Generator Expression with Filter**

```python
# Step 1: Even squares from a range
evens = (x ** 2 for x in range(10) if x % 2 == 0)

# Step 2: Consume lazily
for value in evens:
    print(value, end=" ")
```

**Expected Output:**
```
0 4 16 36 64 
```

**Breakdown:** The `if` clause filters items before the expression is applied. Only even squares are produced.

### Real-World Cases with Explanation

**Case 1: Processing Large Log Files**

```python
with open("huge.log") as f:
    error_count = sum(1 for line in f if "ERROR" in line)
```

**Why it matters:** The generator expression counts errors without loading the entire log file into memory.

**Case 2: Streaming Data Pipelines**

```python
results = (transform(row) for row in data if is_valid(row))
for result in results:
    save(result)
```

**Why it matters:** Generator expressions create lazy pipelines that process data one item at a time, ideal for ETL workflows.


## Core Concept 4: Generator State and Life Cycle

### Definitions

**Core Definition**
A generator has a life cycle: creation, suspension at yield points, resumption, and eventual exhaustion; its state (local variables, instruction pointer) is preserved across suspensions.

**Technical Definition**
When a generator function is called, it returns a generator object without executing the function body. Execution starts when one of the generator's methods is called (`__next__()`, `send()`, `throw()`, or `close()`). At that time, execution proceeds to the first `yield` expression, where it is suspended again, returning the value of `yield` to the generator's caller. By suspended, all local state is retained, including the current bindings of local variables, the instruction pointer, the internal evaluation stack, and the state of any exception handling. When the execution is resumed, the function can proceed exactly as if the `yield` expression were just another external call. The generator is finalized when it is garbage-collected or when `close()` is called, allowing pending `finally` clauses to execute.

**Beginner-Friendly Explanation**
A generator has a simple life: it's created (but doesn't run yet), runs until the first `yield` (then pauses), resumes when you ask for the next value, and eventually finishes (exhaustion). Between pauses, it remembers everything — where it was, what its variables were, and what it was doing. It's like a bookmark in a book: you close the book, and when you open it again, you're right where you left off.

### Purposes

- **To understand how generators pause and resume** without losing state
- **To debug generator behaviour** by knowing when code executes
- **To write generators with proper cleanup** using `try`/`finally`
- **To avoid common pitfalls** like using exhausted generators
- **To implement coroutines** that maintain state across sends

### Syntax Rules and Structure

**Life Cycle Stages**

| Stage | Trigger | What Happens |
|---|---|---|
| **Created** | `gen = func()` | Generator object created; body not executed |
| **Started** | `next(gen)` | Execution begins; runs to first `yield` |
| **Suspended** | `yield` | State saved; control returns to caller |
| **Resumed** | `next(gen)` or `gen.send(v)` | Execution continues from after `yield` |
| **Exhausted** | Function ends or `return` | `StopIteration` raised |
| **Finalized** | GC or `close()` | `finally` blocks execute |

**State Preserved**

| Component | Description |
|---|---|
| Local variables | Current values retained |
| Instruction pointer | Position in bytecode |
| Evaluation stack | Intermediate values |
| Exception handling state | Active `try` blocks |

**Constraints and Limitations**

- **Single-pass**: Once exhausted, a generator cannot be restarted.
- **No reentrancy**: Calling a generator method while it's already executing raises `ValueError`.
- **Garbage collection**: Unreferenced generators are finalized automatically.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Life Cycle Demonstration**

```python
def lifecycle():
    print("Stage 1: Start")
    yield "first"
    print("Stage 2: Resumed")
    yield "second"
    print("Stage 3: Resumed again")
    # Generator ends here

gen = lifecycle()
print("Created (no output from function)")

print(next(gen))   # Runs to first yield
print("Between calls")
print(next(gen))   # Resumes
try:
    next(gen)      # Exhausts
except StopIteration:
    print("Exhausted")
```

**Expected Output:**
```
Created (no output from function)
Stage 1: Start
first
Between calls
Stage 2: Resumed
second
Stage 3: Resumed again
Exhausted
```

**Breakdown:** The generator body doesn't run until the first `next()`. Each `next()` resumes from the previous `yield`, preserving state.

**Example 2: State Preservation**

```python
def counter():
    count = 0
    while True:
        count += 1
        yield count

gen = counter()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# 'count' persists across calls
```

**Expected Output:**
```
1
2
3
```

**Breakdown:** The local variable `count` is preserved between yields. Each resumption continues from where it left off.

**Example 3: Finalization and Cleanup**

```python
def resource_manager():
    print("Open resource")
    try:
        yield "resource"
    finally:
        print("Close resource")

# Case 1: Normal exhaustion
gen = resource_manager()
next(gen)
gen.close()  # Triggers finally

# Case 2: Garbage collection
gen = resource_manager()
next(gen)
del gen  # GC triggers close()
```

**Expected Output:**
```
Open resource
Close resource
Open resource
Close resource
```

**Breakdown:** The `finally` block runs when the generator is closed or garbage-collected, ensuring cleanup.

### Real-World Cases with Explanation

**Case 1: Database Transactions**

```python
def transaction(cursor):
    cursor.execute("BEGIN")
    try:
        yield cursor
    finally:
        cursor.execute("COMMIT")
```

**Why it matters:** The `finally` block ensures the transaction is committed (or rolled back) even if the generator is closed early.

**Case 2: File Handling**

```python
def read_file(path):
    f = open(path)
    try:
        for line in f:
            yield line
    finally:
        f.close()
```

**Why it matters:** Generators with `finally` blocks guarantee resource cleanup, preventing file descriptor leaks.


## Core Concept 5: Lazy Computation & Memory-Efficient Data Streaming

### Definitions

**Core Definition**
Lazy computation means values are produced only when requested; memory-efficient data streaming processes data one item at a time without loading everything into memory.

**Technical Definition**
Lazy evaluation is a strategy where certain objects are not produced until they are needed. This can have important performance implications for both memory management and function run time. Generators implement lazy evaluation: they produce values on demand rather than precomputing all values. This makes them memory-efficient for large or infinite data streams. A generator expression streams one item at a time. Generators turn an N-stage pipeline from "all stages resident at once" into "one row resident at a time" — the same data, the same logic, a fraction of the memory. Generators are ideal for processing large datasets that don't fit in memory, streaming data from files, network, or sensors.

**Beginner-Friendly Explanation**
Lazy computation is like a water tap: water flows only when you open it. You don't store all the water upfront. Generators are lazy — they produce each value only when asked. This means you can process a 10 GB file on a laptop with 8 GB of RAM, because you never load the whole file at once. You just process it line by line, one at a time.

### Purposes

- **To process large datasets** that don't fit in memory
- **To stream data** from files, networks, or sensors
- **To implement pipelines** where data flows through stages lazily
- **To improve startup time** by deferring computation
- **To handle infinite sequences** that cannot be stored

### Syntax Rules and Structure

**Lazy vs. Eager Comparison**

| Aspect | Lazy (Generator) | Eager (List) |
|---|---|---|
| Memory | O(1) — one item at a time | O(n) — all items stored |
| Evaluation | On demand | Immediate |
| Startup | Fast (no computation) | Slow (all items computed) |
| Infinite | Possible | Not possible |
| Reusable | No | Yes |

**Constraints and Limitations**

- **Single-pass**: Generators cannot be reused.
- **No random access**: Cannot index or slice.
- **Debugging**: Harder to inspect intermediate state.
- **Performance**: Generator expressions have slightly more overhead than list comprehensions for small data.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Memory Comparison (Large Data)**

```python
import sys

# Step 1: List comprehension (eager)
list_data = [x for x in range(1000000)]
print(f"List memory: {sys.getsizeof(list_data)} bytes")

# Step 2: Generator expression (lazy)
gen_data = (x for x in range(1000000))
print(f"Generator memory: {sys.getsizeof(gen_data)} bytes")
```

**Expected Output:**
```
List memory: 8448728 bytes
Generator memory: 200 bytes
```

**Breakdown:** The list stores all 1,000,000 integers. The generator stores only the computation metadata, using minimal memory regardless of range size.

**Example 2: Streaming File Processing**

```python
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()

# Process a huge file without loading it all
line_count = 0
for line in read_large_file("huge_file.txt"):
    line_count += 1
    if line_count % 1000000 == 0:
        print(f"Processed {line_count} lines")
```

**Expected Output:** (Depends on file size; prints progress every million lines)

**Breakdown:** The generator yields one line at a time. The file is never fully loaded into memory.

**Example 3: Lazy Pipeline**

```python
def numbers():
    for i in range(100):
        yield i

def squares(nums):
    for n in nums:
        yield n ** 2

def evens(nums):
    for n in nums:
        if n % 2 == 0:
            yield n

# Pipeline: numbers → squares → evens
result = evens(squares(numbers()))
for value in result:
    print(value, end=" ")
```

**Expected Output:**
```
0 4 16 36 64 100 144 196 256 324 400 484 576 676 784 900 1024 1156 1296 1444 1600 1764 1936 2116 2304 2500 2704 2916 3136 3364 3600 3844 4096 4356 4624 4900 5184 5476 5776 6084 6400 6724 7056 7396 7744 8100 8464 8836 9216 9604 9999 
```

**Breakdown:** Each stage processes one item at a time. The entire pipeline uses minimal memory, regardless of the input size.

### Real-World Cases with Explanation

**Case 1: Log File Analysis**

```python
def parse_logs(path):
    with open(path) as f:
        for line in f:
            if "ERROR" in line:
                yield line.strip()

for error in parse_logs("app.log"):
    print(error)
```

**Why it matters:** Only error lines are yielded, and only one line is in memory at a time.

**Case 2: Sensor Data Streaming**

```python
def read_sensor():
    while True:
        yield sensor.read()

# Process sensor data lazily
for reading in read_sensor():
    if reading > THRESHOLD:
        alert(reading)
```

**Why it matters:** The generator produces readings on demand, enabling real-time processing of infinite data streams.


## Core Concept 6: `yield from` (Delegating to Sub-generators)

### Definitions

**Core Definition**
`yield from` is a syntax that delegates part of a generator's operations to another iterable (typically a sub-generator), passing values directly through.

**Technical Definition**
A syntax is proposed for a generator to delegate part of its operations to another generator. This allows a section of code containing `yield` to be factored out and placed in another generator. Additionally, the subgenerator is allowed to return with a value, and the value is made available to the delegating generator. The syntax is `yield from <expr>`, where `<expr>` is an expression evaluating to an iterable, from which an iterator is extracted. The iterator is run to exhaustion, during which time it yields and receives values directly to or from the caller of the delegating generator. When the iterator is another generator, the subgenerator is allowed to execute a `return` statement with a value, and that value becomes the value of the `yield from` expression.

**Beginner-Friendly Explanation**
`yield from` is like saying "I'll let this other generator do the work for a while." Instead of writing a loop that manually yields each value from a sub-generator, you just write `yield from sub_generator`. It's a shortcut that also handles sending values and exceptions correctly. It's like delegating a phone call: instead of repeating what the other person says, you just forward the call directly.

### Purposes

- **To delegate iteration** to another generator or iterable
- **To refactor generator code** by moving part of it to a sub-generator
- **To chain generators** without manual loops
- **To capture return values** from sub-generators
- **To simplify nested generator logic**

### Syntax Rules and Structure

**Complete General Syntax**

```
yield from iterable
result = yield from sub_generator
```

**Breakdown:**
- `yield from iterable`: Delegates iteration to the iterable.
- The value of the `yield from` expression is the sub-generator's return value.

**Syntax Rules**

| Rule | Description |
|---|---|
| Delegates to iterable | Works with any iterable |
| Return value | Captures the sub-generator's `return` value |
| Passes send/throw | Forwards `.send()` and `.throw()` to the sub-generator |
| Simplifies loops | Replaces `for x in sub: yield x` |

**Constraints and Limitations**

- **Python 3.3+**: `yield from` was introduced in Python 3.3.
- **Sub-generator must be iterable**: Not all objects support delegation.
- **Semantics are subtle**: Exception handling and send/throw forwarding have complex rules.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `yield from`**

```python
def sub_generator():
    yield 1
    yield 2
    yield 3

def main_generator():
    yield "Start"
    yield from sub_generator()
    yield "End"

for value in main_generator():
    print(value)
```

**Expected Output:**
```
Start
1
2
3
End
```

**Breakdown:** `yield from sub_generator()` delegates iteration to the sub-generator, yielding all its values in order. Without `yield from`, you'd need a `for` loop.

**Example 2: Capturing Return Value**

```python
def sub_generator():
    yield 1
    yield 2
    return "Sub done"

def main_generator():
    result = yield from sub_generator()
    print(f"Sub-generator returned: {result}")

gen = main_generator()
next(gen)  # 1
next(gen)  # 2
try:
    next(gen)
except StopIteration:
    pass
```

**Expected Output:**
```
Sub-generator returned: Sub done
```

**Breakdown:** The `return` value of the sub-generator is captured by the `yield from` expression and assigned to `result`.

**Example 3: Recursive Generator with `yield from`**

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

data = [1, [2, [3, 4], 5], 6]
print(list(flatten(data)))
```

**Expected Output:**
```
[1, 2, 3, 4, 5, 6]
```

**Breakdown:** `yield from flatten(item)` recursively delegates to sub-generators, flattening nested lists elegantly.

### Real-World Cases with Explanation

**Case 1: Tree Traversal**

```python
def traverse(node):
    yield node.value
    for child in node.children:
        yield from traverse(child)
```

**Why it matters:** `yield from` simplifies recursive tree traversal by delegating to child generators.

**Case 2: Pipeline Composition**

```python
def pipeline(*generators):
    for gen in generators:
        yield from gen
```

**Why it matters:** `yield from` enables composing multiple generators into a single pipeline.


## Core Concept 7: Advanced Generator Methods (`.send()`, `.throw()`, `.close()`)

### Definitions

**Core Definition**
Advanced generator methods—`.send()`, `.throw()`, and `.close()`—allow bidirectional communication with generators, turning them into coroutines.

**Technical Definition**
In addition to `next()`, generators have three methods: `send(value)` resumes execution and sends a value into the generator (the value becomes the result of the current `yield` expression); `throw(type, value, traceback)` raises an exception at the point where the generator was paused; `close()` raises a `GeneratorExit` exception inside the generator to terminate iteration. These methods, introduced in PEP 342, turn generators from one-way producers of information into both producers and consumers. Generators also become coroutines, a more generalized form of subroutines: subroutines are entered at one point and exited at another point, but coroutines can be entered, exited, and resumed at many different points.

**Beginner-Friendly Explanation**
These methods let you talk back to generators. `.send()` lets you send a value into the generator (like answering a question it asked). `.throw()` lets you raise an exception inside it (like interrupting it). `.close()` tells it to stop. With these, generators become full-fledged coroutines — they can produce values, receive values, and handle exceptions, all while maintaining their state.

### Purposes

- **To send values into a generator** (`.send()`)
- **To raise exceptions inside a generator** (`.throw()`)
- **To terminate a generator and run cleanup** (`.close()`)
- **To implement coroutine-style programming**
- **To build stateful pipelines** that react to external input

### Syntax Rules and Structure

**Complete General Syntax**

```
gen.send(value)               # Resume and send a value
gen.throw(type, value, tb)    # Raise an exception inside the generator
gen.close()                   # Terminate the generator
```

**Breakdown:**
- `send(value)`: The value becomes the result of the current `yield` expression.
- `throw(...)`: Raises an exception at the suspension point.
- `close()`: Raises `GeneratorExit` inside the generator.

**Method Behaviour**

| Method | Effect | Returns |
|---|---|---|
| `send(value)` | Resume with value | Next yielded value |
| `throw(type, value, tb)` | Raise exception inside | Next yielded value |
| `close()` | Terminate generator | `None` |

**Constraints and Limitations**

- **`send(None)` to start**: The first `send()` must be `None` (or use `next()`).
- **`GeneratorExit` handling**: Must catch `GeneratorExit` or `StopIteration`; returning a value raises `RuntimeError`.
- **No reentrancy**: Calling a method while the generator is executing raises `ValueError`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: `.send()` for Coroutine-Style Communication**

```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)            # Prime: run to first yield (total = 0)
print(acc.send(10))  # total = 10
print(acc.send(20))  # total = 30
print(acc.send(5))   # total = 35
```

**Expected Output:**
```
10
30
35
```

**Breakdown:** `send()` provides a value that becomes the result of the `yield` expression. The accumulator maintains state across sends.

**Example 2: `.throw()` for Exception Handling**

```python
def resilient_generator():
    try:
        while True:
            yield "waiting"
    except ValueError as e:
        yield f"Caught: {e}"

gen = resilient_generator()
print(next(gen))                    # waiting
print(gen.throw(ValueError, "bad")) # Caught: bad
```

**Expected Output:**
```
waiting
Caught: bad
```

**Breakdown:** `throw()` raises an exception at the `yield` point. The generator catches it and yields a response.

**Example 3: `.close()` for Cleanup**

```python
def resource_generator():
    print("Acquiring")
    try:
        yield "resource"
    except GeneratorExit:
        print("Cleaning up")
        raise  # Re-raise to confirm cleanup

gen = resource_generator()
print(next(gen))   # Acquire and yield
gen.close()        # Trigger cleanup
```

**Expected Output:**
```
Acquiring
resource
Cleaning up
```

**Breakdown:** `close()` raises `GeneratorExit` inside the generator. The `except` block handles cleanup and re-raises to confirm termination.

### Real-World Cases with Explanation

**Case 1: Coroutine-Based State Machine**

```python
def state_machine():
    state = "idle"
    while True:
        command = yield state
        if command == "start":
            state = "running"
        elif command == "stop":
            state = "idle"

sm = state_machine()
next(sm)              # idle
print(sm.send("start"))  # running
print(sm.send("stop"))   # idle
```

**Why it matters:** Generators with `send()` implement simple state machines that react to external commands.

**Case 2: Resource Management with Cleanup**

```python
def database_connection():
    conn = connect()
    try:
        yield conn
    finally:
        conn.close()
```

**Why it matters:** The `finally` block ensures the connection is closed even if the generator is terminated early via `close()`.


## References

- Python Software Foundation. *6. Expressions — Generator expressions*. https://docs.python.org/3/reference/expressions.html#generator-expressions
- Python Software Foundation. *6. Expressions — Generator-iterator methods*. https://docs.python.org/3/reference/expressions.html#generator-iterator-methods
- Python Software Foundation. *PEP 255 – Simple Generators*. https://peps.python.org/pep-0255/
- Python Software Foundation. *PEP 342 – Coroutines via Enhanced Generators*. https://peps.python.org/pep-0342/
- Python Software Foundation. *PEP 380 – Syntax for Delegating to a Subgenerator*. https://peps.python.org/pep-0380/
- Python Software Foundation. *PEP 525 – Asynchronous Generators*. https://peps.python.org/pep-0525/
- Real Python. *How to Use Generators and yield in Python*. https://realpython.com/introduction-to-python-generators/
- Real Python. *Python Generators 101*. https://realpython.com/lessons/python-generators-101/
- Python Mailing List. *lazy? vs not lazy? and yielding*. https://mail.python.org/pipermail/tutor/2010-March/074808.html
- Python Mailing List. *Generator expressions v/s list comprehensions*. https://mail.python.org/pipermail/tutor/2008-May/062779.html
- Python Mailing List. *PEP 255: Simple Generators*. https://mail.python.org/pipermail/python-dev/2001-June/015330.html
- Python Software Foundation. *What's New In Python 2.5 — Generators*. https://docs.python.org/3/whatsnew/2.5.html