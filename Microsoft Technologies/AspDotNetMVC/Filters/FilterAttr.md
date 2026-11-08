# ASP.NET Core MVC Filter Attributes: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Filter attributes are the declarative mechanism for applying filters to controllers and actions. The three attributes covered here—`[ServiceFilter]`, `[TypeFilter]`, and `[MiddlewareFilter]`—all implement `IFilterFactory`, which means they create filter instances at runtime rather than being simple marker attributes. **Generic versions** (`[ServiceFilter<T>]`, `[TypeFilter<T>]`, `[MiddlewareFilter<T>]`) were introduced in **.NET 8** and are the preferred syntax in modern code. `[MiddlewareFilter]` runs middleware as an async resource filter, executing at the same pipeline stage as resource filters—before model binding and after the rest of the pipeline. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filter attributes are declarative .NET attributes applied to controllers or action methods that add filters to the MVC filter pipeline. |
| **Technical Definition** | Filter attributes implement `IFilterMetadata` (directly or via `IFilterFactory`). Simple attributes (e.g., a class inheriting `ActionFilterAttribute`) are instantiated by the framework and cannot receive constructor dependencies from DI. **Factory-based attributes** (`ServiceFilterAttribute`, `TypeFilterAttribute`, `MiddlewareFilterAttribute`) implement `IFilterFactory` and create the actual filter instance at runtime, enabling dependency injection and parameterization. Generic variants (`ServiceFilter<T>`, `TypeFilter<T>`, `MiddlewareFilter<T>`) were added in .NET 8 and provide compile-time type safety. |
| **Beginner-Friendly Explanation** | Filter attributes are **labels you stick on controllers or actions** to add cross-cutting behaviour. Some labels are simple and static (e.g., `[Authorize]`), but when your filter needs services from DI or constructor parameters, you use the factory attributes: `[ServiceFilter]` (the filter is registered in DI and resolved from the container), `[TypeFilter]` (the filter is created on the fly with constructor arguments you supply), or `[MiddlewareFilter]` (a middleware pipeline is run as a filter). |

### Key Characteristics

- **Declarative** – Applied as attributes on controllers or actions.
- **Factory-based** – `ServiceFilter`, `TypeFilter`, and `MiddlewareFilter` implement `IFilterFactory`, creating the filter instance at runtime.
- **DI-capable** – `ServiceFilter` resolves the filter from the DI container; `TypeFilter` uses `ObjectFactory` and still fulfills constructor dependencies from DI.
- **Parameterizable** – `TypeFilter` allows passing constructor arguments to the filter.
- **Lifetime-aware** – `ServiceFilter` respects the filter’s registered lifetime; `TypeFilter` creates a new instance per request (unless `IsReusable` is set).
- **Generic syntax** (.NET 8+) – `[ServiceFilter<T>]`, `[TypeFilter<T>]`, `[MiddlewareFilter<T>]` provide compile-time checking and remove the `typeof()` call.

### Prerequisites

- .NET SDK (6.0+ recommended; generic attributes require .NET 8+), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** (authorization → resource → action → exception → result).
- Familiarity with **dependency injection** and **service lifetimes**.
- Basic understanding of **middleware** (for `MiddlewareFilter`).

### Related Programming Areas

- **Dependency injection** – `ServiceFilter` and `TypeFilter` are the primary DI-enabled filter attributes.
- **Middleware** – `MiddlewareFilter` bridges middleware and the filter pipeline.
- **Filter interfaces** – `IActionFilter`, `IResourceFilter`, etc., define the behaviour the attributes activate.
- **Service lifetimes** – `ServiceFilter` respects the registered lifetime; `TypeFilter` creates a new instance per request.
- **Filter ordering** – `Order` property controls execution sequence.

### Core Concepts / Features

1. **Filter attributes** – The concept, factory-based attributes, and when to use them.
2. **`[ServiceFilter]`** – Resolving filters from DI with lifetime management.
3. **`[TypeFilter]`** – Creating filters with constructor arguments and DI-fulfilled dependencies.
4. **`[MiddlewareFilter]`** – Running a middleware pipeline as a resource filter.
5. **Filter attribute best practices** – DI, lifetimes, ordering, and generic syntax.

