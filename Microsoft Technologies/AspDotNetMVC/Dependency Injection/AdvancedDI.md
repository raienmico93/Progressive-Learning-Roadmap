# ASP.NET Core Advanced Dependency Injection: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the built-in DI container (`Microsoft.Extensions.DependencyInjection`). Keyed services were introduced in **.NET 8**. Decorator support via **Scrutor** is third-party. Custom container replacement via **`IServiceProviderFactory<TContainerBuilder>`** is supported in all modern versions. Captive dependencies are the central advanced pitfall. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Advanced DI covers the features and patterns that go beyond basic `AddTransient`/`AddScoped`/`AddSingleton` registrations: factory functions, open generics, keyed services, decorators, and custom container factories. |
| **Technical Definition** | Advanced DI leverages the extensibility points of `IServiceCollection` and `IServiceProvider`: **factory delegates** (`Func<IServiceProvider, T>`) for conditional or deferred construction; **open generic registrations** (`typeof(IFoo<>)` / `typeof(Foo<>)`) for type-generic services; **keyed services** (`.NET 8+`) for multiple named implementations of the same interface; **decorators** to wrap existing registrations with cross-cutting behaviour; and **`IServiceProviderFactory<TContainerBuilder>`** to replace the built-in container with a third-party one (Autofac, DryIoc, etc.). |
| **Beginner-Friendly Explanation** | Advanced DI is **the toolbox for complex service setups**. Basic DI is “register this interface to this class.” Advanced DI answers: “How do I choose an implementation at runtime?” (factory functions), “How do I register a generic repository for every entity?” (open generics), “How do I have two different email senders?” (keyed services), “How do I add logging around an existing service?” (decorators), and “How do I swap out the whole container?” (`IServiceProviderFactory`). |

### Key Characteristics

- **Factory functions** – Defer or conditionally construct services using `Func<IServiceProvider, T>`.
- **Open generics** – Register a generic type once and resolve closed generic types automatically.
- **Keyed services** – Multiple implementations of the same interface, distinguished by a key.
- **Decorators** – Wrap a service with additional behaviour without modifying the original class.
- **Custom containers** – Replace the built-in container when advanced features (assembly scanning, interceptors) are needed.

### Prerequisites

- .NET SDK (6.0+; keyed services require .NET 8+).
- Working knowledge of **basic DI**, **service lifetimes**, and **constructor injection**.
- For decorators: `Scrutor` NuGet package (or manual registration).
- For custom containers: familiarity with a third-party container (Autofac, DryIoc).

### Related Programming Areas

- **Service registration** – `AddTransient`, `AddScoped`, `AddSingleton`.
- **Service lifetimes** – Transient, Scoped, Singleton.
- **Captive dependencies** – The primary advanced DI pitfall.
- **Factory pattern** – Manual factories vs. DI factory functions.
- **Strategy pattern** – Keyed services are a DI-native way to implement strategy selection.

### Core Concepts / Features

1. **Factory functions** – Deferred/conditional construction.
2. **Open generics** – Generic service registration.
3. **Keyed services** – Named multiple implementations (.NET 8+).
4. **Decorators** – Wrapping services with additional behaviour.
5. **`IServiceProviderFactory`** – Replacing the built-in container.
6. **Advanced DI best practices** – Avoiding captive dependencies and other pitfalls.

---

## 1. Factory Functions

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A factory function is a delegate (`Func<IServiceProvider, T>`) supplied at registration time that the container invokes to construct the service instance. |
| **Technical Definition** | `AddTransient<TService>(Func<IServiceProvider, TService> implementationFactory)`, `AddScoped<TService>(...)`, and `AddSingleton<TService>(...)` overloads accept a factory delegate. The container calls the delegate **each time the service is resolved** (for Transient/Scoped) or **once** (for Singleton). The delegate receives the `IServiceProvider`, enabling it to resolve other dependencies, inspect configuration, or choose an implementation conditionally. |
| **Beginner-Friendly Explanation** | A factory function is a **recipe you give the container instead of a class name**. You say: “To create an `IEmailSender`, look at the configuration and decide whether to return an SMTP sender or a fake sender.” The container runs your recipe whenever the service is needed. |

