# Recursion — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Recursion is a programming technique in which a function solves a problem by **calling itself** on smaller or simpler instances of the same problem, until it reaches a case simple enough to be answered directly.

**Technical Definition**
In C++, a function is *recursive* if it directly or indirectly calls itself. Each call creates a new **activation record** (stack frame) containing the function’s parameters, local variables, and return address. The recursive process terminates when a **base case** is reached; the **recursive case** reduces the problem toward the base case. Recursion is closely related to mathematical induction: the base case establishes the truth, and the recursive step builds the general solution from smaller solutions.

**Beginner-Friendly Explanation**
Recursion is when a function **does part of a job and then asks itself to do the rest**. For example, to compute `5!`, you compute `5 × 4!`, then `4 × 3!`, and so on, until you reach `1! = 1`, which is simple. Each self-call works on a smaller piece of the problem, and eventually the problem is so small that the answer is obvious. Without that stopping point, the function would call itself forever.

### Key Characteristics

- **Self-reference**: The function’s body contains a call to itself.
- **Base case required**: At least one condition must stop the recursion.
- **Progress toward termination**: Each recursive call must move closer to the base case.
- **Stack usage**: Every recursive call consumes stack space; deep recursion risks **stack overflow**.
- **Elegance vs. cost**: Recursion often mirrors the mathematical definition of a problem, but may be slower and more memory-intensive than iteration.
- **Tail recursion**: A special form that compilers can often optimize into a loop.

### Prerequisites

- Understanding of function calls, parameters, and return values.
- Knowledge of the call stack and stack frames.
- Familiarity with conditional statements and loops.
- Basic awareness of algorithmic complexity and performance.

### Related Programming Areas

- **Divide-and-conquer algorithms** – quicksort, mergesort, binary search.
- **Tree and graph traversal** – depth-first search, expression trees.
- **Dynamic programming** – memoization of recursive solutions.
- **Functional programming** – recursion as the primary control structure.
- **Template metaprogramming** – recursive template instantiations.

### Core Concepts / Features

1. Recursion
2. Base Case
3. Recursive Case
4. Tail Recursion
5. Tail Call Optimization
6. Recursion Depth
7. Stack Overflow
8. Recursion vs. Iteration
9. Recursion Best Practices

---

## 1. Recursion

### Definitions

**Core Definition**
Recursion is a technique where a function calls itself to solve a problem by breaking it into smaller instances of the same problem.

**Technical Definition**
A function is recursive if there is a path of calls from the function back to itself, either directly (the function’s body contains a call to itself) or indirectly (through one or more other functions). Each recursive call creates a new stack frame; the recursion terminates when a base case is reached. The recursive case must reduce the problem size so that the base case is eventually hit.

**Beginner-Friendly Explanation**
Recursion is a function **using itself as a helper**. It is like Russian nesting dolls: you open one doll and find a smaller doll inside, and you keep opening dolls until you reach the smallest one. The smallest doll is the base case; opening each larger doll is the recursive case.

### Purposes

- To express algorithms that naturally decompose into smaller subproblems.
- To simplify code for tree and graph traversal.
- To implement divide-and-conquer strategies.
- To provide elegant solutions for mathematical definitions (factorial, Fibonacci, Ackermann).
- To traverse recursive data structures (linked lists, trees).

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type function_name(parameters) {
    if (base_case_condition) {
        return base_case_value;          // base case
    }
    // ...
    return function_name(smaller_input); // recursive case
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Base case | A condition that returns a value without further recursion. |
| Recursive case | A call to the function with a reduced input. |
| Progress | The input must move toward the base case. |

**Syntax Rules**
- A recursive function must have at least one base case that does not recurse.
- Each recursive call must make progress toward the base case.
- The function must have a return type compatible with all return statements.
- The recursion may be direct (self-call) or indirect (mutual recursion).

**Constraints and Limitations**
- Deep recursion can exhaust the stack (see §7).
- The C++ standard does not guarantee tail call optimization (see §5).
- Recursive functions may be slower than iterative equivalents due to call overhead.
- Infinite recursion (no base case or no progress) results in stack overflow.

### Annotated Code Examples

**Example 1: Factorial (direct recursion)**

```cpp
#include <iostream>

int factorial(int n) {
    if (n <= 1) return 1;              // base case
    return n * factorial(n - 1);       // recursive case
}

int main() {
    std::cout << factorial(5) << '\n';
}
```

**Expected Output**
```
120
```

**Why this output?**  
`factorial(5)` expands to `5 * factorial(4)`, which expands to `5 * 4 * factorial(3)`, and so on until `factorial(1)` returns `1`. The chain of multiplications yields `120`.

**Example 2: Fibonacci (naive recursion)**

```cpp
#include <iostream>

int fib(int n) {
    if (n <= 1) return n;              // base cases: fib(0)=0, fib(1)=1
    return fib(n - 1) + fib(n - 2);    // recursive case
}

int main() {
    std::cout << fib(6) << '\n';
}
```

**Expected Output**
```
8
```

**Why this output?**  
`fib(6)` recursively computes `fib(5) + fib(4)`, each of which recursively computes smaller Fibonacci numbers until the base cases. The result is the 6th Fibonacci number (`0, 1, 1, 2, 3, 5, 8`).

### Real-World Cases

- **File system traversal** – recursively visiting directories and subdirectories.
- **Expression parsing** – recursive descent parsers.
- **Tree algorithms** – in-order, pre-order, and post-order traversal.
- **Backtracking** – solving mazes, N-queens, and constraint satisfaction problems.

---

## 2. Base Case

### Definitions

**Core Definition**
A base case is the condition in a recursive function that **stops the recursion** by returning a result without making another recursive call.

**Technical Definition**
The base case (also called the *terminating case*) is a branch of the recursive function that is reached when the input has been reduced to a size or form that can be solved directly. Without a base case, the recursion would continue indefinitely, eventually causing stack overflow (undefined behavior).

**Beginner-Friendly Explanation**
The base case is the **“enough is enough”** rule. It tells the function: “When the problem is this small, just give the answer directly — don’t call yourself again.” Every recursive function must have at least one base case.

### Purposes

- To terminate the recursion.
- To define the simplest instance of the problem.
- To prevent infinite recursion and stack overflow.
- To establish the correctness of the recursive definition (analogous to the base case of induction).

### Syntax Rules and Structure

**General Syntax**
```cpp
if (base_condition) {
    return base_value;   // no recursive call
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `base_condition` | A condition that identifies the simplest case. |
| `base_value` | The result for that case. |

**Syntax Rules**
- The base case must be reachable from every possible input.
- The base case must not make a recursive call.
- There may be multiple base cases (e.g., `fib(0)` and `fib(1)`).

**Constraints and Limitations**
- A missing or unreachable base case causes infinite recursion.
- A base case that is too broad may produce incorrect results for other inputs.
- A base case that is too narrow may cause unnecessary recursion.

### Annotated Code Examples

**Example 1: Single base case**

```cpp
int factorial(int n) {
    if (n <= 1) return 1;          // base case for n = 0 and n = 1
    return n * factorial(n - 1);
}
```

**Expected Output**  
(`factorial(0)` and `factorial(1)` both return `1`.)

**Why this base case?**  
The factorial of `0` and `1` is defined as `1`. This base case stops the recursion when `n` reaches `0` or `1`.

**Example 2: Multiple base cases**

```cpp
int fib(int n) {
    if (n == 0) return 0;          // base case 1
    if (n == 1) return 1;          // base case 2
    return fib(n - 1) + fib(n - 2);
}
```

**Expected Output**  
(`fib(0)` returns `0`; `fib(1)` returns `1`.)

**Why multiple base cases?**  
The Fibonacci sequence is defined with two initial values. Both must be handled directly to stop the recursion.

### Real-World Cases

- **Tree traversal** – the base case is a null child pointer.
- **Linked list processing** – the base case is reaching the end of the list (`nullptr`).
- **Binary search** – the base case is an empty range (low > high).

---

## 3. Recursive Case

### Definitions

**Core Definition**
The recursive case is the part of a recursive function that **calls the function again** on a smaller or simpler input, moving toward the base case.

**Technical Definition**
The recursive case (or *recursive step*) is the branch that performs the self-call. It must reduce the problem size according to a well-founded measure (e.g., decreasing an integer, shortening a list, narrowing a range) so that the base case is eventually reached.

**Beginner-Friendly Explanation**
The recursive case is the **“do part of the work and pass the rest along”** step. It is the function’s way of saying: “I’ve handled this piece; let me handle the smaller remaining piece the same way.”

### Purposes

- To decompose the problem into smaller instances.
- To combine the results of smaller solutions into the final answer.
- To express the core logic of divide-and-conquer algorithms.
- To move the input toward the base case.

### Syntax Rules and Structure

**General Syntax**
```cpp
return combine(function_name(smaller_input), other_data);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `function_name(smaller_input)` | The recursive call. |
| `combine` | An operation that builds the result from the recursive result. |

**Syntax Rules**
- The recursive call must use a strictly smaller input (by some well-founded measure).
- The result of the recursive call must be combined with other data to form the final result.
- The recursive case must be reachable from the initial call.

**Constraints and Limitations**
- If the input does not shrink, the recursion is infinite.
- Multiple recursive calls (e.g., in Fibonacci) can lead to exponential time complexity.
- The combining step must be correct for all valid inputs.

### Annotated Code Examples

**Example 1: Factorial recursive case**

```cpp
return n * factorial(n - 1);   // combine: multiply n by the result of (n-1)!
```

**Expected Output**  
(For `factorial(5)`, the recursive case chains multiplications.)

**Why this recursive case?**  
`factorial(n) = n * factorial(n - 1)`. The input decreases by `1` each time, guaranteeing progress toward the base case.

**Example 2: Sum of an array (recursive case)**

```cpp
int sum(const int* arr, int n) {
    if (n == 0) return 0;               // base case
    return arr[0] + sum(arr + 1, n - 1); // recursive case
}
```

**Expected Output**  
(`sum({1,2,3,4}, 4)` returns `10`.)

**Why this recursive case?**  
The function adds the first element to the sum of the remaining `n-1` elements. The array pointer advances and `n` decreases, so the base case is reached.

### Real-World Cases

- **Quicksort** – partition and recursively sort the two subarrays.
- **Mergesort** – recursively sort halves and merge.
- **Tree search** – recursively search left and right subtrees.
- **String reversal** – recursively reverse the substring after the first character.

---

## 4. Tail Recursion

### Definitions

**Core Definition**
Tail recursion is a special form of recursion in which the recursive call is the **last operation** performed by the function — its result is returned directly without any further computation.

**Technical Definition**
A function is *tail-recursive* if its recursive call is in **tail position**: the value returned by the recursive call is immediately returned by the calling function, with no pending operations (such as multiplication or addition) after the call returns. In C++, a return statement of the form `return f(args);` where `f` is the same function is a tail call. Tail recursion is significant because compilers can often optimize it into a loop (see §5).

**Beginner-Friendly Explanation**
Tail recursion is recursion where the function’s **last action** is to call itself. There is nothing left to do after the call returns — no multiplication, no addition, no cleanup. Because of this, the compiler can sometimes turn the recursion into a simple loop, saving stack space.

### Purposes

- To enable tail call optimization and avoid stack growth.
- To express iterative algorithms recursively without stack overflow.
- To make recursive code as efficient as a loop.
- To support functional-style recursion in C++.

### Syntax Rules and Structure

**General Syntax**
```cpp
return_type f(parameters) {
    if (base_case) return base_value;
    // ... any work ...
    return f(smaller_input);   // tail call: nothing after this
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `return f(...)` | The recursive call is the returned expression. |
| No pending work | No operations are performed after the call returns. |

**Syntax Rules**
- The recursive call must be the **entire** return expression.
- No arithmetic, logic, or other operations may be applied to the result of the call.
- The function may still have local variables and other statements before the tail call.

**Constraints and Limitations**
- Many natural recursions are **not** tail-recursive (e.g., `n * factorial(n-1)` is not tail-recursive because of the pending multiplication).
- Tail recursion must be converted to an accumulator-passing style to become tail-recursive.
- The C++ standard does **not** require compilers to optimize tail calls (see §5).

### Annotated Code Examples

**Example 1: Non-tail-recursive factorial**

```cpp
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);   // NOT tail-recursive: multiplication pending
}
```

**Expected Output**  
(Correct result, but not tail-recursive.)

**Why not tail-recursive?**  
After `factorial(n-1)` returns, the function must multiply the result by `n`. The call is not the last operation.

**Example 2: Tail-recursive factorial (accumulator style)**

```cpp
int factorial_helper(int n, int acc) {
    if (n <= 1) return acc;
    return factorial_helper(n - 1, n * acc);   // tail call
}

int factorial(int n) {
    return factorial_helper(n, 1);
}
```

**Expected Output**  
(`factorial(5)` returns `120`.)

**Why tail-recursive?**  
The multiplication is performed **before** the call and passed as the accumulator `acc`. The recursive call is the final operation; nothing remains after it returns. A compiler that performs tail call optimization can turn this into a loop.

### Real-World Cases

- **State machines** – tail-recursive transition functions.
- **List processing** – tail-recursive `sum`, `length`, and `reverse` with accumulators.
- **Interpreter loops** – tail-recursive instruction dispatch.
- **Functional programming in C++** – expressing loops as tail-recursive functions.

---

## 5. Tail Call Optimization

### Definitions

**Core Definition**
Tail call optimization (TCO) is a compiler optimization that replaces a tail-recursive call with a **jump**, reusing the current stack frame instead of allocating a new one.

**Technical Definition**
When a compiler performs TCO on a tail call, it deallocates the caller’s stack frame (or reuses it) before transferring control to the callee, so the callee effectively “becomes” the caller. This means a tail-recursive function can run in **constant stack space**, equivalent to an iterative loop. TCO is **not mandated** by the C++ standard; whether it is performed depends on the compiler, optimization level, and specific call pattern.

**Beginner-Friendly Explanation**
Tail call optimization is the compiler’s trick of turning a tail-recursive call into a **loop**. Instead of stacking up a new layer every time the function calls itself, the compiler reuses the same layer. This means a tail-recursive function can run for millions of iterations without running out of stack space — just like a `for` loop.

### Purposes

- To eliminate stack growth for tail-recursive functions.
- To make tail recursion as memory-efficient as iteration.
- To enable long-running recursive loops without stack overflow.
- To support functional-style algorithms in C++.

### Syntax Rules and Structure

TCO is **not written explicitly**; it is a compiler optimization. However, compilers provide attributes to **require** it.

**Standard/Attribute-Assisted Syntax (non-portable)**
```cpp
[[gnu::musttail]] return f(args);     // GCC attribute
[[clang::musttail]] return f(args);   // Clang attribute
[[msvc::musttail]] return f(args);    // MSVC attribute (C++23)
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[[...::musttail]]` | Vendor-specific attribute requesting guaranteed TCO. |
| `return f(args);` | A tail call. |

**Syntax Rules**
- TCO is performed at the compiler’s discretion; there is no standard C++ syntax to guarantee it.
- The `musttail` attributes (GCC, Clang, MSVC) **require** the compiler to emit a tail call or produce an error.
- TCO typically requires optimization to be enabled (`-O2` or higher on GCC/Clang).
- The `-foptimize-sibling-calls` flag controls tail-call optimization in GCC.

**Constraints and Limitations**
- The C++ standard **does not mandate TCO**; portable code must not rely on it.
- TCO may be inhibited by destructors, exception-handling constructs, or large stack objects.
- Debug builds (without optimization) typically do **not** perform TCO, which can cause stack overflow in code that works in optimized builds.
- Some platforms (e.g., older WebAssembly) may not support tail calls.

### Annotated Code Examples

**Example 1: Tail-recursive function with `musttail`**

```cpp
#include <iostream>

