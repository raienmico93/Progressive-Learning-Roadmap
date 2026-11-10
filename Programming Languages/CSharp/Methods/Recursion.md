# C# Recursion — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Recursion is a programming technique in which a method **calls itself** to solve a problem by breaking it down into smaller instances of the same problem. Every recursive method must have at least one **base case** that terminates the recursion and one or more **recursive cases** that move the problem strictly closer to the base case. C# supports recursion fully: any method, local function, or lambda (via a delegate) can call itself. Recursion is the natural implementation technique for problems whose structure is itself recursive, such as traversing trees, parsing nested syntax, and divide-and-conquer algorithms.

**Technical Definition**
In C#, recursion occurs when a method invokes itself directly or indirectly (mutual recursion). Each invocation creates a new **stack frame** containing the method’s parameters, local variables, and return address; the call stack grows with each recursive call and shrinks as calls return. The C# compiler does **not** perform tail-call optimisation: it emits ordinary recursive calls in IL, and whether the JIT honours a tail call depends on the runtime, platform, and method structure. Consequently, recursion depth in C# is bounded by the **thread’s stack size** (typically 1 MB for the default .NET stack), and exceeding that bound throws `StackOverflowException`, which cannot be caught in managed code. Recursion and iteration are computationally equivalent in expressive power, but they differ sharply in memory consumption, performance overhead, and code clarity.

**Beginner-Friendly Explanation**
Recursion is a method **solving a problem by solving a smaller version of the same problem**. Think of Russian nesting dolls: to count how many dolls there are, you open the outer doll and see a smaller doll—so you repeat the same process on the smaller doll. When you reach a doll that does not open (the **base case**), you stop. If you never reach that final doll, you keep opening forever and eventually run out of table space (a **stack overflow**). Recursion is elegant for problems that are naturally nested, like folders inside folders or branches on a tree, but it is not always the most efficient choice. C# does not automatically turn recursion into a loop, so very deep recursion can crash your program; iteration is often safer and faster when the problem does not demand recursion’s clarity.

---

### Key Characteristics

- **Two mandatory parts** — Every recursive method must have a **base case** (the termination condition) and at least one **recursive case** (the self-call).
- **Progress toward termination** — Each recursive call must move the problem **strictly closer** to the base case; otherwise the recursion is infinite and exhausts the stack.
- **Stack-frame accumulation** — Each call allocates a new stack frame; memory usage grows linearly with recursion depth.
- **No tail-call optimisation in C#** — The C# compiler emits ordinary recursive calls; the JIT may or may not honour tail calls depending on platform and method structure.
- **`StackOverflowException` is uncatchable** — Exceeding the stack depth terminates the process; there is no managed recovery.
- **Default stack size ≈ 1 MB** — Practical recursion depth is typically in the thousands, varying with frame size and runtime.
- **Expressive for recursive structures** — Trees, graphs, nested data, divide-and-conquer algorithms, and backtracking problems map naturally to recursion.

---

### Prerequisites

- Understanding of C# methods, parameters, and return values.
- Familiarity with the call stack and local variable lifetime.
- Knowledge of `if` statements and conditional logic.
- Awareness of `StackOverflowException` and process termination.
- Basic understanding of algorithm complexity (time and space).

---

### Related Programming Areas

- **Data structures** — Tree traversal (binary trees, file systems, DOM), graph DFS, linked-list processing.
- **Algorithms** — Divide-and-conquer (merge sort, quick sort), backtracking (N-Queens, Sudoku), dynamic programming (with memoisation).
- **Parsing** — Recursive-descent parsers for nested grammars.
- **Functional programming** — Recursion is the primary iteration mechanism in functional languages; C# supports it but without tail-call guarantees.
- **Performance engineering** — Choosing between recursion and iteration affects stack usage, speed, and robustness.

---

### Core Concepts / Features

1. Recursion (Overarching Concept)
2. Base Case
3. Recursive Case
4. Tail Recursion
5. Recursion Depth
6. Stack Overflow
7. Recursion vs. Iteration
8. Recursion Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Recursion (Overarching Concept)

### Definitions

**Core Definition**
Recursion is the technique of having a method call itself, directly or indirectly, to solve progressively smaller instances of a problem.

**Technical Definition**
A recursive method is a method that invokes itself. In C#, recursion is supported for instance methods, static methods, local functions, and lambdas (through delegates). Each invocation pushes a new frame onto the call stack; the frame contains the method’s parameters, locals, and return address. Recursion terminates when the base case is reached and the frames unwind, each returning its result to its caller. The C# compiler does **not** emit tail-call instructions, so recursion is not automatically converted into iteration. Recursion is computationally equivalent to iteration but differs in memory profile and clarity.

