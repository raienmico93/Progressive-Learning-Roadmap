# Java Branching Statements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Branching statements are control-flow statements that transfer execution from one part of a program to another, unconditionally or based on a condition. Java provides three branching statements — `break`, `continue`, and `return` — along with optional statement labels that extend `break` and `continue` to target outer enclosing statements.

**Technical Definition:** The Java Language Specification (JLS) classifies `break`, `continue`, `return`, `yield`, and `throw` as statements that "cause a transfer of control that may prevent normal completion" of the statements that contain them. A `break` statement transfers control out of an enclosing iteration or `switch` statement; a `continue` statement skips the remainder of the current iteration of an enclosing loop and begins a new one; a `return` statement transfers control to the invoker of the innermost enclosing constructor, method, or lambda expression. Labels are identifiers prefixed to statements (`label: statement`) and are used exclusively with `break` or `continue`; Java has no `goto` statement.

**Beginner-Friendly Explanation:** Imagine you are reading a long book (a loop). A **`break`** is like deciding to close the book immediately — you stop reading entirely. A **`continue`** is like skipping the rest of the current page and moving straight to the next page, without stopping the reading process. A **`return`** is like finishing the book, putting it away, and going to tell the person who asked you to read it what you found (the return value). A **label** is like placing a sticky note on a particular chapter, so you can tell `break` or `continue` exactly which chapter you mean when you are deep inside a nested section.

### Key Characteristics

- **Abrupt Completion:** All three statements cause abrupt completion of the statements that contain them; they never "complete normally".
- **`break` — Exit:** Terminates the innermost enclosing `switch`, `for`, `while`, or `do...while`; with a label, terminates any labelled statement.
- **`continue` — Skip:** Ends the current iteration of the innermost enclosing loop and begins the next; with a label, targets the labelled loop.
- **`return` — Exit Method:** Transfers control back to the caller; with a value for non-`void` methods, without a value for `void` methods.
- **Labels:** The only mechanism in Java for naming a statement so that `break` or `continue` can target it from a nested context.
- **`finally` Interaction:** A `return`, `break`, or `continue` inside a `finally` block can disrupt the transfer of control initiated by the `try` or `catch` block, and is strongly discouraged.

### Prerequisites

- Basic Java syntax (variables, operators, statements)
- Understanding of loops (`for`, `while`, `do...while`) and `switch`
- Familiarity with methods and return types
- Knowledge of `try-catch-finally` blocks

### Related Programming Areas

- **Loop Control:** `for`, enhanced `for`, `while`, and `do...while` loops.
- **Conditional Statements:** `if`, `switch` expressions/statements.
- **Methods:** Return types, `void` methods, and lambda expressions.
- **Exception Handling:** `finally` blocks and their interaction with branching statements.
- **Nested Control Flow:** Multi-level loops and labelled statements.

---

## Core Concepts / Features

---

## 1. `break`

### Definitions

**Core Definition:** The `break` statement transfers control out of the innermost enclosing `switch`, `for`, `while`, or `do...while` statement. With a label, it can exit any labelled enclosing statement.

**Technical Definition:** "The `break` statement has two forms: labeled and unlabeled". An unlabeled `break` "terminates the innermost `switch`, `for`, `while`, or `do-while` statement"; a labelled `break` "terminates an outer statement". The `break` statement "transfers control out of an enclosing iteration or switch statement" and, with a label, "can be used to simultaneously break out of multiple nested loops".

**Beginner-Friendly Explanation:** `break` is the "stop now" button. Inside a loop, it immediately ends the loop and jumps to the code after it. Inside a `switch`, it prevents fall-through to the next case.

### Purposes

- To exit a loop early when a condition is met (e.g., a search target is found)
- To prevent fall-through in `switch` statements
- To escape from deeply nested loops using a label
- To terminate an infinite loop from within its body

### Syntax Rules and Structure

**Complete General Syntax:**

```java
// Unlabeled break
break;

// Labeled break
break label;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `break` | Keyword that initiates the transfer of control. |
| `label` | Optional; the label of an enclosing statement to exit. |

**Syntax Rules:**

- An unlabeled `break` targets the innermost enclosing `switch`, `for`, `while`, or `do...while`.
- A labelled `break` targets the enclosing labelled statement with the matching identifier.
- The `break` target need not be a loop or `switch`; a labelled `break` can exit any labelled statement, including a plain block.
- Control resumes at the statement immediately following the terminated statement.

**Constraints and Limitations:**

- A `break` cannot transfer control out of a method (use `return` for that).
- A `break` inside a `finally` block can suppress an exception or a `return` from the `try`/`catch` block and should be avoided.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Unlabeled `break` in a Loop**

```java
// File: BreakLoopDemo.java
public class BreakLoopDemo {
    public static void main(String[] args) {
        int[] numbers = {32, 87, 3, 589, 12, 1076};
        int searchFor = 12;

        // Search for 12; break when found
        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] == searchFor) {
                System.out.println("Found " + searchFor + " at index " + i);
                break; // Exit the loop immediately
            }
        }
    }
}
```

**Expected Output:**
```
Found 12 at index 4
```

**Step-by-Step Explanation:**

1. The `for` loop iterates over the array.
2. When `numbers[i]` equals `12` (at index `4`), the `if` body executes.
3. `break` terminates the `for` loop immediately.
4. Control transfers to the statement after the loop (there is none), and the program ends.

**Example 2: Labelled `break` in Nested Loops**

```java
// File: LabeledBreakDemo.java
public class LabeledBreakDemo {
    public static void main(String[] args) {
        int[][] grid = {
            {32, 87, 3, 589},
            {12, 1076, 2000, 8},
            {622, 127, 77, 955}
        };
        int searchFor = 12;

        search: // Label the outer loop
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[i].length; j++) {
                if (grid[i][j] == searchFor) {
                    System.out.println("Found " + searchFor + " at " + i + ", " + j);
                    break search; // Exit BOTH loops
                }
            }
        }
    }
}
```

**Expected Output:**
```
Found 12 at 1, 0
```

**Step-by-Step Explanation:**

1. The outer loop is labelled `search`.
2. The inner loop scans each row. When `12` is found at `[1][0]`, the condition is met.
3. `break search` terminates the outer loop (and therefore the inner loop as well).
4. Without the label, `break` would only exit the inner loop, and the outer loop would continue searching the remaining rows unnecessarily.

### Real-World Cases

- **Array Search:** Stopping as soon as the target element is located.
- **Input Validation:** Exiting a loop when valid input is received.
- **Game Logic:** Ending a loop when a winning condition is detected.
- **Switch Fall-Through Prevention:** Using `break` to end each `case` in a traditional `switch`.

---

## 2. `continue`

### Definitions

**Core Definition:** The `continue` statement skips the remainder of the current iteration of an enclosing loop and proceeds immediately to the next iteration. With a label, it can target an outer loop.

**Technical Definition:** A `continue` statement "may occur only in a `while`, `do`, or `for` statement"; with no label, it "attempts to transfer control to the innermost enclosing" loop, which "immediately ends the current iteration and begins a new one". A labelled `continue` targets the enclosing labelled loop with the matching identifier. The `continue` target **must** be a `while`, `do`, or `for` statement; otherwise, a compile-time error occurs.

**Beginner-Friendly Explanation:** `continue` is the "skip this one" button. It stops the current iteration of the loop and jumps straight to the next iteration, without executing the rest of the loop body.

### Purposes

- To skip the remainder of the current iteration when a condition is met
- To filter elements in a loop without nesting the processing logic inside an `if`
- To skip to the next iteration of an outer loop from within an inner loop (using a label)
- To improve readability by handling exceptional cases first and continuing

### Syntax Rules and Structure

```java
// Unlabeled continue
continue;

// Labeled continue
continue label;
```

**Syntax Rules:**

- `continue` can appear only inside a `while`, `do`, or `for` loop.
- An unlabeled `continue` targets the innermost enclosing loop.
- A labelled `continue` targets the enclosing labelled loop with the matching identifier.
- For a `for` loop, the update expression is executed after `continue`; for `while`/`do...while`, the condition is re-evaluated.

**Constraints and Limitations:**

- A labelled `continue` must target a loop, not a `switch` or a plain block (unlike `break`).
- A `continue` inside a `finally` block can suppress exceptions and is discouraged.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Unlabeled `continue` to Skip Even Numbers**

```java
// File: ContinueDemo.java
public class ContinueDemo {
    public static void main(String[] args) {
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) {
                continue; // Skip the rest of the body for even numbers
            }
            System.out.println("Odd number: " + i);
        }
    }
}
```

**Expected Output:**
```
Odd number: 1
Odd number: 3
Odd number: 5
Odd number: 7
Odd number: 9
```

**Step-by-Step Explanation:**

1. The loop runs for `i = 1` through `10`.
2. When `i` is even, `continue` skips the `System.out.println` statement and proceeds to the next iteration.
3. Only odd numbers are printed.

**Example 2: Labelled `continue` in Nested Loops**

```java
// File: LabeledContinueDemo.java
public class LabeledContinueDemo {
    public static void main(String[] args) {
        outer: // Label the outer loop
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (j == 2) {
                    continue outer; // Skip to the next iteration of the OUTER loop
                }
                System.out.println("i=" + i + ", j=" + j);
            }
        }
    }
}
```

**Expected Output:**
```
i=1, j=1
i=2, j=1
i=3, j=1
```

**Step-by-Step Explanation:**

1. The outer loop is labelled `outer`.
2. When `j == 2`, `continue outer` skips the remainder of the inner loop and jumps to the next iteration of the outer loop.
3. For each `i`, only `j = 1` is printed; the inner loop never completes `j = 2` or `j = 3`.

### Real-World Cases

- **Data Filtering:** Skipping invalid or unwanted records during processing.
- **Input Validation:** Ignoring blank lines or malformed entries in a file.
- **Game Loops:** Skipping the rest of an update when a pause condition is met.
- **Nested Processing:** Skipping to the next outer element when an inner condition fails.

---

## 3. `return`

### Definitions

**Core Definition:** The `return` statement transfers control from a method, constructor, or lambda expression back to its invoker, optionally providing a value.

**Technical Definition:** "A `return` statement returns control to the invoker of a method or constructor". There are two kinds: `return;` (no value) and `return Expression;` (with a value). The `return` target is the innermost enclosing constructor, method, or lambda expression. "Execution of a `return` statement always completes abruptly". It is a compile-time error if a `return` with no value targets a non-`void` method, or if a `return` with a value targets a constructor or a `void` method. If the method contains `try` statements, any `finally` clauses execute before control actually returns to the caller.

**Beginner-Friendly Explanation:** `return` is the "I'm done, here's the answer" statement. It exits the method immediately. If the method is supposed to produce a value, `return` provides that value. If the method is `void`, `return` simply exits early.

### Purposes

- To provide the result of a non-`void` method to its caller
- To exit a `void` method early when a condition makes further work unnecessary
- To return from a lambda expression or constructor
- To implement guard clauses that handle edge cases first

### Syntax Rules and Structure

```java
// Return without a value (void methods)
return;

// Return with a value (non-void methods)
return expression;
```

**Syntax Rules:**

- A non-`void` method must return a value of a type assignable to the declared return type on every possible path, or a compile-time error occurs.
- A `void` method cannot return a value; `return;` is permitted to exit early.
- A constructor cannot return a value; `return;` is allowed but rarely needed.
- `finally` blocks execute before the method actually returns to the caller, and a `return` inside a `finally` block can override the original return value or suppress an exception.

**Constraints and Limitations:**

- A `return` statement cannot appear inside a static or instance initializer (though the compiler may allow it within a nested class method).
- A `return` inside a `finally` block can discard exceptions from the `try`/`catch` and is strongly discouraged.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Return with a Value**

```java
// File: ReturnValueDemo.java
public class ReturnValueDemo {
    static int square(int n) {
        return n * n; // Return the computed value
    }

    public static void main(String[] args) {
        int result = square(7);
        System.out.println("7 squared is " + result);
    }
}
```

**Expected Output:**
```
7 squared is 49
```

**Step-by-Step Explanation:**

1. `square(7)` is invoked; `n` is `7`.
2. `n * n` evaluates to `49`.
3. `return 49` transfers control back to `main`, and `result` receives `49`.
4. The value is printed.

**Example 2: Early `return` in a `void` Method (Guard Clause)**

```java
// File: GuardClauseDemo.java
public class GuardClauseDemo {
    static void printPositive(int n) {
        if (n <= 0) {
            System.out.println("Not positive");
            return; // Exit early — no further processing
        }
        System.out.println("Positive: " + n);
    }

    public static void main(String[] args) {
        printPositive(5);  // Positive: 5
        printPositive(-3); // Not positive
    }
}
```

**Expected Output:**
```
Positive: 5
Not positive
```

**Step-by-Step Explanation:**

1. For `5`, the guard condition `n <= 0` is `false`, so the method continues and prints `"Positive: 5"`.
2. For `-3`, the guard condition is `true`, so the method prints `"Not positive"` and executes `return;`, exiting immediately without printing the positive message.

### Real-World Cases

- **Computed Results:** Returning the result of a calculation.
- **Guard Clauses:** Exiting early when preconditions are not met.
- **Search Methods:** Returning `true` or `false` based on whether a target is found.
- **Factory Methods:** Returning newly constructed objects.

---

## 4. Labels

### Definitions

**Core Definition:** A label is an identifier followed by a colon (`label:`) prefixed to a statement, enabling `break` or `continue` to target that statement from within nested contexts.

**Technical Definition:** "Statements may have label prefixes (`Identifier: Statement`). The `Identifier` is declared to be the label of the immediately contained `Statement`". "Identifier statement labels are used with `break` or `continue` statements appearing anywhere within the labeled statement". Java has no `goto` statement; labels serve only `break` and `continue`. A labelled `break` can target any labelled statement; a labelled `continue` can target only a labelled loop.

**Beginner-Friendly Explanation:** A label is a sticky note you put on a loop or block. When you are deep inside nested loops and want to `break` or `continue` the outer loop, you refer to the sticky note by name.

### Purposes

- To enable `break` or `continue` to target an outer loop in nested structures
- To break out of multiple nested loops simultaneously with a single `break`
- To skip to the next iteration of an outer loop from within an inner loop
- To name a block of code for early exit via a labelled `break` (even without a loop)

### Syntax Rules and Structure

```java
label: statement
```

**Syntax Rules:**

- The label is an identifier followed by a colon.
- The labelled statement can be any statement, but labels are typically used on loops.
- A labelled `break` must reference an enclosing labelled statement.
- A labelled `continue` must reference an enclosing labelled loop.
- A label cannot be used with `return`; `return` exits the method, not a labelled statement.

**Constraints and Limitations:**

- Labels are not required for single-level loops.
- Overuse of labels can reduce readability; use them only when a single `break` or `continue` would be insufficient.
- A label marks only the immediately following statement; double-labelling (`a: b: for (...)`) means only the outer label marks the inner statement in some interpretations, and `continue a` from inside may be rejected by strict compilers.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Labelled `break` and `continue` Together**

```java
// File: LabelCombinedDemo.java
public class LabelCombinedDemo {
    public static void main(String[] args) {
        loop: // Label the outer loop
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (i == 2 && j == 2) {
                    System.out.println("Skipping i=2, j=2");
                    continue loop; // Skip to next i
                }
                if (i == 3 && j == 3) {
                    System.out.println("Breaking out at i=3, j=3");
                    break loop; // Exit both loops
                }
                System.out.println("i=" + i + ", j=" + j);
            }
        }
    }
}
```

**Expected Output:**
```
i=1, j=1
i=1, j=2
i=1, j=3
i=2, j=1
Skipping i=2, j=2
i=3, j=1
i=3, j=2
Breaking out at i=3, j=3
```

**Step-by-Step Explanation:**

1. For `i=2, j=2`, `continue loop` skips the rest of the inner loop and jumps to the next iteration of the outer loop (`i=3`).
2. For `i=3, j=3`, `break loop` terminates the outer loop entirely.
3. Both labelled `continue` and labelled `break` are used to control the outer loop from within the inner loop.

### Real-World Cases

- **Matrix Search:** Finding an element in a 2D array and breaking out of both loops.
- **Nested Validation:** Exiting all validation loops when a critical error is found.
- **Game Boards:** Checking rows and columns for a winning condition and stopping immediately.

---

## 5. Labelled `break`

### Definitions

**Core Definition:** A labelled `break` terminates the labelled enclosing statement, not just the innermost loop or `switch`.

**Technical Definition:** "A labelled `break` terminates an outer statement". The `break` statement "terminates the labeled statement; it does not transfer the flow of control to the label. Control flow is transferred to the statement immediately following the labeled (terminated) statement". The `break` target need not be a loop or `switch`; it can be any labelled statement.

**Beginner-Friendly Explanation:** A labelled `break` says "break out of **that** loop (the one with this name), not just the one I'm currently in." It is the only way to exit multiple nested loops at once in Java.

### Purposes

- To exit multiple nested loops simultaneously
- To exit a labelled block of code early (even if it is not a loop)
- To avoid using flag variables to signal that an outer loop should stop

### Syntax Rules and Structure

```java
label: for (...) {
    for (...) {
        if (condition) break label; // Exits the labelled loop
    }
}
```

**Syntax Rules:**

- The label must be the identifier of an enclosing labelled statement.
- Control resumes at the statement immediately after the labelled statement.
- The labelled statement can be a loop, a `switch`, or a plain block.

### Annotated Code Example

```java
// File: LabeledBreakBlockDemo.java
public class LabeledBreakBlockDemo {
    public static void main(String[] args) {
        String status = "unknown";

        block: { // A labelled block (not a loop)
            status = "processing";
            if (Math.random() < 0.5) {
                status = "failed";
                break block; // Exit the block early
            }
            status = "completed";
        }

        System.out.println("Status: " + status);
    }
}
```

**Expected Output (varies):**
```
Status: failed
```
or
```
Status: completed
```

**Step-by-Step Explanation:**

1. The block is labelled `block`.
2. If the random condition is `true`, `status` is set to `"failed"` and `break block` exits the labelled block.
3. If the condition is `false`, `status` is set to `"completed"` and the block ends normally.
4. A labelled `break` works on any labelled statement, not just loops.

### Real-World Cases

- **Multi-Level Search:** Breaking out of nested loops when a target is found.
- **Early Exit from Blocks:** Exiting a labelled block when an error condition is detected.
- **State Machines:** Terminating a nested state-processing block when a terminal state is reached.

---

## 6. Labelled `continue`

### Definitions

**Core Definition:** A labelled `continue` skips the remainder of the current iteration of the labelled loop and proceeds to its next iteration.

**Technical Definition:** "A `continue` statement with label Identifier attempts to transfer control to the enclosing labeled statement that has the same Identifier as its label; that statement, which is called the continue target, then immediately ends the current iteration and begins a new one". The `continue` target **must** be a `while`, `do`, or `for` statement, or a compile-time error occurs.

**Beginner-Friendly Explanation:** A labelled `continue` says "skip to the next iteration of **that** loop (the one with this name), not the one I'm currently in." It is useful when an inner loop detects a condition that means the entire outer iteration should be abandoned.

### Purposes

- To skip to the next iteration of an outer loop from within an inner loop
- To abandon the current outer iteration when an inner condition fails
- To avoid deeply nested conditional logic inside inner loops

### Syntax Rules and Structure

```java
outer: for (...) {
    for (...) {
        if (condition) continue outer; // Next iteration of outer loop
    }
}
```

**Syntax Rules:**

- The label must reference an enclosing labelled **loop** (`for`, `while`, or `do...while`).
- A labelled `continue` cannot target a `switch` or a plain block.
- The loop's update expression (for `for` loops) executes after `continue`.

### Annotated Code Example

```java
// File: LabeledContinueMatrixDemo.java
public class LabeledContinueMatrixDemo {
    public static void main(String[] args) {
        int[][] matrix = {
            {1, 2, 3},
            {4, -1, 6},
            {7, 8, 9}
        };

        row: // Label the outer loop
        for (int i = 0; i < matrix.length; i++) {
            int sum = 0;
            for (int j = 0; j < matrix[i].length; j++) {
                if (matrix[i][j] < 0) {
                    System.out.println("Row " + i + " has a negative value; skipping");
                    continue row; // Abandon this row, move to next row
                }
                sum += matrix[i][j];
            }
            System.out.println("Row " + i + " sum: " + sum);
        }
    }
}
```

**Expected Output:**
```
Row 0 sum: 6
Row 1 has a negative value; skipping
Row 2 sum: 24
```

**Step-by-Step Explanation:**

1. The outer loop is labelled `row`.
2. For row 1, the inner loop encounters `-1`; `continue row` skips the rest of row 1 and moves to row 2.
3. Rows 0 and 2 are processed normally, and their sums are printed.

### Real-World Cases

- **Data Validation:** Skipping an entire record when one field is invalid.
- **Matrix Processing:** Abandoning a row when an error is found.
- **Nested Filtering:** Skipping to the next outer element when an inner criterion fails.

---

## 7. Best Practices

### Definitions

**Core Definition:** Best practices for branching statements are guidelines for using `break`, `continue`, `return`, and labels correctly, safely, and readably.

**Technical Definition:** Best practices derive from the JLS, Oracle tutorials, and the SEI CERT Oracle Coding Standard for Java. Key principles include avoiding abrupt completion from `finally` blocks, using labels only when necessary, preferring `return` over `break` for exiting methods, and ensuring that branching statements are used for clarity rather than to compensate for poor structure.

**Beginner-Friendly Explanation:** Best practices help you use these statements without creating confusing or buggy code. They are like traffic rules for branching: use the right signal, in the right place, at the right time.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| **Avoid `break`/`continue`/`return` in `finally`** | These statements can override the `try`/`catch` outcome and suppress exceptions; SEI CERT ERR04-J prohibits abrupt completion from `finally`. |
| **Use `break` for early exit** | When a loop should stop because a condition is met, `break` is clearer than a flag variable. |
| **Use `continue` for filtering** | Handle exceptional cases first with `continue`, then process the normal case without nesting. |
| **Use `return` for guard clauses** | Exit a method early when preconditions fail, reducing nesting. |
| **Use labels sparingly** | Labels are powerful but can reduce readability; use them only when nested `break`/`continue` is genuinely clearer than alternatives. |
| **Prefer labelled `break` over flags** | A labelled `break` expresses "exit both loops" directly without a boolean flag. |
| **Ensure non-`void` methods always return** | Every path in a non-`void` method must return a value or throw; the compiler enforces this. |
| **Do not use `break`/`continue` to exit `switch` unless intended** | `break` inside a `switch` only exits the `switch`, not an enclosing loop; use labels if the loop should be exited. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: BranchingBestPractices.java
import java.util.List;

public class BranchingBestPractices {
    // BEST PRACTICE: Guard clause with return
    static String validate(String input) {
        if (input == null || input.isBlank()) {
            return "invalid"; // Early return avoids nesting
        }
        return "valid: " + input;
    }

    // BEST PRACTICE: continue for filtering
    static void process(List<Integer> numbers) {
        for (int n : numbers) {
            if (n < 0) continue; // Skip negatives, keep body flat
            System.out.println("Processing: " + n);
        }
    }

    // BEST PRACTICE: labelled break instead of a flag
    static boolean contains(int[][] grid, int target) {
        search:
        for (int[] row : grid) {
            for (int value : row) {
                if (value == target) {
                    break search; // Exit both loops without a flag
                }
            }
        }
        return false; // Placeholder — actual return logic omitted for brevity
    }

    public static void main(String[] args) {
        System.out.println(validate(null));          // invalid
        System.out.println(validate("Alice"));       // valid: Alice
        process(List.of(1, -2, 3, -4, 5));           // Processing: 1, 3, 5
    }
}
```

**Expected Output:**
```
invalid
valid: Alice
Processing: 1
Processing: 3
Processing: 5
```

**Step-by-Step Explanation:**

1. `validate` uses `return` as a guard clause to handle invalid input early, avoiding an `else` block.
2. `process` uses `continue` to skip negative numbers, keeping the main processing logic unindented.
3. `contains` uses a labelled `break` to exit both loops when the target is found, without a boolean flag.
4. Each statement is used for its idiomatic purpose.

### Real-World Cases

- **API Validation:** Guard clauses with `return` for invalid request parameters.
- **Data Processing:** `continue` for skipping malformed records in a batch.
- **Search Algorithms:** Labelled `break` for exiting nested search loops.
- **Code Reviews:** Checking that `finally` blocks do not contain branching statements that could mask exceptions.

---

## References

- Branching Statements — Oracle Java Tutorials (JDK 8) — https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html
- Chapter 14. Blocks, Statements, and Patterns — Java Language Specification, Java SE 24 Edition — https://docs.oracle.com/javase/specs/jls/se24/html/jls-14.html
- §14.17 The `return` Statement — Java Language Specification, Java SE 21 Edition — https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.17
- JLS 14.7 Labeled Statements — Java Language Specification (mirror) — https://docs.oracle.com/javase/specs/jls/se14/html/jls-14.html#jls-14.7
- JDK-4139794: Doubly labeled statements should not be continue targets — OpenJDK Bug System — https://bugs.openjdk.org/browse/JDK-4139794
- ERR04-J. Do not complete abruptly from a `finally` block — SEI CERT Oracle Coding Standard for Java — https://wiki.sei.cmu.edu/confluence/display/java/ERR04-J.+Do+not+complete+abruptly+from+a+finally+block
- CSCI-605 Advanced Object-Oriented Programming Concepts — Rochester Institute of Technology (break statement notes) — https://www.cs.rit.edu
- Java `continue` statement — JLS-related notes (via Stack Overflow discussions) — https://stackoverflow.com