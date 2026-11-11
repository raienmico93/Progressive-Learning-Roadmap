# C# LINQ Method Syntax — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**LINQ method syntax** (also called **fluent syntax**) is the functional style of writing LINQ queries as **chains of extension method calls** on a source sequence. Instead of SQL-like query keywords, you write `source.Operator(lambda).Operator(lambda).…`, where each operator is an extension method on `IEnumerable<T>` (via `Enumerable`) or `IQueryable<T>` (via `Queryable`). Method syntax is the **foundational form** of LINQ: query syntax is compiler sugar that translates into exactly these method calls.

**Technical Definition**
LINQ method syntax uses the extension methods declared on `System.Linq.Enumerable` (for `IEnumerable<T>`) and `System.Linq.Queryable` (for `IQueryable<T>`). Each standard query operator (`Where`, `Select`, `OrderBy`, `GroupBy`, `Join`, `Take`, `Skip`, `First`, `Aggregate`, etc.) is a **static generic extension method** whose first parameter is the source sequence and whose remaining parameters are typically **delegates** (`Func<TSource, bool>`, `Func<TSource, TResult>`, `Func<TSource, TKey>`, `IComparer<TKey>`, `IEqualityComparer<TKey>`) for `IEnumerable<T>`, or **expression trees** (`Expression<Func<…>>`) for `IQueryable<T>`. Method syntax composes through **deferred execution** (except for operators that immediately produce a value or materialize a collection), and chains read left-to-right in the order in which operations are applied. Since **C# 3.0**, lambdas provide the concise inline predicate/projection syntax that makes method chains idiomatic. The choice between method syntax and query syntax is largely stylistic; method syntax is required for operators with no query-syntax equivalent (e.g., `Count`, `First`, `Take`, `Skip`, `Aggregate`).

**Beginner-Friendly Explanation**
LINQ method syntax is **a chain of `.Method()` calls** that you read left to right. It is “**start with the data, then call operators one after another**.” For example:

```csharp
var result = numbers.Where(n => n > 5).OrderBy(n => n).Select(n => n * 2);
```

Read it as: “Start with `numbers`, **where** each is greater than 5, **order by** the number, **select** the doubled value.” Each `.Method()` transforms or filters the sequence and hands the result to the next method. Method syntax is the **most flexible** form of LINQ: every operator is available, chains compose cleanly, and the code reads top-to-bottom like a pipeline. Query syntax (`from … where … select …`) is just a prettier way to write the same thing for complex queries—but under the hood, the compiler turns it into these exact method calls. If you can read method syntax, you can read all of LINQ.

---

### Key Characteristics

- **Pipeline composition** — Read left-to-right; each operator transforms the sequence.
- **Deferred execution** — Most operators are lazy; execution happens on enumeration.
- **Strongly typed** — Every operator is generic and preserves or transforms element types.
- **Lambda-driven** — Predicates, projections, and selectors are lambdas (`Func<…>`).
- **Available for both `IEnumerable<T>` and `IQueryable<T>`** — Same syntax, different execution strategies.
- **Complete operator coverage** — Every standard query operator is available; some only exist in method syntax.
- **Composable with query syntax** — The two forms can be mixed (query syntax translates to method calls).

---

### Prerequisites

- Basic understanding of **C# generics** (`List<T>`, `IEnumerable<T>`).
- Familiarity with **lambda expressions** and **delegates** (`Func<T>`, `Action<T>`).
- Knowledge of **extension methods** and `using System.Linq;`.
- Understanding of **deferred vs. immediate execution**.
- Awareness of `IEnumerable<T>` vs. `IQueryable<T>` for in-memory vs. remote execution.

---

### Related Programming Areas

- **Query syntax** — The SQL-like alternative that compiles to method syntax.
- **LINQ to Objects** — In-memory `IEnumerable<T>` operators.
- **LINQ to Entities** — `IQueryable<T>` operators translated to SQL.
- **PLINQ** — Parallel `IEnumerable<T>` using `AsParallel()`.
- **Functional programming** — Higher-order functions and pipelines.

---

### Core Concepts / Features

- **Filtering** — `Where`, `Take`, `Skip`, `TakeWhile`, `SkipWhile`, `Distinct`
- **Projection** — `Select`, `SelectMany`, `Cast`, `OfType`
- **Ordering** — `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Reverse`
- **Grouping and joining** — `GroupBy`, `Join`, `GroupJoin`
- **Set operations** — `Union`, `Intersect`, `Except`, `Concat`
- **Element operators** — `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault`, `ElementAt`, `ElementAtOrDefault`
- **Quantifiers** — `Any`, `All`, `Contains`
- **Aggregation** — `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, `Aggregate`
- **Conversion** — `ToList`, `ToArray`, `ToDictionary`, `ToHashSet`, `AsEnumerable`, `AsQueryable`
- **Best practices**

---

## 1. `Where()`

### Definitions

- **Core:** Filters a sequence to only elements satisfying a predicate.
- **Technical:** `IEnumerable<T> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)`; for `IQueryable<T>`, the delegate is `Expression<Func<TSource, bool>>`. Deferred execution; O(n) per enumeration.
- **Beginner:** “Keep only the items that pass my test.”

### Purposes

- To filter a sequence by a condition.
- To reduce the set of elements processed downstream.
- To translate to SQL `WHERE` for `IQueryable`.
- To compose complex predicates with logical operators.

### Syntax Rules

```
source.Where(x => condition)
```

- `source` — Any `IEnumerable<T>` or `IQueryable<T>`.
- `predicate` — `Func<T, bool>` (or expression tree).
- Returns a filtered sequence (deferred).

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5, 6 };
        var evens = numbers.Where(n => n % 2 == 0);
        Console.WriteLine(string.Join(", ", evens));   // 2, 4, 6
    }
}
```

**Expected Output:** `2, 4, 6`

### Real-World Cases

