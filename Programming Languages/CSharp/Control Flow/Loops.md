# C# Loops — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Loops in C# are **iteration statements** that let a program repeat a block of code multiple times without duplicating the code manually. C# provides four loop constructs — `for`, `foreach`, `while`, and `do...while` — each tailored to a different repetition pattern. Loops are the primary mechanism for processing collections, performing calculations across ranges, and implementing algorithms that require repeated steps until a condition is met.

**Technical Definition**
C# classifies iteration statements as **selection and iteration statements** within the broader category of control-flow statements. The C# language reference defines four iteration statements: the **`for` statement** executes its body while a specified Boolean expression evaluates to `true`; the **`foreach` statement** enumerates the elements of a collection and executes its body for each element; the **`do` statement** conditionally executes its body one or more times; and the **`while` statement** conditionally executes its body zero or more times. At any point within the body of an iteration statement, the **`break` statement** exits the loop, and the **`continue` statement** skips to the next iteration. The **`goto` statement** can transfer control to a labelled statement, which is sometimes used to break out of deeply nested loops. Loop performance varies by construct, data structure, and runtime version; modern JIT compilers optimise loops aggressively, so correctness and readability should generally take precedence over micro-optimisations unless profiling shows a bottleneck.

**Beginner-Friendly Explanation**
A loop is a **repetition machine**. Instead of writing `Console.WriteLine("Hello");` ten times, you write a loop that says “repeat this ten times.” The `for` loop is the **counter-controlled** loop: you know exactly how many times you want to repeat. The `foreach` loop is the **collection walker**: you have a list of items and you want to do something with each one. The `while` loop is the **condition checker**: keep going as long as something is true. The `do...while` loop is the **at-least-once** loop: do something first, then check whether you should keep going. `break` is the **emergency exit**, and `continue` is the **skip button**.

---

### Key Characteristics

- **Four loop constructs** — `for` (count-controlled), `foreach` (collection enumeration), `while` (pre-test condition), and `do...while` (post-test condition).
- **`for` loop structure** — Three sections: initializer, condition, and iterator, each separated by semicolons.
- **`foreach` enumeration** — Works with any type implementing `IEnumerable` or `IEnumerable<T>`; the iteration variable is **read-only**.
- **`while` vs. `do...while`** — `while` checks the condition **before** each iteration (zero or more executions); `do...while` checks **after** (one or more executions).
- **Loop control** — `break` exits the nearest enclosing loop; `continue` starts the next iteration; `goto` transfers to a label.
- **Infinite loops** — Created with `for(;;)`, `while(true)`, or `do { } while(true)`; must be exited with `break`, `goto`, `return`, or an exception.
- **Nested loops** — Loops inside loops; `break` and `continue` affect only the **innermost** enclosing loop.

---

### Prerequisites

- Understanding of Boolean expressions and comparison operators.
- Familiarity with C#’s basic types (`int`, `bool`, `string`, enums).
- Knowledge of arrays and basic collection types (`List<T>`).
- Awareness of `IEnumerable` and `IEnumerable<T>` (for `foreach`).
- Basic understanding of block scope and variable lifetime.

---

### Related Programming Areas

- **Collections** — `foreach` is the primary way to iterate over arrays, lists, dictionaries, and other collections.
- **Algorithms** — Sorting, searching, and numerical algorithms rely on loops.
- **LINQ** — LINQ queries can replace many explicit loops with declarative code.
- **Performance** — Choosing the right loop and minimizing work inside it can significantly affect throughput.
- **Concurrency** — `Parallel.For` and `Parallel.ForEach` extend loop semantics to multi-threaded execution.

---

### Core Concepts / Features

1. `for` Loop
2. `foreach` Loop
3. `while` Loop
4. `do...while` Loop
5. Loop Control: `break`
6. Loop Control: `continue`
7. Loop Control: `goto`
8. Infinite Loops
9. Nested Loops
10. Loop Performance
11. Loop Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. `for` Loop

### Definitions

**Core Definition**
The `for` loop is a **count-controlled** iteration statement that executes its body while a specified Boolean condition evaluates to `true`, using an initializer, a condition, and an iterator to control the repetition.

**Technical Definition**
The `for` statement executes a statement or a block of statements while a specified Boolean expression evaluates to `true`. It has three optional sections: the **initializer** (executed once before the loop), the **condition** (evaluated before each iteration; if `true` or absent, the iteration runs), and the **iterator** (executed after each iteration). All three sections are optional; `for(;;)` creates an infinite loop. The loop variable declared in the initializer is scoped to the `for` statement and cannot be accessed outside it.

**Beginner-Friendly Explanation**
The `for` loop is the **“repeat exactly N times”** loop. You set up a counter, say how long the counter should keep going, and say how the counter changes each time. It is perfect when you know the number of iterations in advance.

