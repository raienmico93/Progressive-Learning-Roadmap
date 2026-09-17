# Java Switch Statement: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:**
The Java `switch` statement is a multiway selection control structure that transfers control to one of several branches based on the value of a single expression. It provides a cleaner alternative to long chains of `if-else if` statements when the condition involves equality against discrete constant values.

**Technical Definition:**
Per the Java Language Specification, a `switch` statement evaluates a selector expression and compares its value against the values specified by `case` labels. The selector expression must be of type `char`, `byte`, `short`, `int`, a corresponding wrapper class, `String`, or an enum type (prior to modern preview features). The statement body, called a switch block, contains labeled statement groups or switch rules that define the possible execution paths.

**Beginner-Friendly Explanation:**
Think of a `switch` statement like a railway switchyard. A train (the expression’s value) arrives at the junction, and based on which track number (case label) matches, it is routed down that path. If no track matches, the train goes down a default path if one exists. This makes it perfect for situations where you have one value and several specific actions to choose from—like selecting a menu option by number.

### Key Characteristics

- **Single-expression evaluation:** The selector expression is evaluated only once, then compared against each case constant.
- **Discrete equality matching:** `switch` checks for exact equality; it cannot evaluate ranges or complex boolean conditions (unlike `if-else`).
- **Fall-through behavior:** Without a `break` (or the modern `->` arrow syntax), execution continues into subsequent cases until a `break` is encountered or the switch block ends.
- **Type restrictions:** Traditional `switch` does not support `long`, `float`, `double`, or `boolean` selector types.
- **Two syntax styles:** The classic colon syntax (`case x:`) and the modern arrow syntax (`case x ->`) coexist; arrow syntax prevents accidental fall-through.
- **Optional default clause:** The `default` case handles values not matched by any explicit case; it is optional but recommended.

### Prerequisites

To fully understand the Java `switch` statement, you should be familiar with:

- **Java primitive data types** (`byte`, `short`, `char`, `int`) and their wrapper classes (`Byte`, `Short`, `Character`, `Integer`).
- **Enumerated types (`enum`)** and how to define and use them.
- **The `String` class** and string equality semantics (since `switch` on strings uses `equals()`, not `==`).
- **Basic control flow statements:** `if-else`, `break`, and `return`.
- **Constants and final variables**, since case labels must be compile-time constants.

### Related Programming Areas

- **Conditional Logic and Control Flow:** `switch` is a fundamental control structure alongside `if-else`, loops, and `try-catch`. Understanding when to use `switch` versus `if-else` is key to writing readable code.
- **Enum-Based State Machines:** `switch` statements on enum values are a common pattern for implementing state transitions in workflows, parsers, and game logic.
- **Pattern Matching (Modern Java):** Java 21+ introduced pattern matching for `switch`, allowing type-based dispatch, record destructuring, and guards (`when` clauses), extending `switch` far beyond constant comparison.
- **String Processing and Command Parsing:** Switching on `String` values (since Java 7) enables elegant command dispatch in CLI tools and configuration parsers.
- **Bytecode Optimization:** The JVM compiles `switch` on `int` and enum types into efficient `tableswitch` or `lookupswitch` instructions, which can offer performance benefits over long `if-else` chains.

## Core Concept 1: The Traditional Switch Statement (Colon Syntax)

**Core Definition:**
The traditional `switch` statement uses colon-labeled `case` branches and optional `break` statements to implement multiway selection. Execution falls through from one case to the next unless explicitly terminated.

**Technical Definition:**
In the colon syntax, the switch block contains one or more `SwitchBlockStatementGroup`s. Each group consists of one or more switch labels (each a `case` or `default` label) followed by a colon (`:`) and a sequence of block statements. Control enters at the matching case label and continues executing statements sequentially until a `break` statement is encountered or the switch block ends.

**Beginner-Friendly Explanation:**
This is the “classic” way of writing switch statements. Each case is like a labeled doorway. When the switch finds a matching label, it enters there and keeps walking through all the open doors that follow—unless you tell it to stop by writing `break`. This “fall-through” is often a source of bugs, but it can also be used intentionally to group cases.

