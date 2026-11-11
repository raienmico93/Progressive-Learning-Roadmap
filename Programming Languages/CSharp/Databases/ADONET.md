# C# ADO.NET — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**ADO.NET** is the foundational data-access technology in .NET for connecting to relational databases, executing commands, reading results, and managing disconnected data. It provides a consistent set of classes—`DbConnection`, `DbCommand`, `DbDataReader`, `DbDataAdapter`, `DataSet`, and `DataTable`—that work across providers (SQL Server, PostgreSQL, MySQL, SQLite, Oracle) via provider-specific implementations such as `SqlConnection` and `SqlCommand` for Microsoft SQL Server.

**Technical Definition**
ADO.NET is a set of classes in the `System.Data` namespace (and provider-specific namespaces such as `Microsoft.Data.SqlClient`) that expose relational data access. It supports two models: **connected** (open a connection, execute commands, stream results via `DbDataReader`, close) and **disconnected** (fill a `DataSet`/`DataTable` via `DbDataAdapter`, work offline, reconcile changes). Connections are pooled by default (`SqlConnection` pooling), transactions are supported via `DbTransaction`, and parameterized commands prevent SQL injection.

**Beginner-Friendly Explanation**
ADO.NET is **the classic way to talk to a database in C#**. You open a connection, send SQL commands, read the results row by row, and close the connection. It is lower-level than an ORM like Entity Framework: you write the SQL yourself, but you get full control and predictable performance. Think of it as **manual driving**—more work, but you know exactly what is happening.

---

### Key Characteristics

- **Provider-based** — Works with multiple databases via provider-specific classes (`SqlConnection` for SQL Server, `NpgsqlConnection` for PostgreSQL, etc.).
- **Connected and disconnected models** — Stream results with `DataReader` or fill a `DataSet`/`DataTable`.
- **Connection pooling** — Enabled by default; reuses physical connections for performance.
- **Parameterized commands** — Prevent SQL injection.
- **Transactions** — ACID guarantees via `DbTransaction`.
- **Async support** — `OpenAsync`, `ExecuteReaderAsync`, `ReadAsync`, `ExecuteNonQueryAsync`.
- **Low-level control** — You write SQL and manage connections explicitly.

---

### Prerequisites

- Basic understanding of **C#** (classes, `using`, async/await).
- Familiarity with **SQL** (SELECT, INSERT, UPDATE, DELETE).
- Knowledge of **`IDisposable`** and `using` statements.
- Awareness of **connection strings** and configuration.
- Understanding of **parameterized queries** and SQL injection.

---

### Related Programming Areas

- **Entity Framework Core** — Higher-level ORM built on ADO.NET concepts.
- **Dapper** — Micro-ORM that wraps ADO.NET for convenience.
- **Data access layers** — Repositories using ADO.NET directly.
- **Reporting and ETL** — `DataSet`/`DataTable` for tabular data.
- **Connection pooling** — Critical for scalability.

---

### Core Concepts / Features

1. ADO.NET (Overarching Concept)
2. `SqlConnection`
3. `SqlCommand`
4. `SqlDataReader`
5. `SqlDataAdapter`
6. `DataSet`
7. `DataTable`
8. Transactions
9. Connection Pooling
10. ADO.NET Best Practices

---

## 1. ADO.NET (Overarching Concept)

### Definitions

**Core Definition:** ADO.NET is the .NET framework for relational data access via provider-specific connection, command, reader, and adapter classes.

**Technical Definition:** ADO.NET exposes `System.Data.Common` abstractions (`DbConnection`, `DbCommand`, `DbDataReader`, `DbDataAdapter`, `DbTransaction`) implemented by providers. The connected model uses `DbConnection` + `DbCommand` + `DbDataReader`; the disconnected model uses `DbDataAdapter` to fill `DataSet`/`DataTable`. Connection strings configure providers and pooling.

**Beginner-Friendly Explanation:** ADO.NET is **the toolbox for talking to databases**. You use its classes to open connections, run SQL, and read results.

---

### Purposes

- To **execute SQL** against relational databases.
- To **read results** efficiently with `DataReader`.
- To **work offline** with `DataSet`/`DataTable`.
- To **manage transactions**.
- To **control performance** and resource usage precisely.

---

### Syntax Rules and Structure

```csharp
using Microsoft.Data.SqlClient;

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
await using var command = new SqlCommand("SELECT Id, Name FROM Products", connection);
await using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
```

