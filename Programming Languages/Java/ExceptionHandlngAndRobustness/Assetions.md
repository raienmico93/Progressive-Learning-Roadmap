# Java Assertions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** An assertion is a statement in the Java programming language that enables you to test your assumptions about your program. Each assertion contains a boolean expression that you believe will be true when the assertion executes. If it is not true, the system will throw an error.

**Technical Definition:** "An assertion is an `assert` statement containing a boolean expression. An assertion is either *enabled* or *disabled*". "If the assertion is enabled, execution of the assertion causes evaluation of the boolean expression and an error is reported if the expression evaluates to `false`". The assertion statement has two forms: `assert Expression1;` and `assert Expression1 : Expression2;`. If `Expression1` evaluates to `false`, an `AssertionError` is thrown. The second form allows a detail message to be included.

**Beginner-Friendly Explanation:** An assertion is like a safety checklist that you run while developing your program. For example, if you're writing a method that calculates the speed of a particle, you might assert that the calculated speed is less than the speed of light. If the assertion fails, it means your assumption was wrong, and there's likely a bug in your code. Assertions are like training wheels — they help you catch mistakes during development, but they are usually removed (disabled) when the program is deployed to production.

### Key Characteristics

- **Developer-Focused:** Assertions are intended for testing assumptions made by the programmer, not for handling runtime errors.
- **Disabled by Default:** Assertions are disabled at runtime unless explicitly enabled with the `-ea` flag.
- **Zero Production Overhead:** When disabled, assertions are essentially empty statements with no performance cost.
- **Two Forms:** Simple form (boolean expression only) and detailed form (boolean expression plus a message).
- **Throws `AssertionError`:** When an assertion fails, an `AssertionError` is thrown.
- **Not for Argument Checking:** Assertions should not be used for validating arguments in public methods.
- **Not for Side Effects:** The boolean expression in an assertion must not have side effects.

### Prerequisites

- Basic understanding of Java syntax and boolean expressions
- Familiarity with compiling and running Java programs
- Knowledge of the command-line `java` tool
- Understanding of exceptions and error handling (for comparison)

### Related Programming Areas

- **Debugging:** Assertions are a debugging aid for finding logic errors.
- **Testing:** Assertions complement unit testing frameworks like JUnit.
- **Design by Contract:** Assertions support preconditions, postconditions, and invariants.
- **Code Documentation:** Assertions document assumptions about program state.
- **Performance:** Disabled assertions have zero runtime cost.

### Core Concepts / Features

---

## 1. `assert`

### Definitions

**Core Definition:** `assert` is a keyword in the Java programming language, introduced in Java 1.4, used to declare an assertion statement.

**Technical Definition:** "An assertion is an `assert` statement containing a boolean expression. An assertion is either *enabled* or *disabled*". The `assert` keyword is used to declare assertions in two forms.

**Beginner-Friendly Explanation:** `assert` is the keyword you use to tell Java "I believe this condition is true." If the condition is false when the code runs, Java will throw an error to let you know your assumption was wrong.

### Purposes

- To test programmer assumptions about program state
- To detect logic errors early during development
- To document preconditions, postconditions, and invariants
- To increase confidence that the program is free of errors
- To serve as an executable form of documentation

### Syntax Rules and Structure

**Complete General Syntax:**

