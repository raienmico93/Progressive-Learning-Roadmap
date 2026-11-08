# ASP.NET Core Custom Middleware: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the minimal hosting model (`WebApplication`). It covers the two middleware authoring models—**convention-based middleware** (the default) and **factory-activated middleware** via `IMiddleware`—along with `RequestDelegate`, `Invoke`/`InvokeAsync`, extension methods, and best practices. Custom middleware is the primary extension point for request-processing logic that is not covered by built-in middleware. The APIs are stable; factory-activated middleware has been available since early ASP.NET Core and remains the recommended approach when Scoped services are needed in the middleware constructor. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Custom middleware is user-authored software that is assembled into the ASP.NET Core request pipeline to handle requests and responses. |
| **Technical Definition** | A custom middleware component is a class (or inline delegate) that receives an `HttpContext` and a `RequestDelegate` (`next`), performs work before and/or after calling `next`, and can **short-circuit** the pipeline by not calling `next`. Middleware is encapsulated in a class and exposed through an extension method on `IApplicationBuilder`. Two activation models exist: **convention-based** (constructor receives `RequestDelegate`; `Invoke`/`InvokeAsync` processes requests; instantiated once at startup) and **factory-activated** (`IMiddleware` implementation registered in DI; activated per request; Scoped services can be constructor-injected). |
| **Beginner-Friendly Explanation** | Custom middleware is **your own processing station** in the request pipeline. ASP.NET Core provides built-in stations (authentication, static files, routing), but when you need something specific—logging every request, adding a custom header, validating an API key—you write your own station. You write a class with a method that receives the request and a “next” delegate, do your work, and then decide whether to pass the request along or stop it. |

### Key Characteristics

- **Two authoring models** – Convention-based middleware (default) and factory-activated middleware (`IMiddleware`).
- **Pipeline participation** – Middleware can run before and after the next component and can short-circuit the chain.
- **DI integration** – Dependencies are injected via constructor (convention-based) or constructor + `InvokeAsync` parameters.
- **Extension-method exposure** – Middleware is typically registered via a custom `Use[MiddlewareName]()` extension method.
- **Per-request activation option** – `IMiddleware` is resolved from DI per request, enabling Scoped service injection.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware pipelines**, **service lifetimes**, and **dependency injection**.
- Familiarity with `Program.cs` (minimal hosting) and C# `async`/`await`.

### Related Programming Areas

- **Middleware pipeline** – The chain through which requests and responses flow.
- **Dependency injection** – Constructor and method injection of services into middleware.
- **Service lifetimes** – Scoped services require `IMiddleware` or `InvokeAsync` parameters.
- **Routing** – Middleware can run before, between, or after routing.
- **Exception handling** – Middleware can wrap downstream processing in try/catch.

### Core Concepts / Features

1. **Custom middleware** – The concept, patterns, and authoring models.
2. **`IMiddleware`** – Factory-activated middleware interface.
3. **`RequestDelegate`** – The delegate type representing the next middleware in the pipeline.
4. **`Invoke()`** – The synchronous convention-based middleware method.
5. **`InvokeAsync()`** – The asynchronous convention-based middleware method (recommended).
6. **Middleware extension methods** – `Use[MiddlewareName]()` extensions for registration.
7. **Custom middleware best practices** – Lifetime, dependency, and performance guidelines.

---

## 1. Custom Middleware (The Concept and Authoring Models)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Custom middleware is a user-defined component that processes HTTP requests and responses in the ASP.NET Core pipeline. |
| **Technical Definition** | Custom middleware can be written **inline** (`app.Use(async (context, next) => ...)`) or as a **class**. Class-based middleware follows two models: **convention-based** (public constructor with `RequestDelegate`; public `Invoke` or `InvokeAsync` method; instantiated once per application lifetime) and **factory-activated** (implements `IMiddleware`; `InvokeAsync(HttpContext, RequestDelegate)`; registered in DI as Scoped or Transient; activated per request). Middleware is generally encapsulated in a class and exposed with an extension method. |
| **Beginner-Friendly Explanation** | Custom middleware is **your own step in the request-processing assembly line**. You can write it as a quick inline lambda for simple logic, or as a proper class for reusable, testable middleware. The class-based approach is preferred for anything beyond the simplest cases. |

