# ASP.NET Core MVC Global Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Global filters are filters registered once and applied to **every controller action and Razor Page** in the application. They are registered through the `Filters` collection on `MvcOptions`, typically configured via `AddControllers()`, `AddControllersWithViews()`, or `AddMvc()`. Two activation mechanisms exist: `options.Filters.Add<T>()` (type-activated, no DI constructor injection) and `options.Filters.AddService<T>()` (DI-resolved, lifetime-managed). Global filters surround controller filters, which surround action filters. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Global filters are filters registered once at application startup that apply to **all controller actions and Razor Pages** without requiring attributes on individual controllers or actions. |
| **Technical Definition** | Global filters are added to the `FilterCollection` exposed by `MvcOptions.Filters`. The `MvcOptions` instance is configured through the `AddControllers()`, `AddControllersWithViews()`, or `AddMvc()` service-registration methods. Global filters have the **broadest scope** in the MVC filter pipeline: they surround controller-scoped filters, which in turn surround action-scoped filters. Registration supports type activation (`Add<T>()`), instance registration (`Add(instance)`), and DI-resolved activation (`AddService<T>()`). |
| **Beginner-Friendly Explanation** | Global filters are **filters that apply to everything**. Instead of putting a `[MyFilter]` attribute on every controller, you register it once in `Program.cs`, and it runs for every request that reaches MVC. This is ideal for truly cross-cutting concerns like global error handling, audit logging, or security headers—but dangerous if the filter is not safe to run everywhere. |

### Key Characteristics

- **Broadest scope** – Global filters apply to all controllers, actions, and Razor Pages.
- **Surrounding order** – Global filters surround controller filters, which surround action filters.
- **Two activation modes** – `Add<T>()` (type-activated, no DI) and `AddService<T>()` (DI-resolved, lifetime-managed).
- **Configured via `MvcOptions`** – Registered through `AddControllers()`, `AddControllersWithViews()`, or `AddMvc()`.
- **Order overrides scope** – The `IOrderedFilter.Order` property takes precedence over scope when determining execution order.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** and **filter scopes** (global → controller → action).
- Familiarity with **dependency injection** and **service lifetimes**.
- Understanding of `Program.cs` (minimal hosting) and service registration.

### Related Programming Areas

- **Filter scopes** – Global, controller, and action scopes determine default ordering.
- **Dependency injection** – `AddService<T>()` enables DI for global filters.
- **Middleware** – The preferred alternative for global exception handling; filters are MVC-specific.
- **`MvcOptions`** – The configuration object that exposes the `Filters` collection.
- **Service registration methods** – `AddControllers`, `AddControllersWithViews`, and `AddMvc` all configure `MvcOptions`.

### Core Concepts / Features

1. **Global filters** – The concept, scope, and execution order.
2. **`AddControllers()`** – The service-registration method that configures global filters (with `AddControllersWithViews` and `AddMvc` variants).
3. **`options.Filters.Add()`** – The registration methods (`Add<T>()`, `AddService<T>()`, `Add(instance)`) and their DI/lifetime implications.
4. **Global filter best practices** – Safety, DI, ordering, and scope discipline.

---

## 1. Global Filters (The Concept and Scope)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Global filters are filters registered at application startup that apply to every controller action and Razor Page in the MVC pipeline. |
| **Technical Definition** | A global filter is added to `MvcOptions.Filters` during service configuration. It participates in the filter pipeline at the **global scope**, which is the outermost scope. When multiple filters for the same stage exist, **scope determines the default order**: global filters surround class (controller) filters, which surround method (action) filters. Because filters are nested, the “after” code of global filters runs **after** the “after” code of controller and action filters. The `Order` property overrides scope when set. |
| **Beginner-Friendly Explanation** | Global filters are **the outermost layer of the filter onion**. They run first on the way in and last on the way out. If you register a global logging filter and an action-level logging filter, the global one logs “before” first and “after” last—because it wraps everything else. |

### Purposes

- **To** apply a filter to **every action and Razor Page** without repeating attributes.
- **To** enforce **application-wide policies** (security headers, global error handling, audit logging) consistently.
- **To** provide the **outermost wrapper** around controller and action filters, enabling before-and-after logic that spans the entire MVC pipeline.
- **To** reduce boilerplate when a cross-cutting concern genuinely applies to all endpoints.
- **To** centralise configuration of filters that should never be forgotten on individual actions.

