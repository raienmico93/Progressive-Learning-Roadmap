# Java Creating Exceptions: A Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition:** Creating exceptions in Java is the practice of defining custom exception classes that extend either `Exception` (for checked exceptions) or `RuntimeException` (for unchecked exceptions), enabling developers to represent domain-specific error conditions with meaningful names, contextual data, and appropriate recovery semantics.

**Technical Definition:** A custom exception class is a user-defined subclass of `Throwable` (typically `Exception` or `RuntimeException`) that provides constructors matching the standard `Throwable` constructor signatures: a no-argument constructor, a message-only constructor, a cause-only constructor, and a message-and-cause constructor. When extending `Exception`, the class is checked and subject to the Catch or Specify Requirement (JLS §11.2). When extending `RuntimeException`, the class is unchecked and exempt from this requirement. Custom exceptions may declare additional instance fields to carry contextual error information, and must declare a `serialVersionUID` if they are intended to be serializable.

**Beginner-Friendly Explanation:** Sometimes the built-in exceptions like `NullPointerException` or `IOException` do not describe your specific problem well enough. You might want an exception called `InsufficientFundsException` or `InvalidCustomerAgeException`. Creating your own exception class lets you give a meaningful name to your error, attach useful information (like the account number that had insufficient funds), and decide whether callers are forced to handle it (checked) or not (unchecked).

### Key Characteristics

- Custom exceptions are characterized by their **Domain specificity** (they name the exact failure mode rather than using a generic type).
- **Hierarchical organization** (related exceptions can share a common superclass for broad catching).
- **Constructual consistency** (they mirror the standard `Throwable` constructor signatures), **contextual richness** (they can carry additional fields with diagnostic data).
- **Serialization compatibility** (they declare `serialVersionUID` when serializable). 
- The choice between checked and unchecked is a design decision: checked exceptions force callers to acknowledge and handle the failure, while unchecked exceptions propagate freely and are suitable for programming errors or unrecoverable conditions.

### Prerequisites

- Readers should be familiar with Java's exception hierarchy (`Throwable`, `Exception`, `Error`, `RuntimeException`). 
- The difference between checked and unchecked exceptions, and basic `try`-`catch`-`finally` syntax. 
- A basic understanding of constructors, inheritance, and the `super` keyword is assumed. Knowledge of `serialVersionUID` is helpful for the serialization section.

### Related Programming Areas with Explanation

- Custom exceptions are related to **API design** (they form part of a library's public contract. 
- Communicating what can go wrong), **domain-driven design** (they express business rules as first-class types).
- **Error handling architecture** (they enable layered error translation).
- **Testing** (they allow precise assertions on failure modes).
- **Software reliability engineering** (they provide actionable diagnostic information for production incidents).


## 1. Custom Checked Exceptions (Extending Exception)

**Core Definition:** A checked exception is a custom exception class that extends `Exception` but not `RuntimeException`; the compiler forces callers to either catch it or declare it in their `throws` clause.

**Technical Definition:** A class `MyException extends Exception` is a checked exception. Any method that can throw such an exception must declare it via a `throws` clause or catch it within a `try`-`catch` block. The compiler enforces this at compile time (JLS §11.2.3). Checked exceptions are appropriate when the caller can reasonably be expected to recover from the exceptional condition, such as a file not being found, a network timeout, or invalid user input.

**Beginner-Friendly Explanation:** A checked exception is like a warning label: “This method might fail in this specific way, and you must deal with it.” The compiler will not let you ignore it. You use checked exceptions when the caller has a realistic chance of doing something useful about the problem.

### 1.1 When to Use Checked Exceptions

**Definitions**

- **Core Definition:** Checked exceptions are appropriate when the failure is recoverable and the caller should be forced to acknowledge it.
- **Technical Definition:** The Java Language Specification states that checked exceptions represent conditions from which the caller can reasonably be expected to recover. The Catch or Specify Requirement forces the caller to either handle the exception or declare it. Oracle's guidance: “If a client can reasonably be expected to recover from an exception, make it a checked exception.”
- **Beginner-Friendly Explanation:** Use a checked exception when the error is predictable and the caller might know what to do about it—for example, “the file was not found, so I will use a default configuration.”

**Purposes**

- To force the caller to acknowledge and handle a recoverable failure mode.
- To make the method's contract explicit about what can go wrong.
- To enable the compiler to verify error handling.

**Syntax Structures and Rules**

Complete general syntax:

```java
public class MyCheckedException extends Exception {
    public MyCheckedException() { super(); }
    public MyCheckedException(String message) { super(message); }
    public MyCheckedException(Throwable cause) { super(cause); }
    public MyCheckedException(String message, Throwable cause) { super(message, cause); }
}
```

Component breakdown:

- `extends Exception`: makes this a checked exception.
- Four constructors mirror the standard `Throwable` signatures.
- Each constructor delegates to the corresponding `super` constructor.

Syntax rules: A checked exception must be declared in the `throws` clause of any method that throws it, or caught within that method.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: A checked exception for configuration errors**

```java
// Step 1: Define the checked exception
public class ConfigNotFoundException extends Exception {
    public ConfigNotFoundException() { super(); }
    public ConfigNotFoundException(String message) { super(message); }
    public ConfigNotFoundException(Throwable cause) { super(cause); }
    public ConfigNotFoundException(String message, Throwable cause) { super(message, cause); }
}

// Step 2: Use it in a method that declares throws
public class ConfigLoader {
    public static String loadConfig(String name) throws ConfigNotFoundException {
        if (name == null || name.isEmpty()) {
            throw new ConfigNotFoundException("Configuration name cannot be empty");
        }
        return "config-value";
    }

    public static void main(String[] args) {
        try {
            String value = loadConfig("");
        } catch (ConfigNotFoundException e) {
            System.out.println("Handled: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Handled: Configuration name cannot be empty
```

**Why the code produces this result:** `loadConfig` declares `throws ConfigNotFoundException`. The caller (`main`) is forced by the compiler to either catch it or declare it. Since `main` catches it, the exception is handled gracefully.

**Real-World Cases with Explanation**

In a banking application, `InsufficientFundsException extends Exception` is thrown when a withdrawal exceeds the balance. The caller (the ATM service) can catch it and display a specific message to the user. In a file-processing library, `FileFormatException extends Exception` is thrown when a file does not conform to the expected format; the caller can decide whether to skip the file, log an error, or attempt recovery.

**References Links**

- Creating Exception Classes (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html
- The Catch or Specify Requirement (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/catchOrDeclare.html
- JLS §11.2: Compile-Time Checking of Exceptions – https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html#jls-11.2


## 2. Custom Unchecked Exceptions (Extending RuntimeException)

**Core Definition:** An unchecked exception is a custom exception class that extends `RuntimeException`; the compiler does not force callers to catch or declare it.

**Technical Definition:** A class `MyException extends RuntimeException` is an unchecked exception. Unchecked exceptions are exempt from the Catch or Specify Requirement. They are appropriate for programming errors (such as invalid arguments, null values, or illegal state) and for conditions from which the caller cannot reasonably be expected to recover. Oracle's guidance: “If a client cannot do anything to recover from the exception, make it an unchecked exception.”

**Beginner-Friendly Explanation:** An unchecked exception is like a runtime surprise: the compiler does not force you to handle it, but it can still occur. You use unchecked exceptions for things that are usually programming mistakes—like passing `null` where a value is required, or calling a method in the wrong state.

### 2.1 When to Use Unchecked Exceptions

**Definitions**

- **Core Definition:** Unchecked exceptions are appropriate when the failure represents a programming error or an unrecoverable condition.
- **Technical Definition:** The Java documentation states that unchecked exceptions “represent the problems that arise from a programming error, such as a null pointer exception, an array index out of bounds, or a class cast exception.” The Tuscany Java Coding Guidelines recommend: “Unchecked exceptions should be used when an error condition is not recoverable.”
- **Beginner-Friendly Explanation:** Use an unchecked exception when the caller cannot do anything useful about the error—it is likely a bug in the code, not a condition the caller can handle.

**Purposes**

- To signal programming errors such as invalid arguments or illegal state.
- To avoid cluttering method signatures with exceptions the caller cannot handle.
- To propagate failures to a global error handler or the JVM.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: An unchecked exception for invalid arguments**

```java
// Step 1: Define the unchecked exception
public class InvalidAgeException extends RuntimeException {
    public InvalidAgeException() { super(); }
    public InvalidAgeException(String message) { super(message); }
    public InvalidAgeException(Throwable cause) { super(cause); }
    public InvalidAgeException(String message, Throwable cause) { super(message, cause); }
}

// Step 2: Use it without declaring throws
public class UserRegistration {
    public static void register(String name, int age) {
        if (age < 0 || age > 150) {
            throw new InvalidAgeException("Age must be between 0 and 150: " + age);
        }
        System.out.println("Registered " + name + " with age " + age);
    }

    public static void main(String[] args) {
        try {
            register("Alice", 200);
        } catch (InvalidAgeException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

Expected Output:

```
Caught: Age must be between 0 and 150: 200
```

**Why the code produces this result:** `InvalidAgeException` extends `RuntimeException`, so it is unchecked. The `register` method does not need a `throws` clause. The caller may optionally catch it, but is not forced to.

**Real-World Cases with Explanation**

In a JSON parser, `InvalidJsonException extends RuntimeException` is thrown when the input is malformed. The library does not force callers to handle it because malformed JSON is typically a programming error or data corruption issue. In a Spring application, `IllegalArgumentException` (a standard unchecked exception) is often thrown for invalid method parameters, and Spring's `@Valid` annotation handles them at the framework level.

**References Links**

- Unchecked Exceptions — The Controversy (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html
- Tuscany Java Coding Guidelines: Checked vs. unchecked exceptions – https://svn.apache.org/repos/asf/tuscany/java/sca/trunk/CodingGuidelines.txt
- SEI CERT ERR51-J: Prefer user-defined exceptions over more general exception types – https://wiki.sei.cmu.edu/confluence/display/java/ERR51-J.+Prefer+user-defined+exceptions+over+more+general+exception+types


## 3. Best Practices for Custom Exception Constructors

**Core Definition:** The standard constructor pattern for a custom exception mirrors the four `Throwable` constructors: no-argument, message-only, cause-only, and message-and-cause.

**Technical Definition:** Josh Bloch's Effective Java (Item 75) recommends that custom exceptions should provide the same constructors as the base `Exception` class. The four constructors delegate to the corresponding `super` constructors. The no-argument constructor is optional but recommended for serialization frameworks. Each constructor should call the appropriate `super` constructor with the matching parameters.

**Beginner-Friendly Explanation:** When you create a custom exception, it is polite to give it the same set of constructors that the standard `Exception` class has. That way, users of your exception can create it with a message, with a cause, with both, or with neither—just like they can with standard exceptions.

### 3.1 The Four Standard Constructors

**Definitions**

- **Core Definition:** A well-designed custom exception provides four constructors that mirror the standard `Throwable` signatures.
- **Technical Definition:** The four constructors are: `MyException()` (no arguments), `MyException(String message)` (message only), `MyException(Throwable cause)` (cause only), and `MyException(String message, Throwable cause)` (message and cause). Each delegates to the corresponding `super` constructor. The cause-only constructor passes the cause to `super(cause)`, which sets the detail message to `cause.toString()` if the cause is non-null.

**Purposes**

- To provide a consistent API that mirrors the standard exception classes.
- To allow callers to supply as much or as little diagnostic information as needed.
- To support exception chaining with a cause.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Complete constructor set**

```java
public class PaymentException extends Exception {
    private static final long serialVersionUID = 1L;

    public PaymentException() {
        super();
    }

    public PaymentException(String message) {
        super(message);
    }

    public PaymentException(Throwable cause) {
        super(cause);
    }

    public PaymentException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**Example 2: Using all four constructors**

```java
public class PaymentExceptionDemo {
    public static void main(String[] args) {
        // No-arg constructor
        PaymentException e1 = new PaymentException();
        System.out.println("No-arg: " + e1.getMessage()); // null

        // Message-only constructor
        PaymentException e2 = new PaymentException("Payment declined");
        System.out.println("Message: " + e2.getMessage()); // Payment declined

        // Cause-only constructor
        PaymentException e3 = new PaymentException(new RuntimeException("Timeout"));
        System.out.println("Cause: " + e3.getCause().getMessage()); // Timeout

        // Message-and-cause constructor
        PaymentException e4 = new PaymentException("Payment declined",
                new RuntimeException("Insufficient funds"));
        System.out.println("Full: " + e4.getMessage() + " / "
                + e4.getCause().getMessage());
    }
}
```

Expected Output:

```
No-arg: null
Message: Payment declined
Cause: Timeout
Full: Payment declined / Insufficient funds
```

**Why the code produces this result:** Each constructor delegates to the corresponding `super` constructor. The no-arg constructor produces `null` message. The cause-only constructor sets the cause, and `getMessage()` returns `cause.toString()` (which for `RuntimeException` includes the message “Timeout”). The message-and-cause constructor sets both independently.

**Real-World Cases with Explanation**

In a persistence framework, a `DataAccessException` provides all four constructors so that DAO implementations can wrap `SQLException` causes with domain-specific messages. In a service layer, a `ServiceException` provides all four constructors to support different levels of diagnostic detail.

**References Links**

- Creating Exception Classes (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html
- Throwable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html
- Lombok @StandardException (Baeldung) – https://www.baeldung.com/lombok-standardexception-annotation

### 3.2 serialVersionUID

**Definitions**

- **Core Definition:** A unique identifier for a serializable class used during deserialization to verify that the sender and receiver of a serialized object have loaded classes for that object that are compatible with respect to serialization.
- **Technical Definition:** `serialVersionUID` is a `static final long` field. If a serializable class does not explicitly declare a `serialVersionUID`, the serialization runtime calculates a default value based on various aspects of the class. Oracle strongly recommends that all serializable classes explicitly declare `serialVersionUID` values because the default process is highly sensitive to compiler implementation details, which can cause unexpected `InvalidClassException` during deserialization.
- **Beginner-Friendly Explanation:** If you plan to send your exception over a network or save it to a file, you should give it a version number. This ensures that when the exception is read back, Java can verify that the class definition has not changed in an incompatible way.

**Syntax Structures and Rules**

Complete general syntax:

```java
private static final long serialVersionUID = 1L;
```

Component breakdown:

- Must be `static`, `final`, and of type `long`.
- The value is arbitrary but should be changed when the class definition changes in an incompatible way.
- If serialization is not required, this field may be omitted.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Exception with serialVersionUID**

```java
import java.io.*;

public class SerializableException extends Exception implements Serializable {
    private static final long serialVersionUID = 1L;

    public SerializableException(String message) {
        super(message);
    }

    public static void main(String[] args) throws Exception {
        SerializableException original = new SerializableException("Test");

        // Serialize
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(original);
        oos.close();

        // Deserialize
        ObjectInputStream ois = new ObjectInputStream(
                new ByteArrayInputStream(baos.toByteArray()));
        SerializableException restored = (SerializableException) ois.readObject();
        ois.close();

        System.out.println("Restored: " + restored.getMessage());
    }
}
```

Expected Output:

```
Restored: Test
```

**Real-World Cases with Explanation**

In an RPC framework, exceptions are serialized and sent across the network. Declaring `serialVersionUID` ensures that both client and server agree on the exception class version. In a distributed system using Java serialization, omitting `serialVersionUID` can cause `InvalidClassException` after a minor code change.

**References Links**

- Serializable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Serializable.html
- Java Object Serialization Specification – https://docs.oracle.com/en/java/javase/21/docs/specs/serialization/index.html

### 3.3 Naming Conventions

**Definitions**

- **Core Definition:** Custom exception classes should be named with a descriptive noun followed by the suffix “Exception.”
- **Technical Definition:** Oracle's Java Tutorials recommend: “For readable code, it's good practice to append the string Exception to the names of all classes that inherit (directly or indirectly) from the Exception class.” Standard Java naming conventions apply: class names use CamelCase with an initial capital letter.
- **Beginner-Friendly Explanation:** Name your exception after the problem it represents, and end the name with “Exception.” For example, `InsufficientFundsException`, not `InsufficientFundsError` or `FundsProblem`.

**Syntax Structures and Rules**

```java
public class InsufficientFundsException extends Exception { ... }
public class InvalidCustomerAgeException extends RuntimeException { ... }
```

**References Links**

- Creating Exception Classes (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html
- Java Code Conventions: Naming Conventions – https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html


## 4. Adding Contextual Error Information and Custom State/Fields

**Core Definition:** Adding contextual information to a custom exception means declaring additional instance fields that carry diagnostic data (such as IDs, values, or timestamps) beyond the standard message and cause.

**Technical Definition:** A custom exception may declare instance fields for domain-specific contextual data. These fields are initialized in the exception's constructors and exposed via getter methods. Because the additional fields cannot be passed to the `superclass` constructors, the exception class manages them itself. The `getMessage()` method may be overridden to include the contextual data in the formatted message, or the fields may be exposed separately for programmatic access.

**Beginner-Friendly Explanation:** Sometimes a simple message like “Payment failed” is not enough. You might want to know the account number, the amount, and the timestamp of the failure. You can add these as fields to your exception class so that whoever catches it can read them and decide what to do.

### 4.1 Adding Custom Fields

**Definitions**

- **Core Definition:** Instance fields that store domain-specific data about the error condition.
- **Technical Definition:** Fields are declared as `private final` (or at least `private`) and initialized in the constructor. Getter methods expose the values. Because `Throwable` does not provide a mechanism to pass arbitrary fields to the superclass, the custom exception class is responsible for managing them. The `getMessage()` method may be overridden to incorporate these fields into the message.
- **Beginner-Friendly Explanation:** You add fields like `accountNumber`, `amount`, or `timestamp` to your exception class. When you throw the exception, you fill in these fields. When someone catches it, they can call getter methods to read the values.

**Purposes**

- To provide diagnostic information that is not captured by the message alone.
- To enable programmatic recovery based on the contextual data.
- To support logging and monitoring systems that need structured error data.

**Syntax Structures and Rules**

Complete general syntax:

```java
public class InsufficientFundsException extends Exception {
    private final String accountNumber;
    private final double requestedAmount;
    private final double availableBalance;

    public InsufficientFundsException(String accountNumber,
                                      double requestedAmount,
                                      double availableBalance) {
        super("Insufficient funds in account " + accountNumber
                + ": requested " + requestedAmount
                + ", available " + availableBalance);
        this.accountNumber = accountNumber;
        this.requestedAmount = requestedAmount;
        this.availableBalance = availableBalance;
    }

    public String getAccountNumber() { return accountNumber; }
    public double getRequestedAmount() { return requestedAmount; }
    public double getAvailableBalance() { return availableBalance; }
}
```

Component breakdown:

- Fields are `private final` for immutability.
- The `super` constructor is called with a formatted message that includes the contextual data.
- The fields are assigned after the `super` call.
- Getter methods expose the fields.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: Exception with multiple context fields**

```java
public class InsufficientFundsException extends Exception {
    private final String accountNumber;
    private final double requestedAmount;
    private final double availableBalance;

    public InsufficientFundsException(String accountNumber,
                                      double requestedAmount,
                                      double availableBalance) {
        super("Insufficient funds in account " + accountNumber
                + ": requested " + requestedAmount
                + ", available " + availableBalance);
        this.accountNumber = accountNumber;
        this.requestedAmount = requestedAmount;
        this.availableBalance = availableBalance;
    }

    public String getAccountNumber() { return accountNumber; }
    public double getRequestedAmount() { return requestedAmount; }
    public double getAvailableBalance() { return availableBalance; }

    public static void main(String[] args) {
        try {
            withdraw("ACC-123", 500.00, 200.00);
        } catch (InsufficientFundsException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Account: " + e.getAccountNumber());
            System.out.println("Shortfall: $" +
                    (e.getRequestedAmount() - e.getAvailableBalance()));
        }
    }

    public static void withdraw(String account, double amount, double balance)
            throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(account, amount, balance);
        }
    }
}
```

Expected Output:

```
Error: Insufficient funds in account ACC-123: requested 500.0, available 200.0
Account: ACC-123
Shortfall: $300.0
```

**Why the code produces this result:** The exception carries three pieces of contextual data: the account number, the requested amount, and the available balance. The `getMessage()` returns a formatted string that includes all three. The catch block can also access the fields programmatically to compute the shortfall.

**Real-World Cases with Explanation**

In a payment processing system, a `PaymentDeclinedException` carries the transaction ID, the decline reason code, and the timestamp. In a file-processing system, a `FileParseException` carries the file name, the line number, and the offending content. In a validation framework, a `ValidationException` carries a list of field-level validation errors.

**References Links**

- How to create exception with specific fields? (Stack Overflow) – https://stackoverflow.com/questions/75672719/how-to-create-exception-with-specific-fields
- Adding custom fields to Java exception (Stack Overflow) – https://stackoverflow.com/questions/22839961/adding-custom-fields-to-java-exception

### 4.2 ContextedException Pattern (Apache Commons Lang)

**Definitions**

- **Core Definition:** A pattern for adding contextual information to any exception without creating a new subclass, using the `ContextedException` and `ContextedRuntimeException` classes from Apache Commons Lang.
- **Technical Definition:** `ContextedException` and `ContextedRuntimeException` provide a mechanism to attach label-value pairs to an exception. The contextual data is automatically included in the message and the printed stack trace. The `addContextValue(String label, Object value)` method adds a pair, and `getContextValues(String label)` retrieves values.
- **Beginner-Friendly Explanation:** If you do not want to create a custom exception class just to add a few pieces of context, the Apache Commons Lang library provides ready-made exception classes that can carry contextual data. You create one, add label-value pairs, and the data is automatically included in the stack trace.

**Purposes**

- To add contextual data to exceptions without creating custom subclasses.
- To produce richer stack traces for debugging and logging.
- To allow contextual data to be added at multiple levels of the call stack.

**Syntax Structures and Rules**

Complete general syntax:

```java
throw new ContextedRuntimeException("Error message", cause)
    .addContextValue("Label1", value1)
    .addContextValue("Label2", value2);
```

Component breakdown:

- `ContextedRuntimeException` or `ContextedException`: the wrapper exception.
- `.addContextValue(label, value)`: adds a label-value pair. Returns the same exception for chaining.
- The contextual data is included in the message when `printStackTrace()` or `getMessage()` is called.

**Multiple Annotated Complete Code Examples and Their Expected Outputs**

**Example 1: ContextedRuntimeException with contextual data**

```java
import org.apache.commons.lang3.exception.ContextedRuntimeException;

public class ContextedExceptionExample {
    public static void main(String[] args) {
        try {
            processTransaction("ACC-456", 1500.00);
        } catch (ContextedRuntimeException e) {
            e.printStackTrace();
        }
    }

    public static void processTransaction(String account, double amount) {
        try {
            throw new RuntimeException("Insufficient funds");
        } catch (RuntimeException cause) {
            throw new ContextedRuntimeException("Transaction failed", cause)
                    .addContextValue("Account Number", account)
                    .addContextValue("Amount", amount)
                    .addContextValue("Timestamp", System.currentTimeMillis());
        }
    }
}
```

Expected Output (approximate):

```
org.apache.commons.lang3.exception.ContextedRuntimeException: Transaction failed
Exception Context:
    [1:Account Number=ACC-456]
    [2:Amount=1500.0]
    [3:Timestamp=1700000000000]
    at ContextedExceptionExample.processTransaction(ContextedExceptionExample.java:14)
    at ContextedExceptionExample.main(ContextedExceptionExample.java:5)
Caused by: java.lang.RuntimeException: Insufficient funds
    at ContextedExceptionExample.processTransaction(ContextedExceptionExample.java:12)
    ... 1 more
```

**Why the code produces this result:** The `ContextedRuntimeException` wraps the original `RuntimeException` and attaches three label-value pairs. The `printStackTrace()` method formats the contextual data into the output.

**Real-World Cases with Explanation**

In a microservices architecture, a `ContextedRuntimeException` is used to attach correlation IDs, user IDs, and request paths to exceptions before they are logged or propagated. In a batch processing system, each record's failure is wrapped in a contexted exception carrying the record's identifier and position.

**References Links**

- ContextedRuntimeException (Apache Commons Lang API) – https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/exception/ContextedRuntimeException.html
- ContextedException (Apache Commons Lang API) – https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/exception/ContextedException.html


## Summary Table of Custom Exception Design

| Aspect | Checked Exception | Unchecked Exception | Context-Rich Exception |
|---|---|---|---|
| **Superclass** | `Exception` | `RuntimeException` | Either |
| **Compiler enforcement** | Must catch or declare | No enforcement | Depends on superclass |
| **When to use** | Recoverable, caller can handle | Programming error, unrecoverable | When diagnostic data is needed |
| **Standard constructors** | 4 (no-arg, message, cause, both) | 4 (same) | Custom + context |
| **serialVersionUID** | Required if serializable | Required if serializable | Required if serializable |
| **Naming** | `SomethingException` | `SomethingException` | `SomethingException` |
| **Context fields** | Optional | Optional | Required |


## Version-Specific Notes

- The `Throwable` cause-chaining mechanism (constructors accepting `Throwable cause`, `initCause()`, `getCause()`) was introduced in Java 1.4. Custom exceptions that predate Java 1.4 used only the message and no cause.
- `ContextedException` and `ContextedRuntimeException` are provided by Apache Commons Lang, not the JDK. They require the Commons Lang dependency.
- Lombok's `@StandardException` annotation (experimental, Lombok v1.18.21+) generates the four standard constructors automatically.
- `serialVersionUID` is recommended for all serializable classes, including exceptions, per the `Serializable` interface documentation.
- The Java Tutorials recommend appending “Exception” to the names of all classes that inherit from `Exception`.


## References

- Creating Exception Classes (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html
- Unchecked Exceptions — The Controversy (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html
- The Catch or Specify Requirement (The Java Tutorials) – https://docs.oracle.com/javase/tutorial/essential/exceptions/catchOrDeclare.html
- JLS §11.2: Compile-Time Checking of Exceptions – https://docs.oracle.com/javase/specs/jls/se21/html/jls-11.html#jls-11.2
- Throwable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Throwable.html
- Serializable (Java SE 21) – https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/Serializable.html
- Java Object Serialization Specification – https://docs.oracle.com/en/java/javase/21/docs/specs/serialization/index.html
- SEI CERT ERR51-J: Prefer user-defined exceptions over more general exception types – https://wiki.sei.cmu.edu/confluence/display/java/ERR51-J.+Prefer+user-defined+exceptions+over+more+general+exception+types
- Tuscany Java Coding Guidelines: Checked vs. unchecked exceptions – https://svn.apache.org/repos/asf/tuscany/java/sca/trunk/CodingGuidelines.txt
- ContextedRuntimeException (Apache Commons Lang API) – https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/exception/ContextedRuntimeException.html
- ContextedException (Apache Commons Lang API) – https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/exception/ContextedException.html
- Lombok @StandardException (Baeldung) – https://www.baeldung.com/lombok-standardexception-annotation
- How to create exception with specific fields? (Stack Overflow) – https://stackoverflow.com/questions/75672719/how-to-create-exception-with-specific-fields
- Adding custom fields to Java exception (Stack Overflow) – https://stackoverflow.com/questions/22839961/adding-custom-fields-to-java-exception
- Java Code Conventions: Naming Conventions – https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html