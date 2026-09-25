# Python Debugging: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Debugging is the systematic process of identifying, isolating, and fixing errors (bugs) in a program, ensuring it behaves as intended.

**Technical Definition**
Debugging encompasses a set of techniques and tools for inspecting a program's state during execution, tracing the flow of control, and identifying the root cause of incorrect behavior. Python's debugging ecosystem includes traceback analysis for post-mortem diagnosis, the `pdb` interactive debugger for live inspection, logging for runtime observability, assertions for invariant checking, and systematic failure reproduction for reliable bug reporting.

**Beginner-Friendly Explanation**
Debugging is like being a detective for your code. When something goes wrong—a crash, wrong output, or unexpected behavior—you need to figure out *what* happened, *where* it happened, and *why*. Python gives you several tools: error messages that point to the problem (tracebacks), a way to pause your program and poke around inside it (`pdb`), a journal that records what your program was doing (logging), and checkpoints that verify your assumptions (assertions).

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Tracebacks** | Automatically generated error reports showing the call stack and exact failure location  |
| **Interactive Debugging** | `pdb` allows pausing execution and inspecting variables, stepping through code, and evaluating expressions  |
| **Logging** | Structured, level-based recording of events for post-mortem analysis  |
| **Assertions** | Inline self-checks that verify assumptions during development  |
| **Reproducibility** | Minimizing failure cases to create reliable, shareable bug reports  |

### Prerequisites

- Basic Python syntax and program structure
- Understanding of exceptions and the call stack
- Familiarity with running Python scripts from the command line
- Basic knowledge of functions and variable scope

### Related Programming Areas

- **Exception Handling**: Catching and responding to errors
- **Testing**: Unit tests and integration tests that verify correctness
- **Logging and Monitoring**: Production observability
- **Development Workflow**: IDE integration, linters, and type checkers

### Core Concepts / Features

1. **Tracebacks** (reading and interpreting error reports)
2. **Stack Frames** (the call stack and frame inspection)
3. **`pdb`** (the Python Debugger)
4. **Breakpoints** (pausing execution at specific points)
5. **Logging** (structured event recording)
6. **Assertions** (inline invariant checking)
7. **Interactive Debugging** (live inspection and stepping)
8. **Reproducing Failures** (creating minimal reproducible examples)


## Core Concept 1: Tracebacks

### Definitions

**Core Definition**
A traceback is the structured text output Python prints when an unhandled exception occurs, showing the sequence of function calls that led to the error.

**Technical Definition**
When an exception propagates to the top level without being caught, the interpreter prints a traceback to `stderr`. The traceback lists each stack frame from the outermost call ("Traceback (most recent call last)") to the innermost frame where the exception was raised. Each frame entry includes the file path, line number, function name, and the source line that was executing. The final line shows the exception type and message .

**Beginner-Friendly Explanation**
A traceback is Python's way of saying "here's exactly where things went wrong and how we got there." It reads from top to bottom: the top shows where the program started, and the bottom shows where it crashed. The last line tells you what kind of error occurred.

### Purposes

- **To locate the exact line** where an exception was raised
- **To trace the call chain** that led to the error
- **To identify the exception type** and its message
- **To provide post-mortem debugging information** without re-running the program

### Syntax Rules and Structure

```python
Traceback (most recent call last):
  File "example.py", line 5, in <module>
    result = divide(10, 0)
  File "example.py", line 2, in divide
    return a / b
ZeroDivisionError: division by zero
```

**Structure Breakdown:**

| Component | Example | Meaning |
|---|---|---|
| Header | `Traceback (most recent call last):` | Indicates order; bottom is most recent |
| File path | `File "example.py"` | Source file containing the call |
| Line number | `line 5` | Line where the call was made |
| Function/scope | `in <module>` or `in divide` | Function or module-level code |
| Source line | `result = divide(10, 0)` | The actual code at that line |
| Exception | `ZeroDivisionError: division by zero` | Error type and message  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Simple Traceback**

