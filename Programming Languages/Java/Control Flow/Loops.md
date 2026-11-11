# Java Loops: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A loop is a control flow statement that repeatedly executes a block of code as long as a specified condition remains true. Java provides four loop constructs: the traditional `for` loop, the enhanced `for` loop (for-each), the `while` loop, and the `do...while` loop.

**Technical Definition:** The Java Language Specification (JLS) defines four iteration statements: the basic `for` statement, the enhanced `for` statement, the `while` statement, and the `do` statement. The `for` statement provides a compact way to iterate over a range of values, executing an initialization expression once, then repeatedly evaluating a termination condition, executing the body, and performing an update expression. The enhanced `for` statement (for-each) is designed for iteration through Collections and arrays, initialising a local variable to successive elements on each iteration. The `while` statement continually executes a block while a boolean expression is true, evaluating the condition at the top of the loop. The `do...while` statement evaluates its expression at the bottom, guaranteeing at least one execution of the body. Loops may be nested, controlled by `break` and `continue` statements, optionally with labels for multi-level control.

**Beginner-Friendly Explanation:** Imagine you need to write "Hello" on a hundred envelopes. Doing this by hand, one line at a time, would be tedious and error-prone. A loop is like telling a helper: "Write 'Hello' on each envelope until you've done all of them." You specify the starting point (the first envelope), the stopping condition (when there are no more envelopes), and how to move to the next one (pick up the next envelope). Java gives you four different ways to write these instructions, each suited to slightly different situations. The `for` loop is best when you know exactly how many times to repeat. The enhanced `for` is perfect for going through every item in a list or array. The `while` loop is useful when you repeat until something happens, without knowing the count in advance. And the `do...while` loop is for when you must do something at least once before checking whether to continue.

### Key Characteristics

- **Iteration Control:** Loops repeat a block of statements based on a boolean condition.
- **Four Constructs:** `for` (traditional), enhanced `for` (for-each), `while`, and `do...while`.
- **Flexible Termination:** Conditions can be evaluated before (`while`, `for`) or after (`do...while`) each iteration.
- **Early Exit:** `break` terminates the loop immediately; `continue` skips the current iteration.
- **Labeled Control:** Labels enable `break` and `continue` to target outer loops in nested structures.
- **Infinite Loops:** Omitting the termination condition (`for(;;)`, `while(true)`) creates an infinite loop, useful for event loops and servers when an exit mechanism exists.
- **Scope-Limited Variables:** Variables declared in a `for` initialisation are scoped to the loop body.

### Prerequisites

- Basic Java syntax (variables, operators, statements)
- Understanding of boolean expressions and conditional statements
- Familiarity with arrays and collections (for enhanced `for` loops)
- Knowledge of the `break` and `continue` keywords

### Related Programming Areas

- **Control Flow:** Conditionals (`if`, `switch`) and branching.
- **Collections Framework:** The enhanced `for` loop iterates over `Iterable` collections.
- **Arrays:** The enhanced `for` loop simplifies array traversal.
- **Streams API:** An alternative to loops for declarative data processing.
- **Concurrency:** Loops in threads, spin-waits, and event loops.

---

## Core Concepts / Features

---

## 1. `for` Loop (Traditional)

### Definitions

**Core Definition:** The traditional `for` loop is a compact iteration statement that executes an initialization, evaluates a termination condition, and performs an update after each iteration.

**Technical Definition:** "The `for` statement provides a compact way to iterate over a range of values". Its general form is `for (initialization; termination; increment) { statement(s) }`. The initialization expression executes once at the start; the termination expression is evaluated before each iteration (the loop ends when it is `false`); the increment expression executes after each iteration. All three expressions are optional; `for(;;)` creates an infinite loop.

**Beginner-Friendly Explanation:** The traditional `for` loop is like a recipe with three parts: where to start, when to stop, and what to do after each step. It is ideal when you know exactly how many times you need to repeat something, or when you need a counter that changes in a predictable pattern.

### Purposes

- To iterate a known number of times (counted loops)
- To iterate over indices of an array or collection
- To provide a compact syntax for initialization, condition, and update
- To create infinite loops when needed (`for(;;)`)
- To enable multiple variable declarations and updates in the loop header

### Syntax Rules and Structure

**Complete General Syntax:**

```java
for (initialization; termination; increment) {
    // statement(s)
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `initialization` | Executes once; typically declares and initialises a loop variable. |
| `termination` | Boolean expression evaluated before each iteration; loop ends when `false`. |
| `increment` | Executes after each iteration; typically increments or decrements the loop variable. |
| `statement(s)` | The body executed on each iteration. |

**Syntax Rules:**

- All three expressions are optional (`for(;;)` is valid).
- Variables declared in the initialization are scoped to the loop body and the other expressions.
- The increment expression can be any statement, including method calls.
- Multiple variables can be declared and updated using commas (e.g., `for(int i = 0, j = 10; i < j; i++, j--)`).

**Constraints and Limitations:**

- The loop variable is not accessible after the loop unless declared outside.
- Improper termination conditions cause infinite loops.
- The `for` loop is not inherently safer or faster than other loops; choose based on readability.

### Annotated Complete Step-by-Step Code Examples

```java
// File: TraditionalForDemo.java
public class TraditionalForDemo {
    public static void main(String[] args) {
        // Step 1: Initialization — int i = 1 executes once
        // Step 2: Termination — i < 11 is checked before each iteration
        // Step 3: Increment — i++ executes after each iteration
        for (int i = 1; i < 11; i++) {
            System.out.println("Count is: " + i);
        }
    }
}
```

**Expected Output:**
```
Count is: 1
Count is: 2
Count is: 3
Count is: 4
Count is: 5
Count is: 6
Count is: 7
Count is: 8
Count is: 9
Count is: 10
```

**Step-by-Step Explanation:**

1. `int i = 1` initialises the loop variable once.
2. Before the first iteration, `i < 11` is `true`, so the body executes.
3. After the body, `i++` increments `i` to `2`.
4. Steps 2–3 repeat until `i` becomes `11`; then `i < 11` is `false` and the loop terminates.

**Example 2: Infinite Loop with `for(;;)`**

```java
// File: InfiniteForDemo.java
public class InfiniteForDemo {
    public static void main(String[] args) {
        // All three expressions omitted — infinite loop
        for (;;) {
            System.out.println("Running...");
            // Exit mechanism required
            if (Math.random() < 0.1) break;
        }
        System.out.println("Exited");
    }
}
```

**Expected Output (varies):**
```
Running...
Running...
...
Exited
```

**Step-by-Step Explanation:**

1. `for(;;)` omits initialization, termination, and increment, creating an infinite loop.
2. The loop runs indefinitely until the `break` condition (`Math.random() < 0.1`) is met.
3. The `break` statement exits the loop, and execution continues after it.

### Real-World Cases

- **Array Indexing:** Iterating over an array by index to access or modify elements.
- **Counted Repetition:** Repeating an action a fixed number of times (e.g., 10 iterations).
- **Nested Loops:** Iterating over a grid or matrix (outer loop for rows, inner for columns).
- **Infinite Event Loops:** Server accept loops and game main loops using `for(;;)`.

---

## 2. Enhanced `for` Loop (for-each)

### Definitions

**Core Definition:** The enhanced `for` loop (also called for-each) iterates over arrays and `Iterable` collections, assigning each element to a local variable in turn.

**Technical Definition:** The enhanced `for` statement has the form `for (declaration : expression) { statement(s) }`. When executed, the local variable is initialized, on each iteration, to successive elements of the `Iterable` or array produced by the expression. It is designed for iteration through Collections and arrays, making loops more compact and easy to read.

**Beginner-Friendly Explanation:** The enhanced `for` loop is like saying "for each item in this list, do the following." You do not need to manage an index or a counter — Java handles the iteration for you. It is the simplest way to visit every element in an array or collection.

### Purposes

- To iterate over every element of an array or `Iterable` collection
- To eliminate off-by-one errors and index management
- To improve readability for full traversal
- To serve as the idiomatic way to traverse collections

### Syntax Rules and Structure

**Complete General Syntax:**

```java
for (Type element : collectionOrArray) {
    // statement(s) using element
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The type of the elements (or a supertype). |
| `element` | A local variable holding the current element. |
| `collectionOrArray` | An array or an `Iterable` (e.g., `List`, `Set`). |
| `statement(s)` | The body executed for each element. |

**Syntax Rules:**

- The expression must be an array or an `Iterable`.
- The loop variable is local to the loop body and cannot be used to modify the underlying collection (for arrays, modifying the variable does not affect the array).
- Cannot iterate in reverse or skip elements with this form.

**Constraints and Limitations:**

- Does not expose the index of the current element.
- Cannot modify the collection structurally during iteration (throws `ConcurrentModificationException` for most collections).
- Not suitable when you need access to the index or need to remove elements.

### Annotated Complete Step-by-Step Code Examples

```java
// File: EnhancedForDemo.java
public class EnhancedForDemo {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

        // For each element in numbers, assign to 'item' and print
        for (int item : numbers) {
            System.out.println("Count is: " + item);
        }
    }
}
```

**Expected Output:**
```
Count is: 1
Count is: 2
Count is: 3
Count is: 4
Count is: 5
Count is: 6
Count is: 7
Count is: 8
Count is: 9
Count is: 10
```

**Step-by-Step Explanation:**

1. The array `numbers` is the expression.
2. On the first iteration, `item` is assigned `numbers[0]` (1), and the body executes.
3. On the second iteration, `item` is assigned `numbers[1]` (2), and so on.
4. The loop terminates after all elements have been processed.

### Real-World Cases

- **Summing Collections:** Computing the sum of all elements in a list.
- **Filtering:** Checking each element against a condition.
- **Printing:** Displaying every item in a collection.
- **Processing Arrays:** Reading every value in a data array.

---

## 3. `while` Loop

### Definitions

**Core Definition:** The `while` loop repeatedly executes a block of statements as long as a boolean condition is true, evaluating the condition before each iteration.

**Technical Definition:** "The `while` statement continually executes a block of statements while a particular condition is true. Its syntax can be expressed as: `while (expression) { statement(s) }`". The expression must return a boolean value; if it evaluates to `true`, the block executes, and the process repeats until the expression evaluates to `false`.

**Beginner-Friendly Explanation:** The `while` loop is like saying "keep doing this as long as the condition holds." You check the condition first; if it is true, you do the work and come back to check again. If it is false from the start, the body never runs.

### Purposes

- To repeat an action while a condition remains true
- To handle loops where the number of iterations is not known in advance
- To implement polling and waiting loops
- To create infinite loops (`while(true)`) with explicit exit conditions

### Syntax Rules and Structure

**Complete General Syntax:**

```java
while (condition) {
    // statement(s)
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | Boolean expression evaluated before each iteration. |
| `statement(s)` | The body executed while the condition is true. |

**Syntax Rules:**

- The condition must be a boolean expression.
- The condition is evaluated before the body; if initially `false`, the body never executes.
- `while(true)` creates an infinite loop.

**Constraints and Limitations:**

- Forgetting to update the loop variable can cause an infinite loop.
- The condition is checked before the body, so the body may execute zero times.

### Annotated Complete Step-by-Step Code Examples

```java
// File: WhileDemo.java
public class WhileDemo {
    public static void main(String[] args) {
        int count = 1;

        // Condition checked before each iteration
        while (count < 11) {
            System.out.println("Count is: " + count);
            count++; // Update the loop variable
        }
    }
}
```

**Expected Output:**
```
Count is: 1
Count is: 2
...
Count is: 10
```

**Step-by-Step Explanation:**

1. `count` is initialised to `1`.
2. The condition `count < 11` is checked; `1 < 11` is `true`, so the body executes.
3. `count` is incremented to `2`.
4. The condition is checked again; the process repeats until `count` is `11`, at which point the condition is `false` and the loop ends.

### Real-World Cases

- **Reading Input:** Reading lines from a file or stream until end-of-file.
- **Polling:** Checking a resource periodically until it becomes available.
- **Infinite Loops:** `while(true)` in server accept loops and event listeners.
- **Game Loops:** Running game logic while the game is active.

---

## 4. `do...while` Loop

### Definitions

**Core Definition:** The `do...while` loop executes its body first and then evaluates a boolean condition at the bottom, guaranteeing at least one execution of the body.

**Technical Definition:** "The Java programming language also provides a `do-while` statement, which can be expressed as follows: `do { statement(s) } while (expression);`". The difference between `do-while` and `while` is that `do-while` evaluates its expression at the bottom of the loop, so the statements within the `do` block are always executed at least once.

**Beginner-Friendly Explanation:** The `do...while` loop is like saying "do this first, then check if you should do it again." It guarantees that the action happens at least once, which is useful when you need to perform an operation before you can decide whether to repeat it.

### Purposes

- To guarantee at least one execution of the loop body
- To implement menus and input validation that must run once
- To handle situations where the condition depends on the result of the first execution

### Syntax Rules and Structure

**Complete General Syntax:**

```java
do {
    // statement(s)
} while (condition);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `do` | Keyword starting the loop body. |
| `statement(s)` | Body executed at least once. |
| `while (condition)` | Condition checked after each execution. |

**Syntax Rules:**

- The body always executes at least once.
- The condition is evaluated after the body; if `true`, the body repeats.
- A semicolon is required after the `while` clause.

**Constraints and Limitations:**

- The body is always executed, even if the condition is initially `false`.
- Less commonly used than `while`; prefer `while` when zero iterations are possible.

### Annotated Complete Step-by-Step Code Examples

```java
// File: DoWhileDemo.java
public class DoWhileDemo {
    public static void main(String[] args) {
        int count = 1;

        do {
            System.out.println("Count is: " + count);
            count++;
        } while (count < 11);
    }
}
```

**Expected Output:**
```
Count is: 1
Count is: 2
...
Count is: 10
```

**Step-by-Step Explanation:**

1. The body executes: prints `1`, increments `count` to `2`.
2. The condition `count < 11` is checked; `2 < 11` is `true`, so the body repeats.
3. The process continues until `count` becomes `11`; the condition is `false` and the loop ends.

### Real-World Cases

- **User Menus:** Displaying a menu and processing choices until the user exits.
- **Input Validation:** Reading input and validating it, repeating if invalid.
- **Retry Logic:** Attempting an operation at least once before checking for success.

---

## 5. Loop Control: `break` and `continue`

### Definitions

**Core Definition:** `break` terminates the innermost enclosing loop (or `switch`), while `continue` skips the remainder of the current iteration and proceeds to the next iteration.

**Technical Definition:** The `break` statement transfers control out of the enclosing loop or `switch`. The `continue` statement skips the rest of the loop body and proceeds with the next iteration (evaluating the condition for `while`/`do-while`, or the update expression for `for`). Both can carry an optional label to target an outer statement.

**Beginner-Friendly Explanation:** `break` is like pressing the emergency stop button — the loop ends immediately. `continue` is like skipping to the next song — the current iteration stops, but the loop carries on with the next one.

### Purposes

- To exit a loop early when a condition is met (`break`)
- To skip specific iterations without ending the loop (`continue`)
- To improve efficiency by avoiding unnecessary iterations
- To implement search algorithms that stop when found

### Syntax Rules and Structure

```java
// break — exits the loop
while (condition) {
    if (exitCondition) break;
    // ...
}

// continue — skips to next iteration
for (int i = 0; i < n; i++) {
    if (skipCondition) continue;
    // ...
}
```

**Syntax Rules:**

- `break` and `continue` apply to the innermost enclosing loop by default.
- With a label, they target the labelled loop.
- `break` is also used in `switch` statements to prevent fall-through.
- `continue` is not valid outside a loop.

**Constraints and Limitations:**

- Overuse of `break` and `continue` can reduce readability.
- `break` in a `switch` inside a loop exits the `switch`, not the loop (unless labelled).

### Annotated Complete Step-by-Step Code Examples

```java
// File: BreakContinueDemo.java
public class BreakContinueDemo {
    public static void main(String[] args) {
        System.out.println("--- break example ---");
        for (int i = 1; i <= 10; i++) {
            if (i == 5) break; // Exit loop when i is 5
            System.out.println("i = " + i);
        }

        System.out.println("--- continue example ---");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) continue; // Skip even numbers
            System.out.println("i = " + i);
        }
    }
}
```

**Expected Output:**
```
--- break example ---
i = 1
i = 2
i = 3
i = 4
--- continue example ---
i = 1
i = 3
i = 5
i = 7
i = 9
```

**Step-by-Step Explanation:**

1. In the `break` example, the loop would normally run from 1 to 10; when `i` reaches 5, `break` exits the loop entirely.
2. In the `continue` example, even values trigger `continue`, which skips the print statement and proceeds to the next iteration.

### Real-World Cases

- **Search:** `break` when the target element is found.
- **Filtering:** `continue` to skip elements that do not meet a criterion.
- **Input Validation:** `break` out of a loop when valid input is received.

---

## 6. Labels

### Definitions

**Core Definition:** A label is an identifier followed by a colon (`label:`) that can be applied to any statement, enabling `break` and `continue` to target outer loops in nested structures.

**Technical Definition:** "Any statement can have a label. If a `break` statement has a label, it must be the label of an enclosing statement. If a `continue` statement has a label, it must be the label of an enclosing loop". A labelled `break` terminates the labelled statement; a labelled `continue` skips the current iteration of the labelled loop.

**Beginner-Friendly Explanation:** Labels are like naming a specific loop so you can tell `break` or `continue` exactly which loop you mean. Without labels, `break` only exits the innermost loop. With a label, you can exit or skip an outer loop from inside a nested loop.

### Purposes

- To break out of multiple nested loops at once
- To continue an outer loop from within an inner loop
- To provide precise control in complex nested structures
- To improve readability in search and matrix algorithms

### Syntax Rules and Structure

```java
outer: // label
for (int i = 0; i < 5; i++) {
    for (int j = 0; j < 5; j++) {
        if (condition) break outer;  // exits both loops
        if (condition2) continue outer; // skips to next i
    }
}
```

**Syntax Rules:**

- A label is an identifier followed by a colon.
- The labelled statement can be any statement, but labels are typically used on loops.
- A labelled `break` must reference an enclosing labelled statement.
- A labelled `continue` must reference an enclosing labelled loop.

**Constraints and Limitations:**

- Labels reduce readability if overused or used in simple loops.
- Labels are not required for single-level loops.

### Annotated Complete Step-by-Step Code Examples

```java
// File: LabelDemo.java
public class LabelDemo {
    public static void main(String[] args) {
        outer: // label for the outer loop
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 5; j++) {
                if (i == 2 && j == 2) {
                    System.out.println("Breaking out at i=2, j=2");
                    break outer; // exits the outer loop
                }
                System.out.println("i=" + i + ", j=" + j);
            }
        }
        System.out.println("After loops");
    }
}
```

**Expected Output:**
```
i=0, j=0
i=0, j=1
i=0, j=2
i=0, j=3
i=0, j=4
i=1, j=0
i=1, j=1
i=1, j=2
i=1, j=3
i=1, j=4
i=2, j=0
i=2, j=1
Breaking out at i=2, j=2
After loops
```

**Step-by-Step Explanation:**

1. The outer loop is labelled `outer`.
2. When `i == 2` and `j == 2`, `break outer` terminates the entire outer loop, not just the inner loop.
3. Execution resumes after the outer loop.

### Real-World Cases

- **Matrix Search:** Finding an element in a 2D array and breaking out of both loops.
- **Nested Validation:** Exiting all validation loops when a critical error is found.
- **Game Boards:** Searching for a winning condition across rows and columns.

---

## 7. Nested Loops

### Definitions

**Core Definition:** A nested loop is a loop placed inside another loop, where the inner loop executes completely for each iteration of the outer loop.

**Technical Definition:** A nested loop consists of an outer loop and one or more inner loops. For every iteration of the outer loop, the inner loop runs through its entire cycle. The total number of iterations of the inner body is the product of the iteration counts of all enclosing loops.

**Beginner-Friendly Explanation:** Nested loops are like a clock with hours and minutes. For every hour (outer loop), the minute hand goes all the way around (inner loop). The inner work happens many times for each outer step.

### Purposes

- To process multi-dimensional data (matrices, grids, tables)
- To generate combinations and permutations
- To implement algorithms that require multiple levels of iteration
- To search through nested structures

### Syntax Rules and Structure

```java
for (int i = 0; i < rows; i++) {
    for (int j = 0; j < cols; j++) {
        // process cell (i, j)
    }
}
```

**Syntax Rules:**

- Inner loops can use the outer loop's variables.
- `break` and `continue` affect the innermost loop unless labelled.
- The total iterations = outer iterations × inner iterations (for simple nested loops).

**Constraints and Limitations:**

- Nested loops can be expensive (O(n²) or worse).
- Deep nesting reduces readability; consider refactoring into methods.

### Annotated Complete Step-by-Step Code Examples

```java
// File: NestedLoopDemo.java
public class NestedLoopDemo {
    public static void main(String[] args) {
        // Outer loop for rows
        for (int i = 1; i <= 3; i++) {
            // Inner loop for columns
            for (int j = 1; j <= 3; j++) {
                System.out.print("(" + i + "," + j + ") ");
            }
            System.out.println(); // New line after each row
        }
    }
}
```

**Expected Output:**
```
(1,1) (1,2) (1,3) 
(2,1) (2,2) (2,3) 
(3,1) (3,2) (3,3) 
```

**Step-by-Step Explanation:**

1. The outer loop runs for `i = 1, 2, 3`.
2. For each `i`, the inner loop runs for `j = 1, 2, 3`.
3. The body prints the coordinates `(i, j)`.
4. A newline is printed after each outer iteration.

### Real-World Cases

- **Matrix Operations:** Multiplication, transposition, and traversal.
- **Pattern Printing:** Generating star patterns and tables.
- **Brute-Force Algorithms:** Checking all pairs or combinations.
- **Game Boards:** Iterating over rows and columns.

---

## 8. Infinite Loops

### Definitions

**Core Definition:** An infinite loop is a loop whose termination condition never becomes false, causing it to run indefinitely unless terminated by an external mechanism (such as `break`, an exception, or interruption).

**Technical Definition:** An infinite loop is created by omitting the termination condition (`for(;;)`), using a constant `true` condition (`while(true)`, `do...while(true)`), or writing a condition that can never be satisfied. Infinite loops are legitimate when paired with an explicit exit mechanism (e.g., `break`, `return`, interrupt).

**Beginner-Friendly Explanation:** An infinite loop is a loop that never stops on its own. It is like a treadmill that keeps running until you press the stop button. In some programs, this is intentional — servers need to keep listening for requests forever. But if you forget to provide a way out, the program hangs.

### Purposes

- To implement event loops and server accept loops
- To keep a service running until explicitly shut down
- To create daemon threads that poll or listen indefinitely
- To express "loop until break" semantics clearly

### Syntax Rules and Structure

```java
// while(true) — recommended for readability
while (true) {
    if (exitCondition) break;
    // work
}

// for(;;) — compact, common in some codebases
for (;;) {
    if (exitCondition) break;
    // work
}
```

**Syntax Rules:**

- `while(true)` is the most readable form.
- `for(;;)` is equivalent and may be favoured in certain codebases.
- An exit mechanism (`break`, `return`, interrupt) is essential to avoid hanging.
- Empty infinite loops (no body work) are harmful and may be removed by the JIT compiler.

**Constraints and Limitations:**

- Forgetting an exit mechanism causes the program to hang.
- Infinite loops without blocking consume CPU cycles.
- Use `Thread.sleep()` or blocking operations to avoid CPU busy-waiting.

### Annotated Complete Step-by-Step Code Examples

```java
// File: InfiniteLoopDemo.java
public class InfiniteLoopDemo {
    public static void main(String[] args) {
        int count = 0;

        while (true) { // Intentional infinite loop
            System.out.println("Iteration " + ++count);
            if (count >= 5) {
                System.out.println("Exit condition met");
                break; // Explicit exit
            }
        }
        System.out.println("Loop exited");
    }
}
```

**Expected Output:**
```
Iteration 1
Iteration 2
Iteration 3
Iteration 4
Iteration 5
Exit condition met
Loop exited
```

**Step-by-Step Explanation:**

1. `while (true)` creates an infinite loop.
2. The body executes, incrementing `count`.
3. When `count >= 5`, `break` exits the loop.
4. Execution continues after the loop.

### Real-World Cases

- **Server Accept Loops:** Continuously accepting client connections.
- **Event Listeners:** Waiting for and handling events.
- **Game Main Loops:** Running game updates and rendering.
- **Daemon Threads:** Polling a queue or resource indefinitely.

---

## 9. Loop Performance

### Definitions

**Core Definition:** Loop performance refers to the efficiency of loop execution, influenced by the loop structure, operations inside the body, memory access patterns, and JIT compiler optimisations.

**Technical Definition:** The HotSpot server compiler optimises loops with an `int` counter, a constant stride, and a loop-invariant limit. Loops over arrays are optimised when the compiler can relate the counter limit to the array length, eliminating redundant range checks. Loop-invariant computations should be hoisted out of the loop, and object creation inside hot loops should be minimised to reduce GC pressure.

**Beginner-Friendly Explanation:** Loops are where programs spend most of their time, so making loops efficient makes the whole program faster. The key ideas are: don't do the same work repeatedly if you can do it once, don't create unnecessary objects inside the loop, and use the loop style that the JIT compiler optimises best.

### Purposes

- To reduce execution time and CPU usage
- To minimise memory allocation and GC pressure
- To help the JIT compiler generate optimal code
- To improve scalability for large data sets

### Key Performance Guidelines

| Guideline | Description |
|-----------|-------------|
| Hoist loop-invariant computations | Move expressions that do not change out of the loop. |
| Use primitive types | Avoid boxing/unboxing in hot loops. |
| Avoid object creation | Reuse objects or move creation outside the loop. |
| Use `int` counters for arrays | The server compiler optimises `int` counters with constant strides. |
| Prefer enhanced `for` for collections | Cleaner and often optimised by the JIT. |
| Exit early with `break` | Avoid unnecessary iterations. |
| Consider parallelisation | Use parallel streams for CPU-bound, independent iterations. |
| Avoid empty infinite loops | They may be optimised away or consume CPU. |

### Annotated Code Example

```java
// File: LoopPerformanceDemo.java
public class LoopPerformanceDemo {
    public static void main(String[] args) {
        int n = 1_000_000;
        long sum = 0;

        // BAD: repeated size() calls (if using a collection)
        // for (int i = 0; i < list.size(); i++) { ... }

        // GOOD: hoist loop-invariant computation
        int[] array = new int[n];
        for (int i = 0; i < n; i++) array[i] = i;

        // GOOD: int counter, constant stride, invariant limit
        for (int i = 0; i < array.length; i++) {
            sum += array[i];
        }

        System.out.println("Sum: " + sum);
    }
}
```

**Expected Output:**
```
Sum: 499999500000
```

**Step-by-Step Explanation:**

1. `array.length` is loop-invariant; the JIT can hoist it.
2. The `int` counter `i` with `i++` and limit `array.length` matches the JIT's preferred pattern.
3. The loop avoids object creation and method calls in the hot path.

### Real-World Cases

- **Numerical Computing:** Matrix multiplication and array summation.
- **Data Processing:** Iterating over large collections with minimal allocation.
- **Web Servers:** Request processing loops where per-request allocation matters.

---

## 10. Loop Best Practices

### Definitions

**Core Definition:** Loop best practices are guidelines for writing loops that are correct, readable, efficient, and maintainable.

**Technical Definition:** Best practices derive from the JLS, Oracle tutorials, and JIT optimisation guidance. Key principles include choosing the right loop for the task, limiting variable scope, avoiding unnecessary nesting, providing exit mechanisms for infinite loops, and using labels judiciously.

**Beginner-Friendly Explanation:** Best practices help you write loops that do what you intend, are easy for others to read, and run efficiently. They prevent classic bugs like off-by-one errors, infinite loops, and unnecessary work.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| Choose the right loop | `for` for counted loops, enhanced `for` for collections, `while` for unknown counts, `do...while` for at-least-once. |
| Declare loop variables in the loop | Limits scope and reduces errors. |
| Prefer enhanced `for` for collections | More readable and eliminates index management. |
| Avoid deep nesting | Extract inner loops into methods for readability. |
| Use labels sparingly | Only when breaking/continuing outer loops is necessary. |
| Provide exit mechanisms for infinite loops | Always include `break`, `return`, or interrupt handling. |
| Hoist invariant computations | Move work out of the loop when possible. |
| Avoid modifying collections during for-each | Use `Iterator.remove()` or collect items to remove later. |
| Prefer `while(true)` over `for(;;)` | Better readability for intentional infinite loops. |
| Test boundary conditions | Verify loops with zero iterations, one iteration, and many iterations. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: LoopBestPractices.java
import java.util.*;

public class LoopBestPractices {
    public static void main(String[] args) {
        // BEST PRACTICE: Enhanced for for collections
        List<String> names = List.of("Alice", "Bob", "Charlie");
        for (String name : names) {
            System.out.println(name);
        }

        // BEST PRACTICE: Traditional for with local variable when index needed
        for (int i = 0; i < names.size(); i++) {
            System.out.println(i + ": " + names.get(i));
        }

        // BEST PRACTICE: while for unknown iteration count
        int attempts = 0;
        while (attempts < 3) {
            System.out.println("Attempt " + (attempts + 1));
            attempts++;
        }

        // BEST PRACTICE: do...while for at-least-once semantics
        Scanner scanner = new Scanner(System.in);
        String input;
        do {
            System.out.print("Enter 'quit' to exit: ");
            input = scanner.nextLine();
        } while (!input.equals("quit"));
    }
}
```

**Expected Output (varies based on input):**
```
Alice
Bob
Charlie
0: Alice
1: Bob
2: Charlie
Attempt 1
Attempt 2
Attempt 3
Enter 'quit' to exit: ...
```

**Step-by-Step Explanation:**

1. The enhanced `for` is used for simple traversal of a collection.
2. The traditional `for` is used when the index is needed.
3. The `while` loop handles an unknown number of attempts.
4. The `do...while` guarantees at least one prompt for user input.
5. Each loop choice matches the task's requirements.

### Real-World Cases

- **Collection Processing:** Enhanced `for` for lists and sets.
- **Index-Based Algorithms:** Traditional `for` for arrays when indices are needed.
- **Input Validation:** `do...while` for menus and prompts.
- **Server Loops:** `while(true)` with proper exit conditions.

---

## References

- The for Statement — Oracle Java Tutorials (JDK 8) — https://docs.oracle.com/javase/tutorial/java/nutsandbolts/for.html
- The while and do-while Statements — Oracle Java Tutorials (JDK 8) — https://docs.oracle.com/javase/tutorial/java/nutsandbolts/while.html
- The Java Language Specification, §14.14 The enhanced for statement — https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- The Java Spec: Statements (MIT mirror of early JLS) — http://web.mit.edu/mmadinot/Desktop/mmadinot/MacData/afs/sipb/user/marc/hotjava/doc/javaspec/javaspec_9.html
- PerformanceTechniques — OpenJDK Wiki — https://wiki.openjdk.org/pages/viewpage.action?pageId=11829375
- Java 中的无限循环 — Baeldung Chinese — https://www.baeldung-cn.com/infinite-loops-java
- 如何优化Java中的循环性能 — Yisu — https://m.yisu.com
- Java 中的 while(true)：深入分析及 JVM 视角的探索 — Huawei Cloud — https://bbs.huaweicloud.com
- 10 Java Performance Optimization Tips — GUVI — https://www.guvi.in
- Java 性能调优：从1198ms到239ms的八个隐形排查 — Tencent Cloud — https://cloud.tencent.com