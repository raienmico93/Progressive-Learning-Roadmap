# C# Entity Framework Core — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Entity Framework Core (EF Core) is a modern, lightweight, extensible, open-source object-database mapper (O/RM) for .NET. It enables .NET developers to work with a database using .NET objects, eliminating the need for most of the data-access code developers usually need to write.

**Technical Definition**
EF Core is an O/RM that maps CLR classes (entities) to database tables, properties to columns, and relationships to foreign-key constraints. It provides a `DbContext` that represents a session with the database, `DbSet<TEntity>` properties that represent queryable collections of entities, a change tracker that tracks modifications to loaded entities, a migration system that evolves the database schema from the model, and a LINQ provider that translates LINQ queries into SQL executed by the database provider.

**Beginner-Friendly Explanation**
Imagine you have C# classes like `Customer` and `Order`, and a database with tables `Customers` and `Orders`. EF Core is the bridge: you write C# code like `context.Customers.Where(c => c.City == "Paris")`, and EF Core translates that into SQL, runs it against the database, and gives you back `Customer` objects. You rarely write SQL by hand, and EF Core handles the plumbing of opening connections, reading rows, and turning them into objects.

---

### Key Characteristics

- **Provider-based** — EF Core supports SQL Server, SQLite, PostgreSQL, MySQL, Oracle, and others through database providers.
- **LINQ-based querying** — Queries are written in LINQ and translated to the provider's SQL dialect.
- **Change tracking** — Loaded entities are tracked for modifications; `SaveChanges()` generates and executes the appropriate INSERT/UPDATE/DELETE statements.
- **Migrations** — Schema changes are expressed as code-based migrations that can be applied incrementally.
- **Cross-platform** — Runs on .NET on Windows, Linux, and macOS.
- **Extensible** — Conventions, interceptors, value converters, and custom functions can be plugged in.
- **Asynchronous by default** — Provides `ToListAsync`, `SaveChangesAsync`, and other async APIs.
- **Not a replacement for SQL knowledge** — Understanding the generated SQL is essential for performance.

---

### Prerequisites

- Working knowledge of C# classes, properties, collections, and LINQ.
- Basic understanding of relational databases: tables, rows, columns, primary keys, foreign keys, and indexes.
- Familiarity with `async`/`await` in C#.
- A database provider package (e.g., `Microsoft.EntityFrameworkCore.SqlServer` or `Microsoft.EntityFrameworkCore.Sqlite`).
- The .NET SDK and a project with EF Core packages installed.

---

### Related Programming Areas

- **Relational databases and SQL** — EF Core generates SQL; understanding SQL helps diagnose performance and correctness.
- **Domain-driven design (DDD)** — EF Core is often used to persist aggregates and entities.
- **Dependency injection** — `DbContext` is typically registered in the DI container with a scoped lifetime.
- **Web development (ASP.NET Core)** — EF Core is the default data-access technology in ASP.NET Core templates.
- **Testing** — In-memory and SQLite in-memory providers support unit and integration testing.

---

### Core Concepts / Features

1. Entity Framework Core (overarching framework)
2. EF Core (terminology, versions, and ecosystem)
3. `DbContext`
4. `DbSet`
5. Entities
6. Relationships
   - One-to-one
   - One-to-many
   - Many-to-many
7. Migrations
8. Change tracking
9. LINQ queries
10. Loading strategies
    - Eager loading
    - Lazy loading
    - Explicit loading
11. Performance
12. EF Core best practices
13. EF Core pitfalls

Each concept follows the same uniform structure: definitions, purposes (each beginning with "To"), syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Entity Framework Core (Overarching Framework)

### Definitions

**Core Definition**
Entity Framework Core is an object-relational mapper that lets .NET developers query and persist data using strongly typed C# objects instead of raw SQL.

**Technical Definition**
EF Core is a set of NuGet packages that implement the O/RM pattern for .NET. It includes a model (the metadata describing entities and relationships), a runtime (`DbContext`, change tracker, query pipeline), a LINQ provider that translates expression trees to SQL, a migration engine, and a set of database providers. EF Core is the successor to Entity Framework 6 (EF6) and is maintained by Microsoft and the .NET community.

**Beginner-Friendly Explanation**
EF Core is the "translator" between your C# code and your database. You describe your data with classes, and EF Core figures out how to read and write it in the database.

---

### Purposes

- To eliminate boilerplate data-access code.
- To allow developers to work with data using C# and LINQ rather than SQL strings.
- To provide a consistent API across multiple database engines.
- To support schema evolution through migrations.
- To integrate with .NET dependency injection and configuration.

---

### Syntax Rules and Structure

#### General Setup Syntax

```
// .csproj (PackageReference entries)
<ItemGroup>
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.0" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.0" />
</ItemGroup>
```

```
// DbContext registration in Program.cs (ASP.NET Core)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

**Component breakdown**

- `Microsoft.EntityFrameworkCore` — Core abstractions and runtime.
- `Microsoft.EntityFrameworkCore.SqlServer` — Provider for SQL Server.
- `Microsoft.EntityFrameworkCore.Design` — Design-time services used by tools.
- `Microsoft.EntityFrameworkCore.Tools` — CLI tools (`dotnet ef`) and Package Manager Console cmdlets.
- `AddDbContext` — Registers the context with a scoped lifetime.

#### Syntax Rules

- Each provider requires its own package.
- The `Design` package is required for migrations and scaffolding.
- The context must be registered in DI or instantiated manually with options.
- EF Core versions should match the target framework (e.g., EF Core 8 for .NET 8).

#### Constraints and Limitations

- EF Core is not a drop-in replacement for EF6; APIs differ.
- Not all LINQ expressions can be translated to SQL; unsupported expressions throw at runtime.
- Some provider-specific features are not portable across databases.
- EF Core does not manage database connections automatically beyond the scope of a context.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Minimal EF Core Application with SQLite

```csharp
// Program.cs
using Microsoft.EntityFrameworkCore;

// 1. Define the entity
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}

// 2. Define the context
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs => Set<Blog>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=blogging.db");
}

