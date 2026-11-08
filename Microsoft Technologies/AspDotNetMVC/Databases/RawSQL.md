# EF Core Raw SQL: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **Entity Framework Core 7–10** and the relational database providers (SQL Server, PostgreSQL, MySQL, SQLite). EF Core 7 introduced the unified `FromSql` method (replacing the older `FromSqlRaw`/`FromSqlInterpolated` split for most scenarios), while `FromSqlRaw` and `ExecuteSqlRaw` remain fully supported for dynamic SQL construction. The security guidance is stable across versions; where the API surface has evolved, it is explicitly noted. Raw SQL is an **escape hatch**, not a default strategy—LINQ remains the recommended query mechanism for the vast majority of cases. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Raw SQL in EF Core refers to the ability to execute SQL statements directly against the database, bypassing LINQ translation, while still leveraging EF Core’s connection management, parameterization infrastructure, and (for queries) entity materialization. |
| **Technical Definition** | EF Core exposes raw SQL through `FromSql`/`FromSqlRaw`/`FromSqlInterpolated` (for queries returning entities or keyless types) and `ExecuteSqlRaw`/`ExecuteSqlInterpolated` (for non-query commands). These methods integrate with the active `DbContext` connection, participate in the current transaction, and (for `FromSql*`) return `IQueryable<TEntity>` that can be further composed with LINQ operators. |
| **Beginner-Friendly Explanation** | Raw SQL is the **“manual override”** for your database queries. When LINQ cannot express what you need—or when LINQ generates inefficient SQL—you write the SQL yourself. EF Core still handles the connection, parameterization (when you use placeholders), and object mapping, so you get full SQL control without abandoning the ORM’s infrastructure. |

### Key Characteristics

- **Escape hatch, not a default** – Intended for the small percentage of queries that LINQ cannot express well or that require hand-tuned SQL.
- **Parameterization is mandatory for user input** – `FromSql`/`FromSqlInterpolated` and `ExecuteSqlInterpolated` automatically parameterize interpolated values; `FromSqlRaw`/`ExecuteSqlRaw` require explicit placeholder parameters or `DbParameter` objects.
- **Composable queries** – `FromSql*` returns `IQueryable<TEntity>`, so you can chain `.Where()`, `.OrderBy()`, `.Include()`, etc., on top of raw SQL.
- **Transactional** – Raw SQL commands participate in the ambient `DbContext` transaction and connection.
- **Provider-specific** – The SQL dialect must match the configured database provider.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core or console project.
- Working knowledge of **EF Core** (`DbContext`, `DbSet`, LINQ queries) and **SQL**.
- Familiarity with **parameterized queries** and the **SQL injection** risk.
- A configured database provider (SQL Server, PostgreSQL, etc.).

### Related Programming Areas

- **LINQ queries** – Raw SQL is the fallback when LINQ cannot express or efficiently translate a query.
- **Dapper** – An alternative micro-ORM for SQL-first data access; EF Core raw SQL provides a middle ground.
- **Stored procedures** – Raw SQL is commonly used to invoke stored procedures.
- **Transactions** – Raw SQL commands share the `DbContext` transaction.
- **Security** – SQL injection prevention is the primary concern when using raw SQL.

### Core Concepts / Features

1. **Raw SQL** – The overall capability, use cases, and API surface.
2. **`FromSqlRaw()`** – Query method with explicit parameter placeholders.
3. **`ExecuteSqlRaw()`** – Non-query command method for INSERT/UPDATE/DELETE/DDL.
4. **Raw SQL best practices** – Parameterization, security auditing, and when to use each API.

---

## 1. Raw SQL (The Capability)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Raw SQL is the capability to execute SQL statements directly through EF Core, bypassing LINQ query translation while retaining EF Core’s connection and transaction management. |
| **Technical Definition** | EF Core provides two families of raw SQL APIs: **query APIs** (`FromSql`, `FromSqlRaw`, `FromSqlInterpolated`) that return `IQueryable<TEntity>` and can be composed with LINQ, and **command APIs** (`ExecuteSqlRaw`, `ExecuteSqlInterpolated`) that execute non-query statements and return the number of rows affected. Query APIs materialize results into entities or keyless types; command APIs do not return result sets. |
| **Beginner-Friendly Explanation** | Raw SQL is the **“write your own SQL”** feature. Instead of fighting with LINQ for a complex report or a stored procedure call, you write the SQL directly. EF Core runs it on the same connection, inside the same transaction, and—for queries—maps the results back to your C# objects. |

### Purposes

- **To** express queries that **LINQ cannot translate** or that would generate inefficient SQL.
- **To** invoke **stored procedures** that return entity data or require complex parameter handling.
- **To** execute **non-query commands** (bulk updates, DDL, `TRUNCATE`) that have no LINQ equivalent.
- **To** allow **hand-tuned SQL** from a DBA to be used verbatim within the application.
- **To** retain EF Core’s **transaction and connection management** while using custom SQL.

### Syntax Rules and Structure

#### API Landscape

| API | Purpose | Returns | Parameterization |
|-----|---------|---------|------------------|
| `FromSql` (EF Core 7+) | Query returning entities | `IQueryable<TEntity>` | **Automatic** (interpolated) |
| `FromSqlInterpolated` | Query returning entities | `IQueryable<TEntity>` | **Automatic** (interpolated) |
| `FromSqlRaw` | Query returning entities | `IQueryable<TEntity>` | **Manual** (placeholders or `DbParameter`) |
| `ExecuteSqlRaw` | Non-query command | `int` (rows affected) | **Manual** (placeholders or `DbParameter`) |
| `ExecuteSqlInterpolated` | Non-query command | `int` (rows affected) | **Automatic** (interpolated) |

**Syntax rules:**

- **`FromSql*` can only be used on a `DbSet`** —it cannot be composed over an arbitrary LINQ query.
- **`FromSql*` returns `IQueryable<TEntity>`** —you can chain `.Where()`, `.OrderBy()`, `.Include()`, etc., on top.
- **Composable SQL must start with `SELECT`** and cannot contain trailing semicolons, query-level hints, or `ORDER BY` without `OFFSET`/`TOP`.
- **`ExecuteSql*` is a method on `DatabaseFacade`** —call it via `context.Database.ExecuteSqlRaw(...)`.
- **Raw SQL uses the active transaction** —if a transaction is open, the command participates in it.

**Constraints and limitations:**

- **Raw SQL is provider-specific** —the SQL dialect must match the configured database.
- **No change tracking for keyless types** —projection into keyless entities does not track changes.
- **Column names must match property names** —for entity materialization, the result set columns must align with the mapped properties (or use aliases).

### Annotated Complete Step-by-Step Code Example

```csharp
// Query using FromSql (EF Core 7+ unified API, automatic parameterization)
var searchTerm = "EF Core";
var blogs = await context.Blogs
    .FromSql($"SELECT * FROM dbo.SearchBlogs({searchTerm})")
    .Where(b => b.Rating > 3)
    .OrderByDescending(b => b.Rating)
    .ToListAsync();

// Non-query command using ExecuteSqlRaw (explicit parameterization)
var rowsAffected = await context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {0}",
    categoryId);
```

**Step-by-step explanation:**

1. `FromSql` starts a LINQ query based on the raw SQL and automatically parameterizes `searchTerm`.
2. The result is `IQueryable<Blog>`, so `.Where()` and `.OrderBy()` are composed on top (EF Core treats the SQL as a subquery).
3. `ExecuteSqlRawAsync` executes an `UPDATE` with a `{0}` placeholder; `categoryId` is bound as a parameter, not concatenated.
4. The return value is the number of rows affected.

**Expected Output:** The `blogs` list contains blogs matching the SQL search with a rating above 3, ordered by rating. `rowsAffected` reports how many product rows were updated.

**Why this output?** `FromSql` materializes entities from the SQL result set and allows further LINQ composition; `ExecuteSqlRaw` executes a non-query command with safe parameter binding.

