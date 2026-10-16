# Java Custom Exceptions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** A custom exception is a user-defined exception class that extends either `Exception` (for checked exceptions) or `RuntimeException` (for unchecked exceptions), created to represent application-specific error conditions that are not adequately covered by Java's built-in exception classes.

**Technical Definition:** "You should write your own exception classes if you answer yes to any of the following questions: Do you need an exception type that isn't represented by those in the Java platform? Would it help users if they could differentiate your exceptions from those thrown by classes written by other vendors? Does your code throw more than one related exception?". A custom exception typically provides constructors that mirror those of its superclass, delegating to `super(message)`, `super(cause)`, or `super(message, cause)`.

**Beginner-Friendly Explanation:** Think of built-in exceptions as generic bandages — they work for common injuries but don't tell you much about what specifically went wrong. A custom exception is like a specialised medical kit labelled "Broken Arm" or "Allergic Reaction." It tells you exactly what problem occurred, making it easier to handle and debug. Instead of throwing a vague `Exception`, you throw a `PaymentDeclinedException` or `InsufficientFundsException` that speaks your application's language.

### Key Characteristics

- **Domain-Specific:** Custom exceptions convey application-specific error conditions with meaningful names.
- **Hierarchical:** They fit into the existing `Throwable` hierarchy, extending `Exception` or `RuntimeException`.
- **Constructor Parity:** They typically provide the same four constructors as `Exception` (no-arg, message, cause, message+cause).
- **Serializable:** Since `Throwable` implements `Serializable`, custom exceptions should declare a `serialVersionUID`.
- **Checked or Unchecked:** The choice of superclass determines whether the compiler enforces handling.
- **Failure-Capture Information:** Custom exceptions can carry additional fields (e.g., `amount`, `customerId`) to capture failure context.

### Prerequisites

- Basic understanding of Java exceptions and the exception hierarchy
- Familiarity with checked and unchecked exceptions
- Knowledge of `try-catch` and `throws` clauses
- Understanding of inheritance and constructors

### Related Programming Areas

- **Exception Handling:** Custom exceptions are part of a broader error-handling strategy.
- **API Design:** Custom exceptions form part of a library's public error contract.
- **Domain-Driven Design:** Business rules are often expressed as domain-specific exceptions.
- **Testing:** Custom exceptions enable precise test assertions for error conditions.
- **Logging:** Custom exceptions provide structured error information for logging frameworks.

### Core Concepts / Features

---

## 1. Custom Exceptions

### Definitions

**Core Definition:** A custom exception is a class that extends `Exception` or `RuntimeException` to represent a specific error condition in an application.

**Technical Definition:** "To create a custom exception, we have to extend the `java.lang.Exception` class". For unchecked exceptions, extend `java.lang.RuntimeException`. The class should provide constructors that delegate to the superclass constructors.

**Beginner-Friendly Explanation:** A custom exception is your own error type. Instead of saying "something went wrong," you say exactly what went wrong in your application's terms.

### Purposes

- To represent domain-specific error conditions that built-in exceptions cannot adequately describe
- To differentiate your application's exceptions from those thrown by third-party libraries
- To provide meaningful error messages tailored to your application
- To carry additional failure-capture information (e.g., invalid values, context data)
- To enable precise exception handling by type rather than by message string

### Syntax Rules and Structure

**Complete General Syntax:**

```java
public class CustomException extends Exception {  // or RuntimeException
    private static final long serialVersionUID = 1L;

    public CustomException() {
        super();
    }

    public CustomException(String message) {
        super(message);
    }

    public CustomException(String message, Throwable cause) {
        super(message, cause);
    }

    public CustomException(Throwable cause) {
        super(cause);
    }
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `extends Exception` | Creates a checked exception. |
| `extends RuntimeException` | Creates an unchecked exception. |
| `serialVersionUID` | Required for serializable classes; prevents deserialization issues. |
| Constructors | Mirror the four standard `Exception` constructors. |
| Additional fields | Optional; capture failure-specific data. |

**Syntax Rules:**

- The class name should end with `Exception` (convention).
- The class should be `public` if used across packages.
- Constructors should delegate to `super()` with appropriate arguments.
- The `serialVersionUID` should be declared as `private static final long`.

**Constraints and Limitations:**

- Custom exceptions should not be created for every possible error; prefer standard exceptions when they fit.
- Adding custom fields increases the exception's footprint and serialization complexity.
- Checked custom exceptions pollute method signatures.

### Annotated Complete Step-by-Step Code Examples

**Example 1: Custom Checked Exception**

```java
// File: InsufficientFundsException.java
public class InsufficientFundsException extends Exception {
    private static final long serialVersionUID = 1L;

