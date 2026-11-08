# Repository Pattern in ASP.NET Core: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) with **Entity Framework Core** as the primary persistence technology. The **repository pattern** is an architectural pattern that abstracts data access behind an interface. A critical framing note: **EF Core’s `DbContext` already implements Unit of Work and Repository patterns**, so wrapping it adds a layer that must justify its cost. The pattern is **still valuable** when it provides genuine decoupling, testability, or support for multiple data sources, but it is **not a universal default**—the official EF Core guidance cautions against layering abstractions without need. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The repository pattern mediates between the domain and data mapping layers using a collection‑like interface for accessing domain objects, hiding the details of data storage behind a clean abstraction. |
| **Technical Definition** | A repository encapsulates the logic required to access data sources. It centralises common data‑access functionality, providing better maintainability and decoupling the infrastructure or technology used to access databases from the domain model layer. Repositories typically expose CRUD operations and query methods via an interface (`IUserRepository`), with a concrete implementation (`UserRepository`) that may use EF Core, Dapper, raw ADO.NET, or an external API. |
| **Beginner‑Friendly Explanation** | The repository pattern is the **“data‑access assistant”** for your application. Instead of your controllers or services talking directly to the database, they talk to a repository—a class whose only job is to fetch, save, and delete data. This keeps database code out of your business logic, makes it easier to test (you can swap in a fake repository), and makes it easier to change databases later (you only change the repository implementation). |

### Key Characteristics

- **Abstraction** – Hides persistence details behind an interface.
- **Centralisation** – Consolidates data‑access logic for an aggregate in one place.
- **Testability** – Interfaces can be mocked or replaced with in‑memory fakes.
- **Decoupling** – Domain/business logic does not depend on EF Core, SQL, or connection strings.
- **Aggregate‑oriented** – Typically one repository per aggregate root, not per table.
- **Layer overlap** – EF Core already provides Unit of Work (`DbContext`) and Repository (`DbSet`) patterns; wrapping them must add value.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **EF Core** (`DbContext`, `DbSet`, LINQ queries).
- Familiarity with **dependency injection** and **service lifetimes**.
- Understanding of **Domain‑Driven Design (DDD)** aggregates (helpful but not required).

### Related Programming Areas

- **EF Core** – The typical underlying data‑access technology.
- **Unit of Work pattern** – Groups multiple repository operations into a single transaction.
- **Dependency injection** – Repositories are registered and injected as Scoped services.
- **Domain‑Driven Design** – Repositories are often used to abstract aggregate persistence.
- **Testing** – Repositories make unit testing services easier by allowing fake implementations.

### Core Concepts / Features

1. **Repository pattern** – The concept, benefits, and critiques.
2. **Repository interfaces** – The contracts that decouple consumers from persistence.
3. **Repository implementations** – The concrete classes that use EF Core, Dapper, or other technologies.
4. **Repository best practices** – Aggregate focus, testing, lifetime management, and avoiding anti‑patterns.

---

## 1. Repository Pattern (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The repository pattern abstracts data access behind a collection‑like interface, mediating between domain objects and data storage. |
| **Technical Definition** | A repository provides methods for retrieving and persisting domain objects (`GetByIdAsync`, `AddAsync`, `Remove`, `ListAsync`) while hiding the underlying data source (EF Core, Dapper, external API, in‑memory store). It typically exposes **IQueryable** for flexible querying or **specification methods** for predefined queries. Repositories are usually scoped per **aggregate root** in DDD, not per table. |
| **Beginner‑Friendly Explanation** | The repository pattern is the **“middleman” between your code and your database**. Your services say “give me the user with ID 5” to the repository, and the repository talks to the database. Your services never see SQL, connection strings, or `DbContext`. This makes your code cleaner and your tests simpler, because you can replace the real repository with a fake one. |

### Purposes

- **To** decouple domain/business logic from data‑access technology (EF Core, SQL, external APIs).
- **To** centralise data‑access logic for an aggregate in a single, testable class.
- **To** enable **unit testing** of business logic by substituting fake repositories.
- **To** provide a **consistent API** for data access across the application.
- **To** support **multiple data sources** or migration between persistence technologies.
- **To** encapsulate **complex queries** (e.g., specifications) behind meaningful method names.

