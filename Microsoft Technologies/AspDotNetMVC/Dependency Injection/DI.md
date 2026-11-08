# ASP.NET Core Dependency Injection Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (all modern versions, .NET 6–10) and the **built-in DI container** (`Microsoft.Extensions.DependencyInjection`). ASP.NET Core has built-in support for dependency injection (DI), which makes apps easier to test and maintain. The concepts apply equally to MVC controllers, Razor Pages, Blazor, Minimal APIs, and Worker Services. Where behaviour differs by hosting model, it is noted. DI is not optional in ASP.NET Core — the framework itself resolves services (controllers, `DbContext`, `ILogger`, etc.) from the container. This cheat sheet does **not** merge sections — each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dependency injection (DI) is a software design pattern in which a class receives its dependencies from an external source rather than creating them itself. |
| **Technical Definition** | DI is a technique for achieving **Inversion of Control (IoC)** between classes and their dependencies. A **dependency** is an object that another object depends on. In .NET, DI is a built-in part of the framework, alongside configuration, logging, and the options pattern. ASP.NET Core uses a built-in DI container (`IServiceProvider`) to manage the services an app uses. Services are registered in an `IServiceCollection` at startup and resolved from the container when needed. |
| **Beginner-Friendly Explanation** | Dependency injection is a **way of giving a class what it needs instead of letting it build what it needs**. Imagine a coffee machine that requires water and beans. Without DI, the machine would have to drill its own well and grow its own beans — tightly coupled and hard to change. With DI, someone hands the machine water and beans, and the machine only focuses on brewing. In code, a class declares “I need an `IDateTime` service” in its constructor, and the DI container supplies it. |

### Key Characteristics

- **Inversion of Control** – The framework, not the class, controls dependency creation and lifetime.
- **Built-in container** – ASP.NET Core provides `IServiceProvider` and `IServiceCollection` out of the box.
- **Constructor injection is the default** – Services are requested explicitly via constructors.
- **Three lifetimes** – Transient, Scoped, and Singleton control how long a service instance lives.
- **Extensible** – Third-party containers (Autofac, DryIoc, etc.) can replace the built-in container if needed.
- **Disposal management** – The container disposes `IDisposable`/`IAsyncDisposable` services according to their lifetime.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project (MVC, Razor Pages, Blazor, or Minimal API).
- Basic understanding of **C# interfaces**, **constructors**, and **classes**.
- Familiarity with `Program.cs` (minimal hosting) or `Startup.cs` (legacy hosting).
- No external DI package is required — the built-in container is part of the shared framework.

### Related Programming Areas

- **Service lifetimes** – Controls instance reuse (Transient, Scoped, Singleton).
- **Configuration** – Often injected as `IOptions<T>` or `IConfiguration`.
- **Logging** – `ILogger<T>` is resolved from the container.
- **Entity Framework Core** – `DbContext` is registered with a **Scoped** lifetime by default.
- **Middleware** – Middleware components can receive services via constructor injection.
- **Authorization** – Authorization handlers are registered in the DI container.

### Core Concepts / Features

1. **Dependency injection** – The pattern and its benefits.
2. **DI** – The built-in infrastructure (`IServiceCollection`, `IServiceProvider`).
3. **DI container** – The component that resolves and manages services.
4. **Service lifetimes** – The overarching concept of instance reuse.
   - **Transient** – A new instance every time.
   - **Scoped** – One instance per request/scope.
   - **Singleton** – One instance for the application lifetime.
5. **DI best practices** – Guidelines for designing services and avoiding common pitfalls.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. Dependency Injection (The Pattern)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Dependency injection is a design pattern where a class’s dependencies are provided externally rather than created internally. |
| **Technical Definition** | DI is a form of **Inversion of Control** in which the responsibility for creating and supplying dependencies is inverted: instead of a class instantiating its collaborators, an external container constructs them and injects them via constructor parameters, properties, or method parameters. In ASP.NET Core, DI is a first-class framework feature: services are registered in `IServiceCollection` and resolved from `IServiceProvider`. |
| **Beginner-Friendly Explanation** | DI means **“don’t build what you need; ask for it.”** A `HomeController` that needs the current time should not write `new SystemDateTime()` inside itself. Instead, it declares `public HomeController(IDateTime dateTime)` and lets the framework supply an `IDateTime`. This makes the controller testable (you can pass a fake time provider) and flexible (you can swap implementations without editing the controller). |

### Purposes

- **To** decouple classes from concrete implementations, enabling easier replacement and testing.
- **To** centralise the configuration and lifetime management of dependencies in one place.
- **To** make unit testing possible by allowing mock or stub dependencies to be injected.
- **To** avoid hard-coded dependencies that scatter configuration code across the application.
- **To** support the **Single Responsibility Principle** by letting classes focus on their core logic rather than dependency construction.

### Syntax Rules and Structure

#### General Syntax (Constructor Injection)

```csharp
public class HomeController : Controller
{
    private readonly IDateTime _dateTime;

    public HomeController(IDateTime dateTime)
    {
        _dateTime = dateTime;
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IDateTime` | The abstraction (interface) the class depends on. |
| Constructor parameter | The dependency is requested explicitly. |
| Private field | Stores the injected instance for later use. |

#### General Syntax (Registration)

```csharp
// Program.cs
builder.Services.AddSingleton<IDateTime, SystemDateTime>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddSingleton` | Registers the service with a Singleton lifetime. |
| `IDateTime` | Service type (usually an interface). |
| `SystemDateTime` | Implementation type. |

#### General Syntax (Action Injection with `[FromServices]`)

```csharp
public IActionResult About([FromServices] IDateTime dateTime)
{
    return Content($"Current server time: {dateTime.Now}");
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `[FromServices]` | Injects a service directly into an action method without constructor injection. |

#### Syntax Rules

- **Constructor injection** is the default and recommended approach.
- Services are typically defined using **interfaces** to allow multiple implementations.
- Registration must occur before `builder.Build()` (or in `ConfigureServices` for legacy hosting).
- Action injection (`[FromServices]`) is available for occasional use but constructor injection is preferred for testability.

#### Constraints and Limitations

- **Constructor injection cannot be used in middleware’s `Invoke` method** — use method injection there instead.
- **Captive dependencies** – A Singleton that captures a Scoped service becomes effectively Scoped, causing state leakage.
- **DI is not a service locator** – Injecting `IServiceProvider` and calling `GetService` is an anti-pattern.

### Annotated Complete Step-by-Step Code Examples

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
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IDateTime, SystemDateTime>();
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Controller
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

**Step-by-step explanation:**

1. `IDateTime` abstracts the concept of “current time.”
2. `SystemDateTime` implements it using `DateTime.Now`.
3. The service is registered as a Singleton.
4. `HomeController` requests `IDateTime` in its constructor.
5. The runtime resolves `SystemDateTime` and injects it.
6. `Index` uses the injected service to choose a greeting.

**Expected Output:** A greeting message based on the server’s current hour (morning vs. afternoon).

**Why this output?** The controller never creates `SystemDateTime` itself; it depends on the abstraction, and the container supplies the configured implementation.

### Real-World Cases

- **Testable controllers:** Injecting `IDateTime` allows unit tests to pass a fixed-time stub.
- **Swappable implementations:** In development, inject a fake email sender; in production, inject the real SMTP sender.
- **Cross-cutting services:** `ILogger<T>`, `IConfiguration`, and `IOptions<T>` are injected into controllers and services throughout the framework.

### References

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- .NET dependency injection – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/overview
- Dependency injection into controllers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection

---

## 2. DI (The Infrastructure)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | DI (in the ASP.NET Core context) refers to the built-in infrastructure — `IServiceCollection`, `IServiceProvider`, and the registration/resolution pipeline — that implements the dependency injection pattern. |
| **Technical Definition** | `IServiceCollection` is a collection of `ServiceDescriptor` objects that describe services (service type, implementation, lifetime). `BuildServiceProvider()` creates an `IServiceProvider` (the container) from the collection. Services are resolved from the provider using `GetService<T>()` or `GetRequiredService<T>()`. In ASP.NET Core, the framework constructs the provider automatically from `builder.Services` during app startup. |
| **Beginner-Friendly Explanation** | DI infrastructure is the **“supply room”** of your app. At startup, you tell the supply room what services you have and how long each should last (`builder.Services.Add...`). When a class needs something, the supply room (`IServiceProvider`) hands it over. You rarely interact with the supply room directly — the framework does it for you. |

### Purposes

- **To** provide a standard, built-in mechanism for registering and resolving services without third-party libraries.
- **To** centralise service configuration in `Program.cs` (or `Startup.ConfigureServices`).
- **To** manage service lifetimes and disposal automatically.
- **To** enable framework features (controllers, Razor Pages, EF Core, logging, configuration) to resolve their own dependencies.
- **To** support replacement with third-party containers while keeping the same registration API.

### Syntax Rules and Structure

#### General Syntax (Registration)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddTransient<IMyService, MyService>();
builder.Services.AddScoped<IRepository, Repository>();
builder.Services.AddSingleton<IClock, SystemClock>();

var app = builder.Build();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `builder.Services` | The `IServiceCollection` where services are registered. |
| `AddTransient` / `AddScoped` / `AddSingleton` | Extension methods that add `ServiceDescriptor` entries. |
| `Build()` | Creates the `IServiceProvider` (container). |

#### General Syntax (Resolution)

```csharp
// Preferred: constructor injection
public class MyClass
{
    public MyClass(IMyService service) { ... }
}

// Direct resolution (use sparingly)
var service = app.Services.GetRequiredService<IMyService>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor parameter | Preferred resolution mechanism. |
| `GetRequiredService<T>()` | Throws if the service is not registered. |
| `GetService<T>()` | Returns `null` if the service is not registered. |

#### General Syntax (`ServiceDescriptor`)

```csharp
services.Add(new ServiceDescriptor(
    typeof(IMyService),
    typeof(MyService),
    ServiceLifetime.Scoped));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ServiceDescriptor` | Describes a service with its service type, implementation, and lifetime. |
| `ServiceLifetime` | Enum: `Transient`, `Scoped`, `Singleton`. |

#### Syntax Rules

- Registrations are **order-sensitive** for multiple implementations of the same service; the **last registration wins** when resolving a single instance.
- `AddScoped`, `AddTransient`, and `AddSingleton` have overloads for:
  - `TService, TImplementation` (type mapping)
  - `TService` (self-registration)
  - Factory delegates (`Func<IServiceProvider, TService>`)
  - Existing instances (`AddSingleton<T>(instance)`)
- **`TryAdd*`** methods (`TryAddTransient`, etc.) register only if the service type is not already registered.

#### Constraints and Limitations

- **Built-in container limitations** – It does not support named/keyed registrations in older versions (keyed services were added in .NET 8), child containers, or advanced lifetime management.
- **Third-party containers** – Autofac, DryIoc, and others can replace the built-in container while retaining the `IServiceCollection` registration API.
- **`BuildServiceProvider` in production** – Avoid calling it multiple times; use the host-provided `IServiceProvider`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Registering and Resolving Services

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddSingleton<IClock, SystemClock>();

builder.Services.AddControllersWithViews();

var app = builder.Build();

// Framework resolves services automatically for controllers
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**Step-by-step explanation:**

1. Three services are registered with different lifetimes.
2. `AddControllersWithViews` registers MVC services.
3. The framework builds the container at `app.Build()`.
4. When a controller requests `IOrderRepository`, the container creates or reuses an instance according to the Scoped lifetime.

**Expected Output:** Controllers and framework services resolve their dependencies automatically.

**Why this output?** The `IServiceCollection` describes the services, and the `IServiceProvider` resolves them when requested.

### Real-World Cases

- **Framework services:** `ILogger<T>`, `IConfiguration`, `IOptions<T>`, and `DbContext` are all resolved through the built-in DI infrastructure.
- **Application services:** Repositories, email senders, and business-logic services are registered in `Program.cs`.
- **Testing:** `WebApplicationFactory<T>` uses the same DI infrastructure to override registrations for integration tests.

### References

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- `IServiceCollection` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- `IServiceProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider

---

## 3. DI Container

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The DI container is the component that stores service registrations and creates/resolves service instances on demand. |
| **Technical Definition** | In ASP.NET Core, the DI container is an implementation of `IServiceProvider` built from an `IServiceCollection`. It reads `ServiceDescriptor` entries (service type, implementation type/factory/instance, and lifetime) and, when asked for a service, either creates a new instance, returns a cached instance, or throws if the service is not registered. The container also manages disposal of `IDisposable`/`IAsyncDisposable` services according to their lifetime. |
| **Beginner-Friendly Explanation** | The DI container is the **“factory + warehouse”** of your app. You give it a blueprint (registrations) for every service. When a class asks for a service, the container either builds a new one or fetches an existing one from the warehouse, depending on the lifetime you configured. |

### Purposes

- **To** decouple service creation from service consumption.
- **To** manage service lifetimes automatically (Transient, Scoped, Singleton).
- **To** dispose services correctly at the end of their lifetime.
- **To** provide a single resolution point for all framework and application services.
- **To** support replacement with third-party containers without changing consumer code.

### Syntax Rules and Structure

#### General Syntax (Container Creation)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IMyService, MyService>();

var app = builder.Build();   // Container is built here
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `builder.Services` | The `IServiceCollection` (registrations). |
| `builder.Build()` | Creates the `IServiceProvider` (container). |
| `app.Services` | The container instance. |

#### General Syntax (Resolution from the Container)

```csharp
using var scope = app.Services.CreateScope();
var service = scope.ServiceProvider.GetRequiredService<IMyService>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CreateScope()` | Creates an explicit scope (needed for Scoped services outside a request). |
| `scope.ServiceProvider` | The scoped container. |
| `GetRequiredService<T>()` | Resolves the service or throws. |

#### General Syntax (Third-Party Container Replacement)

```csharp
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    containerBuilder.RegisterType<MyService>().As<IMyService>();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseServiceProviderFactory` | Replaces the built-in container with a third-party one. |
| `ConfigureContainer` | Configures the third-party container. |

#### Syntax Rules

- The container is **immutable after build** — new registrations after `Build()` are not reflected.
- **Scoped services require a scope** — resolving a Scoped service from the root provider throws `InvalidOperationException`.
- **Disposal is automatic** — do not manually dispose services resolved from the container.

#### Constraints and Limitations

- **Built-in container** – Limited advanced features compared to Autofac/DryIoc (e.g., assembly scanning, decorators, named registrations in older versions).
- **Root provider** – Resolving Scoped services from the root provider is invalid; always create a scope.
- **Captive dependencies** – A Singleton that depends on a Scoped service captures it for the application lifetime, causing state bugs.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Creating a Scope and Resolving a Scoped Service

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IMyScopedService, MyScopedService>();
var app = builder.Build();

// Background work that needs a scoped service
using (var scope = app.Services.CreateScope())
{
    var scopedService = scope.ServiceProvider
        .GetRequiredService<IMyScopedService>();
    scopedService.DoWork();
}
```

**Step-by-step explanation:**

1. A Scoped service is registered.
2. Outside a request (e.g., in a background task), an explicit scope is created.
3. The Scoped service is resolved from the scope’s provider.
4. The scope is disposed, which disposes the Scoped service.

**Expected Output:** The scoped service is used safely outside a request, and is disposed when the scope ends.

**Why this output?** Scoped services must be resolved from a scope; the explicit scope mimics the per-request scope.

#### Example 2: Demonstrating the Captive Dependency Problem

```csharp
// Anti-pattern: Singleton depends on Scoped
builder.Services.AddScoped<IMyScopedService, MyScopedService>();
builder.Services.AddSingleton<IMySingletonService, MySingletonService>();

public class MySingletonService
{
    public MySingletonService(IMyScopedService scoped) { ... } // Captive dependency!
}
```

**Expected Output:** In development, the container throws an exception because a Singleton cannot depend on a Scoped service. In production, the Scoped service behaves like a Singleton, causing state leakage.

**Why this output?** The Singleton is created once and captures the Scoped instance for the application lifetime, violating the Scoped lifetime contract.

### Real-World Cases

- **Per-request scoping:** ASP.NET Core creates a scope per HTTP request; `DbContext` and repositories are resolved from this scope.
- **Background services:** Worker services create explicit scopes to resolve Scoped services safely.
- **Third-party containers:** Large applications with advanced needs (assembly scanning, decorators) replace the built-in container with Autofac.

### References

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- `IServiceProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- `ServiceDescriptor` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor

---

## 4. Service Lifetimes

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A service lifetime defines how long a service instance lives and how often it is created. |
| **Technical Definition** | ASP.NET Core supports three lifetimes: **Transient** (created each time it is requested), **Scoped** (created once per scope, typically per HTTP request), and **Singleton** (created once for the application lifetime). The lifetime is specified via `AddTransient`, `AddScoped`, or `AddSingleton` (or via `ServiceDescriptor`). The container enforces the lifetime: Transient services are disposed at the end of the request; Scoped services at the end of the scope; Singleton services at application shutdown. |
| **Beginner-Friendly Explanation** | Service lifetimes answer the question: **“How often should a new instance be created?”** A **Transient** service is like a disposable cup — a new one every time. A **Scoped** service is like a shared office coffee mug — one per person (per request). A **Singleton** service is like the office water cooler — one for everyone, all day. |

### Purposes

- **To** control resource usage (avoid creating expensive objects unnecessarily).
- **To** control state sharing (Singleton for shared state, Scoped for per-request state, Transient for stateless services).
- **To** ensure correct disposal timing (per-request, per-scope, or at shutdown).
- **To** prevent bugs from captive dependencies and unintended state leakage.
- **To** optimise performance by reusing instances where appropriate.

### Syntax Rules and Structure

#### General Syntax (Registration)

```csharp
services.AddTransient<ITransientService, TransientService>();
services.AddScoped<IScopedService, ScopedService>();
services.AddSingleton<ISingletonService, SingletonService>();
```

**Component breakdown:**

| Method | Lifetime | Instance creation |
|--------|----------|-------------------|
| `AddTransient` | Transient | Each request for the service |
| `AddScoped` | Scoped | Once per scope (per HTTP request) |
| `AddSingleton` | Singleton | Once for the application |

#### Syntax Rules

- The lifetime is **per registration**, not per type. The same type can be registered with different lifetimes under different service types.
- **Scoped services** must be resolved from a scope; resolving from the root provider throws.
- **Singleton services** are created either on first resolution or at startup (if `ValidateOnBuild`/`ValidateScopes` is enabled).

#### Constraints and Limitations

- **Captive dependencies** – A Singleton must not depend on a Scoped or Transient service if it will hold the reference; otherwise, the shorter-lived service becomes effectively Singleton.
- **Transient `IDisposable` services** are tracked by the container for the lifetime of the scope; resolving many transient disposables can cause memory pressure.
- **Scoped services in Singletons** – Use `IServiceScopeFactory` to create an explicit scope inside a Singleton if a Scoped service is needed.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Observing Lifetime Differences

```csharp
public interface IGuidService { Guid Id { get; } }

public class GuidService : IGuidService
{
    public Guid Id { get; } = Guid.NewGuid();
}

// Registration
builder.Services.AddTransient<IGuidService, GuidService>();
builder.Services.AddScoped<IGuidService, GuidService>();
builder.Services.AddSingleton<IGuidService, GuidService>();
```

**Expected Output:**

| Lifetime | Within one request | Across two requests |
|----------|-------------------|---------------------|
| Transient | Different `Id` each resolution | Different `Id` |
| Scoped | Same `Id` within the request | Different `Id` across requests |
| Singleton | Same `Id` always | Same `Id` always |

**Why this output?** The container creates or reuses instances according to the registered lifetime.

### Real-World Cases

- **Transient:** Lightweight, stateless services (e.g., `IEmailSender`).
- **Scoped:** Per-request state (e.g., `DbContext`, repositories, unit-of-work).
- **Singleton:** Application-wide state or expensive resources (e.g., `IClock`, caching services, configuration wrappers).

### References

- Service lifetimes (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection guidelines (disposal) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines

---

### 4.1 Transient

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Transient service is created **every time** it is requested from the container. |
| **Technical Definition** | `AddTransient` registers a service with `ServiceLifetime.Transient`. Each call to `GetService<T>()` (or each constructor injection) produces a **new instance**. In request-processing apps, transient services are disposed at the end of the request. This lifetime incurs per-request allocations because services are constructed every time. |
| **Beginner-Friendly Explanation** | Transient is the **“disposable cup”** lifetime. Every time someone asks for the service, a brand-new instance is handed out. Nothing is shared, and nothing is reused. |

#### Purposes

- **To** ensure that no state is shared between consumers.
- **To** provide lightweight, stateless services that are cheap to create.
- **To** avoid concurrency issues that can arise from shared state.
- **To** guarantee that each consumer gets a fresh instance.

#### Syntax Rules and Structure

```csharp
services.AddTransient<IMyService, MyService>();
services.AddTransient<IMyService>(sp => new MyService(sp.GetRequiredService<IDep>()));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddTransient` | Registers with Transient lifetime. |
| Factory overload | Creates a new instance per resolution. |

**Syntax rules:**

- Transient services can depend on Scoped and Singleton services (but see captive dependency warnings).
- Transient `IDisposable` services are tracked by the container until the end of the scope.

**Constraints and limitations:**

- **High allocation cost** if resolved frequently in hot paths.
- **Transient `IDisposable`** services can cause memory pressure if many are resolved within one scope.

#### Annotated Code Example

```csharp
public class LightweightService : ILightweightService
{
    public Guid InstanceId { get; } = Guid.NewGuid();
}

builder.Services.AddTransient<ILightweightService, LightweightService>();

public class ConsumerA
{
    public ConsumerA(ILightweightService a, ILightweightService b)
    {
        // a.InstanceId != b.InstanceId
    }
}
```

**Expected Output:** `ConsumerA` receives two **different** `LightweightService` instances, even within the same constructor.

**Why this output?** Transient resolution creates a new instance for every request, including multiple requests within the same constructor.

#### Real-World Cases

- **Email senders:** A stateless `IEmailSender` that opens a new SMTP connection per send.
- **Validators:** Lightweight validation services that hold no state.
- **Factories:** Services that create short-lived objects.

#### References

- Service lifetimes (Transient) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#transient

---

### 4.2 Scoped

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Scoped service is created **once per scope** and reused within that scope. |
| **Technical Definition** | `AddScoped` registers a service with `ServiceLifetime.Scoped`. In ASP.NET Core web apps, a scope is created **per HTTP request**, so a Scoped service is created once per request and shared by all consumers within that request. Scoped services are disposed at the end of the request. `DbContext` is registered as Scoped by default via `AddDbContext`. |
| **Beginner-Friendly Explanation** | Scoped is the **“per-request”** lifetime. For each HTTP request, one instance is created and shared by everything that needs it during that request. The next request gets a fresh instance. |

#### Purposes

- **To** share state within a single request (e.g., a unit-of-work or transaction).
- **To** avoid creating expensive per-request objects multiple times.
- **To** align with the lifetime of a database context (`DbContext`), which is not thread-safe and should not be shared across requests.
- **To** ensure per-request isolation and disposal.

#### Syntax Rules and Structure

```csharp
services.AddScoped<IRepository, Repository>();
services.AddScoped<DbContext>(sp => new DbContext(...));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddScoped` | Registers with Scoped lifetime. |
| Scope | Created automatically per HTTP request or explicitly via `CreateScope()`. |

**Syntax rules:**

- Must be resolved from a scope; resolving from the root provider throws.
- A Scoped service can depend on Singleton and other Scoped services.
- A Singleton **must not** depend on a Scoped service (captive dependency).

**Constraints and limitations:**

- **Captive dependencies** – If a Singleton captures a Scoped service, the Scoped service behaves like a Singleton.
- **Background tasks** – Must create an explicit scope with `IServiceScopeFactory` to resolve Scoped services.

#### Annotated Code Example

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

#### Real-World Cases

- **`DbContext`:** EF Core’s `DbContext` is Scoped by default to ensure per-request isolation.
- **Unit-of-work:** A Scoped service that tracks changes across multiple repositories within one request.
- **Per-request caching:** A Scoped cache that avoids repeated database queries within one request.

#### References

- Service lifetimes (Scoped) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped

---

### 4.3 Singleton

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A Singleton service is created **once** and shared for the entire application lifetime. |
| **Technical Definition** | `AddSingleton` registers a service with `ServiceLifetime.Singleton`. The instance is created on first resolution (or at startup if validation is enabled) and reused for every subsequent request. Singleton services are disposed when the service container is disposed, usually at application shutdown. Singletons **must not** depend on Scoped services, because the Scoped instance would be captured for the application lifetime. |
| **Beginner-Friendly Explanation** | Singleton is the **“one for everyone, forever”** lifetime. The first time the service is needed, it is created. After that, every class in every request gets the **same** instance. It lives until the app shuts down. |

#### Purposes

- **To** share application-wide state or configuration.
- **To** avoid the cost of repeatedly creating expensive resources (e.g., caches, connection pools).
- **To** provide a single coordination point (e.g., a background task queue).
- **To** implement the **Singleton design pattern** within the DI container.

#### Syntax Rules and Structure

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

#### Annotated Code Example

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

#### Real-World Cases

- **Clocks:** `IClock` / `IDateTime` (stateless, thread-safe).
- **Caches:** In-memory caches shared across requests.
- **Configuration wrappers:** `IOptions<T>` (registered as Singleton by default).
- **Background task queues:** A Singleton queue that coordinates work across requests.

#### References

- Service lifetimes (Singleton) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#singleton

---

## 5. DI Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | DI best practices are guidelines for designing, registering, and consuming services in a way that is secure, maintainable, and performant. |
| **Technical Definition** | Best practices encompass service design (avoiding stateful statics, keeping services small), lifetime selection (matching lifetime to intended state scope), disposal management (letting the container dispose services), avoidance of captive dependencies, and correct use of constructor injection over service-locator patterns. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for DI — they help you avoid subtle bugs like a Singleton accidentally holding onto a per-request database context, or a class that is impossible to unit test because it creates its own dependencies. |

### Purposes

- **To** prevent captive dependencies and state-leakage bugs.
- **To** ensure services are correctly disposed by the container.
- **To** keep services small, focused, and testable.
- **To** avoid anti-patterns such as service locator and direct instantiation.
- **To** choose lifetimes that match the intended scope of state.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Service design** | Avoid stateful, static classes and members; use singleton services for global state instead. | Create global state through static mutable fields. |
| **Direct instantiation** | Let the container create dependencies. | Use `new` to create dependent classes inside services — this couples code to a specific implementation. |
| **Service size** | Make services small, well-factored, and easily tested. | Create “god” services with many injected dependencies; this violates the Single Responsibility Principle. |
| **Disposal** | Let the container dispose `IDisposable`/`IAsyncDisposable` services based on lifetime. | Manually dispose services resolved from the container. |
| **Lifetime selection** | Match lifetime to state scope: Scoped for per-request state, Singleton for application state, Transient for stateless services. | Capture a Scoped service in a Singleton (captive dependency). |
| **Constructor injection** | Prefer constructor injection for required dependencies. | Use `IServiceProvider` as a service locator (`GetService` calls scattered in business logic). |
| **Scoped in Singletons** | Use `IServiceScopeFactory` to create an explicit scope inside a Singleton when a Scoped service is truly needed. | Resolve a Scoped service directly from a Singleton’s constructor. |
| **Transient disposables** | Be aware that transient `IDisposable` services are tracked until the end of the scope. | Resolve large numbers of transient disposables in a single scope. |

### Annotated Code Example (Best-Practice DI Setup)

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

**Why this follows best practices:**

- **Lifetimes match state scope:** Singleton for the stateless clock, Scoped for `DbContext`/repository, Transient for the stateless email sender.
- **No captive dependencies:** The Singleton (`IClock`) has no Scoped dependencies.
- **Disposal is automatic:** `DbContext` is disposed at the end of each request.
- **Constructor injection:** Controllers and services request dependencies via constructors, keeping them testable.

### Real-World Cases

- **EF Core:** `AddDbContext` registers `DbContext` as Scoped by default, aligning with per-request unit-of-work.
- **Background services:** Use `IServiceScopeFactory` to create scopes for Scoped services.
- **Third-party containers:** Replace the built-in container when advanced features (assembly scanning, decorators) are needed, while following the same lifetime principles.

### References

- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Service lifetimes – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection into controllers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection

---

## Summary Table: Core Concepts at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Dependency injection** | Decouples classes from concrete implementations | Constructor injection, `IDateTime` example | Every class with dependencies |
| **DI infrastructure** | Built-in registration and resolution | `builder.Services`, `IServiceProvider` | Application startup |
| **DI container** | Creates and manages service instances | `Build()`, `CreateScope()`, `GetRequiredService` | Framework and application services |
| **Service lifetimes** | Controls instance reuse and disposal | `AddTransient`, `AddScoped`, `AddSingleton` | Every registration |
| **Transient** | New instance every time | `AddTransient` | Lightweight, stateless services |
| **Scoped** | One instance per request/scope | `AddScoped` | Per-request state (`DbContext`, repositories) |
| **Singleton** | One instance for the application | `AddSingleton` | Application-wide state, caches, clocks |
| **Best practices** | Prevents captive dependencies and leaks | Lifetime matching, container disposal, SRP | Every production DI setup |

---

## References (Full List)

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- .NET dependency injection – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/overview
- Dependency injection into controllers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection
- Service lifetimes (dependency injection) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- `IServiceCollection` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- `IServiceProvider` interface – https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- `ServiceDescriptor` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor
- `IServiceScopeFactory` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicescopefactory