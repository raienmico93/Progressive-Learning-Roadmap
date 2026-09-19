# Java Try-with-Resources: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** The try-with-resources statement is a `try` statement that declares one or more resources, where a resource is an object that must be closed after the program is finished with it, ensuring that each resource is closed at the end of the statement.

**Technical Definition:** A try-with-resources statement (JLS §14.20.3) parameterizes a `try` block with one or more local variables (resources) that are initialized before execution of the `try` block and closed automatically in reverse order of initialization after execution of the `try` block. Any object that implements `java.lang.AutoCloseable`, which includes all objects that implement `java.io.Closeable`, can be used as a resource. The statement was introduced in JDK 1.7 to simplify correct resource management and avoid problems that arise when closing resources with an ordinary `try-catch-finally` block.

**Beginner-Friendly Explanation:** When you open a file, a network connection, or a database connection, you must remember to close it when you are done. If you forget, you leak resources and may crash your program. Before Java 7, you had to write a `finally` block to close everything manually. The try-with-resources statement does this automatically: you declare your resources in parentheses after `try`, and Java closes them for you when the block finishes—no matter what happens inside.

### Key Characteristics

Try-with-resources is characterized by **automatic resource management** (resources are closed without explicit `finally` blocks), **reverse-order closing** (resources are closed in the opposite order from which they were initialized, following the Last-In-First-Out principle), **exception suppression** (if multiple exceptions occur, the primary exception is propagated and subsequent exceptions from closing resources are suppressed and available via `Throwable.getSuppressed()`), and **compile-time safety** (the compiler generates the closing logic, eliminating the risk of forgetting to close a resource). Since Java 9, the statement also supports effectively final variables, allowing existing resource references to be used directly without declaring new variables in the try header.

### Prerequisites

Readers should be familiar with Java's exception hierarchy (`Throwable`, `Exception`, `RuntimeException`), basic `try`-`catch`-`finally` syntax, and the concept of resource management (opening and closing files, connections, or streams). A basic understanding of interfaces and the `implements` keyword is assumed.

### Related Programming Areas with Explanation

Try-with-resources is related to **resource management** (deterministic cleanup of finite system resources such as file handles and database connections), **exception handling** (interaction with `catch` and `finally` blocks, exception suppression), **API design** (designing classes that implement `AutoCloseable` to participate in automatic resource management), and **software reliability engineering** (preventing resource leaks and ensuring deterministic cleanup even under exceptional conditions).


## 1. AutoCloseable vs. Closeable Interfaces

**Core Definition:** `AutoCloseable` is the general-purpose interface that enables try-with-resources; `Closeable` is a specialized sub-interface for I/O resources that extends `AutoCloseable`.

**Technical Definition:** `java.lang.AutoCloseable` is an interface introduced in Java 7 that declares a single method `void close() throws Exception`. `java.io.Closeable` is an older interface (predating Java 7) that extends `AutoCloseable` and overrides `close()` to declare `throws IOException` instead of `throws Exception`. `Closeable` is specifically dedicated to I/O streams, while `AutoCloseable` is more general and can be used for things like transactions or memory management.

**Beginner-Friendly Explanation:** `AutoCloseable` is like a universal "I can be closed" label. Any class that implements it can be used in try-with-resources. `Closeable` is a more specific version for input/output streams (like files and network connections), and its `close()` method can only throw `IOException`, which is a subtype of the more general `Exception` thrown by `AutoCloseable.close()`.

### 1.1 AutoCloseable Interface

**Definitions**

- **Core Definition:** The root interface for objects that hold resources until they are closed.
- **Technical Definition:** `public interface AutoCloseable { void close() throws Exception; }` — the `close()` method is invoked automatically at the end of a try-with-resources block. Unlike `Closeable.close()`, the `close()` method of `AutoCloseable` is not required to be idempotent; calling it more than once may have visible side effects.
- **Beginner-Friendly Explanation:** `AutoCloseable` is the minimum requirement for a class to participate in try-with-resources. If your class holds any resource that needs cleanup, implement this interface.

**Purposes**

- To enable a class to participate in automatic resource management.
- To provide a general mechanism for releasing resources that is not tied to I/O.
- To allow the try-with-resources statement to work with any resource type.

**Syntax Structures and Rules**

Complete general syntax:

```java
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        // Release resources
    }
}
```

Component breakdown:

- `implements AutoCloseable`: marks the class as a resource.
- `close()`: the method invoked automatically by try-with-resources.
- The `close()` method may throw `Exception` (or any subclass).

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Custom AutoCloseable resource**

```java
public class AutoCloseableExample {
    static class Resource implements AutoCloseable {
        private final String name;
        Resource(String name) {
            this.name = name;
            System.out.println("Opened: " + name);
        }
        @Override
        public void close() {
            System.out.println("Closed: " + name);
        }
    }

    public static void main(String[] args) {
        try (Resource r = new Resource("Database")) {
            System.out.println("Using: Database");
        }
    }
}
```

Expected Output:

```
Opened: Database
Using: Database
Closed: Database
```

**Why the code produces this result:** The `Resource` class implements `AutoCloseable`. When the try-with-resources block exits, the `close()` method is invoked automatically, printing "Closed: Database".

**References Links**

- AutoCloseable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/AutoCloseable.html
- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3

### 1.2 Closeable Interface

**Definitions**

- **Core Definition:** A specialized `AutoCloseable` for I/O resources whose `close()` method throws `IOException` and is required to be idempotent.
- **Technical Definition:** `public interface Closeable extends AutoCloseable { void close() throws IOException; }` — the `close()` method is required to have no effect if called more than once (idempotent). This interface predates Java 7 and was retrofitted to extend `AutoCloseable` when try-with-resources was introduced.
- **Beginner-Friendly Explanation:** `Closeable` is the interface you implement for files, streams, and other I/O resources. Its `close()` method can only throw `IOException`, and calling it multiple times is safe—it does nothing after the first call.

**Purposes**

- To provide a standard interface for closing I/O streams.
- To guarantee idempotent closing behavior for I/O resources.
- To allow I/O resources to participate in try-with-resources.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Using Closeable with a file stream**

```java
import java.io.*;

public class CloseableExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new StringReader("Hello, World!"))) {
            System.out.println(br.readLine());
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Hello, World!
```

**Why the code produces this result:** `BufferedReader` implements `Closeable`, so it can be used in try-with-resources. The `close()` method is called automatically, closing the underlying reader.

**References Links**

- Closeable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Closeable.html

### 1.3 Key Differences

| Aspect | AutoCloseable | Closeable |
|---|---|---|
| **Package** | `java.lang` | `java.io` |
| **Introduced** | Java 7 | Java 1.0 (retrofitted in Java 7) |
| **Exception thrown** | `Exception` | `IOException` |
| **Idempotency required** | No | Yes |
| **Intended use** | Any resource | I/O resources only |
| **try-with-resources support** | Directly | Indirectly (via AutoCloseable) |

**References Links**

- AutoCloseable vs Closeable (Stack Overflow) – https://stackoverflow.com/questions/13141302/implements-closeable-or-implements-autocloseable
- Java源码详解:深入 Java I/O 核心之AutoCloseable 源码全景深度解析 – https://cloud.tencent.cn


## 2. Resource Management and Automatic Closing Order

**Core Definition:** Resources declared in a try-with-resources statement are initialized in left-to-right order and closed in reverse order (Last-In-First-Out) after the `try` block completes, whether normally or abruptly.

**Technical Definition:** The JLS specifies: "Resources are initialized in left-to-right order. If a resource fails to initialize, then all resources initialized so far by the try-with-resources statement are closed. ... Resources are closed in the reverse order from that in which they were initialized. A resource is closed only if it initialized to a non-null value". This reverse-order closing mirrors the nesting of `try-finally` blocks that would be required to achieve the same behavior manually.

**Beginner-Friendly Explanation:** If you open Resource A first, then Resource B, then Resource C, they will be closed in the opposite order: C, then B, then A. This is like unwinding a stack—the last thing you opened is the first thing you close. This order ensures that dependent resources are closed before the resources they depend on.

### 2.1 Declaration and Initialization Order

**Definitions**

