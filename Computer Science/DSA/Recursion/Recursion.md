# Recursion Fundamentals: Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
Recursion is a programming technique where a function solves a problem by calling itself on smaller instances of the same problem, combining the results to solve the original problem .

**Technical Definition**
A recursive function is a function that invokes itself, either directly or indirectly, with modified parameters that move the computation toward a terminating condition called the base case .

**Beginner-Friendly Explanation**
Recursion is like asking a friend to help with a task, where that friend asks another friend, and so on, until the task is small enough that someone can do it directly. Then the answers travel back up the chain to you.

### Key Characteristics

- **Self-Reference**: The function's definition includes a call to itself .
- **Problem Reduction**: Each recursive call operates on a smaller or simpler version of the original problem .
- **Termination Guarantee**: A well-formed recursive function must eventually reach a base case .
- **Stack-Based Execution**: Each recursive call creates a new activation frame on the call stack .

### Prerequisites

- Understanding of functions, parameters, and return values
- Basic knowledge of conditionals (if/else statements)
- Familiarity with the concept of a call stack

### Related Programming Areas

- **Divide and Conquer Algorithms**: Merge sort, quick sort, binary search
- **Tree and Graph Traversal**: DFS, tree height, path finding
- **Dynamic Programming**: Recursive solutions with memoization
- **Backtracking**: N-Queens, maze solving, permutations

### Core Concepts

| Concept | Description |
|---------|-------------|
| Base Case | Simple case solved directly without recursion |
| Recursive Case | Case that calls the function on a smaller problem |
| Call Stack | Memory structure tracking active function calls |
| Recursive State | Parameters and local variables at each call level |
| Termination | Guarantee that recursion eventually stops |
| Stack Overflow | Error from exceeding stack memory limits |


## 1. Base Case

### Definitions

**Core Definition**
The base case is the condition under which a recursive function returns a result directly without making any further recursive calls .

**Technical Definition**
A base case is a non-recursive branch in the function's conditional logic that is triggered when the problem has been reduced to its simplest form, providing the terminating condition for the recursion .

**Beginner-Friendly Explanation**
The base case is the "stopping point" — the simplest version of the problem that you already know the answer to. It's like counting down to zero: when you reach zero, you stop.

### Purposes (All begin with "To")

- **To** terminate the recursion and prevent infinite calls 
- **To** provide the initial value from which the final answer is constructed 
- **To** define the simplest problem instance that requires no further decomposition 
- **To** serve as the foundation for mathematical induction proofs of correctness 

### Syntax Rules and Structure

**General Syntax (Python)**
```python
def recursive_function(n):
    if base_condition(n):      # Base case
        return base_result
    else:                       # Recursive case
        return combine(n, recursive_function(smaller(n)))
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `base_condition(n)` | Predicate identifying the simplest case |
| `base_result` | Direct answer without recursion |
| Return statement | Exits the current call, returns to caller |

**Syntax Rules**

- Base case must be checked **before** any recursive call 
- Base case must return a value (not call the function) 
- There may be multiple base cases (e.g., Fibonacci has two) 

**Constraints and Limitations**

- Missing base case → infinite recursion → stack overflow 
- Incorrect base value → wrong final result propagating through all returns

### Annotated Code Examples

**Example 1: Factorial Base Case (Python)**

```python
def factorial(n):
    """Return n! for n >= 0."""
    # BASE CASE: 0! = 1 by definition
    if n == 0:
        return 1  # No recursive call; provides the seed value

    # RECURSIVE CASE: n! = n * (n-1)!
    return n * factorial(n - 1)


print(factorial(5))  # 120
print(factorial(0))  # 1 (base case directly)
```

**Expected Output:**
```
120
1
```

**Why This Output Occurs**: When `n` reaches 0, the base case returns 1 directly. This value is then multiplied by 1, 2, 3, 4, and 5 as the stack unwinds .

### Real-World Cases

- **Linked List Traversal**: Base case is `node is None` (empty list)
- **Tree Height**: Base case is a leaf node (no children)
- **Binary Search**: Base case is empty search range or found element
- **Tower of Hanoi**: Base case is one disk to move

### References

- CMU 15-110 - Recursion Base Cases - https://www.cs.cmu.edu/~15110-s20/slides/week6-3-recursion.pdf


## 2. Recursive Case

### Definitions

**Core Definition**
The recursive case is the branch of a recursive function that solves the problem by calling the function itself on a smaller or simpler input .

**Technical Definition**
A recursive case defines the solution to a problem instance in terms of solutions to one or more smaller instances of the same problem, using self-reference combined with a reduction operation .

**Beginner-Friendly Explanation**
The recursive case is the "ask someone else" step. When the problem is too big to solve directly, you break it into a smaller piece and delegate the rest to another call of the same function.

### Purposes (All begin with "To")

- **To** decompose a complex problem into smaller subproblems of the same type 
- **To** leverage the same solution logic across different problem sizes
- **To** build the final answer by combining the current piece with the subproblem's result 
- **To** make progress toward the base case with each call 

### Syntax Rules and Structure

**General Syntax**
```python
def recursive_function(n):
    if base_condition(n):
        return base_result
    
    # RECURSIVE CASE
    smaller_input = reduce(n)           # Make problem smaller
    smaller_result = recursive_function(smaller_input)  # Delegate
    return combine(n, smaller_result)   # Combine results
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `reduce(n)` | Operation that makes the problem smaller |
| Recursive call | Function calls itself with reduced input |
| `combine(...)` | Uses current value + subproblem result |

**Syntax Rules**

- Recursive call must operate on a **strictly smaller** problem 
- The reduction must eventually reach the base case 
- Multiple recursive calls are allowed (e.g., Fibonacci) 

**Constraints and Limitations**

- Non-reducing calls → infinite recursion 
- Multiple recursive calls can cause exponential time (e.g., naive Fibonacci) 

### Annotated Code Examples

**Example 1: Sum of Digits (Python)**

```python
def sum_digits(n):
    """Return sum of digits of n. Precondition: n >= 0."""
    # BASE CASE: single digit
    if n < 10:
        return n

    # RECURSIVE CASE: last digit + sum of remaining digits
    return n % 10 + sum_digits(n // 10)


print(sum_digits(843))  # 8+4+3 = 15
print(sum_digits(7))    # 7 (base case)
```

**Expected Output:**
```
15
7
```

**Why This Output Occurs**: `sum_digits(843)` returns `3 + sum_digits(84)`, which returns `4 + sum_digits(8)`, which hits the base case and returns 8. The sum propagates: 3 + 4 + 8 = 15 .

### Real-World Cases

- **Power Function**: `power(x, n) = x * power(x, n-1)`
- **String Reversal**: `reverse(s) = reverse(s[1:]) + s[0]`
- **List Sum**: `sum_list(lst) = lst[0] + sum_list(lst[1:])`

### References

- CMU 15-110 - Recursive Cases - https://www.cs.cmu.edu/~15110-s20/slides/week6-3-recursion.pdf
- Cornell CS 2110 - Understanding Recursive Methods - https://www.cs.cornell.edu/courses/cs2110/2019sp/L08-Recursion/cs2110Recursion.pdf


## 3. Call Stack

### Definitions

**Core Definition**
The call stack is a region of memory that stores information about active function calls, including parameters, local variables, and return addresses, using a Last-In-First-Out (LIFO) discipline .

**Technical Definition**
Each function invocation creates an activation frame (stack frame) containing the function's arguments, local variables, and return address. These frames are pushed onto the call stack when a function is called and popped when it returns .

**Beginner-Friendly Explanation**
The call stack is like a stack of sticky notes. Each time you call a function, you write down where you are and what you're doing on a note, then put it on top of the pile. When the function finishes, you remove the top note and continue from where you left off.

### Purposes (All begin with "To")

- **To** track the execution state of each active function call 
- **To** store return addresses so execution can resume after a call completes 
- **To** keep local variables isolated between different function invocations 
- **To** enable recursive calls by allowing multiple simultaneous instances of the same function 

### Syntax Rules and Structure

**Stack Frame Contents**

| Component | Description |
|-----------|-------------|
| Arguments | Parameters passed to the function |
| Local variables | Variables declared inside the function |
| Return address | Where to resume in the caller |
| Saved registers | CPU state if needed |

**Call Sequence**
```
Caller executes → Push arguments → Push return address → Jump to function
Function executes → Creates local variables → Returns
Return → Pop frame → Jump to return address → Caller continues
```

