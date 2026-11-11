# C# Garbage Collection — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Garbage collection (GC)** in .NET is an **automatic memory management** system that reclaims the memory of objects that are no longer reachable by the program. It eliminates the need for manual `free`/`delete` calls, prevents memory leaks from forgotten deallocations, and eliminates dangling-pointer bugs. The GC is a core CLR service that runs periodically (or under memory pressure) to identify unreachable objects and reclaim their memory.

**Technical Definition**
.NET's garbage collector is a **tracing, generational, compacting, mark-and-sweep** collector. The CLR's GC manages the managed heap and uses the following core principles: **generations** (Gen 0, Gen 1, Gen 2) to optimize for the empirical observation that most objects are short-lived; **tracing** from roots (stack, statics, GC handles, finalization queue) to determine reachability; **compaction** (for the small-object heap) to eliminate fragmentation and improve locality; and **concurrent/background** collection to reduce pause times. The GC is **non-deterministic**: you cannot predict when it will run. It supports different **modes** (Workstation, Server, Background) tuned for different workloads. Objects that wrap unmanaged resources should implement the **Dispose pattern** (`IDisposable` / `IAsyncDisposable`) for deterministic cleanup, with a **finalizer** (`~Type()`) as a safety net that runs if `Dispose` is not called. The `using` statement and `using` declaration (C# 8+) guarantee deterministic `Dispose` calls; `await using` does the same for `IAsyncDisposable`.

**Beginner-Friendly Explanation**
Garbage collection is the **automatic cleanup crew** of .NET. You create objects with `new`, and when they are no longer used, the GC finds them and reclaims their memory. You do not have to manually free memory like in C or C++. The GC is smart: it divides objects into **generations** (young, middle-aged, old) and focuses its effort on young objects, which die quickly. It runs in different **modes** (Workstation for client apps, Server for multi-core servers, Background for low-latency scenarios). However, the GC only manages **managed memory** — it does not know how to free **unmanaged resources** like file handles or native memory. For those, you implement `IDisposable` (or `IAsyncDisposable`) and use `using`/`await using` to guarantee deterministic cleanup. A **finalizer** is a last-resort safety net that runs if you forget to call `Dispose`, but finalizers are non-deterministic and should be used sparingly.

---

### Key Characteristics

- **Automatic and tracing** — The GC traces roots to find reachable objects; unreachable ones are collected.
- **Generational** — Gen 0 (young), Gen 1 (middle), Gen 2 (old); the LOH (large objects) is collected during Gen 2.
- **Compacting** — Survivors are moved to eliminate fragmentation (except the LOH).
- **Non-deterministic** — You cannot predict exactly when GC will run.
- **Modes** — Workstation (client), Server (throughput), Background (low-latency concurrent).
- **Deterministic cleanup for unmanaged resources** — `IDisposable` / `IAsyncDisposable` + `using` / `await using`.
- **Finalizers as a safety net** — Non-deterministic, delay collection, require promotion; prefer `SafeHandle`.
- **`GC.SuppressFinalize(this)`** — Called by `Dispose` to prevent redundant finalization.

---

### Prerequisites

- Basic understanding of **reference types** and **the managed heap**.
- Familiarity with **object lifetimes** and **reachability**.
- Awareness of **unmanaged resources** (file handles, native memory, network sockets).
- Knowledge of `try`/`finally` and exception handling.
- Understanding of **threads** (for understanding GC modes).

---

### Related Programming Areas

- **Memory model** — Stack vs. heap, allocation, and layout.
- **Dispose pattern** — `IDisposable`, `IAsyncDisposable`, `SafeHandle`.
- **Finalizers/destructors** — Last-resort cleanup.
- **Performance** — GC pauses, allocations, LOH fragmentation.
- **Hosted services** — `stoppingToken`, graceful shutdown, resource cleanup.

---

### Core Concepts / Features

1. Garbage Collection (Overarching Concept)
2. GC Generations (Gen 0, Gen 1, Gen 2)
3. GC Modes (Workstation, Server, Background)
4. `GC.Collect()`
5. `GC.SuppressFinalize()`
6. Finalizers and Destructors
7. `IDisposable` and the Dispose Pattern
8. `using` Statement
9. `using` Declaration (C# 8)
10. `IAsyncDisposable` and `await using`
11. GC Best Practices
12. GC Performance

---

## 1. Garbage Collection (Overarching Concept)

### Definitions

**Core Definition**
Garbage collection is the automatic process of reclaiming memory occupied by objects that are no longer reachable by the program.

**Technical Definition**
The .NET GC is a **tracing, generational, compacting** garbage collector. It identifies unreachable objects by tracing from roots (stack frames, static fields, GC handles, finalization queue) and reclaims their memory. It uses **generations** to optimize collection of short-lived objects and **compaction** to reduce fragmentation. The GC is **non-deterministic** and runs on its own schedule (or when memory pressure requires it). It manages only the **managed heap**; unmanaged resources require explicit cleanup via `IDisposable`/`IAsyncDisposable` or finalizers.

**Beginner-Friendly Explanation**
The GC is **automatic cleanup for managed memory**. You create objects; when they are no longer used, the GC reclaims them. No manual `free`.

---

### Purposes

- To **eliminate manual memory management** (no `free`/`delete`).
- To prevent **memory leaks** and **dangling pointers** for managed objects.
- To improve **developer productivity** and **application reliability**.
- To optimize **locality** and **fragmentation** via compaction.
- To **reduce pauses** via generational and background collection.

---

### Syntax Rules and Structure

```
// Allocation (GC-managed)
var obj = new MyClass();

// No explicit free needed; GC reclaims when unreachable
```

**Component breakdown**

- `new` — Allocates on the managed heap.
- Roots — Stack, statics, GC handles.
- Reachability — Objects reachable from roots survive.

#### Syntax Rules

- The GC runs automatically; no explicit calls are required.
- `GC.Collect()` forces a collection (discouraged in normal code).
- Managed memory is reclaimed automatically; unmanaged memory is not.

#### Constraints and Limitations

- Non-deterministic; you cannot predict exactly when GC runs.
- GC pauses can affect latency.
- LOH fragmentation is a known issue.

---

### Annotated Code Examples and Expected Outputs

#### Example: GC Reclaims Unreachable Objects

```csharp
using System;

class Program
{
    static void Main()
    {
        var obj = new object();   // reachable
        Console.WriteLine("Created object");

        obj = null;               // unreachable; eligible for collection
        Console.WriteLine("Object dereferenced");

        GC.Collect();             // force collection (typically discouraged)
        GC.WaitForPendingFinalizers();
        Console.WriteLine("Collection requested");
    }
}
```

**Expected Output**
```
Created object
Object dereferenced
Collection requested
```

**Why this output occurs**
After `obj = null`, the object is unreachable; `GC.Collect()` forces a collection, though its finalization/collection is non-deterministic in general.

---

### Real-World Cases

- **Any .NET application** — All managed memory is GC-managed.
- **Web servers** — Server GC optimizes throughput.
- **Desktop apps** — Workstation GC optimizes responsiveness.

---

### References

- Fundamentals of garbage collection — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

---

## 2. GC Generations (Gen 0, Gen 1, Gen 2)

### Definitions

**Core Definition**
Generations are **age-based segments** of the managed heap. The GC collects younger generations more frequently (and cheaply) because most objects die young.

**Technical Definition**
The managed heap is divided into three generations:
- **Gen 0** — Newly allocated objects. Collected most frequently; collections are fast.
- **Gen 1** — Objects that survived a Gen 0 collection. A buffer between short- and long-lived objects.
- **Gen 2** — Objects that survived one or more Gen 1 collections. Long-lived objects; collected during **full GCs**.
The **LOH** (Large Object Heap) is collected during Gen 2 collections and is not compacted by default. Objects are promoted from Gen 0 → Gen 1 → Gen 2 when they survive collections.

**Beginner-Friendly Explanation**
Generations are **age groups**. New objects start in Gen 0; survivors get promoted to Gen 1 and Gen 2. The GC checks Gen 0 often, Gen 2 rarely.

---

### Purposes

- To **optimize for the weak generational hypothesis** (most objects die young).
- To **reduce GC pause times** by collecting young objects cheaply.
- To **improve throughput** by avoiding scanning long-lived objects frequently.

---

### Syntax Rules and Structure

```
Gen 0 → Gen 1 → Gen 2   (promotion on survival)
```

**Component breakdown**

- **Gen 0** — Short-lived objects; frequent, cheap collections.
- **Gen 1** — Buffer generation.
- **Gen 2** — Long-lived objects; full GCs.
- **LOH** — Collected with Gen 2.

#### Syntax Rules

- New objects start in Gen 0.
- Survivors are promoted to older generations.
- Full GCs collect Gen 2 (and the LOH).

#### Constraints and Limitations

- Gen 2 collections are expensive (full heap scan).
- LOH fragmentation persists across Gen 2 collections unless compaction is enabled.

---

### Annotated Code Examples and Expected Outputs

#### Example: Observing Generations

```csharp
using System;

class Program
{
    static void Main()
    {
        var obj = new object();
        Console.WriteLine($"Gen: {GC.GetGeneration(obj)}");   // 0

        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();

        Console.WriteLine($"Gen after GC: {GC.GetGeneration(obj)}");   // 2
    }
}
```

**Expected Output**
```
Gen: 0
Gen after GC: 2
```

**Why this output occurs**
The object starts in Gen 0; after a full collection, it is promoted to Gen 2.

---

### Real-World Cases

- **Short-lived allocations** — Most objects in request processing die in Gen 0.
- **Caches and long-lived services** — Live in Gen 2.

---

### References

- Fundamentals of garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

---

## 3. GC Modes (Workstation, Server, Background)

### Definitions

**Core Definition**
GC modes are **configuration profiles** that tune the GC for different workloads: Workstation (client apps), Server (multi-core servers), and Background (concurrent, low-latency).

**Technical Definition**
- **Workstation GC** — Default for client apps. Optimized for low latency and small heaps.
- **Server GC** — Default for ASP.NET Core. Uses one heap and one thread per CPU; optimized for throughput and scalability.
- **Background GC** — Concurrent Gen 2 collections that run on a background thread, reducing pause times. Enabled by default in Workstation and Server GC.

**Beginner-Friendly Explanation**
GC modes are **tuning profiles**. Workstation is for desktops; Server is for web servers; Background keeps pause times low by collecting in the background.

---

### Purposes

- To **optimize GC behavior for the workload** (client vs. server).
- To **reduce pause times** (Background GC).
- To **maximize throughput** on multi-core servers (Server GC).

---

### Syntax Rules and Structure

```xml
<!-- .csproj -->
<PropertyGroup>
    <ServerGarbageCollection>true</ServerGarbageCollection>
    <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

**Component breakdown**

- **Workstation** — Default for desktop.
- **Server** — Per-CPU heaps; enabled via `ServerGarbageCollection`.
- **Background** — Concurrent Gen 2; enabled via `ConcurrentGarbageCollection`.

#### Syntax Rules

- Workstation GC is the default.
- Server GC is enabled via `ServerGarbageCollection` in `.csproj` or runtime config.
- Background GC is enabled by default.

#### Constraints and Limitations

- Server GC uses more memory (per-CPU heaps).
- Background GC is not available for all modes (e.g., Server GC without concurrent GC).

---

### Annotated Code Examples and Expected Outputs

#### Example: Configuring Server GC

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ServerGarbageCollection>true</ServerGarbageCollection>
    <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
  </PropertyGroup>
</Project>
```

**Expected Output (conceptual)**
```
// ASP.NET Core uses Server GC by default for higher throughput.
```

**Why this configuration matters**
Server GC allocates one heap per CPU, improving throughput on multi-core servers.

---

### Real-World Cases

- **ASP.NET Core** — Server GC by default.
- **Desktop apps** — Workstation GC by default.
- **Low-latency services** — Background GC reduces pauses.

---

### References

- Workstation and server garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc
- Background garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/background-gc

---

## 4. `GC.Collect()`

### Definitions

**Core Definition**
`GC.Collect()` forces an immediate garbage collection.

**Technical Definition**
`GC.Collect()` triggers a garbage collection. It can be called with or without a generation number (`GC.Collect(int generation)`). It is **generally discouraged** in production code because the GC is already tuned to run at optimal times; forcing collection can hurt performance, increase pause times, and disrupt the generational hypothesis. It is sometimes justified in **tests**, **benchmarks**, or **specific diagnostic scenarios**.

**Beginner-Friendly Explanation**
`GC.Collect()` is **"force cleanup now."** Most of the time, let the GC decide when to run.

---

### Purposes

- To force collection in **tests** (to verify object lifetime).
- To release memory at a specific point in **diagnostic scenarios**.
- To observe GC behavior in benchmarks.

---

### Syntax Rules and Structure

```
GC.Collect();               // full GC
GC.Collect(int generation); // collect up to the specified generation
```

**Component breakdown**

- `GC.Collect()` — Forces a full GC (Gen 0, 1, 2, LOH).
- `GC.Collect(generation)` — Collects up to the specified generation.

#### Syntax Rules

- Typically followed by `GC.WaitForPendingFinalizers()` and another `GC.Collect()` to observe finalizer effects.
- Not a substitute for `Dispose` on unmanaged resources.

#### Constraints and Limitations

- **Can significantly hurt performance** in production.
- Does not guarantee finalizers have run (use `WaitForPendingFinalizers`).

---

### Annotated Code Examples and Expected Outputs

#### Example: Forcing GC in a Test

```csharp
using System;

class Program
{
    static void Main()
    {
        var obj = new object();
        WeakReference weak = new WeakReference(obj);

        obj = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();

        Console.WriteLine($"Alive: {weak.IsAlive}");   // False
    }
}
```

**Expected Output**
```
Alive: False
```

**Why this output occurs**
`GC.Collect()` forces collection; the weak reference no longer points to a live object.

---

### Real-World Cases

- **Unit tests** — Verifying finalizer/Dispose behavior.
- **Benchmarks** — Establishing a clean GC baseline.
- **Diagnostics** — Measuring memory before/after.

---

### References

- GC.Collect Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.collect

---

## 5. `GC.SuppressFinalize()`

### Definitions

**Core Definition**
`GC.SuppressFinalize()` tells the GC **not to call the finalizer** for a given object, typically because `Dispose` has already cleaned up the resources.

**Technical Definition**
`GC.SuppressFinalize(object obj)` removes the object from the finalization queue, preventing the finalizer from running. It is typically called at the end of `Dispose()` to avoid the overhead of redundant finalization when `Dispose` has already released resources. The standard Dispose pattern calls `GC.SuppressFinalize(this)` after disposing managed and unmanaged resources.

**Beginner-Friendly Explanation**
`GC.SuppressFinalize()` is **"don't run the finalizer — I already cleaned up."** It's a performance optimization in the Dispose pattern.

---

### Purposes

- To **avoid redundant finalization** after `Dispose`.
- To **improve performance** by preventing finalizer-queue promotion.
- To ensure deterministic cleanup is honored.

---

### Syntax Rules and Structure

```csharp
public void Dispose()
{
    // cleanup
    GC.SuppressFinalize(this);
}
```

**Component breakdown**

- `GC.SuppressFinalize(this)` — Called at the end of `Dispose`.
- Prevents the finalizer from running.

#### Syntax Rules

- Call `GC.SuppressFinalize(this)` after cleanup in `Dispose`.
- For `IDisposable`, the class should still have a finalizer if it wraps unmanaged resources directly.

#### Constraints and Limitations

- Must be called after cleanup; otherwise resources may leak if `Dispose` fails.

---

### Annotated Code Examples and Expected Outputs

#### Example: Dispose with `SuppressFinalize`

```csharp
using System;

public class Resource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        if (_disposed) return;
        // free resources
        _disposed = true;
        GC.SuppressFinalize(this);   // prevent finalizer
        Console.WriteLine("Disposed");
    }

    ~Resource() => Console.WriteLine("Finalizer");
}

class Program
{
    static void Main()
    {
        using (var r = new Resource())
        {
            // use r
        }
        // Only "Disposed" prints; finalizer is suppressed.
    }
}
```

**Expected Output**
```
Disposed
```

**Why this output occurs**
`Dispose` calls `GC.SuppressFinalize(this)`, so the finalizer is not invoked.

---

### Real-World Cases

- **Dispose pattern** — Every `IDisposable` implementation.
- **Resource wrappers** — Classes that wrap unmanaged handles.

---

### References

- GC.SuppressFinalize Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.suppressfinalize
- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

---

## 6. Finalizers and Destructors

### Definitions

**Core Definition**
A finalizer (historically called a **destructor**) is a method that runs **before an object is garbage-collected**, used to release unmanaged resources as a safety net.

**Technical Definition**
In C#, finalizers are declared with `~ClassName()`. They are called by the GC before reclaiming the object's memory, but **non-deterministically** — you cannot control when they run. Objects with finalizers are **promoted** to Gen 1 (and eventually Gen 2) and their finalizers run on a dedicated finalizer thread. Finalizers should only release **unmanaged resources**; they must not access other managed objects (which may already be finalized). The Dispose pattern calls `GC.SuppressFinalize` to avoid redundant finalization when `Dispose` has already cleaned up.

**Beginner-Friendly Explanation**
A finalizer is a **cleanup method that runs when the GC is about to collect the object**. It is a safety net for unmanaged resources.

---

### Purposes

- To **release unmanaged resources** if `Dispose` was not called.
- To serve as a **last-resort cleanup** for resource wrappers.

---

### Syntax Rules and Structure

```csharp
~ClassName()
{
    // release unmanaged resources
}
```

**Component breakdown**

- `~ClassName()` — Finalizer syntax.
- Called by the GC.
- Non-deterministic.

#### Syntax Rules

- Only classes can have finalizers.
- Cannot be overloaded or inherited.
- Cannot be called explicitly.
- Should only touch unmanaged resources.

#### Constraints and Limitations

- Non-deterministic; can delay collection.
- Finalizers run on a single finalizer thread (bottleneck).
- Exceptions in finalizers are fatal (process terminates).
- Objects with finalizers survive longer (promotion).

---

### Annotated Code Examples and Expected Outputs

#### Example: Finalizer as Safety Net

```csharp
using System;

public class FileHandle
{
    ~FileHandle() => Console.WriteLine("Finalizer running");
}

class Program
{
    static void Main()
    {
        var handle = new FileHandle();
        handle = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
        Console.WriteLine("Main done");
    }
}
```

**Expected Output**
```
Finalizer running
Main done
```

**Why this output occurs**
After `handle = null` and forced collection, the finalizer runs.

---

### Real-World Cases

- **Unmanaged resource wrappers** — Prefer `SafeHandle`.
- **Legacy interop** — Classes wrapping native handles.

---

### References

- Finalizers — C# Programming Guide — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers

---

## 7. `IDisposable` and the Dispose Pattern

### Definitions

**Core Definition**
`IDisposable` is an interface defining a **deterministic cleanup** method (`Dispose`) that releases resources (managed and/or unmanaged) when the object is no longer needed.

**Technical Definition**
The Dispose pattern provides a standard way to release resources deterministically. The recommended pattern is: implement `IDisposable.Dispose()`, call `Dispose(true)`, and `GC.SuppressFinalize(this)`. The `protected virtual void Dispose(bool disposing)` method checks a `_disposed` flag, releases managed resources when `disposing` is `true`, releases unmanaged resources regardless, and sets `_disposed = true`. The finalizer calls `Dispose(false)` as a safety net.

**Beginner-Friendly Explanation**
`IDisposable` is the **deterministic cleanup contract**. `Dispose` releases resources immediately when you call it (via `using` or explicitly), instead of waiting for GC.

---

### Purposes

- To release resources **deterministically**.
- To free **unmanaged resources** (file handles, native memory).
- To ensure resources are released even if exceptions occur (via `using`).

---

### Syntax Rules and Structure

```csharp
public class MyResource : IDisposable
{
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

    ~MyResource() => Dispose(false);
}
```

**Component breakdown**

- `Dispose()` — Public entry point.
- `Dispose(bool)` — Core logic.
- `GC.SuppressFinalize` — Avoids finalizer.
- `~MyResource()` — Safety net.

#### Syntax Rules

- Call `Dispose(true)` and `GC.SuppressFinalize(this)` from `Dispose()`.
- Release unmanaged resources regardless of `disposing`.
- Set a `_disposed` flag to make `Dispose` idempotent.

#### Constraints and Limitations

- Must not throw exceptions from `Dispose` unless documented.
- Must be idempotent (safe to call multiple times).

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
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing) { /* dispose managed */ }
        // free unmanaged
        _disposed = true;
        Console.WriteLine("Disposed");
    }

    ~MyResource() => Dispose(false);
}

