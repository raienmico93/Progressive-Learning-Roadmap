# Dapper: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **Dapper**, the lightweight, high‑performance micro‑ORM for .NET. Examples use C# with `IDbConnection` extensions and assume a modern .NET version (6–10). Dapper is not a full ORM like EF Core—it **does not** provide change tracking, migrations, or LINQ translation. It extends ADO.NET with simple object mapping and is designed for scenarios where **SQL control and performance** are paramount. The core API is stable across versions; `QueryUnbufferedAsync` and AOT‑compatibility features are more recent additions. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dapper is a simple object mapper for .NET that extends `IDbConnection` with high‑performance query and execute methods, mapping SQL results directly to POCOs. |
| **Technical Definition** | Dapper is a **micro‑ORM** that provides a minimal abstraction over ADO.NET. It adds extension methods to `IDbConnection` (`Query`, `QueryFirst`, `QuerySingle`, `Execute`, `QueryMultiple`, etc.) that handle **parameterization**, **result materialization**, and **multi‑mapping** without the overhead of a full ORM. It does **not** include change tracking, lazy loading, migrations, or a LINQ provider. |
| **Beginner‑Friendly Explanation** | Dapper is the **“fast and simple” database helper**. You write your own SQL, and Dapper handles the tedious part: turning the rows the database returns into C# objects. It is much lighter than EF Core—no hidden query generation, no change tracker—which makes it very fast, but it also means you are responsible for writing correct, parameterized SQL. |

### Key Characteristics

- **Micro‑ORM** – Minimal abstraction over ADO.NET; fast and lightweight.
- **SQL‑first** – You write the SQL; Dapper maps the results.
- **Parameterized by default** – Prevents SQL injection when used correctly.
- **Multi‑mapping** – Maps a single row to multiple objects via `splitOn`.
- **Async support** – `QueryAsync`, `ExecuteAsync`, and related methods.
- **No change tracking** – Updates and deletes require explicit SQL.
- **No migrations** – Schema management is external to Dapper.

### Prerequisites

- .NET SDK (6.0+ recommended), basic C# knowledge.
- A database provider (e.g., `Microsoft.Data.SqlClient`, `Npgsql`, `MySqlConnector`).
- Familiarity with **ADO.NET** (`IDbConnection`, `IDbCommand`) and **SQL**.
- Basic understanding of **POCOs** and **parameterized queries**.

### Related Programming Areas

- **ADO.NET** – Dapper is built on top of ADO.NET.
- **Entity Framework Core** – A full ORM alternative with different trade‑offs.
- **SQL** – Dapper requires manual SQL writing.
- **Repository pattern** – Dapper is often used inside repositories.
- **Microservices** – Dapper’s performance and explicit SQL suit microservice data access.

### Core Concepts / Features

1. **Dapper** – The library and its role in the .NET data‑access ecosystem.
2. **Micro‑ORM** – The category of lightweight data mappers.
3. **Query execution** – `Query`, `QueryFirst`, `QuerySingle`, `Execute`, and `QueryMultiple`.
4. **Mapping** – How Dapper maps columns to properties, including multi‑mapping with `splitOn`.
5. **Best practices** – Parameterization, connection management, async usage, and performance.

---

## 1. Dapper (The Library)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dapper is a NuGet library that provides extension methods on `IDbConnection` for executing SQL and mapping the results to .NET objects. |
| **Technical Definition** | Dapper is a **micro‑ORM** that sits between raw ADO.NET and a full ORM. It extends `IDbConnection` with methods that handle parameter binding, result materialization, and multi‑mapping. It is **database‑agnostic** because it works with any ADO.NET provider. Dapper is developed and maintained by the .NET community (originally by Stack Overflow). |
| **Beginner‑Friendly Explanation** | Dapper is the **tool you add to your project** to make database access easier than raw ADO.NET but lighter than EF Core. You install it via NuGet, open a connection, and call `connection.Query<User>("SELECT * FROM Users")`—Dapper does the rest. |

### Purposes

- **To** provide **high‑performance** data access without the overhead of a full ORM.
- **To** map SQL query results directly to **POCOs** with minimal configuration.
- **To** give developers **full control over SQL** for complex queries and optimizations.
- **To** support **stored procedures**, **multi‑mapping**, and **multiple result sets**.
- **To** remain **database‑agnostic** through ADO.NET compatibility.

