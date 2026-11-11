# C# Memory Model — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The **C# memory model** describes how a .NET program's data is laid out in memory and how threads observe reads and writes to that data. It covers the **stack** (per-thread, LIFO storage for locals and call frames), the **heap** (shared, garbage-collected storage for objects), the **Managed Heap** (the CLR-managed portion of the heap), and the **Large Object Heap (LOH)** (a special heap segment for large arrays and objects). Understanding this model is essential for writing correct concurrent code and for diagnosing performance and memory issues.

**Technical Definition**
The C# memory model is a set of rules governing **memory layout** and **memory ordering** (visibility of writes across threads). The memory layout consists of the **stack** (fast, thread-private, stores value-type locals and references), the **managed heap** (where reference-type instances and boxed value types live, managed by the garbage collector), and the **LOH** (objects ≥ 85,000 bytes, collected only in Gen 2). The memory ordering model defines the guarantees about when writes made by one thread become visible to another — .NET's model is a *weak* memory model with specific guarantees: **`lock`/`Monitor`**, **`Volatile`**, and **`Interlocked`** operations are synchronization points that enforce memory barriers. The ECMA C# specification defines the **thread-safe** aspects of the model, while the CLI specification defines the memory model in detail. Understanding this is crucial for lock-free programming, publication safety, and avoiding data races.

**Beginner-Friendly Explanation**
The memory model is the **“where does data live and who can see it”** rulebook. Data lives in two places: the **stack** (fast, private to each thread, for local variables and method calls) and the **heap** (shared, garbage-collected, for objects). The **managed heap** is where the .NET runtime keeps objects; it is automatically cleaned up by the GC. The **LOH** is a special heap segment for very large objects (≥ 85 KB), which are expensive to move and are collected only during full (Gen 2) GCs. On top of the layout, the memory model also defines **visibility**: when one thread writes to a field, when does another thread see the new value? The answer is *it depends on synchronization* — without `lock`, `Volatile`, or `Interlocked`, other threads may see stale values or reordered operations.

---

### Key Characteristics

- **Two storage areas** — Stack (thread-private, LIFO, for value-type locals and call frames) and heap (shared, GC-managed, for objects).
- **Managed heap vs. native heap** — The .NET GC manages the managed heap; native memory (allocated via `Marshal.AllocHGlobal`) is outside GC control.
- **Generational GC** — The managed heap is divided into Gen 0, Gen 1, and Gen 2; short-lived objects die young and are collected cheaply.
- **LOH is special** — Objects ≥ 85,000 bytes go to the Large Object Heap, which is collected only during Gen 2 collections and can fragment.
- **Weak memory model** — Without synchronization, the JIT and CPU can reorder memory operations; visibility is not automatic.
- **Synchronization points** — `lock`/`Monitor`, `Volatile`, `Interlocked`, and `Task` coordination act as memory barriers.
- **Stack is not always stack** — The C# spec only requires that lifetimes are respected; the *physical* location (stack vs. heap) is an implementation detail.
- **Publication safety** — Properly publishing an object across threads requires a memory barrier (e.g., `Volatile.Write` or a lock).

---

### Prerequisites

- Basic understanding of **value types vs. reference types** in C#.
- Familiarity with **classes**, **structs**, **fields**, and **local variables**.
- Awareness of **garbage collection** (you should know that the GC reclaims unreachable objects).
- Basic understanding of **threads** and **synchronization** (`lock`, `Interlocked`).
- Knowledge of `IDisposable` and unmanaged resources (helpful but not required).

---

### Related Programming Areas

- **Garbage collection** — The GC manages the managed heap; understanding its generations and LOH helps you write allocation-efficient code.
- **Concurrency** — The memory model's ordering rules determine correctness of multithreaded code.
- **Performance engineering** — Allocation patterns, LOH usage, and GC pressure are dominant performance factors.
- **Interop** — Pinning, `fixed`, and native memory interact with the managed heap.
- **Spans and ref structs** — These modern C# features exploit the distinction between stack and heap.

---

### Core Concepts / Features

1. Memory Model (Overarching Concept)
2. Stack
3. Heap
4. Managed Heap
5. Large Object Heap (LOH)
6. Memory Allocation
7. Memory Layout
8. Memory Model Best Practices

---

## 1. Memory Model (Overarching Concept)

### Definitions

**Core Definition**
The C# memory model defines how program data is laid out in memory (stack vs. heap) and how threads observe memory operations (ordering and visibility).

**Technical Definition**
The memory model has two aspects: **layout** (stack, managed heap, LOH) and **ordering** (visibility of writes across threads). The layout determines where value-type locals, references, and object instances are stored. The ordering rules define when writes made by one thread become visible to another — .NET uses a **weak memory model** with synchronization points (`lock`, `Volatile`, `Interlocked`) that enforce memory barriers. The ECMA C# specification and the CLI specification define the guarantees; the JIT and CPU are free to reorder operations that are not separated by synchronization.

**Beginner-Friendly Explanation**
The memory model is **where data lives and who can see it**. The layout is stack vs. heap; the ordering is "when does another thread see my write?"

---

### Purposes

- To understand where variables and objects are stored.
- To write correct concurrent code that respects ordering and visibility rules.
- To optimize performance by understanding GC and allocation behavior.
- To diagnose memory leaks, fragmentation, and LOH issues.
- To choose appropriate synchronization primitives.

---

### Syntax Rules and Structure (Conceptual Model)

```
┌───────────────────────────┐
│        Stack (per thread)  │  ← value-type locals, references, call frames
├───────────────────────────┤
│        Managed Heap        │  ← GC-managed objects (Gen 0, 1, 2)
│  ┌─────────────────────┐  │
│  │  Large Object Heap  │  │  ← objects ≥ 85 KB
│  └─────────────────────┘  │
├───────────────────────────┤
│        Native Heap         │  ← unmanaged allocations (Marshal.AllocHGlobal)
└───────────────────────────┘
```

**Component breakdown**

- **Stack** — Fast, thread-private, LIFO allocation for locals and call frames.
- **Managed Heap** — GC-managed, shared, generational.
- **LOH** — Part of managed heap for objects ≥ 85 KB.
- **Native Heap** — Outside GC, must be freed manually.

#### Syntax Rules

- Reference-type instances (classes, arrays, strings, boxed value types) live on the managed heap.
- Value-type locals and references live on the stack (implementation detail).
- Objects ≥ 85,000 bytes go to the LOH.
- Memory ordering requires synchronization for cross-thread visibility.

#### Constraints and Limitations

- The specification does not mandate physical stack allocation; lifetimes are guaranteed, but locations are not.
- The LOH is collected only during Gen 2 GCs, causing pauses and fragmentation.
- Without synchronization, reads may see stale values or operations may be reordered.

---

### Annotated Code Examples and Expected Outputs

#### Example: Stack vs. Heap

```csharp
using System;

struct Point   // value type
{
    public int X, Y;
}

class Person   // reference type
{
    public string Name;
}

class Program
{
    static void Main()
    {
        Point p = new Point { X = 1, Y = 2 };          // value type (stack)
        Person person = new Person { Name = "Alice" }; // reference on stack, object on heap

        Console.WriteLine($"Point: {p.X},{p.Y}");
        Console.WriteLine($"Person: {person.Name}");
    }
}
```

**Expected Output**
```
Point: 1,2
Person: Alice
```

**Why this output occurs**
`Point` is a value type; its data is stored directly in the local `p` (typically on the stack). `person` is a reference variable (typically on the stack) pointing to a `Person` object on the managed heap.

---

### Real-World Cases

- **High-performance services** — Understanding stack vs. heap helps reduce allocations.
- **Concurrent code** — Correct memory ordering is essential for lock-free algorithms.
- **Large data processing** — LOH awareness prevents fragmentation and GC pauses.

---

### References

- Memory model — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-model/
- The C# Memory Model in Theory and Practice — MSDN Magazine — https://learn.microsoft.com/en-us/archive/msdn-magazine/2012/december/csharp-the-csharp-memory-model-in-theory-and-practice

---

## 2. Stack

### Definitions

**Core Definition**
The stack is a **thread-private, LIFO memory region** used for method call frames, local variables, and parameter passing.

**Technical Definition**
The stack is a contiguous block of memory managed via a stack pointer. Each method call pushes a **frame** containing parameters, local variables, return address, and saved registers; returning pops the frame. Stack allocation is extremely fast (just moving the pointer) and automatically reclaimed when the method returns. Value-type locals and references are typically allocated on the stack, though captured variables and `async`/iterator state machines may be hoisted to the heap.

**Beginner-Friendly Explanation**
The stack is a **stack of trays**: each method call puts a new tray on top; returning removes it. It is fast and private to the thread.

---

### Purposes

- To provide **fast, automatic allocation** for locals and method state.
- To keep thread-private data isolated (no synchronization needed).
- To support recursion and function calls.
- To enable predictable, scope-based lifetime.

---

### Syntax Rules and Structure

```
void Method()
{
    int x = 42;       // typically stack-allocated
    int y = x + 1;
}   // frame popped here
```

**Component breakdown**

- Method call — Pushes a frame.
- Local variables — Stored in the frame.
- Method return — Pops the frame.

#### Syntax Rules

- Stack frames are LIFO.
- Locals are automatically reclaimed when the frame is popped.
- Captured variables and async/iterator state machines may be moved to the heap.

#### Constraints and Limitations

- Stack size is limited (typically 1 MB per thread).
- Deep recursion or large stack allocations cause `StackOverflowException` (uncatchable).
- Escaping locals (captured by lambdas) are hoisted to the heap.

---

### Annotated Code Examples and Expected Outputs

#### Example: Stack Frame Lifetime

```csharp
using System;

class Program
{
    static void Method()
    {
        int local = 42;   // allocated on stack
        Console.WriteLine(local);
    }   // frame popped; local is gone

    static void Main()
    {
        Method();
        Method();
    }
}
```

**Expected Output**
```
42
42
```

**Why this output occurs**
Each call to `Method` pushes a frame with its own `local`; returning pops the frame.

---

### Real-World Cases

- **Recursive algorithms** — Stack frames hold recursion state.
- **Value-type locals** — Fast, allocation-free computation.
- **`ref struct` / `Span<T>`** — Stack-only types.

---

### References

- The Stack Is An Implementation Detail, Part One — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-one

---

## 3. Heap

### Definitions

**Core Definition**
The heap is a **shared, non-LIFO memory region** where objects with dynamic lifetimes are allocated; in .NET, most of it is managed by the GC.

**Technical Definition**
The heap is a large pool of memory from which allocations can be made and freed in any order. In .NET, reference-type instances (classes, arrays, strings, boxed value types) are allocated on the **managed heap** by the GC. Native memory allocated via `Marshal.AllocHGlobal` or P/Invoke is on the **native heap** and must be freed manually. Heap allocation is slower than stack allocation and requires GC for reclamation.

**Beginner-Friendly Explanation**
The heap is a **warehouse** where objects live until the GC cleans them up. Unlike the stack, objects can outlive the method that created them.

---

### Purposes

- To store objects with dynamic lifetimes.
- To allow sharing of objects across methods and threads.
- To support variable-sized data (arrays, strings).
- To enable object identity (references).

---

### Syntax Rules and Structure

```
var person = new Person();   // allocated on managed heap
```

**Component breakdown**

- `new` — Allocates on the heap.
- Reference — Points to the heap object.
- GC — Reclaims unreachable objects.

#### Syntax Rules

- All `class` instances, arrays, strings, and boxed value types go on the managed heap.
- Object lifetime is determined by reachability, not scope.
- `null` reference means “points to nothing.”

#### Constraints and Limitations

- Heap allocation is slower than stack allocation.
- GC pauses can affect latency.
- Large allocations go to the LOH.

---

### Annotated Code Examples and Expected Outputs

#### Example: Object on the Heap

```csharp
using System;

class Person
{
    public string Name;
}

class Program
{
    static Person Create()
    {
        var person = new Person { Name = "Alice" };
        return person;   // reference escapes; object lives on heap
    }

    static void Main()
    {
        var p = Create();
        Console.WriteLine(p.Name);
    }
}
```

**Expected Output**
```
Alice
```

**Why this output occurs**
The `Person` object is allocated on the managed heap; the reference is returned and remains valid after `Create` returns.

---

### Real-World Cases

- **Domain models** — Entities and value objects.
- **Collections** — Lists, dictionaries, arrays.
- **Long-lived services** — Repositories, caches.

---

### References

- The Truth About Value Types — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-truth-about-value-types

---

## 4. Managed Heap

### Definitions

**Core Definition**
The managed heap is the **GC-controlled portion of the heap** where .NET reference-type objects are allocated and automatically reclaimed.

**Technical Definition**
The CLR's garbage collector manages the managed heap. It uses a **generational** design (Gen 0, Gen 1, Gen 2) plus the LOH. New objects are allocated in Gen 0; surviving objects are promoted to Gen 1 and Gen 2. The GC uses a **mark-and-sweep** (or mark-compact) algorithm: it traces reachable objects from roots (stack, statics, handles) and reclaims unreachable ones. Compaction moves surviving objects to reduce fragmentation (except on the LOH). The GC is **non-deterministic** and can be **concurrent** (background GC) or **blocking**.

**Beginner-Friendly Explanation**
The managed heap is **where .NET objects live**, and the GC is the **cleaning crew**. Objects start in Gen 0 (young); survivors get promoted. The GC finds unreachable objects and reclaims their memory.

---

### Purposes

- To provide **automatic memory management** without manual `free`.
- To reduce memory leaks and dangling pointers.
- To optimize allocation and collection with generations.
- To support **compaction** (defragmentation) for better locality.

---

### Syntax Rules and Structure

```
var list = new List<int>();   // allocated on managed heap
// GC reclaims when unreachable
```

**Component breakdown**

- **Gen 0** — Short-lived objects; collected frequently and cheaply.
- **Gen 1** — Buffer between short- and long-lived.
- **Gen 2** — Long-lived objects; collected during full GCs.
- **LOH** — Large objects; collected only during Gen 2.
- **Mark-and-sweep / compact** — GC algorithm.

#### Syntax Rules

- Objects are allocated in Gen 0.
- Surviving objects are promoted to older generations.
- GC traces roots (stack, statics, handles) to find reachable objects.

#### Constraints and Limitations

- GC pauses can affect latency; use `GCSettings.LatencyMode` or server GC.
- Finalizers delay collection and require promotion.
- LOH fragmentation is a known issue; use `ArrayPool<T>` to avoid repeated LOH allocations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Generations with `GC.GetGeneration`

```csharp
using System;

class Program
{
    static void Main()
    {
        var obj = new object();
        Console.WriteLine(GC.GetGeneration(obj));   // 0

        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();

        Console.WriteLine(GC.GetGeneration(obj));   // 2 (promoted after full GC)
    }
}
```

**Expected Output**
```
0
2
```

**Why this output occurs**
A newly allocated object is in Gen 0. After a full GC, it survives and is promoted to Gen 2.

---

### Real-World Cases

- **Web servers** — Server GC optimizes throughput.
- **Low-latency systems** — Background GC reduces pauses.
- **High-allocation workloads** — Understanding generations helps reduce pressure.

---

### References

- Fundamentals of garbage collection — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

---

## 5. Large Object Heap (LOH)

### Definitions

**Core Definition**
The LOH is a **special heap segment for objects ≥ 85,000 bytes** (approximately 83 KB), collected only during Gen 2 garbage collections.

