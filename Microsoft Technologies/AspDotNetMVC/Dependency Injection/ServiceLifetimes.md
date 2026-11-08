# ASP.NET Core Service Lifetimes: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the built-in DI container (`Microsoft.Extensions.DependencyInjection`). The three lifetimes—**Transient**, **Scoped**, and **Singleton**—are stable across these versions. **Scope validation** behaviour (throwing on captive dependencies) is enabled by default in the Development environment and is flagged where relevant. The guidance applies equally to MVC controllers, Razor Pages, Minimal APIs, and Worker Services. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A service lifetime defines **how long a service instance lives** and **how often a new instance is created** when the service is requested from the DI container. |
| **Technical Definition** | ASP.NET Core supports three lifetimes: **Transient** (created each time it is requested), **Scoped** (created once per scope, typically per HTTP request), and **Singleton** (created once for the application lifetime). The lifetime is configured at registration via `AddTransient`, `AddScoped`, or `AddSingleton`. The container enforces the lifetime: Transient and Scoped services are disposed at the end of the request/scope; Singleton services are disposed at application shutdown. Mismatched lifetimes (e.g., a Singleton depending on a Scoped service) produce a **captive dependency**, which is detected at startup in Development. |
| **Beginner‑Friendly Explanation** | Service lifetimes answer the question: **“How often should a new instance be created?”** A **Transient** service is like a disposable cup—a new one every time. A **Scoped** service is like a shared office mug—one per person (per request). A **Singleton** service is like the office water cooler—one for everyone, all day. Choosing the right lifetime prevents bugs, memory leaks, and state leakage between requests. |

### Key Characteristics

- **Three lifetimes** – Transient, Scoped, and Singleton cover the vast majority of scenarios.
- **Lifetime is per registration** – The same class can be registered with different lifetimes under different service types.
- **Disposal is automatic** – The container disposes `IDisposable`/`IAsyncDisposable` services according to their lifetime.
- **Captive dependencies are the primary pitfall** – A longer-lived service must never capture a shorter-lived one.
- **Scope validation** – In Development, the container throws when it detects a captive dependency.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working understanding of **dependency injection** and **service registration**.
- Familiarity with `Program.cs` (minimal hosting) and constructor injection.

### Related Programming Areas

- **Dependency injection** – The pattern that lifetimes implement.
- **Service registration** – `AddTransient`, `AddScoped`, `AddSingleton`.
- **EF Core** – `DbContext` is registered as **Scoped** by default.
- **Background services** – Must use `IServiceScopeFactory` to resolve Scoped services safely.
- **Middleware** – Scoped services are resolved in `InvokeAsync`, not the constructor.

### Core Concepts / Features

1. **Transient** – New instance per resolution.
2. **Scoped** – One instance per request/scope.
3. **Singleton** – One instance for the application lifetime.
4. **Lifetime selection** – Heuristics for choosing the correct lifetime.
5. **Lifetime best practices** – Avoiding captive dependencies and state leaks.

---

## 1. Transient

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Transient service is created **every time** it is requested from the container. |
| **Technical Definition** | `AddTransient` registers a service with `ServiceLifetime.Transient`. Each call to `GetService<T>()` or each constructor injection produces a **new instance**. In request-processing apps, transient services are disposed at the end of the request. This lifetime incurs per-request allocations because services are resolved and constructed every time. |
| **Beginner‑Friendly Explanation** | Transient is the **“disposable cup”** lifetime. Every time someone asks for the service, a brand-new instance is handed out. Nothing is shared, and nothing is reused. |

### Purposes

- **To** ensure that no state is shared between consumers.
- **To** provide lightweight, stateless services that are cheap to create.
- **To** avoid concurrency issues that can arise from shared state.
- **To** guarantee that each consumer gets a fresh instance.

### Syntax Rules and Structure

```csharp
services.AddTransient<IMyService, MyService>();
services.AddTransient<IMyService>(sp => new MyService(sp.GetRequiredService<IDep>()));
services.AddTransient<MyService>(); // Self-registration
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddTransient<TService, TImpl>()` | Type mapping with Transient lifetime. |
| Factory overload | Creates a new instance per resolution. |
| `AddTransient<T>()` | Self-registration (service type = implementation type). |

**Syntax rules:**

- Transient services can depend on Scoped and Singleton services.
- Transient `IDisposable` services are tracked by the container until the end of the scope.

**Constraints and limitations:**

- **High allocation cost** if resolved frequently in hot paths.
- **Transient `IDisposable`** services can cause memory pressure if many are resolved within one scope.

