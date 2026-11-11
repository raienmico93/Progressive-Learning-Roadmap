# C# IDisposable Pattern — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The **IDisposable pattern** is a standardized design pattern in .NET for **deterministic resource cleanup**. It defines a contract (`IDisposable`) that a type implements to release resources (managed and unmanaged) when they are no longer needed, rather than waiting for the non-deterministic garbage collector. It is the primary mechanism for managing **unmanaged resources** (file handles, native memory, network sockets, database connections, OS handles) safely and predictably.

**Technical Definition**
`System.IDisposable` declares a single method, `void Dispose()`. The **Dispose pattern** (also called the *disposable pattern*) is the recommended implementation guideline for types that hold resources. The full pattern includes: (1) a public `Dispose()` method that calls `Dispose(true)` and `GC.SuppressFinalize(this)`; (2) a `protected virtual void Dispose(bool disposing)` method that releases managed resources when `disposing == true` and unmanaged resources unconditionally, guarded by a `_disposed` flag for idempotency; (3) a finalizer (`~Type()`) that calls `Dispose(false)` as a safety net if `Dispose` was not called. For types that only hold other `IDisposable` managed resources, a simplified pattern (just `Dispose()` without a finalizer) is sufficient. Since **.NET Core 3.0 / C# 8.0**, `IAsyncDisposable` provides an asynchronous counterpart for resources requiring async cleanup. **`SafeHandle`** is the recommended alternative to writing finalizers manually for OS handles, as it encapsulates the finalization logic correctly and avoids many pitfalls.

**Beginner-Friendly Explanation**
The IDisposable pattern is the **"clean up after yourself" contract**. When you open a file, connect to a database, or allocate native memory, you are holding a resource that the garbage collector cannot free properly. The GC only knows about managed memory — it does not know how to close a file handle or free native memory. The IDisposable pattern lets you release those resources **deterministically**, exactly when you are done with them, instead of waiting for the GC (which may run much later, or never, if the process exits). The `using` statement (and `using` declaration) calls `Dispose()` for you automatically, even if exceptions occur. A **finalizer** is a last-resort safety net that runs if you forget to call `Dispose`, but it is non-deterministic and should only be used when directly holding unmanaged resources. **`SafeHandle`** is the modern, recommended way to wrap OS handles — it handles finalization correctly so you don't have to.

---

### Key Characteristics

- **Deterministic cleanup** — `Dispose()` runs exactly when called (via `using` or explicitly), unlike finalizers.
- **Two cleanup paths** — `Dispose(true)` for explicit disposal (managed + unmanaged), `Dispose(false)` from the finalizer (unmanaged only).
- **Idempotent** — `Dispose()` can be called multiple times safely; a `_disposed` flag guards the logic.
- **`GC.SuppressFinalize(this)`** — Called from `Dispose()` to prevent redundant finalization.
- **`SafeHandle` preferred** — For OS handles, `SafeHandle` replaces manual finalizer logic.
- **`IAsyncDisposable`** — Asynchronous cleanup via `DisposeAsync()` and `await using` (C# 8+).
- **`using` / `using var` / `await using`** — Language constructs that guarantee `Dispose`/`DisposeAsync` calls.
- **Exceptions in `Dispose`** — Should generally not be thrown; if they are, they can mask other exceptions.

---

### Prerequisites

- Understanding of **managed vs. unmanaged resources**.
- Familiarity with the **garbage collector** and finalizers.
- Knowledge of `try`/`finally` and exception handling.
- Awareness of `using` statements and declarations.
- Basic understanding of `SafeHandle` (helpful but not required).

---

### Related Programming Areas

- **Garbage collection** — `IDisposable` complements the GC for unmanaged resources.
- **Finalizers** — The safety-net mechanism for unmanaged resources.
- **`SafeHandle`** — The modern alternative to finalizers for OS handles.
- **`IAsyncDisposable`** — Async cleanup for network/database resources.
- **`using` statements** — The idiomatic consumption pattern.

---

### Core Concepts / Features

1. `IDisposable`
2. `Dispose()` Method
3. `Dispose(bool)` Pattern
4. `GC.SuppressFinalize()`
5. Finalizer
6. `SafeHandle`
7. IDisposable Best Practices

---

## 1. `IDisposable`

### Definitions

**Core Definition**
`IDisposable` is the .NET interface that defines a single `Dispose()` method for deterministic resource cleanup.

**Technical Definition**
`System.IDisposable` declares `void Dispose()`. Types that hold resources (managed or unmanaged) implement this interface to provide a deterministic way to release them. The interface is used by the `using` statement, `using` declaration, and `await using` (via `IAsyncDisposable`). The `Dispose` pattern (with `Dispose(bool)` and a finalizer) is the recommended implementation for types holding unmanaged resources directly; simpler implementations suffice for types holding only other `IDisposable` resources.

**Beginner-Friendly Explanation**
`IDisposable` is the **"I have resources to clean up"** interface. Implement it, and callers can use `using` to dispose your object deterministically.

---

### Purposes

- To provide **deterministic cleanup** for resources.
- To release **unmanaged resources** (file handles, native memory, OS handles).
- To release **managed resources** that themselves implement `IDisposable`.
- To enable the `using` statement and declaration.

---

### Syntax Rules and Structure

```csharp
public interface IDisposable
{
    void Dispose();
}
```

**Component breakdown**

- `Dispose()` — The only method.
- Implementations should be **idempotent** (safe to call multiple times).

#### Syntax Rules

- Implementing types must provide `void Dispose()`.
- `Dispose()` should release all resources and set a `_disposed` flag.
- `Dispose()` should call `GC.SuppressFinalize(this)` if a finalizer exists.

#### Constraints and Limitations

- `IDisposable` alone does not guarantee cleanup — callers must call `Dispose` (or use `using`).
- Exceptions from `Dispose` can mask other exceptions.

---

### Annotated Code Examples and Expected Outputs

#### Example: Implementing `IDisposable`

```csharp
using System;

public class MyResource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        if (_disposed) return;
        // release resources
        _disposed = true;
        Console.WriteLine("Disposed");
    }
}

class Program
{
    static void Main()
    {
        using var r = new MyResource();
        Console.WriteLine("Using resource");
    }
}
```

**Expected Output**
```
Using resource
Disposed
```

**Why this output occurs**
`using var` calls `Dispose()` at the end of `Main`, printing "Disposed".

---

### Real-World Cases

- **File streams** — `FileStream` implements `IDisposable`.
- **Database connections** — `DbConnection` implements `IDisposable`.
- **Native memory** — `SafeHandle` implements `IDisposable`.

---

### References

- IDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.idisposable

---

## 2. `Dispose()` Method

### Definitions

**Core Definition**
`Dispose()` is the method declared by `IDisposable` that releases resources deterministically.

**Technical Definition**
`Dispose()` is the public entry point for resource cleanup. It should: (1) release managed resources (if any), (2) release unmanaged resources (if any), (3) call `GC.SuppressFinalize(this)` if the type has a finalizer, and (4) be idempotent (safe to call multiple times). It should **not** throw exceptions under normal circumstances. The `using` statement/declaration calls it automatically on scope exit.

**Beginner-Friendly Explanation**
`Dispose()` is the **cleanup method** you call (or `using` calls for you) to release resources.

---

### Purposes

- To release resources **immediately**.
- To free unmanaged resources deterministically.
- To be called by `using` on scope exit.
- To be idempotent and exception-safe.

---

### Syntax Rules and Structure

```csharp
public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}
```

**Component breakdown**

- `Dispose()` — Public method.
- Calls `Dispose(true)`.
- Calls `GC.SuppressFinalize(this)`.

#### Syntax Rules

- Should be idempotent.
- Should call `Dispose(true)` and `GC.SuppressFinalize(this)`.
- Should not throw exceptions.

#### Constraints and Limitations

- Does not automatically dispose managed resources unless implemented.
- Exceptions in `Dispose` can mask original exceptions.

---

### Annotated Code Examples and Expected Outputs

#### Example: Simple `Dispose`

```csharp
using System;
using System.IO;

public class FileWriter : IDisposable
{
    private StreamWriter _writer;

    public FileWriter(string path) => _writer = new StreamWriter(path);

    public void Write(string text) => _writer.WriteLine(text);

    public void Dispose()
    {
        _writer?.Dispose();
        _writer = null;
        Console.WriteLine("Writer disposed");
    }
}

class Program
{
    static void Main()
    {
        using var fw = new FileWriter("output.txt");
        fw.Write("Hello");
    }
}
```

**Expected Output**
```
Writer disposed
```

**Why this output occurs**
`using var` calls `Dispose()`, which disposes the wrapped `StreamWriter`.

---

### Real-World Cases

- **Stream wrappers** — Dispose the underlying stream.
- **Database wrappers** — Dispose connections and commands.
- **Native memory wrappers** — Free unmanaged memory.

---

### References

- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

---

## 3. `Dispose(bool)` Pattern

### Definitions

**Core Definition**
The `Dispose(bool disposing)` pattern is the **full Dispose implementation** that distinguishes between explicit disposal (`disposing == true`) and finalizer-triggered disposal (`disposing == false`).

**Technical Definition**
`protected virtual void Dispose(bool disposing)` is the core cleanup method. When `disposing` is `true` (called from `Dispose()`), it releases **both managed and unmanaged** resources. When `disposing` is `false` (called from the finalizer), it releases **only unmanaged** resources, because managed objects may have already been finalized or collected. The method uses a `_disposed` flag to ensure idempotency. Derived classes can override `Dispose(bool)` and call `base.Dispose(disposing)` to chain cleanup.

**Beginner-Friendly Explanation**
`Dispose(bool)` is **"the real cleanup method"** with a flag telling it whether it is safe to touch other managed objects.

---

### Purposes

- To safely release managed and unmanaged resources.
- To distinguish explicit disposal from finalizer-triggered cleanup.
- To support **inheritance** (derived classes override `Dispose(bool)`).
- To ensure idempotency via the `_disposed` flag.

---

### Syntax Rules and Structure

```csharp
private bool _disposed;

public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}

protected virtual void Dispose(bool disposing)
{
    if (_disposed) return;

    if (disposing)
    {
        // dispose managed resources
    }

    // free unmanaged resources

    _disposed = true;
}
```

**Component breakdown**

- `_disposed` — Idempotency flag.
- `disposing` — Whether managed resources can be safely disposed.
- `virtual` — Allows derived classes to extend.

#### Syntax Rules

- `Dispose(bool)` must be `protected virtual`.
- Release managed resources only when `disposing == true`.
- Always release unmanaged resources.
- Set `_disposed = true` at the end.

#### Constraints and Limitations

- Derived classes must call `base.Dispose(disposing)`.
- Managed resources should not be touched when `disposing == false`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Full Dispose Pattern

```csharp
using System;
using System.IO;

public class ResourceHolder : IDisposable
{
    private bool _disposed;
    private IntPtr _nativeResource;   // pretend unmanaged handle
    private StreamWriter? _writer;

    public ResourceHolder(string path)
    {
        _writer = new StreamWriter(path);
        _nativeResource = IntPtr.Zero;   // allocate unmanaged if needed
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;

        if (disposing)
        {
            _writer?.Dispose();
            _writer = null;
        }

        // Free unmanaged resources (always)
        if (_nativeResource != IntPtr.Zero)
        {
            // e.g., Marshal.FreeHGlobal(_nativeResource);
            _nativeResource = IntPtr.Zero;
        }

        _disposed = true;
        Console.WriteLine("Disposed");
    }

    ~ResourceHolder() => Dispose(false);
}

class Program
{
    static void Main()
    {
        using var holder = new ResourceHolder("out.txt");
    }
}
```

**Expected Output**
```
Disposed
```

**Why this output occurs**
`using var` calls `Dispose()`, which calls `Dispose(true)` and `GC.SuppressFinalize`, so only "Disposed" prints.

---

### Real-World Cases

- **Any class with unmanaged resources** — Use the full pattern.
- **Classes that may be inherited** — Use `virtual Dispose(bool)`.

---

### References

- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

---

## 4. `GC.SuppressFinalize()`

### Definitions

**Core Definition**
`GC.SuppressFinalize()` tells the garbage collector **not to call the finalizer** for a given object, typically because `Dispose` has already cleaned up.

**Technical Definition**
`GC.SuppressFinalize(object obj)` removes the object from the finalization queue. It is called at the end of `Dispose()` (after cleanup) to prevent the finalizer from running and to avoid the performance cost of finalization (promotion, finalizer thread). It is a no-op if the object has no finalizer.

**Beginner-Friendly Explanation**
`GC.SuppressFinalize()` is **"don't bother with the finalizer; I already cleaned up."**

---

### Purposes

- To prevent **redundant finalization** after `Dispose`.
- To improve **performance** by avoiding finalizer promotion.
- To ensure deterministic cleanup is honored.

---

### Syntax Rules and Structure

```csharp
public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}
```

**Component breakdown**

- `GC.SuppressFinalize(this)` — Called at the end of `Dispose`.

#### Syntax Rules

- Call after cleanup succeeds.
- Safe to call even if no finalizer exists (no-op).

#### Constraints and Limitations

- Must be called after cleanup; if `Dispose` fails, do not suppress.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Dispose` with `SuppressFinalize`

```csharp
using System;

public class MyResource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        if (_disposed) return;
        _disposed = true;
        GC.SuppressFinalize(this);
        Console.WriteLine("Disposed");
    }

    ~MyResource() => Console.WriteLine("Finalizer");
}

class Program
{
    static void Main()
    {
        using (var r = new MyResource()) { }
    }
}
```

**Expected Output**
```
Disposed
```

**Why this output occurs**
`Dispose` calls `GC.SuppressFinalize`, so the finalizer does not run.

---

### Real-World Cases

- **Every full Dispose pattern implementation**.
- **Classes with finalizers**.

---

### References

- GC.SuppressFinalize Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.suppressfinalize

---

## 5. Finalizer

### Definitions

**Core Definition**
A finalizer is a method that runs **before an object is garbage-collected**, used to release unmanaged resources as a safety net.

**Technical Definition**
In C#, finalizers are declared with `~ClassName()`. They are called by the GC non-deterministically, on a dedicated finalizer thread. Objects with finalizers are promoted to Gen 1 (and eventually Gen 2), delaying collection. Finalizers should only release unmanaged resources and should not access other managed objects (which may be finalized). They should call `Dispose(false)` to release unmanaged resources and should be suppressed by `Dispose()`.

**Beginner-Friendly Explanation**
A finalizer is a **cleanup safety net**. It runs if you forget to call `Dispose`, but it is non-deterministic and should be used sparingly.

---

### Purposes

- To release **unmanaged resources** if `Dispose` was not called.
- To serve as a **last-resort safety net**.

---

### Syntax Rules and Structure

```csharp
~ClassName()
{
    Dispose(false);
}
```

**Component breakdown**

- `~ClassName()` — Finalizer syntax.
- Calls `Dispose(false)`.

#### Syntax Rules

- Only classes can have finalizers.
- Cannot be overloaded or inherited.
- Cannot be called explicitly.

#### Constraints and Limitations

- Non-deterministic.
- Runs on a single finalizer thread (bottleneck).
- Exceptions in finalizers are fatal.
- Delays collection (promotion).

---

### Annotated Code Examples and Expected Outputs

#### Example: Finalizer as Safety Net

```csharp
using System;

public class MyResource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        if (_disposed) return;
        _disposed = true;
        GC.SuppressFinalize(this);
    }

    ~MyResource()
    {
        if (!_disposed)
            Console.WriteLine("Finalizer running");
    }
}

class Program
{
    static void Main()
    {
        var r = new MyResource();   // not disposed
        r = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }
}
```

**Expected Output**
```
Finalizer running
```

**Why this output occurs**
The object was not disposed, so the finalizer runs as a safety net.

---

### Real-World Cases

- **Unmanaged resource wrappers** — Prefer `SafeHandle`.
- **Legacy interop** — Classes wrapping native handles.

---

### References

- Finalizers — C# Programming Guide — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers

---

## 6. `SafeHandle`

### Definitions

**Core Definition**
`SafeHandle` is an abstract class in `System.Runtime.InteropServices` that **wraps an OS handle** and correctly implements the finalizer/dispose pattern, preventing handle leaks and handle-recycling bugs.

**Technical Definition**
`SafeHandle` (and its generic variant `SafeHandleZeroOrMinusOneIsInvalid`) encapsulates an operating-system handle. It provides the finalization logic automatically, guaranteeing that the handle is released even if `Dispose` is not called. `SafeHandle` is **critical finalizable** and is preferred over writing custom finalizers for OS handles because it avoids common bugs: double-free, handle recycling, and finalizer ordering issues. `SafeHandle` also supports **reference counting** via `DangerousAddRef`/`DangerousRelease` for interop scenarios.

**Beginner-Friendly Explanation**
`SafeHandle` is **the correct way to wrap OS handles**. It handles finalization automatically, so you do not have to write your own finalizer.

---

### Purposes

- To correctly release OS handles.
- To avoid handle leaks and double-free bugs.
- To simplify interop code.

---

### Syntax Rules and Structure

```csharp
using Microsoft.Win32.SafeHandles;

