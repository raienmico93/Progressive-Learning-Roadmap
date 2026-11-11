# C# ASP.NET Core Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Middleware** in ASP.NET Core is the **software components assembled into a pipeline** that handles every HTTP request and response. Each middleware component can perform work before and after the next component, and can choose whether to pass the request to the next component or short-circuit the pipeline. Middleware is the fundamental building block of the ASP.NET Core request-processing model.

**Technical Definition**
ASP.NET Core applications consist of a sequence of **request delegates** that are executed one after another. The **`RequestDelegate`** type is `public delegate Task RequestDelegate(HttpContext context)` — it represents a function that takes an `HttpContext` and returns a `Task`. Middleware is configured using `IApplicationBuilder` extension methods: **`Use`** (adds a middleware that can call `next`), **`Run`** (adds a terminal middleware that does not call `next`), **`Map`** (branches based on path), **`MapWhen`** (branches based on a predicate), **`UseWhen`** (conditionally adds middleware), and **`UseMiddleware<T>`** (adds a middleware class). Custom middleware can be implemented as **convention-based middleware** (a class with an `InvokeAsync(HttpContext)` method and a `RequestDelegate` constructor parameter) or as **factory-based middleware** implementing **`IMiddleware`** (registered as a service, resolved per-request). Middleware executes **in the order registered** for inbound requests, and **in reverse order** for outbound responses. Ordering is critical: exception handling must be early, HTTPS redirection before static files, routing before authentication, and authorization before endpoints.

**Beginner-Friendly Explanation**
Middleware is a **conveyor belt for HTTP requests**. Each middleware is a station on the belt: the request arrives, passes through the first station (which might log it), then the next (which might authenticate the user), then the next (which might serve a static file or route it to a controller). Once the request reaches the last station (the endpoint), the response travels **backward** through the same stations. A station can **short-circuit** —for example, if authentication fails, it can send a 401 response without passing the request on. Middleware is how ASP.NET Core lets you **compose your request pipeline from small, reusable pieces**.

---

### Key Characteristics

- **Pipeline composition** — Middleware is assembled in a chain; each component wraps the next.
- **`RequestDelegate`** — The delegate type representing an HTTP handler (`Task (HttpContext)`).
- **Inbound/outbound execution** — Middleware runs in registration order for requests, and reverse order for responses.
- **Short-circuiting** — A middleware can stop the pipeline by not calling `next`.
- **Branching** — `Map`, `MapWhen`, and `UseWhen` create conditional sub-pipelines.
- **Convention-based vs. factory-based** — Classes with `InvokeAsync` vs. classes implementing `IMiddleware`.
- **DI-aware** — Constructor and `InvokeAsync` parameters can be resolved from DI (subject to lifetime rules).
- **Ordering is critical** — Incorrect order breaks authentication, authorization, error handling, or static file serving.

---

### Prerequisites

- Basic understanding of **ASP.NET Core fundamentals** (`Program.cs`, DI, configuration).
- Familiarity with **delegates** and **lambda expressions**.
- Knowledge of **`HttpContext`** (request, response, user, items).
- Awareness of **dependency injection** lifetimes (`Singleton`, `Scoped`, `Transient`).
- Understanding of the **HTTP request/response lifecycle**.

---

### Related Programming Areas

- **ASP.NET Core fundamentals** — Middleware is the core of the request pipeline.
- **Dependency injection** — Middleware services and `IMiddleware` lifetimes.
- **Routing and endpoints** — Endpoint routing is itself middleware.
- **Filters** — MVC filters run *inside* the endpoint middleware.
- **Diagnostics and logging** — Middleware is where cross-cutting concerns live.

---

### Core Concepts / Features

1. Middleware (Overarching Concept)
2. Middleware Pipeline
3. `RequestDelegate`
4. Custom Middleware (Convention-Based)
5. `IMiddleware` (Factory-Based Middleware)
6. Middleware Ordering
7. Middleware Best Practices

---

## 1. Middleware (Overarching Concept)

### Definitions

**Core Definition**
Middleware is a component that processes HTTP requests and responses in ASP.NET Core, and can pass the request to the next component in the pipeline.

**Technical Definition**
Middleware is software that's assembled into a pipeline to handle requests and responses. Each component:
- Chooses whether to pass the request to the next component in the pipeline.
- Can perform work before and after the next component in the pipeline.

The pipeline is built using **request delegates**. Middleware is configured via `IApplicationBuilder` extension methods and runs on every request.

**Beginner-Friendly Explanation**
Middleware is **one step in the request-processing conveyor belt**. Each step does something to the request (or response) and can decide whether to pass it along.

---

### Purposes

- To **compose cross-cutting concerns** (logging, authentication, compression) into reusable components.
- To **control the request pipeline** from a single configuration point (`Program.cs`).
- To **short-circuit** requests when needed (e.g., unauthorized, redirected).
- To **branch** the pipeline based on path or conditions.
- To **serve different request types** (static files, APIs, pages) from the same app.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Inline middleware
app.Use(async (context, next) =>
{
    // work before
    await next();
    // work after
});

// Terminal middleware
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});

app.Run();
```

**Component breakdown**

- `app.Use(...)` — Middleware that receives `next`.
- `app.Run(...)` — Terminal middleware; does not receive `next`.
- `await next()` — Passes the request to the next middleware.

#### Syntax Rules

- Middleware is added in the order it should execute for requests.
- `Use` middleware must call `next()` (or short-circuit intentionally).
- `Run` middleware never calls `next` (it is terminal).
- The app is started with `app.Run()` (no conflict with `Run` middleware — the parameterless `Run()` starts the server).

#### Constraints and Limitations

- Middleware runs for **every** request; performance-sensitive code should be efficient.
- Incorrect ordering can break security and functionality.
- Middleware cannot be added after the server has started.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Inline Middleware

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("Before next");
    await next();
    Console.WriteLine("After next");
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});

app.Run();
```

**Expected Output (console, per request)**
```
Before next
After next
```

**Expected Output (HTTP response body)**
```
Hello
```

**Why this output occurs**
The `Use` middleware logs **before** calling `next`, the terminal `Run` middleware writes the response, and the `Use` middleware logs **after** `next` returns. This demonstrates the inbound/outbound nature of the pipeline.

---

#### Example 2: Short-Circuiting Middleware

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Headers["X-Api-Key"] != "secret")
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("Unauthorized");
        return;   // short-circuit
    }
    await next();
});

app.Run(async context => await context.Response.WriteAsync("OK"));
```

**Expected Output**

- Request with valid `X-Api-Key` → `200 OK` with body `OK`.
- Request without key → `401 Unauthorized` with body `Unauthorized`.

**Why this output occurs**
The middleware **does not call `next()`** when the key is missing, so the request never reaches the terminal middleware.

---

### Real-World Cases

- **Authentication/authorization** — `UseAuthentication`, `UseAuthorization`.
- **Exception handling** — `UseExceptionHandler`.
- **HTTPS redirection** — `UseHttpsRedirection`.
- **Static files** — `UseStaticFiles`.
- **Compression** — `UseResponseCompression`.
- **CORS** — `UseCors`.

---

### References

- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md

---

## 2. Middleware Pipeline

### Definitions

**Core Definition**
The middleware pipeline is the **ordered chain of middleware components** that process every HTTP request and response.

**Technical Definition**
The pipeline is constructed at startup by calling `IApplicationBuilder` extension methods. Each call adds a delegate to the pipeline. When a request arrives, the **first** middleware runs; if it calls `next()`, the **second** middleware runs, and so on. When the terminal middleware completes, control returns **backward** through the middleware chain, allowing each middleware to perform post-processing. The pipeline is represented internally as a single composed `RequestDelegate`.

**Beginner-Friendly Explanation**
The pipeline is **the full conveyor belt**. Requests go forward through every station; responses come back through the same stations in reverse.

---

### Purposes

- To **process every request** through a consistent set of steps.
- To **enable post-processing** (e.g., add headers, measure timing) after the response is generated.
- To **support short-circuiting** and branching.
- To **centralize cross-cutting concerns**.

---

### Syntax Rules and Structure

```csharp
app.Use(async (context, next) => { /* before */ await next(); /* after */ });
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

**Component breakdown**

- **Inbound phase** — Middleware executes in **registration order**.
- **Outbound phase** — Middleware resumes in **reverse order** after `next()` returns.

#### Syntax Rules

- The pipeline is built at startup; middleware order is fixed once the app starts.
- Each `Use` middleware wraps the rest of the pipeline.
- The terminal `Run` or endpoint middleware produces the response.

#### Constraints and Limitations

- A middleware that does not call `next()` short-circuits all downstream middleware.
- Response headers must be set before the response body starts; otherwise, an exception is thrown.

---

### Annotated Code Examples and Expected Outputs

#### Example: Bidirectional Pipeline

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("A: before");
    await next();
    Console.WriteLine("A: after");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("B: before");
    await next();
    Console.WriteLine("B: after");
});

app.Run(async context =>
{
    Console.WriteLine("Endpoint");
    await context.Response.WriteAsync("Done");
});
```

**Expected Output (console, per request)**
```
A: before
B: before
Endpoint
B: after
A: after
```

**Why this output occurs**
Requests flow A → B → Endpoint; responses unwind Endpoint → B → A.

---

### Real-World Cases

- **Request timing** — Record start/end times around `next()`.
- **Header injection** — Add security headers after the response is generated.
- **Exception wrapping** — Catch exceptions thrown by downstream middleware.

---

### References

- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md

---

## 3. `RequestDelegate`

### Definitions

**Core Definition**
`RequestDelegate` is the **delegate type** representing an HTTP request handler: `Task (HttpContext context)`.

**Technical Definition**
`public delegate Task RequestDelegate(HttpContext context)` is the fundamental building block of the middleware pipeline. Each middleware receives the **next** `RequestDelegate` in the pipeline and can invoke it (or not) to pass the request along. Terminal middleware is itself a `RequestDelegate` that does not invoke a next delegate.

**Beginner-Friendly Explanation**
`RequestDelegate` is **the type of a middleware step**. It takes an `HttpContext` and returns a `Task`. Calling it means “run the rest of the pipeline.”

---

### Purposes

- To provide a **uniform signature** for middleware components.
- To enable **composition** (each middleware wraps the next `RequestDelegate`).
- To support **short-circuiting** by simply not invoking the delegate.
- To allow **async** processing.

---

### Syntax Rules and Structure

```csharp
RequestDelegate middleware = async context =>
{
    // do work
    await context.Response.WriteAsync("Hello");
};

// Register with Use
app.Use(next => async context =>
{
    // before
    await next(context);
    // after
});
```

**Component breakdown**

- `RequestDelegate` — `Task (HttpContext)`.
- `next` — The next delegate in the pipeline.
- `await next(context)` — Passes the request along.

#### Syntax Rules

- A `RequestDelegate` can be registered with `app.Run()` (terminal) or `app.Use()` (via the `Func<RequestDelegate, RequestDelegate>` overload).
- The delegate must return a `Task`.
- Exceptions propagate back through the pipeline.

#### Constraints and Limitations

- Direct `RequestDelegate` composition is powerful but less readable than `Use`/`Run` extension methods.
- Must handle exceptions or let them propagate to exception-handling middleware.

---

### Annotated Code Examples and Expected Outputs

#### Example: Explicit `RequestDelegate` Composition

```csharp
RequestDelegate log = async context =>
{
    Console.WriteLine($"Request: {context.Request.Path}");
    await context.Response.WriteAsync($"Path: {context.Request.Path}");
};

app.Run(log);
```

**Expected Output (HTTP response for `/hello`)**
```
Path: /hello
```

**Why this output occurs**
The `RequestDelegate` is terminal: it logs and writes the response directly.

---

### Real-World Cases

- **Low-level middleware** — Custom pipelines needing full control.
- **Testing** — Constructing minimal pipelines for tests.
- **Libraries** — Framework components that compose `RequestDelegate`s.

---

### References

- `RequestDelegate` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.requestdelegate
- ASP.NET Core middleware — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/

---

## 4. Custom Middleware (Convention-Based)

### Definitions

**Core Definition**
Convention-based middleware is a class with a **`RequestDelegate` constructor parameter** and a **public `InvokeAsync(HttpContext)` method** (or `Invoke`), registered via `app.UseMiddleware<T>()`.

**Technical Definition**
Convention-based middleware is instantiated **once** when the pipeline is built (constructor runs at startup), so constructor parameters must be services available at that time (singletons or app-level services). The `InvokeAsync` method is called **per request** and can declare additional parameters resolved from DI per-request (e.g., scoped services, `ILogger`). The class must be public, non-static, and have a method named `InvokeAsync` or `Invoke`.

**Beginner-Friendly Explanation**
Convention-based middleware is **a class that the framework calls once at startup and then per request**. You put singleton-ish dependencies in the constructor and per-request dependencies in `InvokeAsync`.

---

### Purposes

- To encapsulate **reusable middleware logic** in a class.
- To allow **constructor and per-request DI**.
- To keep `Program.cs` clean.
- To **test middleware** in isolation.

---

### Syntax Rules and Structure

```csharp
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(
        RequestDelegate next,
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
        _logger.LogInformation("{Path} took {Ms}ms",
            context.Request.Path, sw.ElapsedMilliseconds);
    }
}

// Registration
app.UseMiddleware<RequestTimingMiddleware>();
```

**Component breakdown**

- **Constructor** — Receives `RequestDelegate next` and singleton/startup services.
- **`InvokeAsync(HttpContext)`** — Runs per request; can add DI parameters.
- **`UseMiddleware<T>()`** — Registers the middleware.

#### Syntax Rules

- The class must be public and non-static.
- The constructor must accept `RequestDelegate` (or a compatible delegate).
- The method must be named `InvokeAsync` (preferred) or `Invoke`.
- Constructor DI services must be available at app startup (singletons).
- `InvokeAsync` can declare additional DI parameters (resolved per request).

#### Constraints and Limitations

- Middleware instances are **created once** (singleton behaviour); do not store per-request state in instance fields.
- Scoped services should be passed as `InvokeAsync` parameters, not constructor parameters.

---

### Annotated Code Examples and Expected Outputs

#### Example: Middleware with Scoped `InvokeAsync` Dependency

```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public TenantMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context, ITenantProvider tenant)
    {
        context.Items["tenant"] = tenant.GetCurrent();
        await _next(context);
    }
}

app.UseMiddleware<TenantMiddleware>();
```

**Expected Output (conceptual)**
```
// The scoped ITenantProvider is resolved per request.
```

**Why this pattern is used**
`ITenantProvider` is scoped; resolving it in `InvokeAsync` ensures one instance per request.

---

### Real-World Cases

- **Request timing** — Measure latency.
- **Tenant resolution** — Look up the tenant from the request.
- **Correlation IDs** — Attach a request ID for tracing.
- **Security headers** — Add headers to every response.

---

### References

- Write custom ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/write.md
- ASP.NET Core middleware — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/

---

## 5. `IMiddleware` (Factory-Based Middleware)

### Definitions

**Core Definition**
`IMiddleware` is an interface for **factory-based middleware** that is resolved from DI per request (or per configured lifetime).

**Technical Definition**
`IMiddleware` defines `Task InvokeAsync(HttpContext context, RequestDelegate next)`. Implementations are registered as services (`AddScoped<T>()` or `AddTransient<T>()`) and activated **per request** by `MiddlewareFactory`. This allows **scoped services** to be injected into the middleware constructor. Factory-based middleware is registered with `app.UseMiddleware<T>()` but behaves differently from convention-based middleware: the class is resolved from DI, not instantiated at startup.

**Beginner-Friendly Explanation**
`IMiddleware` is **middleware that is created per request**, so you can inject scoped services into its constructor.

---

### Purposes

- To allow **scoped dependencies** in middleware constructors.
- To support **per-request lifetime** for middleware instances.
- To enable **unit testing** by mocking `IMiddleware` implementations.
- To provide a **strongly typed** middleware contract.

---

### Syntax Rules and Structure

```csharp
public class MyMiddleware : IMiddleware
{
    private readonly IMyScopedService _service;

    public MyMiddleware(IMyScopedService service) => _service = service;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        await _service.DoAsync();
        await next(context);
    }
}

// Register the middleware in DI
builder.Services.AddScoped<MyMiddleware>();

// Add to pipeline
app.UseMiddleware<MyMiddleware>();
```

**Component breakdown**

- `IMiddleware` — The interface.
- `InvokeAsync(HttpContext, RequestDelegate)` — The handler.
- `AddScoped<MyMiddleware>()` — Required DI registration.
- `UseMiddleware<MyMiddleware>()` — Pipeline registration.

#### Syntax Rules

- Must be registered in DI.
- `InvokeAsync` receives both `HttpContext` and `RequestDelegate`.
- The class is resolved per request (based on its registered lifetime).

#### Constraints and Limitations

- **Must** be registered in `builder.Services` (unlike convention-based middleware, which does not require DI registration).
- Slightly more boilerplate than convention-based middleware.
- An instance is created per request (per `IMiddlewareFactory`), so expensive constructors matter.

---

### Annotated Code Examples and Expected Outputs

#### Example: Scoped Middleware with `IMiddleware`

```csharp
public interface IRequestContext { string RequestId { get; } }

public class RequestContext : IRequestContext
{
    public string RequestId { get; } = Guid.NewGuid().ToString("N");
}

public class CorrelationMiddleware : IMiddleware
{
    private readonly IRequestContext _ctx;
    public CorrelationMiddleware(IRequestContext ctx) => _ctx = ctx;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        context.Response.Headers["X-Request-Id"] = _ctx.RequestId;
        await next(context);
    }
}

builder.Services.AddScoped<IRequestContext, RequestContext>();
builder.Services.AddScoped<CorrelationMiddleware>();

app.UseMiddleware<CorrelationMiddleware>();
```

**Expected Output (response headers)**
```
X-Request-Id: 3f8a2c1d9e7b4a5c...
```

**Why this output occurs**
`IRequestContext` is scoped; `IMiddleware` is resolved per request, so a new ID is generated for each request and added to the response headers.

---

### Real-World Cases

- **Request-scoped services** — Tenant, user, transaction context.
- **Per-request logging** — Enriching logs with request context.
- **Unit testing** — Implementing `IMiddleware` and testing directly.

---

### References

- Write custom ASP.NET Core middleware — `IMiddleware` — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- Factory-based middleware activation — ASP.NET Core middleware — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility

---

## 6. Middleware Ordering

### Definitions

**Core Definition**
Middleware ordering is the **sequence in which middleware components are added to the pipeline**; it determines the order of execution for requests and the reverse order for responses.

**Technical Definition**
Middleware executes **in the order registered** for inbound requests and **in reverse order** for outbound responses. Ordering is critical because many middleware components depend on earlier ones (e.g., authorization requires authentication; routing requires the path to be known). Microsoft publishes a recommended ordering for built-in middleware.

**Beginner-Friendly Explanation**
Ordering is **which station comes first on the conveyor belt**. If authorization comes before authentication, it cannot see who the user is. If static files come before routing, they bypass authentication.

---

### Recommended Ordering

| # | Middleware | Purpose |
|---|---|---|
| 1 | `UseExceptionHandler` | Catch exceptions from all downstream middleware |
| 2 | `UseHsts` | Add HSTS header (production) |
| 3 | `UseHttpsRedirection` | Redirect HTTP → HTTPS |
| 4 | `UseStaticFiles` | Serve static files efficiently |
| 5 | `UseRouting` | Match routes |
| 6 | `UseCors` | Apply CORS policy |
| 7 | `UseAuthentication` | Identify the user |
| 8 | `UseAuthorization` | Enforce access control |
| 9 | Custom middleware | Application-specific concerns |
| 10 | `UseEndpoints` / `Map*` | Execute the endpoint |

**Component breakdown**

- **Exception handling first** — Catches exceptions from everything downstream.
- **Security headers early** — Applied before any response.
- **Static files before routing** — Served without routing overhead.
- **Authentication before authorization** — Authorization needs the user.
- **Endpoints last** — They produce the final response.

#### Syntax Rules

- Register middleware in the order they should execute.
- Use `UseRouting` before `UseAuthentication`/`UseAuthorization`.
- Use `UseEndpoints` or `Map*` at the end.

#### Constraints and Limitations

- Wrong order can cause `InvalidOperationException` (e.g., `UseAuthorization` without `UseRouting`).
- Static files served before authentication bypass authentication.

---

### Annotated Code Examples and Expected Outputs

#### Example: Recommended Pipeline

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors("default");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

**Expected Output (conceptual)**
```
// Exceptions caught; HTTP redirected to HTTPS; static files served;
// routing matched; CORS applied; user authenticated; access authorized;
// controllers executed.
```

**Why this order is used**
It follows Microsoft's documented middleware ordering guidance.

---

### Real-World Cases

- **Public web apps** — HTTPS, HSTS, static files, auth, MVC.
- **APIs** — Exception handler, CORS, authentication, authorization, controllers.
- **Multi-tenant apps** — Tenant resolution after authentication.

---

### References

- Middleware order — ASP.NET Core middleware — Microsoft Learn (GitHub) — https://github.com
- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md

---

## 7. Middleware Best Practices

### Definitions

**Core Definition**
Middleware best practices are the guidelines for building, ordering, and composing middleware that is **secure, performant, testable, and maintainable**.

**Technical Definition**
Best practices derive from Microsoft's middleware guidance: **order middleware correctly** (exception handling → HTTPS → static files → routing → auth → endpoints); **keep middleware focused** on one concern; **avoid storing per-request state in instance fields** (convention-based middleware is singleton); **prefer `InvokeAsync` over `Invoke`**; **use `IMiddleware` when scoped dependencies are needed in the constructor**; **prefer `UseWhen`/`MapWhen` over ad-hoc conditionals inside one big middleware**; **avoid modifying the response after it has started**; **test middleware in isolation**; and **keep the pipeline short** — only add middleware that is needed.

**Beginner-Friendly Explanation**
The best practices are: **order correctly**, **keep each middleware small**, **don't store per-request state in fields**, **use `IMiddleware` for scoped services**, **test your middleware**, and **don't add middleware you don't need**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Correct ordering
app.UseExceptionHandler("/error");
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

// 2. Prefer InvokeAsync over Invoke
public async Task InvokeAsync(HttpContext context, RequestDelegate next)
{
    await next(context);
}

// 3. Use IMiddleware for scoped dependencies
public class MyMiddleware : IMiddleware
{
    public MyMiddleware(IScopedService svc) { /* ... */ }
    public Task InvokeAsync(HttpContext ctx, RequestDelegate next) => next(ctx);
}
builder.Services.AddScoped<MyMiddleware>();

// 4. Branch with MapWhen/UseWhen instead of conditionals
app.MapWhen(ctx => ctx.Request.Query.ContainsKey("api"), branch =>
{
    branch.UseApiMiddleware();
});

// 5. Don't modify response after it starts
if (!context.Response.HasStarted)
{
    context.Response.Headers["X-Custom"] = "value";
}
```

**Component breakdown**

- **Ordering** — Security and functionality depend on it.
- **`InvokeAsync`** — Preferred method name.
- **`IMiddleware`** — For scoped dependencies.
- **Branching** — Keep middleware focused.
- **Response checks** — Guard against started responses.

#### Rules and Constraints

- **Do** order middleware correctly.
- **Do** keep middleware focused.
- **Do** prefer `InvokeAsync` over `Invoke`.
- **Do** use `IMiddleware` for scoped services.
- **Do** use `MapWhen`/`UseWhen` for branching.
- **Do not** store per-request state in instance fields.
- **Do not** modify the response after it has started.
- **Do not** add unnecessary middleware.

---

### Annotated Code Examples and Expected Outputs

#### Example: Well-Designed Middleware

```csharp
public sealed class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.OnStarting(() =>
        {
            context.Response.Headers["X-Content-Type-Options"] = "nosniff";
            context.Response.Headers["X-Frame-Options"] = "DENY";
            return Task.CompletedTask;
        });

        await _next(context);
    }
}

app.UseMiddleware<SecurityHeadersMiddleware>();
```

**Expected Output (response headers)**
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

**Why this pattern is used**
`OnStarting` ensures headers are set before the response body starts, following best practices.

---

### Real-World Cases

- **Security headers** — HSTS, X-Frame-Options, CSP.
- **Correlation IDs** — Attach per-request IDs.
- **Request logging** — Structured logs with timing.
- **API key validation** — Short-circuit unauthorized requests.

---

### References

- Write custom ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/write.md
- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md
- Factory-based middleware — ASP.NET Core middleware — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Invoke` method name | **Deprecated preference**; use `InvokeAsync` for async middleware. |
| Convention-based middleware | **Singleton**; do not store per-request state in fields. |
| `IMiddleware` | **Factory-based**; must be registered in DI; resolved per request. |
| `UseMiddleware<T>()` | **Recommended** registration method for typed middleware. |
| `Map` vs. `UseWhen` | `Map` **removes the matched path**; `UseWhen` **rejoins** the main pipeline. |
| `UseAuthorization` without `UseRouting` | **InvalidOperationException** at runtime. |
| Modifying response after start | **InvalidOperationException**; use `OnStarting`. |
| `UseStaticFiles` before `UseAuthentication` | **Bypasses authentication** for static files; intentional but must be understood. |

---

## References (Consolidated)

- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md
- Write custom ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/write.md
- Factory-based middleware activation — ASP.NET Core middleware — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility
- Middleware order — ASP.NET Core middleware — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- `RequestDelegate` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.requestdelegate
- `IMiddleware` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware

---