- **Core Definition:** Resources are declared in a parenthesized list after the `try` keyword and initialized from left to right.
- **Technical Definition:** Multiple resources can be declared in a single try-with-resources statement, separated by semicolons: `try (R1 r1 = ...; R2 r2 = ...) { ... }`. The initializers are executed in left-to-right order. If any initializer throws an exception, the resources already initialized are closed in reverse order before the exception propagates.
- **Beginner-Friendly Explanation:** You can declare more than one resource in the `try` parentheses, separated by semicolons. They are opened in the order you write them.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Multiple resources with reverse-order closing**

```java
public class ReverseOrderExample {
    static class Resource implements AutoCloseable {
        private final String name;
        Resource(String name) {
            this.name = name;
            System.out.println("Opened: " + name);
        }
        @Override
        public void close() {
            System.out.println("Closed: " + name);
        }
    }

    public static void main(String[] args) {
        try (Resource r1 = new Resource("First");
             Resource r2 = new Resource("Second");
             Resource r3 = new Resource("Third")) {
            System.out.println("Inside try block");
        }
    }
}
```

Expected Output:

```
Opened: First
Opened: Second
Opened: Third
Inside try block
Closed: Third
Closed: Second
Closed: First
```

**Why the code produces this result:** Resources are initialized left-to-right (First, Second, Third), and closed in reverse order (Third, Second, First).

**Example 2: Resource initialization failure**

```java
public class InitFailureExample {
    static class Resource implements AutoCloseable {
        private final String name;
        Resource(String name) {
            this.name = name;
            System.out.println("Opened: " + name);
        }
        @Override
        public void close() {
            System.out.println("Closed: " + name);
        }
    }

    static class FailingResource extends Resource {
        FailingResource(String name) {
            super(name);
            throw new RuntimeException("Failed to open: " + name);
        }
    }

    public static void main(String[] args) {
        try {
            try (Resource r1 = new Resource("R1");
                 Resource r2 = new FailingResource("R2");
                 Resource r3 = new Resource("R3")) {
                System.out.println("Inside try");
            }
        } catch (RuntimeException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Opened: R1
Opened: R2
Closed: R1
Caught: Failed to open: R2
```

**Why the code produces this result:** R1 initializes successfully. R2's constructor throws an exception. R1 is closed (reverse order of initialized resources). R3 is never initialized. The exception propagates to the `catch` block.

**References Links**

- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3

### 2.2 Closing Order with catch and finally

**Definitions**

- **Core Definition:** Resources are closed before any `catch` or `finally` blocks associated with the try-with-resources statement execute.
- **Technical Definition:** The try-with-resources statement is equivalent to a nested `try-finally` structure where the `finally` block closes the resources. This means all resources are closed before the `catch` clauses and any explicit `finally` block are executed.
- **Beginner-Friendly Explanation:** If you have a `catch` or `finally` block after your try-with-resources, the resources are closed before those blocks run.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Resource closing vs. finally block**

```java
public class ClosingVsFinallyExample {
    static class Resource implements AutoCloseable {
        @Override
        public void close() {
            System.out.println("Resource closed");
        }
    }

    public static void main(String[] args) {
        try (Resource r = new Resource()) {
            System.out.println("Inside try");
        } catch (Exception e) {
            System.out.println("Inside catch");
        } finally {
            System.out.println("Inside finally");
        }
    }
}
```

Expected Output:

```
Inside try
Resource closed
Inside finally
```

**Why the code produces this result:** The resource is closed after the `try` block but before the `catch` and `finally` blocks. There is no exception, so the `catch` block is skipped, and the `finally` block runs after the resource is closed.

**References Links**

- Stack Overflow: try-with-resources close order – https://stackoverflow.com/revisions/41461cc7-8b22-404e-b7ec-880775b36ba7/view-source
- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3


## 3. Suppressed Exceptions

**Core Definition:** A suppressed exception is an exception that is thrown during the closing of a resource but is not the primary exception propagated; it is attached to the primary exception via `Throwable.addSuppressed()` and retrievable via `Throwable.getSuppressed()`.

**Technical Definition:** When multiple exceptions are thrown in a try-with-resources statement, the exception thrown from the `try` block is the primary exception that is propagated. Any exceptions thrown during the closing of resources are **suppressed**—they are attached to the primary exception via `addSuppressed()` rather than replacing it. If no exception is thrown from the `try` block, the first exception thrown during closing becomes the primary exception, and subsequent closing exceptions are suppressed. The `Throwable` class was enhanced in Java 7 with `addSuppressed(Throwable)`, `getSuppressed()`, and a constructor that supports disabling suppression.

**Beginner-Friendly Explanation:** If your `try` block throws an exception, and then closing a resource also throws an exception, you want to know about both. But Java can only throw one exception at a time. So it throws the first one (from the `try` block) and tucks the second one (from closing) inside it as a "suppressed" exception. You can retrieve the suppressed exception by calling `getSuppressed()`.

### 3.1 Throwable.getSuppressed()

**Definitions**

- **Core Definition:** Returns an array of all exceptions that were suppressed in order to deliver the primary exception.
- **Technical Definition:** `public final Throwable[] getSuppressed()` returns an array containing all exceptions that were suppressed, typically by the try-with-resources statement, in order to deliver this exception. If no exceptions were suppressed or suppression is disabled, an empty array is returned. This method is thread-safe.
- **Beginner-Friendly Explanation:** `getSuppressed()` gives you the list of exceptions that were "hidden" behind the primary exception.

**Purposes**

- To retrieve diagnostic information about failures that occurred during resource cleanup.
- To ensure that no exception is silently lost.
- To provide a complete picture of what went wrong in a try-with-resources block.

**Syntax Structures and Rules**

Complete general syntax:

```java
Throwable[] suppressed = primaryException.getSuppressed();
```

Component breakdown:

- Return value: an array of `Throwable` objects (empty if no suppressed exceptions).
- The array contains exceptions in the order they were suppressed.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Suppressed exception from closing**

```java
public class SuppressedExample {
    static class FailingResource implements AutoCloseable {
        @Override
        public void close() {
            throw new RuntimeException("Exception during close");
        }
    }

    public static void main(String[] args) {
        try {
            try (FailingResource r = new FailingResource()) {
                throw new RuntimeException("Exception in try block");
            }
        } catch (RuntimeException e) {
            System.out.println("Primary: " + e.getMessage());
            for (Throwable suppressed : e.getSuppressed()) {
                System.out.println("Suppressed: " + suppressed.getMessage());
            }
        }
    }
}
```

Expected Output:

```
Primary: Exception in try block
Suppressed: Exception during close
```

**Why the code produces this result:** The `try` block throws a `RuntimeException`. When the resource is closed, its `close()` method also throws a `RuntimeException`. Since the try-block exception is primary, the closing exception is suppressed and attached to it.

**Example 2: No exception in try block, exception during close**

```java
public class NoPrimaryExample {
    static class FailingResource implements AutoCloseable {
        @Override
        public void close() {
            throw new RuntimeException("Close failed");
        }
    }

    public static void main(String[] args) {
        try (FailingResource r = new FailingResource()) {
            System.out.println("Inside try");
        } catch (RuntimeException e) {
            System.out.println("Primary: " + e.getMessage());
            System.out.println("Suppressed count: " + e.getSuppressed().length);
        }
    }
}
```

Expected Output:

```
Inside try
Primary: Close failed
Suppressed count: 0
```

**Why the code produces this result:** Since no exception is thrown in the `try` block, the exception from `close()` becomes the primary exception (it is not suppressed because there is no prior exception to suppress it).

**References Links**

- Throwable.getSuppressed (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html#getSuppressed()
- SEI CERT ERR54-J: Use a try-with-resources statement – https://wiki.sei.cmu.edu/confluence/display/java/ERR54-J.+Use+a+try-with-resources+statement+to+safely+handle+closeable+resources

### 3.2 Suppression Order and Multiple Resources

**Definitions**

- **Core Definition:** When multiple resources are closed and multiple exceptions are thrown, the exceptions are suppressed in the order the resources are closed (reverse declaration order).
- **Technical Definition:** Each exception thrown during the closing of a resource is added to the primary exception's suppressed list. The order of suppressed exceptions corresponds to the order in which the `close()` methods are invoked (i.e., reverse of the declaration order). If an exception is thrown while closing a resource, the remaining resources are still closed, and their exceptions are also suppressed.
- **Beginner-Friendly Explanation:** If you have three resources and all three fail during closing, the primary exception will have three suppressed exceptions attached, in the order the resources were closed.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Multiple suppressed exceptions**

```java
public class MultipleSuppressedExample {
    static class FailingResource implements AutoCloseable {
        private final String name;
        FailingResource(String name) { this.name = name; }
        @Override
        public void close() {
            throw new RuntimeException("Close failed: " + name);
        }
    }

    public static void main(String[] args) {
        try {
            try (FailingResource r1 = new FailingResource("R1");
                 FailingResource r2 = new FailingResource("R2");
                 FailingResource r3 = new FailingResource("R3")) {
                throw new RuntimeException("Try block failure");
            }
        } catch (RuntimeException e) {
            System.out.println("Primary: " + e.getMessage());
            for (Throwable s : e.getSuppressed()) {
                System.out.println("Suppressed: " + s.getMessage());
            }
        }
    }
}
```

Expected Output:

```
Primary: Try block failure
Suppressed: Close failed: R3
Suppressed: Close failed: R2
Suppressed: Close failed: R1
```

**Why the code produces this result:** Resources are closed in reverse order (R3, R2, R1). Each `close()` throws an exception, and all are suppressed under the primary exception from the `try` block.

**References Links**

- Stack Overflow: try-with-resources close order – https://stackoverflow.com/revisions/41461cc7-8b22-404e-b7ec-880775b36ba7/view-source
- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3


## 4. Java 9+ Enhancement: Effectively Final Resources

**Core Definition:** Since Java 9, a resource in a try-with-resources statement can be a reference to a `final` or **effectively final** variable that is already declared, eliminating the need to declare a new variable in the try header.

**Technical Definition:** The Java SE 9 language update states: "If you already have a resource as a final or effectively final variable, you can use that variable in a try-with-resources statement without declaring a new variable in the try-with-resources statement". An effectively final variable is one whose value is never changed after it is initialized. This enhancement makes the try-with-resources statement more concise when the resource variable needs to be used outside the try block or is already available as a parameter or field.

**Beginner-Friendly Explanation:** Before Java 9, you had to write `try (BufferedReader br = new BufferedReader(...))` even if you already had a variable named `br` that you wanted to use. Since Java 9, you can just write `try (br)` if `br` is already declared and never reassigned.

### 4.1 Effectively Final Variable Syntax

**Definitions**

- **Core Definition:** The resource is a reference to an existing variable that is final or effectively final, rather than a new declaration.
- **Technical Definition:** The syntax is `try (existingVariable) { ... }` where `existingVariable` is a final or effectively final local variable or a field. Multiple resources can be specified: `try (var1; var2) { ... }`. The variable must be effectively final, meaning its value is never reassigned after initialization.
- **Beginner-Friendly Explanation:** You can use a variable you already declared in the try-with-resources parentheses, as long as you never change it after assigning it.

**Purposes**

- To reduce redundancy when the resource variable is already available.
- To allow resources to be used outside the try block (e.g., for logging or further processing).
- To make try-with-resources more concise in methods that receive resources as parameters.

**Syntax Structures and Rules**

Complete general syntax:

```java
// Before Java 9: must declare new variable
try (Resource r = existingResource) { ... }

// Java 9+: can use existing variable directly
try (existingResource) { ... }
```

Component breakdown:

- `existingResource`: a final or effectively final variable.
- Multiple resources: `try (var1; var2) { ... }`.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Java 9+ effectively final resource**

```java
public class EffectivelyFinalExample {
    static class Resource implements AutoCloseable {
        private final String name;
        Resource(String name) { this.name = name; }
        @Override
        public void close() {
            System.out.println("Closing " + name);
        }
    }

    public static void main(String[] args) {
        // Resource declared outside the try block
        Resource resource = new Resource("MyResource");

        // Java 9+: use the existing variable directly
        try (resource) {
            System.out.println("Using " + resource.name);
        }
        // resource is still accessible after the try block (but already closed)
        System.out.println("After try block");
    }
}
```

Expected Output:

```
Using MyResource
Closing MyResource
After try block
```

**Why the code produces this result:** The `resource` variable is declared before the try block and is effectively final (never reassigned). Java 9+ allows it to be used directly as a resource without declaring a new variable.

**Example 2: Multiple effectively final resources**

```java
public class MultiEffectivelyFinalExample {
    static class Resource implements AutoCloseable {
        private final String name;
        Resource(String name) { this.name = name; }
        @Override
        public void close() {
            System.out.println("Closing " + name);
        }
    }

    public static void main(String[] args) {
        final Resource r1 = new Resource("R1");
        Resource r2 = new Resource("R2"); // effectively final

        try (r1; r2) {
            System.out.println("Inside try");
        }
    }
}
```

Expected Output:

```
Inside try
Closing R2
Closing R1
```

**Why the code produces this result:** Both `r1` (declared `final`) and `r2` (effectively final) are used directly. They are closed in reverse order.

**References Links**

- Java SE 9 Language Updates: More Concise try-with-resources – https://docs.oracle.com/javase/9/language/toc.htm
- More Concise try-with-resources Statement (OpenJDK) – https://mail.openjdk.org/pipermail/coin-dev/2011-April/003179.html
- Stack Overflow: Effectively final variables in try-with-resources – https://stackoverflow.com/questions/47177635/java-9-effectively-final-variables-in-try-with-resources


## Summary Table

| Feature | Java Version | Key Benefit | Key Constraint |
|---|---|---|---|
| try-with-resources | 7 | Automatic resource closing | Resource must implement `AutoCloseable` |
| `AutoCloseable` interface | 7 | General resource contract | `close()` throws `Exception`; not required to be idempotent |
| `Closeable` interface | 1.0 | I/O resource contract | `close()` throws `IOException`; must be idempotent |
| Reverse-order closing | 7 | Mirrors manual nesting | Fixed order; cannot be changed |
| Suppressed exceptions | 7 | No exception is lost | Primary exception propagated; others suppressed |
| `getSuppressed()` | 7 | Retrieve suppressed exceptions | Returns empty array if none |
| Effectively final resources | 9 | Concise syntax for existing variables | Variable must be final or effectively final |


## Version-Specific Notes

- Try-with-resources and the `AutoCloseable` interface were introduced in Java SE 7.
- The `Throwable.addSuppressed()`, `getSuppressed()`, and the suppression-disabling `Throwable` constructor were introduced in Java SE 7 as part of the library support for try-with-resources.
- The Java 9 enhancement allowing final and effectively final variables to be used as resources in try-with-resources was introduced in Java SE 9.
- `Closeable` predates Java 7 and was retrofitted to extend `AutoCloseable` when try-with-resources was introduced.


## References

- The try-with-resources Statement (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html
- JLS §14.20.3: try-with-resources – https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.20.3
- AutoCloseable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/AutoCloseable.html
- Closeable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Closeable.html
- Throwable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html
- SEI CERT ERR54-J: Use a try-with-resources statement to safely handle closeable resources – https://wiki.sei.cmu.edu/confluence/display/java/ERR54-J.+Use+a+try-with-resources+statement+to+safely+handle+closeable+resources
- Java SE 9 Language Updates: More Concise try-with-resources – https://docs.oracle.com/javase/9/language/toc.htm
- New protocol for disabling exception suppression (OpenJDK) – https://mail.openjdk.org/pipermail/coin-dev/2011-April/003179.html
- More concise try-with-resources statement now in JDK 9 (OpenJDK) – https://mail.openjdk.org/pipermail/coin-dev/2011-April/003179.html
- Stack Overflow: try-with-resources close order – https://stackoverflow.com/revisions/41461cc7-8b22-404e-b7ec-880775b36ba7/view-source
- Stack Overflow: implements Closeable or implements AutoCloseable – https://stackoverflow.com/questions/13141302/implements-closeable-or-implements-autocloseable
- Stack Overflow: Java 9 effectively final variables in try-with-resources – https://stackoverflow.com/questions/47177635/java-9-effectively-final-variables-in-try-with-resources
- Java源码详解:深入 Java I/O 核心之AutoCloseable 源码全景深度解析 – https://cloud.tencent.cn
- Throwable.GetSuppressed (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/api/java.lang.throwable.getsuppressed