### Annotated Code Example

```csharp
public interface ILightweightService { Guid InstanceId { get; } }

public class LightweightService : ILightweightService
{
    public Guid InstanceId { get; } = Guid.NewGuid();
}

builder.Services.AddTransient<ILightweightService, LightweightService>();

public class Consumer
{
    public Consumer(ILightweightService a, ILightweightService b)
    {
        // a.InstanceId != b.InstanceId
    }
}
```

**Expected Output:** `Consumer` receives **two different** `LightweightService` instances, even within the same constructor.

**Why this output?** Transient resolution creates a new instance for every request, including multiple requests within the same constructor.

### Real-World Cases

- **Email senders:** A stateless `IEmailSender` that opens a new SMTP connection per send.
- **Validators:** Lightweight validation services that hold no state.
- **Factories:** Services that create short-lived objects.

### References

- Service lifetimes (Transient) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#transient

---

## 2. Scoped

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Scoped service is created **once per scope** and reused within that scope. |
| **Technical Definition** | `AddScoped` registers a service with `ServiceLifetime.Scoped`. In ASP.NET Core web apps, a scope is created **per HTTP request**, so a Scoped service is created once per request and shared by all consumers within that request. Scoped services are disposed at the end of the request. A Scoped service must always be resolved from a scope—either the implicit per-request scope or an explicit scope created with `IServiceScopeFactory.CreateScope()`. |
| **Beginner‑Friendly Explanation** | Scoped is the **“per-request”** lifetime. For each HTTP request, one instance is created and shared by everything that needs it during that request. The next request gets a fresh instance. |

### Purposes

- **To** share state within a single request (e.g., a unit-of-work or transaction).
- **To** avoid creating expensive per-request objects multiple times.
- **To** align with the lifetime of a database context (`DbContext`), which is not thread-safe and should not be shared across requests.
- **To** ensure per-request isolation and disposal.

### Syntax Rules and Structure

```csharp
services.AddScoped<IRepository, Repository>();
services.AddScoped<DbContext>(sp => new DbContext(...));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddScoped<TService, TImpl>()` | Type mapping with Scoped lifetime. |
| Scope | Created automatically per HTTP request or explicitly via `CreateScope()`. |

**Syntax rules:**

- Must be resolved from a scope; resolving from the root provider throws in Development.
- A Scoped service can depend on Singleton and other Scoped services.
- A Singleton **must not** depend on a Scoped service (captive dependency).

**Constraints and limitations:**

- **Captive dependencies** – If a Singleton captures a Scoped service, the Scoped service behaves like a Singleton, causing state leakage across requests.
- **Background tasks** – Must create an explicit scope with `IServiceScopeFactory` to resolve Scoped services.

### Annotated Code Example

```csharp
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;
    public OrderRepository(AppDbContext db) => _db = db;
}

builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

public class OrdersController : Controller
{
    private readonly IOrderRepository _repo;
    public OrdersController(IOrderRepository repo) => _repo = repo;
}
```

**Expected Output:** Within a single HTTP request, the same `OrderRepository` (and the same `AppDbContext`) instance is shared across all consumers. A new request gets a new instance.

**Why this output?** The per-request scope caches Scoped instances and disposes them at the end of the request.

### Real-World Cases

- **`DbContext`:** EF Core’s `DbContext` is Scoped by default because web app database operations are normally scoped to the client request.
- **Unit-of-work:** A Scoped service that tracks changes across multiple repositories within one request.
- **Per-request caching:** A Scoped cache that avoids repeated database queries within one request.

### References

- Service lifetimes (Scoped) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped

---

## 3. Singleton

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Singleton service is created **once** and shared for the entire application lifetime. |
| **Technical Definition** | `AddSingleton` registers a service with `ServiceLifetime.Singleton`. The instance is created the first time it is requested (or by the developer when providing an implementation instance directly). Every subsequent request uses the same instance. Singleton services are disposed when the service container is disposed, usually at application shutdown. Singletons **must not** depend on Scoped services. |
| **Beginner‑Friendly Explanation** | Singleton is the **“one for everyone, forever”** lifetime. The first time the service is needed, it is created. After that, every class in every request gets the **same** instance. It lives until the app shuts down. |

### Purposes

- **To** share application-wide state or configuration.
- **To** avoid the cost of repeatedly creating expensive resources (e.g., caches, connection pools).
- **To** provide a single coordination point (e.g., a background task queue).
- **To** implement the **Singleton design pattern** within the DI container.

### Syntax Rules and Structure

