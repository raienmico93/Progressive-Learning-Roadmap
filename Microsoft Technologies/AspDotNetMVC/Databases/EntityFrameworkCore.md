# Entity Framework Core: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **Entity Framework Core** (EF Core) 8–11, the modern, cross‑platform ORM for .NET. Examples use C# and ASP.NET Core minimal hosting patterns. Where behaviour differs across versions (e.g., `--add` for migrations in EF Core 11, `AsNoTrackingWithIdentityResolution` restrictions in EF Core 9), it is flagged. EF Core is the **recommended data‑access technology** for .NET applications; it is actively developed and supported. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Entity Framework Core is a modern object‑database mapper (ORM) for .NET that enables .NET developers to work with a database using .NET objects, eliminating the need for most of the data‑access code they usually have to write. |
| **Technical Definition** | EF Core is an ORM that maps .NET classes (**entities**) to database tables, properties to columns, and relationships to foreign keys. It provides **change tracking**, **LINQ query translation** to SQL, **migrations** for schema evolution, and **loading strategies** for related data. The central class is `DbContext`, which represents a session with the database and exposes `DbSet<TEntity>` properties for querying and saving. |
| **Beginner‑Friendly Explanation** | EF Core is the **translator between your C# code and the database**. You define plain C# classes (e.g., `Customer`, `Order`), and EF Core figures out how to save them to tables, retrieve them with LINQ queries, and keep track of changes so `SaveChanges` updates the right rows. Instead of writing SQL by hand, you write C#—and EF Core generates the SQL for you. |

### Key Characteristics

- **LINQ‑based querying** – Queries are written in C# using LINQ and translated to provider‑specific SQL.
- **Change tracking** – The context tracks modifications to loaded entities and generates `UPDATE`/`DELETE` statements on `SaveChanges`.
- **Migrations** – Version‑controlled schema changes are generated from model differences.
- **Multiple providers** – SQL Server, PostgreSQL, SQLite, MySQL, Cosmos DB, and more.
- **Relationship mapping** – One‑to‑one, one‑to‑many, and many‑to‑many relationships are configured via conventions, attributes, or Fluent API.
- **Loading strategies** – Eager (`Include`), explicit (`Load`), and lazy loading (with proxies).

### Prerequisites

- .NET SDK (6.0+ recommended), basic C# knowledge.
- A database provider package (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, `Microsoft.EntityFrameworkCore.Sqlite`).
- Working knowledge of **LINQ** and **SQL fundamentals**.
- Familiarity with **dependency injection** (for ASP.NET Core integration).

### Related Programming Areas

- **ASP.NET Core** – EF Core is the default data‑access technology for MVC, Razor Pages, and APIs.
- **LINQ** – The query language used to express database queries.
- **Migrations** – Schema versioning and deployment.
- **Dependency injection** – `DbContext` is registered as a Scoped service.
- **Transactions** – Grouping operations atomically.

### Core Concepts / Features

1. **Entity Framework Core** – The ORM and its architecture.
2. **EF Core** – The tooling, providers, and ecosystem.
3. **`DbContext`** – The session with the database.
4. **`DbSet`** – The queryable set for an entity type.
5. **Entities** – The .NET classes mapped to tables.
6. **Relationships** – One‑to‑one, one‑to‑many, and many‑to‑many.
7. **Migrations** – Schema evolution.
8. **Change tracking** – Detecting and persisting modifications.
9. **LINQ queries** – Expressing database queries in C#.
10. **Loading strategies** – Eager, lazy, and explicit loading.
11. **Performance** – Query optimization and advanced features.
12. **EF Core best practices** – Design, querying, and operational guidelines.

---

## 1. Entity Framework Core (The ORM)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Entity Framework Core is an open‑source, cross‑platform ORM for .NET that enables object‑oriented data access to relational and non‑relational databases. |
| **Technical Definition** | EF Core implements the **unit of work** and **repository** patterns through `DbContext` and `DbSet<T>`. It maps entities to database schemas via a **model** built from conventions, data annotations, and Fluent API configuration. The model is used for **query translation** (LINQ → SQL), **change tracking**, and **schema management** (migrations). EF Core is extensible through providers, interceptors, and conventions. |
| **Beginner‑Friendly Explanation** | EF Core is the **bridge between C# objects and database rows**. You define classes like `Product` and `Category`; EF Core creates tables for them, lets you query with `context.Products.Where(...)`, and saves changes with `context.SaveChanges()`. It handles the SQL, parameterization, and relationship management so you can focus on your domain logic. |

### Purposes

- **To** eliminate boilerplate data‑access code (ADO.NET commands, readers, manual mapping).
- **To** provide a **strongly typed** query API (LINQ) that is checked at compile time.
- **To** manage **relationships** between entities automatically (foreign keys, navigations).
- **To** enable **schema evolution** through migrations.
- **To** support **multiple database providers** with a consistent API.

### Syntax Rules and Structure

#### General Syntax (DbContext and DbSet)

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `: DbContext` | Inherits the EF Core context class. |
| Constructor | Receives `DbContextOptions<T>` (connection string, provider). |
| `DbSet<T>` properties | Represent entity sets (tables). |
| `Set<T>()` | Alternative to `DbSet<T>` for read‑only access. |

#### General Syntax (Registration in ASP.NET Core)

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddDbContext<T>` | Registers the context as a Scoped service. |
| `UseSqlServer(...)` | Configures the provider and connection. |

#### Syntax Rules

- **`DbContext` is Scoped** – create one per request; do not share across threads.
- **Queries are deferred** – LINQ queries execute when enumerated (`ToList`, `FirstOrDefault`, etc.).
- **`SaveChanges` is transactional** – all pending changes are committed atomically.
- **The model is built on first use** – subsequent contexts reuse the cached model.

#### Constraints and Limitations

- **Not all LINQ operators translate to SQL** – unsupported expressions throw at runtime.
- **Complex queries may generate inefficient SQL** – inspect the generated SQL during development.
- **Change tracking has overhead** – use `AsNoTracking` for read‑only scenarios.

### Annotated Code Example (Minimal EF Core Setup)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Entity
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Order> Orders { get; set; } = new();
}

// Context
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();
}

// Usage
public class CustomerService
{
    private readonly AppDbContext _context;
    public CustomerService(AppDbContext context) => _context = context;

    public async Task<List<Customer>> GetActiveAsync()
        => await _context.Customers
            .AsNoTracking()
            .Where(c => c.Orders.Any())
            .ToListAsync();
}
```

