# C# Try-Catch-Finally — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
`try`, `catch`, and `finally` are the three fundamental keywords of C#'s structured exception handling mechanism. Together they allow a program to **guard a region of code** (`try`), **respond to specific error conditions** (`catch`), and **guarantee cleanup** regardless of whether an error occurred (`finally`). This model separates the code that detects an error from the code that recovers from it, while ensuring that critical resources are released.

**Technical Definition**
The C# language provides exception handling through `try` statements. A `try` block contains guarded code that may throw an exception. One or more `catch` clauses specify handlers for specific exception types; when an exception is thrown, the runtime searches for the nearest enclosing `catch` clause whose type is compatible with the thrown exception's runtime type. A `finally` block, if present, contains cleanup code that executes regardless of whether an exception was thrown, caught, or propagated. Exception filters (`when` clauses) add a Boolean condition to a `catch` clause, allowing handlers to be selected based on runtime state without unwinding the stack. The `throw` statement raises an exception; `throw;` (rethrow) preserves the original stack trace, while `throw ex;` resets it. All of these constructs are defined in the C# language specification and are foundational to reliable .NET applications.

**Beginner-Friendly Explanation**
`try`/`catch`/`finally` is the **safety net** of your program. You put risky code inside `try`. If something goes wrong, the runtime jumps to a `catch` block that knows how to handle that kind of error. Whether things go right or wrong, the `finally` block always runs—so you can close files, release locks, or clean up anything that must be cleaned up. It is like wearing a seatbelt: the `try` block is the drive, `catch` is the airbag, and `finally` is the emergency exit that always opens.

---

### Key Characteristics

- **Structured error handling** — Separates detection (`throw`) from recovery (`catch`).
- **Three-part composition** — `try` (guard), `catch` (handle), `finally` (cleanup).
- **Type-based dispatch** — `catch` clauses match on exception type; order from most derived to least derived is required.
- **Exception filters** — `when` clauses add conditions to `catch` without stack unwinding.
- **Guaranteed cleanup** — `finally` runs even on `return`, `break`, `continue`, or unhandled exceptions.
- **Stack-trace preservation** — `throw;` preserves the original stack trace; `throw ex;` resets it.
- **Performance considerations** — `try`/`catch` has minimal cost when no exception is thrown; the cost is incurred when an exception is actually thrown and caught.

---

### Prerequisites

- Basic understanding of C# **methods**, **statements**, and **control flow**.
- Familiarity with **exceptions** and the `System.Exception` hierarchy.
- Knowledge of **resource management** (`IDisposable`, `using`).
- Awareness of **asynchronous programming** (`Task`, `await`) for async exception handling.
- Understanding of **stack unwinding** and call stacks.

---

### Related Programming Areas

- **Exception hierarchy** — Choosing the right exception type to catch.
- **Resource management** — `using` and `finally` for deterministic cleanup.
- **Asynchronous programming** — `await` unwraps exceptions differently from `Task.Wait`.
- **Logging and diagnostics** — Capturing `Message`, `StackTrace`, and `InnerException`.
- **Security** — Avoiding sensitive information leakage in exception messages.

---

### Core Concepts / Features

1. `try` Block
2. `catch` Clause
3. `finally` Block
4. Multiple `catch` Blocks
5. Exception Filters (`when` Clause)
6. Rethrowing Exceptions
7. `throw` vs. `throw ex`
8. Exception Handling Best Practices

---

## 1. `try` Block

### Definitions

**Core Definition**
A `try` block encloses code that may throw an exception and that the program is prepared to handle.

**Technical Definition**
The `try` block defines a guarded region. If an exception is thrown inside the block, control transfers to the first matching `catch` clause associated with the `try`. If no `catch` clause matches, the exception propagates to the next enclosing `try` statement or terminates the process. A `try` block must be followed by at least one `catch` or `finally` block (or both).

**Beginner-Friendly Explanation**
The `try` block is the **risky part** of your code. It tells the runtime: “Watch this code—if it explodes, I have a plan.”

---

### Purposes