class Program
{
    static void Main()
    {
        using var r = new MyResource();
    }
}
```

**Expected Output**
```
Disposed
```

**Why this output occurs**
`using` calls `Dispose()` deterministically; the finalizer is suppressed.

---

### Real-World Cases

- **File/stream wrappers** — `FileStream` implements `IDisposable`.
- **Database connections** — `DbConnection` implements `IDisposable`.
- **Native handles** — `SafeHandle` implements the pattern.

---

### References

- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose
- IDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.idisposable

---

## 8. `using` Statement

### Definitions

**Core Definition**
The `using` statement guarantees that `Dispose` is called on an `IDisposable` object when the block exits, even if an exception occurs.

**Technical Definition**
`using (var resource = new Resource()) { ... }` is syntactic sugar for:
```csharp
var resource = new Resource();
try { ... }
finally { resource?.Dispose(); }
```
The `using` statement can also be written without braces for a single statement. Multiple resources can be declared in a single `using` statement. It works with any type implementing `IDisposable` (or via pattern-based `Dispose` in C# 8+).

**Beginner-Friendly Explanation**
`using` is **"dispose automatically when you're done."** It guarantees cleanup even on exceptions.

---

### Purposes

- To guarantee `Dispose` is called on scope exit.
- To simplify resource management (no manual `try`/`finally`).
- To ensure cleanup even on exceptions.

---

### Syntax Rules and Structure

```csharp
using (var resource = new Resource())
{
    // use resource
}
// Dispose called here
```

**Component breakdown**

- `using` — Statement keyword.
- `(var resource = ...)` — Resource acquisition.
- `{ ... }` — Scope; `Dispose` called on exit.

#### Syntax Rules

- The object must implement `IDisposable` (or have a compatible `Dispose`).
- `Dispose` is called in `finally`, even on exceptions.
- Multiple resources can be declared: `using (var a = new A()) using (var b = new B()) { ... }`.

#### Constraints and Limitations

- `Dispose` may throw; exceptions propagate.
- Does not suppress finalizers automatically (implementations should call `GC.SuppressFinalize`).

---

### Annotated Code Examples and Expected Outputs

#### Example: `using` Statement

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        using (var writer = new StreamWriter("file.txt"))
        {
            writer.WriteLine("Hello");
        }   // Dispose called here, closing the file
        Console.WriteLine("Done");
    }
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
`StreamWriter.Dispose` closes the file and flushes buffers before the next line.

---

### Real-World Cases

- **File I/O** — `FileStream`, `StreamWriter`, `StreamReader`.
- **Database connections** — `DbConnection`, `DbCommand`.
- **HTTP clients** — `HttpClient` (though usually long-lived).

---

### References

- using statement — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using

---

## 9. `using` Declaration (C# 8)

### Definitions

**Core Definition**
A `using` declaration is a **variable declaration prefixed with `using`** that causes the object to be disposed at the end of the **enclosing scope**.

**Technical Definition**
`using` declarations (C# 8+) provide the same deterministic cleanup as `using` statements but without an extra block. `using var resource = new Resource();` disposes `resource` at the end of the enclosing block (method, loop, etc.). They are especially useful for reducing nesting.

**Beginner-Friendly Explanation**
`using var x = new X();` is **"dispose at the end of the method."** It saves you the block braces.

---

### Purposes

- To **reduce nesting** compared to `using` statements.
- To keep resource acquisition and scope implicit.

---

### Syntax Rules and Structure

```csharp
using var resource = new Resource();
// ...
// Dispose called at end of enclosing scope
```

**Component breakdown**

- `using var` — Declaration with disposal.
- Scope — The enclosing block.

#### Syntax Rules

- The resource is disposed at the end of the enclosing scope.
- Can be used in methods, loops, `if` blocks, etc.
- Requires C# 8+.

#### Constraints and Limitations

- The scope may be larger than intended if placed at method level.

---

### Annotated Code Examples and Expected Outputs

#### Example: `using` Declaration

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        using var writer = new StreamWriter("file.txt");
        writer.WriteLine("Hello");
        Console.WriteLine("Done");
    }   // Dispose called here
}
```

