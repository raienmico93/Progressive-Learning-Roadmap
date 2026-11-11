# Loops — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A loop is a control-flow construct that **repeatedly executes a block of statements** until a termination condition is met, enabling iteration over sequences, repetition of computations, and processing of collections.

**Technical Definition**
C++ provides several iteration statements: the **`for`** loop (initialization, condition, iteration expression), the **range-based `for`** loop (C++11, iterating over a range), the **`while`** loop (condition checked before each iteration), and the **`do`–`while`** loop (condition checked after each iteration). Loop control statements (`break`, `continue`, `goto`) alter the flow of iteration. Loops may be nested, may be infinite (by design or by bug), and may be optimized by the compiler (loop unrolling, vectorization, invariant hoisting). The C++ Core Guidelines and community standards provide rules for writing clear, safe, and efficient loops (ES.71–ES.86, ES.100–ES.107).

**Beginner-Friendly Explanation**
A loop is a way to **repeat a block of code**. Instead of writing the same instruction ten times, you write a loop that says “do this ten times.” C++ gives you four main loop forms: `for` (when you know the bounds and have an initialization/condition/increment pattern), range-based `for` (when you want to walk through a collection), `while` (when you repeat as long as a condition holds), and `do`–`while` (when you want to run at least once). You can control loops with `break` (leave early), `continue` (skip to the next iteration), and even `goto` (jump to a label, usually best avoided). Loops are the backbone of iteration in programming.

### Key Characteristics

- **Repetition**: Loops execute a body repeatedly until a condition changes.
- **Four forms**: `for`, range-based `for`, `while`, `do`–`while`.
- **Loop control**: `break` exits the loop; `continue` skips to the next iteration; `goto` jumps to a label.
- **Infinite loops**: Loops without a terminating condition (sometimes intentional, e.g., event loops).
- **Nested loops**: Loops inside loops (e.g., matrix traversal).
- **Optimization**: Compilers can unroll, vectorize, and hoist loop invariants.
- **Scope**: Loop variables can be scoped to the loop (C++17 `if`/`switch` initializers; `for` initializers).
- **Safety**: Common bugs include off-by-one errors, infinite loops, and modifying the loop variable in the body.

### Prerequisites

- Understanding of expressions, conditions, and boolean logic.
- Familiarity with blocks, scope, and control flow.
- Knowledge of arrays, containers, and iterators (for range-based `for`).
- Basic awareness of compiler optimizations.

### Related Programming Areas

- **Iteration** – processing sequences and collections.
- **Algorithms** – `std::for_each`, `std::transform`, `std::accumulate`.
- **Containers** – range-based `for` over `std::vector`, `std::map`, etc.
- **Control flow** – `break`, `continue`, `goto`.
- **Performance** – loop unrolling, vectorization, cache efficiency.
- **Concurrency** – loop parallelism (`std::for_each` with execution policies).

### Core Concepts / Features

1. `for`
2. Range-Based `for`
3. `while`
4. `do`–`while`
5. Loop Control (`break`, `continue`, `goto`)
6. Infinite Loops
7. Nested Loops
8. Loop Optimization
9. Loop Best Practices

---

## 1. `for`

### Definitions

**Core Definition**
The `for` loop is an iteration statement with an **initialization**, a **condition**, and an **iteration expression**, executing its body while the condition is true.

**Technical Definition**
The `for` statement has the form `for ( init-statement condition ; expression ) statement`. The initialization runs once; the condition is checked before each iteration; the body runs if the condition is true; the iteration expression runs after each iteration. Variables declared in the initializer are scoped to the loop. Since C++17, the initializer may include a declaration (e.g., `for (auto it = v.begin(); it != v.end(); ++it)`).

**Beginner-Friendly Explanation**
The `for` loop is the **“counted loop”** : you set up a counter, check a condition, and update the counter each time. It's perfect when you know how many times to repeat or when iterating with an index.

### Purposes

- To repeat a block a known number of times.
- To iterate over arrays and containers with an index.
- To implement counting and accumulation.
- To traverse sequences with iterators.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (init; condition; increment) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `init` | Initialization (declaration or expression). |
| `condition` | Checked before each iteration. |
| `increment` | Executed after each iteration. |
| `body` | The loop body. |