### Basic Case Matching with Break

**Definitions:**

- **Core Definition:** Using `case` labels to match the selector value and `break` to exit the switch after executing the matched case’s statements.
- **Technical Definition:** A `break` statement (without a label) transfers control out of the innermost enclosing `switch`, `for`, `while`, or `do` statement. In a `switch` statement, `break` terminates the switch block and continues execution at the first statement after it.
- **Beginner-Friendly Explanation:** The `break` is like a “stop” sign at the end of each case. Without it, you’d keep executing the code for the next cases even if they didn’t match.

**Purposes:**

- To execute exactly one branch of code based on a matching value.
- To prevent unintended fall-through into subsequent case branches.
- To implement clean multiway selection that mirrors `if-else if` chains.
- To improve code readability by organizing discrete value-based logic into labeled sections.

**Syntax Structures and Rules:**

**General Syntax:**

```java
switch (expression) {
    case value1:
        // statements
        break;
    case value2:
        // statements
        break;
    default:
        // statements
        break;
}
```

**Component Breakdown:**

- `switch`: Keyword introducing the statement.
- `expression`: The selector expression (must be `char`, `byte`, `short`, `int`, wrapper, `String`, or enum).
- `case value1`: A labeled branch that matches if the expression equals `value1`.
- `break`: Terminates the switch block.
- `default`: Optional catch-all branch for unmatched values.

**Syntax Rules:**

- Case values must be **compile-time constants** (literals, `final` variables, or enum constants).
- No two case values in the same switch may be identical.
- The `default` label is optional and can appear anywhere, though convention places it last.
- A `break` is technically optional, but omitting it causes fall-through.

**Constraints and Limitations:**

- **No ranges or conditions:** Case labels cannot express `case x > 10` or `case x <= 5`; only exact values are permitted.
- **No `long`, `float`, `double`, or `boolean`:** The selector expression cannot be these types in traditional switch.
- **No `null` handling in classic switch:** A `null` selector expression on a `String` switch throws `NullPointerException`.

**Multiple Annotated Complete Code Examples:**

**Example 1: Month Name Lookup**

```java
public class SwitchDemo {
    public static void main(String[] args) {
        int month = 8;
        String monthString;

        switch (month) {
            case 1:
                monthString = "January";
                break;  // Exit switch after this case
            case 2:
                monthString = "February";
                break;
            case 3:
                monthString = "March";
                break;
            case 4:
                monthString = "April";
                break;
            case 5:
                monthString = "May";
                break;
            case 6:
                monthString = "June";
                break;
            case 7:
                monthString = "July";
                break;
            case 8:
                monthString = "August";
                break;
            case 9:
                monthString = "September";
                break;
            case 10:
                monthString = "October";
                break;
            case 11:
                monthString = "November";
                break;
            case 12:
                monthString = "December";
                break;
            default:
                monthString = "Invalid month";
                break;  // Recommended even though it's last
        }

        System.out.println(monthString);
    }
}
```

**Expected Output:**

```
August
```

**Why This Output:**

- The expression `month` evaluates to `8`.
- The switch finds `case 8:` and executes `monthString = "August";`.
- The `break` exits the switch, skipping all subsequent cases.
- The result `"August"` is printed.

**Example 2: Intentional Fall-Through for Case Grouping**

```java
public class FallThroughDemo {
    public static void main(String[] args) {
        int month = 2;
        int year = 2024;
        int numDays = 0;

        switch (month) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 8:
            case 10:
            case 12:
                numDays = 31;  // All these months have 31 days
                break;
            case 4:
            case 6:
            case 9:
            case 11:
                numDays = 30;  // All these months have 30 days
                break;
            case 2:
                // February: check for leap year
                if (((year % 4 == 0) && !(year % 100 == 0)) || (year % 400 == 0)) {
                    numDays = 29;  // Leap year
                } else {
                    numDays = 28;  // Non-leap year
                }
                break;
            default:
                System.out.println("Invalid month.");
                break;
        }

        System.out.println("Number of Days = " + numDays);
    }
}
```