    private final double shortfall;  // Additional failure-capture field

    public InsufficientFundsException(double shortfall) {
        super("Insufficient funds: need an additional $" + shortfall);
        this.shortfall = shortfall;
    }

    public InsufficientFundsException(String message, Throwable cause) {
        super(message, cause);
        this.shortfall = 0;
    }

    public double getShortfall() {
        return shortfall;
    }
}
```

```java
// File: BankAccount.java
public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
        System.out.println("Withdrew $" + amount + ". Remaining: $" + balance);
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount(100.0);
        try {
            account.withdraw(150.0);
        } catch (InsufficientFundsException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Shortfall: $" + e.getShortfall());
        }
    }
}
```

**Expected Output:**
```
Error: Insufficient funds: need an additional $50.0
Shortfall: $50.0
```

**Step-by-Step Explanation:**

1. `InsufficientFundsException` extends `Exception`, making it a checked exception.
2. It declares a `serialVersionUID` for serialization compatibility.
3. The constructor takes a `shortfall` amount, builds a descriptive message, and calls `super(message)`.
4. `BankAccount.withdraw()` throws the custom exception when the withdrawal amount exceeds the balance.
5. The `main` method catches the exception and accesses both the message and the `shortfall` field.

**Example 2: Custom Unchecked Exception**

```java
// File: InvalidConfigurationException.java
public class InvalidConfigurationException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public InvalidConfigurationException(String message) {
        super(message);
    }

    public InvalidConfigurationException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

```java
// File: ConfigLoader.java
public class ConfigLoader {
    public static String loadConfig(String key) {
        if (key == null || key.isEmpty()) {
            throw new InvalidConfigurationException("Configuration key cannot be empty");
        }
        return "value-for-" + key;
    }

    public static void main(String[] args) {
        // Unchecked: no try-catch required by compiler
        String value = loadConfig("database.url");
        System.out.println("Loaded: " + value);

        try {
            loadConfig("");
        } catch (InvalidConfigurationException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Loaded: value-for-database.url
Error: Configuration key cannot be empty
```

**Step-by-Step Explanation:**

1. `InvalidConfigurationException` extends `RuntimeException`, making it unchecked.
2. The compiler does not require callers to catch or declare it.
3. `loadConfig` throws the exception for invalid input.
4. The `main` method demonstrates that catching is optional but can be done for graceful handling.

### Real-World Cases

- **Banking:** `InsufficientFundsException`, `AccountLockedException`.
- **E-commerce:** `PaymentDeclinedException`, `OutOfStockException`.
- **Authentication:** `InvalidCredentialsException`, `AccountExpiredException`.
- **Configuration:** `InvalidConfigurationException`, `MissingPropertyException`.
- **File Processing:** `IncorrectFileNameException`.

---

## 2. Checked Custom Exceptions

### Definitions

**Core Definition:** A checked custom exception is a custom exception that extends `Exception` (but not `RuntimeException`), requiring callers to handle or declare it.

**Technical Definition:** "Checked Exceptions: Extend `Exception`. The compiler forces you to handle them using try-catch blocks or declare them in the method signature with `throws`".

**Beginner-Friendly Explanation:** A checked exception is like a formal contract. When a method says it might throw a checked exception, the compiler forces every caller to either handle the problem or pass the responsibility up the chain. This ensures no one forgets about potential errors.

### Purposes

