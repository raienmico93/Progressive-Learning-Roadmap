# ASP.NET Core Middleware Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the minimal hosting model (`WebApplication`). Middleware is the foundation of request processing: each component can inspect, modify, or short-circuit requests as they flow through the pipeline. The fundamental concepts are stable; version-specific additions (e.g., `MapShortCircuit` in .NET 7+, rate limiting in .NET 7+) are noted where relevant. This cheat sheet does **not** merge sections—each requested topic is treated independently.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware is software assembled into an application pipeline to handle HTTP requests and responses. Each middleware component decides whether to pass the request to the next component and can perform work before and after that call. |
| **Technical Definition** | Middleware components are delegates (`Func<HttpContext, RequestDelegate, Task>`) or classes (convention-based or implementing `IMiddleware`) registered in `Program.cs` via `app.Use`, `app.Run`, `app.Map`, or `app.UseMiddleware<T>()`. They form a **chain of responsibility**: the request flows forward through the chain, and the response flows backward in reverse order. Middleware can **short-circuit** the pipeline by not calling `next`, making it a **terminal middleware**. Order is critical for security, performance, and functionality. |
| **Beginner-Friendly Explanation** | Middleware is a **series of security checkpoints and workstations** that every HTTP request passes through before it reaches your application logic, and every response passes through on the way back out. Think of an airport: check-in → security → passport control → gate. Each step can inspect you, do something, and decide whether to let you continue. If a step stops you (short-circuits), you never reach the later steps. |

### Key Characteristics

- **Chain of responsibility** – Each middleware can choose to call the next middleware or terminate the request.
- **Bidirectional** – Requests flow forward; responses flow in reverse order.
- **Order-sensitive** – The order of registration defines behaviour and security boundaries.
- **Composable** – Custom middleware can be inserted anywhere in the pipeline.
- **Short-circuit capable** – Middleware that does not call `next` terminates the pipeline (terminal middleware).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Basic understanding of **HTTP**, **delegates**, and **async/await** in C#.
- Familiarity with `Program.cs` (minimal hosting) and dependency injection.

### Related Programming Areas

- **Dependency injection** – Middleware can receive services via constructor injection or `InvokeAsync` parameters.
- **Routing** – `UseRouting` selects the endpoint; `UseEndpoints` executes it.
- **Authentication and authorization** – `UseAuthentication` and `UseAuthorization` must run in the correct order.
- **Exception handling** – `UseExceptionHandler` wraps the pipeline to catch downstream exceptions.
- **Static files** – `UseStaticFiles` serves files and short-circuits the pipeline for file requests.

### Core Concepts / Features

1. **Middleware** – The component concept, creation patterns, and built-in middleware.
2. **Middleware pipeline** – The chain, request/response flow, branching, and short-circuiting.
3. **Middleware order** – The canonical ordering and the consequences of getting it wrong.
4. **Middleware best practices** – Security, performance, and maintainability guidelines.

---

## 1. Middleware

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware is a component in the ASP.NET Core request pipeline that can inspect, modify, or act on an HTTP request and response. |
| **Technical Definition** | A middleware component is a delegate that receives an `HttpContext` and a `RequestDelegate` (`next`). It can perform logic before and after calling `next`. Middleware can be written **inline** (`app.Use(async (context, next) => ...)`), as a **convention-based class** (constructor takes `RequestDelegate`, method is `InvokeAsync`), or as a **factory-activated class** implementing `IMiddleware`. The `UseMiddleware<T>()` extension adds a class-based middleware to the pipeline. Built-in middleware includes authentication, authorization, CORS, static files, exception handling, HSTS, HTTPS redirection, routing, session, and more. |
| **Beginner-Friendly Explanation** | Middleware is a **processing step** that every request goes through. Some middleware are built in (like the security guard that checks if you are logged in), and you can write your own (like a step that logs how long each request takes). Each step can do its job and then either pass the request along or stop it. |

### Purposes