**Syntax Rules**
- The initialization runs once.
- The condition is checked before each iteration; if false, the loop ends.
- The increment runs after each iteration.
- Variables declared in `init` are scoped to the loop.
- Any of the three parts may be empty (`for (;;)` is an infinite loop).

**Constraints and Limitations**
- Modifying the loop variable inside the body can cause bugs.
- Off-by-one errors are common.
- The condition must eventually become false to avoid infinite loops.

### Annotated Code Examples

**Example 1: Basic `for` loop**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 5; ++i) {
        std::cout << i << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 2 3 4 
```

**Why this output?**  
`i` starts at `0`, increments by `1` each iteration, and the loop runs while `i < 5`.

### Real-World Cases

- **Array traversal** – `for (int i = 0; i < n; ++i)`.
- **Container iteration** – `for (auto it = v.begin(); it != v.end(); ++it)`.
- **Counting** – `for (int i = 0; i < n; ++i) sum += i;`.
- **Matrix traversal** – nested `for` loops.

---

## 2. Range-Based `for`

### Definitions

**Core Definition**
The range-based `for` loop (C++11) iterates over all elements of a **range** (array, container, or any type with `begin()`/`end()`), binding each element to a variable.

**Technical Definition**
The range-based `for` statement has the form `for ( for-range-declaration : for-range-initializer ) statement`. It iterates over the range, binding each element to the declared variable. The range must provide `begin()` and `end()` (member or free functions). The variable may be a value, reference, or `const` reference. Since C++17, the range initializer may include an initializer and structured bindings are supported.

**Beginner-Friendly Explanation**
Range-based `for` is the **“for each”** loop. It walks through every element of a collection automatically, without indices or iterators: `for (int x : v) { ... }`.

### Purposes

- To iterate over containers and arrays without explicit indices.
- To improve readability and reduce off-by-one errors.
- To process each element uniformly.
- To support structured bindings for key-value pairs.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (declaration : range) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `declaration` | The loop variable (value, reference, `const` reference). |
| `range` | The range to iterate over. |

**Syntax Rules**
- The range must provide `begin()` and `end()`.
- The declaration may be a value, reference, or `const` reference.
- Since C++17, the range initializer may include an initializer.
- Structured bindings are supported (C++17).

**Constraints and Limitations**
- Cannot easily modify the range's size during iteration.
- Iterating over a temporary range can cause dangling references (C++23 fixes some cases).
- The loop variable is a copy unless declared as a reference.

### Annotated Code Examples

**Example 1: Range-based `for` over a vector**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (int x : v) {
        std::cout << x << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
1 2 3 4 5 
```

**Why this output?**  
The range-based `for` iterates over each element of `v`, binding it to `x`.

**Example 2: Modifying elements via reference**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    for (int& x : v) {
        x *= 2;
    }
    for (int x : v) {
        std::cout << x << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
2 4 6 
```

**Why this output?**  
`int& x` binds a reference to each element, allowing modification.

### Real-World Cases

- **Container processing** – `for (const auto& item : items)`.
- **Key-value pairs** – `for (const auto& [key, value] : map)`.
- **Array traversal** – `for (int x : arr)`.
- **String iteration** – `for (char c : str)`.

---

## 3. `while`

### Definitions

**Core Definition**
The `while` loop executes its body **while a condition remains true**, checking the condition **before** each iteration.

**Technical Definition**
The `while` statement has the form `while ( condition ) statement`. The condition is contextually converted to `bool`; if true, the body executes, and the process repeats. If the condition is false initially, the body never executes. The condition is checked before each iteration (pre-test loop).

**Beginner-Friendly Explanation**
The `while` loop is the **“repeat as long as…”** loop. It checks the condition first; if true, it runs the body and checks again.

### Purposes

- To repeat while a condition holds.
- To process input until a sentinel.
- To implement event loops and state machines.
- To handle unknown iteration counts.

### Syntax Rules and Structure

**General Syntax**
```cpp
while (condition) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `condition` | Checked before each iteration. |
| `body` | Executed if the condition is true. |

**Syntax Rules**
- The condition is checked before the body.
- If false initially, the body never runs.
- The body must eventually make the condition false (or use `break`).

**Constraints and Limitations**
- Forgetting to update the condition causes an infinite loop.
- The loop variable must be in scope.

### Annotated Code Examples

**Example 1: `while` loop**

```cpp
#include <iostream>

int main() {
    int i = 0;
    while (i < 5) {
        std::cout << i << ' ';
        ++i;
    }
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 2 3 4 
```

**Why this output?**  
`i` starts at `0`; the loop runs while `i < 5`, incrementing `i` each iteration.

### Real-World Cases

- **Input processing** – `while (std::cin >> x)`.
- **Event loops** – `while (running) { ... }`.
- **Search** – `while (!found && it != end)`.
- **Unknown iteration counts** – reading until EOF.

---

## 4. `do`–`while`

### Definitions

**Core Definition**
The `do`–`while` loop executes its body **at least once**, then checks the condition **after** each iteration.

**Technical Definition**
The `do`–`while` statement has the form `do statement while ( condition ) ;`. The body executes, then the condition is checked; if true, the body repeats. This is a post-test loop: the body always runs at least once.

**Beginner-Friendly Explanation**
The `do`–`while` loop is the **“do this, then check”** loop. It always runs the body at least once, then decides whether to repeat.

### Purposes

- To execute a body at least once.
- To implement menus and prompts.
- To process input until valid.
- To implement retry loops.

### Syntax Rules and Structure

**General Syntax**
```cpp
do {
    // body
} while (condition);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `body` | Executed at least once. |
| `condition` | Checked after each iteration. |

**Syntax Rules**
- The body always executes at least once.
- The condition is checked after the body.
- The semicolon after `while (condition)` is required.

**Constraints and Limitations**
- Forgetting the semicolon is a common syntax error.
- The condition must eventually become false.

### Annotated Code Examples

**Example 1: `do`–`while` loop**

```cpp
#include <iostream>

int main() {
    int i = 0;
    do {
        std::cout << i << ' ';
        ++i;
    } while (i < 5);
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 2 3 4 
```

**Why this output?**  
The body runs at least once, then repeats while `i < 5`.

### Real-World Cases

- **Menu loops** – show menu, then check input.
- **Input validation** – prompt until valid.
- **Retry logic** – attempt operation, then check success.

---

## 5. Loop Control

### Definitions

**Core Definition**
Loop control statements (`break`, `continue`, `goto`) alter the normal flow of loops.

**Technical Definition**
- **`break`** exits the nearest enclosing loop or `switch`.
- **`continue`** skips the rest of the current iteration and proceeds to the next.
- **`goto`** transfers control to a labeled statement (rarely used; discouraged in most codebases).

**Beginner-Friendly Explanation**
- `break` = **leave the loop now**.
- `continue` = **skip to the next iteration**.
- `goto` = **jump to a label** (powerful but dangerous).

### Purposes

- To exit loops early (`break`).
- To skip iterations (`continue`).
- To jump out of nested loops (`goto` or flags).
- To implement search and early termination.

### Syntax Rules and Structure

**`break`**
```cpp
while (condition) {
    if (done) break;
}
```

**`continue`**
```cpp
for (int i = 0; i < n; ++i) {
    if (i % 2 == 0) continue;
    // process odd i
}
```

**`goto`**
```cpp
for (...) {
    for (...) {
        if (found) goto done;
    }
}
done:
    // ...
```

**Component Breakdown**

| Statement | Effect |
|-----------|--------|
| `break` | Exits nearest loop/switch. |
| `continue` | Skips to next iteration. |
| `goto label` | Jumps to label. |

**Syntax Rules**
- `break` and `continue` apply to the nearest enclosing loop.
- `goto` can jump forward or backward within a function.
- `goto` cannot jump into a scope past initialization.

**Constraints and Limitations**
- `goto` is discouraged (spaghetti code).
- `break`/`continue` in nested loops affect only the innermost loop.
- `continue` in a `do`–`while` jumps to the condition check.

### Annotated Code Examples

**Example 1: `break` and `continue`**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 10; ++i) {
        if (i == 3) continue;   // skip 3
        if (i == 7) break;      // stop at 7
        std::cout << i << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 2 4 5 6 
```

**Why this output?**  
`continue` skips `3`; `break` stops the loop when `i == 7`.

### Real-World Cases

- **Search** – `break` when found.
- **Filtering** – `continue` to skip invalid items.
- **Nested loop exit** – `goto` or flags.

---

## 6. Infinite Loops

### Definitions

**Core Definition**
An infinite loop is a loop whose termination condition never becomes false, so it runs forever (intentionally or by bug).

**Technical Definition**
An infinite loop occurs when the loop condition is always true (e.g., `while (true)`, `for (;;)`) or when the loop variable is never updated to make the condition false. Intentional infinite loops are common in event loops, servers, and embedded systems; they are terminated by `break`, `return`, `throw`, or external signals. Accidental infinite loops are bugs.

**Beginner-Friendly Explanation**
An infinite loop is a loop that **never stops** on its own. Sometimes that's intentional (like a server waiting for requests); sometimes it's a bug (forgot to increment the counter).

### Purposes

- To implement event loops and servers.
- To wait for external events.
- To run until explicitly stopped.

### Syntax Rules and Structure

**Intentional Infinite Loop**
```cpp
while (true) {
    // process event
    if (should_stop) break;
}
```

**Accidental Infinite Loop**
```cpp
int i = 0;
while (i < 10) {
    // forgot ++i
}
```

**Syntax Rules**
- `while (true)`, `for (;;)`, and `do { } while (true)` are intentional infinite loops.
- They must have an exit mechanism (`break`, `return`, `throw`).
- Accidental infinite loops occur when the condition never changes.

**Constraints and Limitations**
- Accidental infinite loops hang the program.
- Intentional infinite loops must be interruptible.
- Some compilers optimize `while (true)` differently.

### Annotated Code Examples

**Example 1: Intentional infinite loop with break**

```cpp
#include <iostream>

int main() {
    int i = 0;
    while (true) {
        if (i >= 5) break;
        std::cout << i << ' ';
        ++i;
    }
    std::cout << '\n';
}
```

**Expected Output**
```
0 1 2 3 4 
```

**Why this output?**  
The loop is infinite, but `break` exits when `i >= 5`.

### Real-World Cases

- **Servers** – accept connections forever.
- **Event loops** – process events indefinitely.
- **Embedded systems** – main loop runs forever.
- **Games** – game loop runs until quit.

---

## 7. Nested Loops

### Definitions

**Core Definition**
Nested loops are loops placed **inside other loops**, with the inner loop executing fully for each iteration of the outer loop.

**Technical Definition**
A nested loop is a loop whose body contains another loop. The inner loop runs to completion for each iteration of the outer loop. The total number of iterations is the product of the loop counts (for independent loops). Nested loops are used for multidimensional traversal, matrix operations, and combinatorial algorithms.

**Beginner-Friendly Explanation**
Nested loops are **loops inside loops**. For each step of the outer loop, the inner loop runs all the way through.

### Purposes

- To traverse multidimensional arrays and matrices.
- To implement combinatorial algorithms.
- To generate patterns.
- To process nested data structures.

### Syntax Rules and Structure

**General Syntax**
```cpp
for (int i = 0; i < rows; ++i) {
    for (int j = 0; j < cols; ++j) {
        // process (i, j)
    }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| Outer loop | Controls the outer dimension. |
| Inner loop | Runs fully for each outer iteration. |

**Syntax Rules**
- The inner loop runs completely for each outer iteration.
- `break`/`continue` affect only the innermost loop.
- To exit multiple levels, use `goto` or flags.

**Constraints and Limitations**
- Deep nesting hurts readability.
- Performance can degrade with many levels.
- Exiting multiple levels requires extra care.

### Annotated Code Examples

**Example 1: Nested loops for a matrix**

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 3; ++i) {
        for (int j = 0; j < 3; ++j) {
            std::cout << '(' << i << ',' << j << ") ";
        }
        std::cout << '\n';
    }
}
```

**Expected Output**
```
(0,0) (0,1) (0,2) 
(1,0) (1,1) (1,2) 
(2,0) (2,1) (2,2) 
```

**Why this output?**  
The outer loop iterates over rows; the inner loop iterates over columns, printing each coordinate.

### Real-World Cases

- **Matrix operations** – multiplication, transposition.
- **Image processing** – pixel traversal.
- **Combinatorics** – generating pairs/triples.
- **Pattern printing** – triangles, grids.

---

## 8. Loop Optimization

### Definitions

**Core Definition**
Loop optimization is the set of compiler techniques that improve loop performance, such as unrolling, vectorization, and invariant hoisting.

**Technical Definition**
Compilers apply loop optimizations such as:
- **Loop unrolling**: replicating the body to reduce loop overhead.
- **Loop invariant code motion**: moving invariant computations outside the loop.
- **Vectorization**: using SIMD instructions to process multiple elements.
- **Strength reduction**: replacing expensive operations with cheaper ones.
- **Loop fusion/fission**: combining or splitting loops for better locality.

These optimizations are enabled by optimization flags (e.g., `-O2`, `-O3`) and can be guided by attributes (`[[likely]]`, `[[unlikely]]`, `#pragma omp simd`).

**Beginner-Friendly Explanation**
Loop optimization is the compiler's way of **making loops faster** — unrolling them, vectorizing them, and moving work out of them. You can help by writing simple, predictable loops.

### Purposes

- To improve loop performance.
- To reduce loop overhead.
- To enable SIMD and parallelism.
- To improve cache locality.

### Syntax Rules and Structure

**Optimization Techniques**

| Technique | Description |
|-----------|-------------|
| Unrolling | Replicate body to reduce overhead. |
| Invariant hoisting | Move invariant code out. |
| Vectorization | SIMD processing. |
| Strength reduction | Cheaper operations. |

**Syntax Rules**
- Enabled by compiler flags (`-O2`, `-O3`).
- Helped by simple, predictable loops.
- Hindered by complex control flow, aliasing, and function calls in the body.
- Can be guided by attributes and pragmas.

**Constraints and Limitations**
- Optimization may change floating-point results (fast-math).
- Aliasing can prevent vectorization.
- Complex loops may not be optimized.

### Annotated Code Examples

**Example 1: Loop unrolling (conceptual)**

```cpp
#include <iostream>

int main() {
    int sum = 0;
    for (int i = 0; i < 4; ++i) {
        sum += i;
    }
    std::cout << sum << '\n';
}
```

**Expected Output**
```
6
```

**Why this is optimizable?**  
The loop is simple and predictable; the compiler may unroll it or vectorize it.

### Real-World Cases

- **Numerical computing** – vectorized loops.
- **Image processing** – SIMD loops.
- **Game loops** – optimized hot paths.
- **Embedded systems** – unrolled loops for performance.

---

## 9. Loop Best Practices

### Definitions

**Core Definition**
Loop best practices are guidelines for writing clear, correct, and efficient loops.

**Technical Definition**
These practices are based on the C++ Core Guidelines (especially **ES.71** on preferring range-based `for`, **ES.72** on preferring `for` when there is an obvious loop variable, **ES.73** on preferring `while` when there is no obvious loop variable, **ES.74** on preferring `for` for initialization/condition/increment, **ES.75** on avoiding `do`-statements, **ES.76** on avoiding `goto`, **ES.77** on avoiding `break`/`continue` in loops, **ES.78** on avoiding implicit fallthrough, and **ES.100–ES.107** on performance) and community standards.

**Beginner-Friendly Explanation**
Best practices for loops are simple: **prefer range-based `for` for collections**, **use `for` for counted loops**, **use `while` for unknown counts**, **avoid `goto`**, **keep loops simple**, **don't modify the loop variable in the body**, and **use `break`/`continue` sparingly**.

### Key Guidelines (Summary Table)

| Guideline | Summary | Recommendation |
|-----------|---------|----------------|
| **ES.71** | Prefer range-based `for` over explicit iterator loops. | `for (const auto& x : v)`. |
| **ES.72** | Prefer `for` when there is an obvious loop variable. | Counted loops. |
| **ES.73** | Prefer `while` when there is no obvious loop variable. | Event loops. |
| **ES.74** | Prefer `for` for initialization, condition, and increment. | Explicit loop control. |
| **ES.75** | Avoid `do`-statements. | Use `while` or `for` when possible. |
| **ES.76** | Avoid `goto`. | Use structured control flow. |
| **ES.77** | Minimize `break`/`continue` in loops. | Prefer conditions. |
| **Simplicity** | Keep loop bodies simple. | Aid optimization. |
| **No modification** | Don't modify the loop variable in the body. | Avoid bugs. |
| **Range safety** | Don't modify the range's size during iteration. | Use indices or collect changes. |

### Annotated Code Examples

**Example 1: Prefer range-based `for` (ES.71)**

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (const auto& x : v) {   // preferred
        std::cout << x << ' ';
    }
    std::cout << '\n';
}
```

**Expected Output**
```
1 2 3 4 5 
```

**Why this is best practice?**  
Range-based `for` is clearer and less error-prone than explicit iterator loops (ES.71).

### Real-World Cases

- **Standard library** – algorithms use range-based loops and iterators.
- **Code review** – loop best practices are common checkpoints.
- **Static analysis** – clang-tidy checks for loop issues.

---

## References

- `for` loop (cppreference) – https://en.cppreference.com/w/cpp/language/for
- Range-based `for` loop (cppreference) – https://en.cppreference.com/w/cpp/language/range-for
- `while` loop (cppreference) – https://en.cppreference.com/w/cpp/language/while
- `do`–`while` loop (cppreference) – https://en.cppreference.com/w/cpp/language/do
- `break` statement (cppreference) – https://en.cppreference.com/w/cpp/language/break
- `continue` statement (cppreference) – https://en.cppreference.com/w/cpp/language/continue
- `goto` statement (cppreference) – https://en.cppreference.com/w/cpp/language/goto
- C++ Core Guidelines — ES.71: Prefer a range-`for`-statement to a `for`-statement when there is a choice – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-for-range
- C++ Core Guidelines — ES.72: Prefer a `for`-statement to a `while`-statement when there is an obvious loop variable – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-for-while
- C++ Core Guidelines — ES.73: Prefer a `while`-statement to a `for`-statement when there is no obvious loop variable – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-while-for
- C++ Core Guidelines — ES.74: Prefer to declare a loop variable in the initializer part of a `for`-statement – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-for-init
- C++ Core Guidelines — ES.75: Avoid `do`-statements – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-do
- C++ Core Guidelines — ES.76: Avoid `goto` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-goto
- C++ Core Guidelines — ES.77: Minimize the use of `break` and `continue` in loops – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-break
- C++ Core Guidelines — ES.100: Don't mix signed and unsigned arithmetic – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-mix
- C++ Core Guidelines — ES.107: Don't use `unsigned` for subscripting, use `gsl::index` – https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Res-subscripts
- Microsoft Learn — `for` statement (C++) – https://learn.microsoft.com/en-us/cpp/cpp/for-statement-cpp
- Microsoft Learn — Range-based `for` statement – https://learn.microsoft.com/en-us/cpp/cpp/range-based-for-statement-cpp
- Microsoft Learn — `while` statement – https://learn.microsoft.com/en-us/cpp/cpp/while-statement-cpp
- Microsoft Learn — `do`–`while` statement – https://learn.microsoft.com/en-us/cpp/cpp/do-while-statement-cpp

---

This cheat sheet provides a complete, structured reference for C++ loops, covering `for`, range-based `for`, `while`, `do`–`while`, loop control (`break`, `continue`, `goto`), infinite loops, nested loops, loop optimization, and best practices. Each section follows a uniform template with definitions, purposes, syntax rules, annotated examples, expected outputs, and authoritative references.