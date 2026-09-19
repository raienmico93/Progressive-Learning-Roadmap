# Python Loops: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Python loops are control-flow constructs that repeatedly execute a block of code, either a fixed number of times or until a specified condition changes.

**Technical Definition**
Python provides two primary loop statements: `for` and `while`. The `for` statement implements iteration over the items of any sequence (list, tuple, string, dictionary, set, generator, or any iterable object), assigning each item to a target variable and executing a suite for each. The `while` statement repeatedly tests an expression and executes its suite as long as the expression remains true. Both statements support optional `else` clauses and may be controlled with `break`, `continue`, and `pass` statements.

**Beginner-Friendly Explanation**
Loops are how you tell a program to do something over and over without writing the same code many times. A `for` loop says "for each item in this collection, do this." A `while` loop says "as long as this condition is true, keep doing this." Together, they let programs process lists, count things, wait for events, and handle repeated tasks efficiently.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Iteration-Based (`for`)** | Iterates directly over items of any iterable, in sequence order |
| **Condition-Based (`while`)** | Repeats as long as a condition remains true |
| **Indentation-Based Blocks** | Loop bodies are defined by indentation (PEP 8: 4 spaces) |
| **`else` Clause** | Both loops support an `else` block that runs if no `break` occurs |
| **Control Statements** | `break` exits the loop; `continue` skips to the next iteration; `pass` is a no-op |
| **Iterator Protocol** | The `for` loop relies on Python's iterator protocol (`__iter__`, `__next__`) |

### Prerequisites

- **Basic Python syntax**: variables, expressions, and statements
- **Conditional statements**: `if`/`elif`/`else` (for `while` conditions)
- **Data structures**: lists, tuples, dictionaries, sets, and strings
- **Indentation awareness**: Python uses whitespace to delimit code blocks

### Related Programming Areas

- **Iteration Protocol**: Understanding iterables and iterators (`__iter__`, `__next__`, `StopIteration`)
- **Comprehensions**: List, dict, and set comprehensions are loop-like constructs
- **Generators**: Functions that `yield` values lazily, often used in loops
- **Asynchronous Programming**: `async for` and event loops for concurrent code
- **Functional Programming**: `map()`, `filter()`, and `itertools` for functional iteration

### Core Concepts / Features

1. **`for` Loop** (Iterating over collections and iterables)
2. **`while` Loop** (Condition-based repetition)
3. **Nested Loops** (Loops inside loops)
4. **Loop Conditions** (How conditions control `while` and `for` loops)
5. **Iteration over Collections** (Iterables and iterators)
6. **Iteration over Ranges** (`range()` function)
7. **Infinite Loops and Event Loops** (Intentional infinite repetition and `asyncio` event loops)

---

## Core Concept 1: The `for` Loop

### Definitions

**Core Definition**
The `for` loop iterates over the items of any sequence or iterable object, executing a block of code once for each item.

**Technical Definition**
The `for` statement in Python differs from C-style loops: rather than iterating over an arithmetic progression, it iterates over the items of any sequence (a list or a string), in the order they appear. Its grammar is `for_stmt: "for" target_list "in" starred_expression_list ":" suite ["else" ":" suite]`. The loop repeatedly calls `iter()` on the right-hand expression, then `next()` until `StopIteration` is raised.

**Beginner-Friendly Explanation**
A `for` loop is like going through a stack of cards one by one. You say "for each card in this deck, do something with it." Python handles getting the next card automatically until there are no more cards left.

### Purposes

- **To iterate over collections** (lists, tuples, dictionaries, sets, strings) and process each element
- **To repeat an action a known number of times** using `range()`
- **To unpack values** from sequences of tuples (e.g., `for key, value in dict.items()`)
- **To process files** line by line or character by character
- **To implement comprehensions** and generator expressions
- **To iterate over any iterable** including generators, `itertools` outputs, and custom objects

### Syntax Rules and Structure

**Complete General Syntax**

```
for target in iterable:
    # suite (loop body)
    statement1
    statement2
else:
    # optional else suite (runs if no break)
```

**Breakdown:**
- `for`: The keyword that begins the loop.
- `target`: A variable (or tuple of variables) that receives each item.
- `in`: Separator keyword.
- `iterable`: Any object that can return an iterator.
- `:`: Terminates the header.
- Indented suite: Statements executed for each item.
- `else`: Optional clause executed after normal exhaustion.

**Syntax Rules**

| Rule | Description |
|---|---|
| Target can be tuple | `for key, value in items:` unpacks each element |
| Any iterable | Lists, tuples, strings, dicts, sets, files, generators, ranges |
| Colon required | Header ends with `:` |
| Indentation required | Suite must be indented consistently |
| `else` optional | Runs only if the loop completes without `break` |

**Constraints and Limitations**

- **Cannot modify collection during iteration**: Modifying a list while iterating over it can skip items; iterate over a copy instead.
- **No C-style loop counter**: `for i in range(10)` is idiomatic; `for (i=0; i<10; i++)` is not valid.
- **`else` is not "if empty"**: The `else` clause runs after normal completion, not when the iterable is empty.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Iterating over a List**

```python
# Step 1: Define a list of words
words = ['cat', 'window', 'defenestrate']

# Step 2: Iterate over each word
for w in words:
    # 'w' takes each value in turn: 'cat', then 'window', then 'defenestrate'
    print(w, len(w))

# Step 3: This runs after the loop finishes
print("Done iterating.")
```

**Expected Output:**
```
cat 3
window 6
defenestrate 12
Done iterating.
```

**Breakdown:** The loop assigns each element to `w` and executes the body. After the last element, the loop terminates normally and execution continues after the loop.

**Example 2: Iterating over a Dictionary**

```python
# Step 1: Define a dictionary
users = {'Hans': 'active', 'Éléonore': 'inactive', '景太郎': 'active'}

# Step 2: Iterate over key-value pairs
for user, status in users.items():
    print(f"{user}: {status}")
```

**Expected Output:**
```
Hans: active
Éléonore: inactive
景太郎: active
```

**Breakdown:** `users.items()` returns an iterable of `(key, value)` tuples. The `for` loop unpacks each tuple into `user` and `status`.

**Example 3: `for` Loop with `else` Clause**

```python
# Step 1: Define a list and a search target
numbers = [1, 2, 3, 4, 5]
target = 6

# Step 2: Search for the target
for n in numbers:
    if n == target:
        print(f"Found {target}!")
        break
else:
    # This runs because the loop completed without hitting 'break'
    print(f"{target} not found.")
```

**Expected Output:**
```
6 not found.
```

**Breakdown:** The `else` clause runs only if the loop finishes normally (no `break`). Since `6` is not in the list, the `else` block executes.

### Real-World Cases with Explanation

**Case 1: Processing Database Query Results**

```python
results = cursor.fetchall()  # Returns a list of tuples
for row in results:
    user_id, name, email = row  # Unpack each row
    send_welcome_email(email, name)
```

**Why it matters:** Database libraries return results as iterables of tuples. The `for` loop with tuple unpacking cleanly processes each record.

**Case 2: File Line Processing**

```python
with open('data.txt', 'r') as file:
    for line in file:  # Files are iterable line by line
        process(line.strip())
```

**Why it matters:** File objects are iterable, yielding one line at a time. This is memory-efficient for large files.

---

## Core Concept 2: The `while` Loop

### Definitions

**Core Definition**
The `while` loop repeatedly executes a block of code as long as a specified condition remains true.

**Technical Definition**
The `while` statement is used for repeated execution as long as an expression is true: it repeatedly tests the expression and, if true, executes the suite; if the expression becomes false, the loop terminates. Its grammar is `while_stmt: "while" assignment_expression ":" suite ["else" ":" suite]`.

**Beginner-Friendly Explanation**
A `while` loop is like a "keep doing this until..." instruction. As long as the condition is true, the code inside runs. You use it when you don't know in advance how many times you'll need to repeat something — for example, "while the user hasn't entered the correct password, ask again."

### Purposes

- **To repeat code an unknown number of times** until a condition changes
- **To implement interactive loops** (e.g., "while not quit, keep prompting")
- **To process data streams** where the end is not known in advance
- **To implement game loops** that run until a win/loss condition
- **To wait for events** in concurrent or asynchronous code

### Syntax Rules and Structure

**Complete General Syntax**

```
while condition:
    # suite (loop body)
    statement1
    statement2
else:
    # optional else suite (runs if no break)
```

**Breakdown:**
- `while`: The keyword that begins the loop.
- `condition`: Any expression evaluated for truthiness before each iteration.
- `:`: Terminates the header.
- Indented suite: Statements executed while the condition is true.
- `else`: Optional clause executed after normal termination.

**Syntax Rules**

| Rule | Description |
|---|---|
| Condition re-evaluated | The expression is tested before each iteration |
| Must eventually become false | Otherwise the loop runs forever (infinite loop) |
| Colon and indentation | Same rules as `for` |
| `else` optional | Runs only if the loop completes without `break` |
| Any expression | The condition can be any truthy/falsy expression |

**Constraints and Limitations**

- **Risk of infinite loops**: If the condition never becomes false and no `break` is used, the loop runs forever.
- **Condition variable must change**: Something inside the loop must eventually make the condition false, or a `break` must be used.
- **Not for fixed iteration**: Use `for` when the number of iterations is known; `while` for unknown counts.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `while` Loop**

```python
# Step 1: Initialize a counter
i = 1

# Step 2: Loop while the counter is less than 6
while i < 6:
    print(i)       # Print current value
    i += 1         # Increment — CRITICAL to avoid infinite loop

# Step 3: This runs after the loop ends
print("Loop finished.")
```

**Expected Output:**
```
1
2
3
4
5
Loop finished.
```

**Breakdown:** The condition `i < 6` is checked before each iteration. The `i += 1` ensures the counter eventually reaches 6, at which point the condition becomes false and the loop exits.

**Example 2: `while` Loop with `break`**

```python
# Step 1: Start an infinite loop
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input == 'quit':
        break  # Exit the loop immediately
    print(f"You entered: {user_input}")

print("Goodbye!")
```

**Expected Output (interactive):**
```
Enter 'quit' to exit: hello
You entered: hello
Enter 'quit' to exit: quit
Goodbye!
```

**Breakdown:** `while True` creates an infinite loop. The `break` statement exits when the user types 'quit'. This pattern is common for interactive programs.

**Example 3: `while` Loop with `else` Clause**

```python
# Step 1: Simulate a retry mechanism
attempts = 0
max_attempts = 3

# Step 2: Loop while attempts remain
while attempts < max_attempts:
    attempts += 1
    print(f"Attempt {attempts}")
    if attempts == 2:
        print("Success on attempt 2!")
        break
else:
    # This runs only if the loop completes without break
    print("All attempts exhausted.")
```

**Expected Output:**
```
Attempt 1
Attempt 2
Success on attempt 2!
```

**Breakdown:** The `break` on attempt 2 prevents the `else` clause from running. If no break had occurred, "All attempts exhausted" would print.

### Real-World Cases with Explanation

**Case 1: User Input Validation**

```python
while True:
    age = input("Enter your age: ")
    if age.isdigit():
        age = int(age)
        break
    print("Please enter a valid number.")
```

**Why it matters:** The loop keeps prompting until valid input is received, making the program robust against user errors.

**Case 2: Game Loop**

```python
game_over = False
while not game_over:
    handle_events()
    update_game_state()
    render()
    game_over = check_win_or_loss()
```

**Why it matters:** Games use `while` loops to continuously process input, update state, and render frames until the game ends.

---

## Core Concept 3: Nested Loops

### Definitions

**Core Definition**
Nested loops occur when one loop is placed inside the body of another loop, enabling iteration over multiple sequences or repeated actions with multiple layers of repetition.

**Technical Definition**
A nested loop is a loop that occurs within another loop, structurally similar to nested `if` statements. The outer loop's body contains the inner loop. For every iteration of the outer loop, the inner loop runs completely before the outer loop moves to its next iteration.

**Beginner-Friendly Explanation**
Think of a clock: the minute hand goes around completely (inner loop) for every single movement of the hour hand (outer loop). Nested loops let you do something for every combination of items from two or more collections.

### Purposes

- **To iterate over multidimensional data** (matrices, grids, tables)
- **To generate patterns** (triangles, squares, pyramids)
- **To perform tasks with multiple layers** of repetition (e.g., all pairs of items)
- **To process nested data structures** (lists of lists, dictionaries of lists)
- **To implement algorithms** like matrix multiplication, bubble sort, and combinatorial search

### Syntax Rules and Structure

**Complete General Syntax**

```
for outer_var in outer_iterable:
    # outer body
    for inner_var in inner_iterable:
        # inner body (runs fully for each outer iteration)
```

**Breakdown:**
- The inner loop is indented one level deeper than the outer loop.
- For each outer iteration, the inner loop starts fresh and runs to completion.
- The total number of inner-loop executions is (outer iterations) × (inner iterations per outer).

**Syntax Rules**

| Rule | Description |
|---|---|
| Indentation | Inner loop must be indented under the outer loop's body |
| Any combination | `for` inside `for`, `for` inside `while`, `while` inside `for`, `while` inside `while` |
| Unlimited depth | Theoretically unlimited, but practically avoid more than 3 levels |
| `break` affects innermost | `break` only exits the innermost loop it appears in |
| Variable scope | Inner loop variables are accessible after the inner loop ends (Python has function scope, not block scope) |

**Constraints and Limitations**

- **Performance**: Nested loops multiply iteration counts (O(n²) or worse); consider optimizing or using vectorized operations.
- **Readability**: Deeply nested loops become hard to follow.
- **`break` only exits one level**: To break out of both loops, use a flag or `return` from a function.
- **Variable shadowing**: Reusing the same variable name in inner and outer loops can cause bugs.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Multiplication Table**