### Syntax Rules and Structure

#### Installation

```bash
dotnet add package Dapper
# Plus your database provider, e.g.:
dotnet add package Microsoft.Data.SqlClient   # SQL Server
dotnet add package Npgsql                      # PostgreSQL
```

#### General Syntax (Basic Query)

```csharp
using Dapper;
using System.Data.SqlClient;

using var conn = new SqlConnection(connectionString);
var users = await conn.QueryAsync<User>("SELECT Id, Name, Email FROM Users");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IDbConnection` | Any ADO.NET connection (`SqlConnection`, `NpgsqlConnection`, etc.). |
| `QueryAsync<T>` | Executes the SQL and maps each row to `T`. |
| SQL string | Raw SQL with parameter placeholders (`@Name`). |
| Parameter object | Anonymous object or `DynamicParameters` (optional). |

#### Syntax Rules

- **Connections are not automatically opened** – Dapper opens a closed connection but does not close it if it opened it. Use `using` to dispose connections.
- **Parameter names** in SQL (`@Name`) must match the properties of the parameter object.
- **Column names** must match property names (case‑insensitive by default, but exact match is safest).
- **Dapper does not track changes** – updates and deletes are explicit SQL.

#### Constraints and Limitations

- **No LINQ** – queries are raw SQL strings.
- **No migrations** – schema management is external.
- **No lazy loading** – related data must be eagerly fetched via multi‑mapping or separate queries.
- **Manual SQL** – developer responsible for correctness and parameterization.

### Annotated Code Example (Basic Query with Parameters)

```csharp
public async Task<User?> GetUserByIdAsync(int id)
{
    using var conn = new SqlConnection(_connectionString);
    return await conn.QueryFirstOrDefaultAsync<User>(
        "SELECT Id, Name, Email FROM Users WHERE Id = @Id",
        new { Id = id });
}
```

**Expected Output:** The `User` with the specified `Id`, or `null` if no such user exists.

**Why this output?** `QueryFirstOrDefaultAsync` executes the parameterized SQL and maps the first row to a `User` instance; if no rows are returned, it returns `null`.

### Real-World Cases

- **Read‑heavy APIs:** Dapper is often faster than EF Core for read‑only queries because it skips change tracking and query translation.
- **Reporting and dashboards:** Complex SQL with joins and aggregations is easier to express directly.
- **Legacy databases:** Dapper works with any ADO.NET provider, including older or non‑standard databases.
- **Microservices:** Lightweight data access with explicit SQL suits small, focused services.

### References

- Dapper GitHub repository – https://github.com/DapperLib/Dapper
- Dapper official documentation (GitHub wiki) – https://github.com/DapperLib/Dapper/blob/main/Readme.md
- How to Work With Dapper in .Net (freeCodeCamp) – https://www.freecodecamp.org/news/how-to-work-with-dapper-in-net/

---

## 2. Micro‑ORM (The Category)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A micro‑ORM is a lightweight object‑relational mapper that provides only core mapping and execution services, deliberately omitting the full feature set of a traditional ORM. |
| **Technical Definition** | A micro‑ORM like Dapper offers **core parameterization and materialization services**, but—by design—not the full breadth of services provided by a full ORM (change tracking, lazy loading, migrations, LINQ, identity map, unit of work). It sits between raw ADO.NET and a full ORM, giving developers SQL control with less boilerplate than ADO.NET. |
| **Beginner‑Friendly Explanation** | A micro‑ORM is the **“middle ground”** between writing raw ADO.NET code and using a full ORM. It handles the boring part—mapping rows to objects—but leaves you in charge of the SQL. It is smaller and faster than a full ORM, but you give up conveniences like automatic change tracking and migrations. |

### Purposes

- **To** provide **essential ORM features** (parameterization, mapping) without the overhead of a full ORM.
- **To** give developers **full control over SQL** while automating result materialization.
- **To** offer **high performance** for read‑heavy or performance‑sensitive workloads.
- **To** serve as a **lightweight alternative** when EF Core’s abstractions are unnecessary or counterproductive.

### Syntax Rules and Structure

#### Comparison: Micro‑ORM vs. Full ORM

| Feature | Micro‑ORM (Dapper) | Full ORM (EF Core) |
|---------|--------------------|---------------------|
| **Query language** | Raw SQL | LINQ + raw SQL |
| **Change tracking** | No | Yes |
| **Migrations** | No | Yes |
| **Lazy loading** | No | With proxies |
| **Performance** | Very high (close to ADO.NET) | Good, with more overhead |
| **Learning curve** | Low (SQL + small API) | Higher (LINQ + patterns) |
| **Database agnostic** | Yes (any ADO.NET provider) | Yes (provider packages) |

**Syntax rules:**

- **Micro‑ORMs are not replacements for full ORMs** in all scenarios—they are a specialized tool for SQL‑centric, performance‑sensitive data access.
- **The choice depends on workload characteristics**—read‑heavy, SQL‑controlled workloads favor Dapper; CRUD‑heavy, relationship‑rich workloads may favor EF Core.

**Constraints and limitations:**

- **No change tracking** means more manual SQL for updates and deletes.
- **No migrations** means schema changes must be managed separately.
- **No LINQ** means complex queries must be written in SQL.

### Annotated Code Example (Choosing the Right Tool)

```csharp
// Dapper: ideal for a complex reporting query
public async Task<List<SalesReport>> GetSalesReportAsync(DateTime start, DateTime end)
{
    const string sql = @"
        SELECT c.Name AS CustomerName, SUM(o.Total) AS TotalSales
        FROM Customers c
        JOIN Orders o ON o.CustomerId = c.Id
        WHERE o.CreatedAt BETWEEN @Start AND @End
        GROUP BY c.Name
        ORDER BY TotalSales DESC";

    using var conn = new SqlConnection(_connectionString);
    return (await conn.QueryAsync<SalesReport>(sql, new { Start = start, End = end })).AsList();
}
```

**Expected Output:** A list of `SalesReport` objects with customer names and total sales for the period.

**Why this output?** Dapper maps the SQL result set directly to `SalesReport` objects; the developer controls the exact SQL, including joins and aggregation.

### Real-World Cases

- **High‑performance APIs:** Dapper reduces overhead for read‑heavy endpoints.
- **Complex reporting:** SQL with multiple joins and window functions is easier to express directly.
- **Legacy system integration:** Dapper works with existing ADO.NET providers and stored procedures.
- **Polyglot persistence:** Dapper can target different databases with the same API.

### References

- Dapper is a micro‑ORM (Stack Overflow) – https://stackoverflow.com/revisions/6d67ba87-6ddb-495e-ac23-22484d39e360/view-source
- Dapper vs. Entity Framework Core: Performance & Use Cases (Devart) – https://www.devart.com/

---

## 3. Query Execution (The API)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Query execution in Dapper refers to the set of extension methods that execute SQL commands and return results or affected row counts. |
| **Technical Definition** | Dapper provides synchronous and asynchronous variants of several core methods: **`Execute`** (for INSERT/UPDATE/DELETE, returns affected rows), **`Query`** (returns `IEnumerable<T>`), **`QueryFirst`** (first row, throws if empty), **`QueryFirstOrDefault`** (first row or `null`), **`QuerySingle`** (exactly one row, throws if zero or multiple), **`QuerySingleOrDefault`** (one row or `null`), and **`QueryMultiple`** (multiple result sets). Each method accepts an SQL string, optional parameters, and optional transaction and command type. |
| **Beginner‑Friendly Explanation** | Query execution is **how you talk to the database with Dapper**. You pick the method that matches your intent: `Execute` for changes, `Query` for lists, `QueryFirst` for the first row, `QuerySingle` for exactly one row, and `QueryMultiple` for several result sets in one round‑trip. |

### Purposes

- **To** execute **SELECT** queries and map results to objects (`Query`, `QueryFirst`, `QuerySingle`).
- **To** execute **INSERT/UPDATE/DELETE** statements and return affected row counts (`Execute`).
- **To** retrieve **multiple result sets** in a single database round‑trip (`QueryMultiple`).
- **To** support **stored procedures** via `commandType: CommandType.StoredProcedure`.
- **To** provide **async** variants for scalable server applications.

### Syntax Rules and Structure

#### Core Methods

