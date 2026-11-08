# Unit of Work Pattern in ASP.NET Core: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) with **Entity Framework Core** as the primary persistence technology. A critical framing note: **EF Core’s `DbContext` already implements the Unit of Work pattern**—it tracks changes, batches `SaveChanges`, and manages transactions. Wrapping it in a custom `IUnitOfWork` class is **optional** and must add value (e.g., coordinating multiple repositories, abstracting persistence, or centralizing transaction boundaries). The pattern is genuinely valuable in **Domain‑Driven Design** and **multi‑repository transaction** scenarios, but it is **not a universal requirement**—the official EF Core guidance cautions against layering abstractions without justification. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Unit of Work pattern maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems. |
| **Technical Definition** | The Unit of Work pattern tracks changes to domain entities, groups them into a single logical transaction, and persists them atomically through a **single commit** (`SaveChanges`). In EF Core, `DbContext` is the Unit of Work implementation—it tracks entities via the change tracker, groups changes into one transaction, and writes them on `SaveChanges`. Custom `IUnitOfWork` wrappers coordinate multiple repositories that share the same `DbContext`, ensuring that changes across aggregates are committed atomically. |
| **Beginner‑Friendly Explanation** | The Unit of Work pattern is the **“shopping cart for database changes.”** You add items to the cart (track changes to entities), and when you are done, you check out (call `SaveChanges`)—either all items are purchased or none are. If something goes wrong, the whole transaction is rolled back. It ensures that related changes are committed together, not piecemeal. |

### Key Characteristics

- **Transactional boundary** – Groups multiple operations into one atomic commit.
- **Change tracking** – Tracks entities that have been created, modified, or deleted.
- **Concurrency resolution** – Detects conflicting updates via concurrency tokens.
- **Repository coordination** – Provides a single `SaveChanges` across multiple repositories that share the same context.
- **Implicit in EF Core** – `DbContext` already implements the pattern; a custom wrapper is optional.
- **Scoped lifetime** – A Unit of Work typically lives for one request or one business operation.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **EF Core** (`DbContext`, `DbSet`, change tracking, transactions).
- Familiarity with **dependency injection** and **service lifetimes**.
- Understanding of the **repository pattern** (often used together with Unit of Work).

### Related Programming Areas

- **EF Core `DbContext`** – The default Unit of Work implementation.
- **Repository pattern** – Often paired with Unit of Work to abstract persistence.
- **Transactions** – Explicit transactions for multi-step operations.
- **Concurrency tokens** – Optimistic concurrency checks during `SaveChanges`.
- **Domain‑Driven Design** – Aggregates and repository/Unit of Work coordination.

### Core Concepts / Features

1. **Unit of Work** – The concept, its relationship to `DbContext`, and transactional semantics.
2. **Unit of Work pattern** – The formal pattern, custom `IUnitOfWork` wrappers, and repository coordination.
3. **Unit of Work best practices** – Lifetime management, transaction boundaries, and avoiding anti‑patterns.

---

## 1. Unit of Work (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Unit of Work is a logical container for a set of operations that must be committed atomically as part of a single business transaction. |
| **Technical Definition** | A Unit of Work tracks changes to entities, groups them into a single transaction, and persists them via a single commit. In EF Core, `DbContext` provides this behaviour: the change tracker records entity state (`Added`, `Modified`, `Deleted`, `Unchanged`), and `SaveChanges` generates and executes the corresponding SQL statements inside a transaction. The Unit of Work boundary determines what “all or nothing” means for a business operation. |
| **Beginner‑Friendly Explanation** | A Unit of Work is the **“transaction scope” for your business operation**. If you create an order and decrement inventory, those two changes must succeed or fail together. The Unit of Work groups them, and one commit (or rollback) applies to both. Without a Unit of Work, you might insert the order but fail to decrement inventory—leaving your data inconsistent. |

### Purposes

- **To** group multiple related operations into a **single atomic transaction**.
- **To** ensure **data consistency**—either all changes are committed or none are.
- **To** **track changes** so that only modified entities generate `UPDATE`/`DELETE` statements.
- **To** **resolve concurrency conflicts** via concurrency tokens during commit.
- **To** provide a **clear transaction boundary** in the application architecture.
- **To** **coordinate multiple repositories** that share the same persistence context.

