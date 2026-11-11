# C# PLINQ (Parallel LINQ) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
PLINQ (Parallel LINQ) is a **parallel implementation of LINQ to Objects** that automatically partitions an in-memory data source across multiple processors and executes query operations concurrently, aiming to reduce total execution time for **CPU-bound** workloads.

**Technical Definition**
PLINQ is a parallel implementation of the Language-Integrated Query (LINQ) pattern. It implements the full set of LINQ standard query operators as extension methods for the `System.Linq` namespace and has additional operators for parallel operations. PLINQ queries operate on any in-memory `IEnumerable` or `IEnumerable<T>` data source and, like sequential LINQ, use **deferred execution**—they do not begin executing until the query is enumerated. The primary difference is that PLINQ attempts to fully utilize all processors on the system by partitioning the data source into segments and executing the query on each segment in separate worker threads in parallel across multiple processors. Opting in is done by calling `AsParallel()` on the source sequence. PLINQ combines the simplicity and readability of LINQ syntax with the power of parallel programming.

**Beginner-Friendly Explanation**
PLINQ is **LINQ that uses all your CPU cores**. You write the same `Where`, `Select`, `OrderBy` queries you already know, but you start the chain with `.AsParallel()`. PLINQ then splits the data into chunks, processes each chunk on a separate thread, and merges the results. It is designed for **CPU-heavy work**—for example, running an expensive computation over a large array. It is **not** a magic speed-up button: for small datasets, fast delegates, or I/O-bound operations, PLINQ can actually be slower due to partitioning and synchronization overhead. The rule is simple: **measure first, parallelize only when the workload justifies it**.

---

### Key Characteristics

- **Opt-in parallelism** — PLINQ is activated explicitly with `AsParallel()`; sequential LINQ remains the default.
- **Deferred execution** — PLINQ queries execute only when enumerated, just like sequential LINQ.
- **Automatic partitioning** — PLINQ splits the source data into segments and processes them on separate threads.
- **Order preservation is OFF by default** — PLINQ does not preserve source order unless you explicitly request it with `AsOrdered()`.
- **Thread-pool based** — PLINQ uses the .NET thread pool; the degree of parallelism is controlled by the runtime or by `WithDegreeOfParallelism()`.
- **CPU-bound focus** — PLINQ is designed for compute-bound work; it is not recommended for I/O-bound operations.
- **Thread-safety required** — Any shared state accessed inside a PLINQ query must be protected or eliminated.

---

### Prerequisites

- Solid understanding of **LINQ to Objects** (method syntax, deferred execution, standard query operators).
- Familiarity with **lambda expressions** and **delegates**.
- Basic understanding of **thread safety**, race conditions, and the **.NET thread pool**.
- Awareness of **CPU-bound vs. I/O-bound** workloads.
- Familiarity with the `System.Linq` namespace and `using System.Linq;`.

---

### Related Programming Areas

- **LINQ to Objects** — PLINQ is the parallel counterpart to sequential in-memory LINQ.
- **Task Parallel Library (TPL)** — PLINQ is built on TPL infrastructure (`ParallelEnumerable`, thread pool).
- **Concurrent programming** — Shared state, synchronization, and thread safety are central to correct PLINQ usage.
- **Performance profiling** — PLINQ should only be used after measuring sequential performance and identifying CPU-bound bottlenecks.
- **Data parallelism** — PLINQ is a form of data parallelism (as opposed to task parallelism).

---

### Core Concepts / Features

1. PLINQ (Parallel LINQ) — Overarching Concept
2. `AsParallel()`
3. `AsSequential()`
4. `WithDegreeOfParallelism()`
5. `AsOrdered()`
6. `AsUnordered()`
7. PLINQ Best Practices
8. PLINQ Pitfalls

---

## 1. PLINQ (Parallel LINQ) — Overarching Concept

### Definitions

**Core Definition**
PLINQ is a parallel implementation of the LINQ pattern for in-memory data sources, designed to exploit multiple processors for CPU-bound queries.