| Method | Purpose | Return Type |
|--------|---------|-------------|
| `Execute` / `ExecuteAsync` | INSERT/UPDATE/DELETE | `int` (rows affected) |
| `Query<T>` / `QueryAsync<T>` | SELECT list | `IEnumerable<T>` |
| `QueryFirst<T>` / `QueryFirstAsync<T>` | First row (throws if empty) | `T` |
| `QueryFirstOrDefault<T>` / `QueryFirstOrDefaultAsync<T>` | First row or `null` | `T?` |
| `QuerySingle<T>` / `QuerySingleAsync<T>` | Exactly one row (throws if zero or multiple) | `T` |
| `QuerySingleOrDefault<T>` | One row or `null` (throws if multiple) | `T?` |
| `QueryMultiple` / `QueryMultipleAsync` | Multiple result sets | `SqlMapper.GridReader` |

#### General Syntax (Execute)

```csharp
public async Task<int> UpdateUserAsync(User user)
{
    const string sql = "UPDATE Users SET Name = @Name WHERE Id = @Id";
    using var conn = new SqlConnection(_connectionString);
    return await conn.ExecuteAsync(sql, new { user.Name, user.Id });
}
```

**Expected Output:** The number of rows affected (typically `1` for a successful update).

**Why this output?** `ExecuteAsync` runs the parameterized UPDATE and returns the affected row count.

#### General Syntax (QueryMultiple)

```csharp
using var multi = await conn.QueryMultipleAsync(@"
    SELECT * FROM Users;
    SELECT * FROM Orders;");

var users = multi.Read<User>().AsList();
var orders = multi.Read<Order>().AsList();
```

**Expected Output:** Two lists (`users` and `orders`) populated from a single database call.

**Why this output?** `QueryMultipleAsync` executes both statements and returns a `GridReader` from which each result set is read sequentially.

**Syntax rules:**

- **Use `AsList()`** to materialize the results and avoid multiple enumeration.
- **`QueryFirst` throws** if the result set is empty; use `QueryFirstOrDefault` for nullable results.
- **`QuerySingle` throws** if the result set has zero or more than one row.
- **`commandType: CommandType.StoredProcedure`** is required for stored procedure calls.

**Constraints and limitations:**

- **`QueryAsync` with `buffered: false` streams synchronously** – for true async streaming, use `QueryUnbufferedAsync` (newer API).
- **Multiple result sets** require `QueryMultiple`; separate `Query` calls create separate round‑trips.

### Annotated Complete Step-by-Step Code Example

```csharp
public class UserRepository
{
    private readonly string _connectionString;

    public UserRepository(string connectionString) => _connectionString = connectionString;

    // Execute: INSERT
    public async Task<int> CreateAsync(User user)
    {
        const string sql = @"
            INSERT INTO Users (Name, Email)
            VALUES (@Name, @Email);
            SELECT CAST(SCOPE_IDENTITY() AS INT);";

        using var conn = new SqlConnection(_connectionString);
        return await conn.ExecuteScalarAsync<int>(sql, user);
    }

    // Query: list
    public async Task<List<User>> GetActiveAsync()
    {
        using var conn = new SqlConnection(_connectionString);
        return (await conn.QueryAsync<User>(
            "SELECT Id, Name, Email FROM Users WHERE IsActive = 1")).AsList();
    }

    // QueryFirstOrDefault: single row or null
    public async Task<User?> FindByEmailAsync(string email)
    {
        using var conn = new SqlConnection(_connectionString);
        return await conn.QueryFirstOrDefaultAsync<User>(
            "SELECT Id, Name, Email FROM Users WHERE Email = @Email",
            new { Email = email });
    }
}
```

**Step-by-step explanation:**

1. `CreateAsync` uses `ExecuteScalarAsync` to insert and return the new identity.
2. `GetActiveAsync` uses `QueryAsync` to retrieve a list and materializes it with `AsList`.
3. `FindByEmailAsync` uses `QueryFirstOrDefaultAsync` to return a single user or `null`.

**Expected Output:** `CreateAsync` returns the new user ID; `GetActiveAsync` returns a list of active users; `FindByEmailAsync` returns the matching user or `null`.

**Why this output?** Each method matches the intent: scalar for a single value, query for a list, first‑or‑default for a single nullable result.