**Expected Output:**

```
Number of Days = 29
```

**Why This Output:**

- `month` is `2`, matching `case 2:`.
- The fall-through from earlier cases does not apply because execution enters at `case 2:` directly.
- The year `2024` is a leap year (divisible by 4, not by 100), so `numDays` becomes `29`.
- The `break` exits and prints the result.

**Real-World Cases:**

1. **Menu-driven CLI applications:** Mapping numeric menu choices to actions.
2. **Grade classification:** Converting a numeric score or letter grade to a descriptive result.
3. **HTTP status code handling:** Dispatching behavior based on response codes (e.g., 200, 404, 500).
4. **Leap-year and calendar calculations:** Grouping months by number of days.

**References:**

- Oracle Java Tutorials: The switch Statement - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html
- Java Language Specification, §14.11 The switch Statement - https://docs.oracle.com/javase/specs/jls/se11/html/jls-14.html

### Switch with String Selectors

**Definitions:**

- **Core Definition:** Using a `String` expression as the selector in a `switch` statement, introduced in Java 7.
- **Technical Definition:** When the selector expression is of type `String`, the compiler generates a hash-based dispatch followed by `String.equals()` comparisons to disambiguate hash collisions. The comparison is case-sensitive and uses `equals()`, not `==`.
- **Beginner-Friendly Explanation:** You can now switch on text values, not just numbers. However, if the string is `null`, the switch will throw a `NullPointerException`, so you must check for `null` first.

**Purposes:**

- To dispatch behavior based on textual commands or identifiers.
- To replace long `if-else if` chains comparing strings.
- To improve readability when handling discrete string categories.
- To normalize input and route it to the appropriate handler.

**Syntax Structures and Rules:**

**General Syntax:**

```java
switch (stringExpression) {
    case "value1":
        // statements
        break;
    case "value2":
        // statements
        break;
    default:
        // statements
        break;
}
```

**Component Breakdown:**

- `stringExpression`: A `String` variable or expression.
- `case "value1"`: A string literal or constant expression.

**Syntax Rules:**

- The selector must be of type `String` (or a subclass, though `final` semantics apply).
- Case labels must be string **literals** or `final` string variables.
- Comparison is case-sensitive; use `toLowerCase()` or `equalsIgnoreCase()`-style normalization before switching.
- The switch internally uses `String.hashCode()` then `equals()` to resolve collisions.

**Constraints and Limitations:**

- **`NullPointerException` on null selector:** Always check for `null` before switching on a `String`.
- **No regex or pattern matching:** Only exact string equality is supported.
- **Performance overhead:** Hash computation and `equals()` calls add minor overhead compared to integer switches, though negligible in most applications.

**Multiple Annotated Complete Code Examples:**

**Example 1: String Switch with Null Check**

```java
public class StringSwitchDemo {
    public static void main(String[] args) {
        String month = "August";
        int monthNumber;

        // Critical: null check before switch
        if (month == null) {
            System.out.println("Month cannot be null");
            return;
        }

        switch (month.toLowerCase()) {  // Normalize to lowercase
            case "january":
                monthNumber = 1;
                break;
            case "february":
                monthNumber = 2;
                break;
            case "march":
                monthNumber = 3;
                break;
            case "april":
                monthNumber = 4;
                break;
            case "may":
                monthNumber = 5;
                break;
            case "june":
                monthNumber = 6;
                break;
            case "july":
                monthNumber = 7;
                break;
            case "august":
                monthNumber = 8;
                break;
            case "september":
                monthNumber = 9;
                break;
            case "october":
                monthNumber = 10;
                break;
            case "november":
                monthNumber = 11;
                break;
            case "december":
                monthNumber = 12;
                break;
            default:
                monthNumber = 0;
                break;
        }

        System.out.println("Month number: " + monthNumber);
    }
}
```

**Expected Output:**

```
Month number: 8
```

**Why This Output:**

- The `null` check prevents `NullPointerException`.
- `month.toLowerCase()` converts `"August"` to `"august"`.
- The switch matches `case "august":` and sets `monthNumber = 8`.

**Real-World Cases:**

1. **Command-line parsers:** Dispatching on user-typed commands like `"start"`, `"stop"`, `"restart"`.
2. **Configuration handling:** Routing based on string keys from properties files.
3. **REST API routing:** Simple frameworks dispatch on HTTP method strings (`"GET"`, `"POST"`).
4. **Localization:** Selecting behavior based on language codes (`"en"`, `"fr"`, `"es"`).

**References:**

- Oracle Java Tutorials: Using Strings in switch Statements - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html
- Java Language Specification, §14.11 The switch Statement - https://docs.oracle.com/javase/specs/jls/se11/html/jls-14.html

## Core Concept 2: Modern Switch Expressions (Arrow Syntax)

**Core Definition:**
Switch expressions, introduced as a preview in Java 12 and finalized in Java 14, use arrow (`->`) syntax and can produce a value. They eliminate fall-through and can be used as expressions (assigned to variables, returned from methods).

**Technical Definition:**
A switch expression is a `SwitchExpression` that evaluates to a value. Its switch block uses `SwitchRule`s (arrow syntax) or `yield` statements. The selector expression may be any type except `long`, `float`, `double`, or `boolean` (prior to preview features). A switch expression must be **exhaustive**: all possible values must be covered, either by explicit cases or a `default` clause.

**Beginner-Friendly Explanation:**
The modern switch is like a vending machine: you put in a value, and it directly gives you back a result. There’s no risk of accidentally continuing into the next case, and you can use the whole switch as an expression—like `String result = switch (day) { ... };`.

### Arrow Syntax and Exhaustiveness

**Definitions:**

- **Core Definition:** A switch style using `->` to separate case labels from their expressions/statements, with no fall-through and mandatory exhaustiveness for expressions.
- **Technical Definition:** In a switch rule, `case L -> Expression`, `case L -> Block`, or `case L -> ThrowStatement` is permitted. Only one case label is allowed per switch rule. Multiple values can be combined with commas: `case 1, 2, 3 ->`.
- **Beginner-Friendly Explanation:** The arrow means “do this and then get out.” No `break` needed. And if you’re using it to produce a value, you must cover every possible input.

**Purposes:**

- To prevent accidental fall-through bugs by design.
- To enable switch to be used as an expression that returns a value.
- To write more concise and readable multiway selection code.
- To support exhaustiveness checking, ensuring all cases are handled.

**Syntax Structures and Rules:**

**General Syntax (Statement):**

```java
switch (expression) {
    case value1 -> statement1;
    case value2 -> statement2;
    default -> defaultStatement;
}
```

**General Syntax (Expression):**

```java
resultType result = switch (expression) {
    case value1 -> expression1;
    case value2 -> expression2;
    default -> defaultExpression;
};
```

**General Syntax (Block with yield):**

```java
resultType result = switch (expression) {
    case value1 -> {
        // multiple statements
        yield value1Result;  // yield returns the value
    }
    case value2 -> expression2;
    default -> defaultExpression;
};
```

**Component Breakdown:**

- `->`: Arrow operator separating case label from body.
- `yield`: Returns a value from a block within a switch expression.
- Multiple labels: `case 1, 2 ->` combines cases.

**Syntax Rules:**

- No `break` is needed (or allowed) within a switch rule.
- Fall-through does not occur between switch rules.
- A switch expression **must** be exhaustive: all possible selector values must be covered.
- `yield` is required only when the rule body is a block (`{ }`) and the switch is an expression.

**Constraints and Limitations:**

- `yield` is a **restricted identifier**: it cannot be used as a variable name in switch blocks.
- Exhaustiveness is required for expressions but not for statements (unless enhanced).
- The selector type restrictions still apply: no `long`, `float`, `double`, or `boolean` in traditional switch expressions.

**Multiple Annotated Complete Code Examples:**

**Example 1: Switch Expression with Arrow Syntax (Returning a String)**

```java
public class SwitchExpressionDemo {
    public static void main(String[] args) {
        int day = 5;

        String dayType = switch (day) {
            case 1, 7 -> "Weekend";        // Multiple labels
            case 2, 3, 4, 5, 6 -> "Weekday";
            default -> "Invalid day";
        };

        System.out.println("Day " + day + " is a " + dayType);
    }
}
```

**Expected Output:**

```
Day 5 is a Weekday
```

**Why This Output:**

- `day` is `5`, matching `case 2, 3, 4, 5, 6 -> "Weekday"`.
- The arrow syntax prevents fall-through, so each branch executes independently.

**Example 2: Switch Expression with Arrow Syntax (block and yield)**

```java
public class SwitchExpressionDemo {
    public static void main(String[] args) {
        int month = 2;

        int daysInMonth = switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> 31;
            case 4, 6, 9, 11 -> 30;
            case 2 -> {
                int year = 2024;
                if ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0) {
                    yield 29;  // yield returns 29 from the block
                } else {
                    yield 28;
                }
            }
            default -> throw new IllegalArgumentException("Invalid month: " + month);
        };

        System.out.println("Days in month " + month + ": " + daysInMonth);
    }
}
```

**Expected Output:**

```
Days in month 2: 29
```

**Why This Output:**

- `month` is `2`, entering the block case. The year `2024` is a leap year, so `yield 29` returns `29`.
- The arrow syntax prevents fall-through, so each branch executes independently.

**Example 3: Exhaustiveness with Enums**

```java
enum Season { SPRING, SUMMER, FALL, WINTER }

public class EnumSwitchDemo {
    public static void main(String[] args) {
        Season current = Season.SUMMER;

        // Enum switch expression: exhaustive without default
        String activity = switch (current) {
            case SPRING -> "Planting";
            case SUMMER -> "Swimming";
            case FALL   -> "Harvesting";
            case WINTER -> "Skiing";
        };

        System.out.println("In " + current + ", go " + activity);
    }
}
```

**Expected Output:**

```
In SUMMER, go Swimming
```

**Why This Output:**

- The enum has exactly four constants, and all four are covered.
- The compiler recognizes exhaustiveness, so no `default` is needed.
- If a new enum constant were added, the code would fail to compile until updated—a valuable safety feature.

**Real-World Cases:**

1. **API versioning:** Returning response formats based on version enums.
2. **State machines:** Computing next states in workflow engines.
3. **Unit conversion:** Converting between measurement systems based on unit enums.
4. **Feature flags:** Returning boolean or configuration values based on feature name strings.

**References:**

- Java Language Specification, §15.28 switch Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html
- JEP 361: Switch Expressions - https://openjdk.org/jeps/361

## Core Concept 3: Pattern Matching for switch (Modern Java)

**Core Definition:**
Pattern matching for `switch` (introduced in Java 21) allows `case` labels to contain type patterns, record patterns, and guards (`when` clauses), enabling type-based dispatch and destructuring within a single switch.

**Technical Definition:**
A `case` label may contain a `CasePattern` (a `Pattern` optionally followed by a `Guard`). Patterns include type patterns (`String s`) and record patterns (`Point(int x, int y)`). The selector expression may be any reference type. Dominance rules prevent unreachable or ambiguous patterns. Switch expressions with pattern matching must be exhaustive, often achievable without `default` when using sealed hierarchies.

**Beginner-Friendly Explanation:**
Instead of writing a chain of `if (obj instanceof String) { String s = (String) obj; ... }`, you can now write `case String s ->`. Java handles the type check and the cast for you. You can even deconstruct records directly in the case label.

### Type Patterns and Guards

**Definitions:**

- **Core Definition:** Using `case Type variableName` to match objects of a specific type and bind them to a variable, optionally with a `when` guard condition.
- **Technical Definition:** A type pattern `T t` matches any object that is an instance of `T` and binds the matched object to the pattern variable `t`. A guard `when Expression` further restricts the match.
- **Beginner-Friendly Explanation:** This replaces the old `instanceof` + cast pattern. You say “if it’s a String, call it `s` and do this” all in one line.

