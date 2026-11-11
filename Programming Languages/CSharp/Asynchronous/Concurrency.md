# C# Concurrency — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Concurrency is the ability of a program to have multiple tasks **in progress at the same time**, interleaving their execution to make progress on several fronts without necessarily running them simultaneously on separate CPU cores. Parallelism, a subset of concurrency, is the **simultaneous execution** of multiple tasks on multiple cores. .NET provides high-level abstractions—the `Parallel` class for data and task parallelism, and the `System.Collections.Concurrent` namespace for thread-safe collections—that reduce the complexity and risk of manual multithreaded programming.

**Technical Definition**
Concurrency encompasses all techniques for dealing with multiple activities at once: asynchronous I/O, multithreading, and parallel processing. The Task Parallel Library (TPL) and its `Parallel` class provide **library-based data-parallel replacements** for common operations such as `for` loops, `foreach` loops, and the execution of a set of statements. Parallel methods block the calling thread until all work completes, use the calling thread as part of the work, and throw an `AggregateException` if any work item throws. The `System.Collections.Concurrent` namespace provides **thread-safe and scalable** collection classes that multiple threads can safely add to or remove from without additional user-code synchronization. These collections use fine-grained locking and lock-free mechanisms (`SpinLock`, `SpinWait`, `SemaphoreSlim`, `CountdownEvent`) to achieve high throughput.

**Beginner-Friendly Explanation**
Concurrency is **doing many things at once**—or at least making it look that way. Parallelism is **literally doing many things at the same instant on different CPU cores**. Imagine a chef: concurrency is one chef juggling multiple pots (switching between them), while parallelism is multiple chefs each working on their own pot simultaneously. .NET gives you `Parallel.For` to turn a sequential loop into a parallel one, and concurrent collections (`ConcurrentDictionary`, `ConcurrentQueue`, etc.) that multiple threads can safely use without corrupting data.

---

### Key Characteristics

- **Concurrency ≠ parallelism** — Concurrency is about dealing with many things at once; parallelism is about doing many things at once.
- **`Parallel` blocks until done** — `Parallel.For`, `Parallel.ForEach`, and `Parallel.Invoke` block the calling thread until all work completes.
- **The calling thread participates** — Parallel methods use the calling thread as part of the work, then wait for the rest.
- **`AggregateException` on failure** — If any work item throws, the parallel method throws an `AggregateException` containing all exceptions.
- **Parallelism has overhead** — Delegate allocation and per-item invocation cost mean parallel loops only pay off for sufficiently large or expensive work items.
- **Work items must be independent** — Parallel methods are only safe when work items do not modify shared data; otherwise synchronization is required, which can negate the benefits.
- **Concurrent collections are scalable** — They use fine-grained locking and lock-free techniques, outperforming externally locked collections in many scenarios.
- **Not all scenarios benefit** — For small or fast work items, thread-safe collections may perform the same or even slower than an externally locked non-thread-safe collection.

---

### Prerequisites

- Solid understanding of **C# basics** (methods, loops, collections, generics).
- Familiarity with **delegates** and **lambda expressions**.
- Awareness of **threads** and the **thread pool**.
- Understanding of **race conditions** and **deadlocks** (helpful but not required).
- Knowledge of `IEnumerable<T>` and basic collection types.

---

### Related Programming Areas

- **Task Parallel Library (TPL)** — The infrastructure underlying `Parallel` and tasks.
- **PLINQ** — Parallel LINQ, built on the same partitioning infrastructure.
- **Asynchronous programming** — `async`/`await` for non-blocking I/O; complementary to parallelism.
- **Thread-safe collections** — `System.Collections.Concurrent` for shared state.
- **Synchronization primitives** — `lock`, `Monitor`, `SemaphoreSlim`, `Interlocked` for coordination.

---

### Core Concepts / Features

1. Concurrency
2. Parallelism
3. `Parallel.For()`
4. `Parallel.ForEach()`
5. `Parallel.Invoke()`
6. Concurrent Collections Overview
7. `ConcurrentDictionary<TKey, TValue>`
8. `ConcurrentBag<T>`
9. `ConcurrentQueue<T>`
10. `ConcurrentStack<T>`
11. `BlockingCollection<T>`
12. Concurrency Best Practices

---

## 1. Concurrency

### Definitions

- **Core:** Concurrency is the ability to have multiple tasks in progress at the same time, interleaving their execution.
- **Technical:** Concurrency is a broader concept than parallelism; it encompasses asynchronous I/O, multithreading, and parallel processing. It is about **dealing with** many things at once, not necessarily **doing** them at once.
- **Beginner:** Concurrency is **juggling** multiple tasks—progress happens on all of them, but not necessarily simultaneously.

### Purposes

- To improve **responsiveness** (keeping a UI thread free while background work runs).
- To improve **throughput** (overlapping I/O with computation).
- To model systems that naturally involve multiple activities.

### Syntax Rules and Structure

Concurrency is a concept, not a specific syntax. It is achieved through:
- **Threads** (`new Thread(...)`).
- **Thread pool** (`ThreadPool.QueueUserWorkItem`).
- **Tasks** (`Task.Run`, `Task.Factory.StartNew`).
- **Async/await** (`async`/`await`).
- **Parallel class** (`Parallel.For`, `Parallel.ForEach`, `Parallel.Invoke`).
- **Concurrent collections** (`System.Collections.Concurrent`).

### Constraints and Limitations

- Concurrency introduces **race conditions** and **deadlocks**.
- Shared mutable state requires synchronization, which adds overhead and complexity.

### Annotated Example

```csharp
// Concurrency without parallelism: interleaving on a single core
var task1 = Task.Run(() => Console.WriteLine("Task 1"));
var task2 = Task.Run(() => Console.WriteLine("Task 2"));
await Task.WhenAll(task1, task2);
```

**Expected Output (order may vary)**
```
Task 1
Task 2
```

**Why:** The tasks may run on different threads or interleave on the same thread, but both make progress concurrently.

### Real-World Cases

- **Web servers** — Handling many requests concurrently via async I/O.
- **UI applications** — Keeping the UI thread responsive while background work runs.

### References

- Managed threading best practices — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices

---

## 2. Parallelism

### Definitions

- **Core:** Parallelism is the **simultaneous execution** of multiple tasks on multiple CPU cores.
- **Technical:** Parallelism is a form of concurrency where work is actually executed at the same instant on separate cores. The `Parallel` class provides data parallelism (`For`, `ForEach`) and task parallelism (`Invoke`).
- **Beginner:** Parallelism is **multiple workers doing different things at the exact same time**.

### Purposes

- To reduce the wall-clock time of **CPU-bound** workloads.
- To exploit multi-core hardware.
- To process large data sets faster.

### Syntax Rules and Structure

```csharp
// Data parallelism
Parallel.For(0, N, i => DoWork(i));
Parallel.ForEach(collection, item => DoWork(item));

// Task parallelism
Parallel.Invoke(() => Method1(), () => Method2());
```

### Constraints and Limitations

- Only beneficial for **CPU-bound** work; I/O-bound work should use `async`/`await`.
- Work items must be **independent**; shared state requires synchronization.
- Overhead can outweigh benefits for small or fast work items.
- The calling thread **blocks** until all work completes.

### Annotated Example

```csharp
using System;
using System.Diagnostics;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var sw = Stopwatch.StartNew();
        Parallel.For(0, 100, i => { /* CPU-bound work */ });
        sw.Stop();
        Console.WriteLine($"Parallel.For: {sw.ElapsedMilliseconds} ms");
    }
}
```

**Expected Output (approximate)**
```
Parallel.For: 50 ms
```

**Why:** The work is distributed across thread-pool threads, reducing wall-clock time for CPU-bound operations.

### Real-World Cases

- **Image processing** — Applying filters to each pixel in parallel.
- **Monte Carlo simulations** — Running independent trials in parallel.
- **Batch data transformations** — Processing large arrays or lists in parallel.

### References

- Parallel Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel

---

## 3. `Parallel.For()`

### Definitions

- **Core:** `Parallel.For` executes a `for` loop whose iterations may run in parallel.
- **Technical:** `Parallel.For(int fromInclusive, int toExclusive, Action<int> body)` provides a data-parallel replacement for a sequential `for` loop. It uses thread-pool threads and the calling thread, blocks until all iterations complete, and throws `AggregateException` if any iteration throws.
- **Beginner:** `Parallel.For` is a **parallel `for` loop**.