```python
# Step 1: Outer loop for rows (1 to 3)
for i in range(1, 4):
    # Step 2: Inner loop for columns (1 to 3)
    for j in range(1, 4):
        # Step 3: Print the product, formatted
        print(f"{i*j:3}", end=" ")  # end=" " keeps output on same line
    # Step 4: Newline after each row
    print()
```

**Expected Output:**
```
  1   2   3 
  2   4   6 
  3   6   9 
```

**Breakdown:** For each `i` (1, 2, 3), the inner loop runs for `j` (1, 2, 3). The inner loop completes fully before the outer loop advances. Total iterations: 3 × 3 = 9.

**Example 2: Triangle Pattern**

```python
# Step 1: Outer loop controls the number of rows
for i in range(1, 6):
    # Step 2: Inner loop prints asterisks for the current row
    for j in range(i):
        print("*", end="")
    # Step 3: Move to the next line
    print()
```

**Expected Output:**
```
*
**
***
****
*****
```

**Breakdown:** The outer loop runs 5 times. The inner loop runs `i` times for each outer iteration, producing an increasing number of asterisks per row.

**Example 3: Breaking Out of Nested Loops**

```python
# Step 1: Define a 2D grid
grid = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
target = 5
found = False

# Step 2: Search for the target
for row in grid:
    for cell in row:
        if cell == target:
            print(f"Found {target}!")
            found = True
            break  # Exits only the inner loop
    if found:
        break  # Exits the outer loop
```

**Expected Output:**
```
Found 5!
```

**Breakdown:** The `break` inside the inner loop exits only the inner loop. The flag `found` is checked in the outer loop to break out of that one too.

### Real-World Cases with Explanation

**Case 1: Matrix Operations**

```python
def multiply_matrices(A, B):
    result = [[0] * len(B[0]) for _ in range(len(A))]
    for i in range(len(A)):
        for j in range(len(B[0])):
            for k in range(len(B)):
                result[i][j] += A[i][k] * B[k][j]
    return result
```

**Why it matters:** Matrix multiplication inherently requires nested loops over rows, columns, and the shared dimension.

**Case 2: Generating Combinations**

```python
colors = ['red', 'green', 'blue']
sizes = ['S', 'M', 'L']

for color in colors:
    for size in sizes:
        print(f"{color}-{size}")
```

**Why it matters:** Product catalogues and combinatorial problems often require generating all combinations of two or more attributes.

---

## Core Concept 4: Loop Conditions

### Definitions

**Core Definition**
Loop conditions are the expressions that control whether a `while` loop continues or stops, and the optional `else` clauses that run after loop completion.

**Technical Definition**
A `while` loop repeatedly tests its condition expression; if the expression is truthy, the suite executes. When the expression becomes falsy, the loop terminates and control passes to the `else` clause (if present) or the statement after the loop. The `for` loop does not have a condition in the same sense — it terminates when the iterable is exhausted — but it also supports an `else` clause that runs if no `break` occurred.

**Beginner-Friendly Explanation**
The condition in a `while` loop is like a gatekeeper: before each repetition, it asks "should I keep going?" If yes, the loop body runs. If no, the loop stops. Both `for` and `while` loops can have an `else` block that runs only if the loop finished "naturally" (without a `break`).

### Purposes

- **To control `while` loop termination** based on dynamic conditions
- **To implement search loops** that stop early with `break`
- **To run cleanup or notification code** after a loop completes normally (`else`)
- **To distinguish between "found" and "not found"** outcomes
- **To implement retry logic** with maximum attempt conditions

### Syntax Rules and Structure

**Complete General Syntax**

```
while condition:
    # suite
else:
    # runs if no break occurred
```

**Breakdown:**
- `condition`: Re-evaluated before each iteration.
- `else`: Executes after the loop's condition becomes false (or the `for` iterable is exhausted), provided no `break` occurred.

**Syntax Rules**

| Rule | Description |
|---|---|
| Any expression | Conditions can be comparisons, boolean variables, function calls, etc. |
| Truthiness | Non-boolean values are evaluated for truthiness |
| `else` semantics | `else` runs on normal exhaustion, not on `break` |
| `break` skips `else` | A `break` prevents the `else` clause from running |
| `continue` skips body | `continue` skips the rest of the current iteration but does not skip `else` |

**Constraints and Limitations**

- **Infinite loop risk**: If the condition never becomes false and no `break` exists, the loop is infinite.
- **`else` confusion**: Many beginners mistake `else` for "if the loop was empty"; it actually means "if the loop finished normally."
- **No `elif` for loops**: Loops do not support `elif`.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: `while` with Dynamic Condition**

