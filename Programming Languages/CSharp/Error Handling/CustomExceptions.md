# C# Custom Exceptions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
A **custom exception** is a user-defined exception class that derives from `System.Exception` (or another appropriate base exception) and represents an application-specific error condition that is not adequately described by the predefined .NET exception types. Custom exceptions give errors **semantic identity**, enabling callers to catch, differentiate, and respond to domain-specific failures with precision, while carrying additional data that is meaningful to the application.

**Technical Definition**
A custom exception is any class defined in user code that derives, directly or indirectly, from `System.Exception`. The .NET design guidelines require that custom exception classes (1) derive from `System.Exception` or another common base exception—**not** from `System.ApplicationException`, which is deprecated as a base type—(2) end their class name with the `Exception` suffix for consistency, and (3) provide at least the three standard constructors: a parameterless constructor, a constructor accepting a `string message`, and a constructor accepting a `string message` and an `Exception inner`. Historically, exceptions were also expected to be `[Serializable]` and to provide a protected serialization constructor plus a `GetObjectData` override, but **binary serialization support was removed in .NET Core 1.0 and the relevant APIs are obsolete as of .NET 8 (`SYSLIB0051`)**; new custom exceptions should generally **omit** `[Serializable]` and the serialization members unless compatibility with legacy scenarios is explicitly required. Additional constructors and properties may be added to carry domain-specific information, following the same naming and structural conventions.

**Beginner-Friendly Explanation**
A custom exception is **your own error type**. The .NET framework already provides exceptions like `ArgumentNullException` or `InvalidOperationException`, but sometimes your application has a specific failure that deserves its own name—for example, `StudentNotFoundException`, `InsufficientFundsException`, or `OrderAlreadyShippedException`. By creating a dedicated class, you make the error **catchable by type** (callers can write `catch (StudentNotFoundException ex)`), **self-documenting**, and **capable of carrying extra data** (such as the student’s name or the order ID). Think of it as giving your error a **first-class identity** rather than burying it inside a generic `Exception` with a string message.

---

### Key Characteristics

- **Derived from `System.Exception`** — Custom exceptions form part of the standard exception hierarchy.
- **Semantic identity** — The type name communicates the exact error condition.
- **Catchable by type** — Callers can handle the specific exception without parsing messages.
- **Domain-specific data** — Additional properties carry context (IDs, names, values).
- **Three standard constructors** — Parameterless, message, and message + inner exception.
- **`Exception` suffix convention** — Names end with `Exception` for discoverability.
- **Serialization is obsolete in modern .NET** — `[Serializable]` and serialization members are no longer recommended (`SYSLIB0051`).
- **Not a replacement for standard exceptions** — Use predefined exceptions when they already express the error adequately.

---

### Prerequisites

- Understanding of **exception handling** (`try`/`catch`/`finally`) and the `System.Exception` hierarchy.
- Familiarity with **class inheritance** and constructors in C#.
- Knowledge of the **standard exception types** (`ArgumentException`, `InvalidOperationException`, `NotSupportedException`, etc.).
- Awareness of **exception best practices** (catch specific types, preserve stack traces).
- Basic understanding of **binary serialization** and why it was used (legacy remoting).

---

### Related Programming Areas

- **API design** — Custom exceptions are part of a library’s public contract.
- **Domain-driven design** — Domain exceptions express business rule violations.
- **Service layers** — Translating infrastructure exceptions into domain exceptions.
- **Testing** — Tests can assert that a specific custom exception is thrown.
- **Serialization** — Legacy concern; now largely obsolete in .NET Core+.

---

### Core Concepts / Features

1. Custom Exceptions (Overarching Concept)
2. Exception Inheritance
3. Exception Constructors
4. Exception Serialization (and its Obsolescence)
5. Custom Exception Best Practices

---

## 1. Custom Exceptions (Overarching Concept)

### Definitions

**Core Definition**
A custom exception is an application-defined exception type that derives from `System.Exception` to represent a domain-specific error condition.

**Technical Definition**
Custom exceptions are classes that inherit from `System.Exception` (or a suitable derived base) and are used to signal error conditions that are not covered by the predefined exception hierarchy. They follow the .NET naming convention (suffix `Exception`), provide the three standard constructors, and may expose additional properties for programmatic access to error data.