### Syntax Rules and Structure

#### General Architecture

```text
Application Service
   │  1. Begins Unit of Work
   │  2. Calls repositories (adds/modifies/deletes entities)
   │  3. Commits Unit of Work (SaveChanges)
   ▼
Unit of Work (DbContext)
   │  Tracks changes, begins transaction, executes SQL
   ▼
Database
```

**Component breakdown:**

| Component | Responsibility |
|-----------|----------------|
| **Application service** | Defines the business operation and its transaction boundary. |
| **Unit of Work** (`DbContext`) | Tracks changes and coordinates the commit. |
| **Repositories** | Add/modify/remove entities—no `SaveChanges`. |
| **Database** | Receives the atomic set of statements. |

#### General Syntax (EF Core Implicit Unit of Work)

```csharp
public async Task<Order> PlaceOrderAsync(CreateOrderCommand command)
{
    // 1. Start the unit of work (the DbContext instance)
    var order = new Order { CustomerId = command.CustomerId, Total = command.Total };
    _context.Orders.Add(order);   // Tracked as Added

    var inventory = await _context.Inventory
        .FirstOrDefaultAsync(i => i.ProductId == command.ProductId);
    inventory!.Quantity -= command.Quantity;   // Tracked as Modified

    // 2. Commit the unit of work — all changes in one transaction
    await _context.SaveChangesAsync();

    return order;
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `_context.Orders.Add(order)` | Marks the order as `Added`. |
| `inventory.Quantity -= ...` | Modifies a tracked entity (state becomes `Modified`). |
| `_context.SaveChangesAsync()` | Generates and executes `INSERT` and `UPDATE` in one transaction. |

**Syntax rules:**

- **`SaveChanges` is atomic by default**—all pending changes are committed or rolled back together.
- **Change tracking determines which SQL is generated**—`Unchanged` entities are not written.
- **Explicit transactions** are needed only when multiple `SaveChanges` calls must be atomic or when non‑EF operations are involved.

**Constraints and limitations:**

- **`SaveChanges` is one transaction, but not a long‑running one**—it does not include time spent between loading entities and calling `SaveChanges`.
- **`SaveChanges` is synchronous in nature** unless `SaveChangesAsync` is used—always use the async version in ASP.NET Core.
- **Tracked entities accumulate in the context**—long‑lived contexts consume memory.

### Annotated Complete Step-by-Step Code Example

```csharp
public class OrderService
{
    private readonly AppDbContext _context;

    public OrderService(AppDbContext context) => _context = context;

    public async Task<Order> PlaceOrderAsync(CreateOrderCommand command)
    {
        // Step 1: Begin the logical unit of work (tracked entities)
        var order = new Order
        {
            CustomerId = command.CustomerId,
            Total = command.Total
        };
        _context.Orders.Add(order);

        // Step 2: Add related entities
        foreach (var item in command.Items)
        {
            _context.OrderItems.Add(new OrderItem
            {
                Order = order,
                ProductId = item.ProductId,
                Quantity = item.Quantity
            });
        }

        // Step 3: Modify related entities
        var inventory = await _context.Inventory
            .Where(i => i.ProductId == command.Items[0].ProductId)
            .FirstOrDefaultAsync();

        if (inventory is null || inventory.Quantity < command.Items[0].Quantity)
            throw new InvalidOperationException("Insufficient stock.");

        inventory.Quantity -= command.Items[0].Quantity;

        // Step 4: Commit the unit of work atomically
        await _context.SaveChangesAsync();

        return order;
    }
}
```

**Step-by-step explanation:**

1. The `Order` is added to the context (tracked as `Added`).
2. Each `OrderItem` is added (tracked as `Added`).
3. The inventory entity is loaded and modified (tracked as `Modified`).
4. `SaveChangesAsync` generates and executes all statements inside a single transaction.

**Expected Output:** Either the order, its items, and the inventory decrement are all persisted, or—if any step fails—none are.

**Why this output?** `SaveChangesAsync` wraps all pending changes in a single database transaction. A failure (e.g., a constraint violation) causes the entire transaction to roll back.

### Real-World Cases

- **E‑commerce checkout:** Order + order items + inventory decrement + payment record must commit atomically.
- **Banking transfers:** Debit one account and credit another in a single transaction.
- **Multi‑entity workflows:** Creating a user and assigning a role in one operation.
- **Audit logging:** Writing the business change and its audit entry atomically.

### References

- Unit of Work pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/unitOfWork.html
- DbContext as Unit of Work – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- Saving changes and transactions – https://learn.microsoft.com/en-us/ef/core/saving/transactions

---

## 2. Unit of Work Pattern (The Formal Pattern)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The Unit of Work pattern is a formal design pattern that defines a transactional boundary, tracks changes, and coordinates persistence across repositories. |
| **Technical Definition** | The pattern is defined by a `IUnitOfWork` interface that exposes repositories and a `SaveChanges`/`Commit` method. The implementation holds a shared `DbContext` (or equivalent), constructs repositories that share that context, and exposes a single commit operation. This ensures that changes across multiple aggregates/repositories are committed atomically. In DDD, the Unit of Work often corresponds to a single business operation. |
| **Beginner‑Friendly Explanation** | The Unit of Work **pattern** is the formal version of the concept: a class that gathers repositories and offers one `SaveChanges` call. Instead of calling `SaveChanges` on each repository (which would create separate transactions), you call it once on the Unit of Work—and every repository’s changes are committed together. It is the **orchestrator of the transaction**. |

### Purposes

- **To** provide a **single commit point** for changes across multiple repositories.
- **To** ensure that changes to **multiple aggregates** are committed atomically.
- **To** hide the **persistence context** (`DbContext`) from application services and repositories.
- **To** centralize **transaction management** in one class.
- **To** support **DDD** scenarios where multiple repositories participate in one business transaction.
- **To** enable **unit testing** by allowing a fake Unit of Work to be substituted.

### Syntax Rules and Structure

#### General Syntax (IUnitOfWork Interface)

```csharp
public interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    ICustomerRepository Customers { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}
```

#### General Syntax (Implementation)

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Orders = new OrderRepository(context);
        Customers = new CustomerRepository(context);
    }

    public IOrderRepository Orders { get; }
    public ICustomerRepository Customers { get; }

    public Task<int> SaveChangesAsync(CancellationToken ct = default)
        => _context.SaveChangesAsync(ct);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Repository properties | Expose repositories that share the same `DbContext`. |
| `SaveChangesAsync` | Single commit point. |
| `AppDbContext` | The underlying unit of work. |

**Syntax rules:**

- **All repositories must share the same `DbContext` instance**—otherwise, they do not participate in the same transaction.
- **The `DbContext` should be created per Unit of Work**, not injected as a separate lifetime from the Unit of Work itself.
- **`SaveChanges` should only be called by the Unit of Work**, not by repositories.
- **Register `IUnitOfWork` and `DbContext` as Scoped**—one per request.

**Constraints and limitations:**

- **A custom Unit of Work over EF Core is often redundant**—`DbContext` already provides the pattern.
- **Repository coordination adds complexity**—justify it with genuine multi‑repository transaction scenarios.
- **The Unit of Work must not be captured in Singletons**—it holds a `DbContext` and is Scoped.

### Annotated Complete Step-by-Step Code Example

```csharp
// 1. Unit of Work interface
public interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }
    Task<int> SaveChangesAsync(CancellationToken ct = default);
}

// 2. Implementation — repositories share the same DbContext
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Orders = new OrderRepository(context);
        Inventory = new InventoryRepository(context);
    }

    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }

    public Task<int> SaveChangesAsync(CancellationToken ct = default)
        => _context.SaveChangesAsync(ct);
}

// 3. Repositories — no SaveChanges
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    public OrderRepository(AppDbContext context) => _context = context;

    public async Task AddAsync(Order order, CancellationToken ct = default)
        => await _context.Orders.AddAsync(order, ct);
}

public class InventoryRepository : IInventoryRepository
{
    private readonly AppDbContext _context;
    public InventoryRepository(AppDbContext context) => _context = context;

    public async Task<Inventory?> GetAsync(int productId, CancellationToken ct = default)
        => await _context.Inventory.FirstOrDefaultAsync(i => i.ProductId == productId, ct);
}

// 4. Application service — one commit for both repositories
public class OrderService
{
    private readonly IUnitOfWork _uow;