---

## 1. Filter Attributes (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filter attributes are .NET attributes that, when applied to a controller or action, cause a filter to run at the appropriate stage of the MVC filter pipeline. |
| **Technical Definition** | Filter attributes implement `IFilterMetadata`. Attributes that implement `IFilterFactory` are **factory-based**: they expose a `CreateInstance(IServiceProvider)` method that the framework calls to create the actual filter instance. This decouples the attribute from the filter implementation, enabling DI and parameterization. The framework’s `FilterFactory` and `TypeFilterAttribute` are examples of this pattern. |
| **Beginner-Friendly Explanation** | Filter attributes are **shortcuts for adding filters**. Instead of registering a filter globally, you can put a label on a specific controller or action. But when the filter needs complex construction (DI dependencies or parameters), the simple attribute isn’t enough—you need a factory attribute that knows how to build the filter at runtime. |

### Purposes

- **To** apply filters **declaratively** to specific controllers or actions without global registration.
- **To** enable **dependency injection** into filters that cannot be instantiated by the framework alone.
- **To** **parameterize** filters with constructor arguments at the point of application.
- **To** run **middleware** as a filter, bridging the two extension models.
- **To** keep filter configuration close to where it is used (the controller or action).

### Syntax Rules and Structure

#### Three Approaches to Applying a Filter

```csharp
// 1. Direct attribute (filter is a simple Attribute class)
[SampleActionFilter]
public IActionResult Direct() => Content("Direct");

// 2. TypeFilter (filter type + optional constructor arguments)
[TypeFilter(typeof(SampleActionFilter), Arguments = new object[] { "arg" })]
public IActionResult WithTypeFilter() => Content("TypeFilter");

// 3. ServiceFilter (filter type registered in DI)
[ServiceFilter(typeof(SampleActionFilter))]
public IActionResult WithServiceFilter() => Content("ServiceFilter");
```

**Component breakdown:**

| Approach | Attribute | DI Required? | Parameters? | Lifetime |
|----------|-----------|--------------|-------------|----------|
| Direct | `[MyFilter]` | No | No | New per request (or `IsReusable`) |
| `TypeFilter` | `[TypeFilter(typeof(T))]` | Yes (for dependencies) | Yes (`Arguments`) | New per request (or `IsReusable`) |
| `ServiceFilter` | `[ServiceFilter(typeof(T))]` | Yes (filter registered) | No | As registered (Singleton/Scoped/Transient) |

#### Generic Syntax (.NET 8+)

```csharp
[TypeFilter<SampleActionFilter>]
[ServiceFilter<SampleActionFilter>]
[MiddlewareFilter<FilterMiddlewarePipeline>]
```

**Syntax rules:**

- **Direct attributes** are preferred when the filter has no constructor dependencies.
- **`TypeFilter`** is used when the filter needs constructor arguments or DI-fulfilled dependencies without being registered.
- **`ServiceFilter`** is used when the filter is registered in DI and its lifetime should be managed by the container.
- **Generic attributes** (`.NET 8+`) remove the `typeof()` call and provide compile-time checking.

**Constraints and limitations:**

- **Direct attributes** cannot receive constructor dependencies from DI.
- **`ServiceFilter`** requires the filter to be registered in `IServiceCollection`; otherwise, resolution throws.
- **`TypeFilter`** creates a **new instance per request** unless `IsReusable = true`; it does not respect DI lifetimes for the filter itself.
- **`MiddlewareFilter`** runs middleware as an **async resource filter**—it cannot short-circuit in the same way as an action filter.

### Annotated Complete Step-by-Step Code Example

