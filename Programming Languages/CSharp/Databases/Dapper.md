# Dapper — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Dapper is a lightweight, high-performance micro-ORM for .NET that extends `IDbConnection` with methods to execute SQL and map results to strongly typed objects.

**Technical Definition**
Dapper is a single-file library (distributed via the `Dapper` NuGet package) that provides extension methods on `IDbConnection` and `IDbTransaction`. It uses dynamically generated IL (via `System.Reflection.Emit`) to materialize objects from `IDataReader` rows, avoiding reflection per row. It supports parameterized queries, multi-mapping, multiple result sets, stored procedures, and a flexible mapping API. Dapper does not generate SQL, does not track changes, and does not manage schema — it is a thin, fast mapping layer over ADO.NET.

**Beginner-Friendly Explanation**
Dapper is a "helper" for talking to a database. You write the SQL yourself, and Dapper runs it and turns the rows into C# objects. It is much smaller and faster than a full O/RM like EF Core because it does not try to do everything — it just maps rows to objects very efficiently.

---

### Key Characteristics

- **Micro-ORM** — Focused on query execution and mapping; no change tracking, no migrations, no LINQ-to-SQL translation.
- **Raw SQL** — You write SQL; Dapper executes it with parameterization.
- **High performance** — Uses IL emit to map rows; benchmarks show it is close to hand-written ADO.NET.
- **Minimal API surface** — `Query`, `QueryFirst`, `QuerySingle`, `Execute`, `ExecuteScalar`, `QueryMultiple`, and related overloads.
- **Provider-agnostic** — Works with any ADO.NET provider (SQL Server, PostgreSQL, MySQL, SQLite, Oracle).
- **Composable** — Plays well with repositories, transactions, and existing ADO.NET code.
- **No schema management** — Migrations and model configuration are out of scope.

---

### Prerequisites

- Working knowledge of C# classes, generics, and LINQ (for in-memory operations).
- Basic SQL: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, joins, and parameters.
- Familiarity with ADO.NET concepts: `IDbConnection`, `IDbCommand`, `IDataReader`.
- A database provider package (e.g., `Microsoft.Data.SqlClient` or `Npgsql`).
- The `Dapper` NuGet package.

---

### Related Programming Areas

- **ADO.NET** — Dapper is built on top of ADO.NET abstractions.
- **Repository and unit-of-work patterns** — Dapper is often wrapped in repositories.
- **Stored procedures** — Dapper supports stored procedure execution and output parameters.
- **Bulk operations** — Dapper supports `Execute` with lists; bulk copy requires provider-specific APIs.
- **Testing** — In-memory databases (SQLite) and test containers.

---

### Core Concepts / Features

1. Dapper (overarching library)
2. Micro-ORM
3. Query execution
4. Mapping
5. Performance
6. Dapper best practices

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Dapper (Overarching Library)

### Definitions

**Core Definition**
Dapper is a NuGet library that adds fast, strongly typed query and mapping methods to `IDbConnection`.

**Technical Definition**
The `Dapper` package provides extension methods in the `Dapper` namespace. Key types include `SqlMapper` (the static entry point), `DynamicParameters` (for output and complex parameters), `CommandDefinition` (for command configuration), and mapping interfaces such as `ITypeMap` and `SqlMapper.TypeHandler<T>`. Dapper targets .NET Standard and runs on .NET Framework and modern .NET.

**Beginner-Friendly Explanation**
Dapper is "ADO.NET with less boilerplate." Instead of writing `using` blocks, commands, readers, and manual mapping for every query, you call `connection.Query<T>(sql, params)` and get objects back.

---

### Purposes

- To reduce ADO.NET boilerplate for common query and execute operations.
- To provide fast, strongly typed mapping of rows to objects.
- To allow developers to keep full control over SQL.
- To support multiple result sets and multi-mapping.
- To integrate cleanly with existing ADO.NET code and transactions.

---

### Syntax Rules and Structure

#### General Setup Syntax