// 3. Use the context
class Program
{
    static async Task Main()
    {
        using var db = new BloggingContext();
        await db.Database.EnsureCreatedAsync();   // creates the database file

        db.Blogs.Add(new Blog { Name = "EF Core Intro" });
        await db.SaveChangesAsync();

        var blogs = await db.Blogs.ToListAsync();
        foreach (var blog in blogs)
            Console.WriteLine($"{blog.Id}: {blog.Name}");
    }
}
```

**Line-by-line comments**

- `Blog` — Entity class; `Id` is the primary key by convention; `Name` maps to a column.
- `BloggingContext` — The `DbContext`; `Blogs` is a `DbSet<Blog>`.
- `OnConfiguring` — Configures the SQLite provider with a file-based database.
- `EnsureCreatedAsync` — Creates the database if it does not exist (not for use with migrations).
- `Add` + `SaveChangesAsync` — Inserts a row.
- `ToListAsync` — Queries all blogs.

**Expected Output (first run)**

```
1: EF Core Intro
```

**Why this output occurs**

- The entity is inserted and assigned an `Id` of `1`.
- The query returns the inserted row and maps it to a `Blog` object.

---

### Real-World Cases

- **Web APIs** — ASP.NET Core controllers use EF Core to read and write data.
- **Desktop applications** — WPF and MAUI apps persist local data with EF Core and SQLite.
- **Background services** — Worker services process data using EF Core with scoped contexts.

---

### References

- Entity Framework Core overview - https://learn.microsoft.com/en-us/ef/core/
- EF Core releases and versions - https://learn.microsoft.com/en-us/ef/core/what-is-new/
- Entity Framework Core GitHub repository - https://github.com/dotnet/efcore

---

## 2. EF Core (Terminology, Versions, and Ecosystem)

### Definitions

**Core Definition**
"EF Core" is the short name for Entity Framework Core, distinguishing it from the legacy Entity Framework 6.

**Technical Definition**
EF Core is distributed as a set of NuGet packages versioned independently of .NET. Major versions (EF Core 6, 7, 8, 9) align with .NET versions. EF Core 6 targets .NET 6, EF Core 7 targets .NET 6 and 7, EF Core 8 targets .NET 8, and EF Core 9 targets .NET 8 and 9. EF Core 5 and earlier are out of support.

**Beginner-Friendly Explanation**
"EF Core" is just the nickname. When someone says "we use EF Core 8," they mean the version of the O/RM that works with .NET 8.

---

### Purposes

- To provide a common vocabulary across documentation, tooling, and community.
- To distinguish the modern O/RM from legacy EF6.
- To align version support with .NET support lifecycles.
- To signal feature availability based on version.
- To help teams plan upgrades.

---

### Syntax Rules and Structure

#### Version Alignment

```
.NET 6  -> EF Core 6
.NET 7  -> EF Core 7
.NET 8  -> EF Core 8
.NET 9  -> EF Core 9
```

**Component breakdown**

- .NET version — The runtime and SDK version.
- EF Core version — The NuGet package version compatible with that runtime.

#### Syntax Rules

- All EF Core packages in a project should use the same major version.
- Provider packages must match the EF Core major version.
- EF Core 8 requires .NET 8; it does not run on .NET 6.

#### Constraints and Limitations

- EF Core 5 and earlier are out of support.
- Preview versions should not be used in production.
- Some features are provider-specific and version-specific (e.g., JSON columns, temporal tables).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Checking EF Core Version at Runtime

```csharp
using System;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        var version = typeof(DbContext).Assembly.GetName().Version;
        Console.WriteLine($"EF Core version: {version}");
    }
}
```

**Expected Output (example for EF Core 8)**

```
EF Core version: 8.0.0.0
```

**Why this output occurs**

- The assembly version reflects the EF Core package version.
- The major version aligns with the .NET version in use.

---

### Real-World Cases

- **Upgrade planning** — Teams check EF Core versions before upgrading .NET.
- **Package management** — Central package management ensures consistent EF Core versions across projects.
- **Troubleshooting** — Version mismatches between core and provider packages cause runtime errors.

---

### References

- EF Core releases and planning - https://learn.microsoft.com/en-us/ef/core/what-is-new/
- EF Core support policy - https://learn.microsoft.com/en-us/ef/core/what-is-new/
- .NET support policy - https://dotnet.microsoft.com/platform/support/policy/dotnet-core

---

## 3. `DbContext`

### Definitions

**Core Definition**
`DbContext` represents a session with the database and is the primary class for querying and saving data.

**Technical Definition**
`DbContext` combines the Unit of Work and Repository patterns. It holds a model (metadata), a change tracker, a database connection, and `DbSet<TEntity>` properties. It provides methods such as `SaveChanges`, `Find`, `Add`, `Update`, `Remove`, and `Entry`. The context is designed to be short-lived: typically one per request or one per unit of work.

**Beginner-Friendly Explanation**
`DbContext` is your "conversation" with the database. You open it, ask questions (queries), make changes (add/update/delete), and then save those changes. When you are done, you dispose of it.

---

### Purposes

- To provide a unit-of-work boundary for changes.
- To expose `DbSet<T>` properties for querying.
- To manage change tracking for loaded entities.
- To configure the model and provider through options.
- To coordinate transaction and connection lifetime.

---

### Syntax Rules and Structure

#### General Syntax

```
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Blog> Blogs => Set<Blog>();
    public DbSet<Post> Posts => Set<Post>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().HasKey(b => b.Id);
        // ...
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // provider configuration (if not configured externally)
    }
}
```

**Component breakdown**

- `DbContextOptions<T>` — Configuration object holding provider, connection string, and options.
- `DbSet<T>` — Queryable collection of entities.
- `OnModelCreating` — Fluent API configuration of the model.
- `OnConfiguring` — Provider configuration when not supplied externally.
- `SaveChanges` / `SaveChangesAsync` — Persists tracked changes.

#### Syntax Rules

- The context must be disposed after use (`using` or DI scope).
- The context is not thread-safe; do not use it concurrently from multiple threads.
- The context should be registered as scoped in DI for web apps.
- `OnConfiguring` is only called if options are not supplied to the constructor.
- Model configuration can be done via data annotations or the Fluent API.

#### Constraints and Limitations

- A context instance should not be used across multiple units of work.
- The context is not designed to be long-lived in web applications.
- Concurrent access from multiple threads causes `InvalidOperationException`.
- `OnConfiguring` must not be used when options are supplied externally.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Context with Constructor Injection

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products => Set<Product>();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

// Program.cs (ASP.NET Core)
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

app.MapGet("/products", async (AppDbContext db) =>
    await db.Products.ToListAsync());

app.Run();
```

**Expected Output (conceptual)**

```
HTTP GET /products returns a JSON array of products
```

**Why this output occurs**

- The context is registered with a scoped lifetime.
- The endpoint receives an `AppDbContext` via DI.
- The query is translated to SQL and executed against the database.

---

#### Example 2: SaveChanges and Change Tracking

```csharp
using var db = new AppDbContext(options);

var product = new Product { Name = "Keyboard", Price = 49.99m };
db.Products.Add(product);
await db.SaveChangesAsync();   // product.Id is populated

product.Price = 39.99m;
await db.SaveChangesAsync();   // generates UPDATE for Price column
```

**Expected Output (conceptual SQL)**

```
INSERT INTO [Products] ([Name], [Price]) VALUES ('Keyboard', 49.99);
UPDATE [Products] SET [Price] = 39.99 WHERE [Id] = 1;
```

**Why this output occurs**

- `Add` marks the entity as `Added`.
- `SaveChangesAsync` generates and executes the INSERT.
- Modifying `Price` marks the entity as `Modified`.
- The second `SaveChangesAsync` generates an UPDATE with the changed columns.

---

### Real-World Cases

- **ASP.NET Core APIs** — One `DbContext` per HTTP request via scoped DI.
- **Unit of work in services** — A service method opens a context, performs changes, and saves once.
- **Reporting** — Read-only contexts with `AsNoTracking` for performance.

---

### References

- DbContext lifecycle, configuration, and initialization - https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- DbContext class - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext
- DbContext in EF Core - https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

---

## 4. `DbSet`

### Definitions

**Core Definition**
`DbSet<TEntity>` represents a collection of all entities of a given type in the context, from which queries are started.

**Technical Definition**
`DbSet<TEntity>` is a queryable collection that implements `IQueryable<TEntity>` and `IAsyncEnumerable<TEntity>`. It acts as the entry point for LINQ queries against an entity type. `DbSet` also provides methods such as `Add`, `AddRange`, `Update`, `Remove`, `Find`, `Attach`, and `AsQueryable`. The `Set<TEntity>()` method returns the `DbSet` for a type that is not exposed as a property.

**Beginner-Friendly Explanation**
A `DbSet` is like a "table view" inside your context. `context.Blogs` represents the `Blogs` table. You can query it with LINQ or add new entities to it.

---

### Purposes

- To provide a query entry point for an entity type.
- To expose entity-type-specific CRUD operations.
- To allow LINQ queries to be translated to SQL.
- To support `Find` for primary-key lookups with change-tracking awareness.
- To support `Set<T>()` for entities without explicit properties.

---

### Syntax Rules and Structure

#### General Syntax

```
public DbSet<Blog> Blogs { get; set; }
// or expression-bodied:
public DbSet<Blog> Blogs => Set<Blog>();

// Query
IQueryable<Blog> query = context.Blogs.Where(b => b.Name.StartsWith("A"));

// Find by primary key
Blog? blog = context.Blogs.Find(1);

// Add
context.Blogs.Add(new Blog { Name = "New" });
```

**Component breakdown**

- `DbSet<T>` — The collection property or `Set<T>()` result.
- `Where`, `Select`, etc. — LINQ operators translated to SQL.
- `Find` — Returns a tracked entity if already loaded; otherwise queries by primary key.
- `Add` — Marks an entity as `Added`.

#### Syntax Rules

- `DbSet` properties should be exposed on the context; use `Set<T>()` for types without a property.
- `DbSet` is queryable but not a collection in the `ICollection<T>` sense; it does not implement `IList<T>`.
- `Find` requires the primary key type to match.
- `DbSet` operations respect change tracking.

#### Constraints and Limitations

- `DbSet` is not thread-safe; do not query concurrently from multiple threads.
- `Find` only supports primary-key lookups, not composite keys with more than the key shape.
- `DbSet` does not materialize results until enumerated or converted to a list.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Querying and Finding

```csharp
using Microsoft.EntityFrameworkCore;

var recent = await db.Blogs
    .Where(b => b.CreatedAt > DateTime.UtcNow.AddDays(-7))
    .OrderByDescending(b => b.CreatedAt)
    .ToListAsync();

var blog = await db.Blogs.FindAsync(1);
Console.WriteLine(blog?.Name ?? "not found");
```

**Expected Output (example)**

```
EF Core Intro
```

**Why this output occurs**

- `Where`, `OrderByDescending`, and `ToListAsync` are translated to SQL and executed.
- `FindAsync(1)` queries by primary key and returns the entity.

---

#### Example 2: `Set<T>()` for Entities Without Properties

```csharp
var audit = db.Set<AuditLog>().Add(new AuditLog { Message = "Created" });
await db.SaveChangesAsync();

var logs = await db.Set<AuditLog>().ToListAsync();
foreach (var log in logs) Console.WriteLine(log.Message);
```

**Expected Output (conceptual)**

```
Created
```

**Why this output occurs**

- `Set<AuditLog>()` returns the `DbSet` for the entity type even though it is not exposed as a property.
- The entity is added, saved, and queried.

---

### Real-World Cases

- **Repository-style access** — Services use `DbSet` for querying and CRUD.
- **Dynamic entity types** — `Set<T>()` supports generic repositories.
- **Primary-key lookups** — `Find` avoids a database round trip for already-tracked entities.

---

### References

- DbSet class - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbset-1
- Querying data with EF Core - https://learn.microsoft.com/en-us/ef/core/querying/
- DbSet and Set in EF Core - https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

---

## 5. Entities

### Definitions

**Core Definition**
An entity is a CLR class that EF Core maps to a table and whose instances represent rows.

**Technical Definition**
An entity type is a class or record included in the model. EF Core discovers entities through `DbSet` properties, `OnModelCreating` configuration, or conventions. Entities have a primary key, scalar properties mapped to columns, navigation properties mapped to relationships, and may participate in inheritance hierarchies. EF Core can map to existing databases (database-first) or generate schema from the model (code-first).

**Beginner-Friendly Explanation**
An entity is a C# class that represents a table row. A `Blog` object is a row in the `Blogs` table. Properties are columns.

---

### Purposes

- To model domain data as strongly typed C# objects.
- To map classes to tables through conventions or configuration.
- To define navigation properties for relationships.
- To support inheritance mapping strategies.
- To enable change tracking and persistence.

---

### Syntax Rules and Structure

#### General Syntax

```
public class Blog
{
    public int Id { get; set; }                 // primary key by convention
    public string Name { get; set; } = "";
    public DateTime CreatedAt { get; set; }
    public List<Post> Posts { get; set; } = new();  // navigation property
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public int BlogId { get; set; }             // foreign key
    public Blog? Blog { get; set; }             // reference navigation
}
```

**Component breakdown**

- `Id` — Primary key by convention (`Id` or `<TypeName>Id`).
- Scalar properties — Mapped to columns.
- `BlogId` — Foreign key by convention.
- `Posts` — Collection navigation.
- `Blog` — Reference navigation.

#### Syntax Rules

- A primary key is required; conventions look for `Id` or `<TypeName>Id`.
- Properties must have a getter and setter (or be init-only/backed by a field for some scenarios).
- Navigation properties must be virtual for lazy loading.
- Entities should have a parameterless constructor unless configured otherwise (EF Core 8 supports constructor binding).
- Collections should be initialized to avoid null-reference errors.

#### Constraints and Limitations

- EF Core cannot map fields directly; properties are required (with some exceptions for backing fields).
- Not all CLR types are supported natively; value converters may be needed.
- Entities with no key cannot be tracked for updates.
- Owned types and complex types have different mapping rules.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Code-First Entity and Configuration

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public List<Post> Posts { get; set; } = new();
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public int BlogId { get; set; }
    public Blog? Blog { get; set; }
}

public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs => Set<Blog>();
    public DbSet<Post> Posts => Set<Post>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>(b =>
        {
            b.Property(x => x.Name).HasMaxLength(200).IsRequired();
        });
    }
}
```

**Expected Output (generated schema, conceptual)**

```
CREATE TABLE [Blogs] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [Name] nvarchar(200) NOT NULL
);

CREATE TABLE [Posts] (
    [Id] int NOT NULL IDENTITY PRIMARY KEY,
    [Title] nvarchar(max) NOT NULL,
    [BlogId] int NOT NULL REFERENCES [Blogs]([Id])
);
```

**Why this output occurs**

- Conventions map `Id` to primary keys and `BlogId` to a foreign key.
- Fluent API configures `Name` as required with a maximum length.

---

### Real-World Cases

- **E-commerce** — `Product`, `Order`, `Customer` entities.
- **Content management** — `Article`, `Category`, `Tag` entities.
- **IoT** — `Device`, `Reading`, `Alert` entities.

---

### References

- Creating and configuring a model - https://learn.microsoft.com/en-us/ef/core/modeling/
- Entity types - https://learn.microsoft.com/en-us/ef/core/modeling/entity-types
- Keys and relationships - https://learn.microsoft.com/en-us/ef/core/modeling/keys

---

## 6. Relationships

### Definitions

**Core Definition**
A relationship defines how two entities are associated, typically through foreign keys, and is represented in C# by navigation properties.

**Technical Definition**
EF Core models three relationship cardinalities: one-to-one, one-to-many, and many-to-many. A relationship has a principal end, a dependent end, a foreign key on the dependent, and navigation properties on one or both ends. Relationships are configured by convention, data annotations, or the Fluent API. Many-to-many relationships use a join entity, which can be implicit (shadow) or explicit.

**Beginner-Friendly Explanation**
Relationships are how tables connect. A blog has many posts (one-to-many). A user has one profile (one-to-one). A student has many courses and a course has many students (many-to-many).

---

### Purposes

- To model real-world associations between data.
- To enable navigation from one entity to related entities.
- To enforce referential integrity through foreign keys.
- To support eager, lazy, and explicit loading.
- To allow cascade delete and other referential actions.

---

### Sub-Feature: One-to-One

#### Definitions

**Core Definition**
A one-to-one relationship links one principal entity to at most one dependent entity.

**Technical Definition**
In a one-to-one relationship, the dependent’s foreign key is also unique. EF Core requires the foreign key property on the dependent to be unique (often the dependent’s primary key). The principal navigation is a reference, and the dependent navigation is a reference back.

**Beginner-Friendly Explanation**
A user has one profile, and a profile belongs to one user.

#### Purposes

- To split large entities into optional or sensitive parts.
- To model optional dependent data.
- To enforce uniqueness across two tables.

#### Syntax Rules and Structure

```
public class User
{
    public int Id { get; set; }
    public Profile? Profile { get; set; }   // reference navigation
}

public class Profile
{
    public int Id { get; set; }             // PK and FK (shared primary key)
    public string Bio { get; set; } = "";
    public User User { get; set; } = null!;
}

// Fluent API
modelBuilder.Entity<User>()
    .HasOne(u => u.Profile)
    .WithOne(p => p.User)
    .HasForeignKey<Profile>(p => p.Id);
```

**Component breakdown**

- `User.Profile` — Reference navigation from principal to dependent.
- `Profile.User` — Reference navigation back to principal.
- `HasForeignKey<Profile>(p => p.Id)` — Configures the dependent’s FK; sharing the PK enforces one-to-one.

#### Syntax Rules

- The dependent’s foreign key must be unique.
- One-to-one is configured with `HasOne(...).WithOne(...)`.
- The dependent’s FK is often its primary key.
- Cascade delete behaviour is configurable.

#### Constraints and Limitations

- One-to-one requires a unique constraint on the dependent FK.
- Optional one-to-one requires nullable FK and navigation.
- EF Core cannot enforce one-to-one without a unique index.

#### Annotated Code Example

```csharp
var user = new User { Profile = new Profile { Bio = "Hello" } };
db.Users.Add(user);
await db.SaveChangesAsync();

var loaded = await db.Users.Include(u => u.Profile).FirstAsync();
Console.WriteLine(loaded.Profile?.Bio ?? "no profile");
```

**Expected Output**

```
Hello
```

**Why this output occurs**

- The profile is inserted with the same key as the user.
- `Include` eagerly loads the profile.
- The bio is printed.

#### Real-World Cases

- User/profile, customer/billing address, order/shipping details.
- Sensitive data isolation (e.g., credentials in a separate table).

#### References

- One-to-one relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-one
- Relationships overview - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/

---

### Sub-Feature: One-to-Many

#### Definitions

**Core Definition**
A one-to-many relationship links one principal entity to many dependent entities.

**Technical Definition**
The principal has a collection navigation; the dependent has a reference navigation and a foreign key. The foreign key on the dependent is not unique. EF Core configures one-to-many by convention when it detects a collection navigation and a corresponding FK.

**Beginner-Friendly Explanation**
A blog has many posts; each post belongs to one blog.

#### Purposes

- To model parent/child data (orders and line items).
- To support cascade delete of children.
- To enable filtering and projection across related data.

#### Syntax Rules and Structure

```
public class Blog
{
    public int Id { get; set; }
    public List<Post> Posts { get; set; } = new();
}

public class Post
{
    public int Id { get; set; }
    public int BlogId { get; set; }         // FK
    public Blog? Blog { get; set; }         // reference navigation
}

// Fluent API (optional if conventions apply)
modelBuilder.Entity<Blog>()
    .HasMany(b => b.Posts)
    .WithOne(p => p.Blog)
    .HasForeignKey(p => p.BlogId)
    .OnDelete(DeleteBehavior.Cascade);
```

**Component breakdown**

- `Posts` — Collection navigation on principal.
- `BlogId` — FK on dependent.
- `Blog` — Reference navigation on dependent.
- `OnDelete` — Referential action for cascade delete.

#### Syntax Rules

- The FK must be on the dependent.
- The principal collection and dependent reference are conventionally paired.
- Cascade delete is the default for required relationships.

#### Constraints and Limitations

- Nullable FK means optional relationship; non-nullable means required.
- Cascade delete cycles can be rejected by SQL Server.
- Deleting a principal with dependents requires cascade or explicit handling.

#### Annotated Code Example

```csharp
var blog = new Blog { Posts = { new Post { Title = "A" }, new Post { Title = "B" } } };
db.Blogs.Add(blog);
await db.SaveChangesAsync();

var posts = await db.Posts.Where(p => p.BlogId == blog.Id).ToListAsync();
Console.WriteLine(posts.Count);
```

**Expected Output**

```
2
```

**Why this output occurs**

- The blog and its posts are inserted; FKs are populated.
- The query filters posts by `BlogId`.

#### Real-World Cases

- Blogs/posts, orders/line items, categories/products.
- Parent/child audit trails.

#### References

- One-to-many relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-many
- Cascade delete - https://learn.microsoft.com/en-us/ef/core/saving/cascade-delete

---

### Sub-Feature: Many-to-Many

#### Definitions

**Core Definition**
A many-to-many relationship links many entities on each side through a join table.

**Technical Definition**
EF Core 5+ supports many-to-many with an implicit join entity (shadow properties) or an explicit join entity. The join table has FKs to both sides, and its primary key is typically the composite of both FKs. Navigation properties are collections on both sides.

**Beginner-Friendly Explanation**
Students take many courses; courses have many students. The database needs a third table to link them.

#### Purposes

- To model associations where both sides can have many related rows.
- To allow additional data on the relationship (explicit join entity).
- To simplify querying across the join table.

#### Syntax Rules and Structure

```
// Implicit join
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

// Explicit join
public class Enrollment
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public Student Student { get; set; } = null!;
    public Course Course { get; set; } = null!;
    public DateTime EnrolledAt { get; set; }
}
```

**Component breakdown**

- Collections on both sides — Configure many-to-many.
- Implicit join — EF Core creates a shadow join entity.
- Explicit join — A CLR class with two FKs and optional payload.

#### Syntax Rules

- Many-to-many is configured with `HasMany(...).WithMany(...)`.
- Explicit join entities are configured as normal entities with composite keys.
- Implicit joins can be queried through the collections.

#### Constraints and Limitations

- Implicit joins cannot carry payload data; use explicit joins when extra columns are needed.
- Join table naming and column names follow conventions but can be configured.
- Deleting one side cascades through the join table.

#### Annotated Code Example

```csharp
var s = new Student { Courses = { new Course { Name = "Math" } } };
db.Students.Add(s);
await db.SaveChangesAsync();

var students = await db.Students
    .Include(x => x.Courses)
    .ToListAsync();

