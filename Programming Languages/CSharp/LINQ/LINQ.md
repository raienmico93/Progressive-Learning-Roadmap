# C# LINQ Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**LINQ (Language Integrated Query)** is a set of language extensions and APIs in .NET that provides a **uniform, declarative query syntax** for working with data from virtually any source—in-memory collections, XML documents, relational databases, DataSets, and more. Instead of writing different query languages for different data stores, you write SQL-like queries directly in C# (or method chains) and let **LINQ providers** translate them into the appropriate operations for each source.

**Technical Definition**
LINQ was introduced in **.NET Framework 3.5** and **C# 3.0** as a bridge between the object world and the data world. It is built on **extension methods**, **lambda expressions**, **expression trees**, and **generic delegates**. The architecture revolves around two core interfaces: **`IEnumerable<T>`** (in-memory, delegate-based execution via `Enumerable`) and **`IQueryable<T>`** (remote, expression-tree-based execution via `Queryable`). LINQ is **pluggable**: providers implement `IQueryable<T>` and `IQueryProvider` to translate expression trees into the target data store's language (SQL, XQuery, OData, etc.). The .NET class library ships with several built-in providers: LINQ to Objects, LINQ to XML, LINQ to SQL, LINQ to Entities (Entity Framework Core), LINQ to DataSet, and Parallel LINQ (PLINQ).

**Beginner-Friendly Explanation**
LINQ is a **universal query language for C#**. Instead of writing a `foreach` loop to filter a list, you write `list.Where(x => x > 5)`. Instead of writing SQL strings, you write `db.Customers.Where(c => c.City == "London")`. The same syntax works whether you are querying an array, an XML file, a DataSet, or a SQL database—because LINQ **providers** translate your query into whatever the underlying data source understands. Think of LINQ as a **remote control with one set of buttons** that works on many different devices.

---

### Key Characteristics

- **Uniform query syntax** — Query and method syntax are semantically identical; query syntax compiles to method calls.
- **Pluggable providers** — The same LINQ syntax works across multiple data sources via providers.
- **Two execution models** — `IEnumerable<T>` (delegates, in-memory) and `IQueryable<T>` (expression trees, remote).
- **Deferred execution** — Most queries are lazy; they execute only when enumerated.
- **Composable** — Queries can be built incrementally by chaining operators.
- **Strongly typed** — Compile-time type checking and IntelliSense support.
- **Functional** — Built on higher-order functions (lambdas, `Func<>`).

---

### Prerequisites

- Understanding of C# **generics** (`List<T>`, `IEnumerable<T>`).
- Familiarity with **delegates** and **lambda expressions**.
- Knowledge of **extension methods**.
- Basic awareness of **expression trees** (for `IQueryable`).
- Familiarity with `foreach` loops and collections.

---

### Related Programming Areas

- **Data access** — Entity Framework Core, Dapper, LINQ to SQL.
- **XML processing** — LINQ to XML (`System.Xml.Linq`).
- **Parallel programming** — PLINQ for CPU-bound workloads.
- **Functional programming** — LINQ is C#'s functional query API.
- **Serialization** — LINQ to JSON (Json.NET, System.Text.Json).

---

### Core Concepts / Features

1. LINQ (Language Integrated Query) — Overarching Concept
2. LINQ Architecture: `IEnumerable<T>` vs. `IQueryable<T>`
3. Query Syntax vs. Method Syntax
4. Deferred vs. Immediate Execution
5. LINQ to Objects
6. LINQ to XML
7. LINQ to SQL
8. LINQ to Entities (Entity Framework Core)
9. LINQ to DataSet
10. Parallel LINQ (PLINQ)
11. LINQ Best Practices

Each concept follows the same uniform structure: definitions, purposes, syntax rules, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. LINQ (Language Integrated Query) — Overarching Concept

### Definitions

**Core Definition**
LINQ is a language-integrated query facility that lets you write declarative queries against any data source using C# syntax.

**Technical Definition**
LINQ (Language Integrated Query) is a revolutionary feature introduced in .NET Framework 3.5 that bridges the object world and the data world. It consists of language extensions (query syntax keywords such as `from`, `where`, `select`) and a set of APIs (the `Enumerable` and `Queryable` classes and their standard query operators). LINQ queries are translated into method calls on these classes, which in turn operate on `IEnumerable<T>` or `IQueryable<T>`. LINQ is **pluggable** by design: providers implement `IQueryable<T>` to translate expression trees into a target query language.

**Beginner-Friendly Explanation**
LINQ is **SQL for C#—but for everything, not just databases**. You can query arrays, lists, XML, DataSets, and databases with the same syntax.

---

### Purposes

- To provide a **single, uniform query language** across data sources.
- To enable **compile-time type checking** and IntelliSense for queries.
- To reduce boilerplate compared to manual `foreach` loops.
- To make queries **composable and declarative**.
- To support **functional-style data transformations**.

---

### Syntax Rules and Structure

```
// Query syntax
var query = from item in collection
            where predicate(item)
            orderby key(item)
            select projection(item);

// Method syntax
var query2 = collection
    .Where(item => predicate(item))
    .OrderBy(item => key(item))
    .Select(item => projection(item));
```

**Component breakdown**

- `from` — Introduces the range variable and data source.
- `where` — Filters elements.
- `orderby` — Sorts elements.
- `select` — Projects each element into a new form.
- Method syntax — Equivalent extension method calls (`Where`, `OrderBy`, `Select`).

#### Syntax Rules

- Query syntax and method syntax are **semantically identical**; query syntax compiles to method calls.
- Query syntax is often more readable for complex queries with joins and `let` clauses.
- Method syntax is required for some operators (e.g., `Count()`, `First()`) and is more concise for simple operations.
- Most queries use **deferred execution**; they run when enumerated.

#### Constraints and Limitations