```
// .csproj
<ItemGroup>
  <PackageReference Include="Dapper" Version="2.1.35" />
  <PackageReference Include="Microsoft.Data.SqlClient" Version="5.1.5" />
</ItemGroup>
```

```
// Basic usage
using Dapper;
using Microsoft.Data.SqlClient;

using var connection = new SqlConnection(connectionString);
var blogs = connection.Query<Blog>("SELECT Id, Name FROM Blogs").ToList();
```

**Component breakdown**

- `Dapper` package — Provides extension methods.
- Provider package — Supplies `IDbConnection` implementation.
- `using Dapper;` — Imports extension methods.
- `connection.Query<T>` — Executes SQL and maps rows to `T`.

#### Syntax Rules

- Dapper extension methods are in the `Dapper` namespace.
- The connection must be open before querying (Dapper opens closed connections automatically for most calls, but explicit opening is recommended for transactions).
- Parameter names in SQL should match object property names (case-insensitive).
- Dapper does not dispose the connection.

#### Constraints and Limitations

- Dapper does not generate SQL; you are responsible for correctness and safety.
- No change tracking, migrations, or LINQ translation.
- Provider-specific features (e.g., `SqlBulkCopy`) are outside Dapper's API.
- Dapper's mapping is convention-based; custom maps are required for complex cases.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Minimal Dapper Program

```csharp
using System;
using System.Linq;
using Dapper;
using Microsoft.Data.SqlClient;

public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

class Program
{
    static void Main()
    {
        var connectionString = "Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;";

        using var connection = new SqlConnection(connectionString);
        connection.Open();

        var blogs = connection.Query<Blog>(
            "SELECT Id, Name FROM Blogs WHERE Name LIKE @Pattern",
            new { Pattern = "A%" }).ToList();

        foreach (var blog in blogs)
            Console.WriteLine($"{blog.Id}: {blog.Name}");
    }
}
```

**Line-by-line comments**

- `Blog` — Simple POCO with properties matching column names.
- `SqlConnection` — Provider-specific connection.
- `connection.Open()` — Opens the connection explicitly.
- `Query<Blog>` — Executes the SQL with a parameter object.
- `@Pattern` — Parameter placeholder; `Pattern` property supplies the value.

**Expected Output (example)**

```
1: EF Core Intro
```

**Why this output occurs**

- The parameter is bound safely, preventing SQL injection.
- Rows are mapped to `Blog` objects by matching column names to properties.

---

### Real-World Cases

- **High-performance APIs** — Dapper is common in services where SQL is hand-tuned.
- **Legacy database integration** — Works with existing schemas and stored procedures.
- **Reporting and analytics** — Complex queries mapped to DTOs.

---

### References

- Dapper GitHub repository - https://github.com/DapperLib/Dapper
- Dapper NuGet package - https://www.nuget.org/packages/Dapper
- Dapper documentation (GitHub wiki) - https://github.com/DapperLib/Dapper/blob/main/README.md

---

## 2. Micro-ORM

### Definitions

**Core Definition**
A micro-ORM is a minimal object-relational mapper that focuses on query execution and object mapping without the broader feature set of a full O/RM.

**Technical Definition**
A micro-ORM typically provides methods to execute SQL, bind parameters, and map results to objects. It does not manage schema, change tracking, identity maps, or query generation. Dapper, PetaPoco, and Massive are examples. Micro-ORMs are often used alongside hand-written SQL and stored procedures.

**Beginner-Friendly Explanation**
A micro-ORM is a "small O/RM." It does one job well: run SQL and turn rows into objects. It does not try to manage your database schema or track changes.

---

### Purposes

- To provide a thin layer over ADO.NET without hiding SQL.
- To reduce boilerplate while keeping control over queries.
- To offer high performance with minimal abstraction overhead.
- To integrate easily with existing SQL and DBA workflows.
- To avoid the complexity and learning curve of full O/RMs.

---

### Syntax Rules and Structure

#### General Comparison

```
// ADO.NET (verbose)
using var cmd = connection.CreateCommand();
cmd.CommandText = "SELECT Id, Name FROM Blogs WHERE Id = @Id";
cmd.Parameters.Add(new SqlParameter("@Id", id));
using var reader = cmd.ExecuteReader();
var blogs = new List<Blog>();
while (reader.Read())
    blogs.Add(new Blog { Id = reader.GetInt32(0), Name = reader.GetString(1) });

// Dapper (concise)
var blogs = connection.Query<Blog>(
    "SELECT Id, Name FROM Blogs WHERE Id = @Id", new { Id = id }).ToList();
```

**Component breakdown**

- ADO.NET — Manual command, parameter, reader, and mapping.
- Dapper — Single method call with SQL and parameters.
- Both — Use the same underlying connection and provider.

#### Syntax Rules

- Micro-ORMs require you to write SQL.
- Parameterization is your responsibility (Dapper makes it easy).
- Mapping conventions determine property-to-column matching.
- Transactions are managed through the underlying connection.

#### Constraints and Limitations

- No schema evolution; use migrations or DBA scripts.
- No change tracking; updates are explicit.
- No LINQ-to-SQL translation; queries are strings.
- Security depends on parameterization discipline.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: ADO.NET vs. Dapper

```csharp
using System;
using System.Collections.Generic;
using Dapper;
using Microsoft.Data.SqlClient;

public class Blog { public int Id { get; set; } public string Name { get; set; } = ""; }

class Program
{
    static void Main()
    {
        using var connection = new SqlConnection("Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;");
        connection.Open();

        // ADO.NET
        var list1 = new List<Blog>();
        using (var cmd = connection.CreateCommand())
        {
            cmd.CommandText = "SELECT Id, Name FROM Blogs";
            using var reader = cmd.ExecuteReader();
            while (reader.Read())
                list1.Add(new Blog { Id = reader.GetInt32(0), Name = reader.GetString(1) });
        }

        // Dapper
        var list2 = connection.Query<Blog>("SELECT Id, Name FROM Blogs").ToList();

        Console.WriteLine($"{list1.Count} == {list2.Count}");
    }
}
```

**Expected Output**

```
1 == 1
```

**Why this output occurs**

- Both approaches execute the same query and produce the same number of objects.
- Dapper reduces boilerplate while preserving the result.

---

### Real-World Cases

- **Services with DBA-reviewed SQL** — Micro-ORMs keep SQL visible.
- **Performance-critical paths** — Minimal abstraction overhead.
- **Mixed codebases** — Dapper can be introduced incrementally alongside ADO.NET.

---

### References

- Dapper GitHub repository - https://github.com/DapperLib/Dapper
- Micro-ORM comparison (community) - https://github.com/DapperLib/Dapper#performance

---

## 3. Query Execution

### Definitions

**Core Definition**
Query execution in Dapper is the process of sending SQL to the database, binding parameters, and materializing results.

**Technical Definition**
Dapper's `SqlMapper` builds an `IDbCommand`, binds parameters (from anonymous objects, `DynamicParameters`, or `IDynamicParameters`), executes it, and consumes the `IDataReader`. It supports `Query<T>`, `QueryFirst<T>`, `QuerySingle<T>`, `QueryMultiple`, `Execute`, `ExecuteScalar<T>`, and async variants (`QueryAsync`, `ExecuteAsync`). `CommandDefinition` configures command timeout, transaction, command type, and cancellation.

**Beginner-Friendly Explanation**
Query execution is "running the SQL and getting results." Dapper handles the command setup, parameter binding, and reading rows so you can focus on the SQL and the objects.

---

### Purposes

- To execute SELECT queries and map results.
- To execute INSERT/UPDATE/DELETE and return affected row counts.
- To execute stored procedures and read output parameters.
- To read multiple result sets in one round trip.
- To support asynchronous execution for scalability.

---

### Syntax Rules and Structure

#### General Syntax