### Real-World Cases

- **CRUD APIs:** `ExecuteAsync` for inserts/updates, `QueryAsync` for lists, `QueryFirstOrDefaultAsync` for detail endpoints.
- **Reporting:** `QueryAsync` with complex SQL and projection DTOs.
- **Batch operations:** `ExecuteAsync` with an `IEnumerable` of parameter objects (Dapper batches automatically).
- **Stored procedures:** `QueryAsync`/`ExecuteAsync` with `CommandType.StoredProcedure`.

### References

- Dapper API reference (GitHub) – https://github.com/DapperLib/Dapper
- Dapper ORM Guide: Queries, Mapping & Async (Red Gate) – https://www.red-gate.com/simple-talk/development/dotnet-development/a-practical-guide-to-dapper/

---

## 4. Mapping (Columns to Objects)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Mapping is the process by which Dapper converts database rows and columns into .NET objects and properties. |
| **Technical Definition** | Dapper performs **convention‑based mapping**: column names in the result set are matched to public property names on the target type (case‑insensitive by default). It supports **simple mapping** (one row → one object), **multi‑mapping** (one row → multiple objects via `splitOn`), and **custom type handlers** (for types that need special conversion). Column order in the SELECT statement matters for multi‑mapping. |
| **Beginner‑Friendly Explanation** | Mapping is **how Dapper knows which column goes into which property**. If your SQL returns a column called `Name`, Dapper looks for a public `Name` property on your class and sets it. For joins, you tell Dapper where to “split” the row into multiple objects using `splitOn`. |

### Purposes

- **To** automatically convert database results into **strongly typed .NET objects**.
- **To** support **complex object graphs** through multi‑mapping (JOINs).
- **To** handle **type conversions** via custom type handlers when the database type does not map directly.
- **To** reduce boilerplate compared to manual `IDataReader` mapping.

### Syntax Rules and Structure

#### Simple Mapping

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}

var users = await conn.QueryAsync<User>(
    "SELECT Id, Name, Email FROM Users");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `User` | POCO with public properties. |
| `SELECT Id, Name, Email` | Column names match property names. |
| Mapping | Dapper matches columns to properties by name. |

#### Multi‑Mapping (JOINs)