- Query syntax cannot express all operators (e.g., `Chunk`, `DistinctBy`).
- `IQueryable<T>` queries must be translatable by the provider; unsupported methods throw.
- Statement lambdas cannot be used in expression trees (only expression lambdas).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Query Syntax

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5, 6 };

        var evenSquares = from n in numbers
                          where n % 2 == 0
                          select n * n;

        Console.WriteLine(string.Join(", ", evenSquares));   // 4, 16, 36
    }
}
```

**Expected Output**

```
4, 16, 36
```

**Why this output occurs**

- The query filters even numbers (`2, 4, 6`) and projects each to its square (`4, 16, 36`).

---

#### Example 2: Method Syntax

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        string[] words = { "apple", "banana", "cherry" };

        var lengths = words
            .Where(w => w.Length > 5)
            .Select(w => new { Word = w, Length = w.Length })
            .OrderByDescending(x => x.Length);

        foreach (var item in lengths)
            Console.WriteLine($"{item.Word}: {item.Length}");
        // banana: 6
    }
}
```

**Expected Output**

```
banana: 6
```

**Why this output occurs**

- `Where` filters words longer than 5 characters (`banana`); `Select` projects to an anonymous type; `OrderByDescending` sorts by length.

---

### Real-World Cases

- **Data access** — Querying databases with Entity Framework Core.
- **In-memory processing** — Filtering and transforming lists and arrays.
- **XML manipulation** — Querying XML documents.
- **Reporting** — Aggregating and grouping data.

---

### References

- Language Integrated Query (LINQ) — .NET - https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- Querying in C# (ACM SIGPLAN Companion) - https://dl.acm.org/doi/10.1145/1297846.1297889
- LINQ For Dummies (Google Books) - https://www.google.com/books/edition/LINQ_For_Dummies

---

## 2. LINQ Architecture: `IEnumerable<T>` vs. `IQueryable<T>`

### Definitions

**Core Definition**
LINQ operates through two core interfaces: `IEnumerable<T>` for **in-memory, delegate-based execution** and `IQueryable<T>` for **remote, expression-tree-based execution**.

**Technical Definition**
`IEnumerable<T>` is the basic interface for in-memory enumeration; LINQ operators on it accept `Func<T, bool>` delegates and execute **in process**, one element at a time. `IQueryable<T>` extends `IEnumerable<T>` and exposes an `Expression` property; LINQ operators on it accept **expression trees** (`Expression<Func<T, bool>>`) instead of delegates, allowing a provider to **translate** the query into the data store's language (SQL, OData, etc.) and execute it remotely. `Queryable` provides the extension methods for `IQueryable<T>`; `Enumerable` provides the extension methods for `IEnumerable<T>`.

**Beginner-Friendly Explanation**
`IEnumerable<T>` is **LINQ for lists and arrays in memory**. `IQueryable<T>` is **LINQ for databases and remote sources**—it captures your lambda as data (an expression tree) so the database can translate it to SQL.

---

### Purposes

- To separate **in-memory** from **remote** execution.
- To enable **query translation** for databases and web services.
- To provide **deferred execution** and composability.
- To allow providers to **optimize** queries before execution.

---

### Syntax Rules and Structure

| Aspect | `IEnumerable<T>` | `IQueryable<T>` |
|---|---|---|
| **Execution** | In-memory | Remote (database, service) |
| **Delegates** | `Func<T, bool>` | `Expression<Func<T, bool>>` |
| **Namespace** | `System.Collections.Generic` | `System.Linq` |
| **Extension class** | `Enumerable` | `Queryable` |
| **Translation** | None (direct execution) | Provider translates to target language |

**Component breakdown**

- `IEnumerable<T>` — In-memory sequence.
- `IQueryable<T>` — Query that carries an expression tree.
- `Expression` — The query's expression tree.
- `Provider` — The `IQueryProvider` that executes the query.

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

