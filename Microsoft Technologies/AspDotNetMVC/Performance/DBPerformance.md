# ASP.NET Core Database Performance: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) with **Entity Framework Core** as the primary data‑access technology. Database performance is the single largest bottleneck in most web applications — a single missing index or N+1 query can turn a 50 ms endpoint into a 5‑second timeout. The guidance here draws on Microsoft’s official **EF Core performance documentation**, SQL Server **index design guidance**, and the **ADO.NET connection‑pooling** model. A critical framing note: **EF Core’s `DbContext` pooling is orthogonal to ADO.NET connection pooling** — they solve different problems and must both be configured correctly. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Database performance is the discipline of ensuring that database queries execute efficiently, indexes are used appropriately, connections are reused rather than recreated, and the data‑access layer does not become the bottleneck of the application. |
| **Technical Definition** | Database performance in ASP.NET Core is shaped by **query translation** (LINQ → SQL), **index utilisation** (seeks vs. scans), **data transfer volume** (projection vs. full entities), **change‑tracking overhead** (`AsNoTracking`), **round‑trip count** (N+1 queries, batching), **connection lifecycle** (ADO.NET connection pooling and `DbContext` pooling), and **pagination strategy** (offset vs. keyset). EF Core’s **compiled queries** and **query caching** further reduce CPU overhead for repeated query shapes. |
| **Beginner‑Friendly Explanation** | Database performance is **how fast your app talks to its database**. Slow database calls are the #1 cause of slow APIs. The fix is not one thing — it is a set of habits: write queries that use indexes, fetch only the columns you need, don’t track data you won’t change, don’t call the database in a loop, reuse connections instead of opening new ones, and paginate large result sets instead of loading everything. Each habit shaves milliseconds; together they determine whether your app scales or collapses. |

### Key Characteristics

- **Index‑driven** — A query is fast or slow primarily because of whether it uses an index; everything else is secondary.
- **Volume‑sensitive** — Fetching 10 columns instead of 2, or 10,000 rows instead of 20, has a disproportionate impact on latency and memory.
- **Round‑trip‑sensitive** — Each database round‑trip costs network latency; N+1 queries multiply that cost.
- **Connection‑pool‑dependent** — Opening a connection is expensive; pooling reuses them. `DbContext` pooling is a separate, complementary optimisation.
- **Measurable** — Query plans, execution statistics, and EF Core’s logging reveal exactly where time is spent.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project with EF Core.
- Working knowledge of **LINQ**, **SQL**, and **indexing concepts**.
- Familiarity with **dependency injection** and **`DbContext` lifetime**.
- Access to the database’s **query plan** tooling (SQL Server Management Studio, `EXPLAIN` in PostgreSQL, etc.).

### Related Programming Areas

- **EF Core querying** — LINQ translation, tracking, projection, loading strategies.
- **SQL Server indexing** — Clustered, non‑clustered, composite, filtered, and covering indexes.
- **ADO.NET connection pooling** — The underlying connection‑reuse mechanism.
- **Caching** — Reduces database load for frequently accessed data.
- **Pagination** — Bounds the amount of data transferred per request.

### Core Concepts / Features

1. **Query optimization** — Writing queries that fetch only what is needed, avoid N+1, and use efficient loading strategies.
2. **Indexing** — Designing indexes that match query patterns (ESR rule, composite, filtered, covering).
3. **Connection pooling** — Reusing database connections (ADO.NET) and `DbContext` instances (EF Core pooling).
4. **Database performance best practices** — Bulk operations, batching, pagination, and measurement discipline.

---

## 1. Query Optimization (Fetch Less, Do Less)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Query optimization is the practice of writing database queries that retrieve only the necessary data, use the fewest round‑trips, and avoid patterns that force the database to do unnecessary work. |
| **Technical Definition** | Query optimization in EF Core spans **projection** (`Select` only needed columns), **no‑tracking queries** (`AsNoTracking` for read‑only data), **loading strategy selection** (eager loading vs. split queries vs. projection to avoid N+1), **batching** (fetching multiple records in one query instead of a loop), and **compiled queries** (pre‑translating repeated query shapes). The goal is to minimise **data transfer volume**, **round‑trip count**, and **change‑tracking overhead**. |
| **Beginner‑Friendly Explanation** | Query optimization is **“don’t ask the database for more than you need, and don’t ask it repeatedly in a loop.”** If you need a product’s name, don’t fetch the entire product row with 20 columns. If you need 50 products by ID, don’t run 50 separate queries — fetch them in one. If you’re only reading data, don’t make EF Core track every row in memory. These are not micro‑optimizations; they are the difference between a fast API and a slow one. |