```csharp
public class Order { public int Id { get; set; } public int UserId { get; set; } public User User { get; set; } = null!; }
public class User { public int Id { get; set; } public string Name { get; set; } = ""; }

const string sql = @"
    SELECT o.Id AS OrderId, o.UserId, u.Id AS UserId, u.Name
    FROM Orders o
    JOIN Users u ON o.UserId = u.Id";

var orders = await conn.QueryAsync<Order, User, Order>(
    sql,
    (order, user) => { order.User = user; return order; },
    splitOn: "UserId");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `QueryAsync<Order, User, Order>` | Maps two types (`Order`, `User`) and returns `Order`. |
| Lambda `(order, user) => { ... }` | Combines the mapped objects. |
| `splitOn: "UserId"` | The column where the second type begins. |

**Syntax rules:**

- **Column order matters** – columns for `TFirst` must come first, then the `splitOn` column, then `TSecond` columns.
- **`splitOn`** specifies the column name(s) where the result set splits. If multiple types are mapped, there should be one splitting column per additional type.
- **When all splitting columns are named `id`**, a single `"id"` can be used for `splitOn`.
- **Dapper does not create instances for complex types when all columns for that type are `NULL`**.

**Constraints and limitations:**

- **No lazy loading** – related data must be fetched in the same query or with explicit subsequent queries.
- **No identity resolution** – Dapper does not deduplicate parent objects across rows (you may need a custom helper for one‑to‑many).
- **Column aliases are often required** – when joining tables with overlapping column names (`Id`), use `AS` aliases and reference them in `splitOn`.

### Annotated Complete Step-by-Step Code Example

```csharp
public async Task<List<Order>> GetOrdersWithUsersAsync()
{
    const string sql = @"
        SELECT o.Id AS OrderId, o.Total,
               u.Id AS UserId, u.Name
        FROM Orders o
        INNER JOIN Users u ON o.UserId = u.Id";

    using var conn = new SqlConnection(_connectionString);
    var orders = await conn.QueryAsync<Order, User, Order>(
        sql,
        (order, user) =>
        {
            order.User = user;
            return order;
        },
        splitOn: "UserId");

    return orders.AsList();
}
```

**Step-by-step explanation:**

1. The SQL aliases `o.Id` as `OrderId` and `u.Id` as `UserId` to avoid name collisions.
2. `QueryAsync<Order, User, Order>` tells Dapper to map the first group of columns to `Order` and the second group to `User`.
3. The `splitOn: "UserId"` parameter tells Dapper that the `User` columns begin at `UserId`.
4. The lambda assigns the mapped `User` to `Order.User`.

**Expected Output:** A list of `Order` objects, each with its associated `User` populated.

**Why this output?** Dapper splits the row at `UserId`, maps the columns before it to `Order`, maps the remaining columns to `User`, and invokes the lambda to combine them.

### Real-World Cases

- **Order/User joins:** Mapping an order and its customer in one query.
- **One‑to‑many relationships:** Multi‑mapping with a custom helper to deduplicate parent objects.
- **Custom type handlers:** Mapping `Guid` to `string`, `DateOnly` to `DateTime`, or JSON columns to objects.

### References

- Dapper multi‑mapping and splitOn (Stack Overflow) – https://stackoverflow.com/revisions/79613209/2
- Dapper relationships (LearnDapper) – https://github.com/zzzprojects/docs/blob/master/learndapper.com/pages/relationships.md

---

## 5. Dapper Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dapper best practices are guidelines for using Dapper safely, efficiently, and maintainably. |
| **Technical Definition** | Best practices encompass **parameterized queries** (SQL injection prevention), **connection management** (using `using` blocks), **async usage** (`QueryAsync`/`ExecuteAsync` with `ConfigureAwait(false)`), **buffered vs. unbuffered queries**, **multi‑mapping discipline** (column aliases, `splitOn`), **transaction management**, and **connection pooling**. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for Dapper—they help you avoid SQL injection, connection leaks, thread‑pool starvation, and inefficient queries. |

### Purposes

- **To** prevent **SQL injection** through parameterized queries.
- **To** avoid **connection leaks** by disposing connections properly.
- **To** maintain **scalability** through async I/O and connection pooling.
- **To** ensure **correct mapping** in multi‑table joins.
- **To** manage **transactions** safely across multiple operations.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Parameterization** | Use `@Parameter` placeholders and pass an object or `DynamicParameters`. | Concatenate user input into SQL strings (SQL injection risk). |
| **Connection management** | Wrap connections in `using` blocks; let Dapper open closed connections. | Leave connections open or fail to dispose them. |
| **Async** | Use `QueryAsync`, `ExecuteAsync` with `ConfigureAwait(false)`. | Use synchronous `Query`/`Execute` in async contexts (thread‑pool starvation). |
| **Column mapping** | Use `AS` aliases to avoid name collisions in joins; match `splitOn` to aliases. | Assume overlapping column names (`Id`) map correctly. |
| **Multi‑mapping** | Ensure column order matches the generic type order; one `splitOn` column per additional type. | Put columns for the second type before the `splitOn` column. |
| **Buffered queries** | Use `AsList()` to materialize results when multiple enumerations are needed. | Rely on lazy `IEnumerable` results that re‑execute the query. |
| **Unbuffered/streaming** | Use `QueryUnbufferedAsync` for large result sets that should stream. | Use `QueryAsync` with `buffered: false` expecting true async streaming—it streams synchronously. |
| **Bulk operations** | Use `SqlBulkCopy` or table‑valued parameters for large batches. | Send thousands of individual `ExecuteAsync` calls for bulk inserts. |
| **Transactions** | Use `IDbTransaction` and pass it to Dapper methods; commit/rollback in `try/catch`. | Leave transactions open or rely on implicit commits for multi‑step operations. |

### Annotated Code Example (Best‑Practice Repository Method)

```csharp
public async Task<List<Order>> GetOrdersByDateRangeAsync(DateTime start, DateTime end)
{
    const string sql = @"
        SELECT o.Id AS OrderId, o.Total, o.CreatedAt,
               u.Id AS UserId, u.Name AS UserName
        FROM Orders o
        INNER JOIN Users u ON o.UserId = u.Id
        WHERE o.CreatedAt BETWEEN @Start AND @End
        ORDER BY o.CreatedAt";

    using var conn = new SqlConnection(_connectionString);
    var orders = await conn.QueryAsync<Order, User, Order>(
        sql,
        (order, user) => { order.User = user; return order; },
        new { Start = start, End = end },
        splitOn: "UserId")
        .ConfigureAwait(false);

    return orders.AsList();
}
```

**Why this follows best practices:**

- **Parameters are used** (`@Start`, `@End`) instead of string concatenation—SQL injection prevention.
- **Columns are aliased** (`OrderId`, `UserId`) to avoid collisions and make `splitOn` unambiguous.
- **`ConfigureAwait(false)`** is used in the data‑access layer to avoid synchronization context overhead.
- **`AsList()`** materializes the result, preventing multiple enumerations.
- **The connection is disposed** with `using`.
- **Async is used throughout** for scalability.

### Real-World Cases

- **High‑traffic APIs:** Parameterized async queries with connection pooling.
- **Reporting dashboards:** Multi‑mapping with aliases and `splitOn` for joined data.
- **Batch imports:** `SqlBulkCopy` or table‑valued parameters instead of row‑by‑row Dapper inserts.
- **Transactional operations:** Explicit `IDbTransaction` with commit/rollback.

### References

- Dapper best practices (Microsoft Q&A) – https://learn.microsoft.com/en-nz/answers/questions/2263775/best-way-to-query-azure-synapse-serverless-dedicat
- Dapper FAQ (parameterization) – https://github.com/DapperLib/Dapper
- Dapper patterns and best practices (GitHub) – https://github.com/wshobson/agents/blob/main/agents/plugins/dotnet-contribution/skills/dotnet-backend-patterns/references/dapper-patterns.md
- Dapper transaction best practices – https://github.com/ericksonlopezf/dotnet-transaction/blob/main/docs/cookbook.md

---

## Summary Table: Dapper at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Dapper** | Micro‑ORM for high‑performance data access | `QueryAsync`, `ExecuteAsync`, `QueryMultipleAsync` | Read‑heavy, SQL‑controlled workloads |
| **Micro‑ORM** | Lightweight alternative to full ORMs | Minimal abstraction over ADO.NET | When EF Core overhead is unnecessary |
| **Query execution** | Execute SQL and map results | `Query`, `QueryFirst`, `QuerySingle`, `Execute`, `QueryMultiple` | Every database operation |
| **Mapping** | Convert columns to properties | Simple mapping, multi‑mapping with `splitOn` | When returning objects from queries |
| **Best practices** | Safe, efficient, maintainable Dapper | Parameterization, `using`, async, aliases, `AsList` | Every production implementation |

---

## References (Full List)

- Dapper GitHub repository – https://github.com/DapperLib/Dapper
- Dapper official README – https://github.com/DapperLib/Dapper/blob/main/Readme.md
- How to Work With Dapper in .Net (freeCodeCamp) – https://www.freecodecamp.org/news/how-to-work-with-dapper-in-net/
- Dapper ORM Guide: Queries, Mapping & Async (Red Gate) – https://www.red-gate.com/simple-talk/development/dotnet-development/a-practical-guide-to-dapper/
- Dapper multi‑mapping and splitOn (Stack Overflow) – https://stackoverflow.com/revisions/79613209/2
- Dapper relationships (LearnDapper) – https://github.com/zzzprojects/docs/blob/master/learndapper.com/pages/relationships.md
- Dapper best practices (Microsoft Q&A) – https://learn.microsoft.com/en-nz/answers/questions/2263775/best-way-to-query-azure-synapse-serverless-dedicat
- Dapper patterns and best practices (GitHub) – https://github.com/wshobson/agents/blob/main/agents/plugins/dotnet-contribution/skills/dotnet-backend-patterns/references/dapper-patterns.md
- Dapper transaction best practices – https://github.com/ericksonlopezf/dotnet-transaction/blob/main/docs/cookbook.md
- Comparative Performance Analysis of Dapper and EF Core (Zenodo) – https://zenodo.org/records/21130477
- Dapper vs. Entity Framework Core: Performance & Use Cases (Devart) – https://www.devart.com/