**Expected Output:** A list of customers who have at least one order, retrieved with a single SQL query and no change‑tracking overhead.

**Why this output?** EF Core translates the LINQ query into SQL, executes it, and materializes `Customer` objects. `AsNoTracking` skips tracking for read‑only use.

### Real-World Cases

- **E‑commerce:** Products, orders, customers, inventory.
- **SaaS applications:** Tenants, users, subscriptions, audit logs.
- **Content management:** Articles, authors, categories, media.

### References

- Entity Framework Core documentation – https://learn.microsoft.com/en-us/ef/core/
- EF Core getting started – https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app

---

## 2. EF Core (The Tooling and Ecosystem)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | EF Core encompasses the runtime libraries, command‑line tooling, database providers, and design‑time features that support the ORM. |
| **Technical Definition** | EF Core is distributed as NuGet packages: the core runtime (`Microsoft.EntityFrameworkCore`), provider packages (SQL Server, SQLite, PostgreSQL, etc.), design‑time packages (`Microsoft.EntityFrameworkCore.Design`), and tooling (`dotnet ef` CLI or Package Manager Console). The CLI manages migrations, database updates, and compiled models. |
| **Beginner‑Friendly Explanation** | EF Core is not just a library—it is a **complete toolkit**. The runtime library does the ORM work; the CLI tools generate migrations and update the database; provider packages connect to specific databases. You install the pieces you need, like choosing the right adapter for your database. |

### Purposes

- **To** provide **database‑specific providers** with a unified API.
- **To** enable **design‑time tooling** for migrations and reverse engineering.
- **To** support **advanced performance features** (compiled models, `DbContext` pooling).
- **To** offer **extensibility** through interceptors, conventions, and custom providers.

### Syntax Rules and Structure

#### Installing Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef
```

#### EF Core CLI Commands

| Command | Purpose |
|---------|---------|
| `dotnet ef migrations add <Name>` | Create a new migration. |
| `dotnet ef database update` | Apply pending migrations. |
| `dotnet ef migrations script` | Generate an SQL script (use `--idempotent` for production). |
| `dotnet ef dbcontext optimize` | Generate a compiled model. |
| `dotnet ef dbcontext scaffold` | Reverse‑engineer an existing database. |

**Syntax rules:**

- **The design‑time package** is required for the CLI to work.
- **`dotnet ef` commands** must run from the project directory (or specify `--project` and `--startup-project`).
- **Compiled models** improve startup time for large models.

**Constraints and limitations:**

- **Provider‑specific features** (e.g., `rowversion`, JSON columns) vary by provider.
- **CLI tools require the .NET SDK** and may need to be updated alongside the runtime.

### Annotated Code Example (Compiled Model Generation)

```bash
dotnet ef dbcontext optimize --output-dir MyCompiledModels --namespace MyCompiledModels
```

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseModel(MyCompiledModels.AppDbContextModel.Instance)
        .UseSqlServer(connectionString);
```

**Expected Output:** A compiled model is generated and used, reducing the first‑query startup time for large models.

**Why this output?** The compiled model bypasses runtime model building, which is beneficial for applications with hundreds or thousands of entity types.

### Real-World Cases

- **Large enterprise models:** Compiled models reduce cold‑start latency.
- **Multi‑provider applications:** Different providers for development (SQLite) and production (SQL Server).
- **Database‑first workflows:** Scaffolding entities from an existing database.

### References

- EF Core CLI reference – https://learn.microsoft.com/en-us/ef/core/cli/dotnet
- Advanced performance topics (compiled models) – https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics

---

## 3. `DbContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `DbContext` is the primary class that coordinates EF Core functionality for a given data model, representing a session with the database. |
| **Technical Definition** | `DbContext` derives from `Microsoft.EntityFrameworkCore.DbContext`. It exposes `DbSet<TEntity>` properties for querying and saving, manages **change tracking**, holds the **model** (entity mappings), and provides `SaveChanges`/`SaveChangesAsync` for persisting changes. It is configured via `DbContextOptions<T>` (provider, connection string, logging, etc.). |
| **Beginner‑Friendly Explanation** | `DbContext` is the **session with the database**. You open a context, query for entities, modify them, and call `SaveChanges` to persist the changes. The context tracks what you loaded and what you changed, so it knows exactly which `UPDATE` statements to execute. It also holds the mapping configuration that tells EF Core how your classes correspond to tables. |

### Purposes

- **To** provide a **unit‑of‑work** boundary for database operations.
- **To** expose **queryable sets** (`DbSet<T>`) for LINQ queries.
- **To** track **changes** to loaded entities and generate SQL on `SaveChanges`.
- **To** manage **transactions** (implicit per `SaveChanges`, explicit via `Database.BeginTransaction`).
- **To** configure the **model** (relationships, keys, indexes) via `OnModelCreating`.

### Syntax Rules and Structure

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>()
            .HasOne(o => o.Customer)
            .WithMany(c => c.Orders)
            .HasForeignKey(o => o.CustomerId);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor | Receives `DbContextOptions<T>`. |
| `DbSet<T>` properties | Query entry points. |
| `OnModelCreating` | Fluent API configuration for relationships, keys, indexes. |

**Syntax rules:**

- **`DbContext` is not thread‑safe** – do not use it concurrently.
- **The context is Scoped** in ASP.NET Core – one per request.
- **`OnModelCreating` is called once** when the model is built.
- **`SaveChanges` is transactional** – all changes commit or roll back together.