### Purposes

- **To** **reduce data transfer volume** — fetch only the columns the endpoint actually returns.
- **To** **eliminate N+1 query patterns** — replace loops that query per item with batched queries or projections.
- **To** **avoid change‑tracking overhead** — skip tracking for read‑only data.
- **To** **choose the right loading strategy** — eager loading, split queries, or projection depending on the shape of the data.
- **To** **reduce CPU cost of repeated queries** — compiled queries and query caching.

### Syntax Rules and Structure

#### Projection (Fetch Only Needed Columns)

```csharp
var dtos = await _db.Products
    .AsNoTracking()
    .Where(p => p.CategoryId == categoryId)
    .Select(p => new ProductDto
    {
        Id = p.Id,
        Name = p.Name,
        Price = p.Price
    })
    .ToListAsync(ct);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Select(...)` | Projects only the required columns. |
| `AsNoTracking()` | Disables change tracking for read‑only data. |
| `ToListAsync(ct)` | Materialises the result once. |

#### No‑Tracking Queries

```csharp
var products = await _db.Products
    .AsNoTracking()
    .Where(p => p.IsActive)
    .ToListAsync(ct);
```

#### Split Queries (Avoid Cartesian Explosion)

```csharp
var blogs = await _db.Blogs
    .Include(b => b.Posts)
    .ThenInclude(p => p.Author)
    .AsSplitQuery()
    .ToListAsync(ct);
```

#### Compiled Queries (Repeated Hot Paths)

```csharp
private static readonly Func<AppDbContext, int, Task<Product?>> _getProduct =
    EF.CompileAsyncQuery((AppDbContext db, int id) =>
        db.Products.FirstOrDefault(p => p.Id == id));

var product = await _getProduct(_db, id);
```

#### Batch Queries (Avoid N+1)

```csharp
// ❌ N+1 — one query per ID
foreach (var id in ids)
{
    var entity = await _db.MyEntities.FirstOrDefaultAsync(e => e.Id == id);
}

// ✅ One query for all IDs
var entities = await _db.MyEntities
    .Where(e => ids.Contains(e.Id))
    .ToListAsync();
```

**Syntax rules:**

- **Project to DTOs** for read‑only endpoints — avoid full entities when only a few columns are needed.
- **Use `AsNoTracking()`** for every read‑only query — change tracking is expensive and unnecessary if you won’t modify the entities.
- **Use `AsSplitQuery()`** for large `Include` chains that would produce a Cartesian explosion.
- **Use `EF.CompileAsyncQuery`** for queries executed repeatedly under heavy load — it bypasses translation overhead.
- **Batch by `Contains`** instead of looping — one query beats N queries.

**Constraints and limitations:**

