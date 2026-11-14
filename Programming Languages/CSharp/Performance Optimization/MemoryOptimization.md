# Memory Optimization in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Memory optimization in C# is the discipline of reducing the volume, frequency, and cost of memory allocations and garbage collection (GC) activity in order to improve application latency, throughput, and resource efficiency.

**Technical Definition**
The .NET runtime manages memory through a generational, compacting garbage collector. Objects are allocated on the managed heap in **Generation 0 (Gen0)**; survivors are promoted to **Gen1** and then **Gen2**. Objects of **85,000 bytes or larger** are allocated on the **Large Object Heap (LOH)**, which is collected only during Gen2 collections and is not compacted by default. Every allocation increases GC pressure: a high allocation rate triggers frequent Gen0 collections, and large or long-lived allocations contribute to expensive Gen2 pauses and LOH fragmentation. Memory optimization therefore targets the **allocation rate** and the **lifetime profile** of objects. The .NET platform provides a spectrum of tools: **value types and `readonly struct`** to keep data off the heap, **`stackalloc`** for small stack-only buffers, **`Span<T>` / `ReadOnlySpan<T>`** for zero-allocation slicing of contiguous memory, **`Memory<T>` / `ReadOnlyMemory<T>`** for heap-storable or async-compatible buffers, **`ArrayPool<T>`** and **`MemoryPool<T>`** for renting and reusing buffers, and **`ObjectPool<T>`** for reusing expensive-to-initialize reference types.

**Beginner-Friendly Explanation**
Memory optimization is “using less memory and cleaning up less often.” In .NET, every time you create an object, the garbage collector eventually has to clean it up, and that cleanup costs CPU time. If your code creates millions of temporary objects, the GC runs constantly and your application slows down. Memory optimization means reusing buffers instead of creating new ones, using lightweight “views” over existing data instead of copying it, and keeping small data on the stack instead of the heap. The result is an application that is faster, uses less memory, and scales further.

---

### Key Characteristics

- **Allocation-sensitive** — In most .NET applications, the GC is the dominant performance factor; reducing allocations is usually more effective than tuning GC knobs.
- **Generational** — Gen0 collections are cheap and frequent; Gen2 collections are expensive and rare; LOH allocations are collected with Gen2 and can fragment.
- **Layered primitives** — `stackalloc` → `Span<T>` → `Memory<T>` → `ArrayPool<T>` / `MemoryPool<T>` → `ObjectPool<T>` form a hierarchy from cheapest/shortest-lived to most flexible/longest-lived.
- **Zero-allocation achievable** — `Span<T>`, `stackalloc`, and pooled buffers can eliminate allocations entirely on hot paths.
- **Correctness-critical** — Pooled buffers must be returned exactly once; `Span<T>` cannot cross `await` boundaries; `Memory<T>` ownership must be tracked.
- **Measurement-driven** — Optimize only after profiling shows allocation pressure is the bottleneck; premature pooling can be slower than allocation.

---

### Prerequisites

- Working knowledge of C# classes, structs, arrays, and `async`/`await`.
- Understanding of the .NET garbage collector: generations, LOH, and GC modes.
- Familiarity with the .NET CLI and a profiling tool (e.g., `dotnet-counters`, Visual Studio Profiler).
- Awareness of the difference between value types and reference types, and between stack and heap.
- Basic understanding of `IDisposable` and resource lifetime.

---

### Related Programming Areas

- **Performance fundamentals** — Latency, throughput, and resource utilization.
- **Profiling and benchmarking** — Identifying allocation hotspots with `dotnet-trace`, `dotnet-gcdump`, and BenchmarkDotNet.
- **Garbage collection tuning** — Server GC, DATAS, and LOH compaction.
- **Span and memory primitives** — The `System.Memory` and `System.Buffers` APIs.
- **Pooling** — `ArrayPool<T>`, `MemoryPool<T>`, and `Microsoft.Extensions.ObjectPool`.
- **High-performance I/O** — `PipeReader`/`PipeWriter`, `IBufferWriter<T>`, and `Utf8Formatter`/`Utf8Parser`.

---

### Core Concepts / Features

1. Memory Allocation
2. Object Pooling
3. `ArrayPool<T>`
4. `MemoryPool<T>`
5. `Span<T>`
6. `Memory<T>`
7. Memory Optimization Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Memory Allocation

### Definitions

**Core Definition**
Memory allocation is the process by which a program requests and receives a block of memory from the runtime to store data.

**Technical Definition**
In .NET, managed memory is allocated on the **managed heap** via the GC allocator. Small objects (< 85,000 bytes) are allocated in Gen0; large objects (≥ 85,000 bytes) are allocated on the **Large Object Heap (LOH)** and collected during Gen2. Allocation is fast (pointer bump) but the subsequent collection is not free: Gen0 collections are frequent but cheap; Gen1 and Gen2 collections are progressively more expensive; LOH collections can cause pauses and fragmentation. Additionally, **boxing** (converting a value type to `object` or an interface) allocates on the heap, and **closure captures** in lambdas allocate a display class. The allocation rate — bytes allocated per second — is the primary driver of GC frequency. Reducing allocations, rather than tuning the GC, is the most effective optimization.

**Beginner-Friendly Explanation**
Memory allocation is “asking the runtime for space to store something.” Every `new` object, every string concatenation, every LINQ query, and every boxing conversion asks for memory. The garbage collector later has to clean it up. If you allocate too much, the GC runs often and your app slows down.

---

### Purposes

- To understand where and why memory is consumed.
- To identify the allocation sources that drive GC pressure.
- To choose the right allocation strategy (stack, pool, or heap).
- To avoid hidden allocations such as boxing and closures.
- To reduce Gen0 collection frequency and Gen2 pause duration.
- To keep LOH allocations minimal and well-managed.

---

### Syntax Rules and Structure

#### General Allocation Syntax