**Purposes:**

- To eliminate verbose `instanceof` + cast chains.
- To dispatch behavior based on runtime type in a type-safe manner.
- To add conditional logic within a case using guards.
- To improve readability of polymorphic code analysis.

**Syntax Structures and Rules:**

**General Syntax:**

```java
switch (object) {
    case Type1 var1 -> // handle Type1
    case Type2 var2 when condition -> // handle Type2 with guard
    default -> // fallback
}
```

**Component Breakdown:**

- `Type1 var1`: Type pattern—matches if object is `Type1`, binds to `var1`.
- `when condition`: Guard—additional boolean condition.
- `default`: Required if not exhaustive without it.

**Syntax Rules:**

- Pattern variables are in scope only within their case body.
- Guards use `when`, not `if`.
- Dominance rules apply: a more general pattern before a more specific one causes a compile error if it dominates.
- For a switch expression with pattern matching, exhaustiveness is required.

**Constraints and Limitations:**

- **Preview feature in Java 17–20**, finalized in Java 21.
- A `case` label may not have more than one pattern.
- `case null` must be explicitly handled if the selector can be null (and if present, must precede type patterns in some versions).

**Multiple Annotated Complete Code Examples:**

**Example 1: Type-Based Dispatch with Guards**

```java
public class PatternSwitchDemo {
    static String describe(Object o) {
        return switch (o) {
            case null -> "null value";
            case Integer i -> "Integer: " + i;
            case String s when s.length() > 10 -> "Long string: " + s;
            case String s -> "Short string: " + s;
            case Double d -> "Double: " + d;
            default -> "Unknown type: " + o.getClass().getName();
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(42));
        System.out.println(describe("Hello"));
        System.out.println(describe("Hello, World!"));
        System.out.println(describe(3.14));
        System.out.println(describe(null));
    }
}
```

**Expected Output:**

```
Integer: 42
Short string: Hello
Long string: Hello, World!
Double: 3.14
null value
```

**Why This Output:**

- `42` matches `case Integer i`.
- `"Hello"` (length 5) fails the guard `s.length() > 10`, so it matches the next `case String s`.
- `"Hello, World!"` (length 13) satisfies the guard and matches the first String case.
- `3.14` matches `case Double d`.
- `null` matches `case null`.

**Real-World Cases:**

1. **JSON/XML processing:** Dispatching on node types (object, array, string, number) parsed from data.
2. **AST (Abstract Syntax Tree) visitors:** Traversing and processing different node types in compilers.
3. **Event handling:** Routing UI events to handlers based on event subtype.
4. **Serialization frameworks:** Determining how to serialize objects based on their runtime types.

**References:**

- Java Language Specification, §14.30 Patterns - https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- JEP 441: Pattern Matching for switch - https://openjdk.org/jeps/441

### Record Patterns and Destructuring

**Definitions:**

- **Core Definition:** Using record patterns to deconstruct record components directly in case labels.
- **Technical Definition:** A record pattern `RecordName(ComponentType1 var1, ComponentType2 var2, ...)` matches if the object is an instance of the record and its components match the specified patterns. This enables nested destructuring.
- **Beginner-Friendly Explanation:** If you have a `Point(int x, int y)` record, you can write `case Point(int x, int y)` and Java automatically extracts the `x` and `y` values for you.

**Purposes:**

- To extract record components without accessor method calls.
- To perform nested pattern matching on complex data structures.
- To write concise case analysis for algebraic data types.
- To improve readability when working with records.

**Syntax Structures and Rules:**

**General Syntax:**

```java
switch (point) {
    case Point(int x, int y) when x == 0 && y == 0 -> "Origin";
    case Point(int x, int y) -> "Point at (" + x + ", " + y + ")";
    default -> "Not a point";
}
```

**Component Breakdown:**

- `Point(int x, int y)`: Record pattern matching `Point` and binding `x`, `y`.
- Components can themselves be patterns for nested destructuring.

**Syntax Rules:**