#### Example 1: `IEnumerable<T>` vs. `IQueryable<T>`

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var data = new List<int> { 1, 2, 3, 4, 5 };

        // IEnumerable<T> — delegates, in-memory
        var enumerableQuery = data.Where(n => n > 2);
        Console.WriteLine($"Enumerable type: {enumerableQuery.GetType().Name}");

        // IQueryable<T> — expression trees (simulated)
        var queryable = data.AsQueryable();
        var queryableQuery = queryable.Where(n => n > 2);
        Console.WriteLine($"Queryable type: {queryableQuery.GetType().Name}");
        Console.WriteLine($"Expression: {queryableQuery.Expression}");
    }
}
```

**Expected Output (approximate)**

```
Enumerable type: WhereListIterator`1
Queryable type: EnumerableQuery`1
Expression: value(System.Collections.Generic.List`1[System.Int32]).Where(n => (n > 2))
```

**Why this output occurs**

- `Where` on `IEnumerable<T>` returns an iterator (delegate-based); `Where` on `IQueryable<T>` returns an expression-tree-carrying query.

---

### Real-World Cases

- **Entity Framework Core** — `DbSet<T>` implements `IQueryable<T>`.
- **LINQ to Objects** — Collections implement `IEnumerable<T>`.
- **OData clients** — `IQueryable<T>` queries are translated to OData URLs.

---

### References

- LINQ: The Philosophy — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/charlie/linq-the-philosophy
- What's the difference between Enumerable and Queryable — Microsoft Learn - https://learn.microsoft.com/en-us/archive/blogs/charlie/whats-the-difference-between-enumerable-and-queryable
- The Pain of Implementing LINQ Providers — Communications of the ACM - https://cacm.acm.org/magazines/2011/8/114944-the-pain-of-implementing-linq-providers/fulltext

---

## 3. Query Syntax vs. Method Syntax

### Definitions

**Core Definition**
**Query syntax** uses SQL-like keywords (`from`, `where`, `select`); **method syntax** uses extension method calls (`Where`, `Select`). They are semantically identical.

**Technical Definition**
Query syntax is **syntactic sugar** that the C# compiler translates into method calls on the `Enumerable` or `Queryable` classes. Both forms compile to the same intermediate representation; the choice is largely a matter of readability. Query syntax is often preferred for complex queries with multiple `from` clauses, joins, and `let` bindings. Method syntax is more concise for simple operations and is required for operators that have no query-syntax equivalent (e.g., `Count`, `First`, `Chunk`).

**Beginner-Friendly Explanation**
Query syntax looks like SQL: `from x in list where x > 5 select x`. Method syntax looks like C#: `list.Where(x => x > 5).Select(x => x)`. They do the same thing.

---

### Purposes

- To offer **readable, SQL-like syntax** for complex queries (query syntax).
- To provide **concise, fluent syntax** for simple operations (method syntax).
- To enable **composition** with both styles.
- To support operators that only exist as methods.

---

### Syntax Rules and Structure (Side-by-Side)

```
// Query syntax
var q1 = from n in numbers
         where n > 5
         orderby n descending
         select n * 2;

// Method syntax (equivalent)
var q2 = numbers
    .Where(n => n > 5)
    .OrderByDescending(n => n)
    .Select(n => n * 2);
```

**Component breakdown**

- `from` → `.Where`/`.SelectMany`
- `where` → `.Where`
- `orderby` → `.OrderBy`/`.ThenBy`
- `select` → `.Select`
- `join` → `.Join`/`.GroupJoin`
- `let` → `.Select` with temporary variables

#### Syntax Rules

- Query syntax compiles to method calls; there is no semantic difference.
- Query syntax is often more readable for joins, `let`, and nested projections.
- Method syntax is required for operators without query-syntax equivalents.

#### Constraints and Limitations

- Query syntax cannot express all operators (`Count`, `First`, `Chunk`, `DistinctBy`).
- Method syntax can become unreadable for complex joins.

---

### Annotated Code Examples and Expected Outputs

#### Example: Same Query, Both Syntaxes

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

        // Query syntax
        var querySyntax = from n in numbers
                          where n % 2 == 0
                          orderby n descending
                          select n;

        // Method syntax
        var methodSyntax = numbers
            .Where(n => n % 2 == 0)
            .OrderByDescending(n => n);

        Console.WriteLine(string.Join(", ", querySyntax));   // 10, 8, 6, 4, 2
        Console.WriteLine(string.Join(", ", methodSyntax));   // 10, 8, 6, 4, 2
    }
}
```

**Expected Output**

```
10, 8, 6, 4, 2
10, 8, 6, 4, 2
```

**Why this output occurs**

- Both forms produce the same results; the compiler translates query syntax into method calls.

---

### Real-World Cases

- **Complex joins** — Query syntax is clearer.
- **Simple filters** — Method syntax is more concise.
- **Dynamic queries** — Method syntax is required for programmatic composition.

---

### References

- Use C# to write LINQ queries — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/get-started/write-linq-queries
- Query Syntax versus Method Syntax (C#) — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/bb397947(v=vs.100)

---

## 4. Deferred vs. Immediate Execution

### Definitions

**Core Definition**
**Deferred execution** means a LINQ query does not run when it is defined; it runs when it is enumerated. **Immediate execution** forces the query to run at a specific point in code.

**Technical Definition**
Most LINQ operators use **deferred (lazy) execution**: the query is represented as an object that, when enumerated (e.g., by `foreach`, `ToList()`, `Count()`, `First()`), evaluates the source and produces results. Operators that return a single value (`Count`, `First`, `Max`, `Sum`, `Average`) and conversion operators (`ToList`, `ToArray`, `ToDictionary`) force **immediate execution**. Deferred execution enables **query composition**—building a query incrementally before executing it once. However, it can also cause **multiple enumerations** if the query is evaluated more than once.

**Beginner-Friendly Explanation**
**Deferred** means “I will run when you actually need the results.” **Immediate** means “run right now.” LINQ is lazy by default.

---

### Purposes

- To enable **query composition** without executing prematurely.
- To allow **provider optimization** before execution.
- To avoid unnecessary work when results are not needed.
- To control **when** expensive queries hit the database.

---

### Syntax Rules and Structure

```
// Deferred: query is not executed yet
var query = numbers.Where(n => n > 5);

// Immediate: ToList() forces execution
var list = numbers.Where(n => n > 5).ToList();

// Immediate: aggregate operators
int count = numbers.Count(n => n > 5);
int first = numbers.First(n => n > 5);
```

**Component breakdown**

- **Deferred** — `Where`, `Select`, `OrderBy`, `Take`, `Skip`, `GroupBy`.
- **Immediate** — `Count`, `Sum`, `Average`, `Max`, `Min`, `First`, `FirstOrDefault`, `Single`, `ToList`, `ToArray`, `ToDictionary`.

#### Syntax Rules

- Queries execute when enumerated (`foreach`, `ToList()`, `ToArray()`, aggregate operators).
- Deferred queries re-execute on each enumeration.
- `ToList()` materializes the results once, preventing re-execution.

#### Constraints and Limitations

- Multiple enumerations of a deferred query can cause duplicate database calls.
- Deferred queries capture variables by reference; changing the variable after query definition affects the query result.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Deferred Execution

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };

        var query = numbers.Where(n => n > 2);   // NOT executed yet

        Console.WriteLine("Before iteration");

        foreach (var n in query)   // executed here
            Console.Write(n + " ");
        // Before iteration
        // 3 4 5
    }
}
```

**Expected Output**

```
Before iteration
3 4 5
```

**Why this output occurs**

- The query is defined before the `foreach` but executed only when iterated.

---

#### Example 2: Multiple Enumerations

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        var query = numbers.Where(n => n > 2);

        Console.WriteLine(query.Count());     // 3 (first execution)
        Console.WriteLine(query.First());     // 3 (second execution)
    }
}
```

**Expected Output**

```
3
3
```

**Why this output occurs**

- The deferred query is evaluated twice—once for `Count()`, once for `First()`.

---

#### Example 3: Materialization Prevents Re-execution

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] numbers = { 1, 2, 3, 4, 5 };
        var list = numbers.Where(n => n > 2).ToList();   // executed once

        Console.WriteLine(list.Count);   // 3
        Console.WriteLine(list.First()); // 3
    }
}
```

