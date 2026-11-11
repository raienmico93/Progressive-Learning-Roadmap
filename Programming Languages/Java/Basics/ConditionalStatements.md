# Java Conditional Statements: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Conditional statements are programming constructs that allow a program to execute different blocks of code depending on whether a given boolean condition evaluates to `true` or `false`. They form the foundation of decision-making in Java programs.

**Technical Definition:** In Java, conditional statements are control flow statements that evaluate a boolean expression and redirect execution based on the result. The primary constructs are the `if`, `if-else`, and `if-else-if` statements, the ternary conditional operator (`? :`), the `switch` statement, and the `switch` expression. Since Java 21, pattern matching for `switch` further extends the expressive power of conditional branching by allowing type patterns, guarded patterns (`when`), and record deconstruction within `switch` labels.

**Beginner-Friendly Explanation:** Conditional statements are like a fork in the road. Depending on a condition (like whether you have money), you take one path (buy coffee) or another (skip coffee). Java gives you several ways to express these decisions: simple `if` checks, `if-else` alternatives, the compact ternary operator, and the multi-way `switch` construct. Modern Java makes these even more powerful with switch expressions that can return values and pattern matching that can inspect the shape of your data.

### Key Characteristics

- **Boolean-Driven:** Every condition must evaluate to a `boolean` (or `Boolean`, which is auto-unboxed).
- **Short-Circuit Evaluation:** Logical operators `&&` and `||` short-circuit; the right operand is evaluated only when necessary.
- **Order Matters:** In `if-else-if` chains and `switch` statements with patterns, the first matching branch wins, so the order of conditions is critical.
- **Fall-Through in Legacy Switch:** Traditional colon-based `switch` cases fall through unless terminated by `break`; arrow-based cases eliminate fall-through entirely.
- **Exhaustiveness:** Switch expressions are required to be exhaustive (cover all possible values or include a `default`), reducing runtime surprises.
- **Pattern Matching (Java 21+):** `switch` and `instanceof` support type patterns, guarded patterns (`when`), and record patterns for deconstruction.

### Prerequisites

- Basic understanding of Java data types (`int`, `char`, `String`, `boolean`, enums)
- Familiarity with operators (`==`, `!=`, `&&`, `||`, `>`, `<`)
- Knowledge of Java methods and variable scope
- Java 14+ for switch expressions; Java 21+ for full pattern matching in `switch`

### Related Programming Areas

- **Control Flow:** Loops (`for`, `while`) and branching (`break`, `continue`, `return`) complement conditionals.
- **Object-Oriented Design:** Polymorphism can often replace conditional logic (strategy pattern, visitor pattern).
- **Functional Programming:** The ternary operator and pattern matching in `switch` encourage expression-oriented programming.
- **Error Handling:** Conditional checks often guard against invalid states before throwing exceptions.

### Core Concepts / Features

---

## 1. if

### Definitions

**Core Definition:** The `if` statement is the most basic conditional construct; it executes a block of code only when its boolean condition is `true`.

**Technical Definition:** The `if-then` statement tells the program to execute a certain section of code only if a particular test evaluates to `true`. The condition must be a boolean expression; Java does not permit integers or other types to be used directly as conditions (unlike C/C++).

**Beginner-Friendly Explanation:** `if` is like saying "If it is raining, take an umbrella." The action (taking an umbrella) only happens when the condition (raining) is true.

### Purposes

- To execute a block of statements conditionally based on a boolean expression
- To guard against invalid operations (e.g., checking for `null` before dereferencing)
- To implement simple decision logic

### Syntax Rules and Structure

```java
if (condition) {
    // statements executed when condition is true
}
```

**Syntax Rules:**
- The condition must be a `boolean` expression (primitive `boolean` or `Boolean`).
- Braces are optional for a single statement but recommended for clarity and safety.
- The condition is evaluated once.

**Constraints and Limitations:**
- Omitting braces can lead to subtle bugs when a second statement is later added.
- The condition cannot be a non-boolean value (e.g., `if (x)` is illegal for `int x`).

### Annotated Code Examples

**Example 1: Basic `if`**

```java
// File: IfDemo.java
public class IfDemo {
    public static void main(String[] args) {
        int score = 85;

        // Execute only if score is 90 or above
        if (score >= 90) {
            System.out.println("Excellent!");
        }

        System.out.println("Done.");
    }
}
```

**Expected Output:**
```
Done.
```

**Step-by-Step Explanation:** The condition `score >= 90` is `false` (85 is not ≥ 90), so the `if` block is skipped. Execution continues after the block, printing "Done."

**Example 2: `if` with a Guard Clause**

```java
// File: GuardClauseDemo.java
public class GuardClauseDemo {
    public static void process(String input) {
        // Guard clause: exit early if input is null
        if (input == null) {
            System.out.println("Input is null");
            return;
        }
        System.out.println("Processing: " + input);
    }

    public static void main(String[] args) {
        process(null);
        process("Hello");
    }
}
```

**Expected Output:**
```
Input is null
Processing: Hello
```

**Step-by-Step Explanation:** The guard clause checks `input == null` and returns early, avoiding a `NullPointerException`. This is a common best practice for reducing nesting.

**Example 3: `if` with a Compound Condition**

```java
// File: CompoundIfDemo.java
public class CompoundIfDemo {
    public static void main(String[] args) {
        int age = 25;
        boolean hasLicense = true;

        // Both conditions must be true
        if (age >= 18 && hasLicense) {
            System.out.println("Eligible to drive");
        }
    }
}
```

**Expected Output:**
```
Eligible to drive
```

**Step-by-Step Explanation:** The `&&` operator short-circuits; both `age >= 18` and `hasLicense` must be `true`.

### Real-World Cases

- **Input Validation:** Checking that a string is non-null and non-empty before processing.
- **Permission Checks:** Verifying that a user has the required role before granting access.
- **Range Checking:** Ensuring a number falls within an acceptable range.

---

## 2. else if

### Definitions

**Core Definition:** The `else if` construct allows chaining multiple mutually exclusive conditions, where each subsequent condition is evaluated only if all previous conditions were `false`.

**Technical Definition:** An `if-else-if` ladder evaluates conditions in order. Once a condition evaluates to `true`, its associated block executes, and all remaining conditions in the chain are skipped. The first true branch wins.

**Beginner-Friendly Explanation:** `else if` is like a series of questions: "Are you a child? No. Are you a teenager? No. Are you an adult? Yes — then you're an adult."

### Purposes

- To model multi-way decisions where only one outcome can occur
- To replace deeply nested `if` statements with a flat, readable chain
- To handle mutually exclusive categories (e.g., grade ranges, age groups)

### Syntax Rules and Structure

```java
if (condition1) {
    // block 1
} else if (condition2) {
    // block 2
} else if (condition3) {
    // block 3
} else {
    // fallback block
}
```

**Syntax Rules:**
- Conditions are evaluated top to bottom.
- Only the first true branch executes.
- The final `else` is optional but recommended for exhaustiveness.
- Order conditions from most specific to least specific to avoid shadowing.

**Constraints and Limitations:**
- Overly long chains can become hard to maintain; consider `switch` or polymorphism.
- A broad condition placed before a specific one will swallow the specific case.

### Annotated Code Examples

**Example 1: Grade Classifier**

```java
// File: ElseIfGradeDemo.java
public class ElseIfGradeDemo {
    public static void main(String[] args) {
        int score = 76;

        if (score >= 90) {
            System.out.println("Grade A");
        } else if (score >= 80) {
            System.out.println("Grade B");
        } else if (score >= 70) {
            System.out.println("Grade C");
        } else if (score >= 60) {
            System.out.println("Grade D");
        } else {
            System.out.println("Grade F");
        }
    }
}
```

**Expected Output:**
```
Grade C
```

**Step-by-Step Explanation:** `76 >= 90` is false, `76 >= 80` is false, `76 >= 70` is true, so "Grade C" is printed and the remaining branches are skipped.

**Example 2: Ordering Conditions Correctly**

```java
// File: OrderingDemo.java
public class OrderingDemo {
    public static void classify(int n) {
        // WRONG order: broad condition first swallows specific ones
        if (n > 0) {
            System.out.println("Positive");
        } else if (n > 100) {
            System.out.println("Large positive"); // unreachable!
        }
    }

    public static void classifyCorrectly(int n) {
        // CORRECT order: most specific first
        if (n > 100) {
            System.out.println("Large positive");
        } else if (n > 0) {
            System.out.println("Positive");
        }
    }

    public static void main(String[] args) {
        classifyCorrectly(150); // Large positive
        classifyCorrectly(5);   // Positive
    }
}
```

**Expected Output:**
```
Large positive
Positive
```

**Step-by-Step Explanation:** In the wrong version, any `n > 0` (including `n > 100`) matches the first branch, making the second branch unreachable. Always order from most specific to least specific.

**Example 3: `else if` with Enums**

```java
// File: ElseIfEnumDemo.java
public class ElseIfEnumDemo {
    enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

    public static void main(String[] args) {
        Day today = Day.SATURDAY;

        if (today == Day.SATURDAY || today == Day.SUNDAY) {
            System.out.println("Weekend");
        } else if (today == Day.MONDAY) {
            System.out.println("Start of work week");
        } else {
            System.out.println("Weekday");
        }
    }
}
```

**Expected Output:**
```
Weekend
```

**Step-by-Step Explanation:** The enum comparison uses `==` (safe for enums). `SATURDAY` matches the first condition.

### Real-World Cases

- **Pricing Tiers:** Determining a discount based on order size.
- **Tax Brackets:** Computing tax rate based on income ranges.
- **User Roles:** Assigning permissions based on role hierarchy.

---

## 3. else

### Definitions

**Core Definition:** The `else` clause provides a fallback block that executes when the preceding `if` (or `if-else-if` chain) condition evaluates to `false`.

**Technical Definition:** The `if-then-else` statement provides a secondary path of execution when an `if` clause evaluates to `false`.

**Beginner-Friendly Explanation:** `else` is the "otherwise" branch — what to do when the main condition is not met.

### Purposes

- To specify the default behaviour when no condition matches
- To ensure that all code paths are handled (exhaustiveness)
- To provide user-friendly error messages for invalid input

### Syntax Rules and Structure

```java
if (condition) {
    // then block
} else {
    // else block
}
```

**Syntax Rules:**
- `else` must immediately follow an `if` or `else if` block.
- An `else` cannot exist without a preceding `if`.
- Braces are optional for a single statement but recommended.

**Constraints and Limitations:**
- Overusing `else` can lead to deeply nested code; guard clauses (early returns) are often cleaner.

### Annotated Code Example

```java
// File: ElseDemo.java
public class ElseDemo {
    public static void main(String[] args) {
        int age = 16;

        if (age >= 18) {
            System.out.println("Adult");
        } else {
            System.out.println("Minor");
        }
    }
}
```

**Expected Output:**
```
Minor
```

**Step-by-Step Explanation:** `age >= 18` is false, so the `else` block executes.

### Real-World Cases

- **Form Validation:** Showing an error message when a field is invalid.
- **Default Configuration:** Applying default settings when no override is provided.
- **Fallback Logic:** Using a backup value when a primary source is unavailable.

---

## 4. Nested Conditionals

### Definitions

**Core Definition:** Nested conditionals are conditional statements placed inside the blocks of other conditional statements, creating multiple levels of decision-making.

**Technical Definition:** An `if` statement can contain another `if` statement within its `then` or `else` block. While legal, deep nesting reduces readability and maintainability; flattening techniques (guard clauses, combining conditions, extracting methods) are recommended.

**Beginner-Friendly Explanation:** Nested conditionals are like a decision tree: "If it is a weekday, then if it is morning, then if you have time, get coffee."

### Purposes

- To model complex, hierarchical decision logic
- To handle multi-dimensional conditions (e.g., user role *and* resource state)

### Syntax Rules and Structure

```java
if (outerCondition) {
    if (innerCondition) {
        // both true
    } else {
        // outer true, inner false
    }
} else {
    // outer false
}
```

**Syntax Rules:**
- Each `if` introduces a new scope.
- The `dangling else` problem: an `else` binds to the nearest unmatched `if`. Braces prevent ambiguity.

**Constraints and Limitations:**
- Deep nesting (3+ levels) is considered a code smell and should be refactored.

### Annotated Code Examples

**Example 1: Nested `if` for Multi-Factor Decisions**

```java
// File: NestedDemo.java
public class NestedDemo {
    public static void main(String[] args) {
        boolean isMember = true;
        double purchase = 150.0;

        if (isMember) {
            if (purchase > 100) {
                System.out.println("Member discount: 20%");
            } else {
                System.out.println("Member discount: 10%");
            }
        } else {
            System.out.println("No discount");
        }
    }
}
```

**Expected Output:**
```
Member discount: 20%
```

**Step-by-Step Explanation:** The outer `if` checks membership; the inner `if` checks purchase amount.

**Example 2: Flattening with Guard Clauses**

```java
// File: FlattenedDemo.java
public class FlattenedDemo {
    public static void process(boolean isMember, double purchase) {
        // Flattened: early return instead of nesting
        if (!isMember) {
            System.out.println("No discount");
            return;
        }
        if (purchase > 100) {
            System.out.println("Member discount: 20%");
            return;
        }
        System.out.println("Member discount: 10%");
    }

    public static void main(String[] args) {
        process(true, 150);  // Member discount: 20%
        process(false, 150); // No discount
    }
}
```

**Expected Output:**
```
Member discount: 20%
No discount
```

**Step-by-Step Explanation:** The guard clauses eliminate nesting, making the logic easier to follow.

### Real-World Cases

- **Access Control:** Checking role *and* resource ownership.
- **Game Logic:** Checking whether a move is valid *and* whether the player has enough resources.

---

## 5. Ternary Operator

### Definitions

**Core Definition:** The ternary operator (`? :`) is a compact, expression-based conditional that returns one of two values depending on a boolean condition.

**Technical Definition:** The ternary conditional operator has the form `condition ? expressionIfTrue : expressionIfFalse`. It is the only operator in Java that takes three operands. It can be used wherever an expression is expected (assignment, method argument, return statement).

**Beginner-Friendly Explanation:** The ternary operator is a one-line `if-else` that gives you a value. Instead of "if the score is high, set grade to A; otherwise set it to B," you write `grade = score >= 90 ? 'A' : 'B';`.

### Purposes

- To provide a concise alternative to simple `if-else` assignments
- To enable conditional values within expressions
- To improve readability for trivial decisions

### Syntax Rules and Structure

```java
result = condition ? valueIfTrue : valueIfFalse;
```

**Syntax Rules:**
- The condition must be boolean.
- Both branches must produce compatible types.
- Can be nested, but nesting reduces readability.
- The ternary operator is an expression, not a statement; it produces a value.

**Constraints and Limitations:**
- Overuse leads to unreadable code; use `if-else` for complex logic.
- Both branches are evaluated lazily (only the selected branch is evaluated).

### Annotated Code Examples

**Example 1: Basic Assignment**

```java
// File: TernaryDemo.java
public class TernaryDemo {
    public static void main(String[] args) {
        int num = 8;
        String msg = (num > 10) ? "Greater than 10" : "10 or less";
        System.out.println(msg);
    }
}
```

**Expected Output:**
```
10 or less
```

**Step-by-Step Explanation:** `num > 10` is false, so the value of the ternary expression is `"10 or less"`.

**Example 2: Ternary in Method Return**

```java
// File: TernaryReturnDemo.java
public class TernaryReturnDemo {
    public static String parity(int n) {
        return (n % 2 == 0) ? "even" : "odd";
    }

    public static void main(String[] args) {
        System.out.println(parity(4)); // even
        System.out.println(parity(7)); // odd
    }
}
```

**Expected Output:**
```
even
odd
```

**Step-by-Step Explanation:** The ternary expression is the return value, eliminating a temporary variable.

**Example 3: Nested Ternary (Use Sparingly)**

```java
// File: NestedTernaryDemo.java
public class NestedTernaryDemo {
    public static void main(String[] args) {
        int score = 85;
        String grade = (score >= 90) ? "A"
                     : (score >= 80) ? "B"
                     : (score >= 70) ? "C"
                     : "F";
        System.out.println(grade); // B
    }
}
```

**Expected Output:**
```
B
```

**Step-by-Step Explanation:** Nested ternaries can be readable for simple ladders but should be replaced by `switch` or `if-else` when logic grows.

### Real-World Cases

- **Configuration Defaults:** `String env = System.getenv("ENV") != null ? System.getenv("ENV") : "production";`
- **Null-Safe Values:** `String name = user != null ? user.getName() : "Guest";`
- **Simple Classification:** `String size = length < 10 ? "small" : "large";`

---

## 6. Switch Statement

### Definitions

**Core Definition:** The `switch` statement is a multi-way branching construct that transfers control to one of many `case` labels based on the value of an expression.

**Technical Definition:** Unlike `if-then` and `if-then-else`, the `switch` statement can have a number of possible execution paths. A `switch` works with `byte`, `short`, `char`, `int`, their boxed variants, enums, and `String`. The body is a switch block containing `case` and `default` labels. In the legacy colon form, execution falls through to subsequent cases unless a `break` statement terminates the branch.

**Beginner-Friendly Explanation:** `switch` is like a train station with multiple platforms. You look at your ticket (the expression's value) and go to the matching platform (case). Without `break`, you accidentally ride through to the next platform.

### Purposes

- To replace long `if-else-if` chains that compare a single value against constants
- To provide a more readable, declarative structure for multi-way branching
- To group multiple constants that share the same behaviour

### Syntax Rules and Structure

```java
switch (expression) {
    case constant1:
        // statements
        break;
    case constant2:
    case constant3:
        // shared statements
        break;
    default:
        // fallback
}
```

**Syntax Rules:**
- The expression must be of type `byte`, `short`, `char`, `int` (or boxed), `String`, or an enum.
- Case labels must be compile-time constants (literals, `static final` constants, enum constants).
- `break` exits the switch block; without it, execution falls through.
- `default` is optional and can appear anywhere (usually last).
- Multiple `case` labels can be stacked to share code.

**Constraints and Limitations:**
- Fall-through is a common source of bugs; always add `break` unless intentional.
- Cannot switch on `long`, `float`, `double`, or `boolean`.
- Cannot use non-constant expressions as case labels.

### Annotated Code Examples

**Example 1: Day-of-Week Name**

```java
// File: SwitchStatementDemo.java
public class SwitchStatementDemo {
    public static void main(String[] args) {
        int day = 3;
        String name;

        switch (day) {
            case 1: name = "Monday";    break;
            case 2: name = "Tuesday";   break;
            case 3: name = "Wednesday"; break;
            case 4: name = "Thursday";  break;
            case 5: name = "Friday";    break;
            case 6: name = "Saturday";  break;
            case 7: name = "Sunday";    break;
            default: name = "Invalid";  break;
        }

        System.out.println(name); // Wednesday
    }
}
```

**Expected Output:**
```
Wednesday
```

**Step-by-Step Explanation:** `day = 3` matches `case 3`, assigns "Wednesday", and `break` exits the switch.

**Example 2: Grouping Cases**

```java
// File: SwitchGroupingDemo.java
public class SwitchGroupingDemo {
    public static void main(String[] args) {
        char grade = 'B';
        String description;

        switch (grade) {
            case 'A':
                description = "Excellent";
                break;
            case 'B':
            case 'C':
                description = "Good";
                break;
            case 'D':
                description = "Passing";
                break;
            default:
                description = "Unknown";
        }

        System.out.println(description); // Good
    }
}
```

**Expected Output:**
```
Good
```

**Step-by-Step Explanation:** `'B'` matches, falls through to the `'C'` label (no `break` after `'B'`), and both share the "Good" assignment.

**Example 3: Intentional Fall-Through (Rare)**

```java
// File: SwitchFallThroughDemo.java
public class SwitchFallThroughDemo {
    public static void main(String[] args) {
        int month = 2;
        int days = 0;

        switch (month) {
            case 2:
                days = 28;
                // intentional fall-through for leap year check
            case 1: case 3: case 5: case 7:
            case 8: case 10: case 12:
                if (days == 0) days = 31;
                break;
            case 4: case 6: case 9: case 11:
                days = 30;
                break;
            default:
                System.out.println("Invalid month");
        }

        System.out.println("Days: " + days); // Days: 28
    }
}
```

**Expected Output:**
```
Days: 28
```

**Step-by-Step Explanation:** Intentional fall-through is occasionally useful but should be clearly commented.

### Real-World Cases

- **Command Dispatchers:** Routing commands to handlers based on a string.
- **State Machines:** Transitioning between states based on an enum.
- **Menu Systems:** Executing different actions for different menu choices.

---

## 7. Switch Expression

### Definitions

**Core Definition:** A switch expression is a modern form of `switch` (standard since Java 14) that returns a value and uses arrow labels (`->`) to eliminate fall-through and the need for `break`.

**Technical Definition:** A switch expression evaluates to a single value and can be used wherever an expression is expected (assignment, return, method argument). Arrow cases (`case L ->`) execute only the code to the right of the arrow, with no fall-through. For multi-statement blocks, the `yield` statement specifies the value of the switch expression. Switch expressions must be exhaustive: all possible values must be covered, or a `default` must be provided.

**Beginner-Friendly Explanation:** A switch expression is a `switch` that gives you an answer. Instead of assigning to a variable inside each case, the whole `switch` becomes the value. The arrow syntax makes it impossible to accidentally fall through.

### Purposes

- To produce a value directly from a multi-way decision
- To eliminate fall-through bugs with arrow labels
- To enforce exhaustiveness at compile time
- To enable cleaner, expression-oriented code

### Syntax Rules and Structure

```java
// Arrow form (single expression)
result = switch (expression) {
    case constant1 -> value1;
    case constant2, constant3 -> value2;
    default -> value3;
};

// Block form with yield
result = switch (expression) {
    case constant1 -> {
        // multiple statements
        yield value1;
    }
    default -> value3;
};
```

**Syntax Rules:**
- Arrow cases do not fall through.
- Multiple constants can share a case with commas: `case A, B ->`.
- For blocks, `yield` must be used to produce a value.
- The switch must be exhaustive (cover all possibilities or include `default`).
- Supported types include all legacy switch types plus pattern matching (Java 21+).

**Constraints and Limitations:**
- `break` is not used in arrow form; `yield` is used in blocks.
- Exhaustiveness is enforced; missing cases cause compile errors.

### Annotated Code Examples (Three Examples)

**Example 1: Day-of-Week Letter Count**

```java
// File: SwitchExpressionDemo.java
public class SwitchExpressionDemo {
    enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

    public static void main(String[] args) {
        Day day = Day.WEDNESDAY;

        int numLetters = switch (day) {
            case MONDAY, FRIDAY, SUNDAY -> 6;
            case TUESDAY -> 7;
            case THURSDAY, SATURDAY -> 8;
            case WEDNESDAY -> 9;
        };

        System.out.println(numLetters); // 9
    }
}
```

**Expected Output:**
```
9
```

**Step-by-Step Explanation:** The switch expression returns `9` for `WEDNESDAY`; no `break` or variable assignment inside the cases.

**Example 2: Switch Expression with `yield`**

```java
// File: SwitchYieldDemo.java
public class SwitchYieldDemo {
    public static void main(String[] args) {
        int x = 42;

        String result = switch (x) {
            case 1, 2, 3 -> "small";
            case 42 -> {
                String note = "answer";
                yield "The " + note; // yield returns the block's value
            }
            default -> "other";
        };

        System.out.println(result); // The answer
    }
}
```

**Expected Output:**
```
The answer
```

**Step-by-Step Explanation:** The block case uses `yield` to produce "The answer"; `yield` replaces `return` inside a switch expression block.

**Example 3: Exhaustive Switch Expression over an Enum**

```java
// File: SwitchExhaustiveDemo.java
public class SwitchExhaustiveDemo {
    enum Season { SPRING, SUMMER, FALL, WINTER }

    public static String activity(Season season) {
        return switch (season) {
            case SPRING -> "Planting";
            case SUMMER -> "Swimming";
            case FALL -> "Harvesting";
            case WINTER -> "Skiing";
            // No default needed — exhaustive over enum
        };
    }

    public static void main(String[] args) {
        System.out.println(activity(Season.SUMMER)); // Swimming
    }
}
```

**Expected Output:**
```
Swimming
```

**Step-by-Step Explanation:** Because all enum constants are covered, no `default` is required; the compiler verifies exhaustiveness.

### Real-World Cases

- **HTTP Status Mapping:** `String category = switch (status) { case 200, 201 -> "Success"; case 400, 404 -> "Client Error"; ... };`
- **Configuration Selection:** Returning different configuration objects based on an environment enum.
- **Mathematical Classification:** Categorising numbers (positive, negative, zero) as a value.

---

## 8. Pattern Matching

### Definitions

**Core Definition:** Pattern matching is a language feature (Java 21+) that allows `switch` and `instanceof` to test not only values but also types and structures, binding matched components to variables.

**Technical Definition:** Pattern matching for `switch` was finalized in Java 21. It supports type patterns (`case String s ->`), guarded patterns (`case Integer i when i > 10 ->`), and record patterns that deconstruct records into their components (`case Point(int x, int y) ->`). Dominance rules prevent unreachable cases (e.g., a `Number` pattern before an `Integer` pattern is a compile error).

**Beginner-Friendly Explanation:** Pattern matching lets a `switch` ask "What kind of thing are you?" and "What are your parts?" in one step. Instead of checking the type and then casting, you check the type and bind variables simultaneously.

### Purposes

- To replace `instanceof`-and-cast chains with concise, type-safe branching
- To deconstruct records and extract their components in a single pattern
- To express complex, type-driven decisions declaratively
- To enable exhaustiveness checking over sealed hierarchies

### Syntax Rules and Structure

```java
// Type pattern
switch (obj) {
    case String s -> ...
    case Integer i -> ...
}

// Guarded pattern
case Integer i when i > 10 -> ...

// Record pattern (deconstruction)
case Point(int x, int y) -> ...
```

**Syntax Rules:**
- Type patterns bind the matched value to a variable of the specified type.
- Guards (`when`) add a boolean condition that must also hold.
- Record patterns recursively deconstruct record components.
- The switch must be exhaustive; `default` or a sealed hierarchy is required.
- Dominance rules prevent unreachable cases.

**Constraints and Limitations:**
- Requires Java 21+ for full switch pattern matching.
- Null handling requires an explicit `case null` or `default`.
- Not all types are permitted in patterns (primitive patterns are preview).

### Annotated Code Examples

**Example 1: Type Pattern with `instanceof`**

```java
// File: PatternInstanceofDemo.java
public class PatternInstanceofDemo {
    public static void printType(Object obj) {
        if (obj instanceof String s) {
            System.out.println("String of length " + s.length() + ": " + s);
        } else if (obj instanceof Integer i) {
            System.out.println("Integer: " + i);
        } else {
            System.out.println("Unknown type");
        }
    }

    public static void main(String[] args) {
        printType("Hello"); // String of length 5: Hello
        printType(42);      // Integer: 42
    }
}
```

**Expected Output:**
```
String of length 5: Hello
Integer: 42
```

**Step-by-Step Explanation:** The type pattern `String s` both tests the type and binds the cast value to `s`, eliminating a manual cast.

**Example 2: Switch with Type Patterns and Guards**

```java
// File: PatternSwitchDemo.java
public class PatternSwitchDemo {
    public static String describe(Object obj) {
        return switch (obj) {
            case String s when s.isEmpty() -> "Empty string";
            case String s -> "String: " + s;
            case Integer i when i > 100 -> "Large integer: " + i;
            case Integer i -> "Integer: " + i;
            default -> "Other: " + obj;
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(""));    // Empty string
        System.out.println(describe("Hi"));  // String: Hi
        System.out.println(describe(150));   // Large integer: 150
        System.out.println(describe(5));     // Integer: 5
    }
}
```

**Expected Output:**
```
Empty string
String: Hi
Large integer: 150
Integer: 5
```

**Step-by-Step Explanation:** Guards (`when`) refine type patterns; the first matching label wins.

**Example 3: Record Pattern Deconstruction**

```java
// File: RecordPatternDemo.java
public class RecordPatternDemo {
    record Point(int x, int y) {}

    public static void printPoint(Object obj) {
        if (obj instanceof Point(int x, int y)) {
            System.out.println("Point at (" + x + ", " + y + ")");
        } else {
            System.out.println("Not a point");
        }
    }

    public static void main(String[] args) {
        printPoint(new Point(3, 4)); // Point at (3, 4)
    }
}
```

**Expected Output:**
```
Point at (3, 4)
```

**Step-by-Step Explanation:** The record pattern `Point(int x, int y)` deconstructs the record, binding `x` and `y` directly.

### Real-World Cases

- **AST Processing:** Matching on different node types in a compiler or interpreter.
- **Event Handling:** Dispatching events based on their concrete type.
- **Data Validation:** Checking the shape of incoming data structures.

---

## 9. Conditional Best Practices

### Definitions

**Core Definition:** Conditional best practices are guidelines for writing conditional code that is readable, maintainable, and free of common pitfalls.

**Technical Definition:** Best practices include ordering conditions from most specific to least specific, using braces consistently, preferring guard clauses to deep nesting, choosing the right abstraction (if-else vs. switch vs. polymorphism), and leveraging pattern matching to eliminate `instanceof`-and-cast chains.

**Beginner-Friendly Explanation:** Best practices help you write conditionals that are easy to read and less likely to contain hidden bugs.

### Key Best Practices

| Practice | Description |
|----------|-------------|
| **Order conditions specific → general** | In `else if` chains and pattern switches, place the most specific conditions first to avoid unreachable branches. |
| **Always use braces** | Even for single statements, braces prevent bugs when code is later modified. |
| **Use guard clauses** | Return early for invalid or edge cases to reduce nesting. |
| **Choose the right construct** | Use `if-else` for ranges/complex conditions, `switch` for discrete constants, pattern matching for types. |
| **Avoid `==` for String comparison** | Use `equals()` or `equalsIgnoreCase()` for strings. |
| **Prefer arrow-based switch** | Arrow cases eliminate fall-through and the need for `break`. |
| **Make switch exhaustive** | Cover all enum constants or include a `default` in switch expressions. |
| **Use pattern matching instead of instanceof-cast** | `if (obj instanceof String s)` is cleaner than `if (obj instanceof String) { String s = (String) obj; ... }`. |
| **Extract complex conditions into methods** | Named boolean methods improve readability (e.g., `isEligibleForDiscount()`). |
| **Keep case bodies small** | Extract multi-step logic into methods to keep switch blocks readable. |

### Annotated Code Example

```java
// File: ConditionalBestPractices.java
public class ConditionalBestPractices {

    // GOOD: Guard clause reduces nesting
    public static String classify(int n) {
        if (n < 0) {
            return "Negative";
        }
        if (n == 0) {
            return "Zero";
        }
        return "Positive";
    }

    // GOOD: Arrow switch expression is exhaustive and clear
    enum Status { SUCCESS, FAILURE, PENDING }

    public static String describe(Status status) {
        return switch (status) {
            case SUCCESS -> "Operation succeeded";
            case FAILURE -> "Operation failed";
            case PENDING -> "Operation pending";
        };
    }

    // GOOD: Pattern matching replaces instanceof-and-cast
    public static String format(Object obj) {
        return switch (obj) {
            case Integer i when i > 100 -> "Large integer";
            case Integer i -> "Integer";
            case String s -> "String: " + s;
            default -> "Other";
        };
    }

    public static void main(String[] args) {
        System.out.println(classify(-5));               // Negative
        System.out.println(describe(Status.SUCCESS));   // Operation succeeded
        System.out.println(format(150));                // Large integer
    }
}
```

**Expected Output:**
```
Negative
Operation succeeded
Large integer
```

### Real-World Cases

- **API Validation:** Using guard clauses to reject invalid input early.
- **State Machines:** Exhaustive switch expressions over enums ensure all states are handled.
- **Legacy Refactoring:** Replacing long `if-else-if` chains with pattern-matching switches.

---

## References Links

- The if-then and if-then-else Statements — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/if.html
- The switch Statement — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html
- Switch Expressions and Statements — Oracle Java SE Language Updates - https://docs.oracle.com/en/java/javase/26/language/switch-expressions-statements.html
- Guide to the yield Keyword in Java — Baeldung - https://www.baeldung.com/java-yield-switch
- Pattern Matching for switch — Oracle Java Language Updates - https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html
- Record Patterns — Oracle Java Language Updates - https://docs.oracle.com/en/java/javase/21/language/record-patterns.html
- Java switch Statement: Syntax, Expressions, Pattern Matching, and Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/the-switch-statement-java-tutorials-4827/
- Ternary Operator in Java — Baeldung - https://www.baeldung.com/java-ternary-operator