### Purposes

- **To** construct services conditionally based on configuration, environment, or runtime state.
- **To** resolve dependencies that are not available through constructor injection (e.g., a value from `IConfiguration`).
- **To** defer expensive construction until the service is actually needed.
- **To** create services that require manual wiring beyond what the container’s constructor selection can do.

### Syntax Rules and Structure

```csharp
// Non-generic
services.AddTransient(typeof(IMyService),
    sp => new MyService(sp.GetRequiredService<IDep>()));

// Generic
services.AddScoped<IMyService>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    return config["UseFake"] == "true"
        ? new FakeEmailSender()
        : new SmtpEmailSender(config["SmtpHost"]);
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Func<IServiceProvider, T>` | The factory delegate. |
| `sp` | The `IServiceProvider` for resolving other services. |
| Lifetime | Determined by `AddTransient`/`AddScoped`/`AddSingleton`. |

**Syntax rules:**

- Factory functions can resolve other services from the provided `IServiceProvider`.
- For Singleton, the factory runs **once**; for Transient/Scoped, it runs per resolution/scope.
- Factory functions can return `null`; the container will treat the service as unresolved (use with caution).

**Constraints and limitations:**

- **Captive dependencies** – A Singleton factory must not resolve a Scoped service and hold onto it.
- **Factory logic should be simple** – Complex branching belongs in a dedicated factory class or service.
- **No async factories** – The built-in container does not support asynchronous factory delegates.

### Annotated Code Example

```csharp
builder.Services.AddScoped<IEmailSender>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    var logger = sp.GetRequiredService<ILogger<SmtpEmailSender>>();

    if (config.GetValue<bool>("Email:UseFake"))
    {
        return new FakeEmailSender(logger);
    }

    return new SmtpEmailSender(
        config["Email:SmtpHost"],
        logger);
});
```

**Expected Output:** In development (where `Email:UseFake` is `true`), the container returns a `FakeEmailSender`. In production, it returns a real `SmtpEmailSender` configured from `appsettings.json`.

**Why this output?** The factory delegate inspects `IConfiguration` and conditionally constructs the appropriate implementation, a decision that cannot be expressed with a simple type mapping.

### Real-World Cases

- **Environment-specific implementations:** Fake email/SMS senders in development, real ones in production.
- **Feature flags:** Choose a premium vs. basic implementation based on a feature toggle.
- **Configuration-driven construction:** Build a service using connection strings or API keys resolved at startup.

### References

- `ServiceCollectionServiceExtensions.AddTransient` (factory overload) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addtransient
- Factory-based middleware activation – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility

---

## 2. Open Generics

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Open generic registration maps a generic interface (`IFoo<>`) to a generic implementation (`Foo<>`), allowing the container to construct closed generic types (`IFoo<Bar>`) on demand. |
| **Technical Definition** | `services.AddScoped(typeof(IFoo<>), typeof(Foo<>))` registers an **open generic** service. When a consumer requests `IFoo<Bar>`, the container closes the generic types and creates `Foo<Bar>`. Open generics eliminate the need to register every constructed generic type individually. If the generic type has multiple parameters, use the appropriate number of commas (`typeof(IQueryHandler<,>)`). |
| **Beginner-Friendly Explanation** | Open generics are a **“one registration for all T”** deal. If you have a generic repository `IRepository<T>`, you do not want to write `AddScoped<IRepository<Customer>, Repository<Customer>>()`, `AddScoped<IRepository<Order>, Repository<Order>>()`, and so on for every entity. Open generics let you write one line: “For any `T`, use `Repository<T>`.” |

### Purposes

