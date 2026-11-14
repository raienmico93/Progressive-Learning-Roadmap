# LINQ Optimization in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
LINQ optimization is the discipline of writing Language Integrated Query (LINQ) expressions that produce correct results while minimizing execution time, memory allocation, and database round-trips.

**Technical Definition**
LINQ optimization in .NET addresses four interconnected concerns: **deferred execution** (queries are not executed when defined, but when enumerated), **multiple enumeration** (re-executing the same deferred query, often against a database, because its results were not materialized), **`IQueryable<T>` vs. `IEnumerable<T>`** (server-side expression-tree translation vs. client-side in-memory delegate execution), and **query composition order** (filtering before projecting, projecting before materializing, using `Any()` instead of `Count() > 0`, and applying `Take()` early). The .NET runtime and EF Core have progressively improved LINQ performance — for example, .NET 7+ recognizes `ICollection<T>` so that `Count()` is O(1), and EF Core can compile queries once and reuse them across calls — but the largest gains come from understanding when and where queries execute .

**Beginner-Friendly Explanation**
LINQ is “SQL for C# collections,” but it has a hidden trick: most LINQ methods do not run when you write them — they run when you loop over the result. This is called deferred execution, and it is both a superpower and a trap. If you accidentally enumerate the same query twice, you might hit the database twice. If you call `ToList()` too early, you might load a million rows into memory just to filter them in C#. LINQ optimization is about knowing exactly when your query runs, what it runs against, and how much data it moves.

---

### Key Characteristics

- **Deferred by default** — `Where`, `Select`, `OrderBy`, and most operators build a pipeline; nothing executes until enumeration .
- **Immediate execution triggers** — `ToList()`, `ToArray()`, `ToDictionary()`, `Count()`, `Sum()`, `First()`, `Any()`, and `foreach` force execution .
- **Two worlds** — `IQueryable<T>` represents a query that can be translated to SQL and executed by the database; `IEnumerable<T>` represents in-memory execution .
- **Materialization is the boundary** — `AsEnumerable()`, `ToList()`, and casts to `IEnumerable<T>` switch from server-side to client-side evaluation .
- **Multiple enumeration is the most common performance bug** — Analyzer CA1851 detects it automatically .
- **Composition order matters** — Filtering early, projecting early, and paging early reduce data volume and CPU work .

---

### Prerequisites

- Working knowledge of C# collections (`List<T>`, arrays, dictionaries).
- Familiarity with LINQ query syntax and method syntax.
- Basic understanding of Entity Framework Core or another LINQ provider (for `IQueryable<T>` scenarios).
- Awareness of the difference between client-side (in-memory) and server-side (database) execution.
- A profiler or BenchmarkDotNet for measuring query performance.

---

### Related Programming Areas

- **Entity Framework Core** — `IQueryable<T>` translation, compiled queries, and `AsNoTracking` .
- **Memory optimization** — Allocation reduction from iterators, closures, and materialized collections.
- **Async optimization** — `ToListAsync()`, `FirstOrDefaultAsync()`, and `IAsyncEnumerable<T>`.
- **Performance fundamentals** — Latency, throughput, and database load.
- **Benchmarking** — BenchmarkDotNet for measuring LINQ alternatives.

---

### Core Concepts / Features

1. LINQ Performance
2. Deferred Execution
3. Multiple Enumeration
4. `IQueryable` vs. `IEnumerable`
5. LINQ Optimization Best Practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with “To”), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. LINQ Performance

### Definitions

**Core Definition**
LINQ performance is the measure of how efficiently LINQ queries execute — how much CPU time, memory allocation, and database work they consume.

**Technical Definition**
LINQ performance is shaped by **where** execution happens (database server vs. application memory), **how many times** it happens (deferred execution and multiple enumeration), **how much data** is moved (projection and filtering order), and **what overhead** the LINQ operators themselves add (iterator allocation, delegate invocation, closure capture). In .NET 7+, the LINQ-to-Objects implementation recognizes common collection interfaces (`ICollection<T>`) and optimizes `Count()`, `ToArray()`, and `First()` accordingly, but these optimizations do not apply to `IQueryable<T>` or to custom iterators . On the database side, EF Core compiles expression trees into SQL; compiled queries eliminate repeated compilation overhead for hot paths .

**Beginner-Friendly Explanation**
LINQ performance is “how fast your LINQ queries run and how much work they do.” The biggest factors are: are you filtering on the database or in memory? Are you running the same query twice? Are you loading entire tables when you only need two columns? Getting these right matters far more than micro-optimizing individual LINQ operators.

---

### Purposes

- To avoid hidden database round-trips caused by deferred execution.
- To reduce memory allocation from unnecessary materialization.
- To ensure filtering and projection happen at the correct layer (server vs. client).
- To eliminate multiple enumeration of expensive queries.
- To choose the right LINQ method for the task (`Any` vs. `Count`, `First` vs. `Where().First()`).
- To establish a measurable baseline before optimizing.

---

### Syntax Rules and Structure

#### General Performance Decision Flow

```
1. Is the source IQueryable<T> (database) or IEnumerable<T> (in-memory)?
2. If IQueryable<T>: filter, project, and page on the server before materializing.
3. If IEnumerable<T>: avoid multiple enumeration; materialize once if reused.
4. Use the cheapest method for the intent (Any, First, Take).
5. Measure with BenchmarkDotNet or EF Core logging.
```

**Component breakdown**

- **Source type** — `IQueryable<T>` translates to SQL; `IEnumerable<T>` runs in memory .
- **Materialization point** — The call that forces execution (`ToList`, `foreach`, `Count`) .
- **Method choice** — `Any()` short-circuits; `Count() > 0` may count everything .
- **Measurement** — BenchmarkDotNet for micro-benchmarks; EF Core query logging for SQL translation .

#### Syntax Rules

- Filter before projecting; project before materializing .
- Use `Any()` instead of `Count() > 0` for existence checks .
- Use `FirstOrDefault(predicate)` instead of `Where(predicate).FirstOrDefault()` .
- Apply `Take()` before `ToList()` to limit data at the source .
- Materialize once with `ToList()` or `ToArray()` if the result is used multiple times .

#### Constraints and Limitations

- `IQueryable<T>` optimizations depend on the provider; not all C# expressions translate to SQL.
- `IEnumerable<T>` LINQ-to-Objects overhead is real but usually secondary to database round-trips.
- Compiled queries cannot handle dynamic `Include()` or conditional `Where()` shapes .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Filtering Before Projecting vs. After

```csharp
// ❌ INEFFICIENT: projects every row, then filters the projected names
var names = people
    .Select(p => ExpensiveNameFormatting(p))
    .Where(name => name.StartsWith("A"))
    .ToList();

// ✅ EFFICIENT: filters first, then projects only matching rows
var names = people
    .Where(p => p.Name.StartsWith("A"))
    .Select(p => ExpensiveNameFormatting(p))
    .ToList();
```

**Expected Output (conceptual)**

```
Inefficient: ExpensiveNameFormatting runs on every element.
Efficient: ExpensiveNameFormatting runs only on elements whose name starts with "A".
```

**Why this output occurs**

- LINQ processes operators in pipeline order.
- Filtering first reduces the number of elements that reach the projection .

---

#### Example 2: `Any()` vs. `Count() > 0`

```csharp
// ❌ SLOWER: counts all matching elements
bool exists = collection.Count(x => x.Id == 5) > 0;

// ✅ FASTER: stops at the first match
bool exists = collection.Any(x => x.Id == 5);
```

**Expected Output (conceptual)**

```
Any(): returns true after examining the first matching element.
Count() > 0: examines every element to compute the count.
```

**Why this output occurs**

- `Any()` short-circuits on the first match.
- `Count()` must enumerate the entire sequence (unless the source is `ICollection<T>` and .NET optimizes it) .

---

### Real-World Cases

- **ASP.NET Core APIs** — Filtering and paging on the database before materializing DTOs.
- **Reporting dashboards** — Aggregating on the server with `GroupBy` and `Sum`, not in memory.
- **Search endpoints** — Using `Any()` for existence checks instead of `Count()`.

---

### References

- C# LINQ Performance — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/10-c-sharp-linq/13-c-sharp-linq-performance.mdx
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- LINQ (ZA06xx) performance rules — dotnet/roslyn-analyzers - https://github.com/dotnet/roslyn-analyzers

---

## 2. Deferred Execution

### Definitions

**Core Definition**
Deferred execution means a LINQ query is not executed when it is defined, but only when its results are enumerated.

**Technical Definition**
Most LINQ operators (`Where`, `Select`, `SelectMany`, `OrderBy`, `Take`, `Skip`, `Distinct`) build a query pipeline — an object that implements `IEnumerable<T>` or `IQueryable<T>` — without executing it. Execution is triggered by enumeration (`foreach`), by conversion to a collection (`ToList`, `ToArray`, `ToDictionary`), or by scalar-returning operators (`Count`, `Sum`, `First`, `Any`). For `IQueryable<T>`, the pipeline is an expression tree that is translated to SQL only when enumerated. Deferred execution enables query composition and short-circuiting, but it also creates the conditions for multiple enumeration and surprising re-execution .

**Beginner-Friendly Explanation**
Deferred execution is “lazy evaluation.” When you write `var query = numbers.Where(n => n > 5);`, nothing happens yet. The query is a recipe, not a result. It only cooks when you ask for the food — by looping over it or calling `ToList()`. This is great for composition, but dangerous if you accidentally cook the same recipe twice.

---

### Purposes

- To enable query composition without executing intermediate steps.
- To allow short-circuiting (e.g., `First()`, `Any()`, `Take()`).
- To avoid unnecessary work when only part of the result is needed.
- To support `IQueryable<T>` expression-tree translation.
- To make LINQ queries declarative and readable.
- To create the conditions for multiple enumeration bugs (which must be understood and prevented).

---

### Syntax Rules and Structure

#### Deferred vs. Immediate Operators

| Category | Operators |
|---|---|
| **Deferred** | `Where`, `Select`, `SelectMany`, `OrderBy`, `ThenBy`, `Take`, `Skip`, `Distinct`, `GroupBy`, `Join` |
| **Immediate** | `ToList`, `ToArray`, `ToDictionary`, `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, `First`, `FirstOrDefault`, `Single`, `SingleOrDefault`, `Any`, `All`, `Contains` |

*Sources: Compile-N-Run LINQ performance guide; Syncfusion LINQ optimization blog* 

#### General Syntax

```csharp
// Deferred: nothing executes yet
var query = numbers.Where(n => n > 5);

// Execution happens here
foreach (var n in query) { ... }

// Or here
var list = query.ToList();
```

**Component breakdown**

- `query` — A deferred pipeline object.
- `foreach` — Triggers enumeration.
- `ToList()` — Triggers enumeration and materializes into a `List<T>`.

#### Syntax Rules

- Deferred queries are re-executed every time they are enumerated.
- Materialize with `ToList()`, `ToArray()`, or `ToDictionary()` to execute exactly once.
- For `IQueryable<T>`, deferred execution means the SQL is generated at enumeration time.
- Deferred queries capture variables by reference; loop variables require careful handling .

#### Constraints and Limitations

- Deferred execution makes it easy to accidentally run the same query multiple times.
- A deferred query can become stale if the underlying data changes between enumerations.
- Some `IEnumerable<T>` sources (e.g., `Stream`) can only be enumerated once .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Deferred Execution Pipeline

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

var query = numbers
    .Where(n => n > 5)       // deferred
    .Select(n => n * n)       // deferred
    .OrderBy(n => n);         // deferred

Console.WriteLine("Query defined, nothing executed.");

foreach (var n in query)      // execution starts here
    Console.WriteLine(n);
```

**Expected Output**

```
Query defined, nothing executed.
36
49
64
81
100
```

**Why this output occurs**

- `Where`, `Select`, and `OrderBy` build a pipeline without executing.
- The `foreach` triggers enumeration, which executes the full pipeline .

---

#### Example 2: Re-execution of a Deferred Query

```csharp
var query = numbers.Where(n => n > 5);

var count1 = query.Count();      // executes the query
var count2 = query.Count();      // executes the query AGAIN

Console.WriteLine($"{count1}, {count2}");
```

**Expected Output**

```
5, 5
```

**Why this output occurs**

- `Count()` is an immediate operator; it enumerates the deferred query each time.
- If the source is a database, this results in two SQL queries .

---

### Real-World Cases

- **EF Core queries** — Building a query with multiple `.Where()` and `.Include()` calls before materializing.
- **Streaming large datasets** — Using `IAsyncEnumerable<T>` to process results without buffering.
- **Short-circuiting searches** — `FirstOrDefault(predicate)` stops at the first match without enumerating the rest.