**Beginner-Friendly Explanation**
Recursion is a method **calling itself to do part of the work**. The method solves the biggest piece, then asks itself to solve the next smaller piece, and so on, until the problem is small enough to answer directly. Then the answers come back up the chain.

---

### Purposes

- To express algorithms on **recursively defined data** (trees, graphs, nested structures) naturally.
- To implement **divide-and-conquer** algorithms cleanly.
- To write **backtracking** solutions (N-Queens, Sudoku) where the call stack records the search path.
- To simplify **recursive-descent parsers** for nested grammars.
- To provide a **declarative** alternative to complex iterative state management.

---

### Syntax Rules and Structure

```
ReturnType MethodName(parameters)
{
    // base case
    if (terminationCondition)
        return baseValue;

    // recursive case
    return combine(parameters, MethodName(smallerParameters));
}
```

**Component breakdown**

- **Method declaration** — Ordinary method, static method, or local function.
- **Base case** — An `if` (or equivalent) that returns without recursing.
- **Recursive case** — A call to the same method with arguments that move toward the base case.
- **Combine step** — The operation that uses the recursive result to build the current result.

#### Syntax Rules

- A recursive method must have at least one base case.
- Each recursive call must make progress toward the base case.
- The recursive call can be direct (method calls itself) or indirect (mutual recursion between two or more methods).

#### Constraints and Limitations

- Each recursive call consumes stack space; depth is limited by the thread’s stack size.
- C# does not guarantee tail-call optimisation; deep recursion can overflow the stack.
- Naive recursion can cause exponential time complexity (e.g., naive Fibonacci) without memoisation.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Factorial (Classic Recursion)

```csharp
using System;

class Program
{
    static int Factorial(int n)
    {
        if (n <= 1)
            return 1;                 // base case
        return n * Factorial(n - 1);  // recursive case
    }

    static void Main()
    {
        Console.WriteLine(Factorial(5));   // 120
    }
}
```

**Expected Output**

```
120
```

**Why this output occurs**

- `Factorial(5)` calls `Factorial(4)`, which calls `Factorial(3)`, and so on until `Factorial(1)` returns `1`. The frames unwind, multiplying the results: `1 × 2 × 3 × 4 × 5 = 120`.

---

#### Example 2: Recursive Array Sum

```csharp
using System;

class Program
{
    static int Sum(int[] arr, int i)
    {
        if (i == arr.Length)
            return 0;                        // base case
        return arr[i] + Sum(arr, i + 1);     // recursive case
    }

    static void Main()
    {
        int[] data = { 10, 20, 30 };
        Console.WriteLine(Sum(data, 0));   // 60
    }
}
```

**Expected Output**

```
60
```

**Why this output occurs**

- `Sum` adds `arr[0]` to `Sum(arr, 1)`, which adds `arr[1]` to `Sum(arr, 2)`, and so on until `i` reaches the array length and the base case returns `0`.

---

### Real-World Cases

- **File system traversal** — Recursing into directories and subdirectories.
- **Tree structures** — Evaluating expression trees, rendering UI hierarchies.
- **Graph algorithms** — Depth-first search with backtracking.
- **Parsing** — Recursive-descent parsers for JSON, XML, and programming languages.

---

### References

- Recursion and nested methods — .NET docs (GitHub) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Introduction to Recursion in C# Programming — Educative - https://www.educative.io/courses/learn-c-sharp/introduction-to-recursion-in-c-sharp-programming
- Understanding Recursion in C# — CodeSignal - https://codesignal.com/learn/courses/recursion-and-backtracking-in-c/lessons/understanding-recursion-in-c

---

## 2. Base Case

### Definitions

**Core Definition**
The base case is the **termination condition** of a recursive method: the condition under which the method returns a result **without making another recursive call**.

**Technical Definition**
The base case is the branch of a recursive method that does not recurse. It provides a direct answer for the smallest instance of the problem and is the **sine qua non** of recursion—without it, the method recurses infinitely until the stack is exhausted. In C#, the base case is typically expressed with an `if` statement that returns a constant or a trivially computed value.

**Beginner-Friendly Explanation**
The base case is the **“stop here” rule**. It tells the method: “If the problem is this small, just answer directly—do not call yourself again.” Without a base case, the method keeps calling itself forever.

---

### Purposes

- To terminate the recursion.
- To provide direct answers for the simplest instances.
- To prevent infinite recursion and stack overflow.
- To establish the foundation on which recursive results are built.

---

### Syntax Rules and Structure

```
if (baseCondition)
    return baseValue;
```

**Component breakdown**

- `baseCondition` — A condition that identifies the smallest instance.
- `baseValue` — The direct answer for that instance.

#### Syntax Rules

- The base case must be reachable from the initial call.
- The base case must be checked **before** the recursive call.
- A method may have multiple base cases (e.g., `n == 0` and `n == 1`).

#### Constraints and Limitations

- A missing base case causes infinite recursion and `StackOverflowException`.
- The base case must not depend on the recursive call.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Single Base Case

```csharp
using System;

class Program
{
    static int Factorial(int n)
    {
        if (n == 0)
            return 1;                  // base case
        return n * Factorial(n - 1);
    }

    static void Main()
    {
        Console.WriteLine(Factorial(0));   // 1
    }
}
```

**Expected Output**

```
1
```

**Why this output occurs**

- `Factorial(0)` hits the base case and returns `1` immediately.

---

#### Example 2: Multiple Base Cases

```csharp
using System;

class Program
{
    static int Fibonacci(int n)
    {
        if (n == 0) return 0;   // base case 1
        if (n == 1) return 1;   // base case 2
        return Fibonacci(n - 1) + Fibonacci(n - 2);
    }

    static void Main()
    {
        Console.WriteLine(Fibonacci(6));   // 8
    }
}
```

**Expected Output**

```
8
```

**Why this output occurs**

- The two base cases handle `n = 0` and `n = 1`; the recursive case builds from there.

---

### Real-World Cases

- **Tree leaves** — A leaf node is the base case for tree traversal.
- **Empty collections** — An empty array or list is the base case for recursive aggregation.
- **String termination** — The empty string is the base case for string-processing recursion.

---

### References

- Recursion and nested methods — .NET docs (GitHub) (Base Case) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Introduction to Recursion in C# Programming — Educative (Base and recursive cases) - https://www.educative.io/courses/learn-c-sharp/introduction-to-recursion-in-c-sharp-programming
- Understanding Recursion in C# — CodeSignal (Defining the Base Case) - https://codesignal.com/learn/courses/recursion-and-backtracking-in-c/lessons/understanding-recursion-in-c

---

## 3. Recursive Case

### Definitions

**Core Definition**
The recursive case is the part of a recursive method that **calls the method again** with a smaller or simpler input, moving the problem toward the base case.

**Technical Definition**
The recursive case defines how the problem is broken down into smaller sub-problems that are similar in nature but simpler to solve. In C#, the recursive case is typically a `return` statement that invokes the same method with modified arguments. The arguments must move the problem **strictly closer** to the base case; otherwise, the recursion does not progress and eventually overflows the stack.

**Beginner-Friendly Explanation**
The recursive case is the **“solve a smaller version” rule**. It says: “To solve this problem, first solve a smaller version of the same problem, then combine the result with the current step.”

---

### Purposes

- To decompose the problem into smaller sub-problems.
- To leverage the method’s own logic for the sub-problems.
- To build the final result from recursive results.
- To express algorithms declaratively.

---

### Syntax Rules and Structure

```
return combine(currentPart, MethodName(smallerArguments));
```

**Component breakdown**

- `combine` — The operation that uses the recursive result.
- `MethodName` — The recursive call.
- `smallerArguments` — Arguments that move toward the base case.

#### Syntax Rules

- The recursive case must call the same method (directly or indirectly).
- The arguments must make progress toward the base case.
- The recursive call can appear more than once (e.g., Fibonacci calls twice).

#### Constraints and Limitations

- Multiple recursive calls can cause exponential time complexity.
- The recursive case must not prevent the base case from being reached.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Recursive Case with Progress

```csharp
using System;

class Program
{
    static int Factorial(int n)
    {
        if (n <= 1) return 1;                 // base case
        return n * Factorial(n - 1);          // recursive case (n decreases)
    }

    static void Main()
    {
        Console.WriteLine(Factorial(4));   // 24
    }
}
```

**Expected Output**

```
24
```

**Why this output occurs**

- Each recursive call uses `n - 1`, moving strictly toward the base case `n <= 1`.

---

#### Example 2: Recursive Case Without Progress (Infinite)

```csharp
using System;

class Program
{
    static int BadSum(int n)
    {
        if (n == 0) return 0;
        return BadSum(n) + 1;   // SAME argument: no progress!
    }

    static void Main()
    {
        // Console.WriteLine(BadSum(5)); // StackOverflowException
    }
}
```

**Expected Output (conceptual)**

```
StackOverflowException
```

**Why this output occurs**

- `BadSum(5)` calls `BadSum(5)` again; the base case is never reached.

---

### Real-World Cases

- **Tree recursion** — Recursing into left and right children.
- **Divide-and-conquer** — Recursing on halves of an array.
- **Backtracking** — Recursing on the next decision point.

---

### References

- Recursion and nested methods — .NET docs (GitHub) (Recursive case and progress rule) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Introduction to Recursion in C# Programming — Educative (Defining the Recursive Case) - https://www.educative.io/courses/learn-c-sharp/introduction-to-recursion-in-c-sharp-programming