```python
# Step 1: Start with a value
value = 100

# Step 2: Halve the value until it drops below 10
while value > 10:
    print(f"Value: {value}")
    value = value // 2  # Integer division

print(f"Final: {value}")
```

**Expected Output:**
```
Value: 100
Value: 50
Value: 25
Value: 12
Final: 6
```

**Breakdown:** The condition `value > 10` is re-evaluated each iteration. The value halves each time: 100, 50, 25, 12, then 6 (which is not > 10), so the loop stops.

**Example 2: `for` Loop with `else` (Search Pattern)**

```python
# Step 1: Define a list and a target
fruits = ['apple', 'banana', 'cherry']
target = 'banana'

# Step 2: Search for the target
for fruit in fruits:
    if fruit == target:
        print(f"Found {target}!")
        break
else:
    print(f"{target} not in list.")
```

**Expected Output:**
```
Found banana!
```

**Breakdown:** The `break` prevents the `else` from running. If `target` were `'grape'`, the loop would complete normally and the `else` would print "grape not in list."

**Example 3: Retry Loop with Maximum Attempts**

```python
# Step 1: Simulate a connection attempt
attempts = 0
max_attempts = 3
connected = False

# Step 2: Retry until connected or max attempts reached
while attempts < max_attempts and not connected:
    attempts += 1
    print(f"Attempt {attempts}...")
    # Simulate success on the third attempt
    if attempts == 3:
        connected = True

if connected:
    print("Connection established.")
else:
    print("Failed to connect after maximum attempts.")
```

**Expected Output:**
```
Attempt 1...
Attempt 2...
Attempt 3...
Connection established.
```

**Breakdown:** The condition combines two checks: `attempts < max_attempts` (limit not reached) and `not connected` (not yet successful). The loop stops when either condition becomes false.

### Real-World Cases with Explanation

**Case 1: Password Retry with Lockout**

```python
max_attempts = 5
attempts = 0

while attempts < max_attempts:
    password = input("Enter password: ")
    if check_password(password):
        print("Access granted.")
        break
    attempts += 1
    print(f"Incorrect. {max_attempts - attempts} attempts remaining.")
else:
    print("Account locked.")
```

**Why it matters:** The `while-else` pattern clearly distinguishes between successful login (break) and lockout (else), making the security logic explicit.

**Case 2: Data Polling**

```python
import time

while not data_available():
    time.sleep(1)  # Wait before checking again
else:
    process_data()
```

**Why it matters:** Polling loops check for external conditions repeatedly. The `else` clause runs when the condition finally becomes true, triggering downstream processing.

---

## Core Concept 5: Iteration over Collections (Iterables and Iterators)

### Definitions

**Core Definition**
Iterables are objects that can return their elements one at a time; iterators are objects that produce those elements sequentially via the iterator protocol.

**Technical Definition**
An **iterable** is an object capable of returning its members one at a time, implemented via `__iter__()` (which returns an iterator) or `__getitem__()`. An **iterator** is an object representing a stream of data, implementing `__iter__()` (returning itself) and `__next__()` (returning the next item or raising `StopIteration`). The `for` loop works by calling `iter()` on the iterable, then repeatedly calling `next()` until `StopIteration` is raised.

**Beginner-Friendly Explanation**
An iterable is anything you can loop over — a list, a string, a dictionary, a file. An iterator is the "engine" that actually produces the items one by one. The `for` loop handles the engine automatically, so you just say "for each item in this collection."

### Purposes

- **To traverse collections** without manual index management
- **To process data lazily** (especially with generators) without loading everything into memory
- **To write generic code** that works with any iterable (lists, files, generators, custom objects)
- **To enable Pythonic idioms** like `for line in file:`
- **To support functional programming** patterns with `map()`, `filter()`, and `itertools`

### Syntax Rules and Structure

**Complete General Syntax**

```
for item in iterable:
    # process item
```

**Iteration Protocol:**
- `iter(iterable)` → returns an iterator
- `next(iterator)` → returns the next item or raises `StopIteration`

**Breakdown:**
- Any object with `__iter__()` or `__getitem__()` is iterable.
- The `for` loop internally calls `iter()` once, then `next()` repeatedly.
- `StopIteration` signals the end of iteration.

**Syntax Rules**

| Rule | Description |
|---|---|
| `__iter__` method | Returns an iterator object |
| `__next__` method | Returns next item; raises `StopIteration` when done |
| Iterators are iterable | `__iter__` on an iterator returns itself |
| `for` handles protocol | No manual `next()` calls needed in a `for` loop |
| Lazy evaluation | Iterators produce items on demand, not all at once |

**Constraints and Limitations**

