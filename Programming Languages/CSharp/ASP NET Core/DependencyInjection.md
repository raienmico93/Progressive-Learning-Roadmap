# C# Dependency Injection — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Dependency Injection (DI) is a design pattern and a built-in .NET infrastructure where a class receives its dependencies from an external container rather than creating them itself. This achieves Inversion of Control (IoC), decoupling classes from concrete implementations, improving testability, and centralizing the management of service lifetimes. ASP.NET Core ships with a lightweight, high-performance DI container (`Microsoft.Extensions.DependencyInjection`) used uniformly across ASP.NET Core, Worker Services, MAUI, Azure Functions, and Blazor.

**Technical Definition:** The .NET DI system is built around two core abstractions: **`IServiceCollection`** (a collection of `ServiceDescriptor` objects that describe registrations) and **`IServiceProvider`** (the resolved container that creates and supplies service instances). Services are registered with a **lifetime**—Transient, Scoped, or Singleton—that determines when instances are created and disposed. Resolution is typically performed via **constructor injection**, where the container inspects a type's constructor and supplies the required dependencies. Scoped services are bounded to an **`IServiceScope`**, created implicitly per request in ASP.NET Core or explicitly via **`IServiceScopeFactory.CreateScope()`**.

**Beginner-Friendly Explanation:** Dependency Injection is **“don't build what you need—ask for it.”** Instead of a class doing `new SqlConnection()` inside itself, it declares a constructor parameter like `ITodoRepository repo`, and a central container hands it the right object. This makes code **testable** (you can pass a fake repository in tests), **flexible** (swap implementations without changing the class), and **manageable** (the container handles when to create and dispose objects). Think of the DI container as a **smart vending machine**: you register what snacks it should stock (services), and whenever a class asks for one, the machine dispenses the correct item according to the rules you set (lifetimes).

---

### Key Characteristics

- **Inversion of Control** — The container, not the class, controls dependency creation.
- **Three lifetimes** — Transient (new every time), Scoped (one per request/scope), Singleton (one for the entire application).
- **Constructor injection** — The preferred resolution mechanism; dependencies are explicit.
- **Interface-based** — Services are typically registered and resolved by interface, not concrete type.
- **Scope validation** — In development, resolving a Scoped service from a Singleton throws an exception by default, catching the captive dependency anti-pattern early.
- **Disposal management** — The container automatically disposes services it creates that implement `IDisposable` or `IAsyncDisposable`.
- **Built-in, not third-party** — `Microsoft.Extensions.DependencyInjection` is a small NuGet package used by all modern .NET workloads.

---

### Prerequisites

- Understanding of **C# interfaces**, classes, and constructors.
- Familiarity with **ASP.NET Core fundamentals** (`Program.cs`, `WebApplicationBuilder`).
- Basic knowledge of **object lifetimes** and **`IDisposable`**.
- Awareness of **thread safety** for singleton services.
- Familiarity with **async/await** (for async disposal and scoped services in background tasks).

---

### Related Programming Areas

- **ASP.NET Core fundamentals** — DI is the backbone of service configuration.
- **Middleware** — Middleware can resolve services via constructor or `InvokeAsync` parameters.
- **Controllers and Minimal APIs** — Both support constructor and parameter injection.
- **Background services** — Use `IServiceScopeFactory` to create scopes for scoped work.
- **Testing** — DI enables injecting fakes and mocks.

---

### Core Concepts / Features

1. Dependency Injection (Overarching Concept)
2. `IServiceCollection`
3. `IServiceProvider`
4. Service Lifetimes (Transient, Scoped, Singleton)
5. Service Registration
6. Service Resolution
7. DI Best Practices
8. DI Anti-Patterns

---

## 1. Dependency Injection (Overarching Concept)

### Definitions

**Core Definition:** DI is the principle that a class should receive its collaborators from outside rather than constructing them internally.

**Technical Definition:** DI is implemented by a **service container** that manages a registry of service descriptors (interface → implementation → lifetime). When a type is activated (e.g., a controller or a middleware), the container resolves its constructor parameters recursively, building the **dependency graph** and returning a fully constructed object. The container also owns the lifetime and disposal of the services it creates.

**Beginner-Friendly Explanation:** DI is **“ask, don't new.”** You tell the container what you need in your constructor, and it provides it.

---

### Purposes

- To **decouple** classes from concrete implementations.
- To **improve testability** by allowing dependencies to be substituted.
- To **centralize lifetime management** (creation and disposal).
- To **reduce boilerplate** for wiring dependencies.
- To enable **configuration-driven** implementation selection.

---

### Syntax Rules and Structure

```csharp
// Registration
builder.Services.AddScoped<ITodoRepository, SqlTodoRepository>();

// Consumption
public class TodoService
{
    private readonly ITodoRepository _repo;
    public TodoService(ITodoRepository repo) => _repo = repo;
}
```

**Component breakdown**

- **Registration** — Maps an interface to an implementation with a lifetime.
- **Consumption** — The class declares its dependencies in the constructor.
- **Resolution** — The container supplies the dependencies when the class is activated.

#### Syntax Rules

- Dependencies are typically declared as interfaces.
- Constructor injection is the preferred mechanism.
- The container resolves the dependency graph recursively.

#### Constraints and Limitations

- DI is a library, not a framework; it does not enforce usage.
- Overuse can lead to “constructor bloat” and the service locator anti-pattern.

---

### Annotated Code Examples and Expected Outputs

#### Example: DI-Enabled Service

```csharp
public interface ITodoRepository
{
    Task<Todo?> FindAsync(int id, CancellationToken ct);
}

public sealed class SqlTodoRepository : ITodoRepository
{
    public Task<Todo?> FindAsync(int id, CancellationToken ct) { /* ... */ }
}

public sealed class TodoService(ITodoRepository repo)
{
    public Task<Todo?> FindAsync(int id, CancellationToken ct) => repo.FindAsync(id, ct);
}

// Registration
builder.Services.AddScoped<ITodoRepository, SqlTodoRepository>();
builder.Services.AddScoped<TodoService>();
```

**Expected Output (conceptual)**
```
// TodoService receives the scoped ITodoRepository automatically.
```

**Why this pattern is used:** `TodoService` cannot be constructed without a repository, and the container supplies the correct implementation and lifetime.

---

### Real-World Cases

- **Repository pattern** — `IUserRepository` → `SqlUserRepository`.
- **Logging** — `ILogger<T>` injected everywhere.
- **Configuration** — `IOptions<T>` for strongly typed settings.
- **HTTP clients** — `HttpClient` managed by `IHttpClientFactory`.

---

### References

- Dependency injection in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
- Dependency injection in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

---

## 2. `IServiceCollection`

### Definitions

**Core Definition:** `IServiceCollection` is a collection of `ServiceDescriptor` objects that represents the registration phase of DI.

**Technical Definition:** `Microsoft.Extensions.DependencyInjection.IServiceCollection` is an `ICollection<ServiceDescriptor>` populated at application startup. Each `ServiceDescriptor` captures the service type, implementation type (or factory/instance), and lifetime. Extension methods (`AddTransient`, `AddScoped`, `AddSingleton`, `AddHttpClient`, `AddDbContext`, etc.) provide fluent registration.

**Beginner-Friendly Explanation:** `IServiceCollection` is the **order form** you fill in before the container starts. You list what services you need and how they should be created.

---

### Purposes

- To **register services** before the container is built.
- To **configure implementations** and lifetimes.
- To **allow framework extensions** to add their own services.
- To provide a **single, discoverable registration surface**.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddScoped<IMyService, MyService>();
builder.Services.AddSingleton<ICache, MemoryCache>();

var app = builder.Build();
```

**Component breakdown**

- `builder.Services` — The `IServiceCollection` instance.
- `Add...` — Extension methods that append `ServiceDescriptor`s.
- `builder.Build()` — Validates and creates the `IServiceProvider`.

#### Syntax Rules

- Registration happens before `Build()`.
- Multiple registrations of the same service are allowed; the last one wins for single resolution.
- `TryAdd...` methods avoid duplicate registrations.

#### Constraints and Limitations

- The collection is not thread-safe during registration; registration is single-threaded at startup.
- `IServiceCollection` does not resolve services; only `IServiceProvider` does.

---

### Annotated Code Examples and Expected Outputs

#### Example: Registering Framework and Custom Services

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddScoped<ITodoRepository, SqlTodoRepository>();
builder.Services.AddTransient<IEmailService, SmtpEmailService>();
builder.Services.AddSingleton<IClock, SystemClock>();

var app = builder.Build();
```

**Expected Output (conceptual)**
```
// Controllers, repositories, email services, and clock are registered.
```

**Why this pattern is used:** All registrations are centralized and visible in `Program.cs`.

---

### Real-World Cases

- **ASP.NET Core startup** — Every app populates `builder.Services`.
- **Library extensions** — `AddMyLibrary()` methods add their services.
- **Testing** — A test-specific `IServiceCollection` can register fakes.

---

### References

- `IServiceCollection` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- Dependency injection in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection

---

## 3. `IServiceProvider`

### Definitions

**Core Definition:** `IServiceProvider` is the **resolved container** that creates and supplies service instances.

**Technical Definition:** `System.IServiceProvider` exposes `GetService(Type)`. The DI-specific extension methods include `GetRequiredService<T>()` (throws if not registered) and `GetService<T>()` (returns `null` if not registered). The provider is built from the `IServiceCollection` via `BuildServiceProvider()` or `WebApplicationBuilder.Build()`. It manages the lifetimes and disposal of the services it creates.

**Beginner-Friendly Explanation:** `IServiceProvider` is the **vending machine** that dispenses services. You ask it for a service, and it returns an instance according to the registered rules.

---

### Purposes

- To **resolve services** at runtime.
- To **create scopes** via `IServiceScopeFactory`.
- To **own lifetime management** and disposal.
- To serve as the root of the dependency graph.

---

### Syntax Rules and Structure

```csharp
// Resolve from the provider (prefer constructor injection over this)
var service = app.Services.GetRequiredService<IMyService>();

// Create a scope (for scoped services in background work)
using var scope = app.Services.CreateScope();
var scopedService = scope.ServiceProvider.GetRequiredService<IScopedService>();
```

**Component breakdown**

- `GetRequiredService<T>()` — Throws if missing.
- `GetService<T>()` — Returns `null` if missing.
- `CreateScope()` — Creates an `IServiceScope` with its own `ServiceProvider`.
- `scope.ServiceProvider` — Resolves scoped services.

#### Syntax Rules

- The root provider is the application-level provider.
- Scoped services must be resolved from a scope, not the root provider.
- `GetRequiredService` is preferred over `GetService` when the service must exist.

#### Constraints and Limitations

- Direct use of `IServiceProvider` inside application logic is the **service locator anti-pattern**.
- The root provider should not be used to resolve scoped services (they would behave like singletons).

---

### Annotated Code Examples and Expected Outputs

#### Example: Creating an Explicit Scope

```csharp
using var scope = app.Services.CreateScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
// use db within this scope
```

**Expected Output (conceptual)**
```
// A scoped AppDbContext is resolved within the explicit scope and disposed when the scope ends.
```

**Why this pattern is used:** Background services and singletons that need scoped services must create an explicit scope via `IServiceScopeFactory`.

---

### Real-World Cases

- **Background services** — `IServiceScopeFactory` to create scopes per work item.
- **Middleware** — Resolve scoped services in `InvokeAsync`.
- **Health checks** — Resolve services within a scope.

---

### References

- `IServiceProvider` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- `IServiceScopeFactory` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicescopefactory

---

## 4. Service Lifetimes

### Definitions

**Core Definition:** A service lifetime determines **when a service instance is created and how long it lives**.

**Technical Definition:** Three lifetimes are supported: **Transient** (new instance each request), **Scoped** (one instance per scope, typically per HTTP request), and **Singleton** (one instance for the application's lifetime). The lifetime affects allocation, disposal, and state sharing.

**Beginner-Friendly Explanation:** Lifetimes control **how often the vending machine makes a fresh snack**. Transient = a new snack every time; Scoped = one snack per customer (request); Singleton = one snack shared by everyone forever.

---

### Purposes

- To **control state sharing** (per-request vs. global).
- To **manage resource usage** (allocations vs. reuse).
- To **ensure thread safety** (singletons must be thread-safe).
- To **align with underlying resource lifetimes** (e.g., `DbContext` is scoped).

---

### Syntax Rules and Structure

| Lifetime | Registration Method | Instance Created | Instance Shared |
|---|---|---|---|
| **Transient** | `AddTransient<T>()` | Every resolution | Never |
| **Scoped** | `AddScoped<T>()` | Once per scope (request) | Within the scope |
| **Singleton** | `AddSingleton<T>()` | Once (first resolution) | Entire application |

**Component breakdown**

- **Transient** — Lightweight, stateless services; disposed at end of request.
- **Scoped** — Per-request services (e.g., `DbContext`); disposed at end of request.
- **Singleton** — Application-wide state; must be thread-safe; disposed at shutdown.

#### Syntax Rules

- Scoped services require a scope; resolving them from the root provider is invalid.
- Singletons must be thread-safe.
- Transient services are disposed at the end of the request in request-processing apps.

#### Constraints and Limitations

- **Captive dependency** — A singleton capturing a scoped service causes the scoped service to live forever.
- Singleton memory is not released until shutdown; consider memory use.

---

### Annotated Code Examples and Expected Outputs

#### Example: Observing Lifetimes

```csharp
builder.Services.AddTransient<IOperationTransient, Operation>();
builder.Services.AddScoped<IOperationScoped, Operation>();
builder.Services.AddSingleton<IOperationSingleton, Operation>();

// In a page/controller, inject all three and log OperationId
```

**Expected Output (within a single request)**
```
Transient: A1B2
Scoped: C3D4
Singleton: E5F6
```

**Expected Output (second request)**
```
Transient: G7H8 (new)
Scoped: I9J0 (new per request)
Singleton: E5F6 (same)
```

**Why this output occurs:** Transient IDs change every resolution; scoped IDs change per request; the singleton ID stays the same.

---

### Real-World Cases

- **`DbContext`** — Scoped by default with `AddDbContext`.
- **`ILogger<T>`** — Singleton.
- **`IHttpClientFactory`** — Singleton managing pooled handlers.
- **Repositories** — Scoped or transient depending on state.

---

### References

- Service lifetimes — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/docs/main/docs/core/extensions/dependency-injection/service-lifetimes.md
- Service lifetimes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes

---

## 5. Service Registration

### Definitions

**Core Definition:** Service registration is the process of mapping a service type to an implementation and lifetime in the `IServiceCollection`.

**Technical Definition:** Registration is performed via `AddTransient`, `AddScoped`, `AddSingleton`, or the generic `ServiceDescriptor` constructor. Overloads accept a concrete type, a factory delegate, or a pre-built instance. `TryAdd...` variants avoid duplicates. Multiple registrations of the same service allow resolving the last registration by default or all registrations via `IEnumerable<T>`.

**Beginner-Friendly Explanation:** Registration is **writing the vending machine's menu**: “When someone asks for `IMyService`, give them `MyService`, and create it this way.”

---

### Purposes

- To **define the mapping** between abstraction and implementation.
- To **choose the lifetime** appropriate for the service.
- To **provide factories** for complex construction.
- To **support multiple implementations** resolved as collections.

---

### Syntax Rules and Structure

```csharp
// Interface → implementation
services.AddScoped<IMyService, MyService>();

// Concrete type
services.AddSingleton<MyService>();

// Factory
services.AddSingleton<IClock>(_ => new SystemClock());

// Instance
services.AddSingleton<IConfig>(existingConfig);

// Multiple registrations
services.AddTransient<IPlugin, PluginA>();
services.AddTransient<IPlugin, PluginB>();
// Resolve all: IEnumerable<IPlugin>
```

**Component breakdown**

- `AddTransient`/`AddScoped`/`AddSingleton` — Lifetime-specific registration.
- `ServiceDescriptor` — The underlying descriptor.
- Factory overload — For custom construction logic.
- `TryAdd...` — Idempotent registration.

#### Syntax Rules

- Registration order matters for single resolution (last wins).
- Use `TryAdd` to avoid duplicate registrations in library extensions.
- Factories can resolve other services from the provider.

#### Constraints and Limitations

- Registering a service after `Build()` is not supported.
- Factories should not capture scoped services from the root provider.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Implementations

```csharp
services.AddTransient<IPlugin, PluginA>();
services.AddTransient<IPlugin, PluginB>();

// Resolve all
var plugins = app.Services.GetServices<IPlugin>();   // PluginA, PluginB
```

**Expected Output (conceptual)**
```
// Both PluginA and PluginB are resolved.
```

**Why this pattern is used:** `GetServices<T>()` returns all registrations, enabling plugin architectures.

---

### Real-World Cases

- **Plugin systems** — Multiple `IPlugin` implementations.
- **Strategy pattern** — Keyed services or named registrations.
- **Library extensions** — `TryAdd` to avoid conflicts.

---

### References

- Service registration methods — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#register-services
- `ServiceDescriptor` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor

---

## 6. Service Resolution

### Definitions

**Core Definition:** Service resolution is the act of obtaining a service instance from the `IServiceProvider`.

**Technical Definition:** Resolution can occur via **constructor injection** (preferred), **method parameter injection** (in Minimal APIs and middleware `InvokeAsync`), or direct calls to `GetService`/`GetRequiredService` (service locator, discouraged in application logic). The container resolves the full dependency graph recursively, honoring lifetimes and scopes.

**Beginner-Friendly Explanation:** Resolution is **getting the snack from the vending machine**. Constructor injection is the normal way; direct `GetService` is like bypassing the machine's intended workflow.

---

### Purposes

- To **supply dependencies** to classes and methods.
- To **respect lifetimes** (scoped services from the correct scope).
- To **support multiple resolution strategies** (constructor, parameter, direct).
- To **enable testing** via substituted providers.

---

### Syntax Rules and Structure

```csharp
// Constructor injection (preferred)
public class MyService(IRepo repo) { ... }

// Minimal API parameter injection
app.MapGet("/", (IRepo repo) => repo.GetAll());

// Direct resolution (discouraged in app logic)
var svc = app.Services.GetRequiredService<IMyService>();
```

**Component breakdown**

- **Constructor injection** — Dependencies are explicit and testable.
- **Parameter injection** — Minimal APIs and middleware resolve parameters per request.
- **Direct resolution** — Use only at the composition root or in genuinely dynamic scenarios.

#### Syntax Rules

- Scoped services must be resolved from a scope.
- `GetRequiredService` throws if missing; `GetService` returns `null`.
- The container recursively resolves constructor parameters.

#### Constraints and Limitations

- Resolving scoped services from the root provider creates captive dependencies.
- Service locator hides dependencies and complicates testing.

---

### Annotated Code Examples and Expected Outputs

#### Example: Constructor vs. Service Locator

```csharp
// Preferred: explicit dependency
public class GoodService(IRepo repo) { ... }

// Discouraged: service locator
public class BadService(IServiceProvider provider)
{
    public void DoWork()
    {
        var repo = provider.GetRequiredService<IRepo>();   // hidden dependency
    }
}
```

**Expected Output (conceptual)**
```
// GoodService's dependency is visible in its signature.
// BadService hides the dependency inside DoWork.
```

**Why this distinction matters:** Explicit constructor injection makes dependencies discoverable and testable; service locator hides them.

---

### Real-World Cases

- **Controllers** — Constructor injection.
- **Minimal APIs** — Parameter injection.
- **Middleware** — Constructor for singletons, `InvokeAsync` for scoped.
- **Background services** — `IServiceScopeFactory` for scoped resolution.

---

### References

- Service resolution — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-resolution
- `GetRequiredService` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice

---

## 7. DI Best Practices

### Definitions

**Core Definition:** DI best practices are the guidelines that keep registrations, lifetimes, and resolutions correct, maintainable, and testable.

**Technical Definition:** Best practices derive from Microsoft's DI guidance: **prefer constructor injection** over service locator; **match lifetimes** (do not inject scoped services into singletons); **keep singletons thread-safe**; **use `IServiceScopeFactory`** for scoped work in singletons/background services; **register services at startup**; **avoid resolving services in constructors of other services** (let the container do it); **use `TryAdd`** in library extensions; and **validate scopes in development** (`ValidateScopes = true`).

**Beginner-Friendly Explanation:** The best practices are: **ask in the constructor**, **match lifetimes**, **don't capture scoped in singleton**, **keep singletons thread-safe**, and **validate your scopes**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Constructor injection
public class MyService(IRepo repo) { ... }

// 2. Match lifetimes
services.AddScoped<IRepo, Repo>();
services.AddScoped<MyService>();

// 3. Use IServiceScopeFactory in singletons
public class Worker(IServiceScopeFactory factory)
{
    public async Task RunAsync()
    {
        using var scope = factory.CreateScope();
        var repo = scope.ServiceProvider.GetRequiredService<IRepo>();
    }
}

// 4. Validate scopes in development
builder.Host.UseDefaultServiceProvider(o =>
{
    o.ValidateScopes = true;
    o.ValidateOnBuild = true;
});
```

**Component breakdown**

- **Constructor injection** — Dependencies explicit.
- **Lifetime matching** — Scoped with scoped; singleton with singleton.
- **`IServiceScopeFactory`** — For scoped work in long-lived services.
- **Scope validation** — Catches captive dependencies early.

#### Rules and Constraints

- **Do** use constructor injection.
- **Do** match lifetimes.
- **Do** keep singletons thread-safe.
- **Do** use `IServiceScopeFactory` in background services.
- **Do** enable scope validation in development.
- **Do not** resolve scoped services from the root provider.
- **Do not** use the service locator pattern in application logic.

---

### Annotated Code Examples and Expected Outputs

#### Example: Correct Lifetime Matching

```csharp
services.AddScoped<AppDbContext>();
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<UserService>();
```

**Expected Output (conceptual)**
```
// All three services share the request scope and are disposed together.
```

**Why this pattern is correct:** Matching lifetimes avoids captive dependencies and ensures consistent state.

---

### Real-World Cases

- **Web APIs** — Scoped services per request.
- **Background workers** — Scoped work via `IServiceScopeFactory`.
- **Singletons** — Thread-safe caches and configuration.

---

### References

- Dependency injection guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- Scope validation — Dependency injection in .NET — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scope-validation

---

## 8. DI Anti-Patterns

### Definitions

**Core Definition:** DI anti-patterns are common misuses of DI that lead to hidden dependencies, memory leaks, incorrect state, or testability problems.

**Technical Definition:** The two most critical anti-patterns are the **captive dependency** (a singleton capturing a scoped or transient service, causing the shorter-lived service to live forever) and the **service locator** (resolving dependencies via `IServiceProvider` inside application logic, hiding the true dependency graph). Other issues include **constructor bloat** (too many dependencies), **asynchronous constructor logic**, and **undisposed scopes**.

**Beginner-Friendly Explanation:** Anti-patterns are **the ways DI goes wrong**: a Singleton accidentally “kidnapping” a Scoped service, or code secretly asking the container for things instead of declaring them openly.

---

### Common Anti-Patterns

#### 1. Captive Dependency

```csharp
// BAD: Singleton captures a Scoped service
services.AddSingleton<MySingleton>();
services.AddScoped<MyScoped>();

public class MySingleton(MyScoped scoped) { ... }   // ❌ captured forever
```

**Why it is wrong:** The Scoped service lives as long as the Singleton, sharing state across requests and potentially causing data corruption. **Fix:** Use `IServiceScopeFactory` or match lifetimes.

---

#### 2. Service Locator

```csharp
// BAD: hidden dependency
public class MyService(IServiceProvider provider)
{
    public void DoWork() => provider.GetRequiredService<IRepo>().Save();
}
```

**Why it is wrong:** Dependencies are hidden, making the class harder to test and reason about. **Fix:** Inject `IRepo` directly via the constructor.

---

#### 3. Constructor Bloat

```csharp
// BAD: too many dependencies
public class OrderService(IRepo repo, ILogger logger, IEmail email,
    ISms sms, IPayment payment, IInventory inventory, ...) { ... }
```

**Why it is wrong:** A class with many dependencies likely has too many responsibilities. **Fix:** Refactor into smaller services or use a facade.

---

#### 4. Undisposed Scope

```csharp
// BAD: scope not disposed
var scope = app.Services.CreateScope();
var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
// scope leaked; db never disposed
```

**Why it is wrong:** Scoped services (like `DbContext`) are not disposed, causing resource leaks. **Fix:** Use `using var scope = ...`.

---

### Annotated Code Examples and Expected Outputs

#### Example: Detecting Captive Dependencies

```csharp
builder.Host.UseDefaultServiceProvider(o =>
{
    o.ValidateScopes = true;   // throws at startup if captive dependency exists
    o.ValidateOnBuild = true;
});
```

**Expected Output (conceptual)**
```
// Startup throws: Cannot consume scoped service from singleton.
```

**Why this configuration is used:** Scope validation catches captive dependencies early in development.

---

### Real-World Cases

- **`DbContext` in singleton** — Classic captive dependency; always use `IServiceScopeFactory` or scoped services.
- **Service locator in controllers** — Resolving `HttpContext.RequestServices` directly instead of injecting.

---

### References

- Dependency injection guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- Captive dependency — Service lifetimes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped
- Service locator anti-pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines#avoid-service-locator

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Startup.ConfigureServices` | **Legacy**; replaced by `builder.Services` in .NET 6+. |
| `IServiceProvider` direct resolution | **Discouraged** in app logic (service locator anti-pattern). |
| Singleton capturing scoped | **Captive dependency**; throws with scope validation in development. |
| `AddDbContext` | Registers `DbContext` as **scoped** by default. |
| `GetService` vs. `GetRequiredService` | `GetService` returns `null`; `GetRequiredService` throws. |
| `ValidateScopes` | **Development only** by default; throws on lifetime mismatches. |
| `IKeyedServiceProvider` | **.NET 8+**; resolves keyed services. |
| `IServiceScopeFactory` in singletons | **Required** for scoped work in long-lived services. |

---

## References (Consolidated)

- Dependency injection in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
- Dependency injection in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Service lifetimes — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- Dependency injection guidelines — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- `IServiceCollection` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection
- `IServiceProvider` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/system.iserviceprovider
- `IServiceScopeFactory` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iservicescopefactory
- `ServiceDescriptor` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor
- `GetRequiredService` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.serviceproviderserviceextensions.getrequiredservice
- Scope validation — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scope-validation
- Captive dependency — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes#scoped
- Service locator anti-pattern — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines#avoid-service-locator