public sealed class MySafeHandle : SafeHandleZeroOrMinusOneIsInvalid
{
    public MySafeHandle() : base(true) { }

    protected override bool ReleaseHandle()
    {
        return NativeMethods.CloseHandle(handle);   // release OS handle
    }
}
```

**Component breakdown**

- `SafeHandleZeroOrMinusOneIsInvalid` — Base class for invalid handles.
- `ReleaseHandle()` — Override to release the handle.
- `base(true)` — Owns the handle.

#### Syntax Rules

- Derive from `SafeHandle` or `SafeHandleZeroOrMinusOneIsInvalid`.
- Override `ReleaseHandle()`.
- Pass `ownsHandle: true` to the base constructor.

#### Constraints and Limitations

- More complex than finalizers for simple scenarios, but safer.
- Requires P/Invoke declarations for `ReleaseHandle`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using a `SafeHandle`

```csharp
using System;
using Microsoft.Win32.SafeHandles;

public sealed class MySafeHandle : SafeHandleZeroOrMinusOneIsInvalid
{
    public MySafeHandle() : base(true) { }

    protected override bool ReleaseHandle()
    {
        Console.WriteLine("Releasing handle");
        return true;
    }
}

class Program
{
    static void Main()
    {
        using var handle = new MySafeHandle();
        Console.WriteLine("Using handle");
    }
}
```

**Expected Output**
```
Using handle
Releasing handle
```

**Why this output occurs**
`using var` calls `Dispose()`, which invokes `ReleaseHandle`.

---

### Real-World Cases

- **File handles** — `SafeFileHandle`.
- **Registry keys** — `SafeRegistryHandle`.
- **Native memory** — `SafeHGlobalHandle`.

---

### References

- SafeHandle Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle
- SafeHandleZeroOrMinusOneIsInvalid Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.win32.safehandles.safehandlezeroorminusoneisinvalid

---

## 7. IDisposable Best Practices

### Definitions

**Core Definition**
IDisposable best practices are the guidelines that ensure resource cleanup is **deterministic, safe, idempotent, and correct** across inheritance and finalization scenarios.

**Technical Definition**
Best practices derive from Microsoft's Dispose pattern guidance: **implement `IDisposable` on any type holding resources**; **use the full pattern (with `Dispose(bool)` and a finalizer) only when holding unmanaged resources directly**; **use `SafeHandle` instead of finalizers for OS handles**; **make `Dispose` idempotent**; **call `GC.SuppressFinalize`**; **never throw from `Dispose`**; **dispose managed resources in reverse order of acquisition**; **and use `using` / `using var` for consumption**. For types implementing both `IDisposable` and `IAsyncDisposable`, prefer `await using` in async code.

**Beginner-Friendly Explanation**
The best practices are: **implement it when you hold resources**, **use `SafeHandle` for OS handles**, **make `Dispose` idempotent**, **suppress finalization**, and **use `using` everywhere**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Implement IDisposable when holding resources
public class FileService : IDisposable { ... }

// 2. Use the full pattern for unmanaged resources
protected virtual void Dispose(bool disposing) { ... }
~FileService() => Dispose(false);

// 3. Use SafeHandle for OS handles
public sealed class MySafeHandle : SafeHandleZeroOrMinusOneIsInvalid { ... }

// 4. Make Dispose idempotent
private bool _disposed;
public void Dispose() { if (_disposed) return; ... _disposed = true; }

// 5. Suppress finalization
GC.SuppressFinalize(this);

// 6. Do not throw from Dispose
public void Dispose() { try { ... } catch { } }

// 7. Use using / using var / await using
using var resource = new Resource();
await using var asyncResource = new AsyncResource();

// 8. Dispose in reverse order of acquisition
```