---

### References

- Deferred Execution — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/10-c-sharp-linq/11-c-sharp-linq-deferred-execution.mdx
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- .NET LINQ Deferred Execution — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/10-net-linq-deferred-execution.mdx

---

## 3. Multiple Enumeration

### Definitions

**Core Definition**
Multiple enumeration occurs when the same deferred LINQ query is enumerated more than once, causing the underlying operation — often a database query — to execute repeatedly.

**Technical Definition**
Because LINQ queries are deferred, each enumeration re-executes the pipeline. For `IEnumerable<T>` sources backed by in-memory collections, this is usually a CPU cost (re-running the iterator). For `IQueryable<T>` sources backed by a database, each enumeration issues a new SQL query. Analyzer CA1851 (Possible multiple enumerations of `IEnumerable` collection) detects this pattern at compile time and is enabled by default in .NET 10 . The standard fix is to materialize the query once with `ToList()`, `ToArray()`, or `ToDictionary()` and reuse the materialized collection .

**Beginner-Friendly Explanation**
Multiple enumeration is “running the same query twice by accident.” You write `var activeUsers = users.Where(u => u.IsActive);`, then call `activeUsers.Count()` and `activeUsers.FirstOrDefault()` — and each call re-runs the filter, and if `users` is a database table, each call hits the database. The fix is to call `ToList()` once and reuse the list.

---

### Purposes

- To understand why deferred queries execute more than once.
- To detect multiple enumeration with analyzer CA1851.
- To fix multiple enumeration by materializing once.
- To avoid duplicate database round-trips and redundant CPU work.
- To prevent side effects from being applied multiple times.

---

### Syntax Rules and Structure

#### Detection

```
Enable CA1851 (enabled by default in .NET 10).
Look for repeated use of the same IEnumerable/IQueryable variable in:
  - Count(), Any(), First(), FirstOrDefault()
  - foreach loops
  - LINQ operators that enumerate
```

#### Fix Pattern

```csharp
// ❌ BUG: enumerates twice
var query = users.Where(u => u.IsActive);
var count = query.Count();
var first = query.FirstOrDefault();

// ✅ FIX: materialize once
var activeUsers = users.Where(u => u.IsActive).ToList();
var count = activeUsers.Count;        // List<T>.Count property, no enumeration
var first = activeUsers.FirstOrDefault(); // in-memory enumeration
```

**Component breakdown**

- `ToList()` — Executes the query once and stores the results.
- `List<T>.Count` — A property, not a LINQ method, so no re-enumeration .
- Reusing the materialized list — Safe for multiple reads.

#### Syntax Rules

- Materialize with `ToList()`, `ToArray()`, or `ToDictionary()` when the result is used more than once.
- For `IQueryable<T>`, materializing also prevents repeated SQL generation and execution.
- Use `List<T>.Count` instead of `Count()` after materialization.
- Be especially careful in `foreach` loops that call LINQ methods on the same query inside the loop .

#### Constraints and Limitations

- Materializing large result sets increases memory usage.
- Some sources (e.g., `Stream`) can only be enumerated once; materialization is mandatory .
- CA1851 may not detect all cases, especially when the query is passed across method boundaries .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiple Enumeration with `IQueryable`

```csharp
// ❌ BUG: two SQL queries
IQueryable<Order> query = dbContext.Orders.Where(o => o.Status == Status.Active);
var count = query.Count();        // SQL query 1
var items = query.ToList();       // SQL query 2

// ✅ FIX: one SQL query
var items = dbContext.Orders.Where(o => o.Status == Status.Active).ToList();
var count = items.Count;          // in-memory
```

**Expected Output (conceptual)**

```
Bug: 2 SQL queries executed against the database.
Fix: 1 SQL query executed; count is computed in memory from the materialized list.
```

**Why this output occurs**

- `Count()` and `ToList()` each enumerate the `IQueryable<T>`, generating and executing SQL.
- Materializing once and reusing the list eliminates the second query .

---

#### Example 2: CA1851 Analyzer Warning

```csharp
public void Process(IEnumerable<int> input)
{
    var count = input.Count();   // CA1851: first enumeration
    foreach (var i in input)    // second enumeration
    {
        // ...
    }
}
```

**Expected Output (conceptual)**

```
CA1851: Possible multiple enumerations of 'IEnumerable' collection.
```

**Why this output occurs**

- The analyzer detects that `input` is enumerated by both `Count()` and `foreach` .

---

### Real-World Cases

- **API endpoints** — Returning both a count and a page of results; materialize once and reuse.
- **Validation** — Checking `Any()` and then iterating; materialize first.
- **Logging** — Logging `Count()` before processing; materialize once.

---

### References

- CA1851: Possible multiple enumerations of 'IEnumerable' collection — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Do you avoid iterating multiple times? — dotnet/skills - https://github.com/dotnet/skills
- LINQ multiple enumeration fix — Stack Overflow - https://stackoverflow.com/questions/…/linq-multiple-enumeration

---

## 4. `IQueryable` vs. `IEnumerable`

### Definitions

**Core Definition**
`IQueryable<T>` represents a query that can be translated to a provider-specific language (such as SQL) and executed remotely; `IEnumerable<T>` represents an in-memory sequence that is iterated in the current process.

**Technical Definition**
`IQueryable<T>` extends `IEnumerable<T>` and carries an **expression tree** rather than a delegate. When enumerated, a LINQ provider (e.g., EF Core) translates the expression tree into SQL, sends it to the database, and returns the results. `IEnumerable<T>` uses delegates (`Func<T, bool>`, `Func<T, TResult>`) that execute in memory. The critical performance consequence: filtering, projection, and paging on `IQueryable<T>` happen **on the database server**; on `IEnumerable<T>` they happen **in application memory** after all data has been loaded . Casting `IQueryable<T>` to `IEnumerable<T>` (or calling `AsEnumerable()`) switches to client-side evaluation and can cause the entire table to be loaded .

**Beginner-Friendly Explanation**
`IQueryable` is “the database query.” You build it in C#, but the database executes it. `IEnumerable` is “the in-memory collection.” You build it in C#, and your application executes it. If you accidentally turn an `IQueryable` into an `IEnumerable` before filtering, you force the database to send you the entire table so you can filter it in memory — which is usually much slower.

---

### Purposes

- To push filtering, projection, and paging to the database server.
- To minimize data transferred over the network.
- To reduce application memory usage and CPU time.
- To enable provider-specific optimizations (SQL indexes, query plans).
- To avoid accidental full-table loads caused by casting to `IEnumerable`.
- To understand when client-side evaluation is necessary (custom C# logic).

---

### Syntax Rules and Structure

#### Comparison Table

| Aspect | `IQueryable<T>` | `IEnumerable<T>` |
|---|---|---|
| **Namespace** | `System.Linq` | `System.Collections.Generic` |
| **Execution location** | Database server (or remote provider) | Application memory |
| **Operator parameter** | `Expression<Func<…>>` (expression tree) | `Func<…>` (delegate) |
| **Filtering** | Translated to `WHERE` clause | Runs in memory |
| **Projection** | Translated to `SELECT` columns | Runs in memory |
| **Paging** | Translated to `LIMIT`/`OFFSET` | Runs in memory after full load |
| **Deferred execution** | Yes | Yes |
| **Best for** | Database queries, remote data sources | In-memory collections, custom logic |

*Sources: Microsoft Learn; Stack Overflow; Syncfusion* 

#### General Syntax

```csharp
// ✅ Server-side filtering
IQueryable<Order> query = dbContext.Orders
    .Where(o => o.Status == Status.Active);
var result = query.ToList();   // SQL: SELECT ... WHERE Status = 'Active'

// ❌ Client-side filtering (full table load)
IEnumerable<Order> query = dbContext.Orders.AsEnumerable();
var result = query.Where(o => o.Status == Status.Active).ToList();
// SQL: SELECT * FROM Orders  (no WHERE clause)
```

**Component breakdown**

- `IQueryable<T>` — Keeps the expression tree intact for SQL translation.
- `AsEnumerable()` — Switches to client-side evaluation .
- `.Where()` on `IQueryable<T>` — Adds to the expression tree; no execution.
- `.Where()` on `IEnumerable<T>` — Executes in memory .

#### Syntax Rules

- Keep the source as `IQueryable<T>` until the last possible moment.
- Filter, project, and page **before** calling `ToList()` or `AsEnumerable()`.
- Use `AsEnumerable()` only when you need client-side logic that cannot be translated to SQL.
- Never cast `IQueryable<T>` to `IEnumerable<T>` before filtering unless you intentionally want client-side evaluation.

#### Constraints and Limitations

- Not all C# expressions translate to SQL; unsupported expressions throw or fall back to client evaluation.
- `IQueryable<T>` requires a LINQ provider (EF Core, LINQ to SQL, etc.); plain collections do not implement it.
- Client-side evaluation may be necessary for custom methods or complex logic .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Server-Side vs. Client-Side Filtering

```csharp
// ✅ SERVER-SIDE: WHERE clause translated to SQL
var activeOrders = dbContext.Orders
    .Where(o => o.Status == Status.Active)
    .ToList();
// SQL: SELECT ... FROM Orders WHERE Status = 1

// ❌ CLIENT-SIDE: full table loaded, then filtered in memory
var activeOrders = dbContext.Orders
    .AsEnumerable()
    .Where(o => o.Status == Status.Active)
    .ToList();
// SQL: SELECT * FROM Orders
```

**Expected Output (conceptual)**

```
Server-side: only active orders are transferred from the database.
Client-side: all orders are transferred, then filtered in memory.
```

**Why this output occurs**

- `IQueryable<T>.Where` adds a `WHERE` clause to the SQL.
- `AsEnumerable()` forces materialization of the entire table before the `Where` runs .

---

#### Example 2: Projection Before Materialization

```csharp
// ❌ Loads all columns, then projects in memory
var names = dbContext.Orders
    .ToList()
    .Select(o => o.CustomerName);

// ✅ Projects on the server, loads only the needed column
var names = dbContext.Orders
    .Select(o => o.CustomerName)
    .ToList();
// SQL: SELECT CustomerName FROM Orders
```

**Expected Output (conceptual)**

```
Inefficient: SELECT * FROM Orders, then extract CustomerName in memory.
Efficient: SELECT CustomerName FROM Orders.
```

**Why this output occurs**

- Projecting before materialization translates to a `SELECT` of only the required columns .

---

### Real-World Cases

- **API list endpoints** — `IQueryable<T>` with `Where`, `OrderBy`, `Skip`, `Take`, then `ToList()`.
- **Search** — Server-side filtering on indexed columns, not in-memory string matching.
- **Reporting** — Aggregations (`GroupBy`, `Sum`) executed as SQL, not in memory.

---

### References

- What's the difference between Enumerable and Queryable — Microsoft Learn - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/b749e6bf-6465-4ca0-8e3d-f220d779e7f2
- Do you take care when casting IQueryable to IEnumerable? — dotnet/skills - https://github.com/dotnet/skills
- dotnet-linq-optimization — wshaddix/dotnet-skills - https://github.com/wshaddix/dotnet-skills

---

## 5. LINQ Optimization Best Practices

### Definitions

**Core Definition**
LINQ optimization best practices are empirically grounded guidelines for writing LINQ queries that minimize execution time, memory allocation, and database load while remaining correct and readable.

**Technical Definition**
Best practices span **materialization discipline** (materialize once, not multiple times), **layer discipline** (filter and project on `IQueryable<T>` before switching to `IEnumerable<T>`), **method selection** (use `Any` instead of `Count() > 0`, `FirstOrDefault(predicate)` instead of `Where(predicate).FirstOrDefault()`), **composition order** (filter before project, project before materialize, page early), and **measurement** (BenchmarkDotNet, EF Core query logging). They are supported by analyzer CA1851 and by .NET runtime improvements that optimize `Count()` and `ToArray()` for `ICollection<T>` sources .

**Beginner-Friendly Explanation**
LINQ best practices are “the rules that keep LINQ fast.” They include: don’t run the same query twice, filter on the database not in memory, use `Any()` instead of `Count() > 0`, and project only the columns you need. Following these rules prevents the most common LINQ performance traps.

---

### Purposes

- To eliminate multiple enumeration and duplicate database queries.
- To push filtering, projection, and paging to the correct layer.
- To reduce memory allocation from unnecessary materialization.
- To choose the cheapest LINQ method for each intent.
- To make LINQ queries measurable and optimizable.
- To prevent performance regressions through analyzers and benchmarks.

---

### Guidelines and Structure

#### Materialize Once, Reuse Many Times

```csharp
// ✅ Materialize once
var activeUsers = users.Where(u => u.IsActive).ToList();
var count = activeUsers.Count;        // List<T>.Count property
var first = activeUsers.FirstOrDefault();

// ❌ Multiple enumeration
var query = users.Where(u => u.IsActive);
var count = query.Count();            // enumeration 1
var first = query.FirstOrDefault();   // enumeration 2
```

- Use `ToList()`, `ToArray()`, or `ToDictionary()` when the result is reused .
- After materialization, use `List<T>.Count` instead of `Count()`.

#### Keep `IQueryable<T>` Until the Last Moment

```csharp
// ✅ Server-side filtering and projection
var dtos = dbContext.Orders
    .Where(o => o.Status == Status.Active)
    .Select(o => new { o.Id, o.Total })
    .ToList();

// ❌ Client-side filtering
var dtos = dbContext.Orders
    .AsEnumerable()
    .Where(o => o.Status == Status.Active)
    .ToList();
```

- Filter, project, and page on `IQueryable<T>` before materializing .

#### Use the Right Method

| Intent | Best Method | Avoid |
|---|---|---|
| Existence check | `Any(predicate)` | `Count(predicate) > 0`  |
| First match | `FirstOrDefault(predicate)` | `Where(predicate).FirstOrDefault()`  |
| Single match | `SingleOrDefault(predicate)` | `Where(predicate).SingleOrDefault()` |
| Count of a materialized list | `List<T>.Count` | `Count()` on `IEnumerable<T>` |

#### Filter Before Projecting, Project Before Materializing

```csharp
// ✅ Filter → project → materialize
var names = people
    .Where(p => p.IsActive)
    .Select(p => p.Name)
    .ToList();

// ❌ Project → filter → materialize (does more work)
var names = people
    .Select(p => p.Name)
    .Where(n => n.StartsWith("A"))
    .ToList();
```

- Filtering first reduces the number of elements that reach the projection .

#### Use `Take()` Early for Paging

```csharp
// ✅ Paging at the source
var firstTen = dbContext.Users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .Take(10)
    .ToList();
```

- `Take()` before `ToList()` translates to `LIMIT`/`OFFSET` on the server .

#### Avoid Mixing `IEnumerable` and `IQueryable`

```csharp
// ❌ Mixing causes client-side evaluation
IQueryable<User> query = dbContext.Users;
IEnumerable<string> names = new[] { "Alice", "Bob" };
var result = query.Where(u => names.Contains(u.Name)).ToList();

// ✅ Use a server-translatable predicate
var result = dbContext.Users
    .Where(u => u.Name == "Alice" || u.Name == "Bob")
    .ToList();
```

- Mixing an in-memory collection into an `IQueryable` predicate can force client evaluation .

#### Use Compiled Queries for Hot Paths

```csharp
private static readonly Func<AppDbContext, Guid, Task<Order?>> s_findById =
    EF.CompileAsyncQuery((AppDbContext ctx, Guid id) =>
        ctx.Orders.FirstOrDefault(o => o.Id == id));
```

- Compiled queries eliminate repeated expression-tree compilation for high-frequency queries .

#### Measure, Don’t Guess

```bash
# BenchmarkDotNet for micro-benchmarks
dotnet run -c Release --project benchmarks

# EF Core query logging for SQL verification
# Check that WHERE, SELECT, and LIMIT appear in the generated SQL
```

- Use BenchmarkDotNet for before/after comparison.
- Use EF Core logging to verify server-side translation .

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Fully Optimized `IQueryable` Query

```csharp
public async Task<List<OrderDto>> GetRecentOrdersAsync(
    DateTime cutoff,
    CancellationToken cancellationToken)
{
    return await dbContext.Orders
        .Where(o => o.CreatedAt > cutoff && o.Status == Status.Active)  // server-side filter
        .OrderByDescending(o => o.CreatedAt)                            // server-side sort
        .Take(50)                                                       // server-side LIMIT
        .Select(o => new OrderDto { Id = o.Id, Total = o.Total })       // server-side projection
        .ToListAsync(cancellationToken);                                // materialize once
}
```

**Expected Output (conceptual SQL)**

```sql
SELECT TOP 50 Id, Total
FROM Orders
WHERE CreatedAt > @cutoff AND Status = 1
ORDER BY CreatedAt DESC
```

**Why this output occurs**

- All operations stay on `IQueryable<T>` until `ToListAsync()`.
- Filtering, sorting, paging, and projection are translated to SQL .

---

#### Example 2: Materialize Once for Reuse

```csharp
var activeUsers = await dbContext.Users
    .Where(u => u.IsActive)
    .ToListAsync();

var count = activeUsers.Count;             // property access
var first = activeUsers.FirstOrDefault();  // in-memory enumeration
var names = activeUsers.Select(u => u.Name).ToList(); // in-memory projection
```

**Expected Output (conceptual)**

```
One SQL query: SELECT * FROM Users WHERE IsActive = 1
All subsequent operations run in memory.
```

**Why this output occurs**

- `ToListAsync()` executes the query once.
- `List<T>` supports multiple enumerations without re-querying .

---

### Real-World Cases

- **High-traffic APIs** — Server-side filtering, projection, and paging with `IQueryable<T>`.
- **Dashboards** — Aggregations executed as SQL, not in memory.
- **Search endpoints** — `Any()` for existence checks, `FirstOrDefault(predicate)` for first match.
- **Hot-path queries** — Compiled EF Core queries for repeated lookups.

---

### References

- C# LINQ Performance — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/10-c-sharp-linq/13-c-sharp-linq-performance.mdx
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- dotnet-linq-optimization — wshaddix/dotnet-skills - https://github.com/wshaddix/dotnet-skills
- LINQ (ZA06xx) performance rules — dotnet/roslyn-analyzers - https://github.com/dotnet/roslyn-analyzers

---

## Summary Table: LINQ Optimization Quick Reference

| Trap | Symptom | Fix |
|---|---|---|
| **Multiple enumeration** | Same query executes twice (two SQL queries) | Materialize once with `ToList()` and reuse  |
| **Early `ToList()`** | Entire table loaded before filtering | Filter on `IQueryable<T>` before `ToList()`  |
| **Late `ToList()`** | Multiple database queries from deferred execution | Materialize once when reused  |
| **`IEnumerable` before filtering** | Full table loaded, filtered in memory | Keep `IQueryable<T>` until after filtering  |
| **Projecting too late** | All columns loaded, then one selected | `Select()` before `ToList()`  |
| **`Count() > 0`** | Counts all matching elements | Use `Any()`  |
| **`Where().FirstOrDefault()`** | Builds a pipeline then takes first | Use `FirstOrDefault(predicate)`  |
| **Mixing `IEnumerable` and `IQueryable`** | Client-side evaluation | Use server-translatable predicates  |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **CA1851 analyzer** | Introduced in .NET 7; enabled by default in .NET 10  |
| **`Count()` on `ICollection<T>`** | O(1) optimization in .NET 7+  |
| **`ToArray()` pre-sizing** | Optimized in .NET 7+ for `ICollection<T>`  |
| **`AsEnumerable()`** | Switches to client-side evaluation; can cause full table loads  |
| **Compiled queries** | Cannot handle dynamic `Include()` or conditional `Where()` shapes  |
| **`IAsyncEnumerable<T>`** | Streaming alternative to materializing large result sets |
| **PLINQ** | Parallel LINQ can help CPU-bound queries but may not always be faster  |

---

## References (Consolidated)

- C# LINQ Performance — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/10-c-sharp-linq/13-c-sharp-linq-performance.mdx
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- What's the difference between Enumerable and Queryable — Microsoft Learn - https://learn.microsoft.com/en-us/archive/msdn-technet-forums/b749e6bf-6465-4ca0-8e3d-f220d779e7f2
- Deferred Execution — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/10-c-sharp-linq/11-c-sharp-linq-deferred-execution.mdx
- .NET LINQ Deferred Execution — Compile-N-Run - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/10-net-linq-deferred-execution.mdx
- dotnet-linq-optimization — wshaddix/dotnet-skills - https://github.com/wshaddix/dotnet-skills
- Do you avoid iterating multiple times? — dotnet/skills - https://github.com/dotnet/skills
- LINQ (ZA06xx) performance rules — dotnet/roslyn-analyzers - https://github.com/dotnet/roslyn-analyzers
- Do you take care when casting IQueryable to IEnumerable? — dotnet/skills - https://github.com/dotnet/skills
- Advanced Performance Topics — EF Core - https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics
- Working with LINQ — C# - https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- Potential Pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Single vs. Split Queries — EF Core - https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries