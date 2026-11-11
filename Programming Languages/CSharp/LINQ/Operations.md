# C# LINQ Operations — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**
LINQ operations are the **standard query operators** — the methods and query keywords that form the LINQ pattern. They provide a uniform, declarative way to filter, project, sort, group, join, aggregate, partition, compare, and transform sequences of data, whether those sequences live in memory (`IEnumerable<T>`) or are translated to a remote source (`IQueryable<T>`). Each operation is a reusable building block: complex queries are composed by chaining operators together.

**Technical Definition**
The standard query operators are the keywords and methods that form the LINQ pattern. The C# language defines LINQ query keywords for the most common query expressions; the compiler translates expressions using these keywords to the equivalent method calls. The two forms are synonymous. Other methods that are part of the `System.Linq` namespace don't have equivalent query keywords; in those cases, method syntax is required. The methods that make up each set are **extension members** defined in the `Enumerable` and `Queryable` classes, where the receiver type is either `IEnumerable<T>` or `IQueryable<T>`. The distinction between `IEnumerable<T>` and `IQueryable<T>` determines **how** the query executes at runtime: for `IEnumerable<T>`, the returned enumerable captures the arguments, and the query logic runs when the sequence is enumerated; for `IQueryable<T>`, the query is translated into an **expression tree** that a provider (such as Entity Framework Core) can translate into a native query. Expression trees have limitations on allowed C# syntax, and each `IQueryProvider` may impose additional restrictions.

**Beginner-Friendly Explanation**
LINQ operations are the **verbs of querying**. Think of them as a toolbox with different categories of tools: filters that keep only what you want, projectors that reshape data, sorters that order it, groupers that bucket it, joiners that match data from two sources, aggregators that compute summaries, and converters that materialize the results. Each tool takes a sequence and produces a new sequence or a single value. You chain them together like an assembly line: `source.Where(...).OrderBy(...).Select(...).ToList()`. The same tools work on arrays, lists, XML, and databases — that's the power of LINQ. This cheat sheet organises the toolbox into twelve categories and provides the key operators, syntax, and best practices for each.

---

### Prerequisites

- Basic understanding of C# **generics** (`List<T>`, `IEnumerable<T>`) and **lambda expressions**.
- Familiarity with **extension methods** and `using System.Linq;`.
- Awareness of **deferred execution** and the difference between `IEnumerable<T>` and `IQueryable<T>`.
- Basic understanding of **relational concepts** (joins, grouping) for the joining and grouping operations.

### Related Programming Areas

- **LINQ query syntax** — The SQL-like alternative to method syntax; both compile to the same operators.
- **LINQ method syntax** — The fluent form; required for operators without query-keyword equivalents.
- **Expression trees** — The underlying mechanism for `IQueryable<T>` translation.
- **PLINQ** — Parallel execution of LINQ operations.
- **Entity Framework Core** — A major consumer of LINQ operations for database querying.

### Core Concepts / Features

1. Filtering
2. Projection
3. Sorting
4. Grouping
5. Joining
6. Aggregation
7. Quantifiers
8. Partitioning
9. Set Operations
10. Conversion
11. Element Operations
12. Generation Operations
13. LINQ Operation Best Practices

---

## 1. Filtering

### Definitions

**Core Definition:** Filtering restricts a sequence to only the elements that satisfy a condition.

**Technical Definition:** Filtering operators return a new sequence containing only elements that match a specified predicate or type. The primary operators are `Where` (predicate-based) and `OfType` (type-based). These operations use **deferred execution**: the predicate is applied when the result is enumerated, not when the query is defined.

**Beginner-Friendly:** “Keep only the items I want.”

### Purposes

- To reduce a sequence to elements of interest.
- To translate to SQL `WHERE` clauses for `IQueryable`.
- To compose multiple conditions via chained `Where` calls or combined predicates.

### Syntax Rules

```
source.Where(x => condition)
source.OfType<T>()
```

- `Where` takes a `Func<T, bool>` (or `Expression<Func<T, bool>>` for `IQueryable`).
- `OfType` filters by runtime type; elements not of type `T` are excluded.
- Both return a filtered sequence (deferred).

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        object[] mixed = { 1, "two", 3, "four", 5.0 };
        var numbers = mixed.OfType<int>().Where(n => n > 1);
        Console.WriteLine(string.Join(", ", numbers));   // 3
    }
}
```

**Expected Output:** `3`

### Real-World Cases

- Filtering active users in a database (`db.Users.Where(u => u.IsActive)`).
- Extracting only numeric elements from a heterogeneous collection.
- Applying multiple search criteria in reporting queries.

### References

- Standard query operators overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/
- Enumerable.Where — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.where

---

## 2. Projection

### Definitions

**Core Definition:** Projection transforms each element of a sequence into a new form.

**Technical Definition:** Projection operators map each element to a new value or object using a selector function. The primary operators are `Select` (one-to-one mapping) and `SelectMany` (one-to-many flattening). For `IQueryable<T>`, projections translate to SQL `SELECT` lists.

**Beginner-Friendly:** “Reshape each item into what I need.”

### Purposes

- To shape data into DTOs, anonymous types, or computed values.
- To extract only the needed properties (reducing data transfer).
- To flatten nested collections with `SelectMany`.
- To translate to SQL `SELECT` projections.

### Syntax Rules

```
source.Select(x => transformation)
source.SelectMany(x => x.Collection)
source.SelectMany(x => x.Collection, (x, y) => result)
```

- `Select` returns a sequence of transformed elements.
- `SelectMany` flattens nested sequences into a single sequence.

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
            new { Name = "Bob", Orders = new[] { "B1" } }
        };

        // Select: project to a new shape
        var names = customers.Select(c => c.Name.ToUpper());
        Console.WriteLine(string.Join(", ", names));   // ALICE, BOB

        // SelectMany: flatten nested orders
        var allOrders = customers.SelectMany(c => c.Orders);
        Console.WriteLine(string.Join(", ", allOrders)); // A1, A2, B1
    }
}
```

**Expected Output:** `ALICE, BOB` and `A1, A2, B1`

### Real-World Cases

- DTO projection: `db.Customers.Select(c => new CustomerDto { Id = c.Id, Name = c.Name })`.
- Extracting tags from posts with `SelectMany`.
- Creating anonymous types for API responses.

### References

- Enumerable.Select — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select
- Enumerable.SelectMany — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.selectmany
- Projection operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/projection-operations

---

## 3. Sorting

### Definitions

**Core Definition:** Sorting orders the elements of a sequence by one or more keys.

**Technical Definition:** Sorting operators return a new sequence ordered by a key selector, either ascending or descending. `OrderBy`/`OrderByDescending` establish the primary sort; `ThenBy`/`ThenByDescending` add secondary sorts. `Reverse` reverses the sequence. Sorting translates to SQL `ORDER BY` for `IQueryable`.

**Beginner-Friendly:** “Put the items in order.”

### Purposes

- To present data in a meaningful order.
- To support multi-key sorting (`OrderBy(...).ThenBy(...)`).
- To translate to SQL `ORDER BY` clauses.
- To enable stable ordering for in-memory data.

### Syntax Rules

```
source.OrderBy(x => key)
source.OrderByDescending(x => key)
source.OrderBy(x => key1).ThenBy(x => key2)
source.OrderByDescending(x => key1).ThenByDescending(x => key2)
source.Reverse()
```

- `OrderBy`/`OrderByDescending` return `IOrderedEnumerable<T>`.
- `ThenBy`/`ThenByDescending` can only follow `OrderBy`/`OrderByDescending` or another `ThenBy`.
- `Reverse` reverses the entire sequence.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var people = new[]
        {
            ("Alice", 30), ("Bob", 25), ("Carol", 30), ("Dave", 25)
        };

        var sorted = people.OrderBy(p => p.Item2).ThenBy(p => p.Item1);
        foreach (var p in sorted)
            Console.WriteLine($"{p.Item1} ({p.Item2})");
        // Bob (25)
        // Dave (25)
        // Alice (30)
        // Carol (30)
    }
}
```

**Expected Output:** `Bob (25)`, `Dave (25)`, `Alice (30)`, `Carol (30)`

### Real-World Cases

- Alphabetical name lists.
- Leaderboards sorted by score descending, then by name.
- Multi-column sorting in data grids.

### References

- Enumerable.OrderBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderby
- Enumerable.ThenBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.thenby
- Sorting operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/sorting-operations

---

## 4. Grouping

### Definitions

**Core Definition:** Grouping partitions a sequence into groups of elements that share a common key.

**Technical Definition:** `GroupBy` returns a sequence of `IGrouping<TKey, TElement>` objects, each containing a `Key` property and the elements that match that key. Grouping enables per-group aggregation and hierarchical results. It translates to SQL `GROUP BY` for `IQueryable`.

**Beginner-Friendly:** “Bucket items that share something in common.”

### Purposes

- To categorise data by a key.
- To enable per-group aggregation (`Count`, `Sum`, `Max`, etc.).
- To translate to SQL `GROUP BY`.
- To support hierarchical or nested results.

### Syntax Rules

```
source.GroupBy(x => key)
source.GroupBy(x => key, x => element)
source.GroupBy(x => key, x => element, (key, group) => result)
```

- The result is `IEnumerable<IGrouping<TKey, TElement>>`.
- The `Key` property is the grouping key.
- Groups are enumerable sequences themselves.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string[] words = { "apple", "banana", "cherry", "date", "fig" };
        var groups = words.GroupBy(w => w.Length);

        foreach (var g in groups)
            Console.WriteLine($"{g.Key}: {string.Join(", ", g)}");
        // 5: apple
        // 6: banana, cherry
        // 4: date
        // 3: fig
    }
}
```

**Expected Output:**
```
5: apple
6: banana, cherry
4: date
3: fig
```

### Real-World Cases

- Sales grouped by product category.
- Users grouped by country.
- Log entries grouped by severity.

### References

- Enumerable.GroupBy — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupby
- Grouping operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/grouping-operations
- IGrouping<TKey,TElement> — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.igrouping-2

---

## 5. Joining

### Definitions

**Core Definition:** Joining correlates elements from two sequences based on matching keys.

**Technical Definition:** `Join` performs an **inner join**, returning pairs of matching elements. `GroupJoin` performs a **group join**, pairing each outer element with the collection of matching inner elements (used for left outer joins with `DefaultIfEmpty`). Both translate to SQL `JOIN` for `IQueryable`.

**Beginner-Friendly:** “Match items from two lists by a common key.”

### Purposes

- To combine related data from separate sources.
- To perform inner joins and group joins.
- To translate to SQL `JOIN` clauses.
- To model one-to-many relationships.

### Syntax Rules

```
outer.Join(inner, o => o.Key, i => i.Key, (o, i) => result)
outer.GroupJoin(inner, o => o.Key, i => i.Key, (o, group) => result)
```

- `Join` returns pairs of matching elements.
- `GroupJoin` returns each outer element with its matching inner collection.
- For left outer joins, combine `GroupJoin` with `SelectMany` and `DefaultIfEmpty`.

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

        // Inner join
        var rows = customers.Join(orders,
            c => c.Id, o => o.CustomerId,
            (c, o) => new { c.Name, o.Id, o.Total });

        foreach (var r in rows)
            Console.WriteLine($"{r.Name}: Order {r.Id} = {r.Total}");
        // Alice: Order 101 = 100
        // Bob: Order 102 = 200

        // Group join (left outer join pattern)
        var summary = customers.GroupJoin(orders,
            c => c.Id, o => o.CustomerId,
            (c, os) => new { c.Name, Count = os.Count(), Total = os.Sum(o => o.Total) });

        foreach (var s in summary)
            Console.WriteLine($"{s.Name}: {s.Count} orders, {s.Total}");
        // Alice: 1 orders, 100
        // Bob: 1 orders, 200
    }
}
```

**Expected Output:**
```
Alice: Order 101 = 100
Bob: Order 102 = 200
Alice: 1 orders, 100
Bob: 1 orders, 200
```

### Real-World Cases

- Customers joined with orders.
- Products joined with categories.
- Entity Framework Core query joins.

### References

- Enumerable.Join — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.join
- Enumerable.GroupJoin — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.groupjoin
- Join operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/join-operations

---

## 6. Aggregation

### Definitions

**Core Definition:** Aggregation computes a single value from a sequence, such as a sum, count, average, minimum, or maximum.

**Technical Definition:** Aggregation operators reduce a sequence to a single scalar value. The standard operators are `Count`, `LongCount`, `Sum`, `Min`, `Max`, `Average`, and `Aggregate` (a general-purpose fold). Most aggregation operators execute **immediately** (not deferred) because they must consume the entire sequence to produce a result.

**Beginner-Friendly:** “Summarise the data into one number.”

### Purposes

- To compute totals, averages, extremes, and counts.
- To implement custom accumulations with `Aggregate`.
- To produce summary statistics for reporting.
- To translate to SQL aggregate functions (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`).

### Syntax Rules

```
source.Count()
source.Count(predicate)
source.Sum()
source.Sum(x => selector)
source.Min()
source.Max()
source.Average()
source.Aggregate(seed, (acc, x) => combine(acc, x))
source.Aggregate((a, b) => combine(a, b))
```

- `Aggregate` without a seed uses the first element as the initial accumulator.
- `Aggregate` with a seed starts from the seed value.
- `Sum`, `Min`, `Max`, `Average` have overloads that accept a selector.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        Console.WriteLine(numbers.Sum());          // 15
        Console.WriteLine(numbers.Min());           // 1
        Console.WriteLine(numbers.Max());           // 5
        Console.WriteLine(numbers.Average());       // 3
        Console.WriteLine(numbers.Count(n => n % 2 == 0)); // 2

        // Custom aggregation: product
        int product = numbers.Aggregate(1, (acc, n) => acc * n);
        Console.WriteLine(product);                 // 120
    }
}
```

**Expected Output:** `15`, `1`, `5`, `3`, `2`, `120`

### Real-World Cases

- Financial totals and averages.
- Counting records matching a condition.
- Finding the maximum or minimum value in a dataset.

### References

- Enumerable.Aggregate — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate
- Enumerable.Sum — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.sum
- Aggregation operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/aggregation-operations

---

## 7. Quantifiers

### Definitions

**Core Definition:** Quantifiers return a Boolean value indicating whether some or all elements in a sequence satisfy a condition.

**Technical Definition:** Quantifier operations return a `Boolean` value that indicates whether some or all of the elements in a sequence satisfy a condition. The standard operators are `All` (all elements satisfy a predicate), `Any` (any element satisfies a predicate, or the sequence is non-empty), and `Contains` (the sequence contains a specified element). All three execute **immediately**.

**Beginner-Friendly:** “Ask a yes/no question about the data.”

### Purposes

- To validate that all elements meet a condition (`All`).
- To check for existence (`Any`).
- To test membership (`Contains`).
- To short-circuit expensive processing when the answer is already known.

### Syntax Rules

```
source.All(x => condition)
source.Any()
source.Any(x => condition)
source.Contains(value)
source.Contains(value, comparer)
```

- `All` returns `true` if the sequence is empty (vacuously true).
- `Any()` returns `true` if the sequence contains at least one element.
- `Any(predicate)` returns `true` if at least one element matches.
- `Contains` uses the default equality comparer unless one is supplied.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 2, 4, 6, 8 };

        Console.WriteLine(numbers.All(n => n % 2 == 0));   // True
        Console.WriteLine(numbers.Any(n => n > 10));       // False
        Console.WriteLine(numbers.Contains(4));            // True
    }
}
```

**Expected Output:** `True`, `False`, `True`

### Real-World Cases

- “Does this user exist?” (`Any`).
- “Are all items in stock?” (`All`).
- Permission checks (`Contains`).

### References

- Quantifier operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/quantifier-operations
- Enumerable.All — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.all
- Enumerable.Any — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.any

---

## 8. Partitioning

### Definitions

**Core Definition:** Partitioning returns a contiguous sub-sequence of a sequence by skipping and taking elements.

**Technical Definition:** Partitioning operators divide a sequence into two parts and return one of them. The operators are `Take` (first *n* elements), `Skip` (all but the first *n* elements), `TakeWhile` (elements while a predicate is true), `SkipWhile` (skip elements while a predicate is true, then return the rest), `TakeLast` and `SkipLast` (from the end). Partitioning uses deferred execution and translates to `TOP`/`LIMIT`/`OFFSET` in SQL.

**Beginner-Friendly:** “Slice out a piece of the sequence.”

### Purposes

- To implement pagination (`Skip(pageSize * page).Take(pageSize)`).
- To limit results to the first *n* elements.
- To take or skip while a condition holds.
- To translate to SQL `TOP`, `LIMIT`, `OFFSET`.

### Syntax Rules

```
source.Take(n)
source.Skip(n)
source.TakeWhile(x => condition)
source.SkipWhile(x => condition)
source.TakeLast(n)
source.SkipLast(n)
```

- `Take(n)` returns the first *n* elements (or all if fewer than *n*).
- `Skip(n)` skips the first *n* elements and returns the rest.
- `TakeWhile`/`SkipWhile` stop at the first element where the predicate is false.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] nums = { 1, 2, 3, 10, 4, 5 };

        Console.WriteLine(string.Join(", ", nums.Take(3)));       // 1, 2, 3
        Console.WriteLine(string.Join(", ", nums.Skip(3)));       // 10, 4, 5
        Console.WriteLine(string.Join(", ", nums.TakeWhile(n => n < 5))); // 1, 2, 3
        Console.WriteLine(string.Join(", ", nums.SkipWhile(n => n < 5))); // 10, 4, 5
    }
}
```

**Expected Output:** `1, 2, 3`, `10, 4, 5`, `1, 2, 3`, `10, 4, 5`

### Real-World Cases

- Pagination in web APIs.
- Taking the top *N* results.
- Skipping header rows in a data file.

### References

- Partitioning operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/partitioning-operations
- Enumerable.Take — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.take
- Enumerable.Skip — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.skip

---

## 9. Set Operations

### Definitions

**Core Definition:** Set operations produce a result set based on the presence or absence of equivalent elements within the same or separate collections.

**Technical Definition:** Set operations in LINQ refer to query operations that produce a result set based on the presence or absence of equivalent elements within the same or separate collections. The operators are `Distinct`/`DistinctBy` (unique elements), `Union`/`UnionBy` (set union), `Intersect`/`IntersectBy` (set intersection), `Except`/`ExceptBy` (set difference), and `Concat` (concatenation without removing duplicates). These operators use the default equality comparer unless one is supplied.

**Beginner-Friendly:** “Treat the sequences like mathematical sets.”

### Purposes

- To remove duplicates (`Distinct`).
- To find common elements (`Intersect`).
- To find elements unique to one sequence (`Except`).
- To combine sequences with or without deduplication (`Union` vs. `Concat`).

### Syntax Rules

```
source.Distinct()
source.Distinct(comparer)
source.DistinctBy(x => key)   // .NET 6+
first.Union(second)
first.Intersect(second)
first.Except(second)
first.Concat(second)
```

- `Union`, `Intersect`, `Except` remove duplicates.
- `Concat` preserves duplicates.
- `DistinctBy`, `UnionBy`, `IntersectBy`, `ExceptBy` use a key selector (available in .NET 6+).

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] a = { 1, 2, 3, 4 };
        int[] b = { 3, 4, 5, 6 };

        Console.WriteLine(string.Join(", ", a.Union(b)));      // 1, 2, 3, 4, 5, 6
        Console.WriteLine(string.Join(", ", a.Intersect(b)));  // 3, 4
        Console.WriteLine(string.Join(", ", a.Except(b)));     // 1, 2
        Console.WriteLine(string.Join(", ", a.Concat(b)));     // 1, 2, 3, 4, 3, 4, 5, 6
    }
}
```

**Expected Output:** `1, 2, 3, 4, 5, 6`, `3, 4`, `1, 2`, `1, 2, 3, 4, 3, 4, 5, 6`

### Real-World Cases

- Deduplicating tags or identifiers.
- Finding common subscribers between lists.
- Computing set differences for permissions.

### References

- Set operations (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/set-operations
- Enumerable.Union — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.union
- Enumerable.Distinct — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.distinct

---

## 10. Conversion

### Definitions

**Core Definition:** Conversion operators change the type of the input sequence, either by materializing it into a concrete collection or by changing its static type.

**Technical Definition:** Conversion methods change the type of input objects. The `As`-prefixed methods (`AsEnumerable`, `AsQueryable`) change the static type of the source but do not enumerate it. The `To`-prefixed methods (`ToArray`, `ToList`, `ToDictionary`, `ToLookup`) **enumerate** the source and put the items into the corresponding collection type, forcing **immediate execution**. `Cast` and `OfType` convert element types.

**Beginner-Friendly:** “Turn the query into something concrete, or change how it’s viewed.”

### Purposes

- To force immediate execution (`ToList`, `ToArray`).
- To materialize into a specific collection type (`Dictionary`, `Lookup`, `HashSet`).
- To change the static type without executing (`AsEnumerable`, `AsQueryable`).
- To cast or filter by type (`Cast`, `OfType`).

### Syntax Rules

```
source.AsEnumerable()
source.AsQueryable()
source.ToArray()
source.ToList()
source.ToDictionary(x => key)
source.ToDictionary(x => key, x => value)
source.ToLookup(x => key)
source.Cast<T>()
source.OfType<T>()
source.ToHashSet()   // .NET Core 2.0+
```

- `As` methods do **not** enumerate; `To` methods **do** enumerate.
- `ToDictionary` throws on duplicate keys; `ToLookup` allows duplicate keys (one-to-many).
- `Cast` throws `InvalidCastException` on incompatible elements; `OfType` filters them out.

### Annotated Example

```csharp
using System;
using System.Collections;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new[] { 1, 2, 3, 4, 5 };

        var list = data.Where(n => n % 2 == 0).ToList();
        Console.WriteLine($"List count: {list.Count}");   // 2

        var dict = data.ToDictionary(n => n, n => n * n);
        Console.WriteLine(dict[3]);                       // 9

        var set = data.ToHashSet();
        Console.WriteLine(set.Count);                     // 5

        // Cast/OfType with non-generic collection
        var arrayList = new ArrayList { 1, "two", 3 };
        var ints = arrayList.OfType<int>();
        Console.WriteLine(string.Join(", ", ints));       // 1, 3
    }
}
```

**Expected Output:** `List count: 2`, `9`, `5`, `1, 3`

### Real-World Cases

- Materializing query results for caching or reuse.
- Building lookup dictionaries from sequences.
- Working with legacy non-generic collections (`Cast`, `OfType`).

### References

- Converting Data Types — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2013/bb546162(v=vs.120)
- Enumerable.ToList — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.tolist
- Enumerable.ToDictionary — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.todictionary

---

## 11. Element Operations

### Definitions

**Core Definition:** Element operations return a single, specific element from a sequence.

**Technical Definition:** Element operations return a single, specific element from a sequence. The standard operators are `ElementAt`, `ElementAtOrDefault`, `First`, `FirstOrDefault`, `Last`, `LastOrDefault`, `Single`, `SingleOrDefault`, and `DefaultIfEmpty`. These operators execute **immediately** and are used to extract individual values from query results.

**Beginner-Friendly:** “Pick one item out of the sequence.”

### Purposes

- To retrieve the first, last, or only element.
- To access by index (`ElementAt`).
- To provide safe defaults for missing elements (`*OrDefault` variants).
- To supply a default value for empty sequences (`DefaultIfEmpty`).

### Syntax Rules

```
source.First()
source.First(predicate)
source.FirstOrDefault()
source.FirstOrDefault(predicate)
source.FirstOrDefault(defaultValue)   // .NET 6+

source.Last()
source.LastOrDefault()

source.Single()
source.SingleOrDefault()

source.ElementAt(index)
source.ElementAtOrDefault(index)

source.DefaultIfEmpty()
source.DefaultIfEmpty(defaultValue)
```

- `First`/`Last`/`Single` throw `InvalidOperationException` if no element (or for `Single`, more than one) is found.
- `*OrDefault` variants return `default(T)` (or a supplied default) instead of throwing.
- `Single` throws if there is more than one matching element; `SingleOrDefault` returns `default` only when the sequence is empty.
- `DefaultIfEmpty` is commonly used with `GroupJoin` to implement left outer joins.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 10, 20, 30 };

        Console.WriteLine(numbers.First());          // 10
        Console.WriteLine(numbers.Last());           // 30
        Console.WriteLine(numbers.ElementAt(1));     // 20

        int[] empty = { };
        Console.WriteLine(empty.FirstOrDefault());   // 0
        Console.WriteLine(empty.FirstOrDefault(-1)); // -1

        int[] one = { 42 };
        Console.WriteLine(one.Single());             // 42
    }
}
```

**Expected Output:** `10`, `30`, `20`, `0`, `-1`, `42`

### Real-World Cases

- Getting the newest record (`First` after ordering).
- Safe lookups that may return nothing (`FirstOrDefault`).
- Fetching a unique record by ID (`Single`).

### References

- Element Operations (C#) — Microsoft Learn (GitHub) - https://raw.githubusercontent.com/dotnet/docs/85d14aea60de33362057b87c6d46dad1d0d94f4d/docs/csharp/programming-guide/concepts/linq/element-operations.md
- Enumerable.First — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.first
- Enumerable.Single — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single

---

## 12. Generation Operations

### Definitions

**Core Definition:** Generation operations create a new sequence of values from scratch, without an existing data source.

**Technical Definition:** Generation refers to creating a new sequence of values. The standard query operator methods that perform generation are `Empty` (returns an empty collection of a specified type), `Range` (generates a sequence of integer numbers), `Repeat` (generates a collection with one repeated value), and `DefaultIfEmpty` (replaces an empty collection with a default-valued singleton). Generation operators are **static** on `Enumerable` and are not extension methods — they are called as `Enumerable.Range(...)` rather than on an existing sequence.

**Beginner-Friendly:** “Create a sequence out of thin air.”

### Purposes

- To create test data or placeholder sequences.
- To generate ranges of numbers for iteration.
- To repeat a value a specified number of times.
- To provide a default sequence for empty results.

### Syntax Rules

```
Enumerable.Empty<T>()
Enumerable.Range(start, count)
Enumerable.Repeat(element, count)
source.DefaultIfEmpty()
source.DefaultIfEmpty(defaultValue)
```

- `Empty` returns an empty `IEnumerable<T>`.
- `Range(start, count)` generates `count` integers starting at `start`.
- `Repeat(element, count)` repeats `element` `count` times.
- `DefaultIfEmpty` is an extension method; `Empty`, `Range`, and `Repeat` are static methods.

### Annotated Example

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        // Empty sequence
        var empty = Enumerable.Empty<int>();
        Console.WriteLine(empty.Count());   // 0

        // Range
        var range = Enumerable.Range(1, 5);
        Console.WriteLine(string.Join(", ", range));   // 1, 2, 3, 4, 5

        // Repeat
        var repeated = Enumerable.Repeat("A", 3);
        Console.WriteLine(string.Join(", ", repeated)); // A, A, A

        // DefaultIfEmpty
        var defaulted = Enumerable.Empty<int>().DefaultIfEmpty(-1);
        Console.WriteLine(string.Join(", ", defaulted)); // -1
    }
}
```