```csharp
services.AddSingleton<IClock, SystemClock>();
services.AddSingleton<ICache>(new InMemoryCache());
services.AddSingleton<IMyService>(sp => new MyService(sp.GetRequiredService<IConfig>()));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddSingleton<TService, TImpl>()` | Type-based registration. |
| `AddSingleton<TService>(instance)` | Registers an existing instance. |
| Factory overload | Creates the instance once via a factory. |

**Syntax rules:**

- Can depend on other Singletons and Transients (but **not** Scoped).
- Must be **thread-safe** if used concurrently.
- Created lazily by default; `ValidateOnBuild` can force creation at startup.

**Constraints and limitations:**

- **Captive dependencies** – A Singleton must not capture a Scoped service.
- **Thread safety** – Singletons are shared across threads; mutable state must be synchronised.
- **No per-request state** – Singletons cannot hold request-specific data.

### Annotated Code Example

```csharp
public class SystemClock : IClock
{
    public DateTime Now => DateTime.UtcNow;
}

builder.Services.AddSingleton<IClock, SystemClock>();

public class HomeController : Controller
{
    private readonly IClock _clock;
    public HomeController(IClock clock) => _clock = clock;
}
```

**Expected Output:** Every request uses the **same** `SystemClock` instance. If the class held mutable state, that state would be shared across all requests.

**Why this output?** The Singleton instance is created once and reused for the application’s lifetime.

### Real-World Cases

- **Clocks:** `IClock` / `IDateTime` (stateless, thread-safe).
- **Caches:** In-memory caches shared across requests.
- **Configuration wrappers:** `IOptions<T>` (registered as Singleton by default).
- **Background task queues:** A Singleton queue that coordinates work across requests.

### References

- Service lifetimes (Singleton) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#singleton

---

## 4. Lifetime Selection

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Lifetime selection is the process of choosing the correct lifetime (Transient, Scoped, or Singleton) for each registered service based on its state, thread-safety, and disposal requirements. |
| **Technical Definition** | Lifetime selection is guided by the service’s intended state scope: **Singleton** for application-wide shared state (thread-safe), **Scoped** for per-request state (e.g., `DbContext`, unit-of-work), and **Transient** for stateless, lightweight services. A longer-lived service must never depend on a shorter-lived one (captive dependency). EF Core’s `DbContext` is Scoped by default; services of a given lifetime should not use a `DbContext` with a shorter lifetime. |
| **Beginner‑Friendly Explanation** | Lifetime selection is **matching the service’s “lifespan” to its job**. If it holds data that everyone should share, make it Singleton. If it holds data that belongs to one request, make it Scoped. If it holds no data and is cheap to create, make it Transient. |

### Purposes

- **To** prevent state leakage and concurrency bugs caused by mismatched lifetimes.
- **To** optimise resource usage (avoid creating expensive objects unnecessarily).
- **To** ensure correct disposal timing (per-request, per-scope, or at shutdown).
- **To** avoid captive dependencies that turn per-request services into de facto Singletons.

### Syntax Rules and Structure (Decision Heuristic)

| Question | Recommended Lifetime |
|----------|----------------------|
| Does the service hold **application-wide state** that must be shared across all requests (cache, configuration, connection pool)? | **Singleton** (ensure thread safety). |
| Does the service hold **per-request state** (e.g., `DbContext`, unit-of-work, request-specific cache)? | **Scoped**. |
| Is the service **stateless and lightweight**, with no shared state and no disposal concerns? | **Transient**. |
| Does the service depend on a **Scoped** service (`DbContext`)? | The consumer must be **Scoped** or **Transient**—never Singleton. |
| Is the service used in a **background task** outside a request? | Create an **explicit scope** with `IServiceScopeFactory` to resolve Scoped services. |

**Constraints and limitations:**

- **Captive dependencies** – A Singleton must not depend on a Scoped service. The container detects this in Development and throws.
- **Thread safety** – Singletons are shared across threads; mutable state must be synchronised.
- **`DbContext` is Scoped** – Services of a given lifetime should not use a `DbContext` with a shorter lifetime.

### Annotated Code Example (Correct Lifetime Selection)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Singleton: stateless, application-wide
builder.Services.AddSingleton<IClock, SystemClock>();

// Scoped: per-request state (EF Core)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Transient: stateless, lightweight
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

builder.Services.AddControllersWithViews();
var app = builder.Build();
```

**Expected Output:** The clock is shared across all requests; each request gets its own `DbContext` and repository; each resolution of `IEmailSender` gets a fresh instance.

**Why this output?** Lifetimes are matched to state scope: Singleton for the stateless clock, Scoped for the per-request database context, Transient for the stateless email sender.

### Real-World Cases

- **EF Core `DbContext`:** Always Scoped because database operations are scoped to the client request.
- **Caching:** A memory cache is Singleton; a request-scoped cache is Scoped.
- **Authentication handlers:** Often Scoped because they depend on per-request user context.

### References

- Service lifetimes (lifetime selection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection in ASP.NET Core (EF Core scoped lifetime) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

---

## 5. Lifetime Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Lifetime best practices are guidelines for choosing, configuring, and consuming services so that lifetimes are correct, disposal is automatic, and captive dependencies are avoided. |
| **Technical Definition** | Best practices encompass avoiding stateful static classes, avoiding direct instantiation of dependencies, keeping services small and well-factored, letting the container dispose services, ensuring thread safety for Singletons, and using explicit scopes (`IServiceScopeFactory`) when resolving Scoped services outside a request. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for service lifetimes—they help you avoid subtle bugs like a Singleton accidentally holding a per-request database context, or a service that is impossible to test because it creates its own dependencies. |

### Purposes

- **To** prevent captive dependencies and state-leakage bugs.
- **To** ensure services are correctly disposed by the container.
- **To** keep services small, focused, and testable.
- **To** avoid anti-patterns such as service locator and direct instantiation.
- **To** choose lifetimes that match the intended scope of state.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **State and statics** | Avoid stateful, static classes and members; use Singleton services for global state instead. | Create global state through static mutable fields. |
| **Direct instantiation** | Let the container create dependencies. | Use `new` to create dependent classes inside services—this couples code to a specific implementation. |
| **Service size** | Make services small, well-factored, and easily tested. | Create “god” services with many injected dependencies; this violates the Single Responsibility Principle. |
| **Disposal** | Let the container dispose `IDisposable`/`IAsyncDisposable` services based on lifetime. | Manually dispose services resolved from the container. |
| **Lifetime selection** | Match lifetime to state scope: Scoped for per-request state, Singleton for application state, Transient for stateless services. | Capture a Scoped service in a Singleton (captive dependency). |
| **Scoped in Singletons** | Use `IServiceScopeFactory` to create an explicit scope inside a Singleton when a Scoped service is truly needed. | Resolve a Scoped service directly from a Singleton’s constructor. |
| **Thread safety** | Ensure Singleton services are thread-safe if used concurrently. | Assume Singleton services are only accessed by one thread. |
| **Scope validation** | Keep `ValidateScopes` enabled in Development to catch captive dependencies early. | Disable scope validation and risk subtle state bugs. |

### Annotated Code Example (Best-Practice Lifetime Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Singleton: thread-safe, application-wide state
builder.Services.AddSingleton<IClock, SystemClock>();

// Scoped: per-request state
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Transient: stateless, lightweight
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

builder.Services.AddControllersWithViews();
var app = builder.Build();
```

**Why this follows best practices:**

- **Lifetimes match state scope:** Singleton for the stateless clock, Scoped for `DbContext`/repository, Transient for the stateless email sender.
- **No captive dependencies:** The Singleton (`IClock`) has no Scoped dependencies.
- **Disposal is automatic:** `DbContext` is disposed at the end of each request.
- **Services are small and focused:** Each service has a clear responsibility.

### Real-World Cases

- **EF Core:** `AddDbContext` registers `DbContext` as Scoped by default, aligning with per-request unit-of-work.
- **Background services:** Use `IServiceScopeFactory` to create scopes for Scoped services.
- **Caching:** Use a Singleton cache for application-wide data; use a Scoped cache for request-specific data.

### References

- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Service lifetimes (best practices) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes

---

## Summary Table: Lifetimes at a Glance

| Lifetime | Instance creation | Sharing scope | Disposal | Best for |
|----------|-------------------|---------------|----------|----------|
| **Transient** | Every resolution | None | End of request/scope | Lightweight, stateless services |
| **Scoped** | Once per scope | Within the scope (request) | End of scope/request | Per-request state (`DbContext`, repositories, unit-of-work) |
| **Singleton** | Once for the application | Application-wide | Application shutdown | Application-wide state, caches, clocks, configuration wrappers |

---

## References (Full List)

- Service lifetimes (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- `ServiceLifetime` Enum – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicelifetime
- `AddTransient` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addtransient
- `AddScoped` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addscoped
- `AddSingleton` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addsingleton
- Captive dependency (NDLRCOR012) – https://github.com/dotnet/docs/blob/main/docs/core/extensions/dependency-injection/guidelines.md