- **To** avoid repetitive registrations for every closed generic type.
- **To** support generic infrastructure (repositories, validators, handlers) without per-type boilerplate.
- **To** keep registration code concise and maintainable as new entity types are added.
- **To** enable generic decorators and pipelines.

### Syntax Rules and Structure

```csharp
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
services.AddTransient(typeof(IValidator<>), typeof(Validator<>));
services.AddScoped(typeof(IQueryHandler<,>), typeof(QueryHandler<,>));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `typeof(IFoo<>)` | Open generic service type (no type argument). |
| `typeof(Foo<>)` | Open generic implementation type. |
| Lifetime | Transient, Scoped, or Singleton. |

**Syntax rules:**

- The number of generic parameters must match (`IFoo<,>` for two parameters).
- The implementation must be assignable to the service type for any valid type argument.
- Constraints on the generic parameters are respected by the container when closing the type.

**Constraints and limitations:**

- **Constraint violations** – If a consumer requests a closed type that violates the implementation’s generic constraints, resolution throws.
- **Decorators on open generics** require library support (e.g., Scrutor’s `Decorate`).

### Annotated Code Example

```csharp
public interface IRepository<T> { /* ... */ }
public class Repository<T> : IRepository<T> { /* ... */ }

// Registration
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));

// Consumer
public class CustomerService
{
    private readonly IRepository<Customer> _repo;
    public CustomerService(IRepository<Customer> repo) => _repo = repo;
}
```

**Expected Output:** `CustomerService` receives a `Repository<Customer>` instance without any explicit registration for `IRepository<Customer>`.

**Why this output?** The open generic registration tells the container: “For any closed `IRepository<T>`, create `Repository<T>`.”

### Real-World Cases

- **Generic repositories:** `IRepository<T>` for all entity types.
- **Validators:** `IValidator<T>` for request/command validation.
- **CQRS handlers:** `IQueryHandler<TQuery, TResult>` for every query type.

### References

- Dependency injection in ASP.NET Core (open generics) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Open generics in .NET DI (Stack Overflow) – https://stackoverflow.com/questions/33542623/open-generics-in-net-core-di

---

## 3. Keyed Services (.NET 8+)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Keyed services allow multiple implementations of the same interface to be registered and resolved using a **key** (string, enum, or other object). |
| **Technical Definition** | .NET 8 introduced `AddKeyedScoped`, `AddKeyedSingleton`, and `AddKeyedTransient`, which register a `ServiceDescriptor` with a `ServiceKey`. Resolution uses `GetRequiredKeyedService<T>(key)` or the `[FromKeyedServices("key")]` attribute on constructor parameters. A regular `GetService<T>()` returns `null` for keyed registrations; keyed resolution returns the implementation associated with the key. |
| **Beginner-Friendly Explanation** | Keyed services are **“labeled” services**. You can have an `IPaymentGateway` for Stripe and another `IPaymentGateway` for PayPal. You label one `"stripe"` and the other `"paypal"`. When a class needs the Stripe gateway, it asks for `[FromKeyedServices("stripe")] IPaymentGateway`, and the container hands over the right one. |

### Purposes

- **To** register multiple implementations of the same interface without ambiguity.
- **To** select an implementation at the point of injection using a key.
- **To** replace manual factory patterns for strategy selection.
- **To** keep registrations explicit and discoverable (keys act as names).

### Syntax Rules and Structure

#### Registration

```csharp
services.AddKeyedScoped<IPaymentGateway, StripeGateway>("stripe");
services.AddKeyedScoped<IPaymentGateway, PayPalGateway>("paypal");
services.AddKeyedTransient<INotificationSender, EmailSender>("email");
```

#### Resolution (Constructor Injection)

```csharp
public class CheckoutService(
    [FromKeyedServices("stripe")] IPaymentGateway gateway)
{
    // gateway is StripeGateway
}
```

#### Resolution (Service Provider)

```csharp
var gateway = provider.GetRequiredKeyedService<IPaymentGateway>("stripe");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddKeyedScoped<TService, TImpl>(key)` | Registers with a key. |
| `[FromKeyedServices("key")]` | Injects the implementation associated with the key. |
| `GetRequiredKeyedService<T>(key)` | Resolves programmatically. |

**Syntax rules:**

- Keys can be `string`, enum, integer, or any object; resolution compares keys with `Equals`.
- A service registered **without** a key is not returned by keyed resolution and vice versa.
- Keyed services respect lifetimes (Transient, Scoped, Singleton).

**Constraints and limitations:**

- **Requires .NET 8 or later**.
- **Keys must be compile-time constants** when used in `[FromKeyedServices]` (attribute arguments).
- **Keyed registrations are not returned by `GetService<T>()`**; you must use the keyed APIs.

### Annotated Code Example

```csharp
public interface INotificationSender
{
    Task SendAsync(string message);
}