Console.WriteLine(students[0].Courses.Count);
```

**Expected Output**

```
1
```

**Why this output occurs**

- EF Core inserts the student, course, and join row.
- `Include` loads the courses for the student.

#### Real-World Cases

- Students/courses, products/tags, users/roles.
- Many-to-many with metadata (enrollment date, role assignment date).

#### References

- Many-to-many relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/many-to-many
- Joining relationships - https://learn.microsoft.com/en-us/ef/core/querying/related-data/

---

## 7. Migrations

### Definitions

**Core Definition**
Migrations are a code-based way to incrementally update the database schema to match the EF Core model.

**Technical Definition**
A migration is a C# class with `Up` and `Down` methods generated by the EF Core tools. Migrations are stored in the project and applied in order. The `__EFMigrationsHistory` table records which migrations have been applied. Migrations support generating SQL scripts, applying at startup, and bundling into executables.

**Beginner-Friendly Explanation**
Migrations are "version control for your database schema." When you change an entity, you create a migration, and EF Core generates the SQL to update the database.

---

### Purposes

- To evolve the schema without losing data.
- To keep schema changes under source control.
- To generate SQL scripts for review and deployment.
- To apply changes consistently across environments.
- To support rollback through `Down` methods.

---

### Syntax Rules and Structure

#### General Syntax

```
# Install tools
dotnet tool install --global dotnet-ef

# Add packages
dotnet add package Microsoft.EntityFrameworkCore.Design

# Create a migration
dotnet ef migrations add InitialCreate

# Apply migrations
dotnet ef database update

# Generate SQL script
dotnet ef migrations script --idempotent -o migrate.sql

# List migrations
dotnet ef migrations list

# Remove last migration (if not applied)
dotnet ef migrations remove

# Bundle migrations into an executable
dotnet ef migrations bundle
```

**Component breakdown**

- `migrations add` — Scaffolds a migration from the current model diff.
- `database update` — Applies pending migrations.
- `migrations script` — Outputs SQL for deployment.
- `migrations bundle` — Creates a self-contained executable for deployment.

#### Syntax Rules

- The `Design` package is required for tooling.
- The project must reference a provider and the design package.
- Migrations are applied in order; never edit applied migrations.
- Use idempotent scripts for production deployments.

#### Constraints and Limitations

- Migrations are provider-specific; switching providers requires new migrations.
- Some schema changes (e.g., column renames) require manual intervention to avoid data loss.
- Concurrent migrations from multiple instances require coordination.
- `EnsureCreated` is incompatible with migrations; choose one approach.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Create and Apply a Migration

```csharp
// After defining Blog and Post entities:
// dotnet ef migrations add InitialCreate

// Generated migration (simplified)
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Blogs",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 200, nullable: false)
            },
            constraints: table => table.PrimaryKey("PK_Blogs", x => x.Id));
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Blogs");
    }
}
```

**Expected Output (conceptual SQL)**

```
CREATE TABLE [Blogs] (
    [Id] int NOT NULL IDENTITY,
    [Name] nvarchar(200) NOT NULL,
    CONSTRAINT [PK_Blogs] PRIMARY KEY ([Id])
);
```

**Why this output occurs**

- The migration captures the model diff from the previous snapshot.
- `database update` executes the `Up` method's operations as SQL.

---

### Real-World Cases

- **CI/CD pipelines** — Generate idempotent SQL scripts and apply during deployment.
- **Multi-environment** — Development, staging, and production databases evolve through the same migration history.
- **Bundles** — Deploy migrations as self-contained executables in restricted environments.

---

### References

- Migrations overview - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/
- Managing migrations - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying migrations - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying

---

## 8. Change Tracking

### Definitions

**Core Definition**
Change tracking is the mechanism by which EF Core monitors changes to loaded entities so that `SaveChanges` can generate the appropriate SQL.

**Technical Definition**
When an entity is queried, EF Core creates a tracked instance and records its state (`Unchanged`, `Added`, `Modified`, `Deleted`, `Detached`). Property values are compared to original values to determine modifications. `ChangeTracker` exposes entries, original values, and state manipulation. `AsNoTracking` disables tracking for read-only queries. `AsNoTrackingWithIdentityResolution` supports identity resolution without tracking.

**Beginner-Friendly Explanation**
Change tracking is EF Core "remembering" what the database looked like when it loaded an entity, so it can tell what you changed and generate the right UPDATE statement.

---

### Purposes

- To generate minimal UPDATE statements with only changed columns.
- To support optimistic concurrency through concurrency tokens.
- To enable `SaveChanges` to know what to insert, update, or delete.
- To allow inspection of original and current values.
- To support `AsNoTracking` for read-only performance.

---

### Syntax Rules and Structure

#### General Syntax

```
// Default tracking
var blog = await db.Blogs.FirstAsync();   // tracked

// No tracking (read-only)
var blogs = await db.Blogs.AsNoTracking().ToListAsync();

// No tracking with identity resolution
var blogs2 = await db.Blogs
    .AsNoTrackingWithIdentityResolution()
    .Include(b => b.Posts)
    .ToListAsync();

// Inspect change tracker
var entries = db.ChangeTracker.Entries();
foreach (var e in entries)
    Console.WriteLine($"{e.Entity.GetType().Name}: {e.State}");
```

**Component breakdown**

- `AsNoTracking` — Disables tracking; entities are detached.
- `AsNoTrackingWithIdentityResolution` — Deduplicates entities without tracking.
- `ChangeTracker.Entries()` — Returns tracked entries and their states.
- `Entry(entity).State` — Gets or sets the state.

#### Syntax Rules

- Tracked entities are unique per context instance by primary key.
- `SaveChanges` processes entries in dependency order.
- `AsNoTracking` is recommended for read-only queries.
- `Attach`, `Update`, `Remove` change state explicitly.

#### Constraints and Limitations

- Tracking has memory and CPU overhead.
- Tracked entities are not thread-safe.
- `AsNoTracking` entities are not updated by `SaveChanges`.
- Identity resolution with `AsNoTracking` requires `AsNoTrackingWithIdentityResolution`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Tracking vs. No Tracking

```csharp
var tracked = await db.Blogs.FirstAsync();
tracked.Name = "Changed";
await db.SaveChangesAsync();   // UPDATE executed

var untracked = await db.Blogs.AsNoTracking().FirstAsync();
untracked.Name = "Ignored";
await db.SaveChangesAsync();   // no UPDATE executed
```

**Expected Output (conceptual SQL)**

```
UPDATE [Blogs] SET [Name] = 'Changed' WHERE [Id] = 1;
-- no SQL for the untracked entity
```

**Why this output occurs**

- The tracked entity is monitored; the name change is detected.
- The untracked entity is detached; `SaveChanges` ignores it.

---

### Real-World Cases

- **CRUD APIs** — Tracked queries for updates; no-tracking queries for lists.
- **Reporting** — `AsNoTracking` reduces overhead.
- **Concurrency** — Rowversion columns for optimistic concurrency.

---

### References

- Change tracking in EF Core - https://learn.microsoft.com/en-us/ef/core/change-tracking/
- Tracking vs. no-tracking queries - https://learn.microsoft.com/en-us/ef/core/querying/tracking
- ChangeTracker API - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.changetracking.changetracker

---

## 9. LINQ Queries

### Definitions

**Core Definition**
LINQ queries in EF Core are C# query expressions or method chains that EF Core translates into SQL.

**Technical Definition**
EF Core's LINQ provider translates expression trees into provider-specific SQL. Supported operators include filtering (`Where`), projection (`Select`), ordering (`OrderBy`), grouping (`GroupBy`), joins, aggregations, and set operations. Queries are deferred until enumerated. Unsupported expressions throw `InvalidOperationException` at translation time.

**Beginner-Friendly Explanation**
You write C# LINQ like `db.Blogs.Where(b => b.Name.StartsWith("A"))`, and EF Core turns it into SQL like `SELECT * FROM Blogs WHERE Name LIKE 'A%'`.

---

### Purposes

- To query data using strongly typed C# instead of SQL strings.
- To compose queries dynamically with method chaining.
- To project results into DTOs.
- To perform aggregations and joins.
- To enable provider-agnostic queries.

---

### Syntax Rules and Structure

#### General Syntax

```
// Query syntax
var q = from b in db.Blogs
        where b.Name.StartsWith("A")
        orderby b.Name
        select b;

// Method syntax
var q2 = db.Blogs
    .Where(b => b.Name.StartsWith("A"))
    .OrderBy(b => b.Name)
    .Select(b => new { b.Id, b.Name });

// Projection to DTO
var dtos = await db.Blogs
    .Select(b => new BlogDto { Id = b.Id, Name = b.Name })
    .ToListAsync();
```

**Component breakdown**

- `from ... in ...` — Query source.
- `where` — Filtering.
- `orderby` — Ordering.
- `select` — Projection.
- `ToListAsync` — Execution (materialization).

#### Syntax Rules

- Queries are deferred; they execute when enumerated.
- Use `ToListAsync`, `FirstAsync`, `CountAsync`, etc. for async execution.
- Projection to DTOs reduces data transferred.
- Avoid client-side evaluation of unsupported expressions.

#### Constraints and Limitations

- Not all LINQ operators translate to SQL.
- Some expressions are evaluated client-side in older versions; EF Core 3+ throws for unsupported server-side translation.
- Complex queries may generate inefficient SQL.
- String comparisons and culture-sensitive operations may behave differently across providers.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Filtering and Projection

```csharp
var dtos = await db.Blogs
    .Where(b => b.Posts.Count > 0)
    .OrderBy(b => b.Name)
    .Select(b => new { b.Id, b.Name, PostCount = b.Posts.Count })
    .ToListAsync();

foreach (var d in dtos)
    Console.WriteLine($"{d.Id}: {d.Name} ({d.PostCount})");
```

**Expected Output (example)**

```
1: EF Core Intro (2)
```

**Why this output occurs**

- `Where` filters blogs with posts.
- `Select` projects id, name, and post count.
- The count is translated to a SQL aggregate or subquery.

---

#### Example 2: Join and GroupBy

```csharp
var counts = await db.Posts
    .GroupBy(p => p.BlogId)
    .Select(g => new { BlogId = g.Key, Count = g.Count() })
    .ToListAsync();

foreach (var c in counts)
    Console.WriteLine($"Blog {c.BlogId}: {c.Count} posts");
```

**Expected Output (example)**

```
Blog 1: 2 posts
```

**Why this output occurs**

- `GroupBy` and `Count` are translated to SQL `GROUP BY` and `COUNT`.
- Results are materialized as anonymous objects.

---

### Real-World Cases

- **APIs** — Filtering, sorting, and paging with LINQ.
- **Reports** — Aggregations and joins.
- **Search** — Dynamic query composition.

---

### References

- LINQ queries in EF Core - https://learn.microsoft.com/en-us/ef/core/querying/
- Querying data - https://learn.microsoft.com/en-us/ef/core/querying/
- Client vs. server evaluation - https://learn.microsoft.com/en-us/ef/core/querying/client-eval

---

## 10. Loading Strategies

### Definitions

**Core Definition**
Loading strategies determine when and how related data is loaded from the database.

**Technical Definition**
EF Core supports eager loading (`Include`/`ThenInclude`), lazy loading (proxies or `ILazyLoader`), and explicit loading (`Entry(...).Load()`). Eager loading uses joins or split queries; lazy loading issues queries on navigation access; explicit loading issues queries on demand. Each strategy has performance and correctness implications.

**Beginner-Friendly Explanation**
When you load a blog, do you also want its posts? Eager loading says "yes, load them now." Lazy loading says "load them only when I touch the `Posts` property." Explicit loading says "I will ask for them when I need them."

---

### Purposes

- To control the amount and timing of data fetched.
- To avoid N+1 query problems.
- To reduce memory usage for large graphs.
- To support read-only and projection scenarios.
- To balance latency and payload size.

---

### Sub-Feature: Eager Loading

#### Definitions

**Core Definition**
Eager loading fetches related data in the same query as the principal entity.

**Technical Definition**
`Include` and `ThenInclude` specify navigation properties to load. EF Core generates joins or split queries depending on configuration (`AsSplitQuery`). Filtered includes are supported.

**Beginner-Friendly Explanation**
Eager loading is "bring the related data along with the main data in one go."

#### Purposes

- To avoid N+1 queries.
- To load a complete graph in one round trip.
- To support deterministic data shape for APIs.

#### Syntax Rules and Structure

```
var blogs = await db.Blogs
    .Include(b => b.Posts)
        .ThenInclude(p => p.Author)
    .ToListAsync();

// Filtered include
var blogs2 = await db.Blogs
    .Include(b => b.Posts.Where(p => p.IsPublished))
    .ToListAsync();

// Split query
var blogs3 = await db.Blogs
    .Include(b => b.Posts)
    .AsSplitQuery()
    .ToListAsync();
```

**Component breakdown**

- `Include` — Specifies a navigation to load.
- `ThenInclude` — Specifies nested navigation.
- `AsSplitQuery` — Uses separate SQL queries instead of joins.

#### Constraints and Limitations

- Multiple collection includes can cause Cartesian explosion.
- `AsSplitQuery` avoids explosion but increases round trips.
- Filtered includes have limitations (e.g., cannot filter on the principal).

#### Annotated Code Example

```csharp
var blog = await db.Blogs
    .Include(b => b.Posts)
    .FirstAsync(b => b.Id == 1);

Console.WriteLine(blog.Posts.Count);
```

**Expected Output**

```
2
```

**Why this output occurs**

- `Include` loads posts in the same query (or split query).
- The navigation is populated without additional queries.

#### Real-World Cases

- API endpoints returning aggregates.
- Reporting with related data.

#### References

- Eager loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/eager
- Single vs. split queries - https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries

---

### Sub-Feature: Lazy Loading

#### Definitions

**Core Definition**
Lazy loading defers loading of related data until the navigation property is accessed.

**Technical Definition**
Lazy loading requires either proxies (`Microsoft.EntityFrameworkCore.Proxies`) or `ILazyLoader` injection. Navigation properties must be `virtual`. When accessed, EF Core issues a query if the navigation is not loaded. Lazy loading can cause N+1 queries and must be used carefully.

**Beginner-Friendly Explanation**
Lazy loading is "load the related data only when I ask for it."

#### Purposes

- To avoid loading data that may not be needed.
- To simplify code that navigates graphs.
- To support optional navigation scenarios.

#### Syntax Rules and Structure

```
// Install package
// dotnet add package Microsoft.EntityFrameworkCore.Proxies

optionsBuilder.UseLazyLoadingProxies();

public class Blog
{
    public int Id { get; set; }
    public virtual List<Post> Posts { get; set; } = new();
}

// Usage
var blog = await db.Blogs.FirstAsync();
var count = blog.Posts.Count;   // triggers query
```

**Component breakdown**

- `UseLazyLoadingProxies` — Enables proxy-based lazy loading.
- `virtual` — Required on navigation properties.
- Accessing navigation — Triggers a query.

#### Constraints and Limitations

- N+1 query risk.
- Proxies require virtual navigations and public parameterless constructors.
- Lazy loading is not available with `AsNoTracking` proxies.
- Serialization can trigger unintended queries.

#### Annotated Code Example

```csharp
var blog = await db.Blogs.FirstAsync();
Console.WriteLine(blog.Posts.Count);   // issues SELECT for posts
```

**Expected Output**

```
2
```

**Why this output occurs**

- Accessing `Posts` triggers a lazy load query.
- The count reflects the loaded posts.

#### Real-World Cases

- UI applications with navigation.
- Complex graphs where only some branches are needed.

#### References

- Lazy loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy
- Lazy loading proxies - https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy#lazy-loading-with-proxies

---

### Sub-Feature: Explicit Loading

#### Definitions

**Core Definition**
Explicit loading loads related data on demand through explicit API calls.

**Technical Definition**
`Entry(entity).Reference(...).Load()` and `Entry(entity).Collection(...).Load()` issue queries for the specified navigation. `Query()` allows filtering and further composition.

**Beginner-Friendly Explanation**
Explicit loading is "I will explicitly ask EF Core to load this navigation when I need it."

#### Purposes

- To load related data only when needed without proxies.
- To support conditional loading.
- To avoid N+1 by batching explicit loads.

#### Syntax Rules and Structure

```
var blog = await db.Blogs.FirstAsync();

await db.Entry(blog).Collection(b => b.Posts).LoadAsync();

await db.Entry(blog).Reference(b => b.Owner).LoadAsync();

// Filtered explicit load
await db.Entry(blog)
    .Collection(b => b.Posts)
    .Query()
    .Where(p => p.IsPublished)
    .LoadAsync();
```

**Component breakdown**

- `Entry(entity)` — Returns an `EntityEntry`.
- `Collection(...)` / `Reference(...)` — Specify navigation.
- `Load` / `LoadAsync` — Executes the query.
- `Query()` — Returns an `IQueryable` for filtering.

#### Constraints and Limitations

- Explicit loading requires a tracked entity.
- Multiple explicit loads can cause multiple round trips.
- `Query()` is only available for collection navigations.

#### Annotated Code Example

```csharp
var blog = await db.Blogs.FirstAsync();
await db.Entry(blog).Collection(b => b.Posts).LoadAsync();
Console.WriteLine(blog.Posts.Count);
```

**Expected Output**

```
2
```

**Why this output occurs**

- The entity is tracked.
- `LoadAsync` queries the posts and populates the navigation.

#### Real-World Cases

- APIs with optional includes.
- Conditional loading based on user input.

#### References

- Explicit loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/explicit

---

## 11. Performance

### Definitions

**Core Definition**
EF Core performance is the efficiency with which queries execute and changes persist against the database.

**Technical Definition**
Performance encompasses query translation quality, indexing, query execution plans, data transfer volume, change-tracking overhead, connection management, and batching. Key techniques include `AsNoTracking`, projection, compiled queries, split queries, batching, and indexing.

**Beginner-Friendly Explanation**
Performance is "how fast and cheap your database operations are." Good EF Core performance means efficient SQL, minimal data transfer, and avoiding unnecessary work.

---

### Purposes

- To reduce query latency and database load.
- To minimize data transferred over the network.
- To avoid N+1 queries and Cartesian explosions.
- To use indexes effectively.
- To batch writes and reduce round trips.

---

### Syntax Rules and Structure

#### Performance Techniques

```
// No tracking for read-only
var list = await db.Blogs.AsNoTracking().ToListAsync();

// Projection
var dtos = await db.Blogs.Select(b => new { b.Id, b.Name }).ToListAsync();

// Compiled query
var compiled = EF.CompileAsyncQuery((BloggingContext ctx, int id) =>
    ctx.Blogs.First(b => b.Id == id));

// Split query
var graph = await db.Blogs.Include(b => b.Posts).AsSplitQuery().ToListAsync();

// Batching
await db.AddRangeAsync(entities);
await db.SaveChangesAsync();

// Raw SQL for hot paths
var rows = await db.Blogs.FromSqlRaw("SELECT * FROM Blogs WHERE Id = {0}", id).ToListAsync();
```

**Component breakdown**

- `AsNoTracking` — Reduces tracking overhead.
- `Select` — Projects only needed columns.
- `EF.CompileAsyncQuery` — Caches query translation.
- `AsSplitQuery` — Avoids Cartesian explosion.
- `AddRange` — Batches inserts.

#### Syntax Rules

- Use `AsNoTracking` for read-only queries.
- Prefer projection to full entity loading.
- Index foreign keys and frequently filtered columns.
- Avoid `Include` for unused navigations.

#### Constraints and Limitations

- Over-optimization can reduce readability.
- Raw SQL bypasses some safety and portability.
- Compiled queries are limited in composition.
- Batching behavior depends on the provider.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: No Tracking and Projection

```csharp
var dtos = await db.Blogs
    .AsNoTracking()
    .Select(b => new { b.Id, b.Name })
    .ToListAsync();

Console.WriteLine(dtos.Count);
```

**Expected Output (example)**

```
1
```

**Why this output occurs**

- `AsNoTracking` avoids tracking overhead.
- Projection reduces data transferred.
- The count reflects the number of rows.

---

#### Example 2: Compiled Query

```csharp
static readonly Func<BloggingContext, int, Task<Blog?>> GetBlog =
    EF.CompileAsyncQuery((BloggingContext ctx, int id) =>
        ctx.Blogs.FirstOrDefault(b => b.Id == id));

var blog = await GetBlog(db, 1);
Console.WriteLine(blog?.Name ?? "none");
```

**Expected Output**

```
EF Core Intro
```

**Why this output occurs**

- The query is compiled once and reused.
- Execution avoids repeated translation overhead.

---

### Real-World Cases

- **High-traffic APIs** — No-tracking reads and projection.
- **Reporting** — Compiled queries and raw SQL.
- **Bulk operations** — `AddRange`, `ExecuteUpdate`, `ExecuteDelete` (EF Core 7+).

---

### References

- Performance considerations - https://learn.microsoft.com/en-us/ef/core/performance/
- Efficient querying - https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying
- Compiled queries - https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics#compiled-queries

---

## 12. EF Core Best Practices

### Definitions

**Core Definition**
EF Core best practices are recommended patterns and techniques for building maintainable, performant, and correct data-access code.

**Technical Definition**
Best practices cover context lifetime, query design, change tracking, migrations, concurrency, validation, and testing. They are supported by analyzers, documentation, and community guidance.

**Beginner-Friendly Explanation**
Best practices are "the habits that keep your data layer healthy." They include short-lived contexts, no-tracking reads, projection, explicit includes, and safe migrations.

---

### Purposes

- To prevent common bugs (N+1, stale data, concurrency issues).
- To improve performance and scalability.
- To keep the model and database schema in sync.
- To simplify testing.
- To align with .NET conventions.

---

### Guidelines and Structure

#### Context Lifetime

```
// Scoped per request (ASP.NET Core)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

- Use one context per request or unit of work.
- Do not share a context across threads.
- Dispose contexts promptly.

#### Querying

```
var dtos = await db.Blogs
    .AsNoTracking()
    .Where(b => b.IsActive)
    .Select(b => new BlogDto { Id = b.Id, Name = b.Name })
    .ToListAsync();
```

- Use `AsNoTracking` for reads.
- Project to DTOs.
- Use `Include` deliberately; avoid N+1.
- Use `AsSplitQuery` for multiple collections.

#### Writes

```
db.Blogs.Add(blog);
await db.SaveChangesAsync();
```

- Batch writes with `AddRange`.
- Use `ExecuteUpdate`/`ExecuteDelete` for bulk operations (EF Core 7+).
- Handle concurrency with rowversion tokens.

#### Migrations

```
dotnet ef migrations add AddRating
dotnet ef database update
```

- Review migrations before applying.
- Use idempotent scripts for production.
- Never edit applied migrations.

#### Testing

```
// SQLite in-memory
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open();
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlite(connection).Options;
```

- Use SQLite in-memory or a test container for integration tests.
- Avoid the in-memory provider for relational-behaviour tests.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Repository-Style Service with Best Practices

```csharp
public class BlogService
{
    private readonly AppDbContext _db;

    public BlogService(AppDbContext db) => _db = db;

    public async Task<List<BlogDto>> GetActiveBlogsAsync()
    {
        return await _db.Blogs
            .AsNoTracking()
            .Where(b => b.IsActive)
            .Select(b => new BlogDto { Id = b.Id, Name = b.Name })
            .ToListAsync();
    }

    public async Task AddBlogAsync(string name)
    {
        _db.Blogs.Add(new Blog { Name = name });
        await _db.SaveChangesAsync();
    }
}
```

**Expected Output (conceptual)**

```
GET /blogs returns active blogs; POST /blogs inserts a blog
```

**Why this output occurs**

- No-tracking projection for reads.
- Add + SaveChanges for writes.
- Context is scoped and disposed by DI.

---

### Real-World Cases

- **Enterprise APIs** — Scoped contexts, DTO projections, and migrations in CI.
- **Microservices** — Context per service with database-per-service.
- **Testing** — SQLite in-memory and test containers.

---

### References

- EF Core best practices - https://learn.microsoft.com/en-us/ef/core/
- DbContext configuration - https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- Testing with EF Core - https://learn.microsoft.com/en-us/ef/core/testing/

---

## 13. EF Core Pitfalls

### Definitions

**Core Definition**
EF Core pitfalls are common mistakes that cause bugs, performance problems, or data corruption.

**Technical Definition**
Pitfalls include N+1 queries, Cartesian explosion, improper context lifetime, concurrency issues, client evaluation, missing indexes, misuse of `AsNoTracking`, and migration mistakes.

**Beginner-Friendly Explanation**
Pitfalls are "the traps people fall into." Knowing them helps you avoid debugging sessions and production incidents.

---

### Purposes

- To help developers recognize and avoid common mistakes.
- To reduce production incidents.
- To improve performance and correctness.
- To guide code reviews.
- To support safe schema evolution.

---

### Guidelines and Structure

#### Pitfall List

1. **N+1 queries** — Lazy loading or querying in loops.
2. **Cartesian explosion** — Multiple collection includes with joins.
3. **Long-lived contexts** — Memory growth and stale data.
4. **Concurrent context use** — `InvalidOperationException`.
5. **Client evaluation surprises** — Unsupported expressions.
6. **Missing indexes** — Slow queries.
7. **Over-fetching** — Loading full entities for read-only views.
8. **Improper `AsNoTracking`** — Updates not persisted.
9. **Migration conflicts** — Concurrent or edited migrations.
10. **`EnsureCreated` vs. migrations** — Incompatible approaches.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: N+1 and How to Fix It

```csharp
// Pitfall: N+1
var blogs = await db.Blogs.ToListAsync();
foreach (var b in blogs)
    Console.WriteLine(b.Posts.Count);   // one query per blog

// Fix: eager loading
var blogs2 = await db.Blogs.Include(b => b.Posts).ToListAsync();
foreach (var b in blogs2)
    Console.WriteLine(b.Posts.Count);   // no extra queries
```

**Expected Output**

```
2
2
```

**Why this output occurs**

- The first loop issues one query per blog for `Posts`.
- The second loop uses included data.

---

#### Example 2: Long-Lived Context Pitfall

```csharp
// Pitfall: singleton context
services.AddSingleton<AppDbContext>();

// Fix: scoped context
services.AddScoped<AppDbContext>();
```

**Expected Output (conceptual)**

```
Scoped context: one instance per request
```

**Why this output occurs**

- A singleton context accumulates tracked entities and is not thread-safe.
- A scoped context isolates work per request.

---

### Real-World Cases

- **Production incidents** — N+1 queries causing timeouts.
- **Data corruption** — Concurrent context use.
- **Migration failures** — Edited applied migrations.

---

### References

- EF Core performance - https://learn.microsoft.com/en-us/ef/core/performance/
- EF Core pitfalls and anti-patterns - https://learn.microsoft.com/en-us/ef/core/
- Testing and troubleshooting - https://learn.microsoft.com/en-us/ef/core/testing/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| `EnsureCreated` | Not for use with migrations; suitable only for tests and prototypes. |
| Lazy loading proxies | Optional package; can cause N+1; use deliberately. |
| `AsNoTracking` | Safe for reads; entities are detached and cannot be updated by `SaveChanges`. |
| Client evaluation | EF Core 3+ throws for unsupported server-side translation instead of silently evaluating. |
| Split queries | Version-specific (EF Core 5+); avoid Cartesian explosion but increase round trips. |
| `ExecuteUpdate` / `ExecuteDelete` | EF Core 7+; bypass change tracking. |
| JSON columns | EF Core 7+ for some providers. |
| Temporal tables | SQL Server-specific. |
| In-memory provider | Not a relational database; not recommended for integration tests that rely on relational semantics. |
| Compiled queries | Limited composition; version-specific APIs. |
| `FromSqlRaw` | Bypasses some safety; use parameterized queries. |

---

## References (Consolidated)

- Entity Framework Core overview - https://learn.microsoft.com/en-us/ef/core/
- EF Core releases and versions - https://learn.microsoft.com/en-us/ef/core/what-is-new/
- DbContext lifecycle and configuration - https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- DbContext class - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext
- DbSet class - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbset-1
- Creating and configuring a model - https://learn.microsoft.com/en-us/ef/core/modeling/
- Entity types - https://learn.microsoft.com/en-us/ef/core/modeling/entity-types
- Relationships overview - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/
- One-to-one relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-one
- One-to-many relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-many
- Many-to-many relationships - https://learn.microsoft.com/en-us/ef/core/modeling/relationships/many-to-many
- Cascade delete - https://learn.microsoft.com/en-us/ef/core/saving/cascade-delete
- Migrations overview - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/
- Managing migrations - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing
- Applying migrations - https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying
- Change tracking in EF Core - https://learn.microsoft.com/en-us/ef/core/change-tracking/
- Tracking vs. no-tracking queries - https://learn.microsoft.com/en-us/ef/core/querying/tracking
- ChangeTracker API - https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.changetracking.changetracker
- Querying data with EF Core - https://learn.microsoft.com/en-us/ef/core/querying/
- Client vs. server evaluation - https://learn.microsoft.com/en-us/ef/core/querying/client-eval
- Eager loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/eager
- Lazy loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy
- Explicit loading - https://learn.microsoft.com/en-us/ef/core/querying/related-data/explicit
- Single vs. split queries - https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries
- Performance considerations - https://learn.microsoft.com/en-us/ef/core/performance/
- Efficient querying - https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying
- Compiled queries - https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics#compiled-queries
- Testing with EF Core - https://learn.microsoft.com/en-us/ef/core/testing/
- EF Core GitHub repository - https://github.com/dotnet/efcore