```
assert Expression1;
assert Expression1 : Expression2;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `assert` | Keyword that begins the assertion statement. |
| `Expression1` | A boolean expression that the programmer believes is true. |
| `:` | Separator (only in the second form). |
| `Expression2` | An expression whose value is used as the detail message for the `AssertionError`. |

**Syntax Rules:**

- `Expression1` must have type `boolean` or a compile-time error occurs.
- `Expression2` must have a value; it cannot be a `void` method invocation.
- The detail message is passed to the `AssertionError` constructor, which uses its string representation.
- The `assert` statement is a statement, not a method; it does not use parentheses like method calls.

**Constraints and Limitations:**

- Assertions are disabled by default at runtime.
- The boolean expression must not have side effects.
- Assertions should not be used for argument checking in public methods.
- Assertions should not be used for runtime error checking.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Basic Assertion**

```java
// File: BasicAssertDemo.java
public class BasicAssertDemo {
    public static void main(String[] args) {
        int x = 10;
        // Assert that x is positive
        assert x > 0;

        System.out.println("x is positive: " + x);
    }
}
```

**Expected Output (with `-ea`):**
```
x is positive: 10
```

**Expected Output (without `-ea`):**
```
x is positive: 10
```

**Step-by-Step Explanation:**

1. The `assert x > 0;` statement declares an assertion.
2. When assertions are enabled (`-ea`), the expression `x > 0` is evaluated. Since `10 > 0` is `true`, nothing happens.
3. When assertions are disabled (default), the statement is ignored entirely.
4. The program continues and prints the message.

### Real-World Cases

- **Method Preconditions:** Asserting that input parameters meet certain conditions in private methods.
- **Postconditions:** Asserting that a method's return value satisfies certain properties.
- **Class Invariants:** Asserting that an object's state remains consistent after operations.
- **Control Flow:** Asserting that a particular branch is unreachable.

---

## 2. Assertion Statements

### Definitions

**Core Definition:** An assertion statement is a complete statement that uses the `assert` keyword to declare an assertion.

**Technical Definition:** "An assertion is an `assert` statement containing a boolean expression. An assertion is either *enabled* or *disabled*". "If the assertion is enabled, execution of the assertion causes evaluation of the boolean expression and an error is reported if the expression evaluates to `false`".

**Beginner-Friendly Explanation:** An assertion statement is the full sentence you write to declare an assumption. It says "I assert that this condition is true."

### Purposes

- To embed diagnostic tests directly into the code
- To verify assumptions without writing separate test code
- To catch bugs at the point where they occur
- To document and enforce invariants

### Syntax Rules and Structure

**Complete General Syntax:**

```
// Form 1: Simple
assert booleanExpression;

// Form 2: With detail message
assert booleanExpression : detailExpression;
```

**Component Breakdown:**

| Form | Syntax | Description |
|------|--------|-------------|
| Simple | `assert condition;` | Throws `AssertionError` with no message if condition is false. |
| Detailed | `assert condition : message;` | Throws `AssertionError` with the message if condition is false. |

**Syntax Rules:**

- The boolean expression is evaluated only when assertions are enabled.
- The detail message expression is also evaluated only when assertions are enabled.
- The detail message is not a user-level error message; it is for developers.
- The detail message should capture the details of the assertion failure.

**Constraints and Limitations:**

- The detail message should not be internationalised.
- The detail message is meant to be interpreted with a full stack trace.
- The second form should be used when the program has additional diagnostic information.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Both Forms of Assertion**

```java
// File: AssertionStatementsDemo.java
public class AssertionStatementsDemo {
    public static void main(String[] args) {
        int x = 10;
        int y = 20;

        // Form 1: Simple assertion
        assert x < y;

        // Form 2: Assertion with detail message
        assert x < y : "x: " + x + ", y: " + y;

        System.out.println("Both assertions passed");
    }
}
```

**Expected Output (with `-ea`):**
```
Both assertions passed
```

**Expected Output (without `-ea`):**
```
Both assertions passed
```

**Step-by-Step Explanation:**

1. Both assertions evaluate `x < y` (i.e., `10 < 20`), which is `true`.
2. Since the condition is true, no `AssertionError` is thrown.
3. The detail message expression is only evaluated when the assertion is enabled and the condition is false.
4. The program prints the success message.

**Example 2: Failing Assertion**

```java
// File: FailingAssertDemo.java
public class FailingAssertDemo {
    public static void main(String[] args) {
        int age = -5;
        assert age >= 0 : "Age cannot be negative: " + age;
        System.out.println("Age: " + age);
    }
}
```

**Expected Output (with `-ea`):**
```
Exception in thread "main" java.lang.AssertionError: Age cannot be negative: -5
    at FailingAssertDemo.main(FailingAssertDemo.java:4)
