# Java Exception Handling: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Exception handling is a programming language mechanism that allows a program to detect and respond to exceptional conditions (errors or unexpected events) that occur during execution, preventing abrupt termination and enabling graceful recovery.

**Technical Definition:** In Java, exception handling is implemented through the `Throwable` class hierarchy and the `try`, `catch`, and `finally` statements. When an exceptional condition arises, an exception object is thrown from the point of error. The runtime system searches the call stack for a handler—a `catch` block whose parameter type matches the thrown exception. If no handler is found, the thread terminates and the default exception handler prints a stack trace. Java distinguishes between checked exceptions (which must be caught or declared in a `throws` clause, per the Catch or Specify Requirement) and unchecked exceptions (which are not subject to this requirement).

**Beginner-Friendly Explanation:** Exception handling is like having a safety net for your program. When something goes wrong—a file is missing, a network connection fails, or a number is divided by zero—the program throws an "exception" to signal the problem. Instead of crashing, the program can "catch" that exception and decide what to do: log an error, use a default value, retry the operation, or clean up resources. The `try` block contains the risky code, the `catch` block contains the recovery logic, and the `finally` block contains cleanup code that runs no matter what.

### Key Characteristics

Java exception handling is characterized by **structured error recovery** (using `try`, `catch`, and `finally` blocks rather than `goto`-style error codes), **type safety** (exceptions are objects with inheritance hierarchies, allowing precise or general handling), **propagation** (unhandled exceptions propagate up the call stack until a handler is found), and **resource management** (the `finally` block and, since Java 7, the try-with-resources statement ensure that resources are released). Java enforces the **Catch or Specify Requirement** for checked exceptions, meaning the compiler forces developers to either handle or declare such exceptions, promoting robust error handling.

### Prerequisites

Readers should be familiar with Java's class hierarchy (particularly `Throwable`, `Exception`, and `Error`), the difference between checked and unchecked exceptions, and basic object-oriented concepts such as inheritance and polymorphism. A basic understanding of method call stacks is helpful for understanding exception propagation.

### Related Programming Areas with Explanation

Exception handling is related to **software reliability engineering** (through fault tolerance and graceful degradation), **resource management** (through deterministic cleanup with `finally` and try-with-resources), **API design** (through the design of custom exception hierarchies and `throws` clauses), and **concurrent programming** (where exceptions in threads must be handled carefully to avoid silent failures).


## 1. Core Keywords

### 1.1 try

#### Definitions

- **Core Definition:** Defines a block of code to be tested for exceptions while it is being executed.
- **Technical Definition:** A `try` statement (JLS §14.20) encloses a block of statements in which exceptions may be thrown. The `try` block is the first component of a `try`-`catch`-`finally` construct. If an exception is thrown within the `try` block, the runtime system searches the associated `catch` clauses for a matching handler. If no exception is thrown, all `catch` clauses are skipped and execution continues after the `try` statement (after the `finally` block, if present).
- **Beginner-Friendly Explanation:** The `try` block is where you put code that might fail. It is like saying, "I am going to attempt this, and if something goes wrong, I have a plan."

#### Purposes

- To demarcate a section of code in which exceptions are to be monitored.
- To provide the context for associated `catch` and `finally` blocks.
- To enable structured recovery from errors without terminating the program.

#### Syntax Structures and Rules

Complete general syntax:

```java
try {
    // Statements that may throw exceptions
}
```

Component breakdown:

- The `try` keyword is followed by a block of statements enclosed in braces.
- The block may throw any exception; the exception is matched against the `catch` clauses that follow.
- The `try` block must be followed by at least one `catch` block, or a `finally` block, or both (or, since Java 7, a resource specification for try-with-resources).

Syntax rules: A `try` statement without `catch` or `finally` is a compile-time error unless it is a try-with-resources statement. The `try` block itself may contain nested `try` statements.

Constraints and limitations: The `try` block cannot exist in isolation. Local variables declared inside the `try` block are not visible in the `catch` or `finally` blocks.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Basic try block with no exception**

```java
public class TryExample {
    public static void main(String[] args) {
        // Step 1: The try block contains code that might throw
        try {
            int result = 10 / 2;
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        System.out.println("Program continues");
    }
}
```

Expected Output:

```
Result: 5
Program continues
```

**Why the code produces this result:** The division `10 / 2` does not throw an exception, so the `catch` block is skipped entirely. Execution continues after the `try`-`catch` statement.

**Example 2: try block with an exception**

```java
public class TryExceptionExample {
    public static void main(String[] args) {
        try {
            int result = 10 / 0; // This throws ArithmeticException
            System.out.println("This line is never reached");
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
        System.out.println("Program continues");
    }
}
```

Expected Output:

```
Caught: / by zero
Program continues
```

**Why the code produces this result:** The division by zero throws an `ArithmeticException`. The runtime immediately transfers control to the matching `catch` block. The line after the exception in the `try` block is never executed. After the `catch` block, execution resumes.

**Real-World Cases with Explanation**

In a file-reading application, the `try` block contains code that opens and reads a file, which may throw `IOException`. In a database application, the `try` block contains SQL execution code that may throw `SQLException`. In a network client, the `try` block contains socket operations that may throw `IOException` or `ConnectException`.

**References Links**

- The try Block (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/try.html
- JLS §14.20: The try statement – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20

### 1.2 catch

#### Definitions

- **Core Definition:** Defines a block of code to be executed if a specific exception occurs in the `try` block.
- **Technical Definition:** A `catch` clause (JLS §14.20) specifies an exception parameter whose type must be a subclass of `Throwable`. When an exception is thrown in the associated `try` block, the runtime system examines each `catch` clause in order. The first `catch` clause whose parameter type is assignable from the runtime type of the exception is selected; its block is executed with the exception object bound to the parameter.
- **Beginner-Friendly Explanation:** The `catch` block is your recovery plan. It says, "If this specific type of error happens, here is what I want to do." You can have multiple `catch` blocks to handle different types of errors differently.

#### Purposes

- To handle specific types of exceptions and provide recovery logic.
- To prevent unhandled exceptions from propagating and terminating the program.
- To log, translate, or recover from errors in a controlled manner.

#### Syntax Structures and Rules

Complete general syntax:

```java
catch (ExceptionType parameterName) {
    // Recovery or handling logic
}
```

Component breakdown:

- `ExceptionType`: the type of exception this handler can catch. Must be a subtype of `Throwable`.
- `parameterName`: the variable that will hold the caught exception object.
- The catch block is executed only if the exception type matches.

Syntax rules: Multiple `catch` blocks may be attached to a single `try`. The first matching catch block is executed; subsequent catch blocks are skipped. A catch block can rethrow the exception or throw a different exception.

Constraints and limitations: The exception parameter cannot be assigned a new value if it is implicitly final (in a multi-catch) or explicitly declared final. It is a compile-time error to have a catch clause whose exception type is a supertype of a preceding catch clause's exception type (unreachable code).

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Catching a specific exception**

```java
public class CatchExample {
    public static void main(String[] args) {
        try {
            String s = null;
            System.out.println(s.length()); // Throws NullPointerException
        } catch (NullPointerException e) {
            System.out.println("Caught NullPointerException: " + e.getMessage());
        }
        System.out.println("Program continues");
    }
}
```

Expected Output:

```
Caught NullPointerException: null
Program continues
```

**Why the code produces this result:** Calling `length()` on a `null` reference throws a `NullPointerException`. The catch block matches the exception type and executes, printing the message (which is `null` for `NullPointerException` without a custom message).

**Example 2: Catching a broader exception type**

```java
public class CatchBroadExample {
    public static void main(String[] args) {
        try {
            int[] arr = new int[5];
            arr[10] = 100; // Throws ArrayIndexOutOfBoundsException
        } catch (RuntimeException e) {
            System.out.println("Caught RuntimeException: " + e.getClass().getSimpleName());
        }
    }
}
```

Expected Output:

```
Caught RuntimeException: ArrayIndexOutOfBoundsException
```

**Why the code produces this result:** `ArrayIndexOutOfBoundsException` is a subclass of `RuntimeException`. The catch block catches the supertype, which is valid because the thrown exception is assignable to `RuntimeException`.

**Real-World Cases with Explanation**

In a REST API client, a `catch (IOException e)` block handles network failures by retrying the request or returning a user-friendly error message. In a parsing application, a `catch (ParseException e)` block handles malformed input by logging the error and skipping the problematic record.

**References Links**

- The catch Blocks (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html
- JLS §14.20: The try statement – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20

### 1.3 finally

#### Definitions

- **Core Definition:** Defines a block of code that always executes after the `try`/`catch` blocks leave, regardless of whether an exception was thrown or caught (used for cleanup).
- **Technical Definition:** A `finally` clause (JLS §14.20.2) is executed after the `try` block and any `catch` blocks, regardless of how control leaves the `try` statement. If the `try` block completes normally, the `finally` block is executed. If an exception is thrown and caught, the `finally` block is executed after the catch block. If an exception is thrown and not caught, the `finally` block is executed before the exception propagates. The `finally` block is guaranteed to execute unless the JVM exits abnormally (e.g., `System.exit()` or a crash).
- **Beginner-Friendly Explanation:** The `finally` block is your cleanup crew. It runs no matter what—whether the code succeeded, failed, or was interrupted. It is where you close files, release connections, or reset state.

#### Purposes

- To guarantee that cleanup code is executed regardless of how the `try` block exits.
- To release resources such as file handles, network connections, or database connections.
- To restore state or perform logging that must happen in all cases.

#### Syntax Structures and Rules

Complete general syntax:

```java
try {
    // Risky code
} catch (ExceptionType e) {
    // Handler
} finally {
    // Cleanup code that always runs
}
```

Component breakdown:

- The `finally` block is optional but, if present, must follow all `catch` blocks.
- The `finally` block executes after the `try` and `catch` blocks.
- The `finally` block may contain any statements, including `return`, `break`, `continue`, or `throw`—but doing so can cause surprising behavior.

Syntax rules: A `try` statement can have `catch` blocks, a `finally` block, or both. If both are present, the `catch` blocks come first.

Constraints and limitations: If the `finally` block completes abruptly (via `return`, `break`, `continue`, or `throw`), any exception or return value from the `try` or `catch` blocks is discarded. The SEI CERT Oracle Coding Standard explicitly advises: "Never use `return`, `break`, `continue`, or `throw` statements within a `finally` block". Attempting to write to a variable in a `finally` block after a `return` in the `try` block does not change the returned value if the variable is a primitive (the return value is already determined).

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: finally executes after normal completion**

```java
public class FinallyNormalExample {
    public static void main(String[] args) {
        try {
            System.out.println("Inside try");
        } catch (Exception e) {
            System.out.println("Inside catch");
        } finally {
            System.out.println("Inside finally");
        }
        System.out.println("After try-catch-finally");
    }
}
```

Expected Output:

```
Inside try
Inside finally
After try-catch-finally
```

**Why the code produces this result:** No exception is thrown, so the `catch` block is skipped. The `finally` block executes after the `try` block completes. Execution then continues after the entire `try` statement.

**Example 2: finally executes after an exception is caught**

```java
public class FinallyCaughtExample {
    public static void main(String[] args) {
        try {
            System.out.println("Inside try");
            throw new RuntimeException("Test exception");
        } catch (RuntimeException e) {
            System.out.println("Inside catch: " + e.getMessage());
        } finally {
            System.out.println("Inside finally");
        }
        System.out.println("After try-catch-finally");
    }
}
```

Expected Output:

```
Inside try
Inside catch: Test exception
Inside finally
After try-catch-finally
```

**Why the code produces this result:** The exception is thrown in the `try` block, caught in the `catch` block, and then the `finally` block executes. Execution continues after the `try` statement.

**Example 3: finally executes even when an exception is not caught**

```java
public class FinallyUncaughtExample {
    public static void main(String[] args) {
        try {
            System.out.println("Inside try");
            throw new RuntimeException("Uncaught");
        } finally {
            System.out.println("Inside finally");
        }
        // This line is not reached because the exception propagates
    }
}
```

Expected Output:

```
Inside try
Inside finally
Exception in thread "main" java.lang.RuntimeException: Uncaught
    at FinallyUncaughtExample.main(FinallyUncaughtExample.java:5)
```

**Why the code produces this result:** The exception is not caught (there is no `catch` block). The `finally` block executes before the exception propagates up the call stack. The program terminates with an uncaught exception.

**Example 4: return in finally overrides return in try**

```java
public class FinallyReturnExample {
    public static int getValue() {
        try {
            return 1;
        } finally {
            return 2; // This return overrides the return in try
        }
    }

    public static void main(String[] args) {
        System.out.println("Returned: " + getValue());
    }
}
```

Expected Output:

```
Returned: 2
```

**Why the code produces this result:** The `return` statement in the `finally` block overrides the `return` statement in the `try` block. This is why using `return` in a `finally` block is strongly discouraged.

**Real-World Cases with Explanation**

In a file I/O application, the `finally` block closes the `FileInputStream` and `FileOutputStream` regardless of whether an `IOException` occurred. In a database application, the `finally` block closes the `Connection`, `Statement`, and `ResultSet` to prevent resource leaks. In a locking system, the `finally` block releases a lock to prevent deadlock.

**References Links**

- The finally Block (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html
- JLS §14.20.2: Execution of try-finally and try-catch-finally – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.2
- SEI CERT ERR04-J: Do not complete abruptly from a finally block – https://wiki.sei.cmu.edu/confluence/display/java/ERR04-J.+Do+not+complete+abruptly+from+a+finally+block


## 2. Advanced Catch Mechanics

### 2.1 Multiple Catch Blocks

#### Definitions

- **Core Definition:** Ordering catches from the most specific (subclass) to the most general (superclass) to avoid compile-time "unreachable code" errors.
- **Technical Definition:** When a `try` block is followed by multiple `catch` clauses, the Java compiler examines them in the order they appear. It is a compile-time error if a preceding `catch` clause can catch the same exception type or a supertype of the exception type of a subsequent `catch` clause, because the subsequent clause would be unreachable. Therefore, catch blocks must be ordered from the most specific exception type to the most general.
- **Beginner-Friendly Explanation:** If you have multiple `catch` blocks, put the most specific exceptions first. If you put a general exception like `Exception` first, it will catch everything, and the more specific catch blocks below it will never run—and the compiler will refuse to compile your code.

#### Purposes

- To handle different exception types with different recovery logic in a single `try` statement.
- To provide precise error handling for each failure mode.
- To prevent overly broad exception catching that masks specific errors.

#### Syntax Structures and Rules

Complete general syntax:

```java
try {
    // Risky code
} catch (SpecificException e) {
    // Handle specific case
} catch (BroaderException e) {
    // Handle broader case
} catch (Exception e) {
    // Handle any remaining exception
}
```

Component breakdown:

- Each `catch` clause has its own exception parameter.
- The compiler checks the ordering: a catch clause for a supertype must come after catch clauses for its subtypes.

Syntax rules: The exception types in catch clauses must be disjoint or ordered from specific to general. If a catch clause is unreachable (i.e., its exception type is a supertype of a preceding catch clause's type), the code will not compile.

Constraints and limitations: You cannot have two catch clauses for the exact same exception type. You cannot have a catch clause for a checked exception that cannot be thrown by the `try` block (this is a compile-time error).

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Correct ordering (specific to general)**

```java
public class MultipleCatchCorrectExample {
    public static void main(String[] args) {
        try {
            String s = null;
            System.out.println(s.length());
            int[] arr = new int[3];
            arr[5] = 10;
        } catch (NullPointerException e) {
            System.out.println("Caught NullPointerException");
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Caught ArrayIndexOutOfBoundsException");
        } catch (RuntimeException e) {
            System.out.println("Caught RuntimeException");
        }
    }
}
```

Expected Output:

```
Caught NullPointerException
```

**Why the code produces this result:** The `NullPointerException` is thrown first, and the first catch clause matches. The remaining catch clauses are skipped.

**Example 2: Incorrect ordering (compile-time error)**

```java
public class MultipleCatchIncorrectExample {
    public static void main(String[] args) {
        try {
            int[] arr = new int[3];
            arr[5] = 10;
        } catch (RuntimeException e) {      // Broader exception first
            System.out.println("Caught RuntimeException");
        } catch (ArrayIndexOutOfBoundsException e) { // Unreachable: compiler error
            System.out.println("Caught ArrayIndexOutOfBoundsException");
        }
    }
}
```

Expected Output:

```
Compile-time error: Unreachable catch block for ArrayIndexOutOfBoundsException.
It is already handled by the catch block for RuntimeException.
```

**Why the code does not compile:** `ArrayIndexOutOfBoundsException` is a subclass of `RuntimeException`. Since the first catch clause already catches `RuntimeException`, the second catch clause can never be reached. The Java compiler rejects unreachable code.

**Real-World Cases with Explanation**

In a file-processing application, a `try` block might throw `FileNotFoundException` (specific), `IOException` (broader), and `Exception` (most general). Handling `FileNotFoundException` first allows a specific message like "File not found: config.xml," while the `IOException` handler provides a general I/O error message.

**References Links**

- The catch Blocks: Catching More Than One Type of Exception – https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html
- JLS §14.20: The try statement (unreachable catch clauses) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20

### 2.2 Multi-Catch

#### Definitions

- **Core Definition:** Handling multiple unrelated exception types in a single `catch` block using the pipe operator (`catch (IOException | SQLException e)`). The exception variable in a multi-catch is implicitly final.
- **Technical Definition:** In Java SE 7 and later, a single `catch` block can handle more than one type of exception. The exception types are separated by the vertical bar (`|`). The catch parameter is implicitly final, meaning it cannot be reassigned within the catch block. The bytecode generated by a multi-catch block is smaller and more efficient than multiple separate catch blocks with duplicated code.
- **Beginner-Friendly Explanation:** If you have several different exceptions that you want to handle in exactly the same way, you can combine them into one catch block using the `|` symbol. This reduces code duplication and makes your intent clearer. However, you cannot change the exception variable to point to a different exception inside the block.

#### Purposes

- To reduce code duplication when multiple exception types require the same handling logic.
- To avoid catching an overly broad exception type (such as `Exception`) just to handle several unrelated types.
- To improve code readability and maintainability.

#### Syntax Structures and Rules

Complete general syntax:

```java
catch (ExceptionType1 | ExceptionType2 | ExceptionType3 parameterName) {
    // Shared handling logic
}
```

Component breakdown:

- `ExceptionType1`, `ExceptionType2`, etc.: the exception types to catch. Must not be related by subclassing (a multi-catch cannot include both a type and its subtype).
- `parameterName`: the variable holding the caught exception. It is implicitly final.
- The catch block handles all listed exception types identically.

Syntax rules: The exception types in a multi-catch must be disjoint (no type can be a subclass of another). The catch parameter is implicitly final; any attempt to assign to it is a compile-time error. The parameter's type is the least upper bound of the listed types.

Constraints and limitations: Multi-catch was introduced in Java 7. It cannot be used with a single exception type (that is a regular catch). The exception types cannot be related by inheritance.

#### Multiple Annotated Complete Code Examples and Their Expected Outputs

**Example 1: Multi-catch with IOException and SQLException**

```java
import java.io.IOException;
import java.sql.SQLException;

public class MultiCatchExample {
    public static void main(String[] args) {
        try {
            // Simulate an operation that may throw IOException or SQLException
            if (Math.random() > 0.5) {
                throw new IOException("I/O error");
            } else {
                throw new SQLException("Database error");
            }
        } catch (IOException | SQLException e) {
            System.out.println("Caught: " + e.getClass().getSimpleName() + " - " + e.getMessage());
        }
    }
}
```

Expected Output (may vary based on random value):

```
Caught: IOException - I/O error
```

or

```
Caught: SQLException - Database error
```

**Why the code produces this result:** The multi-catch block handles both `IOException` and `SQLException` with the same logic. Whichever exception is thrown, the same catch block executes.

**Example 2: Multi-catch with implicit final (assignment error)**

```java
public class MultiCatchFinalExample {
    public static void main(String[] args) {
        try {
            throw new java.io.IOException("Test");
        } catch (java.io.IOException | RuntimeException e) {
            // e = new RuntimeException("New"); // COMPILE-TIME ERROR
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Caught: Test
```

**Why the assignment is a compile-time error:** The catch parameter `e` is implicitly final in a multi-catch. Attempting to reassign it causes a compilation error: "The parameter e cannot be assigned".

**Real-World Cases with Explanation**

In a web service client, a multi-catch block handles both `ConnectException` and `SocketTimeoutException` with the same retry logic. In a data access layer, a multi-catch block handles `SQLException` and `DataAccessException` with the same logging and translation to a domain-specific exception.

**References Links**

- Catching Multiple Exception Types (Oracle) – https://docs.oracle.com/javase/8/docs/technotes/guides/language/catch-multiple.html
- JLS §14.20: The try statement (multi-catch) – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20
- DigitalOcean: Java Catch Multiple Exceptions – https://www.digitalocean.com/community/tutorials/java-catch-multiple-exceptions-rethrow-exception


## Summary Table of Exception Handling Keywords

| Keyword/Mechanism | Java Version | Purpose | Key Constraint |
|---|---|---|---|
| `try` | 1.0 | Encloses code that may throw exceptions | Must be followed by `catch`, `finally`, or resource specification |
| `catch` | 1.0 | Handles specific exception types | Must be ordered specific to general |
| `finally` | 1.0 | Guarantees cleanup execution | Avoid `return`, `break`, `continue`, `throw` inside |
| Multiple `catch` blocks | 1.0 | Handles different exceptions differently | Specific exceptions before general ones |
| Multi-catch (`|`) | 7 | Handles multiple exceptions in one block | Types must be disjoint; parameter implicitly final |
| try-with-resources | 7 | Automatic resource management | Resources must implement `AutoCloseable` |


## Version-Specific Notes

- Multi-catch (`catch (A | B e)`) was introduced in Java SE 7.
- Try-with-resources was introduced in Java SE 7 and is the preferred way to manage resources that implement `AutoCloseable`.
- The implicit finality of multi-catch parameters was introduced in Java SE 7. In a uni-catch clause, the exception parameter is never implicitly final but may be effectively final.
- The precise rethrow analysis (allowing more specific `throws` clauses when rethrowing a caught exception) was introduced in Java SE 7.


## References

- Catching and Handling Exceptions (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/handling.html
- The try Block (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/try.html
- The catch Blocks (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/catch.html
- The finally Block (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/finally.html
- Catching Multiple Exception Types and Rethrowing Exceptions with Improved Type Checking (Oracle) – https://docs.oracle.com/javase/8/docs/technotes/guides/language/catch-multiple.html
- JLS §14.20: The try statement – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20
- JLS §14.20.2: Execution of try-finally and try-catch-finally – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.2
- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3
- The Catch or Specify Requirement (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/catchOrDeclare.html
- SEI CERT ERR04-J: Do not complete abruptly from a finally block – https://wiki.sei.cmu.edu/confluence/display/java/ERR04-J.+Do+not+complete+abruptly+from+a+finally+block
- SEI CERT ERR54-J: Use a try-with-resources statement to safely handle closeable resources – https://wiki.sei.cmu.edu/confluence/display/java/ERR54-J.+Use+a+try-with-resources+statement+to+safely+handle+closeable+resources
- DigitalOcean: Java Catch Multiple Exceptions, Rethrow Exception – https://www.digitalocean.com/community/tutorials/java-catch-multiple-exceptions-rethrow-exception
- Stack Overflow: Why is the catch parameter implicitly final? – https://stackoverflow.com/questions/7737257/why-is-the-catch-parameter-implicitly-final
- Stack Overflow: Java multiple catch blocks unreachable code order – https://stackoverflow.com/questions/3568630/java-multiple-catch-blocks-unreachable-code-order