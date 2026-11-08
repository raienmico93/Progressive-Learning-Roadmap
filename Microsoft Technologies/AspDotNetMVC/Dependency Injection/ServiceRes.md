# ASP.NET Core Service Resolution: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the built-in DI container (`Microsoft.Extensions.DependencyInjection`). It covers the mechanisms for **resolving** services from the container—`IServiceProvider`, `GetService()`, `GetRequiredService()`, and `[FromServices]`—along with resolution best practices. Registration is covered in a separate cheat sheet. The APIs are stable; scope-validation behaviour differs between Development and Production environments and is flagged where relevant. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Service resolution is the process of obtaining a constructed service instance from the DI container. |
| **Technical Definition** | Resolution is performed against an `IServiceProvider` (the container). The built-in container exposes two primary resolution methods: `GetService(Type)` (returns `null` if the service is not registered) and the generic extension `GetRequiredService<T>()` (throws `InvalidOperationException` if the service is not registered). In ASP.NET Core, services are resolved automatically for controller constructors, Razor Page models, and middleware `Invoke`/`InvokeAsync` parameters. `[FromServices]` enables **action-level** resolution without constructor injection. Resolution respects service lifetimes and scope validation. |
| **Beginner-Friendly Explanation** | Service resolution is **asking the DI container for something you need**. Instead of writing `new MyService()`, you say “give me the `IMyService`” and the container hands it to you—already constructed, with all its own dependencies filled in. The container knows whether to create a fresh instance, reuse a per-request one, or return the shared singleton. |

### Key Characteristics

- **Constructor injection is the default** – The framework resolves constructor parameters automatically for controllers and page models.
- **Two resolution methods** – `GetService` (nullable) and `GetRequiredService` (throws).
- **Action-level injection** – `[FromServices]` resolves services directly into action method parameters.
- **Scope-aware** – Scoped services must be resolved from a scope (per-request or explicit); the root provider rejects them when validation is enabled.
- **Extensible** – Third-party containers implement the same `IServiceProvider` contract.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- A working understanding of **service registration** and **service lifetimes**.
- Familiarity with **constructor injection** in controllers or Razor Pages.

### Related Programming Areas

- **Service registration** – Services must be registered before they can be resolved.
- **Service lifetimes** – Determines whether resolution creates, reuses, or rejects an instance.
- **Dependency injection** – The overarching pattern that resolution implements.
- **Middleware** – Scoped services are resolved in `InvokeAsync`, not the middleware constructor.
- **Minimal APIs** – Handler parameters are resolved from DI automatically (with `[FromServices]` or inference).

### Core Concepts / Features

1. **`IServiceProvider`** – The container interface used for resolution.
2. **`GetService()`** – Nullable resolution method.
3. **`GetRequiredService()`** – Required resolution method (throws on failure).
4. **`[FromServices]`** – Action-level service injection attribute.
5. **Service resolution best practices** – Avoiding service locator, respecting scopes, and choosing the right method.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. `IServiceProvider`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IServiceProvider` is the interface that represents the DI container and exposes the `GetService` method for resolving services. |
| **Technical Definition** | `IServiceProvider` (in `System`) declares a single method: `object? GetService(Type serviceType)`. In ASP.NET Core, the built-in container implements this interface and also provides extension methods in `Microsoft.Extensions.DependencyInjection` (`GetService<T>()`, `GetRequiredService<T>()`). The provider is created from an `IServiceCollection` via `BuildServiceProvider()` (or automatically during host startup). `HttpContext.RequestServices` exposes the **scoped** provider for the current request. |
| **Beginner-Friendly Explanation** | `IServiceProvider` is the **“service counter”** of your app. You walk up to the counter and ask for a service by its type. The counter either hands you an instance or tells you it is not available. The counter also knows whether to give you a new instance, a per-request instance, or the same shared instance every time. |

### Purposes

- **To** provide a standard, container-agnostic contract for resolving services.
- **To** expose the `GetService` method that all DI containers implement.
- **To** serve as the foundation for extension methods like `GetRequiredService<T>()`.
- **To** enable explicit scope creation and resolution in background tasks.
- **To** support third-party containers that implement the same interface.

### Syntax Rules and Structure

#### General Syntax (Accessing the Provider)

```csharp
// Root provider (application-level)
IServiceProvider rootProvider = app.Services;

