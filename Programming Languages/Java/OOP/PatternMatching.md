# Java Pattern Matching: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Pattern matching is a language feature that allows a program to test whether a value has a particular type or structure, and if so, to extract data from that value into new variables — all in a single, concise expression.

**Technical Definition:** Per the Java Language Specification, "Pattern matching is the process of testing a value against a pattern at run time." A pattern is a combination of (1) a predicate (or test) that can be applied to a target, and (2) a set of local variables, known as pattern variables, that are extracted from the target only if the predicate successfully applies to it. Pattern matching in Java spans three major features: pattern matching for `instanceof` (finalized in JDK 16 via JEP 394), record patterns (finalized in JDK 21 via JEP 440), and pattern matching for `switch` (finalized in JDK 21 via JEP 441). Together, these features enable declarative, composable data navigation and processing.

**Beginner-Friendly Explanation:** Pattern matching is like a smart sorting machine. Instead of checking "Is this a box?" and then manually opening the box to see what's inside, you can say "If this is a box containing an apple and a banana, give me the apple and the banana as named items." Java's pattern matching lets you test the shape of your data and extract its contents in one step, without tedious casting or repeated getter calls.

### Key Characteristics

- **Concise Type Inspection:** Combines type testing and variable binding in a single expression, eliminating the verbose `instanceof`-and-cast idiom.
- **Flow Scoping:** Pattern variables are in scope only where the pattern has definitely matched, as determined by the compiler's flow analysis.
- **Composability:** Record patterns and type patterns can be nested to enable powerful, declarative data navigation and processing.
- **Exhaustiveness:** When combined with sealed classes, pattern matching in `switch` can be verified as exhaustive by the compiler, eliminating the need for a `default` clause.
- **Null Handling:** Modern `switch` supports `case null` labels, relaxing the historical null-hostility of `switch`.
- **Guarded Patterns:** The `when` clause allows additional boolean conditions to be attached to pattern labels.

### Prerequisites

- Java 16 or later (for `instanceof` pattern matching)
- Java 21 or later (for record patterns and pattern matching for `switch`)
- Basic understanding of Java classes, interfaces, and inheritance
- Familiarity with `instanceof` and type casting
- Knowledge of `switch` statements and expressions
- Awareness of records and sealed classes (for exhaustive pattern matching)

### Related Programming Areas

- **Algebraic Data Types (ADTs):** Pattern matching, combined with sealed classes and records, brings sum-type modeling to Java.
- **Visitor Pattern:** Exhaustive `switch` over sealed hierarchies replaces many uses of the Visitor pattern.
- **Compilers and Interpreters:** Pattern matching is ideal for processing ASTs and other tree-like data structures.
- **Protocol Handling:** Exhaustive matching on sealed message types ensures all protocol variants are handled.
- **Functional Programming:** Pattern matching encourages a declarative, data-oriented style of programming.

### Core Concepts / Features

---

## 1. Pattern Matching

### Definitions

**Core Definition:** Pattern matching is a language mechanism that combines type testing with data extraction, allowing a value to be tested against a pattern and, if it matches, its components to be bound to new variables.

**Technical Definition:** "Pattern matching is the process of testing a value against a pattern at run time." A pattern consists of a predicate that can be applied to a target, and a set of pattern variables that are extracted from the target if the predicate successfully applies. Java supports two kinds of patterns: **type patterns** (which test whether a value is an instance of a type) and **record patterns** (which test whether a value is an instance of a record class type and recursively match its component values).

**Beginner-Friendly Explanation:** Pattern matching is like describing the shape of the data you're looking for. You say "If this is a circle with radius r, then use r." Java checks if the data matches that shape and, if it does, gives you the named pieces.

### Purposes

- To eliminate the boilerplate of the `instanceof`-and-cast idiom
- To enable concise, declarative extraction of data from objects
- To make type-driven branching safer and more readable
- To support exhaustiveness checking when combined with sealed classes
- To enable powerful data navigation through nested patterns

### Syntax Rules and Structure

**Complete General Syntax (Type Pattern):**

```java
Type variable
```

**Complete General Syntax (Record Pattern):**

```java
RecordType(componentPattern1, componentPattern2, ...)
```

**Component Breakdown:**

| Pattern Kind | Syntax | Description |
|--------------|--------|-------------|
| Type pattern | `Type variable` | Tests if a value is an instance of `Type` and binds it to `variable`. |
| Record pattern | `RecordType(comp1, comp2, ...)` | Tests if a value is an instance of `RecordType` and recursively matches its components. |

**Syntax Rules:**

- A type pattern consists of a type and a single pattern variable.
- A record pattern consists of a record class type and a (possibly empty) pattern list.
- Patterns can be nested — a record pattern's component patterns can themselves be type patterns or record patterns.
- Pattern variables are in scope only where the pattern has definitely matched (flow scoping).

**Constraints and Limitations:**

- Pattern matching is a compile-time feature; patterns are not first-class values.
- Primitive types are not currently supported in type patterns (preview features are exploring this).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Before and After Pattern Matching**

```java
// File: PatternMatchingDemo.java
public class PatternMatchingDemo {

    // BEFORE pattern matching (Java 15 and earlier)
    static String formatOld(Object obj) {
        String formatted = "unknown";
        if (obj instanceof Integer) {
            Integer i = (Integer) obj;  // Manual cast
            formatted = String.format("int %d", i);
        } else if (obj instanceof Long) {
            Long l = (Long) obj;        // Manual cast
            formatted = String.format("long %d", l);
        } else if (obj instanceof Double) {
            Double d = (Double) obj;    // Manual cast
            formatted = String.format("double %f", d);
        }
        return formatted;
    }

    // AFTER pattern matching (Java 16+)
    static String formatNew(Object obj) {
        if (obj instanceof Integer i) {   // Type pattern: test + bind
            return String.format("int %d", i);
        } else if (obj instanceof Long l) {
            return String.format("long %d", l);
        } else if (obj instanceof Double d) {
            return String.format("double %f", d);
        }
        return "unknown";
    }

    public static void main(String[] args) {
        System.out.println(formatOld(42));        // int 42
        System.out.println(formatNew(42));        // int 42
        System.out.println(formatOld(3.14));      // double 3.140000
        System.out.println(formatNew(3.14));      // double 3.140000
    }
}
```

**Expected Output:**
```
int 42
int 42
double 3.140000
double 3.140000
```

**Step-by-Step Explanation:**

1. In the old style, each branch requires an `instanceof` test followed by a manual cast and a new local variable.
2. In the new style, the type pattern `Integer i` performs both the test and the cast in one step.
3. The pattern variable `i` is in scope only within the `if` block where the pattern matched (flow scoping).
4. Both versions produce the same output, but the new version is more concise and eliminates the risk of cast errors.

### Real-World Cases

- **Parsing JSON/XML:** Extracting values of different types from parsed data structures.
- **Event Handling:** Dispatching on event types in GUI or network applications.
- **Compiler AST Processing:** Matching on different node types in an abstract syntax tree.

---

## 2. instanceof Pattern

### Definitions

**Core Definition:** The `instanceof` pattern is a language feature that combines a type check with a pattern variable binding, replacing the traditional `instanceof`-and-cast idiom.

**Technical Definition:** "Pattern matching for the `instanceof` operator has been made a final and permanent feature of the Java language in JDK 16." "A type pattern consists of a predicate that specifies a type, along with a single pattern variable." The `instanceof` operator is extended to take a type pattern instead of just a type, allowing the conditional extraction of components from objects to be expressed more concisely and safely.

**Beginner-Friendly Explanation:** The `instanceof` pattern is like saying "If this object is a String, call it `s` and use it." Java checks the type and gives you a named variable in one step.

### Purposes

- To eliminate the repetitive `instanceof`-and-cast boilerplate
- To reduce the risk of errors caused by manual casting
- To make type-checking code more concise and readable
- To enable flow-scoped pattern variables that are only accessible where the pattern matches