---

### Purposes

- To repeat a block of code a specific number of times.
- To iterate over arrays and collections using an index.
- To implement counting loops and accumulators.
- To traverse sequences in reverse or with custom step sizes.
- To serve as the backbone of many algorithms (sorting, searching).

---

### Syntax Rules and Structure

```
for (initializer; condition; iterator)
{
    // loop body
}
```

**Component breakdown**

- `initializer` — Executed once before the loop; typically declares and initializes a loop variable.
- `condition` — A `bool` expression; if `true` (or absent), the body executes.
- `iterator` — Executed after each iteration; typically increments or decrements the loop variable.

#### Syntax Rules

- All three sections are optional; `for(;;)` is an infinite loop.
- The loop variable declared in the initializer is scoped to the `for` statement.
- Multiple expressions in the initializer and iterator can be separated by commas.
- The condition must be a `bool` expression (or absent).

#### Constraints and Limitations

- The loop variable cannot be accessed outside the `for` statement.
- Off-by-one errors are common; careful bounds checking is required.
- Modifying the loop variable inside the body can cause unexpected behaviour.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Counting Loop

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 0; i < 3; i++)
        {
            Console.Write(i);
        }
        Console.WriteLine();   // Output: 012
    }
}
```

**Expected Output**

```
012
```

**Why this output occurs**

- `i` starts at `0`; the condition `i < 3` is `true` for `0`, `1`, and `2`; `i++` increments after each iteration.

---

#### Example 2: Iterating an Array with `for`

```csharp
using System;

class Program
{
    static void Main()
    {
        string[] fruits = { "apple", "banana", "cherry" };

        for (int i = 0; i < fruits.Length; i++)
        {
            Console.WriteLine($"{i}: {fruits[i]}");
        }
    }
}
```

**Expected Output**

```
0: apple
1: banana
2: cherry
```

**Why this output occurs**

- The loop uses the index `i` to access each element of the array.

---

### Real-World Cases

- **Array processing** — Accessing elements by index.
- **Matrix operations** — Iterating rows and columns.
- **Countdowns** — `for (int i = 10; i > 0; i--)`.
- **Stepped loops** — `for (int i = 0; i < 100; i += 10)`.

---

### References

- Iteration statements — `for` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-for-statement
- The for statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1386-the-for-statement

---

## 2. `foreach` Loop

### Definitions

**Core Definition**
The `foreach` loop is a **collection enumeration** statement that executes its body once for each element in a collection, without requiring an explicit index or counter.

**Technical Definition**
The `foreach` statement enumerates the elements of a collection and executes its body for each element of the collection. It works with any type that implements `IEnumerable` or `IEnumerable<T>` (including arrays, `List<T>`, `Dictionary<TKey,TValue>`, and strings). The iteration variable is **read-only**; you cannot assign to it. The compiler generates an enumerator (`GetEnumerator`, `MoveNext`, `Current`) for the collection; for arrays and some types, the JIT compiler can optimise the enumeration to avoid the enumerator overhead.

**Beginner-Friendly Explanation**
The `foreach` loop is the **“do this for every item in the list”** loop. You do not need to know how many items there are or how to access them by index—`foreach` handles it all.

---

### Purposes

- To iterate over any collection without managing an index.
- To process each element of an array, list, or dictionary.
- To write clean, readable code for collection traversal.
- To support LINQ and other enumerable sequences.
- To avoid off-by-one errors associated with index-based loops.

---

### Syntax Rules and Structure

```
foreach (type variable in collection)
{
    // loop body
}
```

**Component breakdown**

- `type` — The element type (or `var` for implicit typing).
- `variable` — The read-only iteration variable.
- `collection` — An expression implementing `IEnumerable` or `IEnumerable<T>`.

#### Syntax Rules

- The collection must implement `IEnumerable` or `IEnumerable<T>`.
- The iteration variable is read-only; assignment produces a compile error.
- `var` can be used for the iteration variable.
- The iteration variable’s scope is limited to the loop body.

#### Constraints and Limitations

- Cannot modify the collection inside the loop (adding/removing elements may invalidate the enumerator).
- Cannot directly access the index; use a manual counter if needed.
- Enumeration order depends on the collection type.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Iterating an Array

```csharp
using System;

class Program
{
    static void Main()
    {
        int[] numbers = { 10, 20, 30 };

        foreach (int number in numbers)
        {
            Console.WriteLine(number);
        }
    }
}
```

**Expected Output**

```
10
20
30
```

**Why this output occurs**

- `foreach` visits each element of the array in order.

---

#### Example 2: Iterating a Dictionary

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var ages = new Dictionary<string, int>
        {
            ["Alice"] = 30,
            ["Bob"] = 25
        };

        foreach (KeyValuePair<string, int> pair in ages)
        {
            Console.WriteLine($"{pair.Key} is {pair.Value}");
        }
    }
}
```

