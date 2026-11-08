# ASP.NET Core Service Registration: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the built-in DI container (`Microsoft.Extensions.DependencyInjection`). Examples use minimal hosting (`Program.cs`). Service registration APIs are stable; key differences between `AddControllers`, `AddMvc`, `AddRazorPages`, and `AddDbContext` are explained. Where behaviour is version-specific, it is flagged. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Service registration is the process of telling the DI container which services are available, how to create them, and how long they should live. |
| **Technical Definition** | Service registration involves adding `ServiceDescriptor` objects to an `IServiceCollection`. Each descriptor specifies a **service type** (usually an interface), an **implementation type** (or factory/instance), and a **lifetime** (Transient, Scoped, or Singleton). Registration occurs at startup via extension methods (`AddTransient`, `AddScoped`, `AddSingleton`, `AddDbContext`, etc.) before `builder.Build()` creates the `IServiceProvider`. |
| **Beginner-Friendly Explanation** | Service registration is **telling your app’s “supply room” what’s available**. You say: “If someone asks for `IEmailSender`, give them `SmtpEmailSender`, and make a new one each time.” The supply room (DI container) then hands out the right object whenever a class needs it. |

### Key Characteristics

- **Extension-method based** – `IServiceCollection` is extended with `Add*` methods for registration.
- **Three lifetimes** – Transient, Scoped, and Singleton control instance reuse.
- **Convention-driven** – Framework features follow the `Add{GROUP_NAME}` convention.
- **Composable** – Custom extension methods group related registrations.
- **Immutable after build** – Registrations must occur before `builder.Build()`.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Basic understanding of **C# interfaces** and **constructors**.
- Familiarity with `Program.cs` (minimal hosting).

### Related Programming Areas

- **Dependency injection** – The overarching pattern that service registration implements.
- **Service lifetimes** – Determines instance reuse (Transient, Scoped, Singleton).
- **EF Core** – `AddDbContext` registers `DbContext` with a Scoped lifetime by default.
- **MVC / Razor Pages** – `AddControllers`, `AddMvc`, and `AddRazorPages` register framework services.
- **Options pattern** – `AddOptions` and `IOptions<T>` are registered through the same infrastructure.

### Core Concepts / Features

1. **`IServiceCollection`** – The registration collection.
2. **`AddTransient()`** – New instance per resolution.
3. **`AddScoped()`** – One instance per scope (per request).
4. **`AddSingleton()`** – One instance for the application.
5. **`AddControllers()`** – API controller services only.
6. **`AddMvc()`** – Full MVC stack (controllers + views + Razor Pages + APIs).
7. **`AddRazorPages()`** – Razor Pages services only.
8. **`AddDbContext()`** – EF Core `DbContext` registration (Scoped by default).
9. **Service registration best practices** – Organisation, lifetimes, and maintainability.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. `IServiceCollection`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IServiceCollection` is the collection of service descriptors that defines all services available to the DI container. |
| **Technical Definition** | `IServiceCollection` (in `Microsoft.Extensions.DependencyInjection`) is an `ICollection<ServiceDescriptor>`. Each `ServiceDescriptor` holds the service type, implementation (type, factory, or instance), and lifetime. Registration extension methods (`AddTransient`, `AddScoped`, etc.) add descriptors to this collection. After `builder.Build()`, the collection is used to construct the `IServiceProvider` (the container). |
| **Beginner-Friendly Explanation** | `IServiceCollection` is the **“ingredient list”** for your app’s services. Before the app starts, you write down every service you need. The DI container reads this list and knows how to create each service when it is requested. |

### Purposes

- **To** provide a standard, ordered collection where all service registrations are recorded.
- **To** enable framework and application code to register services via a consistent API.
- **To** serve as the input for building the `IServiceProvider` (the container).
- **To** support composition through extension methods that group related registrations.
- **To** allow inspection and modification of registrations before the container is built.

### Syntax Rules and Structure

#### General Syntax (Accessing `IServiceCollection`)

```csharp
var builder = WebApplication.CreateBuilder(args);
IServiceCollection services = builder.Services;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `builder.Services` | The `IServiceCollection` instance provided by the host. |
| Type | `IServiceCollection` (interface). |