**Beginner-Friendly Explanation**
A custom exception is **your own named error**. It lets callers say “catch the *student-not-found* error” instead of “catch the generic error and inspect the message.”

---

### Purposes

- To give application-specific errors a **distinct, catchable type**.
- To **carry domain data** (IDs, names, values) that is relevant to the error.
- To **improve readability** and self-documentation of error handling code.
- To **decouple** error detection from error recovery at domain boundaries.
- To **translate** low-level infrastructure errors into meaningful domain errors.

---

### Syntax Rules and Structure

```csharp
public class DomainException : Exception
{
    public DomainException() { }
    public DomainException(string message) : base(message) { }
    public DomainException(string message, Exception inner) : base(message, inner) { }
}
```

**Component breakdown**

- `: Exception` — Inheritance from the base exception type.
- `DomainException()` — Parameterless constructor.
- `DomainException(string message)` — Message constructor.
- `DomainException(string message, Exception inner)` — Message + inner exception constructor.

#### Syntax Rules

- Derive from `System.Exception` (or a suitable derived base), **not** `System.ApplicationException`.
- End the class name with `Exception`.
- Provide the three standard constructors.
- Mark the class `sealed` unless you intend to allow further derivation.
- Add domain-specific properties and constructors as needed.

#### Constraints and Limitations

- Do not create deep exception hierarchies.
- Do not use custom exceptions where a predefined exception is sufficient.
- Do not expose sensitive information through exception messages or properties.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Custom Exception

```csharp
using System;

public sealed class StudentNotFoundException : Exception
{
    public StudentNotFoundException() { }
    public StudentNotFoundException(string message) : base(message) { }
    public StudentNotFoundException(string message, Exception inner) : base(message, inner) { }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new StudentNotFoundException("The student cannot be found.");
        }
        catch (StudentNotFoundException ex)
        {
            Console.WriteLine($"Caught: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Caught: The student cannot be found.
```

**Why this output occurs**
The custom exception derives from `Exception`, follows the naming convention, provides the three standard constructors, and is caught by its specific type.

---

#### Example 2: Custom Exception with Domain Data

```csharp
using System;

public sealed class StudentNotFoundException : Exception
{
    public string StudentName { get; }

    public StudentNotFoundException() { }
    public StudentNotFoundException(string message) : base(message) { }
    public StudentNotFoundException(string message, Exception inner) : base(message, inner) { }

    public StudentNotFoundException(string message, string studentName) : this(message)
    {
        StudentName = studentName;
    }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new StudentNotFoundException("The student cannot be found.", "John");
        }
        catch (StudentNotFoundException ex)
        {
            Console.WriteLine($"Student: {ex.StudentName}");
            Console.WriteLine($"Message: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Student: John
Message: The student cannot be found.
```

**Why this output occurs**
The additional constructor and read-only property carry domain data (`StudentName`) alongside the standard exception message.

---

### Real-World Cases

- **E-commerce** — `InsufficientStockException`, `PaymentDeclinedException`.
- **Education** — `StudentNotFoundException`, `CourseFullException`.
- **Banking** — `InsufficientFundsException`, `AccountFrozenException`.
- **Workflow** — `InvalidStateTransitionException`.

---

### References

- How to: Create User-Defined Exceptions — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
- Designing Custom Exceptions — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-3.0/ms229064(v=vs.85)

---

## 2. Exception Inheritance

### Definitions

**Core Definition**
Exception inheritance is the derivation of custom exception types from `System.Exception` or from an appropriate existing exception base, establishing a type hierarchy that enables catch-by-type semantics.

**Technical Definition**
Custom exceptions inherit from `System.Exception`, either directly or through an intermediate base such as `ArgumentException` or `InvalidOperationException` when the error semantically belongs to that family. The .NET guidelines explicitly state that custom exceptions should derive from `System.Exception` or one of the other common base exceptions, and **not** from `System.ApplicationException`. Inheritance determines the order of `catch` clauses (most derived first) and the set of handlers that will catch the exception.

**Beginner-Friendly Explanation**
Exception inheritance means **choosing the right parent for your error**. If your error is a kind of “bad argument,” derive from `ArgumentException`; if it is a kind of “invalid operation,” derive from `InvalidOperationException`; otherwise, derive directly from `Exception`. The parent determines which existing `catch` blocks will also catch your exception.

---

### Purposes

- To **place the custom exception in the correct family** of standard exceptions.
- To enable **hierarchical catching** (catch a base type to handle all derived types).
- To follow .NET design guidelines and ecosystem conventions.
- To **avoid deep hierarchies** that complicate error handling.
- To ensure compatibility with existing framework exception-handling logic.

---

### Syntax Rules and Structure

```csharp
// Directly from Exception
public sealed class DomainException : Exception { ... }

// From a standard base exception
public sealed class InvalidOrderException : InvalidOperationException { ... }

// From ArgumentException
public sealed class InvalidStudentIdException : ArgumentException { ... }
```

**Component breakdown**

- `: Exception` — Direct derivation.
- `: InvalidOperationException` — Semantic family derivation.
- `: ArgumentException` — Argument-specific family derivation.

#### Syntax Rules

- Derive from `System.Exception` or a standard base exception that fits the error semantics.
- Do **not** derive from `System.ApplicationException`.
- Avoid deep hierarchies (prefer flat designs).
- Use `sealed` unless further derivation is explicitly intended.

#### Constraints and Limitations

- Deriving from a base exception means the custom exception will be caught by handlers for that base type—this may or may not be desired.
- Deep hierarchies make it difficult to reason about which handler catches what.

---

### Annotated Code Examples and Expected Outputs

#### Example: Deriving from `InvalidOperationException`

```csharp
using System;

public sealed class OrderAlreadyShippedException : InvalidOperationException
{
    public OrderAlreadyShippedException() { }
    public OrderAlreadyShippedException(string message) : base(message) { }
    public OrderAlreadyShippedException(string message, Exception inner) : base(message, inner) { }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new OrderAlreadyShippedException("Order 42 has already shipped.");
        }
        catch (InvalidOperationException ex)   // catches the custom exception too
        {
            Console.WriteLine($"Invalid operation: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Invalid operation: Order 42 has already shipped.
```

**Why this output occurs**
The custom exception derives from `InvalidOperationException`; a handler for the base type catches it, demonstrating hierarchical catching.

---

### Real-World Cases

- **Argument validation** — Deriving from `ArgumentException` to signal invalid inputs.
- **State violations** — Deriving from `InvalidOperationException` for invalid object state.
- **Feature gaps** — Deriving from `NotSupportedException` for unsupported operations.
- **Domain errors** — Deriving directly from `Exception` when no standard family fits.

---

### References

- Designing Custom Exceptions — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-3.0/ms229064(v=vs.85)
- MA0013: Types should not extend System.ApplicationException — https://github.com

---

## 3. Exception Constructors

### Definitions

**Core Definition**
Exception constructors are the public constructors on a custom exception class that initialize the exception’s state, typically including the message, the inner exception, and any domain-specific data.

**Technical Definition**
The .NET guidelines require custom exceptions to provide **at least three standard constructors**: `()`, `(string message)`, and `(string message, Exception inner)`. These constructors forward to the corresponding base-class constructors via `: base(...)`. Additional constructors may accept domain-specific parameters and initialize additional properties. Historically, a **protected serialization constructor** `(SerializationInfo, StreamingContext)` was also required, but this is now obsolete in .NET 8+ (`SYSLIB0051`) and should be removed or marked obsolete.

**Beginner-Friendly Explanation**
Constructors are **how you create the error object**. The three standard ones let callers create the exception with no message, with a message, or with a message and an inner exception (the error that caused this one).

---

### Purposes

- To **initialize the message** and **inner exception**.
- To **satisfy .NET design guidelines** and enable consistent usage.
- To **carry domain-specific data** through additional parameters.
- To **preserve the original cause** via the inner exception.
- To support **serialization** (legacy; now obsolete).

---

### Syntax Rules and Structure

```csharp
public sealed class MyException : Exception
{
    public MyException() { }
    public MyException(string message) : base(message) { }
    public MyException(string message, Exception inner) : base(message, inner) { }

    // Domain-specific constructor
    public MyException(string message, string domainId) : this(message)
    {
        DomainId = domainId;
    }

    public string DomainId { get; }
}
```