### Syntax Rules and Structure

**Complete General Syntax:**

```java
if (expression instanceof Type variable) {
    // variable is in scope here
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `expression` | The value to be tested. |
| `instanceof` | The operator that performs the type test. |
| `Type` | The type to test against. |
| `variable` | The pattern variable that receives the cast value if the test succeeds. |

**Syntax Rules:**

- The pattern variable is in scope only where the pattern has definitely matched (flow scoping).
- The pattern variable is not implicitly `final` (as of JDK 16).
- It is a compile-time error for a pattern `instanceof` expression to compare an expression of type `S` against a pattern of type `T` where `S` is a subtype of `T` (the expression would always succeed and is pointless).

**Constraints and Limitations:**

- Only reference types can be used in type patterns.
- The pattern variable cannot be used outside the scope where the pattern is guaranteed to match.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic instanceof Pattern**

```java
// File: InstanceofPatternDemo.java
public class InstanceofPatternDemo {

    static void process(Object obj) {
        // Type pattern: test and bind in one step
        if (obj instanceof String s) {
            // s is in scope here
            System.out.println("String of length " + s.length() + ": " + s);
        } else if (obj instanceof Integer i) {
            // i is in scope here
            System.out.println("Integer value: " + i);
        } else {
            System.out.println("Unknown type: " + obj.getClass().getSimpleName());
        }
    }

    public static void main(String[] args) {
        process("Hello, World!");   // String of length 13: Hello, World!
        process(42);                // Integer value: 42
        process(3.14);              // Unknown type: Double
    }
}
```

**Expected Output:**
```
String of length 13: Hello, World!
Integer value: 42
Unknown type: Double
```

**Step-by-Step Explanation:**

1. `obj instanceof String s` tests whether `obj` is a `String` and, if so, binds it to `s`.
2. Inside the `if` block, `s` can be used directly without a cast.
3. `s.length()` accesses the string's length.
4. In the `else if` branch, `i` is bound to the `Integer` value.
5. The final `else` handles all other types.

**Example 2: Negated Pattern**

```java
// File: NegatedPatternDemo.java
public class NegatedPatternDemo {
    static void process(String input) {
        // Pattern variable is NOT in scope in the negated branch
        if (!(input instanceof String s)) {
            // s is NOT in scope here
            System.out.println("Not a string");
            return;
        }
        // s IS in scope here (after the negated check)
        System.out.println("String: " + s.toUpperCase());
    }

    public static void main(String[] args) {
        process("hello");  // String: HELLO
    }
}
```

**Expected Output:**
```
String: HELLO
```

**Step-by-Step Explanation:**

1. The negated pattern `!(input instanceof String s)` checks if `input` is not a `String`.
2. In the `if` block (where the pattern did not match), `s` is not in scope.
3. After the `if` block, the compiler knows the pattern matched, so `s` is in scope.

### Real-World Cases

- **Type-Safe Downcasting:** Safely casting objects from collections or generic containers.
- **Visitor Pattern Replacement:** Using `instanceof` patterns instead of visitor methods.
- **Input Validation:** Checking the type of configuration values or user input.

---

## 3. Type Patterns

### Definitions

**Core Definition:** A type pattern is a pattern that tests whether a value is an instance of a specified type and binds the value to a pattern variable of that type.

**Technical Definition:** "A type pattern is used to test whether a value is an instance of the type appearing in the pattern." A type pattern declares one local variable, known as a pattern variable. If the declaration includes an identifier, this specifies the name of the pattern variable; otherwise, the pattern variable is unnamed.

**Beginner-Friendly Explanation:** A type pattern is the building block of pattern matching. It says "if this value is of this type, call it this name."

### Purposes

- To test the type of a value and bind it to a named variable
- To serve as the foundation for more complex patterns (record patterns, switch patterns)
- To enable flow-scoped type narrowing

### Syntax Rules and Structure

**Complete General Syntax:**

```java
Type variable
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Type` | The type to test against. |
| `variable` | The name of the pattern variable (or `_` for unnamed). |

**Syntax Rules:**

- The pattern variable is in scope only where the pattern has definitely matched.
- Unnamed patterns (using `_`) are supported in preview (JDK 21+).
- A type pattern can appear in `instanceof` expressions and `case` labels.

**Constraints and Limitations:**

- The type must be a reference type (primitive type patterns are in preview).
- The pattern variable cannot be used outside its flow scope.

### Annotated Code Example

```java
// File: TypePatternDemo.java
public class TypePatternDemo {
    sealed interface Shape permits Circle, Rectangle { }
    record Circle(double radius) implements Shape { }
    record Rectangle(double length, double width) implements Shape { }