---

## 4. Tail Recursion

### Definitions

**Core Definition**
Tail recursion is a special form of recursion in which the **recursive call is the very last operation** performed by the method—nothing is done with its result after the call returns.

**Technical Definition**
A recursive call is in **tail position** if it is the final action of the method and its result is returned directly, without further computation. In languages that support **tail-call optimisation (TCO)**, a tail-recursive method can reuse the current stack frame for the recursive call, turning recursion into iteration and eliminating stack growth. **C# does not perform tail-call optimisation at the compiler level**: the C# compiler emits ordinary recursive calls in IL, and the JIT may or may not honour the `tail.` IL instruction depending on the runtime, platform, and method structure. The 64-bit JIT can sometimes apply TCO, but it is not guaranteed, so tail recursion cannot be relied upon to prevent stack overflow in C#.

**Beginner-Friendly Explanation**
Tail recursion is **recursion where the last thing you do is call yourself**. In some languages, the compiler can treat this as a loop and reuse the same stack space. In C#, you **cannot rely on that**—the compiler does not guarantee it, so tail recursion does not reliably save you from stack overflow.

---

### Purposes

- To express iteration recursively in languages that support TCO.
- To write algorithms that are naturally recursive but can be optimised into loops.
- To understand the limits of recursion optimisation in C#.
- To recognise when refactoring to iteration is advisable.

---

### Syntax Rules and Structure

```
ReturnType MethodName(parameters)
{
    if (baseCondition)
        return baseValue;                    // base case

    return MethodName(nextParameters);        // tail call: no work after
}
```

**Component breakdown**

- **Tail call** — The recursive call is returned directly.
- **No pending work** — The method does not combine the result with anything after the call.

#### Syntax Rules

- The recursive call must be the last operation.
- The method must return the result of the recursive call directly.
- Tail recursion can be refactored into a loop with an accumulator.

#### Constraints and Limitations

- **C# does not guarantee TCO** — the compiler emits ordinary calls; the JIT may or may not optimise.
- Tail recursion may still overflow the stack in C# on many configurations.
- The 64-bit JIT sometimes honours tail calls, but this is implementation-dependent.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Tail-Recursive Countdown

```csharp
using System;

class Program
{
    static void CountDown(int num)
    {
        Console.WriteLine(num);
        if (num > 0)
            CountDown(num - 1);   // tail call
    }

    static void Main()
    {
        CountDown(3);   // 3 2 1 0
    }
}
```

**Expected Output**

```
3
2
1
0
```

**Why this output occurs**

- The recursive call is the last operation, but **C# does not guarantee TCO**; the stack still grows with each call.

---

#### Example 2: Tail-Recursive Factorial with Accumulator

```csharp
using System;

class Program
{
    static int FactorialTail(int n, int acc = 1)
    {
        if (n <= 1) return acc;
        return FactorialTail(n - 1, acc * n);   // tail call
    }

    static void Main()
    {
        Console.WriteLine(FactorialTail(5));   // 120
    }
}
```

**Expected Output**

```
120
```

**Why this output occurs**

- The accumulator carries the result, making the recursive call tail-position. In C#, this does **not** guarantee stack reuse, but it can be manually converted to a loop.

---

### Real-World Cases

- **Functional algorithms ported to C#** — Understanding why they may overflow.
- **Loop refactoring** — Tail-recursive methods can be rewritten as `while` loops.
- **Understanding JIT behaviour** — Recognising that TCO is not a C# guarantee.

---

### References