**Technical Definition**
Parallel LINQ (PLINQ) is a parallel implementation of the Language-Integrated Query (LINQ) pattern. PLINQ implements the full set of LINQ standard query operators as extension methods for the `System.Linq` namespace and has additional operators for parallel operations. PLINQ queries operate on any in-memory `IEnumerable` or `IEnumerable<T>` data source and defer execution until enumeration. The primary difference from sequential LINQ is that PLINQ attempts to fully utilize all processors on the system by partitioning the source data into segments and executing the query on each segment in separate worker threads. The `ParallelEnumerable` class provides the PLINQ-specific operators.

**Beginner-Friendly Explanation**
PLINQ is **“LINQ with multiple workers.”** You give it a big list and an expensive operation, and it splits the work across CPU cores.

---

### Purposes

- To reduce execution time for **CPU-bound queries** on multi-core machines.
- To parallelize expensive per-element computations without manually managing threads.
- To leverage the simplicity of LINQ syntax while gaining parallel execution.
- To process large in-memory datasets faster when the per-element work dominates.

---

### Syntax Rules and Structure

```
// Opt in to PLINQ
var query = source.AsParallel()
    .Where(item => ExpensivePredicate(item))
    .Select(item => ExpensiveTransform(item));

// Execution (deferred until enumeration)
var results = query.ToList();
```

**Component breakdown**

- `source` — Any in-memory `IEnumerable` or `IEnumerable<T>`.
- `.AsParallel()` — Converts the sequence to a `ParallelQuery<T>`.
- Standard query operators — Compose the query; they execute in parallel.
- Enumeration — `ToList()`, `foreach`, aggregate operators trigger execution.

#### Syntax Rules

- PLINQ queries operate on `IEnumerable` or `IEnumerable<T>` and use deferred execution.
- `AsParallel()` is the entry point; it returns `ParallelQuery<T>`.
- PLINQ implements the full set of LINQ standard query operators.
- Additional PLINQ operators (`AsOrdered`, `AsUnordered`, `WithDegreeOfParallelism`, `WithMergeOptions`, `ForAll`) are available.

#### Constraints and Limitations

- PLINQ is **not** recommended for I/O-bound operations; it is designed for CPU-bound work.
- Not all query operators benefit from parallelism; some (e.g., `Take` on an unordered query) may not parallelize.
- Shared state must be eliminated or synchronized; otherwise race conditions occur.
- Order preservation is off by default and costs performance when enabled.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic PLINQ Query

```csharp
using System;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] data = Enumerable.Range(0, 1_000_000).ToArray();

        // Sequential
        var sw = Stopwatch.StartNew();
        var seqSum = data.Where(n => n % 2 == 0).Select(n => n * n).Sum();
        sw.Stop();
        Console.WriteLine($"Sequential: {sw.ElapsedMilliseconds} ms, Sum={seqSum}");

        // PLINQ
        sw.Restart();
        var parSum = data.AsParallel()
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .Sum();
        sw.Stop();
        Console.WriteLine($"PLINQ: {sw.ElapsedMilliseconds} ms, Sum={parSum}");
    }
}
```

**Expected Output (approximate)**
```
Sequential: 120 ms, Sum=...
PLINQ: 35 ms, Sum=...
```

**Why this output occurs**
The CPU-bound computation benefits from parallelism; PLINQ partitions the array and processes segments on multiple threads.

---