    static double area(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle r -> r.length() * r.width();
        };
    }

    public static void main(String[] args) {
        System.out.println("Circle: " + area(new Circle(5)));
        System.out.println("Rectangle: " + area(new Rectangle(4, 3)));
    }
}
```

**Expected Output:**
```
Circle: 78.53981633974483
Rectangle: 12.0
```

**Step-by-Step Explanation:**

1. `Circle c` is a type pattern that matches if `shape` is a `Circle`.
2. `Rectangle r` is a type pattern that matches if `shape` is a `Rectangle`.
3. The `switch` is exhaustive because `Shape` is sealed and permits only these two subclasses.

### Real-World Cases

- **Sealed Hierarchy Processing:** Matching on each permitted subtype of a sealed interface.
- **Event Dispatching:** Routing events to handlers based on their type.
- **Data Extraction:** Binding values to variables for further processing.

---

## 4. Record Patterns

### Definitions

**Core Definition:** A record pattern is a pattern that tests whether a value is an instance of a record class and, if so, recursively matches its component values against nested patterns.

**Technical Definition:** "You can use a record pattern to test whether a value is an instance of a record class type and, if it is, to recursively perform pattern matching on its component values." "A record pattern consists of a record class type and a (possibly empty) pattern list which is used to match against the corresponding record component values." Record patterns and type patterns can be nested to enable a powerful, declarative, and composable form of data navigation and processing.

**Beginner-Friendly Explanation:** A record pattern lets you "deconstruct" a record into its components in one step. Instead of calling `point.x()` and `point.y()`, you write `Point(int x, int y)` and get `x` and `y` directly.

### Purposes

- To deconstruct record values into their component parts without calling accessor methods
- To enable nested data navigation and processing
- To make code that processes records more concise and readable
- To support exhaustive pattern matching over sealed record hierarchies

### Syntax Rules and Structure

**Complete General Syntax:**

```java
RecordType(componentPattern1, componentPattern2, ...)
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `RecordType` | The record class type to match against. |
| `componentPatternN` | A pattern (type pattern, record pattern, or unnamed pattern) for each component. |

**Syntax Rules:**

- The number of component patterns must match the number of record components.
- Component patterns can be nested record patterns.
- Unnamed patterns (`_`) can be used to skip components.

**Constraints and Limitations:**

- Record patterns only work with record classes.
- Record patterns in enhanced `for` statements were removed in JDK 21 (may be re-proposed).

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Record Pattern**

```java
// File: RecordPatternDemo.java
public class RecordPatternDemo {
    record Point(int x, int y) { }

    static void printSum(Object obj) {
        // Record pattern: deconstruct Point into x and y
        if (obj instanceof Point(int x, int y)) {
            System.out.println("Sum: " + (x + y));
        } else {
            System.out.println("Not a point");
        }
    }

    public static void main(String[] args) {
        printSum(new Point(3, 4));   // Sum: 7
        printSum("hello");           // Not a point
    }
}
```

**Expected Output:**
```
Sum: 7
Not a point
```

**Step-by-Step Explanation:**

1. `obj instanceof Point(int x, int y)` tests if `obj` is a `Point` and, if so, binds its components to `x` and `y`.
2. The pattern variable `x` is bound to `point.x()` and `y` to `point.y()` automatically.
3. The sum `x + y` is computed directly.