**Component breakdown**

- `SqlConnection` — Opens the connection.
- `SqlCommand` — Executes SQL.
- `SqlDataReader` — Streams results.
- `using`/`await using` — Ensures disposal.

#### Syntax Rules

- Connections must be opened before executing commands.
- Commands can be reused; parameter collections must be cleared.
- Readers must be closed before reusing the connection for another command.

#### Constraints and Limitations

- Connections are expensive; use pooling and short-lived connections.
- DataReaders are forward-only and read-only.
- SQL injection risk if parameters are not used.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Query

```csharp
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
await using var cmd = new SqlCommand("SELECT COUNT(*) FROM Products", conn);
var count = (int)(await cmd.ExecuteScalarAsync())!;
Console.WriteLine($"Products: {count}");
```

**Expected Output (example)**
```
Products: 42
```

**Why this output occurs:** `ExecuteScalarAsync` returns the first column of the first row.

---

### Real-World Cases

- **Legacy data access** — Direct SQL in existing apps.
- **High-performance queries** — Fine-grained control over SQL and connections.
- **Reporting** — `DataSet`/`DataTable` for tabular data.
- **ETL** — Bulk operations with `SqlBulkCopy`.

---

### References

- ADO.NET overview — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview
- `Microsoft.Data.SqlClient` — Microsoft Learn — https://learn.microsoft.com/en-us/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace

---

## 2. `SqlConnection`

### Definitions

**Core Definition:** `SqlConnection` represents a connection to a SQL Server database.

**Technical Definition:** `Microsoft.Data.SqlClient.SqlConnection` derives from `DbConnection`. It is opened with `Open()`/`OpenAsync()`, closed with `Close()`/`DisposeAsync()`. Connection strings configure server, database, authentication, pooling, timeouts, and encryption. Connection pooling is enabled by default; physical connections are reused when `Open()` is called on a new `SqlConnection`.

**Beginner-Friendly Explanation:** `SqlConnection` is **the pipe to the database**. You open it, use it, and close it (or let `using` do it).

---

### Purposes

- To **establish a connection** to SQL Server.
- To **manage connection lifetime** and pooling.
- To **configure authentication and security** via connection strings.
- To **support async operations**.

---

### Syntax Rules and Structure

```csharp
var connStr = "Server=.;Database=App;Integrated Security=true;Encrypt=true;";
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
```

**Component breakdown**

- `Server` — Host.
- `Database` — Initial catalog.
- `Integrated Security` — Windows auth.
- `Encrypt` — TLS encryption.

#### Syntax Rules

- Use `using`/`await using` to ensure disposal.
- Open as late as possible and close as early as possible.
- Use `Encrypt=true` in production.

#### Constraints and Limitations

- Opening is expensive; pooling mitigates.
- Connection strings may contain secrets; store them securely.

---

### Annotated Code Examples and Expected Outputs

#### Example: Open and Close

```csharp
await using (var conn = new SqlConnection(connStr))
{
    await conn.OpenAsync();
    Console.WriteLine(conn.State);   // Open
}
Console.WriteLine("Disposed");
```

**Expected Output**
```
Open
Disposed
```

**Why this output occurs:** `OpenAsync` opens the connection; `await using` disposes it, returning it to the pool.

---

### Real-World Cases

- **Every ADO.NET operation** — Connections are the entry point.
- **Health checks** — Verifying database connectivity.
- **Multi-tenant apps** — Per-tenant connection strings.

---

### References

- `SqlConnection` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqlconnection
- Connection strings — Microsoft Learn — https://learn.microsoft.com/en-us/sql/connect/ado-net/connection-string-syntax

---

## 3. `SqlCommand`

### Definitions

**Core Definition:** `SqlCommand` represents a SQL statement or stored procedure to execute against a SQL Server database.

**Technical Definition:** `SqlCommand` derives from `DbCommand`. It supports `ExecuteNonQuery` (no result set), `ExecuteScalar` (single value), `ExecuteReader` (stream results), and async equivalents. Parameters are added to the `Parameters` collection and bind safely to `@name` placeholders.

**Beginner-Friendly Explanation:** `SqlCommand` is **the SQL you send**. It can insert, update, delete, or query.

---

### Purposes

- To **execute SQL** and stored procedures.
- To **parameterize queries** and prevent injection.
- To **return scalar values** or affected row counts.
- To **stream results** via readers.

---

### Syntax Rules and Structure

