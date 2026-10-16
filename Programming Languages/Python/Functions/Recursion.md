# Python Recursion: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Recursion is a programming technique in which a function calls itself to solve a problem by breaking it down into smaller, simpler subproblems of the same type. A recursive function continues to call itself until it reaches a condition where the answer is known directly.

**Technical Definition**

In Python, recursion occurs when a function's body contains one or more calls to the function itself. Each recursive call creates a new stack frame on the call stack, storing the function's local variables and return address. Python enforces a default recursion depth limit of 1,000 frames to prevent infinite recursion from causing a C stack overflow and crashing the interpreter. This limit can be queried with `sys.getrecursionlimit()` and modified with `sys.setrecursionlimit()`. Python does not implement automatic tail call optimization (TCO), meaning that even properly tail-recursive functions allocate a new stack frame for each call. Exceeding the recursion limit raises a `RecursionError`.

**Beginner-Friendly Explanation**

Recursion is like Russian nesting dolls. To find the smallest doll, you open the outer doll, then the next one inside it, and keep going until there are no more dolls. Each step is the same action — open a doll — applied to a smaller doll. In Python, a recursive function is a function that calls itself with a smaller version of the problem until it reaches a stopping point.

### Key Characteristics

- **Self-Reference**: A recursive function calls itself during execution.
- **Base Case**: Every correct recursive function must have at least one base case that stops the recursion.
- **Recursive Case**: The part of the function that calls itself with a smaller or simpler input.
- **Call Stack**: Each recursive call creates a new stack frame; Python's default limit is ~1,000 frames.
- **No Tail Call Optimization**: Python does not eliminate tail calls, so deep recursion is not memory-efficient.
- **Declarative**: Recursion expresses *what* to compute, not *how* to iterate.
- **Natural Fit**: Recursion is ideal for problems with recursive structure (trees, graphs, fractals, divide-and-conquer).

### Prerequisites

- Basic understanding of Python functions and the `def` keyword.
- Familiarity with function calls, parameters, and return values.
- Knowledge of the call stack and local scope.
- Awareness of iterative constructs (`for`, `while`).

### Related Programming Areas

- **Data Structures**: Trees, linked lists, graphs, and nested structures.
- **Algorithms**: Divide-and-conquer (merge sort, quicksort), backtracking (N-Queens, sudoku), dynamic programming.
- **Functional Programming**: Recursion is a fundamental technique in languages like Haskell and Scheme.
- **File Systems**: Recursive directory traversal and tree walking.
- **Mathematical Computation**: Factorials, Fibonacci numbers, GCD, and fractals.

---

## Core Concepts / Features

---

### 1. Recursion

#### Definitions

**Core Definition**

Recursion is a process where a function calls itself once or multiple times to solve a problem by reducing it to smaller instances of the same problem.

**Technical Definition**

A recursive function is a function that invokes itself within its own body. Each invocation creates a new stack frame with its own local namespace. The function must eventually reach a base case that returns a result without further recursion, allowing the stack to unwind. Python's default recursion limit is 1,000 frames, though this is platform-dependent.

**Beginner-Friendly Explanation**

Recursion is when a function solves a problem by calling itself on a smaller version of the same problem. Think of counting down: to count from 5, you say "5" and then count from 4; to count from 4, you say "4" and then count from 3, and so on until you reach 0.

#### Purposes

- To solve problems that have a naturally recursive structure (trees, graphs, nested data).
- To implement divide-and-conquer algorithms (merge sort, quicksort).
- To express backtracking algorithms cleanly (N-Queens, sudoku solver).
- To traverse hierarchical data (file systems, XML/JSON trees).
- To provide elegant solutions for mathematical recurrences (factorial, Fibonacci).

#### Syntax Rules and Structure

**Complete General Syntax**

```
def recursive_function(parameters):
    if base_condition:
        return base_value          # Base case
    else:
        # Recursive case: smaller input
        return recursive_function(modified_parameters)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `base_condition` | Condition that stops recursion. |
| `base_value` | Value returned when the base case is reached. |
| `recursive_function(modified_parameters)` | The function calls itself with a smaller/simpler input. |

**Syntax Rules**

- Every recursive function must have at least one base case.
- The recursive call must make progress toward the base case (e.g., decrease `n`, shrink a list).
- All paths through the function must eventually reach a base case.
- Each recursive call creates a new stack frame; memory grows linearly with depth.
- Python's default recursion limit is 1,000 frames; exceeding it raises `RecursionError`.

**Constraints and Limitations**

- Python does not optimize tail recursion; deep recursion can cause stack overflow.
- Recursive solutions are often slower than iterative ones due to function call overhead.
- Excessive recursion depth can exhaust memory resources.
- Debugging recursive functions can be challenging due to the nested call stack.

#### Annotated Code Examples

**Example 1: Factorial (Classic Recursion)**

```python
def factorial(n):
    """Return n! (n factorial)."""
    if n <= 1:          # Base case: 0! = 1! = 1
        return 1
    return n * factorial(n - 1)  # Recursive case

print(factorial(5))   # 5 * 4 * 3 * 2 * 1 = 120
print(factorial(10))  # 3628800
```

**Expected Output:**

```
120
3628800
```

**Step-by-Step Breakdown:**

1. `factorial(5)` checks `5 <= 1` → False, so it calls `factorial(4)`.
2. `factorial(4)` calls `factorial(3)`, which calls `factorial(2)`, which calls `factorial(1)`.
3. `factorial(1)` returns `1` (base case).
4. The stack unwinds: `2 * 1 = 2`, `3 * 2 = 6`, `4 * 6 = 24`, `5 * 24 = 120`.

**Why the Output is Produced:**

Each recursive call multiplies `n` by the result of the smaller factorial. The base case stops the recursion when `n` reaches 1.

---

**Example 2: Recursive Sum of a List**

```python
def recursive_sum(numbers):
    """Return the sum of a list of numbers using recursion."""
    if not numbers:          # Base case: empty list
        return 0
    return numbers[0] + recursive_sum(numbers[1:])  # Recursive case

print(recursive_sum([1, 2, 3, 4, 5]))  # 15
```

**Expected Output:**

```
15
```

**Step-by-Step Breakdown:**

1. `recursive_sum([1,2,3,4,5])` returns `1 + recursive_sum([2,3,4,5])`.
2. Each call peels off the first element and recurses on the rest.
3. When the list is empty, the base case returns `0`.
4. The stack unwinds: `1+2+3+4+5 = 15`.

**Why the Output is Produced:**

The recursive case reduces the list by one element each time, moving toward the empty-list base case.

#### Real-World Cases

- **Mathematical Computation**: Factorials, Fibonacci numbers, greatest common divisor (GCD).
- **File System Traversal**: Walking directory trees, computing disk usage.
- **Parsing**: Processing nested JSON or XML structures.
- **Fractals**: Generating fractal trees and Sierpinski triangles.

---

### 2. Base Case

#### Definitions

**Core Definition**

A base case is the condition in a recursive function that stops the recursion and returns a result directly without making further recursive calls.

**Technical Definition**

The base case is a conditional branch that is reached when the problem has been reduced to a trivial instance. It provides the terminating condition for the recursion. Without a base case (or with a base case that is never reached), the recursion would continue indefinitely until the interpreter raises a `RecursionError`.

**Beginner-Friendly Explanation**

The base case is the "stop" signal. It's like the bottom of a staircase — when you reach it, you stop walking down.

#### Purposes

- To terminate the recursion and prevent infinite loops.
- To provide the simplest instance of the problem whose answer is known directly.
- To enable the stack to unwind and return results.

#### Syntax Rules and Structure

```
if base_condition:
    return base_value
```

**Syntax Rules**

- The base case must be checked before any recursive call.
- There can be multiple base cases (e.g., Fibonacci has two).
- The base case must be reachable from every path through the function.
- The base case should handle the smallest/simplest input.

**Constraints and Limitations**

- Missing base case causes `RecursionError`.
- Incorrect base case may produce wrong results or infinite recursion.
- Multiple base cases increase code complexity but are sometimes necessary.

#### Annotated Code Examples

**Example 1: Fibonacci with Two Base Cases**

```python
def fibonacci(n):
    """Return the nth Fibonacci number."""
    if n == 0:      # Base case 1
        return 0
    if n == 1:      # Base case 2
        return 1
    return fibonacci(n - 1) + fibonacci(n - 2)  # Recursive case