**Component breakdown**

- **Full pattern** — Only when holding unmanaged resources.
- **`SafeHandle`** — For OS handles.
- **Idempotent** — `_disposed` flag.
- **`SuppressFinalize`** — Avoid redundant finalization.
- **No exceptions** — Dispose should be safe.
- **`using`** — Deterministic cleanup.

#### Rules and Constraints

- **Do** implement `IDisposable` for resource holders.
- **Do** use the full pattern only for unmanaged resources.
- **Do** use `SafeHandle` for OS handles.
- **Do** make `Dispose` idempotent and exception-safe.
- **Do** call `GC.SuppressFinalize`.
- **Do not** throw from `Dispose`.
- **Do not** rely on finalizers for deterministic cleanup.
- **Do not** access managed objects from a finalizer.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Disposable Type

```csharp
using System;
using System.IO;

public sealed class FileService : IDisposable
{
    private readonly StreamWriter _writer;
    private bool _disposed;

    public FileService(string path) => _writer = new StreamWriter(path);

    public void Write(string text)
    {
        if (_disposed) throw new ObjectDisposedException(nameof(FileService));
        _writer.WriteLine(text);
    }

    public void Dispose()
    {
        if (_disposed) return;
        _writer.Dispose();
        _disposed = true;
        GC.SuppressFinalize(this);
        Console.WriteLine("Disposed");
    }
}

class Program
{
    static void Main()
    {
        using var fs = new FileService("out.txt");
        fs.Write("Hello");
    }
}
```

**Expected Output**
```
Disposed
```

**Why this output occurs**
`using var` calls `Dispose`, which disposes the writer, sets `_disposed`, and suppresses finalization.

---

### Real-World Cases

- **`FileStream`** — Full pattern with `SafeFileHandle`.
- **`DbConnection`** — Implements both `IDisposable` and `IAsyncDisposable`.
- **`HttpClient`** — Long-lived; dispose at application end.

---

### References

- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose
- Implement a DisposeAsync method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-disposeasync
- IDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.idisposable
- SafeHandle Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `IDisposable` | Core since **.NET 1.0**; stable. |
| `Dispose(bool)` pattern | Recommended for unmanaged resources; stable. |
| Finalizers | **Non-deterministic**; prefer `SafeHandle`. |
| `GC.SuppressFinalize` | Required in full Dispose pattern. |
| `SafeHandle` | Recommended over finalizers for OS handles. |
| `IAsyncDisposable` | Introduced in **.NET Core 3.0 / C# 8.0**; version-specific. |
| `using var` | Introduced in **C# 8.0**; version-specific. |
| `await using` | Introduced in **C# 8.0**; version-specific. |
| Throwing from `Dispose` | **Discouraged**; can mask other exceptions. |

---

## References (Consolidated)

- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose
- Implement a DisposeAsync method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-disposeasync
- IDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.idisposable
- IAsyncDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.iasyncdisposable
- GC.SuppressFinalize Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.suppressfinalize
- Finalizers — C# Programming Guide — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- SafeHandle Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle
- SafeHandleZeroOrMinusOneIsInvalid Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.win32.safehandles.safehandlezeroorminusoneisinvalid
- using statement — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using
- using declaration — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using

---