- To force callers to acknowledge and handle recoverable error conditions
- To document a method's error contract in its signature
- To represent errors that the caller can realistically recover from (e.g., file not found, network timeout)

### Syntax Rules and Structure

```java
public class MyCheckedException extends Exception {
    public MyCheckedException(String message) {
        super(message);
    }
}
```

**Syntax Rules:**

- Must extend `Exception` directly (not `RuntimeException`).
- Callers must either catch it with `try-catch` or declare it with `throws`.
- Should be used when the caller can reasonably recover from the error.

**Constraints and Limitations:**

- Checked exceptions pollute method signatures and propagate up the call stack.
- They are incompatible with lambda expressions and streams, which cannot throw checked exceptions.
- Overuse leads to "throws pollution" where every method declares many checked exceptions.

### Annotated Complete Step-by-Step Code Examples

```java
// File: CheckedCustomDemo.java
public class CheckedCustomDemo {

    static class ValidationException extends Exception {
        public ValidationException(String message) {
            super(message);
        }
    }

    static void validateEmail(String email) throws ValidationException {
        if (email == null || !email.contains("@")) {
            throw new ValidationException("Invalid email: " + email);
        }
        System.out.println("Email valid: " + email);
    }

    public static void main(String[] args) {
        try {
            validateEmail("alice@example.com");  // Output: Email valid: alice@example.com
            validateEmail("invalid-email");       // Throws ValidationException
        } catch (ValidationException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Email valid: alice@example.com
Caught: Invalid email: invalid-email
```

**Step-by-Step Explanation:**

1. `ValidationException` extends `Exception`, making it checked.
2. `validateEmail` declares `throws ValidationException`, so callers must handle it.
3. The compiler enforces the `try-catch` in `main`.
4. The first call succeeds; the second throws the exception, which is caught and handled.

### Real-World Cases

- **File I/O:** `IOException` and its subclasses are checked.
- **Database:** `SQLException` is checked.
- **Reflection:** `ClassNotFoundException`, `NoSuchMethodException` are checked.
- **Custom APIs:** Libraries that want to force callers to handle domain-specific errors.

---

## 3. Unchecked Custom Exceptions

### Definitions

**Core Definition:** An unchecked custom exception is a custom exception that extends `RuntimeException`, which the compiler does not require to be caught or declared.

**Technical Definition:** "Unchecked Exceptions: Extend `RuntimeException`. They don't need to be explicitly caught or declared".

**Beginner-Friendly Explanation:** An unchecked exception is like an unexpected event that the compiler doesn't force you to plan for. They usually indicate programming mistakes (like invalid arguments) that should be fixed in the code rather than caught at runtime.

### Purposes

- To signal programming errors (bugs) that should be fixed rather than caught
- To avoid cluttering method signatures with exceptions that callers cannot recover from
- To work seamlessly with lambda expressions and streams
- To enable fail-fast behaviour for invalid state or arguments

### Syntax Rules and Structure

```java
public class MyUncheckedException extends RuntimeException {
    public MyUncheckedException(String message) {
        super(message);
    }
}
```

**Syntax Rules:**

- Must extend `RuntimeException`.
- The compiler does not require handling.
- Should be used for programming errors (invalid arguments, illegal state transitions).

**Constraints and Limitations:**

- Not suitable for recoverable conditions.
- "Do not throw a `RuntimeException` or create a subclass of `RuntimeException` simply because you don't want to be bothered with specifying the exceptions your methods can throw" (Effective Java).

### Annotated Complete Step-by-Step Code Examples

```java
// File: UncheckedCustomDemo.java
public class UncheckedCustomDemo {

    static class InvalidOrderException extends RuntimeException {
        public InvalidOrderException(String message) {
            super(message);
        }
        public InvalidOrderException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    static void processOrder(String orderId, double amount) {
        if (orderId == null || orderId.isEmpty()) {
            throw new InvalidOrderException("Order ID cannot be empty");
        }
        if (amount <= 0) {
            throw new InvalidOrderException("Amount must be positive: " + amount);
        }
        System.out.println("Order " + orderId + " processed for $" + amount);
    }

    public static void main(String[] args) {
        processOrder("ORD-001", 99.99);  // Output: Order ORD-001 processed for $99.99

        try {
            processOrder("", 50.0);
        } catch (InvalidOrderException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Order ORD-001 processed for $99.99
Error: Order ID cannot be empty
```

**Step-by-Step Explanation:**

1. `InvalidOrderException` extends `RuntimeException`, making it unchecked.
2. The compiler does not require a `try-catch` for `processOrder`.
3. The first call succeeds; the second throws the exception, which is caught optionally.
4. The `try-catch` is used for graceful error handling, not because the compiler requires it.

### Real-World Cases

- **Argument Validation:** `IllegalArgumentException` is the standard unchecked exception.
- **State Validation:** `IllegalStateException` for invalid object state.
- **Domain Errors:** `InvalidOrderException`, `PaymentDeclinedException` when the caller cannot recover.

---

## 4. Exception Constructors

### Definitions

**Core Definition:** Exception constructors are the constructors provided by a custom exception class that initialise the exception with a message, a cause, or both.

**Technical Definition:** The four standard constructors are: no-argument, message-only, cause-only, and message+cause. "So yes, providing the same constructors as `Exception`, and delegating to the corresponding super constructor, is a good rule of thumb".

**Beginner-Friendly Explanation:** Exception constructors are like different ways to fill out an incident report. You can write just the "what happened" (message), just the "why it happened" (cause), both, or leave it blank.

### Purposes

- To provide flexibility in how the exception is created
- To ensure the exception carries all necessary information (message, cause)
- To maintain consistency with the standard `Exception` class

### Syntax Rules and Structure

```java
public class CustomException extends Exception {
    // 1. No-argument constructor
    public CustomException() {
        super();
    }

    // 2. Message-only constructor
    public CustomException(String message) {
        super(message);
    }

    // 3. Message + cause constructor
    public CustomException(String message, Throwable cause) {
        super(message, cause);
    }

    // 4. Cause-only constructor
    public CustomException(Throwable cause) {
        super(cause);
    }
}
```

**Component Breakdown:**

| Constructor | Description |
|-------------|-------------|
| `CustomException()` | Creates an exception with no message or cause. |
| `CustomException(String message)` | Creates an exception with a message. |
| `CustomException(String message, Throwable cause)` | Creates an exception with a message and a cause. |
| `CustomException(Throwable cause)` | Creates an exception with a cause; message is derived from cause. |

**Syntax Rules:**

- Each constructor should delegate to the corresponding `super()` constructor.
- The `serialVersionUID` should be declared as `private static final long`.
- Additional fields should be initialised in the constructors.

**Constraints and Limitations:**

- The no-argument constructor provides minimal debugging information.
- The cause-only constructor derives the message from `cause.toString()`, which may not be ideal.
- Lombok's `@StandardException` can generate all four constructors automatically.

### Annotated Complete Step-by-Step Code Examples

```java
// File: FullConstructorException.java
public class FullConstructorException extends Exception {
    private static final long serialVersionUID = 1L;

    public FullConstructorException() {
        super();
    }

    public FullConstructorException(String message) {
        super(message);
    }

    public FullConstructorException(String message, Throwable cause) {
        super(message, cause);
    }

    public FullConstructorException(Throwable cause) {
        super(cause);
    }

    public static void main(String[] args) {
        // Using each constructor
        FullConstructorException e1 = new FullConstructorException();
        FullConstructorException e2 = new FullConstructorException("Message only");
        FullConstructorException e3 = new FullConstructorException("Message and cause",
            new RuntimeException("Root cause"));
        FullConstructorException e4 = new FullConstructorException(
            new RuntimeException("Cause only"));

        System.out.println("e1 message: " + e1.getMessage());
        System.out.println("e2 message: " + e2.getMessage());
        System.out.println("e3 message: " + e3.getMessage());
        System.out.println("e3 cause: " + e3.getCause().getMessage());
        System.out.println("e4 message: " + e4.getMessage());
        System.out.println("e4 cause: " + e4.getCause().getMessage());
    }
}
```

**Expected Output:**
```
e1 message: null
e2 message: Message only
e3 message: Message and cause
e3 cause: Root cause
e4 message: java.lang.RuntimeException: Cause only
e4 cause: Cause only
```

**Step-by-Step Explanation:**

1. The class provides all four standard constructors.
2. `e1` has no message or cause.
3. `e2` has a message only.
4. `e3` has both a message and a cause.
5. `e4` has a cause only; the message is derived from `cause.toString()`.

### Real-World Cases

- **Library Design:** Public exceptions should provide all four constructors for maximum flexibility.
- **Lombok Users:** `@StandardException` generates all four constructors automatically.
- **Framework Integration:** Frameworks may require specific constructors for exception mapping.

---

## 5. Exception Messages

### Definitions

**Core Definition:** An exception message is a human-readable string that describes the error condition.

**Technical Definition:** "The detail message is saved for later retrieval by the `getMessage()` method". The message should contain all relevant data that contributed to the exception.

**Beginner-Friendly Explanation:** The exception message is the "what happened" part of the error report. It should tell you enough to understand the problem without needing to read the code.

### Purposes

- To provide a human-readable description of the error
- To include failure-capture information (values, context)
- To aid debugging and logging
- To communicate the problem to users or developers

### Syntax Rules and Structure

```java
throw new CustomException("Invalid value: " + value + " (expected: " + expected + ")");
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| Literal text | Describes the error condition. |
| Variable values | Captures the actual values that caused the error. |
| Context | Additional information (expected values, ranges). |

**Syntax Rules:**

- Include all relevant data: the invalid value, the expected condition, and context.
- Avoid including sensitive information (passwords, keys).
- Use clear, descriptive language.
- Prefer including values in the message rather than requiring the caller to inspect fields.

**Constraints and Limitations:**

- Messages should not be parsed programmatically; use exception types for programmatic handling.
- Messages should not expose sensitive information.
- Very long messages may be truncated in logs.

### Annotated Complete Step-by-Step Code Examples

```java
// File: MessageDemo.java
public class MessageDemo {

    static class RangeException extends RuntimeException {
        public RangeException(int value, int min, int max) {
            super("Value " + value + " is outside the allowed range [" + min + ", " + max + "]");
        }
    }

    static void checkRange(int value) {
        if (value < 0 || value > 100) {
            throw new RangeException(value, 0, 100);
        }
        System.out.println("Value accepted: " + value);
    }

    public static void main(String[] args) {
        checkRange(50);   // Output: Value accepted: 50
        try {
            checkRange(150);
        } catch (RangeException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

**Expected Output:**
```
Value accepted: 50
Error: Value 150 is outside the allowed range [0, 100]
```

**Step-by-Step Explanation:**

1. `RangeException` builds a message that includes the invalid value and the allowed range.
2. `checkRange` throws the exception when the value is out of range.
3. The message provides all the information needed to understand the error without inspecting the exception object.

### Real-World Cases

- **Validation:** "Age -5 is invalid; must be between 0 and 150."
- **File Processing:** "File '/data/input.csv' not found."
- **Network:** "Connection to 'api.example.com:443' timed out after 30 seconds."

---

## 6. Exception Causes

### Definitions

**Core Definition:** An exception cause is the original exception that triggered the creation of a new exception, preserved for diagnostic purposes.

**Technical Definition:** "The `Throwable` argument to `initCause` and the `Throwable` constructors is the exception that caused the current exception. `getCause` returns the exception that caused the current exception". "Note that the detail message associated with `cause` is not automatically incorporated in this exception's detail message".

**Beginner-Friendly Explanation:** A cause is like a chain of dominoes. When one exception triggers another, the original exception is preserved as the cause so you can trace the problem back to its source.

### Purposes

- To preserve the original exception when wrapping it in a higher-level exception
- To provide a complete chain of errors for debugging
- To translate low-level exceptions into domain-specific exceptions while retaining context
- To enable root-cause analysis in logs

### Syntax Rules and Structure

```java
try {
    // risky code
} catch (LowLevelException e) {
    throw new HighLevelException("Context message", e);  // e is the cause
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `HighLevelException` | The wrapping exception. |
| `e` | The original cause. |
| `getCause()` | Retrieves the original exception. |

**Syntax Rules:**

- The cause is set via the constructor `(String message, Throwable cause)` or `initCause()`.
- `getCause()` returns the original exception, or `null` if unknown.
- Chained exceptions form a linked list accessible via repeated `getCause()` calls.

**Constraints and Limitations:**

- The cause's message is not automatically included in the wrapper's message.
- Cause chains can become deep; use `getCause()` iteratively to traverse.

### Annotated Complete Step-by-Step Code Examples

```java
// File: CauseDemo.java
import java.io.*;

public class CauseDemo {

    static class ProcessingException extends RuntimeException {
        public ProcessingException(String message, Throwable cause) {
            super(message, cause);
        }
    }

    static void processFile(String path) {
        try {
            throw new IOException("File not found: " + path);
        } catch (IOException e) {
            throw new ProcessingException("Failed to process: " + path, e);
        }
    }

    public static void main(String[] args) {
        try {
            processFile("data.txt");
        } catch (ProcessingException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Cause: " + e.getCause().getMessage());
        }
    }
}
```

**Expected Output:**
```
Error: Failed to process: data.txt
Cause: File not found: data.txt
```

**Step-by-Step Explanation:**

1. `processFile` catches an `IOException` and wraps it in a `ProcessingException`.
2. The original `IOException` is preserved as the cause via the constructor.
3. `getCause()` retrieves the original exception for diagnostic purposes.
4. The output shows both the wrapper message and the root cause.

### Real-World Cases

- **Service Layers:** Wrapping `SQLException` in a domain-specific `DataAccessException`.
- **API Clients:** Wrapping `IOException` in a `ServiceException`.
- **Framework Integration:** Translating framework exceptions to application exceptions.
- **Logging:** Logging full cause chains for root-cause analysis.

---

## 7. Exception Best Practices

### Definitions

**Core Definition:** Exception best practices are guidelines for creating, throwing, catching, and documenting custom exceptions effectively.

**Technical Definition:** Best practices derive from authoritative sources including *Effective Java* by Joshua Bloch, the SEI CERT Oracle Coding Standard for Java, and Oracle's Java Tutorials. Key principles include using exceptions only for exceptional conditions, favouring standard exceptions when possible, and preserving failure-capture information.

**Beginner-Friendly Explanation:** Best practices help you use exceptions correctly without making your code hard to read or maintain. They're like traffic rules for exceptions — they keep things orderly and prevent accidents.

### Purposes

- To improve code readability and maintainability
- To ensure exceptions are used for their intended purpose
- To provide meaningful error information
- To avoid common pitfalls

### Key Best Practices

| Practice | Description |
|----------|-------------|
| Prefer User-Defined Exceptions Over General Types | "Because an exception is caught by its type, it is better to define exceptions for specific purposes than to use the general exception types". |
| Use Exceptions Only for Exceptional Conditions | Do not use exceptions for control flow. |
| Use Checked Exceptions for Recoverable Conditions | Use unchecked for programming errors. |
| Favor Standard Exceptions | Use `IllegalArgumentException`, `IllegalStateException`, etc., when they fit. |
| Document Exceptions with Javadoc | Use `@throws` tags to document conditions. |
| Include Failure-Capture Information | Exception messages should contain all relevant data. |
| Preserve the Cause | Always pass the original exception when wrapping. |
| Provide All Four Constructors | Mirror the standard `Exception` constructors. |
| Declare `serialVersionUID` | Since `Throwable` is `Serializable`, declare `private static final long serialVersionUID = 1L`. |
| Name with "Exception" Suffix | End the class name with `Exception` for easy identification. |
| Avoid Exposing Sensitive Information | Do not include passwords, keys, or internal paths in messages. |
| Use Lombok's `@StandardException` | Generate all four constructors automatically. |

### Annotated Complete Step-by-Step Code Examples

```java
// File: BestPracticesCustomException.java
import java.util.*;

public class BestPracticesCustomException {

    // GOOD: Descriptive name ending with "Exception"
    // GOOD: All four constructors
    // GOOD: serialVersionUID declared
    // GOOD: Additional failure-capture field
    public static class InvalidOrderException extends Exception {
        private static final long serialVersionUID = 1L;

        private final String orderId;
        private final double amount;

        public InvalidOrderException() {
            this(null, 0.0, "Invalid order");
        }

        public InvalidOrderException(String message) {
            this(null, 0.0, message);
        }

        public InvalidOrderException(String message, Throwable cause) {
            this(null, 0.0, message, cause);
        }

        public InvalidOrderException(Throwable cause) {
            this(null, 0.0, cause.getMessage(), cause);
        }

        public InvalidOrderException(String orderId, double amount, String message) {
            super(message);
            this.orderId = orderId;
            this.amount = amount;
        }

        public InvalidOrderException(String orderId, double amount, String message, Throwable cause) {
            super(message, cause);
            this.orderId = orderId;
            this.amount = amount;
        }

        public String getOrderId() { return orderId; }
        public double getAmount() { return amount; }
    }

    // GOOD: Validate and throw with failure-capture information
    public static void processOrder(String orderId, double amount) throws InvalidOrderException {
        if (orderId == null || orderId.isEmpty()) {
            throw new InvalidOrderException(orderId, amount, "Order ID cannot be empty");
        }
        if (amount <= 0) {
            throw new InvalidOrderException(orderId, amount,
                "Amount must be positive: " + amount);
        }
        System.out.println("Order " + orderId + " processed for $" + amount);
    }

    public static void main(String[] args) {
        try {
            processOrder("ORD-001", 99.99);
            // Output: Order ORD-001 processed for $99.99

            processOrder("", 50.0);
        } catch (InvalidOrderException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Order ID: " + e.getOrderId());
            System.out.println("Amount: " + e.getAmount());
        }
    }
}
```

**Expected Output:**
```
Order ORD-001 processed for $99.99
Error: Order ID cannot be empty
Order ID: 
Amount: 50.0
```

**Step-by-Step Explanation:**

1. `InvalidOrderException` follows the naming convention (ends with `Exception`).
2. It declares `serialVersionUID` for serialization compatibility.
3. It provides all four standard constructors plus additional constructors with failure-capture fields.
4. `processOrder` throws the exception with the invalid order ID and amount.
5. The `catch` block accesses both the message and the custom fields.

### Real-World Cases

- **Library APIs:** Public exceptions should provide all four constructors and document their conditions.
- **Domain Models:** Business exceptions should carry context (order ID, customer ID, amounts).
- **Framework Integration:** Custom exceptions should preserve causes for root-cause analysis.
- **Code Reviews:** Best practices form the basis for reviewing exception handling code.

---

## References Links

- Creating Exception Classes — Oracle Java Tutorials - https://docs.oracle.com/javase/tutorial/essential/exceptions/creating.html
- Create a Custom Exception in Java — Baeldung - https://www.baeldung.com/java-new-custom-exception
- ERR51-J. Prefer user-defined exceptions over more general exception types — SEI CERT Oracle Coding Standard for Java - https://wiki.sei.cmu.edu/confluence/display/java/ERR51-J.+Prefer+user-defined+exceptions+over+more+general+exception+types
- Java Custom Exceptions — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/java/5-java-exception-handling/2-java-custom-exceptions.mdx
- Class Exception — Java Platform SE API Specification - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Exception.html
- @StandardException Annotation — Project Lombok - https://projectlombok.org/features/StandardException
- @StandardException Annotation in Lombok — Baeldung - https://www.baeldung.com/java-lombok-standard-exception
- Effective Java (3rd Edition) by Joshua Bloch — Chapter 10: Exceptions
- Java Exceptions Tutorial: Try-Catch, Throws, Checked vs Unchecked, Best Practices — Cleverence - https://www.cleverence.com/articles/oracle-documentation/java-exceptions-tutorial-4821/