```

**Expected Output (without `-ea`):**
```
Age: -5
```

**Step-by-Step Explanation:**

1. With `-ea`, the assertion evaluates `age >= 0` (i.e., `-5 >= 0`), which is `false`.
2. An `AssertionError` is thrown with the detail message `"Age cannot be negative: -5"`.
3. The stack trace shows the file and line number where the assertion failed.
4. Without `-ea`, the assertion is ignored, and the program continues normally.

### Real-World Cases

- **Private Method Validation:** Asserting preconditions in private helper methods.
- **Loop Invariants:** Asserting that a loop invariant holds at the start of each iteration.
- **State Validation:** Asserting that an object is in a valid state before performing operations.
- **Unreachable Code:** Asserting `false` in a branch that should never be reached.

---

## 3. Assertion Messages

### Definitions

**Core Definition:** An assertion message is the detail expression provided in the second form of the assertion statement, used to communicate information about the assertion failure.

**Technical Definition:** "Use this version of the `assert` statement to provide a detail message for the `AssertionError`. The system passes the value of `Expression2` to the appropriate `AssertionError` constructor, which uses the string representation of the value as the error's detail message".

**Beginner-Friendly Explanation:** An assertion message is like a note you leave for yourself or your teammates explaining what went wrong. If the assertion fails, the message tells you exactly which values caused the failure.

### Purposes

- To capture and communicate the details of an assertion failure
- To aid diagnosis and debugging
- To provide context for the failed condition
- To include relevant variable values in the error report

### Syntax Rules and Structure

```
assert condition : detailExpression;
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `condition` | The boolean expression being asserted. |
| `detailExpression` | An expression whose value is used as the detail message. |

**Syntax Rules:**

- The detail expression must produce a value (cannot be `void`).
- The string representation of the value is used as the message.
- The message is only constructed when the assertion is enabled and the condition is false.
- The message is intended for developers, not end users.

**Constraints and Limitations:**

- The message should not be internationalised.
- The message is not a user-level error message.
- Expensive message construction should be avoided because it may execute when the assertion is enabled.

### Annotated Complete Step-by-Step Code Examples

```java
// File: AssertionMessageDemo.java
public class AssertionMessageDemo {
    public static int calculateAverage(int[] numbers) {
        assert numbers != null : "Array must not be null";
        assert numbers.length > 0 : "Array must not be empty";

        int sum = 0;
        for (int n : numbers) {
            sum += n;
        }

        int average = sum / numbers.length;

        // Postcondition: average should be within the range of min and max
        assert average >= numbers[0] || average <= numbers[numbers.length - 1]
            : "Average out of range: average=" + average + ", array=" + java.util.Arrays.toString(numbers);

        return average;
    }

    public static void main(String[] args) {
        int[] data = {10, 20, 30, 40, 50};
        System.out.println("Average: " + calculateAverage(data));
    }
}
```

**Expected Output (with `-ea`):**
```
Average: 30
```

**Step-by-Step Explanation:**

1. The first assertion checks that the array is not `null`.
2. The second assertion checks that the array is not empty.
3. The postcondition assertion checks that the calculated average is within the range of the array's first and last elements.
4. If any assertion fails, the detail message includes the specific values for debugging.
5. In this example, all assertions pass, so the average is printed.

### Real-World Cases

- **Debugging Complex Calculations:** Including intermediate values in assertion messages.
- **Array Processing:** Including array contents in failure messages.
- **State Validation:** Including object state in assertion messages.
- **Boundary Checks:** Including boundary values and actual values in messages.

---

## 4. Assertion Enablement

### Definitions

**Core Definition:** Assertion enablement is the process of activating assertion statements at runtime using command-line flags.

**Technical Definition:** "To ensure that assertions are not a performance liability in deployed applications, assertions can be enabled or disabled when the program is started, and are disabled by default". "The `-enableassertions` (`-ea`) option enables assertions".

**Beginner-Friendly Explanation:** Assertion enablement is like flipping a switch that turns on all the safety checks in your program. By default, the switch is off (assertions are disabled). You turn it on during development and testing.

### Purposes

- To activate assertions during development and testing
- To deactivate assertions in production for performance
- To selectively enable assertions for specific packages or classes
- To control assertion behaviour at runtime

### Syntax Rules and Structure

**Command-Line Flags Table:**

| Flag | Description |
|------|-------------|
| `-ea` or `-enableassertions` | Enables assertions in all classes except system classes. |
| `-ea:<package>...` | Enables assertions in the specified package and subpackages. |
| `-ea:<class>` | Enables assertions in the specified class. |
| `-da` or `-disableassertions` | Disables assertions in all classes. |
| `-esa` or `-enablesystemassertions` | Enables assertions in system classes. |
| `-dsa` or `-disablesystemassertions` | Disables assertions in system classes. |

**Syntax Rules:**

- Assertions are disabled by default; you must use `-ea` to enable them.
- Multiple flags are processed in order.
- The `-ea` flag without arguments enables assertions in all packages and classes except system classes.
- To enable assertions in system classes, use `-esa`.