**Expected Output**
```
Done
```

**Why this output occurs**
The `using var` declaration disposes `writer` at the end of `Main`.

---

### Real-World Cases

- **Methods with multiple resources** — Cleaner than nested `using` blocks.
- **Loops** — Dispose at the end of each iteration.

---

### References

- using declaration — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using

---

## 10. `IAsyncDisposable` and `await using`

### Definitions

**Core Definition**
`IAsyncDisposable` is an interface defining an **asynchronous cleanup method** (`DisposeAsync`), used for resources that require asynchronous release (e.g., network streams).

**Technical Definition**
`IAsyncDisposable.DisposeAsync()` returns a `ValueTask` that completes when cleanup is done. The `await using` statement (or declaration) calls `DisposeAsync` and awaits it. This is essential for resources whose disposal involves asynchronous I/O (e.g., flushing network buffers, closing database connections).

**Beginner-Friendly Explanation**
`IAsyncDisposable` is **async cleanup**. `await using` is **"dispose asynchronously when done."**

---

### Purposes

- To release resources that require asynchronous operations.
- To avoid blocking during disposal in async code.

---

### Syntax Rules and Structure

```csharp
public class AsyncResource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        // asynchronous cleanup
        await Task.Yield();
    }
}

class Program
{
    static async Task Main()
    {
        await using var resource = new AsyncResource();
        // DisposeAsync called and awaited at end of scope
    }
}
```