```csharp
// Heap allocation (managed heap, GC-collected)
var obj = new MyClass();
var array = new byte[1024];
var boxed = (object)42;              // boxing — heap allocation
var closure = () => Console.WriteLine(value); // capture — display class allocation

// Stack allocation (no GC, method-lifetime)
Span<byte> stackBuffer = stackalloc byte[256];

// Pooled allocation (rented, must be returned)
var rented = ArrayPool<byte>.Shared.Rent(1024);
try { /* use rented */ }
finally { ArrayPool<byte>.Shared.Return(rented); }

// Zero-allocation view (no allocation)
ReadOnlySpan<char> slice = "Hello".AsSpan(0, 3);
```

**Component breakdown**

- `new` — Heap allocation; managed by GC.
- `stackalloc` — Stack allocation; no GC; limited to ~256–512 bytes for safety.
- `ArrayPool<T>.Shared.Rent` — Rents a pooled array; must be returned.
- `AsSpan` — Returns a `Span<T>` or `ReadOnlySpan<T>` view over existing memory without allocation.
- **Boxing** — Value type → `object`/interface; heap allocation.
- **Closure** — Lambda capturing outer variables; display class allocation.

#### Syntax Rules

- Prefer `Span<T>`/`Memory<T>` over arrays for buffer operations.
- Use `stackalloc` only for small, fixed-size buffers (≤ 512 bytes recommended).
- Return pooled buffers exactly once, in a `finally` block.
- Avoid boxing: use generic constraints (`where T : struct, IFoo`) instead of casting structs to interfaces.
- Mark lambdas `static` when they do not capture variables, to avoid closure allocations.
- Avoid `params object[]` and `string.Format` in hot paths; prefer `params ReadOnlySpan<T>` overloads.

#### Constraints and Limitations

- `stackalloc` buffers cannot be stored in fields or used across `await` boundaries.
- Pooled buffers are not zero-initialized by default (unless `clearArray: true` is used on return).
- `Span<T>` is a `ref struct` and cannot be boxed, stored on the heap, or used in `async` methods.
- LOH allocations (≥ 85,000 bytes) are expensive to collect and can fragment the heap.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Hidden Allocations

```csharp
using System;

public class HiddenAllocations
{
    // ❌ Boxing: value type → object
    public void Box(int value) => Console.WriteLine((object)value);

    // ❌ Closure: lambda captures 'value'
    public Action Capture(int value) => () => Console.WriteLine(value);

    // ✅ No boxing: generic constraint
    public void NoBox<T>(T value) where T : struct, IFormattable
        => Console.WriteLine(value.ToString(null, null));

    // ✅ Static lambda: no closure
    public Action StaticLambda(int value)
    {
        Console.WriteLine(value);
        return static () => Console.WriteLine("done");
    }
}
```

**Expected Output (conceptual)**

```
Box allocates 24 bytes per call (boxed int + object header).
Capture allocates a display class per call.
NoBox and StaticLambda allocate zero bytes on the hot path.
```

**Why this output occurs**

- `(object)value` boxes the `int` into a heap object.
- The lambda `() => Console.WriteLine(value)` captures `value`, forcing a display class allocation.
- The generic constraint `where T : struct, IFormattable` allows the JIT to specialize and avoid boxing.
- `static` lambdas do not capture and are cached as a single delegate instance.

---

#### Example 2: stackalloc vs. Heap Array

```csharp
using System;

public class StackVsHeap
{
    // ✅ Stack allocation: no GC
    public int StackAllocSum()
    {
        Span<int> buffer = stackalloc int[256];
        for (int i = 0; i < buffer.Length; i++) buffer[i] = i;
        int sum = 0;
        foreach (var x in buffer) sum += x;
        return sum;
    }

    // ❌ Heap allocation: Gen0 GC pressure
    public int HeapAllocSum()
    {
        var buffer = new int[256];
        for (int i = 0; i < buffer.Length; i++) buffer[i] = i;
        int sum = 0;
        foreach (var x in buffer) sum += x;
        return sum;
    }
}
```

**Expected Output (conceptual)**

```
StackAllocSum: 0 bytes allocated on the managed heap.
HeapAllocSum: 2,048 bytes allocated per call (256 × 4 bytes + header).
```

**Why this output occurs**

- `stackalloc` places the buffer on the stack; it is released when the method returns and never touches the GC.
- `new int[256]` allocates a Gen0 array; repeated calls drive Gen0 collection frequency.

---

### Real-World Cases

- **ASP.NET Core request pipelines** — Per-request allocations of headers, buffers, and DTOs drive Gen0 GC; minimizing them reduces latency.
- **Serialization and parsing** — `string.Substring` and `byte[]` allocations in hot loops; replaced by `Span<T>` and pooled buffers.
- **Game loops** — Per-frame allocations cause GC spikes and frame drops; pooling and stack allocation eliminate them.
- **High-frequency APIs** — LOH allocations from large response buffers; chunking below 85,000 bytes avoids LOH.

---

### References

- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/ff8de6dc1b80916adce4bbed98a81f38edc4e19c/.claude/skills/dotnet-performance/SKILL.md
- SOTA-skills .NET performance — martinholovsky/SOTA-skills - https://github.com/martinholovsky/SOTA-skills/blob/main/skills/sota-dotnet/rules/05-performance.md
- Optimizing memory allocation — SkillMD.ai - https://skillmd.ai/ja/skills/optimizing-memory-allocation/SKILL.md
- Memory- and span-related types — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/

---

## 2. Object Pooling

### Definitions

**Core Definition**
Object pooling is the practice of reusing a set of pre-initialized objects instead of allocating and garbage-collecting new instances for each use.

**Technical Definition**
An object pool maintains a collection of reusable objects. A caller **rents** an object from the pool, uses it, and **returns** it. The pool retains returned objects up to a configurable limit and discards excess objects for GC. In .NET, the primary abstraction is `Microsoft.Extensions.ObjectPool.ObjectPool<T>`, with `DefaultObjectPool<T>` and `DefaultPooledObjectPolicy<T>` providing a default implementation. Pools are thread-safe. Objects that implement `IDisposable` are disposed if not returned; if the pool itself is disposed, all pooled items are disposed. Pooling is beneficial when objects are **expensive to allocate/initialize**, represent a **limited resource**, or are **used predictably and frequently** — and it can be slower when initialization is cheap.

**Beginner-Friendly Explanation**
Object pooling is “renting a tool instead of buying a new one every time.” If creating an object is expensive (like a `StringBuilder` with a large buffer), you keep a set of them around and reuse them. When you are done, you give the object back to the pool so someone else can use it.

---

### Purposes

- To avoid repeated expensive initialization of complex objects.
- To reduce Gen0 and Gen2 allocation pressure.
- To keep long-lived object graphs stable.
- To limit the number of instances of a limited resource.
- To improve throughput in high-frequency scenarios.
- To centralize object reset and cleanup logic.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using Microsoft.Extensions.ObjectPool;

// Create a pool with a default policy
var policy = new DefaultPooledObjectPolicy<MyHeavyObject>();
var pool = new DefaultObjectPool<MyHeavyObject>(policy, maximumRetained: 100);

// Rent and return
var obj = pool.Get();
try
{
    obj.DoWork();
}
finally
{
    pool.Return(obj);
}
```

**Component breakdown**

- `ObjectPool<T>` — The abstraction with `Get()` and `Return()`.
- `PooledObjectPolicy<T>` — Customizes object creation and reset on return.
- `DefaultObjectPool<T>` — The default implementation with a retention limit.
- `maximumRetained` — Maximum number of objects retained; excess objects are GC’d.
- `IResettable` — Automatically resets objects when returned.

#### Syntax Rules

- Use `Get()` to rent and `Return()` to return.
- There is no requirement to return every object; unreturned objects are GC’d.
- If `T` implements `IDisposable`, unreturned objects are disposed when the pool is disposed.
- Implement `IResettable` or a custom `PooledObjectPolicy<T>` to reset state before reuse.
- Use object pooling only after profiling shows allocation is a bottleneck.

#### Constraints and Limitations

- Pooling can be slower than allocation when initialization is cheap.
- Pooled objects are not deallocated until the pool is deallocated.
- The pool does not limit allocations, only retention.
- After the pool is disposed, `Get()` throws `ObjectDisposedException` and `Return()` disposes the item.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Pooling a StringBuilder

```csharp
using System;
using System.Text;
using Microsoft.Extensions.ObjectPool;

public sealed class StringBuilderPoolExample
{
    private static readonly ObjectPool<StringBuilder> Pool =
        new DefaultObjectPool<StringBuilder>(
            new StringBuilderPooledObjectPolicy());

    public string BuildMessage(string name)
    {
        var sb = Pool.Get();
        try
        {
            sb.Append("Hello, ");
            sb.Append(name);
            sb.Append('!');
            return sb.ToString();
        }
        finally
        {
            Pool.Return(sb);
        }
    }
}

// Policy that clears the builder on return
public sealed class StringBuilderPooledObjectPolicy : PooledObjectPolicy<StringBuilder>
{
    public override StringBuilder Create() => new StringBuilder();
    public override bool Return(StringBuilder obj)
    {
        obj.Clear();
        return true;
    }
}
```

**Expected Output (conceptual)**

```
Each call reuses the same StringBuilder instance (up to maximumRetained).
The pool avoids allocating a new StringBuilder and its internal char[] per call.
```

**Why this output occurs**

- `Get()` rents a `StringBuilder` from the pool.
- `Clear()` resets the builder before it is returned.
- The next call reuses the same instance, eliminating allocation.

---

#### Example 2: Pooling with IResettable

```csharp
using Microsoft.Extensions.ObjectPool;

public sealed class ReusableBuffer : IResettable
{
    public byte[] Data { get; private set; } = new byte[1024];
    public int Length { get; private set; }

    public void Write(ReadOnlySpan<byte> source)
    {
        source.CopyTo(Data);
        Length = source.Length;
    }

    public bool TryReset()
    {
        Array.Clear(Data, 0, Length);
        Length = 0;
        return true;
    }
}

var pool = new DefaultObjectPool<ReusableBuffer>(
    new DefaultPooledObjectPolicy<ReusableBuffer>());

var buffer = pool.Get();
try
{
    buffer.Write(new byte[] { 1, 2, 3 });
}
finally
{
    pool.Return(buffer);
}
```

**Expected Output (conceptual)**

```
The buffer is reset automatically when returned, ready for reuse.
No new byte[1024] is allocated on subsequent rentals.
```

**Why this output occurs**

- `IResettable.TryReset` is called by the default policy on `Return`.
- The internal array is cleared and the length is reset.

---

### Real-World Cases

- **ASP.NET Core** — `StringBuilder` pooling for URL generation and response formatting.
- **Serialization** — Pooling `JsonSerializer` state or buffers.
- **Database drivers** — Pooling connection objects (though connection pooling is a specialized form).
- **Game engines** — Pooling bullets, particles, and temporary objects.

---

### References

- Object reuse with ObjectPool in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/1a347f480357e679f40c87738f4d4fe13c458ce8/aspnetcore/performance/ObjectPool.md
- Microsoft.Extensions.ObjectPool — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool
- Optimizing memory allocation — SkillMD.ai - https://skillmd.ai/ja/skills/optimizing-memory-allocation/SKILL.md

---

## 3. `ArrayPool<T>`

### Definitions

**Core Definition**
`ArrayPool<T>` is a .NET class that provides a shared pool of reusable arrays, allowing callers to rent an array of at least a requested minimum length and return it when done.

**Technical Definition**
`System.Buffers.ArrayPool<T>` is a thread-safe array pool. The default instance is `ArrayPool<T>.Shared`. `Rent(int minimumLength)` returns an array that is at least the requested size (often larger due to power-of-two bucketing). `Return(T[] array, bool clearArray = false)` returns the array to the pool. The pool may discard arrays when it is trimmed or when the pool is under pressure. `ArrayPool<T>` is the appropriate choice when an API requires a `T[]` or when you need a non-allocating array without `Memory<T>` semantics. It is designed to reduce GC pressure from temporary arrays.

**Beginner-Friendly Explanation**
`ArrayPool<T>` is “a library of reusable arrays.” Instead of writing `new byte[4096]` every time you need a temporary buffer, you ask the pool for one, use it, and give it back. The next caller can reuse the same array, so the garbage collector does not have to clean up thousands of temporary arrays.

---

### Purposes

- To reduce Gen0 allocation pressure from temporary arrays.
- To avoid LOH allocations for large temporary buffers.
- To provide a non-allocating `T[]` for APIs that require arrays.
- To improve throughput in I/O-heavy and serialization-heavy workloads.
- To centralize buffer reuse without custom pooling code.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using System.Buffers;

// Rent
byte[] buffer = ArrayPool<byte>.Shared.Rent(minimumLength: 4096);

try
{
    // Use buffer (actual length may be larger than 4096)
    int length = 4096;
    Process(buffer.AsSpan(0, length));
}
finally
{
    // Return (clearArray: true for sensitive data)
    ArrayPool<byte>.Shared.Return(buffer);
}
```

**Component breakdown**

- `ArrayPool<T>.Shared` — The shared pool instance.
- `Rent(minimumLength)` — Returns an array of at least the requested length.
- `Return(array, clearArray)` — Returns the array; `clearArray: true` zeroes it for sensitive data.
- `Create(maxArrayLength, maxArraysPerBucket)` — Creates a custom pool with bounded retention.

#### Syntax Rules

- Always return rented arrays, preferably in a `finally` block.
- Use `AsSpan(0, requestedLength)` when the rented array is larger than needed.
- Use `clearArray: true` when the buffer contained sensitive data.
- Do not return the same array twice; this is a bug.
- Do not hold references to rented arrays after returning them.
- Do not use `try/finally` as a substitute for correct ownership; the pool does not track ownership.

#### Constraints and Limitations

- Rented arrays may be larger than requested.
- Pooled arrays are not guaranteed to be zero-initialized.
- The shared pool may be trimmed, discarding idle arrays.
- Returning an array that was not rented from the same pool is undefined behavior.
- `ArrayPool<T>` is not a substitute for `MemoryPool<T>` when `Memory<T>` semantics are required.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Renting and Returning a Buffer

```csharp
using System;
using System.Buffers;

public sealed class FileHasher
{
    public int ComputeHash(ReadOnlySpan<byte> data)
    {
        byte[] buffer = ArrayPool<byte>.Shared.Rent(data.Length);
        try
        {
            data.CopyTo(buffer);
            int hash = 0;
            foreach (var b in buffer.AsSpan(0, data.Length))
                hash = HashCode.Combine(hash, b);
            return hash;
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(buffer);
        }
    }
}
```

**Expected Output (conceptual)**

```
The buffer is rented from the shared pool.
After the method returns, the buffer is returned to the pool for reuse.
Repeated calls do not allocate new byte[] instances.
```

**Why this output occurs**

- `Rent` retrieves a pooled array.
- `AsSpan(0, data.Length)` limits processing to the valid region.
- `Return` puts the array back for the next caller.

---

#### Example 2: Clearing Sensitive Data

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(1024);
try
{
    // ... use buffer for sensitive data ...
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer, clearArray: true);
}
```

**Expected Output (conceptual)**

```
The buffer is zeroed before being returned to the pool.
Subsequent renters cannot observe the previous contents.
```

**Why this output occurs**

- `clearArray: true` zeroes the array on return.

---

### Real-World Cases

- **Kestrel and ASP.NET Core** — Request/response buffers are pooled with `ArrayPool<T>`.
- **Serialization** — `System.Text.Json` rents buffers for UTF-8 encoding.
- **Cryptography** — Temporary byte arrays for hashing and encryption.
- **Network protocols** — Packet buffers rented and returned per message.

---

### References

- ArrayPool<T> — Microsoft Learn (System.Buffers) - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1
- How to use ArrayPool and MemoryPool in C# — InfoWorld - https://www.infoworld.com/article/2252344/how-to-use-arraypool-and-memorypool-in-c.html
- ArrayPool<T>/MemoryPool<T> Best Practice — dotnet/runtime Discussion #48257 - https://github.com/dotnet/runtime/discussions/48257
- Optimizing memory allocation — SkillMD.ai - https://skillmd.ai/ja/skills/optimizing-memory-allocation/SKILL.md

---

## 4. `MemoryPool<T>`

### Definitions

**Core Definition**
`MemoryPool<T>` is a .NET class that provides a pool of `IMemoryOwner<T>` instances, each owning a block of memory that can be accessed as `Memory<T>`.

**Technical Definition**
`System.Buffers.MemoryPool<T>` is an abstraction for pooling memory blocks. `MemoryPool<T>.Shared` returns the default pool. `Rent(int minBufferSize)` returns an `IMemoryOwner<T>` whose `Memory` property provides a `Memory<T>` view. The owner must be disposed to return the memory to the pool. `MemoryPool<T>` is appropriate when you need `Memory<T>` semantics — for example, when the buffer must be stored in a field, passed to an async method, or used where `Span<T>` cannot cross an `await` boundary. It is a higher-level abstraction than `ArrayPool<T>` and is designed for scenarios where the underlying storage is not necessarily a `T[]`.

**Beginner-Friendly Explanation**
`MemoryPool<T>` is “a pool for memory that you can use in async code.” Unlike `ArrayPool<T>`, which gives you a raw array, `MemoryPool<T>` gives you an `IMemoryOwner<T>` that wraps a `Memory<T>`. You can pass that `Memory<T>` to async methods, store it in fields, and then dispose the owner to return the memory to the pool.

---

### Purposes

- To pool memory for async-compatible `Memory<T>` scenarios.
- To avoid allocations when `Span<T>` cannot be used (e.g., across `await`).
- To provide a unified abstraction for pooled memory ownership.
- To support `IBufferWriter<T>` and pipeline scenarios.
- To reduce GC pressure in async I/O and serialization.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using System.Buffers;

// Rent
using IMemoryOwner<byte> owner = MemoryPool<byte>.Shared.Rent(4096);
Memory<byte> memory = owner.Memory;

// Use memory (can be passed to async methods)
await ProcessAsync(memory);

// Disposal returns the memory to the pool
```

**Component breakdown**

- `MemoryPool<T>.Shared` — The shared pool.
- `Rent(minBufferSize)` — Returns an `IMemoryOwner<T>`.
- `IMemoryOwner<T>.Memory` — The `Memory<T>` view.
- `Dispose()` — Returns the memory to the pool.

#### Syntax Rules

- Always dispose the `IMemoryOwner<T>` to return the memory.
- Use `owner.Memory` (not the owner itself) for buffer operations.
- `Memory<T>` can be stored in fields and used in `async` methods; `Span<T>` cannot.
- Do not access the `Memory<T>` after disposing the owner.
- Use `MemoryPool<T>` when you need `Memory<T>`; use `ArrayPool<T>` when you need `T[]`.

#### Constraints and Limitations

- The actual buffer size may be larger than requested.
- `MemoryPool<T>` is an abstraction; the underlying storage may not be a `T[]`.
- The owner must be disposed exactly once.
- Not all `MemoryPool<T>` implementations support all operations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Async Buffer Processing

```csharp
using System;
using System.Buffers;
using System.Threading.Tasks;

public sealed class AsyncProcessor
{
    public async Task<int> ProcessAsync(int size)
    {
        using IMemoryOwner<byte> owner = MemoryPool<byte>.Shared.Rent(size);
        Memory<byte> memory = owner.Memory.Slice(0, size);

        await FillBufferAsync(memory);
        return ComputeHash(memory.Span);
    }

    private Task FillBufferAsync(Memory<byte> buffer) => Task.CompletedTask;
    private int ComputeHash(ReadOnlySpan<byte> data) => data.Length;
}
```

**Expected Output (conceptual)**

```
The buffer is rented from MemoryPool<byte>.Shared.
It is passed as Memory<byte> across the await boundary.
Disposing the owner returns the memory to the pool.
```

**Why this output occurs**

- `Memory<T>` can cross `await` boundaries, unlike `Span<T>`.
- The `using` statement ensures the owner is disposed and the memory is returned.

---

#### Example 2: MemoryPool vs. ArrayPool

```csharp
// ArrayPool: raw T[] array
byte[] array = ArrayPool<byte>.Shared.Rent(1024);
try { /* use array */ }
finally { ArrayPool<byte>.Shared.Return(array); }

// MemoryPool: IMemoryOwner<T> with Memory<T>
using IMemoryOwner<byte> owner = MemoryPool<byte>.Shared.Rent(1024);
Memory<byte> memory = owner.Memory;
// use memory
// disposed automatically at end of using scope
```

**Expected Output (conceptual)**

```
ArrayPool returns byte[]; MemoryPool returns IMemoryOwner<byte>.
MemoryPool supports Memory<T> semantics for async and field storage.
```

**Why this output occurs**

- `ArrayPool<T>` is optimized for `T[]`-based APIs.
- `MemoryPool<T>` is optimized for `Memory<T>`-based APIs and ownership semantics.

---

### Real-World Cases

- **ASP.NET Core pipelines** — `PipeReader` and `PipeWriter` use `MemoryPool<T>`.
- **Async serialization** — `System.Text.Json` uses `MemoryPool<T>` for async writes.
- **Network I/O** — Socket buffers rented as `Memory<T>` and passed to async operations.
- **Streaming APIs** — `IBufferWriter<T>` implementations backed by `MemoryPool<T>`.

---

### References

- MemoryPool<T> — Microsoft Learn (System.Buffers) - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.memorypool-1
- Memory- and span-related types — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/
- Memory<T> and Span<T> usage guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines

---

## 5. `Span<T>`

### Definitions

**Core Definition**
`Span<T>` is a `ref struct` that represents a contiguous region of arbitrary memory — managed heap, stack, or native memory — and provides safe, bounds-checked, zero-allocation access to that memory.

**Technical Definition**
`System.Span<T>` and `System.ReadOnlySpan<T>` are stack-only types (`ref struct`) that wrap a pointer and a length. They can be backed by arrays, `stackalloc` memory, strings, or native memory. Slicing a `Span<T>` produces another `Span<T>` without copying or allocating. `Span<T>` cannot be boxed, stored in fields of classes, used as a generic type argument, or cross an `await` boundary. It is the primary tool for zero-allocation parsing, formatting, and buffer processing in modern .NET. `stackalloc` combined with `Span<T>` provides a safe way to use stack memory for small buffers.

**Beginner-Friendly Explanation**
`Span<T>` is “a window into a piece of memory.” Instead of copying a substring or a slice of an array into a new object, you create a `Span<T>` that looks at the existing memory. No allocation occurs, and the window is bounds-checked for safety. It is the fastest way to work with slices of arrays and strings without copying.

---

### Purposes

- To slice arrays and strings without allocating.
- To parse and format data without creating intermediate strings.
- To use `stackalloc` memory safely.
- To process buffers with zero GC pressure.
- To provide a unified abstraction over managed and native memory.
- To enable high-performance APIs such as `Utf8Parser` and `BinaryPrimitives`.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using System;

// From an array
Span<int> span = array.AsSpan();
Span<int> slice = span.Slice(2, 5);
Span<int> range = span[2..7];

// From a string
ReadOnlySpan<char> chars = "Hello, World".AsSpan(0, 5);

// From stackalloc
Span<byte> buffer = stackalloc byte[256];

// Zero-allocation string parsing
ReadOnlySpan<char> part = text.AsSpan(0, 10);
```

**Component breakdown**

- `AsSpan()` — Creates a `Span<T>` over an array or string.
- `Slice(start, length)` — Returns a sub-span.
- `[start..end]` — Range operator for slicing.
- `stackalloc` — Allocates stack memory; assign to `Span<T>`.
- `ReadOnlySpan<T>` — Read-only view.

#### Syntax Rules

- Use `Span<T>` for synchronous methods; use `Memory<T>` for async methods.
- Use `ReadOnlySpan<T>` for read-only parameters.
- Prefer `Span<T>` over `Substring` for string slicing.
- Keep `stackalloc` sizes small (≤ 512 bytes) to avoid stack overflow.
- Do not use `Span<T>` in `async` methods or store it in fields.
- Use `MemoryMarshal.Cast<TFrom, TTo>` to reinterpret spans for bulk processing.

#### Constraints and Limitations

- `Span<T>` is a `ref struct`; it cannot be boxed or stored on the heap.
- Cannot cross `await` boundaries; use `Memory<T>` instead.
- Cannot be used as a generic type argument.
- Cannot be captured by lambdas.
- `stackalloc` memory is only valid for the lifetime of the method.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Zero-Allocation String Slicing

```csharp
using System;

public sealed class SpanParsing
{
    public static (ReadOnlySpan<char> First, ReadOnlySpan<char> Second) Split(
        ReadOnlySpan<char> input)
    {
        int comma = input.IndexOf(',');
        return (input.Slice(0, comma), input.Slice(comma + 1));
    }
}

// Usage
var result = SpanParsing.Split("Hello,World");
Console.WriteLine(result.First);   // Hello
Console.WriteLine(result.Second);  // World
```

**Expected Output (conceptual)**

```
Hello
World
Zero managed heap allocations per call.
```

**Why this output occurs**

- `AsSpan` and `Slice` create views over the existing string memory.
- No new strings are allocated; the spans reference the original string.

---

#### Example 2: stackalloc with Span

```csharp
public int SumStackBuffer()
{
    Span<int> buffer = stackalloc int[128];
    for (int i = 0; i < buffer.Length; i++) buffer[i] = i;
    int sum = 0;
    foreach (var x in buffer) sum += x;
    return sum;
}
```

**Expected Output (conceptual)**

```
The buffer is allocated on the stack; no GC occurs.
Sum is computed and returned.
```

**Why this output occurs**

- `stackalloc` places the buffer on the stack.
- The `Span<int>` provides safe, bounds-checked access.

---

### Real-World Cases

- **ASP.NET Core request parsing** — Header and query-string parsing with `ReadOnlySpan<char>`.
- **JSON and UTF-8 processing** — `Utf8JsonReader` operates over `ReadOnlySpan<byte>`.
- **Binary protocol parsing** — `BinaryPrimitives.ReadInt32LittleEndian(span)` without unsafe code.
- **High-performance string formatting** — `string.Create` with `Span<char>`.

---

### References

- Span<T> and Memory<T> usage guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines
- Memory- and span-related types — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/
- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/ff8de6dc1b80916adce4bbed98a81f38edc4e19c/.claude/skills/dotnet-performance/SKILL.md

---

## 6. `Memory<T>`

### Definitions

**Core Definition**
`Memory<T>` is a heap-storable, async-compatible wrapper over a contiguous region of memory that can be sliced and passed to asynchronous methods.

**Technical Definition**
`System.Memory<T>` and `System.ReadOnlyMemory<T>` wrap a contiguous memory region and can be stored in fields of classes, used in `async` methods, and passed across `await` boundaries — unlike `Span<T>`. `Memory<T>` provides a `.Span` property for synchronous operations. It was introduced in .NET Core 2.1 alongside `IMemoryOwner<T>` and `MemoryPool<T>` to solve the problem of `Span<T>` being stack-only. `Memory<T>` is backed by managed or native memory and supports slicing via `Slice()` and the range operator. Ownership and lifetime must be managed explicitly: the creator of a `Memory<T>` is responsible for its buffer’s lifetime.

**Beginner-Friendly Explanation**
`Memory<T>` is “a Span that can survive an `await`.” If you need to hold onto a slice of memory across asynchronous calls, or store it in a field, you cannot use `Span<T>` — you use `Memory<T>`. You can get a `Span<T>` from it when you need synchronous access.

---

### Purposes

- To carry buffer slices across `await` boundaries.
- To store buffer references in fields of classes.
- To support asynchronous APIs that process memory.
- To provide a heap-storable alternative to `Span<T>`.
- To enable ownership-based memory management with `IMemoryOwner<T>`.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using System;

// From an array
Memory<byte> memory = new byte[1024].AsMemory();
Memory<byte> slice = memory.Slice(0, 512);

// Store in a field
public sealed class BufferHolder
{
    private Memory<byte> _buffer;
    public BufferHolder(int size) => _buffer = new byte[size].AsMemory();
}

// Use in async method
public async Task ProcessAsync(Memory<byte> buffer)
{
    await FillAsync(buffer);
    int length = buffer.Span.Length; // Span access for synchronous work
}
```

**Component breakdown**

- `AsMemory()` — Creates a `Memory<T>` from an array.
- `Slice(start, length)` — Returns a sub-`Memory<T>`.
- `.Span` — Returns a `Span<T>` for synchronous operations.
- `Memory<T>` can be stored in fields and passed to `async` methods.

#### Syntax Rules

- Use `Memory<T>` when the buffer must cross `await` or be stored in a field.
- Use `ReadOnlyMemory<T>` for read-only buffers.
- Use `.Span` for synchronous operations; do not store the `Span<T>` beyond the current method.
- Manage ownership explicitly: the creator of the `Memory<T>` is responsible for the buffer’s lifetime.
- Use `MemoryPool<T>` to rent `Memory<T>`-backed buffers that can be returned to a pool.

#### Constraints and Limitations

- `Memory<T>` is not a `ref struct`; it can be boxed and stored on the heap, but it does not provide the stack-only safety of `Span<T>`.
- The underlying buffer’s lifetime must be managed by the owner.
- `Memory<T>` cannot be used to access `stackalloc` memory (which is stack-only).
- `Memory<T>` is slightly slower than `Span<T>` due to the additional indirection.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Async Buffer with Memory<T>

```csharp
using System;
using System.Threading.Tasks;

public sealed class AsyncMemoryProcessor
{
    private Memory<byte> _buffer;

    public AsyncMemoryProcessor(int size) => _buffer = new byte[size].AsMemory();

    public async Task<int> ProcessAsync()
    {
        await FillAsync(_buffer);
        return ComputeHash(_buffer.Span);
    }

    private Task FillAsync(Memory<byte> buffer) => Task.CompletedTask;
    private int ComputeHash(ReadOnlySpan<byte> data) => data.Length;
}
```

**Expected Output (conceptual)**

```
The buffer is stored as Memory<byte> in a field.
It is passed to an async method without allocation.
.Span is used for the synchronous hash computation.
```

**Why this output occurs**

- `Memory<T>` can be stored in a field and passed across `await`.
- `.Span` provides zero-allocation synchronous access.

---

#### Example 2: Memory<T> from a Pooled Owner

```csharp
using System.Buffers;

using IMemoryOwner<byte> owner = MemoryPool<byte>.Shared.Rent(1024);
Memory<byte> memory = owner.Memory;

// Use memory in async method
await ProcessAsync(memory);
// owner disposal returns the memory to the pool
```

**Expected Output (conceptual)**

```
Memory<byte> is obtained from a pooled IMemoryOwner<byte>.
Disposing the owner returns the memory to the pool.
```

**Why this output occurs**

- `MemoryPool<T>.Shared.Rent` returns an `IMemoryOwner<T>` whose `Memory` is a `Memory<T>`.
- Disposing the owner releases the memory back to the pool.

---

### Real-World Cases

- **ASP.NET Core `PipeReader`/`PipeWriter`** — Operate on `ReadOnlyMemory<byte>` and `Memory<byte>`.
- **`System.IO.Pipelines`** — Async buffer processing with `Memory<T>`.
- **`System.Text.Json` async serialization** — Uses `Memory<T>` for async writes.
- **Custom async APIs** — Any API that accepts a buffer and awaits I/O.

---

### References

- Memory<T> and Span<T> usage guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines
- Memory- and span-related types — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/
- Span and Memory in C# — CodeMag - https://www.codemag.com/article/1807051/Writing-High-Performance-Code-Using-SpanT-and-MemoryT-in-C

---

## 7. Memory Optimization Best Practices

### Definitions

**Core Definition**
Memory optimization best practices are empirically grounded guidelines for reducing allocation volume, GC pressure, and memory-related latency in .NET applications.

**Technical Definition**
Best practices span allocation avoidance (structs, `Span<T>`, `stackalloc`), pooling (`ArrayPool<T>`, `MemoryPool<T>`, `ObjectPool<T>`), lifetime management (ownership, disposal, `finally` returns), and measurement discipline (profile before optimizing). They are grounded in the GC’s generational architecture and the runtime’s memory primitives. The overarching principle is: **allocate less, not tune more**. Pooling is beneficial only when initialization is expensive and usage is predictable; otherwise, allocation is cheaper.

**Beginner-Friendly Explanation**
Best practices are “the habits that keep your app from drowning in memory.” They include using `Span<T>` for slicing, pooling arrays for temporary buffers, avoiding boxing and closures, and measuring before changing anything.

---

### Purposes

- To reduce Gen0 allocation rate and collection frequency.
- To avoid LOH allocations and fragmentation.
- To eliminate hidden allocations (boxing, closures, `params`).
- To manage pooled buffers correctly and avoid leaks.
- To choose the right primitive for each scenario.
- To validate optimizations with measurement.

---

### Guidelines and Structure

#### Allocation Hierarchy

```
Smallest / Shortest-lived → Largest / Longest-lived

stackalloc + Span<T>
  → Span<T> over existing array/string
  → ArrayPool<T> / MemoryPool<T> (rented)
  → ObjectPool<T> (expensive reference types)
  → new (heap allocation, GC-collected)
```

Choose the highest applicable level for the scenario.

#### Avoid Hidden Allocations

```csharp
// ❌ Boxing
object boxed = 42;

// ✅ Generic constraint
void NoBox<T>(T value) where T : struct, IFormattable
    => value.ToString(null, null);

// ❌ Closure
Func<int> f = () => value;

// ✅ Static lambda
Func<int> g = static () => 42;

// ❌ params object[]
string.Format("{0}", value);

// ✅ params ReadOnlySpan<T> overload
```

- Use generic constraints to avoid boxing.
- Use `static` lambdas when no capture is needed.
- Avoid `params object[]` and `string.Format` in hot paths.

#### Pool Correctly

```csharp
// ArrayPool
byte[] buffer = ArrayPool<byte>.Shared.Rent(size);
try { /* use */ }
finally { ArrayPool<byte>.Shared.Return(buffer); }

// MemoryPool
using IMemoryOwner<byte> owner = MemoryPool<byte>.Shared.Rent(size);
Memory<byte> memory = owner.Memory;

// ObjectPool
var obj = pool.Get();
try { /* use */ }
finally { pool.Return(obj); }
```

- Always return rented buffers exactly once.
- Use `clearArray: true` for sensitive data.
- Do not return an object more than once.

#### Use the Right Primitive

| Scenario | Primitive |
|---|---|
| Synchronous slicing of arrays/strings | `Span<T>` / `ReadOnlySpan<T>` |
| Small temporary buffer (< 512 bytes) | `stackalloc` + `Span<T>` |
| Temporary buffer needing `T[]` | `ArrayPool<T>` |
| Buffer crossing `await` or stored in field | `Memory<T>` / `IMemoryOwner<T>` |
| Expensive reference-type object | `ObjectPool<T>` |
| Read-only lookup tables | `FrozenDictionary` / `FrozenSet` |

#### Minimize LOH Allocations

- Keep individual allocations below **85,000 bytes** to avoid the LOH.
- Split large buffers into chunks or use pooled segmented buffers.
- Avoid short-lived LOH allocations; they trigger expensive Gen2 collections.

#### Measure Before and After

```bash
dotnet-counters monitor --process-id <pid> System.Runtime[alloc-rate,gc-heap-size,gen-2-gc-count]
dotnet-trace collect --process-id <pid>
```

- Use `dotnet-counters` to observe allocation rate and GC frequency.
- Use `dotnet-gcdump` to analyze heap composition.
- Use BenchmarkDotNet to validate micro-optimizations.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Correct Pooling with finally

```csharp
public int ProcessBuffer(ReadOnlySpan<byte> input)
{
    byte[] buffer = ArrayPool<byte>.Shared.Rent(input.Length);
    try
    {
        input.CopyTo(buffer);
        return ComputeHash(buffer.AsSpan(0, input.Length));
    }
    finally
    {
        ArrayPool<byte>.Shared.Return(buffer);
    }
}
```

**Expected Output (conceptual)**

```
The buffer is rented once and returned in finally.
Repeated calls reuse pooled arrays; allocation rate drops to near zero.
```

**Why this output occurs**

- The `finally` block guarantees return even if `ComputeHash` throws.

---

#### Example 2: Span-Based Parsing (Zero Allocation)

```csharp
public static bool TryParseVersion(ReadOnlySpan<char> input, out int major, out int minor)
{
    major = 0; minor = 0;
    int dot = input.IndexOf('.');
    if (dot < 0) return false;

    return int.TryParse(input.Slice(0, dot), out major)
        && int.TryParse(input.Slice(dot + 1), out minor);
}

// Usage
var span = "1.2".AsSpan();
if (TryParseVersion(span, out var major, out var minor))
    Console.WriteLine($"{major}.{minor}");
```

**Expected Output**

```
1.2
Zero heap allocations per call.
```

**Why this output occurs**

- `AsSpan` and `Slice` avoid string allocation.
- `int.TryParse` has a `ReadOnlySpan<char>` overload that parses without allocating.

---

### Real-World Cases

- **ASP.NET Core middleware** — Zero-allocation header parsing with `ReadOnlySpan<char>`.
- **High-frequency trading** — Per-operation allocation budgets of zero bytes.
- **Game servers** — Pooled network buffers and object pools for entities.
- **Serialization libraries** — Pooled UTF-8 buffers and `IBufferWriter<T>`.

---

### References

- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/ff8de6dc1b80916adce4bbed98a81f38edc4e19c/.claude/skills/dotnet-performance/SKILL.md
- SOTA-skills .NET performance — martinholovsky/SOTA-skills - https://github.com/martinholovsky/SOTA-skills/blob/main/skills/sota-dotnet/rules/05-performance.md
- Object reuse with ObjectPool — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/1a347f480357e679f40c87738f4d4fe13c458ce8/aspnetcore/performance/ObjectPool.md
- ArrayPool<T>/MemoryPool<T> Best Practice — dotnet/runtime Discussion #48257 - https://github.com/dotnet/runtime/discussions/48257
- Memory<T> and Span<T> usage guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`Span<T>` in `async` methods** | Not allowed; `ref struct` cannot cross `await`; use `Memory<T>` |
| **`stackalloc` for large buffers** | Risk of stack overflow; keep ≤ 512 bytes |
| **Returning a pooled array twice** | Bug; undefined behavior |
| **`try/finally` as ownership substitute** | Not a substitute for correct ownership; pool does not track rentals |
| **Pooled arrays without `clearArray`** | May contain sensitive data from previous use; use `clearArray: true` |
| **`ObjectPool<T>` with cheap objects** | Can be slower than allocation; pool only expensive objects |
| **LOH allocations (≥ 85,000 bytes)** | Collected during Gen2; cause pauses and fragmentation; split or pool |
| **`Span<T>` as a generic type argument** | Not allowed; `ref struct` restriction |
| **Storing `Span<T>` in a field** | Not allowed; use `Memory<T>` |
| **`MemoryPool<T>` without disposing owner** | Memory is not returned to the pool; leaks pooled resources |
| **Server GC DATAS** | On by default in .NET 9+; may increase startup allocation; disable for throughput-bound single-owner services |

---

## References (Consolidated)

- .NET Performance Guidelines — xian55/hermesproxy - https://github.com/xian55/hermesproxy/blob/ff8de6dc1b80916adce4bbed98a81f38edc4e19c/.claude/skills/dotnet-performance/SKILL.md
- SOTA-skills .NET performance — martinholovsky/SOTA-skills - https://github.com/martinholovsky/SOTA-skills/blob/main/skills/sota-dotnet/rules/05-performance.md
- Optimizing memory allocation — SkillMD.ai - https://skillmd.ai/ja/skills/optimizing-memory-allocation/SKILL.md
- Object reuse with ObjectPool in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/1a347f480357e679f40c87738f4d4fe13c458ce8/aspnetcore/performance/ObjectPool.md
- Microsoft.Extensions.ObjectPool — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/performance/objectpool
- ArrayPool<T> — Microsoft Learn (System.Buffers) - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1
- MemoryPool<T> — Microsoft Learn (System.Buffers) - https://learn.microsoft.com/en-us/dotnet/api/system.buffers.memorypool-1
- Memory- and span-related types — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/
- Memory<T> and Span<T> usage guidelines — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines
- ArrayPool<T>/MemoryPool<T> Best Practice — dotnet/runtime Discussion #48257 - https://github.com/dotnet/runtime/discussions/48257
- How to use ArrayPool and MemoryPool in C# — InfoWorld - https://www.infoworld.com/article/2252344/how-to-use-arraypool-and-memorypool-in-c.html
- Span and Memory in C# — CodeMag - https://www.codemag.com/article/1807051/Writing-High-Performance-Code-Using-SpanT-and-MemoryT-in-C
- Writing High-Performance Code Using Span<T> and Memory<T> in C# — CodeMag - https://www.codemag.com/article/1807051/Writing-High-Performance-Code-Using-SpanT-and-MemoryT-in-C