```csharp
await using var cmd = new SqlCommand(
    "SELECT Id, Name FROM Products WHERE Price > @minPrice", conn);
cmd.Parameters.AddWithValue("@minPrice", 10m);
```

**Component breakdown**

- `CommandText` — SQL or stored procedure.
- `Parameters` — Parameter collection.
- `CommandType` — `Text` or `StoredProcedure`.

#### Syntax Rules

- Always use parameters for user input.
- Use `Add` with explicit `SqlDbType` for precision.
- Set `CommandTimeout` for long-running queries.

#### Constraints and Limitations

- `AddWithValue` can cause implicit conversions; prefer explicit types.
- Commands are not thread-safe.

---

### Annotated Code Examples and Expected Outputs

#### Example: Parameterized Insert

```csharp
await using var cmd = new SqlCommand(
    "INSERT INTO Products (Name, Price) VALUES (@name, @price)", conn);
cmd.Parameters.Add("@name", SqlDbType.NVarChar, 100).Value = "Widget";
cmd.Parameters.Add("@price", SqlDbType.Decimal).Value = 9.99m;
var rows = await cmd.ExecuteNonQueryAsync();
Console.WriteLine($"Inserted {rows} row(s)");
```

**Expected Output**
```
Inserted 1 row(s)
```

**Why this output occurs:** `ExecuteNonQueryAsync` returns the number of affected rows.

---

### Real-World Cases

- **CRUD operations** — Insert/update/delete.
- **Stored procedures** — `CommandType.StoredProcedure`.
- **Batch operations** — Multiple statements.

---

### References

- `SqlCommand` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqlcommand

---

## 4. `SqlDataReader`

### Definitions

**Core Definition:** `SqlDataReader` is a forward-only, read-only stream of rows returned by a query.

**Technical Definition:** `SqlDataReader` provides efficient, streaming access to result sets. `Read()` advances to the next row; `Get*` methods read typed columns by ordinal. It must be closed before the connection can execute another command. Async variants (`ReadAsync`) are supported.

**Beginner-Friendly Explanation:** `SqlDataReader` is **a cursor that moves forward one row at a time**. It is fast and memory-efficient.

---

### Purposes

- To **stream large result sets** without loading all rows into memory.
- To **read typed columns** by ordinal.
- To **process rows** efficiently.
- To support **async reading**.

---

### Syntax Rules and Structure

```csharp
await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    var id = reader.GetInt32(0);
    var name = reader.GetString(1);
}
```

**Component breakdown**

- `Read`/`ReadAsync` — Advance to next row.
- `GetInt32`, `GetString`, etc. — Typed access by ordinal.
- `IsDBNull` — Check for null.

#### Syntax Rules

- Reader must be closed before reusing the connection.
- Use `GetOrdinal` for column names.
- Check `IsDBNull` before reading nullable columns.

#### Constraints and Limitations

- Forward-only; cannot rewind.
- One active reader per connection (MARS aside).

---

### Annotated Code Examples and Expected Outputs

#### Example: Read Rows

```csharp
await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
```

**Expected Output (example)**
```
1: Widget
2: Gadget
```

**Why this output occurs:** `ReadAsync` advances row by row; `Get*` reads columns.

---

### Real-World Cases

- **Large reports** — Streaming rows.
- **Export jobs** — Processing each row.
- **Data migration** — Reading source tables.

---

### References

- `SqlDataReader` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqldatareader

---

## 5. `SqlDataAdapter`

### Definitions

**Core Definition:** `SqlDataAdapter` bridges a `DataSet`/`DataTable` and a SQL Server database, filling and updating data.

**Technical Definition:** `SqlDataAdapter` uses `SelectCommand`, `InsertCommand`, `UpdateCommand`, and `DeleteCommand` to fill and reconcile changes. `Fill` populates a `DataTable`/`DataSet`; `Update` sends pending changes back. `SqlCommandBuilder` can generate commands automatically for simple queries.

**Beginner-Friendly Explanation:** `SqlDataAdapter` is **the bridge for disconnected data**. It fills a `DataTable` from the database and can send changes back.

---

### Purposes

- To **fill `DataSet`/`DataTable`** from a query.
- To **reconcile changes** back to the database.
- To support **disconnected scenarios**.
- To simplify **bulk updates**.

---

### Syntax Rules and Structure

```csharp
var adapter = new SqlDataAdapter("SELECT Id, Name FROM Products", conn);
var table = new DataTable();
adapter.Fill(table);
```

**Component breakdown**

- `SelectCommand` — Query to fill.
- `Fill` — Populates the `DataTable`.
- `Update` — Sends changes.
- `SqlCommandBuilder` — Auto-generates CRUD commands.

#### Syntax Rules

- The connection is opened/closed by `Fill` if not already open.
- `Update` requires `InsertCommand`/`UpdateCommand`/`DeleteCommand`.
- `SqlCommandBuilder` requires a primary key.

#### Constraints and Limitations

- Not ideal for high-volume updates.
- Concurrency conflicts require handling.

---

### Annotated Code Examples and Expected Outputs

#### Example: Fill a DataTable

```csharp
var adapter = new SqlDataAdapter("SELECT Id, Name FROM Products", conn);
var table = new DataTable();
adapter.Fill(table);
Console.WriteLine(table.Rows.Count);
```

**Expected Output (example)**
```
42
```

**Why this output occurs:** `Fill` executes the query and populates rows in the `DataTable`.

---

### Real-World Cases

- **WinForms apps** — Binding `DataTable` to grids.
- **Reporting** — Offline data manipulation.
- **Legacy apps** — `DataSet`-based data access.

---

### References

- `SqlDataAdapter` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqldataadapter
- Populating a DataSet from a DataAdapter — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/populating-a-dataset-from-a-dataadapter

---

## 6. `DataSet`

### Definitions

**Core Definition:** `DataSet` is an in-memory, disconnected representation of relational data, containing one or more `DataTable` objects and their relationships.

**Technical Definition:** `DataSet` is a serializable, in-memory cache of data. It holds `DataTable`s, `DataRelation`s, and constraints. It supports XML serialization, change tracking, and merging. It is provider-agnostic.

**Beginner-Friendly Explanation:** `DataSet` is **a mini-database in memory**. It can hold multiple tables and relationships, and it works even when disconnected.

---

### Purposes

- To **work offline** with relational data.
- To hold **multiple related tables**.
- To support **XML serialization**.
- To track **changes** for later reconciliation.
- To bind to **UI controls**.

---

### Syntax Rules and Structure

```csharp
var dataSet = new DataSet("App");
var table = new DataTable("Products");
dataSet.Tables.Add(table);
```

**Component breakdown**

- `Tables` — Collection of `DataTable`.
- `Relations` — `DataRelation` objects.
- `AcceptChanges`/`RejectChanges` — Change management.

#### Syntax Rules

- Tables must be added before use.
- Relationships require matching keys.
- Serialization is supported via `WriteXml`.

#### Constraints and Limitations

- Memory-heavy for large datasets.
- Not ideal for high-throughput writes.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Tables

```csharp
var ds = new DataSet();
var products = ds.Tables.Add("Products");
products.Columns.Add("Id", typeof(int));
products.Columns.Add("Name", typeof(string));
products.Rows.Add(1, "Widget");
```

**Expected Output (conceptual)**
```
DataSet contains Products table with one row.
```

**Why this pattern is used:** `DataSet` holds multiple tables and can be serialized or bound to UI.

---

### Real-World Cases

- **Legacy WinForms** — Data binding.
- **Offline apps** — Local cache.
- **ETL** — Intermediate data storage.

---

### References

- `DataSet` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.data.dataset
- DataSets, DataTables, and DataViews — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/dataset-datatable-dataview/

---

## 7. `DataTable`

### Definitions

**Core Definition:** `DataTable` is an in-memory representation of a single table with rows and columns.

**Technical Definition:** `DataTable` holds `DataColumn` definitions and `DataRow` instances. It supports constraints, change tracking, and views. It can be filled by `SqlDataAdapter` or created manually. `DataTable` can be serialized to XML.

**Beginner-Friendly Explanation:** `DataTable` is **a single in-memory table**. It is useful for holding query results and binding to UI.

---

### Purposes

- To **hold tabular data** in memory.
- To **bind to UI** controls.
- To **track changes**.
- To **serialize to XML**.

---

### Syntax Rules and Structure

```csharp
var table = new DataTable("Products");
table.Columns.Add("Id", typeof(int));
table.Columns.Add("Name", typeof(string));
table.Rows.Add(1, "Widget");
```

**Component breakdown**

- `Columns` — Column definitions.
- `Rows` — Data rows.
- `Constraints` — Primary keys, unique constraints.

#### Syntax Rules

- Columns must be defined before adding rows.
- Primary keys enable change reconciliation.
- `DataTable` can be filtered with `DataView`.

#### Constraints and Limitations

- Not thread-safe.
- Memory-heavy for large data.

---

### Annotated Code Examples and Expected Outputs

#### Example: Manual Table Creation

```csharp
var table = new DataTable("People");
table.Columns.Add("Name", typeof(string));
table.Columns.Add("Age", typeof(int));
table.Rows.Add("Alice", 30);
Console.WriteLine(table.Rows[0]["Name"]);
```

**Expected Output**
```
Alice
```

**Why this output occurs:** Rows are added and accessed by column name.

---

### Real-World Cases

- **UI grids** — Binding.
- **CSV/XML exports** — Table serialization.
- **In-memory filters** — `DataView`.

---

### References

- `DataTable` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.data.datatable
- Creating a DataTable — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/dataset-datatable-dataview/creating-a-datatable

---

## 8. Transactions

### Definitions

**Core Definition:** A transaction groups multiple database operations into an atomic unit that either fully commits or fully rolls back.

**Technical Definition:** ADO.NET transactions are represented by `DbTransaction` (e.g., `SqlTransaction`). Commands must set `Transaction` to participate. `Commit()` persists changes; `Rollback()` reverts them. Isolation levels control concurrency. `TransactionScope` provides ambient transactions across multiple resources.

**Beginner-Friendly Explanation:** A transaction is **“all or nothing.”** If one step fails, everything rolls back.

---

### Purposes

- To **ensure atomicity** of multi-step operations.
- To **maintain data integrity**.
- To **handle concurrency** via isolation levels.
- To **coordinate multiple commands**.

---

### Syntax Rules and Structure

```csharp
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
await using var tx = await conn.BeginTransactionAsync();
try
{
    await using var cmd = new SqlCommand("UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1", conn, (SqlTransaction)tx);
    await cmd.ExecuteNonQueryAsync();
    // second command...
    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

**Component breakdown**

- `BeginTransaction` — Starts a transaction.
- `Command.Transaction` — Associates command with transaction.
- `Commit`/`Rollback` — Finalizes.

#### Syntax Rules

- All commands in the transaction must set `Transaction`.
- Always `Commit` or `Rollback`.
- Use `TransactionScope` for distributed transactions.

#### Constraints and Limitations

- Long transactions cause locking and blocking.
- Distributed transactions may require MSDTC.

---

### Annotated Code Examples and Expected Outputs

#### Example: Commit or Rollback

```csharp
await using var tx = await conn.BeginTransactionAsync();
try
{
    // execute commands with tx
    await tx.CommitAsync();
    Console.WriteLine("Committed");
}
catch
{
    await tx.RollbackAsync();
    Console.WriteLine("Rolled back");
    throw;
}
```

**Expected Output (success)**
```
Committed
```

**Expected Output (failure)**
```
Rolled back
```

**Why this pattern is used:** It guarantees atomicity and releases locks promptly.

---

### Real-World Cases

- **Banking transfers** — Debit and credit atomically.
- **Order processing** — Inventory and payment.
- **Batch imports** — All-or-nothing.

---

### References

- Transactions in ADO.NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/local-transactions
- `SqlTransaction` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqltransaction

---

## 9. Connection Pooling

### Definitions

**Core Definition:** Connection pooling reuses physical database connections to reduce the cost of opening and closing connections.

**Technical Definition:** ADO.NET providers maintain a pool of physical connections keyed by connection string. When `Open()` is called on a new `SqlConnection`, a pooled connection is returned if available; when `Dispose()`/`Close()` is called, the connection is returned to the pool instead of being physically closed. Pooling parameters (`Min Pool Size`, `Max Pool Size`, `Connection Lifetime`, `Connection Timeout`) configure behavior.

**Beginner-Friendly Explanation:** Connection pooling is **a shared fleet of database connections**. Instead of creating a new one every time, you borrow from the pool and return it.

---

### Purposes

- To **reduce connection overhead**.
- To **improve scalability**.
- To **control the number of physical connections**.
- To **reuse authenticated connections**.

---

### Syntax Rules and Structure

```
Server=.;Database=App;Integrated Security=true;Min Pool Size=0;Max Pool Size=100;
```

**Component breakdown**

- `Min Pool Size` — Minimum retained connections.
- `Max Pool Size` — Maximum total.
- `Connection Lifetime` — Maximum age before recycling.

#### Syntax Rules

- Use identical connection strings to reuse pools.
- Close/dispose connections promptly to return them to the pool.
- Avoid changing connection strings per request (creates new pools).

#### Constraints and Limitations

- Pool exhaustion causes timeouts.
- Connection strings with secrets must be secured.
- Pool fragmentation from many distinct connection strings.

---

### Annotated Code Examples and Expected Outputs

#### Example: Pool Reuse

```csharp
for (int i = 0; i < 10; i++)
{
    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();
    // use connection
}   // returned to pool
```

**Expected Output (conceptual)**
```
// Only a small number of physical connections are opened; most Opens reuse pooled connections.
```

**Why this pattern is used:** Pooling dramatically reduces connection latency under load.

---

### Real-World Cases

- **Web APIs** — High request rates.
- **Background services** — Repeated database access.
- **Multi-tenant apps** — Per-tenant pool configurations.

---

### References

- SQL Server Connection Pooling — Microsoft Learn — https://learn.microsoft.com/en-us/sql/connect/ado-net/sql-server-connection-pooling
- Connection Pooling in ADO.NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/connection-pooling

---

## 10. ADO.NET Best Practices

### Definitions

**Core Definition:** ADO.NET best practices are the guidelines for writing efficient, secure, and maintainable data-access code.

**Technical Definition:** Best practices include: **always use parameterized commands**; **use `using`/`await using`** for connections, commands, and readers; **keep connections open as briefly as possible**; **use `DataReader` for streaming**; **use `DataSet`/`DataTable` only for disconnected scenarios**; **use transactions for multi-step operations**; **avoid `AddWithValue`** for precise types; **use async APIs**; **store connection strings securely**; **handle transient failures with retries**; and **monitor connection pooling**.

**Beginner-Friendly Explanation:** The best practices are: **parameterize**, **dispose**, **keep connections short**, **use async**, **use transactions**, and **don’t hard-code secrets**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Parameterize
cmd.Parameters.Add("@id", SqlDbType.Int).Value = id;

// 2. Dispose
await using var conn = new SqlConnection(connStr);

// 3. Keep connections short
await conn.OpenAsync();
// use
await conn.CloseAsync();

// 4. Stream with DataReader
await using var reader = await cmd.ExecuteReaderAsync();

// 5. Use transactions
await using var tx = await conn.BeginTransactionAsync();

// 6. Async APIs
await cmd.ExecuteNonQueryAsync();
```

**Component breakdown**

- **Parameters** — Prevent injection.
- **Disposal** — Prevent leaks.
- **Short connections** — Improve pooling.
- **`DataReader`** — Stream, don’t buffer.
- **Transactions** — Atomicity.
- **Async** — Scalability.

#### Rules and Constraints

- **Do** use parameters for all user input.
- **Do** dispose connections, commands, readers, and transactions.
- **Do** keep connections open only as long as needed.
- **Do** use `DataReader` for large result sets.
- **Do** use transactions for multi-step operations.
- **Do not** use `AddWithValue` for types where precision matters.
- **Do not** concatenate user input into SQL.
- **Do not** store secrets in source control.

---

### Annotated Code Examples and Expected Outputs

#### Example: Best-Practice Query

```csharp
await using var conn = new SqlConnection(config.GetConnectionString("Default"));
await conn.OpenAsync();

await using var cmd = new SqlCommand(
    "SELECT Id, Name FROM Products WHERE Price > @min", conn);
cmd.Parameters.Add("@min", SqlDbType.Decimal).Value = 10m;

await using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
```

**Expected Output (example)**
```
1: Widget
2: Gadget
```

**Why this pattern is used:** It combines parameterization, disposal, async APIs, and streaming.

---

### Real-World Cases

- **High-throughput APIs** — Async + pooling + parameterization.
- **Reporting** — `DataReader` streaming.
- **Financial systems** — Transactions + `decimal` parameters.

---

### References

- ADO.NET overview — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview
- SQL Server Connection Pooling — Microsoft Learn — https://learn.microsoft.com/en-us/sql/connect/ado-net/sql-server-connection-pooling
- Transactions in ADO.NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/local-transactions
- `SqlCommand` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqlcommand
- `SqlDataReader` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqldatareader
- `SqlDataAdapter` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqldataadapter
- `DataSet` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.data.dataset
- `DataTable` — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/api/system.data.datatable

---