- **To** handle cross-cutting concerns (logging, authentication, CORS, error handling) in a composable, reusable way.
- **To** transform requests and responses before they reach application endpoints or the client.
- **To** short-circuit requests that should not proceed (e.g., unauthorized requests, static file hits).
- **To** add headers, correlation IDs, or timing information to every response.
- **To** wrap downstream processing with try/catch, retry, or caching logic.

### Syntax Rules and Structure

#### Inline Middleware (`app.Use`)

```csharp
app.Use(async (context, next) =>
{
    // Work before the next middleware
    await next(context);   // Call the next middleware
    // Work after the next middleware
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context` | The `HttpContext` for the current request. |
| `next` | The `RequestDelegate` representing the rest of the pipeline. |
| `await next(context)` | Passes control downstream. Omit to short-circuit. |

#### Convention-Based Class Middleware

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Request: {Method} {Path}",
            context.Request.Method, context.Request.Path);
        await _next(context);
    }
}

// Registration
app.UseMiddleware<RequestLoggingMiddleware>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| Constructor | Receives `RequestDelegate next` and optionally other services. |
| `InvokeAsync` | Must be public; receives `HttpContext` (and optionally scoped services). |
| `UseMiddleware<T>()` | Adds the class to the pipeline. |

#### Factory-Activated Middleware (`IMiddleware`)

```csharp
public class FactoryActivatedMiddleware : IMiddleware
{
    private readonly SampleDbContext _db;

    public FactoryActivatedMiddleware(SampleDbContext db) => _db = db;

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        // Work with scoped services
        await next(context);
    }
}

// Registration
builder.Services.AddTransient<FactoryActivatedMiddleware>();
app.UseMiddleware<FactoryActivatedMiddleware>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IMiddleware` | Interface with `InvokeAsync(HttpContext, RequestDelegate)`. |
| Scoped services | Can be injected into the constructor because `IMiddleware` is activated per request. |
| Registration | Must be registered as a service (Transient or Scoped). |

**Syntax rules:**

- **Inline middleware** is suitable for simple logic; **class middleware** is better for complex or reusable logic.
- **Convention-based middleware** is instantiated once (singleton-like); do **not** inject Scoped services into its constructor. Use `InvokeAsync` parameters for Scoped services.
- **`IMiddleware`** is activated per request, so Scoped services **can** be constructor-injected.

**Constraints and limitations:**

- **Convention-based middleware constructor** runs once at startup; Scoped services must be injected into `InvokeAsync` or resolved from `context.RequestServices`.
- **`IMiddleware`** must be registered in the DI container, unlike convention-based middleware.
- **`UseMiddleware<T>()` cannot pass arbitrary arguments** to factory-activated middleware; use constructor injection instead.

### Annotated Code Example (Convention-Based Logging Middleware)

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
        _logger.LogInformation("Request {Path} took {Elapsed}ms",
            context.Request.Path, sw.ElapsedMilliseconds);
    }
}

// Program.cs
var app = builder.Build();
app.UseMiddleware<RequestTimingMiddleware>();
```

**Expected Output:** Each request logs its path and duration after the downstream pipeline completes.

**Why this output?** The middleware runs before `_next` (starting the stopwatch) and after `_next` (stopping and logging), wrapping the rest of the pipeline.

### Real-World Cases

- **Request logging:** Capture method, path, status code, and duration.
- **Exception handling:** Catch downstream exceptions and return structured error responses.
- **Security headers:** Add `X-Content-Type-Options`, `X-Frame-Options`, etc. to every response.
- **Correlation IDs:** Attach a unique ID to each request for distributed tracing.

### References

- ASP.NET Core Middleware (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Factory-based middleware activation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility
- Write custom ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write

---

## 2. Middleware Pipeline

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The middleware pipeline is the ordered chain of middleware components that an HTTP request traverses, with the response traversing the chain in reverse. |
| **Technical Definition** | The pipeline is constructed by the order in which middleware are added in `Program.cs`. Each middleware receives a `RequestDelegate next` pointing to the subsequent middleware. The request flows forward through the chain; the response flows backward. Middleware can **short-circuit** by not calling `next`, in which case the pipeline is terminated early (terminal middleware). Branching (`Map`, `MapWhen`, `UseWhen`) creates sub-pipelines that either rejoin the main pipeline (`UseWhen`) or terminate separately (`MapWhen`, `Map`). |
| **Beginner-Friendly Explanation** | The pipeline is like an **assembly line**: the request enters at one end and passes through each station (middleware) in order. Each station can do something, then pass the request to the next station. When the request reaches the end (the endpoint), the response travels back through the stations in reverse order. A station can also decide to stop the request and send a response immediately (short-circuit). |

### Purposes

- **To** compose request-processing logic in a modular, reusable way.
- **To** control the exact order in which cross-cutting concerns execute.
- **To** allow early termination (short-circuiting) for efficiency or security.
- **To** support conditional branching for different request types or paths.
- **To** wrap downstream processing with before-and-after logic (timing, logging, error handling).

### Syntax Rules and Structure

#### Pipeline Construction (`Use`, `Run`, `Map`)

```csharp
// Use: can call next
app.Use(async (context, next) =>
{
    await next(context);
});

// Run: terminal — does not call next
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from terminal middleware.");
});

// Map: branches based on path
app.Map("/branch", branch =>
{
    branch.Run(async context =>
    {
        await context.Response.WriteAsync("Hello from /branch.");
    });
});
```

**Component breakdown:**

| Method | Description |
|--------|-------------|
| `Use` | Adds middleware that can call `next`. |
| `Run` | Adds a **terminal** middleware that does not call `next`. |
| `Map` | Creates a branch based on the request path. |

#### Branching with `MapWhen` and `UseWhen`

```csharp
// MapWhen: branches and does not rejoin
app.MapWhen(context => context.Request.Query.ContainsKey("branch"),
    branch =>
    {
        branch.Run(async context =>
        {
            await context.Response.WriteAsync("Branch reached.");
        });
    });

// UseWhen: branches and rejoins if not terminal
app.UseWhen(context => context.Request.Query.ContainsKey("branch"),
    branch =>
    {
        branch.Use(async (context, next) =>
        {
            // Branch logic
            await next(context);
        });
    });
```

**Component breakdown:**

| Method | Description |
|--------|-------------|
| `MapWhen` | Branches based on a predicate; the branch does **not** rejoin the main pipeline. |
| `UseWhen` | Branches based on a predicate; the branch **rejoins** the main pipeline if it does not short-circuit. |

#### Short-Circuiting

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/blocked"))
    {
        context.Response.StatusCode = 403;
        await context.Response.WriteAsync("Forbidden");
        return;   // Do NOT call next — short-circuit
    }
    await next(context);
});
```

**Component breakdown:**

| Pattern | Description |
|---------|-------------|
| No `await next(context)` | The pipeline terminates; downstream middleware never run. |
| Terminal middleware | Any middleware that does not call `next` is terminal. |

**Syntax rules:**

- **`Run`** is always terminal; it does not receive a `next` delegate.
- **`Map`** branches on path prefix; **`MapWhen`** branches on a predicate and does not rejoin; **`UseWhen`** branches and rejoins.
- Middleware order in code **is** the order of request processing; response processing is in reverse.

**Constraints and limitations:**

- **Short-circuiting** means downstream middleware (including endpoint execution) never run. Use it deliberately.
- **`MapWhen` branches do not rejoin**; if the branch does not short-circuit, the request still does not continue in the main pipeline after the branch. Use `UseWhen` when rejoining is desired.
- **Response body reading** – Middleware that reads the request body must enable buffering (`context.Request.EnableBuffering()`) because the body stream is forward-only by default.

### Annotated Code Example (Branching and Rejoining)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Main pipeline
app.Use(async (context, next) =>
{
    await context.Response.WriteAsync("Main: Before\n");
    await next(context);
    await context.Response.WriteAsync("Main: After\n");
});

// Branch that rejoins (UseWhen)
app.UseWhen(
    context => context.Request.Query.ContainsKey("branch"),
    branch =>
    {
        branch.Use(async (context, next) =>
        {
            await context.Response.WriteAsync("Branch: Before\n");
            await next(context);
            await context.Response.WriteAsync("Branch: After\n");
        });
    });

// Terminal middleware
app.Run(async context =>
{
    await context.Response.WriteAsync("Terminal\n");
});

app.Run();
```

**Expected Output (request without `?branch`):**

```
Main: Before
Terminal
Main: After
```

**Expected Output (request with `?branch`):**

```
Main: Before
Branch: Before
Terminal
Branch: After
Main: After
```

**Why this output?** The main middleware runs first. When the `branch` query key is present, `UseWhen` inserts the branch middleware before the terminal middleware. The response flows back through the branch and then the main middleware in reverse order.

### Real-World Cases

- **Static file serving:** `UseStaticFiles` short-circuits the pipeline for file requests, avoiding unnecessary routing and controller work.
- **Health checks:** A branch handles `/health` and short-circuits, bypassing authentication and business logic.
- **Multi-tenant routing:** `MapWhen` branches based on host or headers to different sub-pipelines.
- **API versioning:** `Map` branches on path prefixes to version-specific pipelines.

### References

- ASP.NET Core Middleware (pipeline and branching) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- `UseWhen`, `MapWhen` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#branch-the-middleware-pipeline
- Short-circuiting and terminal middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#short-circuiting-the-request-pipeline

---

## 3. Middleware Order

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware order is the sequence in which middleware components are added to the pipeline; this order determines request processing order (and reverse order for responses). |
| **Technical Definition** | Middleware are invoked in the order they are added in `Program.cs` for requests, and in **reverse order** for responses. The order is critical for security, performance, and functionality. Certain middleware have mandatory relative ordering: `UseAuthentication` must come before `UseAuthorization`; `UseCors` must come before `UseResponseCaching`; `UseRouting` must come before endpoint-aware middleware; endpoint mapping (`MapControllers`, `MapRazorPages`) must occur after routing and authorization. |
| **Beginner-Friendly Explanation** | Middleware order is **who checks you first**. If authorization runs before authentication, the system does not yet know who you are—so authorization cannot work. If CORS runs after authentication, browser preflight requests may be rejected before CORS headers are added. The order is not arbitrary; it follows the logical flow of inspecting, identifying, and then acting on a request. |

### Purposes

- **To** ensure security features (authentication, authorization, CORS) execute in the correct logical sequence.
- **To** guarantee that exception handling wraps the widest possible scope of the pipeline.
- **To** allow performance optimisations (static files, response caching) to short-circuit early.
- **To** ensure endpoint routing metadata is available to middleware that depend on it.
- **To** prevent subtle bugs such as auth bypass, CORS failures, or unhandled exceptions.

### Syntax Rules and Structure (Recommended Order)

```csharp
var app = builder.Build();

// 1. Exception handling (first to catch all)
app.UseExceptionHandler("/error");
app.UseHsts();

// 2. HTTPS redirection
app.UseHttpsRedirection();

// 3. Static files (before routing)
app.UseStaticFiles();

// 4. Routing
app.UseRouting();

// 5. CORS (after routing, before auth)
app.UseCors();

// 6. Authentication
app.UseAuthentication();

// 7. Authorization
app.UseAuthorization();

// 8. Custom middleware
app.UseMiddleware<RequestLoggingMiddleware>();

// 9. Endpoints
app.MapControllers();
app.MapRazorPages();
```

**Component breakdown:**

| Position | Middleware | Reason |
|----------|------------|--------|
| 1 | Exception handler | Must catch exceptions from all downstream middleware. |
| 2 | HSTS / HTTPS redirection | Security; must run before anything that consumes the URL. |
| 3 | Static files | Short-circuits file requests before routing overhead. |
| 4 | Routing | Selects the endpoint; must precede endpoint-aware middleware. |
| 5 | CORS | Must run before authentication and response caching. |
| 6 | Authentication | Identifies the user; must precede authorization. |
| 7 | Authorization | Checks permissions; must follow authentication. |
| 8 | Custom middleware | Application-specific logic; placement depends on intent. |
| 9 | Endpoints | Terminal execution of controllers/pages. |

**Syntax rules:**

- **`UseAuthentication()` must come before `UseAuthorization()`**.
- **`UseCors()` must come before `UseResponseCaching()`** (currently required due to a known issue).
- **`UseRouting()` must come before middleware that inspects endpoint metadata**, and endpoint mapping must come after authorization.
- **Exception handling must be first** so it can catch exceptions from the entire downstream pipeline.

**Constraints and limitations:**

- **Order is security-sensitive** – Reversing authentication and authorization allows unauthenticated requests to bypass authorization.
- **Not every middleware appears in the canonical order** – caching and compression order is scenario-specific.
- **Custom middleware placement** depends on whether it needs to run before routing (e.g., header inspection) or after routing (e.g., endpoint metadata inspection).

### Annotated Code Example (Common Ordering Mistake)

```csharp
// ❌ WRONG: Authorization before Authentication
app.UseAuthorization();
app.UseAuthentication();

// ✅ CORRECT: Authentication before Authorization
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output (wrong order):** Authorization runs before the user is identified; `HttpContext.User` is unauthenticated, so protected endpoints may incorrectly allow or deny access.

**Why this output?** Authorization depends on the identity established by authentication. Reversing the order breaks the security model.

### Real-World Cases

- **CORS failures:** Placing `UseCors` after `UseAuthorization` causes preflight requests to be rejected.
- **Auth bypass:** Placing `UseAuthorization` before `UseAuthentication` means authorization cannot see the authenticated user.
- **Unhandled exceptions:** Placing `UseExceptionHandler` after middleware that throws leaves those exceptions unhandled.

### References