**Expected Output (order may vary)**

```
Alice is 30
Bob is 25
```

**Why this output occurs**

- `foreach` enumerates the key-value pairs of the dictionary.

---

### Real-World Cases

- **Processing collections** — Applying an operation to every item.
- **Logging** — Iterating over a list of records.
- **Summing/aggregating** — Computing totals or averages.
- **LINQ pipelines** — `foreach` consumes the results of a query.

---

### References

- Iteration statements — `foreach` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-foreach-statement
- The foreach statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1387-the-foreach-statement

---

## 3. `while` Loop

### Definitions

**Core Definition**
The `while` loop is a **pre-test** iteration statement that executes its body **zero or more times**, checking a Boolean condition before each iteration.

**Technical Definition**
The `while` statement conditionally executes its body **zero or more times**. The condition is evaluated **before** each iteration; if it is `true`, the body executes; if `false`, the loop terminates. If the condition is initially `false`, the body never executes. The condition must be a `bool` expression.

**Beginner-Friendly Explanation**
The `while` loop is the **“keep going as long as this is true”** loop. It checks the condition first; if the condition is false from the start, the loop never runs.

---

### Purposes

- To repeat code as long as a condition remains true.
- To implement loops where the number of iterations is unknown beforehand.
- To process data until a sentinel value is encountered.
- To implement polling and waiting loops.
- To complement `for` when the iteration logic is not counter-based.

---

### Syntax Rules and Structure

```
while (condition)
{
    // loop body
}
```

**Component breakdown**

- `condition` — A `bool` expression evaluated before each iteration.
- `{ ... }` — The loop body.

#### Syntax Rules

- The condition is evaluated before the body; if initially `false`, the body is skipped.
- The condition must be a `bool` expression.
- The body must eventually make the condition `false` (or use `break`) to avoid an infinite loop.

#### Constraints and Limitations

- If the condition never becomes `false`, the loop is infinite.
- Forgetting to update the loop control variable is a common bug.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic `while` Loop

```csharp
using System;

class Program
{
    static void Main()
    {
        int n = 0;
        while (n < 3)
        {
            Console.Write(n);
            n++;
        }
        Console.WriteLine();   // Output: 012
    }
}
```

**Expected Output**

```
012
```

**Why this output occurs**

- `n` starts at `0`; the condition `n < 3` is `true` for `0`, `1`, and `2`; `n++` increments after each iteration.

---

#### Example 2: `while` That Never Runs

```csharp
using System;

class Program
{
    static void Main()
    {
        int n = 10;
        while (n < 3)
        {
            Console.WriteLine(n);
            n++;
        }
        Console.WriteLine("Done");   // Output: Done
    }
}
```

**Expected Output**

```
Done
```

**Why this output occurs**

- The condition `n < 3` is `false` initially (`10 < 3` is `false`), so the body never executes.

---

### Real-World Cases

- **Reading input** — `while ((line = reader.ReadLine()) != null)`.
- **Polling** — Waiting for a resource to become available.
- **Game loops** — Continuing while the game is running.
- **Processing queues** — `while (queue.Count > 0)`.

---

### References

- Iteration statements — `while` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-while-statement
- The while statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-while-statement

---

## 4. `do...while` Loop

### Definitions

**Core Definition**
The `do...while` loop is a **post-test** iteration statement that executes its body **one or more times**, checking a Boolean condition **after** each iteration.

**Technical Definition**
The `do` statement conditionally executes its body **one or more times**. The condition is evaluated **after** each execution of the body; if it is `true`, the loop repeats; if `false`, the loop terminates. Because the condition is evaluated at the end, the body always executes at least once. The condition must be a `bool` expression.

**Beginner-Friendly Explanation**
The `do...while` loop is the **“do this first, then check if you should keep going”** loop. It always runs the body at least once, even if the condition is false from the start.

---

### Purposes

- To execute a block of code at least once before checking a condition.
- To implement menu systems that should display at least once.
- To validate input and repeat until valid.
- To complement `while` when the first iteration must always happen.

---

### Syntax Rules and Structure

```
do
{
    // loop body
} while (condition);
```

**Component breakdown**

- `do` — Introduces the loop body.
- `{ ... }` — The loop body, executed at least once.
- `while (condition);` — The condition, evaluated after the body.

#### Syntax Rules

- The body always executes at least once.
- The condition is evaluated after each iteration.
- The condition must be a `bool` expression.
- The semicolon after the `while` condition is **required**.

#### Constraints and Limitations