**Constraints and limitations:**

- **Long‑lived contexts accumulate tracked entities** – dispose and recreate contexts per unit of work.
- **`DbContext` pooling** reuses instances; do not rely on instance state between requests.
- **Lazy loading requires proxies** – the context must be alive when navigations are accessed.

### Annotated Code Example (DbContext with Fluent API Configuration)

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Blog> Blogs => Set<Blog>();
    public DbSet<Post> Posts => Set<Post>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>(entity =>
        {
            entity.HasKey(b => b.Id);
            entity.Property(b => b.Url).IsRequired().HasMaxLength(200);
        });

        modelBuilder.Entity<Post>(entity =>
        {
            entity.HasOne(p => p.Blog)
                  .WithMany(b => b.Posts)
                  .HasForeignKey(p => p.BlogId)
                  .OnDelete(DeleteBehavior.Cascade);
        });
    }
}
```

**Expected Output:** The model includes the `Blog` and `Post` entities with configured keys, required properties, and a cascade‑delete one‑to‑many relationship.

**Why this output?** The Fluent API in `OnModelCreating` overrides conventions and explicitly defines the schema and relationship behaviour.

### Real-World Cases

- **Multi‑tenant SaaS:** A context per request, with tenant filtering configured in `OnModelCreating` or global query filters.
- **Audit logging:** Overriding `SaveChanges` to set `CreatedAt`/`ModifiedAt` automatically.
- **Soft deletes:** Global query filters to exclude `IsDeleted` entities.

### References

- DbContext class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext
- DbContext configuration – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

---

## 4. `DbSet`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `DbSet<TEntity>` represents a collection of all entities of a given type that can be queried from the database. |
| **Technical Definition** | `DbSet<TEntity>` is a property on `DbContext` that provides `IQueryable<TEntity>` access for LINQ queries and methods for adding, attaching, and removing entities (`Add`, `Attach`, `Remove`). It also supports `Find` (by primary key), `Local` (tracked entities), and `AsQueryable`. |
| **Beginner‑Friendly Explanation** | `DbSet` is the **table in C# form**. If you have a `Customers` table, the context has a `DbSet<Customer> Customers` property. You query it with LINQ (`context.Customers.Where(...)`), add new customers (`context.Customers.Add(...)`), and find one by key (`context.Customers.Find(id)`). |

### Purposes

- **To** provide the **query entry point** for an entity type.
- **To** support **adding** new entities to the context.
- **To** support **attaching** existing entities for update/delete.
- **To** enable **`Find` by primary key** without a query.
- **To** expose **tracked entities** via `Local`.

### Syntax Rules and Structure

```csharp
public DbSet<Customer> Customers => Set<Customer>();
```

```csharp
// Query
var active = await context.Customers.Where(c => c.IsActive).ToListAsync();

// Find by key
var customer = await context.Customers.FindAsync(customerId);

