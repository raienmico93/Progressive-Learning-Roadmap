# C# LINQ Performance — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
LINQ performance is the study of how efficiently LINQ queries execute — covering **when** queries run (deferred vs. immediate execution), **how many times** they run (multiple enumeration), **where** they run (in-memory vs. translated to a remote provider), and **what overhead** they incur (expression-tree construction and compilation, iterator allocations, closure captures). Understanding these factors is essential for writing LINQ code that is correct, scalable, and fast in production.

**Technical Definition**
LINQ performance is governed by several interacting mechanisms: **deferred execution** (queries are descriptions of work, executed only on enumeration), **iterator-based pipelines** (`IEnumerable<T>` chains that allocate enumerator objects and delegate invocations per element), **expression-tree translation** (`IQueryable<T>` queries are captured as `Expression` trees, translated by a provider into the target query language, with per-query construction and translation overhead), and **materialization semantics** (`ToList()` / `ToArray()` force immediate execution and allocate a result collection, preventing re-execution but also preventing streaming). The .NET runtime has progressively optimized LINQ: since **.NET 7**, `Enumerable.Count()` on an `ICollection<T>` is **O(1)** (it checks the `ICollection<T>.Count` property rather than iterating), `ToArray()` pre-sizes when possible, and `First()` short-circuits after the first match. The **.NET 7+ LINQ optimizer** is significantly better at recognizing patterns, fusing predicates, caching delegates, and specializing iterator implementations. Despite these improvements, the dominant performance decisions remain architectural: **where evaluation happens** (server vs. client) and **how many times it happens** (single vs. multiple enumeration).

**Beginner-Friendly Explanation**
LINQ is convenient, but convenience can hide expensive operations. A LINQ query is **lazy** — it does nothing until you ask for results. That is usually good, but it becomes a problem when you accidentally **run the same query twice** (multiple enumeration), **load an entire table into memory** before filtering (casting `IQueryable` to `IEnumerable`), or **materialize too early** with `ToList()` before applying filters. LINQ performance is about knowing **when your query actually runs** and **where the work is being done** — on the database server (fast, filtered) or in your application's memory (slow, unfiltered). This cheat sheet covers the key mechanisms, the common traps, and the practices that keep LINQ fast.

---

### Key Characteristics

- **Deferred execution is the default** — Most LINQ operators are lazy; execution happens on enumeration.
- **Multiple enumeration is the #1 pitfall** — A deferred query re-executes every time it is enumerated; this is especially costly for database queries.
- **`ToList()` is a double-edged sword** — It prevents re-execution but forces immediate, full materialization; calling it too early defeats filtering.
- **`IQueryable<T>` delegates work to the provider** — Filtering happens on the server (e.g., SQL `WHERE`), which is dramatically faster than client-side filtering.
- **`IEnumerable<T>` executes in memory** — All data must be loaded before filtering; this is the slow path for database-backed sources.
- **Expression trees add overhead** — Constructing, translating, and compiling expression trees costs CPU; query compilation can eliminate most of this overhead for repeated queries.
- **Modern .NET has LINQ optimizations** — .NET 7+ recognizes `ICollection<T>.Count`, pre-sizes `ToArray()`, and short-circuits `First()`.

---

### Prerequisites

- Understanding of **LINQ fundamentals** (query syntax and method syntax).
- Familiarity with **deferred execution** and **immediate execution**.
- Knowledge of **`IEnumerable<T>`** vs. **`IQueryable<T>`**.
- Awareness of **Entity Framework Core** or another LINQ provider (for the database sections).
- Basic understanding of **time complexity** and **memory allocation**.

---

### Related Programming Areas

- **LINQ fundamentals** — Deferred execution, method syntax, and standard query operators.
- **Expression trees** — The mechanism behind `IQueryable<T>` translation.
- **Entity Framework Core** — The primary consumer of `IQueryable<T>` performance considerations.
- **PLINQ** — Parallel LINQ has its own performance characteristics.
- **Memory management** — Iterator allocations, closure captures, and materialized collections.

---

### Core Concepts / Features

1. LINQ Performance (Overarching Concept)
2. Deferred Execution
3. Multiple Enumeration
4. `ToList()` Optimization
5. `IQueryable` vs. `IEnumerable`
6. Expression Trees
7. LINQ Performance Best Practices

---

