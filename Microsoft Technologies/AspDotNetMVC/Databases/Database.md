# Database Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet covers **database fundamentals** as they apply to modern application development, with examples oriented toward **ASP.NET Core + Entity Framework Core** (.NET 6–10) as the reference stack. The concepts themselves are technology‑agnostic: relational databases (SQL Server, PostgreSQL, MySQL, SQLite) and NoSQL databases (document, key‑value, wide‑column, graph) are covered. Where behavior or terminology differs across engines, it is flagged. Provider‑specific SQL features (e.g., `rowversion` in SQL Server vs. `xmin` in PostgreSQL) are identified where they affect application code. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A database is an organized collection of structured or semi‑structured data stored and accessed electronically, typically managed by a Database Management System (DBMS) that enforces integrity, concurrency, security, and durability. |
| **Technical Definition** | A database system comprises the **data** itself, a **schema** describing its structure, a **DBMS** managing storage and access, and a **query interface** (SQL for relational systems, API/query languages for NoSQL systems). Relational databases store data in **tables** with rows and columns, enforce **constraints** (primary keys, foreign keys, check constraints), and support **ACID transactions**. NoSQL databases relax some of these guarantees in exchange for scalability, flexible schemas, or specialized access patterns (document, key‑value, wide‑column, graph). |
| **Beginner‑Friendly Explanation** | A database is the **long‑term memory of your application**. Instead of keeping users, orders, or posts in memory (which disappears when the app restarts), you store them in a database that persists across restarts, handles many users at once, and protects against data loss. Think of a relational database as a set of **well‑organized spreadsheets** with links between them; think of a NoSQL database as a set of **flexible containers** optimized for particular shapes of data. |

### Key Characteristics

- **Persistence** – Data survives application restarts, crashes, and deployments.
- **Concurrency** – Multiple users and processes can read and write simultaneously with defined isolation guarantees.
- **Integrity** – Constraints (primary keys, foreign keys, uniqueness, checks) prevent invalid data.
- **Durability** – Committed data survives system failures (via write‑ahead logging, replication, backups).
- **Queryability** – Structured query languages (SQL) or specialized APIs enable efficient retrieval.
- **Security** – Authentication, authorization, encryption at rest and in transit.

### Prerequisites

- Basic understanding of **data structures** (tables, rows, columns, keys).
- Familiarity with **C#** and **Entity Framework Core** (for the .NET examples).
- A database server or embedded engine (SQL Server, PostgreSQL, SQLite).
- Basic knowledge of **SQL** (SELECT, INSERT, UPDATE, DELETE, JOIN).

### Related Programming Areas

- **Entity Framework Core** – The ORM that maps .NET classes to database tables.
- **ASP.NET Core** – Applications use databases through repositories, `DbContext`, or raw ADO.NET.
- **Caching** – Reduces database load; complements the database tier.
- **Migrations** – Version‑controlled schema changes (EF Core migrations, Flyway, Liquibase).
- **Transactions** – Grouping operations atomically (ACID).
- **Indexing** – Speeds up queries at the cost of write performance.

### Core Concepts / Features

1. **Databases** – The overall concept, DBMS, persistence, and querying.
2. **Relational databases** – Tables, keys, constraints, normalization, SQL, ACID, and EF Core integration.
3. **NoSQL databases** – Document, key‑value, wide‑column, and graph models; eventual consistency; when to choose them.
4. **Database best practices** – Schema design, indexing, security, backups, migrations, and performance.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real‑World Cases**, and **References**.

---

## 1. Databases (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A database is an organized collection of data managed by a DBMS that provides persistence, querying, concurrency control, and integrity enforcement. |
| **Technical Definition** | A database system consists of (1) **storage** (files, pages, buffers), (2) a **DBMS** that manages access, transactions, and recovery, (3) a **schema** defining structure and constraints, and (4) a **query interface** (SQL or API). ACID properties—Atomicity, Consistency, Isolation, Durability—define the transactional guarantees of relational systems; NoSQL systems may offer BASE (Basically Available, Soft state, Eventual consistency) instead. |
| **Beginner‑Friendly Explanation** | A database is the **filing cabinet** of your app. It stores data in a way that can be searched, updated, and protected against loss. The DBMS is the **librarian**—it decides who can access what, keeps the shelves organized, and ensures that two people editing the same file do not corrupt it. |

### Purposes

- **To** persist application data across restarts and deployments.
- **To** allow many users and processes to read and write concurrently without corrupting data.
- **To** enforce data integrity rules (required fields, uniqueness, referential integrity) at the storage layer.
- **To** provide efficient querying and retrieval for large datasets.
- **To** enable backup, recovery, and replication for durability and availability.
- **To** centralize access control, auditing, and encryption.

### Syntax Rules and Structure

#### General Architecture

```text
Application (ASP.NET Core)
   │  uses ORM (EF Core) or raw ADO.NET
   ▼
Database Driver / Provider
   │  sends SQL or protocol messages
   ▼
DBMS (SQL Server, PostgreSQL, ...)
   │  manages
   ▼
Storage (files, WAL, indexes)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| **Application** | Sends queries and receives results. |
| **ORM / driver** | Translates between C# objects and SQL/protocol. |
| **DBMS** | Parses queries, plans execution, manages transactions. |
| **Storage** | Persists data in files, pages, and indexes. |

#### General Syntax (Querying via EF Core)

```csharp
var customers = await _context.Customers
    .Where(c => c.IsActive)
    .OrderBy(c => c.Name)
    .ToListAsync();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `_context.Customers` | `DbSet<Customer>` mapped to the `Customers` table. |
| `.Where(...)` | Translated to SQL `WHERE`. |
| `.OrderBy(...)` | Translated to SQL `ORDER BY`. |
| `.ToListAsync()` | Executes the query and materializes results. |

#### Syntax Rules

- **All database access should be asynchronous** in ASP.NET Core—use `ToListAsync`, `FirstOrDefaultAsync`, `SaveChangesAsync`.
- **`DbContext` is Scoped**—do not share it across requests or threads.
- **Queries are translated to SQL** by the provider; unsupported expressions throw at runtime.
- **Connections are pooled** by ADO.NET; use `await using` for explicit disposal of `DbContext` only when managing it manually.

#### Constraints and Limitations

- **Database round‑trips are expensive**—batch queries and use `Include`/projection to avoid N+1.
- **Transactions have isolation costs**—long transactions block or slow other writers.
- **Schema changes require migrations**—in production, migrations must be applied carefully.

### Annotated Code Example (Minimal EF Core Setup and Query)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// A repository/service using the context
public class CustomerService
{
    private readonly AppDbContext _context;
    public CustomerService(AppDbContext context) => _context = context;

    public async Task<List<Customer>> GetActiveAsync()
    {
        return await _context.Customers
            .AsNoTracking()
            .Where(c => c.IsActive)
            .ToListAsync();
    }
}

// AppDbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Customer> Customers => Set<Customer>();
}

// Entity
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public bool IsActive { get; set; }
}
```

**Expected Output:** A list of active customers retrieved with a single SQL query (e.g., `SELECT * FROM Customers WHERE IsActive = 1`).

**Why this output?** EF Core translates the LINQ query into SQL, executes it, and materializes `Customer` objects. `AsNoTracking` improves performance for read‑only queries.

### Real‑World Cases

- **E‑commerce:** Products, orders, customers, inventory.
- **SaaS applications:** Tenants, users, subscriptions, audit logs.
- **Content management:** Articles, authors, categories, media.
- **Analytics:** Events, metrics, time‑series data.

### References

- Entity Framework Core documentation – https://learn.microsoft.com/en-us/ef/core/
- Database design basics – https://learn.microsoft.com/en-us/office/troubleshoot/access/database-design-basics
- ACID properties – https://en.wikipedia.org/wiki/ACID

---

## 2. Relational Databases

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A relational database stores data in **tables** consisting of rows and columns, with relationships between tables expressed via keys, and enforces integrity through constraints. |
| **Technical Definition** | Relational databases are based on the relational model (Codd, 1970). Data is stored in **relations** (tables) with **attributes** (columns) and **tuples** (rows). **Primary keys** uniquely identify rows; **foreign keys** enforce referential integrity between tables. **SQL** is the standard query language. Relational systems offer **ACID transactions**, **normalization** to reduce redundancy, and **indexes** for performance. Examples: SQL Server, PostgreSQL, MySQL, Oracle, SQLite. |
| **Beginner‑Friendly Explanation** | A relational database is like a set of **well‑organized spreadsheets** that can reference each other. An `Orders` table has a `CustomerId` column that points to a row in the `Customers` table. The database enforces that you cannot create an order for a customer that does not exist. This structure keeps data consistent and makes complex queries (joins) possible. |

### Purposes

- **To** store structured data with a **well‑defined schema** and enforced integrity constraints.
- **To** support **complex queries** across related tables using JOINs, aggregations, and subqueries.
- **To** provide **ACID transactions** so multi‑step operations either fully succeed or fully roll back.
- **To** enable **normalization** to eliminate redundancy and update anomalies.
- **To** leverage mature tooling, ecosystems, and expertise (SQL, migrations, backups, replication).
- **To** support **strong consistency** guarantees required by financial, inventory, and transactional systems.

### Syntax Rules and Structure

#### Table and Relationship Structure

```sql
CREATE TABLE Customers (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100) NOT NULL,
    Email NVARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY(1,1),
    CustomerId INT NOT NULL,
    Total DECIMAL(18,2) NOT NULL CHECK (Total >= 0),
    CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT FK_Orders_Customers
        FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `PRIMARY KEY` | Uniquely identifies each row. |
| `NOT NULL` | Column cannot contain `NULL`. |
| `UNIQUE` | Column values must be unique across rows. |
| `FOREIGN KEY` | Enforces that `CustomerId` references an existing `Customers.Id`. |
| `CHECK` | Enforces a domain rule (`Total >= 0`). |
| `DEFAULT` | Supplies a value when none is provided. |

#### General Syntax (EF Core Entity Configuration)

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;
    public decimal Total { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class AppDbContext : DbContext
{
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>(entity =>
        {
            entity.HasKey(o => o.Id);
            entity.Property(o => o.Total).HasPrecision(18, 2);
            entity.HasOne(o => o.Customer)
                  .WithMany(c => c.Orders)
                  .HasForeignKey(o => o.CustomerId)
                  .OnDelete(DeleteBehavior.Restrict);
        });
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `HasKey` | Explicitly configures the primary key. |
| `HasPrecision` | Maps `decimal` to `DECIMAL(18,2)`. |
| `HasOne/WithMany` | Defines the one‑to‑many relationship. |
| `HasForeignKey` | Specifies the foreign key property. |
| `OnDelete(DeleteBehavior.Restrict)` | Prevents deleting a customer with orders. |

#### Syntax Rules

- **Every table should have a primary key**—EF Core requires it for tracking.
- **Use foreign keys for relationships**—do not store related data as comma‑separated strings.
- **Choose appropriate column types**—`NVARCHAR(255)` vs. `NVARCHAR(MAX)` affects performance and storage.
- **Normalize to 3NF unless there is a justified denormalization** for performance.
- **Use transactions for multi‑step operations**—`SaveChanges` is atomic by default, but multiple calls need explicit transactions.

#### Constraints and Limitations

- **Schema changes require migrations**—altering tables on live data needs care.
- **Joins can be expensive**—large multi‑table joins need appropriate indexes and query tuning.
- **Vertical scaling has limits**—very large write loads may need partitioning or sharding.
- **Strong consistency can limit availability**—distributed relational setups involve trade‑offs.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Defining Related Entities and Querying with a Join

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Order> Orders { get; set; } = new();
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; } = null!;
    public decimal Total { get; set; }
}

// Query with join via navigation property
var customersWithOrders = await _context.Customers
    .Include(c => c.Orders)
    .Where(c => c.Orders.Any(o => o.Total > 100))
    .ToListAsync();
```

**Step-by-step explanation:**

1. `Customer` has a `List<Order>` navigation property; `Order` has a `CustomerId` foreign key.
2. `Include(c => c.Orders)` eager‑loads the related orders (translates to a JOIN or split query).
3. `.Where(c => c.Orders.Any(...))` translates to an `EXISTS`/`IN` subquery.
4. The result is a list of `Customer` objects with populated `Orders`.

**Expected Output:** Customers who have at least one order with `Total > 100`, each with their orders loaded.

**Why this output?** EF Core generates SQL that joins or uses a subquery to filter and load related data, then materializes the object graph.

#### Example 2: Transaction Across Multiple Operations

```csharp
await using var transaction = await _context.Database.BeginTransactionAsync();
try
{
    var order = new Order { CustomerId = customerId, Total = 250m };
    _context.Orders.Add(order);
    await _context.SaveChangesAsync();

    var inventory = await _context.Inventory.FindAsync(productId);
    inventory!.Stock -= 1;
    await _context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

**Step-by-step explanation:**

1. A transaction is started explicitly.
2. Both `SaveChangesAsync` calls participate in the transaction.
3. `CommitAsync` makes the changes durable atomically.
4. Any exception triggers `RollbackAsync`, undoing both changes.

**Expected Output:** Either both the order and inventory change are persisted, or neither is.

**Why this output?** The explicit transaction groups the two `SaveChanges` calls into one atomic unit.

### Real-World Cases

- **Banking:** Transfers require atomic debit/credit operations (ACID).
- **E‑commerce:** Orders, payments, and inventory must be consistent.
- **HR systems:** Employees, departments, roles with referential integrity.
- **Reporting:** Complex JOINs and aggregations across normalized schemas.

### References

- Relational model (Codd) – https://en.wikipedia.org/wiki/Relational_model
- Entity Framework Core relationships – https://learn.microsoft.com/en-us/ef/core/modeling/relationships
- Normalization (database design) – https://learn.microsoft.com/en-us/office/troubleshoot/access/database-design-basics

---

## 3. NoSQL Databases

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | NoSQL databases are non‑relational databases designed for flexible schemas, horizontal scalability, and specialized data models (document, key‑value, wide‑column, graph). |
| **Technical Definition** | NoSQL systems typically relax strict relational constraints and may trade ACID for **BASE** (Basically Available, Soft state, Eventual consistency). Major categories: **document** (MongoDB, Couchbase, RavenDB—store JSON‑like documents), **key‑value** (Redis, DynamoDB—map keys to values), **wide‑column** (Cassandra, HBase—column families across distributed nodes), and **graph** (Neo4j—nodes and edges for traversal). They excel at horizontal scaling, high write throughput, and schema flexibility. |
| **Beginner‑Friendly Explanation** | NoSQL databases are **purpose‑built containers** for data that does not fit neatly into tables. A document database stores a whole JSON‑like object (e.g., an order with its line items embedded) as one record. A key‑value store is like a giant dictionary. A graph database is optimized for “who is connected to whom.” They trade some of the guarantees of relational databases for flexibility and scale. |

### Purposes

- **To** handle **flexible or evolving schemas** where documents of the same collection can differ.
- **To** scale **horizontally** across many servers (sharding, partitioning).
- **To** support **high write throughput** and low‑latency access patterns.
- **To** store **denormalized, aggregate‑oriented** data in a single record (avoids joins).
- **To** provide **specialized models** (graph traversal, time‑series, full‑text search).
- **To** support **eventual consistency** when strong consistency is not required.

### Syntax Rules and Structure

#### Category Comparison

| Category | Example | Data Model | Typical Use |
|----------|---------|------------|-------------|
| **Document** | MongoDB, RavenDB | JSON/BSON documents in collections | Content, catalogs, user profiles |
| **Key‑value** | Redis, DynamoDB | Key → value (string, blob, JSON) | Caching, sessions, feature flags |
| **Wide‑column** | Cassandra, HBase | Rows with dynamic column families | Time‑series, messaging, large‑scale writes |
| **Graph** | Neo4j | Nodes and relationships | Social networks, recommendations, fraud detection |

#### General Syntax (MongoDB Document Example)

```javascript
// Insert a document
db.orders.insertOne({
  orderId: "ORD-1001",
  customer: { id: 42, name: "Alice" },
  items: [
    { sku: "LAPTOP-01", qty: 1, price: 999.00 },
    { sku: "MOUSE-02", qty: 2, price: 25.00 }
  ],
  total: 1049.00,
  createdAt: ISODate("2026-01-15T10:00:00Z")
});

// Query
db.orders.find({ "customer.id": 42, total: { $gt: 100 } });
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `insertOne` | Inserts a single document into a collection. |
| Embedded `customer` object | Denormalized related data—no join needed. |
| `items` array | Embedded line items. |
| `find({...})` | Queries by nested fields and operators (`$gt`). |

#### General Syntax (Redis Key‑Value Example)

```text
SET session:abc123 "{""userId"":42}" EX 3600
GET session:abc123
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SET key value EX 3600` | Sets a key with a 1‑hour expiry. |
| `GET key` | Retrieves the value. |

#### Syntax Rules

- **Model data around access patterns**—in NoSQL, you design collections/tables based on how they will be queried.
- **Denormalize intentionally**—embed data that is read together.
- **Choose a partition/shard key carefully**—it determines data distribution and query performance.
- **Understand the consistency model**—eventual consistency means reads may return stale data briefly.

#### Constraints and Limitations

- **No joins (or limited joins)**—data must be modeled to avoid them.
- **Eventual consistency** may not suit financial transactions.
- **Schema flexibility** can lead to inconsistent data if not validated at the application layer.
- **Limited multi‑document transactions** in some systems (though MongoDB and others now support them).

### Annotated Complete Step-by-Step Code Examples

#### Example 1: MongoDB Document CRUD via EF Core Provider

```csharp
// EF Core MongoDB provider (MongoDB.EntityFrameworkCore)
public class OrderDocument
{
    public string Id { get; set; } = "";
    public string OrderId { get; set; } = "";
    public CustomerInfo Customer { get; set; } = new();
    public List<OrderItem> Items { get; set; } = new();
    public decimal Total { get; set; }
}

public class CustomerInfo { public int Id { get; set; } public string Name { get; set; } = ""; }
public class OrderItem { public string Sku { get; set; } = ""; public int Qty { get; set; } public decimal Price { get; set; } }

// Query
var bigOrders = await _mongoContext.Orders
    .Where(o => o.Customer.Id == 42 && o.Total > 100)
    .ToListAsync();
```

**Step-by-step explanation:**

1. The `OrderDocument` embeds customer information and line items—no joins needed.
2. A query filters by nested fields (`Customer.Id`) and `Total`.
3. The provider translates the LINQ expression to a MongoDB query.

**Expected Output:** A list of order documents belonging to customer 42 with `Total > 100`, each containing embedded items.

**Why this output?** Document databases store the entire aggregate in one record; queries operate on nested fields without joins.

#### Example 2: Redis for Session Caching

```csharp
var cacheKey = $"session:{sessionId}";
await _redis.StringSetAsync(cacheKey, JsonSerializer.Serialize(session), TimeSpan.FromHours(1));

var cached = await _redis.StringGetAsync(cacheKey);
if (!cached.IsNullOrEmpty)
{
    var session = JsonSerializer.Deserialize<SessionData>(cached!);
}
```

**Expected Output:** Session data is cached for one hour; subsequent reads hit Redis instead of the database.

**Why this output?** Key‑value stores excel at simple, high‑speed lookups by key.

### Real-World Cases

- **Content platforms:** Article bodies and metadata stored as documents.
- **Real‑time analytics:** Time‑series data in wide‑column stores.
- **Caching and sessions:** Redis for fast, ephemeral data.
- **Social graphs:** Neo4j for friend‑of‑friend and recommendation queries.
- **IoT:** High‑volume device telemetry in wide‑column or time‑series databases.

### References

- NoSQL database types – https://www.mongodb.com/resources/basics/databases/nosql-explained
- MongoDB documentation – https://www.mongodb.com/docs/
- Redis documentation – https://redis.io/docs/
- CAP theorem – https://en.wikipedia.org/wiki/CAP_theorem

---

## 4. Database Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Database best practices are guidelines for designing, accessing, securing, and operating databases that are correct, performant, secure, and maintainable. |
| **Technical Definition** | Best practices span **schema design** (normalization, keys, constraints), **indexing** (covering indexes, avoiding over‑indexing), **query performance** (avoiding N+1, parameterization), **transactions** (isolation levels, scope), **security** (least privilege, encryption, parameterized queries), **migrations** (versioned, reversible, tested), **backup and recovery** (RPO/RTO, tested restores), and **operational monitoring** (slow query logs, connection pooling). |
| **Beginner‑Friendly Explanation** | Database best practices are the **“do’s and don’ts”** for working with databases—they help you avoid slow queries, data corruption, security breaches, and unrecoverable failures. |

### Purposes

- **To** ensure **data integrity** through keys, constraints, and normalized design.
- **To** maintain **performance** as data grows through indexing and query optimization.
- **To** prevent **SQL injection** through parameterized queries and least‑privilege accounts.
- **To** support **safe schema evolution** with versioned migrations.
- **To** guarantee **recoverability** with tested backups and documented recovery procedures.
- **To** protect **sensitive data** with encryption at rest and in transit.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Schema design** | Define primary keys, foreign keys, and constraints. | Store multiple values in one column (comma‑separated lists). |
| **Normalization** | Normalize to 3NF by default; denormalize only with measured justification. | Duplicate data without a plan to keep copies in sync. |
| **Indexing** | Index foreign keys, frequently filtered columns, and ORDER BY columns. | Over‑index (slows writes) or ignore missing indexes (slow reads). |
| **Query performance** | Use `AsNoTracking` for read‑only queries; project only needed columns. | Send `SELECT *` when only a few columns are needed. |
| **N+1 queries** | Use `Include`/`ThenInclude` or projection to avoid per‑row queries. | Loop over results and query per item. |
| **SQL injection** | Use parameterized queries (EF Core does this automatically). | Concatenate user input into SQL strings. |
| **Transactions** | Use explicit transactions for multi‑step operations; keep them short. | Hold transactions open during user interaction or HTTP calls. |
| **Connection management** | Rely on connection pooling; keep connections short‑lived. | Open/close connections per query manually or leak connections. |
| **Migrations** | Use EF Core migrations (or Flyway/Liquibase); test on a copy of production data. | Modify production schemas manually without version control. |
| **Backups** | Automate backups; **test restores** regularly. | Assume backups work without ever restoring them. |
| **Security** | Use least‑privilege accounts; encrypt at rest and in transit. | Use `sa` or `postgres` superuser accounts from the application. |
| **Sensitive data** | Hash passwords (PBKDF2/Argon2); encrypt PII at rest. | Store plain‑text passwords or unencrypted PII. |
| **Monitoring** | Enable slow‑query logs; monitor connection counts and replication lag. | Wait for users to report performance problems. |

### Annotated Code Example (Best‑Practice Query and Transaction)

```csharp
public async Task<OrderSummary> GetOrderSummaryAsync(int orderId)
{
    // Projection: fetch only needed columns, no tracking
    var summary = await _context.Orders
        .AsNoTracking()
        .Where(o => o.Id == orderId)
        .Select(o => new OrderSummary
        {
            OrderId = o.Id,
            CustomerName = o.Customer.Name,
            Total = o.Total,
            ItemCount = o.Items.Count
        })
        .FirstOrDefaultAsync();

    return summary ?? throw new NotFoundException();
}

public async Task PlaceOrderAsync(CreateOrderCommand command)
{
    await using var tx = await _context.Database.BeginTransactionAsync();
    try
    {
        var order = new Order { CustomerId = command.CustomerId, Total = command.Total };
        _context.Orders.Add(order);

        var stock = await _context.Inventory
            .Where(i => i.ProductId == command.ProductId)
            .FirstOrDefaultAsync();

        if (stock is null || stock.Quantity < command.Quantity)
            throw new InvalidOperationException("Insufficient stock.");

        stock.Quantity -= command.Quantity;
        await _context.SaveChangesAsync();
        await tx.CommitAsync();
    }
    catch
    {
        await tx.RollbackAsync();
        throw;
    }
}
```

**Why this follows best practices:**

- **Projection** fetches only the columns needed for the summary.
- **`AsNoTracking`** avoids change‑tracking overhead for read‑only queries.
- **Explicit transaction** groups the order insert and inventory decrement atomically.
- **Stock check** prevents overselling—a business rule enforced under transaction.
- **Rollback on failure** ensures no partial state persists.
- **Async throughout** avoids blocking threads in ASP.NET Core.

### Real-World Cases

- **E‑commerce checkout:** Atomic order + inventory + payment record.
- **Multi‑tenant SaaS:** Tenant‑scoped queries and row‑level security.
- **Financial systems:** Serialized transactions with appropriate isolation levels.
- **Analytics workloads:** Read replicas and indexed projection queries.

### References

- Database design best practices – https://learn.microsoft.com/en-us/sql/relational-databases/database-design
- EF Core performance best practices – https://learn.microsoft.com/en-us/ef/core/performance/
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Index design guidance – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide

---

## Summary Table: Database Fundamentals at a Glance

| Concept | Primary Role | Key Tool / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Databases** | Persist and query application data | EF Core, ADO.NET, SQL | Every application with durable state |
| **Relational databases** | Structured data with integrity and ACID | Tables, keys, constraints, SQL, EF Core | Transactions, relationships, reporting |
| **NoSQL databases** | Flexible, scalable, aggregate‑oriented storage | Document, key‑value, wide‑column, graph | High scale, flexible schemas, specialized access |
| **Best practices** | Correct, fast, secure, recoverable | Constraints, indexes, transactions, migrations, backups, parameterization | Every production database |

---

## References (Full List)

- Entity Framework Core documentation – https://learn.microsoft.com/en-us/ef/core/
- EF Core performance best practices – https://learn.microsoft.com/en-us/ef/core/performance/
- Entity Framework Core relationships – https://learn.microsoft.com/en-us/ef/core/modeling/relationships
- Database design basics – https://learn.microsoft.com/en-us/office/troubleshoot/access/database-design-basics
- SQL Server index design guide – https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide
- Relational model (Codd) – https://en.wikipedia.org/wiki/Relational_model
- ACID properties – https://en.wikipedia.org/wiki/ACID
- CAP theorem – https://en.wikipedia.org/wiki/CAP_theorem
- NoSQL database types – https://www.mongodb.com/resources/basics/databases/nosql-explained
- MongoDB documentation – https://www.mongodb.com/docs/
- Redis documentation – https://redis.io/docs/
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Database Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html
- Microsoft SQL Server database design – https://learn.microsoft.com/en-us/sql/relational-databases/database-design
- PostgreSQL documentation – https://www.postgresql.org/docs/