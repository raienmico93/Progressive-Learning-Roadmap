# EF Core Database Providers: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **EF Core 9–11** and covers the officially maintained and community‑recommended database providers for the major engines: **SQL Server**, **PostgreSQL**, **MySQL**, **SQLite**, **In‑Memory**, and **Azure Cosmos DB**. Each provider exposes engine‑specific features, limitations, and configuration surfaces. The choice of provider is a **production‑critical architectural decision**—it cannot be changed casually after schema and queries are written. Each requested topic is treated independently—no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A database provider is a NuGet package that implements EF Core’s provider model for a specific database engine, translating LINQ queries and `SaveChanges` operations into engine‑specific commands. |
| **Technical Definition** | An EF Core provider supplies an `IDatabaseProvider`, a `DatabaseProviderServices` collection, query SQL generators, type mappings, migration SQL generators, and an ADO.NET‑based execution strategy. Providers extend the shared relational or non‑relational infrastructure with engine‑specific features (JSON columns, `rowversion`, `xmin`, arrays, etc.). The provider is configured in `DbContextOptionsBuilder` via an engine‑specific `Use…()` extension method. |
| **Beginner‑Friendly Explanation** | The provider is the **adapter between EF Core and your database**. EF Core speaks a generic “database language”; the provider translates it into the exact dialect the database understands (T‑SQL for SQL Server, PL/pgSQL for PostgreSQL, etc.). Different providers expose different features and have different quirks—choosing the right one matters for performance, feature availability, and long‑term maintenance. |

### Key Characteristics

- **Not interchangeable at runtime** – The provider is fixed at configuration time; switching providers requires migration and query review.
- **Feature‑rich for relational engines** – SQL Server and PostgreSQL offer the deepest EF Core integration; MySQL and SQLite have limitations.
- **In‑Memory is not a production database** – It is a test double with behaviour that diverges significantly from relational engines.
- **Cosmos DB is non‑relational** – It uses a document model with different consistency and query semantics.
- **Provider‑specific migrations** – Migrations are generated per provider and are not portable across engines.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core or console project.
- Basic knowledge of **EF Core** (`DbContext`, `DbSet`, migrations).
- A database server or cloud instance for the chosen engine (SQL Server, PostgreSQL, MySQL, Cosmos DB) or an embedded engine (SQLite, In‑Memory).
- Familiarity with **dependency injection** for ASP.NET Core integration.

### Related Programming Areas

- **EF Core core concepts** – `DbContext`, migrations, LINQ queries, change tracking.
- **Connection string management** – Provider‑specific connection strings.
- **Database migrations** – Provider‑specific migration SQL and limitations.
- **Performance tuning** – Provider‑specific indexing, query plans, and batch operations.
- **Testing strategies** – In‑Memory vs. SQLite in‑memory vs. Testcontainers.

### Core Concepts / Features

1. **SQL Server** – The default Microsoft relational provider.
2. **PostgreSQL** – The Npgsql provider with advanced PostgreSQL features.
3. **MySQL** – The Pomelo community provider.
4. **SQLite** – The embedded, file‑based relational provider.
5. **In‑Memory** – The test‑double provider (not for production).
6. **Cosmos DB** – The Azure NoSQL document provider.
7. **Provider best practices** – Selection, testing, and operational guidance.


## 1. SQL Server (Microsoft.EntityFrameworkCore.SqlServer)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The SQL Server provider allows EF Core to be used with Microsoft SQL Server, Azure SQL Database, and Azure Synapse Analytics. |
| **Technical Definition** | The provider is maintained as part of the EF Core project and uses `Microsoft.Data.SqlClient` as its underlying ADO.NET provider. It supports SQL Server‑specific features including `rowversion` concurrency tokens, JSON indexes (EF Core 11+), compatibility levels, and Azure SQL/Synapse‑specific configuration methods (`UseAzureSql`, `UseAzureSynapse` in EF Core 9+). |
| **Beginner‑Friendly Explanation** | SQL Server is the **default, first‑class database for .NET applications**. It is the most tightly integrated provider, with full support for migrations, concurrency, and SQL Server‑specific features. If you are building a commercial .NET application and have no reason to choose otherwise, this is the safe default. |