- Record patterns work with records that have **canonical constructors**.
- Nested patterns are allowed: `case Line(Point(int x1, int y1), Point(int x2, int y2)) ->`.
- Guards can use the extracted components.

**Constraints and Limitations:**

- Only works with `record` types, not arbitrary classes.
- Component patterns must match the record’s component types.
- Requires Java 21+ (finalized; preview in Java 19–20).

**Multiple Annotated Complete Code Examples:**

**Example 1: Record Destructuring in Switch**

```java
record Point(int x, int y) {}
record Circle(Point center, int radius) {}

public class RecordPatternDemo {
    static String describe(Object shape) {
        return switch (shape) {
            case Point(int x, int y) when x == 0 && y == 0 -> "Origin";
            case Point(int x, int y) when x > 0 && y > 0 -> "First quadrant";
            case Point(int x, int y) -> "Point(" + x + ", " + y + ")";
            case Circle(Point(int cx, int cy), int r) ->
                "Circle at (" + cx + ", " + cy + ") with radius " + r;
            default -> "Unknown shape";
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new Point(0, 0)));
        System.out.println(describe(new Point(3, 4)));
        System.out.println(describe(new Circle(new Point(1, 2), 5)));
    }
}
```

**Expected Output:**

```
Origin
First quadrant
Circle at (1, 2) with radius 5
```

**Why This Output:**

- `new Point(0, 0)` matches the guard `x == 0 && y == 0`.
- `new Point(3, 4)` fails the origin guard, then matches `x > 0 && y > 0`.
- `new Circle(new Point(1, 2), 5)` matches the nested record pattern, extracting `cx=1`, `cy=2`, `r=5`.

**Real-World Cases:**

1. **Geometric computations:** Processing points, lines, circles, and polygons with pattern matching.
2. **Expression evaluators:** Matching on AST record types for arithmetic expressions.
3. **Configuration parsing:** Extracting nested configuration records from parsed data.
4. **Game development:** Handling entity types with positional records.

**References:**

- JEP 440: Record Patterns - https://openjdk.org/jeps/440
- Java Language Specification, §14.30.3 Record Patterns - https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html

## Summary Table: Switch Variants

| Feature           | Syntax                | Fall-Through?       | Exhaustiveness           | Java Version |
| ----------------- | --------------------- | ------------------- | ------------------------ | ------------ |
| Classic switch    | `case x:` + `break`   | Yes (without break) | Not required             | 1.0+         |
| String switch     | `case "text":`        | Yes                 | Not required             | 7+           |
| Switch expression | `case x ->` + `yield` | No                  | Required                 | 14+          |
| Pattern matching  | `case Type t ->`      | No                  | Required for expressions | 21+          |
| Record patterns   | `case Rec(int a) ->`  | No                  | Required for expressions | 21+          |

## References

1. Oracle Java Tutorials: The switch Statement - https://docs.oracle.com/javase/tutorial/java/nutsandbolts/switch.html
2. Java Language Specification, §14.11 The switch Statement - https://docs.oracle.com/javase/specs/jls/se11/html/jls-14.html
3. Java Language Specification, §15.28 switch Expressions - https://docs.oracle.com/javase/specs/jls/se17/html/jls-15.html
4. JEP 361: Switch Expressions - https://openjdk.org/jeps/361
5. JEP 441: Pattern Matching for switch - https://openjdk.org/jeps/441
6. JEP 440: Record Patterns - https://openjdk.org/jeps/440
7. Runestone Academy: Java, Java, Java - Switch Multiway Selection - https://runestone.academy/ns/books/published/javajavajava/switch.html
8. Kansas State University CIS 200: Switch Statements - https://textbooks.cs.ksu.edu/cis200/02-conditionals/02_4-switchstatements/index.print.html
9. ISO/IEC JTC 1/SC 22/WG 23 N1469: Notes on Java Switch Statement - https://www.rap.no/JTC1/SC22/WG23/docs/ISO-IECJTC1-SC22-WG23_N1469-notes-on-java-switch-stmt-EP-SM-20250218.pdf