- **Compiled queries provide modest gains in EF Core 10** — typically 8–12% improvement for repeated queries; they are a last‑2% optimisation, not a first step.
- **Projection breaks change tracking** — if you need to update the fetched data, you must load entities (or attach modified stubs).
- **Split queries can be slower for small result sets** — they add round‑trips; measure before applying.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public async Task<IActionResult> GetOrderSummariesAsync(
    DateTime start, CancellationToken ct)
{
    var summaries = await _db.Orders
        .AsNoTracking()
        .Where(o => o.CreatedAt >= start)
        .Select(o => new OrderSummaryDto
        {
            OrderId = o.Id,
            CustomerName = o.Customer.Name,
            Total = o.Total,
            ItemCount = o.Items.Count
        })
        .ToListAsync(ct);

    return Ok(summaries);
}
```

**Step‑by‑step explanation:**

1. `AsNoTracking()` disables change tracking — this is a read‑only query.
2. `Where(o => o.CreatedAt >= start)` filters at the database.
3. `Select(...)` projects only the required columns, including a navigation (`Customer.Name`) and a count (`Items.Count`), which EF Core translates into a single SQL query with a `JOIN` and subquery.
4. `ToListAsync(ct)` materialises the result once.

**Expected Output:** A single SQL query that fetches only the four columns needed for the summary, with no tracking overhead.

**Why this output?** Projection eliminates unnecessary columns; `AsNoTracking` eliminates tracking overhead; the query is batched into one round‑trip.

### Real‑World Cases

- **Product listings:** Project to `ProductListDto` with `Id`, `Name`, `Price` — not the full entity.
- **Order history:** Use `AsNoTracking` and projection; never track thousands of read‑only rows.
- **Dashboard aggregations:** Project to anonymous types with `Count`, `Sum`, `Average` — EF Core translates them to SQL aggregates.
- **Repeated lookups:** Use compiled queries for the hottest query shapes.

### References

- Efficient Querying (EF Core) – https://github.com/dotnet/entityframework.docs/blob/b0c8daf21d4dd8157e9696e77942278b9722b226/entity-framework/core/performance/efficient-querying.md
- EF Core Query Optimization: Projections, AsSplitQuery, AsNoTracking – https://thecodeman.net
- Do you optimize your EF Core queries? (SSW Rules) – https://www.ssw.com.au/rules/optimize-ef-core-queries
- EF Core 10 Compiled Queries Boost Performance – https://www.linkedin.com

---

## 2. Indexing (The Primary Performance Lever)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An index is a data structure (typically a B‑tree) that allows the database to locate rows without scanning the entire table, turning an O(n) table scan into an O(log n) seek. |
| **Technical Definition** | Indexes in EF Core are configured via `[Index]` attributes or the Fluent API (`HasIndex`). A **composite index** covers multiple columns; the order of columns matters (the **ESR rule**: Equality predicates first, Sort columns second, Range predicates last). **Filtered indexes** include only a subset of rows (e.g., `WHERE IsActive = 1`). **Covering indexes** include all columns needed by a query so the table itself is never accessed. |
| **Beginner‑Friendly Explanation** | An index is the **“index at the back of a book”** for your database. Without it, the database reads every row to find the ones matching your `WHERE` clause — a full table scan. With an index, it jumps directly to the matching rows. But indexes have a cost: they slow down inserts and updates (the index must be maintained), and they consume storage. The art is creating the right indexes for your query patterns without over‑indexing. |

### Purposes

- **To** **turn table scans into index seeks** — the single biggest query‑speed improvement.
- **To** **support `WHERE` predicates** — especially equality filters and range filters.
- **To** **support `ORDER BY`** — a sorted index can eliminate a sort operation.
- **To** **support `JOIN` operations** — foreign key columns should be indexed.
- **To** **reduce I/O** — a covering index avoids reading the base table entirely.

### Syntax Rules and Structure

#### Single‑Column Index

```csharp
[Index(nameof(Url))]
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; } = "";
}
```

#### Composite Index (ESR Order)

```csharp
// Equality columns first, sort column second, range column last
[Index(nameof(Status), nameof(CreatedAt), nameof(Total))]
public class Order
{
    public int Id { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public decimal Total { get; set; }
}
```

#### Filtered Index (Subset of Rows)

```csharp
modelBuilder.Entity<Order>()
    .HasIndex(o => o.CreatedAt)
    .HasFilter("[Status] = 'Pending'");
```

#### Covering Index (Include Non‑Key Columns)

```csharp
modelBuilder.Entity<Product>()
    .HasIndex(p => p.CategoryId)
    .IncludeProperties(p => new { p.Name, p.Price });
```

**Syntax rules:**

- **Index foreign keys** — EF Core creates indexes for foreign keys by convention, but verify.
- **Follow the ESR rule** for composite indexes: Equality → Sort → Range. Equality columns with high selectivity should come first.
- **Use filtered indexes** for well‑defined subsets (e.g., `IsActive = 1`, `Status = 'Pending'`) — they reduce index size and maintenance cost.
- **Use covering indexes** (`IncludeProperties`) when queries need only a few extra columns — they avoid key lookups.

**Constraints and limitations:**

- **Indexes slow down writes** — every insert/update must update the index; avoid over‑indexing.
- **Index column order matters** — an index on `(A, B)` helps queries filtering on `A` or `(A, B)`, but **not** queries filtering only on `B`.
- **Expressions in predicates** (e.g., `WHERE Price / 2 > 100`) prevent index use — use persisted computed columns or expression indexes if supported.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// Query: pending orders sorted by creation date, filtered by customer
public async Task<List<Order>> GetPendingOrdersAsync(
    int customerId, CancellationToken ct)
    => await _db.Orders
        .AsNoTracking()
        .Where(o => o.CustomerId == customerId   // Equality
                 && o.Status == OrderStatus.Pending) // Equality
        .OrderByDescending(o => o.CreatedAt)     // Sort
        .ToListAsync(ct);

// Index following ESR: Equality (CustomerId, Status) → Sort (CreatedAt)
modelBuilder.Entity<Order>()
    .HasIndex(o => new { o.CustomerId, o.Status, o.CreatedAt });
```

**Step‑by‑step explanation:**

1. The query filters on `CustomerId` and `Status` (equality), then sorts by `CreatedAt`.
2. The composite index places the equality columns first, followed by the sort column.
3. The database can seek directly to the matching rows and return them already sorted — no table scan, no sort operation.

**Expected Output:** The query plan shows an **index seek** on the composite index, with no sort operator.

**Why this output?** The index columns match the query’s predicate and sort order, satisfying the ESR rule.

### Real‑World Cases

- **E‑commerce product listings:** Composite index on `(CategoryId, IsActive, CreatedAt)` for filtered, sorted listings.
- **Order history:** Index on `(CustomerId, CreatedAt)` for per‑customer chronological queries.
- **Task queues:** Filtered index on `Status = 'Pending'` for fast polling of unprocessed work.
- **Search dashboards:** Covering index including the columns needed by the read model.

### References

- Indexes – EF Core – https://learn.microsoft.com/vi-vn/ef/core/modeling/indexes
- SQL Server Index Architecture and Design Guide – https://learn.microsoft.com/fr-ch/SQL/relational-databases/sql-server-index-design-guide
- Equality, Sort, Range Indexing In SQL Server – https://erikdarling.com/equality-sort-range-indexing-in-sql-server-when-it-works/
- SQL Server Filtered Indexes: How and When to Use Them – https://www.devart.com

---

## 3. Connection Pooling (Reuse Connections and Contexts)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Connection pooling is the technique of maintaining a cache of open database connections so that new requests reuse an existing connection instead of paying the cost of opening a new one. |
| **Technical Definition** | ADO.NET providers (e.g., `Microsoft.Data.SqlClient`) maintain a **connection pool** keyed by connection string. When a connection is closed, it returns to the pool; when a new connection is requested, it is taken from the pool if available. The pool has `Min Pool Size` (default 0) and `Max Pool Size` (default 100) settings. **EF Core’s `DbContext` pooling** is a separate, orthogonal optimisation: it caches `DbContext` instances (not connections) to avoid the cost of constructing the context and its internal services on every request. |
| **Beginner‑Friendly Explanation** | Connection pooling is the **“reuse the same phone line”** idea. Opening a database connection is slow (network handshake, authentication). Instead of opening a new one for every request, the app keeps a pool of already‑open connections and hands them out. When you’re done, the connection goes back to the pool for the next request. **DbContext pooling** is similar but for the EF Core context object itself — it avoids re‑creating the context’s internal machinery on every request. They are complementary: you want both. |

### Purposes

- **To** **avoid the cost of opening connections** — connection establishment is expensive (TCP, TLS, authentication).
- **To** **bound resource usage** — `Max Pool Size` prevents the application from exhausting the database’s connection limit.
- **To** **reduce latency** — pooled connections are ready immediately.
- **To** **reduce `DbContext` allocation overhead** — EF Core’s context pooling reuses context instances and their internal services.

### Syntax Rules and Structure

#### ADO.NET Connection Pooling (Connection String)

```json
{
  "ConnectionStrings": {
    "Default": "Server=...;Database=...;Min Pool Size=5;Max Pool Size=100;"
  }
}
```

**Component breakdown:**

| Setting | Default | Purpose |
|---------|---------|---------|
| `Min Pool Size` | 0 | Connections kept open even when idle. |
| `Max Pool Size` | 100 | Maximum simultaneous connections. |
| `Connection Timeout` | 15s | How long to wait for a pooled connection. |

#### EF Core `DbContext` Pooling

```csharp
// ❌ Without pooling — a new context (and its internal services) per request
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// ✅ With pooling — context instances are reused from a pool
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString),
    poolSize: 128);   // Default is 1024
```

**Syntax rules:**

- **Connection pooling is enabled by default** in ADO.NET providers — you do not need to “turn it on”, but you should tune `Max Pool Size` based on measured concurrency and database capacity.
- **`AddDbContextPool` replaces `AddDbContext`** to enable context pooling.
- **`poolSize` sets the maximum number of cached context instances** (default 1024); beyond that, EF falls back to creating new instances on demand.
- **Do not access request‑scoped state in `OnConfiguring`** when using pooled contexts — `OnConfiguring` runs only once per pooled instance.

**Constraints and limitations:**

- **`Max Pool Size` is per connection string, per process** — if you have multiple app instances, the total connections across all instances must stay under the database limit.
- **Pooling is not a cure for leaked connections** — a connection that is not disposed is never returned to the pool. Always use `using`/`await using` for connections and contexts.
- **`DbContext` pooling does not pool connections** — it pools contexts; ADO.NET pools connections independently.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// ADO.NET connection pooling is configured via the connection string
var connectionString = builder.Configuration.GetConnectionString("Default");

// EF Core context pooling
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString),
    poolSize: 128);