- Filtering active records in a database (`db.Users.Where(u => u.IsActive)`).
- Removing invalid or unwanted items from a collection.
- Composing conditions in reporting queries.

### References

- Enumerable.Where — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.where
- Queryable.Where — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable.where

---

## 2. `Select()`

### Definitions

- **Core:** Projects each element into a new form.
- **Technical:** `IEnumerable<TResult> Select<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector)`.
- **Beginner:** “Transform each item into something new.”

### Purposes

- To shape output into a different type.
- To project a subset of properties (DTOs, anonymous types).
- To perform value transformations (arithmetic, formatting).
- To translate to SQL `SELECT` for `IQueryable`.

### Syntax Rules

```
source.Select(x => transform(x))
```

- Returns a sequence of transformed elements (deferred).

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3 };
        var squares = numbers.Select(n => n * n);
        Console.WriteLine(string.Join(", ", squares));   // 1, 4, 9
    }
}
```

**Expected Output:** `1, 4, 9`

### Real-World Cases

- DTO projection: `customers.Select(c => new CustomerDto { Id = c.Id, Name = c.Name })`.
- Creating anonymous types in LINQ queries.
- Mapping domain objects to view models.

### References

- Enumerable.Select — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select

---

## 3. `SelectMany()`

### Definitions

- **Core:** Projects each element into a sequence and flattens the results into a single sequence.
- **Technical:** `IEnumerable<TResult> SelectMany<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, IEnumerable<TResult>> selector)`.
- **Beginner:** “Flatten nested collections into one big list.”

### Purposes

- To flatten nested collections.
- To translate `from a in xs from b in a.Ys` to method syntax.
- To enable implicit joins and Cartesian products.

### Syntax Rules

```
source.SelectMany(x => x.InnerCollection)
source.SelectMany(x => x.Collection, (x, y) => project(x, y))
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var customers = new[]
        {
            new { Name = "Alice", Orders = new[] { "A1", "A2" } },
            new { Name = "Bob",   Orders = new[] { "B1" } }
        };

        var allOrders = customers.SelectMany(c => c.Orders, (c, o) => new { c.Name, Order = o });
        foreach (var row in allOrders)
            Console.WriteLine($"{row.Name}: {row.Order}");
    }
}
```

**Expected Output:**
```
Alice: A1
Alice: A2
Bob: B1
```

### Real-World Cases

- Flattening one-to-many relationships.
- Extracting tags from posts.
- Combining nested collections.

### References

- Enumerable.SelectMany — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.selectmany

---

## 4. `OrderBy()`

### Definitions

- **Core:** Sorts a sequence in ascending order by a key selector.
- **Technical:** `IOrderedEnumerable<T> OrderBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector)`.
- **Beginner:** “Sort ascending by this key.”

### Purposes

- To sort data ascending.
- To enable multi-key sorting with `ThenBy`.
- To translate to SQL `ORDER BY … ASC` for `IQueryable`.

### Syntax Rules

```
source.OrderBy(x => key)
source.OrderBy(x => key, comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string[] words = { "banana", "apple", "cherry" };
        var sorted = words.OrderBy(w => w);
        Console.WriteLine(string.Join(", ", sorted));   // apple, banana, cherry
    }
}
```

**Expected Output:** `apple, banana, cherry`

### Real-World Cases

- Alphabetical sorting of names.
- Sorting by numeric values (age, price, date).
- Leaderboards.

### References

- Enumerable.OrderBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderby

---

## 5. `OrderByDescending()`

### Definitions

- **Core:** Sorts a sequence in **descending** order by a key selector.
- **Technical:** `IOrderedEnumerable<T> OrderByDescending<TSource, TKey>(…)`.
- **Beginner:** “Sort descending by this key.”

### Purposes

- To sort data descending.
- To enable multi-key sorting with `ThenByDescending`.
- To translate to SQL `ORDER BY … DESC`.

### Syntax Rules

```
source.OrderByDescending(x => key)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 3, 1, 4, 1, 5 };
        var desc = numbers.OrderByDescending(n => n);
        Console.WriteLine(string.Join(", ", desc));   // 5, 4, 3, 1, 1
    }
}
```

**Expected Output:** `5, 4, 3, 1, 1`

### Real-World Cases

- Sorting scores or prices highest-first.
- Recent-first timelines.
- Ranking.

### References

- Enumerable.OrderByDescending — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderbydescending

---

## 6. `ThenBy()`

### Definitions

- **Core:** Adds a **secondary ascending** sort key after an `OrderBy`/`OrderByDescending`.
- **Technical:** `IOrderedEnumerable<T> ThenBy<T>(this IOrderedEnumerable<T> source, Func<T, TKey> keySelector)`.
- **Beginner:** “If two items are equal by the first key, sort them by this second key.”

### Purposes

- To implement multi-key sorting.
- To refine ordering within primary groups.
- To mirror SQL `ORDER BY a, b`.

### Syntax Rules

```
source.OrderBy(x => key1).ThenBy(x => key2)
```

### Annotated Example

```csharp
using System;
using System.Linq;

record Person(string Name, int Age);

class Program
{
    static void Main()
    {
        var people = new[]
        {
            new Person("Alice", 30), new Person("Bob", 25),
            new Person("Carol", 30), new Person("Dave", 25)
        };

        var sorted = people.OrderBy(p => p.Age).ThenBy(p => p.Name);
        foreach (var p in sorted)
            Console.WriteLine($"{p.Name} ({p.Age})");
    }
}
```

**Expected Output:**
```
Bob (25)
Dave (25)
Alice (30)
Carol (30)
```

### Real-World Cases

- Sort by category, then by name.
- Multi-column sorting in data grids.
- Entity Framework `OrderBy(...).ThenBy(...)`.

### References

- Enumerable.ThenBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenby

---

## 7. `ThenByDescending()`

### Definitions

- **Core:** Adds a **secondary descending** sort key.
- **Technical:** `IOrderedEnumerable<T> ThenByDescending<T>(…)`.
- **Beginner:** “If two items tie, sort them descending by this second key.”

### Purposes

- To add descending sub-ordering.
- To mirror SQL `ORDER BY a ASC, b DESC`.

### Syntax Rules

```
source.OrderBy(x => key1).ThenByDescending(x => key2)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new[] { ("A", 1), ("B", 3), ("C", 2) };
        var sorted = data.OrderBy(t => t.Item2).ThenByDescending(t => t.Item1);
        foreach (var t in sorted) Console.WriteLine($"{t.Item1}: {t.Item2}");
    }
}
```

**Expected Output:**
```
A: 1
C: 2
B: 3
```

### Real-World Cases

- Sort by date, then by priority descending.
- Multi-key sorting in reports.

### References

- Enumerable.ThenByDescending — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenbydescending

---

## 8. `GroupBy()`

### Definitions

- **Core:** Groups elements by a key selector, producing `IGrouping<TKey, TElement>`.
- **Technical:** `IEnumerable<IGrouping<TKey, TElement>> GroupBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector)`.
- **Beginner:** “Bucket the items by key.”

### Purposes

- To categorize data by a key.
- To enable per-group aggregation.
- To translate to SQL `GROUP BY`.

### Syntax Rules

```
source.GroupBy(x => key)
source.GroupBy(x => key, x => element)
source.GroupBy(x => key, x => element, (k, g) => result)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string[] words = { "apple", "banana", "cherry", "date" };
        var groups = words.GroupBy(w => w.Length);
        foreach (var g in groups)
            Console.WriteLine($"{g.Key}: {string.Join(", ", g)}");
    }
}
```

**Expected Output:**
```
5: apple
6: banana, cherry
4: date
```

### Real-World Cases

- Sales grouped by category.
- Users grouped by country.
- Log entries grouped by severity.

### References

- Enumerable.GroupBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupby

---

## 9. `Join()`

### Definitions

- **Core:** Correlates two sequences based on matching keys (inner join).
- **Technical:** `IEnumerable<TResult> Join<TOuter, TInner, TKey, TResult>(this IEnumerable<TOuter> outer, IEnumerable<TInner> inner, Func<TOuter, TKey> outerKeySelector, Func<TInner, TKey> innerKeySelector, Func<TOuter, TInner, TResult> resultSelector)`.
- **Beginner:** “Match items from two lists by key.”

### Purposes

- To perform an inner join.
- To combine related data sources.
- To translate to SQL `JOIN`.

### Syntax Rules

```
outer.Join(inner, o => o.Key, i => i.Key, (o, i) => result)
```

### Annotated Example

```csharp
using System;
using System.Linq;

record Customer(int Id, string Name);
record Order(int Id, int CustomerId, decimal Total);

class Program
{
    static void Main()
    {
        var customers = new[] { new Customer(1, "Alice"), new Customer(2, "Bob") };
        var orders = new[] { new Order(101, 1, 100m), new Order(102, 2, 200m) };

        var rows = customers.Join(orders,
            c => c.Id, o => o.CustomerId,
            (c, o) => new { c.Name, o.Id, o.Total });

        foreach (var r in rows) Console.WriteLine($"{r.Name}: {r.Id} = {r.Total}");
    }
}
```

**Expected Output:**
```
Alice: 101 = 100
Bob: 102 = 200
```

### Real-World Cases

- Customers joined with orders.
- Products joined with categories.
- EF Core query joins.

### References

- Enumerable.Join — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.join

---

## 10. `GroupJoin()`

### Definitions

- **Core:** Joins two sequences and groups the matching inner elements per outer element.
- **Technical:** `IEnumerable<TResult> GroupJoin<TOuter, TInner, TKey, TResult>(…)`.
- **Beginner:** “Match items from two lists, but keep unmatched inners as empty collections.”

### Purposes

- To perform a **left outer join** (with `DefaultIfEmpty`).
- To group inner matches per outer element.
- To model hierarchical relationships.

### Syntax Rules

```
outer.GroupJoin(inner, o => o.Key, i => i.Key, (o, group) => result)
```

### Annotated Example

```csharp
using System;
using System.Linq;

record Customer(int Id, string Name);
record Order(int Id, int CustomerId, decimal Total);

class Program
{
    static void Main()
    {
        var customers = new[] { new Customer(1, "Alice"), new Customer(2, "Bob"), new Customer(3, "Carol") };
        var orders = new[] { new Order(101, 1, 100m), new Order(102, 1, 200m) };

        var summary = customers.GroupJoin(orders,
            c => c.Id, o => o.CustomerId,
            (c, os) => new { c.Name, Count = os.Count(), Total = os.Sum(o => o.Total) });

        foreach (var s in summary) Console.WriteLine($"{s.Name}: {s.Count} orders, {s.Total}");
    }
}
```

**Expected Output:**
```
Alice: 2 orders, 300
Bob: 0 orders, 0
Carol: 0 orders, 0
```

### Real-World Cases

- Left outer joins.
- Customer order summaries.
- Parent-child aggregation.

### References

- Enumerable.GroupJoin — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupjoin

---

## 11. `Take()`

### Definitions

- **Core:** Returns the first `n` elements of a sequence.
- **Technical:** `IEnumerable<T> Take<T>(this IEnumerable<T> source, int count)`.
- **Beginner:** “Give me the first N items.”

### Purposes

- To limit results (pagination).
- To sample the first items.
- To translate to SQL `TOP` / `LIMIT`.

### Syntax Rules

```
source.Take(n)
source.Take(Range)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        Console.WriteLine(string.Join(", ", numbers.Take(3)));   // 1, 2, 3
    }
}
```

**Expected Output:** `1, 2, 3`

### Real-World Cases

- Pagination (`Take(pageSize)`).
- Limiting preview results.
- Sampling.

### References

- Enumerable.Take — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.take

---

## 12. `Skip()`

### Definitions

- **Core:** Skips the first `n` elements, returning the rest.
- **Technical:** `IEnumerable<T> Skip<T>(this IEnumerable<T> source, int count)`.
- **Beginner:** “Skip the first N items.”

### Purposes

- To implement pagination (`Skip(pageSize * page)`).
- To ignore headers or early data.
- To translate to SQL `OFFSET`.

### Syntax Rules

```
source.Skip(n)
source.Skip(Range)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        Console.WriteLine(string.Join(", ", numbers.Skip(2)));   // 3, 4, 5
    }
}
```

**Expected Output:** `3, 4, 5`

### Real-World Cases

- Pagination (`Skip((page - 1) * pageSize).Take(pageSize)`).
- Removing header rows.

### References

- Enumerable.Skip — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skip

---

## 13. `TakeWhile()`

### Definitions

- **Core:** Returns elements from the start of the sequence **while** the predicate is true, stopping at the first false.
- **Technical:** `IEnumerable<T> TakeWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate)`.
- **Beginner:** “Keep taking items until one fails the test.”

### Purposes

- To take a contiguous prefix satisfying a condition.
- To process a stream until a boundary condition.

### Syntax Rules

```
source.TakeWhile(x => condition)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 10, 4, 5 };
        Console.WriteLine(string.Join(", ", nums.TakeWhile(n => n < 5)));   // 1, 2, 3
    }
}
```

**Expected Output:** `1, 2, 3`

### Real-World Cases

- Reading a file line-by-line until a marker.
- Processing a sorted stream until a threshold.

### References

- Enumerable.TakeWhile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.takewhile

---

## 14. `SkipWhile()`

### Definitions

- **Core:** Skips elements from the start **while** the predicate is true, yielding the remainder.
- **Technical:** `IEnumerable<T> SkipWhile<T>(this IEnumerable<T> source, Func<T, bool> predicate)`.
- **Beginner:** “Skip items until the test fails, then keep the rest.”

### Purposes

- To skip a contiguous prefix satisfying a condition.
- To start processing once a boundary is crossed.

### Syntax Rules

```
source.SkipWhile(x => condition)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 10, 4, 5 };
        Console.WriteLine(string.Join(", ", nums.SkipWhile(n => n < 5)));   // 10, 4, 5
    }
}
```

**Expected Output:** `10, 4, 5`

### Real-World Cases

- Skipping file headers.
- Ignoring leading whitespace or comments.

### References

- Enumerable.SkipWhile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skipwhile

---

## 15. `Distinct()`

### Definitions

- **Core:** Removes duplicate elements from a sequence.
- **Technical:** `IEnumerable<T> Distinct<T>(this IEnumerable<T> source)`; optional `IEqualityComparer<T>`.
- **Beginner:** “Give me unique items only.”

### Purposes

- To eliminate duplicates.
- To implement set semantics.
- To translate to SQL `SELECT DISTINCT`.

### Syntax Rules

```
source.Distinct()
source.Distinct(comparer)
source.DistinctBy(x => key)   // .NET 6+
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 2, 3, 1, 4 };
        Console.WriteLine(string.Join(", ", nums.Distinct()));   // 1, 2, 3, 4
    }
}
```

**Expected Output:** `1, 2, 3, 4`

### Real-World Cases

- Unique tags.
- Removing duplicate results.
- SQL `DISTINCT`.

### References

- Enumerable.Distinct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinct
- Enumerable.DistinctBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinctby

---

## 16. `Union()`

### Definitions

- **Core:** Returns the **set union** of two sequences (distinct elements in either).
- **Technical:** `IEnumerable<T> Union<T>(this IEnumerable<T> first, IEnumerable<T> second)`.
- **Beginner:** “All unique items from both lists.”

### Purposes

- To combine two sequences uniquely.
- To implement set-union semantics.
- To translate to SQL `UNION`.

### Syntax Rules

```
first.Union(second)
first.Union(second, comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] a = { 1, 2, 3 }, b = { 3, 4, 5 };
        Console.WriteLine(string.Join(", ", a.Union(b)));   // 1, 2, 3, 4, 5
    }
}
```

**Expected Output:** `1, 2, 3, 4, 5`

### Real-World Cases

- Combining two collections into a distinct set.
- Union of subscribed tags.

### References

- Enumerable.Union — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.union

---

## 17. `Intersect()`

### Definitions

- **Core:** Returns the **set intersection** of two sequences (elements in both).
- **Technical:** `IEnumerable<T> Intersect<T>(this IEnumerable<T> first, IEnumerable<T> second)`.
- **Beginner:** “Only items that appear in both lists.”

### Purposes

- To find common elements.
- To implement set-intersection semantics.

### Syntax Rules

```
first.Intersect(second)
first.Intersect(second, comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] a = { 1, 2, 3 }, b = { 3, 4, 5 };
        Console.WriteLine(string.Join(", ", a.Intersect(b)));   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- Common tags between two items.
- Intersection of permission sets.

### References

- Enumerable.Intersect — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.intersect

---

## 18. `Except()`

### Definitions

- **Core:** Returns the **set difference** — elements in the first sequence but not the second.
- **Technical:** `IEnumerable<T> Except<T>(this IEnumerable<T> first, IEnumerable<T> second)`.
- **Beginner:** “Items in the first list that are not in the second.”

### Purposes

- To find elements unique to the first sequence.
- To implement set-difference semantics.

### Syntax Rules

```
first.Except(second)
first.Except(second, comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] a = { 1, 2, 3 }, b = { 3, 4, 5 };
        Console.WriteLine(string.Join(", ", a.Except(b)));   // 1, 2
    }
}
```

**Expected Output:** `1, 2`

### Real-World Cases

- Items to remove from a list.
- Subscription differences.

### References

- Enumerable.Except — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.except

---

## 19. `Concat()`

### Definitions

- **Core:** Concatenates two sequences into one, **preserving duplicates**.
- **Technical:** `IEnumerable<T> Concat<T>(this IEnumerable<T> first, IEnumerable<T> second)`.
- **Beginner:** “Join two lists end-to-end, keeping everything.”

### Purposes

- To append one sequence to another.
- To preserve duplicates (unlike `Union`).
- To translate to SQL `UNION ALL`.

### Syntax Rules

```
first.Concat(second)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] a = { 1, 2 }, b = { 3, 4 };
        Console.WriteLine(string.Join(", ", a.Concat(b)));   // 1, 2, 3, 4
    }
}
```

**Expected Output:** `1, 2, 3, 4`

### Real-World Cases

- Appending collections together.
- Combining results from multiple sources.

### References

- Enumerable.Concat — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.concat

---

## 20. `Reverse()`

### Definitions

- **Core:** Returns the sequence in reverse order.
- **Technical:** `IEnumerable<T> Reverse<T>(this IEnumerable<T> source)`.
- **Beginner:** “Flip the list end-to-end.”

### Purposes

- To reverse the order of a sequence.
- To process items in reverse.

### Syntax Rules

```
source.Reverse()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3 };
        Console.WriteLine(string.Join(", ", nums.Reverse()));   // 3, 2, 1
    }
}
```

**Expected Output:** `3, 2, 1`

### Real-World Cases

- Reverse chronological order.
- Undo/redo stacks.

### References

- Enumerable.Reverse — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.reverse

---

## 21. `First()`

### Definitions

- **Core:** Returns the first element; throws if empty.
- **Technical:** `T First<T>(this IEnumerable<T> source)`; overload with predicate.
- **Beginner:** “Give me the first item.”

### Purposes

- To retrieve the first element.
- To assert non-empty sequences.

### Syntax Rules

```
source.First()
source.First(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 10, 20, 30 };
        Console.WriteLine(nums.First());   // 10
    }
}
```

**Expected Output:** `10`

### Real-World Cases

- Picking the newest record.
- Getting the first matching item.

### References

- Enumerable.First — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.first

---

## 22. `FirstOrDefault()`

### Definitions

- **Core:** Returns the first element, or `default(T)` if empty.
- **Technical:** `T? FirstOrDefault<T>(this IEnumerable<T> source)`.
- **Beginner:** “Give me the first item, or nothing if empty.”

### Purposes

- To safely retrieve the first element.
- To avoid exceptions on empty sequences.

### Syntax Rules

```
source.FirstOrDefault()
source.FirstOrDefault(predicate)
source.FirstOrDefault(defaultValue)   // .NET 6+
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] empty = { };
        Console.WriteLine(empty.FirstOrDefault());   // 0
        Console.WriteLine(empty.FirstOrDefault(-1)); // -1
    }
}
```

**Expected Output:** `0` and `-1`

### Real-World Cases

- Safe queries that may return nothing.
- Default fallbacks.

### References

- Enumerable.FirstOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.firstordefault

---

## 23. `Last()`

### Definitions

- **Core:** Returns the last element; throws if empty.
- **Technical:** `T Last<T>(this IEnumerable<T> source)`.

### Purposes

- To retrieve the last element.
- To assert non-empty sequences.

### Syntax Rules

```
source.Last()
source.Last(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3 };
        Console.WriteLine(nums.Last());   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- The most recent record.
- Trailing items.

### References

- Enumerable.Last — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.last

---

## 24. `LastOrDefault()`

### Definitions

- **Core:** Returns the last element, or `default(T)` if empty.
- **Technical:** `T? LastOrDefault<T>(this IEnumerable<T> source)`.

### Purposes

- To safely retrieve the last element.

### Syntax Rules

```
source.LastOrDefault()
source.LastOrDefault(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] empty = { };
        Console.WriteLine(empty.LastOrDefault());   // 0
    }
}
```

**Expected Output:** `0`

### Real-World Cases

- Safe retrieval of latest items.

### References

- Enumerable.LastOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.lastordefault

---

## 25. `Single()`

### Definitions

- **Core:** Returns the **only** element; throws if empty or more than one.
- **Technical:** `T Single<T>(this IEnumerable<T> source)`.

### Purposes

- To assert exactly one element exists.
- To retrieve a uniquely identified item.

### Syntax Rules

```
source.Single()
source.Single(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] one = { 42 };
        Console.WriteLine(one.Single());   // 42
        // new int[] { 1, 2 }.Single();   // InvalidOperationException
    }
}
```

**Expected Output:** `42`

### Real-World Cases

- Fetching a unique record by primary key.

### References

- Enumerable.Single — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single

---

## 26. `SingleOrDefault()`

### Definitions

- **Core:** Returns the only element, or `default(T)` if empty; throws if more than one.
- **Technical:** `T? SingleOrDefault<T>(this IEnumerable<T> source)`.

### Purposes

- To safely retrieve an exactly-zero-or-one element.

### Syntax Rules

```
source.SingleOrDefault()
source.SingleOrDefault(predicate)
source.SingleOrDefault(defaultValue)   // .NET 6+
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] empty = { };
        Console.WriteLine(empty.SingleOrDefault());   // 0
    }
}
```

**Expected Output:** `0`

### Real-World Cases

- Optional unique lookups.

### References

- Enumerable.SingleOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.singleordefault

---

## 27. `ElementAt()`

### Definitions

- **Core:** Returns the element at a specified index; throws if out of range.
- **Technical:** `T ElementAt<T>(this IEnumerable<T> source, int index)`.

### Purposes

- To access by index in LINQ.

### Syntax Rules

```
source.ElementAt(index)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 10, 20, 30 };
        Console.WriteLine(nums.ElementAt(1));   // 20
    }
}
```

**Expected Output:** `20`

### Real-World Cases

- Indexing a filtered or projected sequence.

### References

- Enumerable.ElementAt — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.elementat

---

## 28. `ElementAtOrDefault()`

### Definitions

- **Core:** Returns the element at index, or `default(T)` if out of range.
- **Technical:** `T? ElementAtOrDefault<T>(this IEnumerable<T> source, int index)`.

### Purposes

- Safe indexed access.

### Syntax Rules

```
source.ElementAtOrDefault(index)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 10, 20, 30 };
        Console.WriteLine(nums.ElementAtOrDefault(5));   // 0
    }
}
```

**Expected Output:** `0`

### Real-World Cases

- Safe index access on filtered results.

### References

- Enumerable.ElementAtOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.elementatordefault

---

## 29. `Any()`

### Definitions

- **Core:** Returns `true` if the sequence contains any elements (optionally matching a predicate).
- **Technical:** `bool Any<T>(this IEnumerable<T> source)` or `bool Any<T>(this IEnumerable<T> source, Func<T, bool> predicate)`.

### Purposes

- To check for existence.
- To short-circuit (more efficient than `Count() > 0`).

### Syntax Rules

```
source.Any()
source.Any(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3 };
        Console.WriteLine(nums.Any(n => n > 2));   // True
    }
}
```

**Expected Output:** `True`

### Real-World Cases

- “Does this user exist?”
- Efficient existence checks in EF Core.

### References

- Enumerable.Any — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.any

---

## 30. `All()`

### Definitions

- **Core:** Returns `true` if **all** elements satisfy a predicate.
- **Technical:** `bool All<T>(this IEnumerable<T> source, Func<T, bool> predicate)`.

### Purposes

- To validate all elements.

### Syntax Rules

```
source.All(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 2, 4, 6 };
        Console.WriteLine(nums.All(n => n % 2 == 0));   // True
    }
}
```

**Expected Output:** `True`

### Real-World Cases

- Validating all items.

### References

- Enumerable.All — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.all

---

## 31. `Contains()`

### Definitions

- **Core:** Returns `true` if a specific element exists in the sequence.
- **Technical:** `bool Contains<T>(this IEnumerable<T> source, T value)`; optional `IEqualityComparer<T>`.

### Purposes

- To check for membership.

### Syntax Rules

```
source.Contains(value)
source.Contains(value, comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3 };
        Console.WriteLine(nums.Contains(2));   // True
    }
}
```

**Expected Output:** `True`

### Real-World Cases

- Membership checks.

### References

- Enumerable.Contains — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.contains

---

## 32. `Count()`

### Definitions

- **Core:** Returns the number of elements (optionally matching a predicate).
- **Technical:** `int Count<T>(this IEnumerable<T> source)`.

### Purposes

- To count elements.

### Syntax Rules

```
source.Count()
source.Count(predicate)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 4 };
        Console.WriteLine(nums.Count(n => n % 2 == 0));   // 2
    }
}
```

**Expected Output:** `2`

### Real-World Cases

- Counting records.
- Statistics.

### References

- Enumerable.Count — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.count

---

## 33. `LongCount()`

### Definitions

- **Core:** Returns the count as `long` (for very large sequences).
- **Technical:** `long LongCount<T>(this IEnumerable<T> source)`.

### Purposes

- To count sequences that may exceed `int.MaxValue`.

### Syntax Rules

```
source.LongCount()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        long count = Enumerable.Range(1, 100).LongCount();
        Console.WriteLine(count);   // 100
    }
}
```

**Expected Output:** `100`

### Real-World Cases

- Very large dataset counting.

### References

- Enumerable.LongCount — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.longcount

---

## 34. `Sum()`

### Definitions

- **Core:** Computes the sum of numeric elements (or a projection).
- **Technical:** `int/… Sum<T>(this IEnumerable<T> source, Func<T, numeric> selector)`.

### Purposes

- To aggregate totals.

### Syntax Rules

```
source.Sum()
source.Sum(x => selector)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 4 };
        Console.WriteLine(nums.Sum());   // 10
    }
}
```

**Expected Output:** `10`

### Real-World Cases

- Financial totals.
- Aggregations.

### References

- Enumerable.Sum — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sum

---

## 35. `Min()`

### Definitions

- **Core:** Returns the minimum element (or selected value).
- **Technical:** `T Min<T>(this IEnumerable<T> source)`.

### Purposes

- To find the minimum.

### Syntax Rules

```
source.Min()
source.Min(x => selector)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 5, 2, 9 };
        Console.WriteLine(nums.Min());   // 2
    }
}
```

**Expected Output:** `2`

### Real-World Cases

- Lowest price, youngest age, earliest date.

### References

- Enumerable.Min — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.min

---

## 36. `Max()`

### Definitions

- **Core:** Returns the maximum element (or selected value).
- **Technical:** `T Max<T>(this IEnumerable<T> source)`.

### Purposes

- To find the maximum.

### Syntax Rules

```
source.Max()
source.Max(x => selector)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 5, 2, 9 };
        Console.WriteLine(nums.Max());   // 9
    }
}
```

**Expected Output:** `9`

### Real-World Cases

- Highest score, latest date.

### References

- Enumerable.Max — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.max

---

## 37. `Average()`

### Definitions

- **Core:** Computes the average of numeric elements (or a projection).
- **Technical:** `double Average<T>(this IEnumerable<T> source)`.

### Purposes

- To compute averages.

### Syntax Rules

```
source.Average()
source.Average(x => selector)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 4 };
        Console.WriteLine(nums.Average());   // 2.5
    }
}
```

**Expected Output:** `2.5`

### Real-World Cases

- Mean price, mean score.

### References

- Enumerable.Average — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.average

---

## 38. `Aggregate()`

### Definitions

- **Core:** Applies an accumulator function over a sequence, optionally with a seed.
- **Technical:** `TResult Aggregate<T, TResult>(this IEnumerable<T> source, TResult seed, Func<TResult, T, TResult> func)`.

### Purposes

- To implement custom fold operations.
- To concatenate, multiply, or combine elements.

### Syntax Rules

```
source.Aggregate(seed, (acc, x) => combine(acc, x))
source.Aggregate((a, b) => combine(a, b))
source.Aggregate(seed, func, resultSelector)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 4 };
        int product = nums.Aggregate(1, (acc, n) => acc * n);
        Console.WriteLine(product);   // 24
    }
}
```

**Expected Output:** `24`

### Real-World Cases

- Multiplying values.
- Custom aggregations.
- String concatenation.

### References

- Enumerable.Aggregate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate

---

## 39. `ToList()`

### Definitions

- **Core:** Materializes the sequence into a `List<T>`.
- **Technical:** `List<T> ToList<T>(this IEnumerable<T> source)`.

### Purposes

- To force immediate execution.
- To obtain a mutable list.
- To avoid multiple enumeration.

### Syntax Rules

```
source.ToList()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var list = Enumerable.Range(1, 3).ToList();
        Console.WriteLine(list.Count);   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- Materializing query results.
- Returning `List<T>` from APIs.

### References

- Enumerable.ToList — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tolist

---

## 40. `ToArray()`

### Definitions

- **Core:** Materializes the sequence into an array.
- **Technical:** `T[] ToArray<T>(this IEnumerable<T> source)`.

### Purposes

- To force immediate execution and get a fixed-size array.

### Syntax Rules

```
source.ToArray()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] arr = Enumerable.Range(1, 3).ToArray();
        Console.WriteLine(arr.Length);   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- Passing query results to array-consuming APIs.

### References

- Enumerable.ToArray — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.toarray

---

## 41. `ToDictionary()`

### Definitions

- **Core:** Materializes the sequence into a `Dictionary<TKey, TValue>`.
- **Technical:** `Dictionary<TKey, TValue> ToDictionary<T, TKey, TValue>(this IEnumerable<T> source, Func<T, TKey> keySelector, Func<T, TValue> elementSelector)`.

### Purposes

- To build a lookup dictionary from a sequence.
- To index items by a unique key.

### Syntax Rules

```
source.ToDictionary(x => key)
source.ToDictionary(x => key, x => value)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var dict = new[] { "Alice", "Bob" }.ToDictionary(n => n.Length, n => n);
        Console.WriteLine(dict[5]);   // Alice
    }
}
```

**Expected Output:** `Alice`

### Real-World Cases

- Building lookups by ID.
- Grouping by unique key.

### References

- Enumerable.ToDictionary — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.todictionary

---

## 42. `ToHashSet()`

### Definitions

- **Core:** Materializes the sequence into a `HashSet<T>`.
- **Technical:** `HashSet<T> ToHashSet<T>(this IEnumerable<T> source)`.

### Purposes

- To obtain a set (removes duplicates).
- To support fast O(1) membership.

### Syntax Rules

```
source.ToHashSet()
source.ToHashSet(comparer)
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var set = new[] { 1, 2, 2, 3 }.ToHashSet();
        Console.WriteLine(set.Count);   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- Deduplicating and enabling fast lookup.

### References

- Enumerable.ToHashSet — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tohashset

---

## 43. `AsEnumerable()`

### Definitions

- **Core:** Returns the source typed as `IEnumerable<T>`, **forcing subsequent LINQ operators to execute in memory**.
- **Technical:** `IEnumerable<T> AsEnumerable<T>(this IEnumerable<T> source)`.

### Purposes

- To force client-side evaluation of an `IQueryable`.
- To switch from a provider to LINQ to Objects.

### Syntax Rules

```
queryable.AsEnumerable()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new[] { 1, 2, 3 }.AsQueryable().AsEnumerable();
        Console.WriteLine(data.Where(n => n > 1).Count());   // 2
    }
}
```

**Expected Output:** `2`

### Real-World Cases

- Forcing client evaluation after server-side filtering.
- Unit-testing `IQueryable` code with in-memory data.

### References

- Enumerable.AsEnumerable — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.asenumerable

---

## 44. `AsQueryable()`

### Definitions

- **Core:** Returns the source typed as `IQueryable<T>`.
- **Technical:** `IQueryable<T> AsQueryable<T>(this IEnumerable<T> source)`.

### Purposes

- To simulate `IQueryable` behavior over in-memory data.
- To enable testing of `IQueryable`-based APIs.

### Syntax Rules

```
source.AsQueryable()
```

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var q = new[] { 1, 2, 3 }.AsQueryable().Where(n => n > 1);
        Console.WriteLine(q.Count());   // 2
    }
}
```

**Expected Output:** `2`

### Real-World Cases

- Testing repository methods that accept `IQueryable<T>`.
- Prototyping.

### References

- Queryable.AsQueryable — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable.asqueryable

---

## 45. `Cast()`

### Definitions

- **Core:** Casts each element of a non-generic sequence to a specified type.
- **Technical:** `IEnumerable<TResult> Cast<TResult>(this IEnumerable source)`.

### Purposes

- To convert a non-generic sequence to a typed one.
- To enable LINQ over `ArrayList`, `DataTable.Rows`, etc.

### Syntax Rules

```
source.Cast<T>()
```

### Annotated Example

```csharp
using System;
using System.Collections;
using System.Linq;

class Program
{
    static void Main()
    {
        var list = new ArrayList { 1, 2, 3 };
        var evens = list.Cast<int>().Where(n => n % 2 == 0);
        Console.WriteLine(string.Join(", ", evens));   // 2
    }
}
```

**Expected Output:** `2`

### Real-World Cases

- Working with legacy non-generic collections.
- Querying `DataTable.Rows`.

### References

- Enumerable.Cast — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.cast

---

## 46. `OfType()`

### Definitions

- **Core:** Filters a non-generic sequence to elements of the specified type.
- **Technical:** `IEnumerable<TResult> OfType<TResult>(this IEnumerable source)`.

### Purposes

- To extract elements of a specific type from a mixed sequence.
- To safely query heterogeneous collections.

### Syntax Rules

```
source.OfType<T>()
```

### Annotated Example

```csharp
using System;
using System.Collections;
using System.Linq;

class Program
{
    static void Main()
    {
        var mixed = new ArrayList { 1, "two", 3, "four" };
        var numbers = mixed.OfType<int>();
        Console.WriteLine(string.Join(", ", numbers));   // 1, 3
    }
}
```

**Expected Output:** `1, 3`

### Real-World Cases

- Filtering heterogeneous object collections by type.
- Processing UI control hierarchies.

### References

- Enumerable.OfType — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.oftype

---

## 47. Method Syntax Best Practices

### Definitions

- **Core:** Guidelines for writing readable, correct, and efficient LINQ method chains.

### Guidelines

```
// 1. Prefer method syntax for simple filters and projections
var adults = people.Where(p => p.Age >= 18).Select(p => p.Name).ToList();

// 2. Filter early to reduce work downstream
var result = source.Where(x => x.IsActive).OrderBy(x => x.Name).Select(x => x.Id);

// 3. Materialize once when reusing results
var list = query.ToList();
var count = list.Count;

// 4. Use Any() instead of Count() > 0
if (items.Any()) { ... }   // not items.Count() > 0

// 5. Prefer Single() when exactly one match is expected
var item = items.Single(i => i.Id == id);

// 6. Prefer FirstOrDefault() when absence is acceptable
var user = users.FirstOrDefault(u => u.Name == "Alice");

// 7. Avoid using Where(predicate).Any() — use Any(predicate) instead
if (items.Any(i => i.IsActive)) { ... }

// 8. Avoid multiple enumeration of deferred queries
var query = source.Where(x => x.IsActive);   // deferred
var count = query.Count();                     // enumeration 1
var first = query.FirstOrDefault();            // enumeration 2 (bad)
```

### Purposes

- To write clearer and faster queries.
- To avoid multiple enumeration.
- To choose appropriate operators.
- To keep `IQueryable` queries translatable.

### Best Practices

- **Do** prefer method syntax for simple operations.
- **Do** filter early, project late.
- **Do** materialize once if reusing.
- **Do** use `Any()` instead of `Count() > 0`.
- **Do** use `FirstOrDefault` / `SingleOrDefault` for optional lookups.
- **Do** use `Any(predicate)` instead of `Where(predicate).Any()`.
- **Do not** enumerate deferred queries multiple times.
- **Do not** mix `IQueryable` and client-side evaluation unnecessarily.
- **Do not** use `Cast`/`OfType` when typed collections are already available.

### References

- LINQ (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- CA1851: Possible multiple enumerations — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Enumerable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable
- Queryable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| All operators | Introduced with **LINQ in .NET 3.5 / C# 3.0**; stable. |
| `DistinctBy`, `Chunk`, `MaxBy`, `MinBy`, `Zip` 3-arg | Introduced in **.NET 6**; version-specific. |
| `TryGetNonEnumeratedCount` | Introduced in **.NET 6**; version-specific. |
| `Order` / `OrderDescending` | Introduced in **.NET 7**; version-specific. |
| `Aggregate` with seed | Available since .NET 3.5. |
| `Take(Range)` / `Skip(Range)` | Introduced in **.NET 6**; version-specific. |
| `FirstOrDefault(defaultValue)` | Introduced in **.NET 6**. |
| `AsQueryable` semantics | For in-memory only; `IQueryProvider` does not translate to external systems. |
| `Cast` on value types | Boxing may occur; use `OfType` for filtering. |

---

## References (Consolidated)

- LINQ (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- Enumerable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable
- Queryable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable
- Standard Query Operators Overview (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/standard-query-operators-overview
- Enumerable.Where — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.where
- Enumerable.Select — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select
- Enumerable.SelectMany — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.selectmany
- Enumerable.OrderBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderby
- Enumerable.OrderByDescending — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderbydescending
- Enumerable.ThenBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenby
- Enumerable.ThenByDescending — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenbydescending
- Enumerable.GroupBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupby
- Enumerable.Join — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.join
- Enumerable.GroupJoin — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupjoin
- Enumerable.Take — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.take
- Enumerable.Skip — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skip
- Enumerable.TakeWhile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.takewhile
- Enumerable.SkipWhile — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skipwhile
- Enumerable.Distinct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinct
- Enumerable.DistinctBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinctby
- Enumerable.Union — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.union
- Enumerable.Intersect — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.intersect
- Enumerable.Except — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.except
- Enumerable.Concat — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.concat
- Enumerable.Reverse — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.reverse
- Enumerable.First — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.first
- Enumerable.FirstOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.firstordefault
- Enumerable.Last — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.last
- Enumerable.LastOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.lastordefault
- Enumerable.Single — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single
- Enumerable.SingleOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.singleordefault
- Enumerable.ElementAt — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.elementat
- Enumerable.ElementAtOrDefault — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.elementatordefault
- Enumerable.Any — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.any
- Enumerable.All — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.all
- Enumerable.Contains — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.contains
- Enumerable.Count — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.count
- Enumerable.LongCount — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.longcount
- Enumerable.Sum — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sum
- Enumerable.Min — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.min
- Enumerable.Max — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.max
- Enumerable.Average — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.average
- Enumerable.Aggregate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate
- Enumerable.ToList — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tolist
- Enumerable.ToArray — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.toarray
- Enumerable.ToDictionary — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.todictionary
- Enumerable.ToHashSet — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tohashset
- Enumerable.AsEnumerable — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.asenumerable
- Queryable.AsQueryable — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable.asqueryable
- Enumerable.Cast — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.cast
- Enumerable.OfType — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.oftype
- CA1851: Possible multiple enumerations — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851

---