**Technical Definition**
Objects of 85,000 bytes or more are allocated on the LOH. The LOH is collected only during **Gen 2** collections (full GCs) and is **not compacted by default** (since .NET 4.5, compaction is optional via `GCSettings.LargeObjectHeapCompactionMode`). Because the LOH is not compacted, it can **fragment** over time, leading to out-of-memory errors even when free memory exists. The LOH is used for large arrays (e.g., `byte[]`), large strings, and large object graphs. The threshold is an implementation detail of the current CLR; the spec does not mandate 85 KB.

**Beginner-Friendly Explanation**
The LOH is the **“big stuff” heap**. Objects ≥ 85 KB skip the small-object heap and go here. The LOH is collected less often and can fragment, so it should be used carefully.

---

### Purposes

- To avoid the overhead of copying large objects during compaction.
- To handle large arrays and strings efficiently.
- To allow developers to choose when to compact the LOH.

---

### Syntax Rules and Structure

```
byte[] largeBuffer = new byte[100_000];   // allocated on the LOH (≥ 85 KB)
```

**Component breakdown**

- ≥ 85,000 bytes — LOH threshold.
- Gen 2 collection — LOH is collected only during full GCs.
- Compaction — Optional, via `GCSettings.LargeObjectHeapCompactionMode`.

#### Syntax Rules

- Objects ≥ 85,000 bytes go to the LOH.
- The LOH is collected only during Gen 2.
- The LOH is not compacted by default.

#### Constraints and Limitations

- Fragmentation can cause `OutOfMemoryException`.
- LOH collections are expensive (full GCs).
- The threshold is an implementation detail.

---

### Annotated Code Examples and Expected Outputs

#### Example: Observing the LOH

```csharp
using System;

class Program
{
    static void Main()
    {
        byte[] small = new byte[50_000];
        byte[] large = new byte[100_000];

        Console.WriteLine(GC.GetGeneration(small));   // 0 (small object heap)
        Console.WriteLine(GC.GetGeneration(large));   // 2 (LOH objects report as Gen 2)
    }
}
```

**Expected Output**
```
0
2
```

**Why this output occurs**
`small` (50 KB) is below the LOH threshold; `large` (100 KB) is on the LOH and reports as Gen 2.

---

### Real-World Cases

- **Large buffers** — Use `ArrayPool<byte>.Shared.Rent` to reuse LOH buffers.
- **Large strings** — Avoid repeated large string allocations.
- **Image processing** — Large pixel buffers.

---

### References

- The Large Object Heap uncovered — .NET Blog — https://devblogs.microsoft.com/dotnet/the-large-object-heap-uncovered/
- Large object heap (LOH) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap

---

## 6. Memory Allocation

### Definitions

**Core Definition**
Memory allocation is the process of **reserving memory** for a variable or object — on the stack for locals and call frames, and on the managed heap for objects.

**Technical Definition**
Stack allocation is **pointer-bump** — fast, automatic, LIFO. Managed-heap allocation involves the GC: the allocation pointer advances in Gen 0; when Gen 0 fills, a GC occurs. Allocations ≥ 85,000 bytes go to the LOH. **Boxing** allocates a heap object for a value type. **Closures** allocate a heap object to hold captured variables. **Async state machines** and **iterator state machines** may allocate on the heap (unless they complete synchronously and the compiler can optimize). Allocation cost is low on the managed heap, but the *GC pressure* it creates is the real cost.

**Beginner-Friendly Explanation**
Allocation is **getting memory for your data**. Stack allocation is fast and automatic; heap allocation is also fast but creates garbage that the GC must clean up.

---

### Purposes

- To understand where different kinds of data are stored.
- To reduce unnecessary allocations and GC pressure.
- To choose between stack-friendly (`struct`, `Span<T>`) and heap-based types.
- To diagnose allocation hot spots.

---

### Syntax Rules and Structure (Allocation Sources)

| Source | Where | Notes |
|---|---|---|
| Local value types | Stack | Fast, automatic |
| `new` class/array/string | Managed heap (Gen 0 or LOH) | GC-managed |
| Boxing | Managed heap | `int` → `object` |
| Closures | Managed heap | Captured variables |
| Async/iterator state machines | Managed heap (usually) | May be optimized |
| `ArrayPool<T>` | Managed heap (reused) | Reduces GC pressure |

**Component breakdown**

- **Stack** — Locals, references.
- **Gen 0** — Most allocations.
- **LOH** — Large allocations.
- **Pooling** — Reuse to reduce pressure.

#### Syntax Rules

- `new` on reference types allocates on the managed heap.
- Boxing a value type allocates a heap object.
- Capturing a variable in a lambda allocates a closure.
- `ArrayPool<T>` and `MemoryPool<T>` reduce allocation churn.

#### Constraints and Limitations

- Each heap allocation adds GC pressure.
- LOH allocations are expensive to collect.
- Closures and boxing can be hidden allocations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Boxing Allocation

```csharp
using System;

class Program
{
    static void Main()
    {
        int value = 42;
        object boxed = value;   // boxing: heap allocation

        Console.WriteLine(boxed);   // 42
    }
}
```

**Expected Output**
```
42
```

**Why this output occurs**
Boxing allocates a heap object that wraps the `int` value.

---

### Real-World Cases

- **High-throughput services** — Reducing allocations improves throughput.
- **Game engines** — Minimizing GC pauses.
- **Data processing** — Using `ArrayPool<T>` and `Span<T>`.

---

### References

- Memory allocation — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-allocation

---

## 7. Memory Layout

### Definitions

**Core Definition**
Memory layout describes the **physical arrangement** of program data in memory — object headers, fields, alignment, and the separation of stack and heap.

**Technical Definition**
Each managed object on the heap has an **object header** (sync block index + method table pointer) followed by its fields, aligned according to the platform's alignment rules. Value types have their fields laid out inline; reference types are laid out with their fields and a method table pointer. On 64-bit platforms, references are 8 bytes; on 32-bit, 4 bytes. `StructLayout` attributes (`LayoutKind.Sequential`, `Explicit`, `Auto`) control unmanaged layout for interop. **False sharing** occurs when threads on different cores write to different variables on the same cache line, degrading performance.

**Beginner-Friendly Explanation**
Memory layout is **how bytes are arranged** for an object: a header, then fields, possibly with padding for alignment. Understanding layout helps with interop and performance tuning.

---

### Purposes

- To understand object size and overhead.
- To design structs for efficient interop and cache locality.
- To avoid false sharing in concurrent code.
- To diagnose alignment and padding issues.

---

### Syntax Rules and Structure

```
┌──────────────────────────────┐
│  Object Header (sync + MT)    │
├──────────────────────────────┤
│  Field 1                      │
│  Field 2                      │
│  ...                          │
└──────────────────────────────┘
```

**Component breakdown**

- **Object header** — Sync block + method table pointer.
- **Fields** — Inline data.
- **Padding** — For alignment.

#### Syntax Rules

- Reference types have an object header.
- Value types have fields inline.
- `[StructLayout]` controls interop layout.

#### Constraints and Limitations

- Layout is **implementation-dependent**; do not rely on it for correctness.
- False sharing can degrade multi-threaded performance.

---

### Annotated Code Examples and Expected Outputs

#### Example: Struct Layout with `StructLayout`

```csharp
using System;
using System.Runtime.InteropServices;

[StructLayout(LayoutKind.Sequential)]
struct Point
{
    public int X;
    public int Y;
}

class Program
{
    static void Main()
    {
        Console.WriteLine(Marshal.SizeOf<Point>());   // 8 (two ints)
    }
}
```

**Expected Output**
```
8
```

**Why this output occurs**
`Point` has two `int` fields, each 4 bytes, laid out sequentially = 8 bytes.

---

### Real-World Cases

- **Interop** — P/Invoke struct layouts must match native layout.
- **Performance** — Struct layout affects cache locality.
- **Concurrency** — Avoiding false sharing.

---

### References

- StructLayoutAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.structlayoutattribute

---

## 8. Memory Model Best Practices

### General Guidelines

```
// 1. Prefer stack-friendly types for hot paths
Span<byte> buffer = stackalloc byte[256];

// 2. Avoid unnecessary heap allocations
// Use ArrayPool<T> for large buffers
var rented = ArrayPool<byte>.Shared.Rent(100_000);
try { /* use rented */ }
finally { ArrayPool<byte>.Shared.Return(rented); }

// 3. Avoid boxing value types
// BAD: object o = 42;
// GOOD: use generics or Span<T>

// 4. Minimize LOH allocations
// Use ArrayPool<T> for large buffers; avoid frequent large allocations.

// 5. Use synchronization for cross-thread visibility
lock (_lock) { sharedCounter++; }
Volatile.Write(ref _flag, true);
Interlocked.Increment(ref _counter);

// 6. Do not rely on physical stack vs heap location
// The spec only guarantees lifetimes.

// 7. Use ConfigureAwait(false) in libraries to avoid context overhead
await SomeAsync().ConfigureAwait(false);

// 8. Avoid finalizers where possible; use IDisposable and SafeHandle
```

### Purposes

- To reduce allocations and GC pressure.
- To avoid memory-related bugs (race conditions, leaks).
- To improve latency and throughput.
- To write correct concurrent code.

### Rules and Constraints

- **Do** prefer `Span<T>`, `stackalloc`, and `ArrayPool<T>` for hot paths.
- **Do** use synchronization (`lock`, `Interlocked`, `Volatile`) for shared mutable state.
- **Do** dispose unmanaged resources with `IDisposable` / `SafeHandle`.
- **Do not** rely on physical stack vs. heap locations.
- **Do not** box unnecessarily.
- **Do not** allocate large buffers repeatedly (LOH fragmentation).

### Real-World Cases

- **High-throughput services** — Allocation reduction and LOH avoidance.
- **Concurrent data structures** — Proper synchronization and memory barriers.
- **Low-latency systems** — Server GC, background GC, and pooling.

### References

- Memory model — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-model/
- Fundamentals of garbage collection — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
- The Large Object Heap uncovered — .NET Blog — https://devblogs.microsoft.com/dotnet/the-large-object-heap-uncovered/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Stack allocation of value types | **Implementation-dependent**; spec guarantees lifetimes only. |
| LOH threshold (85,000 bytes) | **Implementation detail**; not specified. |
| LOH compaction | **Off by default**; can be enabled via `GCSettings.LargeObjectHeapCompactionMode`. |
| `GC.Collect` | **Discouraged** in production code (except specific scenarios). |
| `Finalize` / destructors | **Non-deterministic**; use `IDisposable`/`SafeHandle` instead. |
| `stackalloc` | **Unsafe** if not bounded; can cause stack overflow. |
| `Marshal.AllocHGlobal` | **Unmanaged**; must be freed manually. |
| False sharing | **Performance pitfall**; layout-dependent. |

---

## References (Consolidated)

- Memory model — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-model/
- The C# Memory Model in Theory and Practice — MSDN Magazine — https://learn.microsoft.com/en-us/archive/msdn-magazine/2012/december/csharp-the-csharp-memory-model-in-theory-and-practice
- Fundamentals of garbage collection — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals
- The Large Object Heap uncovered — .NET Blog — https://devblogs.microsoft.com/dotnet/the-large-object-heap-uncovered/
- Large object heap (LOH) — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap
- Memory allocation — .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-allocation
- The Stack Is An Implementation Detail, Part One — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-one
- The Truth About Value Types — Microsoft Learn (archive) — https://learn.microsoft.com/en-us/archive/blogs/ericlippert/the-truth-about-value-types
- StructLayoutAttribute — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.structlayoutattribute

---