## 1. LINQ Performance (Overarching Concept)

### Definitions

**Core Definition**
LINQ performance is the measure of how efficiently LINQ queries execute in terms of CPU time, memory allocation, and database load.

**Technical Definition**
LINQ performance is determined by the interaction of execution model (deferred vs. immediate), execution location (client vs. server), and implementation overhead (iterator allocation, delegate invocation, expression-tree construction and translation). The .NET runtime and LINQ providers have been progressively optimized, but the largest performance gains come from **architectural decisions** — filtering on the server, avoiding multiple enumeration, and projecting only needed columns — rather than from micro-optimizations.

**Beginner-Friendly Explanation**
LINQ performance is about **doing less work**: filtering on the database instead of in memory, running the query once instead of twice, and asking for only the columns you need.

---

### Purposes

- To avoid **hidden performance costs** in LINQ queries.
- To choose the right execution model (`IEnumerable` vs. `IQueryable`).
- To prevent **multiple database round-trips**.
- To reduce **memory allocations** and GC pressure.
- To write queries that scale with data size.

---

### Syntax Rules and Structure (Performance Factors)

| Factor | Impact | Mitigation |
|---|---|---|
| **Deferred execution** | Query runs on enumeration | Understand when it runs; materialize deliberately |
| **Multiple enumeration** | Query runs N times | Materialize once with `ToList()` |
| **`ToList()` too early** | Loads unnecessary data | Filter before materializing |
| **`IQueryable` → `IEnumerable` cast** | Loads all rows before filtering | Keep `IQueryable` until after filtering |
| **Expression-tree overhead** | CPU cost per query | Compile repeated queries (EF Core compiled queries) |
| **Projection (no `Select`)** | Loads all columns | Use `Select` to project only needed columns |

#### Syntax Rules

- Deferred operators return `IEnumerable<T>` or `IQueryable<T>`; immediate operators return concrete values or collections.
- `AsEnumerable()` forces client evaluation; use only after server-side filtering.
- `AsQueryable()` simulates `IQueryable` over in-memory data; it does not provide real translation.

#### Constraints and Limitations

- Provider translation is not guaranteed for all C# constructs; unsupported methods throw or fall back to client evaluation.
- Materialized collections consume memory proportional to the result set size.

---

### Annotated Code Examples and Expected Outputs

#### Example: Deferred vs. Immediate Performance Impact

```csharp
using System;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] data = Enumerable.Range(0, 10_000_000).ToArray();

        // Deferred: no work at definition
        var sw = Stopwatch.StartNew();
        var deferred = data.Where(n => n % 2 == 0).Select(n => n * 2);
        sw.Stop();
        Console.WriteLine($"Deferred definition: {sw.ElapsedMilliseconds} ms");

        // Immediate: materializes all results
        sw.Restart();
        var immediate = data.Where(n => n % 2 == 0).Select(n => n * 2).ToList();
        sw.Stop();
        Console.WriteLine($"Immediate materialization: {sw.ElapsedMilliseconds} ms");
        Console.WriteLine($"Result count: {immediate.Count}");
    }
}
```

**Expected Output (approximate)**
```
Deferred definition: 0 ms
Immediate materialization: 250 ms
Result count: 5000000
```

**Why this output occurs**
Defining a deferred query does no work; materializing it with `ToList()` executes the filter and projection across 10 million elements.

---

### Real-World Cases

- **Web APIs** — A single unfiltered `ToList()` on a large table can cause timeouts and memory pressure.
- **Reporting** — Multiple enumeration of a complex query can multiply database load.
- **High-throughput services** — Unnecessary allocations from iterators and closures degrade throughput.

---

### References

- Work with LINQ — C# (performance section) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/working-with-linq
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries

---

## 2. Deferred Execution

### Definitions

**Core Definition**
Deferred execution is the LINQ execution model where a query is **defined now but executed later** — specifically, when its results are enumerated.

**Technical Definition**
Deferred execution means that the operation is not performed at the point in the code where the query is declared; it is performed only when the query variable is enumerated (e.g., by `foreach`, `ToList()`, or an aggregate operator). Almost all standard query operators returning `IEnumerable<T>` or `IOrderedEnumerable<T>` execute in a deferred manner. Deferred execution can greatly improve performance when you have to manipulate large data collections, especially in programs that contain a series of chained queries or manipulations. However, it also means that the results depend on the contents of the data source **when the query is executed**, and that each enumeration re-executes the query.