### Syntax Rules and Structure

#### General Architecture

```text
Controller/Service → IRepository<T> → Repository<T> → DbContext → Database
```

**Component breakdown:**

| Layer | Responsibility |
|-------|----------------|
| **Consumer** (controller/service) | Depends on `IRepository<T>`, not EF Core. |
| **Interface** (`IRepository<T>`) | Defines the contract. |
| **Implementation** (`Repository<T>`) | Uses `DbContext`/`DbSet<T>` to fulfil the contract. |
| **DbContext** | EF Core’s unit of work and identity map. |
| **Database** | Actual persistence. |

#### General Syntax (Basic Interface and Implementation)

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<List<T>> ListAsync();
    Task AddAsync(T entity);
    void Remove(T entity);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    public Repository(AppDbContext context) => _context = context;

    public Task<T?> GetByIdAsync(int id) => _context.Set<T>().FindAsync(id).AsTask();
    public Task<List<T>> ListAsync() => _context.Set<T>().ToListAsync();
    public Task AddAsync(T entity) => _context.Set<T>().AddAsync(entity).AsTask();
    public void Remove(T entity) => _context.Set<T>().Remove(entity);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IRepository<T>` | Generic CRUD contract. |
| `AppDbContext` | The EF Core context—only the repository sees it. |
| `Set<T>()` | Accesses the `DbSet<T>` without exposing it publicly. |

**Syntax rules:**

- **Repositories should depend on abstractions** (their interfaces) at the consumer level.
- **The `DbContext` should not leak** through the repository interface (avoid returning `IQueryable` if you want strict encapsulation).
- **Repositories are Scoped** – one per request, matching the `DbContext` lifetime.

**Constraints and limitations:**

- **Repositories over EF Core are often redundant** – `DbContext` and `DbSet<T>` already implement Unit of Work and Repository patterns.
- **Returning `IQueryable<T>` leaks EF Core** – consumers can compose queries that the repository cannot control.
- **Generic repositories are rarely sufficient** – domain‑specific queries belong in specialized repositories.

### Annotated Complete Step-by-Step Code Example

```csharp
// 1. Generic repository interface
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<List<T>> ListAsync();
    Task AddAsync(T entity);
    void Remove(T entity);
}

// 2. EF Core implementation
public class EfRepository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;

    public EfRepository(AppDbContext context) => _context = context;

    public async Task<T?> GetByIdAsync(int id)
        => await _context.Set<T>().FindAsync(id);

    public async Task<List<T>> ListAsync()
        => await _context.Set<T>().AsNoTracking().ToListAsync();

    public async Task AddAsync(T entity)
        => await _context.Set<T>().AddAsync(entity);

    public void Remove(T entity)
        => _context.Set<T>().Remove(entity);
}

// 3. Registration
builder.Services.AddScoped(typeof(IRepository<>), typeof(EfRepository<>));

// 4. Consumer
public class CustomerService
{
    private readonly IRepository<Customer> _customers;
    public CustomerService(IRepository<Customer> customers) => _customers = customers;

    public async Task<Customer?> GetAsync(int id) => await _customers.GetByIdAsync(id);
}
```

**Step-by-step explanation:**

1. `IRepository<T>` defines a generic CRUD contract.
2. `EfRepository<T>` implements it using EF Core’s `Set<T>()`—the `DbContext` is hidden.
3. The repository is registered as an open generic, so any `IRepository<T>` resolves to `EfRepository<T>`.
4. `CustomerService` depends on `IRepository<Customer>`, not `DbContext`.

**Expected Output:** `CustomerService.GetAsync(1)` returns the customer with ID 1, retrieved via the repository; the service has no EF Core dependency.

**Why this output?** The repository abstracts `DbContext` behind `IRepository<Customer>`, fulfilling the pattern’s decoupling goal.

### Real-World Cases

- **Domain‑Driven Design applications:** Repositories abstract aggregate persistence and express domain concepts.
- **Applications with complex query logic:** Specialized repositories encapsulate reusable specifications.
- **Testing‑focused architectures:** Fake repositories replace real ones in unit tests.
- **Multi‑source applications:** Repositories abstract different data sources (SQL, external APIs, caches).

### References

- Repository pattern (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- Repository pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/repository.html
- EF Core: Do you need the repository pattern? – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design#the-repository-pattern

---

## 2. Repository Interfaces

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A repository interface is the contract that defines how consumers interact with a repository, decoupling them from the concrete data‑access implementation. |
| **Technical Definition** | Repository interfaces are typically `I{Entity}Repository` or `IRepository<T>`, declaring synchronous and asynchronous methods for CRUD and query operations. They should express **domain intent** (`GetActiveCustomersAsync`, `FindByEmailAsync`) rather than exposing raw data‑access primitives. The interface is what consumers depend on; implementations live in the infrastructure layer. |
| **Beginner‑Friendly Explanation** | A repository interface is the **“menu” of data operations**. It lists what you can do—find by ID, add a new entity, list all items—without saying **how**. Your services order from the menu; the kitchen (implementation) decides how to prepare the dish. Because services only see the menu, you can swap kitchens without changing the order. |

### Purposes

- **To** define a **clear contract** for data access that consumers depend on.
- **To** enable **dependency inversion**—high‑level modules depend on abstractions, not concrete data access.
- **To** support **testing** by allowing fake implementations of the interface.
- **To** express **domain intent** through meaningful method names rather than leaking EF Core concepts.
- **To** isolate **infrastructure concerns** (EF Core, SQL) from the application core.

### Syntax Rules and Structure

#### General Syntax (Generic Interface)

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IReadOnlyList<T>> ListAsync();
    Task AddAsync(T entity);
    void Remove(T entity);
}
```

#### General Syntax (Specialized Interface)

```csharp
public interface ICustomerRepository
{
    Task<Customer?> GetByIdAsync(int id);
    Task<Customer?> FindByEmailAsync(string email);
    Task<IReadOnlyList<Customer>> ListActiveAsync();
    Task AddAsync(Customer customer);
    void Remove(Customer customer);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Method names | Express **domain intent** (e.g., `FindByEmailAsync`). |
| Return types | Use `Task<T?>` for possibly missing entities; `IReadOnlyList<T>` for lists. |
| Parameters | Use domain concepts, not persistence primitives. |

**Syntax rules:**

- **Prefer `IReadOnlyList<T>` or `IEnumerable<T>`** over `List<T>` in interfaces to prevent callers from mutating collections.
- **Use `CancellationToken`** on asynchronous methods for graceful cancellation.
- **Return `Task`** for all database operations—synchronous calls block threads.
- **Do not return `IQueryable<T>`** unless you intentionally want consumers to compose queries (which leaks EF Core).

**Constraints and limitations:**

- **Returning `IQueryable<T>` breaks encapsulation** – it exposes EF Core’s query composition and lazy execution.
- **Interface bloat** – adding too many methods to a generic interface undermines its generality; use specialized interfaces instead.
- **Interfaces that mirror `DbSet<T>` add little value** – if the interface is just `Add`, `Remove`, `Find`, `ToList`, you may not need the abstraction.

### Annotated Complete Step-by-Step Code Example

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IReadOnlyList<Order>> ListForCustomerAsync(int customerId, CancellationToken ct = default);
    Task<IReadOnlyList<Order>> ListPendingAsync(CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    void Remove(Order order);
}

public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context) => _context = context;

    public Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
        => _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id, ct);

    public Task<IReadOnlyList<Order>> ListForCustomerAsync(int customerId, CancellationToken ct = default)
        => _context.Orders
            .AsNoTracking()
            .Where(o => o.CustomerId == customerId)
            .ToListAsync(ct)
            .ContinueWith(t => (IReadOnlyList<Order>)t.Result, ct);

    public Task<IReadOnlyList<Order>> ListPendingAsync(CancellationToken ct = default)
        => _context.Orders
            .AsNoTracking()
            .Where(o => o.Status == OrderStatus.Pending)
            .ToListAsync(ct)
            .ContinueWith(t => (IReadOnlyList<Order>)t.Result, ct);

    public Task AddAsync(Order order, CancellationToken ct = default)
        => _context.Orders.AddAsync(order, ct).AsTask();

    public void Remove(Order order) => _context.Orders.Remove(order);
}
```

**Expected Output:** `OrderRepository` provides domain‑specific queries (`ListForCustomerAsync`, `ListPendingAsync`) and hides EF Core behind the interface.

**Why this output?** The interface expresses **domain intent** and uses types that do not leak EF Core. Consumers call meaningful methods without needing to know about `DbContext`.

### Real-World Cases

- **DDD aggregates:** `IOrderRepository` with methods like `FindByIdAsync`, `AddAsync`, and `ListPendingAsync`.
- **Read models:** Separate `IOrderReadRepository` for optimized query‑side methods.
- **External system abstraction:** `IPaymentRepository` that abstracts both database and external payment API access.

### References

- Repository pattern interfaces (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- Interface segregation principle – https://en.wikipedia.org/wiki/Interface_segregation_principle

---

## 3. Repository Implementations

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A repository implementation is the concrete class that fulfils a repository interface, using a specific data‑access technology to retrieve and persist data. |
| **Technical Definition** | Implementations live in the **infrastructure layer** and use technologies such as EF Core (`DbContext`, `DbSet<T>`), Dapper (raw SQL), or external APIs. They translate interface methods into queries and commands, and typically **do not call `SaveChanges`**—that responsibility belongs to the Unit of Work at the application‑service layer. |
| **Beginner‑Friendly Explanation** | A repository implementation is the **“kitchen” that prepares the dishes**. The interface is the menu; the implementation does the actual work of retrieving data from the database, mapping it to objects, and saving changes. Different implementations can fulfil the same interface—one using EF Core, another using Dapper, another using an in‑memory store for testing. |

### Purposes

- **To** provide a concrete implementation of a repository interface using a specific data‑access technology.
- **To** translate domain‑oriented method calls into queries and commands.
- **To** keep EF Core (or Dapper, or APIs) **isolated** to the infrastructure layer.
- **To** support **swappable implementations** (e.g., fake repositories in tests).
- **To** encapsulate **query logic** (Include, filters, projections) in one place.

### Syntax Rules and Structure

#### General Syntax (EF Core Implementation)

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    public OrderRepository(AppDbContext context) => _context = context;

    public async Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
        => await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id, ct);

    public async Task AddAsync(Order order, CancellationToken ct = default)
        => await _context.Orders.AddAsync(order, ct);

    public void Remove(Order order) => _context.Orders.Remove(order);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `_context` | The EF Core `DbContext`—only the repository sees it. |
| `Include(...)` | Eager loading of related data. |
| `AddAsync` / `Remove` | Mark entity state; `SaveChanges` happens in the Unit of Work. |

**Syntax rules:**

- **Repositories should not call `SaveChanges`** – transaction boundaries belong to the Unit of Work / application service.
- **Use `AsNoTracking`** for read‑only queries to reduce overhead.
- **Use `CancellationToken`** for cancellation support.
- **Implementation lives in the infrastructure layer** – the interface lives in the domain/application layer.

**Constraints and limitations:**

- **Repository implementations can become anemic** if they merely wrap `DbSet<T>`—they add a layer without adding value.
- **Complex queries may require raw SQL** or Dapper, which is fine inside the implementation but should not leak to consumers.
- **Transactions across multiple repositories** require a Unit of Work that shares the `DbContext`.

### Annotated Complete Step-by-Step Code Example