**Expected Output**

```
3
3
```

**Why this output occurs**

- `ToList()` materializes the results; subsequent operations use the list, not the query.

---

### Real-World Cases

- **Database queries** — Deferred execution allows building queries before hitting the database.
- **Pagination** — `Skip`/`Take` compose without executing.
- **Caching** — Materialize once and reuse.

---

### References

- Classification of Standard Query Operators by Manner of Execution — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb546142(v=vs.100)
- Query Execution — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb397906(v=vs.100)
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries

---

## 5. LINQ to Objects

### Definitions

**Core Definition**
**LINQ to Objects** is the use of LINQ queries directly against any `IEnumerable<T>` or `IEnumerable` collection **in memory**, without an intermediate provider or API.

**Technical Definition**
LINQ to Objects refers to using LINQ queries with any `IEnumerable<T>` collection (such as `List<T>`, arrays, `Dictionary<TKey,TValue>`) directly, without using an intermediate LINQ provider or API such as LINQ to SQL or LINQ to XML. The queries execute in process using delegates (`Func<T, bool>`, `Func<T, TResult>`). LINQ to Objects provides **declarative filtering, ordering, grouping, and transformation** over in-memory data, offering more concise and readable code than manual `foreach` loops.

**Beginner-Friendly Explanation**
LINQ to Objects is **LINQ for your lists and arrays**. It runs entirely in memory, using delegates.

---

### Purposes

- To query **in-memory collections** declaratively.
- To replace complex `foreach` loops with readable queries.
- To enable **functional-style transformations** on collections.
- To provide **compile-time type checking** for in-memory queries.

---

### Syntax Rules and Structure

```
var query = from item in collection
            where condition
            orderby key
            select projection;
// or method syntax
var query2 = collection.Where(item => condition).Select(item => projection);
```

**Component breakdown**

- Source — Any `IEnumerable<T>` (arrays, lists, dictionaries).
- Operators — Standard query operators (`Where`, `Select`, `OrderBy`, etc.).
- Execution — In-memory, delegate-based, deferred.

#### Syntax Rules

- Works with any `IEnumerable<T>`.
- Executes in memory using delegates.
- Supports all standard query operators.

#### Constraints and Limitations

- No translation to external languages (unlike `IQueryable`).
- Performance depends on the in-memory data structure.

---

### Annotated Code Examples and Expected Outputs

#### Example: Querying a `List<T>`

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

class Program
{
    static void Main()
    {
        var students = new List<Student>
        {
            new("Terry", "Adams", 120, new[] { 99, 82, 81, 79 }),
            new("Fadi", "Fakhouri", 116, new[] { 99, 86, 90, 94 })
        };

        var highScorers = from s in students
                          where s.ExamScores.Average() > 85
                          select new { s.FirstName, s.LastName, Avg = s.ExamScores.Average() };

        foreach (var s in highScorers)
            Console.WriteLine($"{s.FirstName} {s.LastName}: {s.Avg:F1}");
        // Fadi Fakhouri: 92.3
    }
}

public record Student(string FirstName, string LastName, int ID, int[] ExamScores);
```

**Expected Output**

```
Fadi Fakhouri: 92.3
```

**Why this output occurs**

- LINQ to Objects filters students by average exam score and projects the results in memory.

---

### Real-World Cases

- **In-memory data processing** — Filtering and transforming lists.
- **Unit tests** — Querying test data.
- **Business logic** — Aggregating domain objects.

---

### References

- Query a collection of objects (LINQ in C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/query-a-collection-of-objects
- LINQ to Objects — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/11-net-linq-to-objects.mdx

---

## 6. LINQ to XML

### Definitions

**Core Definition**
**LINQ to XML** is an in-memory XML programming interface (`System.Xml.Linq`) that uses LINQ queries to create, parse, query, and modify XML documents.

**Technical Definition**
LINQ to XML provides a modern, developer-friendly way to work with XML in .NET applications. It is a powerful in-memory XML programming interface that leverages LINQ capabilities to simplify creating, parsing, and manipulating XML documents. Unlike the traditional DOM approach, LINQ to XML offers a more intuitive programming model with better performance characteristics. The core classes are `XDocument`, `XElement`, `XAttribute`, `XComment`, `XText`, and `XNamespace`. You can construct XML documents functionally, load from files/strings/URLs, and query with LINQ operators such as `Elements`, `Descendants`, `Attributes`, and `Where`.

**Beginner-Friendly Explanation**
LINQ to XML is **XML processing with LINQ queries**. You can build XML with nested `new XElement(...)` calls and query it with `from e in doc.Descendants("Book") where ...`.

---

### Purposes

- To **create XML** documents programmatically with intuitive syntax.
- To **query XML** with LINQ operators.
- To **transform XML** (filter, sort, project).
- To provide a **modern alternative** to `XmlDocument`.

---

### Syntax Rules and Structure

```
// Construct XML
var doc = new XDocument(
    new XElement("Books",
        new XElement("Book", new XAttribute("Id", "1"),
            new XElement("Title", "LINQ in Action"))));

// Query XML
var titles = from b in doc.Descendants("Book")
             where (decimal)b.Element("Price") < 40
             select (string)b.Element("Title");
```

**Component breakdown**

- `XDocument` — The document root.
- `XElement` — An XML element.
- `XAttribute` — An attribute.
- `Descendants` — Navigates to descendant elements.
- `Element` — Gets a child element.
- `Value` — Gets the text value.

#### Syntax Rules

- XML construction is functional: nesting `XElement` calls mirrors the XML hierarchy.
- Queries use standard LINQ operators.
- `XElement` and `XAttribute` support implicit conversions to `string`, `int`, `decimal`, etc.

#### Constraints and Limitations

- Fully in-memory; not suitable for very large documents without streaming.
- Namespace handling requires `XNamespace`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Creating and Querying XML

```csharp
using System;
using System.Linq;
using System.Xml.Linq;

class Program
{
    static void Main()
    {
        var doc = new XDocument(
            new XElement("Books",
                new XElement("Book", new XAttribute("Id", "1"),
                    new XElement("Title", "LINQ in Action"),
                    new XElement("Price", 39.99)),
                new XElement("Book", new XAttribute("Id", "2"),
                    new XElement("Title", "C# Programming"),
                    new XElement("Price", 44.95))));

        var cheapBooks = from b in doc.Descendants("Book")
                         where (decimal)b.Element("Price") < 40
                         select (string)b.Element("Title");

        foreach (var title in cheapBooks)
            Console.WriteLine(title);   // LINQ in Action
    }
}
```

**Expected Output**

```
LINQ in Action
```

**Why this output occurs**

- The XML is constructed functionally; the query filters books with price < 40 and projects the title.

---

### Real-World Cases

- **Configuration files** — Reading and writing XML settings.
- **Data exchange** — Parsing and generating XML payloads.
- **Reporting** — Transforming XML data into reports.

---

### References

- LINQ to XML — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/12-net-linq-to-xml.mdx
- LINQ to XML overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/linq-xml-overview

---

## 7. LINQ to SQL

### Definitions

**Core Definition**
**LINQ to SQL** is an ORM technology (part of .NET Framework) that maps relational database tables to C# classes and allows querying with LINQ syntax.

**Technical Definition**
LINQ to SQL is an Object-Relational Mapping (ORM) technology that bridges the gap between the object-oriented world of C# and the relational database world of SQL. It provides a runtime infrastructure for managing relational data as objects without losing relational capabilities. You define a `DataContext` (representing the database) and entity classes decorated with `[Table]` and `[Column]` attributes. LINQ queries against `Table<T>` properties are translated to SQL and executed on the database. LINQ to SQL was introduced in .NET Framework 3.5 and is largely superseded by Entity Framework Core, but remains available in .NET Framework.

**Beginner-Friendly Explanation**
LINQ to SQL is **LINQ for SQL Server** (legacy). You define classes that map to tables and query them with LINQ; it translates to SQL.

---

### Purposes

- To query SQL Server databases using LINQ syntax.
- To map tables to C# classes (ORM).
- To perform CRUD operations with LINQ.
- To provide type-safe database access.

---

### Syntax Rules and Structure

```
[Database]
public class NorthwindDataContext : DataContext
{
    public Table<Customer> Customers;
    public NorthwindDataContext(string connection) : base(connection) { }
}

[Table(Name = "Customers")]
public class Customer
{
    [Column(IsPrimaryKey = true)] public string CustomerID;
    [Column] public string CompanyName;
}

// Query
var query = from c in db.Customers
            where c.Country == "Germany"
            select c;
```

**Component breakdown**

- `DataContext` — The database connection and query gateway.
- `Table<T>` — Represents a database table.
- `[Table]` / `[Column]` — Mapping attributes.
- Query — LINQ to SQL translates to SQL.

#### Syntax Rules

- Requires .NET Framework (LINQ to SQL is not in .NET Core / .NET 5+).
- Entity classes must be decorated with mapping attributes.
- Queries against `Table<T>` are translated to SQL.

#### Constraints and Limitations

- **Legacy technology**; not recommended for new development.
- SQL Server only (no other databases).
- Superseded by Entity Framework Core.

---

### Annotated Code Examples and Expected Outputs

#### Example: Querying with LINQ to SQL

```csharp
using System;
using System.Data.Linq;
using System.Data.Linq.Mapping;
using System.Linq;

[Database]
public class NorthwindDataContext : DataContext
{
    public Table<Customer> Customers;
    public NorthwindDataContext(string connection) : base(connection) { }
}

[Table(Name = "Customers")]
public class Customer
{
    [Column(IsPrimaryKey = true)] public string CustomerID;
    [Column] public string CompanyName;
    [Column] public string City;
    [Column] public string Country;
}

class Program
{
    static void Main()
    {
        var db = new NorthwindDataContext("...connection string...");

        var germanCustomers = from c in db.Customers
                              where c.Country == "Germany"
                              select c;

        foreach (var c in germanCustomers)
            Console.WriteLine($"{c.CustomerID}: {c.CompanyName} ({c.City})");
        // ALFKI: Alfreds Futterkiste (Berlin)
        // ...
    }
}
```

**Expected Output (approximate)**

```
ALFKI: Alfreds Futterkiste (Berlin)
ANATR: Ana Trujillo Emparedados y helados (México D.F.)
```

**Why this output occurs**

- LINQ to SQL translates the query to `SELECT * FROM Customers WHERE Country = 'Germany'` and materializes the results as `Customer` objects.

---

### Real-World Cases

- **Legacy .NET Framework applications** — Maintaining existing LINQ to SQL code.
- **Simple SQL Server CRUD** — Quick data access without full ORM.

---

### References

- C# LINQ to SQL — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/15-c-sharp-database-ac-sharpc-sharpess/8-c-sharp-linq-to-sql.mdx
- Walkthrough: Simple Object Model and Query (C#) — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb386944(v=vs.100)

---

## 8. LINQ to Entities (Entity Framework Core)

### Definitions

**Core Definition**
**LINQ to Entities** is the LINQ provider for **Entity Framework Core** (and EF6), translating LINQ queries against `DbSet<T>` into SQL for the target database.

**Technical Definition**
Entity Framework Core (EF Core) is an open-source, lightweight, extensible Object-Relational Mapper (ORM) for .NET. It serves as a bridge between your C# object model and your database, eliminating the need for most data-access code. EF Core uses LINQ to translate C# queries into SQL. `DbSet<T>` implements `IQueryable<T>`, so queries composed with LINQ operators are captured as expression trees and translated to the database's SQL dialect. EF Core supports multiple databases (SQL Server, PostgreSQL, MySQL, SQLite, Azure Cosmos DB) and three modeling approaches: Model First, Database First, and Code First (the predominant approach).

**Beginner-Friendly Explanation**
LINQ to Entities is **LINQ for databases via Entity Framework Core**. You write `db.Blogs.Where(b => b.Url.Contains("dotnet"))` and EF Core turns it into SQL.

---

### Purposes

- To query databases with **type-safe LINQ**.
- To map C# objects to relational tables (ORM).
- To support **multiple database providers**.
- To enable **Code First** development with migrations.
- To provide **LINQ-based data access** without writing SQL.

---

### Syntax Rules and Structure

```
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}

// Query
var blogs = from b in context.Blogs
            where b.Url.Contains("dotnet")
            orderby b.Url
            select b;

// Or method syntax
var blogs2 = context.Blogs
    .Where(b => b.Url.Contains("dotnet"))
    .OrderBy(b => b.Url);
```

**Component breakdown**

- `DbContext` — The database session.
- `DbSet<T>` — A table in the database.
- LINQ query — Translated to SQL by EF Core.
- `ToList()` / `foreach` — Triggers execution.

#### Syntax Rules

- `DbSet<T>` implements `IQueryable<T>`.
- Queries are translated to SQL; unsupported methods throw `InvalidOperationException`.
- `AsEnumerable()` / `ToList()` forces client-side evaluation (use sparingly).
- Code First uses migrations to create/update the schema.

#### Constraints and Limitations

- Not all LINQ methods are translatable; client-side evaluation must be explicit.
- Lazy loading requires `virtual` navigation properties and proxy configuration.
- Performance depends on query translation quality and indexing.

---

### Annotated Code Examples and Expected Outputs

#### Example: Querying with EF Core

```csharp
using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;

public class Blog { public int BlogId { get; set; } public string Url { get; set; } }
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=blogs.db");
}

class Program
{
    static void Main()
    {
        using var context = new BloggingContext();

        var dotnetBlogs = from b in context.Blogs
                          where b.Url.Contains("dotnet")
                          orderby b.Url
                          select b;

        foreach (var blog in dotnetBlogs)
            Console.WriteLine(blog.Url);
    }
}
```

**Expected Output (approximate)**

```
https://devblogs.microsoft.com/dotnet/
https://learn.microsoft.com/dotnet/
```

**Why this output occurs**

- EF Core translates the LINQ query to SQL (`SELECT * FROM Blogs WHERE Url LIKE '%dotnet%' ORDER BY Url`) and materializes the results as `Blog` objects.

---

### Real-World Cases

- **Web applications** — ASP.NET Core with EF Core.
- **Enterprise applications** — Code First migrations and LINQ queries.
- **Cross-platform data access** — SQLite, PostgreSQL, SQL Server.

---

### References

- Entity Framework Core — ClickNLearn - https://mintlify.wiki/anandstays-creator/clicknlearn/dotnet/ef-core
- Entity Framework Core — Microsoft Learn - https://learn.microsoft.com/en-us/ef/core/

---

## 9. LINQ to DataSet

### Definitions

**Core Definition**
**LINQ to DataSet** enables LINQ queries against `DataSet` and `DataTable` objects, providing type-safe access to ADO.NET disconnected data.

**Technical Definition**
LINQ to DataSet provides LINQ query capabilities over `DataSet` and `DataTable`. You query `DataTable` objects by calling `AsEnumerable()` (from `DataTableExtensions`), which returns an `IEnumerable<DataRow>`. With a **typed DataSet**, property names are available at compile time and type mismatch errors are caught at compile time rather than at run time. LINQ to DataSet also provides `DataRowComparer` for value-based comparison of `DataRow` objects in set operations such as `Distinct`, `Union`, `Intersect`, and `Except`.

**Beginner-Friendly Explanation**
LINQ to DataSet is **LINQ for ADO.NET DataSets**. You call `table.AsEnumerable()` and then use LINQ.

---

### Purposes

- To query `DataSet` and `DataTable` with LINQ.
- To provide **type-safe access** to DataSet columns.
- To enable **set operations** on DataRows.
- To modernize ADO.NET data access with LINQ.

---

### Syntax Rules and Structure

```
// Untyped DataSet
var query = from row in dataTable.AsEnumerable()
            where row.Field<string>("City") == "London"
            select row;

// Typed DataSet (compile-time column names)
var query2 = from order in typedDataSet.Orders
             where order.OnlineOrderFlag == true
             select new { order.SalesOrderID, order.OrderDate };
```

**Component breakdown**

- `AsEnumerable()` — Converts `DataTable` to `IEnumerable<DataRow>`.
- `Field<T>("Column")` — Typed access to column values.
- `DataRowComparer.Default` — Value-based comparison for set operations.

#### Syntax Rules

- Requires `System.Data.DataSetExtensions` assembly.
- `AsEnumerable()` extension is on `DataTable`.
- Typed DataSets provide compile-time column access.

#### Constraints and Limitations

- `DataRow` comparisons default to reference equality; use `DataRowComparer` for value comparison.
- Not recommended for new development (prefer EF Core).

---

### Annotated Code Examples and Expected Outputs

#### Example: Querying a DataTable

```csharp
using System;
using System.Data;
using System.Linq;

class Program
{
    static void Main()
    {
        var table = new DataTable();
        table.Columns.Add("Name", typeof(string));
        table.Columns.Add("City", typeof(string));
        table.Rows.Add("Alice", "London");
        table.Rows.Add("Bob", "Paris");

        var londoners = from row in table.AsEnumerable()
                        where row.Field<string>("City") == "London"
                        select row.Field<string>("Name");

        foreach (var name in londoners)
            Console.WriteLine(name);   // Alice
    }
}
```

**Expected Output**

```
Alice
```

**Why this output occurs**

- LINQ to DataSet queries the `DataTable` in memory, filtering by city.

---

### Real-World Cases

- **Legacy ADO.NET applications** — Adding LINQ queries to existing DataSets.
- **Reporting** — Querying disconnected data.
- **Data transformation** — Converting DataRows to objects.

---

### References

- LINQ to DataSet — Microsoft Docs (PDF) - https://docs.microsoft.com/en-us/dotnet/opbuildpdf/framework/data/adonet/toc.pdf
- LINQ to DataSet — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb386943(v=vs.100)

---

## 10. Parallel LINQ (PLINQ)

### Definitions

**Core Definition**
**PLINQ (Parallel LINQ)** is a parallel implementation of LINQ to Objects that uses the `AsParallel()` extension method to execute queries across multiple threads and CPU cores.

**Technical Definition**
PLINQ is the parallel version of LINQ to Objects. Just as it is possible to execute a loop in parallel using `Parallel.For()`, it is also possible to execute LINQ queries in parallel using the Parallel LINQ API (PLINQ). You opt in by calling `AsParallel()` on an `IEnumerable<T>`, which returns a `ParallelQuery<T>`. PLINQ automatically partitions the source data, executes the query on multiple threads, and merges the results. The `System.Linq.ParallelEnumerable` class provides parallel versions of all standard query operators. PLINQ can provide **significant performance improvements** for CPU-bound queries on multi-core machines, but it also introduces overhead (partitioning, synchronization) and pitfalls (shared state, over-parallelization).

**Beginner-Friendly Explanation**
PLINQ is **LINQ that uses multiple CPU cores**. You add `.AsParallel()` and the query runs in parallel.

---

### Purposes

- To **speed up CPU-bound queries** on multi-core machines.
- To parallelize expensive per-element computations.
- To leverage multiple cores without manual threading.
- To process large datasets faster.

---

### Syntax Rules and Structure

```
var result = data
    .AsParallel()                     // opt in to PLINQ
    .WithDegreeOfParallelism(4)       // limit threads
    .Where(item => ExpensiveComputation(item))
    .Select(item => Transform(item))
    .ToList();
```

**Component breakdown**

- `AsParallel()` — Converts `IEnumerable<T>` to `ParallelQuery<T>`.
- `WithDegreeOfParallelism(int)` — Limits the number of threads.
- `AsOrdered()` — Preserves original ordering.
- `WithMergeOptions(...)` — Controls result buffering.
- `ForAll(...)` — Executes an action in parallel.

#### Syntax Rules

- PLINQ is **opt-in** via `AsParallel()`.
- Query operators return `ParallelQuery<T>` or `OrderedParallelQuery<T>`.
- `AsSequential()` reverts to sequential execution.
- PLINQ works best for CPU-bound, independent per-element operations.

#### Constraints and Limitations

- **Not always faster** — small datasets or fast delegates may see slowdowns due to overhead.
- **Avoid shared mutable state** — race conditions can occur.
- **Avoid over-parallelization** — nested PLINQ queries can hurt performance.
- **Avoid non-thread-safe methods** — `FileStream.Write` and similar APIs must not be called from PLINQ.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: PLINQ vs. Sequential LINQ

```csharp
using System;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        int[] data = Enumerable.Range(0, 10_000_000).ToArray();

        // Sequential
        var sw = Stopwatch.StartNew();
        var seq = data.Where(n => n % 2 == 0).Select(n => n * n).Sum();
        sw.Stop();
        Console.WriteLine($"Sequential: {sw.ElapsedMilliseconds} ms, Sum={seq}");

        // PLINQ
        sw.Restart();
        var par = data.AsParallel()
            .Where(n => n % 2 == 0)
            .Select(n => n * n)
            .Sum();
        sw.Stop();
        Console.WriteLine($"PLINQ: {sw.ElapsedMilliseconds} ms, Sum={par}");
    }
}
```

**Expected Output (approximate)**

```
Sequential: 120 ms, Sum=...
PLINQ: 35 ms, Sum=...
```

**Why this output occurs**

- PLINQ partitions the data and processes it on multiple threads; the CPU-bound computation benefits from parallelism.

---

#### Example 2: `WithDegreeOfParallelism`

```csharp
using System;
using System.Linq;