print(fibonacci(10))  # 55
```

**Expected Output:**

```
55
```

**Step-by-Step Breakdown:**

1. `fibonacci(10)` calls `fibonacci(9)` and `fibonacci(8)`.
2. This branches recursively until reaching the base cases `fibonacci(0)` and `fibonacci(1)`.
3. The results are summed as the stack unwinds.

**Why the Output is Produced:**

The two base cases define the starting values of the Fibonacci sequence (0 and 1). The recursive case builds larger values from these.

#### Real-World Cases

- **Tree Traversal**: Base case is an empty tree or a leaf node.
- **Divide-and-Conquer**: Base case is a subproblem of size 1 or 0.
- **Backtracking**: Base case is a complete solution.

---

### 3. Recursive Case

#### Definitions

**Core Definition**

The recursive case is the part of a recursive function that calls the function itself with a smaller or simpler version of the problem.

**Technical Definition**

The recursive case is the branch of the function that makes one or more recursive calls. It must reduce the problem size so that progress is made toward the base case. The recursive call may be a single call (linear recursion) or multiple calls (tree recursion).

**Beginner-Friendly Explanation**

The recursive case is the part where the function says, "I don't know the answer yet, but if I solve a smaller version of this problem, I can figure it out."

#### Purposes

- To break a large problem into smaller subproblems.
- To leverage the function's own logic to solve subproblems.
- To express divide-and-conquer and backtracking algorithms naturally.

#### Syntax Rules and Structure

```
return recursive_function(smaller_input)
```

**Syntax Rules**

- The input to the recursive call must be smaller or simpler than the current input.
- The recursive call must move the problem closer to the base case.
- There can be one or multiple recursive calls.
- The result of the recursive call may be combined with other values.

**Constraints and Limitations**

- Failure to reduce the problem size causes infinite recursion.
- Multiple recursive calls can lead to exponential time complexity (e.g., naive Fibonacci).
- The order of recursive calls matters for some algorithms.

#### Annotated Code Examples

**Example 1: Linear Recursion (Factorial)**

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)  # Recursive case: one call
```

**Why the Output is Produced:**

Each call reduces `n` by 1, moving toward the base case `n <= 1`.

**Example 2: Tree Recursion (Fibonacci)**

```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)  # Two recursive calls
```

**Why the Output is Produced:**

Each call branches into two smaller calls, creating a tree of calls.

#### Real-World Cases

- **Merge Sort**: Two recursive calls on halves of the list.
- **Quicksort**: Two recursive calls on partitions.
- **Tree Traversal**: One recursive call per child node.

---

### 4. Recursion Depth

#### Definitions

**Core Definition**

Recursion depth is the number of active recursive calls on the call stack at any given time. It represents how deeply the recursion has nested.

**Technical Definition**

Recursion depth is the length of the call stack at its deepest point during a recursive computation. Python tracks this depth and raises a `RecursionError` when it exceeds the limit set by `sys.setrecursionlimit()` (default 1,000). The actual maximum depth is limited by the C stack size, which is platform-dependent (e.g., 1 MiB on Windows, typically 8 MiB on Linux).

**Beginner-Friendly Explanation**

Recursion depth is how many times the function has called itself without returning yet. It's like how many nested boxes you've opened — if you open too many, the stack of boxes gets too tall and falls over.

#### Purposes

- To understand the memory cost of recursion.
- To determine whether a recursive solution is feasible for a given input size.
- To decide when to convert recursion to iteration.

#### Syntax Rules and Structure

```python
import sys
print(sys.getrecursionlimit())  # Check current limit
sys.setrecursionlimit(5000)     # Set a new limit
```

**Syntax Rules**

- Default limit: 1,000 frames.
- `sys.getrecursionlimit()` returns the current limit.
- `sys.setrecursionlimit(n)` sets the limit to `n`.
- Exceeding the limit raises `RecursionError`.

**Constraints and Limitations**