// Add
context.Customers.Add(new Customer { Name = "Alice" });
await context.SaveChangesAsync();
```

**Component breakdown:**

| Operation | Method | Purpose |
|-----------|--------|---------|
| Query | `DbSet<T>` as `IQueryable<T>` | LINQ queries. |
| Find | `FindAsync(key)` | Retrieve by primary key (checks local cache first). |
| Add | `Add(entity)` | Mark entity as `Added`. |
| Attach | `Attach(entity)` | Track an existing entity as `Unchanged`. |
| Remove | `Remove(entity)` | Mark entity as `Deleted`. |

**Syntax rules:**

- **`DbSet` properties are typically pluralised** (`Customers`, `Orders`).
- **`Find` accepts primary key values** (composite keys use an array).
- **`Add`/`Attach`/`Remove` mark entity state**; `SaveChanges` executes the SQL.

**Constraints and limitations:**

- **`DbSet` is not a collection** – it does not implement `ICollection<T>`; use `Local` for tracked entities.
- **`Find` returns `null`** if the entity is not found.
- **`Add` does not immediately hit the database** – the insert occurs on `SaveChanges`.

### Annotated Code Example (DbSet Operations)

```csharp
var customer = await context.Customers.FindAsync(1);
if (customer is null)
{
    context.Customers.Add(new Customer { Name = "Alice" });
}
else
{
    customer.Name = "Alice Smith";
    context.Customers.Update(customer);
}
await context.SaveChangesAsync();
```

**Expected Output:** Either a new customer is inserted or the existing customer’s name is updated.

**Why this output?** `FindAsync` checks the local cache and then the database. `Add`/`Update` mark the entity state, and `SaveChanges` generates the appropriate `INSERT` or `UPDATE`.

### Real-World Cases

- **CRUD operations:** Add, find, update, and delete entities via `DbSet`.
- **Lookup tables:** Query reference data (countries, categories).
- **Bulk operations:** Use `AddRange`/`RemoveRange` for multiple entities.

### References

- DbSet<TEntity> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbset-1
- Querying data – https://learn.microsoft.com/en-us/ef/core/querying/

---

## 5. Entities

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An entity is a .NET class (POCO) that EF Core maps to a database table; its properties map to columns, and its relationships map to foreign keys. |
| **Technical Definition** | Entities are plain C# classes (no base class required) with public properties. EF Core discovers them through `DbSet<T>` properties or `OnModelCreating`. Keys are configured by convention (`Id`, `<Type>Id`), attributes (`[Key]`), or Fluent API. Navigations (reference and collection) express relationships. |
| **Beginner‑Friendly Explanation** | An entity is a **class that represents a row in a table**. A `Customer` entity has properties like `Id`, `Name`, and `Email`. EF Core creates a `Customers` table with those columns. Relationships are expressed through **navigation properties**—a `Customer` might have a `List<Order> Orders` navigation. |

### Purposes

- **To** define the **domain model** in C#.
- **To** map properties to **columns** with appropriate types and constraints.
- **To** express **relationships** through navigations and foreign keys.
- **To** enable **LINQ querying** against typed objects.
- **To** support **change tracking** and persistence via `SaveChanges`.

### Syntax Rules and Structure

```csharp
public class Customer
{
    public int Id { get; set; }                 // Primary key by convention
    public string Name { get; set; } = "";       // Required by non-nullable string
    public string? Email { get; set; }           // Optional
    public List<Order> Orders { get; set; } = new(); // Collection navigation
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }          // Foreign key
    public Customer Customer { get; set; } = null!; // Reference navigation
    public decimal Total { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Id` | Primary key by convention. |
| Non‑nullable `string` | Implicitly required in nullable‑enabled projects. |
| `List<Order> Orders` | Collection navigation (one‑to‑many). |
| `CustomerId` | Foreign key property. |
| `Customer Customer` | Reference navigation. |

**Syntax rules:**

- **Entities must have a key** – by convention `Id` or `<Type>Id`, or configured explicitly.
- **Collection navigations should be initialized** to avoid null checks.
- **Reference navigations should not be initialized** to a non‑null default unless the relationship is required.

**Constraints and limitations:**

- **EF Core requires a parameterless constructor** for entities (unless using constructor binding).
- **Navigation properties do not need to be `virtual`** unless using lazy‑loading proxies.
- **Arrays cannot be used for collection navigations** because `Add` throws.

### Annotated Code Example (Entity with Navigations)

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Url { get; set; } = "";
    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public int BlogId { get; set; }
    public Blog Blog { get; set; } = null!;
}
```

**Expected Output:** EF Core creates `Blogs` and `Posts` tables with a one‑to‑many relationship (a blog has many posts; each post belongs to one blog).

**Why this output?** The `Posts` collection navigation on `Blog` and the `Blog`/`BlogId` pair on `Post` express the relationship; EF Core infers the foreign key and cardinality.

### Real-World Cases

- **Blog engine:** `Blog` and `Post` entities with a one‑to‑many relationship.
- **E‑commerce:** `Customer`, `Order`, `OrderItem`, `Product` with multiple relationships.
- **HR system:** `Employee`, `Department`, `Project` with many‑to‑many assignments.

### References

- Entity types – https://learn.microsoft.com/en-us/ef/core/modeling/entity-types
- Relationship navigations – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/navigations

---

## 6. Relationships

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A relationship defines how two entities are associated—one‑to‑one, one‑to‑many, or many‑to‑many—through foreign keys and navigations. |
| **Technical Definition** | EF Core relationships are defined by **foreign keys**. **Navigations** provide an object‑oriented view over the foreign keys. The principal entity holds the reference/collection navigation; the dependent entity holds the foreign key. Cardinality is determined by the shape of the navigations and the nullability of the foreign key. Many‑to‑many uses a **join table** whose primary key is composed of the foreign keys from both related tables. |
| **Beginner‑Friendly Explanation** | Relationships describe **how tables are connected**. A customer has many orders (one‑to‑many): the `Orders` table has a `CustomerId` column pointing to `Customers`. A user has one profile (one‑to‑one). A student enrolls in many courses, and a course has many students (many‑to‑many)—this needs a third “enrollment” table linking them. |

### Purposes

- **To** model **real‑world associations** between entities.
- **To** enable **navigation** from one entity to related entities.
- **To** enforce **referential integrity** through foreign keys.
- **To** support **eager, explicit, and lazy loading** of related data.
- **To** configure **cascade delete** behaviour.

### Syntax Rules and Structure

#### One‑to‑Many (Convention)

```csharp
public class Blog
{
    public int Id { get; set; }
    public List<Post> Posts { get; set; } = new();
}

public class Post
{
    public int Id { get; set; }
    public int BlogId { get; set; }       // FK by convention
    public Blog Blog { get; set; } = null!;
}
```

#### One‑to‑One

```csharp
public class User
{
    public int Id { get; set; }
    public Profile Profile { get; set; } = null!;
}

public class Profile
{
    public int Id { get; set; }
    public int UserId { get; set; }        // FK (unique)
    public User User { get; set; } = null!;
}
```

#### Many‑to‑Many (EF Core 5+)

```csharp
public class Student
{
    public int Id { get; set; }
    public List<Course> Courses { get; set; } = new();
}

public class Course
{
    public int Id { get; set; }
    public List<Student> Students { get; set; } = new();
}
```

EF Core creates a join table automatically; explicit join entity configuration is available when additional columns are needed.

**Syntax rules:**

- **Foreign key naming** – `<Navigation>Id` or `<PrincipalType>Id` is discovered by convention.
- **Nullability** – a nullable FK makes the relationship optional; a non‑nullable FK makes it required.
- **Many‑to‑many** – requires a join table (implicit or explicit).
- **Cascade delete** – enabled by convention for required relationships; configure with `OnDelete(DeleteBehavior.…)`.

**Constraints and limitations:**

- **Multiple relationships between the same types** require explicit configuration to disambiguate.
- **One‑to‑one** requires a unique constraint on the dependent FK.
- **Many‑to‑many with payload** (extra columns on the join table) requires an explicit join entity.

### Annotated Code Example (Fluent API Relationship Configuration)

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Post>()
        .HasOne(p => p.Blog)
        .WithMany(b => b.Posts)
        .HasForeignKey(p => p.BlogId)
        .OnDelete(DeleteBehavior.Cascade);

    modelBuilder.Entity<Profile>()
        .HasOne(p => p.User)
        .WithOne(u => u.Profile)
        .HasForeignKey<Profile>(p => p.UserId);
}
```

**Expected Output:** A one‑to‑many cascade relationship between `Blog` and `Post`, and a one‑to‑one relationship between `User` and `Profile`.

**Why this output?** The Fluent API explicitly configures the cardinality, foreign key, and delete behaviour, overriding conventions where needed.

### Real-World Cases

- **E‑commerce:** `Order` → `OrderItem` (one‑to‑many), `Product` ↔ `Category` (many‑to‑many).
- **Social network:** `User` ↔ `User` (self‑referencing many‑to‑many for friends).
- **HR system:** `Employee` ↔ `Project` (many‑to‑many with role payload).

### References

- Relationships in EF Core – https://learn.microsoft.com/en-us/ef/core/modeling/relationships
- Relationship navigations – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/navigations

---

## 7. Migrations

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Migrations are a feature that enables **version‑controlled schema changes** by generating code from model differences and applying those changes to the database. |
| **Technical Definition** | `dotnet ef migrations add <Name>` compares the current model snapshot with the new model and generates a migration class with `Up` (apply) and `Down` (revert) methods. `dotnet ef database update` applies pending migrations. Migration files are checked into source control, providing an audit trail of schema changes. EF Core 11 introduced `--add` to create and apply a migration in one step. |
| **Beginner‑Friendly Explanation** | Migrations are **version control for your database schema**. When you add a property to an entity, EF Core generates a migration file that describes how to add the corresponding column. You review the file, commit it, and apply it to the database. Other developers (and production) apply the same migrations in order, so everyone’s database matches the code. |

### Purposes

- **To** evolve the database schema **safely and repeatably** alongside code changes.
- **To** provide a **versioned history** of schema changes in source control.
- **To** enable **deployment automation** through SQL scripts or runtime application.
- **To** support **team collaboration** – all developers apply the same migrations.
- **To** allow **rollback** to a previous schema state (`Down` method).

### Syntax Rules and Structure

#### Common Commands

```bash
# Add a migration
dotnet ef migrations add AddCreatedAt

# Apply pending migrations
dotnet ef database update

# Generate an idempotent SQL script for production
dotnet ef migrations script --idempotent --output migrations.sql

# Create and apply in one step (EF Core 11+)
dotnet ef database update AddCreatedAt --add
```

**Component breakdown:**

| Command | Purpose |
|---------|---------|
| `migrations add` | Scaffold a migration from model changes. |
| `database update` | Apply pending migrations to the database. |
| `migrations script` | Generate an SQL script for controlled deployment. |

**Syntax rules:**

- **Migration names should be descriptive** (like commit messages).
- **Migration files are checked into source control**.
- **Review migration code** before applying—especially column renames, which EF Core may generate as drop+add.

**Constraints and limitations:**

- **Data loss is possible** – dropping columns/tables loses data; review scripts before applying.
- **Production deployments** should use idempotent SQL scripts rather than `database update`.
- **Migration conflicts** can occur in team environments when two branches add migrations concurrently.

### Annotated Code Example (Migration Workflow)

```bash
# 1. Change the model (add CreatedAt property)
# 2. Generate a migration
dotnet ef migrations add AddCreatedAt

# 3. Review the generated migration file in Migrations/
# 4. Apply to the development database
dotnet ef database update

# 5. Generate a production script
dotnet ef migrations script --idempotent --output artifacts/migrations.sql
```

**Expected Output:** The development database gains a `CreatedAt` column; the production script contains idempotent `ALTER TABLE` statements.

**Why this output?** The migration file describes the schema change; applying it updates the database; the script allows controlled production deployment.

### Real-World Cases

- **Continuous deployment:** Migrations are applied automatically as part of the deployment pipeline.
- **Team development:** Each developer applies the same migrations locally.
- **Rollback:** Revert to a previous migration using `database update <PreviousMigration>`.

### References

- Managing migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying

---

## 8. Change Tracking

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Change tracking is the mechanism by which EF Core monitors changes to loaded entities and generates the appropriate `INSERT`, `UPDATE`, and `DELETE` statements on `SaveChanges`. |
| **Technical Definition** | When entities are queried, the context stores them in the **change tracker** with their original values. On `SaveChanges`, the tracker compares current and original values, determines the entity state (`Added`, `Modified`, `Deleted`, `Unchanged`), and generates SQL. `AsNoTracking` disables this behaviour for read‑only queries, avoiding the overhead of tracking. `QueryTrackingBehavior` can set the default tracking mode for the context. |
| **Beginner‑Friendly Explanation** | Change tracking is how EF Core **remembers what you loaded and what you changed**. If you fetch a customer and change their name, the context notices the difference and, on `SaveChanges`, issues an `UPDATE` for that row. If you are only reading data and not modifying it, you can disable tracking to save memory and time. |

### Purposes

- **To** automatically generate **`UPDATE`/`DELETE`** statements for modified entities.
- **To** maintain **identity resolution** – the same database row maps to the same in‑memory instance.
- **To** support **relationship fix‑up** – navigations are populated as entities are loaded.
- **To** enable **`AsNoTracking`** for read‑only scenarios, improving performance.
- **To** provide **`ChangeTracker`** APIs for inspecting and manipulating tracked entities.

### Syntax Rules and Structure

```csharp
// Tracking query (default)
var customer = await context.Customers.FirstAsync(c => c.Id == 1);
customer.Name = "New Name";   // Tracked — SaveChanges will update
await context.SaveChangesAsync();

// No-tracking query (read-only)
var customers = await context.Customers
    .AsNoTracking()
    .ToListAsync();

// Set default tracking behaviour for the context
context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
```

**Component breakdown:**

| Operation | Behaviour |
|-----------|-----------|
| Default query | Tracks entities; changes are detected on `SaveChanges`. |
| `AsNoTracking()` | No tracking; changes are **not** persisted. |
| `QueryTrackingBehavior.NoTracking` | Sets the default for all queries on the context. |

**Syntax rules:**

- **Use `AsNoTracking` for read‑only queries** to reduce memory and CPU overhead.
- **Do not use `AsNoTracking`** when you intend to modify and save the entities.
- **Identity resolution** is not performed with `AsNoTracking`—duplicate rows become different instances.

**Constraints and limitations:**

- **EF Core 9 restricts `AsNoTrackingWithIdentityResolution`** for certain JSON collection query patterns.
- **Change tracking has memory overhead** for large result sets; project only needed columns.
- **Contexts should be short‑lived** to avoid accumulating tracked entities.

### Annotated Code Example (Tracking vs. No‑Tracking)

```csharp
// Read-only projection with no tracking
var summaries = await context.Orders
    .AsNoTracking()
    .Select(o => new { o.Id, o.Total })
    .ToListAsync();

// Tracked entity for update
var order = await context.Orders.FindAsync(orderId);
order.Total = 99.99m;
await context.SaveChangesAsync();   // UPDATE generated
```

**Expected Output:** The no‑tracking query returns lightweight anonymous objects without tracking overhead; the tracked query allows the total to be updated.

**Why this output?** `AsNoTracking` skips change tracking for read‑only data; the tracked entity is monitored and updated on `SaveChanges`.

### Real-World Cases

- **Read‑heavy APIs:** Use `AsNoTracking` for list and report endpoints.
- **CRUD operations:** Use tracking for entities that will be modified.
- **Bulk reads:** Project to DTOs with `AsNoTracking` to minimize memory.

### References

- Tracking vs. no-tracking queries – https://learn.microsoft.com/en-us/ef/core/querying/tracking
- AsNoTracking method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking

---

## 9. LINQ Queries

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | LINQ (Language Integrated Query) is the C# query syntax that EF Core translates into database SQL, enabling strongly typed queries against entities. |
| **Technical Definition** | EF Core supports the standard LINQ operators (`Where`, `Select`, `OrderBy`, `Join`, `GroupBy`, `SelectMany`, etc.) and translates them to provider‑specific SQL. Queries are deferred—execution occurs when enumerated (`ToList`, `FirstOrDefault`, `Count`, etc.). The `IQueryable<T>` interface allows composition before translation. |
| **Beginner‑Friendly Explanation** | LINQ is **SQL written in C#**. Instead of `SELECT * FROM Customers WHERE IsActive = 1`, you write `context.Customers.Where(c => c.IsActive)`. EF Core reads your LINQ expression and generates the SQL. You get compile‑time checking, IntelliSense, and the ability to compose queries with C# methods. |

### Purposes

- **To** provide a **strongly typed, compile‑checked** query language.
- **To** compose queries dynamically using C# expressions.
- **To** translate LINQ operators to efficient SQL (joins, aggregations, subqueries).
- **To** support **projection** to DTOs or anonymous types, reducing data transfer.
- **To** enable **provider‑agnostic** queries that work across databases.

### Syntax Rules and Structure

```csharp
// Basic query
var activeCustomers = await context.Customers
    .Where(c => c.IsActive)
    .OrderBy(c => c.Name)
    .ToListAsync();

// Join
var orderDetails = await context.Orders
    .Join(context.Customers,
        o => o.CustomerId,
        c => c.Id,
        (o, c) => new { o.Id, c.Name, o.Total })
    .ToListAsync();

// Projection
var summaries = await context.Orders
    .Select(o => new OrderSummary
    {
        Id = o.Id,
        Total = o.Total,
        ItemCount = o.Items.Count
    })
    .ToListAsync();
```

**Component breakdown:**

| Operator | SQL Equivalent |
|----------|----------------|
| `Where` | `WHERE` |
| `OrderBy` / `OrderByDescending` | `ORDER BY` |
| `Select` | `SELECT` (projection) |
| `Join` | `INNER JOIN` |
| `GroupBy` | `GROUP BY` |
| `SelectMany` | `CROSS APPLY` / `OUTER APPLY` |

**Syntax rules:**

- **Queries are deferred** – nothing executes until enumeration.
- **Projection with `Select`** fetches only the required columns.
- **Use `AsNoTracking`** for read‑only queries.
- **Unsupported expressions throw** at runtime with a clear message.

**Constraints and limitations:**

- **Some LINQ operators are not translatable** (e.g., custom methods, `String.Format` in some cases).
- **Client evaluation** of complex expressions was removed in EF Core 3.0—unsupported expressions throw.
- **Complex joins may generate inefficient SQL**—inspect the SQL in development.

### Annotated Code Example (Join and Projection)

```csharp
var results = await context.Blogs
    .Join(context.Posts,
        b => b.Id,
        p => p.BlogId,
        (b, p) => new { BlogUrl = b.Url, PostTitle = p.Title })
    .Where(x => x.PostTitle.Contains("EF"))
    .ToListAsync();
```

**Expected Output:** A list of blog URLs and post titles where the post title contains “EF”.

**Why this output?** The `Join` operator is translated to an SQL `INNER JOIN`; the `Where` adds a filter; the projection returns only the needed columns.

### Real-World Cases

- **Search endpoints:** `Where` with multiple predicates and `Contains`.
- **Reporting:** `GroupBy` and aggregate functions (`Count`, `Sum`).
- **API projections:** `Select` into DTOs for lightweight responses.

### References

- Querying data with EF Core – https://learn.microsoft.com/en-us/ef/core/querying/
- Complex query operators – https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators

---

## 10. Loading Strategies

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Loading strategies determine when and how related data is loaded into navigation properties: **eager loading** (along with the main query), **explicit loading** (on demand via code), or **lazy loading** (automatically on first access). |
| **Technical Definition** | **Eager loading** uses `Include` and `ThenInclude` to retrieve related data in the same query (or split queries). **Explicit loading** uses `Entry(entity).Collection/Reference(...).Load()` to load related data on demand. **Lazy loading** requires proxies (`Microsoft.EntityFrameworkCore.Proxies`) and loads related data when a navigation property is first accessed. |
| **Beginner‑Friendly Explanation** | Loading strategies answer the question: **“When do related records get fetched?”** Eager loading fetches them **up front** with the main query. Explicit loading waits until you **explicitly ask** for them. Lazy loading fetches them **automatically** the first time you touch the navigation property—convenient but can cause many small queries. |

### Purposes

- **To** control **when** related data is retrieved, balancing performance and convenience.
- **To** avoid the **N+1 query problem** through eager loading.
- **To** support **on‑demand loading** when related data is needed only for some entities.
- **To** enable **lazy loading** for convenience in UI‑focused scenarios (with caution).

### Syntax Rules and Structure

#### Eager Loading

```csharp
var blogs = await context.Blogs
    .Include(b => b.Posts)
    .ThenInclude(p => p.Author)
    .ToListAsync();
```

#### Explicit Loading

```csharp
var blog = await context.Blogs.FindAsync(blogId);
await context.Entry(blog)
    .Collection(b => b.Posts)
    .LoadAsync();
```

#### Lazy Loading (with Proxies)

```csharp
// Install Microsoft.EntityFrameworkCore.Proxies
optionsBuilder.UseLazyLoadingProxies();

// Navigations must be virtual
public virtual ICollection<Post> Posts { get; set; }
```

**Component breakdown:**

| Strategy | Method | When Data Loads |
|----------|--------|-----------------|
| Eager | `Include` / `ThenInclude` | With the main query. |
| Explicit | `Entry(...).Load()` | When explicitly called. |
| Lazy | `UseLazyLoadingProxies()` + `virtual` navigation | On first access of the navigation. |

**Syntax rules:**

- **Eager loading** can be combined with `AsSplitQuery()` for large joins to avoid Cartesian explosion.
- **Explicit loading** requires an open context.
- **Lazy loading** requires virtual navigations and the proxies package; the context must remain alive.

**Constraints and limitations:**

- **Lazy loading can cause N+1 queries**—each navigation access triggers a separate query.
- **Lazy loading is not supported in EF Core 1.0**; it was added in later versions.
- **Explicit loading requires the entity to be tracked**; `AsNoTracking` entities cannot be explicitly loaded.

### Annotated Code Example (Eager vs. Explicit)

```csharp
// Eager: one query (or split query)
var departments = await context.Departments
    .Include(d => d.Courses)
    .ToListAsync();

// Explicit: separate query only when needed
var department = await context.Departments.FindAsync(id);
if (department is not null && needCourses)
{
    await context.Entry(department)
        .Collection(d => d.Courses)
        .LoadAsync();
}
```

**Expected Output:** Eager loading retrieves departments and their courses together; explicit loading fetches the department first and courses only if requested.

**Why this output?** `Include` adds the related data to the query; `Entry(...).Load()` issues a separate query for the navigation when called.

### Real-World Cases

- **API responses:** Eager loading for endpoints that always need related data.
- **UI grids:** Explicit loading for expandable rows.
- **Legacy convenience:** Lazy loading in small internal apps where query counts are not critical.

### References

- Loading related data – https://learn.microsoft.com/en-us/ef/core/querying/related-data/
- Eager, explicit, and lazy loading – https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/read-related-data

---

## 11. Performance

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | EF Core performance encompasses the techniques and features that reduce query latency, memory usage, and database round‑trips in EF Core applications. |
| **Technical Definition** | Performance topics include **`DbContext` pooling** (reusing context instances), **compiled queries** (avoiding repeated translation), **compiled models** (faster startup), **no‑tracking queries** (skipping change‑tracking overhead), **projection** (fetching only needed columns), **split queries** (avoiding Cartesian explosion), and **query caching** (reusing translation for identical query shapes). |
| **Beginner‑Friendly Explanation** | EF Core performance is about **doing less work**. Fetch only the columns you need (projection), don’t track entities you won’t modify (`AsNoTracking`), reuse context instances (pooling), and avoid queries that generate massive joins. Small changes—like adding `AsNoTracking()` or projecting to a DTO—can have a big impact. |

### Purposes

- **To** reduce **query execution time** and database load.
- **To** minimize **memory allocations** from tracking and materialization.
- **To** avoid **N+1 queries** and Cartesian explosion.
- **To** improve **startup time** with compiled models.
- **To** increase **throughput** under load.

### Syntax Rules and Structure

#### Key Techniques

| Technique | How |
|-----------|-----|
| **No‑tracking** | `.AsNoTracking()` for read‑only queries. |
| **Projection** | `.Select(x => new { ... })` or DTOs. |
| **DbContext pooling** | `AddDbContextPool<T>(...)` in DI. |
| **Split queries** | `.AsSplitQuery()` for large includes. |
| **Compiled queries** | `EF.CompileAsyncQuery(...)` for hot paths. |
| **Compiled models** | `dotnet ef dbcontext optimize` for large models. |

#### Syntax Rules

- **Use `AsNoTracking`** when entities will not be modified.
- **Project to DTOs** instead of fetching full entities for read‑only APIs.
- **Avoid `Include` on collections** when only aggregate data is needed—use projection with `Count`/`Sum`.
- **Pool contexts** in high‑throughput apps with `AddDbContextPool`.

**Constraints and limitations:**

- **Compiled queries** are less flexible—use them for hot paths with stable shapes.
- **DbContext pooling** requires that the context has no per‑request state beyond what can be reset.
- **Split queries** can be slower for small result sets due to multiple round trips.

### Annotated Code Example (Performance‑Optimized Query)

```csharp
// Projection + no-tracking: minimal data transfer
var dtos = await context.Orders
    .AsNoTracking()
    .Where(o => o.CreatedAt >= cutoff)
    .Select(o => new OrderDto
    {
        Id = o.Id,
        Total = o.Total,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();

// DbContext pooling
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString), poolSize: 128);
```

**Expected Output:** The query fetches only the required columns without tracking overhead; the context is reused from a pool, reducing allocation pressure.

**Why this output?** Projection reduces the data transferred; `AsNoTracking` avoids tracking overhead; pooling reuses context instances.

### Real-World Cases

- **High‑traffic APIs:** Pooling, projection, and no‑tracking for list endpoints.
- **Large models:** Compiled models to reduce cold‑start time.
- **Reporting:** Split queries or projection with aggregates to avoid Cartesian explosion.

### References

- EF Core performance best practices – https://learn.microsoft.com/en-us/ef/core/performance/
- Advanced performance topics – https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics

---

## 12. EF Core Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | EF Core best practices are guidelines for designing, querying, and operating EF Core applications that are correct, performant, secure, and maintainable. |
| **Technical Definition** | Best practices span **context lifetime** (Scoped, short‑lived), **query design** (projection, no‑tracking, avoiding N+1), **relationship configuration** (explicit Fluent API for clarity), **migration hygiene** (reviewed, versioned), **security** (parameterization, least‑privilege database accounts), and **performance** (pooling, compiled models). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for EF Core—they help you avoid slow queries, memory leaks, data corruption, and deployment surprises. |

### Purposes

- **To** keep contexts **short‑lived** and scoped to a unit of work.
- **To** write **efficient queries** that fetch only the needed data.
- **To** configure **relationships explicitly** for clarity and correctness.
- **To** manage **migrations safely** and review them before applying.
- **To** protect against **SQL injection** and **over‑posting**.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Context lifetime** | Use Scoped contexts (one per request); dispose promptly. | Share a `DbContext` across threads or long‑lived services. |
| **Querying** | Use `AsNoTracking` for read‑only queries; project to DTOs. | Fetch full entities for every read; use `Include` unnecessarily. |
| **N+1 queries** | Use `Include`/projection to load related data in one query. | Loop over entities and query per item. |
| **Relationships** | Configure with Fluent API when conventions are insufficient. | Rely on implicit conventions for complex relationships. |
| **Migrations** | Review migration code; use idempotent scripts for production. | Apply migrations blindly or edit the database manually. |
| **Security** | Use parameterized queries (EF Core does this); least‑privilege DB accounts. | Concatenate user input into SQL; use `sa`/superuser accounts. |
| **Performance** | Enable `DbContext` pooling; use compiled models for large schemas. | Create a new context per query without pooling in high‑traffic apps. |
| **Change tracking** | Disable tracking for read‑only data; keep contexts short‑lived. | Track thousands of entities in a single context. |

### Annotated Code Example (Best‑Practice Setup)

```csharp
// Scoped context with pooling
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString), poolSize: 128);

// Read-only query with projection and no-tracking
var summaries = await context.Orders
    .AsNoTracking()
    .Where(o => o.Status == OrderStatus.Active)
    .Select(o => new OrderSummaryDto
    {
        Id = o.Id,
        Total = o.Total,
        CustomerName = o.Customer.Name
    })
    .ToListAsync();
```

**Why this follows best practices:**

- **`AddDbContextPool`** reuses context instances, reducing allocation overhead.
- **`AsNoTracking`** skips change‑tracking for read‑only data.
- **Projection** fetches only the needed columns, reducing data transfer.
- **Scoped lifetime** ensures the context is disposed per request.

### Real-World Cases

- **High‑traffic e‑commerce APIs:** Pooling, no‑tracking, and projection.
- **Multi‑tenant SaaS:** Global query filters or tenant‑scoped repositories.
- **Enterprise applications:** Explicit Fluent API configuration and reviewed migrations.

### References

- EF Core performance best practices – https://learn.microsoft.com/en-us/ef/core/performance/
- DbContext configuration and lifetime – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- Migrations best practices – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing

---

## Summary Table: EF Core Topics at a Glance

| Concept | Primary Role | Key API / Tool | When to Use |
|---------|--------------|----------------|-------------|
| **Entity Framework Core** | ORM for .NET | `DbContext`, `DbSet`, LINQ | Every data‑access layer |
| **EF Core** | Tooling and ecosystem | `dotnet ef`, providers, compiled models | Setup, migrations, optimization |
| **`DbContext`** | Session with the database | `AddDbContext`, `SaveChanges` | Unit of work per request |
| **`DbSet`** | Queryable entity set | `context.Customers`, `FindAsync`, `Add` | CRUD and queries |
| **Entities** | .NET classes mapped to tables | POCOs with navigations | Domain model definition |
| **Relationships** | Associations between entities | `HasOne/WithMany`, conventions | One‑to‑one, one‑to‑many, many‑to‑many |
| **Migrations** | Schema versioning | `migrations add`, `database update` | Evolving the database schema |
| **Change tracking** | Detect and persist modifications | `AsNoTracking`, `ChangeTracker` | CRUD vs. read‑only queries |
| **LINQ queries** | Query language for EF Core | `Where`, `Select`, `Join`, `GroupBy` | Every database query |
| **Loading strategies** | When related data is fetched | `Include`, `Entry().Load()`, proxies | Related data retrieval |
| **Performance** | Query and context optimization | Pooling, projection, compiled models | High‑traffic or large‑model apps |
| **Best practices** | Correct, secure, maintainable EF Core | Scoped contexts, DTOs, reviewed migrations | Every production application |

---

## References (Full List)

- Entity Framework Core documentation – https://learn.microsoft.com/en-us/ef/core/
- EF Core getting started – https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app
- DbContext class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext
- DbSet<TEntity> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbset-1
- Entity types – https://learn.microsoft.com/en-us/ef/core/modeling/entity-types
- Relationships in EF Core – https://learn.microsoft.com/en-us/ef/core/modeling/relationships
- Relationship navigations – https://learn.microsoft.com/en-us/ef/core/modeling/relationships/navigations
- Managing migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying migrations – https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
- Tracking vs. no-tracking queries – https://learn.microsoft.com/en-us/ef/core/querying/tracking
- AsNoTracking method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking
- Querying data with EF Core – https://learn.microsoft.com/en-us/ef/core/querying/
- Complex query operators – https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators
- Loading related data – https://learn.microsoft.com/en-us/ef/core/querying/related-data/
- Eager, explicit, and lazy loading – https://learn.microsoft.com/en-us/aspnet/core/data/ef-mvc/read-related-data
- EF Core performance best practices – https://learn.microsoft.com/en-us/ef/core/performance/
- Advanced performance topics – https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics
- DbContext configuration – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- EF Core CLI reference – https://learn.microsoft.com/en-us/ef/core/cli/dotnet
- Razor Pages with EF Core tutorial – https://learn.microsoft.com/en-us/aspnet/core/data/ef-rp/intro