```python
def divide(a, b):
    return a / b

def main():
    result = divide(10, 0)
    print(result)

main()
```

**Expected Output:**
```
Traceback (most recent call last):
  File "example.py", line 7, in <module>
    main()
  File "example.py", line 4, in main
    result = divide(10, 0)
  File "example.py", line 2, in divide
    return a / b
ZeroDivisionError: division by zero
```

**Breakdown:**
1. The traceback lists three frames: module-level call to `main()`, then `main()`'s call to `divide()`, then the failing line in `divide()`.
2. Reading bottom-to-top: the error occurred in `divide` at line 2; it was called from `main` at line 4; `main` was called from module level at line 7.
3. The final line identifies the exception: `ZeroDivisionError` with message "division by zero."

### Real-World Cases with Explanation

**Case 1: Missing File**

```python
with open("missing.txt") as f:
    content = f.read()
```

**Traceback:**
```
FileNotFoundError: [Errno 2] No such file or directory: 'missing.txt'
```

**Why it matters:** The traceback immediately tells you the file doesn't exist and shows the exact path attempted. For `FileNotFoundError`, the `Errno 2` is a POSIX error code for "No such file or directory" .


## Core Concept 2: Stack Frames

### Definitions

**Core Definition**
A stack frame (or execution frame) is a data structure that Python creates each time a function is called, containing the function's local variables, arguments, and the point of execution.

**Technical Definition**
The call stack is a LIFO (last-in, first-out) structure of frames. Each frame holds the function's code object, local namespace, global namespace reference, and the instruction pointer. When an exception occurs, the traceback captures the current stack of frames. The `pdb` debugger allows moving between frames with `up` and `down` commands to inspect different scopes .

**Beginner-Friendly Explanation**
Think of stack frames as a stack of papers. When function A calls function B, a new paper is placed on top. When B returns, its paper is removed. Each paper has a snapshot of what that function was doing. When an error happens, Python shows you all the papers in the stack.

### Purposes

- **To understand the call chain** at any point in execution
- **To inspect local variables** in different scopes during debugging
- **To trace how the program reached** a particular line

### Syntax Rules and Structure

In `pdb`, frame navigation commands include:

| Command | Purpose |
|---|---|
| `w(here)` | Print stack trace with current frame marked  |
| `u(p) [count]` | Move up to older frame(s)  |
| `d(own) [count]` | Move down to newer frame(s)  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Navigating Frames in pdb**

```python
def level_three():
    x = 10
    y = 20
    breakpoint()  # Pause here
    return x + y

def level_two():
    a = 1
    result = level_three()
    return result

def level_one():
    return level_two()

level_one()
```

**Expected Output (at pdb prompt):**
```
> /tmp/example.py(4)level_three()
-> return x + y
(Pdb) w
  /tmp/example.py(12)<module>()
-> level_one()
  /tmp/example.py(10)level_one()
-> return level_two()
  /tmp/example.py(7)level_two()
-> result = level_three()
> /tmp/example.py(4)level_three()
-> return x + y
(Pdb) u
> /tmp/example.py(7)level_two()
-> result = level_three()
(Pdb) p a
1
```

**Breakdown:**
1. `w` shows the full stack: module → `level_one` → `level_two` → `level_three` (current).
2. `u` moves up one frame to `level_two`.
3. `p a` prints the local variable `a` in `level_two`'s frame, showing `1` .

### Real-World Cases with Explanation

**Case 1: Recursion Depth**

```python
def recurse(n):
    if n == 0:
        return 0
    return recurse(n - 1) + 1

recurse(5)
```

**Why it matters:** When debugging recursive functions, the stack frames show the depth of recursion. `pdb` can navigate up and down to inspect each recursive call's local `n` value.


## Core Concept 3: `pdb`

### Definitions

**Core Definition**
`pdb` is Python's built-in interactive source code debugger, allowing you to pause execution, inspect state, and step through code line by line.