```csharp
public class CustomerRepository : ICustomerRepository
{
    private readonly AppDbContext _context;
    public CustomerRepository(AppDbContext context) => _context = context;

    public async Task<Customer?> GetByIdAsync(int id, CancellationToken ct = default)
        => await _context.Customers
            .AsNoTracking()
            .FirstOrDefaultAsync(c => c.Id == id, ct);

    public async Task<Customer?> FindByEmailAsync(string email, CancellationToken ct = default)
        => await _context.Customers
            .AsNoTracking()
            .FirstOrDefaultAsync(c => c.Email == email, ct);

    public async Task<IReadOnlyList<Customer>> ListActiveAsync(CancellationToken ct = default)
        => await _context.Customers
            .AsNoTracking()
            .Where(c => c.IsActive)
            .ToListAsync(ct);

    public async Task AddAsync(Customer customer, CancellationToken ct = default)
        => await _context.Customers.AddAsync(customer, ct);

    public void Remove(Customer customer) => _context.Customers.Remove(customer);
}
```

**Step-by-step explanation:**

1. The repository injects `AppDbContext` via constructor.
2. Read queries use `AsNoTracking()` for efficiency.
3. `GetByIdAsync` and `FindByEmailAsync` return nullable entities.
4. `AddAsync` and `Remove` mark entity state—`SaveChanges` is called by the caller.

**Expected Output:** `CustomerRepository` fulfils `ICustomerRepository` using EF Core; consumers can query by ID, email, or active status without touching `DbContext`.

**Why this output?** The implementation encapsulates EF Core queries behind domain‑oriented interface methods.

### Real-World Cases

- **EF Core repositories:** Standard implementation for relational databases.
- **Dapper repositories:** For performance‑critical read queries that are hard to express in LINQ.
- **API repositories:** Abstract access to external services (e.g., payment gateways) behind the same interface pattern.
- **Fake repositories:** In‑memory implementations used in unit tests.

### References