#### Example 2: PLINQ with Deferred Execution

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        var query = numbers.AsParallel().Where(n => n > 2);   // NOT executed

        Console.WriteLine("Query defined — nothing evaluated yet.");

        foreach (var n in query)   // executed HERE
            Console.Write(n + " ");
    }
}
```

**Expected Output**
```
Query defined — nothing evaluated yet.
3 4 5
```

**Why this output occurs**
PLINQ uses deferred execution, just like sequential LINQ; the query runs only when enumerated.

---

### Real-World Cases

- **Image processing** — Applying a filter to every pixel in parallel.
- **Financial modeling** — Monte Carlo simulations over large datasets.
- **Data analysis** — Parallel aggregation over large in-memory collections.
- **Scientific computing** — CPU-intensive numerical computations.

---

### References

- Introduction to PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq
- ParallelEnumerable Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable

---

## 2. `AsParallel()`

### Definitions

**Core Definition**
`AsParallel()` is the **opt-in operator** that converts an `IEnumerable<T>` or `IEnumerable` into a `ParallelQuery<T>`, enabling parallel execution for subsequent query operators.

**Technical Definition**
`System.Linq.ParallelEnumerable.AsParallel<TSource>(this IEnumerable<TSource> source)` returns a `ParallelQuery<TSource>` that binds the source to PLINQ's parallel execution engine. Once `AsParallel()` is called, subsequent standard query operators are bound to the PLINQ implementations and may execute in parallel. PLINQ decides whether to actually parallelize the query at runtime based on factors such as the query structure, data size, and available processors. The operator does not force parallelism; it enables it.

**Beginner-Friendly Explanation**
`AsParallel()` is the **“go parallel” switch**. You put it right after your collection, and PLINQ takes over.

---

### Purposes

- To **opt in** to PLINQ for a query.
- To bind the source to PLINQ's parallel execution engine.
- To enable subsequent query operators to execute in parallel.
- To allow the runtime to decide whether parallelism is beneficial.

---

### Syntax Rules and Structure

```csharp
source.AsParallel()
```

**Component breakdown**

- `source` — Any `IEnumerable<T>` or `IEnumerable`.
- `.AsParallel()` — Extension method from `ParallelEnumerable`.

#### Syntax Rules

- `AsParallel()` returns `ParallelQuery<TSource>`.
- Subsequent operators are PLINQ operators (from `ParallelEnumerable`).
- PLINQ may choose not to parallelize if it determines the query is not suitable.

#### Constraints and Limitations

- `AsParallel()` does not guarantee parallel execution; the runtime makes the final decision.
- Not all query operators have parallel implementations; some fall back to sequential execution.
- Overhead of partitioning and synchronization can outweigh benefits for small datasets.

---

### Annotated Code Examples and Expected Outputs

#### Example: `AsParallel()` with a Simple Query

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] data = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        var query = data.AsParallel()
            .Where(n => n % 2 == 0)
            .Select(n => n * n);

        Console.WriteLine(string.Join(", ", query));   // 4, 16, 36, 64, 100
    }
}
```

**Expected Output (order may vary without `AsOrdered`)**
```
4, 16, 36, 64, 100
```

**Why this output occurs**
`AsParallel()` enables parallel execution; the filter and projection run across partitions.

---

### Real-World Cases

- **Large array processing** — Converting a sequential LINQ query to PLINQ by adding `AsParallel()`.
- **Parallel aggregation** — `data.AsParallel().Sum(...)`.
- **Parallel filtering and projection** — `data.AsParallel().Where(...).Select(...)`.

---

### References

- ParallelEnumerable.AsParallel Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asparallel

---

## 3. `AsSequential()`

### Definitions

**Core Definition**
`AsSequential()` converts a `ParallelQuery<T>` back into an `IEnumerable<T>`, forcing **subsequent operators to execute sequentially**.

**Technical Definition**
`System.Linq.ParallelEnumerable.AsSequential<TSource>(this ParallelQuery<TSource> source)` converts a `ParallelQuery<TSource>` into an `IEnumerable<T>` to force sequential evaluation of the query. It is used when you want the first part of a query to run in parallel but the remainder to run sequentially—typically to preserve order established in a previous clause or to avoid parallelizing a portion that is not thread-safe or not beneficial.

**Beginner-Friendly Explanation**
`AsSequential()` is the **“go back to sequential” switch**. It stops parallel processing for the rest of the query.

---

### Purposes

- To **return to sequential execution** after a parallel section.
- To preserve the order established by a previous clause.
- To avoid parallelizing operations that are not thread-safe.
- To selectively parallelize only the beneficial part of a query.

---

### Syntax Rules and Structure

```csharp
parallelQuery.AsSequential()
```

**Component breakdown**

- `parallelQuery` — A `ParallelQuery<T>` (result of `AsParallel()` or other PLINQ operators).
- `.AsSequential()` — Converts to `IEnumerable<T>`.

#### Syntax Rules

- Must be called on a `ParallelQuery<T>` (or a type that has `AsSequential` as an extension method).
- After `AsSequential()`, subsequent operators use sequential LINQ implementations.

#### Constraints and Limitations

- Sequential processing is often slower than parallel processing for CPU-bound work.
- Use only when necessary (order preservation, thread safety, or measured benefit).