### Annotated Code Examples

**Example 1: Stack Frames During Factorial (Python)**

```python
def factorial(n):
    """Each call creates a new stack frame."""
    print(f"  PUSH: factorial({n})")
    if n == 0:
        result = 1
    else:
        result = n * factorial(n - 1)
    print(f"  POP:  factorial({n}) = {result}")
    return result


print(factorial(3))
```

**Expected Output:**
```
  PUSH: factorial(3)
  PUSH: factorial(2)
  PUSH: factorial(1)
  PUSH: factorial(0)
  POP:  factorial(0) = 1
  POP:  factorial(1) = 1
  POP:  factorial(2) = 2
  POP:  factorial(3) = 6
6
```

**Why This Output Occurs**: Each call pushes a frame before recursing (PUSH lines) and pops it after completing (POP lines). The LIFO order is visible: factorial(0) completes first, then unwinds up to factorial(3) .

### Real-World Cases

- **Debugging**: Stack traces show the call stack at the point of error
- **Exception Handling**: Stack unwinding during exception propagation
- **Profiling**: Sampling the call stack identifies performance bottlenecks

### References

- IIT Bombay CS 101 - Activation Frames and Stack Memory - https://www.cse.iitb.ac.in/~cs101/2016.2/slides/Lecture13.pdf
- CMU 15-213 - Stack Frame Layout - http://www.cs.cmu.edu/afs/cs/academic/class/15213-s01/lectures/class07.pdf


## 4. Recursive State

### Definitions

**Core Definition**
Recursive state refers to the collection of parameter values and local variable values that exist simultaneously across all active recursive calls .

**Technical Definition**
At any point during recursive execution, each active call has its own independent set of parameter bindings and local variables, preserved in its stack frame until the call completes .

**Beginner-Friendly Explanation**
When a function calls itself, the new call gets its own "copy" of the function's variables. The original call's values are saved on the stack and restored when the nested call finishes.

### Purposes (All begin with "To")

- **To** preserve the computation context of each recursive call independently 
- **To** enable combining results from multiple subproblems correctly 
- **To** ensure that different invocations do not interfere with each other's data 
- **To** support the mathematical induction proof structure of recursion 

### Syntax Rules and Structure

**State Isolation**

| Aspect | Behavior |
|--------|----------|
| Parameters | Each call has its own binding |
| Local variables | Fresh storage per call |
| Global variables | Shared across all calls (usually avoided) |

### Annotated Code Examples

**Example 1: Independent State in Recursive Calls**

```python
def countdown(n, message):
    """Each call has independent n and message."""
    print(f"n={n}, message='{message}'")

    if n <= 0:
        return

    # The nested call gets its own n and message
    countdown(n - 1, message + "!")


countdown(3, "Go")
```

**Expected Output:**
```
n=3, message='Go'
n=2, message='Go!'
n=1, message='Go!!'
n=0, message='Go!!!'
```

**Why This Output Occurs**: Each recursive call receives a modified `n` and `message`. The original call's state is preserved on the stack; the nested call operates on its own copies .

### Real-World Cases

- **Binary Search**: Each recursive call has its own low/high bounds
- **Tree Traversal**: Each call has its own subtree root
- **Backtracking**: Each call has its own path state

### References

- IIT Bombay CS 101 - Recursion and Data Spaces - https://www.cse.iitb.ac.in/~cs101/2016.2/slides/Lecture13.pdf


## 5. Termination

### Definitions

**Core Definition**
Termination means that a recursive function is guaranteed to eventually stop calling itself and return a result .

**Technical Definition**
A recursive function terminates if every sequence of recursive calls reaches a base case in finite time, which is guaranteed when each recursive call operates on a strictly smaller input according to a well-founded measure .

**Beginner-Friendly Explanation**
Termination is the guarantee that recursion will actually finish — not loop forever. It's like knowing that if you keep halving a number, you'll eventually reach 1.

### Purposes (All begin with "To")

- **To** prevent infinite loops and stack overflow errors 
- **To** ensure the function produces a result in finite time 
- **To** enable formal correctness proofs via induction 
- **To** make recursion a viable alternative to iteration

### Syntax Rules and Structure

**Termination Conditions**

| Requirement | Description |
|-------------|-------------|
| Base case exists | At least one non-recursive branch |
| Progress made | Each call reduces the problem |
| Well-founded order | Reduction cannot continue indefinitely |

### Annotated Code Examples

**Example 1: Termination Verification**

```python
def gcd(m, n):
    """Euclidean algorithm. Terminates because n decreases each call."""
    if m % n == 0:
        return n  # Base case
    return gcd(n, m % n)  # m % n < n, so progress is made


print(gcd(205, 123))
```

**Expected Output:**
```
41
```

**Why This Output Occurs**: `gcd(205, 123)` → `gcd(123, 82)` → `gcd(82, 41)` → base case (82 % 41 == 0) returns 41. The second argument strictly decreases each call, guaranteeing termination .

### Real-World Cases

- **Euclidean Algorithm**: GCD terminates because remainder decreases
- **Tree Recursion**: Terminates because subtrees are smaller
- **Divide and Conquer**: Terminates because subproblems are smaller

### References

- Cornell CS 2110 - Termination Step - https://www.cs.cornell.edu/courses/cs2110/2019sp/L08-Recursion/cs2110Recursion.pdf


## 6. Stack Overflow Risks and Mitigation

### Definitions

**Core Definition**
Stack overflow occurs when recursive calls consume all available stack memory, typically because the recursion is too deep or lacks proper termination .

**Technical Definition**
Each recursive call allocates a stack frame. When the cumulative size of active frames exceeds the stack's memory limit, the program crashes with a StackOverflowError (or equivalent) .

**Beginner-Friendly Explanation**
The call stack has limited space. If you keep piling on function calls without ever finishing them, the pile gets too high and collapses — that's a stack overflow.

### Purposes (All begin with "To")

- **To** understand the practical limits of recursion depth 
- **To** recognize when iterative solutions are safer 
- **To** implement mitigations that prevent crashes in production systems 

### Risk Factors

| Risk Factor | Description |
|-------------|-------------|
| Deep recursion | n calls for large n (e.g., factorial(10000)) |
| Missing base case | Infinite recursion |
| Large frames | Each call uses significant stack memory |
| Attacker-controlled input | User can force deep recursion  |

### Mitigation Strategies

| Strategy | Description |
|----------|-------------|
| **Iteration** | Convert to loops (no stack growth)  |
| **Tail Recursion** | Some languages optimize tail calls to reuse frames  |
| **Increase Stack Size** | OS-level limit adjustment (not portable) |
| **Memoization** | Reduces call count but not depth |
| **Explicit Stack** | Use heap-allocated stack data structure |

### Annotated Code Examples

**Example 1: Detecting and Preventing Stack Overflow**

```python
import sys

def factorial_recursive(n):
    """Recursive: depth n. Fails for large n."""
    if n <= 1:
        return 1
    return n * factorial_recursive(n - 1)


def factorial_iterative(n):
    """Iterative: O(1) stack space."""
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result


# Test with large n
N = 5000

print(f"Recursion limit: {sys.getrecursionlimit()}")

# Iterative works fine
print(f"Iterative factorial({N}) computed successfully")

# Recursive may hit the limit
try:
    factorial_recursive(N)
    print(f"Recursive factorial({N}) computed successfully")
except RecursionError:
    print(f"Recursive factorial({N}) raised RecursionError")
```

**Expected Output:**
```
Recursion limit: 1000
Iterative factorial(5000) computed successfully
Recursive factorial(5000) raised RecursionError
```

**Why This Output Occurs**: Python's default recursion limit (1000) prevents deep recursion. The iterative version uses O(1) stack space, so it handles n=5000. The recursive version exceeds the limit and raises `RecursionError` .

### Real-World Cases

- **OS Stack Limits**: Linux default ~8MB stack; each frame ~100 bytes → ~80,000 frames max
- **Web Servers**: Deeply recursive parsing code is vulnerable to StackOverflow DoS 
- **Embedded Systems**: Very limited stack requires iterative solutions

### References

- SEI CERT - MEM05-C: Avoid Large Stack Allocations - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20240923-230924-1847-1131/MEM05-C.+Avoid+large+stack+alloc_7e5d7ef6d42c4cc19f18f9dff97b12a9-230924-1847-1132.pdf
- Ruby VM Documentation - Stack Overflow Mechanics - https://docs.ruby-lang.org/en/master/contributing/vm_stack_and_frames_md.html


## 7. Memory Complexity of the Call Stack (O(N) Auxiliary Space)

### Definitions

**Core Definition**
The auxiliary space complexity of a recursive function is O(N) when the maximum recursion depth is proportional to the input size N, because each active call occupies stack space .

**Technical Definition**
For a recursive function with maximum depth d and constant frame size, the stack space usage is O(d). For linear recursion (e.g., factorial), d = N, yielding O(N) auxiliary space .

**Beginner-Friendly Explanation**
If a recursive function calls itself N times before finishing, there are N unfinished calls on the stack at the deepest point. Each takes up space, so the total stack memory is proportional to N.

### Purposes (All begin with "To")

- **To** correctly analyze space complexity of recursive algorithms 
- **To** compare recursive and iterative memory usage 
- **To** identify memory bottlenecks in recursive solutions 
- **To** understand when recursion is memory-inefficient 

### Syntax Rules and Structure

**Space Complexity Calculation**

| Component | Description |
|-----------|-------------|
| Frame size | O(1) per call (constant local variables) |
| Maximum depth | d (depends on input and algorithm) |
| Stack space | O(d) |

**Common Patterns**

| Recursion Type | Depth | Space Complexity |
|----------------|-------|------------------|
| Linear (factorial) | N | O(N) |
| Divide and conquer (binary search) | log N | O(log N) |
| Divide and conquer (merge sort) | log N | O(log N) |
| Exponential (naive Fibonacci) | N | O(N) |

### Annotated Code Examples

**Example 1: Measuring Stack Depth**

```python
def sum_recursive(n, depth=0):
    """Linear recursion: depth = n."""
    print(f"Depth {depth}: n={n}")
    if n == 0:
        return 0
    return n + sum_recursive(n - 1, depth + 1)


def sum_iterative(n):
    """Iterative: constant stack space."""
    total = 0
    for i in range(n + 1):
        total += i
    return total


# Demonstrate space usage
print("Recursive (note depth increments):")
sum_recursive(4)

print(f"\nRecursive space: O(n) = O(5) frames")
print(f"Iterative space: O(1) frame")

print(f"\nsum_recursive(1000) would use ~1000 frames")
print(f"sum_iterative(1000) uses 1 frame")
```

**Expected Output:**
```
Recursive (note depth increments):
Depth 0: n=4
Depth 1: n=3
Depth 2: n=2
Depth 3: n=1
Depth 4: n=0

Recursive space: O(n) = O(5) frames
Iterative space: O(1) frame

sum_recursive(1000) would use ~1000 frames
sum_iterative(1000) uses 1 frame
```

**Why This Output Occurs**: The recursive function creates a new frame for each value of n from 4 down to 0. The maximum depth is 5 (n+1 frames). The iterative version uses only one frame regardless of n .

### Real-World Cases

- **Merge Sort**: O(log N) stack space for recursion depth
- **Quick Sort**: O(log N) expected, O(N) worst-case
- **Tree Traversal**: O(height) stack space
- **Factorial**: O(N) stack space — often converted to iterative for large n

### References

- GitHub - Space Complexity Analysis - Recursion Takes Space - https://notebooks.githubusercontent.com/view/ipynb?browser=chrome&bypass_fastly=true&color_mode=auto&commit=3fd8009e08e03e67f912ccbb062b19398ebf9cc6&device=unknown_device&docs_host=https%3A%2F%2Fdocs.github.com&enc_url=68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f7368726176616e6b756d6172303831312f436f64696e675f4e696e6a61735f496e5f507974686f6e2f33666438303039653038653033336536376639313263636262303632623139333938656266396363362f446174612d537472756374757265732d616e642d416c676f726974686d732d696e2d507974686f6e2d6d61737465722f30372532305370616365253230436f6d706c65786974792f5370616365253230436f6d706c6578697479253230416e616c797369732e6970796e62&logged_in=false&nwo=shravankumar0811%2FCoding_Ninjas_In_Python&path=Data-Structures-and-Algorithms-in-Python-master%2F07+Space+Complexity%2FSpace+Complexity+Analysis.ipynb&platform=mac&repository_id=406628108&repository_type=Repository&version=148#152c2a4b-4d99-4582-aa5d-6b5ac3ddd317
- Educative - Space Complexity and Recursion - https://d.edurev.in/t/187351/space-complexity
- Hello Algo - Tail Recursion Space Complexity - https://github.com/krahets/hello-algo/releases/download/1.3.0/hello-algo_1.3.0_en_javascript.pdf