class Program
{
    static void Main()
    {
        var result = Enumerable.Range(0, 100)
            .AsParallel()
            .WithDegreeOfParallelism(4)
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

- `WithDegreeOfParallelism(4)` limits PLINQ to four threads; the query still produces the correct 50 even squares.

---

### Real-World Cases

- **Image processing** — Parallel pixel transformations.
- **Financial modeling** — Monte Carlo simulations.
- **Data analysis** — Parallel aggregation over large datasets.

---

### References

- Essential C#: Running LINQ Queries in Parallel — https://essentialcsharp.com/running-linq-queries-in-parallel
- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Parallel LINQ (PLINQ) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq

---

## 11. LINQ Best Practices

### Definitions

**Core Definition**
LINQ best practices are the guidelines that ensure LINQ queries are **correct, efficient, and maintainable**, avoiding common pitfalls such as multiple enumeration, premature materialization, and untranslatable expressions.

**Technical Definition**
Best practices for LINQ derive from Microsoft's documentation and community experience. Key recommendations include: **understand deferred execution** and its implications; **materialize once** with `ToList()`/`ToArray()` when reusing results; **filter before projecting** to reduce data transfer; **use `Any()` instead of `Count() > 0`** for existence checks; **avoid multiple enumerations** of deferred queries; **use `AsParallel()` only for CPU-bound, independent work**; **keep `IQueryable` queries translatable**; and **prefer query syntax for complex joins, method syntax for simple operations**.

**Beginner-Friendly Explanation**
LINQ best practices are the habits that keep queries fast: **materialize once**, **filter early**, **avoid repeated enumeration**, and **use PLINQ only when it helps**.

---

### Purposes

- To **avoid performance pitfalls** (multiple enumeration, premature materialization).
- To **choose the right execution model** (`IEnumerable` vs. `IQueryable`).
- To keep queries **readable and maintainable**.
- To **optimize database interactions**.
- To **use PLINQ responsibly**.

---

### Guidelines and Structure

#### General Guidelines

```
// 1. Materialize once when reusing results
var activeUsers = users.Where(u => u.IsActive).ToList();   // not multiple .Where() calls

// 2. Filter before projecting
var names = db.Users.Where(u => u.IsActive).Select(u => u.Name).ToList(); // not Select(...).Where(...)

// 3. Use Any() over Count() > 0
bool hasActive = users.Any(u => u.IsActive);   // not users.Count(u => u.IsActive) > 0

// 4. Avoid multiple enumerations
var query = users.Where(u => u.IsActive);
var list = query.ToList();   // materialize once
var count = list.Count;
var first = list.FirstOrDefault();

// 5. Keep IQueryable queries translatable
var good = db.Users.Where(u => u.Age > 30);   // translatable
// var bad = db.Users.Where(u => MyLocalMethod(u)); // not translatable

// 6. Use PLINQ only for CPU-bound work
var parallelResult = data.AsParallel().Where(n => ExpensiveCompute(n)).ToList();
```

**Component breakdown**

- **Materialize once** — Avoid re-executing deferred queries.
- **Filter early** — Reduce data before projection.
- **`Any()` for existence** — More efficient than `Count() > 0`.
- **Translatable `IQueryable`** — Keep provider-compatible operations.
- **PLINQ for CPU-bound** — Avoid I/O-bound or shared-state scenarios.

#### Rules and Constraints

- **Do** materialize once with `ToList()`/`ToArray()` when reusing results.
- **Do** filter before projecting to reduce data transfer.
- **Do** use `Any()` instead of `Count() > 0`.
- **Do** keep `IQueryable` queries translatable.
- **Do not** use PLINQ for I/O-bound or small datasets.
- **Do not** mutate shared state in PLINQ queries.
- **Do not** enumerate a deferred query multiple times.

---

### Annotated Code Examples and Expected Outputs

#### Example: Materialize Once

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
        var count = query.Count();            // enumeration 1
        var first = query.FirstOrDefault();    // enumeration 2

        // GOOD: materialize once
        var list = users.Where(u => u.IsActive).ToList();
        var count2 = list.Count;
        var first2 = list.FirstOrDefault();

        Console.WriteLine(count2);   // 2
        Console.WriteLine(first2?.Name); // Alice
    }
}
```

**Expected Output**

```
2
Alice
```

**Why this output occurs**

- The bad pattern enumerates twice; the good pattern materializes once and reuses the list.

---

### Real-World Cases

- **Database queries** — Materialize once to avoid repeated SQL execution.
- **PLINQ** — Use only for CPU-bound work; measure before committing.
- **API responses** — Filter before projecting to reduce payload size.

---

### References

- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851
- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| LINQ | Introduced in **.NET Framework 3.5 / C# 3.0**; stable. |
| LINQ to SQL | **Legacy**; .NET Framework only; superseded by EF Core. |
| LINQ to DataSet | **Legacy**; still available but not recommended for new code. |
| LINQ to Entities (EF Core) | **Current**; cross-platform, actively developed. |
| PLINQ | Introduced in **.NET Framework 4.0**; stable. |
| `AsParallel()` | Opt-in; **not always faster**; measure before using. |
| Deferred execution | **Language-defined**; most operators are lazy. |
| `IQueryable<T>` translation | **Provider-dependent**; not all methods are translatable. |
| `AsQueryable()` | For in-memory testing only; does not provide real translation. |

---

## References (Consolidated)

- Language Integrated Query (LINQ) — .NET - https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- LINQ (Language Integrated Query) — GitHub (raw) - https://raw.githubusercontent.com/dotnet/docs/main/docs/csharp/linq/linq-overview.md
- Querying in C# (ACM SIGPLAN Companion) - https://dl.acm.org/doi/10.1145/1297846.1297889
- LINQ: The Philosophy — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/archive/blogs/charlie/linq-the-philosophy
- What's the difference between Enumerable and Queryable — Microsoft Learn - https://learn.microsoft.com/en-us/archive/blogs/charlie/whats-the-difference-between-enumerable-and-queryable
- The Pain of Implementing LINQ Providers — Communications of the ACM - https://cacm.acm.org/magazines/2011/8/114944-the-pain-of-implementing-linq-providers/fulltext
- Use C# to write LINQ queries — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/get-started/write-linq-queries
- Query Syntax versus Method Syntax (C#) — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2010/bb397947(v=vs.100)
- Classification of Standard Query Operators by Manner of Execution — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb546142(v=vs.100)
- Query Execution — Microsoft Learn (archive) - https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/bb397906(v=vs.100)
- Query a collection of objects (LINQ in C#) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/csharp/linq/query-a-collection-of-objects
- LINQ to Objects — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/11-net-linq-to-objects.mdx
- LINQ to XML — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/dotnet/8-net-linq/12-net-linq-to-xml.mdx
- LINQ to XML overview — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/linq/linq-xml-overview
- C# LINQ to SQL — Compile-N-Run (GitHub) - https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/language/c-sharp/15-c-sharp-database-ac-sharpc-sharpess/8-c-sharp-linq-to-sql.mdx
- Entity Framework Core — ClickNLearn - https://mintlify.wiki/anandstays-creator/clicknlearn/dotnet/ef-core
- Entity Framework Core — Microsoft Learn - https://learn.microsoft.com/en-us/ef/core/
- LINQ to DataSet — Microsoft Docs (PDF) - https://docs.microsoft.com/en-us/dotnet/opbuildpdf/framework/data/adonet/toc.pdf
- Essential C#: Running LINQ Queries in Parallel — https://essentialcsharp.com/running-linq-queries-in-parallel
- Potential pitfalls with PLINQ — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-with-plinq
- Parallel LINQ (PLINQ) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq
- Are Your LINQ Queries Slowing Down Your App? — Syncfusion Blogs - https://www.syncfusion.com/blogs/post/optimize-linq-queries
- CA1851: Possible multiple enumerations of 'IEnumerable' collection — .NET - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1851

---