- Repository pattern implementation (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- EF Core DbContext – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

---

## 4. Repository Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Repository best practices are guidelines for designing and using repositories that add genuine value, remain maintainable, and avoid the anti‑patterns of unnecessary abstraction. |
| **Technical Definition** | Best practices encompass **aggregate‑oriented design** (one repository per aggregate root), **avoiding `IQueryable` leakage**, **keeping `SaveChanges` in the Unit of Work**, **using Scoped lifetimes**, **specializing interfaces** for domain intent, **testing with fakes or real databases**, and **acknowledging when the repository pattern is not needed** (i.e., when EF Core alone suffices). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for repositories—they help you avoid adding a layer that makes things worse (leaking `IQueryable`, creating generic repositories that add no value) and help you use the pattern where it genuinely improves your code (testable business logic, domain‑oriented queries, swappable data sources). |

### Purposes

- **To** ensure repositories **add value** rather than being a thin, leaky wrapper around `DbContext`.
- **To** keep **transaction boundaries** in the Unit of Work, not scattered across repositories.
- **To** express **domain intent** through meaningful method names.
- **To** enable **testability** through interfaces and fake implementations.
- **To** avoid **leaking persistence details** (EF Core, `IQueryable`, `DbContext`) to consumers.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **When to use** | Use repositories when they provide **decoupling, testability, or domain intent**. | Add a repository layer that merely mirrors `DbSet<T>` with no added value. |
| **Granularity** | One repository per **aggregate root** (DDD). | One repository per **table**. |
| **Interface design** | Express **domain intent** (`FindByEmailAsync`). | Expose `IQueryable<T>`—it leaks EF Core and lazy execution. |
| **Transactions** | Let the **Unit of Work** (`DbContext`) own `SaveChanges` and transactions. | Call `SaveChanges` inside repositories—transactions span multiple repositories. |
| **Lifetime** | Register repositories as **Scoped** to match `DbContext`. | Register repositories as Singleton (captures `DbContext`—captive dependency). |
| **Return types** | Return `IReadOnlyList<T>`, `Task<T?>`, and domain types. | Return `List<T>` or `IQueryable<T>`—both leak implementation or mutability. |
| **Testing** | Use **fake repositories** for unit tests or **Testcontainers/SQLite in‑memory** for integration tests. | Test only against the In‑Memory EF Core provider. |
| **Async** | Use `Task`‑returning methods with `CancellationToken`. | Use synchronous data access in ASP.NET Core. |
| **Query encapsulation** | Encapsulate complex queries in specialized repositories. | Scatter LINQ queries across services and controllers. |
| **Evaluation** | Re‑evaluate whether the repository layer is still needed as the project evolves. | Keep the layer “just because” when EF Core alone would suffice. |

### Annotated Code Example (Best‑Practice Repository and Unit of Work)

```csharp
// Domain-oriented interface
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<IReadOnlyList<Order>> ListPendingAsync(CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    void Remove(Order order);
}

// Implementation — no SaveChanges inside
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    public OrderRepository(AppDbContext context) => _context = context;

    public Task<Order?> GetByIdAsync(int id, CancellationToken ct = default)
        => _context.Orders.Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id, ct);

    public Task<IReadOnlyList<Order>> ListPendingAsync(CancellationToken ct = default)
        => _context.Orders.AsNoTracking()
            .Where(o => o.Status == OrderStatus.Pending)
            .ToListAsync(ct)
            .ContinueWith(t => (IReadOnlyList<Order>)t.Result, ct);

    public Task AddAsync(Order order, CancellationToken ct = default)
        => _context.Orders.AddAsync(order, ct).AsTask();

    public void Remove(Order order) => _context.Orders.Remove(order);
}

// Unit of Work wraps SaveChanges
public interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Orders = new OrderRepository(context);
    }

    public IOrderRepository Orders { get; }

    public Task<int> SaveChangesAsync(CancellationToken ct = default)
        => _context.SaveChangesAsync(ct);
}

// Registration
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

**Why this follows best practices:**

- **Domain‑oriented interface** (`ListPendingAsync`) expresses intent rather than mirroring `DbSet<T>`.
- **No `SaveChanges` in the repository**—the Unit of Work owns the transaction boundary.
- **`AsNoTracking`** is used for read‑only queries.
- **`CancellationToken`** is propagated for graceful cancellation.
- **Scoped lifetime** matches the `DbContext`—no captive dependencies.
- **Return types** avoid leaking `IQueryable` or mutable collections.

### Real-World Cases

- **DDD applications:** Repositories abstract aggregate persistence; the Unit of Work coordinates transactions.
- **Testing‑heavy codebases:** Interfaces allow fake repositories in unit tests.
- **Multi‑source applications:** Different repository implementations for SQL, APIs, or caches behind the same interface.
- **Applications that do not need repositories:** Simple CRUD apps using EF Core directly—this is a legitimate and increasingly recommended choice.

### References

- Repository pattern and Unit of Work (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- Repository pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/repository.html
- EF Core DbContext lifetime and DI – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/#using-a-dbcontext-in-dependency-injection
- Testing EF Core applications – https://learn.microsoft.com/en-us/ef/core/testing/
- Is the repository pattern worth it? – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design

---

## Summary Table: Repository Pattern at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Repository pattern** | Abstract data access behind an interface | `IRepository<T>` / specialized interfaces | When decoupling, testability, or domain intent justifies the layer |
| **Repository interfaces** | Define the data‑access contract | `IOrderRepository`, `ICustomerRepository` | Always—this is what consumers depend on |
| **Repository implementations** | Fulfil the contract with EF Core, Dapper, etc. | `OrderRepository : IOrderRepository` | In the infrastructure layer |
| **Best practices** | Avoid leaks and anti‑patterns | No `IQueryable` leakage, Scoped lifetime, Unit of Work for `SaveChanges` | Every repository implementation |

---

## References (Full List)

- Repository pattern (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- Repository pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/repository.html
- EF Core DbContext configuration and DI – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- Testing EF Core applications – https://learn.microsoft.com/en-us/ef/core/testing/
- Interface segregation principle – https://en.wikipedia.org/wiki/Interface_segregation_principle
- Domain-Driven Design aggregates – https://martinfowler.com/bliki/DDD_Aggregate.html
- Unit of Work pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/unitOfWork.html