**Component breakdown**

- `MyException()` — Parameterless; used by serializers and reflection.
- `MyException(string message)` — Message only.
- `MyException(string message, Exception inner)` — Message + inner.
- `MyException(string message, string domainId)` — Domain-specific data.

#### Syntax Rules

- Provide the three standard constructors on every custom exception.
- Forward to the base class with `: base(...)`.
- Initialize domain-specific properties in additional constructors.
- The serialization constructor is **obsolete** in .NET 8+; remove it unless required for compatibility.

#### Constraints and Limitations

- The parameterless constructor is required for serialization (legacy) and for reflection-based instantiation.
- The message constructor should not be used to hide the original cause; use the inner exception constructor for that.

---

### Annotated Code Examples and Expected Outputs

#### Example: Three Standard Constructors and a Domain Constructor

```csharp
using System;

public sealed class PaymentFailedException : Exception
{
    public string TransactionId { get; }

    public PaymentFailedException() { }
    public PaymentFailedException(string message) : base(message) { }
    public PaymentFailedException(string message, Exception inner) : base(message, inner) { }

    public PaymentFailedException(string message, string transactionId) : this(message)
    {
        TransactionId = transactionId;
    }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new PaymentFailedException("Payment declined.", "TXN-12345");
        }
        catch (PaymentFailedException ex)
        {
            Console.WriteLine($"Transaction: {ex.TransactionId}");
            Console.WriteLine($"Message: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Transaction: TXN-12345
Message: Payment declined.
```

**Why this output occurs**
The domain-specific constructor delegates to the message constructor and initializes `TransactionId`.

---

### Real-World Cases

- **API libraries** — Standard constructors allow callers to throw the exception with or without context.
- **Service layers** — Inner exception constructor preserves the low-level cause.
- **Testing** — Parameterless constructor allows simple instantiation.

---

### References

- How to: Create User-Defined Exceptions — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
- CA1032: Implement standard exception constructors — .NET Code Analysis — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1032

---

## 4. Exception Serialization

### Definitions

**Core Definition**
Exception serialization is the legacy practice of making an exception class serializable so it can cross remoting or application-domain boundaries. In modern .NET, this practice is **obsolete and generally unnecessary**.

**Technical Definition**
Historically, the .NET guidelines recommended marking custom exceptions with `[Serializable]` and implementing a protected serialization constructor `(SerializationInfo, StreamingContext)` plus a `GetObjectData` override. However, **binary serialization support was removed in .NET Core 1.0**, and as of **.NET 8**, the remaining serialization APIs (`SerializationInfo`, `StreamingContext`, `GetObjectData`) are **obsolete** and produce diagnostic `SYSLIB0051`. The current guidance is to **remove `[Serializable]`, the serialization constructor, and `GetObjectData`** from new custom exceptions unless compatibility with legacy scenarios is explicitly required; if removal is impossible (for example, in a library with strict API compatibility requirements), mark those members with `[Obsolete(DiagnosticId = "SYSLIB0051")]`.

**Beginner-Friendly Explanation**
Serialization is **turning an object into bytes** so it can be stored or sent across boundaries. Older .NET versions required exceptions to be serializable for remoting. Modern .NET **does not support binary serialization for exceptions** and considers the related APIs obsolete. For new custom exceptions, **you do not need to add `[Serializable]` or write serialization constructors**.

---

### Purposes (Historical and Modern)

- **Historical** — To allow exceptions to cross remoting and AppDomain boundaries.
- **Historical** — To support binary serialization of exception graphs.
- **Modern** — Generally **not needed**; modern applications use JSON/XML serialization of error DTOs, not binary exception serialization.
- **Modern** — If compatibility is required, mark obsolete members to suppress warnings for consumers.

---

### Syntax Rules and Structure