### Purposes

- **To** implement request-processing logic that is not covered by built-in middleware.
- **To** encapsulate cross-cutting concerns (logging, custom headers, API key validation, tenant resolution) in a reusable component.
- **To** short-circuit requests that should not proceed (e.g., blocked IPs, missing headers).
- **To** run logic before and after downstream processing (timing, logging, response modification).
- **To** integrate with DI for dependencies while respecting service lifetimes.

### Syntax Rules and Structure

#### Inline Middleware

```csharp
app.Use(async (context, next) =>
{
    // Before next
    await next(context);
    // After next
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context` | The `HttpContext`. |
| `next` | The `RequestDelegate` representing the rest of the pipeline. |

#### Convention-Based Class Middleware

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        // Before
        await _next(context);
        // After
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor | Receives `RequestDelegate next`. |
| `InvokeAsync` | Processes the request; must return `Task`. |

#### Factory-Activated Middleware (`IMiddleware`)

```csharp
public class MyFactoryMiddleware : IMiddleware
{
    private readonly IMyScopedService _service;

    public MyFactoryMiddleware(IMyScopedService service) => _service = service;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Use Scoped service
        await next(context);
    }
}

// Registration
builder.Services.AddScoped<MyFactoryMiddleware>();
app.UseMiddleware<MyFactoryMiddleware>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IMiddleware` | Interface with `InvokeAsync(HttpContext, RequestDelegate)`. |
| Constructor | Can receive Scoped services because activation is per request. |
| Registration | Must be registered in DI (Scoped or Transient). |

**Syntax rules:**

- **Convention-based middleware** is instantiated **once** at app startup; its constructor runs once per application lifetime.
- **Convention-based middleware** can inject Scoped services into `InvokeAsync` parameters (not the constructor).
- **`IMiddleware`** is activated **per request**, so Scoped services can be constructor-injected.
- Middleware is generally exposed via an extension method (see Section 6).

**Constraints and limitations:**

- **Convention-based middleware constructor** must not depend on Scoped services (captive dependency risk).
- **`IMiddleware`** must be registered in the DI container; convention-based middleware does not require registration.
- **`UseMiddleware<T>()` cannot pass arbitrary arguments** to factory-activated middleware; use constructor injection instead.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Convention-Based Middleware with `InvokeAsync`

```csharp
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(RequestDelegate next,
        ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var sw = Stopwatch.StartNew();
        await _next(context);
        sw.Stop();
        _logger.LogInformation("{Method} {Path} took {Elapsed}ms",
            context.Request.Method, context.Request.Path,
            sw.ElapsedMilliseconds);
    }
}

// Program.cs
var app = builder.Build();
app.UseMiddleware<RequestTimingMiddleware>();
```

**Step-by-step explanation:**

1. The constructor receives `RequestDelegate next` and `ILogger<T>` (Singleton service, safe for constructor injection).
2. `InvokeAsync` starts a stopwatch, calls `next`, stops the stopwatch, and logs.
3. `UseMiddleware<T>()` adds the middleware to the pipeline.

**Expected Output:** Each request logs its method, path, and duration after downstream processing completes.

**Why this output?** The middleware runs before `_next` (starting the stopwatch) and after `_next` (stopping and logging), wrapping the rest of the pipeline.

### Real-World Cases

- **Request logging and correlation IDs:** Attach identifiers and log request metadata.
- **Custom security headers:** Add `X-Content-Type-Options`, `X-Frame-Options`, etc. to every response.
- **API key validation:** Reject requests without a valid API key before they reach endpoints.
- **Tenant resolution:** Identify the tenant from the host or header and store it in `HttpContext.Items`.

### References

- Write custom ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Factory-based middleware activation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility

---

## 2. `IMiddleware`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IMiddleware` is an interface that defines middleware for the application’s request pipeline with strong typing and per-request activation. |
| **Technical Definition** | `IMiddleware` (in `Microsoft.AspNetCore.Http`) declares `Task InvokeAsync(HttpContext context, RequestDelegate next)`. Middleware implementing this interface is **factory-activated**: `UseMiddleware` extension methods check whether the registered middleware type implements `IMiddleware`; if so, the `IMiddlewareFactory` resolves the implementation from DI **instead of** using convention-based activation. The middleware is registered as a Scoped or Transient service. `IMiddleware` is activated **per client request (connection)**, so Scoped services **can** be injected into the constructor. |
| **Beginner-Friendly Explanation** | `IMiddleware` is the **strongly typed, per-request** middleware interface. If your middleware needs a Scoped service (like a `DbContext`) in its constructor, you implement `IMiddleware` and register it in DI. The framework creates a new middleware instance for each request, so Scoped services work correctly. |

### Purposes

- **To** enable **per-request middleware activation**, allowing Scoped services in the constructor.
- **To** provide **strong typing** for middleware implementations.
- **To** support middleware that needs request-specific state or scoped dependencies.
- **To** offer an alternative to convention-based middleware when Scoped services are required.

### Syntax Rules and Structure

```csharp
public class FactoryActivatedMiddleware : IMiddleware
{
    private readonly IMyScopedService _service;

    public FactoryActivatedMiddleware(IMyScopedService service)
        => _service = service;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Use _service
        await next(context);
    }
}

// Registration
builder.Services.AddScoped<FactoryActivatedMiddleware>();
app.UseMiddleware<FactoryActivatedMiddleware>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IMiddleware` | Interface with `InvokeAsync`. |
| Constructor | Receives Scoped services (because activation is per request). |
| `InvokeAsync` | Receives `HttpContext` and `RequestDelegate`. |
| Registration | `AddScoped` or `AddTransient` in DI. |

**Syntax rules:**

- Must be registered in the DI container (Scoped or Transient).
- `UseMiddleware<T>()` automatically uses the `IMiddlewareFactory` for `IMiddleware` implementations.
- Scoped services **can** be constructor-injected.

**Constraints and limitations:**

- **Cannot pass additional parameters** to factory-activated middleware via `UseMiddleware`; use constructor injection.
- **Per-request activation** has a slight performance cost compared to singleton convention-based middleware.
- **Must be registered in DI**—unlike convention-based middleware, which does not require registration.

### Annotated Code Example

```csharp
public class AuditMiddleware : IMiddleware
{
    private readonly IAuditService _audit;

    public AuditMiddleware(IAuditService audit) => _audit = audit;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        await _audit.LogRequestAsync(context);
        await next(context);
    }
}

// Program.cs
builder.Services.AddScoped<IAuditService, AuditService>();
builder.Services.AddScoped<AuditMiddleware>();

var app = builder.Build();
app.UseMiddleware<AuditMiddleware>();
```

**Expected Output:** Each request is audited using a Scoped `IAuditService` instance, and the pipeline continues normally.

**Why this output?** `AuditMiddleware` implements `IMiddleware`, so `UseMiddleware` resolves it from DI per request, allowing the Scoped `IAuditService` to be constructor-injected.

### Real-World Cases

- **Database-backed audit logging:** Middleware that writes to a `DbContext` (Scoped).
- **Per-request tenant resolution:** Resolving tenant information from a Scoped service.
- **User-specific processing:** Middleware that depends on the authenticated user’s Scoped services.

### References

- Factory-based middleware activation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility
- `IMiddleware` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware

---

## 3. `RequestDelegate`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `RequestDelegate` is a delegate that represents the next middleware in the request pipeline. |
| **Technical Definition** | `RequestDelegate` (in `Microsoft.AspNetCore.Http`) is defined as `delegate Task RequestDelegate(HttpContext context)`. Middleware receives this delegate (conventionally named `next`) and invokes it to pass control to the next middleware. If the delegate is not invoked, the pipeline is **short-circuited**. `RequestDelegate` is the backbone of the middleware pipeline: each middleware is constructed with a `RequestDelegate` that represents the rest of the pipeline. |
| **Beginner-Friendly Explanation** | `RequestDelegate` is the **“next station” button**. When your middleware is done with its work, it calls `next(context)` to pass the request along. If it does not call `next`, the request stops there. |

### Purposes

- **To** pass control from one middleware to the next in the pipeline.
- **To** enable short-circuiting by simply not invoking the delegate.
- **To** provide a uniform contract for all middleware components.
- **To** support composition of middleware as a chain of delegates.

### Syntax Rules and Structure

```csharp
public delegate Task RequestDelegate(HttpContext context);
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `HttpContext` | The current request context. |
| Return `Task` | Asynchronous completion of the middleware. |

**Syntax rules:**

- Convention-based middleware receives `RequestDelegate next` in its constructor.
- `IMiddleware.InvokeAsync` receives `RequestDelegate next` as a parameter.
- Inline middleware receives `next` as the second parameter of the `Use` delegate.

**Constraints and limitations:**

- **Do not invoke `next` more than once** unless you fully understand the consequences (the pipeline is not designed for re-entry).
- **Do not call `next` after the response has started**—it will throw or cause undefined behaviour.
- **Short-circuiting** is achieved by not calling `next`; this is a deliberate design decision.

### Annotated Code Example

```csharp
public class ShortCircuitMiddleware
{
    private readonly RequestDelegate _next;
    public ShortCircuitMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        if (context.Request.Headers["X-Block"] == "true")
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Blocked");
            return;   // Do NOT call _next — short-circuit
        }
        await _next(context);
    }
}
```

**Expected Output:** A request with the `X-Block: true` header receives a 403 response; downstream middleware never run. A request without the header passes through normally.

**Why this output?** The middleware conditionally decides not to invoke `_next`, terminating the pipeline early.

### Real-World Cases

- **Blocking requests:** Rejecting requests from blocked IPs or without required headers.
- **Health-check endpoints:** Short-circuiting `/health` to avoid unnecessary processing.
- **Static file serving:** Built-in static files middleware short-circuits the pipeline for file requests.

### References

- `RequestDelegate` delegate – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.requestdelegate
- ASP.NET Core middleware (request delegates) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/

---

## 4. `Invoke()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Invoke()` is the synchronous method name that convention-based middleware can use to process requests. |
| **Technical Definition** | Convention-based middleware must expose a public method named either `Invoke` or `InvokeAsync`. `Invoke` returns `Task` and accepts `HttpContext` as its first parameter. `InvokeAsync` is preferred for asynchronous work; `Invoke` is available for compatibility and simple synchronous scenarios. Additional parameters are populated by DI. |
| **Beginner-Friendly Explanation** | `Invoke()` is the **synchronous processing method** for convention-based middleware. You can use it if your middleware does not need to `await` anything, but `InvokeAsync()` is generally preferred because it is more flexible. |

### Purposes

- **To** provide a synchronous entry point for convention-based middleware.
- **To** support simple middleware that does not perform asynchronous I/O.
- **To** maintain compatibility with older middleware implementations.

### Syntax Rules and Structure

```csharp
public Task Invoke(HttpContext context)
{
    // Synchronous work
    return _next(context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `Invoke` | Method name. |
| `HttpContext` | First parameter. |
| Return `Task` | Required return type. |

**Syntax rules:**

- Must be `public`.
- Must return `Task`.
- Must accept `HttpContext` as the first parameter.
- Additional parameters are resolved from DI.

**Constraints and limitations:**

- **Synchronous only**—cannot use `await` inside `Invoke` (use `InvokeAsync` for asynchronous work).
- **Less flexible** than `InvokeAsync` for modern asynchronous code.

### Annotated Code Example

```csharp
public class SimpleHeaderMiddleware
{
    private readonly RequestDelegate _next;
    public SimpleHeaderMiddleware(RequestDelegate next) => _next = next;

    public Task Invoke(HttpContext context)
    {
        context.Response.Headers.Append("X-Custom", "Value");
        return _next(context);
    }
}
```

**Expected Output:** Every response includes the `X-Custom: Value` header.

**Why this output?** The synchronous `Invoke` method modifies the response headers before calling `_next`.

### Real-World Cases

- **Simple header addition:** Adding a static header to every response.
- **Legacy middleware:** Maintaining older middleware that predates `InvokeAsync`.
- **Trivial synchronous checks:** Very lightweight processing with no I/O.

### References

- Write custom ASP.NET Core middleware (`Invoke`/`InvokeAsync` requirements) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write

---

## 5. `InvokeAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `InvokeAsync()` is the asynchronous method name used by convention-based middleware and the method defined by `IMiddleware` for processing requests. |
| **Technical Definition** | Convention-based middleware can implement `public Task InvokeAsync(HttpContext context, ...)`. `IMiddleware` defines `Task InvokeAsync(HttpContext context, RequestDelegate next)`. `InvokeAsync` supports `await` for asynchronous I/O and can accept additional parameters (convention-based) that are populated by DI. It is the **recommended** method name for middleware because it supports asynchronous work and aligns with modern ASP.NET Core patterns. |
| **Beginner-Friendly Explanation** | `InvokeAsync()` is the **asynchronous processing method**. It is preferred over `Invoke()` because it can `await` asynchronous operations—like database calls, HTTP requests, or file I/O—without blocking threads. |

### Purposes

- **To** support asynchronous request processing without blocking threads.
- **To** serve as the method defined by `IMiddleware`.
- **To** enable DI of additional parameters in convention-based middleware.
- **To** align with the asynchronous nature of ASP.NET Core.

### Syntax Rules and Structure

```csharp
public async Task InvokeAsync(HttpContext context)
{
    // await asynchronous work
    await _next(context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `InvokeAsync` | Method name. |
| `async Task` | Asynchronous return type. |
| `HttpContext` | First parameter. |
| Additional parameters (optional) | Resolved from DI. |

**Syntax rules:**

- Must be `public`.
- Must return `Task`.
- Must accept `HttpContext` as the first parameter.
- Additional parameters are resolved from DI (convention-based only).

**Constraints and limitations:**

- **`IMiddleware.InvokeAsync`** also receives `RequestDelegate next` as a parameter.
- **Convention-based `InvokeAsync`** receives `next` via the constructor (stored in `_next`).

### Annotated Code Example (Convention-Based)

```csharp
public class AsyncAuditMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<AsyncAuditMiddleware> _logger;

    public AsyncAuditMiddleware(RequestDelegate next,
        ILogger<AsyncAuditMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context,
        IAuditService audit)
    {
        await audit.LogAsync(context);
        await _next(context);
    }
}
```

**Expected Output:** The middleware awaits the audit service and then passes control downstream.

**Why this output?** `InvokeAsync` supports `await`, and the `IAuditService` is resolved from DI as an additional parameter (Scoped services can be method-injected in convention-based middleware).

### Real-World Cases

- **Asynchronous logging:** Awaiting a logging service or database write.
- **HTTP calls:** Middleware that enriches requests with data from an external service.
- **File I/O:** Reading or writing files asynchronously.

### References

- Write custom ASP.NET Core middleware (`InvokeAsync`) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- `IMiddleware.InvokeAsync` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware.invokeasync

---

## 6. Middleware Extension Methods

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A middleware extension method is a custom extension on `IApplicationBuilder` (e.g., `UseMyMiddleware()`) that encapsulates middleware registration. |
| **Technical Definition** | Middleware is generally encapsulated in a class and exposed with an extension method. The extension method typically calls `app.UseMiddleware<TMiddleware>()` and may accept configuration parameters (e.g., options). `UseMiddleware` can accept additional parameters directly, which are passed to the middleware constructor. Extension methods provide a clean, discoverable API for adding middleware to the pipeline. |
| **Beginner-Friendly Explanation** | An extension method is a **custom `Use...` method** that you write so other developers (or future you) can add your middleware with a single, readable line. Instead of `app.UseMiddleware<MyComplexMiddleware>()`, you write `app.UseMyComplexMiddleware(options)`. |

### Purposes

- **To** provide a clean, discoverable API for middleware registration.
- **To** encapsulate configuration parameters and pass them to the middleware.
- **To** follow the `Use[MiddlewareName]` convention used by built-in middleware.
- **To** improve readability in `Program.cs`.

### Syntax Rules and Structure

```csharp
public static class MyMiddlewareExtensions
{
    public static IApplicationBuilder UseMyMiddleware(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<MyMiddleware>();
    }

    public static IApplicationBuilder UseMyMiddleware(
        this IApplicationBuilder builder,
        MyMiddlewareOptions options)
    {
        return builder.UseMiddleware<MyMiddleware>(options);
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `this IApplicationBuilder` | The extension point. |
| `UseMiddleware<T>()` | Internal call to add the middleware. |
| Options parameter (optional) | Passed to the middleware constructor. |

**Syntax rules:**

- Extension methods should be in a static class.
- Return `IApplicationBuilder` for fluent chaining.
- The method name conventionally starts with `Use`.
- Configuration objects can be passed to the middleware constructor.

**Constraints and limitations:**

- **Factory-activated middleware (`IMiddleware`)** cannot receive additional parameters via `UseMiddleware`; use constructor injection instead.
- **Convention-based middleware** can receive additional constructor parameters via `UseMiddleware`.

### Annotated Code Example

```csharp
public class MyMiddlewareOptions
{
    public string HeaderName { get; set; } = "X-Custom";
    public string HeaderValue { get; set; } = "Default";
}

public class MyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly MyMiddlewareOptions _options;

    public MyMiddleware(RequestDelegate next, MyMiddlewareOptions options)
    {
        _next = next;
        _options = options;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.Headers.Append(_options.HeaderName, _options.HeaderValue);
        await _next(context);
    }
}

public static class MyMiddlewareExtensions
{
    public static IApplicationBuilder UseMyMiddleware(
        this IApplicationBuilder builder, MyMiddlewareOptions options)
    {
        return builder.UseMiddleware<MyMiddleware>(options);
    }
}

// Program.cs
app.UseMyMiddleware(new MyMiddlewareOptions
{
    HeaderName = "X-Powered-By",
    HeaderValue = "MyApp"
});
```

**Expected Output:** Every response includes the `X-Powered-By: MyApp` header.

**Why this output?** The extension method passes the options object to the middleware constructor via `UseMiddleware`.

### Real-World Cases

- **Configurable security headers:** Passing header names and values via options.
- **Environment-specific middleware:** Passing environment flags to conditionally enable logic.
- **Named policies:** Passing policy names or configuration objects.

### References

- Write custom ASP.NET Core middleware (extension methods) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- `UseMiddlewareExtensions.UseMiddleware` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.usemiddlewareextensions.usemiddleware

---

## 7. Custom Middleware Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Custom middleware best practices are guidelines for writing middleware that is correct, secure, performant, and maintainable. |
| **Technical Definition** | Best practices encompass choosing the correct activation model (convention-based vs. `IMiddleware`), respecting service lifetimes (avoiding captive dependencies in convention-based constructors), keeping middleware focused and stateless, using `async`/`await` throughout, avoiding blocking calls, and ensuring correct pipeline placement. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for custom middleware—they help you avoid thread starvation, captive dependencies, middleware that silently never runs, and code that is hard to test. |

### Purposes

- **To** prevent performance problems (blocking calls, thread starvation).
- **To** avoid lifetime bugs (captive dependencies, Scoped services in Singletons).
- **To** keep middleware testable, maintainable, and focused.
- **To** ensure middleware runs when and where intended (correct placement).
- **To** align with the Explicit Dependencies Principle.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Activation model** | Use `IMiddleware` when Scoped services are needed in the constructor. | Inject Scoped services into a convention-based middleware constructor (captive dependency). |
| **Async** | Use `InvokeAsync` and `await` throughout. | Use blocking calls (`.Result`, `.Wait()`)—causes thread starvation. |
| **Dependencies** | Follow the Explicit Dependencies Principle—expose dependencies in the constructor. | Use service locator (`context.RequestServices.GetService`) inside middleware logic. |
| **Single responsibility** | Keep middleware focused on one concern. | Create “god” middleware that does logging, auth, and header manipulation. |
| **Short-circuiting** | Short-circuit deliberately and document it. | Short-circuit accidentally by forgetting to call `next`. |
| **Pipeline placement** | Place middleware according to its dependencies (before/after routing, auth, etc.). | Place middleware after terminal middleware and expect it to run. |
| **Request body** | Call `context.Request.EnableBuffering()` before reading the body if downstream middleware also needs it. | Read the body without buffering and break downstream model binding. |
| **Testing** | Unit-test middleware by passing a mock `RequestDelegate` and `HttpContext`. | Test only through full integration tests when unit tests are possible. |

### Annotated Code Example (Best-Practice Middleware)

```csharp
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers["X-Correlation-ID"]
            .FirstOrDefault() ?? Guid.NewGuid().ToString();

        context.Response.Headers.Append("X-Correlation-ID", correlationId);
        using (context.TraceIdentifier = correlationId)
        {
            await _next(context);
        }
    }
}

// Extension method
public static class CorrelationIdMiddlewareExtensions
{
    public static IApplicationBuilder UseCorrelationId(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CorrelationIdMiddleware>();
    }
}

// Program.cs (correct placement)
app.UseExceptionHandler("/Error");
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.UseCorrelationId();   // After auth, before endpoints
app.MapControllers();
```

**Why this follows best practices:**

- **`InvokeAsync`** is used with `await`—no blocking calls.
- **No Scoped services in the constructor**—only `RequestDelegate`.
- **Single responsibility**—the middleware only handles correlation IDs.
- **Extension method** provides a clean registration API.
- **Correct placement**—after auth and before endpoints.

### Real-World Cases

- **High-traffic APIs:** Async middleware avoids thread starvation.
- **Scoped-service middleware:** `IMiddleware` is used when a `DbContext` or other Scoped service is required.
- **Testable middleware:** Constructor injection of `RequestDelegate` makes unit testing straightforward.

### References

- Write custom ASP.NET Core middleware (best practices) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- ASP.NET Core middleware best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Factory-based middleware activation (Scoped services) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility

---

## Summary Table: Custom Middleware at a Glance

| Concept | Primary Role | Key Syntax / Tool | When to Use |
|---------|--------------|-------------------|-------------|
| **Custom middleware** | User-defined pipeline component | Class with `Invoke`/`InvokeAsync` or inline `Use` | When built-in middleware is insufficient |
| **`IMiddleware`** | Factory-activated, per-request middleware | `IMiddleware.InvokeAsync(HttpContext, RequestDelegate)` | When Scoped services are needed in the constructor |
| **`RequestDelegate`** | Delegate representing the next middleware | `delegate Task RequestDelegate(HttpContext)` | Every middleware (constructor or `InvokeAsync` parameter) |
| **`Invoke()`** | Synchronous convention-based method | `public Task Invoke(HttpContext)` | Simple synchronous middleware (rare) |
| **`InvokeAsync()`** | Asynchronous convention-based method | `public async Task InvokeAsync(HttpContext)` | Recommended for all middleware with async work |
| **Extension methods** | Clean registration API | `public static IApplicationBuilder UseMyMiddleware(this IApplicationBuilder)` | Every custom middleware |
| **Best practices** | Correctness, performance, maintainability | `IMiddleware` for Scoped, async/await, single responsibility | Every production middleware |

---

## References (Full List)

- Write custom ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Factory-based middleware activation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility
- `IMiddleware` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware
- `IMiddleware.InvokeAsync` method – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware.invokeasync
- `RequestDelegate` delegate – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.requestdelegate
- `UseMiddlewareExtensions.UseMiddleware` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.usemiddlewareextensions.usemiddleware
- Dependency injection guidelines (captive dependencies) – https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines