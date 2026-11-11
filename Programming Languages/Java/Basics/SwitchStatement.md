# Java Switch Expressions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A switch expression is an enhanced form of the traditional `switch` statement that evaluates to a single value, supports concise arrow labels, and enforces exhaustiveness. Pattern matching for `switch` (standard since Java 21) extends this by allowing cases to match on types, guarded conditions, and `null`.

**Technical Definition:** The Java Language Specification (JLS) distinguishes between the traditional `switch` statement (colon labels, fall-through, `break`) and modern `switch` expressions/statements (arrow labels, no fall-through, `yield`). An enhanced `switch` statement or expression must be exhaustive if it uses pattern or `null` labels. The arrow form (`case L ->`) executes only the code to the right and never falls through. Pattern matching for `switch` allows case labels to contain type patterns, record patterns, and guarded patterns (`when` clauses).

**Beginner-Friendly Explanation:** Think of `switch` as a more organised alternative to a long chain of `if-else` checks. The old `switch` works like a set of stairs — if you forget to stop (`break`), you fall down to the next step. The new `switch` expression works like a lift: you select a floor, the doors open exactly there, and you get a result. Arrow labels (`->`) eliminate the “fall-through” bug entirely. Pattern matching takes this further: instead of only matching constant values, you can say “if the object is a `Circle`, bind it to a variable and do something with it.”

### Key Characteristics

- **Expression Form:** Switch expressions return a value; switch statements do not.
- **Arrow Labels:** `case L ->` eliminates fall-through and the need for `break`.
- **`yield` Keyword:** Returns a value from a block within a switch expression.
- **Multiple Labels:** A single case can list several comma-separated labels (`case A, B, C ->`).
- **Exhaustiveness:** Switch expressions (and enhanced switch statements using patterns/null) must cover all possible values or include a `default`.
- **Pattern Matching (Java 21+):** Cases can match on types, record patterns, and guarded conditions.
- **Guarded Patterns:** `case Type t when condition ->` adds a boolean guard to a pattern.
- **Null Handling:** `case null` can be used explicitly; without it, a `null` selector throws `NullPointerException` (except with `default` in some cases).
- **Compile-Time Safety:** Sealed hierarchies enable exhaustiveness checks without `default`.

### Prerequisites

- Basic Java syntax and control flow
- Familiarity with `switch` statements
- Understanding of Java 14+ features (switch expressions)
- Basic knowledge of Java 21+ pattern matching (sealed classes, records)

### Related Programming Areas

- **Pattern Matching:** Type patterns, record patterns, and guarded patterns.
- **Sealed Classes:** Enable exhaustive switches without `default`.
- **Records:** Record patterns in switch cases.
- **Control Flow:** `if-else`, loops, and branching.
- **Expression-Oriented Programming:** Switch expressions as first-class values.

---

## Core Concepts / Features

---

## 1. Traditional Switch

### Definitions

**Core Definition:** The traditional `switch` statement selects a branch of code based on the value of an expression, using colon labels and `break` to prevent fall-through.

**Technical Definition:** A traditional `switch` statement consists of a selector expression and a switch block containing `case` labels (`case L:`) and an optional `default` label. If a `case` matches, execution begins at that label and continues until a `break` or the end of the block (fall-through).

**Beginner-Friendly Explanation:** The old `switch` is like a staircase. You step onto the stair that matches your value. If you don’t stop (`break`), you slide down to the next stair — sometimes useful, often a bug.

### Purposes

- To select one of many branches based on a value
- To improve readability over long `if-else` chains
- To support primitive, enum, and `String` selectors (with limitations)

### Syntax Rules and Structure

**Complete General Syntax:**

```java
switch (selector) {
    case value1:
        // code
        break;
    case value2:
    case value3:
        // code for both
        break;
    default:
        // default code
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `selector` | An expression of type `byte`, `short`, `char`, `int`, `String`, or enum. |
| `case L:` | A label; execution begins here if it matches. |
| `break` | Exits the switch block. |
| `default` | Executes if no case matches. |

**Constraints and Limitations:**

- Fall-through can cause bugs if `break` is forgotten.
- The selector cannot be `long`, `float`, `double`, or `boolean`.
- The selector cannot be `null` (throws `NullPointerException`).

### Annotated Code Example

```java
// File: TraditionalSwitchDemo.java
public class TraditionalSwitchDemo {
    public static void main(String[] args) {
        char grade = 'B';
        String description;

        switch (grade) {
            case 'A':
                description = "Excellent";
                break;
            case 'B':
            case 'C':
                description = "Good"; // Fall-through from 'B'
                break;
            case 'D':
                description = "Passing";
                break;
            default:
                description = "Unknown";
        }

        System.out.println(description); // Output: Good
    }
}
```

**Expected Output:**
```
Good
```

**Step-by-Step Explanation:**

1. `grade = 'B'` matches `case 'B'`.
2. Execution falls through to `case 'C'` because there is no `break` after `'B'`.
3. The code under `'C'` runs, setting `description = "Good"`, then `break` exits.
4. The result is printed.

### Real-World Cases

- **Legacy Codebases:** Many existing Java projects use traditional `switch`.
- **Simple Routing:** Quick branching on enum values where fall-through is intentional.

---

## 2. Switch Expression

### Definitions

**Core Definition:** A switch expression evaluates to a single value, using either arrow labels or colon labels with `yield`.

**Technical Definition:** A switch expression is a `switch` used in an expression context. It must be exhaustive. It can use arrow labels (`case L ->`) or colon labels (`case L:`) with `yield` to produce a value.

**Beginner-Friendly Explanation:** A switch expression is a `switch` that gives you something back. Instead of assigning a value inside each case, the whole `switch` becomes the value.

### Purposes

- To produce a value directly from a `switch`
- To eliminate temporary variables
- To enable switch usage in assignments, returns, and method arguments
- To enforce exhaustiveness and reduce runtime surprises

### Syntax Rules and Structure

**Complete General Syntax (Arrow Form):**

```java
Type result = switch (selector) {
    case L1 -> expression1;
    case L2 -> expression2;
    default -> expressionN;
};
```

**Complete General Syntax (Colon + `yield`):**

```java
Type result = switch (selector) {
    case L1:
        yield expression1;
    default:
        yield expressionN;
};
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `selector` | Same as traditional switch. |
| `case L ->` | Arrow rule; no fall-through. |
| `yield` | Returns a value from a block (colon form only). |
| `default` | Required unless exhaustive by other means. |

**Syntax Rules:**

- A switch expression must be exhaustive.
- Arrow rules and colon groups cannot be mixed in the same switch.
- A block in an arrow rule must contain `yield` to produce a value.

### Annotated Code Example

```java
// File: SwitchExpressionDemo.java
public class SwitchExpressionDemo {
    public static void main(String[] args) {
        char grade = 'B';

        String description = switch (grade) {
            case 'A' -> "Excellent";
            case 'B', 'C' -> "Good";
            case 'D' -> "Passing";
            default -> "Unknown";
        };

        System.out.println(description); // Output: Good
    }
}
```

**Expected Output:**
```
Good
```

**Step-by-Step Explanation:**

1. `grade = 'B'` matches the arrow rule `case 'B', 'C'`.
2. The expression `"Good"` is the value of the switch expression.
3. The value is assigned to `description`.
4. No fall-through, no `break`, no temporary variable.

### Real-World Cases

- **Configuration Mapping:** Mapping enum values to descriptions or codes.
- **Stream Operations:** Using switch expressions inside lambdas.
- **Method Returns:** Returning a computed value directly.

---

## 3. Arrow Syntax

### Definitions

**Core Definition:** Arrow syntax (`case L ->`) is a concise switch label form that executes only the code to the right of the arrow, with no fall-through.

**Technical Definition:** An arrow case has the form `case label_1, ..., label_n -> expression; | throw-statement; | block`. When a label matches, the code to the right executes and control leaves the switch — no subsequent cases run.

**Beginner-Friendly Explanation:** The arrow is a one-way door. Once you go through it, you don’t come back to the switch. This completely eliminates the “forgot `break`” bug.

### Purposes

- To eliminate accidental fall-through
- To make switch code more concise and readable
- To allow switch expressions and statements to be used safely
- To group multiple labels without fall-through

### Syntax Rules and Structure

```java
case label -> expression;          // single expression (value in switch expression)
case label -> throw new ...;       // throws
case label -> { /* block */ };     // block (statement; use yield if expression)
case A, B, C -> ...;               // multiple labels
```

**Syntax Rules:**

- No `break` is needed (or allowed) after an arrow rule.
- The right side can be an expression, a `throw`, or a block.
- In a switch expression, a block must contain `yield` to produce a value.
- Arrow and colon forms cannot be mixed in the same switch.

**Constraints and Limitations:**

- Arrow form cannot use fall-through (intentional, by design).
- Cannot use `continue` to jump out of a switch expression.

### Annotated Code Example

```java
// File: ArrowSyntaxDemo.java
public class ArrowSyntaxDemo {
    public static void main(String[] args) {
        int day = 3;

        String type = switch (day) {
            case 1, 7 -> "Weekend";
            case 2, 3, 4, 5, 6 -> "Weekday";
            default -> throw new IllegalArgumentException("Invalid day: " + day);
        };

        System.out.println(type); // Output: Weekday
    }
}
```

**Expected Output:**
```
Weekday
```

**Step-by-Step Explanation:**

1. `day = 3` matches `case 2, 3, 4, 5, 6`.
2. The arrow rule returns `"Weekday"`.
3. No `break` or fall-through occurs.
4. The result is printed.

### Real-World Cases

- **Enum Routing:** Clean mapping of enum constants to behaviour.
- **State Machines:** Simple state transitions without fall-through bugs.
- **Validation:** Throwing exceptions from arrow rules for invalid inputs.

---

## 4. `yield`

### Definitions

**Core Definition:** `yield` is a statement that returns a value from a block within a switch expression (or colon-form switch expression).

**Technical Definition:** A `yield` statement transfers control out of a switch expression, making its operand the value of the switch expression. It is required when a switch expression uses a block or colon-form group that must produce a value.

**Beginner-Friendly Explanation:** When a switch case needs multiple lines of code to compute its result, you write a block and use `yield` to say “this block’s result is this value.”

### Purposes

- To return a value from a multi-statement case in a switch expression
- To support complex logic within a case while still producing a value
- To replace the traditional “assign to a temp variable then break” pattern

### Syntax Rules and Structure

```java
Type result = switch (selector) {
    case A -> {
        // statements
        yield value;
    }
    default -> value;
};
```

**Syntax Rules:**

- `yield` is a contextual keyword (not reserved).
- A block in an arrow rule must contain `yield` if the switch expression needs a value.
- A colon-form group must contain `yield` to produce a value.
- `yield` cannot be used outside a switch expression.

**Constraints and Limitations:**

- A switch expression cannot complete without a value or an exception; every path must yield or throw.
- `break`, `return`, and `continue` cannot be used to jump through a switch expression.

### Annotated Code Example

```java
// File: YieldDemo.java
public class YieldDemo {
    public static void main(String[] args) {
        int value = 42;

        String result = switch (value) {
            case 1, 2, 3 -> "small";
            case 42 -> {
                String note = "answer";
                yield "The " + note; // yield returns the value
            }
            default -> "other";
        };

        System.out.println(result); // Output: The answer
    }
}
```

**Expected Output:**
```
The answer
```

**Step-by-Step Explanation:**

1. `value = 42` matches `case 42`.
2. The block executes, creating `note = "answer"`.
3. `yield "The " + note` returns `"The answer"` as the switch expression’s value.
4. The result is printed.

### Real-World Cases

- **Complex Calculations:** Cases that require multiple steps before returning a value.
- **Conditional Logic:** Using `if` statements inside a case block and yielding a result.

---

## 5. Multiple Labels

### Definitions

**Core Definition:** Multiple labels allow a single case to match several values, separated by commas.

**Technical Definition:** In an arrow rule, `case A, B, C ->` matches if the selector equals any of the listed labels. In colon form, multiple labels are stacked (`case A: case B:`), which implies fall-through between them.

**Beginner-Friendly Explanation:** Multiple labels are like a single key that opens several locks. One case handles several values without duplicating code.

### Purposes

- To group related values under one case
- To reduce code duplication
- To make the mapping from values to behaviour explicit

### Syntax Rules and Structure

```java
// Arrow form (preferred)
case A, B, C -> action;

// Colon form (fall-through grouping)
case A:
case B:
case C:
    action;
    break;
```

**Syntax Rules:**