**Example 2: Nested Record Pattern**

```java
// File: NestedRecordPatternDemo.java
public class NestedRecordPatternDemo {
    record Point(int x, int y) { }
    record Line(Point start, Point end) { }

    static void describe(Object obj) {
        // Nested record pattern: deconstruct Line and its Points
        if (obj instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
            System.out.println("Line from (" + x1 + "," + y1 + ") to (" + x2 + "," + y2 + ")");
        } else {
            System.out.println("Not a line");
        }
    }

    public static void main(String[] args) {
        Line line = new Line(new Point(1, 2), new Point(3, 4));
        describe(line);  // Line from (1,2) to (3,4)
    }
}
```

**Expected Output:**
```
Line from (1,2) to (3,4)
```

**Step-by-Step Explanation:**

1. The outer record pattern `Line(Point(...), Point(...))` matches a `Line` and recursively matches its `start` and `end` components.
2. The inner record patterns `Point(int x1, int y1)` and `Point(int x2, int y2)` deconstruct the `Point` components.
3. All four coordinates are bound to variables and used in the output.

### Real-World Cases

- **Geometry:** Processing points, lines, and polygons.
- **AST Processing:** Deconstructing expression nodes in a compiler.
- **API Response Handling:** Extracting nested data from record-based responses.

---

## 5. Switch Patterns

### Definitions

**Core Definition:** Switch patterns allow `switch` expressions and statements to test an expression against a series of patterns, each with a specific action, rather than only against constant values.

**Technical Definition:** "Pattern Matching for `switch` enhances the Java programming language with pattern matching for `switch` expressions and statements. Extending pattern matching to `switch` allows an expression to be tested against a number of patterns, each with a specific action, so that complex data-oriented queries can be expressed concisely and safely." Prior to Java 21, `switch` was limited to integral types, enums, and `String`; Java 21 relaxed this to allow any reference type and added support for `case null` labels.

**Beginner-Friendly Explanation:** Switch patterns let you write a `switch` statement that checks not just for specific values, but for types and shapes. It's like a multi-way `if-else` chain but much cleaner.

### Purposes

- To replace long `if-else` chains of `instanceof` checks with a single, readable `switch`
- To enable exhaustive pattern matching over sealed hierarchies
- To support null handling in `switch` via `case null`
- To allow complex data-oriented queries to be expressed concisely

### Syntax Rules and Structure

**Complete General Syntax:**

```java
switch (selectorExpression) {
    case Pattern1 -> action1;
    case Pattern2 -> action2;
    case null -> nullAction;
    default -> defaultAction;
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `selectorExpression` | The value to be tested against patterns. |
| `case Pattern` | A pattern label (type pattern, record pattern, or constant). |
| `case null` | A label that matches `null`. |
| `default` | Optional; required only if the switch is not exhaustive. |

**Syntax Rules:**

- The selector expression can be of any reference type (Java 21+).
- `case null` is permitted and must be handled explicitly if the selector can be null.
- A `default` label is permitted but not required if the switch is exhaustive.
- The first matching label wins; order matters.
- Exhaustiveness is required for pattern `switch` statements (enhanced switch).

**Constraints and Limitations:**

- A guarded pattern with a `when` clause does not contribute to exhaustiveness.
- The compiler checks dominance: a constant label must appear before a guarded pattern that would match it.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Switch Pattern with Type Patterns**

```java
// File: SwitchPatternDemo.java
public class SwitchPatternDemo {
    static String formatter(Object obj) {
        return switch (obj) {
            case Integer i -> String.format("int %d", i);
            case Long l -> String.format("long %d", l);
            case Double d -> String.format("double %f", d);
            case String s -> String.format("String %s", s);
            case null -> "null";
            default -> obj.toString();
        };
    }