### Real-World Cases

- **Complex reporting queries:** Multi-table joins, window functions, or CTEs that are cumbersome in LINQ.
- **Stored procedure calls:** `EXEC dbo.GetMostPopularBlogs` returning entity data.
- **Bulk maintenance:** `TRUNCATE TABLE`, index rebuilds, or data-fix scripts executed via `ExecuteSqlRaw`.
- **DBA-tuned queries:** Hand-optimized SQL provided by a DBA and used verbatim.

### References

- SQL Queries (EF Core) – https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- Raw SQL Queries (EF Core docs) – https://github.com/dotnet/EntityFramework.Docs/blob/main/entity-framework/core/querying/raw-sql.md

---

## 2. `FromSqlRaw()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `FromSqlRaw` creates a LINQ query based on a raw SQL string, with **explicit** parameter placeholders (`{0}`, `{1}`) or `DbParameter` objects supplied separately. |
| **Technical Definition** | `RelationalQueryableExtensions.FromSqlRaw<TEntity>(DbSet<TEntity>, string sql, params object[] parameters)` returns `IQueryable<TEntity>`. The SQL string is used as-is; parameters are bound positionally via `{0}`, `{1}` placeholders or by passing `DbParameter` instances. It is the **unsafe-by-default** variant—**misuse (string concatenation of user input) leads to SQL injection**—but is necessary when the SQL template itself must be built dynamically (e.g., varying `WHERE` clauses). |
| **Beginner-Friendly Explanation** | `FromSqlRaw` is the **“I’ll write the SQL myself”** query method. You pass a SQL string with `{0}` placeholders, and EF Core binds your parameter values safely—**as long as you do not concatenate user input into the SQL string**. It is the right tool when the SQL text itself needs to change at runtime, but it is also the API most easily misused. |

### Purposes

- **To** execute a raw SQL query that returns entity data when the **SQL template must vary** (e.g., different `WHERE` clauses depending on a runtime flag).
- **To** use **explicit `DbParameter` objects** when precise control over parameter types, sizes, or facets is needed.
- **To** invoke stored procedures with **named parameters** or positional placeholders.
- **To** allow dynamic SQL construction when parameterization of the entire statement is impossible (e.g., column names)—**with strict allowlisting**.

### Syntax Rules and Structure

```csharp
// Positional placeholders — safe when the template is a constant
var name = "johndoe";
var blogs = await context.Blogs
    .FromSqlRaw("SELECT * FROM Blogs WHERE Name = {0}", name)
    .ToListAsync();

// DbParameter — safe and precise
var userParam = new SqlParameter("user", "johndoe");
var blogs = await context.Blogs
    .FromSqlRaw("EXECUTE dbo.GetMostPopularBlogsForUser @filterByUser={0}", userParam)
    .ToListAsync();

// Dynamic template (safe only if the template is built from compile-time literals)
var template = useExactMatch
    ? "SELECT * FROM Products WHERE Name = {0}"
    : "SELECT * FROM Products WHERE Name LIKE {0}";
var products = await context.Products
    .FromSqlRaw(template, name)
    .ToListAsync();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `sql` | The raw SQL string—**must be a compile-time constant or built from safe literals only**. |
| `parameters` | Positional values for `{0}`, `{1}` or `DbParameter` objects. |
| Return | `IQueryable<TEntity>` that can be composed with LINQ. |

**Syntax rules:**

- **`{0}`, `{1}` are parameter placeholders**, not C# string formatting—EF Core rewrites them to `@p0`, `@p1` and binds the values.
- **`FromSqlRaw` can only be used on a `DbSet`** —it cannot be composed over an arbitrary LINQ query.
- **The SQL must be composable** (start with `SELECT`, no trailing semicolon) if you intend to chain LINQ operators on top.

**Constraints and limitations:**

- **Vulnerable to SQL injection if user input is concatenated into the SQL string** —never do this.
- **Dynamic identifiers (column names, table names) cannot be parameterized** —use a strict allowlist if you must support them.
- **Column names in the result set must match property names** (or be aliased) for entity materialization.

### Annotated Complete Step-by-Step Code Example

```csharp
public async Task<List<Product>> SearchProductsAsync(string name, bool exactMatch)
{
    // Template built from compile-time literals — safe
    var template = exactMatch
        ? "SELECT * FROM Products WHERE Name = {0}"
        : "SELECT * FROM Products WHERE Name LIKE {0}";

    return await _context.Products
        .FromSqlRaw(template, name)
        .AsNoTracking()
        .ToListAsync();
}
```

**Step-by-step explanation:**

1. The SQL template is selected from two compile-time literals—**no user input enters the template**.
2. `name` is passed as the `{0}` parameter and bound safely.
3. `AsNoTracking()` reduces overhead for read-only results.

**Expected Output:** A list of products matching either exactly or partially, depending on `exactMatch`.

**Why this output?** `FromSqlRaw` binds `name` as a parameter regardless of the template chosen; the template itself is safe because it comes from literals, not user input.

### Real-World Cases

- **Dynamic filtering:** Different `WHERE` clauses based on a user-selected filter mode (exact vs. partial match).
- **Stored procedures with named parameters:** `EXEC dbo.sp_GetCategoryStats @Month={0}`.
- **Complex joins:** Hand-written SQL with multiple joins that LINQ translates poorly.

### References

- FromSqlRaw method (API reference) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationalqueryableextensions.fromsqlraw
- Raw SQL Queries (parameters and safety) – https://github.com/dotnet/EntityFramework.Docs/blob/main/entity-framework/core/querying/raw-sql.md

---

## 3. `ExecuteSqlRaw()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ExecuteSqlRaw` executes a **non-query** SQL command (INSERT, UPDATE, DELETE, DDL) directly against the database, returning the number of rows affected. |
| **Technical Definition** | `RelationalDatabaseFacadeExtensions.ExecuteSqlRaw(DatabaseFacade databaseFacade, string sql, params object[] parameters)` executes the SQL string as-is, binding parameters via `{0}` placeholders or `DbParameter` objects. It does **not** return a result set and cannot be composed with LINQ. It is the command-side counterpart to `FromSqlRaw` and shares the same parameterization and injection risks. |
| **Beginner-Friendly Explanation** | `ExecuteSqlRaw` is for **commands that change data**—`UPDATE`, `DELETE`, `INSERT`, `TRUNCATE`, `CREATE TABLE`. You write the SQL, pass parameter values, and EF Core executes it, returning how many rows were affected. It is the raw-SQL equivalent of `SaveChanges` for bulk or non-entity operations. |

### Purposes

- **To** execute **bulk updates or deletes** that do not need entity tracking.
- **To** perform **DDL operations** (create/drop tables, indexes, functions) that have no LINQ equivalent.
- **To** call **stored procedures that do not return result sets**.
- **To** run **maintenance commands** (`TRUNCATE TABLE`, index rebuilds).
- **To** execute **raw SQL that returns only a row count** (rows affected).

### Syntax Rules and Structure

```csharp
// Parameterized UPDATE
var rowsAffected = await context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = {0}",
    categoryId);

// DbParameter for precise control
var param = new SqlParameter("@CategoryId", categoryId);
var rows = await context.Database.ExecuteSqlRawAsync(
    "UPDATE Products SET Price = Price * 1.1 WHERE CategoryId = @CategoryId",
    param);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context.Database` | The `DatabaseFacade` exposing command methods. |
| `sql` | The non-query SQL string. |
| `parameters` | Positional values or `DbParameter` objects. |
| Return | `int` — number of rows affected. |

**Syntax rules:**

- **`ExecuteSqlRaw` is called on `DatabaseFacade`** —`context.Database.ExecuteSqlRaw(...)`.
- **Parameters use `{0}` placeholders** —not C# string interpolation (unless using `ExecuteSqlInterpolated`).
- **The command participates in the current transaction** —if a transaction is open, the command is included.

**Constraints and limitations:**

- **Vulnerable to SQL injection if user input is concatenated into the SQL string** —same as `FromSqlRaw`.
- **No result set** —use `FromSql*` for queries that return data.
- **No change tracking** —the command bypasses the change tracker entirely.

### Annotated Complete Step-by-Step Code Example

```csharp
public async Task<int> ApplyDiscountAsync(int categoryId, decimal discountPercent)
{
    var sql = "UPDATE Products SET Price = Price * (1 - {0}) WHERE CategoryId = {1}";
    return await _context.Database.ExecuteSqlRawAsync(sql, discountPercent, categoryId);
}
```

**Step-by-step explanation:**

1. The SQL is a compile-time constant with `{0}` and `{1}` placeholders.
2. `discountPercent` and `categoryId` are bound safely as parameters.
3. The method returns the number of product rows updated.

**Expected Output:** An integer representing how many products in the category had their price updated.

**Why this output?** `ExecuteSqlRawAsync` executes the parameterized `UPDATE` and returns the affected row count; no entities are tracked or materialized.

### Real-World Cases

- **Bulk price adjustments:** Updating prices across a product category without loading every entity.
- **Data maintenance:** `TRUNCATE TABLE` or `DELETE FROM` for temporary/staging tables.
- **Schema operations:** Creating functions, triggers, or indexes during deployment.
- **Stored procedure calls without result sets:** `EXEC dbo.ArchiveOldOrders`.

### References

- ExecuteSqlRaw method (API reference) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.executesqlraw
- Raw SQL Queries (command execution) – https://github.com/dotnet/EntityFramework.Docs/blob/main/entity-framework/core/querying/raw-sql.md

---

## 4. Raw SQL Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Raw SQL best practices are guidelines for using raw SQL safely, securely, and maintainably within an EF Core application. |
| **Technical Definition** | Best practices encompass **mandatory parameterization** for user input, **preferring the safe interpolated APIs** (`FromSql`, `FromSqlInterpolated`, `ExecuteSqlInterpolated`) over the raw variants unless dynamic SQL construction is genuinely required, **strict allowlisting for dynamic identifiers**, **codebase auditing** for raw SQL calls, **composability awareness**, and **testing** of raw SQL queries. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for raw SQL—they help you avoid SQL injection, use the safest API for the job, and keep raw SQL from becoming the default rather than the exception. |

### Purposes

- **To** prevent **SQL injection**—the primary risk of raw SQL.
- **To** choose the **safest API** for each scenario (interpolated APIs are safe by default; raw APIs require discipline).
- **To** keep raw SQL **rare and intentional**, not a replacement for LINQ.
- **To** handle **dynamic identifiers** safely through allowlists.
- **To** make raw SQL **auditable and testable** in the codebase.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **API choice** | Use `FromSql`/`FromSqlInterpolated` and `ExecuteSqlInterpolated` for automatic parameterization. | Use `FromSqlRaw`/`ExecuteSqlRaw` unless the SQL template must vary at runtime. |
| **User input** | **Always parameterize** user-provided values—use `{0}` placeholders or `DbParameter`. | Concatenate or interpolate user input into the SQL string. |
| **Dynamic identifiers** | Use a **strict allowlist** for column/table names that cannot be parameterized. | Accept arbitrary identifiers from users—they enable injection and performance abuse. |
| **Auditing** | Grep for `FromSqlRaw`, `ExecuteSqlRaw`, and direct `SqlCommand` usage in code review and CI. | Assume raw SQL is safe because the ORM is “secure by default.” |
| **Composability** | Keep SQL composable (`SELECT`-first, no trailing semicolons) if you intend to chain LINQ. | Compose LINQ on top of non-composable SQL (it will throw at runtime). |
| **Testing** | Test raw SQL against a real database (SQLite in-memory or Testcontainers). | Test raw SQL only against the EF Core In-Memory provider (it does not execute SQL). |
| **Stored procedures** | Parameterize procedure calls and verify the procedure itself does not build dynamic SQL unsafely. | Assume a stored procedure is safe simply because it is a procedure. |
| **Documentation** | Document **why** raw SQL was necessary (which LINQ limitation it overcomes). | Let raw SQL accumulate without justification. |

### Annotated Code Example (Best-Practice Raw SQL Usage)

```csharp
public async Task<List<OrderSummary>> GetHighValueOrdersAsync(
    DateTime start, DateTime end, decimal threshold)
{
    // Use FromSql with interpolation — automatic parameterization
    var sql = $@"
        SELECT o.Id AS OrderId, o.Total, c.Name AS CustomerName
        FROM Orders o
        JOIN Customers c ON o.CustomerId = c.Id
        WHERE o.CreatedAt BETWEEN {start} AND {end}
          AND o.Total > {threshold}
        ORDER BY o.Total DESC";

    return await _context.Set<OrderSummary>()
        .FromSql(sql)
        .AsNoTracking()
        .ToListAsync();
}
```

**Why this follows best practices:**

- **`FromSql` with interpolation** automatically parameterizes `start`, `end`, and `threshold`—no manual placeholder management and no injection risk.
- **`AsNoTracking`** is used because the result is read-only.
- **The SQL is a compile-time constant** —the interpolation slots are parameters, not concatenated values.
- **The query is composable** (starts with `SELECT`, no trailing semicolon), though no further LINQ is applied here.

### Real-World Cases

- **High-value order reports:** Complex joins and filters that are clearer in SQL than in LINQ.
- **DBA-provided queries:** Hand-tuned SQL reviewed and approved by a DBA, integrated via `FromSql`.
- **Data migration scripts:** `ExecuteSqlInterpolated` for one-off data fixes during deployment.

### References

- SQL Injection Prevention in C# with EF Core – https://safeguard.sh/resources/blog/sql-injection-prevention-in-c-with-entity-frameworklinq
- Raw SQL safety lecture notes – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/main/curriculum/week-10-entity-framework-core-deep/lecture-notes/03-raw-sql-converters-compiled-queries-observability.md
- Raw SQL audit guardrails (GitHub) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/blob/main/…

---

## Summary Table: Raw SQL APIs at a Glance

| API | Purpose | Parameterization | When to Use |
|-----|---------|------------------|-------------|
| **`FromSql`** (EF Core 7+) | Query returning entities | **Automatic** (interpolated) | Default choice for raw SQL queries with dynamic values |
| **`FromSqlInterpolated`** | Query returning entities | **Automatic** (interpolated) | Older EF Core versions; still fully supported and safe |
| **`FromSqlRaw`** | Query returning entities | **Manual** (`{0}` or `DbParameter`) | Dynamic SQL templates (built from literals); precise `DbParameter` control |
| **`ExecuteSqlRaw`** | Non-query command | **Manual** (`{0}` or `DbParameter`) | Bulk updates/deletes/DDL with dynamic parameters |
| **`ExecuteSqlInterpolated`** | Non-query command | **Automatic** (interpolated) | Default choice for raw SQL commands with dynamic values |

---

## References (Full List)

- SQL Queries (EF Core) – https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- Raw SQL Queries (EF Core docs, GitHub) – https://github.com/dotnet/EntityFramework.Docs/blob/main/entity-framework/core/querying/raw-sql.md
- FromSqlRaw method (API reference) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationalqueryableextensions.fromsqlraw
- ExecuteSqlRaw method (API reference) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.executesqlraw
- SQL Injection Prevention in C# with EF Core – https://safeguard.sh/resources/blog/sql-injection-prevention-in-c-with-entity-frameworklinq
- Raw SQL safety lecture notes (CODECRUNCH) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/main/curriculum/week-10-entity-framework-core-deep/lecture-notes/03-raw-sql-converters-compiled-queries-observability.md
- Raw SQL audit guardrails (GitHub) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/blob/main/…
- EF Core 7 FromSql changes – https://www.codemag.com/article/…
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html