### Purposes

- To parallelize index-based loops over a range.
- To speed up CPU-bound iterations.
- To reduce the wall-clock time of embarrassingly parallel work.

### Syntax Rules and Structure

```csharp
Parallel.For(fromInclusive, toExclusive, i =>
{
    // work for iteration i
});
```

- `fromInclusive` — Start index (inclusive).
- `toExclusive` — End index (exclusive).
- `body` — `Action<int>` executed for each index.

### Constraints and Limitations

- Iteration order is **not guaranteed**.
- Shared state must be synchronized or eliminated.
- Overhead may not be worth it for small ranges or fast bodies.
- Cannot `break` out of a parallel loop using `break`; use `ParallelLoopState` or `ParallelOptions.CancellationToken`.

### Annotated Example

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        Parallel.For(0, 10, i =>
        {
            Console.WriteLine($"Iteration {i} on thread {Environment.CurrentManagedThreadId}");
        });
    }
}
```

**Expected Output (order and thread IDs vary)**
```
Iteration 0 on thread 1
Iteration 3 on thread 4
Iteration 1 on thread 5
...
```

**Why:** Iterations are distributed across thread-pool threads and may execute in any order.

### Real-World Cases

- **Matrix operations** — Parallelizing row or column computations.
- **Parameter sweeps** — Running a simulation for many parameter values.

### References

- Parallel.For Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.for

---

## 4. `Parallel.ForEach()`

### Definitions

- **Core:** `Parallel.ForEach` executes a `foreach` loop whose iterations may run in parallel.
- **Technical:** `Parallel.ForEach<TSource>(IEnumerable<TSource> source, Action<TSource> body)` provides a data-parallel replacement for a sequential `foreach` loop over any `IEnumerable<T>`. Like `Parallel.For`, it blocks until all work completes and throws `AggregateException` on failure.
- **Beginner:** `Parallel.ForEach` is a **parallel `foreach` loop**.

### Purposes

- To parallelize iteration over a collection.
- To process each element of a large collection concurrently.

### Syntax Rules and Structure

```csharp
Parallel.ForEach(collection, item =>
{
    // work for item
});
```

- `source` — Any `IEnumerable<TSource>`.
- `body` — `Action<TSource>` executed for each element.

### Constraints and Limitations

- Iteration order is **not guaranteed**.
- Shared state must be synchronized.
- The source collection must not be modified during iteration.
- Overhead may not be worth it for small collections or fast bodies.

### Annotated Example

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        Parallel.ForEach(numbers, n =>
        {
            Console.WriteLine($"Processing {n}");
        });
    }
}
```

**Expected Output (order varies)**
```
Processing 1
Processing 3
Processing 2
Processing 4
Processing 5
```

**Why:** Elements are distributed across threads and processed in parallel.

### Real-World Cases

- **Batch processing** — Processing each item in a list (e.g., resizing images).
- **Data transformation** — Applying a function to every element of a collection.

### References

- Parallel.ForEach Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.foreach

---

## 5. `Parallel.Invoke()`

### Definitions

- **Core:** `Parallel.Invoke` executes a set of actions, possibly in parallel.
- **Technical:** `Parallel.Invoke(params Action[] actions)` provides **task parallelism**—it runs each action concurrently and blocks until all complete. It is a convenient way to run arbitrary statements concurrently.
- **Beginner:** `Parallel.Invoke` is **“run these methods in parallel.”**

### Purposes

- To execute multiple independent methods or statements concurrently.
- To parallelize task-parallel workloads (as opposed to data-parallel loops).

### Syntax Rules and Structure

```csharp
Parallel.Invoke(
    () => Method1(),
    () => Method2(),
    () => Method3());
```

- `actions` — An array of `Action` delegates.

### Constraints and Limitations

- The actions must be independent; shared state requires synchronization.
- Blocks until all actions complete.
- Throws `AggregateException` if any action throws.

### Annotated Example

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        Parallel.Invoke(
            () => Console.WriteLine("Task A"),
            () => Console.WriteLine("Task B"),
            () => Console.WriteLine("Task C"));
    }
}
```

**Expected Output (order varies)**
```
Task B
Task A
Task C
```

**Why:** Each action runs on a thread-pool thread; completion order is not guaranteed.

### Real-World Cases

- **Independent initializations** — Loading multiple resources concurrently.
- **Concurrent service calls** — Calling multiple independent services in parallel.

### References

- Parallel.Invoke Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.invoke

---

## 6. Concurrent Collections Overview

### Definitions

- **Core:** The `System.Collections.Concurrent` namespace provides **thread-safe collection classes** designed for concurrent access.
- **Technical:** These classes use fine-grained locking and lock-free mechanisms (`SpinLock`, `SpinWait`, `SemaphoreSlim`, `CountdownEvent`) to achieve thread safety and scalability. They are preferred over `System.Collections.Generic` collections with external locks when multiple threads add or remove items.
- **Beginner:** Concurrent collections are **collections that many threads can safely use at the same time**.

### The Concurrent Collections

| Collection | Description |
|---|---|
| `ConcurrentDictionary<TKey,TValue>` | Thread-safe key/value dictionary |
| `ConcurrentQueue<T>` | Thread-safe FIFO queue |
| `ConcurrentStack<T>` | Thread-safe LIFO stack |
| `ConcurrentBag<T>` | Thread-safe unordered bag |
| `BlockingCollection<T>` | Blocking/bounding wrapper for producer-consumer |

### When to Use Thread-Safe vs. Locked Non-Thread-Safe Collections

| Scenario | Recommendation |
|---|---|
| **Pure producer-consumer, very fast items** | `ConcurrentQueue<T>` can be modestly faster than `Queue<T>` + lock |
| **Processing time ≥ ~500 FLOPS** | `ConcurrentQueue<T>` scales very well; `Queue<T>` does not |
| **Mixed producer-consumer, fast items** | Externally locked `Queue<T>` scales better than `ConcurrentQueue<T>` |
| **Reading only** | Use `System.Collections.Generic` collections |

---

## 7. `ConcurrentDictionary<TKey, TValue>`

### Definitions

- **Core:** `ConcurrentDictionary<TKey,TValue>` is a thread-safe key/value dictionary.
- **Technical:** Multiple threads can safely add, remove, and update key/value pairs concurrently. It provides atomic operations like `GetOrAdd`, `AddOrUpdate`, and `TryUpdate`.
- **Beginner:** A **thread-safe dictionary**.

### Key Operations

```csharp
var dict = new ConcurrentDictionary<string, int>();

// Add or update atomically
int value = dict.GetOrAdd("key", k => ComputeValue(k));
dict.AddOrUpdate("key", 1, (k, old) => old + 1);

// Try-pattern operations
if (dict.TryAdd("key", 42)) { ... }
if (dict.TryUpdate("key", newValue, comparisonValue)) { ... }
if (dict.TryRemove("key", out int removed)) { ... }
```

### Constraints

- `GetOrAdd` may invoke the factory multiple times under contention.
- Iteration is not a snapshot; the dictionary may change during enumeration.

### Annotated Example

```csharp
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var counts = new ConcurrentDictionary<string, int>();
        Parallel.ForEach(new[] { "a", "b", "a", "c", "a" }, item =>
        {
            counts.AddOrUpdate(item, 1, (key, old) => old + 1);
        });

        foreach (var kvp in counts)
            Console.WriteLine($"{kvp.Key}: {kvp.Value}");
    }
}
```

**Expected Output**
```
a: 3
b: 1
c: 1
```

**Why:** `AddOrUpdate` atomically increments the count for each key.

### References

- ConcurrentDictionary<TKey,TValue> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2

---

## 8. `ConcurrentBag<T>`

### Definitions

- **Core:** `ConcurrentBag<T>` is a thread-safe, unordered collection that allows duplicates.
- **Technical:** Optimized for scenarios where the **same thread** both adds and removes items. It uses thread-local storage internally for fast adds and takes.
- **Beginner:** A **thread-safe bag** optimized for same-thread producer-consumer.

### Key Operations

```csharp
var bag = new ConcurrentBag<int>();
bag.Add(1);
bag.Add(2);

if (bag.TryTake(out int item)) { ... }
if (bag.TryPeek(out int peeked)) { ... }
```

### Constraints

- Unordered; no guarantee of FIFO or LIFO.
- Less efficient when producer and consumer are different threads.

### Annotated Example

```csharp
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var bag = new ConcurrentBag<int>();
        Parallel.For(0, 10, i => bag.Add(i));

        Console.WriteLine($"Bag count: {bag.Count}");
    }
}
```

**Expected Output**
```
Bag count: 10
```

**Why:** Each parallel iteration adds an item to the bag; `Add` is thread-safe.

### References

- ConcurrentBag<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1

---

## 9. `ConcurrentQueue<T>`

### Definitions

- **Core:** `ConcurrentQueue<T>` is a thread-safe FIFO queue.
- **Technical:** Represents a thread-safe first-in-first-out collection. It uses lock-free `Interlocked` operations for high performance.
- **Beginner:** A **thread-safe FIFO queue**.

### Key Operations

```csharp
var queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
if (queue.TryDequeue(out int item)) { ... }
if (queue.TryPeek(out int peeked)) { ... }
```

### Constraints

- No blocking; use `BlockingCollection<T>` for blocking behavior.
- No capacity limit.

### Annotated Example

```csharp
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        var queue = new ConcurrentQueue<int>();
        Parallel.For(0, 5, i => queue.Enqueue(i));

        while (queue.TryDequeue(out int item))
            Console.WriteLine(item);
    }
}
```

**Expected Output (order may vary)**
```
0
1
2
3
4
```

**Why:** `Enqueue` is thread-safe; `TryDequeue` retrieves items in FIFO order.

### References

- ConcurrentQueue<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1

---

## 10. `ConcurrentStack<T>`

### Definitions

- **Core:** `ConcurrentStack<T>` is a thread-safe LIFO stack.
- **Technical:** Represents a thread-safe last-in-first-out collection. Uses lock-free `Interlocked` operations for high performance.
- **Beginner:** A **thread-safe LIFO stack**.

### Key Operations

```csharp
var stack = new ConcurrentStack<int>();
stack.Push(1);
if (stack.TryPop(out int item)) { ... }
if (stack.TryPeek(out int peeked)) { ... }
```

### Constraints

- No blocking; use `BlockingCollection<T>` for blocking behavior.
- No capacity limit.

### Annotated Example

```csharp
using System;
using System.Collections.Concurrent;

class Program
{
    static void Main()
    {
        var stack = new ConcurrentStack<int>();
        stack.Push(1);
        stack.Push(2);
        stack.Push(3);

        while (stack.TryPop(out int item))
            Console.WriteLine(item);
    }
}
```

**Expected Output**
```
3
2
1
```

**Why:** `TryPop` retrieves items in LIFO order.

### References

- ConcurrentStack<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentstack-1

---

## 11. `BlockingCollection<T>`

### Definitions

- **Core:** `BlockingCollection<T>` provides blocking and bounding capabilities for thread-safe producer-consumer collections.
- **Technical:** Wraps any `IProducerConsumerCollection<T>` and adds **blocking** (waiting when empty or full) and **bounded capacity** (limiting the collection size). It supports `Add` (blocks when full), `Take` (blocks when empty), `TryAdd`, `TryTake`, and `CompleteAdding`.
- **Beginner:** A **thread-safe collection that can block** producers and consumers.

### Key Operations

```csharp
var collection = new BlockingCollection<int>(boundedCapacity: 100);

// Producer
collection.Add(item);           // blocks if full

// Consumer
int item = collection.Take();   // blocks if empty

// Signal completion
collection.CompleteAdding();

// Consume until complete
foreach (var item in collection.GetConsumingEnumerable()) { ... }
```

### Constraints

- `Add` blocks when the collection is at capacity.
- `Take` blocks when the collection is empty.
- Must call `CompleteAdding` to signal that no more items will be added.

### Annotated Example

```csharp
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        using var collection = new BlockingCollection<int>(boundedCapacity: 5);

        var producer = Task.Run(() =>
        {
            for (int i = 0; i < 10; i++)
            {
                collection.Add(i);
                Console.WriteLine($"Produced {i}");
            }
            collection.CompleteAdding();
        });

        var consumer = Task.Run(() =>
        {
            foreach (var item in collection.GetConsumingEnumerable())
                Console.WriteLine($"Consumed {item}");
        });

        Task.WaitAll(producer, consumer);
    }
}
```

**Expected Output (interleaved)**
```
Produced 0
Consumed 0
Produced 1
Consumed 1
...
```

**Why:** `Add` blocks when the bounded capacity is reached; `GetConsumingEnumerable` blocks until items are available or `CompleteAdding` is called.

### References

- BlockingCollection<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.blockingcollection-1

---

## 12. Concurrency Best Practices

### General Guidelines

```
// 1. Use Parallel only for CPU-bound, independent work
Parallel.For(0, 1000, i => ComputeIntensive(i));   // CPU-bound, independent

// 2. Avoid shared mutable state in parallel work items
// BAD: total += i;   // race condition
// GOOD: use Interlocked or local aggregation

// 3. Measure before parallelizing
// Parallel has overhead; small work items may be slower in parallel.

// 4. Use concurrent collections for shared state
var dict = new ConcurrentDictionary<string, int>();   // not Dictionary + lock

// 5. Prefer BlockingCollection for producer-consumer
var queue = new BlockingCollection<WorkItem>(100);

// 6. Do not use Thread.Abort
// Use cooperative cancellation (CancellationToken) instead.

// 7. Do not lock on types or 'this'
// Use private readonly object _lock = new object();

// 8. Acquire multiple locks in a consistent order to avoid deadlocks
```

### Purposes

- To avoid **race conditions** and **deadlocks**.
- To ensure **scalability** and **correctness**.
- To choose the right concurrency primitive for the task.

### Rules and Constraints

- **Do** use `Parallel` only when work items are independent and CPU-bound.
- **Do** use concurrent collections for shared state.
- **Do** use `BlockingCollection<T>` for producer-consumer patterns.
- **Do** use cooperative cancellation (`CancellationToken`), not `Thread.Abort`.
- **Do not** lock on `this`, types, or string instances.
- **Do not** assume parallel is always faster; measure first.
- **Do not** modify shared state without synchronization in parallel loops.

### Real-World Cases

- **Web servers** — Using concurrent collections for shared caches.
- **Data pipelines** — Using `BlockingCollection<T>` for producer-consumer queues.
- **Batch processing** — Using `Parallel.ForEach` for CPU-bound transformations.

### References

- Managed threading best practices — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices
- Thread-safe collections — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/collections/thread-safe/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Parallel.For` / `ForEach` / `Invoke` | Introduced in **.NET Framework 4.0**; stable. |
| `System.Collections.Concurrent` | Introduced in **.NET Framework 4.0**; stable. |
| `Thread.Abort` | **Unsafe and deprecated**; use cooperative cancellation. |
| Locking on `this` or types | **Dangerous**; can cause deadlocks. |
| `Parallel` with async delegates | **Problematic**; parallel methods only run each async delegate until the first `await`. |
| `ConcurrentBag<T>` same-thread optimization | **Implementation-dependent**; optimized for same-thread producer-consumer. |
| Thread-safe collection performance | **Workload-dependent**; may not always be faster than externally locked non-thread-safe collections. |

---

## References (Consolidated)

- Parallel Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel
- Parallel.For Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.for
- Parallel.ForEach Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.foreach
- Parallel.Invoke Method — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel.invoke
- Thread-safe collections — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/collections/thread-safe/
- System.Collections.Concurrent Namespace — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent
- ConcurrentDictionary<TKey,TValue> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2
- ConcurrentBag<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1
- ConcurrentQueue<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1
- ConcurrentStack<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentstack-1
- BlockingCollection<T> Class — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.blockingcollection-1
- Managed threading best practices — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices
- When to use a thread-safe collection — Microsoft Learn (raw) — https://raw.githubusercontent.com/mialkin/docs/e12dbe0be5542e70e8aecfc3319fdf6677c8a92b/csharp/concurrency/collections/when-to-use-thread-safe.md
- CLR via C# (Chapter 27: Compute-Bound Async) — GitHub — https://github.com/wangjunicode/CLR_via_CSharp/blob/main/chapters/ch27_ComputeBoundAsync.md
- 多執行緒的實務最佳實踐 .NET篇 ── 在增加執行緒之前先決定的事 — Zenodo — https://zenodo.org/records/17722496

---