// Scoped provider for the current request
IServiceProvider requestProvider = HttpContext.RequestServices;

// Explicit scope (for background work)
using IServiceScope scope = app.Services.CreateScope();
IServiceProvider scopedProvider = scope.ServiceProvider;
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `app.Services` | The root `IServiceProvider`. |
| `HttpContext.RequestServices` | The per-request scoped provider. |
| `CreateScope()` | Creates an explicit scope with its own provider. |

#### General Syntax (Interface Definition)

```csharp
public interface IServiceProvider
{
    object? GetService(Type serviceType);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GetService` | The single method; returns `null` if not found. |
| `serviceType` | The type to resolve. |

#### Syntax Rules

- `IServiceProvider` itself is **not generic**; generic resolution comes from extension methods.
- The **root provider** is for Singletons; Scoped services must be resolved from a scope.
- `HttpContext.RequestServices` is the correct provider for per-request Scoped services.
- `IServiceProvider` can be injected into a class, but doing so to resolve services manually is the **service locator anti-pattern**.

#### Constraints and Limitations

- **Root provider rejects Scoped services** when `ValidateScopes` is `true` (default in Development). Resolving a Scoped service from `app.Services` throws: *“Cannot resolve scoped service from root provider.”*
- **`IServiceProvider` is not a service locator** – injecting it to call `GetService` inside business logic is discouraged.
- **Disposal** – Do not dispose services resolved from the container; the container manages disposal according to lifetime.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Resolving from the Correct Scope

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IMyScopedService, MyScopedService>();
var app = builder.Build();

// CORRECT: create an explicit scope for background work
using (var scope = app.Services.CreateScope())
{
    var scoped = scope.ServiceProvider
        .GetRequiredService<IMyScopedService>();
    scoped.DoWork();
}

// INCORRECT: resolving a scoped service from the root provider
// var bad = app.Services.GetRequiredService<IMyScopedService>(); // throws
```

**Step-by-step explanation:**

1. A Scoped service is registered.
2. An explicit scope is created with `CreateScope()`.
3. The service is resolved from `scope.ServiceProvider`, which is a scoped provider.
4. The scope is disposed, disposing the Scoped service.

**Expected Output:** The Scoped service is resolved successfully from the explicit scope. Resolving from `app.Services` throws an `InvalidOperationException` (when scope validation is enabled).

**Why this output?** Scoped services are tied to a scope; the root provider is not a scope and rejects them.

---

#### Example 2: `HttpContext.RequestServices` for Per-Request Resolution

```csharp
app.Use(async (context, next) =>
{
    var scoped = context.RequestServices
        .GetRequiredService<IMyScopedService>();
    scoped.DoWork();
    await next();
});
```

**Expected Output:** The Scoped service is resolved from the per-request scope, shared with the rest of the request pipeline.

**Why this output?** `RequestServices` exposes the scoped provider created for the current request.

### Real-World Cases

- **Background services:** `IServiceScopeFactory.CreateScope()` provides a scoped provider for resolving Scoped services outside a request.
- **Middleware:** Scoped services are resolved from `context.RequestServices` or via `InvokeAsync` parameters.
- **Startup tasks:** A short-lived explicit scope resolves Scoped services during app startup.

### References

- `IServiceProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- Dependency injection in ASP.NET Core (Request Services) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Service lifetimes (scope validation) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes

---

## 2. `GetService()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `GetService()` resolves a service from the container and returns `null` if the service is not registered. |
| **Technical Definition** | `GetService(Type)` is the single method on `IServiceProvider`. The generic extension `GetService<T>()` (in `Microsoft.Extensions.DependencyInjection`) calls the non-generic method and casts the result. Unlike `GetRequiredService`, it does **not** throw when the service is missing; it returns `null`. This makes it suitable for **optional** dependencies. |
| **Beginner-Friendly Explanation** | `GetService` is the **“ask nicely”** method. You ask the container for a service, and if it does not have it, you get `null` back—no exception, no crash. It is useful when a dependency is genuinely optional. |

### Purposes

- **To** resolve services that may or may not be registered, without throwing.
- **To** support optional dependencies (e.g., a plugin service that may be absent).
- **To** provide the base method that `GetRequiredService` builds upon.
- **To** enable conditional logic when a service’s presence determines behaviour.

### Syntax Rules and Structure

```csharp
// Non-generic
object? service = provider.GetService(typeof(IMyService));

// Generic extension
IMyService? service = provider.GetService<IMyService>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GetService(typeof(T))` | Non-generic call; returns `object?`. |
| `GetService<T>()` | Generic extension; returns `T?`. |
| Return value | `null` if the service is not registered. |

**Syntax rules:**

- Available on any `IServiceProvider`.
- Does **not** throw for unregistered services.
- Still respects lifetimes and scopes (a Scoped service from the root provider throws).

**Constraints and limitations:**

- **Null handling required** – The caller must check for `null` before using the result.
- **Not recommended for required dependencies** – Silent `null` can hide misconfiguration; use `GetRequiredService` when the service must exist.
- **Service locator risk** – Calling `GetService` inside business logic is an anti-pattern.

### Annotated Code Example

```csharp
public class NotificationService
{
    private readonly IEmailSender? _emailSender;

    public NotificationService(IServiceProvider provider)
    {
        // Optional dependency: may not be registered
        _emailSender = provider.GetService<IEmailSender>();
    }

    public void Notify(string message)
    {
        if (_emailSender != null)
        {
            _emailSender.Send(message);
        }
        else
        {
            Console.WriteLine("Email sender not configured.");
        }
    }
}
```

**Expected Output:** If `IEmailSender` is registered, the email is sent; otherwise, a fallback message is logged. No exception is thrown.

**Why this output?** `GetService` returns `null` for the unregistered `IEmailSender`, and the service handles the absence gracefully.

### Real-World Cases

- **Optional plugins:** A feature that integrates with an external service only if that service is registered.
- **Diagnostics:** Checking for the presence of a diagnostics provider before using it.
- **Feature flags:** Conditionally resolving a service that is only registered when a feature is enabled.

### References

- `IServiceProvider.GetService` method – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider.getservice
- `ServiceProviderServiceExtensions.GetService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getservice
- GetService vs GetRequiredService – https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/

---

## 3. `GetRequiredService()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `GetRequiredService<T>()` resolves a service from the container and throws an exception if the service is not registered. |
| **Technical Definition** | `GetRequiredService<T>()` is an extension method in `Microsoft.Extensions.DependencyInjection`. It calls `GetService` internally and throws `InvalidOperationException` if the result is `null`. This makes it the preferred method for **required** dependencies, because misconfiguration fails fast with a clear error rather than producing a silent `null` later. |
| **Beginner-Friendly Explanation** | `GetRequiredService` is the **“I need this, no excuses”** method. If the container does not have the service, it throws an exception immediately, telling you exactly what is missing. It is safer than `GetService` when the service is not optional. |

### Purposes

- **To** resolve services that must be present, failing fast on misconfiguration.
- **To** provide a clear `InvalidOperationException` when a required service is missing.
- **To** replace `GetService` in scenarios where a `null` result would be a bug.
- **To** simplify code by eliminating `null` checks for required dependencies.

### Syntax Rules and Structure

```csharp
IMyService service = provider.GetRequiredService<IMyService>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GetRequiredService<T>()` | Generic extension method. |
| Return value | The resolved service; never `null` for a registered service. |
| Exception | `InvalidOperationException` if the service is not registered. |

**Syntax rules:**

- Available on any `IServiceProvider`.
- Throws if the service is not registered.
- Respects lifetimes and scopes (Scoped service from root provider throws).

**Constraints and limitations:**

- **Throws on missing services** – Only use it when the service is genuinely required.
- **Service locator risk** – Same anti-pattern caveat as `GetService`.
- **Exception type** – `InvalidOperationException`, not `NullReferenceException`.

### Annotated Code Example

```csharp
public class OrderProcessor
{
    private readonly IOrderRepository _repository;

    public OrderProcessor(IServiceProvider provider)
    {
        _repository = provider.GetRequiredService<IOrderRepository>();
    }

    public void Process(int orderId)
    {
        var order = _repository.Get(orderId);
        // ...
    }
}
```

**Expected Output:** If `IOrderRepository` is registered, it is resolved successfully. If it is not registered, an `InvalidOperationException` is thrown at construction time with a message like *“No service for type 'IOrderRepository' has been registered.”*

**Why this output?** `GetRequiredService` throws immediately when the service is missing, surfacing the misconfiguration early.

### Real-World Cases