- Tail call optimization — Microsoft Learn (archive) (C# compiler does not optimise; JIT may honour) - https://learn.microsoft.com/zh-tw/archive/blogs/abhinaba/tail-call-optimization
- Tail call optimization — Microsoft Learn (original) - https://learn.microsoft.com/en-us/archive/blogs/abhinaba/tail-call-optimization
- Why doesn't .NET/C# optimize for tail-call recursion? — Stack Overflow - https://stackoverflow.com/questions/491376/why-doesnt-net-c-optimize-for-tail-call-recursion

---

## 5. Recursion Depth

### Definitions

**Core Definition**
Recursion depth is the **number of active recursive calls** on the call stack at any given moment—equivalently, the length of the chain of frames from the initial call to the current call.

**Technical Definition**
Recursion depth is determined by the nesting of recursive calls. Each call pushes a frame onto the thread’s stack; the maximum depth is limited by the stack size and the size of each frame (which depends on the number and types of parameters and locals). In .NET, the default stack size is typically **1 MB**, and practical recursion depth is commonly in the **thousands**—one test reported a depth of slightly over **18,000** before overflow. Depth can be increased by running the recursion on a thread with a larger stack, but this is rarely the right solution.

**Beginner-Friendly Explanation**
Recursion depth is **how deep the chain of self-calls goes**. If your recursion goes 100 levels deep, the stack holds 100 frames. Go too deep—thousands of levels—and the stack runs out of room.

---

### Purposes

- To reason about the memory cost of a recursive algorithm.
- To decide whether recursion is safe for a given input size.
- To recognise when to refactor to iteration.
- To understand the limits imposed by the default stack size.

---

### Syntax Rules and Structure (Depth Analysis)

| Factor | Effect on depth |
|---|---|
| **Stack size** | Larger stack → greater maximum depth. |
| **Frame size** | More/larger parameters and locals → smaller maximum depth. |
| **Recursion pattern** | Linear recursion depth ≈ n; tree recursion depth ≈ tree height. |
| **Runtime / platform** | 32-bit vs. 64-bit, .NET version, JIT behaviour affect limits. |

**Component breakdown**

- **Stack size** — Default ≈ 1 MB; configurable per thread.
- **Frame size** — Parameters, locals, return address.
- **Input size** — Depth grows with `n` in linear recursion.

#### Syntax Rules

- Depth grows by one for each active recursive call.
- The maximum safe depth is not fixed; it depends on frame size and stack size.
- Depth can be reduced by converting recursion to iteration.

#### Constraints and Limitations

- Exceeding the maximum depth throws `StackOverflowException`, which **cannot be caught**.
- Increasing stack size postpones but does not eliminate the risk.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Measuring Depth with a Counter

```csharp
using System;

class Program
{
    static void Recurse(int depth, int maxDepth)
    {
        if (depth >= maxDepth)
        {
            Console.WriteLine($"Reached depth {depth}");
            return;
        }
        Recurse(depth + 1, maxDepth);
    }

    static void Main()
    {
        Recurse(0, 5);   // Reached depth 5
    }
}
```

**Expected Output**

```
Reached depth 5
```

**Why this output occurs**

- The counter tracks the current depth; the base case stops at `maxDepth`.

---

#### Example 2: Depth-Limited Recursion with Guard

```csharp
using System;

class Program
{
    static void SafeRecurse(int depth, int maxDepth)
    {
        if (depth > maxDepth)
            throw new InvalidOperationException("Maximum recursion depth reached.");
        // ... recursive work ...
        SafeRecurse(depth + 1, maxDepth);
    }
}
```

**Expected Output (conceptual)**

```
InvalidOperationException: Maximum recursion depth reached.
```

**Why this output occurs**

- A guard throws a catchable exception before the stack overflows, giving the program a chance to recover.

---

### Real-World Cases

- **Deep directory trees** — Recursion depth equals the directory nesting level.
- **Linked lists** — Recursive traversal depth equals the list length.
- **Expression trees** — Depth equals the nesting of expressions.

---

### References

- C# recursion depth — how deep can you go? — Tencent Cloud (citing Stack Overflow) (≈18k depth observed) - https://cloud.tencent.cn/developer/ask/sof/103056542
- StackOverflowException with recursion depth — Stack Overflow (depth 12737 observed) - https://stackoverflow.com/questions/47321223/stackoverflowexception-with-lazy-when-throwing-exception
- Recursion and nested methods — .NET docs (GitHub) (space complexity O(n)) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md

---

## 6. Stack Overflow

### Definitions

**Core Definition**
A stack overflow occurs when a program’s call stack exceeds its allotted size, typically because of **unbounded or excessively deep recursion**. In .NET, this results in a `StackOverflowException` that **terminates the process** and cannot be caught by managed code.

**Technical Definition**
`StackOverflowException` is thrown when the execution stack overflows because it contains too many nested method calls. In .NET, this exception **cannot be caught** and the process is terminated; there is no way to recover within managed code. Common causes include infinite recursion (missing base case or no progress toward it) and deep but finite recursion that exceeds the thread’s stack capacity. The default .NET stack size is approximately **1 MB**, and the maximum depth varies with frame size and runtime. Mitigation strategies include adding a depth guard, converting recursion to iteration, or running on a thread with a larger stack.

**Beginner-Friendly Explanation**
A stack overflow is **running out of stack space because the chain of calls got too deep**. It is like a stack of plates that grows taller and taller until it topples. In C#, once it topples, the program **crashes immediately**—you cannot catch the exception and continue.

---

### Purposes

- To understand the failure mode of unbounded recursion.
- To design recursive algorithms that stay within safe depth.
- To choose iteration when depth cannot be bounded.
- To add guards that fail gracefully before the stack overflows.

---

### Syntax Rules and Structure (Causes and Symptoms)

| Cause | Description |
|---|---|
| **Missing base case** | The method never stops recursing. |
| **No progress** | The recursive call does not move toward the base case. |
| **Deep finite recursion** | Input size exceeds stack capacity. |
| **Large frames** | Large parameters/locals reduce maximum depth. |

**Component breakdown**

- **`StackOverflowException`** — Process-terminating; uncatchable.
- **Default stack** — ≈ 1 MB.
- **Depth guard** — A check that throws a catchable exception before overflow.

#### Syntax Rules

- `StackOverflowException` cannot be caught with `try`/`catch` in .NET.
- A depth guard can throw `InvalidOperationException` or a custom exception instead.
- Iteration eliminates stack growth entirely.

#### Constraints and Limitations

- There is no managed recovery from a stack overflow.
- Increasing stack size only postpones the problem.
- Depth limits are implementation-dependent.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Infinite Recursion (Stack Overflow)

```csharp
using System;

class Program
{
    static int BadFactorial(int n)
    {
        return n * BadFactorial(n - 1);   // NO BASE CASE
    }

    static void Main()
    {
        // Console.WriteLine(BadFactorial(5)); // StackOverflowException
    }
}
```

**Expected Output (conceptual)**

```
StackOverflowException (process terminates)
```

**Why this output occurs**

- The method recurses indefinitely because there is no base case; the stack grows until it overflows.

---

#### Example 2: Depth Guard Instead of Overflow

```csharp
using System;

class Program
{
    static string Recurse(int value, int counter)
    {
        const int MAX_RECURSION_LEVEL = 1000;
        if (counter > MAX_RECURSION_LEVEL)
            throw new InvalidOperationException("Maximum recursion depth reached.");
        return Recurse(value, counter + 1);
    }
}
```

**Expected Output (conceptual)**

```
InvalidOperationException: Maximum recursion depth reached.
```

**Why this output occurs**

- The guard throws a catchable exception before the stack overflows, allowing the program to handle the condition.

---

### Real-World Cases

- **Infinite loops disguised as recursion** — Missing termination conditions.
- **Deep file-system traversal** — Extremely deep directory trees.
- **Unbounded user input** — Recursion depth driven by user-supplied data.

---

### References

- StackOverflowException — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.stackoverflowexception
- Recursion and nested methods — .NET docs (GitHub) (stack overflow pitfalls) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- StackOverflowException with recursion depth — Stack Overflow - https://stackoverflow.com/questions/47321223/stackoverflowexception-with-lazy-when-throwing-exception
- C# recursion depth — Tencent Cloud (citing Stack Overflow) - https://cloud.tencent.cn/developer/ask/sof/103056542

---

## 7. Recursion vs. Iteration

### Definitions

**Core Definition**
Recursion and iteration are two equivalent techniques for repetition: recursion solves a problem by calling a method on smaller inputs, while iteration solves it by repeating a block of code with a loop. They differ in memory profile, performance overhead, and code clarity.

**Technical Definition**
Recursion and iteration are computationally equivalent in expressive power. Recursion consumes stack space proportional to depth (O(n) space for linear recursion), incurs function-call overhead, and can overflow the stack. Iteration uses constant extra space (O(1) for simple loops), has lower per-step overhead (comparison and branch rather than call/return), and does not risk stack overflow. However, recursion is often **clearer and more concise** for recursively structured problems, and the performance difference is frequently negligible outside hot paths. C# does **not** guarantee tail-call optimisation, so recursion does not automatically become iteration.

**Beginner-Friendly Explanation**
Recursion and iteration are **two ways to repeat work**. Iteration is a loop: “do this again and again.” Recursion is a method calling itself: “solve a smaller version of the problem.” Iteration is usually faster and safer for deep repetition; recursion is often clearer when the data itself is nested.

---

### Purposes

- To choose the right technique for a given problem.
- To understand the memory and performance trade-offs.
- To decide when to refactor recursion to iteration.
- To leverage recursion’s clarity without incurring its risks.

---

### Syntax Rules and Structure (Comparison)

| Aspect | Recursion | Iteration |
|---|---|---|
| **Memory** | O(n) stack for linear recursion | O(1) extra space |
| **Speed** | Call/return overhead per step | Loop overhead is cheaper |
| **Risk** | Stack overflow for deep recursion | No stack overflow |
| **Clarity** | Excellent for recursive structures | Excellent for linear repetition |
| **TCO in C#** | Not guaranteed | Not applicable |

**Component breakdown**

- **Memory** — Recursion uses stack frames; iteration uses loop variables.
- **Speed** — Iteration avoids call overhead.
- **Risk** — Recursion can overflow; iteration cannot.
- **Clarity** — Recursion matches recursive data; iteration matches linear repetition.

#### Syntax Rules

- Use recursion for trees, graphs, divide-and-conquer, and backtracking.
- Use iteration for linear repetition and when depth may be large (> 1000).
- Any recursion can be converted to iteration with an explicit stack.

#### Constraints and Limitations

- Iterative versions of recursive algorithms can be more complex to write.
- Recursion is often preferable when performance is not the bottleneck.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Factorial — Recursive vs. Iterative

```csharp
using System;

class Program
{
    static int FactorialRecursive(int n)
    {
        if (n <= 1) return 1;
        return n * FactorialRecursive(n - 1);
    }

    static int FactorialIterative(int n)
    {
        int result = 1;
        for (int i = 2; i <= n; i++)
            result *= i;
        return result;
    }

    static void Main()
    {
        Console.WriteLine(FactorialRecursive(5));   // 120
        Console.WriteLine(FactorialIterative(5));   // 120
    }
}
```

**Expected Output**

```
120
120
```

**Why this output occurs**

- Both compute the same result; the iterative version avoids stack growth.

---

#### Example 2: Fibonacci — Naive Recursion vs. Iteration

```csharp
using System;

class Program
{
    static int FibRecursive(int n)
    {
        if (n <= 1) return n;
        return FibRecursive(n - 1) + FibRecursive(n - 2);
    }

    static int FibIterative(int n)
    {
        if (n <= 1) return n;
        int prev = 0, curr = 1;
        for (int i = 2; i <= n; i++)
        {
            int next = prev + curr;
            prev = curr;
            curr = next;
        }
        return curr;
    }

    static void Main()
    {
        Console.WriteLine(FibRecursive(10));   // 55
        Console.WriteLine(FibIterative(10));   // 55
    }
}
```

**Expected Output**

```
55
55
```

**Why this output occurs**

- The naive recursive Fibonacci is O(2^n) due to repeated subproblems; the iterative version is O(n).

---

### Real-World Cases

- **Tree traversal** — Recursion is natural; iteration requires an explicit stack.
- **Directory walking** — Recursion is clearer; iteration avoids overflow on deep trees.
- **Sorting** — Merge sort and quick sort are naturally recursive; iterative variants are possible but more complex.
- **Numerical loops** — Iteration is almost always preferred.

---

### References

- Chapter 13 — Code Review: .NET Application Performance — If your code uses recursion, consider using a loop instead — Microsoft Learn - https://learn.microsoft.com/vi-vn/previous-versions/msp-n-p/ff647802(v=pandp.10)
- Recursion and nested methods — .NET docs (GitHub) (recursion guidelines table) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Recursion and Iteration Techniques for Problem Solving in C# — Educative - https://www.educative.io/courses/learn-c-sharp/recursion-and-iteration-techniques-for-problem-solving-in-c

---

## 8. Recursion Best Practices

### Definitions

**Core Definition**
Recursion best practices are the guidelines that ensure recursive methods are **correct, safe, and appropriately chosen** for the problem at hand.

**Technical Definition**
Best practices for recursion derive from .NET performance guidance and algorithm-design principles. Key recommendations include: **always define a clear base case**; **ensure every recursive call makes progress toward the base case**; **use recursion for recursively structured problems** (trees, graphs, divide-and-conquer, backtracking); **avoid recursion for simple linear repetition**; **guard against deep recursion** with a depth check or by converting to iteration; **use memoisation** to avoid exponential recomputation; and **prefer iteration when depth may exceed ~1000**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep recursion safe: **always have a stopping rule**, **always move toward that rule**, **use recursion when the problem is naturally nested**, and **switch to a loop when the depth could get large**.

---

### Purposes

- To prevent infinite recursion and stack overflow.
- To choose recursion only when it adds clarity.
- To avoid exponential time complexity.
- To keep recursive code readable and maintainable.
- To align with performance and safety guidance.

---

### Guidelines and Structure

#### General Guidelines

```
// Always define a base case
if (baseCondition) return baseValue;

// Always make progress
return MethodName(smallerArguments);

// Use recursion for recursive structures
// (trees, graphs, divide-and-conquer, backtracking)

// Avoid recursion for simple linear repetition
// Use iteration when depth may exceed ~1000

// Guard against deep recursion
if (depth > MAX_DEPTH)
    throw new InvalidOperationException();

// Use memoisation for overlapping subproblems
```

**Component breakdown**

- **Base case** — Mandatory termination condition.
- **Progress** — Each call must move toward the base case.
- **Recursive structures** — Trees, graphs, nested data.
- **Iteration for linear repetition** — Simple loops do not need recursion.
- **Depth guard** — Fail gracefully before stack overflow.
- **Memoisation** — Cache results to avoid recomputation.

#### Rules and Constraints

- **Do** define a base case that is reachable from every call.
- **Do** ensure each recursive call makes progress.
- **Do** use recursion for tree/graph traversal, divide-and-conquer, and backtracking.
- **Do** guard against deep recursion or convert to iteration.
- **Do** use memoisation for problems with overlapping subproblems.
- **Do not** use recursion for simple linear loops.
- **Do not** rely on tail-call optimisation in C#—it is not guaranteed.
- **Do not** ignore the risk of `StackOverflowException`, which cannot be caught.

---

### Annotated Code Examples and Expected Outputs

#### Example: Safe, Memoised Recursion

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static Dictionary<int, long> _memo = new();

    static long FibMemo(int n)
    {
        if (n <= 1) return n;                 // base case
        if (_memo.TryGetValue(n, out var cached))
            return cached;                     // memoisation
        long result = FibMemo(n - 1) + FibMemo(n - 2);
        _memo[n] = result;
        return result;
    }

    static void Main()
    {
        Console.WriteLine(FibMemo(50));   // 12586269025
    }
}
```

**Expected Output**

```
12586269025
```

**Why this output occurs**

- Memoisation avoids the exponential recomputation of naive Fibonacci, making `FibMemo(50)` feasible.

---

### Real-World Cases

- **Tree traversal** — Use recursion with a clear base case (leaf node).
- **Divide-and-conquer** — Use recursion with a depth guard for very large inputs.
- **Parsing** — Recursive descent with explicit termination conditions.
- **Dynamic programming** — Recursion with memoisation for overlapping subproblems.

---

### References

- Recursion and nested methods — .NET docs (GitHub) (recursion guidelines and pitfalls) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Chapter 13 — Code Review: .NET Application Performance — If your code uses recursion, consider using a loop instead — Microsoft Learn - https://learn.microsoft.com/vi-vn/previous-versions/msp-n-p/ff647802(v=pandp.10)
- Recursion and Iteration Techniques for Problem Solving in C# — Educative - https://www.educative.io/courses/learn-c-sharp/recursion-and-iteration-techniques-for-problem-solving-in-c

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Tail-call optimisation | **Not guaranteed** in C#; JIT-dependent, platform-dependent. |
| `StackOverflowException` | **Uncatchable** in .NET; process terminates. |
| Default stack size | ≈ **1 MB**; maximum depth varies with frame size and runtime. |
| Recursion depth limit | **Implementation-dependent**; one test observed ≈ 18,000. |
| Naive recursive Fibonacci | **O(2^n)** time; use memoisation or iteration. |
| Recursion for deep structures | **Discouraged** when depth > 1000; prefer iteration. |
| Increasing stack size | Possible per-thread, but **not a general solution**. |

---

## References (Consolidated)

- Recursion and nested methods — .NET docs (GitHub) (base/recursive cases, pitfalls, guidelines) - https://raw.githubusercontent.com/rkttu/csharp-book/812f50dccf4d7be7683e4c6a97b7ca4be07aea5f/docs/part1-basics/08-advanced-methods.md
- Introduction to Recursion in C# Programming — Educative - https://www.educative.io/courses/learn-c-sharp/introduction-to-recursion-in-c-sharp-programming
- Understanding Recursion in C# — CodeSignal - https://codesignal.com/learn/courses/recursion-and-backtracking-in-c/lessons/understanding-recursion-in-c
- Tail call optimization — Microsoft Learn (archive) (C# compiler does not optimise) - https://learn.microsoft.com/zh-tw/archive/blogs/abhinaba/tail-call-optimization
- Tail call optimization — Microsoft Learn (original) - https://learn.microsoft.com/en-us/archive/blogs/abhinaba/tail-call-optimization
- Why doesn't .NET/C# optimize for tail-call recursion? — Stack Overflow - https://stackoverflow.com/questions/491376/why-doesnt-net-c-optimize-for-tail-call-recursion
- C# recursion depth — Tencent Cloud (citing Stack Overflow) - https://cloud.tencent.cn/developer/ask/sof/103056542
- StackOverflowException with recursion depth — Stack Overflow - https://stackoverflow.com/questions/47321223/stackoverflowexception-with-lazy-when-throwing-exception
- Chapter 13 — Code Review: .NET Application Performance — If your code uses recursion, consider using a loop instead — Microsoft Learn - https://learn.microsoft.com/vi-vn/previous-versions/msp-n-p/ff647802(v=pandp.10)
- Recursion and Iteration Techniques for Problem Solving in C# — Educative - https://www.educative.io/courses/learn-c-sharp/recursion-and-iteration-techniques-for-problem-solving-in-c
- Recursion & Algorithms in C# — C# Corner - https://www.c-sharpcorner.com/article/recursion-algorithms-in-c-sharp/
- Recursion in C# — Scaler Topics - https://www.scaler.com/topics/csharp/recursion-in-csharp/
- Recursive Methods — LaunchCode Education - https://education.launchcode.org/intro-to-programming-csharp/chapters/methods/recursion.html

---