### Purposes

- **To** use EF Core with **SQL Server, Azure SQL, or Azure Synapse** with full feature support.
- **To** leverage SQL Server‑specific features (`rowversion`, JSON indexes, compatibility levels).
- **To** benefit from **tight Microsoft ecosystem integration** (tooling, Azure, diagnostics).
- **To** serve as the **default relational provider** for commercial .NET projects.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseSqlServer(...)` | Configures the SQL Server provider. |
| Connection string | Standard SQL Server connection string (integrated security or SQL auth). |
| `UseAzureSql(...)` | EF Core 9+; for Azure SQL. |
| `UseAzureSynapse(...)` | EF Core 9+; for Azure Synapse Analytics. |

**Syntax rules:**

- **`UseSqlServer`** is the general entry point for SQL Server, Azure SQL, and Synapse.
- **`UseCompatibilityLevel(...)`** configures the SQL Server compatibility level EF uses when generating SQL.
- **JSON indexes** (EF Core 11+) can be created for paths inside complex types mapped to JSON columns.

**Constraints and limitations:**

- **Azure SQL and Synapse** have different feature sets—some SQL Server features may not be available.
- **Compatibility levels** may need to be aligned with the database’s actual compatibility level, not just EF’s configuration.

### Annotated Code Example

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Default"),
        sqlOptions => sqlOptions.UseCompatibilityLevel(170)));
```

**Expected Output:** EF Core connects to SQL Server and generates SQL compatible with SQL Server 2025 (compatibility level 170).

**Why this output?** `UseSqlServer` configures the provider; `UseCompatibilityLevel` controls the SQL feature set EF targets.

### Real‑World Cases

- **Enterprise .NET applications** on SQL Server or Azure SQL.
- **Commercial SaaS** using Azure SQL with elastic pools.
- **Azure Synapse** analytics workloads accessed via EF Core.

### References

- Microsoft SQL Server EF Core Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/sql-server/
- `UseAzureSql` and `UseAzureSynapse` (EF Core 9+) – https://learn.microsoft.com/en-us/ef/core/providers/sql-server/


## 2. PostgreSQL (Npgsql.EntityFrameworkCore.PostgreSQL)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Npgsql provider allows EF Core to be used with PostgreSQL, bringing full support for PostgreSQL‑specific features while maintaining standard EF Core patterns. |
| **Technical Definition** | The provider is maintained by the Npgsql project and built on the Npgsql ADO.NET provider. It supports PostgreSQL features such as arrays, JSONB, `xmin` concurrency tokens, enums, range types, and NodaTime integration. Configuration is consolidated in `UseNpgsql()` (EF Core 9+), which can configure the provider version, plugins (NodaTime), and enum mappings. |
| **Beginner‑Friendly Explanation** | PostgreSQL is the **most feature‑rich open‑source relational database**, and the Npgsql provider exposes its unique capabilities to EF Core. If you are building a free/open‑source project or need advanced data types (arrays, JSONB, ranges), PostgreSQL is the recommended choice. |

### Purposes

- **To** use EF Core with **PostgreSQL** while leveraging PostgreSQL‑specific features (arrays, JSONB, enums, ranges).
- **To** benefit from **strong open‑source ecosystem support** and active development.
- **To** serve as the **default relational provider** for free/open‑source .NET projects.
- **To** integrate with **NodaTime** for advanced date/time handling.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseNpgsql(
        builder.Configuration.GetConnectionString("Default"),
        o => o.SetPostgresVersion(13, 0)
              .UseNodaTime()
              .MapEnum<Mood>("mood")));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseNpgsql(...)` | Configures the Npgsql provider. |
| `SetPostgresVersion` | Ensures EF generates SQL compatible with the target PostgreSQL version. |
| `UseNodaTime()` | Enables NodaTime type mapping. |
| `MapEnum<T>(...)` | Maps a .NET enum to a PostgreSQL enum type. |

**Syntax rules:**

- **`UseNpgsql`** is the single entry point for provider configuration (EF Core 9+).
- **PostgreSQL version** should be set explicitly to avoid generating SQL for newer versions than the target.
- **NodaTime and enum plugins** require both EF‑level and ADO.NET‑level configuration; `UseNpgsql` handles both.

**Constraints and limitations:**

- **Provider version must match EF Core version** – Npgsql 10.x targets EF Core 10; Npgsql 9.x targets EF Core 9.
- **Some PostgreSQL features may not translate** to LINQ and require raw SQL.

### Annotated Code Example

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(connectionString));
```

**Expected Output:** EF Core connects to PostgreSQL and generates PL/pgSQL‑compatible SQL.

**Why this output?** `UseNpgsql` configures the Npgsql provider with the supplied connection string.

### Real‑World Cases

- **Open‑source SaaS** and community projects.
- **Applications using JSONB** for semi‑structured data.
- **Analytics with arrays and range types.**

### References

- Npgsql Entity Framework Core Provider – https://www.npgsql.org/efcore/index.html
- Npgsql EF Core configuration – https://www.npgsql.org/efcore/index.html?tabs=onconfiguring


## 3. MySQL (Pomelo.EntityFrameworkCore.MySql)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Pomelo provider allows EF Core to be used with MySQL (and MariaDB), built on top of MySqlConnector. |
| **Technical Definition** | Pomelo.EntityFrameworkCore.MySql is the **community‑maintained, recommended** EF Core provider for MySQL. It is not maintained by Microsoft or Oracle; it is built on `MySqlConnector` (a fully asynchronous ADO.NET provider). It supports MySQL‑specific features and is updated independently of EF Core releases. |
| **Beginner‑Friendly Explanation** | MySQL is a popular open‑source relational database, but Microsoft does not maintain an official EF Core provider for it. The community‑maintained **Pomelo** provider fills that gap and is the recommended choice. Because it is community‑maintained, its release cadence may lag behind EF Core versions. |

### Purposes

- **To** use EF Core with **MySQL or MariaDB**.
- **To** leverage the **recommended community provider** (Pomelo) built on MySqlConnector.
- **To** support MySQL‑specific features and connection handling.
- **To** provide an alternative for projects already standardized on MySQL.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(
        builder.Configuration.GetConnectionString("Default"),
        ServerVersion.AutoDetect(connectionString)));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseMySql(...)` | Configures the Pomelo MySQL provider. |
| `ServerVersion.AutoDetect(...)` | Automatically detects the MySQL server version for compatibility. |

**Syntax rules:**

- **Pomelo provider version must match EF Core version** – as of EF Core 10, Pomelo may not yet have a matching release; the application may need to stay on EF Core 9 until Pomelo ships EF Core 10 support.
- **MySqlConnector** is the underlying ADO.NET provider—not Oracle’s `MySql.Data`.
- **MariaDB** is supported through the same provider.

**Constraints and limitations:**

- **Community‑maintained** – release cadence is independent of EF Core; version alignment requires planning.
- **Not an official Microsoft provider** – feature completeness and bug fixes depend on the community.
- **Version lag** – EF Core 10 projects may need to remain on EF Core 9 until Pomelo releases a compatible version.

### Annotated Code Example

```csharp
var connectionString = builder.Configuration.GetConnectionString("Default");
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));
```

**Expected Output:** EF Core connects to MySQL and generates MySQL‑compatible SQL.

**Why this output?** `UseMySql` configures the Pomelo provider; `ServerVersion.AutoDetect` ensures the generated SQL matches the server version.

### Real‑World Cases

- **Legacy applications** already standardized on MySQL.
- **Cost‑sensitive projects** avoiding SQL Server licensing.
- **MariaDB deployments** using the same provider.

### References

- Pomelo.EntityFrameworkCore.MySql provider – https://entityframework-extensions.net/efcore-pomelo-mysql-provider
- Pomelo MySQL provider documentation – https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql


## 4. SQLite (Microsoft.EntityFrameworkCore.Sqlite)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The SQLite provider allows EF Core to be used with SQLite, a file‑based, embedded relational database engine. |
| **Technical Definition** | The provider is maintained by Microsoft and uses `Microsoft.Data.Sqlite` as the underlying ADO.NET provider. SQLite is a full relational engine, but it has **significant limitations** in type support (no native `DateTimeOffset`, `TimeSpan`, `ulong`, or `decimal`) and schema operations (many migrations operations require table rebuilds). It is suitable for **development, testing, and small‑scale embedded scenarios**, but not for high‑concurrency production workloads. |
| **Beginner‑Friendly Explanation** | SQLite is a **database in a file**. It requires no server, which makes it perfect for local development, mobile apps, and simple desktop tools. However, it is not designed for many concurrent writers and lacks several data types and schema operations that server databases support. |

### Purposes

- **To** provide a **zero‑configuration, file‑based database** for development and embedded scenarios.
- **To** support **testing** with a real relational engine (SQLite in‑memory mode) that behaves more like production than the In‑Memory provider.
- **To** enable **mobile and desktop apps** that need local relational storage.
- **To** offer a **lightweight alternative** for small‑scale applications.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("Default")));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseSqlite(...)` | Configures the SQLite provider. |
| Connection string | Path to the database file, or `:memory:` for in‑memory SQLite. |

**Syntax rules:**

- **`DateTimeOffset` and `TimeSpan`** are not natively supported; comparison and ordering are evaluated client‑side.
- **`decimal`** is not natively supported; EF Core translates operations to `ef_compare`/`EF_DECIMAL` functions. A value converter to `double` can be used if precision is not critical.
- **Migrations** – Many schema operations trigger a **table rebuild** (`NotSupportedException` if the table is not part of the EF model).

**Constraints and limitations:**

- **No schemas, sequences, or database‑generated concurrency tokens**.
- **Limited data type support** – `DateTimeOffset`, `TimeSpan`, `ulong`, `decimal` have restrictions.
- **Table rebuilds** for many migration operations – manual migration workarounds may be needed.
- **Concurrency** – SQLite uses database‑level locking; not suitable for high write concurrency.

### Annotated Code Example

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite("Data Source=app.db"));

// Workaround for decimal precision
modelBuilder.Entity<MyEntity>()
    .Property(e => e.DecimalProperty)
    .HasConversion<double>();
```

**Expected Output:** EF Core connects to the SQLite file `app.db` and stores decimals as `double` to avoid precision limitations.

**Why this output?** SQLite lacks native decimal support; the value converter maps `decimal` to `double` for storage and retrieval.

### Real‑World Cases

- **Local development and testing** with SQLite in‑memory mode.
- **Mobile apps** (MAUI, Xamarin) using SQLite for offline storage.
- **Desktop tools** and small utilities.

### References

- SQLite EF Core Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/sqlite/
- SQLite provider limitations – https://learn.microsoft.com/en-us/ef/core/providers/sqlite/limitations


## 5. In‑Memory (Microsoft.EntityFrameworkCore.InMemory)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The In‑Memory provider allows EF Core to be used with an in‑memory database that exists only for the lifetime of the process. |
| **Technical Definition** | The In‑Memory provider is maintained by Microsoft but is **explicitly not designed for production use**. It is a test double that does not enforce relational constraints (foreign keys, unique constraints) and does not behave like a real database. Microsoft discourages its use for testing, recommending **SQLite in‑memory mode** or a real database (via Testcontainers) instead. |
| **Beginner‑Friendly Explanation** | The In‑Memory provider is a **fake database for quick experiments**. It holds data in RAM and disappears when the app stops. It is convenient for demos and simple unit tests, but it does not act like a real database—it won’t enforce foreign keys or behave the same way under complex queries. Do **not** use it in production. |

### Purposes

- **To** provide a **quick, zero‑setup database** for demos, prototypes, and simple unit tests.
- **To** allow isolated tests that do not require a real database server.
- **To** serve as a temporary placeholder during early development.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseInMemoryDatabase(name)` | Configures the In‑Memory provider with a named database. |

**Syntax rules:**

- **Data is scoped to the service provider** – different `DbContext` instances with the same name share the database if they use the same root service provider.
- **No relational constraints** – foreign keys and unique indexes are not enforced.
- **No raw SQL** – `FromSqlRaw` is not supported.

**Constraints and limitations:**

- **Not for production** – explicitly warned by Microsoft.
- **Behaviour diverges from relational databases** – transactions, constraints, and complex queries behave differently.
- **Discouraged for testing** – SQLite in‑memory mode is recommended instead.
- **New features are not being added**.

### Annotated Code Example

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseInMemoryDatabase("TestDb"));

// In tests
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase("TestDb")
    .Options;
using var context = new AppDbContext(options);
context.Customers.Add(new Customer { Name = "Test" });
await context.SaveChangesAsync();
```

**Expected Output:** An in‑memory database named `TestDb` holds the customer for the lifetime of the process; no database server is required.

**Why this output?** `UseInMemoryDatabase` configures a process‑local, non‑persistent store that behaves like a simple dictionary, not a relational database.

### Real‑World Cases

- **Demos and prototypes** where a real database is overkill.
- **Simple unit tests** that do not depend on relational behaviour.
- **Early development** before a real database is provisioned.

### References

- EF Core In‑Memory Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/in-memory/


## 6. Azure Cosmos DB (Microsoft.EntityFrameworkCore.Cosmos)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Cosmos DB provider allows EF Core to be used with Azure Cosmos DB for NoSQL, a globally distributed, multi‑model NoSQL database. |
| **Technical Definition** | The provider is maintained as part of the EF Core project and supports the **NoSQL API** only. It maps entities to JSON documents in containers, supports embedded owned types (denormalization), and uses transactional batches for `SaveChanges` (EF Core 11+). Partitioning strategy must be planned in advance; session token management is the developer’s responsibility. |
| **Beginner‑Friendly Explanation** | Cosmos DB is a **planet‑scale NoSQL database** from Azure. Instead of tables and rows, it stores JSON documents in containers. The EF Core provider lets you use familiar LINQ queries and `SaveChanges`, but you must think in terms of **documents and partition keys** rather than joins and foreign keys. |

### Purposes

- **To** use EF Core with **Azure Cosmos DB for NoSQL**.
- **To** store and query **JSON document data** with global distribution and low latency.
- **To** support **denormalized, aggregate‑oriented** models (embedded owned types).
- **To** leverage **transactional batches** for atomic multi‑document writes (EF Core 11+).

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<OrderContext>(options =>
    options.UseCosmos(
        accountEndpoint: "https://localhost:8081",
        accountKey: "<key>",
        databaseName: "OrdersDB"));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseCosmos(...)` | Configures the Cosmos DB provider. |
| `accountEndpoint` | The Cosmos DB account endpoint. |
| `accountKey` or token credential | Authentication—RBAC with token credentials is recommended. |
| `databaseName` | The target database. |

**Syntax rules:**

- **Only Azure Cosmos DB for NoSQL is supported**—not the MongoDB, Cassandra, Gremlin, or Table APIs.
- **Partitioning strategy must be planned in advance**—the partition key is central to performance and scalability.
- **`EnsureCreatedAsync` should only be called during deployment**, not normal operation, due to performance implications.
- **RBAC with token credentials** is the recommended access‑control mechanism; `EnsureCreatedAsync` does not support RBAC for management‑plane operations.

**Constraints and limitations:**

- **No relational joins** – data must be denormalized or embedded.
- **Transactional batches** are limited to documents with the same partition key.
- **Session token management** – the developer must handle consistency tokens for session consistency.
- **No migrations in the relational sense** – container and partition‑key changes require careful planning.

### Annotated Code Example

```csharp
public class Order
{
    public int Id { get; set; }
    public string PartitionKey { get; set; } = "";
    public StreetAddress ShippingAddress { get; set; } = new();
}

public class StreetAddress
{
    public string Street { get; set; } = "";
    public string City { get; set; } = "";
}

// Save an order as a single document with embedded address
using var context = new OrderContext();
context.Add(new Order
{
    Id = 1,
    PartitionKey = "customer-42",
    ShippingAddress = new StreetAddress { City = "London", Street = "221 B Baker St" }
});
await context.SaveChangesAsync();
```

**Expected Output:** A single JSON document is stored in the container, containing the order and its embedded address.

**Why this output?** Cosmos DB stores denormalized documents; owned types are embedded rather than stored in separate tables.

### Real‑World Cases

- **Globally distributed applications** requiring low‑latency reads and writes.
- **IoT and telemetry** storing high‑volume JSON documents.
- **Personalization and session data** with flexible schemas.

### References

- EF Core Azure Cosmos DB Provider – https://learn.microsoft.com/en-us/ef/core/providers/cosmos/
- Cosmos DB provider modeling and partitioning – https://learn.microsoft.com/en-us/ef/core/providers/cosmos/modeling


## 7. Database Provider Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Provider best practices are guidelines for choosing, configuring, testing, and operating EF Core database providers that are correct, performant, and maintainable. |
| **Technical Definition** | Best practices encompass **provider selection** (SQL Server or PostgreSQL for new relational projects), **version alignment** (provider version must match EF Core version), **testing strategy** (SQLite in‑memory or Testcontainers instead of In‑Memory), **migration management** (provider‑specific scripts and deployment), **connection security** (secure storage of connection strings and keys), and **performance** (provider‑specific indexing and batching). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for working with database providers—they help you avoid choosing a provider you cannot use in production, testing against a fake database, and running into version mismatches. |

### Purposes

- **To** choose the **right provider** for the project’s requirements and constraints.
- **To** ensure **version compatibility** between EF Core and the provider.
- **To** test against **realistic database behaviour** (SQLite in‑memory or a real engine).
- **To** manage **migrations and deployment** safely per provider.
- **To** protect **connection strings and credentials**.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Provider selection** | Choose **SQL Server** for commercial .NET projects; **PostgreSQL** for free/open‑source projects. | Choose In‑Memory for production; choose MySQL without accepting community‑maintenance risk. |
| **Version alignment** | Match the provider version to the EF Core version (e.g., Npgsql 10.x for EF Core 10). | Mix EF Core 10 with a provider that only supports EF Core 9. |
| **Testing** | Use **SQLite in‑memory** or **Testcontainers** with the real engine. | Use the **In‑Memory provider** for tests that depend on relational behaviour. |
| **SQLite limitations** | Use `DateTime` instead of `DateTimeOffset`; convert `decimal` to `double` if precision permits. | Assume SQLite supports all relational features. |
| **Cosmos DB** | Plan the **partition key** before modeling; use RBAC token credentials; call `EnsureCreatedAsync` only during deployment. | Assume relational joins are available; hard‑code account keys in source. |
| **Migrations** | Generate **provider‑specific idempotent scripts** for production. | Apply migrations blindly across different providers. |
| **Connection security** | Store connection strings in **Azure Key Vault** or User Secrets. | Commit connection strings to source control. |
| **Batch operations** | Use **EFCore.BulkExtensions** for large batch operations. | Use EF Core `AddRange` for thousands of rows. |

### Annotated Code Example (Best‑Practice Multi‑Provider Configuration)

```csharp
// Production: SQL Server
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Production")));

// Testing: SQLite in-memory (not In-Memory provider)
public static AppDbContext CreateTestContext()
{
    var connection = new SqliteConnection("DataSource=:memory:");
    connection.Open();
    var options = new DbContextOptionsBuilder<AppDbContext>()
        .UseSqlite(connection)
        .Options;
    var context = new AppDbContext(options);
    context.Database.EnsureCreated();
    return context;
}
```

**Why this follows best practices:**

- **SQL Server is used for production**, matching the commercial .NET project recommendation.
- **SQLite in‑memory is used for testing**, providing relational behaviour that the In‑Memory provider lacks.
- **Connection strings are loaded from configuration**, not hard‑coded.
- **The test context is disposed** after each test, ensuring isolation.

### Real-World Cases

- **Multi‑environment setups:** SQL Server in production, SQLite in development/testing.
- **Open‑source projects:** PostgreSQL with Npgsql, tested with Testcontainers.
- **Azure‑native apps:** SQL Server/Azure SQL for relational data, Cosmos DB for globally distributed documents.

### References

- Choosing a database for .NET projects – https://github.com/AterDev/Perigon.docs/blob/main/Content/docs/Perigon/en-US/10.0/Best-Practices/Database.md
- Testing EF Core applications – https://learn.microsoft.com/en-us/ef/core/testing/
- EFCore.BulkExtensions – https://github.com/borisdj/EFCore.BulkExtensions


## Summary Table: EF Core Providers at a Glance

| Provider | Package | Maintained By | Best For | Key Limitation |
|----------|---------|---------------|----------|----------------|
| **SQL Server** | `Microsoft.EntityFrameworkCore.SqlServer` | Microsoft | Commercial .NET, Azure SQL, Synapse | Azure feature differences |
| **PostgreSQL** | `Npgsql.EntityFrameworkCore.PostgreSQL` | Npgsql project | Open‑source, advanced types (JSONB, arrays, ranges) | Version alignment with EF Core |
| **MySQL** | `Pomelo.EntityFrameworkCore.MySql` | Community (Pomelo) | MySQL/MariaDB projects | Community‑maintained; version lag |
| **SQLite** | `Microsoft.EntityFrameworkCore.Sqlite` | Microsoft | Development, testing, mobile, embedded | Limited types; table rebuilds; concurrency |
| **In‑Memory** | `Microsoft.EntityFrameworkCore.InMemory` | Microsoft | Demos, simple unit tests | **Not for production**; no relational behaviour |
| **Cosmos DB** | `Microsoft.EntityFrameworkCore.Cosmos` | Microsoft | Global NoSQL document workloads | No joins; partition planning; session tokens |

---

## References (Full List)

- Microsoft SQL Server EF Core Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/sql-server/
- Npgsql Entity Framework Core Provider – https://www.npgsql.org/efcore/index.html
- Pomelo.EntityFrameworkCore.MySql provider – https://entityframework-extensions.net/efcore-pomelo-mysql-provider
- SQLite EF Core Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/sqlite/
- SQLite provider limitations – https://learn.microsoft.com/en-us/ef/core/providers/sqlite/limitations
- EF Core In‑Memory Database Provider – https://learn.microsoft.com/en-us/ef/core/providers/in-memory/
- EF Core Azure Cosmos DB Provider – https://learn.microsoft.com/en-us/ef/core/providers/cosmos/
- Cosmos DB provider modeling and partitioning – https://learn.microsoft.com/en-us/ef/core/providers/cosmos/modeling
- Choosing a database for .NET projects – https://github.com/AterDev/Perigon.docs/blob/main/Content/docs/Perigon/en-US/10.0/Best-Practices/Database.md
- Testing EF Core applications – https://learn.microsoft.com/en-us/ef/core/testing/
- EFCore.BulkExtensions – https://github.com/borisdj/EFCore.BulkExtensions
- EF Core releases and planning – https://learn.microsoft.com/en-us/ef/core/what-is-new/release-planning