---

### Annotated Code Examples and Expected Outputs

#### Example: Combining Parallel and Sequential Sections

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] data = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        var result = data.AsParallel()
            .Where(n => n % 2 == 0)   // parallel
            .AsSequential()            // switch to sequential
            .Take(2);                  // sequential Take

        Console.WriteLine(string.Join(", ", result));   // 2, 4 (first two evens)
    }
}
```

**Expected Output**
```
2, 4
```

**Why this output occurs**
The `Where` runs in parallel; `AsSequential()` forces the subsequent `Take` to run sequentially, preserving the order of the filtered sequence.

---

### Real-World Cases

- **Order preservation** — Parallel filter, then sequential `Take` to get the first N in source order.
- **Thread-safety** — Parallel compute, then sequential aggregation that requires a single thread.
- **Hybrid pipelines** — Parallelize only the expensive portion.

---

### References

- ParallelEnumerable.AsSequential Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.assequential
- How to: Combine Parallel and Sequential LINQ Queries — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/how-to-combine-parallel-and-sequential-linq-queries

---

## 4. `WithDegreeOfParallelism()`

### Definitions

**Core Definition**
`WithDegreeOfParallelism()` limits the number of **worker threads** that PLINQ will use to execute the query.

**Technical Definition**
`System.Linq.ParallelEnumerable.WithDegreeOfParallelism<TSource>(this ParallelQuery<TSource> source, int degreeOfParallelism)` specifies the maximum number of processors that PLINQ should use to parallelize the query. The degree of parallelism is an upper bound; PLINQ may use fewer threads if it determines that doing so is more efficient. The default is the number of logical processors on the machine. This operator is useful when you want to leave CPU capacity for other work or when the overhead of too many threads outweighs the benefit.

**Beginner-Friendly Explanation**
`WithDegreeOfParallelism(n)` is the **“use at most n threads”** switch. It limits how many CPU cores PLINQ will use.

---

### Purposes

- To **limit CPU usage** and leave capacity for other processes.
- To **reduce synchronization overhead** when the query does not scale to many cores.
- To **avoid over-parallelization** in nested or concurrent scenarios.
- To provide **deterministic resource usage** in server environments.

---

### Syntax Rules and Structure

```csharp
source.AsParallel().WithDegreeOfParallelism(2)
```

**Component breakdown**

- `source.AsParallel()` — Opt in to PLINQ.
- `.WithDegreeOfParallelism(n)` — Limit to `n` threads.

#### Syntax Rules

- `degreeOfParallelism` must be a positive integer.
- PLINQ may use fewer threads than specified; it will not use more.
- The default is the number of logical processors.

#### Constraints and Limitations

- Setting the degree too low can underutilize the CPU.
- Setting the degree too high can increase contention and reduce performance.
- Not a guarantee; PLINQ may choose a lower degree.

---

### Annotated Code Examples and Expected Outputs

#### Example: Limiting Degree of Parallelism

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var result = Enumerable.Range(0, 100)
            .AsParallel()
            .WithDegreeOfParallelism(2)
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .ToList();

        Console.WriteLine(result.Count);   // 50
    }
}
```

**Expected Output**
```
50
```

**Why this output occurs**
`WithDegreeOfParallelism(2)` limits PLINQ to two threads; the query still produces the correct 50 even squares.

---

### Real-World Cases

- **Server environments** — Limiting PLINQ to avoid starving other requests.
- **Nested parallelism** — Preventing PLINQ from consuming all cores when already inside a parallel operation.
- **Benchmarking** — Testing different degrees of parallelism to find the sweet spot.

---

### References

- ParallelEnumerable.WithDegreeOfParallelism Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.withdegreeofparallelism

---

## 5. `AsOrdered()`

### Definitions

**Core Definition**
`AsOrdered()` enables **order preservation** in a PLINQ query, ensuring that the results appear in the same order as the source sequence.

**Technical Definition**
By default, PLINQ does **not** preserve the order of the source sequence. `System.Linq.ParallelEnumerable.AsOrdered<TSource>(this ParallelQuery<TSource> source)` overrides this default and enables order preservation for the remainder of the query (or until `AsUnordered()` is called). Order preservation is expensive: PLINQ must track the original position of each element and buffer results to restore order during merging. It should be used only when correctness requires it, and only for the portion of the query that needs it.

**Beginner-Friendly Explanation**
`AsOrdered()` is the **“keep the original order”** switch. By default, PLINQ scrambles results; `AsOrdered()` unscrambles them.

---

### Purposes

- To **preserve source order** in the output.
- To ensure that `Take`, `First`, or `ElementAt` return the correct first elements in source order.
- To satisfy business or UI requirements that depend on ordering.
- To make parallel query results **deterministic** with respect to source order.

---

### Syntax Rules and Structure

```csharp
source.AsParallel().AsOrdered()
```

**Component breakdown**

- `.AsParallel()` — Opt in to PLINQ.
- `.AsOrdered()` — Enable order preservation.

#### Syntax Rules

- Must be called on a `ParallelQuery<T>` (after `AsParallel()` or another PLINQ operator).
- Order preservation applies to subsequent operators until `AsUnordered()` is called.
- `OrderBy` and `ThenBy` imply `AsOrdered` for the sorted portion.

#### Constraints and Limitations

- **Performance cost** — Order preservation requires buffering and position tracking.
- Use only when needed; overuse can eliminate the benefits of parallelism.
- Not all operators respect order preservation equally.

---

### Annotated Code Examples and Expected Outputs

#### Example: `Take` with and without `AsOrdered`

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var cities = new[] { "A", "B", "C", "D", "E", "F", "G", "H" };

        // WITHOUT AsOrdered: Take(3) may return any 3 elements
        var unordered = cities.AsParallel().Where(c => c.CompareTo("C") > 0).Take(3).ToList();
        Console.WriteLine("Unordered: " + string.Join(", ", unordered.OrderBy(x => x)));

        // WITH AsOrdered: Take(3) returns the first 3 matching elements in source order
        var ordered = cities.AsParallel().AsOrdered().Where(c => c.CompareTo("C") > 0).Take(3).ToList();
        Console.WriteLine("Ordered: " + string.Join(", ", ordered));   // D, E, F
    }
}
```

**Expected Output (unordered order may vary)**
```
Unordered: D, E, F   (or another combination)
Ordered: D, E, F
```

**Why this output occurs**
`AsOrdered()` ensures `Take(3)` returns the first three matching elements in source order (`D`, `E`, `F`). Without `AsOrdered`, `Take` may return any three matching elements.

---

### Real-World Cases

- **Pagination** — `AsOrdered().Skip(page * size).Take(size)` to get the correct page in source order.
- **Top-N queries** — `AsOrdered().Take(n)` to get the first N elements in source order.
- **UI display** — Results must appear in a predictable order.

---

### References

- Order Preservation in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/order-preservation-in-plinq
- ParallelEnumerable.AsOrdered Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asordered

---

## 6. `AsUnordered()`

### Definitions

**Core Definition**
`AsUnordered()` **disables order preservation** for the remainder of a PLINQ query, restoring the default unordered (and faster) execution mode.

**Technical Definition**
`System.Linq.ParallelEnumerable.AsUnordered<TSource>(this ParallelQuery<TSource> source)` turns off order preservation that was previously enabled with `AsOrdered()` (or implied by `OrderBy`). After `AsUnordered()`, PLINQ is free to return results in any order, which can improve performance. The recommended pattern is to use `AsOrdered()` only for the portion of the query that requires ordering, then call `AsUnordered()` to release the ordering constraint for the rest of the query.

**Beginner-Friendly Explanation**
`AsUnordered()` is the **“stop keeping order, go fast again”** switch. Use it after the ordered portion is done.

---

### Purposes

- To **release the ordering constraint** when it is no longer needed.
- To **recover parallel performance** after an ordered section.
- To avoid unnecessary buffering in later stages of a query.
- To combine ordered and unordered sections in a single query efficiently.

---

### Syntax Rules and Structure

```csharp
source.AsParallel().AsOrdered()...AsUnordered()
```

**Component breakdown**

- `.AsOrdered()` — Enable ordering for an initial portion.
- `.AsUnordered()` — Disable ordering for the remainder.

#### Syntax Rules

- `AsUnordered()` must be called on a `ParallelQuery<T>`.
- It can follow `AsOrdered()` or `OrderBy`/`ThenBy`.
- It restores the default unordered behaviour for subsequent operators.

#### Constraints and Limitations

- Once `AsUnordered()` is called, the guarantee of source order is lost.
- Use only when downstream operators do not require order.

---

### Annotated Code Examples and Expected Outputs

#### Example: Ordered Section then Unordered Section

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = Enumerable.Range(0, 100).ToArray();

        var result = data.AsParallel()
            .AsOrdered()
            .Where(n => n % 2 == 0)
            .Take(5)                 // first 5 evens in source order
            .AsUnordered()           // release ordering
            .Select(n => n * 2)      // parallel, unordered
            .ToList();

        Console.WriteLine(string.Join(", ", result.OrderBy(x => x)));   // 0, 4, 8, 12, 16
    }
}
```

**Expected Output (result order may vary after `AsUnordered`)**
```
0, 4, 8, 12, 16
```

**Why this output occurs**
`AsOrdered()` ensures `Take(5)` returns the first five evens in source order; `AsUnordered()` then allows the `Select` to run without order-preservation overhead.

---

### Real-World Cases

- **Hybrid pipelines** — Ordered extraction followed by unordered transformation.
- **Performance tuning** — Removing unnecessary ordering after the requirement has been satisfied.
- **Join scenarios** — Ordered filtering followed by unordered joining.

---

### References

- Order Preservation in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/order-preservation-in-plinq
- ParallelEnumerable.AsUnordered Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asunordered

---

## 7. PLINQ Best Practices

### Definitions

**Core Definition**
PLINQ best practices are the guidelines for deciding **when** and **how** to use PLINQ so that parallelism improves performance rather than introducing overhead, race conditions, or unnecessary complexity.

**Technical Definition**
Best practices for PLINQ derive from Microsoft's parallel-programming guidance. Key recommendations include: **measure before parallelizing** (do not assume parallel is faster); **avoid shared mutable state** (eliminate race conditions); **avoid over-parallelization** (limit nested or excessive parallelism); **avoid calls to non-thread-safe methods**; **avoid unnecessary ordering operations** (`AsOrdered` is expensive); **use `AsSequential()` for sections that must not be parallelized**; and **consider PLINQ only for CPU-bound workloads** (not I/O-bound operations).

**Beginner-Friendly Explanation**
Best practices for PLINQ are: **measure first**, **do not share state**, **do not over-parallelize**, and **only use it for CPU-heavy work**.

---

### Guidelines and Structure

#### General Guidelines

```
// 1. Measure before parallelizing
// Compare sequential vs. PLINQ with BenchmarkDotNet or Stopwatch.

// 2. Avoid shared mutable state
// BAD: int total = 0; data.AsParallel().ForAll(n => total += n);
// GOOD: var total = data.AsParallel().Sum(n => n);

// 3. Avoid over-parallelization
// Do not nest PLINQ inside PLINQ or inside Parallel.For.

// 4. Avoid non-thread-safe methods
// Do not call FileStream.Write or other non-thread-safe APIs inside PLINQ.

// 5. Limit unnecessary ordering
// Use AsOrdered() only for the portion that needs it; then AsUnordered().

// 6. Use AsSequential() for the parts that must be sequential
var result = data.AsParallel().Where(...).AsSequential().Take(5);

// 7. PLINQ is for CPU-bound work
// Do not use PLINQ for I/O-bound operations; use async/await instead.
```

**Component breakdown**

- **Measure first** — PLINQ can be slower; verify with benchmarks.
- **No shared state** — Use thread-safe aggregation (`Sum`, `Aggregate` with seed).
- **No over-parallelization** — Nested parallelism hurts performance.
- **Thread-safe APIs only** — Non-thread-safe calls cause data corruption.
- **Limit ordering** — `AsOrdered` is costly; use `AsUnordered` when done.
- **Selective sequential** — `AsSequential` for order-sensitive or non-thread-safe parts.
- **CPU-bound only** — PLINQ is not for I/O-bound work.

#### Rules and Constraints

- **Do** measure sequential vs. PLINQ before adopting PLINQ.
- **Do** use thread-safe aggregation operators (`Sum`, `Average`, `Aggregate` with seed).
- **Do** call `AsUnordered()` after the ordered portion of a query.
- **Do** use `AsSequential()` for sections that must preserve order or avoid parallelism.
- **Do not** assume parallel is always faster.
- **Do not** write to shared memory locations inside a PLINQ query.
- **Do not** call non-thread-safe methods from PLINQ.
- **Do not** use PLINQ for I/O-bound operations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Thread-Safe Aggregation with PLINQ

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = Enumerable.Range(0, 10_000).ToArray();

        // SAFE: PLINQ aggregation with Sum
        var sum = data.AsParallel().Sum(n => n);
        Console.WriteLine($"Sum: {sum}");   // 49,995,000

        // SAFE: thread-local aggregation with Aggregate
        var product = data.AsParallel()
            .Aggregate(
                seed: 1L,
                func: (acc, n) => acc * (n + 1),
                resultSelector: acc => acc);
        Console.WriteLine($"Product is positive: {product > 0}");   // True
    }
}
```

**Expected Output**
```
Sum: 49995000
Product is positive: True
```

**Why this output occurs**
`Sum` is a thread-safe aggregation operator implemented by PLINQ; `Aggregate` with a seed and a combining function is also safe because PLINQ partitions the work and combines the partial results.

---

### Real-World Cases

- **Server-side processing** — Limit degree of parallelism to avoid starving request threads.
- **Data analysis** — Use PLINQ for CPU-bound aggregation over large datasets.
- **Image/audio processing** — Parallelize per-pixel or per-sample computations.

---

### References

- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Understanding Speedup in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/understanding-speedup-in-plinq
- Parallel LINQ (PLINQ) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq
- Essential C#: Running LINQ Queries in Parallel — https://essentialcsharp.com/running-linq-queries-in-parallel

---

## 8. PLINQ Pitfalls

### Definitions

**Core Definition**
PLINQ pitfalls are the **common mistakes and hazards** that cause PLINQ queries to produce incorrect results, perform worse than sequential LINQ, or introduce subtle concurrency bugs.

**Technical Definition**
PLINQ introduces parallelism-related complexity that does not exist in sequential code. Microsoft documents several specific pitfalls: **assuming parallel is always faster** (it is not, especially for small datasets or fast delegates); **writing to shared memory locations** (race conditions); **over-parallelization** (nested PLINQ or excessive `WithDegreeOfParallelism`); **calling non-thread-safe methods** (data corruption); and **unnecessary ordering operations** (`AsOrdered` is expensive). These pitfalls are not caught by the compiler and often manifest as intermittent, hard-to-reproduce bugs.

**Beginner-Friendly Explanation**
PLINQ pitfalls are **the ways PLINQ bites you**. The biggest trap is assuming it will make everything faster; the second is accidentally corrupting data by sharing state.

---

### Common Pitfalls

#### 1. Assuming Parallel Is Always Faster

```csharp
// BAD: small dataset, fast delegate — PLINQ is likely slower
var small = new[] { 1, 2, 3, 4, 5 };
var result = small.AsParallel().Where(n => n > 2).Select(n => n * 2).ToList();
```

**Why it is a pitfall:** The overhead of partitioning and synchronization outweighs the benefit. **Fix:** Measure; use PLINQ only for CPU-bound work on sufficiently large datasets.

---

#### 2. Writing to Shared Memory (Race Condition)

```csharp
// BAD: race condition on 'total'
int total = 0;
data.AsParallel().ForAll(n => total += n);   // unpredictable result
Console.WriteLine(total);                     // NOT 49,995,000
```

**Why it is a pitfall:** `total += n` is not atomic; multiple threads read and write concurrently, losing updates. **Fix:** Use PLINQ's thread-safe aggregation: `var total = data.AsParallel().Sum(n => n);`.

---

#### 3. Over-Parallelization

```csharp
// BAD: nested PLINQ — parallelism inside parallelism
var result = outer.AsParallel().SelectMany(o =>
    inner.AsParallel().Select(i => o + i)).ToList();