- To **identify** code that may throw exceptions.
- To **scope** exception handling to a specific region.
- To **enable** `catch` and `finally` clauses to operate on the guarded code.
- To **separate** normal logic from error handling.

---

### Syntax Rules and Structure

```csharp
try
{
    // guarded code that may throw
}
```

**Component breakdown**

- `try` — The keyword.
- `{ ... }` — The guarded block (cannot be empty in a useful sense; must contain the code to protect).

#### Syntax Rules

- A `try` block must be followed by one or more `catch` blocks, or a `finally` block, or both.
- A `try` block cannot appear alone.
- The `try` block can contain any statements, including nested `try` blocks.

#### Constraints and Limitations

- A `try` block alone is not a complete statement; the compiler requires a `catch` or `finally`.
- Exceptions thrown in a `try` block are caught only by matching `catch` clauses.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic `try` with `catch`

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int result = 10 / int.Parse("0");   // throws DivideByZeroException
            Console.WriteLine(result);
        }
        catch (DivideByZeroException ex)
        {
            Console.WriteLine($"Caught: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Caught: Attempted to divide by zero.
```

**Why this output occurs**
`int.Parse("0")` returns `0`; `10 / 0` throws `DivideByZeroException`, which is caught by the matching `catch` clause.

---

### Real-World Cases

- **Parsing user input** — Wrapping `int.Parse` in `try` to handle invalid formats.
- **File operations** — Guarding `File.ReadAllText` against missing files.
- **Network calls** — Guarding `HttpClient.GetStringAsync` against timeouts and network errors.

---

### References

- Exception Handling (C# Programming Guide) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/exception-handling
- try-catch (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch

---

## 2. `catch` Clause

### Definitions

**Core Definition**
A `catch` clause handles an exception that was thrown inside the associated `try` block, specifying the exception type it can handle and optionally a variable to reference the exception object.

**Technical Definition**
A `catch` clause has the form `catch (ExceptionType ex)` or `catch` (with no type, catching all exceptions). When an exception is thrown, the runtime searches the `catch` clauses of the enclosing `try` statements in order. The first clause whose exception type is the same as, or a base class of, the thrown exception's runtime type is selected. The exception object is bound to the variable `ex` (if provided), and the handler block executes.

**Beginner-Friendly Explanation**
The `catch` block is the **“what to do when things go wrong”** part. You specify which kind of error you can handle, and you get the error object to inspect.

---

### Purposes

- To **handle** specific exception types.
- To **log** or **report** errors.
- To **recover** from errors (retry, fallback, default value).
- To **translate** low-level exceptions into domain-specific ones.
- To **prevent** unhandled exceptions from crashing the process.

---

### Syntax Rules and Structure

```csharp
catch (SpecificException ex)
{
    // handle
}

catch (GeneralException ex)
{
    // handle broader
}

catch
{
    // handle any exception (no variable)
}
```

**Component breakdown**

- `catch` — The keyword.
- `(ExceptionType ex)` — Optional type and variable.
- `{ ... }` — The handler block.

#### Syntax Rules

- `catch` clauses are evaluated **top to bottom**.
- More derived exception types must appear **before** less derived types (otherwise compiler error **CS0160**).
- A `catch` without a type catches all exceptions.
- The exception variable `ex` is read-only and is scoped to the catch block.

#### Constraints and Limitations

- Catching `System.Exception` broadly can hide bugs; catch specific types when possible.
- A `catch` block that does nothing (swallows) is discouraged.

---

### Annotated Code Examples and Expected Outputs

#### Example: Specific Catch with Variable

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int[] numbers = { 1, 2, 3 };
            Console.WriteLine(numbers[5]);
        }
        catch (IndexOutOfRangeException ex)
        {
            Console.WriteLine($"Index error: {ex.Message}");
        }
    }
}
```

**Expected Output**
```
Index error: Index was outside the bounds of the array.
```

**Why this output occurs**
Accessing `numbers[5]` throws `IndexOutOfRangeException`, which is caught and its `Message` is printed.

---

### Real-World Cases

- **API error handling** — Catching `HttpRequestException` and mapping to HTTP status codes.
- **Validation** — Catching `FormatException` from parsing and returning a user-friendly message.
- **Resource cleanup** — Catching exceptions to roll back transactions before rethrowing.

---

### References

- try-catch (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch
- Compiler Error CS0160 — https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0160

---

## 3. `finally` Block

### Definitions

**Core Definition**
A `finally` block contains cleanup code that executes **regardless** of whether an exception was thrown in the `try` block, and regardless of whether the exception was caught.

**Technical Definition**
The `finally` block is executed when control leaves the `try` statement. It runs on normal completion, on `return`, `break`, `continue`, on caught exceptions, and on uncaught exceptions (before the exception propagates). If the `try` block has a `catch` clause, `finally` runs after the `catch` block. If no `catch` matches, `finally` runs before the exception propagates up the call stack. Code in a `finally` clause is almost always executed even when exceptions are thrown.

**Beginner-Friendly Explanation**
The `finally` block is the **cleanup crew**. Whether the operation succeeds or fails, `finally` runs—so you can close files, release connections, and free resources.

---

### Purposes

- To **guarantee resource cleanup**.
- To **restore state** after an exception.
- To **release locks** and close connections.
- To **log** that an operation completed (success or failure).

---

### Syntax Rules and Structure

```csharp
try
{
    // guarded code
}
finally
{
    // cleanup code — always runs
}
```

**Component breakdown**

- `finally` — The keyword.
- `{ ... }` — The cleanup block.

#### Syntax Rules

- A `try` can have at most one `finally` block.
- `finally` can be used with or without `catch` blocks.
- `finally` cannot contain `return`, `break`, or `continue` that would exit the `finally` block (the compiler disallows `return` in `finally`; `break`/`continue` are also disallowed).
- If `finally` throws an exception, it replaces the original exception.

#### Constraints and Limitations

- `finally` cannot suppress an exception (unlike `catch`).
- `finally` runs even if the `try` block returns.

---

### Annotated Code Examples and Expected Outputs

#### Example: `try`/`finally` Guarantees Cleanup

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        StreamReader? reader = null;
        try
        {
            reader = new StreamReader("data.txt");
            Console.WriteLine(reader.ReadToEnd());
        }
        finally
        {
            reader?.Dispose();
            Console.WriteLine("Cleanup done");
        }
    }
}
```

**Expected Output (if file exists)**
```
(file contents)
Cleanup done
```

**Expected Output (if file is missing)**
```
Unhandled exception: FileNotFoundException
Cleanup done
```

**Why this output occurs**
`finally` runs even when the `try` block throws (the exception is not caught here, so it propagates after `finally` runs).

---

### Real-World Cases

- **File I/O** — Closing streams.
- **Database connections** — Closing connections and disposing commands.
- **Locks** — Releasing `Monitor`/`lock` state.
- **Transaction rollback** — Ensuring rollback on failure.

---

### References

- try-finally (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-finally
- Best practices for exceptions — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions

---

## 4. Multiple `catch` Blocks

### Definitions

**Core Definition**
Multiple `catch` clauses allow a single `try` block to handle different exception types with different recovery strategies.

**Technical Definition**
A `try` statement can have multiple `catch` clauses. The runtime evaluates them in order from top to bottom. The first clause whose type is compatible with the thrown exception's runtime type is selected. Because of this, catch clauses must be ordered from **most derived to least derived**; placing a base type before a derived type makes the derived clause unreachable and causes compiler error **CS0160**.

**Beginner-Friendly Explanation**
Multiple catch blocks let you say: **“If it’s this specific error, do A; if it’s that error, do B; otherwise, do C.”**

---

### Purposes

- To **handle different exception types differently**.
- To **provide specific recovery** for known errors and **generic recovery** for others.
- To **log** with different levels of detail depending on the error.
- To **translate** specific exceptions into domain exceptions.

---

### Syntax Rules and Structure

```csharp
try
{
    // guarded code
}
catch (MostDerivedException ex)
{
    // handle most derived
}
catch (BaseException ex)
{
    // handle base
}
catch (Exception ex)
{
    // handle all remaining
}
```

**Component breakdown**

- Order matters: **most derived first, least derived last**.
- The final `catch (Exception)` acts as a catch-all.

#### Syntax Rules

- Catch clauses are evaluated in order.
- A derived exception type after a base type is unreachable and causes **CS0160**.
- A `catch` without a type can be used as the last catch-all.

#### Constraints and Limitations

- Overlapping catch blocks can hide specific handlers.
- Catching `Exception` broadly can mask unexpected errors.

---

### Annotated Code Examples and Expected Outputs

#### Example: Ordered Catch Blocks

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int result = int.Parse("abc");   // throws FormatException
        }
        catch (OverflowException ex)
        {
            Console.WriteLine("Overflow");
        }
        catch (FormatException ex)
        {
            Console.WriteLine("Format");   // this branch
        }
        catch (Exception ex)
        {
            Console.WriteLine("Other");
        }
    }
}
```

**Expected Output**
```
Format
```

**Why this output occurs**
`FormatException` is more specific than `Exception` and is placed before it; it matches the thrown exception.

---

### Real-World Cases

- **Input validation** — `FormatException` vs. `OverflowException` vs. generic.
- **I/O operations** — `FileNotFoundException` vs. `IOException` vs. generic.
- **Network operations** — `HttpRequestException` vs. `TaskCanceledException` vs. generic.

---

### References

- Compiler Error CS0160 — https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0160
- Best practices for exceptions — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions

---

## 5. Exception Filters (`when` Clause)

### Definitions

**Core Definition**
An exception filter is a `when` clause attached to a `catch` block that adds a Boolean condition. The handler executes only if the exception type matches **and** the filter expression evaluates to `true`.

**Technical Definition**
Exception filters (introduced in **C# 6.0**) use the `when` contextual keyword: `catch (ExceptionType ex) when (expr)`. The `expr` must evaluate to `bool`. If `expr` returns `true`, the handler runs; if `false`, the runtime continues searching for another matching handler. Filters are evaluated **before** the stack is unwound, so they do not incur the cost of unwinding and rethrowing. They also preserve the call stack for debugging.

**Beginner-Friendly Explanation**
An exception filter is a **bouncer for your catch block**. The catch block only lets the exception in if an additional condition is true. For example, you might only want to handle an `HttpRequestException` if the message contains “404”.

---

### Purposes

- To **conditionally handle** exceptions based on runtime state.
- To **avoid catching and rethrowing** (which resets the stack trace).
- To **improve debugging** by leaving the stack intact when a filter returns `false`.
- To **implement retry or fallback logic** based on exception details.

---

### Syntax Rules and Structure

```csharp
try
{
    // guarded code
}
catch (ExceptionType ex) when (condition)
{
    // handler runs only if condition is true
}
```

**Component breakdown**

- `catch (ExceptionType ex)` — The exception type and variable.
- `when (condition)` — A Boolean expression.
- `{ ... }` — The handler.

#### Syntax Rules

- `condition` must evaluate to `bool`.
- If `condition` is `false`, the runtime continues searching for another matching `catch` clause.
- Filters are evaluated before stack unwinding.
- Multiple filtered catches can be chained.

#### Constraints and Limitations

- The filter expression cannot use `await` (in most contexts) and cannot contain assignments that change state unexpectedly.
- The filter runs in the context of the `catch` clause; side effects should be minimal.

---

### Annotated Code Examples and Expected Outputs

#### Example: Filtering by Exception Message

```csharp
using System;
using System.Net.Http;

class Program
{
    static async System.Threading.Tasks.Task Main()
    {
        using var client = new HttpClient();
        try
        {
            await client.GetStringAsync("https://example.com/missing");
        }
        catch (HttpRequestException ex) when (ex.Message.Contains("404"))
        {
            Console.WriteLine("Page not found");
        }
        catch (HttpRequestException ex) when (ex.Message.Contains("500"))
        {
            Console.WriteLine("Server error");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"Other HTTP error: {ex.Message}");
        }
    }
}
```

**Expected Output (if 404)**
```
Page not found
```

**Why this output occurs**
The first filter matches the “404” condition; the handler runs. If the message did not contain “404”, the runtime would continue to the next filter.

---

### Real-World Cases

- **HTTP status handling** — Distinguish 404, 500, and other errors.
- **Retry logic** — Only retry on transient exceptions (e.g., `TimeoutException`).
- **Logging** — Log only exceptions with specific inner exception types.

---

### References

- when contextual keyword (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when
- Exception filters vs. traditional exception handling — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements#exception-filters-vs-traditional-exception-handling

---

## 6. Rethrowing Exceptions

### Definitions

**Core Definition**
Rethrowing is the act of propagating a caught exception to an outer handler, optionally adding context or performing cleanup before propagation.

**Technical Definition**
Rethrowing uses the `throw` statement inside a `catch` block. There are two forms: `throw;` (rethrow) preserves the original stack trace by emitting the IL `rethrow` instruction, while `throw ex;` resets the stack trace by throwing the exception object as if it originated at the current location. Rethrowing is used when the current method cannot fully handle the exception but needs to perform cleanup or logging.

**Beginner-Friendly Explanation**
Rethrowing is **passing the problem up the chain**. You catch the exception, do something (log, clean up), and then throw it again so a higher-level handler can deal with it. The key is to use `throw;` so the original trail of where the error happened is preserved.

---

### Purposes

- To **propagate** exceptions after partial handling.
- To **log** and rethrow without losing diagnostic information.
- To **roll back** partial work before letting the caller see the error.
- To **translate** exceptions into domain-specific types while preserving the original as `InnerException`.

---

### Syntax Rules and Structure

```csharp
try
{
    // guarded code
}
catch (Exception ex)
{
    // cleanup or logging
    throw;   // preserves stack trace
}
```

**Component breakdown**

- `throw;` — Rethrows the current exception with the original stack trace.
- `throw ex;` — Throws the exception object, resetting the stack trace to the current point.
- `throw new CustomException("message", ex);` — Wraps the original exception as `InnerException`.

#### Syntax Rules

- `throw;` is only valid inside a `catch` block.
- `throw ex;` is valid but resets the stack trace (generally discouraged).
- Wrapping with `throw new ... (message, ex)` preserves the original as `InnerException`.

#### Constraints and Limitations

- `throw ex;` loses the original stack trace (except in some .NET Core+ scenarios).
- Rethrowing without cleanup is usually unnecessary; let the exception propagate naturally.

---

### Annotated Code Examples and Expected Outputs

#### Example: Rethrowing with `throw;`

```csharp
using System;

class Program
{
    static void MethodB() => throw new NotImplementedException();

    static void MethodA()
    {
        try
        {
            MethodB();
        }
        catch (Exception ex)
        {
            Console.WriteLine("Logging in MethodA");
            throw;   // preserves original stack trace
        }
    }

    static void Main()
    {
        try
        {
            MethodA();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Caught: {ex.GetType().Name}");
            Console.WriteLine(ex.StackTrace);
        }
    }
}
```

**Expected Output (approximate)**
```
Logging in MethodA
Caught: NotImplementedException
   at Program.MethodB()
   at Program.MethodA()
   at Program.Main()
```

**Why this output occurs**
`throw;` rethrows the same exception with the original stack trace, so `MethodB` appears at the top.

---

### Real-World Cases

- **Transaction rollback** — Roll back and rethrow.
- **Logging middleware** — Log and rethrow.
- **API boundaries** — Wrap and rethrow with domain context.

---

### References

- How can I throw an exception without losing the original stack trace information in .NET? — https://learn.microsoft.com/en-us/archive/blogs/perfworld/how-can-i-throw-an-exception-without-losing-the-original-stack-trace-information-in-net

---

## 7. `throw` vs. `throw ex`

### Definitions

**Core Definition**
`throw;` and `throw ex;` are both rethrow statements, but they differ in how they treat the original stack trace: `throw;` preserves it, while `throw ex;` resets it.

**Technical Definition**
In IL, `throw ex;` compiles to the `throw` instruction, which starts a new exception throw at the current location. `throw;` compiles to the `rethrow` instruction, which continues propagation of the existing exception without modifying its stack trace. Because `throw ex;` resets the stack trace to the current method, it destroys the evidence of where the exception originally originated, making debugging harder.

**Beginner-Friendly Explanation**
`throw;` says **“keep the original error trail.”** `throw ex;` says **“start a new error trail here.”** Always prefer `throw;` unless you have a specific reason to reset.

---

### Purposes

- `throw;` — To preserve diagnostic information.
- `throw ex;` — Rarely justified; sometimes used to intentionally reset the trace (e.g., to hide internal details).

---

### Syntax Rules and Structure

```csharp
catch (Exception ex)
{
    throw;      // preserves stack trace (preferred)
    // throw ex; // resets stack trace (avoid)
}
```

**Component breakdown**

- `throw;` — Rethrow current exception.
- `throw ex;` — Throw the exception object.

#### Syntax Rules

- `throw;` is valid only inside a `catch` block.
- `throw ex;` is valid anywhere but resets the trace.

#### Constraints and Limitations

- Some .NET Core+ optimizations may alter the exact behavior, but the general rule holds.

---

### Annotated Code Examples and Expected Outputs

#### Example: Comparing `throw` and `throw ex`

```csharp
using System;

class Program
{
    static void MethodB() => throw new InvalidOperationException("Boom");

    static void MethodAWithThrow()
    {
        try { MethodB(); }
        catch (Exception) { throw; }   // preserves
    }

    static void MethodAWithThrowEx()
    {
        try { MethodB(); }
        catch (Exception ex) { throw ex; }   // resets
    }

    static void Main()
    {
        try { MethodAWithThrow(); }
        catch (Exception ex)
        {
            Console.WriteLine("--- throw; ---");
            Console.WriteLine(ex.StackTrace);
        }

        try { MethodAWithThrowEx(); }
        catch (Exception ex)
        {
            Console.WriteLine("--- throw ex; ---");
            Console.WriteLine(ex.StackTrace);
        }
    }
}
```

**Expected Output (approximate)**
```
--- throw; ---
   at Program.MethodB()
   at Program.MethodAWithThrow()
   at Program.Main()
--- throw ex; ---
   at Program.MethodAWithThrowEx()
   at Program.Main()
```

**Why this output occurs**
`throw;` keeps `MethodB` at the top; `throw ex;` starts the trace at `MethodAWithThrowEx`, losing `MethodB`.

---

### Real-World Cases

- **Debugging** — Always use `throw;` to retain the origin.
- **Security** — `throw ex;` may hide internal implementation details (rarely the goal).

---

### References

- How can I throw an exception without losing the original stack trace information in .NET? — https://learn.microsoft.com/en-us/archive/blogs/perfworld/how-can-i-throw-an-exception-without-losing-the-original-stack-trace-information-in-net

---

## 8. Exception Handling Best Practices

### Definitions

**Core Definition**
Exception handling best practices are the guidelines for writing `try`/`catch`/`finally` code that is **correct, diagnosable, and maintainable**.

**Technical Definition**
Best practices derive from the .NET documentation and Framework Design Guidelines. Key recommendations include: **use `try`/`catch`/`finally` to recover from errors or release resources**; **catch specific exceptions and order them from most derived to least derived**; **use `throw;` to preserve stack traces**; **use `using` or `finally` for cleanup**; **avoid catching `Exception` broadly** unless rethrowing or at a top-level handler; **use exception filters** instead of catch-and-rethrow; **handle common conditions without exceptions**; **design classes so exceptions can be avoided**; **restore state when methods don't complete**; and **do not expose sensitive information in exception messages**.

**Beginner-Friendly Explanation**
The best practices are: **catch what you can handle**, **catch specific types**, **don’t swallow exceptions**, **clean up resources**, **use `throw;`**, and **don’t use exceptions for normal flow**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Catch specific, derived-first
try { ... }
catch (ArgumentNullException ex) { ... }
catch (ArgumentException ex) { ... }

// 2. Preserve stack trace
catch (Exception ex)
{
    // log
    throw;
}

// 3. Clean up with using/finally
using var stream = File.OpenRead("file.txt");

// 4. Use exception filters instead of catch-rethrow
catch (HttpRequestException ex) when (ex.Message.Contains("404")) { ... }

// 5. Handle common conditions without exceptions
if (!connection.IsClosed) connection.Close();

// 6. Design classes to avoid exceptions
// e.g., TryParse, TryGetValue
```

**Component breakdown**

- **Specific catches** — Handle only what you can recover from.
- **`throw;`** — Preserves the original stack trace.
- **`using`/`finally`** — Guarantees cleanup.
- **Filters** — Avoid stack-unwinding rethrows.
- **Condition checks** — Avoid exceptions for expected conditions.

#### Rules and Constraints

- **Do** order catch clauses from most derived to least derived.
- **Do** use `throw;` to rethrow.
- **Do** use `using` or `finally` to release resources.
- **Do** use exception filters for conditional handling.
- **Do not** catch `Exception` broadly without rethrowing.
- **Do not** use `throw ex;` (resets stack trace).
- **Do not** use exceptions for normal control flow.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Structured Exception Handling

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        try
        {
            using var reader = new StreamReader("data.txt");
            Console.WriteLine(reader.ReadToEnd());
        }
        catch (FileNotFoundException ex)
        {
            Console.WriteLine($"Missing: {ex.FileName}");
        }
        catch (IOException ex) when (ex.Message.Contains("disk"))
        {
            Console.WriteLine("Disk error");
        }
        catch (IOException ex)
        {
            Console.WriteLine($"I/O error: {ex.Message}");
        }
    }
}
```

**Expected Output (if file missing)**
```
Missing: data.txt
```

**Why this output occurs**
`FileNotFoundException` is caught first (most specific); the filter on `IOException` refines the second handler; `using` guarantees disposal.

---

### Real-World Cases

- **ASP.NET Core middleware** — Top-level exception handling.
- **Background services** — Catch `OperationCanceledException` for graceful shutdown.
- **Library APIs** — Throw `ArgumentException` subtypes for invalid inputs.

---

### References

- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- try-catch (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch
- try-finally (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-finally
- when contextual keyword — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when
- Compiler Error CS0160 — https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0160
- How can I throw an exception without losing the original stack trace information in .NET? — https://learn.microsoft.com/en-us/archive/blogs/perfworld/how-can-i-throw-an-exception-without-losing-the-original-stack-trace-information-in-net

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `when` exception filters | Introduced in **C# 6.0**; stable. |
| `throw;` vs. `throw ex;` | `throw;` preserves stack trace; `throw ex;` resets it. |
| Catch ordering | **CS0160** if a base type precedes a derived type. |
| `finally` with `return` | **Compile-time error** (cannot return from `finally`). |
| `finally` throwing | Replaces the original exception. |
| `catch (Exception)` broadly | **Discouraged** except at top-level handlers. |
| `throw ex;` | **Discouraged** because it resets the stack trace. |

---

## References (Consolidated)

- Exception Handling (C# Programming Guide) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/exception-handling
- try-catch (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-catch
- try-finally (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/try-finally
- when contextual keyword (C# Reference) — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/when
- Best practices for exceptions — .NET — https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions
- Compiler Error CS0160 — https://learn.microsoft.com/en-us/dotnet/csharp/misc/cs0160
- Exception filters vs. traditional exception handling — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements#exception-filters-vs-traditional-exception-handling
- How can I throw an exception without losing the original stack trace information in .NET? — https://learn.microsoft.com/en-us/archive/blogs/perfworld/how-can-i-throw-an-exception-without-losing-the-original-stack-trace-information-in-net
- C# language specification — Exceptions — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/exceptions

---