- **One-pass**: Iterators are exhausted after one full traversal; you cannot "rewind" them.
- **Not all iterables are iterators**: A list is iterable but not an iterator; `iter(list)` creates an iterator.
- **Modifying during iteration**: Changing a collection while iterating can skip items or raise `RuntimeError`.
- **`StopIteration` in generators**: Manually raising `StopIteration` inside a generator is treated as a `RuntimeError` in Python 3.7+.

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Iterating over Different Collection Types**

```python
# Step 1: Iterate over a string (character by character)
for char in "Python":
    print(char, end=" ")
print()

# Step 2: Iterate over a tuple
for num in (10, 20, 30):
    print(num, end=" ")
print()

# Step 3: Iterate over a set (unordered)
for item in {1, 2, 3}:
    print(item, end=" ")
print()

# Step 4: Iterate over a dictionary's keys (default)
for key in {'a': 1, 'b': 2}:
    print(key, end=" ")
print()
```

**Expected Output:**
```
P y t h o n 
10 20 30 
1 2 3 
a b 
```

**Breakdown:** All these types implement the iterator protocol. Strings yield characters, tuples and sets yield elements, and dictionaries yield keys by default.

**Example 2: Custom Iterable Class**

```python
# Step 1: Define a custom iterable
class Countdown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        # Return an iterator (here, using a generator)
        for i in range(self.start, 0, -1):
            yield i

# Step 2: Use the custom iterable in a for loop
for num in Countdown(5):
    print(num, end=" ")
```

**Expected Output:**
```
5 4 3 2 1 
```

**Breakdown:** The `Countdown` class defines `__iter__`, making its instances iterable. The `yield` statement creates a generator that produces values from `start` down to 1.

**Example 3: Iterators Are One-Pass**

```python
# Step 1: Create an iterator from a list
my_list = [1, 2, 3]
my_iter = iter(my_list)

# Step 2: Consume the iterator
print(next(my_iter))  # 1
print(next(my_iter))  # 2
print(next(my_iter))  # 3
# print(next(my_iter))  # Would raise StopIteration

# Step 3: The original list is still iterable
for item in my_list:
    print(item, end=" ")
```

**Expected Output:**
```
1
2
3
1 2 3 
```

**Breakdown:** The iterator `my_iter` is exhausted after three `next()` calls. The original list remains iterable because it is not itself an iterator — `iter()` creates a fresh iterator each time.

### Real-World Cases with Explanation

**Case 1: Reading Large Files Lazily**

```python
with open('huge_file.txt') as f:
    for line in f:  # File object is an iterator
        process(line)
```

**Why it matters:** Files are iterators that yield lines one at a time, so you don't load the entire file into memory.

**Case 2: Generator Pipelines**

```python
def read_numbers():
    with open('numbers.txt') as f:
        for line in f:
            yield int(line)

def filter_even(numbers):
    for n in numbers:
        if n % 2 == 0:
            yield n

for even in filter_even(read_numbers()):
    print(even)
```

**Why it matters:** Generators compose into pipelines, processing data lazily and memory-efficiently.

---

## Core Concept 6: Iteration over Ranges (`range()`)

### Definitions

**Core Definition**
The `range()` function generates an immutable sequence of numbers, commonly used for iterating over a sequence of numbers in `for` loops.

**Technical Definition**
`range()` returns a `range` object representing an arithmetic progression. Its signature is `range(stop)`, `range(start, stop)`, or `range(start, stop, step)`. The `start` and `step` arguments are optional and default to `0` and `1`, respectively. The `stop` value is never included in the generated sequence.

**Beginner-Friendly Explanation**
`range()` is a shortcut for creating a sequence of numbers without typing them all out. `range(5)` gives you 0, 1, 2, 3, 4. You can also specify where to start and how much to jump by each time.

### Purposes

- **To iterate a specific number of times** (`for i in range(n):`)
- **To generate arithmetic sequences** with custom start, stop, and step
- **To iterate backwards** using a negative step (`range(10, 0, -1)`)
- **To access list elements by index** (`for i in range(len(items)):`)
- **To create numeric lists** (`list(range(5))`)

### Syntax Rules and Structure

**Complete General Syntax**

```
range(stop)
range(start, stop)
range(start, stop, step)
```

**Breakdown:**
- `start`: First number in the sequence (default `0`).
- `stop`: Sequence stops before this number (exclusive).
- `step`: Increment between numbers (default `1`; can be negative).

**Syntax Rules**

| Rule | Description |
|---|---|
| Exclusive stop | `range(5)` produces 0, 1, 2, 3, 4 (not 5) |
| Lazy evaluation | `range` objects are lazy; they compute numbers on demand |
| Negative step | `range(10, 0, -2)` produces 10, 8, 6, 4, 2 |
| Zero step | `range(0, 10, 0)` raises `ValueError` |
| Memory efficient | A `range` object uses the same memory regardless of size |