    public static void main(String[] args) {
        System.out.println(formatter(42));          // int 42
        System.out.println(formatter(3.14));        // double 3.140000
        System.out.println(formatter("hello"));     // String hello
        System.out.println(formatter(null));        // null
        System.out.println(formatter(true));        // true (default)
    }
}
```

**Expected Output:**
```
int 42
double 3.140000
String hello
null
true
```

**Step-by-Step Explanation:**

1. The `switch` tests `obj` against a series of type patterns.
2. `case Integer i` matches if `obj` is an `Integer` and binds it to `i`.
3. `case null` handles the `null` value.
4. The `default` handles all other types.

**Example 2: Exhaustive Switch with Sealed Types**

```java
// File: ExhaustiveSwitchDemo.java
public class ExhaustiveSwitchDemo {
    sealed interface Shape permits Circle, Rectangle, Triangle { }
    record Circle(double radius) implements Shape { }
    record Rectangle(double length, double width) implements Shape { }
    record Triangle(double base, double height) implements Shape { }

    static double area(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle r -> r.length() * r.width();
            case Triangle t -> 0.5 * t.base() * t.height();
            // No default needed — exhaustive!
        };
    }

    public static void main(String[] args) {
        System.out.println("Circle: " + area(new Circle(5)));
        System.out.println("Rectangle: " + area(new Rectangle(4, 3)));
        System.out.println("Triangle: " + area(new Triangle(6, 2)));
    }
}
```

**Expected Output:**
```
Circle: 78.53981633974483
Rectangle: 12.0
Triangle: 6.0
```

**Step-by-Step Explanation:**

1. `Shape` is a sealed interface permitting exactly three record types.
2. The `switch` covers all three permitted types.
3. The compiler verifies exhaustiveness — no `default` is needed.
4. If a new permitted subtype were added, the compiler would report an error.

### Real-World Cases

- **State Machines:** Exhaustive switch over sealed state types.
- **AST Processing:** Matching on different node types in a compiler.
- **Protocol Handling:** Handling all message variants in a protocol.

---

## 6. Guarded Patterns

### Definitions

**Core Definition:** A guarded pattern is a pattern label in a `switch` that includes a `when` clause, adding an additional boolean condition that must be true for the pattern to match.

**Technical Definition:** "Guarded patterns are replaced with `when` clauses in switch blocks." A pattern case element is said to be unguarded if it has no associated `when` expression, or if it has a `when` expression that is a constant expression with value `true`. A guarded pattern with a `when` clause makes a pattern case conditional, removing its contribution to total branch exhaustiveness.

**Beginner-Friendly Explanation:** A guarded pattern says "Match this pattern, but only if this extra condition is also true." For example, "If this is an Integer and it's greater than 10, then..."

### Purposes

- To add additional conditions to pattern matching beyond type and shape
- To refine matches based on the values of extracted components
- To express complex data-oriented queries in a single switch

### Syntax Rules and Structure

**Complete General Syntax:**

```java
case Pattern when condition -> action;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Pattern` | The pattern to match (type or record pattern). |
| `when condition` | A boolean expression that must be true for the match. |

**Syntax Rules:**

- The `when` clause is optional; without it, the pattern is unguarded.
- A guarded pattern does not contribute to exhaustiveness — an unguarded fallback or `default` is required.
- The first matching label wins; order matters.

**Constraints and Limitations:**

- A `when` clause is a boolean expression and cannot have side effects.
- Guarded patterns are not exhaustive on their own.

### Annotated Code Example

```java
// File: GuardedPatternDemo.java
public class GuardedPatternDemo {
    static String classify(Object obj) {
        return switch (obj) {
            case Integer i when i > 100 -> "Large integer: " + i;
            case Integer i when i > 0 -> "Positive integer: " + i;
            case Integer i -> "Non-positive integer: " + i;
            case String s when s.isEmpty() -> "Empty string";
            case String s -> "String: " + s;
            case null -> "null";
            default -> "Unknown: " + obj;
        };
    }