- **Critical infrastructure:** Resolving a repository, logger, or configuration service that the application cannot function without.
- **Startup validation:** `ValidateOnBuild` and `GetRequiredService` together ensure required services are registered before the app starts serving requests.
- **Background services:** Resolving Scoped services from an explicit scope where the service is mandatory.

### References

- `ServiceProviderServiceExtensions.GetRequiredService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice
- GetService vs GetRequiredService – https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/

---

## 4. `[FromServices]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[FromServices]` is an attribute that injects a service directly into an **action method parameter** (or a controller property with a public setter) without requiring constructor injection. |
| **Technical Definition** | `FromServicesAttribute` implements `IFromServiceMetadata` and tells MVC’s parameter binding system to resolve the parameter from the DI container rather than from route/query/form data. It enables **action-level injection**, where a dependency is needed by only one action in a controller. In minimal APIs, `[FromServices]` works similarly, and inference may resolve service-typed parameters automatically. |
| **Beginner-Friendly Explanation** | `[FromServices]` lets a **single action method** ask for a service without making the whole controller depend on it. It is useful when only one action needs a particular service, keeping the controller’s constructor lean. |

### Purposes

- **To** inject a service into a single action method without adding it to the controller constructor.
- **To** keep controller constructors focused on dependencies used across all actions.
- **To** support action-specific dependencies (e.g., a service used only by a “download” action).
- **To** enable property injection on controllers (with a public setter), though this is less common.

### Syntax Rules and Structure

```csharp
public IActionResult MyAction([FromServices] IMyService service)
{
    service.DoWork();
    return Ok();
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[FromServices]` | Marks the parameter for DI resolution. |
| `IMyService service` | The service type; resolved from the container. |

#### Property Injection

```csharp
public class MyController : Controller
{
    [FromServices]
    public IMyService MyService { get; set; }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[FromServices]` on a property | Requires a public setter. |
| `MyService` | Resolved from the container when the controller is created. |

#### Syntax Rules

- `[FromServices]` is supported on **action method parameters** and **controller properties with public setters**.
- In minimal APIs, service-typed parameters may be inferred as `[FromServices]`; use the attribute explicitly to be clear.
- **Action filters** can also receive services via `[FromServices]` in their constructor (filters are resolved from DI).

#### Constraints and Limitations

- **Property injection is not recommended** – Constructor injection is preferred; `[FromServices]` on properties is supported but discouraged for required dependencies.
- **Not a replacement for constructor injection** – If the service is used by multiple actions, inject it via the constructor instead.
- **Minimal API inference** – In some configurations, service parameters may be inferred automatically; explicit `[FromServices]` removes ambiguity.

### Annotated Code Example

```csharp
public class ReportsController : Controller
{
    private readonly IReportGenerator _generator;

    // Constructor injection: used by all actions
    public ReportsController(IReportGenerator generator)
    {
        _generator = generator;
    }

    // Action-level injection: only this action needs IAuditLogger
    public IActionResult Download(int id, [FromServices] IAuditLogger audit)
    {
        audit.Log($"Report {id} downloaded.");
        var report = _generator.Generate(id);
        return File(report, "application/pdf");
    }
}
```

**Expected Output:** The `Download` action receives both the constructor-injected `IReportGenerator` and the action-level `IAuditLogger`. Other actions in the controller are unaffected by `IAuditLogger`.

**Why this output?** `[FromServices]` resolves the `IAuditLogger` only for the `Download` action, keeping the controller constructor focused on shared dependencies.

### Real-World Cases

- **Single-action dependencies:** A logging or auditing service used by only one action.
- **Conditional features:** A service needed only by a specific endpoint.
- **Minimal APIs:** Marking service parameters explicitly in handler delegates.

### References

- Dependency injection into controllers in ASP.NET Core (Action injection with `FromServices`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- `FromServicesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute

---

## 5. Service Resolution Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Service resolution best practices are guidelines for obtaining services in a way that is testable, scope-correct, and free of anti-patterns. |
| **Technical Definition** | Best practices encompass preferring **constructor injection** over manual resolution, avoiding the **service locator pattern**, using `GetRequiredService` for required dependencies and `GetService` only for optional ones, respecting lifetimes (never resolving Scoped services from Singletons), creating explicit scopes for background work, and not disposing container-managed services. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for resolution—they help you avoid hidden `null` bugs, captive dependencies, and code that is hard to test because it reaches into the container instead of declaring what it needs. |

### Purposes

- **To** keep classes testable by making dependencies explicit in constructors.
- **To** prevent captive dependencies and scope-related bugs.
- **To** avoid the service locator anti-pattern.
- **To** fail fast on missing required services.
- **To** ensure services are disposed by the container, not by application code.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Injection style** | Use constructor injection for required dependencies. | Inject `IServiceProvider` and call `GetService` inside business logic (service locator). |
| **Required vs. optional** | Use `GetRequiredService<T>()` for required dependencies. | Use `GetService<T>()` for required dependencies and risk silent `null`. |
| **Optional dependencies** | Use `GetService<T>()` and handle `null`. | Throw or assume presence of optional services. |
| **Scoped services** | Resolve Scoped services from a scope (request scope or explicit `CreateScope()`). | Resolve Scoped services from the root provider or from a Singleton. |
| **Singletons** | Keep Singletons free of Scoped dependencies. | Capture a Scoped service in a Singleton (captive dependency). |
| **Disposal** | Let the container dispose services it creates. | Manually dispose services resolved from the container. |
| **Middleware** | Resolve Scoped services in `InvokeAsync`, not the constructor. | Inject Scoped services into middleware constructors. |
| **Action-specific needs** | Use `[FromServices]` for single-action dependencies. | Add action-only dependencies to the controller constructor. |

### Annotated Code Example (Best-Practice Resolution)

```csharp
// GOOD: constructor injection for a required dependency
public class OrdersController : Controller
{
    private readonly IOrderRepository _repository;

    public OrdersController(IOrderRepository repository)
    {
        _repository = repository;
    }

    public IActionResult Index()
    {
        var orders = _repository.GetAll();
        return View(orders);
    }
}

// GOOD: action-level injection for an action-specific dependency
public IActionResult Download(int id, [FromServices] IAuditLogger audit)
{
    audit.Log($"Order {id} downloaded.");
    return File(/* ... */);
}

// GOOD: explicit scope for background work with a Scoped service
using (var scope = app.Services.CreateScope())
{
    var scoped = scope.ServiceProvider
        .GetRequiredService<IMyScopedService>();
    scoped.DoWork();
}
```

**Why this follows best practices:**

- **Constructor injection** makes `OrdersController` testable and its dependencies explicit.
- **`[FromServices]`** keeps `IAuditLogger` out of the controller constructor because only `Download` needs it.
- **`GetRequiredService`** inside an explicit scope ensures the Scoped service is resolved correctly and fails fast if missing.
- **No service locator** – the controller does not inject `IServiceProvider` to resolve dependencies manually.

### Real-World Cases

- **Testable controllers:** Constructor injection lets unit tests pass mock repositories without a DI container.
- **Background processing:** `IServiceScopeFactory` creates scopes for Scoped services in hosted services.
- **Middleware with Scoped dependencies:** Resolution happens in `InvokeAsync` parameters or `context.RequestServices`, not the constructor.

### References

- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Service lifetimes (avoid resolving scoped from singleton) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection in ASP.NET Core (avoid service locator) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- GetService vs GetRequiredService – https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **`IServiceProvider`** | The container interface for resolution | `app.Services`, `RequestServices`, `CreateScope()` | Explicit scope creation, framework integration |
| **`GetService()`** | Nullable resolution | `provider.GetService<T>()` | Optional dependencies that may not be registered |
| **`GetRequiredService()`** | Required resolution (throws) | `provider.GetRequiredService<T>()` | Required dependencies; fail-fast on misconfiguration |
| **`[FromServices]`** | Action-level injection | `[FromServices] IMyService svc` | Single-action dependencies; keep constructors lean |
| **Best practices** | Testability, scope correctness, anti-pattern avoidance | Constructor injection, explicit scopes, `GetRequiredService` for required | Every production DI setup |

---

## References (Full List)

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Service lifetimes (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Dependency injection into controllers in ASP.NET Core (Action injection with `FromServices`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- `IServiceProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- `IServiceProvider.GetService` method – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider.getservice
- `ServiceProviderServiceExtensions.GetService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getservice
- `ServiceProviderServiceExtensions.GetRequiredService<T>` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice
- `FromServicesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute
- GetService vs GetRequiredService – https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/