- The semicolon after `while (condition)` is easy to forget.
- The body must eventually make the condition `false` (or use `break`) to avoid an infinite loop.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `do...while` That Runs Once

```csharp
using System;

class Program
{
    static void Main()
    {
        int n = 10;
        do
        {
            Console.WriteLine(n);
            n++;
        } while (n < 3);
        Console.WriteLine("Done");   // Output: 10 Done
    }
}
```

**Expected Output**

```
10
Done
```

**Why this output occurs**

- The body executes once, printing `10`; then `n` becomes `11`. The condition `n < 3` is `false`, so the loop terminates.

---

#### Example 2: Input Validation

```csharp
using System;

class Program
{
    static void Main()
    {
        string? input;
        do
        {
            Console.Write("Enter a number: ");
            input = Console.ReadLine();
        } while (!int.TryParse(input, out _));

        Console.WriteLine("Valid number entered");
    }
}
```

**Expected Output (interactive)**

```
Enter a number: abc
Enter a number: 42
Valid number entered
```

**Why this output occurs**

- The loop continues until the user enters a valid number.

---

### Real-World Cases

- **Menu systems** — Displaying a menu and processing choices until exit.
- **Input validation** — Repeating prompts until valid input is received.
- **Game loops** — A “play again?” prompt after each round.

---

### References

- Iteration statements — `do` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements#the-do-statement
- The do statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1389-the-do-statement

---

## 5. Loop Control: `break`

### Definitions

**Core Definition**
The `break` statement **terminates the nearest enclosing loop** (or `switch` statement) immediately and transfers control to the statement following the terminated construct.

**Technical Definition**
The `break` statement terminates the closest enclosing iteration statement (`for`, `foreach`, `while`, or `do`) or `switch` statement. In nested loops, `break` terminates only the **innermost** loop that contains it. It transfers control to the statement that follows the terminated statement, if any.

**Beginner-Friendly Explanation**
`break` is the **emergency exit**. When the loop encounters `break`, it stops immediately and jumps to the code after the loop.

---

### Purposes

- To exit a loop early when a condition is met.
- To terminate infinite loops.
- To stop searching once a desired element is found.
- To exit a `switch` section.
- To avoid unnecessary iterations.

---

### Syntax Rules and Structure

```
break;
```

**Component breakdown**

- `break` — The keyword.
- `;` — Statement terminator.

#### Syntax Rules

- `break` can appear only inside a loop or `switch`.
- It exits the innermost enclosing loop or switch.
- It makes the endpoint of the loop body unreachable.

#### Constraints and Limitations

- In nested loops, `break` affects only the innermost loop; use a flag or `goto` to exit multiple levels.
- Cannot be used outside a loop or switch.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Early Exit

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 0; i < 10; i++)
        {
            if (i == 3)
                break;
            Console.Write(i);
        }
        Console.WriteLine();   // Output: 012
    }
}
```

**Expected Output**

```
012
```

**Why this output occurs**

- When `i` reaches `3`, `break` exits the loop immediately.

---

#### Example 2: `break` in Nested Loops

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int outer = 0; outer < 3; outer++)
        {
            for (int inner = 0; inner < 3; inner++)
            {
                if (inner == 1)
                    break;
                Console.Write($"{outer},{inner} ");
            }
        }
        Console.WriteLine();   // Output: 0,0 1,0 2,0
    }
}
```

**Expected Output**

```
0,0 1,0 2,0
```

**Why this output occurs**

- `break` exits only the inner loop; the outer loop continues.

---

### Real-World Cases

- **Search** — Stop iterating once the target is found.
- **Infinite loops** — Exit when a condition is met.
- **Validation** — Stop processing when an error is detected.

---

### References

- Jump statements — `break` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-break-statement
- The break statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13102-the-break-statement

---

## 6. Loop Control: `continue`

### Definitions

**Core Definition**
The `continue` statement **skips the remainder of the current iteration** and starts a new iteration of the nearest enclosing loop.

**Technical Definition**
The `continue` statement starts a new iteration of the closest enclosing iteration statement. It does not exit the loop; instead, it jumps to the loop’s condition (for `while`/`do`) or iterator (for `for`/`foreach`) and proceeds with the next iteration.

**Beginner-Friendly Explanation**
`continue` is the **skip button**. When the loop encounters `continue`, it skips the rest of the current iteration and jumps to the next one.

---

### Purposes

- To skip processing for certain elements.
- To avoid deeply nested `if` statements inside loops.
- To filter items during iteration.
- To implement “do nothing for this case” logic.

---

### Syntax Rules and Structure

```
continue;
```

**Component breakdown**

- `continue` — The keyword.
- `;` — Statement terminator.

#### Syntax Rules

- `continue` can appear only inside a loop.
- It skips the remainder of the current iteration.
- It proceeds to the next iteration (condition or iterator).

#### Constraints and Limitations

- In nested loops, `continue` affects only the innermost loop.
- Cannot be used outside a loop.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Skipping Even Numbers

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 0; i < 6; i++)
        {
            if (i % 2 == 0)
                continue;
            Console.Write(i);
        }
        Console.WriteLine();   // Output: 135
    }
}
```

**Expected Output**

```
135
```

**Why this output occurs**

- `continue` skips the `Console.Write` for even numbers.

---

### Real-World Cases

- **Filtering** — Skipping invalid or uninteresting items.
- **Validation** — Continuing to the next item after an error.
- **Game logic** — Skipping dead enemies in a loop.

---

### References

- Jump statements — `continue` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-continue-statement
- The continue statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13103-the-continue-statement

---

## 7. Loop Control: `goto`

### Definitions

**Core Definition**
The `goto` statement **unconditionally transfers control** to a statement marked with a **label** within the same scope.

**Technical Definition**
The `goto` statement transfers control to a statement that is marked by a label. It is a general jump statement that can be used to exit deeply nested loops, although this usage is **discouraged** in favour of structured alternatives such as flags, helper methods, or `break`/`return`. The label must be in the same scope as the `goto` statement; jumping into a block from outside is not allowed.

**Beginner-Friendly Explanation**
`goto` is the **teleport** statement. It jumps directly to a labelled line of code. It is powerful but can make code hard to follow, so it should be used sparingly.

---

### Purposes

- To break out of deeply nested loops.
- To transfer control to a specific labelled section.
- To implement simple state machines (historically).
- To simulate fall-through between switch cases (`goto case`).

---

### Syntax Rules and Structure

```
goto   label;
...
label:
    // statements
```

**Component breakdown**

- `goto label;` — The jump.
- `label:` — The target label.

#### Syntax Rules

- The label must be in the same scope as the `goto`.
- Labels are identifiers followed by a colon.
- Cannot jump into a block or out of a `try`/`catch`/`finally` in disallowed ways.

#### Constraints and Limitations

- Excessive use of `goto` reduces readability (“spaghetti code”).
- Prefer `break`, `return`, or helper methods for normal control flow.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Exiting Nested Loops with `goto`

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 0; i < 3; i++)
        {
            for (int j = 0; j < 3; j++)
            {
                if (i == 1 && j == 1)
                    goto exit;
                Console.Write($"{i},{j} ");
            }
        }

    exit:
        Console.WriteLine("Exited at 1,1");
    }
}
```

**Expected Output**

```
0,0 0,1 0,2 1,0 Exited at 1,1
```

**Why this output occurs**

- `goto exit` jumps out of both loops to the `exit` label.

---

### Real-World Cases

- **Breaking out of multiple nested loops** — When a flag or method extraction would be awkward.
- **Legacy code** — Maintaining old code that uses `goto`.
- **Simple state machines** — Though `switch` expressions are preferred.

---

### References

- Jump statements — `goto` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-goto-statement
- The goto statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13105-the-goto-statement

---

## 8. Infinite Loops

### Definitions

**Core Definition**
An **infinite loop** is a loop whose terminating condition never becomes `false`, causing it to run indefinitely unless explicitly exited with `break`, `goto`, `return`, or an exception.

**Technical Definition**
Infinite loops are intentionally created with constructs such as `for(;;)`, `while(true)`, or `do { } while(true)`. They are a valid and common pattern in programs that run until an external event occurs, such as a game loop, a server accept loop, or a command-line REPL. The loop must contain a mechanism to exit (`break`, `return`, `goto`, or `throw`); otherwise, the program will hang.

**Beginner-Friendly Explanation**
An infinite loop is a **loop that never stops on its own**. You intentionally create one when you want the program to keep running until something specific happens—like the user typing “quit” or the game window closing.

---

### Purposes

- To implement event loops (UI frameworks, game engines).
- To run a server that accepts connections indefinitely.
- To create command-line REPLs that run until the user exits.
- To implement polling loops that wait for a condition.
- To serve as the outer loop in state machines.

---

### Syntax Rules and Structure

```
// Common infinite loop forms
for (;;) { ... }
while (true) { ... }
do { ... } while (true);
```

**Component breakdown**

- `for(;;)` — A `for` loop with no condition (always `true`).
- `while(true)` — A `while` loop with a literal `true` condition.
- `do { } while(true)` — A `do...while` with a literal `true` condition.

#### Syntax Rules

- Must contain a `break`, `return`, `goto`, or `throw` to exit.
- Without an exit, the loop runs forever (or until the process is terminated).

#### Constraints and Limitations

- Can hang the application or consume CPU if not designed carefully.
- Should include a way to terminate for clean shutdown.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Infinite Loop with `break`