```csharp
// MODERN (recommended): no serialization members
public sealed class MyException : Exception
{
    public MyException() { }
    public MyException(string message) : base(message) { }
    public MyException(string message, Exception inner) : base(message, inner) { }
}

// LEGACY (not recommended): [Serializable] + serialization members
[Serializable]
public class LegacyException : Exception
{
    public LegacyException() { }
    public LegacyException(string message) : base(message) { }
    public LegacyException(string message, Exception inner) : base(message, inner) { }

    protected LegacyException(SerializationInfo info, StreamingContext context)
        : base(info, context) { }

    public override void GetObjectData(SerializationInfo info, StreamingContext context)
    {
        base.GetObjectData(info, context);
    }
}
```

**Component breakdown**

- **Modern** — Only the three standard constructors.
- **Legacy** — `[Serializable]`, serialization constructor, `GetObjectData`.

#### Syntax Rules

- **Modern** — Omit `[Serializable]` and all serialization members.
- **Legacy** — `[Serializable]` is required for binary serialization; the serialization constructor and `GetObjectData` are part of the pattern.
- **If legacy members must remain** — Mark them `[Obsolete(DiagnosticId = "SYSLIB0051")]`.

#### Constraints and Limitations

- Binary serialization is **unsafe** and has been removed from modern .NET.
- The `SYSLIB0051` warning is produced when using the obsolete serialization APIs.
- Cross-targeting .NET Framework and .NET 8+ may require conditional compilation (`#if NET8_0_OR_GREATER`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Modern Custom Exception (No Serialization)

```csharp
using System;

public sealed class ModernException : Exception
{
    public ModernException() { }
    public ModernException(string message) : base(message) { }
    public ModernException(string message, Exception inner) : base(message, inner) { }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new ModernException("Modern exception, no serialization.");
        }
        catch (ModernException ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
}
```

**Expected Output**
```
Modern exception, no serialization.
```

**Why this output occurs**
The exception follows modern guidance: it omits `[Serializable]` and serialization members, and works correctly for normal exception handling.

---

#### Example: Legacy Serialization Members Marked Obsolete

```csharp
using System;
using System.Runtime.Serialization;

[Serializable]
public sealed class LegacyException : Exception
{
    public LegacyException() { }
    public LegacyException(string message) : base(message) { }
    public LegacyException(string message, Exception inner) : base(message, inner) { }

    [Obsolete(DiagnosticId = "SYSLIB0051")]
    private LegacyException(SerializationInfo info, StreamingContext context)
        : base(info, context) { }
}
```

**Expected Output (conceptual)**
```
// Compiles with SYSLIB0051 warnings only when the obsolete members are used.
```

**Why this pattern is used**
It maintains binary compatibility with legacy consumers while signalling that the serialization members are obsolete.

---

### Real-World Cases

- **Modern .NET services** — No serialization members on custom exceptions.
- **Cross-targeted libraries** — Conditional compilation to support .NET Framework and .NET 8+.
- **Legacy remoting** — Only where remoting is still in use (rare).

---

### References

- SYSLIB0051: Legacy serialization support APIs are obsolete — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/syslib-diagnostics/syslib0051
- Designing Custom Exceptions (serialization guidance, historical) — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-3.0/ms229064(v=vs.85)

---

## 5. Custom Exception Best Practices

### Definitions

**Core Definition**
Custom exception best practices are the design guidelines that ensure custom exceptions are **correctly named, correctly structured, safely serialized (or not), and appropriately used**.

**Technical Definition**
Best practices derive from the .NET Framework Design Guidelines and the .NET documentation. Key recommendations include: **derive from `System.Exception` (not `ApplicationException`)**; **end names with `Exception`**; **provide the three standard constructors**; **avoid deep hierarchies**; **consider providing properties for programmatic access to extra information**; **omit obsolete serialization members in modern .NET**; **do not expose sensitive information**; and **use custom exceptions only when predefined exceptions are insufficient**.

**Beginner-Friendly Explanation**
The best practices are: **name it right**, **give it the three constructors**, **keep it flat**, **add useful properties**, **don’t bother with serialization**, and **don’t overuse custom exceptions**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Derive from Exception (not ApplicationException)
public sealed class OrderException : Exception { ... }

// 2. End with Exception
public sealed class OrderAlreadyShippedException : Exception { ... }

// 3. Provide three standard constructors
public OrderException() { }
public OrderException(string message) : base(message) { }
public OrderException(string message, Exception inner) : base(message, inner) { }

// 4. Add domain-specific properties (read-only)
public string OrderId { get; }

// 5. Do not add obsolete serialization members
// Omit [Serializable], SerializationInfo constructor, GetObjectData.

// 6. Use custom exceptions only when predefined ones are insufficient
// Prefer ArgumentException, InvalidOperationException, etc., when they fit.

// 7. Avoid deep hierarchies
// Keep exception types flat and specific.
```

**Component breakdown**

- **Correct base** — `System.Exception` or a semantically appropriate standard base.
- **Naming** — Suffix `Exception`.
- **Constructors** — Three standard + optional domain-specific.
- **Properties** — Read-only data for programmatic access.
- **No serialization** — Omit obsolete members.
- **Appropriate use** — Only when predefined exceptions do not suffice.

#### Rules and Constraints

- **Do** derive from `System.Exception`, not `ApplicationException`.
- **Do** end names with `Exception`.
- **Do** provide the three standard constructors.
- **Do** consider adding domain-specific properties.
- **Do not** add `[Serializable]` or serialization members in modern .NET.
- **Do not** create deep exception hierarchies.
- **Do not** expose sensitive information in messages or properties.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Custom Exception

```csharp
using System;

public sealed class InsufficientFundsException : Exception
{
    public decimal Available { get; }
    public decimal Required { get; }

    public InsufficientFundsException() { }
    public InsufficientFundsException(string message) : base(message) { }
    public InsufficientFundsException(string message, Exception inner) : base(message, inner) { }

    public InsufficientFundsException(string message, decimal available, decimal required)
        : this(message)
    {
        Available = available;
        Required = required;
    }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new InsufficientFundsException(
                "Insufficient funds for withdrawal.",
                available: 50m,
                required: 100m);
        }
        catch (InsufficientFundsException ex)
        {
            Console.WriteLine($"Available: {ex.Available:C}");
            Console.WriteLine($"Required:  {ex.Required:C}");
            Console.WriteLine($"Message:   {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Available: $50.00
Required:  $100.00
Message:   Insufficient funds for withdrawal.
```

**Why this output occurs**
The exception follows all best practices: it derives from `Exception`, ends with `Exception`, provides the three standard constructors, adds a domain-specific constructor and read-only properties, and omits serialization members.

---

### Real-World Cases

- **Banking** — `InsufficientFundsException` with available/required amounts.
- **Inventory** — `OutOfStockException` with SKU and requested quantity.
- **Education** — `StudentNotFoundException` with student name or ID.
- **Workflow** — `WorkflowStateException` with current and expected states.

---

### References

- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- Designing Custom Exceptions — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-3.0/ms229064(v=vs.85)
- CA1032: Implement standard exception constructors — .NET Code Analysis — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1032
- CA1064: Exceptions should be public — .NET Code Analysis — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1064

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `[Serializable]` on custom exceptions | **Obsolete** for new code in .NET 8+; produces `SYSLIB0051` when serialization APIs are used. |
| Serialization constructor `(SerializationInfo, StreamingContext)` | **Obsolete** in .NET 8+; remove or mark `[Obsolete(DiagnosticId = "SYSLIB0051")]`. |
| `GetObjectData` override | **Obsolete** in .NET 8+; same guidance as serialization constructor. |
| `System.ApplicationException` | **Deprecated** as a base for custom exceptions; derive from `System.Exception` or a standard base. |
| Parameterless constructor | **Required** for serialization (legacy) and reflection-based instantiation. |
| Deep exception hierarchies | **Discouraged**; keep hierarchies flat. |
| Sensitive information in exceptions | **Security risk**; do not expose secrets, keys, or personal data. |

---

## References (Consolidated)

- How to: Create User-Defined Exceptions — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
- Designing Custom Exceptions — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-3.0/ms229064(v=vs.85)
- SYSLIB0051: Legacy serialization support APIs are obsolete — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/fundamentals/syslib-diagnostics/syslib0051
- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- CA1032: Implement standard exception constructors — .NET Code Analysis — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1032
- CA1064: Exceptions should be public — .NET Code Analysis — https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1064
- MA0013: Types should not extend System.ApplicationException — https://github.com
- How to create user-defined exceptions with localized exception messages — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-localized-exception-messages

---