```

**Why it is a pitfall:** Nested parallelism creates excessive contention and thread-pool starvation. **Fix:** Parallelize only the outer loop, or use `AsSequential()` inside.

---

#### 4. Calling Non-Thread-Safe Methods

```csharp
// BAD: writing to a non-thread-safe stream from PLINQ
data.AsParallel().ForAll(n => writer.WriteLine(n));   // data corruption
```

**Why it is a pitfall:** `StreamWriter` is not thread-safe; concurrent writes corrupt data. **Fix:** Use a thread-safe sink or collect results and write sequentially.

---

#### 5. Unnecessary Ordering

```csharp
// BAD: AsOrdered when order is not needed
var result = data.AsParallel().AsOrdered().Where(n => n % 2 == 0).ToList();
```

**Why it is a pitfall:** Order preservation adds buffering and position-tracking overhead, slowing the query. **Fix:** Use `AsOrdered()` only when correctness requires it; call `AsUnordered()` afterward.

---

### Annotated Code Examples and Expected Outputs

#### Example: Race Condition Demonstrated

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int total = 0;
        var numbers = Enumerable.Range(0, 10_000);

        // UNSAFE: multiple threads writing to shared variable
        numbers.AsParallel().ForAll(n => total += n);

        Console.WriteLine($"Total (with race condition): {total}");
        // Expected: 49,995,000 but result is unpredictable
    }
}
```

**Expected Output (unpredictable, often less than expected)**
```
Total (with race condition): 3,217,456   (or any other value)
```

**Why this output occurs**
`total += n` is not atomic; concurrent reads and writes overwrite each other, losing updates.

---

#### Example: Correct Approach with `Sum`

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var numbers = Enumerable.Range(0, 10_000);
        var total = numbers.AsParallel().Sum(n => n);
        Console.WriteLine($"Total: {total}");   // 49,995,000
    }
}
```

**Expected Output**
```
Total: 49995000
```

**Why this output occurs**
`Sum` is implemented by PLINQ using thread-local partial sums combined safely, eliminating the race condition.

---

### Real-World Cases

- **Financial calculations** — A race condition in PLINQ aggregation can produce wrong totals.
- **Logging** — Concurrent writes to a `StreamWriter` can corrupt log files.
- **Server applications** — Over-parallelization can starve the thread pool and degrade throughput.

---

### References

- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Understanding Speedup in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/understanding-speedup-in-plinq
- Parallel LINQ (PLINQ) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| PLINQ (`AsParallel`) | Introduced in **.NET Framework 4.0**; stable. |
| `WithDegreeOfParallelism` | Core PLINQ operator; stable. |
| `AsOrdered` / `AsUnordered` | Core PLINQ operators; stable. |
| `AsSequential` | Core PLINQ operator; stable. |
| `WithMergeOptions` | Core PLINQ operator; stable. |
| PLINQ for I/O-bound work | **Not recommended**; use `async`/`await` or TPL Dataflow instead. |
| Nested PLINQ | **Discouraged**; leads to over-parallelization. |
| Shared mutable state in PLINQ | **Unsafe**; causes race conditions. |
| Non-thread-safe method calls in PLINQ | **Unsafe**; causes data corruption. |
| Speedup from PLINQ | **Implementation- and workload-dependent**; measure before adopting. |

---

## References (Consolidated)

- Introduction to PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq
- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Understanding Speedup in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/understanding-speedup-in-plinq
- Order Preservation in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/order-preservation-in-plinq
- Merge Options in PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/merge-options-in-plinq
- How to: Combine Parallel and Sequential LINQ Queries — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/how-to-combine-parallel-and-sequential-linq-queries
- ParallelEnumerable Class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable
- ParallelEnumerable.AsParallel Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asparallel
- ParallelEnumerable.AsSequential Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.assequential
- ParallelEnumerable.WithDegreeOfParallelism Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.withdegreeofparallelism
- ParallelEnumerable.AsOrdered Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asordered
- ParallelEnumerable.AsUnordered Method — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.parallelenumerable.asunordered
- Essential C#: Running LINQ Queries in Parallel — https://essentialcsharp.com/running-linq-queries-in-parallel
- Parallel LINQ (PLINQ) — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/13-c-sharp-async-sharphronous-programming/3-c-sharp-parallel-programming.mdx

---