```
// Query: returns IEnumerable<T>
IEnumerable<T> Query<T>(string sql, object? param = null, IDbTransaction? transaction = null, bool buffered = true, int? commandTimeout = null, CommandType? commandType = null)

// QueryFirst: returns first row or throws
T QueryFirst<T>(string sql, object? param = null, ...)

// QueryFirstOrDefault: returns first row or default
T? QueryFirstOrDefault<T>(string sql, object? param = null, ...)

// QuerySingle: exactly one row or throws
T QuerySingle<T>(string sql, object? param = null, ...)

// QuerySingleOrDefault: zero or one row
T? QuerySingleOrDefault<T>(string sql, object? param = null, ...)

// Execute: returns affected row count
int Execute(string sql, object? param = null, ...)

// ExecuteScalar: returns single value
T? ExecuteScalar<T>(string sql, object? param = null, ...)

// QueryMultiple: multiple result sets
SqlMapper.GridReader QueryMultiple(string sql, object? param = null, ...)
```

**Component breakdown**

- `sql` — The SQL text or stored procedure name.
- `param` — Parameter object (anonymous type, `DynamicParameters`, dictionary).
- `transaction` — Optional transaction.
- `buffered` — Whether to buffer results (default `true`).
- `commandTimeout` — Timeout in seconds.
- `commandType` — `Text` or `StoredProcedure`.

#### Syntax Rules

- Use parameter placeholders (`@name` for SQL Server, `:name` for Oracle, etc.) and pass values via the parameter object.
- `QueryFirst` throws if no rows; `QueryFirstOrDefault` returns default.
- `QuerySingle` throws if zero or more than one row.
- `Execute` returns the number of affected rows.
- `QueryMultiple` returns a `GridReader` that must be disposed.

#### Constraints and Limitations

- `buffered: false` streams results; the reader must be fully consumed before the connection is reused.
- Async methods require provider support.
- `QueryMultiple` result sets must be read in order.
- Command timeout applies per command, not per batch.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Query, Execute, and ExecuteScalar

```csharp
using System;
using System.Linq;
using Dapper;
using Microsoft.Data.SqlClient;

public class Blog { public int Id { get; set; } public string Name { get; set; } = ""; }

class Program
{
    static void Main()
    {
        using var connection = new SqlConnection("Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;");
        connection.Open();

        // Execute: INSERT
        var affected = connection.Execute(
            "INSERT INTO Blogs (Name) VALUES (@Name)",
            new { Name = "Dapper Intro" });
        Console.WriteLine($"Inserted: {affected}");

        // ExecuteScalar: COUNT
        var count = connection.ExecuteScalar<int>("SELECT COUNT(*) FROM Blogs");
        Console.WriteLine($"Count: {count}");

        // Query: SELECT
        var blogs = connection.Query<Blog>("SELECT Id, Name FROM Blogs").ToList();
        foreach (var b in blogs)
            Console.WriteLine($"{b.Id}: {b.Name}");
    }
}
```

**Expected Output (example)**

```
Inserted: 1
Count: 1
1: Dapper Intro
```

**Why this output occurs**

- `Execute` returns the affected row count.
- `ExecuteScalar<int>` returns the first column of the first row.
- `Query<Blog>` maps rows to objects.

---

#### Example 2: Multiple Result Sets

```csharp
using var multi = connection.QueryMultiple(
    "SELECT Id, Name FROM Blogs; SELECT Id, Title FROM Posts;");

var blogs = multi.Read<Blog>().ToList();
var posts = multi.Read<Post>().ToList();

Console.WriteLine($"blogs={blogs.Count}, posts={posts.Count}");
```

**Expected Output (example)**

```
blogs=1, posts=2
```

**Why this output occurs**

- `QueryMultiple` executes both statements in one round trip.
- `Read<T>` consumes each result set in order.

---

#### Example 3: Stored Procedure with Output Parameter

```csharp
var p = new DynamicParameters();
p.Add("@Name", "SP Blog");
p.Add("@Id", dbType: System.Data.DbType.Int32, direction: System.Data.ParameterDirection.Output);

connection.Execute("usp_InsertBlog", p, commandType: System.Data.CommandType.StoredProcedure);
Console.WriteLine(p.Get<int>("@Id"));
```

**Expected Output (example)**

```
5
```

**Why this output occurs**

- `DynamicParameters` supplies input and captures output.
- `CommandType.StoredProcedure` executes the procedure.
- The output parameter is read after execution.

---

### Real-World Cases

- **CRUD APIs** — `Query` for reads, `Execute` for writes.
- **Stored procedure-heavy systems** — `DynamicParameters` for output and return values.
- **Dashboards** — `QueryMultiple` for multiple aggregates in one round trip.

---

### References

- Dapper README — Query, Execute, QueryMultiple - https://github.com/DapperLib/Dapper/blob/main/README.md
- Dapper API documentation (community) - https://github.com/DapperLib/Dapper

---

## 4. Mapping

### Definitions

**Core Definition**
Mapping is the process of converting database rows into CLR objects and vice versa.

**Technical Definition**
Dapper maps columns to properties by name (case-insensitive) using generated IL. It supports constructor mapping, custom type handlers (`SqlMapper.TypeHandler<T>`), custom type maps (`ITypeMap`), multi-mapping (`Query<T1, T2, TResult>`), and `splitOn` for joined results. It also supports `DynamicParameters` and `DapperRow` for dynamic results.

**Beginner-Friendly Explanation**
Mapping is "turning a row into an object." Dapper matches column names to property names automatically. If names differ or types are custom, you can tell Dapper how to map them.

---

### Purposes

- To materialize query results as strongly typed objects.
- To map joins to nested or related objects.
- To handle custom database types through type handlers.
- To support constructor-based mapping for immutable types.
- To map dynamic results when the shape is not known at compile time.

---

### Syntax Rules and Structure

#### General Syntax

```
// Simple mapping
var blogs = connection.Query<Blog>("SELECT Id, Name FROM Blogs");

// Multi-mapping (join)
var posts = connection.Query<Post, Blog, Post>(
    "SELECT p.Id, p.Title, b.Id, b.Name FROM Posts p JOIN Blogs b ON p.BlogId = b.Id",
    (post, blog) => { post.Blog = blog; return post; },
    splitOn: "Id");

// Custom type handler
SqlMapper.AddTypeHandler(new DateOnlyTypeHandler());

// Dynamic
var rows = connection.Query("SELECT Id, Name FROM Blogs");
foreach (IDictionary<string, object> row in rows)
    Console.WriteLine(row["Name"]);
```

**Component breakdown**

- `Query<T>` — Maps columns to `T` properties.
- `Query<T1, T2, TResult>` — Multi-mapping with a mapping function.
- `splitOn` — Column where the next object starts.
- `SqlMapper.AddTypeHandler` — Registers a custom handler.
- `Query` (non-generic) — Returns dynamic rows.

#### Syntax Rules

- Column names should match property names (case-insensitive).
- For multi-mapping, `splitOn` identifies the first column of the next object.
- Constructor mapping requires parameter names to match column names (or use `[ExplicitConstructor]`).
- Type handlers must be registered before use.

#### Constraints and Limitations

- Dapper does not map private setters by default (unless configured).
- Complex nested mappings may require custom logic.
- Multi-mapping with more than seven types requires special overloads.
- Custom type handlers are global per `AppDomain`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multi-Mapping

```csharp
using System;
using System.Linq;
using Dapper;
using Microsoft.Data.SqlClient;

public class Blog { public int Id { get; set; } public string Name { get; set; } = ""; }
public class Post { public int Id { get; set; } public string Title { get; set; } = ""; public Blog? Blog { get; set; } }

class Program
{
    static void Main()
    {
        using var connection = new SqlConnection("Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;");
        connection.Open();

        var posts = connection.Query<Post, Blog, Post>(
            @"SELECT p.Id, p.Title, b.Id AS BlogId, b.Name
              FROM Posts p JOIN Blogs b ON p.BlogId = b.Id",
            (post, blog) => { post.Blog = blog; return post; },
            splitOn: "BlogId").ToList();

        foreach (var p in posts)
            Console.WriteLine($"{p.Title} by {p.Blog?.Name}");
    }
}
```

**Expected Output (example)**

```
Dapper Intro by EF Core Intro
```

**Why this output occurs**

- `splitOn: "BlogId"` tells Dapper where the `Blog` columns start.
- The mapping function assigns the `Blog` to the `Post`.
- The nested navigation is printed.

---

#### Example 2: Custom Type Handler

```csharp
using System;
using System.Data;
using Dapper;

public class DateOnlyTypeHandler : SqlMapper.TypeHandler<DateOnly>
{
    public override DateOnly Parse(object value) => DateOnly.FromDateTime((DateTime)value);
    public override void SetValue(IDbDataParameter parameter, DateOnly value)
        => parameter.Value = value.ToDateTime(TimeOnly.MinValue);
}

SqlMapper.AddTypeHandler(new DateOnlyTypeHandler());
var dates = connection.Query<DateOnly>("SELECT CAST('2024-01-01' AS date)").ToList();
Console.WriteLine(dates[0]);
```

**Expected Output**

```
01/01/2024
```

**Why this output occurs**

- The handler converts database `date` to `DateOnly`.
- The query returns a `DateOnly` value.

---

### Real-World Cases

- **DTO projections** — Mapping query results to API DTOs.
- **Joins** — Mapping related objects in one query.
- **Custom types** — Mapping `DateOnly`, `TimeOnly`, `JsonDocument`, or spatial types.

---

### References

- Dapper README — Mapping and multi-mapping - https://github.com/DapperLib/Dapper/blob/main/README.md
- Dapper type handlers (community) - https://github.com/DapperLib/Dapper

---

## 5. Performance

### Definitions

**Core Definition**
Dapper performance is the speed and efficiency with which it executes queries and maps results.

**Technical Definition**
Dapper's performance comes from cached IL mapping, minimal allocations, and direct `IDataReader` consumption. Benchmarks (e.g., Dapper's own performance tests) show it is close to hand-written ADO.NET and significantly faster than reflection-based mappers for large result sets. Performance depends on query design, indexing, buffering, and connection management.

**Beginner-Friendly Explanation**
Performance is "how fast Dapper runs." Dapper is fast because it generates mapping code once and reuses it. But the database query itself and the indexes matter just as much.

---

### Purposes

- To minimize mapping overhead for large result sets.
- To reduce boilerplate without sacrificing speed.
- To support high-throughput APIs.
- To allow tuning through buffering and streaming.
- To avoid reflection-per-row costs.

---

### Syntax Rules and Structure

#### Performance Techniques

```
// Buffered (default): loads all rows into memory
var all = connection.Query<Blog>("SELECT * FROM Blogs");

// Unbuffered: streams rows
var streamed = connection.Query<Blog>("SELECT * FROM Blogs", buffered: false);

// Async
var asyncList = await connection.QueryAsync<Blog>("SELECT * FROM Blogs");

// Command timeout
var timed = connection.Query<Blog>("SELECT * FROM Blogs", commandTimeout: 30);

// Parameterized queries (plan reuse)
var filtered = connection.Query<Blog>("SELECT * FROM Blogs WHERE Name = @Name", new { Name = "A" });
```

**Component breakdown**

- `buffered: true` — Loads all rows; faster for small result sets.
- `buffered: false` — Streams rows; lower memory for large sets.
- Async methods — Avoid blocking threads.
- Parameters — Enable plan reuse and prevent injection.

#### Syntax Rules

- Use parameters for repeated queries to benefit from plan caching.
- Buffer small result sets; stream large ones.
- Avoid `SELECT *` when possible.
- Reuse connections through pooling.

#### Constraints and Limitations

- Unbuffered queries require the reader to be fully consumed before reusing the connection.
- Async performance depends on provider implementation.
- Dapper mapping is fast, but database round trips and query plans often dominate.
- Large object graphs may require multiple queries or multi-mapping.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Buffered vs. Unbuffered

```csharp
using System;
using System.Linq;
using Dapper;
using Microsoft.Data.SqlClient;

public class Blog { public int Id { get; set; } public string Name { get; set; } = ""; }

class Program
{
    static void Main()
    {
        using var connection = new SqlConnection("Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;");
        connection.Open();

        var buffered = connection.Query<Blog>("SELECT Id, Name FROM Blogs").ToList();
        var streamed = connection.Query<Blog>("SELECT Id, Name FROM Blogs", buffered: false).ToList();

        Console.WriteLine($"buffered={buffered.Count}, streamed={streamed.Count}");
    }
}
```

**Expected Output**

```
buffered=1, streamed=1
```

**Why this output occurs**

- Both methods return the same rows.
- Buffering affects memory usage, not the result count.

---

#### Example 2: Parameterized Query Plan Reuse

```csharp
for (int i = 0; i < 100; i++)
{
    connection.Execute(
        "INSERT INTO Logs (Message) VALUES (@Message)",
        new { Message = $"entry {i}" });
}
```

**Expected Output (conceptual)**

```
100 rows inserted; one cached query plan reused
```

**Why this output occurs**

- The same SQL text and parameter shape allow plan reuse.
- Dapper caches the command preparation per SQL/parameter shape.

---

### Real-World Cases

- **High-throughput APIs** — Dapper's low overhead is a key advantage.
- **Batch processing** — Streaming large result sets.
- **Read-heavy services** — Parameterized queries and connection pooling.

---

### References

- Dapper performance benchmarks - https://github.com/DapperLib/Dapper#performance
- Dapper README — Buffered and unbuffered - https://github.com/DapperLib/Dapper/blob/main/README.md

---

## 6. Dapper Best Practices

### Definitions

**Core Definition**
Dapper best practices are recommended patterns for using Dapper safely, efficiently, and maintainably.

**Technical Definition**
Best practices cover parameterization, connection lifetime, transaction management, mapping conventions, async usage, batching, and testing. They are supported by community guidance and Dapper's design.

**Beginner-Friendly Explanation**
Best practices are "the habits that keep Dapper code fast, safe, and easy to maintain."

---

### Purposes

- To prevent SQL injection through parameterization.
- To avoid connection leaks and transaction issues.
- To keep mapping predictable.
- To improve performance through batching and streaming.
- To make code testable and maintainable.

---

### Guidelines and Structure

#### Parameterization

```
// Good: parameterized
connection.Query<Blog>("SELECT * FROM Blogs WHERE Name = @Name", new { Name = name });

// Bad: string concatenation (SQL injection risk)
connection.Query<Blog>($"SELECT * FROM Blogs WHERE Name = '{name}'");
```

- Always use parameters for user input.
- Use `DynamicParameters` for stored procedures and output parameters.

#### Connection Lifetime

```
using var connection = new SqlConnection(connectionString);
connection.Open();
// use connection
// disposed at end of scope
```

- Create connections per unit of work.
- Rely on connection pooling; do not cache `SqlConnection` instances.
- Dispose connections and readers promptly.

#### Transactions

```
using var connection = new SqlConnection(connectionString);
connection.Open();
using var transaction = connection.BeginTransaction();

try
{
    connection.Execute("INSERT ...", param, transaction);
    connection.Execute("UPDATE ...", param, transaction);
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

- Pass the transaction to every Dapper call in the unit of work.
- Roll back on exceptions.

#### Async

```
var blogs = await connection.QueryAsync<Blog>("SELECT Id, Name FROM Blogs");
await connection.ExecuteAsync("INSERT ...", new { Name = "A" });
```

- Prefer async methods in scalable services.
- Avoid mixing sync and async on the same connection concurrently.

#### Mapping

```
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}
```

- Keep property names aligned with column names (or use aliases).
- Use DTOs for projections.
- Register type handlers once at startup.

#### Testing

```
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open();
connection.Execute("CREATE TABLE Blogs (Id INTEGER PRIMARY KEY, Name TEXT)");
```

- Use SQLite in-memory or test containers for integration tests.
- Test SQL against the real provider when possible.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Safe Transactional Write

```csharp
using System;
using Dapper;
using Microsoft.Data.SqlClient;