**Beginner-Friendly Explanation**
Deferred execution means **“the query runs when you ask for the results, not when you write it.”** This is usually good, but it can cause hidden multiple executions.

---

### Purposes

- To enable **query composition** without executing each step prematurely.
- To support **lazy evaluation** — processing only the elements actually consumed.
- To allow providers to translate the entire query into a single optimized statement.
- To improve **memory efficiency** on large sequences.

---

### Syntax Rules and Structure

```csharp
// Deferred: nothing runs yet
var query = source.Where(x => predicate(x)).Select(x => projection(x));

// Execution triggered by enumeration
foreach (var item in query) { ... }
var list = query.ToList();   // forces immediate execution
```

**Component breakdown**

- `source` — Any `IEnumerable<T>` or `IQueryable<T>`.
- `query` — A deferred sequence (iterator or expression tree).
- Enumeration — `foreach`, `ToList()`, `ToArray()`, aggregate operators.

#### Syntax Rules

- Deferred operators return `IEnumerable<T>` or `IQueryable<T>` and do not execute immediately.
- Each enumeration **re-executes** the query.
- `yield return` in an iterator method produces a deferred sequence.

#### Constraints and Limitations

- If the query is never enumerated, it never executes (no side effects, no database calls).
- Multiple enumerations can cause **multiple executions** (performance and correctness issue).
- For `IQueryable<T>`, unsupported C# constructs may throw at execution time.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Deferred Execution

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        var query = numbers.Where(n => n > 2);   // NOT executed

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
The `Where` call returns an iterator object; the predicate runs only when `foreach` pulls elements.

---

#### Example 2: Side Effects at Enumeration Time

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        var query = numbers.Where(n =>
        {
            Console.WriteLine($"Evaluating {n}");
            return n % 2 == 0;
        });

        Console.WriteLine("Before loop");
        foreach (var n in query) Console.WriteLine($"Keeping {n}");
    }
}
```

**Expected Output**
```
Before loop
Evaluating 1
Evaluating 2
Keeping 2
Evaluating 3
Evaluating 4
Keeping 4
Evaluating 5
```

**Why this output occurs**
The predicate runs element-by-element during enumeration, not at query definition.

---

### Real-World Cases

- **Entity Framework Core** — A chain of `.Where().OrderBy().Select()` translates to a single SQL query executed on enumeration.
- **File processing** — `File.ReadLines` is deferred; `File.ReadAllLines` is eager.
- **Large datasets** — Streaming results without loading everything into memory.

---

### References

- Deferred Execution and Lazy Evaluation in LINQ to XML — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/deferred-execution-lazy-evaluation
- Work with LINQ — C# (performance section) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/working-with-linq

---

## 3. Multiple Enumeration

### Definitions

**Core Definition**
Multiple enumeration occurs when a **deferred query is enumerated more than once**, causing the query to execute repeatedly and potentially producing different results or unnecessary work.

**Technical Definition**
A collection of type `IEnumerable` or `IEnumerable<T>` has the ability to defer enumeration when it is generated. Many LINQ methods, such as `Select`, use deferred execution. Enumeration starts when the collection is passed into a LINQ enumeration method, like `ElementAt`, or used in a `foreach` statement. The enumeration result is not calculated once and cached, like `Lazy<T>`. If the enumeration operation itself is expensive — for example, a query into a database — multiple enumerations would hurt the performance of the program. If the enumeration operation has side effects, multiple enumerations might result in bugs. Analyzer **CA1851** detects possible multiple enumerations.

**Beginner-Friendly Explanation**
Multiple enumeration is **running the same query twice without realizing it**. If your query hits a database, you just doubled the database load.

---

### Purposes

- To **avoid duplicate query execution**.
- To prevent **inconsistent results** when the source changes between enumerations.
- To avoid **side-effect bugs** in predicates with side effects.
- To reduce **database load** and CPU consumption.

---

### Syntax Rules and Structure (The Anti-Pattern)

```csharp
// BAD: multiple enumerations
var query = users.Where(u => u.IsActive);   // deferred
var count = query.Count();                   // enumeration 1
var first = query.FirstOrDefault();          // enumeration 2