**Technical Definition**
The `pdb` module provides a command-line interface for debugging. It can be invoked programmatically via `breakpoint()` (Python 3.7+) or `pdb.set_trace()`, or from the command line via `python -m pdb script.py`. Once inside the debugger, commands like `next`, `step`, `continue`, `print`, and `where` control execution and inspection .

**Beginner-Friendly Explanation**
`pdb` is like a "pause button" for your Python program. When you hit a breakpoint, you can look at all your variables, run lines one at a time, and see exactly what's happening. It's the most direct way to find bugs.

### Purposes

- **To pause execution** at a specific point and inspect program state
- **To step through code** line by line or function by function
- **To evaluate expressions** in the current context
- **To set breakpoints** dynamically and conditionally
- **To navigate the call stack** and inspect different frames

### Syntax Rules and Structure

**Entering the Debugger:**

```python
# Programmatic entry
breakpoint()           # Python 3.7+
import pdb; pdb.set_trace()  # Older versions
```

**Command-line entry:**

```bash
python -m pdb my_script.py
```

**Essential pdb Commands:**

| Command | Shortcut | Purpose |
|---|---|---|
| `help` | `h` | Show available commands  |
| `next` | `n` | Execute current line, don't step into functions  |
| `step` | `s` | Execute current line, step into functions  |
| `continue` | `c` | Resume execution until next breakpoint  |
| `print expr` | `p` | Evaluate and print expression  |
| `where` | `w` | Print stack trace  |
| `up` / `down` | `u` / `d` | Move between frames  |
| `list` | `l` | Show source code around current line  |
| `quit` | `q` | Exit debugger  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic pdb Session**

```python
def calculate(a, b):
    result = a + b
    breakpoint()  # Enter debugger here
    return result * 2

calculate(5, 10)
```

**Expected Output (at pdb prompt):**
```
> /tmp/example.py(3)calculate()
-> return result * 2
(Pdb) p a
5
(Pdb) p b
10
(Pdb) p result
15
(Pdb) n
--Return--
> /tmp/example.py(3)calculate()->30
-> return result * 2
(Pdb) p result * 2
30
(Pdb) c
```

**Breakdown:**
1. `breakpoint()` pauses execution before the `return` statement.
2. `p a`, `p b`, and `p result` inspect the local variables, showing `5`, `10`, and `15`.
3. `n` executes the current line and shows the return value `30`.
4. `c` continues execution to completion .

**Example 2: Stepping vs. Next**

```python
def helper(x):
    return x * 2

def main():
    breakpoint()
    a = 5
    b = helper(a)  # 'next' will not step into helper
    c = b + 1
    return c

main()
```

**Expected Output (at pdb prompt):**
```
(Pdb) n
> /tmp/example.py(7)main()
-> b = helper(a)
(Pdb) n
> /tmp/example.py(8)main()
-> c = b + 1
(Pdb) p b
10
```

**Breakdown:** Using `next` executes `helper(a)` fully without stepping into it. The result `b = 10` is visible afterward. Using `step` instead would enter `helper`'s frame .

### Real-World Cases with Explanation

**Case 1: Debugging a Logic Error**

```python
def find_max(numbers):
    if not numbers:
        return None
    max_val = numbers[0]
    for n in numbers:
        if n < max_val:  # BUG: should be >
            max_val = n
    return max_val

breakpoint()
print(find_max([3, 1, 4, 1, 5, 9, 2, 6]))
```

**Why it matters:** With `pdb`, you can step through the loop and inspect `max_val` after each comparison to see why the wrong value is returned. The debugger reveals that the comparison operator is backwards.


## Core Concept 4: Breakpoints

### Definitions

**Core Definition**
A breakpoint is a marker that tells the debugger to pause execution at a specific line or function.

**Technical Definition**
Breakpoints can be set programmatically (`breakpoint()` or `pdb.set_trace()`) or interactively within `pdb` using the `break` command. `pdb` supports conditional breakpoints (pause only if a condition is true), temporary breakpoints (removed after first hit), and breakpoints with associated commands (e.g., automatically print a variable and continue) .