- ASP.NET Core middleware order (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Middleware ordering changes behaviour – https://github.com/AsiBackbone/Learning/blob/e95c01c0e53f95477d4b7df3200601dbe814bce8/docs/aspnetcore/middleware-ordering-changes-behavior.md

---

## 4. Middleware Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware best practices are guidelines for designing, ordering, and implementing middleware that is secure, performant, and maintainable. |
| **Technical Definition** | Best practices encompass correct ordering (exception handling first, auth before authorization, CORS before caching), avoiding blocking calls (`async`/`await` throughout), not injecting Scoped services into convention-based middleware constructors, handling request body buffering correctly, avoiding service locator patterns, and keeping middleware focused on a single responsibility. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for middleware—they help you avoid security holes, thread starvation, mysterious CORS failures, and middleware that silently never runs because it was placed after a terminal component. |

### Purposes

- **To** prevent security vulnerabilities (auth bypass, CORS failures, missing security headers).
- **To** avoid performance problems (thread starvation from blocking calls, unnecessary pipeline work).
- **To** ensure middleware executes when and where intended (correct placement relative to terminal middleware).
- **To** keep middleware testable, maintainable, and free of hidden dependencies.
- **To** handle request/response streams correctly (buffering, rewind, content length).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Ordering** | Follow the canonical order: exception handler → HSTS/HTTPS → static files → routing → CORS → authentication → authorization → endpoints. | Place authorization before authentication; place CORS after response caching. |
| **Exception handling** | Place `UseExceptionHandler` first so it catches downstream exceptions. | Place it after middleware that may throw. |
| **Async** | Use `async Task InvokeAsync` and `await` throughout. | Use blocking calls (`.Result`, `.Wait()`)—causes thread starvation. |
| **Scoped services** | In convention-based middleware, inject Scoped services into `InvokeAsync` parameters, not the constructor. | Inject `DbContext` into a convention-based middleware constructor (captive dependency). |
| **`IMiddleware`** | Use `IMiddleware` when Scoped services are needed in the constructor; register it as Transient/Scoped. | Assume `IMiddleware` works without DI registration. |
| **Request body** | Call `context.Request.EnableBuffering()` before reading the body if downstream middleware also needs it. | Read the body without buffering and break downstream model binding. |
| **Short-circuiting** | Short-circuit deliberately and document it (e.g., health checks, static files). | Place custom middleware after a terminal middleware and expect it to run. |
| **Service locator** | Inject dependencies explicitly via constructor or `InvokeAsync` parameters. | Call `context.RequestServices.GetService()` inside business logic. |

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

// Program.cs — correct placement
var app = builder.Build();

app.UseExceptionHandler("/error");          // 1. Catch all
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();                       // 3. Short-circuit files
app.UseRouting();                           // 4. Route selection
app.UseCors();                              // 5. CORS before auth
app.UseAuthentication();                    // 6. Identify user
app.UseAuthorization();                     // 7. Check permissions
app.UseMiddleware<CorrelationIdMiddleware>(); // 8. Custom logic
app.MapControllers();                       // 9. Endpoints
```

**Why this follows best practices:**

- **Exception handling is first** so it catches exceptions from every downstream middleware.
- **Static files come before routing** to short-circuit file requests early.
- **CORS is before authentication** and **authentication is before authorization**.
- **The custom middleware is async** and uses `InvokeAsync` without blocking.
- **No Scoped services are captured** in the constructor—only `RequestDelegate` is injected.

### Real-World Cases

- **API gateways:** Middleware order determines whether authentication, rate limiting, and CORS work correctly.
- **High-traffic services:** Async middleware and early short-circuiting (static files, health checks) reduce CPU and thread usage.
- **Multi-tenant SaaS:** Custom middleware for tenant resolution must run before authentication in some designs, or after routing in others—placement depends on the dependency graph.

### References

- ASP.NET Core middleware best practices – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Middleware anti-patterns (Quick Reference) – https://github.com/claude-dev-suite/claude-dev-suite/blob/e10ab644ba43abe2abc923399226df4af7337214/skills/backend-frameworks/aspnet-middleware/SKILL.md
- How to Configure Middleware in ASP.NET Core – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-configure-middleware-aspnet-core/README.md

---

## Summary Table: Middleware Fundamentals at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|------------------|-------------|
| **Middleware** | Processing component in the pipeline | `app.Use`, `app.Run`, `UseMiddleware<T>`, `IMiddleware` | Cross-cutting concerns, request/response transformation |
| **Middleware pipeline** | Ordered chain of components | `Use`, `Run`, `Map`, `MapWhen`, `UseWhen` | Every ASP.NET Core app |
| **Middleware order** | Correct sequence for security and functionality | Exception → HSTS → HTTPS → Static → Routing → CORS → AuthN → AuthZ → Endpoints | Every production pipeline |
| **Best practices** | Security, performance, maintainability | Async, correct ordering, no Scoped in constructors, buffer body | Every middleware implementation |

---

## References (Full List)

- ASP.NET Core Middleware (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- ASP.NET Core middleware order (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Factory-based middleware activation in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility
- Write custom ASP.NET Core middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write
- `IMiddleware` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.imiddleware
- `UseWhen`, `MapWhen` branching – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#branch-the-middleware-pipeline
- Short-circuiting and terminal middleware – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#short-circuiting-the-request-pipeline
- ASP.NET Core Middleware – Quick Reference (GitHub) – https://github.com/claude-dev-suite/claude-dev-suite/blob/e10ab644ba43abe2abc923399226df4af7337214/skills/backend-frameworks/aspnet-middleware/SKILL.md
- How to Configure Middleware in ASP.NET Core (OneUptime) – https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2025-12-23-configure-middleware-aspnet-core/README.md
- Middleware ordering changes behaviour (GitHub) – https://github.com/AsiBackbone/Learning/blob/e95c01c0e53f95477d4b7df3200601dbe814bce8/docs/aspnetcore/middleware-ordering-changes-behavior.md
- WebApplication and WebApplicationBuilder (automatic middleware insertion) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/webapplication