- Arrow form supports comma-separated labels directly.
- Colon form requires stacking labels and a shared `break`.
- Labels must be distinct within a switch block (no duplicate labels).

### Annotated Code Example

```java
// File: MultipleLabelsDemo.java
public class MultipleLabelsDemo {
    public static void main(String[] args) {
        int month = 4;

        String season = switch (month) {
            case 12, 1, 2 -> "Winter";
            case 3, 4, 5 -> "Spring";
            case 6, 7, 8 -> "Summer";
            case 9, 10, 11 -> "Autumn";
            default -> "Invalid month";
        };

        System.out.println(season); // Output: Spring
    }
}
```

**Expected Output:**
```
Spring
```

**Step-by-Step Explanation:**

1. `month = 4` matches `case 3, 4, 5`.
2. The arrow rule returns `"Spring"`.
3. No separate case for each month is needed.

### Real-World Cases

- **Date Handling:** Grouping months into seasons.
- **Status Mapping:** Mapping several HTTP status codes to one category.
- **Enum Grouping:** Grouping enum constants by shared behaviour.

---

## 6. Pattern Matching for `switch`

### Definitions

**Core Definition:** Pattern matching for `switch` allows case labels to contain type patterns, record patterns, and guarded patterns, extending switch beyond constant matching.

**Technical Definition:** Since Java 21, a switch case can use a type pattern (`case String s ->`), a record pattern (`case Point(int x, int y) ->`), or a guarded pattern (`case Integer i when i > 10 ->`). The selector must be of a reference type (or a primitive preview). Exhaustiveness rules apply when patterns are used.

**Beginner-Friendly Explanation:** Instead of only checking “is this value equal to 5?”, pattern matching lets you check “is this object a `String`? If so, bind it to `s` and run this code.” It’s like having a smart sorting machine that recognises the type and shape of each item.

### Purposes

- To replace `if (x instanceof Type)` + cast chains with concise switch cases
- To deconstruct records directly in case labels
- To handle heterogeneous data types uniformly
- To enable exhaustiveness checking with sealed hierarchies

### Syntax Rules and Structure

```java
switch (obj) {
    case String s -> ...;                // Type pattern
    case Point(int x, int y) -> ...;     // Record pattern
    case Integer i when i > 10 -> ...;   // Guarded pattern
    case null -> ...;                     // Null case
    default -> ...;
}
```

**Syntax Rules:**

- Type patterns bind a variable (`s`) of the matched type.
- Record patterns deconstruct records into their components.
- Guarded patterns use `when` to add a boolean condition.
- The selector can be any reference type (or primitive preview).
- If any pattern or `null` label is used, the switch must be exhaustive.

**Constraints and Limitations:**

- Pattern matching for `switch` is standard in Java 21+.
- Dominance rules: a more specific pattern must precede a more general one.
- Guarded patterns do not contribute to exhaustiveness (an unguarded fallback is required).

### Annotated Code Example

```java
// File: PatternMatchingSwitchDemo.java
public class PatternMatchingSwitchDemo {
    sealed interface Shape permits Circle, Rectangle {}
    record Circle(double radius) implements Shape {}
    record Rectangle(double width, double height) implements Shape {}

    static double area(Shape shape) {
        return switch (shape) {
            case Circle(double r) -> Math.PI * r * r;
            case Rectangle(double w, double h) -> w * h;
        }; // exhaustive — no default needed
    }

    public static void main(String[] args) {
        Shape circle = new Circle(5);
        Shape rect = new Rectangle(4, 3);

        System.out.println("Circle area: " + area(circle));
        System.out.println("Rectangle area: " + area(rect));
    }
}
```

**Expected Output:**
```
Circle area: 78.53981633974483
Rectangle area: 12.0
```

**Step-by-Step Explanation:**

1. `Shape` is sealed, permitting only `Circle` and `Rectangle`.
2. The switch uses record patterns to deconstruct each type.
3. No `default` is needed because the sealed hierarchy is fully covered.
4. The compiler enforces exhaustiveness.

### Real-World Cases

- **AST Processing:** Matching different node types in a compiler.
- **Event Handling:** Dispatching on event subtypes.
- **Data Validation:** Matching and validating different record shapes.

---

## 7. Guarded Patterns

### Definitions

**Core Definition:** A guarded pattern adds a boolean condition (`when`) to a pattern, so the case matches only if both the pattern matches and the guard evaluates to `true`.

**Technical Definition:** A guarded pattern has the form `case Pattern when Expr ->`. The pattern is matched first; if it succeeds, the guard `Expr` is evaluated. If the guard is `false`, the case does not match and the next case is tried.

**Beginner-Friendly Explanation:** A guarded pattern is a pattern with a filter. “Match this type, but only if this extra condition is true.”

### Purposes

- To add conditions to type or record patterns
- To express range checks or validity checks inline
- To avoid nested `if` statements inside cases

### Syntax Rules and Structure

```java
case Integer i when i > 10 -> "big";
case Integer i -> "small";
```

**Syntax Rules:**

- `when` is a contextual keyword.
- The guard is a boolean expression.
- Guarded patterns do not count toward exhaustiveness; an unguarded fallback (or `default`) is required.
- The pattern variable is in scope in the guard.

**Constraints and Limitations:**

- Guarded patterns are not considered for exhaustiveness computation.
- The compiler does not reason about complementary guards (e.g., `i > 0` and `i <= 0`).

### Annotated Code Example

```java
// File: GuardedPatternDemo.java
public class GuardedPatternDemo {
    static String classify(Object obj) {
        return switch (obj) {
            case null -> "null";
            case Integer i when i > 100 -> "large integer: " + i;
            case Integer i -> "integer: " + i;
            case String s when s.isBlank() -> "blank string";
            case String s -> "string: " + s;
            default -> "other";
        };
    }

    public static void main(String[] args) {
        System.out.println(classify(150));   // large integer: 150
        System.out.println(classify(42));    // integer: 42
        System.out.println(classify("  "));  // blank string
        System.out.println(classify("hello")); // string: hello
        System.out.println(classify(null));  // null
    }
}
```

**Expected Output:**
```
large integer: 150
integer: 42
blank string
string: hello
null
```

**Step-by-Step Explanation:**

1. `150` matches `Integer i` and the guard `i > 100` is `true` → “large integer”.
2. `42` matches `Integer i` but the guard is `false`; the next case `Integer i` matches.
3. `"  "` matches `String s` and the guard `s.isBlank()` is `true` → “blank string”.
4. `"hello"` matches `String s` but the guard is `false`; the next case matches.
5. `null` matches `case null`.

### Real-World Cases

- **Validation:** Checking ranges or formats inline.
- **Domain Rules:** Applying business conditions in pattern matching.
- **Parsing:** Matching tokens only if they meet syntactic constraints.

---

## 8. Null Handling

### Definitions

**Core Definition:** `switch` expressions and pattern-matching switch statements can handle `null` explicitly via `case null`, or indirectly through `default` (in some contexts). Without explicit handling, a `null` selector throws `NullPointerException`.

**Technical Definition:** A `case null` label matches a `null` selector. A total pattern (a pattern that matches all values of the selector type) also matches `null` in Java 21+. If no `null`-compatible label exists, a `null` selector causes `NullPointerException` (except for older switch semantics).

**Beginner-Friendly Explanation:** `null` is a special value. The new switch lets you say `case null -> "it's null"` instead of crashing. If you don’t handle `null`, the switch will throw an exception when the value is `null`.

### Purposes

- To handle `null` gracefully without separate `if (x == null)` checks
- To make `null` handling explicit and readable
- To avoid `NullPointerException` in switch expressions

### Syntax Rules and Structure

```java
switch (obj) {
    case null -> "null";
    case String s -> "string";
    default -> "other";
}
```

**Syntax Rules:**

- `case null` must appear before `default` (or anywhere, but dominance rules apply).
- A `case null` label counts toward exhaustiveness.
- A total pattern (e.g., `case Object o ->`) also matches `null` in Java 21+.

**Constraints and Limitations:**

- Without `case null` or a total pattern, a `null` selector throws `NullPointerException`.
- `case null` cannot be combined with other labels in a comma-separated list (in some versions).

### Annotated Code Example

```java
// File: NullSwitchDemo.java
public class NullSwitchDemo {
    static String describe(String s) {
        return switch (s) {
            case null -> "null value";
            case "" -> "empty string";
            default -> "string of length " + s.length();
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(null));  // null value
        System.out.println(describe(""));    // empty string
        System.out.println(describe("hi"));  // string of length 2
    }
}
```

**Expected Output:**
```
null value
empty string
string of length 2
```

**Step-by-Step Explanation:**

1. `null` matches `case null`.
2. `""` matches `case ""`.
3. `"hi"` matches `default`.

### Real-World Cases

- **API Responses:** Handling nullable fields from external services.
- **Optional Alternatives:** Using switch instead of `Optional` checks.
- **Data Cleaning:** Classifying values including `null`.

---

## 9. Exhaustiveness

### Definitions

**Core Definition:** A switch is exhaustive if it covers all possible values of the selector, either through explicit cases or a `default`. Switch expressions must be exhaustive; enhanced switch statements using patterns or `null` must also be exhaustive.

**Technical Definition:** The compiler checks exhaustiveness by computing the type coverage of the switch labels. A `default` label covers all types. A sealed hierarchy is exhaustive if all permitted subtypes are covered. A `case null` label is required for `null` coverage unless a total pattern is present.

**Beginner-Friendly Explanation:** Exhaustiveness means “no value can slip through the cracks.” The compiler forces you to handle every possibility. This is especially powerful with sealed classes: if you add a new subtype, the compiler tells you exactly which switches need updating.

### Purposes

- To prevent runtime surprises from unhandled values
- To enable safe evolution of sealed hierarchies
- To eliminate the need for `default` in many cases
- To make switch logic complete and verifiable at compile time

### Syntax Rules and Structure

**Exhaustiveness Mechanisms:**

| Mechanism | Covers |
|-----------|--------|
| `default` | All remaining values |
| All enum constants | All values of the enum |
| All permitted subtypes of a sealed type | All instances of the sealed type |
| `case null` | The `null` value |
| Total pattern (`case Object o`) | All values including `null` (Java 21+) |

**Syntax Rules:**

- Switch expressions require exhaustiveness.
- Enhanced switch statements using patterns or `null` require exhaustiveness.
- Traditional switch statements (colon labels, no patterns) are not required to be exhaustive.

**Constraints and Limitations:**

- Guarded patterns do not count toward exhaustiveness.
- If a sealed hierarchy changes and is not recompiled with the switch, a `MatchException` may be thrown at runtime.
- Floating-point representationally equivalent labels are rejected as duplicates.

### Annotated Code Example

```java
// File: ExhaustivenessDemo.java
public class ExhaustivenessDemo {
    sealed interface Result permits Success, Failure {}
    record Success(String data) implements Result {}
    record Failure(String error) implements Result {}

    static String handle(Result result) {
        return switch (result) {
            case Success(String data) -> "OK: " + data;
            case Failure(String error) -> "ERROR: " + error;
            // No default needed — exhaustive over sealed hierarchy
        };
    }

    public static void main(String[] args) {
        System.out.println(handle(new Success("data")));
        System.out.println(handle(new Failure("oops")));
    }
}
```

**Expected Output:**
```
OK: data
ERROR: oops
```

**Step-by-Step Explanation:**

1. `Result` is sealed, permitting only `Success` and `Failure`.
2. The switch covers both permitted subtypes.
3. The compiler verifies exhaustiveness; no `default` is required.
4. If a new subtype were added to `Result`, the switch would fail to compile until updated.

### Real-World Cases

- **Domain Modelling:** Sealed hierarchies with exhaustive pattern matching.
- **Compiler Design:** AST node handling with sealed types.
- **State Machines:** Exhaustive state transitions.

---

## 10. Switch Best Practices

### Definitions

**Core Definition:** Switch best practices are guidelines for using switch expressions and statements safely, concisely, and maintainably.

**Technical Definition:** Best practices derive from the JLS, JEPs, and community experience. Key principles include preferring arrow labels to eliminate fall-through, using switch expressions when a value is needed, handling `null` explicitly, leveraging sealed hierarchies for exhaustiveness, and keeping case logic simple.

**Beginner-Friendly Explanation:** Best practices help you avoid the classic `switch` bugs (fall-through, missing `default`, `null` crashes) and write code that is easy to read and evolve.

### Key Best Practices Table

| Practice | Description |
|----------|-------------|
| Prefer arrow labels (`->`) | Eliminates fall-through and the need for `break`. |
| Use switch expressions for values | Avoid temporary variables; return values directly. |
| Handle `null` explicitly | Use `case null` when the selector may be `null`. |
| Leverage sealed hierarchies | Enable exhaustiveness without `default`. |
| Avoid guarded patterns for exhaustiveness | Guards do not count toward coverage; provide an unguarded fallback. |
| Use `yield` only when necessary | Single expressions do not need a block or `yield`. |
| Do not mix arrow and colon forms | Choose one style per switch block. |
| Keep cases focused | Avoid overly complex logic inside a single case. |
| Order cases by dominance | More specific patterns must precede more general ones. |
| Recompile after changing sealed hierarchies | Avoid `MatchException` at runtime. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: SwitchBestPractices.java
public sealed interface Payment permits Card, Cash, Wallet {}
record Card(String number) implements Payment {}
record Cash(double amount) implements Payment {}
record Wallet(String id, double balance) implements Payment {}

public class SwitchBestPractices {
    // BEST PRACTICE: Arrow labels + sealed exhaustiveness (no default)
    static String describe(Payment p) {
        return switch (p) {
            case Card(String n) -> "Card ending " + n.substring(n.length() - 4);
            case Cash(double a) -> "Cash: $" + a;
            case Wallet(String id, double b) -> "Wallet " + id + ": $" + b;
        };
    }

    // BEST PRACTICE: Explicit null handling with guard
    static String safeDescribe(Payment p) {
        return switch (p) {
            case null -> "No payment";
            case Card(String n) -> "Card";
            case Cash(double a) when a > 100 -> "Large cash";
            case Cash(double a) -> "Cash";
            case Wallet(String id, double b) -> "Wallet";
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new Card("1234567890123456")));
        System.out.println(describe(new Cash(50)));
        System.out.println(describe(new Wallet("W1", 200)));
        System.out.println(safeDescribe(null));
    }
}
```

**Expected Output:**
```
Card ending 3456
Cash: $50.0
Wallet W1: $200.0
No payment
```

**Step-by-Step Explanation:**

1. Arrow labels are used throughout — no fall-through.
2. Record patterns deconstruct each `Payment` subtype.
3. The sealed hierarchy makes `describe` exhaustive without `default`.
4. `safeDescribe` handles `null` explicitly and uses a guard (`when a > 100`).
5. Best practices are applied consistently.

### Real-World Cases

- **Domain-Driven Design:** Exhaustive switches over sealed domain types.
- **API Response Handling:** Explicit `null` and error case handling.
- **Configuration Mapping:** Concise mapping with multiple labels and arrows.

---

## References

- Java Language Updates — Switch Expressions and Statements, Oracle (Java SE 26) — https://docs.oracle.com/en/java/javase/26/language/switch-expressions-statements.html
- Java Language Updates — Pattern Matching for `switch`, Oracle (Java SE 26) — https://docs.oracle.com/en/java/javase/26/language/pattern-matching-switch.html
- Java Language Specification, §14.11 The `switch` Statement — https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- JEP 361: Switch Expressions — OpenJDK — https://openjdk.org/jeps/361
- JEP 441: Pattern Matching for `switch` — OpenJDK — https://openjdk.org/jeps/441
- JEP 406: Pattern Matching for `switch` (Preview) — OpenJDK — https://openjdk.org/jeps/406
- Pattern Matching for `switch` — Changes to the JLS (Java 17 preview) — https://docs.oracle.com/javase/specs/jls/se17/preview/specs/patterns-switch-jls.html
- Java switch Statement: Syntax, Expressions, Pattern Matching, and Best Practices — Cleverence — https://www.cleverence.com/articles/oracle-documentation/the-switch-statement-java-tutorials-4827/
- The Art of Code — Chapter 4: Expressiveness with patterns and the enhanced switch — Manning — https://www-qa.manning.com/preview/the-art-of-code/chapter-4
- Modern Java idioms by release — GitHub Copilot Skill — https://github.com/github/awesome-copilot/blob/18362138723066ce0dbaf7976d1a2921e79a54e4/skills/java-development/references/modern-java.md
- Java 21 guarded pattern exhaustiveness — Stack Overflow — https://stackoverflow.com/questions/77189424/java-21-guarded-pattern-exhaustiveness