**Constraints and Limitations**

- **Integers only**: `range()` works with integers, not floats. Use `numpy.arange()` for float ranges.
- **Not a list**: `range(5)` returns a `range` object, not a list. Wrap in `list()` to materialise.
- **`stop` exclusive**: `range(1, 5)` produces 1, 2, 3, 4 (not 5).

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic `range()` Iteration**

```python
# Step 1: Iterate from 0 to 4
for i in range(5):
    print(i, end=" ")
print()

# Step 2: Iterate from 2 to 6
for i in range(2, 7):
    print(i, end=" ")
print()

# Step 3: Iterate with a step of 2
for i in range(0, 10, 2):
    print(i, end=" ")
```

**Expected Output:**
```
0 1 2 3 4 
2 3 4 5 6 
0 2 4 6 8 
```

**Breakdown:** `range(5)` produces 0–4. `range(2, 7)` produces 2–6. `range(0, 10, 2)` produces even numbers from 0 to 8.

**Example 2: Counting Backwards**

```python
# Step 1: Count down from 5 to 1
for i in range(5, 0, -1):
    print(i, end=" ")
print("Liftoff!")
```

**Expected Output:**
```
5 4 3 2 1 Liftoff!
```

**Breakdown:** A negative step of `-1` reverses the sequence. The loop stops before `0` (exclusive), so it ends at `1`.

**Example 3: Using `range()` with Indices**

```python
# Step 1: Define a list
fruits = ['apple', 'banana', 'cherry']

# Step 2: Iterate by index
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")
```

**Expected Output:**
```
0: apple
1: banana
2: cherry
```

**Breakdown:** `range(len(fruits))` produces indices 0, 1, 2. This pattern is useful when you need both the index and the element, though `enumerate()` is more Pythonic.

### Real-World Cases with Explanation

**Case 1: Batch Processing**

```python
batch_size = 100
for start in range(0, len(data), batch_size):
    batch = data[start:start + batch_size]
    process_batch(batch)
```

**Why it matters:** `range()` with a step efficiently divides data into fixed-size batches for processing.

**Case 2: Repeating an Action N Times**

```python
for _ in range(3):
    print("Hello!")
# Output:
# Hello!
# Hello!
# Hello!
```

**Why it matters:** When you don't need the loop variable, using `_` as the target is a convention indicating the value is ignored.

---

## Core Concept 7: Infinite Loops and Event Loops

### Definitions

**Core Definition**
An infinite loop is a loop that never terminates naturally; an event loop is a programming construct that waits for and dispatches events or messages in a program, often implemented as an infinite loop.

**Technical Definition**
An **infinite loop** occurs when a `while` condition never becomes false or a `for` loop iterates over an infinite iterator. An **event loop** is a central component of asynchronous programming that runs continuously, managing tasks, callbacks, network I/O, and subprocesses. In Python's `asyncio`, the event loop is the core of every asynchronous application.

**Beginner-Friendly Explanation**
An infinite loop runs forever — usually on purpose, like a server that waits for requests. An event loop is a special kind of infinite loop that manages many tasks at once, switching between them as needed. It's like a receptionist who keeps checking for new visitors and directing them to the right place.

### Purposes

- **To run a program continuously** (servers, daemons, game loops)
- **To wait for external events** (network requests, user input, sensor data)
- **To implement event-driven architectures** where the program reacts to events as they occur
- **To manage concurrent tasks** in asynchronous programming
- **To implement polling loops** that check for conditions at intervals

### Syntax Rules and Structure

**Complete General Syntax**

```
# Infinite loop
while True:
    # body
    if exit_condition:
        break

# Event loop (asyncio)
import asyncio

async def main():
    while True:
        await asyncio.sleep(1)
        # process events

asyncio.run(main())
```

**Breakdown:**
- `while True`: Creates an infinite loop.
- `break`: The exit mechanism (must be present unless the loop is genuinely meant to run forever).
- `asyncio.run()`: Runs the event loop until the coroutine completes.
- `await asyncio.sleep()`: Yields control to the event loop, allowing other tasks to run.

**Syntax Rules**

| Rule | Description |
|---|---|
| `while True` | Idiomatic infinite loop construct |
| `break` required | Unless truly infinite (e.g., event loop) |
| `asyncio.run()` | High-level API to run the event loop |
| `await` | Yields control to the event loop |
| `asyncio.get_running_loop()` | Returns the running event loop (preferred over `get_event_loop()`) |

**Constraints and Limitations**