```csharp
// The filter implementation
public class SampleActionFilter : IActionFilter
{
    private readonly ILogger<SampleActionFilter> _logger;
    private readonly string? _label;

    // Constructor with optional parameter
    public SampleActionFilter(ILogger<SampleActionFilter> logger, string? label = null)
    {
        _logger = logger;
        _label = label;
    }

    public void OnActionExecuting(ActionExecutingContext context)
        => _logger.LogInformation("Before {Label}", _label ?? "default");

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage in a controller
public class FilterFactoryController : Controller
{
    // Direct attribute (no DI, no parameters)
    [SampleActionFilter]
    public IActionResult Direct() => Content("Direct");

    // TypeFilter with constructor arguments
    [TypeFilter(typeof(SampleActionFilter), Arguments = new object[] { "TypeArg" })]
    public IActionResult WithTypeFilter() => Content("TypeFilter");

    // ServiceFilter (filter registered in DI)
    [ServiceFilter(typeof(SampleActionFilter))]
    public IActionResult WithServiceFilter() => Content("ServiceFilter");
}
```

**Step-by-step explanation:**

1. `SampleActionFilter` implements `IActionFilter` and requires `ILogger<T>` plus an optional `label` parameter.
2. **Direct attribute** `[SampleActionFilter]`—the framework attempts to instantiate the filter, but cannot supply `ILogger<T>` or `label`; this only works if the filter has a parameterless constructor or the framework can satisfy its dependencies (which it cannot for arbitrary services without DI).
3. **`TypeFilter`**—creates the filter using `ObjectFactory`, supplying `"TypeArg"` as the constructor argument and resolving `ILogger<T>` from DI. A new instance is created per request.
4. **`ServiceFilter`**—resolves the filter from the DI container. The filter must be registered (e.g., `services.AddScoped<SampleActionFilter>()`), and its lifetime is managed by the container.

**Expected Output:** The `WithTypeFilter` action logs `"Before TypeArg"`; the `WithServiceFilter` action logs `"Before default"` (or the label supplied via DI configuration).

**Why this output?** `TypeFilter` passes the explicit argument; `ServiceFilter` uses the DI-resolved instance, which may have a different configuration based on how it was registered.

### Real-World Cases

- **DI-backed filters:** A logging filter that needs `ILogger<T>` and a configuration value uses `TypeFilter` to pass the value while DI supplies the logger.
- **Lifetime-managed filters:** A caching filter registered as a Singleton uses `ServiceFilter` so the cache is shared across requests.
- **Middleware in filters:** A filter that needs to run a small middleware pipeline (e.g., header manipulation) uses `MiddlewareFilter`.

### References

- Filters in ASP.NET Core (Dependency injection) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- `IFilterFactory` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.ifilterfactory
- Filter factory approaches – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#4

---

## 2. `[ServiceFilter]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[ServiceFilter]` is an attribute that resolves a filter instance from the DI container, allowing the filter’s lifetime to be managed by the container. |
| **Technical Definition** | `ServiceFilterAttribute` implements `IFilterFactory`. Its `CreateInstance(IServiceProvider)` method calls `serviceProvider.GetRequiredService(FilterType)`, so the filter **must be registered** in `IServiceCollection`. The filter’s lifetime (Singleton, Scoped, or Transient) is respected, giving the developer full control over instance reuse. `ServiceFilterAttribute.IsReusable` indicates whether the resolved instance can be reused across requests. |
| **Beginner-Friendly Explanation** | `[ServiceFilter]` says: **“Get this filter from the DI container.”** You must register the filter in `Program.cs`, just like any other service. The benefit is that the container controls the filter’s lifetime—you can make it a singleton, scoped, or transient. If you forget to register it, you get a clear error at runtime. |

### Purposes

- **To** resolve a filter from the DI container, enabling **lifetime management** (Singleton, Scoped, Transient).
- **To** ensure the filter’s dependencies are resolved by the container, following the same lifetime rules as other services.
- **To** allow the filter to be **shared** across requests when registered as a Singleton or Scoped.
- **To** provide a **fail-fast** mechanism: an unregistered filter type throws `InvalidOperationException` at runtime.
- **To** support **generic attribute syntax** (`[ServiceFilter<T>]`) in .NET 8+, improving compile-time safety.

### Syntax Rules and Structure

```csharp
// Registration (required)
builder.Services.AddScoped<SampleActionFilter>();

// Usage (non-generic)
[ServiceFilter(typeof(SampleActionFilter))]
public IActionResult Index() => View();

// Usage (generic, .NET 8+)
[ServiceFilter<SampleActionFilter>]
public IActionResult Index() => View();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FilterType` | The filter type to resolve from DI (required). |
| `IsReusable` | Indicates whether the instance can be reused across requests. |
| Registration | **Mandatory**—the filter must be added to `IServiceCollection`. |

**Syntax rules:**

- The filter type **must be registered** in DI; otherwise, resolution throws `InvalidOperationException`.
- The filter’s registered lifetime determines instance reuse: `AddSingleton` → one instance for the app; `AddScoped` → one per request; `AddTransient` → new per resolution.
- `IsReusable` defaults to `false`; setting it to `true` signals that the instance can be reused across requests (relevant when the filter is created once and cached).
- Generic syntax `[ServiceFilter<T>]` is available in .NET 8+.

**Constraints and limitations:**

- **Cannot pass constructor arguments** directly—use `TypeFilter` if parameters are needed.
- **Must be registered**—unlike `TypeFilter`, forgetting registration causes a runtime error.
- **Filter lifetime is controlled by DI**—if registered as Scoped, the filter is created once per request; if Singleton, once for the app.

### Annotated Complete Step-by-Step Code Example

```csharp
// The filter
public class LogFilter : IActionFilter
{
    private readonly ILogger<LogFilter> _logger;
    private readonly Guid _instanceId = Guid.NewGuid();

    public LogFilter(ILogger<LogFilter> logger) => _logger = logger;

    public void OnActionExecuting(ActionExecutingContext context)
        => _logger.LogInformation("Filter instance {Id} executing", _instanceId);

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Registration — Singleton
builder.Services.AddSingleton<LogFilter>();

// Usage
public class HomeController : Controller
{
    [ServiceFilter(typeof(LogFilter))]
    public IActionResult Index() => View();
}
```

**Step-by-step explanation:**

1. `LogFilter` implements `IActionFilter` and requires `ILogger<T>`. It also assigns a unique `_instanceId` to demonstrate lifetime.
2. The filter is registered as a **Singleton**, so the container creates exactly one instance for the entire application.
3. The `[ServiceFilter]` attribute resolves that single instance every time the `Index` action is called.
4. The logged `_instanceId` is **the same** across all requests, proving that the Singleton lifetime is honoured.

**Expected Output:** Every request to `Index` logs the **same** `Filter instance {Id}` value, because the filter is a Singleton.

**Why this output?** `ServiceFilter` delegates instance creation to the DI container, and `AddSingleton` ensures only one instance exists. In contrast, `TypeFilter` would create a new instance per request, producing a different `_instanceId` each time.

### Real-World Cases

- **Singleton caching filters:** A response-caching filter registered as a Singleton shares a single cache across all requests.
- **Scoped audit filters:** An audit filter that depends on a Scoped `DbContext` is registered as Scoped and resolved per request via `ServiceFilter`.
- **Shared configuration filters:** A filter that reads application-wide configuration once and is reused across all requests.

### References

- `ServiceFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.servicefilterattribute
- Filters in ASP.NET Core (ServiceFilter) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#servicefilterattribute
- ServiceFilter vs TypeFilter (Stack Overflow) – https://stackoverflow.com/questions/43658066/-/43660170#43660170

---

## 3. `[TypeFilter]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[TypeFilter]` is an attribute that creates a filter instance at runtime using `ObjectFactory`, allowing constructor arguments to be passed while still resolving dependencies from DI. |
| **Technical Definition** | `TypeFilterAttribute` implements `IFilterFactory`. Its `CreateInstance(IServiceProvider)` method uses `Microsoft.Extensions.DependencyInjection.ObjectFactory` to instantiate the filter type, passing any `Arguments` supplied via the attribute and fulfilling remaining constructor parameters from the DI container. Unlike `ServiceFilter`, the filter type **does not need to be registered** in `IServiceCollection`. By default, a **new instance is created per request**; `IsReusable` can signal that the instance may be reused across requests. |
| **Beginner-Friendly Explanation** | `[TypeFilter]` says: **“Create this filter for me, here are some constructor arguments, and fill in the rest from DI.”** You don’t need to register the filter in `Program.cs`. It is ideal when the filter needs parameters that vary by action or controller, or when you want a new instance per request without DI registration. |

### Purposes

- **To** create a filter instance **without registering it** in the DI container.
- **To** pass **constructor arguments** directly from the attribute (e.g., a role name, a policy, a threshold).
- **To** resolve the filter’s **other dependencies** from DI (e.g., `ILogger<T>`, configuration services).
- **To** create a **new filter instance per request** by default, avoiding shared state between requests.
- **To** support **generic attribute syntax** (`[TypeFilter<T>]`) in .NET 8+.

### Syntax Rules and Structure

```csharp
// Non-generic
[TypeFilter(typeof(MyFilter), Arguments = new object[] { "arg1", 42 })]
public IActionResult Index() => View();

// Generic (.NET 8+)
[TypeFilter<MyFilter>]
public IActionResult Index() => View();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FilterType` | The filter type to instantiate (required). |
| `Arguments` | Optional array of constructor arguments. |
| `IsReusable` | Indicates whether the instance can be reused across requests (default `false`). |
| Registration | **Not required**—the filter is created by `ObjectFactory`. |

**Syntax rules:**

- The filter type **does not need to be registered** in DI.
- `Arguments` are passed to the filter’s constructor **positionally**.
- Remaining constructor parameters are resolved from DI.
- By default, a **new instance is created per request**; set `IsReusable = true` to allow reuse.
- Generic syntax `[TypeFilter<T>]` is available in .NET 8+ but **does not support `Arguments`**—use the non-generic form when passing parameters.

**Constraints and limitations:**

- **No lifetime management for the filter itself**—the filter is not resolved from DI, so its lifetime is not controlled by `AddSingleton`/`AddScoped`. Use `ServiceFilter` if lifetime management is needed.
- **`Arguments` are positional and order-sensitive**—mismatched order causes runtime errors.
- **`IsReusable`** only signals reuse; it does not guarantee a single instance.

### Annotated Complete Step-by-Step Code Example

```csharp
// The filter with constructor parameters
public class RoleCheckFilter : IActionFilter
{
    private readonly string _requiredRole;
    private readonly ILogger<RoleCheckFilter> _logger;

    public RoleCheckFilter(string requiredRole, ILogger<RoleCheckFilter> logger)
    {
        _requiredRole = requiredRole;
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.HttpContext.User.IsInRole(_requiredRole))
        {
            context.Result = new ForbidResult();
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage with different arguments per action
public class AdminController : Controller
{
    [TypeFilter(typeof(RoleCheckFilter), Arguments = new object[] { "Admin" })]
    public IActionResult AdminOnly() => Content("Admin");

    [TypeFilter(typeof(RoleCheckFilter), Arguments = new object[] { "SuperAdmin" })]
    public IActionResult SuperAdminOnly() => Content("SuperAdmin");
}
```

**Step-by-step explanation:**

1. `RoleCheckFilter` requires a `string requiredRole` and `ILogger<T>`.
2. The filter is **not registered** in DI.
3. `[TypeFilter]` on `AdminOnly` passes `"Admin"` as the first constructor argument; `ILogger<T>` is resolved from DI.
4. `[TypeFilter]` on `SuperAdminOnly` passes `"SuperAdmin"` as the first constructor argument.
5. A new filter instance is created for each request.

**Expected Output:** A user with the `Admin` role can access `AdminOnly`; a user with `SuperAdmin` can access `SuperAdminOnly`; unauthorized users receive 403.

**Why this output?** `TypeFilter` instantiates the filter with the supplied argument and resolves the remaining dependency (`ILogger<T>`) from DI. Different actions can use the same filter type with different parameters.

### Real-World Cases

- **Role/policy-specific filters:** The same `RoleCheckFilter` applied with different role names per action.
- **Threshold-based filters:** A rate-limiting filter parameterized with different limits per endpoint.
- **Feature-flag filters:** A filter that checks whether a named feature is enabled, with the feature name passed as a constructor argument.

### References

- `TypeFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.typefilterattribute
- Filters in ASP.NET Core (TypeFilter) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute
- TypeFilter vs ServiceFilter (Stack Overflow) – https://stackoverflow.com/questions/43658066/servicefilter-and-typefilter-what-is-the-difference-in-injection-those-filters/45437900#45437900

---

## 4. `[MiddlewareFilter]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[MiddlewareFilter]` is an attribute that runs a middleware pipeline as an **async resource filter**, allowing middleware to be applied at the controller or action level. |
| **Technical Definition** | `MiddlewareFilterAttribute` implements `IFilterFactory` and `IOrderedFilter`. It accepts a `ConfigurationType`—a type with a `Configure(IApplicationBuilder)` method that defines the middleware pipeline. The attribute creates a `MiddlewareFilter` instance that **runs the configured middleware as an async resource filter**, executing at the same pipeline stage as resource filters: before model binding and after the rest of the pipeline. The middleware pipeline runs as a single filter, not as separate middleware in the application pipeline. Generic syntax `[MiddlewareFilter<T>]` is available in .NET 8+. |
| **Beginner-Friendly Explanation** | `[MiddlewareFilter]` lets you **reuse a middleware pipeline as a filter**. You write a small configuration class that defines the middleware you want (e.g., adding a header, logging, or short-circuiting), and then apply `[MiddlewareFilter]` to a controller or action. The middleware runs as part of the filter pipeline for that specific controller or action—not globally. |

### Purposes

- **To** run **middleware** at the **controller or action level** without registering it globally in the application pipeline.
- **To** reuse existing middleware logic as a filter, bridging the two extension models.
- **To** apply middleware **conditionally** to specific controllers or actions.
- **To** run middleware as an **async resource filter**, benefiting from the resource filter’s broad pipeline coverage (before model binding, after result execution).
- **To** encapsulate a **focused middleware pipeline** (e.g., response header manipulation, request validation) in a reusable filter attribute.

### Syntax Rules and Structure

#### Defining the Middleware Pipeline

```csharp
public class FilterMiddlewarePipeline
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            context.Response.Headers.Add("Pipeline", "Middleware");
            await next();
        });
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Configure(IApplicationBuilder)` | Defines the middleware pipeline. |
| `app.Use(...)` | Adds middleware to the pipeline. |

#### Applying the Attribute

```csharp
// Non-generic
[MiddlewareFilter(typeof(FilterMiddlewarePipeline))]
public class FilterMiddlewareController : Controller
{
    public IActionResult Index() => Content("Index");
}

// Generic (.NET 8+)
[MiddlewareFilter<FilterMiddlewarePipeline>]
public class FilterMiddlewareController : Controller { ... }
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ConfigurationType` | A type with a `Configure(IApplicationBuilder)` method. |
| Pipeline stage | Runs as an **async resource filter**. |
| Scope | Controller or action level. |

**Syntax rules:**

- The `ConfigurationType` must have a public `Configure(IApplicationBuilder)` method.
- The middleware pipeline is treated as an **async resource filter**—it runs **before model binding** and **after the rest of the pipeline**.
- The attribute can be applied to **controllers or action methods**.
- Generic syntax `[MiddlewareFilter<T>]` is available in .NET 8+.

**Constraints and limitations:**

- **Runs as a resource filter**—it cannot perform action-specific result transformation in the same way as action filters.
- **Short-circuiting behaviour** is that of resource filters—the middleware can terminate the pipeline by not calling `next`.
- **The middleware pipeline is scoped to the filter**, not the application—it does not run for requests that do not reach the decorated controller or action.

### Annotated Complete Step-by-Step Code Example

```csharp
// Middleware configuration
public class HeaderMiddlewarePipeline
{
    public void Configure(IApplicationBuilder app)
    {
        app.Use(async (context, next) =>
        {
            context.Response.Headers.Add("X-Powered-By", "MyApp");
            await next();
        });
    }
}

// Usage on a controller
[MiddlewareFilter(typeof(HeaderMiddlewarePipeline))]
public class PoweredController : Controller
{
    public IActionResult Index() => Content("Powered by MyApp");
}
```

**Step-by-step explanation:**

1. `HeaderMiddlewarePipeline` defines a middleware that adds the `X-Powered-By: MyApp` header.
2. `[MiddlewareFilter]` is applied to `PoweredController`.
3. When any action on `PoweredController` is invoked, the middleware pipeline runs as an async resource filter—adding the header **before model binding** and continuing to the action.
4. Requests to other controllers **do not** receive the header.

**Expected Output:** Responses from `PoweredController` include the `X-Powered-By: MyApp` header; responses from other controllers do not.

**Why this output?** `MiddlewareFilter` runs the configured middleware only for the decorated controller, treating the middleware pipeline as an async resource filter scoped to that controller.

### Real-World Cases

- **Controller-specific headers:** Add custom headers only to API controllers or admin controllers.
- **Request validation middleware:** Run a middleware that validates a specific header or token for a subset of endpoints.
- **Tenant resolution:** Run tenant-detection middleware only for controllers that handle tenant-specific operations.
- **Response enrichment:** Apply middleware that transforms responses (e.g., adding metadata) only for specific actions.

### References

- `MiddlewareFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.middlewarefilterattribute
- Filters in ASP.NET Core (MiddlewareFilter) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#middleware-filter
- Middleware filter runs as async resource filter – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#middleware-filter

---

## 5. Filter Attribute Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filter attribute best practices are guidelines for choosing, applying, and configuring filter attributes to ensure correctness, testability, and maintainability. |
| **Technical Definition** | Best practices encompass choosing the right attribute (`ServiceFilter` for lifetime-managed filters, `TypeFilter` for parameterized filters, `MiddlewareFilter` for middleware reuse), using **generic attributes** (.NET 8+) for compile-time safety, understanding **lifetime implications** (Singleton vs. per-request), avoiding **shared mutable state** in Singleton filters, and preferring **direct attributes** when no DI or parameters are needed. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for filter attributes—they help you avoid runtime errors (forgetting to register a `ServiceFilter`), unexpected state sharing (Singleton filters with mutable state), and over-engineering (using `TypeFilter` when a direct attribute would suffice). |

### Purposes

- **To** choose the **simplest attribute** that meets the requirements (direct → `TypeFilter` → `ServiceFilter`).
- **To** avoid **runtime errors** by understanding when registration is required.
- **To** prevent **thread-safety bugs** in Singleton filters.
- **To** use **generic syntax** where available for compile-time safety and cleaner code.
- **To** keep filter configuration **close to its usage** while respecting lifetime rules.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Attribute choice** | Use a **direct attribute** when the filter has no DI dependencies or parameters. | Use `TypeFilter` or `ServiceFilter` unnecessarily. |
| **`ServiceFilter`** | Register the filter in DI; choose the lifetime deliberately (Singleton, Scoped, Transient). | Forget to register the filter (causes `InvalidOperationException`). |
| **`TypeFilter`** | Use when passing **constructor arguments** or avoiding DI registration. | Use when the filter should have a managed lifetime (use `ServiceFilter` instead). |
| **`MiddlewareFilter`** | Use for **controller/action-specific middleware** reuse. | Use for logic that belongs in a regular action or resource filter. |
| **Generic syntax** | Use `[ServiceFilter<T>]`, `[TypeFilter<T>]`, `[MiddlewareFilter<T>]` (.NET 8+). | Use `typeof()` syntax in new code targeting .NET 8+. |
| **Thread safety** | Keep Singleton filters **stateless** or thread-safe. | Store mutable state in a Singleton filter (shared across requests). |
| **Order** | Use `Order` when specific execution order is required. | Rely on arbitrary ordering between filters from different scopes. |

### Annotated Code Example (Best-Practice Attribute Selection)

```csharp
// ✅ Simplest case: direct attribute (no DI, no parameters)
public class SimpleHeaderAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
        => context.HttpContext.Response.Headers.Add("X-Simple", "true");
}

// ✅ DI-backed filter with lifetime management: ServiceFilter
public class LogFilter : IActionFilter
{
    private readonly ILogger<LogFilter> _logger;
    public LogFilter(ILogger<LogFilter> logger) => _logger = logger;
    public void OnActionExecuting(ActionExecutingContext context)
        => _logger.LogInformation("Executing");
    public void OnActionExecuted(ActionExecutedContext context) { }
}
// Registration: builder.Services.AddScoped<LogFilter>();

// ✅ Parameterized filter: TypeFilter
public class RoleFilter : IActionFilter
{
    private readonly string _role;
    public RoleFilter(string role) => _role = role;
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.HttpContext.User.IsInRole(_role))
            context.Result = new ForbidResult();
    }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage
public class HomeController : Controller
{
    [SimpleHeader]                                          // direct
    public IActionResult A() => View();

    [ServiceFilter(typeof(LogFilter))]                      // DI + lifetime
    public IActionResult B() => View();

    [TypeFilter(typeof(RoleFilter), Arguments = new object[] { "Admin" })] // parameterized
    public IActionResult C() => View();
}
```

**Why this follows best practices:**

- **`SimpleHeader`** is a direct attribute because it has no dependencies or parameters.
- **`LogFilter`** uses `ServiceFilter` because it is registered in DI and its lifetime (Scoped) is managed by the container.
- **`RoleFilter`** uses `TypeFilter` because it requires a constructor argument (`"Admin"`) and is not registered in DI.
- Each attribute choice matches the filter’s requirements—no over-engineering.

### Real-World Cases

- **API controllers:** `ServiceFilter` with a Scoped audit filter; `TypeFilter` with a parameterized rate-limit filter.
- **Admin areas:** `MiddlewareFilter` to add admin-specific headers or run admin-only middleware.
- **Mixed applications:** Direct attributes for simple cross-cutting logic; factory attributes for anything requiring DI or parameters.

### References

- Filters in ASP.NET Core (Dependency injection) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- `ServiceFilterAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.servicefilterattribute
- `TypeFilterAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.typefilterattribute
- `MiddlewareFilterAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.middlewarefilterattribute
- ServiceFilter vs TypeFilter (Stack Overflow) – https://stackoverflow.com/questions/43658066/servicefilter-and-typefilter-what-is-the-difference-in-injection-those-filters/45437900#45437900
- Generic filter attributes in .NET 8 – https://renatogroffe.medium.com/novidades-do-net-8-melhorias-na-configura%C3%A7%C3%A3o-de-apis-rest-no-asp-net-core-com-generic-attributes-8b7c9e5f5a7e

---

## Summary Table: Filter Attributes at a Glance

| Attribute | Primary Role | DI Required? | Parameters? | Lifetime Control | When to Use |
|-----------|--------------|--------------|-------------|-------------------|-------------|
| **Direct attribute** | Simple filter | No | No | Framework-managed | No DI dependencies or parameters |
| **`[ServiceFilter]`** | Resolve filter from DI | Yes (registration) | No | DI-managed (Singleton/Scoped/Transient) | Filter needs DI dependencies and lifetime control |
| **`[TypeFilter]`** | Create filter with arguments | Yes (for dependencies) | Yes (`Arguments`) | New per request (default) | Filter needs constructor arguments or no DI registration |
| **`[MiddlewareFilter]`** | Run middleware as resource filter | No | Configuration type | Runs as async resource filter | Controller/action-specific middleware reuse |
| **Generic syntax** | `[ServiceFilter<T>]`, `[TypeFilter<T>]`, `[MiddlewareFilter<T>]` | Same as above | Same as above | Same as above | .NET 8+ projects; compile-time safety |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- `ServiceFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.servicefilterattribute
- `TypeFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.typefilterattribute
- `MiddlewareFilterAttribute` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.middlewarefilterattribute
- `IFilterFactory` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.ifilterfactory
- Filters in ASP.NET Core (Dependency injection) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#dependency-injection
- Filter factory approaches (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#4
- Middleware filter runs as async resource filter – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#middleware-filter
- ServiceFilter vs TypeFilter (Stack Overflow) – https://stackoverflow.com/questions/43658066/servicefilter-and-typefilter-what-is-the-difference-in-injection-those-filters/45437900#45437900
- Generic filter attributes in .NET 8 – https://renatogroffe.medium.com/novidades-do-net-8-melhorias-na-configura%C3%A7%C3%A3o-de-apis-rest-no-asp-net-core-com-generic-attributes-8b7c9e5f5a7e