// GOOD: materialize once
var list = users.Where(u => u.IsActive).ToList();
var count2 = list.Count;
var first2 = list.FirstOrDefault();
```

**Component breakdown**

- `query` — A deferred sequence.
- `query.Count()` — Executes the query to count.
- `query.FirstOrDefault()` — Executes the query again.
- `ToList()` — Materializes once; subsequent operations use the list.

#### Syntax Rules

- Deferred queries re-execute on every enumeration.
- Materialize with `ToList()`, `ToArray()`, or `ToDictionary()` when you need to enumerate multiple times.
- `Lazy<T>` caches its value; deferred LINQ queries do **not**.

#### Constraints and Limitations

- Materialization allocates a collection; use it only when reuse is needed.
- CA1851 is enabled by default in .NET 10+; it warns about possible multiple enumerations.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Enumeration Bug

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        var query = numbers.Where(n => n % 2 == 0);

        Console.WriteLine(query.Count());      // 2 (first execution)
        numbers.Add(6);
        Console.WriteLine(query.Count());      // 3 (second execution — different result!)
    }
}
```

**Expected Output**
```
2
3
```

**Why this output occurs**
The deferred query re-executes on each `Count()` call; the source changed between enumerations.

---

### Real-World Cases

- **Entity Framework Core** — Multiple enumeration causes duplicate SQL queries.
- **ASP.NET Core** — A query enumerated once for count and once for data doubles database load.
- **Background services** — Stale data in long-running deferred queries.

---

### References

- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries

---

## 4. `ToList()` Optimization

### Definitions

**Core Definition**
`ToList()` is an **immediate execution operator** that materializes a deferred query into a `List<T>`, forcing execution and caching the results.

**Technical Definition**
`ToList()` enumerates the source sequence immediately and returns a `List<T>` containing all elements. This prevents multiple enumeration and provides a mutable, reusable result. However, calling `ToList()` **too early** — before filtering or projecting — pulls all data into memory, defeating the purpose of server-side filtering. The golden rule is: **filter and project first, then materialize**. `ToList()` allocates a `List<T>` (and its backing array); `ToArray()` allocates a `T[]`. .NET 7+ pre-sizes `ToArray()` when possible.

**Beginner-Friendly Explanation**
`ToList()` is **“run the query now and give me a list.”** It is useful for reuse, but harmful if you call it before filtering.

---

### Purposes

- To **prevent multiple enumeration** by materializing once.
- To **force execution** at a specific point (e.g., before a `DbContext` is disposed).
- To obtain a **mutable collection** for further processing.
- To **cache** results for repeated use.

---

### Syntax Rules and Structure (Correct vs. Incorrect Usage)

```csharp
// BAD: ToList() too early — loads ALL users, then filters in memory
var allUsers = dbContext.Users.ToList();
var active = allUsers.Where(u => u.IsActive);

// GOOD: filter on the server, then materialize
var active = dbContext.Users
    .Where(u => u.IsActive)
    .ToList();

// GOOD: project only needed columns, then materialize
var names = dbContext.Users
    .Where(u => u.IsActive)
    .Select(u => u.Name)
    .ToList();
```

**Component breakdown**

- `dbContext.Users` — `IQueryable<User>`.
- `.Where(...)` — Composes an expression tree (deferred).
- `.Select(...)` — Composes a projection (deferred).
- `.ToList()` — Executes the query and materializes the result.

#### Syntax Rules

- `ToList()` forces immediate execution.
- Materialize **after** filtering and projection.
- Use `ToListAsync()` for asynchronous database queries.
- Avoid `ToList().Where(...)` — that filters in memory.

#### Constraints and Limitations

- Materialization loads the entire result set into memory.
- Calling `ToList()` too early can cause excessive memory usage and slow queries.
- `ToList()` allocates a new list even if the source is already a list (use `AsEnumerable()` or cast to avoid duplication if safe).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Early `ToList()` vs. Late `ToList()`

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var users = new List<(string Name, bool IsActive)>
        {
            ("Alice", true), ("Bob", false), ("Charlie", true), ("Dave", false)
        };

        // BAD: materializes all, then filters in memory
        var all = users.ToList();
        var activeBad = all.Where(u => u.IsActive);

        // GOOD: filters before materializing (for IEnumerable, this is deferred until ToList)
        var activeGood = users.Where(u => u.IsActive).ToList();

        Console.WriteLine($"Bad count: {activeBad.Count()}");   // 2
        Console.WriteLine($"Good count: {activeGood.Count}");   // 2
    }
}
```

**Expected Output**
```
Bad count: 2
Good count: 2
```

**Why this output occurs**
For in-memory `List<T>`, the difference is less dramatic, but for `IQueryable<T>` the early `ToList()` would load the entire table before filtering.

---

#### Example 2: `ToList()` Prevents Multiple Enumeration

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5 };
        var query = numbers.Where(n => n % 2 == 0);

        var list = query.ToList();   // materialize once
        Console.WriteLine(list.Count);   // 2
        Console.WriteLine(list.First());  // 2 (no re-execution)
    }
}
```