class Program
{
    static void Main()
    {
        using var connection = new SqlConnection("Server=.;Database=Blogging;Trusted_Connection=True;TrustServerCertificate=True;");
        connection.Open();
        using var tx = connection.BeginTransaction();

        try
        {
            connection.Execute("INSERT INTO Blogs (Name) VALUES (@Name)", new { Name = "A" }, tx);
            connection.Execute("INSERT INTO Blogs (Name) VALUES (@Name)", new { Name = "B" }, tx);
            tx.Commit();
            Console.WriteLine("Committed");
        }
        catch (Exception ex)
        {
            tx.Rollback();
            Console.WriteLine($"Rolled back: {ex.Message}");
        }
    }
}
```

**Expected Output**

```
Committed
```

**Why this output occurs**

- Both inserts participate in the same transaction.
- The transaction is committed only if both succeed.

---

#### Example 2: Async Repository Method

```csharp
public class BlogRepository
{
    private readonly string _connectionString;
    public BlogRepository(string connectionString) => _connectionString = connectionString;

    public async Task<List<Blog>> GetBlogsAsync()
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        var blogs = await connection.QueryAsync<Blog>("SELECT Id, Name FROM Blogs");
        return blogs.ToList();
    }
}
```

**Expected Output (conceptual)**

```
Returns the list of blogs without blocking the calling thread
```

**Why this output occurs**

- `OpenAsync` and `QueryAsync` use asynchronous I/O.
- The connection is disposed after the method completes.

---

### Real-World Cases

- **ASP.NET Core services** — Scoped repositories with async Dapper calls.
- **Background workers** — Batched writes and streaming reads.
- **Data migration tools** — Transactions and parameterized bulk operations.

---

### References

- Dapper GitHub repository - https://github.com/DapperLib/Dapper
- Dapper README — Parameterization and transactions - https://github.com/DapperLib/Dapper/blob/main/README.md
- Microsoft.Data.SqlClient documentation - https://learn.microsoft.com/en-us/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| String concatenation in SQL | Unsafe; SQL injection risk. Always parameterize. |
| `buffered: false` | Requires full consumption before connection reuse; not suitable for all scenarios. |
| `Query` (non-generic) | Returns `dynamic`; loses compile-time safety. |
| Type handlers | Global per `AppDomain`; registration order matters. |
| Multi-mapping with >7 types | Requires special overloads; complex mapping. |
| `CommandDefinition` | Version-specific features (cancellation, flags) depend on Dapper version. |
| Async support | Provider-dependent; not all providers implement true async I/O. |
| Bulk copy | Not part of Dapper; use provider-specific `SqlBulkCopy` or `COPY`. |
| `QueryFirst` vs. `QueryFirstOrDefault` | `QueryFirst` throws on no rows; choose deliberately. |
| Connection caching | Do not cache `SqlConnection` instances; rely on pooling. |

---

## References (Consolidated)

- Dapper GitHub repository - https://github.com/DapperLib/Dapper
- Dapper NuGet package - https://www.nuget.org/packages/Dapper
- Dapper README and documentation - https://github.com/DapperLib/Dapper/blob/main/README.md
- Microsoft.Data.SqlClient documentation - https://learn.microsoft.com/en-us/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace
- ADO.NET overview - https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview
- SQL injection prevention - https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection
- Connection pooling - https://learn.microsoft.com/en-us/sql/connect/ado-net/sql-server-connection-pooling
- Async programming in C# - https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/
- Dapper performance benchmarks (community) - https://github.com/DapperLib/Dapper#performance