    public OrderService(IUnitOfWork uow) => _uow = uow;

    public async Task<Order> PlaceOrderAsync(CreateOrderCommand command)
    {
        var order = new Order { CustomerId = command.CustomerId, Total = command.Total };
        await _uow.Orders.AddAsync(order);

        var inventory = await _uow.Inventory.GetAsync(command.ProductId);
        if (inventory is null || inventory.Quantity < command.Quantity)
            throw new InvalidOperationException("Insufficient stock.");

        inventory.Quantity -= command.Quantity;

        await _uow.SaveChangesAsync();   // Both repositories commit atomically
        return order;
    }
}

// 5. Registration
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IInventoryRepository, InventoryRepository>();
```

**Step-by-step explanation:**

1. `IUnitOfWork` exposes repositories and a single `SaveChangesAsync`.
2. `UnitOfWork` constructs both repositories with the **same** `AppDbContext` instance.
3. Repositories mark entities as `Added`/`Modified` but **do not call `SaveChanges`**.
4. `OrderService` uses both repositories and calls `_uow.SaveChangesAsync()` once.
5. Both changes (order insert + inventory update) commit atomically.

**Expected Output:** The order and inventory change are committed in one transaction. If the inventory update fails, the order insert is rolled back.

**Why this output?** Both repositories share the same `DbContext`, and the single `SaveChangesAsync` call wraps all changes in one transaction.

### Real-World Cases

- **DDD applications with multiple aggregates:** Order + Customer + Inventory coordinated in one business operation.
- **Multi‑repository transactions:** Payment record + order status + audit log.
- **Testing‑focused architectures:** Fake Unit of Work and repositories allow testing services without a database.
- **Legacy migrations:** Introducing a Unit of Work facade over an existing `DbContext` to enable gradual refactoring.

### References

- Unit of Work pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/unitOfWork.html
- Repository and Unit of Work patterns (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- DbContext as Unit of Work – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/

---

## 3. Unit of Work Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Unit of Work best practices are guidelines for implementing and using the pattern so that it is correct, maintainable, and genuinely beneficial. |
| **Technical Definition** | Best practices encompass **Scoped lifetime**, **shared `DbContext` across repositories**, **single `SaveChanges` per business operation**, **explicit transactions only when needed**, **avoiding repository `SaveChanges`**, **handling concurrency tokens**, **testing with fakes or real databases**, and **recognizing when EF Core’s built‑in Unit of Work suffices**. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for Unit of Work—they help you avoid inconsistent commits, captive dependencies, and unnecessary complexity. |

### Purposes

- **To** ensure changes are committed **atomically** and consistently.
- **To** keep transaction boundaries **clear and correct**.
- **To** avoid **lifetime mismatches** (captive dependencies).
- **To** prevent **repositories from committing** independently.
- **To** enable **testability** and **maintainability**.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Lifetime** | Register `IUnitOfWork` and `DbContext` as **Scoped**. | Register as Singleton (captures `DbContext`—captive dependency). |
| **Repository sharing** | All repositories in a Unit of Work must share the **same `DbContext`**. | Create repositories with separate contexts—they cannot commit together. |
| **Commit** | Call `SaveChanges` **once** at the end of the business operation. | Call `SaveChanges` inside repositories or multiple times mid‑operation. |
| **Explicit transactions** | Use `Database.BeginTransactionAsync` only when spanning multiple `SaveChanges` or non‑EF operations. | Wrap every operation in a transaction—`SaveChanges` is already transactional. |
| **Unit of Work scope** | Keep the Unit of Work **short‑lived** (one request/operation). | Hold it across multiple HTTP requests or long‑running tasks. |
| **Concurrency** | Configure concurrency tokens (`[Timestamp]`, `[ConcurrencyCheck]`) and handle `DbUpdateConcurrencyException`. | Ignore concurrency conflicts—they cause silent data loss. |
| **Repository `SaveChanges`** | Let the Unit of Work own `SaveChanges`. | Let repositories call `SaveChanges`—it breaks the atomic boundary. |
| **When to wrap** | Use a custom Unit of Work when **multiple repositories must commit together** or when abstracting persistence. | Wrap `DbContext` “just because”—EF Core already implements the pattern. |
| **Testing** | Test with **fake Unit of Work/repositories** for unit tests and **SQLite in‑memory or Testcontainers** for integration tests. | Test only against the EF Core In‑Memory provider. |
| **Exception handling** | Let exceptions propagate after `SaveChanges` fails; consider retry strategies for transient failures. | Swallow exceptions—partial state may persist without rollback. |

### Annotated Code Example (Best‑Practice Setup with Concurrency)

```csharp
// 1. Unit of Work with Scoped lifetime
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// 2. Transaction boundary in the service
public class OrderService
{
    private readonly IUnitOfWork _uow;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IUnitOfWork uow, ILogger<OrderService> logger)
    {
        _uow = uow;
        _logger = logger;
    }

    public async Task<Order> PlaceOrderAsync(CreateOrderCommand command)
    {
        var order = new Order { CustomerId = command.CustomerId, Total = command.Total };
        await _uow.Orders.AddAsync(order);

        var inventory = await _uow.Inventory.GetAsync(command.ProductId);
        if (inventory is null || inventory.Quantity < command.Quantity)
            throw new InvalidOperationException("Insufficient stock.");

        inventory.Quantity -= command.Quantity;

        try
        {
            await _uow.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            _logger.LogWarning(ex, "Concurrency conflict for product {ProductId}", command.ProductId);
            throw;
        }

        return order;
    }
}

// 3. Explicit transaction when multiple SaveChanges must be atomic
public async Task ProcessBatchAsync(IEnumerable<CreateOrderCommand> commands)
{
    await using var tx = await _context.Database.BeginTransactionAsync();
    try
    {
        foreach (var command in commands)
        {
            await PlaceOrderAsync(command);   // Each call invokes SaveChanges
        }
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

- **`IUnitOfWork` is Scoped**, matching the `DbContext` lifetime.
- **One `SaveChanges` per business operation**—the order and inventory change commit atomically.
- **Concurrency tokens** are handled via `DbUpdateConcurrencyException`.
- **Explicit transaction** is used only when multiple `SaveChanges` calls must be atomic.
- **Repositories do not call `SaveChanges`**—the Unit of Work owns the transaction boundary.
- **Logging** captures concurrency conflicts without swallowing them.

### Real-World Cases

- **Multi‑aggregate workflows:** Order + inventory + audit log committed atomically.
- **Batch processing:** Multiple orders processed inside an explicit transaction.
- **DDD applications:** Repositories coordinated by a Unit of Work per business operation.
- **Applications without a custom Unit of Work:** EF Core’s `DbContext` alone suffices for many CRUD apps—this is a legitimate and recommended choice when the pattern adds no value.

### References

- Repository and Unit of Work patterns (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- DbContext lifetime and DI – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/#using-a-dbcontext-in-dependency-injection
- Transactions in EF Core – https://learn.microsoft.com/en-us/ef/core/saving/transactions
- Concurrency conflicts – https://learn.microsoft.com/en-us/ef/core/saving/concurrency

---

## Summary Table: Unit of Work at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Unit of Work** | Groups changes into one atomic transaction | `DbContext.SaveChanges`, `SaveChangesAsync` | Every business operation that modifies data |
| **Unit of Work pattern** | Formal wrapper coordinating multiple repositories | `IUnitOfWork`, shared `DbContext`, single `SaveChanges` | When multiple repositories/aggregates must commit together |
| **Best practices** | Correct lifetime, single commit, concurrency handling | Scoped lifetime, no repository `SaveChanges`, explicit transactions when needed | Every Unit of Work implementation |

---

## References (Full List)

- Unit of Work pattern (Martin Fowler) – https://martinfowler.com/eaaCatalog/unitOfWork.html
- Repository and Unit of Work patterns (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design
- DbContext configuration and DI – https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/
- Transactions in EF Core – https://learn.microsoft.com/en-us/ef/core/saving/transactions
- Concurrency conflicts in EF Core – https://learn.microsoft.com/en-us/ef/core/saving/concurrency
- Testing EF Core applications – https://learn.microsoft.com/en-us/ef/core/testing/
- Domain‑Driven Design aggregates – https://martinfowler.com/bliki/DDD_Aggregate.html
- Saving changes in EF Core – https://learn.microsoft.com/en-us/ef/core/saving/