    public static void main(String[] args) {
        System.out.println(classify(150));    // Large integer: 150
        System.out.println(classify(50));     // Positive integer: 50
        System.out.println(classify(-5));     // Non-positive integer: -5
        System.out.println(classify(""));     // Empty string
        System.out.println(classify("hello")); // String: hello
        System.out.println(classify(null));   // null
    }
}
```

**Expected Output:**
```
Large integer: 150
Positive integer: 50
Non-positive integer: -5
Empty string
String: hello
null
```

**Step-by-Step Explanation:**

1. `case Integer i when i > 100` matches only if `obj` is an `Integer` and its value is greater than 100.
2. The next guarded pattern `when i > 0` catches positive integers that are not large.
3. The unguarded `case Integer i` catches all remaining integers.
4. Similarly, the `String` patterns use a `when` clause for empty strings and an unguarded pattern for all other strings.

### Real-World Cases

- **Validation:** Matching on types with additional value constraints.
- **Filtering:** Selecting data based on both type and value.
- **Business Rules:** Expressing complex conditions in a readable switch.

---

## 7. Pattern Matching Best Practices

### Definitions

**Core Definition:** Pattern matching best practices are guidelines for using pattern matching effectively, safely, and maintainably in Java code.

**Technical Definition:** Best practices for pattern matching emphasize leveraging exhaustiveness with sealed types, keeping case bodies short, using `when` clauses judiciously, and combining records, sealed classes, and patterns to model sum types. The use of `default` should be omitted when a switch over a sealed hierarchy is exhaustive, so that adding a new permitted subtype becomes a compile-time error that forces the developer to handle it.

**Beginner-Friendly Explanation:** Pattern matching best practices help you write code that is safe, readable, and easy to maintain. They encourage you to let the compiler catch mistakes and to keep your matching logic focused.

### Purposes

- To maximise the safety and maintainability benefits of pattern matching
- To leverage compiler checking for exhaustiveness and dominance
- To keep pattern-matching code readable and focused
- To avoid common pitfalls such as forgetting null or missing cases

### Syntax Rules and Structure

**Key Best Practices:**

| Practice | Description |
|----------|-------------|
| Combine sealed classes with records and patterns | Use `sealed` interfaces with record implementations to model sum types and enable exhaustive matching. |
| Omit `default` when exhaustive | When a switch over a sealed hierarchy covers all permitted subtypes, omit `default` so that adding a new subtype becomes a compile-time error. |
| Keep case bodies short | Move complex logic to separate methods; keep `case` bodies focused. |
| Limit logic in `when` clauses | Keep guard conditions simple; complex logic should be extracted. |
| Use `case null` explicitly | Handle `null` explicitly when the selector can be null. |
| Order matters | Place more specific patterns before more general ones; the first match wins. |
| Use unnamed patterns for ignored components | Use `_` for record components you don't need. |
| Prefer switch expressions over statements | Switch expressions are more concise and enforce exhaustiveness. |

**Syntax Rules:**

- A guarded pattern with a `when` clause does not contribute to exhaustiveness.
- Constant labels must appear before guarded patterns that would match them.
- The compiler performs dominance checking to detect unreachable cases.

**Constraints and Limitations:**

- Overusing guards can make switch blocks hard to read.
- Pattern matching does not replace polymorphism; use it when externalised decision logic is appropriate.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Best Practices in Action**

```java
// File: BestPracticesPatternDemo.java
public class BestPracticesPatternDemo {
    /**
     * Sealed hierarchy modeling payment results.
     * Sealed + records + patterns = safe, exhaustive handling.
     */
    sealed interface PaymentResult
        permits PaymentSuccess, PaymentFailure, PaymentPending {
        record PaymentSuccess(String transactionId, double amount)
            implements PaymentResult { }
        record PaymentFailure(String errorCode, String message)
            implements PaymentResult { }
        record PaymentPending(String referenceId)
            implements PaymentResult { }
    }

    static String describe(PaymentResult result) {
        return switch (result) {
            case PaymentResult.PaymentSuccess s ->
                "Success: " + s.transactionId();
            case PaymentResult.PaymentFailure f ->
                "Failure: " + f.errorCode();
            case PaymentResult.PaymentPending p ->
                "Pending: " + p.referenceId();
            // No default needed — exhaustive!
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new PaymentResult.PaymentSuccess("TXN-001", 99.99)));
        System.out.println(describe(new PaymentResult.PaymentFailure("ERR-500", "Insufficient funds")));
        System.out.println(describe(new PaymentResult.PaymentPending("REF-123")));
    }
}
```

**Expected Output:**
```
Success: TXN-001
Failure: ERR-500
Pending: REF-123
```

**Step-by-Step Explanation:**

1. `PaymentResult` is a sealed interface permitting three record types.
2. The `switch` covers all three permitted types and omits `default`.
3. If a new permitted subtype were added, the compiler would report a non-exhaustive switch error.
4. Each `case` body is short and delegates to a single expression.

**Example 2: Avoiding Common Pitfalls**

```java
// File: PitfallsPatternDemo.java
public class PitfallsPatternDemo {

    // BAD: default hides the exhaustiveness benefit
    static String badDescribe(Shape shape) {
        return switch (shape) {
            case Circle c -> "Circle";
            case Rectangle r -> "Rectangle";
            default -> "Unknown";  // Hides missing cases!
        };
    }

    // GOOD: no default, compiler catches new subtypes
    static String goodDescribe(Shape shape) {
        return switch (shape) {
            case Circle c -> "Circle";
            case Rectangle r -> "Rectangle";
            // No default — compiler will flag if new subtype added
        };
    }

    sealed interface Shape permits Circle, Rectangle { }
    record Circle(double radius) implements Shape { }
    record Rectangle(double length, double width) implements Shape { }

    public static void main(String[] args) {
        System.out.println(badDescribe(new Circle(5)));     // Circle
        System.out.println(goodDescribe(new Circle(5)));    // Circle
    }
}
```

**Expected Output:**
```
Circle
Circle
```

**Step-by-Step Explanation:**

1. The `badDescribe` method uses `default`, which means if a new permitted subtype is added, the compiler will not warn.
2. The `goodDescribe` method omits `default`, so the compiler will report an error if the switch is no longer exhaustive.
3. This demonstrates the best practice of omitting `default` for sealed hierarchies.

### Real-World Cases

- **Payment Systems:** Exhaustive handling of payment results.
- **Event Processing:** Exhaustive matching on sealed event types.
- **State Machines:** Exhaustive matching on sealed state types.

---

## References Links

- JEP 394: Pattern Matching for instanceof - https://bugs.openjdk.org/browse/jdk-8250623
- JEP 440: Record Patterns - https://bugs.openjdk.org/browse/JDK-8300541
- JEP 441: Pattern Matching for switch - https://bugs.openjdk.org/browse/JDK-8300542
- Pattern Matching for the instanceof Operator — Oracle Java Language Updates - https://docs.oracle.com/en/java/javase/16/language/pattern-matching-instanceof-operator.html
- Record Patterns — Oracle Java Language Updates - https://docs.oracle.com/en/java/javase/21/language/record-patterns.html
- Pattern Matching for switch — Oracle Java Language Updates - https://docs.oracle.com/en/java/javase/21/language/pattern-matching-switch.html
- Pattern Matching for switch (Third Preview) — OpenJDK JEP 427 - https://openjdk.org/jeps/427
- Java Type Patterns: Pattern Matching, Records, Sealed Classes, and Generics — Cleverence - https://www.cleverence.com/articles/oracle-documentation/type-patterns-java-7284/
- Primitive types in patterns, instanceof, and switch — Oracle Java Specification - https://docs.oracle.com/javase/specs/jls/se26/preview/specs/primitive-types-in-patterns-instanceof-switch-jls.html
- Pattern Matching for switch — Baeldung - https://www.baeldung.com/java-pattern-matching-switch
- Java 21 Pattern Matching — Azul - https://www.azul.com/blog/java-21-pattern-matching/