var app = builder.Build();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. The connection string includes `Max Pool Size=100` (or is left at the default).
2. `AddDbContextPool` registers `AppDbContext` as a pooled service — when a context is disposed, it is reset and returned to the pool.
3. The next request receives a pooled context instead of constructing a new one.

**Expected Output:** Requests reuse both database connections (ADO.NET pool) and `DbContext` instances (EF pool), reducing allocation and connection‑setup overhead.

**Why this output?** The two pools operate at different layers: ADO.NET manages the physical connection; EF Core manages the context object.

### Real-World Cases

- **High‑traffic APIs:** `Max Pool Size` tuned to measured concurrency; `AddDbContextPool` with a `poolSize` matching expected concurrency.
- **Multi‑tenant apps:** Pooled contexts require careful handling of tenant‑specific state — do not store tenant ID in context fields that persist across requests.
- **Serverless / bursty workloads:** Pooling reduces cold‑start connection overhead.

### References

- Advanced Performance Topics (DbContext pooling) – https://learn.microsoft.com/sl-si/ef/core/performance/advanced-performance-topics
- Connection Pooling Considerations – https://learn.microsoft.com
- Connection pooling and `Max Pool Size` – https://learn.microsoft.com

---

## 4. Database Performance Best Practices (The Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Database performance best practices are the proven techniques — beyond query optimization, indexing, and pooling — that keep the data‑access layer fast, predictable, and scalable under production load. |
| **Technical Definition** | Best practices span **bulk operations** (`AddRange` + one `SaveChanges`), **batching** (fetch in one query instead of a loop), **pagination** (offset vs. keyset), **measurement discipline** (query plans, EF Core logging, benchmarks), **avoiding lazy loading in APIs**, and **caching** frequently accessed data outside the database. |
| **Beginner‑Friendly Explanation** | These are the **“habits of a fast database layer”** — they help you avoid the classic production disasters: inserting 10,000 rows one at a time, calling `SaveChanges` inside a loop, using `OFFSET 100000` for deep pagination, and guessing at performance instead of measuring. |

### Purposes

- **To** **minimise round‑trips** — batch inserts, updates, and reads.
- **To** **keep memory bounded** — paginate large result sets instead of loading everything.
- **To** **avoid lazy‑loading surprises** — N+1 queries from serialisation or navigation access.
- **To** **measure before optimising** — query plans reveal the real bottleneck.
- **To** **cache outside the database** — reduce load for frequently accessed, relatively static data.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Bulk inserts** | Use `AddRange` + one `SaveChangesAsync`; for very large batches, use `EFCore.BulkExtensions`. | Call `SaveChangesAsync` inside a loop — one transaction per row. |
| **Batching reads** | Use `Where(e => ids.Contains(e.Id))` to fetch many records in one query. | Run one query per ID in a loop — N+1. |
| **Pagination** | Use **keyset (seek) pagination** for deep pages; `OFFSET` degrades linearly with page depth. | Use `OFFSET 100000` — the database scans and discards all preceding rows. |
| **Change tracking** | `AsNoTracking()` for every read‑only query. | Track entities you never modify. |
| **Lazy loading** | Avoid lazy loading in APIs — it causes N+1 when serialising navigation properties. | Return entities with lazy navigations from API endpoints. |
| **`SaveChanges`** | Call `SaveChangesAsync` **once** per unit of work. | Call it per entity — each call is a separate transaction. |
| **Measurement** | Inspect query plans; enable EF Core sensitive‑data logging in development. | Guess at performance without measuring. |
| **Caching** | Cache reference data and hot read models outside the database. | Hit the database for data that changes hourly. |

### Annotated Code Example (Best‑Practice Data Access)

```csharp
public async Task<IActionResult> CreateOrdersAsync(
    List<CreateOrderDto> inputs, CancellationToken ct)
{
    var orders = inputs.Select(i => new Order
    {
        CustomerId = i.CustomerId,
        Total = i.Total,
        CreatedAt = DateTime.UtcNow
    }).ToList();

    _db.Orders.AddRange(orders);       // Bulk add — no database call yet
    await _db.SaveChangesAsync(ct);     // One transaction, one round-trip

    return Ok(orders.Select(o => o.Id));
}

// Keyset pagination for a large order list
public async Task<List<Order>> GetOrdersPageAsync(
    DateTime lastCreatedAt, int take, CancellationToken ct)
    => await _db.Orders
        .AsNoTracking()
        .Where(o => o.CreatedAt < lastCreatedAt)   // keyset predicate
        .OrderByDescending(o => o.CreatedAt)
        .Take(take)
        .ToListAsync(ct);
```

**Why this follows best practices:**

- **`AddRange` + one `SaveChanges`** — inserts all orders in one transaction instead of N transactions.
- **Keyset pagination** — `WHERE CreatedAt < last` uses the index and avoids `OFFSET` scanning.
- **`AsNoTracking`** — the paginated read is read‑only.
- **`CancellationToken`** — cancellation propagates to the database.

### Real-World Cases

- **Order import:** Bulk insert thousands of orders with `AddRange` + `EFCore.BulkExtensions`.
- **Activity feeds:** Keyset pagination on `(UserId, CreatedAt)` for infinite scroll.
- **Reference data:** Cache countries, categories, and configuration in memory with a TTL.

### References

- Do you optimize your EF Core queries? (SSW Rules) – https://www.ssw.com.au/rules/optimize-ef-core-queries
- Efficient Querying (EF Core) – https://github.com/dotnet/entityframework.docs
- Offset and Keyset Pagination – https://thorben-janssen.com
- Advanced Performance Topics (compiled queries, DbContext pooling) – https://learn.microsoft.com

---

## Summary Table: Database Performance at a Glance

| Concept | Primary Role | Key Technique | When to Use |
|---------|--------------|---------------|-------------|
| **Query optimization** | Fetch only what is needed, avoid N+1 | Projection, `AsNoTracking`, split queries, compiled queries, batching | Every query, especially read‑heavy endpoints |
| **Indexing** | Turn scans into seeks | ESR composite indexes, filtered indexes, covering indexes | Every table with `WHERE`/`ORDER BY`/`JOIN` patterns |
| **Connection pooling** | Reuse connections and contexts | ADO.NET pool (`Max Pool Size`), `AddDbContextPool` | Every ASP.NET Core app |
| **Best practices** | Bulk operations, batching, pagination, measurement | `AddRange`, `Contains`, keyset pagination, query plans | Every production data‑access layer |

---

## References (Full List)

- Efficient Querying (EF Core) – https://github.com/dotnet/entityframework.docs/blob/b0c8daf21d4dd8157e9696e77942278b9722b226/entity-framework/core/performance/efficient-querying.md
- Advanced Performance Topics (DbContext pooling) – https://learn.microsoft.com/sl-si/ef/core/performance/advanced-performance-topics
- Indexes – EF Core – https://learn.microsoft.com/vi-vn/ef/core/modeling/indexes
- SQL Server Index Architecture and Design Guide – https://learn.microsoft.com/fr-ch/SQL/relational-databases/sql-server-index-design-guide
- Equality, Sort, Range Indexing In SQL Server – https://erikdarling.com/equality-sort-range-indexing-in-sql-server-when-it-works/
- Do you optimize your EF Core queries? (SSW Rules) – https://www.ssw.com.au/rules/optimize-ef-core-queries
- EF Core Query Optimization: Projections, AsSplitQuery, AsNoTracking – https://thecodeman.net
- EF Core 10 Compiled Queries Boost Performance – https://www.linkedin.com
- Offset and Keyset Pagination – https://thorben-janssen.com
- SQL Server Filtered Indexes: How and When to Use Them – https://www.devart.com
- Connection Pooling Considerations (EF Core) – https://learn.microsoft.com
- `Max Pool Size` and connection pooling – https://learn.microsoft.com