**Expected Output:** `0`, `1, 2, 3, 4, 5`, `A, A, A`, `-1`

### Real-World Cases

- Generating test data for unit tests.
- Creating a sequence of page numbers or indices.
- Providing a fallback sequence when a query returns empty.

### References

- Generation Operations — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/bb546129(v=vs.90)
- Enumerable.Range — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.range
- Enumerable.Repeat — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.repeat

---

## 13. LINQ Operation Best Practices

### Definitions

**Core Definition:** LINQ operation best practices are the guidelines that ensure queries are correct, efficient, and maintainable, avoiding common pitfalls such as multiple enumeration, premature materialization, and untranslatable expressions.

**Technical Definition:** Best practices for LINQ operations derive from Microsoft’s documentation and community experience. Key recommendations include: understanding **deferred execution** and its implications; **materializing once** with `ToList()` or `ToArray()` when reusing results; **filtering before projecting** to reduce data transfer; using `Any()` instead of `Count() > 0` for existence checks; **avoiding multiple enumerations** of deferred queries; keeping `IQueryable` queries **translatable**; and choosing the right operator for the task (e.g., `FirstOrDefault` vs. `SingleOrDefault`, `Union` vs. `Concat`).

**Beginner-Friendly:** The habits that keep LINQ queries fast and correct.

### Guidelines

```
// 1. Materialize once when reusing results
var activeUsers = users.Where(u => u.IsActive).ToList();
var count = activeUsers.Count;
var first = activeUsers.FirstOrDefault();

// 2. Filter before projecting (reduce data)
var names = db.Users.Where(u => u.IsActive).Select(u => u.Name).ToList();

// 3. Use Any() instead of Count() > 0
bool hasActive = users.Any(u => u.IsActive);

// 4. Avoid multiple enumeration of deferred queries
var query = users.Where(u => u.IsActive);   // deferred
// var c = query.Count();                   // enumeration 1
// var f = query.FirstOrDefault();           // enumeration 2 — avoid

// 5. Keep IQueryable queries translatable
var good = db.Users.Where(u => u.Age > 30); // translatable
// var bad = db.Users.Where(u => MyLocalMethod(u)); // not translatable

// 6. Choose the right element operator
var user = users.FirstOrDefault(u => u.Id == id); // absence is OK
var exact = users.Single(u => u.Id == id);          // exactly one required

// 7. Use Union for deduplication, Concat for concatenation
var union = a.Union(b);     // removes duplicates
var concat = a.Concat(b);   // preserves duplicates
```

### Purposes

- To avoid performance pitfalls (multiple enumeration, repeated database calls).
- To choose the correct operator for the task.
- To keep queries readable and translatable.
- To prevent memory and resource leaks.

### Annotated Example

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class User { public string Name { get; set; } public bool IsActive { get; set; } }

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

        // BAD: multiple enumerations
        var query = users.Where(u => u.IsActive);
        var count = query.Count();          // enumeration 1
        var first = query.FirstOrDefault();  // enumeration 2

        // GOOD: materialize once
        var list = users.Where(u => u.IsActive).ToList();
        var count2 = list.Count;
        var first2 = list.FirstOrDefault();

        Console.WriteLine(count2);           // 2
        Console.WriteLine(first2?.Name);     // Alice
    }
}
```

**Expected Output:** `2`, `Alice`

### Real-World Cases

- Database queries: materialize once to avoid repeated SQL execution.
- API responses: filter before projecting to reduce payload size.
- Large in-memory collections: avoid re-evaluating expensive predicates.

### References

- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Standard query operators overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Core LINQ operators | Introduced in **.NET 3.5 / C# 3.0**; stable. |
| `DistinctBy`, `UnionBy`, `IntersectBy`, `ExceptBy`, `Chunk`, `MaxBy`, `MinBy` | Introduced in **.NET 6**; version-specific. |
| `Take(Range)`, `Skip(Range)` | Introduced in **.NET 6**; version-specific. |
| `FirstOrDefault(defaultValue)`, `LastOrDefault(defaultValue)`, `SingleOrDefault(defaultValue)` | Introduced in **.NET 6**; version-specific. |
| `ToHashSet()` | Introduced in **.NET Core 2.0**; version-specific. |
| `Order` / `OrderDescending` | Introduced in **.NET 7**; version-specific. |
| `IQueryable<T>` translation | **Provider-dependent**; not all operators translate. |
| Deferred execution | **Language-defined**; most operators are lazy, but aggregations and conversions are immediate. |
| Multiple enumeration | **Performance pitfall**; detected by analyzer CA1851. |

---

## References (Consolidated)

- Standard query operators overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/
- Generation Operations — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2008/bb546129(v=vs.90)
- Quantifier Operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/quantifier-operations
- Set operations (C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/set-operations
- Converting Data Types — Microsoft Learn - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2013/bb546162(v=vs.120)
- Element Operations (C#) — Microsoft Learn (GitHub) - https://raw.githubusercontent.com/dotnet/docs/85d14aea60de33362057b87c6d46dad1d0d94f4d/docs/csharp/programming-guide/concepts/linq/element-operations.md
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Enumerable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable
- Queryable class — .NET API - https://learn.microsoft.com/en-us/dotnet/api/system.linq.queryable
- Projection operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/projection-operations
- Sorting operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/sorting-operations
- Grouping operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/grouping-operations
- Join operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/join-operations
- Aggregation operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/aggregation-operations
- Partitioning operations — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/partitioning-operations

---