```csharp
using System;

class Program
{
    static void Main()
    {
        int count = 0;
        while (true)
        {
            Console.WriteLine($"Iteration {count}");
            count++;
            if (count >= 3)
                break;
        }
        Console.WriteLine("Loop exited");
    }
}
```

**Expected Output**

```
Iteration 0
Iteration 1
Iteration 2
Loop exited
```

**Why this output occurs**

- `while (true)` is infinite, but `break` exits after three iterations.

---

### Real-World Cases

- **Game loops** — `while (gameRunning)` with a `break` when the user quits.
- **Server loops** — `while (true) { acceptConnection(); }` with a shutdown signal.
- **REPLs** — Reading and processing commands until “exit”.

---

### References

- Iteration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements
- Jump statements — `break` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-break-statement

---

## 9. Nested Loops

### Definitions

**Core Definition**
A **nested loop** is a loop placed **inside the body** of another loop. The inner loop executes completely for each iteration of the outer loop.

**Technical Definition**
Nested loops consist of an **outer loop** and one or more **inner loops**, where the inner loop is contained entirely within the outer loop. The total number of iterations is the product of the iteration counts of the loops (for simple nesting). `break` and `continue` affect only the **innermost** enclosing loop. Excessively deep nesting (more than 2–3 levels) reduces readability and is often refactored into helper methods or flattened algorithms.

**Beginner-Friendly Explanation**
A nested loop is a **loop inside a loop**. For every single step of the outer loop, the inner loop runs all the way through. It is like the hour and minute hands of a clock: for each hour, the minute hand goes around the full circle.

---

### Purposes

- To process two-dimensional data (matrices, grids, tables).
- To generate combinations or permutations.
- To implement algorithms like bubble sort or matrix multiplication.
- To traverse hierarchical data (nested collections).
- To work with coordinates or board games.

---

### Syntax Rules and Structure

```
for (outerInitializer; outerCondition; outerIterator)
{
    for (innerInitializer; innerCondition; innerIterator)
    {
        // inner body
    }
    // outer body
}
```

**Component breakdown**

- `outer...` — The outer loop.
- `inner...` — The inner loop, executed fully for each outer iteration.

#### Syntax Rules

- The inner loop executes completely for each outer iteration.
- `break` and `continue` affect only the innermost loop.
- Loop variables must have different names (or be in different scopes).

#### Constraints and Limitations

- Deep nesting (> 2–3 levels) harms readability.
- Total iterations = outer × inner (can explode quickly).
- `break` cannot exit multiple levels; use a flag, `goto`, or a helper method.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiplication Table

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 1; i <= 3; i++)
        {
            for (int j = 1; j <= 3; j++)
            {
                Console.Write($"{i * j} ");
            }
            Console.WriteLine();
        }
    }
}
```

**Expected Output**

```
1 2 3
2 4 6
3 6 9
```

**Why this output occurs**

- The outer loop controls rows; the inner loop controls columns; the product is printed for each cell.

---

#### Example 2: `break` in Nested Loops

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 0; i < 3; i++)
        {
            for (int j = 0; j < 3; j++)
            {
                if (i == 1 && j == 1)
                    break;
                Console.Write($"{i},{j} ");
            }
        }
        Console.WriteLine();   // Output: 0,0 0,1 0,2 1,0 2,0 2,1 2,2
    }
}
```

**Expected Output**

```
0,0 0,1 0,2 1,0 2,0 2,1 2,2
```

**Why this output occurs**

- `break` exits only the inner loop at `1,1`; the outer loop continues.

---

### Real-World Cases

- **Matrix operations** — Multiplying matrices, transposing.
- **Image processing** — Iterating pixels in a 2D grid.
- **Game boards** — Checking neighbouring cells.
- **Combinatorial algorithms** — Generating pairs or triples.

---

### References

- Iteration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements
- Jump statements — `break` — C# reference (nested loops) - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements#the-break-statement

---

## 10. Loop Performance

### Definitions

**Core Definition**
Loop performance refers to the **execution speed and resource consumption** of loops, which varies based on the loop construct, the collection type, the work inside the loop, and the runtime version.

**Technical Definition**
Loop performance in C# is influenced by several factors: the **loop construct** (`for`, `foreach`, `while`), the **data structure** (array, `List<T>`, `IEnumerable<T>`), **bounds-check elimination** by the JIT compiler, **enumerator allocation** for `foreach` over interfaces, and **loop-invariant code** that can be hoisted. Benchmarks show that **`foreach` is fastest over arrays** in modern .NET, while **`for` is most efficient on `List<T>`**; storing `Length` or `Count` in a variable does **not** improve performance and may decrease it. For performance-critical `List<T>` iteration, accessing the internal array via `CollectionsMarshal.AsSpan(list)` is the fastest approach. Loop unrolling and caching of invariant expressions can yield up to approximately **10% improvement** depending on the loop construct and data structure.

**Beginner-Friendly Explanation**
Loop performance is **how fast your loop runs**. The difference between `for` and `foreach` is usually tiny, and modern C# is very good at optimising loops. The bigger wins come from **not doing slow work inside the loop**—like creating objects, concatenating strings, or running LINQ queries on every iteration.

---

### Purposes

- To choose the most efficient loop for a given data structure.
- To avoid unnecessary allocations inside loops.
- To understand when micro-optimisations matter.
- To write loops that the JIT compiler can optimise effectively.
- To profile and benchmark before optimizing.

---

### Guidelines and Structure (Performance Patterns)

```
// Prefer foreach over arrays (fastest in modern .NET)
foreach (int x in array) { ... }

// For List<T>, for is often fastest
for (int i = 0; i < list.Count; i++) { ... }

// For maximum List<T> performance, use Span
Span<int> span = CollectionsMarshal.AsSpan(list);
for (int i = 0; i < span.Length; i++) { ... }

// Avoid LINQ in hot loops
// Avoid string concatenation with += in loops (use StringBuilder)
// Avoid creating objects inside loops
```

**Component breakdown**

- **`foreach` over arrays** — JIT eliminates enumerator and bounds checks.
- **`for` over `List<T>`** — Avoids enumerator overhead.
- **`Span<T>`** — Provides direct access to the underlying array.
- **Avoid allocations** — Reuse objects, use `StringBuilder`.

#### Rules and Constraints

- **Measure before optimising** — Use BenchmarkDotNet or a profiler.
- **Prioritise readability** — Write clear loops first; optimise only hot paths.
- **Avoid LINQ in tight loops** — LINQ allocates iterators and delegates.
- **Cache loop-invariant expressions** — Hoist `Length` or `Count` only if the JIT does not already do so (modern JIT often hoists automatically).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `foreach` vs. `for` over an Array

```csharp
using System;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        int[] data = new int[1_000_000];
        for (int i = 0; i < data.Length; i++) data[i] = i;

        var sw = Stopwatch.StartNew();
        long sum1 = 0;
        foreach (int x in data) sum1 += x;
        sw.Stop();
        Console.WriteLine($"foreach: {sw.ElapsedMilliseconds} ms, sum={sum1}");

        sw.Restart();
        long sum2 = 0;
        for (int i = 0; i < data.Length; i++) sum2 += data[i];
        sw.Stop();
        Console.WriteLine($"for: {sw.ElapsedMilliseconds} ms, sum={sum2}");
    }
}
```

**Expected Output (approximate)**

```
foreach: 2 ms, sum=499999500000
for: 2 ms, sum=499999500000
```

**Why this output occurs**

- Modern .NET optimises both loops effectively for arrays; differences are negligible.

---

### Real-World Cases

- **High-throughput servers** — Minimising allocations per request.
- **Game engines** — Avoiding GC pauses in the update loop.
- **Data processing** — Using `Span<T>` for large numeric arrays.

---

### References

- C# Array and List Fastest Loop in 2026 — NDepend Blog - https://blog.ndepend.com/c-array-and-list-fastest-loop/
- Performance Analysis of Loops in the C# Programming Language — Sinteza - https://portal.sinteza.singidunum.ac.rs
- Performance Improvements in .NET 10 — .NET Blog - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/

---

## 11. Loop Best Practices

### Definitions

**Core Definition**
Loop best practices are the guidelines that ensure loops are **correct, readable, maintainable, and performant**.

**Technical Definition**
Best practices for loops derive from Microsoft’s C# coding conventions, the .NET Framework Design Guidelines, and community standards. Key recommendations include: **choosing the right loop for the task** (`for` for known counts, `foreach` for collections, `while` for unknown counts, `do...while` for at-least-once); **keeping loop bodies small and focused**; **avoiding off-by-one errors** by using `<` rather than `<=` for zero-based collections; **using `break` and `continue` sparingly**; **preferring `foreach` over manual index management when the index is not needed**; **avoiding modifying the collection inside a `foreach`**; **limiting nesting depth** (ideally ≤ 2–3 levels); and **profiling before micro-optimising**.

**Beginner-Friendly Explanation**
Best practices are the habits that keep loops from becoming buggy or unreadable: use the right loop for the job, keep the body simple, do not nest too deeply, and do not change a collection while you are looping over it.

---

### Purposes

- To prevent off-by-one and infinite-loop bugs.
- To improve readability and maintainability.
- To choose the most appropriate loop construct.
- To avoid performance pitfalls.
- To align with C# conventions and tooling.

---

### Guidelines and Structure

#### General Guidelines

```
// Choose the right loop
for (int i = 0; i < array.Length; i++) { ... }   // known count, need index
foreach (var item in collection) { ... }          // collection, no index needed
while (condition) { ... }                        // unknown count, pre-test
do { ... } while (condition);                    // at-least-once

// Keep loop bodies focused
// Avoid deep nesting (≤ 2–3 levels)
// Do not modify a collection inside foreach
// Use break/continue sparingly
// Prefer early exit for search loops
```

**Component breakdown**

- **Right loop** — Match the construct to the iteration pattern.
- **Small bodies** — Extract complex logic into methods.
- **Shallow nesting** — Flatten with helper methods or LINQ.
- **No collection mutation** — In `foreach`, adding/removing elements invalidates the enumerator.
- **Clear exit** — Avoid complex `break`/`continue` logic.

#### Rules and Constraints

- **Do** use `foreach` when you do not need the index.
- **Do** use `for` when you need the index or are iterating a `List<T>` for performance.
- **Do** initialize loop variables correctly and update them toward the terminating condition.
- **Do** use `<` for zero-based collections to avoid out-of-range errors.
- **Do not** nest more than 2–3 levels without refactoring.
- **Do not** modify a collection while iterating with `foreach`.
- **Do not** use `goto` for normal loop control; reserve it for escaping deep nesting.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Choosing the Right Loop

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 10, 20, 30 };

        // foreach: no index needed
        foreach (int n in numbers)
            Console.WriteLine(n);

        // for: index needed
        for (int i = 0; i < numbers.Count; i++)
            Console.WriteLine($"{i}: {numbers[i]}");
    }
}
```

**Expected Output**

```
10
20
30
0: 10
1: 20
2: 30
```

**Why this output occurs**

- `foreach` is used when the index is not needed; `for` when the index is needed.

---

#### Example 2: Avoiding Collection Mutation in `foreach`

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        var items = new List<int> { 1, 2, 3 };

        // BAD: modifying the collection inside foreach
        // foreach (int item in items) { if (item == 2) items.Remove(item); }

        // GOOD: collect items to remove, then remove after the loop
        var toRemove = new List<int>();
        foreach (int item in items)
        {
            if (item == 2) toRemove.Add(item);
        }
        foreach (int item in toRemove)
            items.Remove(item);

        Console.WriteLine(string.Join(", ", items));   // 1, 3
    }
}
```

**Expected Output**

```
1, 3
```

**Why this output occurs**

- The collection is not modified during the `foreach`; removals happen afterward.

---

### Real-World Cases

- **Code reviews** — Loop nesting and collection mutation are common review criteria.
- **Performance tuning** — Choosing `foreach` vs. `for` based on the data structure.
- **Algorithm implementation** — Using the appropriate loop for the problem.

---

### References

- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions
- Iteration statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements
- Jump statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements
- C# Array and List Fastest Loop in 2026 — NDepend Blog - https://blog.ndepend.com/c-array-and-list-fastest-loop/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `foreach` over arrays | **Optimised by JIT** in modern .NET; fastest for arrays. |
| `for` over `List<T>` | Often fastest for `List<T>`; `Span<T>` via `CollectionsMarshal.AsSpan` is faster still. |
| `goto` | **Discouraged** for normal control flow; acceptable for escaping deep nesting. |
| Modifying collection in `foreach` | **InvalidOperationException** at runtime; use a snapshot or index-based loop. |
| Infinite loops | **Valid pattern** for event loops; must include an exit. |
| Loop unrolling | **JIT-dependent**; manual unrolling can yield ~10% improvement in some cases. |
| `Parallel.For`/`Parallel.ForEach` | **Not covered here**; use for CPU-bound parallel loops. |
| `await foreach` | **C# 8.0+**; for asynchronous streams (`IAsyncEnumerable<T>`). |

---

## References (Consolidated)

- Iteration statements — `for`, `foreach`, `do`, and `while` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements
- Jump statements — `break`, `continue`, `return`, and `goto` — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements
- The for statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1386-the-for-statement
- The foreach statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1387-the-foreach-statement
- The while statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1388-the-while-statement
- The do statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#1389-the-do-statement
- The break statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13102-the-break-statement
- The continue statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13103-the-continue-statement
- The goto statement — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements#13105-the-goto-statement
- C# Array and List Fastest Loop in 2026 — NDepend Blog - https://blog.ndepend.com/c-array-and-list-fastest-loop/
- Performance Analysis of Loops in the C# Programming Language — Sinteza - https://portal.sinteza.singidunum.ac.rs
- Performance Improvements in .NET 10 — .NET Blog - https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/
- C# coding conventions — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions

---