**Expected Output**
```
2
2
```

**Why this output occurs**
`ToList()` materializes the results; subsequent operations use the list, not the query.

---

### Real-World Cases

- **API responses** — Materialize once before returning from a repository.
- **Caching** — Store materialized results for reuse.
- **Pagination** — `Take(n).ToList()` to get a page of results.

---

### References

- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs (ToList patterns) - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- Do you avoid materializing an IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-avoid-materializing-an-ienumerable/
- C9-CRUNCH-SHARP — Allocation Reduction (ToList vs ToArray) — GitHub - https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-07-performance-benchmarkdotnet-spans/lecture-notes/03-allocation-reduction-arraypool-valuetask.md

---

## 5. `IQueryable` vs. `IEnumerable`

### Definitions

**Core Definition**
`IEnumerable<T>` executes LINQ queries **in memory** on the client; `IQueryable<T>` translates LINQ queries into the **target language** (e.g., SQL) and executes them **on the server**.

**Technical Definition**
The main difference is that LINQ operators for `IQueryable<T>` take **expression trees** (`Expression<Func<…>>`) instead of delegates. This means the query logic is captured as data that a provider can translate. When querying a database with `IEnumerable`, the `select` query executes on the server, loads all data in-memory on the client, and then filters locally — doing more work and becoming slow. With `IQueryable`, the `select` query executes on the server **with all filters applied**, doing less work and becoming fast. Casting `IQueryable<T>` to `IEnumerable<T>` forces the provider to materialize all data at the point of the cast, causing significant extra database load.

**Beginner-Friendly Explanation**
`IQueryable` is **“send the filter to the database.”** `IEnumerable` is **“download everything and filter in memory.”** The first is fast; the second is slow.

---

### Purposes

- To choose the **correct execution location** for queries.
- To filter on the **database server** rather than in the application.
- To minimize **data transfer** over the network.
- To leverage **database indexes** and query optimization.

---

### Syntax Rules and Structure (Comparison)

| Aspect | `IEnumerable<T>` | `IQueryable<T>` |
|---|---|---|
| **Execution** | In-memory (client) | Remote (database, service) |
| **Delegates** | `Func<T, bool>` | `Expression<Func<T, bool>>` |
| **Extension class** | `Enumerable` | `Queryable` |
| **Filtering** | After loading data | Before loading data (server-side) |
| **Performance** | Slower for database queries | Faster — filters applied at source |
| **Cast to other** | N/A | `AsEnumerable()` forces client evaluation |

**Component breakdown**

- `IEnumerable<T>` — Pull-based, in-memory sequence.
- `IQueryable<T>` — Carries an expression tree; translated by the provider.
- `AsEnumerable()` — Forces the switch to client evaluation; use only after server-side filtering.

#### Syntax Rules

- `IQueryable<T>` inherits `IEnumerable<T>`; enumeration triggers execution.
- `AsQueryable()` converts an `IEnumerable<T>` to `IQueryable<T>` (for in-memory testing).
- `AsEnumerable()` converts an `IQueryable<T>` to `IEnumerable<T>` (forces client-side evaluation).

#### Constraints and Limitations

- Provider may not translate all expression nodes.
- Mixing client-side and server-side evaluation can cause unexpected behavior.
- `IQueryable<T>` queries are composable; they execute only on enumeration.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: `IQueryable` vs. `IEnumerable` Filtering

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var users = new List<User>
        {
            new() { Name = "Alice", IsActive = true },
            new() { Name = "Bob", IsActive = false },
            new() { Name = "Charlie", IsActive = true }
        };

        // IEnumerable: filtering happens in memory (after loading all)
        IEnumerable<User> enumerable = users;
        var activeEnumerable = enumerable.Where(u => u.IsActive);

        // IQueryable: filtering is translated (simulated with AsQueryable)
        IQueryable<User> queryable = users.AsQueryable();
        var activeQueryable = queryable.Where(u => u.IsActive);

        Console.WriteLine($"Enumerable count: {activeEnumerable.Count()}");
        Console.WriteLine($"Queryable count: {activeQueryable.Count()}");
    }
}