public class EmailSender : INotificationSender { /* ... */ }
public class SmsSender : INotificationSender { /* ... */ }

// Registration
builder.Services.AddKeyedScoped<INotificationSender, EmailSender>("email");
builder.Services.AddKeyedScoped<INotificationSender, SmsSender>("sms");

// Consumer
public class NotificationService(
    [FromKeyedServices("email")] INotificationSender emailSender,
    [FromKeyedServices("sms")] INotificationSender smsSender)
{
    public Task NotifyAsync(string message)
    {
        // Use both senders
    }
}
```

**Expected Output:** `NotificationService` receives the `EmailSender` and `SmsSender` implementations, distinguished by their keys.

**Why this output?** `[FromKeyedServices]` instructs the container to resolve the implementation associated with the specified key.

### Real-World Cases

- **Payment gateways:** Stripe, PayPal, and Square implementations of `IPaymentGateway`.
- **Notification channels:** Email, SMS, and push implementations of `INotificationSender`.
- **Storage providers:** Local, S3, and Azure Blob implementations of `IFileStorage`.

### References

- Keyed services (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#keyed-services
- Keyed services in .NET 8 (GitHub) – https://github.com/immediateplatform/immediate.dev/blob/ce221f8dfcf904181ed8f602bdfa6294370f1cc3/src/content/docs/Immediate.Injections/keyed-services.md

---

## 4. Decorators

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A decorator wraps an existing service registration with another implementation that adds behaviour before or after delegating to the inner service. |
| **Technical Definition** | The decorator pattern adds cross-cutting behaviour (logging, caching, validation, retry) without modifying the original class. In .NET DI, decorators are not natively supported by the built-in container; they are typically implemented with the **`Scrutor`** library (`services.Decorate<TService, TDecorator>()`), which replaces the registration so that consumers receive the decorator, which in turn receives the original implementation. Manual decoration (registering the decorator explicitly and injecting the inner service) is also possible but more verbose. |
| **Beginner-Friendly Explanation** | A decorator is a **wrapper around a service**. Imagine an `IOrderService` that saves orders. You want to add logging without changing `OrderService`. A decorator (`LoggingOrderService`) wraps the real service: it logs “saving order,” calls the real service, and logs “order saved.” Consumers ask for `IOrderService` and get the wrapper, which adds logging transparently. |

### Purposes

- **To** add cross-cutting concerns (logging, caching, validation, retry, metrics) without modifying the original service.
- **To** compose behaviour in layers (multiple decorators can wrap one service).
- **To** adhere to the Open/Closed Principle—services are open for extension but closed for modification.
- **To** keep core business logic clean and focused.

### Syntax Rules and Structure (Scrutor)

```csharp
// Install-Package Scrutor

