# ASP.NET Core DI Injection Patterns: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and its built-in DI container (`Microsoft.Extensions.DependencyInjection`). It covers the three injection patterns—**constructor injection**, **property injection**, and **method injection**—with a dedicated section on constructor injection best practices. A critical distinction must be made upfront: **the built-in ASP.NET Core DI container natively supports constructor injection and method injection (via `[FromServices]`), but does not natively support property injection** for arbitrary services. Property injection is only supported on **controller properties** via `[FromServices]` with a public setter; for general property injection, a third-party container (Autofac, etc.) is required. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Injection patterns describe **how** a class receives its dependencies from the DI container: through its constructor, through properties, or through method parameters. |
| **Technical Definition** | Constructor injection passes dependencies as constructor parameters at object creation time. Property injection assigns dependencies to public properties after construction (or during activation). Method injection passes dependencies as method arguments when the method is invoked. ASP.NET Core’s built-in container implements **constructor injection** for all resolved services and **method injection** for controller action parameters (`[FromServices]`) and middleware `InvokeAsync` parameters; **property injection** is limited to controller properties with `[FromServices]` and is not a general container feature. |
| **Beginner‑Friendly Explanation** | Injection patterns are **different ways of handing a class what it needs**. Constructor injection gives dependencies at birth (when the object is created). Property injection sets them afterward like plugging in accessories. Method injection provides them only when a specific method runs. Constructor injection is the default and most recommended approach because it makes dependencies explicit and guarantees they are present before the object is used. |

### Key Characteristics

- **Constructor injection is the gold standard** – It is the industry standard and most recommended pattern.
- **Built-in container limitations** – Native support exists for constructor and method injection; property injection requires workarounds or third-party containers.
- **Explicit dependencies** – Constructor injection forces all required dependencies to be declared upfront, ensuring object integrity.
- **Action-scoped injection** – `[FromServices]` enables per-action method injection without polluting the controller constructor.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working understanding of **service registration** and **service lifetimes**.
- Familiarity with controllers, Razor Pages, or middleware.

### Related Programming Areas

- **Service resolution** – How services are obtained from the container.
- **Service lifetimes** – Determines instance reuse and scope correctness.
- **Dependency injection guidelines** – Best practices for avoiding captive dependencies and service locator anti-patterns.

### Core Concepts / Features

1. **Constructor injection** – The default, recommended pattern.
2. **Property injection** – Limited native support; requires third-party containers for general use.
3. **Method injection** – Action-level injection via `[FromServices]`; middleware `InvokeAsync` injection.
4. **Constructor injection best practices** – Guidelines for designing constructors and avoiding common pitfalls.

---

## 1. Constructor Injection

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Constructor injection passes a class’s dependencies as parameters to its constructor, so the dependencies are available before the object is used. |
| **Technical Definition** | In ASP.NET Core, constructor injection is the primary DI pattern. Services are added as constructor parameters, and the runtime resolves them from the service container. Services are typically defined using interfaces. The runtime uses `DefaultControllerActivator` (or the equivalent for other component types) to create instances and resolve constructor parameters from the DI container. |
| **Beginner‑Friendly Explanation** | Constructor injection means **“tell me what I need when you create me.”** A `HomeController` that needs a clock declares `public HomeController(IClock clock)` in its constructor. The DI container reads that constructor, finds an `IClock` implementation, and passes it in. The controller can never be created without its dependencies, so it can never be in a half‑initialized state. |

### Purposes

- **To** make a class’s dependencies **explicit and unavoidable**—the object cannot be constructed without them.
- **To** guarantee object integrity—all required dependencies are present before any method is called.
- **To** enable unit testing by allowing mock dependencies to be passed through the constructor.
- **To** enforce the **Explicit Dependencies Principle**—classes do not hide their needs behind service locator calls.
- **To** serve as the default injection pattern for controllers, Razor Page models, services, and middleware components.

### Syntax Rules and Structure

#### General Syntax

```csharp
public class MyController : Controller
{
    private readonly IDependency _dependency;

    public MyController(IDependency dependency)
    {
        _dependency = dependency;
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor parameter | The dependency is requested explicitly. |
| `private readonly` field | Stores the injected instance for the lifetime of the object. |
| Interface type | Dependencies are typically defined using interfaces. |

#### General Syntax (Registration)

```csharp
builder.Services.AddScoped<IDependency, Dependency>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddScoped` | Lifetime selection (Transient, Scoped, or Singleton). |
| `IDependency` | Service type (interface). |
| `Dependency` | Implementation type. |

#### Syntax Rules

- **Constructor injection is the default** for controllers, Razor Page models, services, and middleware (except middleware `Invoke`/`InvokeAsync`).
- **Services are typically defined using interfaces** to enable replacement and testing.
- The constructor must be **public**; the container uses it to activate the type.
- Multiple dependencies can be requested; the container resolves each in order.

#### Constraints and Limitations

- **Middleware constructors cannot receive Scoped services**—Scoped services must be injected into `InvokeAsync` parameters instead.
- **Controllers are not resolved as services by default**—the runtime uses `DefaultControllerActivator` to create them and resolves their constructor parameters, but the controller itself is not a registered service unless `AddControllersAsServices()` is called.
- **Constructor injection is mandatory for required dependencies**—there is no way to omit a constructor parameter.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Constructor Injection in a Controller

```csharp
// Abstraction
public interface IDateTime
{
    DateTime Now { get; }
}

// Implementation
public class SystemDateTime : IDateTime
{
    public DateTime Now => DateTime.Now;
}

// Registration
builder.Services.AddSingleton<IDateTime, SystemDateTime>();
builder.Services.AddControllersWithViews();

// Controller with constructor injection
public class HomeController : Controller
{
    private readonly IDateTime _dateTime;

    public HomeController(IDateTime dateTime)
    {
        _dateTime = dateTime;
    }

    public IActionResult Index()
    {
        var serverTime = _dateTime.Now;
        ViewData["Message"] = serverTime.Hour < 12
            ? "Good Morning!"
            : "Good Afternoon!";
        return View();
    }
}
```

**Step‑by‑step explanation:**

1. `IDateTime` abstracts the concept of “current time.”
2. `SystemDateTime` implements it using `DateTime.Now`.
3. The service is registered as a Singleton.
4. `HomeController` requests `IDateTime` in its constructor.
5. The runtime resolves `SystemDateTime` from the container and injects it.
6. `Index` uses the injected service to choose a greeting.

**Expected Output:** A greeting message based on the server’s current hour (morning vs. afternoon).

**Why this output?** The controller never creates `SystemDateTime` itself; it depends on the abstraction, and the container supplies the configured implementation at construction time.

### Real‑World Cases

- **Testable controllers:** Constructor injection allows unit tests to pass a fixed‑time stub for `IDateTime`.
- **Swappable implementations:** Development uses a fake email sender; production uses the real SMTP sender—both injected via the same constructor.
- **Cross‑cutting services:** `ILogger<T>`, `IConfiguration` (via `IOptions<T>`), and repositories are injected into controllers and services throughout the framework.

### References

- Dependency injection into controllers in ASP.NET Core (Constructor injection) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

---

## 2. Property Injection

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Property injection assigns dependencies to a class’s public properties after construction (or during activation), rather than through the constructor. |
| **Technical Definition** | Property injection sets dependencies on properties, often marked with an attribute (e.g., `[Inject]`, `[FromServices]`). **The built‑in ASP.NET Core DI container does not support property injection for general services**; it intentionally keeps the container simple and follows the explicit dependencies principle. The only native form of property injection in ASP.NET Core is `[FromServices]` on **controller properties with public setters**, resolved during model binding. For general property injection, a third‑party container (Autofac, etc.) must be used. |
| **Beginner‑Friendly Explanation** | Property injection is like **plugging in accessories after the object is built**. Instead of passing everything through the constructor, you leave a public property and the container (or framework) fills it in later. The built‑in ASP.NET Core container mostly refuses to do this for arbitrary services—it wants dependencies to be explicit in constructors. The exception is controller properties decorated with `[FromServices]`. |

### Purposes

- **To** inject dependencies into properties when constructor injection is impractical (e.g., framework‑created types).
- **To** provide a native mechanism for **controller property injection** via `[FromServices]` when a dependency is needed by multiple actions but not in the constructor.
- **To** support third‑party containers that offer richer property injection features (Autofac, etc.).
- **To** enable property‑level needs in specific frameworks (e.g., `[Inject]` in Blazor components, which is a different mechanism).

### Syntax Rules and Structure

#### General Syntax (Controller Property Injection — Native)

```csharp
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [FromServices]
    public required ILogger<WeatherForecastController> Logger { get; set; }

    [FromServices]
    public required IWeatherService WeatherService { get; set; }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        Logger.LogInformation("Fetching weather forecast");
        return WeatherService.GetForecast();
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[FromServices]` | Marks the property for DI resolution. |
| `public required` | The property must have a public setter. |
| Property type | The service type (interface or class). |

**Syntax rules (native):**

- `[FromServices]` on **controller properties** is supported **with public setters**.
- The runtime resolves and populates these properties from the DI container during model binding.
- **This is not a general container feature**—it is specific to MVC controllers.

**Constraints and limitations (native):**

- **Not supported for general services** resolved through the DI container.
- **No `[Inject]` attribute** for general property injection in ASP.NET Core (the `[Inject]` attribute is for Blazor components and Razor views via `@inject`).
- **Third‑party containers required** for general property injection (Autofac, Quickwire, etc.).

### Annotated Code Example

```csharp
public class ReportsController : Controller
{
    // Constructor injection for shared dependencies
    private readonly IReportGenerator _generator;

    public ReportsController(IReportGenerator generator)
    {
        _generator = generator;
    }

    // Property injection for a dependency used across multiple actions
    [FromServices]
    public required IAuditLogger Audit { get; set; }

    public IActionResult Download(int id)
    {
        Audit.Log($"Report {id} downloaded.");
        var report = _generator.Generate(id);
        return File(report, "application/pdf");
    }

    public IActionResult Preview(int id)
    {
        Audit.Log($"Report {id} previewed.");
        var report = _generator.Generate(id);
        return File(report, "application/pdf");
    }
}
```

**Expected Output:** The `Audit` property is populated by the container before any action runs. Both `Download` and `Preview` can use `Audit` without adding it to the constructor.

**Why this output?** `[FromServices]` on the controller property instructs MVC’s model binding to resolve `IAuditLogger` and assign it to the property. This is the only native property injection scenario in ASP.NET Core.

### Real‑World Cases

- **Controller‑wide audit logging:** A property‑injected `IAuditLogger` used by multiple actions in a controller, without constructor clutter.
- **Third‑party containers:** Applications using Autofac for advanced property injection (e.g., optional dependencies, property‑based configuration).
- **Blazor components:** `[Inject]` property injection in `.razor` components (a separate mechanism from MVC property injection).

### References

- Dependency injection into controllers in ASP.NET Core (Property injection with `FromServices`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- Built‑in DI does not support property injection (Stack Overflow discussion) – https://stackoverflow.com/questions/53884417/net-core-di-property-injection
- ASP.NET Core dependency injection best practices (property injection limitations) – https://volosoft.com/blog/aspnet-core-dependency-injection-best-practices

---

## 3. Method Injection

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Method injection passes dependencies as arguments to a method when that method is invoked, rather than storing them as fields or properties. |
| **Technical Definition** | In ASP.NET Core, method injection is implemented for **controller action parameters** via `[FromServices]` and for **middleware `Invoke`/`InvokeAsync` parameters**. The `FromServicesAttribute` enables injecting a service directly into an action method without using constructor injection. The runtime resolves the parameter from the DI container at invocation time. For middleware, Scoped services must be injected as `InvokeAsync` parameters because the middleware constructor runs only once (Singleton‑like). |
| **Beginner‑Friendly Explanation** | Method injection means **“give me what I need only when this specific method runs.”** Instead of making the whole controller depend on a service, a single action method can declare `[FromServices] IAuditLogger audit` as a parameter. The container supplies it just for that action. This keeps the controller’s constructor clean and limits the dependency’s exposure to the one method that needs it. |

### Purposes

- **To** limit dependency exposure to a single action or method (minimal dependency surface).
- **To** enable **lifecycle isolation**—Scoped or Transient dependencies that must not be captured in a Singleton middleware constructor can be method‑injected into `InvokeAsync`.
- **To** improve testability by allowing each test to inject only the dependencies relevant to the method under test.
- **To** avoid constructor clutter when a dependency is used by only one action.

### Syntax Rules and Structure

#### General Syntax (Controller Action Injection)

```csharp
public IActionResult About([FromServices] IDateTime dateTime)
{
    return Content($"Current server time: {dateTime.Now}");
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[FromServices]` | Marks the parameter for DI resolution. |
| `IDateTime dateTime` | The service type and parameter name. |

#### General Syntax (Middleware `InvokeAsync` Injection)

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, IMyScopedService scoped)
    {
        scoped.DoWork();
        await _next(context);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor | Receives only Singleton services (`RequestDelegate`). |
| `InvokeAsync` parameter | Scoped services are injected here. |

#### Syntax Rules

- `[FromServices]` works on **action method parameters** and **minimal API handler parameters**.
- Middleware **must not** receive Scoped services in its constructor; use `InvokeAsync` parameters instead.
- The runtime resolves method‑injected services at invocation time, respecting lifetimes.

#### Constraints and Limitations

- **Method injection is not a general replacement for constructor injection**—if a dependency is used by multiple methods, constructor injection is preferred.
- **Property injection on action parameters is not supported**—`[FromServices]` is for method parameters and (limited) controller properties.
- **Middleware method injection only works for `Invoke`/`InvokeAsync` parameters**, not for helper methods.

### Annotated Complete Step‑by‑Step Code Examples

#### Example 1: Action‑Level Method Injection

```csharp
public class ReportsController : Controller
{
    private readonly IReportGenerator _generator;

    public ReportsController(IReportGenerator generator)
    {
        _generator = generator;
    }

    // Only this action needs the audit logger
    public IActionResult Download(int id, [FromServices] IAuditLogger audit)
    {
        audit.Log($"Report {id} downloaded.");
        var report = _generator.Generate(id);
        return File(report, "application/pdf");
    }
}
```

**Step‑by‑step explanation:**

1. `IReportGenerator` is injected via the constructor (used across actions).
2. `IAuditLogger` is injected via `[FromServices]` only into the `Download` action.
3. The container resolves `IAuditLogger` when `Download` is invoked.

**Expected Output:** The `Download` action receives both dependencies; other actions are unaffected by `IAuditLogger`.

**Why this output?** `[FromServices]` resolves the service as an action parameter, limiting its scope to that single method.

---

#### Example 2: Middleware Scoped Service Injection

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context,
        IMyScopedService scopedService)
    {
        scopedService.LogRequest(context);
        await _next(context);
    }
}

// Registration
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Expected Output:** Each request gets a fresh `IMyScopedService` instance injected into `InvokeAsync`, shared with the rest of the request pipeline.

**Why this output?** Scoped services are resolved per request; the middleware constructor runs once, so Scoped services must be method‑injected into `InvokeAsync`.

### Real‑World Cases

- **Controller actions with unique dependencies:** A `Download` action needing an `IAuditLogger` that no other action uses.
- **Middleware:** Scoped services (e.g., `DbContext`) injected into `InvokeAsync` for per‑request work.
- **Minimal APIs:** Service parameters in handler delegates (with `[FromServices]` or inference).

### References

- Dependency injection into controllers in ASP.NET Core (Action injection with `FromServices`) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- `FromServicesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute
- Middleware method injection – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware

---

## 4. Constructor Injection Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Constructor injection best practices are guidelines for designing constructors and using constructor injection in a way that is safe, maintainable, and testable. |
| **Technical Definition** | Best practices encompass keeping constructors focused on required dependencies, avoiding captive dependencies (e.g., a Singleton depending on a Scoped service), avoiding asynchronous constructor logic, guarding against constructor over‑injection, and understanding when constructor injection is inappropriate (e.g., middleware constructors for Scoped services). |
| **Beginner‑Friendly Explanation** | Best practices are the **“do’s and don’ts”** for constructor injection—they help you avoid common mistakes like a constructor with 15 parameters, a Singleton accidentally holding a per‑request database context, or async code in a constructor that cannot be awaited. |

### Purposes

- **To** keep classes focused and adhere to the Single Responsibility Principle.
- **To** prevent lifetime mismatches that cause state leakage and concurrency bugs.
- **To** ensure constructors remain simple, synchronous, and testable.
- **To** avoid the **constructor over‑injection** code smell, which signals a class with too many responsibilities.
- **To** align dependency lifetimes correctly (Singleton → Singleton, Scoped → Scoped/Transient, etc.).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Default pattern** | Use constructor injection as the primary pattern for required dependencies. | Reach for property or method injection first. |
| **Dependency count** | Keep constructors small; if a class has many dependencies, consider splitting responsibilities. | Inject 10+ services into a single constructor. |
| **Captive dependencies** | Ensure a Singleton never depends on a Scoped service. | Capture a Scoped service in a Singleton. |
| **Async logic** | Keep constructors synchronous. | Perform async I/O or blocking calls in a constructor. |
| **Middleware** | Inject Scoped services into `InvokeAsync`, not the constructor. | Inject Scoped services into middleware constructors. |
| **Interfaces** | Depend on interfaces for swappable implementations. | Depend on concrete classes that cannot be mocked. |
| **Readonly fields** | Store injected dependencies in `private readonly` fields. | Allow fields to be reassigned after construction. |
| **Controllers as services** | Understand that controllers are not resolved as services by default. | Assume controller lifetimes can be managed without `AddControllersAsServices()`. |

### Annotated Code Example (Best‑Practice Constructor Injection)

```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;
    private readonly IPaymentGateway _payment;
    private readonly ILogger<OrderService> _logger;

    public OrderService(
        IOrderRepository repository,
        IPaymentGateway payment,
        ILogger<OrderService> logger)
    {
        _repository = repository;
        _payment = payment;
        _logger = logger;
    }

    public async Task PlaceOrderAsync(Order order)
    {
        _logger.LogInformation("Placing order {OrderId}", order.Id);
        await _payment.ChargeAsync(order.Total);
        await _repository.SaveAsync(order);
    }
}
```

**Why this follows best practices:**

- **Dependencies are explicit** in the constructor—no hidden service locator calls.
- **Three focused dependencies**—the class has a clear responsibility (order placement).
- **`private readonly` fields** ensure dependencies cannot be replaced after construction.
- **Interfaces** allow mocking in unit tests.
- **No async logic in the constructor**—the async work happens in `PlaceOrderAsync`.
- **Lifetimes align**—the service, repository, and gateway would all be registered as Scoped or Transient consistently.

### Real-World Cases

- **Domain services:** Constructor injection of repositories, loggers, and gateways keeps services testable and explicit.
- **Controllers:** Constructor injection of shared dependencies; `[FromServices]` for action‑specific ones.
- **Hosted services:** Constructor injection of Singleton dependencies; explicit scopes for Scoped services via `IServiceScopeFactory`.

### References

- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- The ASP.NET Core Dependency Injection System (best practices, captive dependencies, async constructor logic) – https://github.com/abpframework/abp/blob/dev/docs/en/Community-Articles/2025-10-20-The-ASP-DotNET-Core-Dependency-Injection%20System/post.md
- Constructor injection is the gold standard – https://academy.optimizely.com/dependency-injection

---

## Summary Table: Injection Patterns at a Glance

| Pattern | Native ASP.NET Core Support | Key Mechanism | When to Use |
|---------|----------------------------|---------------|-------------|
| **Constructor injection** | ✅ Full support (default) | Constructor parameters | Required dependencies; default for controllers, services, page models |
| **Property injection** | ⚠️ Limited to controller properties via `[FromServices]` | Public setter + `[FromServices]` | Controller‑wide dependencies not needed in constructor; general property injection requires third‑party container |
| **Method injection** | ✅ `[FromServices]` on action parameters; middleware `InvokeAsync` parameters | Method/action parameters | Action‑specific dependencies; Scoped services in middleware |
| **Best practices** | — | Explicit constructors, no captive dependencies, no async constructors | Every production DI setup |

---

## References (Full List)

- Dependency injection into controllers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- `FromServicesAttribute` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute
- Built‑in DI does not support property injection (Stack Overflow) – https://stackoverflow.com/questions/53884417/net-core-di-property-injection
- Property and method injection in ASP.NET Core (Stack Overflow) – https://stackoverflow.com/questions/75633461/property-and-method-injection-in-asp-net-core-web-api
- The ASP.NET Core Dependency Injection System (GitHub) – https://github.com/abpframework/abp/blob/dev/docs/en/Community-Articles/2025-10-20-The-ASP-DotNET-Core-Dependency-Injection%20System/post.md
- ASP.NET Core dependency injection best practices – https://volosoft.com/blog/aspnet-core-dependency-injection-best-practices
- Constructor injection is the gold standard – https://academy.optimizely.com/dependency-injection
- .NET dependency injection (Aliyun) – https://developer.aliyun.com/article/1625684