### Syntax Rules and Structure

#### Filter Scope and Ordering

```text
Global filter “before” code
  → Controller filter “before” code
    → Action filter “before” code
      → Action method
    ← Action filter “after” code
  ← Controller filter “after” code
← Global filter “after” code
```

**Component breakdown:**

| Scope | Position in the nesting | Default order behaviour |
|-------|-------------------------|--------------------------|
| **Global** | Outermost | “Before” runs first; “after” runs last |
| **Controller** | Middle | Surrounds action filters |
| **Action** | Innermost | “Before” runs last; “after” runs first |

**Syntax rules:**

- **Global filters surround controller filters, which surround action filters**—this is the default order determined by scope.
- **The `Order` property trumps scope**: filters are sorted first by `Order`, and scope is used only to break ties when `Order` values are equal.
- Controller-level filters set `Order` to `int.MinValue` by default, meaning they run **before** action filters unless the action filter explicitly sets a lower `Order`.

**Constraints and limitations:**

- **Global filters cannot be applied to Razor Page handler methods**—they apply to the page model or globally, but not to individual handler methods.
- **Global filters affect every request**, including health checks, error endpoints, and static-file-adjacent MVC actions—design them defensively.

### Annotated Code Example (Global vs. Controller vs. Action Ordering)

```csharp
// Global filter
public class GlobalLogFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
        => Console.WriteLine("Global: Before");
    public void OnActionExecuted(ActionExecutedContext context)
        => Console.WriteLine("Global: After");
}

// Controller filter
public class ControllerLogFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
        => Console.WriteLine("Controller: Before");
    public void OnActionExecuted(ActionExecutedContext context)
        => Console.WriteLine("Controller: After");
}

// Action filter
public class ActionLogFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
        => Console.WriteLine("Action: Before");
    public void OnActionExecuted(ActionExecutedContext context)
        => Console.WriteLine("Action: After");
}

// Controller with controller and action filters
[ControllerLogFilter]
public class OrderController : Controller
{
    [ActionLogFilter]
    public IActionResult Index() => Content("Done");
}

// Registration
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<GlobalLogFilter>();
});
```

**Expected Output:**

```
Global: Before
Controller: Before
Action: Before
Action: After
Controller: After
Global: After
```

**Why this output?** Global filters surround controller filters, which surround action filters. The “before” code runs from outermost to innermost; the “after” code runs in reverse, from innermost to outermost.

### Real-World Cases

- **Global security headers:** Add `X-Content-Type-Options`, `X-Frame-Options`, etc. to every MVC response.
- **Global audit logging:** Log every action invocation with user and route information.
- **Global model-state guard:** Return 400 automatically for invalid `ModelState` across all API actions.

### References

- Filter scopes and order of execution – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filter-scopes-and-order-of-execution
- Global filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filters
- `IOrderedFilter.Order` overrides scope – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#override-the-default-order

---

## 2. `AddControllers()` (and Service-Registration Variants)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AddControllers()` is an `IServiceCollection` extension method that registers the services required for MVC controllers and exposes `MvcOptions` for configuring global filters. |
| **Technical Definition** | `AddControllers()` registers MVC controller services **without** view or Razor Pages services. It accepts an optional `Action<MvcOptions>` configuration delegate, which is the canonical place to add global filters via `options.Filters.Add<T>()` or `options.Filters.AddService<T>()`. Related methods include `AddControllersWithViews()` (adds view services and is required for antiforgery filters) and `AddMvc()` (registers the full MVC stack). All three configure the same `MvcOptions.Filters` collection. |
| **Beginner-Friendly Explanation** | `AddControllers()` is **where you set up controllers and tell MVC about global filters**. It is the entry point for configuring `MvcOptions`, which contains the `Filters` collection. If your app only has APIs, use `AddControllers()`; if it has views, use `AddControllersWithViews()`; if you need the full MVC stack, use `AddMvc()`. |

### Purposes

- **To** register the MVC controller services required for action execution.
- **To** expose `MvcOptions` so global filters, formatters, and other MVC settings can be configured.
- **To** provide a **centralised configuration point** for application-wide filter policies.
- **To** differentiate between API-only (`AddControllers`), view-based (`AddControllersWithViews`), and full-stack (`AddMvc`) scenarios.
- **To** enable `AddService<T>()` registration of DI-resolved global filters through the `Filters` collection.

### Syntax Rules and Structure

#### `AddControllers()` with Global Filters

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers(options =>
{
    options.Filters.Add<GlobalSampleActionFilter>();          // type-activated
    options.Filters.AddService<GlobalDiFilter>();             // DI-resolved
    options.Filters.Add(new SampleGlobalActionFilter());     // instance
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddControllers(Action<MvcOptions>)` | Registers controller services and configures MVC options. |
| `options.Filters` | The `FilterCollection` where global filters are added. |
| `Add<T>()` | Type-activated global filter (no DI constructor injection). |
| `AddService<T>()` | DI-resolved global filter (respects lifetime). |
| `Add(instance)` | Pre-constructed instance. |

#### Variants

| Method | Views? | Razor Pages? | When to use |
|--------|--------|--------------|-------------|
| `AddControllers()` | No | No | API-only apps |
| `AddControllersWithViews()` | Yes | No | MVC apps with views |
| `AddMvc()` | Yes | Yes | Full MVC stack |

**Syntax rules:**

- The `Action<MvcOptions>` delegate is executed during service configuration, before the container is built.
- Global filters registered here apply to **all controllers and actions** in the MVC pipeline.
- `AddControllersWithViews()` is **required** for antiforgery filters to function; `AddControllers()` alone does not register them.

**Constraints and limitations:**

- **`AddControllers()` is API-focused**—it does not add view or Razor Pages services.
- **Filters registered via `Add<T>()` do not receive DI constructor injection**—use `AddService<T>()` for DI-backed filters.
- **Global filters registered here run for every controller action**—ensure they are safe for all endpoints.

### Annotated Code Example (Configuring Global Filters via `AddControllersWithViews`)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register a global filter via AddControllersWithViews
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
    options.Filters.AddService<AuditFilter>();
    options.Filters.Add(new SecurityHeadersFilter());
});

// Register the DI-resolved filter
builder.Services.AddScoped<AuditFilter>();

var app = builder.Build();

app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Step-by-step explanation:**

1. `AddControllersWithViews` registers MVC services and configures `MvcOptions`.
2. `options.Filters.Add<GlobalExceptionFilter>()` registers a type-activated global exception filter.
3. `options.Filters.AddService<AuditFilter>()` registers a DI-resolved global filter; `AuditFilter` must be registered in DI (Scoped in this example).
4. `options.Filters.Add(new SecurityHeadersFilter())` registers a pre-constructed instance.
5. All three filters apply to every controller action.

**Expected Output:** Every controller action runs the global exception filter, the DI-resolved audit filter, and the security headers filter. The audit filter receives its Scoped dependencies from DI.

**Why this output?** `AddControllersWithViews` exposes `MvcOptions`, and the `Filters` collection is populated with the global filters. `AddService<T>()` resolves the filter from DI per request, respecting its Scoped lifetime.

### Real-World Cases

- **API-only applications:** `AddControllers(options => options.Filters.Add<ApiExceptionFilter>())` for global JSON error handling.
- **MVC with views:** `AddControllersWithViews(options => options.Filters.Add<AntiforgeryFilter>())` for global antiforgery validation.
- **Full-stack MVC:** `AddMvc(options => options.Filters.Add<GlobalAuditFilter>())` when controllers, views, and Razor Pages all need the filter.

### References

- `MvcServiceCollectionExtensions.AddControllers` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollers
- `MvcServiceCollectionExtensions.AddControllersWithViews` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollerswithviews
- Global filters via `AddControllersWithViews` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filters
- Antiforgery filters require `AddControllersWithViews` – https://github.com/dotnet/AspNetCore.Docs/issues/25871

---

## 3. `options.Filters.Add()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `options.Filters.Add()` is the method used to register a global filter in the `FilterCollection` exposed by `MvcOptions`. |
| **Technical Definition** | The `FilterCollection` on `MvcOptions` provides several overloads: `Add<T>()` (type-activated filter, no DI constructor injection), `Add(Type)` (non-generic type-activated), `Add(IFilterMetadata)` (pre-constructed instance), and `AddService<T>()` (DI-resolved filter, respects lifetime). `Add<T>()` creates a `TypeFilterAttribute` internally, which uses `ObjectFactory` and **does not** resolve arbitrary constructor dependencies from DI. `AddService<T>()` resolves the filter from `IServiceProvider`, so the filter must be registered in DI and its lifetime is managed by the container. |
| **Beginner-Friendly Explanation** | `options.Filters.Add()` is **how you tell MVC about a global filter**. There are three main flavours: `Add<T>()` creates the filter without DI (it must have a parameterless constructor or dependencies the framework can supply), `AddService<T>()` gets the filter from DI (so it can have constructor dependencies and a managed lifetime), and `Add(instance)` registers an already-created instance. |

### Purposes

- **To** register a global filter **type** when the filter has no DI dependencies (`Add<T>()`).
- **To** register a global filter **resolved from DI** when it needs constructor dependencies or a managed lifetime (`AddService<T>()`).
- **To** register a **pre-constructed filter instance** when you already have an object (`Add(instance)`).
- **To** control **ordering** by passing an `Order` value to the registration.
- **To** centralise filter configuration in `Program.cs` rather than scattering attributes across controllers.

### Syntax Rules and Structure

#### Overloads

```csharp
options.Filters.Add<MyFilter>();                          // type-activated
options.Filters.Add<MyFilter>(int.MinValue);              // with Order
options.Filters.AddService<MyDiFilter>();                 // DI-resolved
options.Filters.AddService<MyDiFilter>(int.MaxValue);     // DI-resolved with Order
options.Filters.Add(new MyFilterInstance());              // pre-constructed instance
```

**Component breakdown:**

| Overload | Activation | DI constructor injection? | Lifetime |
|----------|------------|---------------------------|----------|
| `Add<T>()` | `ObjectFactory` | **No** (only framework-resolvable dependencies) | New per request |
| `AddService<T>()` | `IServiceProvider.GetRequiredService` | **Yes** | As registered (Scoped/Transient/Singleton) |
| `Add(instance)` | None (instance supplied) | N/A | Instance lifetime |

**Syntax rules:**

- **`Add<T>()`** does **not** resolve arbitrary constructor dependencies from DI; the filter must have a parameterless constructor or dependencies the framework can supply (e.g., `ILogger<T>`).
- **`AddService<T>()`** requires the filter to be registered in DI; an unregistered type throws at runtime.
- **`Order`** can be passed to `Add<T>()` and `AddService<T>()`; lower values run earlier.
- **Instance registration** does not support DI resolution for the instance—create it manually with its dependencies.

**Constraints and limitations:**

- **`Add<T>()` is often mistaken for DI-aware**—it is not; use `AddService<T>()` when constructor injection is needed.
- **Global filters registered with `Add<T>()` are created once and reused**—they should be stateless or thread-safe.
- **`AddService<T>()` respects the filter’s DI lifetime**—a Scoped filter is created per request; a Singleton filter is shared across all requests.

### Annotated Code Example (`Add<T>` vs. `AddService<T>`)

```csharp
// ❌ No DI — parameterless constructor required
public class SimpleGlobalFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// ✅ DI-backed — constructor dependencies resolved
public class DiGlobalFilter : IActionFilter
{
    private readonly ILogger<DiGlobalFilter> _logger;
    public DiGlobalFilter(ILogger<DiGlobalFilter> logger) => _logger = logger;
    public void OnActionExecuting(ActionExecutingContext context)
        => _logger.LogInformation("Global filter executing");
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Registration
builder.Services.AddControllers(options =>
{
    options.Filters.Add<SimpleGlobalFilter>();          // ✅ no DI
    // options.Filters.Add<DiGlobalFilter>();           // ❌ would throw — no parameterless ctor
    options.Filters.AddService<DiGlobalFilter>();       // ✅ DI-resolved
});

builder.Services.AddScoped<DiGlobalFilter>();           // required for AddService
```

**Step-by-step explanation:**

1. `SimpleGlobalFilter` has a parameterless constructor and is registered via `Add<T>()`.
2. `DiGlobalFilter` requires `ILogger<T>` and cannot be activated via `Add<T>()`.
3. `AddService<DiGlobalFilter>()` resolves the filter from DI, where `ILogger<T>` is supplied and the filter’s Scoped lifetime is respected.

**Expected Output:** Both filters run globally. `SimpleGlobalFilter` is created per request; `DiGlobalFilter` receives its `ILogger<T>` from DI and is created per request (Scoped).

**Why this output?** `Add<T>()` uses type activation without DI; `AddService<T>()` delegates resolution to the container, enabling constructor injection and lifetime management.

### Real-World Cases

- **Global exception filter (no DI):** `options.Filters.Add<GlobalExceptionFilter>()` when the filter has only framework-resolvable dependencies.
- **Global audit filter (DI-backed):** `options.Filters.AddService<AuditFilter>()` with `AuditFilter` registered as Scoped so it can use a `DbContext`.
- **Global security headers (instance):** `options.Filters.Add(new SecurityHeadersFilter())` when the filter is stateless and can be created once.

### References

- `FilterCollection.Add<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercollection.add
- `FilterCollection.AddService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercollection.addservice
- Global filter registration via `MvcOptions.Filters` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filters
- `Add<T>()` vs `AddService<T>()` (CSDN) – https://blog.csdn.net/lxy_123456789/article/details/159964136

---

## 4. Global Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Global filter best practices are guidelines for registering and implementing global filters that are safe, performant, and maintainable across every request in the application. |
| **Technical Definition** | Best practices encompass using `AddService<T>()` for DI-backed filters, registering filters with appropriate DI lifetimes (Scoped for request-specific state, Singleton for stateless global state), preferring **middleware** for global exception handling and request-level concerns, **avoiding scoped dependencies in singleton global filters**, using `IOrderedFilter.Order` deliberately, and keeping global filters **stateless and defensive** because they affect every endpoint. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for global filters—they help you avoid a filter that crashes on every request, leaks state between users, or duplicates middleware that already handles the concern better. |

### Purposes

- **To** choose the right registration method (`Add<T>()` vs. `AddService<T>()`) based on DI needs.
- **To** prevent **lifetime mismatches** (Scoped services captured in Singleton global filters).
- **To** avoid **duplicating middleware** for concerns that middleware handles better (exception handling, CORS, authentication).
- **To** keep global filters **safe for all endpoints**, including error and health-check endpoints.
- **To** ensure **thread safety** when filters are reused across requests.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **DI registration** | Use `AddService<T>()` when the filter needs constructor dependencies. | Use `Add<T>()` and expect arbitrary constructor dependencies to be injected. |
| **Lifetime** | Register DI-backed global filters as **Scoped** (per request) or **Transient**; use Singleton only for stateless filters. | Inject Scoped services (e.g., `DbContext`) into a Singleton global filter. |
| **Middleware first** | Use **exception-handling middleware** (`UseExceptionHandler`) for global error handling. | Rely solely on a global exception filter—it does not catch resource/result filter exceptions. |
| **Endpoint safety** | Make global filters defensive—they run on error endpoints, health checks, and all MVC actions. | Assume the filter only runs on “normal” business actions. |
| **Ordering** | Use `Order` deliberately when a global filter must run before or after other filters. | Assume registration order alone determines execution order. |
| **Statelessness** | Keep global filters stateless or thread-safe. | Store per-request or per-user state in a global filter instance. |
| **Scope awareness** | Remember global filters **do not run on short-circuited responses** unless they implement `IAlwaysRunResultFilter`. | Expect ordinary result filters to run on 401/403 responses. |
| **Antiforgery** | Use `AddControllersWithViews()` (not `AddControllers()`) when global antiforgery filters are needed. | Use `AddControllers()` and expect antiforgery filters to work. |

### Annotated Code Example (Best-Practice Global Filter Setup)

```csharp
// ✅ Stateless, thread-safe global filter (Singleton-safe)
public class SecurityHeadersGlobalFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers.Append(
            "X-Content-Type-Options", "nosniff");
        context.HttpContext.Response.Headers.Append(
            "X-Frame-Options", "DENY");
    }
    public void OnResultExecuted(ResultExecutedContext context) { }
}

// ✅ Scoped DI-backed global filter
public class AuditGlobalFilter : IAsyncActionFilter
{
    private readonly IAuditService _audit;
    public AuditGlobalFilter(IAuditService audit) => _audit = audit;

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        await _audit.LogAsync(context.ActionDescriptor.DisplayName);
        await next();
    }
}

// Registration
builder.Services.AddScoped<AuditGlobalFilter>();
builder.Services.AddScoped<IAuditService, AuditService>();

builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<SecurityHeadersGlobalFilter>();     // stateless, type-activated
    options.Filters.AddService<AuditGlobalFilter>();        // DI-resolved, Scoped
});

// Middleware handles global exceptions (preferred over a global exception filter)
app.UseExceptionHandler("/error");
```

**Why this follows best practices:**

- **`SecurityHeadersGlobalFilter`** is stateless and registered via `Add<T>()`; it is safe to reuse and does not depend on DI.
- **`AuditGlobalFilter`** uses `AddService<T>()` and is registered as **Scoped**, so it receives a per-request `IAuditService` without capturing Scoped state in a Singleton.
- **Exception handling is delegated to middleware** (`UseExceptionHandler`), which is the recommended approach for global error handling.
- **The security headers filter implements `IResultFilter`**, so it runs around result execution and adds headers before the response is written.

### Real-World Cases

- **Global security headers:** A stateless `IResultFilter` registered via `Add<T>()` adds headers to every MVC response.
- **Global audit logging:** A Scoped `IAsyncActionFilter` registered via `AddService<T>()` logs every action with a per-request audit service.
- **API error handling:** Middleware handles exceptions globally; an action-specific exception filter is applied only where JSON error responses differ from HTML error pages.

### References

- Filters in ASP.NET Core (Dependency injection) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- `AddService<T>()` for DI-resolved global filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#dependency-injection
- Exception filters vs. middleware – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Global filter ordering and `IOrderedFilter` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#override-the-default-order
- Antiforgery filters require `AddControllersWithViews` – https://github.com/dotnet/AspNetCore.Docs/issues/25871

---

## Summary Table: Global Filters at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Global filters** | Apply to all controllers/actions | `options.Filters.Add...` | Truly cross-cutting concerns (security headers, audit logging) |
| **`AddControllers()`** | Configure MVC and global filters | `AddControllers(options => ...)` | API-only apps |
| **`AddControllersWithViews()`** | Configure MVC + views + global filters | `AddControllersWithViews(options => ...)` | MVC apps with views; required for antiforgery filters |
| **`options.Filters.Add<T>()`** | Type-activated global filter | `Add<T>()`, `Add<T>(order)` | Filters with no DI constructor dependencies |
| **`options.Filters.AddService<T>()`** | DI-resolved global filter | `AddService<T>()`, `AddService<T>(order)` | Filters needing constructor injection and lifetime management |
| **Best practices** | Safety and maintainability | `AddService<T>()` for DI, Scoped lifetime, middleware for exceptions, stateless filters | Every global filter registration |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Filter scopes and order of execution – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filter-scopes-and-order-of-execution
- Global filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filters
- Override the default order (`IOrderedFilter`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#override-the-default-order
- Dependency injection for filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#dependency-injection
- Exception filters (middleware preferred) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- `MvcServiceCollectionExtensions.AddControllers` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollers
- `MvcServiceCollectionExtensions.AddControllersWithViews` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addcontrollerswithviews
- `FilterCollection.Add<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercollection.add
- `FilterCollection.AddService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercollection.addservice
- Antiforgery filters require `AddControllersWithViews` – https://github.com/dotnet/AspNetCore.Docs/issues/25871
- `Add<T>()` vs `AddService<T>()` (CSDN) – https://blog.csdn.net/lxy_123456789/article/details/159964136
- Global filter ordering and scope (Stack Overflow) – https://stackoverflow.com/questions/43658066/servicefilter-and-typefilter-what-is-the-difference-in-injection-those-filters/45437900#45437900