services.AddScoped<IOrderService, OrderService>();
services.Decorate<IOrderService, LoggingOrderService>();
services.Decorate<IOrderService>((inner, provider) =>
    new CachingOrderService(inner, provider.GetRequiredService<ICache>()));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddScoped<IOrderService, OrderService>()` | Registers the inner service. |
| `Decorate<IOrderService, LoggingOrderService>()` | Wraps the registration with a decorator. |
| Factory overload | Allows constructing the decorator with additional dependencies. |

**Syntax rules:**

- **`Decorate` must be called after the service is registered.**
- Multiple decorators can be chained; resolution returns the outermost decorator.
- The decorator’s constructor must accept the inner service type (e.g., `IOrderService`).

**Constraints and limitations:**

- **Scrutor is a third-party package**; the built-in container does not support decorators natively.
- **Open generic decorators** require Scrutor’s `Decorate` with open generic types.
- **Order of decoration matters**—the last registered decorator is the outermost.

### Annotated Code Example

```csharp
public interface IOrderService
{
    Task PlaceOrderAsync(Order order);
}

public class OrderService : IOrderService
{
    public async Task PlaceOrderAsync(Order order) { /* core logic */ }
}

public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;

    public LoggingOrderService(IOrderService inner,
        ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task PlaceOrderAsync(Order order)
    {
        _logger.LogInformation("Placing order {OrderId}", order.Id);
        await _inner.PlaceOrderAsync(order);
        _logger.LogInformation("Order {OrderId} placed", order.Id);
    }
}

// Registration
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.Decorate<IOrderService, LoggingOrderService>();
```

**Expected Output:** When `IOrderService` is resolved, consumers receive `LoggingOrderService`, which logs before and after delegating to `OrderService`.

**Why this output?** Scrutor replaces the `IOrderService` registration with the decorator, injecting the original `OrderService` into the decorator’s constructor.

### Real-World Cases

- **Logging:** Wrap repositories or services to log method calls and parameters.
- **Caching:** Wrap expensive services with a cache decorator that returns cached results.
- **Validation:** Wrap command handlers with a validation decorator.
- **Retry policies:** Wrap HTTP clients or database operations with retry logic.

### References

- Scrutor (GitHub) – https://github.com/dolunay/Scrutor
- Decorator pattern in .NET DI (Microsoft Learn) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines

---

## 5. `IServiceProviderFactory`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IServiceProviderFactory<TContainerBuilder>` is an extension point that allows a third-party DI container to replace the built-in container while still using the `IServiceCollection` registration API. |
| **Technical Definition** | `IServiceProviderFactory<TContainerBuilder>` (in `Microsoft.Extensions.DependencyInjection`) defines two methods: `TContainerBuilder CreateBuilder(IServiceCollection services)` and `IServiceProvider CreateServiceProvider(TContainerBuilder containerBuilder)`. It is registered via `builder.Host.UseServiceProviderFactory(factory)` (or `builder.WebHost.UseServiceProviderFactory`). The factory adapts the `IServiceCollection` registrations to the third-party container’s builder and produces an `IServiceProvider` backed by that container. |
| **Beginner-Friendly Explanation** | `IServiceProviderFactory` is the **“adapter”** that lets you swap the built-in DI container for a more powerful one (Autofac, DryIoc, etc.) without rewriting all your registrations. You keep using `builder.Services.AddScoped(...)` as usual, but the factory translates those registrations into the third-party container’s configuration and returns a provider that the rest of ASP.NET Core uses. |

### Purposes

- **To** replace the built-in container with a third-party container that offers advanced features (assembly scanning, interceptors, child containers, property injection).
- **To** preserve the `IServiceCollection` registration API while changing the underlying container implementation.
- **To** integrate containers like Autofac, DryIoc, or Simple Injector into ASP.NET Core.
- **To** control the lifetime and disposal of the root service provider.

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    containerBuilder.RegisterType<MyService>().As<IMyService>();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseServiceProviderFactory(factory)` | Registers the custom factory. |
| `AutofacServiceProviderFactory` | Example factory for Autofac. |
| `ConfigureContainer<ContainerBuilder>` | Configures the third-party container directly. |

**Syntax rules:**

- The factory must implement `IServiceProviderFactory<TContainerBuilder>`.
- `CreateBuilder` receives the `IServiceCollection` and returns the container-specific builder.
- `CreateServiceProvider` receives the builder and returns the `IServiceProvider`.
- `ConfigureContainer` is called after `CreateBuilder` to allow direct container configuration.

**Constraints and limitations:**

- **Third-party container required** – The built-in container does not need (or implement) this interface for itself.
- **Some built-in container features may not map 1:1** to third-party containers.
- **Performance and behaviour may differ** from the built-in container; test thoroughly.

### Annotated Code Example (Autofac)

```csharp
using Autofac;
using Autofac.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);

// Replace the built-in container with Autofac
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    containerBuilder.RegisterType<SystemClock>()
        .As<IClock>()
        .SingleInstance();

    containerBuilder.RegisterType<OrderRepository>()
        .As<IOrderRepository>()
        .InstancePerLifetimeScope();
});

builder.Services.AddControllersWithViews();

var app = builder.Build();
```

**Expected Output:** The application uses Autofac to resolve services. `IClock` is a singleton, `IOrderRepository` is scoped (per lifetime scope/request).

**Why this output?** The `AutofacServiceProviderFactory` translates `IServiceCollection` registrations into Autofac registrations, and `ConfigureContainer` allows direct Autofac configuration.

### Real-World Cases

- **Large applications** needing assembly scanning (Autofac).
- **Applications requiring interceptors** (AOP) for logging or transactions.
- **Teams with existing investments** in a specific DI container.

### References

- `IServiceProviderFactory<TContainerBuilder>` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderfactory-1
- Autofac integration with ASP.NET Core – https://github.com/autofac/Autofac.Extensions.DependencyInjection

---

## 6. Advanced DI Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Advanced DI best practices are guidelines for using factories, open generics, keyed services, decorators, and custom containers without introducing lifetime bugs, performance problems, or maintainability issues. |
| **Technical Definition** | The central advanced DI concern is the **captive dependency**: a service with a longer lifetime (Singleton) holding a reference to a service with a shorter lifetime (Scoped/Transient). Other concerns include avoiding service locator patterns, ensuring thread safety for Singletons, keeping factory logic simple, and not overusing decorators or custom containers. |
| **Beginner-Friendly Explanation** | Advanced DI best practices are the **“do’s and don’ts”** for complex DI setups. The biggest “don’t” is letting a Singleton accidentally hold onto a per-request database context—that causes memory leaks, stale data, and crashes under load. The “do’s” include using `IServiceScopeFactory` when a Singleton truly needs a Scoped service, and keeping advanced features (decorators, keyed services) focused and well-documented. |

### Purposes

- **To** prevent captive dependencies and the bugs they cause (memory leaks, stale data, concurrency crashes).
- **To** keep Singletons thread-safe and free of Scoped state.
- **To** avoid over-engineering with advanced DI features when simpler registrations suffice.
- **To** ensure decorators and keyed services are applied consistently and documented.
- **To** validate the container configuration at startup (scope validation).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Captive dependencies** | Ensure a Singleton never depends on a Scoped service. | Inject `DbContext` (Scoped) into a Singleton. |
| **Scoped in Singletons** | Use `IServiceScopeFactory` to create an explicit scope inside a Singleton. | Resolve a Scoped service directly from a Singleton’s constructor. |
| **Thread safety** | Ensure Singletons are thread-safe (use `ConcurrentDictionary`, locks, etc.). | Hold mutable state in a Singleton without synchronisation. |
| **Factory simplicity** | Keep factory delegates simple; move complex logic to a dedicated factory class. | Put business logic or I/O inside a factory delegate. |
| **Keyed services** | Use keyed services for strategy selection; document keys. | Overuse keyed services when a simple interface + one implementation suffices. |
| **Decorators** | Apply decorators for cross-cutting concerns (logging, caching, validation). | Stack many decorators without measuring performance impact. |
| **Custom containers** | Use `IServiceProviderFactory` only when the built-in container lacks required features. | Replace the built-in container without a clear need. |
| **Scope validation** | Keep `ValidateScopes` and `ValidateOnBuild` enabled in Development. | Disable scope validation and risk silent captive dependencies. |

### Annotated Code Example (Captive Dependency Fix)

```csharp
// ❌ WRONG: Singleton captures Scoped DbContext
public class OrderProcessingWorker : BackgroundService
{
    private readonly AppDbContext _db;   // Captive!
    public OrderProcessingWorker(AppDbContext db) => _db = db;
}

// ✅ CORRECT: Use IServiceScopeFactory to create a scope per iteration
public class OrderProcessingWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public OrderProcessingWorker(IServiceScopeFactory scopeFactory)
        => _scopeFactory = scopeFactory;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            // Use db within the scope
            await db.Orders.ToListAsync(stoppingToken);
        }
    }
}
```

**Why this follows best practices:**

- **The incorrect version** injects a Scoped `DbContext` into a Singleton (`BackgroundService`), causing the context to live for the entire application lifetime—stale data, memory leaks, and thread-safety crashes.
- **The correct version** injects `IServiceScopeFactory`, creates an explicit scope per iteration, and resolves the `DbContext` from that scope. The context is disposed at the end of each scope.

### Real-World Cases

- **Background services:** Use `IServiceScopeFactory` for Scoped services.
- **Singleton caches:** Ensure thread safety and avoid capturing Scoped services.
- **Decorators in production:** Monitor performance impact of stacked decorators.
- **Keyed services:** Document keys in a central location to avoid typos.

### References

- Dependency injection guidelines (captive dependencies) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Captive dependency (NDLRCOR012) – https://github.com/dotnet/docs/blob/main/docs/core/extensions/dependency-injection/guidelines.md
- ASP.NET Core DI pitfalls (Scope validation) – https://weblogs.asp.net/ricardoperes/asp-net-core-pitfalls-scope-validation

---

## Summary Table: Advanced DI Features at a Glance

| Feature | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|------------------|-------------|
| **Factory functions** | Conditional/deferred construction | `AddScoped<T>(sp => ...)` | Environment-specific or configuration-driven implementations |
| **Open generics** | Generic service registration | `AddScoped(typeof(IFoo<>), typeof(Foo<>))` | Generic repositories, validators, handlers |
| **Keyed services** | Multiple named implementations | `AddKeyedScoped<T, TImpl>("key")`, `[FromKeyedServices]` | Strategy selection (payment gateways, notification channels) |
| **Decorators** | Wrap services with cross-cutting behaviour | Scrutor `services.Decorate<TService, TDecorator>()` | Logging, caching, validation, retry |
| **`IServiceProviderFactory`** | Replace the built-in container | `UseServiceProviderFactory(factory)` | Advanced container features (Autofac, DryIoc) |
| **Best practices** | Avoid captive dependencies and lifetime bugs | `IServiceScopeFactory`, scope validation, thread safety | Every advanced DI setup |

---

## References (Full List)

- Dependency injection in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Dependency injection guidelines – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines
- Service lifetimes – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes
- `ServiceCollectionServiceExtensions.AddTransient` (factory overload) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions.addtransient
- `IServiceProviderFactory<TContainerBuilder>` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderfactory-1
- Keyed services – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#keyed-services
- Scrutor (GitHub) – https://github.com/dolunay/Scrutor
- Autofac integration with ASP.NET Core – https://github.com/autofac/Autofac.Extensions.DependencyInjection
- Captive dependency (NDLRCOR012) – https://github.com/dotnet/docs/blob/main/docs/core/extensions/dependency-injection/guidelines.md
- ASP.NET Core DI pitfalls (Scope validation) – https://weblogs.asp.net/ricardoperes/asp-net-core-pitfalls-scope-validation