**Constraints and Limitations:**

- Assertions should be enabled during development and testing.
- Assertions should typically be disabled in production for performance.
- Do not rely on assertions for program correctness.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Enabling Assertions for a Specific Package**

```java
// File: com/example/Calculator.java
package com.example;

public class Calculator {
    public int divide(int a, int b) {
        assert b != 0 : "Divisor cannot be zero";
        return a / b;
    }
}
```

```java
// File: Main.java
import com.example.Calculator;

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.divide(10, 2));
    }
}
```

**Command:**
```
java -ea:com.example... Main
```

**Expected Output:**
```
5
```

**Step-by-Step Explanation:**

1. The assertion is in the `com.example` package.
2. The flag `-ea:com.example...` enables assertions only for that package and its subpackages.
3. The assertion is evaluated when `divide` is called.
4. Since `b = 2`, the assertion passes.

**Example 2: Disabling Assertions in Production**

```java
// File: ProductionDemo.java
public class ProductionDemo {
    public static void main(String[] args) {
        int value = 10;
        assert value > 0 : "Value must be positive: " + value;
        System.out.println("Value: " + value);
    }
}
```

**Command (Production):**
```
java -da ProductionDemo
```

**Expected Output:**
```
Value: 10
```

**Step-by-Step Explanation:**

1. The `-da` flag disables all assertions.
2. The assertion is not evaluated, so it has no performance cost.
3. The program runs without the assertion check.

### Real-World Cases

- **Development:** Enabling assertions in IDE run configurations.
- **Testing:** Enabling assertions in test runners and CI pipelines.
- **Production:** Disabling assertions in production deployments.
- **Selective Enablement:** Enabling assertions only for critical packages in production.

---

## 5. `-ea` Flag

### Definitions

**Core Definition:** The `-ea` flag (short for `-enableassertions`) is a command-line option that enables assertion checking in Java classes.

**Technical Definition:** "With no arguments, `-enableassertions` (`-ea`) enables assertions in all packages and classes" except system classes. With a package name ending in `...`, the switch enables assertions in the specified package and any subpackages. With a class name argument, the switch enables assertions in the specified class.

**Beginner-Friendly Explanation:** `-ea` is the switch you flip to turn on assertions. Without it, all your `assert` statements are ignored.

### Purposes

- To enable assertions during development and testing
- To selectively enable assertions for specific packages or classes
- To verify program assumptions at runtime

### Syntax Rules and Structure

```
java -ea MainClass
java -ea:com.example... MainClass
java -ea:com.example.MyClass MainClass
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `-ea` | Enables assertions in all classes except system classes. |
| `-ea:package...` | Enables assertions in the package and subpackages. |
| `-ea:class` | Enables assertions in the specified class. |

**Syntax Rules:**

- `-ea` applies to all class loaders and system classes, except when no arguments are provided.
- Multiple `-ea` and `-da` flags can be combined.
- The flags are processed in order before loading any classes.

**Constraints and Limitations:**

- Assertions should not be enabled in production for performance reasons.
- System classes are not covered by `-ea` without arguments; use `-esa` for system classes.

### Annotated Complete Step-by-Step Code Examples

```java
// File: EaFlagDemo.java
public class EaFlagDemo {
    public static void main(String[] args) {
        int x = 5;
        assert x > 0 : "x must be positive";
        System.out.println("x = " + x);
    }
}
```

**Command:**
```
java -ea EaFlagDemo
```

**Expected Output:**
```
x = 5
```

**Command (with failing assertion):**
```
java -ea EaFlagDemo
```
(if `x` were negative):
```
Exception in thread "main" java.lang.AssertionError: x must be positive
    at EaFlagDemo.main(EaFlagDemo.java:5)
```

**Step-by-Step Explanation:**

1. The `-ea` flag enables assertions.
2. The assertion `x > 0` is evaluated. Since `5 > 0`, it passes.
3. If `x` were negative, an `AssertionError` would be thrown with the message.
4. Without `-ea`, the assertion would be ignored.

### Real-World Cases

- **IDE Configuration:** Adding `-ea` to VM options in IntelliJ IDEA, Eclipse, or VS Code.
- **Build Tools:** Configuring Maven Surefire or Gradle Test tasks with `-ea`.
- **CI/CD Pipelines:** Running tests with assertions enabled.

---

## 6. `-da` Flag

### Definitions

**Core Definition:** The `-da` flag (short for `-disableassertions`) is a command-line option that disables assertion checking in Java classes.

**Technical Definition:** "The `-disableassertions` (`-da`) option applies to all class loaders and to system classes (which don't have a class loader)". With no arguments, `-da` disables assertions in all packages and classes except system classes.

**Beginner-Friendly Explanation:** `-da` is the switch you flip to turn off assertions. This is the default behaviour, so you only need `-da` if you want to disable assertions that were previously enabled.

### Purposes

- To disable assertions in production for performance
- To selectively disable assertions for specific packages or classes
- To override previous `-ea` flags

### Syntax Rules and Structure

```
java -da MainClass
java -da:com.example... MainClass
java -da:com.example.MyClass MainClass
```

**Component Breakdown:**

| Syntax | Description |
|--------|-------------|
| `-da` | Disables assertions in all classes except system classes. |
| `-da:package...` | Disables assertions in the package and subpackages. |
| `-da:class` | Disables assertions in the specified class. |

**Syntax Rules:**

- `-da` is the default; assertions are disabled unless `-ea` is specified.
- Multiple `-ea` and `-da` flags are processed in order.
- Use `-dsa` to disable assertions in system classes.

**Constraints and Limitations:**

- Disabling assertions removes their diagnostic value.
- Do not rely on assertions for program correctness.

### Annotated Complete Step-by-Step Code Examples

```java
// File: DaFlagDemo.java
public class DaFlagDemo {
    public static void main(String[] args) {
        int x = -5;
        assert x > 0 : "x must be positive: " + x;
        System.out.println("x = " + x);
    }
}
```

**Command:**
```
java -da DaFlagDemo
```

**Expected Output:**
```
x = -5
```

**Command (with `-ea`):**
```
java -ea DaFlagDemo
```

**Expected Output:**
```
Exception in thread "main" java.lang.AssertionError: x must be positive: -5
    at DaFlagDemo.main(DaFlagDemo.java:5)
```

**Step-by-Step Explanation:**

1. With `-da`, the assertion is disabled, so the negative value does not cause an error.
2. The program prints `x = -5`.
3. With `-ea`, the assertion is enabled, and the negative value triggers an `AssertionError`.

### Real-World Cases

- **Production Deployment:** Running Java applications with `-da` (default).
- **Selective Disabling:** Disabling assertions for specific packages that are performance-critical.
- **Testing:** Overriding `-ea` with `-da` for specific test scenarios.

---

## 7. Assertion Use Cases

### Definitions

**Core Definition:** Assertion use cases are the scenarios where assertions are appropriate and effective for detecting programming errors.

**Technical Definition:** "There are many situations where it is good to use assertions, including: Internal Invariants, Control-Flow Invariants, Preconditions, Postconditions, and Class Invariants".

**Beginner-Friendly Explanation:** Assertions are best used for checking things that you, the programmer, believe are true — things that should never be false if your code is correct. They are not for checking things outside your control, like user input or file availability.

### Purposes

- To verify internal invariants that should always hold
- To check control-flow invariants (unreachable code)
- To validate preconditions of private methods
- To validate postconditions of methods
- To check class invariants

### Key Use Cases Table

| Use Case | Description | Example |
|----------|-------------|---------|
| Internal Invariants | Assertions that check conditions within a method | `assert x > 0;` |
| Control-Flow Invariants | Assertions in branches that should never execute | `assert false : "Unreachable";` |
| Preconditions | Assertions that check method preconditions | `assert param != null;` |
| Postconditions | Assertions that check method results | `assert result >= 0;` |
| Class Invariants | Assertions that check object state consistency | `assert balance >= 0;` |

**Syntax Rules:**

- Use assertions for internal logic checks within your code.
- Use exceptions for error conditions outside your immediate code's control.
- Use assertions for things you believe will never happen.
- Use exceptions for things that might happen and need graceful handling.

**Constraints and Limitations:**

- Do not use assertions for argument checking in public methods.
- Do not use assertions to do any work that your application requires for correct operation.
- Do not use assertions to verify the absence of runtime errors.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Preconditions and Postconditions**

```java
// File: UseCasesDemo.java
public class UseCasesDemo {

    /**
     * Calculates the square root of a number.
     * Precondition: number >= 0
     * Postcondition: result >= 0
     */
    public static double sqrt(double number) {
        // Precondition assertion
        assert number >= 0 : "Number must be non-negative: " + number;

        double result = Math.sqrt(number);

        // Postcondition assertion
        assert result >= 0 : "Result must be non-negative: " + result;

        return result;
    }

    public static void main(String[] args) {
        System.out.println(sqrt(25.0));  // Output: 5.0
    }
}
```

**Expected Output (with `-ea`):**
```
5.0
```

**Step-by-Step Explanation:**

1. The precondition assertion checks that the input is non-negative.
2. The postcondition assertion checks that the result is non-negative.
3. Both assertions pass for the valid input `25.0`.
4. If the input were negative, the precondition assertion would fail.

**Example 2: Control-Flow Invariant**

```java
// File: ControlFlowDemo.java
public class ControlFlowDemo {
    public static String getDayType(int day) {
        switch (day) {
            case 1: case 7:
                return "Weekend";
            case 2: case 3: case 4: case 5: case 6:
                return "Weekday";
            default:
                assert false : "Invalid day: " + day;
                return "Unknown";
        }
    }

    public static void main(String[] args) {
        System.out.println(getDayType(1));  // Output: Weekend
        System.out.println(getDayType(3));  // Output: Weekday
    }
}
```

**Expected Output (with `-ea`):**
```
Weekend
Weekday
```

**Step-by-Step Explanation:**

1. The `assert false` in the `default` branch indicates that this branch should never be reached.
2. If an invalid day is passed, the assertion fails with a descriptive message.
3. When assertions are disabled, the `default` branch returns `"Unknown"` as a fallback.

### Real-World Cases

- **Algorithm Implementation:** Asserting loop invariants in sorting and searching algorithms.
- **Data Structure Operations:** Asserting that a stack is not empty before popping.
- **Financial Calculations:** Asserting that a balance never goes negative.
- **Parser Development:** Asserting that a token stream is not exhausted prematurely.

---

## 8. Assertion Best Practices

### Definitions

**Core Definition:** Assertion best practices are guidelines for using assertions effectively without introducing bugs or security vulnerabilities.

**Technical Definition:** Best practices derive from authoritative sources including Oracle's "Programming With Assertions" guide, the SEI CERT Oracle Coding Standard for Java, and the Java Language Specification. Key principles include using assertions for internal logic checks only, avoiding side effects in assertion expressions, and never relying on assertions for program correctness.

**Beginner-Friendly Explanation:** Best practices are like rules of the road for assertions. They help you use assertions correctly so they help you find bugs without causing problems in production.

### Purposes

- To improve code reliability by catching bugs early
- To avoid introducing new bugs through assertion misuse
- To ensure assertions do not affect program behaviour
- To maintain consistent style across a codebase

### Key Best Practices Table

| Practice | Description | Source |
|----------|-------------|--------|
| Use for Internal Logic Only | Assertions should check programmer assumptions, not runtime errors. | SEI CERT |
| Never Use for Public Method Arguments | Public method contracts require exceptions, not assertions. | Oracle |
| Never Use for Runtime Error Checking | Input availability, file errors, network errors are not assertion material. | SEI CERT |
| Never Use Side Effects | The boolean expression must not modify program state. | JPCERT |
| Provide Meaningful Messages | Detail messages should help diagnose the failure. | Oracle |
| Do Not Use for Work Required for Correctness | Assertions may be disabled; never rely on them for logic. | Oracle |
| Use for Preconditions in Private Methods | Private method preconditions are good candidates for assertions. | Oracle |
| Use for Postconditions and Invariants | Assertions are ideal for checking method results and class invariants. | Oracle |
| Use `assert false` for Unreachable Code | This documents control-flow invariants. | Oracle |
| Enable Assertions During Testing | Use `-ea` in development and CI/CD pipelines. | Oracle |
| Disable Assertions in Production | Use `-da` (default) for performance. | Oracle |
| Avoid Expensive Assertions | Expensive checks should be disabled in production. | Oracle |
| Do Not Internationalize Assertion Messages | Messages are for developers, not users. | Oracle |
| Consider Assertion Impact on Inlining | Assertions can increase method size and affect JIT inlining. | OpenJDK |

### Annotated Complete Step-by-Step Code Examples

```java
// File: BestPracticesDemo.java
import java.util.Objects;

public class BestPracticesDemo {

    // GOOD: Precondition assertion in a private method
    private static int divide(int a, int b) {
        assert b != 0 : "Divisor must not be zero: b=" + b;
        return a / b;
    }

    // GOOD: Postcondition assertion
    private static int absoluteValue(int x) {
        int result = (x < 0) ? -x : x;
        assert result >= 0 : "Absolute value must be non-negative: " + result;
        return result;
    }

    // GOOD: Control-flow invariant
    private static String getStatus(int code) {
        if (code == 200) return "OK";
        if (code == 404) return "Not Found";
        if (code == 500) return "Server Error";
        assert false : "Unknown status code: " + code;
        return "Unknown";
    }

    // BAD: Using assertion for public method argument checking
    public static void processOrder(String orderId, double amount) {
        // BAD: This should be an exception, not an assertion
        // assert orderId != null : "Order ID cannot be null";
        // assert amount > 0 : "Amount must be positive";

        // GOOD: Use exceptions for public method validation
        Objects.requireNonNull(orderId, "Order ID cannot be null");
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive: " + amount);
        }

        System.out.println("Processing order " + orderId + " for $" + amount);
    }

    // BAD: Side effect in assertion
    private static int counter = 0;
    private static int incrementAndReturn() {
        counter++;  // Side effect!
        return counter;
    }

    // BAD: This assertion has a side effect
    // assert incrementAndReturn() > 0 : "Counter must be positive";

    public static void main(String[] args) {
        System.out.println("Divide: " + divide(10, 2));
        System.out.println("Absolute: " + absoluteValue(-5));
        System.out.println("Status: " + getStatus(200));
        processOrder("ORD-001", 99.99);
    }
}
```

**Expected Output (with `-ea`):**
```
Divide: 5
Absolute: 5
Status: OK
Processing order ORD-001 for $99.99
```

**Step-by-Step Explanation:**

1. `divide` uses an assertion for a precondition in a private method — good practice.
2. `absoluteValue` uses an assertion for a postcondition — good practice.
3. `getStatus` uses `assert false` for a control-flow invariant — good practice.
4. `processOrder` uses exceptions (not assertions) for public method argument checking — good practice.
5. The side-effect example shows what to avoid: assertions must not modify state.

### Real-World Cases

- **Library Development:** Using assertions internally while providing exceptions for public APIs.
- **Algorithm Development:** Asserting loop invariants to catch logic errors.
- **Code Reviews:** Best practices form the basis for reviewing assertion usage.
- **Static Analysis:** Tools like IntelliJ IDEA report assertion misuse.

---

## References Links

- Programming With Assertions — Oracle Java Documentation - https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html
- Programming With Assertions — OpenJDK - https://cr.openjdk.org/~naoto/idna2008/docs_www/technotes/guides/language/assert.html
- Chapter 14. Blocks and Statements — Java Language Specification, Java SE 21 Edition - https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html
- Diagnostic tests can be incorporated into programs by using the `assert` statement — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88867563
- EXP06-J. アサーションを行う式で副作用を発生させない — JPCERT - https://www.jpcert.or.jp/java-rules/exp06-j.html
- Java Development Kit Version 17 API Specification — `-disableassertions` (`-da`) option - https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html
- `-disableassertions` (`-da`) option applies to all class loaders — Oracle JDK 17 Documentation - https://docs.oracle.com/en/java/javase/17/docs/specs/man/java.html
- 8. The `assert` statement — JavaHyperText and Data Structures, Cornell University - https://www.cs.cornell.edu/courses/JavaAndDS/eclipse/Ecl10Assert.html
- On Asserts and Exceptions — Stack Overflow - https://browse.library.kiwix.org/content/stackoverflow.com_en_all_nopic_2022-07/questions/4863694/on-asserts-and-exceptions-java
- JDK-6445664: Eliminate remaining performance penalty for using `assert` — OpenJDK Bug System - https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6445664
- Assert in Java — BrowserStack - https://www.browserstack.com/guide/assert-in-java
- MSC60-J. Do not use assertions to verify the absence of runtime errors — SEI CERT - https://wiki.sei.cmu.edu/confluence/display/java/MSC60-J.+Do+not+use+assertions+to+verify+the+absence+of+runtime+errors