**Component breakdown**

- `IAsyncDisposable` — Interface.
- `DisposeAsync()` — Async cleanup method.
- `await using` — Statement/declaration that awaits disposal.

#### Syntax Rules

- `await using` can only be used in an `async` method.
- The resource must implement `IAsyncDisposable` (or have a compatible `DisposeAsync`).
- `await using var` — Declaration form (C# 8+).

#### Constraints and Limitations

- Some resources implement both `IDisposable` and `IAsyncDisposable`; prefer `await using` in async code.

---

### Annotated Code Examples and Expected Outputs

#### Example: `await using`

```csharp
using System;
using System.Threading.Tasks;

public class AsyncResource : IAsyncDisposable
{
    public ValueTask DisposeAsync()
    {
        Console.WriteLine("Disposed asynchronously");
        return ValueTask.CompletedTask;
    }
}

class Program
{
    static async Task Main()
    {
        await using var resource = new AsyncResource();
        Console.WriteLine("Using resource");
    }
}
```

**Expected Output**
```
Using resource
Disposed asynchronously
```

**Why this output occurs**
`await using` calls `DisposeAsync` at the end of scope and awaits its completion.

---

### Real-World Cases

- **Network streams** — `NetworkStream` (or custom wrappers) with async flush.
- **Database connections** — `DbConnection.DisposeAsync`.
- **Async file handles** — `FileStream.DisposeAsync`.

---

### References

- IAsyncDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.iasyncdisposable

---

## 11. GC Best Practices

### General Guidelines

```
// 1. Do not call GC.Collect() in production
// Let the GC choose when to run.

// 2. Implement IDisposable for types holding resources
// Use using / using var / await using

// 3. Call GC.SuppressFinalize in Dispose
// Prevent redundant finalization.

// 4. Prefer SafeHandle over finalizers
// SafeHandle handles finalization correctly.

// 5. Avoid finalizers unless necessary
// Non-deterministic, delay collection, bottleneck.

// 6. Reduce allocations in hot paths
// Use Span<T>, ArrayPool<T>, stackalloc.

// 7. Be mindful of LOH
// Reuse large buffers with ArrayPool<T>.

// 8. Choose the right GC mode
// Server GC for servers; Workstation GC for clients.
```

### Purposes

- To minimize GC pauses and throughput impact.
- To ensure deterministic resource cleanup.
- To avoid memory leaks and fragmentation.
- To choose the right GC mode for the workload.

### Rules and Constraints

- **Do** implement `IDisposable` for resource holders.
- **Do** use `using` / `using var` / `await using`.
- **Do** call `GC.SuppressFinalize` in `Dispose`.
- **Do** prefer `SafeHandle` over finalizers.
- **Do not** call `GC.Collect()` in production.
- **Do not** rely on finalizers for deterministic cleanup.
- **Do not** allocate large buffers repeatedly (LOH).

### Real-World Cases

- **Web servers** — Server GC, allocation reduction, pooling.
- **Desktop apps** — Workstation GC, `Dispose` for UI resources.
- **Native interop** — `SafeHandle` + `IDisposable`.

### References

- Fundamentals of garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose

---

## 12. GC Performance

### Definitions

**Core Definition**
GC performance refers to the **throughput, latency, and memory footprint** impact of garbage collection on an application.

**Technical Definition**
GC performance is dominated by **allocation rate**, **object lifetimes**, **LOH usage**, **GC mode**, and **heap size**. Key metrics: **GC pause time** (latency), **GC throughput** (percentage of time not spent in GC), and **heap size**. Reducing allocations, avoiding LOH churn, using **Server GC** for throughput, and **Background GC** for latency are the main levers. **Pooling** (`ArrayPool<T>`, `MemoryPool<T>`) and **stack-friendly types** (`Span<T>`, `struct`) reduce allocation pressure.

**Beginner-Friendly Explanation**
GC performance is **how much time and memory the GC consumes**. Fewer allocations and the right GC mode mean less time in GC and lower latency.

---

### Purposes

- To reduce GC pauses and throughput impact.
- To avoid LOH fragmentation and out-of-memory issues.
- To choose the right GC mode.
- To optimize allocation-heavy code.

---

### Guidelines

```
// 1. Reduce allocations
// Use Span<T>, stackalloc, and ArrayPool<T>.

// 2. Avoid LOH churn
// Use ArrayPool<T> for large buffers.

// 3. Choose Server GC for throughput
<ServerGarbageCollection>true</ServerGarbageCollection>

// 4. Choose Workstation GC for client responsiveness
// (default for desktop apps)

// 5. Use Background GC for low latency
<ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>

// 6. Pre-size collections to avoid resizing
var list = new List<int>(capacity: 1000);
```

#### Constraints and Limitations

- Server GC increases memory usage.
- Background GC does not eliminate pauses entirely.
- LOH compaction is optional and expensive.

---

### Annotated Code Examples and Expected Outputs

#### Example: Using `ArrayPool<T>` to Reduce LOH Pressure

```csharp
using System;
using System.Buffers;

class Program
{
    static void Main()
    {
        var buffer = ArrayPool<byte>.Shared.Rent(100_000);   // may reuse an existing LOH buffer
        try
        {
            // use buffer
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(buffer);
        }
        Console.WriteLine("Pooled buffer returned");
    }
}
```

**Expected Output**
```
Pooled buffer returned
```

**Why this output occurs**
`ArrayPool<T>` reuses large buffers, avoiding repeated LOH allocations and fragmentation.

---

### Real-World Cases

- **ASP.NET Core** — Server GC and allocation reduction.
- **Game engines** — Avoiding GC pauses.
- **Data processing** — Pooling large buffers.

---

### References

- Fundamentals of garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
- Workstation and server garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `GC.Collect()` | **Discouraged** in production; use only in tests/diagnostics. |
| Finalizers (`~ClassName`) | **Non-deterministic**; prefer `IDisposable`/`SafeHandle`. |
| `SafeHandle` | **Recommended** for unmanaged resources. |
| `using` declaration (C# 8) | Introduced in **C# 8.0**; version-specific. |
| `IAsyncDisposable` / `await using` | Introduced in **C# 8.0 / .NET Core 3.0**; version-specific. |
| Server GC | **Default for ASP.NET Core**; configurable. |
| Background GC | **Enabled by default**; not available in all configurations. |
| LOH threshold (85,000 bytes) | **Implementation detail**. |
| LOH compaction | **Off by default**; optional via `GCSettings.LargeObjectHeapCompactionMode`. |
| `GC.SuppressFinalize` | **Required** in the Dispose pattern. |

---

## References (Consolidated)

- Fundamentals of garbage collection — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
- Workstation and server garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/workstation-server-gc
- Background garbage collection — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/background-gc
- Implement a Dispose method — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose
- GC.Collect Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.collect
- GC.SuppressFinalize Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.gc.suppressfinalize
- Finalizers — C# Programming Guide — https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
- IDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.idisposable
- IAsyncDisposable Interface — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.iasyncdisposable
- using statement — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using
- using declaration — C# reference — https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using
- Large object heap (LOH) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap

---