[[gnu::musttail]] int count_down(int n) {
    if (n == 0) return 0;
    return count_down(n - 1);   // guaranteed tail call with the attribute
}

int main() {
    std::cout << count_down(1000000) << '\n';
}
```

**Expected Output**
```
0
```

**Why this works?**  
The `[[gnu::musttail]]` attribute instructs GCC to generate a tail call or error. The recursion runs in constant stack space, so `1,000,000` iterations do not overflow the stack.

**Example 2: Without TCO, deep recursion overflows**

```cpp
// Without optimization or TCO:
int count_down(int n) {
    if (n == 0) return 0;
    return count_down(n - 1);
}

int main() {
    count_down(1000000);   // may cause stack overflow
}
```

**Expected Output**  
(Possible stack overflow / crash.)

**Why this may fail?**  
Without TCO, each call allocates a new stack frame. A million frames can exhaust the stack. This illustrates why relying on TCO without guarantees is dangerous.

### Real-World Cases

- **Compilers** – GCC and Clang perform TCO at `-O2`/`-O3` and provide `musttail` attributes.
- **Interpreter loops** – tail-recursive dispatch avoids stack growth.
- **Functional programming** – tail recursion is the idiomatic replacement for loops.
- **Embedded systems** – TCO can reduce stack usage in constrained environments.

---

## 6. Recursion Depth

### Definitions

**Core Definition**
Recursion depth is the **number of nested recursive calls** active at any point during the execution of a recursive function.

**Technical Definition**
The recursion depth is the height of the call stack attributable to recursive calls. Each recursive call increases the depth by one; returning from a call decreases it by one. The maximum recursion depth is the largest depth reached during execution. Exceeding the available stack space causes stack overflow (see §7). For `constexpr` evaluation, compilers impose an implementation-defined recursion depth limit (e.g., Clang’s `-fconstexpr-depth` defaults to 512).

**Beginner-Friendly Explanation**
Recursion depth is **how deep the nesting goes**. If `f` calls `f`, which calls `f`, the depth is 3 at the third call. The deeper the recursion, the more stack space is used. Every recursive algorithm has a maximum depth determined by the input size and the structure of the recursion.

### Purposes

- To measure and reason about stack usage.
- To determine whether an input is safe for recursive processing.
- To compare recursive and iterative memory requirements.
- To set appropriate stack sizes for threads (e.g., `std::thread::stack_size_hint`).

### Syntax Rules and Structure

Recursion depth is not written in code; it is a runtime property. However, it can be **instrumented** by passing a depth parameter.

**Instrumented Syntax**
```cpp
return_type f(parameters, int depth = 0) {
    // optionally check depth
    return f(smaller_input, depth + 1);
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `depth` | An explicit counter tracking the current depth. |

**Syntax Rules**
- The recursion depth is determined by the algorithm’s structure.
- Linear recursion (one call per level) has depth proportional to the input size.
- Tree recursion (multiple calls per level) may have depth proportional to the tree height.
- The available stack size depends on the platform and thread configuration.

**Constraints and Limitations**
- The C++ standard does not specify a maximum recursion depth; it is implementation-dependent.
- Deep recursion without TCO risks stack overflow.
- `constexpr` recursion has implementation-defined limits (e.g., 512 in Clang).

### Annotated Code Examples

**Example 1: Linear recursion depth**

```cpp
int sum_to(int n) {
    if (n == 0) return 0;
    return n + sum_to(n - 1);   // depth = n
}

int main() {
    sum_to(1000);   // recursion depth 1000
}
```

**Expected Output**  
(`sum_to(1000)` returns `500500`.)

**Why depth 1000?**  
Each call reduces `n` by 1, so the call chain has `n + 1` frames (including the base case). The maximum depth is `1001`.

**Example 2: Tree recursion depth (binary tree height)**

```cpp
int height(Node* node) {
    if (!node) return 0;
    return 1 + std::max(height(node->left), height(node->right));
}
```

**Expected Output**  
(Returns the height of the tree.)

**Why tree height?**  
The recursion depth follows the longest path from the root to a leaf. A balanced tree of `n` nodes has depth `O(log n)`; a degenerate tree has depth `O(n)`.

### Real-World Cases

- **Linked lists** – recursion depth equals list length.
- **Binary trees** – recursion depth equals tree height.
- **Quicksort** – worst-case depth `O(n)` (already-sorted input with naive pivot).
- **`constexpr` evaluation** – depth limited by compiler settings.

---

## 7. Stack Overflow

### Definitions

**Core Definition**
Stack overflow is a runtime error that occurs when a program’s call stack exceeds its allocated size, typically due to excessively deep or infinite recursion.

**Technical Definition**
Each function call allocates a stack frame containing parameters, local variables, and return address. The stack has a finite size (platform-dependent, often 1–8 MB for the main thread). When the stack pointer exceeds the stack limit, the operating system raises a **stack overflow** (often manifesting as a segmentation fault or access violation). In C++, stack overflow is **undefined behavior**. `constexpr` evaluation exceeding the compile-time recursion depth limit results in a compile-time error.

**Beginner-Friendly Explanation**
Stack overflow is what happens when the **call stack runs out of room**. Every time a function calls itself, it adds a new layer to the stack. If there are too many layers — because the recursion is too deep or infinite — the stack overflows and the program crashes. It is like a stack of plates that gets too tall and topples over.

### Purposes

- (This is a failure mode, not a feature.) Understanding stack overflow helps prevent crashes.
- To motivate the use of base cases, iteration, or tail call optimization.
- To guide decisions about stack size and recursion depth limits.
- To explain why deep recursion is dangerous in production code.

### Syntax Rules and Structure (Prevention and Diagnosis)

**Prevention Techniques**

| Technique | Description |
|-----------|-------------|
| Ensure a base case | Every recursion must terminate. |
| Ensure progress | Each call must reduce the problem size. |
| Limit recursion depth | Add an explicit depth check. |
| Use iteration | Replace recursion with a loop. |
| Use tail recursion + TCO | Constant stack space for tail calls. |
| Increase stack size | Platform-specific; not a general solution. |

**Syntax Example: Depth Check**
```cpp
int safe_recursion(int n, int depth = 0) {
    if (depth > 10000) throw std::runtime_error("recursion too deep");
    if (n == 0) return 0;
    return safe_recursion(n - 1, depth + 1);
}
```

**Constraints and Limitations**
- Stack overflow cannot be caught as a C++ exception in a portable way; it typically terminates the process.
- The available stack size is platform-dependent and may be smaller for threads.
- Debug builds may use more stack per frame than optimized builds, making overflow more likely.

### Annotated Code Examples

**Example 1: Infinite recursion causes stack overflow**

```cpp
// DO NOT RUN
int infinite(int n) {
    return infinite(n + 1);   // no base case: infinite recursion
}

int main() {
    infinite(0);   // stack overflow (undefined behavior)
}
```

**Expected Output**  
(Program crashes with stack overflow / segmentation fault.)

**Why this crashes?**  
There is no base case, so each call adds a frame until the stack is exhausted.

**Example 2: Deep but finite recursion may overflow**

```cpp
int sum_to(int n) {
    if (n == 0) return 0;
    return n + sum_to(n - 1);
}

int main() {
    sum_to(10000000);   // depth 10,000,000: likely stack overflow
}
```

**Expected Output**  
(Possible stack overflow, depending on stack size.)

**Why this may overflow?**  
Even though the recursion is finite, the depth (`10,000,000`) may exceed the available stack space. Without TCO, each frame consumes stack.

### Real-World Cases

- **Accidental infinite recursion** – missing or incorrect base case.
- **Deep tree traversal** – a degenerate tree with `O(n)` height.
- **Naive Fibonacci** – exponential call tree can exhaust time and stack for larger `n`.
- **Production crashes** – MISRA and other safety standards restrict recursion due to unpredictable stack usage.

---

## 8. Recursion vs. Iteration

### Definitions

**Core Definition**
Recursion and iteration are two fundamental ways to repeat computation: recursion expresses repetition through **self-calls**, while iteration uses **loops** (`for`, `while`, `do-while`).

**Technical Definition**
Recursion solves a problem by reducing it to smaller subproblems and combining their results; iteration solves a problem by repeating a block of statements until a condition is met, maintaining state in variables. Recursion uses the call stack for state; iteration uses local variables and loop counters. Any recursive algorithm can be transformed into an iterative one (and vice versa), but the transformation may require an explicit stack or accumulator.

**Beginner-Friendly Explanation**
Recursion is like **asking someone else to do the smaller job**; iteration is like **doing the same job over and over yourself** with a checklist. Recursion can be more elegant and closer to the mathematical definition; iteration is often faster and uses less memory.

### Purposes

- To choose the right tool for a given problem.
- To understand the trade-offs between elegance, performance, and memory.
- To recognize when recursion should be converted to iteration.
- To justify design decisions in code reviews.

### Comparison Table

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| **Expressiveness** | Natural for divide-and-conquer and recursive structures. | Natural for linear repetitive tasks. |
| **Stack usage** | Grows with depth; risk of stack overflow. | Constant (or explicit stack) space. |
| **Performance** | Function-call overhead; often slower. | Typically faster (no call overhead). |
| **Memory** | Uses call stack. | Uses loop variables and explicit state. |
| **Readability** | Can be clearer for recursive definitions. | Can be clearer for simple loops. |
| **Tail recursion** | Can be as efficient as iteration with TCO. | Inherently loop-based. |
| **Debugging** | Call stack shows recursive structure. | Easier to step through linearly. |
| **Risk** | Infinite recursion → stack overflow. | Infinite loop → CPU spin (but no stack growth). |

### Annotated Code Examples

**Example 1: Recursive vs. iterative factorial**

```cpp
// Recursive
int factorial_rec(int n) {
    if (n <= 1) return 1;
    return n * factorial_rec(n - 1);
}

// Iterative
int factorial_iter(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) result *= i;
    return result;
}
```

**Expected Output**  
(Both return `120` for `n = 5`.)

**Why compare?**  
The iterative version avoids stack growth and function-call overhead. The recursive version is closer to the mathematical definition `n! = n × (n-1)!`.

**Example 2: Recursive vs. iterative tree traversal**

```cpp
// Recursive in-order traversal
void inorder(Node* node) {
    if (!node) return;
    inorder(node->left);
    std::cout << node->value << ' ';
    inorder(node->right);
}
```

**Expected Output**  
(Prints the in-order traversal of the tree.)

**Why recursion here?**  
Tree traversal is naturally recursive; an iterative version requires an explicit stack and is more complex. This is a case where recursion is often preferred for clarity.

### Real-World Cases

- **Quicksort** – commonly recursive for its natural decomposition; iterative variants use an explicit stack.
- **Linked lists** – iterative traversal is typically preferred to avoid `O(n)` stack depth.
- **Binary search** – both recursive and iterative versions are common; the iterative version is slightly more efficient.
- **Compile-time computation** – C++11 `constexpr` required recursion; C++14 allowed loops, offering a choice.

---

## 9. Recursion Best Practices

### Definitions

**Core Definition**
Recursion best practices are guidelines for writing recursive functions that are correct, safe, efficient, and maintainable.

**Technical Definition**
These practices address termination (base case), progress (recursive case), stack safety (depth and TCO), performance (avoiding redundant computation), and the choice between recursion and iteration. They are informed by safety standards such as MISRA C++:2023 Rule 8.2.10, which restricts recursion due to uncontrolled stack usage.

**Beginner-Friendly Explanation**
Best practices for recursion are simple: **always have a base case**, **make sure every call gets closer to it**, **don’t recurse too deeply**, **consider whether a loop would be simpler**, and **use tail recursion when you can**. Recursion is powerful, but it must be used carefully.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **Always have a base case** | Every recursion must terminate. | Write the base case first. |
| **Ensure progress** | Each call must reduce the problem. | Use a well-founded measure. |
| **Limit depth** | Deep recursion risks stack overflow. | Add depth checks or prefer iteration for deep inputs. |
| **Prefer iteration for linear problems** | Loops are simpler and safer for simple repetition. | Use recursion for recursive structures (trees, divide-and-conquer). |
| **Use tail recursion + TCO** | Tail calls can run in constant stack space. | Convert to accumulator style and enable optimization. |
| **Avoid redundant computation** | Naive recursion can be exponential. | Use memoization or dynamic programming. |
| **Document the recursion** | Explain the base case and recursive step. | Comment the termination condition and the measure. |
| **Consider safety standards** | MISRA restricts recursion. | Use recursion only when justified and controlled. |

### Annotated Code Examples

**Example 1: Correct base case and progress**

```cpp
int gcd(int a, int b) {
    if (b == 0) return a;          // base case
    return gcd(b, a % b);          // progress: b decreases
}
```

**Expected Output**  
(`gcd(48, 18)` returns `6`.)

**Why this is best practice?**  
The base case (`b == 0`) is clear, and the recursive case (`gcd(b, a % b)`) strictly reduces `b`, guaranteeing termination.

**Example 2: Tail recursion with accumulator**

```cpp
int sum_helper(int n, int acc) {
    if (n == 0) return acc;        // base case
    return sum_helper(n - 1, acc + n);  // tail call
}

int sum(int n) { return sum_helper(n, 0); }
```

**Expected Output**  
(`sum(1000)` returns `500500`.)

**Why this is best practice?**  
The accumulator makes the recursion tail-recursive, enabling TCO and avoiding stack growth for large `n`.

**Example 3: Avoiding redundant computation with memoization**

```cpp
#include <unordered_map>

std::unordered_map<int, long long> memo;

long long fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];
    return memo[n] = fib(n - 1) + fib(n - 2);
}
```

**Expected Output**  
(`fib(40)` returns quickly; naive recursion would be very slow.)

**Why this is best practice?**  
Memoization caches previously computed results, turning an exponential-time recursion into a linear-time one.

### Real-World Cases

- **Safety-critical systems** – MISRA C++ restricts recursion; iterative solutions are preferred.
- **Competitive programming** – recursion with memoization (dynamic programming).
- **Parser combinators** – recursive descent is natural but must be depth-controlled.
- **Functional programming in C++** – tail recursion and accumulators replace loops.

---

## References

- Recursion (computer science) – https://en.wikipedia.org/wiki/Recursion_(computer_science)
- Function declaration (cppreference) – https://en.cppreference.com/w/cpp/language/function
- `constexpr` specifier (cppreference) – https://en.cppreference.com/w/cpp/language/constexpr
- Constant expressions — recursion depth limit (cppreference) – https://en.cppreference.com/w/cpp/language/constant_expression
- MISRA C++:2023 Rule 8.2.10 — Functions shall not call themselves, either directly or indirectly – https://rules.sonarsource.com/c/m23_145/
- GCC `musttail` statement attribute – https://gcc.gnu.org/onlinedocs/gcc-15.1.0/gcc/Statement-Attributes.html
- D3939R0: Tail recursion enforcement (isocpp.org) – https://isocpp.org/files/papers/D3939R0.html
- GCC `-foptimize-sibling-calls` (tail-call optimization) – https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html
- Clang `musttail` attribute documentation – https://clang.llvm.org/docs/AttributeReference.html#musttail
- C++ Tail Recursion Using 64-bit Variables (isocpp.org) – https://isocpp.org/blog/2015/03/cpp-tail-recursion-using-64-bit-variables-giovanni-campo

---

This cheat sheet provides a complete, structured reference for C++ recursion, covering the recursive mechanism, base and recursive cases, tail recursion, tail call optimization, recursion depth, stack overflow, recursion versus iteration, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.