- Setting the limit too high can cause a C stack overflow and crash the interpreter.
- The actual stack size is platform-dependent.
- Deep recursion is memory-intensive regardless of the limit.

#### Annotated Code Examples

**Example 1: Checking and Setting Recursion Limit**

```python
import sys

print("Default limit:", sys.getrecursionlimit())

def deep_recursion(n):
    if n == 0:
        return 0
    return 1 + deep_recursion(n - 1)

# With default limit (1000)
try:
    print(deep_recursion(999))
except RecursionError:
    print("Hit recursion limit!")

# Increase limit and try again
sys.setrecursionlimit(5000)
print(deep_recursion(4000))
```

**Expected Output:**

```
Default limit: 1000
999
4000
```

**Step-by-Step Breakdown:**

1. `sys.getrecursionlimit()` returns `1000`.
2. `deep_recursion(999)` succeeds (under the limit).
3. After `sys.setrecursionlimit(5000)`, `deep_recursion(4000)` succeeds.

**Why the Output is Produced:**

Increasing the recursion limit allows deeper recursion, but the actual stack size still limits how far you can go safely.

#### Real-World Cases

- **Tree Depth**: Recursion depth equals tree depth.
- **Divide-and-Conquer**: Depth is O(log n) for balanced splits.
- **Linear Recursion**: Depth equals input size.

---

### 5. `sys.setrecursionlimit()`

#### Definitions

**Core Definition**

`sys.setrecursionlimit()` is a function in the `sys` module that sets the maximum depth of the Python interpreter stack.

**Technical Definition**

`sys.setrecursionlimit(limit)` sets the maximum depth of the Python interpreter stack to `limit`. This limit prevents infinite recursion from causing an overflow of the C stack and crashing Python. The highest possible limit is platform-dependent. A too-high limit can lead to a crash.

**Beginner-Friendly Explanation**

`sys.setrecursionlimit()` lets you change how deep recursion can go before Python stops it. It's like adjusting the maximum number of nested boxes you're allowed to open.

#### Purposes

- To allow deeper recursion for algorithms that require it.
- To prevent infinite recursion from crashing the interpreter.
- To tune recursion depth for specific platforms and use cases.

#### Syntax Rules and Structure

```
import sys
sys.setrecursionlimit(limit)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `limit` | Integer specifying the new maximum recursion depth. |

**Syntax Rules**

- The default limit is 1,000.
- The limit is platform-dependent.
- Raising the limit too high can cause a segfault.
- The limit applies to Python code only (Python 3.12+).

**Constraints and Limitations**

- Too high a limit can crash the interpreter due to C stack overflow.
- The actual stack size is independent of the recursion limit.
- Threads have their own stack sizes.

#### Annotated Code Examples

**Example 1: Using `setrecursionlimit`**

```python
import sys

# Check default limit
print("Default:", sys.getrecursionlimit())

# Set a higher limit
sys.setrecursionlimit(10000)
print("New limit:", sys.getrecursionlimit())

# Recursive function that needs deep recursion
def count_down(n):
    if n == 0:
        return 0
    return 1 + count_down(n - 1)

print(count_down(5000))  # Works with increased limit
```

**Expected Output:**

```
Default: 1000
New limit: 10000
5000
```

**Step-by-Step Breakdown:**

1. `sys.getrecursionlimit()` returns `1000`.
2. `sys.setrecursionlimit(10000)` raises the limit to 10,000.
3. `count_down(5000)` now succeeds because it needs 5,000 frames.

**Why the Output is Produced:**

Raising the limit allows deeper recursion, but the C stack must still accommodate the frames.

#### Real-World Cases

- **Deep Tree Traversal**: When tree depth exceeds 1,000.
- **Recursive Parsing**: Parsing deeply nested structures.
- **Algorithm Prototyping**: Temporarily raising the limit during development.

---

### 6. Stack Overflow

#### Definitions

**Core Definition**

A stack overflow occurs when the call stack exceeds its allocated size, typically because of excessive recursion depth or infinite recursion.

**Technical Definition**

In Python, a stack overflow is prevented by the `RecursionError` exception, which is raised when the recursion depth exceeds the limit set by `sys.setrecursionlimit()`. However, if the C stack itself overflows (e.g., due to a too-high recursion limit or a segfault in C code), the interpreter may crash.

**Beginner-Friendly Explanation**

A stack overflow is like a tower of blocks that gets too tall and collapses. In Python, you usually get a friendly `RecursionError` before things get really bad, but setting the limit too high can cause a crash.

#### Purposes

- To understand the risks of deep recursion.
- To recognize the symptoms of stack overflow (RecursionError, segfault).
- To motivate iterative alternatives.

#### Syntax Rules and Structure

```python
# RecursionError is raised when the limit is exceeded.
def infinite_recursion():
    return infinite_recursion()  # Never reaches base case

infinite_recursion()  # RecursionError: maximum recursion depth exceeded
```

**Constraints and Limitations**

- `RecursionError` is a subclass of `RuntimeError`.
- Catching `RecursionError` and then recursing again can crash the interpreter.
- The actual stack size is platform-dependent.

#### Annotated Code Examples

**Example 1: Infinite Recursion**

```python
def count_up(n):
    print(n)
    count_up(n + 1)  # No base case!

try:
    count_up(0)
except RecursionError as e:
    print(f"Error: {e}")
```

**Expected Output:**

```
0
1
2
...
996
997
Error: maximum recursion depth exceeded
```

**Step-by-Step Breakdown:**

1. `count_up(0)` prints 0 and calls `count_up(1)`.
2. This continues until the recursion limit is reached.
3. Python raises `RecursionError`.

**Why the Output is Produced:**

Without a base case, the recursion never stops, eventually exceeding the limit.

#### Real-World Cases

- **Infinite Recursion Bugs**: Forgetting a base case.
- **Too-Deep Recursion**: Algorithms with depth exceeding the limit.
- **C Stack Overflow**: Setting the limit too high and causing a segfault.

---

### 7. Tail Recursion

#### Definitions

**Core Definition**

Tail recursion is a special case of recursion where the recursive call is the very last operation performed in the function. There is no pending computation after the recursive call returns.

**Technical Definition**

A function is tail-recursive if the recursive call appears in tail position — that is, the return value of the recursive call is immediately returned without any further operations. In languages with tail call optimization (TCO), tail-recursive functions can be executed in constant stack space. Python does not implement TCO, so tail-recursive functions still grow the stack.

**Beginner-Friendly Explanation**

Tail recursion is when the function's last action is to call itself. In some languages, this is optimized to a loop, but Python does not do this, so it still uses stack space.

#### Purposes

- To enable recursion in constant stack space in languages with TCO.
- To express iterative algorithms recursively.
- To understand why Python's lack of TCO limits recursion depth.

#### Syntax Rules and Structure

```
# Tail-recursive factorial
def factorial_tail(n, accumulator=1):
    if n <= 1:
        return accumulator
    return factorial_tail(n - 1, n * accumulator)  # Tail call
```

**Syntax Rules**

- The recursive call must be the last operation.
- The result of the recursive call must be returned immediately.
- No operations (addition, multiplication, etc.) may follow the recursive call.

**Constraints and Limitations**

- Python does not optimize tail calls.
- Tail-recursive functions still allocate a new frame per call.
- The accumulator pattern is often used in tail recursion.

#### Annotated Code Examples

**Example 1: Tail-Recursive Factorial**

```python
def factorial_tail(n, acc=1):
    """Tail-recursive factorial."""
    if n <= 1:
        return acc
    return factorial_tail(n - 1, n * acc)

print(factorial_tail(5))  # 120
```

**Expected Output:**

```
120
```

**Step-by-Step Breakdown:**

1. `factorial_tail(5, 1)` calls `factorial_tail(4, 5)`.
2. `factorial_tail(4, 5)` calls `factorial_tail(3, 20)`.
3. Continues until `n <= 1`, returning the accumulator.

**Why the Output is Produced:**

The accumulator carries the partial result forward, so no multiplication is needed after the recursive call.

#### Real-World Cases

- **Functional Programming**: Tail recursion is idiomatic in Scheme, Haskell, and Erlang.
- **Python**: Tail recursion is not optimized, so it is rarely used for deep recursion.

---

### 8. Tail Call Optimization (TCO)

#### Definitions

**Core Definition**

Tail call optimization (TCO) is a compiler or interpreter optimization that eliminates the stack frame of a tail-recursive call, allowing tail recursion to execute in constant stack space.

**Technical Definition**

TCO transforms tail-recursive calls into jumps, reusing the current stack frame instead of allocating a new one. This eliminates the risk of stack overflow for tail-recursive functions. Python does **not** implement TCO. The decision to omit TCO was deliberate: it preserves clear tracebacks and simplifies the interpreter.

**Beginner-Friendly Explanation**

TCO is like recycling — instead of stacking a new frame for each recursive call, the language reuses the same frame. Python doesn't do this, so tail recursion still uses stack space.

#### Purposes

- To enable deep recursion without stack overflow.
- To make recursion as efficient as iteration.
- To support functional programming idioms.

#### Syntax Rules and Structure

```
# In languages with TCO, this runs in constant stack space:
def loop(n, acc):
    if n == 0:
        return acc
    return loop(n - 1, acc + n)  # Optimized to a jump
```

**Constraints and Limitations**

- Python does not support TCO.
- Workarounds: `tacopy` library transforms tail recursion into loops.
- Trampolines are another workaround.

#### Annotated Code Examples

**Example 1: TCO with the `tacopy` Library**

```python
from tacopy import tacopy

@tacopy
def factorial(n, acc=1):
    if n <= 1:
        return acc
    return factorial(n - 1, n * acc)

print(factorial(10000))  # Works without RecursionError
```

**Expected Output:**

```
284625968091705451890641321211986889014805140170279923...
```

**Step-by-Step Breakdown:**

1. `@tacopy` transforms the tail-recursive function into an iterative loop.
2. The function can handle much deeper recursion than normal.

**Why the Output is Produced:**

`tacopy` eliminates stack growth by converting tail calls into loop iterations.

#### Real-World Cases

- **Functional Languages**: TCO is standard in Scheme, Haskell, and Erlang.
- **Python Workarounds**: Libraries like `tacopy` and `trampoline` provide TCO-like behavior.

---

### 9. Recursion vs Iteration

#### Definitions

**Core Definition**

Recursion and iteration are two fundamental approaches to repeating a computation. Recursion uses self-referential function calls; iteration uses loops.

**Technical Definition**

Recursion solves problems by reducing them to smaller instances and calling the function itself. Iteration uses `for` or `while` loops to repeat a block of code. In Python, iteration is generally more memory-efficient and faster because it avoids function call overhead and stack growth.

**Beginner-Friendly Explanation**

Recursion is like asking someone to solve a smaller version of the problem. Iteration is like doing the steps yourself in a loop. Iteration is usually faster and uses less memory in Python.

#### Purposes

- To choose the right approach based on problem structure and constraints.
- To optimize performance and memory usage.
- To write clearer code for naturally recursive problems.

#### Comparison Table

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| Mechanism | Function calls itself | Loop repeats a block |
| Memory | O(depth) stack frames | O(1) usually |
| Speed | Slower (call overhead) | Faster |
| Clarity | Natural for recursive structures | Natural for linear processes |
| Depth Limit | Limited by `sys.setrecursionlimit()` | Unlimited |
| TCO | Not supported in Python | N/A |
| Best For | Trees, graphs, divide-and-conquer | Flat data, linear processes |

#### Annotated Code Examples

**Example 1: Factorial — Recursive vs Iterative**

```python
# Recursive
def factorial_rec(n):
    if n <= 1:
        return 1
    return n * factorial_rec(n - 1)

# Iterative
def factorial_iter(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

print(factorial_rec(5))   # 120
print(factorial_iter(5))  # 120
```

**Expected Output:**

```
120
120
```

**Step-by-Step Breakdown:**

1. Both produce the same result.
2. The iterative version uses a loop and constant memory.
3. The recursive version uses 5 stack frames.

**Why the Output is Produced:**

Both algorithms compute the same product; they differ in implementation.

#### Real-World Cases

- **Tree Traversal**: Recursion is natural; iteration requires an explicit stack.
- **Fibonacci**: Naive recursion is exponential; iteration or memoized recursion is linear.
- **File Walking**: Iteration with `os.walk` is often preferred for deep trees.

---

### 10. Recursion Examples

#### 10.1 Factorial

**Core Definition**

The factorial of a non-negative integer `n` (written `n!`) is the product of all positive integers up to `n`. `0! = 1` by definition.

**Recursive Definition**

```
factorial(n) = 1                  if n <= 1
factorial(n) = n * factorial(n-1) otherwise
```

**Annotated Code Example**

```python
def factorial(n):
    """Return n! (factorial of n)."""
    if n <= 1:          # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case

print(factorial(5))   # 120
print(factorial(10))  # 3628800
```

**Expected Output:**

```
120
3628800
```

**Why the Output is Produced:**

Each call multiplies `n` by the factorial of `n-1`, reducing the problem until the base case is reached.

**Real-World Case**

Factorials appear in combinatorics, probability, and permutations.

---

#### 10.2 Fibonacci

**Core Definition**

The Fibonacci sequence is defined by `F(0) = 0`, `F(1) = 1`, and `F(n) = F(n-1) + F(n-2)` for `n > 1`.

**Recursive Definition**

```
fibonacci(n) = n                        if n <= 1
fibonacci(n) = fibonacci(n-1) + fibonacci(n-2)  otherwise
```

**Annotated Code Example**

```python
def fibonacci(n):
    """Return the nth Fibonacci number."""
    if n <= 1:          # Base cases
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)  # Recursive case

print(fibonacci(10))  # 55
```

**Expected Output:**

```
55
```

**Why the Output is Produced:**

The recursive case sums the two previous Fibonacci numbers, building the sequence from the base cases.

**Real-World Case**

Fibonacci numbers appear in nature (spiral shells, branching plants) and in algorithms (Fibonacci search, heap analysis).

---

#### 10.3 Tree Traversal

**Core Definition**

Tree traversal is the process of visiting every node in a tree exactly once. Recursion is a natural fit because trees are recursive structures.

**Annotated Code Example (Binary Tree)**

```python
class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right

def inorder(node):
    """In-order traversal: left, root, right."""
    if node is None:          # Base case
        return
    inorder(node.left)        # Recursive case: left subtree
    print(node.value, end=" ")
    inorder(node.right)       # Recursive case: right subtree

# Build a tree
root = Node(1, Node(2, Node(4)), Node(3, Node(5)))
inorder(root)  # 4 2 1 5 3
```

**Expected Output:**

```
4 2 1 5 3
```

**Why the Output is Produced:**

The recursion visits the left subtree, then the node, then the right subtree, following the in-order traversal pattern.

**Real-World Case**

Tree traversal is used in file systems, HTML/XML parsing, and expression evaluation.

---

#### 10.4 Directory Traversal

**Core Definition**

Directory traversal is the process of recursively visiting every file and subdirectory in a directory tree.

**Annotated Code Example (Custom Recursive Walk)**

```python
import os

def walk_directory(path, indent=0):
    """Recursively print all files and directories."""
    for entry in os.listdir(path):
        full_path = os.path.join(path, entry)
        print(" " * indent + entry)
        if os.path.isdir(full_path):
            walk_directory(full_path, indent + 2)  # Recursive case

walk_directory(".")
```

**Expected Output (example):**

```
file1.txt
subdir
  file2.txt
  nested
    file3.txt
```

**Why the Output is Produced:**

The function lists the current directory, and for each subdirectory, it calls itself recursively.

**Real-World Case**

Directory traversal is used in backup tools, file search utilities, and disk usage analyzers.

---

#### 10.5 Tower of Hanoi

**Core Definition**

The Tower of Hanoi is a classic puzzle where `n` disks must be moved from one peg to another, with the constraint that a larger disk can never be placed on top of a smaller disk.

**Recursive Solution**

```
hanoi(n, source, auxiliary, destination):
    if n == 1:
        move disk 1 from source to destination
    else:
        hanoi(n-1, source, destination, auxiliary)
        move disk n from source to destination
        hanoi(n-1, auxiliary, source, destination)
```

**Annotated Code Example**

```python
def tower_of_hanoi(n, source, auxiliary, destination):
    """Solve Tower of Hanoi recursively."""
    if n == 1:
        print(f"Move disk 1 from {source} → {destination}")
        return
    tower_of_hanoi(n - 1, source, destination, auxiliary)
    print(f"Move disk {n} from {source} → {destination}")
    tower_of_hanoi(n - 1, auxiliary, source, destination)

tower_of_hanoi(3, "A", "B", "C")
```

**Expected Output:**

```
Move disk 1 from A → C
Move disk 2 from A → B
Move disk 1 from C → B
Move disk 3 from A → C
Move disk 1 from B → A
Move disk 2 from B → C
Move disk 1 from A → C
```

**Why the Output is Produced:**

The recursion moves `n-1` disks to the auxiliary peg, moves the largest disk to the destination, then moves the `n-1` disks from the auxiliary to the destination.

**Real-World Case**

Tower of Hanoi illustrates recursive problem-solving and is used in algorithm courses.

---

#### 10.6 Backtracking (N-Queens)

**Core Definition**

Backtracking is a recursive algorithm that builds solutions incrementally and abandons a partial solution ("backtracks") as soon as it determines the solution cannot be completed.

**Annotated Code Example (N-Queens)**

```python
def solve_n_queens(n):
    """Solve the N-Queens problem using backtracking."""
    solutions = []

    def backtrack(row, cols, diag1, diag2, board):
        if row == n:                 # Base case: all queens placed
            solutions.append(board[:])
            return
        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue  # Prune: queen is threatened
            # Place queen
            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)
            board.append(col)
            # Recurse
            backtrack(row + 1, cols, diag1, diag2, board)
            # Backtrack
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)
            board.pop()

    backtrack(0, set(), set(), set(), [])
    return solutions

solutions = solve_n_queens(4)
print(f"Found {len(solutions)} solutions for 4-Queens")
for sol in solutions:
    print(sol)
```

**Expected Output:**

```
Found 2 solutions for 4-Queens
[1, 3, 0, 2]
[2, 0, 3, 1]
```

**Why the Output is Produced:**

The backtracking function places queens row by row, pruning invalid positions, and backtracks when no valid position exists in a row.

**Real-World Case**

Backtracking is used in puzzle solvers (sudoku, crosswords), constraint satisfaction, and combinatorial optimization.

---

## References

- Python Official Documentation – `sys.setrecursionlimit` - https://docs.python.org/3/library/sys.html#sys.setrecursionlimit
- Python Official Documentation – `sys.getrecursionlimit` - https://docs.python.org/3/library/sys.html#sys.getrecursionlimit
- Real Python – Recursion in Python - https://realpython.com/courses/python-recursion/
- Real Python – Thinking Recursively in Python - https://realpython.com/python-thinking-recursively/
- FreeCodeCamp – Recursion in Python – A Practical Introduction for Beginners - https://www.freecodecamp.org/news/recursion-in-python-intro-for-beginners
- Educative – Recursion in Python Tutorial - https://www.educative.io/blog/recursion-in-python-tutorial
- Python Bug Tracker – os.walk is limited by Python's recursion limit - https://bugs.python.org/issue26545
- CPython Source – N-Queens Demo (queens.py) - https://llvm.googlesource.com/third_party/cpython/+/refs/tags/v3.11.4/Tools/demo/queens.py
- Tacopy – Tail-Call Optimization for Python - https://pypi.org/project/tacopy-optimization/
- Python Mailing List – Recursion depth discussion - https://mail.python.org/archives/list/tutor@python.org/thread/TAMHRWX4MQD3W5MZOF3J6BJJ7OH53Z7C/
- Python Mailing List – Tail call optimization not supported - https://mail.python.org/archives/list/python-checkins@python.org/thread/...
- ISO/IEC TR 24772-1:2019 – Recursion in Python - https://www9.open-std.org
- Princeton – Recursion (Sedgewick, Wayne, Dondero) - https://introcs.cs.princeton.edu
- Cornell – Recursive Functions (CS 1110) - https://www.cs.cornell.edu/courses/cs1110/2014fa/lectures/10-16-14/modules/recursion.py.txt
- MIT OpenCourseWare – Recursion on Non-numerics - https://live.ocw.mit.edu