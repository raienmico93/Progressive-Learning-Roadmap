# C# Exceptions — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An **exception** is an object that represents an error or unexpected condition that disrupts the normal flow of a program. C# uses exceptions for a structured, uniform, and type-safe way of handling both system-level and application-level error conditions. When an error occurs, an exception object is **thrown**, which unwinds the call stack until a matching **catch** clause handles it or the program terminates.

**Technical Definition**
Exceptions in C# provide a structured, uniform, and type-safe way of handling both system-level and application-level error conditions. Exceptions can be thrown in two ways: by an explicit `throw` statement, or by the runtime when an exceptional condition (such as division by zero or an out-of-range array index) prevents an operation from completing normally. The `System.Exception` class is the base type of all exceptions. When an exception is thrown, the system searches for the nearest enclosing `catch` clause whose named exception type is the same as, or a base class of, the runtime type of the thrown exception. All exception types inherit from `System.Exception` through a specific class hierarchy; for example, `InvalidCastException` inherits from `SystemException`, which inherits from `Exception`, which inherits from `Object`.

**Beginner-Friendly Explanation**
An exception is a **structured error report**. Instead of returning error codes or silently failing, a method that encounters a problem creates an exception object and **throws** it. The runtime then searches up the call stack for a **catch** block that knows how to handle that kind of error. This separates the code that detects the error from the code that recovers from it, and ensures that errors cannot be accidentally ignored. Exceptions are **type-safe**: the catch block specifies which exception type it handles, and the runtime matches on the actual type. `System.Exception` is the root of the entire hierarchy—every exception is an `Exception`.

---

### Key Characteristics

- **Structured error handling** — Separates error detection from error recovery.
- **Type-safe** — Exceptions are objects; catch clauses match on type.
- **Hierarchical** — All exceptions derive from `System.Exception`; specific types derive from general ones.
- **Rich metadata** — Every exception carries a `Message`, `StackTrace`, `InnerException`, and more.
- **Unwinding** — When thrown, an exception unwinds the stack until a matching handler is found.
- **`try`/`catch`/`finally`** — `try` guards code; `catch` handles exceptions; `finally` cleans up resources.
- **Two base branches** — `SystemException` (CLR-thrown) and `ApplicationException` (legacy application-thrown; now discouraged as a base).

---

### Prerequisites

- Basic understanding of C# **classes**, **inheritance**, and **namespaces**.
- Familiarity with **methods**, **call stacks**, and **object lifetimes**.
- Knowledge of `try`/`catch`/`finally` syntax.
- Awareness of `IDisposable` and `using` statements (for resource cleanup).
- Basic understanding of **asynchronous programming** (`Task`, `await`) for async exception handling.

---

### Related Programming Areas

- **Asynchronous programming** — `Task` and `await` propagate exceptions differently (`AggregateException`, `TaskCanceledException`).
- **Resource management** — `using` and `finally` ensure cleanup even when exceptions occur.
- **Framework design** — The choice of exception types (argument, state, not-supported) is part of API design.
- **Logging and diagnostics** — Exception properties (`Message`, `StackTrace`, `InnerException`) are the primary diagnostic tools.
- **Security** — Exception messages and stack traces can leak sensitive information if exposed to end users.

---

### Core Concepts / Features

1. Exceptions (Overarching Concept)
2. `Exception` Class
3. Exception Hierarchy (`SystemException`, `ApplicationException`, and common derived types)
4. Exception Properties (`Message`, `StackTrace`, `InnerException`, `Data`, `HelpLink`, `Source`, `TargetSite`)
5. Exception Best Practices

---

## 1. Exceptions (Overarching Concept)

### Definitions

**Core Definition**
An exception is an object representing an error condition that disrupts normal program flow.

**Technical Definition**
Exceptions in C# provide a structured, uniform, and type-safe way of handling both system-level and application-level error conditions. They are thrown either explicitly via `throw` or implicitly by the runtime when an operation cannot complete. The runtime searches for the nearest `catch` clause whose type is compatible with the thrown exception’s runtime type.

**Beginner-Friendly Explanation**
An exception is **an error object that interrupts normal execution** and is handled by a matching `catch` block.

---

### Purposes

- To **signal errors** without relying on return codes.
- To **separate error detection** from error recovery.
- To **enforce handling** (unhandled exceptions terminate the process).
- To carry **rich diagnostic information** (`Message`, `StackTrace`, `InnerException`).
- To enable **centralized error handling** at appropriate boundaries.

---

### Syntax Rules and Structure

```csharp
try
{
    // code that may throw
}
catch (SpecificException ex)
{
    // handle specific exception
}
catch (Exception ex)
{
    // handle broader exception
}
finally
{
    // cleanup (always runs)
}
```

**Component breakdown**

- `try` — Guards code that may throw.
- `catch (T ex)` — Handles exceptions of type `T` or derived from `T`.
- `finally` — Runs regardless of whether an exception occurred.

#### Syntax Rules

- `catch` clauses are evaluated **top to bottom**; place more derived exceptions first.
- A `catch` without a type catches all exceptions.
- `finally` runs even if an exception is thrown or a `return` executes.
- Exceptions propagate up the call stack until a matching handler is found.

#### Constraints and Limitations

- Unhandled exceptions terminate the process.
- Catching `Exception` broadly can hide bugs; catch specific types when possible.
- Exception handling has performance overhead; do not use it for normal control flow.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Try/Catch/Finally

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int result = 10 / int.Parse("0");   // DivideByZeroException
        }
        catch (DivideByZeroException ex)
        {
            Console.WriteLine($"Caught: {ex.Message}");
        }
        finally
        {
            Console.WriteLine("Finally runs");
        }
    }
}
```

**Expected Output**
```
Caught: Attempted to divide by zero.
Finally runs
```

**Why this output occurs**
`int.Parse("0")` returns `0`; `10 / 0` throws `DivideByZeroException`, which is caught; `finally` always runs.

---

### Real-World Cases

- **API boundaries** — Translating low-level exceptions into domain-specific ones.
- **Resource cleanup** — `finally` and `using` for files, connections.
- **Async operations** — `await` propagates exceptions from tasks.

---

### References

- Exceptions — C# language specification — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/exceptions
- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions

---

## 2. `Exception` Class

### Definitions

**Core Definition**
`System.Exception` is the **base class of all exceptions** in .NET; every exception object is an instance of `Exception` or a derived type.

**Technical Definition**
The `System.Exception` class is the base type of all exceptions. It provides the core properties that all exceptions share: `Message`, `InnerException`, `StackTrace`, `Source`, `TargetSite`, `Data`, `HelpLink`, and `HResult`. Concrete exception types typically inherit from `Exception` or from `SystemException` (for runtime-thrown exceptions) and add no additional functionality; they exist to provide **type identity** for catching.

**Beginner-Friendly Explanation**
`Exception` is the **root of all errors**. Every catchable error is an `Exception` (or a subclass), so catching `Exception` catches everything.

---

### Purposes

- To provide a **common base** for all exception types.
- To expose **diagnostic properties** (`Message`, `StackTrace`, `InnerException`).
- To enable **hierarchical catching** (catch a base type to handle all derived types).
- To support **exception chaining** via `InnerException`.

---

### Syntax Rules and Structure

```csharp
public class MyException : Exception
{
    public MyException() { }
    public MyException(string message) : base(message) { }
    public MyException(string message, Exception inner) : base(message, inner) { }
}
```

**Component breakdown**

- `: Exception` — Inheritance.
- Constructors — Standard `message` and `innerException` overloads.

#### Syntax Rules

- Custom exceptions should inherit from `Exception` (not `ApplicationException`).
- Provide at least the three standard constructors: `()`, `(string)`, `(string, Exception)`.
- The `Message` and `InnerException` properties are read-only.

#### Constraints and Limitations

- Do not throw or catch `System.Exception` or `System.SystemException` directly in framework code, unless rethrowing or at a top-level handler.
- `Exception` is not sealed; custom exceptions derive from it.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Exception

```csharp
using System;

public class OrderNotFoundException : Exception
{
    public OrderNotFoundException() { }
    public OrderNotFoundException(string message) : base(message) { }
    public OrderNotFoundException(string message, Exception inner) : base(message, inner) { }
}

class Program
{
    static void Main()
    {
        try
        {
            throw new OrderNotFoundException("Order 42 not found");
        }
        catch (OrderNotFoundException ex)
        {
            Console.WriteLine(ex.Message);   // Order 42 not found
        }
    }
}
```

**Expected Output**
```
Order 42 not found
```

**Why this output occurs**
The custom exception derives from `Exception` and carries a descriptive message.

---

### Real-World Cases

- **Domain exceptions** — `OrderNotFoundException`, `PaymentFailedException`.
- **API exceptions** — Wrapping low-level errors with domain-specific types.

---

### References

- Exception Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.exception
- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions

---

## 3. Exception Hierarchy

### Definitions

**Core Definition**
The exception hierarchy is the **inheritance tree** of exception types, rooted at `System.Exception`, with two main branches (`SystemException` and `ApplicationException`) and many specific derived types for common error conditions.

**Technical Definition**
All exceptions derive from `System.Exception`. The two direct derived branches are `SystemException` (thrown by the CLR for runtime errors) and `ApplicationException` (originally intended for application-thrown exceptions; now **discouraged** as a base). `SystemException` is the parent of most runtime exceptions, including `NullReferenceException`, `IndexOutOfRangeException`, `InvalidCastException`, `ArgumentException` (and its subtypes), `InvalidOperationException`, `NotSupportedException`, `IOException` (and its subtypes), `TimeoutException`, `OperationCanceledException`, and `AggregateException`.

**Beginner-Friendly Explanation**
The hierarchy is a **family tree of errors**. At the top is `Exception`; below it are broad categories (`SystemException`) and below those are specific errors (`NullReferenceException`, `ArgumentException`, etc.). Catching a base type catches all its descendants.

---

### Common Exception Types

#### `SystemException`

**Definitions**

- **Core:** The base class for exceptions thrown by the CLR.
- **Technical:** `System.SystemException` is the base for runtime-thrown exceptions. Most runtime exceptions derive from it.
- **Beginner:** The category for “system-level” errors.

**Purposes**

- To group CLR-thrown exceptions.
- To allow catching runtime errors broadly.

**Constraints:** Do **not** catch `SystemException` broadly; catch specific types.

---

#### `ApplicationException`

**Definitions**

- **Core:** A legacy base class originally intended for application-thrown exceptions.
- **Technical:** `System.ApplicationException` was designed to distinguish application errors from system errors, but this distinction has **not proven valuable**; modern guidance is to derive custom exceptions from `Exception`, not `ApplicationException`.
- **Beginner:** A deprecated base class for custom exceptions.

**Constraints:** **Do not** throw or derive from `ApplicationException`.

---

#### `ArgumentException`

**Definitions**

- **Core:** Thrown when a non-null argument passed to a method is invalid.
- **Technical:** `System.ArgumentException` is the base for argument-related exceptions. Its primary derived types are `ArgumentNullException` (null argument) and `ArgumentOutOfRangeException` (argument outside valid range).
- **Beginner:** “You gave me a bad argument.”

**Purposes**

- To signal invalid method arguments.
- To distinguish null vs. out-of-range via subtypes.

**Syntax Rules**

- Set `ParamName` when throwing.
- Prefer the most-derived subtype applicable.

**Annotated Example**

```csharp
public void SetAge(int age)
{
    if (age < 0) throw new ArgumentOutOfRangeException(nameof(age), "Age cannot be negative");
}
```

**References:** ArgumentException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.argumentexception

---

#### `ArgumentNullException`

**Definitions**

- **Core:** Thrown when a `null` argument is passed to a method that does not accept it.
- **Technical:** A subclass of `ArgumentException`.
- **Beginner:** “You passed null where null is not allowed.”

**Annotated Example**

```csharp
public void SetName(string name)
{
    if (name is null) throw new ArgumentNullException(nameof(name));
}
```

**References:** ArgumentNullException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.argumentnullexception

---

#### `ArgumentOutOfRangeException`

**Definitions**

- **Core:** Thrown when an argument is outside the valid range of values.
- **Technical:** A subclass of `ArgumentException`.
- **Beginner:** “Your argument is too big or too small.”

**Annotated Example**

```csharp
public void SetIndex(int index)
{
    if (index < 0 || index >= 10) throw new ArgumentOutOfRangeException(nameof(index));
}
```

**References:** ArgumentOutOfRangeException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception

---

#### `InvalidOperationException`

**Definitions**

- **Core:** Thrown when a method call or property set is inappropriate for the object’s current state.
- **Technical:** `System.InvalidOperationException` signals that the operation is invalid given the object’s state (e.g., writing to a read-only stream).
- **Beginner:** “You can’t do that right now.”

**Annotated Example**

```csharp
public void Close()
{
    if (_state == State.Closed) throw new InvalidOperationException("Already closed");
}
```

**References:** InvalidOperationException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.invalidoperationexception

---

#### `NotSupportedException`

**Definitions**

- **Core:** Thrown when a method is not supported by an implementation (e.g., a derived class does not support an inherited method).
- **Technical:** Signals that the operation is intentionally not implemented in this implementation.
- **Beginner:** “This feature isn’t available here.”

**References:** NotSupportedException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.notsupportedexception

---

#### `NotImplementedException`

**Definitions**

- **Core:** Thrown when a method or accessor is not yet implemented.
- **Technical:** Indicates incomplete code, typically a placeholder.
- **Beginner:** “I haven’t written this yet.”

**Constraints:** Use `NotSupportedException` if the member is intentionally unsupported.

**References:** NotImplementedException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.notimplementedexception

---

#### `NullReferenceException`

**Definitions**

- **Core:** Thrown when dereferencing a `null` reference.
- **Technical:** The runtime throws this when a member is accessed on a `null` object.
- **Beginner:** “You tried to use something that isn’t there.”

**References:** NullReferenceException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.nullreferenceexception

---

#### `IndexOutOfRangeException`

**Definitions**

- **Core:** Thrown when an array index is outside the bounds of the array.
- **Technical:** Thrown when an index is less than zero or greater than or equal to the array length.
- **Beginner:** “Your index is out of bounds.”

**References:** IndexOutOfRangeException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.indexoutofrangeexception

---

#### `KeyNotFoundException`

**Definitions**

- **Core:** Thrown when a key is not found in a dictionary or collection.
- **Technical:** Thrown when the key specified for accessing an element does not exist in the collection.
- **Beginner:** “That key isn’t in the dictionary.”

**References:** KeyNotFoundException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.keynotfoundexception

---

#### `FormatException`

**Definitions**

- **Core:** Thrown when the format of an argument is invalid (e.g., parsing a non-numeric string).
- **Technical:** Signals that a string is not in the correct format for conversion.
- **Beginner:** “The text isn’t in the right format.”

**References:** FormatException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.formatexception

---

#### `OverflowException`

**Definitions**

- **Core:** Thrown when an arithmetic operation overflows the target type.
- **Technical:** Signals that a value is outside the range of the destination type.
- **Beginner:** “The number is too big.”

**References:** OverflowException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.overflowexception

---

#### `DivideByZeroException`

**Definitions**

- **Core:** Thrown when dividing an integral value by zero.
- **Technical:** Thrown on integer or decimal division by zero.
- **Beginner:** “You can’t divide by zero.”

**References:** DivideByZeroException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.dividebyzeroexception

---

#### `IOException`

**Definitions**

- **Core:** The base class for I/O-related exceptions.
- **Technical:** Base for `FileNotFoundException`, `DirectoryNotFoundException`, `DriveNotFoundException`, `PathTooLongException`, etc..
- **Beginner:** “Something went wrong with input/output.”

**References:** IOException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.io.ioexception

---

#### `FileNotFoundException`

**Definitions**

- **Core:** Thrown when a file cannot be found.
- **Technical:** A subclass of `IOException`.
- **Beginner:** “The file doesn’t exist.”

**References:** FileNotFoundException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.io.filenotfoundexception

---

#### `DirectoryNotFoundException`

**Definitions**

- **Core:** Thrown when a directory cannot be found.
- **Technical:** A subclass of `IOException`.
- **Beginner:** “The folder doesn’t exist.”

**References:** DirectoryNotFoundException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.io.directorynotfoundexception

---

#### `UnauthorizedAccessException`

**Definitions**

- **Core:** Thrown when the OS denies access to a resource.
- **Technical:** Signals insufficient permissions.
- **Beginner:** “You don’t have permission.”

**References:** UnauthorizedAccessException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.unauthorizedaccessexception

---

#### `TimeoutException`

**Definitions**

- **Core:** Thrown when an operation times out.
- **Technical:** Signals that a time limit was exceeded.
- **Beginner:** “It took too long.”

**References:** TimeoutException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.timeoutexception

---

#### `OperationCanceledException`

**Definitions**

- **Core:** Thrown when an operation is canceled via a `CancellationToken`.
- **Technical:** The standard exception for cooperative cancellation.
- **Beginner:** “The operation was canceled.”

**References:** OperationCanceledException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.operationcanceledexception

---

#### `TaskCanceledException`

**Definitions**

- **Core:** A subclass of `OperationCanceledException` thrown when a `Task` is canceled.
- **Technical:** Inherits from `OperationCanceledException`.
- **Beginner:** “The task was canceled.”

**References:** TaskCanceledException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcanceledexception

---

#### `AggregateException`

**Definitions**

- **Core:** Wraps multiple exceptions thrown by parallel or asynchronous operations.
- **Technical:** Contains an `InnerExceptions` collection; thrown by `Task.Wait`, `Task.Result`, and `Task.WhenAll`.
- **Beginner:** “Multiple errors happened at once.”

**Annotated Example**

```csharp
try
{
    Task.WaitAll(task1, task2);
}
catch (AggregateException ex)
{
    foreach (var inner in ex.InnerExceptions)
        Console.WriteLine(inner.Message);
}
```

**References:** AggregateException — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.aggregateexception

---

## 4. Exception Properties

### Definitions

**Core Definition**
Exception properties are the **diagnostic members** on `System.Exception` that describe the error, its cause, its location, and related metadata.

**Technical Definition**
`System.Exception` exposes: `Message` (read-only human-readable description), `InnerException` (read-only reference to the causing exception), `StackTrace` (read-only string with the call stack), `Source` (read/write name of the application/object), `TargetSite` (read-only `MethodBase` of the throwing method), `Data` (read/write `IDictionary` of arbitrary key-value pairs), `HelpLink` (read/write URL/URN to help), and `HResult` (read-only numeric code).

**Beginner-Friendly Explanation**
The properties are **the details of the error**: what happened (`Message`), where (`StackTrace`, `Source`, `TargetSite`), why (`InnerException`), and extra data (`Data`, `HelpLink`).

---

### Property Breakdown

| Property | Type | Description |
|---|---|---|
| `Message` | `string` | Human-readable description |
| `StackTrace` | `string` | Call stack at throw point |
| `InnerException` | `Exception?` | The exception that caused this one |
| `Data` | `IDictionary` | Arbitrary key-value data |
| `HelpLink` | `string?` | URL/URN to a help file |
| `Source` | `string?` | Name of app/object that threw |
| `TargetSite` | `MethodBase?` | Method that threw the exception |

---

### Annotated Code Examples and Expected Outputs

#### Example: Inspecting Exception Properties

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            throw new InvalidOperationException("Something failed",
                new ArgumentException("Bad argument"));
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Message: {ex.Message}");
            Console.WriteLine($"Inner: {ex.InnerException?.Message}");
            Console.WriteLine($"Source: {ex.Source}");
            Console.WriteLine($"TargetSite: {ex.TargetSite?.Name}");
        }
    }
}
```