## Consolidated References

- CMU 15-110 - Recursion Fundamentals - https://www.cs.cmu.edu/~15110-s20/slides/week6-3-recursion.pdf
- IIT Bombay CS 101 - Recursive Functions and Activation Frames - https://www.cse.iitb.ac.in/~cs101/2016.2/slides/Lecture13.pdf
- SEI CERT - MEM05-C: Avoid Large Stack Allocations - https://wiki.sei.cmu.edu/confluence/download/export/pdfexport-20240923-230924-1847-1131/MEM05-C.+Avoid+large+stack+alloc_7e5d7ef6d42c4cc19f18f9dff97b12a9-230924-1847-1132.pdf
- GitHub - Space Complexity: Recursion Takes Space - https://notebooks.githubusercontent.com/view/ipynb?browser=chrome&bypass_fastly=true&color_mode=auto&commit=3fd8009e08e03e67f912ccbb062b19398ebf9cc6&device=unknown_device&docs_host=https%3A%2F%2Fdocs.github.com&enc_url=68747470733a2f2f7261772e67697468756275736572636f6e74656e742e636f6d2f7368726176616e6b756d6172303831312f436f64696e675f4e696e6a61735f496e5f507974686f6e2f33666438303039653038653033336536376639313263636262303632623139333938656266396363362f446174612d537472756374757265732d616e642d416c676f726974686d732d696e2d507974686f6e2d6d61737465722f30372532305370616365253230436f6d706c65786974792f5370616365253230436f6d706c6578697479253230416e616c797369732e6970796e62&logged_in=false&nwo=shravankumar0811%2FCoding_Ninjas_In_Python&path=Data-Structures-and-Algorithms-in-Python-master%2F07+Space+Complexity%2FSpace+Complexity+Analysis.ipynb&platform=mac&repository_id=406628108&repository_type=Repository&version=148#152c2a4b-4d99-4582-aa5d-6b5ac3ddd317
- Cornell CS 2110 - Recursion and Understanding Methods - https://www.cs.cornell.edu/courses/cs2110/2019sp/L08-Recursion/cs2110Recursion.pdf
- UC San Diego CSE 11 - Recursion Basics - https://users.sdsc.edu/~phil/cse11/lectures/lecture20/Lecture20.pdf
- Cornell CS 1109 - Recursion Examples - https://www.cs.cornell.edu/courses/cs1109/2024su/lectures/lec13_recursion.pdf
- Ruby VM Documentation - Stack Frames and Overflow - https://docs.ruby-lang.org/en/master/contributing/vm_stack_and_frames_md.html
- Stanford CS 106X - Recursion and Stack Frames - https://web.stanford.edu/class/archive/cs/cs106x/cs106x.1142/lectures/08-recursion.pdf
- GitHub Ada Developers Academy - Introduction to Recursion - https://raw.githubusercontent.com/Ada-Developers-Academy/textbook-curriculum/refs/heads/master/04-cs-fundamentals/classroom/recursion.md
- MIT 6.0001 - Recursion and Fibonacci - https://ocw.mit.edu/courses/6-0001-introduction-to-computer-science-and-programming-in-python-fall-2016/876348c652c5353daccc96e1b7d577bb_MIT6_0001F16_Lec6.pdf
- CNRS - Fonctions Récursives et Pile d'Exécution - https://usr.lmf.cnrs.fr/~kn/teaching/ipf/cours/02.html
- Python.org Mailing List - Stack Overflow Discussion - https://mail.python.org/pipermail/python-list/2011-May/602914.html
- Educative - Space Complexity and Recursion - https://d.edurev.in/t/187351/space-complexity
- York University EECS 2030 - Tracing Method Calls - https://www.eecs.yorku.ca/~wangcw/teaching/lectures/2018/F/EECS2030/slides/09-Recursion.pdf
- Hello Algo - Tail Recursion and Space Complexity - https://github.com/krahets/hello-algo/releases/download/1.3.0/hello-algo_1.3.0_en_javascript.pdf