#### General Syntax (Adding Descriptors Directly)

```csharp
services.Add(new ServiceDescriptor(
    typeof(IMyService),
    typeof(MyService),
    ServiceLifetime.Scoped));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ServiceDescriptor` | Describes a service. |
| `typeof(IMyService)` | Service type. |
| `typeof(MyService)` | Implementation type. |
| `ServiceLifetime.Scoped` | Lifetime. |

#### Syntax Rules

- `IServiceCollection` is a collection of `ServiceDescriptor` objects.
- Extension methods like `AddTransient`, `AddScoped`, and `AddSingleton` are the preferred way to add descriptors.
- Registrations are **order-sensitive** for multiple implementations: the last registration wins when resolving a single instance.
- **`TryAdd*`** methods (`TryAddTransient`, etc.) register only if the service type is not already registered.

#### Constraints and Limitations

- **Immutable after build** – once `builder.Build()` is called, new registrations are not reflected in the container.
- **`IServiceCollection` is not the container** – it is the collection of descriptors; the `IServiceProvider` is the actual container.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Access IServiceCollection
IServiceCollection services = builder.Services;

// Register via extension method (preferred)
services.AddScoped<IMyService, MyService>();

// Register via ServiceDescriptor directly
services.Add(new ServiceDescriptor(
    typeof(IOtherService),
    typeof(OtherService),
    ServiceLifetime.Transient));

var app = builder.Build();
```

**Expected Output:** The container can resolve `IMyService` (Scoped) and `IOtherService` (Transient) after `Build()`.

**Why this output?** The descriptors are added to `IServiceCollection`; `Build()` uses them to construct the `IServiceProvider`.

### Real-World Cases

- **Framework registration:** `AddControllers()`, `AddRazorPages()`, and `AddDbContext()` all add descriptors to `IServiceCollection`.
- **Custom extension methods:** Groups of related services are registered via `Add{Group}()` extension methods on `IServiceCollection`.
- **Testing:** Integration tests replace or augment registrations in `IServiceCollection` via `WebApplicationFactory`.

### References

- Service registration (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration
- `IServiceCollection` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- `ServiceDescriptor` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor

---

## 2. `AddTransient()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddTransient()` registers a service with a **Transient** lifetime, meaning a new instance is created every time the service is requested. |
| **Technical Definition** | `AddTransient<TService, TImplementation>()` adds a `ServiceDescriptor` with `ServiceLifetime.Transient`. In request-processing apps, transient services are disposed at the end of the request. This lifetime incurs per-request allocations because services are constructed every time they are resolved. |
| **Beginner-Friendly Explanation** | Transient is the **“disposable cup”** lifetime. Every time someone asks for the service, a brand-new instance is created. Nothing is shared, and nothing is reused. |

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
| `AddTransient<TService, TImpl>()` | Type mapping. |
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
- Service registration methods – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration

---

## 3. `AddScoped()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddScoped()` registers a service with a **Scoped** lifetime, meaning one instance is created per scope (typically per HTTP request). |
| **Technical Definition** | `AddScoped<TService, TImplementation>()` adds a `ServiceDescriptor` with `ServiceLifetime.Scoped`. In web apps, a scope is created per HTTP request, so the service is created once per request and shared by all consumers within that request. Scoped services are disposed at the end of the request. |
| **Beginner-Friendly Explanation** | Scoped is the **“per-request”** lifetime. For each HTTP request, one instance is created and shared by everything that needs it during that request. The next request gets a fresh instance. |

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

- Must be resolved from a scope; resolving from the root provider throws.
- A Scoped service can depend on Singleton and other Scoped services.
- A Singleton **must not** depend on a Scoped service (captive dependency).

**Constraints and limitations:**

- **Captive dependencies** – If a Singleton captures a Scoped service, the Scoped service behaves like a Singleton.
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

- **`DbContext`:** EF Core’s `DbContext` is Scoped by default to ensure per-request isolation.
- **Unit-of-work:** A Scoped service that tracks changes across multiple repositories within one request.
- **Per-request caching:** A Scoped cache that avoids repeated database queries within one request.

### References

- Service lifetimes (Scoped) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped
- `AddDbContext` default lifetime – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

---

## 4. `AddSingleton()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddSingleton()` registers a service with a **Singleton** lifetime, meaning one instance is created and shared for the entire application. |
| **Technical Definition** | `AddSingleton<TService, TImplementation>()` adds a `ServiceDescriptor` with `ServiceLifetime.Singleton`. The instance is created on first resolution (or at startup if validation is enabled) and reused for every subsequent request. Singleton services are disposed when the service container is disposed, usually at application shutdown. Singletons **must not** depend on Scoped services. |
| **Beginner-Friendly Explanation** | Singleton is the **“one for everyone, forever”** lifetime. The first time the service is needed, it is created. After that, every class in every request gets the **same** instance. It lives until the app shuts down. |

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

- Can depend on other Singletons and Transients (but not Scoped).
- Must be thread-safe if used concurrently.
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
- Service registration methods – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration

---

## 5. `AddControllers()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddControllers()` registers the services required for **API controllers only**—controllers that return data (JSON/XML) but do not use Razor views or Razor Pages. |
| **Technical Definition** | `AddControllers()` (in `MvcServiceCollectionExtensions`) adds MVC services for controllers without view rendering, Razor Pages, or view-related services. It is the recommended registration for **Web API** projects that use attribute-routed controllers and return `IActionResult` or typed results. It does **not** register services for views or pages. |
| **Beginner-Friendly Explanation** | `AddControllers()` is for **API-only apps**. It sets up everything needed for controllers that return JSON or other data, but it does **not** include Razor views or Razor Pages. Use it when you are building a REST API. |

### Purposes

- **To** register only the services needed for API controllers, avoiding unnecessary view/page services.
- **To** keep the application lightweight and modular.
- **To** support attribute-routed API controllers in Web API projects.
- **To** serve as the foundation for `AddControllersWithViews()` (which adds views on top).

### Syntax Rules and Structure

```csharp
builder.Services.AddControllers();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddControllers()` | Registers API controller services (no views, no pages). |
| Returns | `IMvcBuilder` for further configuration. |

**Syntax rules:**

- **Does not** register Razor view services or Razor Pages services.
- Typically paired with `app.MapControllers()` for attribute-routed APIs.
- Can be extended with `.AddJsonOptions()`, `.AddXmlSerializerFormatters()`, etc.

**Constraints and limitations:**

- **No view rendering** – `AddControllers()` alone cannot serve Razor views or Razor Pages.
- **Not suitable for MVC web apps** that return HTML views; use `AddControllersWithViews()` instead.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers(); // Maps attribute-routed API controllers

app.Run();
```

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(new[] { "Laptop", "Mouse" });
}
```

**Expected Output:** `GET /api/products` returns a JSON array of product names.

**Why this output?** `AddControllers()` registers the services needed for API controllers, and `MapControllers()` maps their attribute routes.

### Real-World Cases

- **REST APIs:** Backend services consumed by mobile apps or SPAs.
- **Microservices:** Lightweight HTTP APIs without UI concerns.
- **Mixed solutions:** API projects within a larger solution, registered separately from UI projects.

### References

- `MvcServiceCollectionExtensions.AddControllers` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollers
- Choosing the right service registration method – https://www.linkedin.com/posts/arvind-gonaki-bb3335280_service-registration-methods-in-aspnet-core-activity-7376564082730196993-lDhH

---

## 6. `AddMvc()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddMvc()` registers the **full MVC stack**: controllers, views, Razor Pages, and API support. |
| **Technical Definition** | `AddMvc()` is a composite registration method that adds all MVC-related services, including controllers, views, Razor Pages, formatters, and validation. In ASP.NET Core 3.0+, it is effectively a wrapper around `AddControllersWithViews()` and `AddRazorPages()`. Modern templates typically use the more granular methods instead of `AddMvc()`, but it remains available for scenarios that need the entire stack. |
| **Beginner-Friendly Explanation** | `AddMvc()` is the **“everything”** registration. It sets up controllers, Razor views, and Razor Pages all at once. It is convenient when you need all MVC features, but modern apps often prefer more targeted methods to avoid loading unnecessary services. |

### Purposes

- **To** register the entire MVC feature set in one call.
- **To** support applications that combine controllers, views, and Razor Pages.
- **To** provide backward compatibility with older ASP.NET Core versions that used `AddMvc()`.
- **To** simplify registration when all MVC features are genuinely needed.

### Syntax Rules and Structure

```csharp
builder.Services.AddMvc();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddMvc()` | Registers the full MVC stack (controllers, views, pages, APIs). |

**Syntax rules:**

- **Includes** controllers, views, Razor Pages, and API formatters.
- In ASP.NET Core 3.0+, more granular methods (`AddControllers()`, `AddControllersWithViews()`, `AddRazorPages()`) are recommended for clarity and performance.
- Returns `IMvcBuilder` for further configuration.

**Constraints and limitations:**

- **Heavier than needed** if you only need APIs or only Razor Pages.
- **Less clear** than granular methods about which features are enabled.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMvc();

var app = builder.Build();

app.UseRouting();
app.MapDefaultControllerRoute();
app.MapRazorPages();

app.Run();
```

**Expected Output:** The app supports both MVC controllers with views and Razor Pages.

**Why this output?** `AddMvc()` registers all MVC-related services, enabling both controllers and Razor Pages.

### Real-World Cases

- **Legacy applications:** Migrated from earlier ASP.NET Core versions that used `AddMvc()`.
- **Mixed applications:** Apps that genuinely need controllers, views, and Razor Pages in the same project.
- **Prototyping:** When the exact feature mix is not yet determined.

### References

- Choosing the right service registration method – https://www.linkedin.com/posts/arvind-gonaki-bb3335280_service-registration-methods-in-aspnet-core-activity-7376564082730196993-lDhH
- `AddMvc` vs granular methods – https://stackoverflow.com/questions/59554510/do-i-need-services-addmvc-or-not

---

## 7. `AddRazorPages()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddRazorPages()` registers the services required for **Razor Pages**—the page-focused UI framework in ASP.NET Core. |
| **Technical Definition** | `AddRazorPages()` (in `MvcServiceCollectionExtensions`) adds services for Razor Pages, including page models, handlers, and page-specific routing. It does **not** register MVC controller services by default; controllers must be added separately with `AddControllers()` or `AddControllersWithViews()` if needed. |
| **Beginner-Friendly Explanation** | `AddRazorPages()` is for **page-focused apps**. It sets up everything needed for Razor Pages but does not include MVC controllers. Use it when your app is built around individual pages rather than controllers. |

### Purposes

- **To** register only the services needed for Razor Pages, keeping the app lightweight.
- **To** support the page-focused programming model (each page has its own model and handlers).
- **To** enable page-specific routing and handler conventions.
- **To** serve as the foundation for Razor Pages applications.

### Syntax Rules and Structure

```csharp
builder.Services.AddRazorPages();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddRazorPages()` | Registers Razor Pages services. |
| Returns | `IMvcBuilder` for further configuration (e.g., `.AddRazorPagesOptions()`). |

**Syntax rules:**

- **Does not** register MVC controller services by default.
- Typically paired with `app.MapRazorPages()`.
- Can be combined with `AddControllers()` if controllers are also needed.

**Constraints and limitations:**

- **No MVC controllers** unless added separately.
- **No view services for MVC views**; Razor Pages have their own rendering pipeline.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddRazorPages();

var app = builder.Build();

app.MapRazorPages();

app.Run();
```

```cshtml
@page
@model IndexModel
<h1>Hello from Razor Pages</h1>
```

**Expected Output:** Navigating to `/` or `/Index` renders the Razor Page.

**Why this output?** `AddRazorPages()` registers page services, and `MapRazorPages()` maps page routes.

### Real-World Cases

- **Page-focused apps:** Content sites, admin panels, and simple CRUD apps built around pages.
- **Form-heavy applications:** Razor Pages’ handler model simplifies form processing.
- **Prototypes:** Rapid development with self-contained pages.

### References

- `MvcServiceCollectionExtensions.AddRazorPages` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addrazorpages
- Choosing the right service registration method – https://www.linkedin.com/posts/arvind-gonaki-bb3335280_service-registration-methods-in-aspnet-core-activity-7376564082730196993-lDhH

---

## 8. `AddDbContext()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddDbContext()` registers an Entity Framework Core `DbContext` (and related services) in the DI container with a **Scoped** lifetime by default. |
| **Technical Definition** | `AddDbContext<TContext>()` registers the `DbContext` type, `DbContextOptions<TContext>`, and related EF Core services. The default lifetime is **Scoped**, aligning with the per-request scope in web apps, because database operations are normally scoped to the client request. Overloads allow specifying a different lifetime (e.g., `ServiceLifetime.Transient` or `Singleton`), but Scoped is recommended for web apps. |
| **Beginner-Friendly Explanation** | `AddDbContext()` is how you tell the app: **“Use this database context, and give each request its own instance.”** It sets up EF Core so that repositories and services can request the `DbContext` and get a properly configured instance for the current request. |

### Purposes

- **To** register EF Core’s `DbContext` and its options in the DI container.
- **To** ensure per-request isolation of database operations (Scoped lifetime).
- **To** configure the database provider (SQL Server, SQLite, PostgreSQL, etc.) and connection string.
- **To** enable constructor injection of `DbContext` into repositories and services.
- **To** support testability by allowing in-memory or mock providers to be registered.

### Syntax Rules and Structure

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddDbContext<TContext>()` | Registers the context and options. |
| `options => ...` | Configures the database provider and connection. |
| Default lifetime | **Scoped** (one per request). |

**Syntax rules:**

- The default lifetime is **Scoped**.
- A different lifetime can be specified with an overload, but Scoped is recommended for web apps.
- Services of a given lifetime **should not** depend on a `DbContext` with a shorter lifetime.
- For background services, consider `AddDbContextFactory<T>()` to create short-lived contexts on demand.

**Constraints and limitations:**

- **Captive dependencies** – A Singleton must not depend on a Scoped `DbContext`.
- **Thread safety** – `DbContext` is not thread-safe; Scoped lifetime prevents concurrent access.
- **Migrations** – `AddDbContext` is for runtime registration; migrations use separate tooling.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<IProductRepository, ProductRepository>();

var app = builder.Build();

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Product> Products { get; set; }
}

public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _db;
    public ProductRepository(AppDbContext db) => _db = db;
}
```

**Expected Output:** Each HTTP request gets a new `AppDbContext` instance, shared by all repositories within that request, and disposed at the end of the request.

**Why this output?** `AddDbContext` registers the context with a Scoped lifetime, and the per-request scope manages its creation and disposal.

### Real-World Cases

- **Web applications:** Standard registration for EF Core in MVC, Razor Pages, and APIs.
- **Repositories and unit-of-work:** Scoped `DbContext` ensures a consistent transaction within a request.
- **Testing:** Replace the provider with an in-memory database in integration tests.

### References

- `AddDbContext` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext
- Dependency injection in ASP.NET Core (EF Core context lifetime) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Service lifetimes (Scoped) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped

---

## 9. Service Registration Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Service registration best practices are guidelines for organizing, naming, and scoping registrations to keep `Program.cs` clean and maintainable. |
| **Technical Definition** | Best practices encompass grouping related registrations into **`Add{GROUP_NAME}` extension methods**, returning `IServiceCollection` for fluent chaining, placing extension methods near the services they register, choosing lifetimes explicitly, and avoiding massive `Program.cs` files with hundreds of registrations. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for registration — they help you avoid a 200-line `Program.cs` and make it easy to find and reuse service configurations. |

### Purposes

- **To** keep `Program.cs` clean and readable.
- **To** make service configurations reusable across production and tests.
- **To** establish clear boundaries between feature areas.
- **To** reduce merge conflicts in large teams.
- **To** make it easy to locate and modify related registrations.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Organisation** | Group related services into `Add{GroupName}()` extension methods. | Dump hundreds of registrations directly in `Program.cs`. |
| **Return type** | Return `IServiceCollection` for fluent chaining. | Return `void` from extension methods. |
| **Location** | Place extension methods near the services they register. | Scatter registration logic across unrelated folders. |
| **Lifetimes** | Explicitly choose and document lifetimes (Transient/Scoped/Singleton). | Rely on defaults without understanding their implications. |
| **Captive dependencies** | Ensure Singletons do not depend on Scoped services. | Capture a Scoped service in a Singleton. |
| **Framework methods** | Use granular methods (`AddControllers()`, `AddRazorPages()`) instead of `AddMvc()` when possible. | Use `AddMvc()` when only APIs or only Razor Pages are needed. |
| **Testing** | Reuse `Add{GroupName}()` methods in test setups. | Duplicate registration logic between production and tests. |

### Annotated Code Example (Best-Practice Registration)

```csharp
// UserServiceCollectionExtensions.cs
namespace MyApp.Users;

public static class UserServiceCollectionExtensions
{
    public static IServiceCollection AddUserServices(this IServiceCollection services)
    {
        services.AddScoped<IUserRepository, UserRepository>();
        services.AddScoped<IUserService, UserService>();
        return services;
    }
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddUserServices()
    .AddOrderServices()
    .AddEmailServices()
    .AddControllers();

var app = builder.Build();
```

**Why this follows best practices:**

- **Grouped registrations** keep `Program.cs` minimal and readable.
- **Fluent chaining** returns `IServiceCollection` for composability.
- **Feature-based extension methods** have clear boundaries and are reusable in tests.
- **Granular `AddControllers()`** avoids loading unnecessary view/page services.

### Real-World Cases

- **Large applications:** Modular `Add{Feature}()` methods keep registration organized across teams.
- **Test suites:** Reuse extension methods to configure the DI container consistently in integration tests.
- **Library authors:** Provide `AddMyLibrary()` extension methods that follow the `Add{GROUP_NAME}` convention.

### References

- Service registration (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration
- Dependency Injection Patterns (GitHub) – https://github.com/tomevault-io/skills-registry/blob/main/aaronontheweb--dotnet-skills--microsoft-extensions-dependency-injection/SKILL.md
- How to Create Custom DI Extensions in .NET – https://raw.githubusercontent.com/.../custom-di-extensions.md

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **`IServiceCollection`** | Collection of service descriptors | `builder.Services` | All registrations |
| **`AddTransient()`** | New instance per resolution | `AddTransient<TService, TImpl>()` | Lightweight, stateless services |
| **`AddScoped()`** | One instance per request/scope | `AddScoped<TService, TImpl>()` | Per-request state (`DbContext`, repositories) |
| **`AddSingleton()`** | One instance for the application | `AddSingleton<TService, TImpl>()` | Application-wide state, caches, clocks |
| **`AddControllers()`** | API controller services only | `AddControllers()` + `MapControllers()` | REST APIs |
| **`AddMvc()`** | Full MVC stack | `AddMvc()` | Apps needing controllers + views + Razor Pages |
| **`AddRazorPages()`** | Razor Pages services only | `AddRazorPages()` + `MapRazorPages()` | Page-focused apps |
| **`AddDbContext()`** | EF Core `DbContext` (Scoped) | `AddDbContext<TContext>(options => ...)` | Every EF Core web app |
| **Best practices** | Clean, composable registrations | `Add{GroupName}()` extension methods | Every production app |

---

## References (Full List)

- Service registration (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration
- Service lifetimes (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- `IServiceCollection` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- `ServiceDescriptor` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor
- `MvcServiceCollectionExtensions` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions
- `AddDbContext` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext
- Choosing the right service registration method – https://www.linkedin.com/posts/arvind-gonaki-bb3335280_service-registration-methods-in-aspnet-core-activity-7376564082730196993-lDhH
- Dependency Injection Patterns (GitHub) – https://github.com/tomevault-io/skills-registry/blob/main/aaronontheweb--dotnet-skills--microsoft-extensions-dependency-injection/SKILL.md
- How to Create Custom DI Extensions in .NET – https://raw.githubusercontent.com/.../custom-di-extensions.md