**Beginner-Friendly Explanation**
A breakpoint is like a "stop sign" you place in your code. When the program reaches that line, it pauses and lets you inspect everything. You can place them by editing the code or by telling `pdb` to set one.

### Purposes

- **To pause execution** at a precise location for inspection
- **To avoid stepping through** many lines to reach the interesting point
- **To pause conditionally** (only when a specific condition is met)
- **To automate inspection** with breakpoint commands

### Syntax Rules and Structure

**pdb Breakpoint Commands:**

```
(Pdb) b [filename:]lineno | function [, condition]
(Pdb) tbreak [filename:]lineno | function [, condition]
(Pdb) cl [filename:lineno | bpnumber ...]
(Pdb) disable [bpnumber ...]
(Pdb) enable [bpnumber ...]
(Pdb) ignore bpnumber [count]
(Pdb) condition bpnumber [condition]
(Pdb) commands [bpnumber]
```

**Breakdown:**

| Command | Purpose |
|---|---|
| `b lineno` | Set breakpoint at line  |
| `b function` | Set breakpoint at function entry  |
| `b lineno, condition` | Set conditional breakpoint  |
| `tbreak` | Temporary breakpoint (auto-removed)  |
| `ignore bp count` | Skip breakpoint first `count` times  |
| `commands bp` | Specify commands to run when breakpoint hits  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Conditional Breakpoint**

```python
def process(items):
    for i, item in enumerate(items):
        result = item * 2
    return result

process([1, 2, 3, 4, 5])
```

**pdb session:**
```
(Pdb) b 3, i == 3
Breakpoint 1 at /tmp/example.py:3
(Pdb) c
> /tmp/example.py(3)process()
-> result = item * 2
(Pdb) p i
3
(Pdb) p item
4
```

**Breakdown:** The conditional breakpoint only pauses when `i == 3`. When it hits, `i` is `3` and `item` is `4` .

**Example 2: Breakpoint with Commands**

```python
def loop():
    for i in range(5):
        x = i * 10

loop()
```

**pdb session:**
```
(Pdb) b 3
Breakpoint 1 at /tmp/example.py:3
(Pdb) commands 1
(com) silent
(com) p i, x
(com) c
(com) end
(Pdb) c
0 0
1 10
2 20
3 30
4 40
```

**Breakdown:** The `commands` block tells the debugger to silently print `i` and `x`, then continue. The breakpoint doesn't pause execution but logs values automatically .

### Real-World Cases with Explanation

**Case 1: Debugging a Loop Bug**

```python
def find_target(items, target):
    for i, item in enumerate(items):
        if item == target:
            return i
    return -1
```

**Why it matters:** Setting a conditional breakpoint `b 3, item == target` pauses only when the target is found, letting you inspect `i` and `item` at that exact moment without stepping through irrelevant iterations.


## Core Concept 5: Logging

### Definitions

**Core Definition**
Logging is the practice of recording events that occur during a program's execution for later analysis, debugging, or monitoring.

**Technical Definition**
Python's `logging` module provides a flexible framework for emitting log messages. Loggers are organized hierarchically by name (typically `__name__` per module). Messages have severity levels (DEBUG, INFO, WARNING, ERROR, CRITICAL) and are dispatched to handlers, which format and output them to destinations (console, files, network). The root logger can be configured with `basicConfig()` .

**Beginner-Friendly Explanation**
Logging is like keeping a diary for your program. Instead of using `print()` statements that you have to remove later, you write log messages that can be turned on or off, sent to different places, and formatted consistently. It's how professional programs report what they're doing.

### Purposes

- **To record program events** without modifying code for production vs. development
- **To provide runtime observability** for debugging and monitoring
- **To replace `print()` statements** with structured, controllable output
- **To capture errors and warnings** with full context and timestamps

### Syntax Rules and Structure

```python
import logging

# Module-level logger
logger = logging.getLogger(__name__)

# Basic configuration
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='app.log'
)

# Log messages
logger.debug("Detailed information")
logger.info("General information")
logger.warning("Something unexpected")
logger.error("A failure occurred")
logger.critical("Severe failure")
```

**Log Levels:**

| Level | Numeric Value | Use Case |
|---|---|---|
| DEBUG | 10 | Detailed diagnostic information |
| INFO | 20 | General progress messages |
| WARNING | 30 | Something unexpected but non-fatal |
| ERROR | 40 | A serious problem |
| CRITICAL | 50 | A severe error  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Logging**

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

logging.info("Application started")
logging.warning("Low disk space")
logging.error("Failed to connect to database")
```

**Expected Output:**
```
2026-09-25 14:30:00,000 - INFO - Application started
2026-09-25 14:30:00,001 - WARNING - Low disk space
2026-09-25 14:30:00,002 - ERROR - Failed to connect to database
```

**Breakdown:** `basicConfig` sets the format and level. Messages below INFO (like DEBUG) are suppressed .

**Example 2: Module-Level Logger**

```python
# mylib.py
import logging

logger = logging.getLogger(__name__)

def do_something():
    logger.info("Doing something in mylib")
    return 42
```

**Breakdown:** Using `getLogger(__name__)` creates a logger named after the module (`mylib`). This allows fine-grained control: you can set `mylib` to DEBUG while keeping root at WARNING .

### Real-World Cases with Explanation

**Case 1: Debugging in Production**

```python
import logging

logger = logging.getLogger(__name__)

def process_order(order_id):
    logger.info(f"Processing order {order_id}")
    try:
        result = charge_card(order_id)
        logger.info(f"Order {order_id} charged: {result}")
    except PaymentError as e:
        logger.error(f"Payment failed for order {order_id}: {e}", exc_info=True)
        raise
```

**Why it matters:** `exc_info=True` includes the full traceback in the log, so you can debug production failures after the fact without reproducing them .


## Core Concept 6: Assertions

### Definitions

**Core Definition**
An assertion is a statement that verifies a condition is true, raising `AssertionError` if it is false.

**Technical Definition**
The `assert` statement is a debugging aid that tests a condition. If the condition is `False`, an `AssertionError` is raised with an optional message. Assertions are compiled into bytecode only when `__debug__` is `True` (the default). Running Python with `-O` or `-OO` removes assertions, making them unsuitable for production validation .

**Beginner-Friendly Explanation**
An assertion is like a "sanity check" you write into your code. You say "I believe this is true at this point." If it's not, Python stops and tells you. It's a way to catch your own mistakes early.

### Purposes

- **To document assumptions** about program state
- **To catch bugs early** by failing fast when invariants are violated
- **To validate internal logic** during development (not production)
- **To serve as inline documentation** of what the code expects 

### Syntax Rules and Structure

```python
assert condition, "optional error message"
```

**Syntax Rules:**

| Rule | Description |
|---|---|
| Condition is any expression | If truthy, nothing happens; if falsy, `AssertionError` is raised  |
| Message is optional | Printed with the `AssertionError`  |
| Removed with `-O` | `python -O script.py` strips all assertions  |
| Not for production validation | Use `if`/`raise` for production checks  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Basic Assertion**

```python
def calculate_average(numbers):
    assert len(numbers) > 0, "Cannot calculate average of empty list"
    return sum(numbers) / len(numbers)

print(calculate_average([1, 2, 3]))
# Output: 2.0

print(calculate_average([]))
# AssertionError: Cannot calculate average of empty list
```

**Breakdown:** The assertion catches the empty-list edge case. The message explains the precondition .

**Example 2: Assertion for Type Checking**

```python
def process_age(age):
    assert isinstance(age, int), f"Age must be int, got {type(age)}"
    assert age >= 0, f"Age cannot be negative: {age}"
    return age * 2

print(process_age(25))
# Output: 50

print(process_age("twenty"))
# AssertionError: Age must be int, got <class 'str'>
```

**Breakdown:** Multiple assertions verify different conditions. Each fails with a specific message .

### Real-World Cases with Explanation

**Case 1: Invariant Checking in Data Structures**

```python
class MyDB:
    def __init__(self):
        self._id2name = {}
        self._name2id = {}

    def add(self, id, name):
        self._name2id[name] = id
        self._id2name[id] = name

    def by_name(self, name):
        id = self._name2id[name]
        assert self._id2name[id] == name, "Map invariant violated"
        return id
```

**Why it matters:** The assertion verifies that the two maps remain inverses of each other. If a bug breaks this invariant, the assertion catches it immediately at the point of access .


## Core Concept 7: Interactive Debugging

### Definitions

**Core Definition**
Interactive debugging is the practice of pausing a running program to inspect its state, execute code in context, and control execution flow.

**Technical Definition**
Interactive debugging in Python is primarily accomplished through `pdb`, which provides a REPL (read-eval-print loop) within the context of a paused program. This allows not only inspection but also modification of variables, calling functions, and testing hypotheses about the bug in real time.

**Beginner-Friendly Explanation**
Interactive debugging is like being able to freeze time while your program runs. You can look at everything, try things out, and see what happens—all without restarting the program.

### Purposes

- **To inspect variables** and their values at the point of failure
- **To test hypotheses** about what's causing the bug
- **To understand complex state** that's hard to reason about statically
- **To experiment safely** without modifying source code

### Syntax Rules and Structure

**Common Interactive Commands (in addition to those in Core Concept 3):**

| Command | Purpose |
|---|---|
| `pp expression` | Pretty-print a data structure  |
| `whatis expression` | Print the type of an expression |
| `interact` | Start an interactive interpreter in the current frame |
| `run [args]` | Restart the debugged Python program |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Inspecting and Modifying State**

```python
def buggy_function(data):
    total = 0
    for item in data:
        total += item  # BUG: accumulates instead of something else
    breakpoint()
    return total

result = buggy_function([1, 2, 3, 4, 5])
```

**pdb session:**
```
> /tmp/example.py(6)buggy_function()
-> return total
(Pdb) p total
15
(Pdb) p data
[1, 2, 3, 4, 5]
(Pdb) pp data
[1, 2, 3, 4, 5]
(Pdb) total = 100  # Modify variable
(Pdb) p total
100
(Pdb) c
100
```

**Breakdown:**
1. `p total` shows the accumulated value `15`.
2. `pp data` pretty-prints the list.
3. `total = 100` modifies the local variable directly, demonstrating that interactive debugging allows state changes .

**Example 2: Using `interact` for Complex Inspection**

```python
def process(data):
    breakpoint()
    return data

process({"key": [1, 2, 3]})
```

**pdb session:**
```
(Pdb) interact
Now in an interactive Python session.
>>> d = {"key": [1, 2, 3]}
>>> d["key"].append(4)
>>> d
{'key': [1, 2, 3, 4]}
>>> exit()
(Pdb) c
{'key': [1, 2, 3]}
```

**Breakdown:** The `interact` command drops into a full Python REPL, allowing complex experimentation. Changes made in the interactive session are visible when resuming .

### Real-World Cases with Explanation

**Case 1: Debugging a Complex Algorithm**

```python
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    breakpoint()  # Inspect the partition
    return quicksort(left) + middle + quicksort(right)
```

**Why it matters:** During recursion, `pdb` lets you inspect `left`, `middle`, and `right` at each level. You can verify the partition logic is correct before proceeding to the recursive calls.


## Core Concept 8: Reproducing Failures

### Definitions

**Core Definition**
Reproducing a failure means creating a reliable, minimal scenario that consistently triggers a bug, making it easier to diagnose and fix.

**Technical Definition**
A Minimal Reproducible Example (MRE) is a self-contained code snippet that demonstrates the bug with minimal dependencies and complexity. It should include all necessary imports, a clear description of expected vs. actual behavior, and the exact error output. MREs are essential for effective bug reports and collaborative debugging .

**Beginner-Friendly Explanation**
When you find a bug, you want to strip away everything that isn't related to it, leaving only the essential code that shows the problem. This makes it much easier to figure out what's wrong—and to get help from others.

### Purposes

- **To isolate the root cause** by eliminating irrelevant code
- **To enable consistent testing** of fixes
- **To communicate bugs effectively** in issue reports
- **To verify that a fix resolves the problem** reliably

### Syntax Rules and Structure

**Elements of a Good MRE:**

| Element | Description |
|---|---|
| Minimal code | Only what's needed to reproduce  |
| Self-contained | No missing imports or external data |
| Clear description | Expected vs. actual behavior |
| Error output | Full traceback or wrong result |
| Dependencies listed | Exact versions of libraries |
| Tested | Verify it reproduces the bug  |

### Multiple Annotated Complete Code Examples and Expected Outputs

**Example 1: Poor vs. Good Bug Report**

**Poor:**
```
My code doesn't work. Please help.
```

**Good:**
```python
import pandas as pd

# Expected: 3 rows
# Actual: 4 rows (duplicate)
df = pd.DataFrame({"a": [1, 1, 2], "b": [3, 4, 5]})
result = df.drop_duplicates()
print(len(result))
```

**Expected Output:**
```
3
```

**Actual Output (bug):**
```
4
```

**Breakdown:** The good report includes:
1. Minimal code (no unrelated imports or functions)
2. Clear expected vs. actual behavior
3. Reproducible output
4. No external dependencies beyond pandas 

**Example 2: Isolating a Bug in a Larger System**

```python
# Original code (complex)
def process_pipeline(data):
    cleaned = clean(data)
    transformed = transform(cleaned)
    result = aggregate(transformed)
    return result

# Minimal reproduction (isolated)
def aggregate(data):
    return sum(data) / len(data)  # Bug: fails on empty list

# MRE
result = aggregate([])
# ZeroDivisionError: division by zero
```

**Breakdown:** By extracting just the `aggregate` function and testing it with an empty list, the bug is isolated and the root cause (missing empty-list check) is obvious .

### Real-World Cases with Explanation

**Case 1: Reporting a Bug to an Open-Source Project**

```python
# Minimal Reproducible Example for Ultralytics bug
import torch
from ultralytics import YOLO

model = YOLO("yolo26n.pt")
image = torch.rand(1, 0, 640, 640)  # 0-channel image
results = model(image)

# Error:
# RuntimeError: Expected input[1, 0, 640, 640] to have 3 channels,
# but got 0 channels instead
```

**Why it matters:** This MRE uses public models and datasets, includes exact dependency versions, and clearly demonstrates the bug. Maintainers can copy-paste and run it immediately .


## References

- Python Software Foundation. *pdb — The Python Debugger*. https://docs.python.org/3.13/library/pdb.html 
- Python Software Foundation. *logging — Logging facility for Python*. https://docs.python.org/3.14/library/logging.html 
- Real Python. *Python's assert: Debug and Test Your Code Like a Pro*. https://realpython.com/python-assert-statement/ 
- Microsoft Learn. *Use tracebacks to find errors*. https://learn.microsoft.com/en-us/training/modules/python-error-handling/2-tracebacks 
- Real Python. *traceback | Python Glossary*. https://realpython.com/ref/glossary/traceback/ 
- PSF. *Using Assertions Effectively*. https://psf.github.io/wiki/python/archive/UsingAssertionsEffectively.html 
- Ultralytics. *Creating a Minimum Reproducible Example*. https://docs.ultralytics.com/help/minimum-reproducible-example 
- GitHub. *python/Pdb.py at main · tnear/python*. https://github.com/tnear/python/blob/main/Pdb.py 