- **CPU consumption**: A busy `while True` loop without `await` or `sleep` consumes 100% CPU.
- **Blocking the event loop**: Long-running synchronous code inside an event loop blocks all other tasks.
- **`get_event_loop()` deprecation**: `asyncio.get_event_loop()` is deprecated in Python 3.12+ when no running loop exists; use `asyncio.run()` or `asyncio.get_running_loop()`.
- **Not for all problems**: Event loops are ideal for I/O-bound tasks, not CPU-bound tasks (use multiprocessing instead).

### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic Infinite Loop with Break**

```python
# Step 1: Initialize a counter
count = 0

# Step 2: Loop until count reaches 3
while True:
    count += 1
    print(f"Count: {count}")
    if count >= 3:
        break  # Exit the infinite loop

print("Loop exited.")
```

**Expected Output:**
```
Count: 1
Count: 2
Count: 3
Loop exited.
```

**Breakdown:** `while True` creates an infinite loop, but the `if count >= 3: break` provides an exit condition. Without the `break`, the loop would run forever.

**Example 2: Simple Event Loop with `asyncio`**

```python
import asyncio

# Step 1: Define an async function (coroutine)
async def periodic_task():
    for i in range(3):
        print(f"Task running: {i}")
        await asyncio.sleep(0.5)  # Yield control to the event loop
    print("Task complete.")

# Step 2: Run the event loop
asyncio.run(periodic_task())
print("Event loop finished.")
```

**Expected Output:**
```
Task running: 0
Task running: 1
Task running: 2
Task complete.
Event loop finished.
```

**Breakdown:** `asyncio.run()` creates an event loop, runs the coroutine, and closes the loop. The `await asyncio.sleep(0.5)` yields control back to the event loop during the sleep.

**Example 3: Multiple Tasks in an Event Loop**

```python
import asyncio

async def task(name, delay):
    for i in range(2):
        print(f"{name}: step {i}")
        await asyncio.sleep(delay)
    print(f"{name}: done")

async def main():
    # Run two tasks concurrently
    await asyncio.gather(
        task("A", 0.5),
        task("B", 0.3)
    )

asyncio.run(main())
```

**Expected Output:**
```
A: step 0
B: step 0
B: step 1
A: step 1
B: done
A: done
```

**Breakdown:** `asyncio.gather()` schedules both coroutines on the same event loop. They interleave because each `await` yields control, allowing the other task to run.

### Real-World Cases with Explanation

**Case 1: Web Server Event Loop**

```python
async def handle_request(reader, writer):
    data = await reader.read(100)
    writer.write(data)
    await writer.drain()
    writer.close()

async def main():
    server = await asyncio.start_server(
        handle_request, '127.0.0.1', 8888
    )
    async with server:
        await server.serve_forever()

asyncio.run(main())
```

**Why it matters:** Web servers use event loops to handle thousands of concurrent connections without creating a thread per connection.

**Case 2: Game Loop**

```python
import pygame

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
    # Update game state
    # Render frame
    pygame.display.flip()
pygame.quit()
```

**Why it matters:** Games use infinite loops to continuously process input, update state, and render frames until the player quits.

---

## References

- Python Software Foundation. *4. More Control Flow Tools — Python 3.14 documentation*. https://docs.python.org/3.14/tutorial/controlflow.html
- Python Software Foundation. *8. Compound Statements — Python Language Reference*. https://docs.python.org/3/reference/compound_stmts.html
- Python Software Foundation. *Functional Programming HOWTO*. https://docs.python.org/3.14/howto/functional.html
- Python Software Foundation. *Event Loop — asyncio documentation*. https://docs.python.org/3.12/library/asyncio-eventloop.html
- Python Software Foundation. *asyncio — Asynchronous I/O*. https://docs.python.org/3.13/library/asyncio.html
- Real Python. *Nested Loops in Python*. https://realpython.com/nested-loops-python/
- Python Software Foundation. *PEP 204 – Range Literals*. https://peps.python.org/pep-0204/
- Python Software Foundation. *PEP 284 – Integer for-loops*. https://peps.python.org/pep-0284/
- Real Python. *Python "for" Loops (Definite Iteration)*. https://realpython.com/python-for-loop/
- Real Python. *Python "while" Loops (Indefinite Iteration)*. https://realpython.com/python-while-loop/
- Real Python. *Iterators and Iterables in Python: Run Efficient Iterations*. https://realpython.com/python-iterators-iterables/
- Python Software Foundation. *Built-in Functions — range()*. https://docs.python.org/3/library/functions.html#func-range
- Python Software Foundation. *itertools — Functions creating iterators for efficient looping*. https://docs.python.org/3/library/itertools.html