public class User { public string Name { get; set; } public bool IsActive { get; set; } }
```

**Expected Output**
```
Enumerable count: 2
Queryable count: 2
```

**Why this output occurs**
For in-memory data, the results are the same, but for a real `IQueryable<T>` (e.g., EF Core), the filtering would be sent to the database as SQL.

---

#### Example 2: Casting `IQueryable` to `IEnumerable` (The Slow Path)

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        // Simulating a database query with IQueryable
        var dbUsers = new[] { "Alice", "Bob", "Charlie" }.AsQueryable();

        // BAD: cast to IEnumerable BEFORE filtering — loads all data
        IEnumerable<string> bad = dbUsers.AsEnumerable();
        var badResult = bad.Where(u => u.StartsWith("A"));

        // GOOD: filter on IQueryable, THEN cast/materialize
        var goodResult = dbUsers.Where(u => u.StartsWith("A")).ToList();

        Console.WriteLine($"Bad: {badResult.Count()}, Good: {goodResult.Count}");
        // Bad: 1, Good: 1
    }
}
```

**Expected Output**
```
Bad: 1, Good: 1
```

**Why this output occurs**
The bad path loads all data at the `AsEnumerable()` call; the good path filters on the server (simulated) before materializing.

---

### Real-World Cases

- **Entity Framework Core** — `DbSet<T>` is `IQueryable<T>`; filtering is translated to SQL.
- **OData clients** — `IQueryable<T>` queries are translated to OData URLs.
- **Repository patterns** — Returning `IQueryable<T>` allows callers to compose server-side filters.

---

### References

- What's the difference between Enumerable and Queryable — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/b749e6bf-6465-4ca0-8e3d-f220d779e7f2
- Do you take care when casting IQueryable to IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-take-care-when-casting-iqueryable-to-ienumerable/
- IQueryable vs IEnumerable performance — Microsoft Learn (Turkish) - https://learn.microsoft.com/tr-tr/dotnet/standard/linq/iqueryable-vs-ienumerable

---

## 6. Expression Trees

### Definitions

**Core Definition**
An expression tree is an **in-memory data structure that represents code**, used by `IQueryable<T>` providers to translate LINQ queries into another language (e.g., SQL).

**Technical Definition**
Expression trees represent code in a tree-like data structure, where each node is an expression. For `IQueryable<T>`, LINQ operators accept `Expression<Func<…>>` instead of delegates, capturing the query as data. The provider walks the tree to produce the target query (SQL, OData, etc.). This translation has a **non-trivial CPU cost** — expression trees can become quite big, and traversing them (and potentially creating copies due to changes) can take a while. LINQ to SQL and EF often use more CPU on the client than in SQL Server because they excessively manipulate the Expression AST. **Compiled queries** eliminate most of this overhead by performing the expression-tree construction, translation, and compilation once, then reusing the compiled delegate.

**Beginner-Friendly Explanation**
Expression trees are **the “recipe” that the database provider reads**. Building and translating the recipe costs CPU. If you run the same query many times, you can compile it once and skip the translation cost.

---

### Purposes

- To enable **query translation** by providers.
- To allow **dynamic query construction** at runtime.
- To support **compiled queries** for repeated execution.
- To provide a data representation of code for inspection.

---

### Syntax Rules and Structure (Overhead Model)

```csharp
// Expression tree construction and translation (per query)
var query = db.Users.Where(u => u.Age > 30);   // builds expression tree
var list = query.ToList();                       // translates to SQL and executes

// Compiled query (EF Core): translation happens once
var compiled = EF.CompileQuery((DbContext db, int age) =>
    db.Users.Where(u => u.Age > age));
```

**Component breakdown**

- `Where(u => u.Age > 30)` — The lambda is captured as an `Expression<Func<User, bool>>`.
- Provider translation — Walks the tree to produce SQL.
- Compilation — Produces a reusable delegate (EF Core `CompileQuery`).

#### Syntax Rules

- Expression-tree construction happens when the query is defined.
- Translation to SQL happens when the query is executed.
- Compiled queries cache the translation and delegate.

#### Constraints and Limitations

- Expression-tree overhead is per-query; for one-off queries, it may be negligible.
- Compiled queries add code complexity; use only for hot-path repeated queries.
- Not all C# constructs are expressible in expression trees.

---

### Annotated Code Examples and Expected Outputs

#### Example: Compiled Query in EF Core

```csharp
using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;

public class User { public int Id { get; set; } public string Name { get; set; } public int Age { get; set; } }
public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
}

class Program
{
    static readonly Func<AppDbContext, int, IQueryable<User>> _usersOverAge =
        EF.CompileQuery((AppDbContext db, int age) =>
            db.Users.Where(u => u.Age > age));

    static void Main()
    {
        using var db = new AppDbContext();
        var query = _usersOverAge(db, 30);   // translation cached
        // var list = query.ToList();         // executes SQL
    }
}
```

**Expected Output (conceptual)**
```
-- SQL executed on ToList():
SELECT [u].[Id], [u].[Name], [u].[Age]
FROM [Users] AS [u]
WHERE [u].[Age] > @age
```

**Why this output occurs**
The compiled query translates the expression tree to SQL once and reuses the compiled delegate, eliminating per-query translation overhead.

---

### Real-World Cases

- **Entity Framework Core** — Compiled queries for hot-path, repeated queries.
- **LINQ to SQL** — Compiled queries for performance-critical operations.
- **Dynamic query builders** — Expression trees enable runtime query composition.

---

### References