**Expected Output (approximate)**
```
Message: Something failed
Inner: Bad argument
Source: MyApp
TargetSite: Main
```

**Why this output occurs**
`Message` returns the description; `InnerException` returns the wrapped cause; `Source` and `TargetSite` identify the throwing context.

---

### Real-World Cases

- **Logging** — `Message`, `StackTrace`, and `InnerException` are logged for diagnosis.
- **API responses** — Only `Message` is exposed to end users (to avoid leaking stack traces).
- **Exception chaining** — `InnerException` preserves the original cause.

---

### References

- Exception Class — .NET API (properties) — https://learn.microsoft.com/en-us/dotnet/api/system.exception
- Exercise - Catch specific exception types (properties) — https://learn.microsoft.com/en-us/training/modules/implement-exception-handling-c-sharp/7-exercise-catch-explicit-exception-types

---

## 5. Exception Best Practices

### Definitions

**Core Definition**
Exception best practices are the guidelines for **throwing, catching, and designing exceptions** so that error handling is reliable, diagnosable, and maintainable.

**Technical Definition**
Best practices derive from the .NET documentation and Framework Design Guidelines. Key recommendations include: **use `try`/`catch`/`finally` to recover from errors or release resources**; **handle common conditions to avoid exceptions** (e.g., check state before closing); **catch specific exceptions and order from most derived to least derived**; **do not catch `Exception` or `SystemException` broadly** unless rethrowing or at a top-level handler; **derive custom exceptions from `Exception`, not `ApplicationException`**; **throw `ArgumentException` subtypes for bad arguments** and set `ParamName`; **throw `InvalidOperationException` for invalid object state**; **use `NotSupportedException` for intentionally unsupported members** and `NotImplementedException` only for incomplete code; **preserve the original stack trace when rethrowing** (use `throw;` not `throw ex;`)**; **clean up with `using` or `finally`**; and **do not expose sensitive information in exception messages**.

**Beginner-Friendly Explanation**
The best practices are: **catch what you can handle**, **catch specific types**, **don’t swallow exceptions**, **clean up resources**, **use the right exception type**, and **preserve stack traces**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Catch specific exceptions, derived first
try { ... }
catch (ArgumentNullException ex) { ... }
catch (ArgumentException ex) { ... }

// 2. Preserve stack trace when rethrowing
try { ... }
catch (Exception ex)
{
    // log ex
    throw;   // NOT throw ex;
}

// 3. Use using or finally for cleanup
using var stream = File.OpenRead("file.txt");

// 4. Throw the right type
if (arg is null) throw new ArgumentNullException(nameof(arg));
if (state == State.Closed) throw new InvalidOperationException("Already closed");
if (!Supported) throw new NotSupportedException("This feature is not supported");

// 5. Avoid catching Exception broadly
// Only catch at top-level handlers or when rethrowing.

// 6. Handle common conditions without exceptions
if (!connection.IsClosed) connection.Close();
```

**Component breakdown**

- **Specific catches** — Handle only what you can recover from.
- **`throw;`** — Preserves the original stack trace.
- **`using`/`finally`** — Guarantees resource cleanup.
- **Correct exception types** — Communicate the nature of the error.
- **Avoid broad catches** — Prevent hiding bugs.

#### Rules and Constraints

- **Do** catch specific exceptions.
- **Do** order catch clauses from most derived to least derived.
- **Do** use `throw;` to preserve the stack trace.
- **Do** clean up resources with `using` or `finally`.
- **Do** throw `ArgumentException` subtypes for invalid arguments.
- **Do not** catch `Exception` or `SystemException` broadly without rethrowing.
- **Do not** derive custom exceptions from `ApplicationException`.
- **Do not** use exceptions for normal control flow.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Handled Exception

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        try
        {
            using var reader = new StreamReader("missing.txt");
            Console.WriteLine(reader.ReadToEnd());
        }
        catch (FileNotFoundException ex)
        {
            Console.WriteLine($"File missing: {ex.FileName}");
        }
        catch (IOException ex)
        {
            Console.WriteLine($"I/O error: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
File missing: missing.txt
```

**Why this output occurs**
`FileNotFoundException` is caught specifically (and before `IOException`), and `using` guarantees the reader is disposed.

---

### Real-World Cases

- **API boundaries** — Catch low-level exceptions and wrap them in domain exceptions.
- **Top-level handlers** — Catch `Exception` only at the application boundary (e.g., `Main`, middleware).
- **Async code** — Handle `AggregateException` and `TaskCanceledException` appropriately.

---

### References

- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- Catching and throwing standard exception types — Framework Design Guidelines — https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/using-standard-exception-types
- Exceptions — C# language specification — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/exceptions

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `ApplicationException` | **Discouraged** as a base for custom exceptions. |
| Catching `Exception`/`SystemException` broadly | **Discouraged** except in top-level handlers or when rethrowing. |
| `throw ex;` | **Resets the stack trace**; use `throw;` to preserve it. |
| `NullReferenceException` | Indicates a bug; avoid catching it—fix the null dereference instead. |
| `NotImplementedException` | Use only for **incomplete code**, not for intentionally unsupported members (use `NotSupportedException`). |
| `AggregateException` | Thrown by `Task.Wait`/`Task.Result`; `await` unwraps to the first inner exception. |
| `TaskCanceledException` | Inherits from `OperationCanceledException`; thrown when a task is canceled. |
| Exception messages and stack traces | **Security-sensitive**; do not expose to end users. |

---

## References (Consolidated)

- Exceptions — C# language specification — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/exceptions
- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- Catching and throwing standard exception types — Framework Design Guidelines — https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/using-standard-exception-types
- Exception Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.exception
- Exercise - Catch specific exception types — https://learn.microsoft.com/en-us/training/modules/implement-exception-handling-c-sharp/7-exercise-catch-explicit-exception-types
- Exception hierarchy — Microsoft Learn (legacy) — https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-1.1/z4c5tckx(v=vs.80)

---