- DLinq (Linq to SQL) Performance (Part 5) — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/ricom/dlinq-linq-to-sql-performance-part-5
- Expression Trees (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- The cost of inappropriate linqing — Ayende - https://www.ayende.net/archive/2007/12/04/the-cost-of-inappropriate-linqing.aspx

---

## 7. LINQ Performance Best Practices

### Definitions

**Core Definition**
LINQ performance best practices are the guidelines that ensure queries execute **once**, **in the right place**, and **with minimal data**, avoiding the common traps of multiple enumeration, premature materialization, and client-side evaluation.

**Technical Definition**
Best practices derive from Microsoft's documentation and community experience. Key recommendations include: **filter before materializing**; **project only needed columns with `Select`**; **materialize once when reusing results**; **avoid casting `IQueryable` to `IEnumerable` before filtering**; **use `Any()` instead of `Count() > 0`**; **use `Take()` early for paging**; **use async versions (`ToListAsync`) for database queries**; and **compile repeated EF Core queries**.

**Beginner-Friendly Explanation**
The best practices are: **filter on the database, not in memory**, **run the query once**, and **ask for only what you need**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Filter before materializing
var active = db.Users.Where(u => u.IsActive).ToList();

// 2. Project only needed columns
var names = db.Users.Where(u => u.IsActive).Select(u => u.Name).ToList();

// 3. Materialize once when reusing results
var list = db.Users.Where(u => u.IsActive).ToList();
var count = list.Count;
var first = list.FirstOrDefault();

// 4. Use Any() instead of Count() > 0
bool hasActive = db.Users.Any(u => u.IsActive);

// 5. Use Take() early for paging
var firstTen = db.Users.Where(u => u.IsActive).OrderBy(u => u.Name).Take(10).ToList();

// 6. Keep IQueryable until after filtering
// BAD: db.Users.AsEnumerable().Where(...)
// GOOD: db.Users.Where(...).AsEnumerable()

// 7. Use compiled queries for hot paths (EF Core)
var compiled = EF.CompileQuery((DbContext db, bool active) =>
    db.Users.Where(u => u.IsActive == active));
```

**Component breakdown**

- **Filter before materializing** — Reduces data transfer and memory usage.
- **Project only needed columns** — Reduces payload size and memory.
- **Materialize once** — Avoids multiple database round-trips.
- **`Any()`** — Short-circuits after the first match; `Count()` counts everything.
- **`Take()` early** — Translates to SQL `TOP`/`LIMIT`, preventing full table reads.
- **Keep `IQueryable`** — Filtering stays on the server.
- **Compiled queries** — Eliminate per-query expression-tree overhead for repeated queries.

#### Rules and Constraints

- **Do** filter before calling `ToList()`.
- **Do** use `Select` to project only needed columns.
- **Do** materialize once when reusing results.
- **Do** use `Any()` for existence checks.
- **Do** use `Take()` for paging.
- **Do not** cast `IQueryable` to `IEnumerable` before filtering.
- **Do not** call `ToList()` too early.
- **Do not** enumerate the same deferred query multiple times.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correct Ordering of Operations

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var users = new[]
        {
            new { Name = "Alice", IsActive = true, Age = 30 },
            new { Name = "Bob", IsActive = false, Age = 25 },
            new { Name = "Charlie", IsActive = true, Age = 35 }
        }.AsQueryable();

        // BAD: materialize early, then filter in memory
        var bad = users.ToList().Where(u => u.IsActive);

        // GOOD: filter on IQueryable, then materialize
        var good = users.Where(u => u.IsActive).ToList();

        Console.WriteLine($"Bad count: {bad.Count()}, Good count: {good.Count}");
        // Bad count: 2, Good count: 2
    }
}
```

**Expected Output**
```
Bad count: 2, Good count: 2
```

**Why this output occurs**
For in-memory data, the difference is subtle; for a real `IQueryable`, the bad path loads the entire table before filtering, while the good path filters on the server.

---

### Real-World Cases

- **EF Core queries** — Filter and project on the server; materialize once.
- **High-traffic APIs** — Avoid multiple enumeration and unnecessary column loading.
- **Reporting** — Use `Take` for paging and `Select` for lightweight projections.

---

### References

- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Do you avoid iterating multiple times? — SSW.Rules - https://www.ssw.com.au/rules/do-you-avoid-iterating-multiple-times/
- Do you avoid materializing an IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-avoid-materializing-an-ienumerable/
- Do you take care when casting IQueryable to IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-take-care-when-casting-iqueryable-to-ienumerable/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Deferred execution | Core since **.NET 3.5 / C# 3.0**; stable. |
| `IQueryable<T>` | Core since **.NET 3.5**; stable. |
| CA1851 (multiple enumeration) | Introduced in **.NET 7**; enabled by default in .NET 10. |
| .NET 7+ LINQ optimizations | `Count()` O(1) on `ICollection<T>`; pre-sized `ToArray()`; short-circuiting `First()`. |
| Compiled queries (EF Core) | Recommended for hot-path repeated queries; per-query overhead for one-off queries may not justify compilation. |
| `AsEnumerable()` | Forces client evaluation; use deliberately after server-side filtering. |
| `AsQueryable()` | Simulates `IQueryable` over in-memory data; does not provide real translation. |
| Expression-tree overhead | **Implementation-dependent**; significant for large trees and frequent queries. |

---

## References (Consolidated)

- Work with LINQ — C# (performance section) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/tutorials/working-with-linq
- Deferred Execution and Lazy Evaluation in LINQ to XML — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/deferred-execution-lazy-evaluation
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- What's the difference between Enumerable and Queryable — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/b749e6bf-6465-4ca0-8e3d-f220d779e7f2
- DLinq (Linq to SQL) Performance (Part 5) — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/ricom/dlinq-linq-to-sql-performance-part-5
- Expression Trees (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/
- Do you avoid iterating multiple times? — SSW.Rules - https://www.ssw.com.au/rules/do-you-avoid-iterating-multiple-times/
- Do you avoid materializing an IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-avoid-materializing-an-ienumerable/
- Do you take care when casting IQueryable to IEnumerable? — SSW.Rules - https://www.ssw.com.au/rules/do-you-take-care-when-casting-iqueryable-to-ienumerable/
- The cost of inappropriate linqing — Ayende - https://www.ayende.net/archive/2007/12/04/the-cost-of-inappropriate-linqing.aspx
- C9-CRUNCH-SHARP — Allocation Reduction (ToList vs ToArray) — GitHub - https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/f9331d18afa92513d15ad5ae2ac9192f25ec35d1/curriculum/week-07-performance-benchmarkdotnet-spans/lecture-notes/03-allocation-reduction-arraypool-valuetask.md
- dotnet-linq-optimization — NeverSight Skills (GitHub) - https://raw.githubusercontent.com/NeverSight/skills_feed/refs/heads/main/data/skills-md/wshaddix/dotnet-skills/dotnet-linq-optimization/SKILL.md

---