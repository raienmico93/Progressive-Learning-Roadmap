# ASP.NET Core Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
ASP.NET Core is a **cross-platform, high-performance, open-source framework** for building modern, cloud-enabled, internet-connected web applications using .NET. It provides a unified programming model for building web apps, APIs, real-time services (SignalR), and gRPC services, all running on the .NET runtime. ASP.NET Core is the successor to ASP.NET (Framework) and is designed from the ground up for modularity, testability, and performance.

**Technical Definition**
ASP.NET Core is a modular framework built on top of the .NET runtime (CoreCLR or .NET Framework). It implements a **lightweight and modular HTTP request pipeline** composed of middleware components. It includes an integrated **dependency injection (DI) container**, environment-based configuration, and rich logging, tracing, and runtime metrics. The framework ships with **Kestrel**, a high-performance, cross-platform HTTP server, and supports hosting behind IIS, IIS Express, or HTTP.sys. ASP.NET Core applications are typically started via `Program.cs` (top-level statements) using `WebApplication.CreateBuilder(args)` and `WebApplication.Build()`.

**Beginner-Friendly Explanation**
ASP.NET Core is the **modern way to build websites and web APIs with C#**. Think of it as a set of Lego bricks: you pick the pieces you need (routing, authentication, static files) and snap them together. Unlike the old ASP.NET, it runs on Windows, Linux, and macOS, and it is **fast** because it uses a lean HTTP server called Kestrel. Everything in ASP.NET Core is a **service** that you register in a built-in dependency injection container, and every request flows through a configurable **middleware pipeline** that you control step by step.

---

### Key Characteristics

- **Cross-platform** — Runs on Windows, Linux, and macOS.
- **High performance** — Kestrel is one of the fastest web servers available; ASP.NET Core consistently ranks at the top of TechEmpower benchmarks.
- **Modular and lightweight** — The request pipeline is composed of opt-in middleware; nothing is imposed by default.
- **Built-in dependency injection** — A first-class DI container manages services throughout the application.
- **Environment-based configuration** — Configuration providers for JSON, environment variables, command-line arguments, and more.
- **Rich logging and diagnostics** — Structured logging via `ILogger`, tracing, and metrics.
- **Unified programming model** — MVC, Razor Pages, Minimal APIs, Blazor, SignalR, and gRPC all run on the same foundation.
- **Security-first** — Built-in authentication, authorization, data protection, HTTPS redirection, and HSTS.

---

### Prerequisites

- Solid understanding of **C#** (classes, interfaces, async/await, generics).
- Familiarity with **HTTP** (verbs, status codes, headers).
- Basic knowledge of **dependency injection** as a design pattern.
- Understanding of **configuration** (JSON files, environment variables).
- Awareness of **middleware** and the request/response pipeline concept.

---

### Related Programming Areas

- **REST API development** — ASP.NET Core is the primary framework for building HTTP APIs in .NET.
- **Microservices** — Minimal APIs and built-in DI make ASP.NET Core ideal for microservices.
- **Real-time applications** — SignalR provides WebSocket-based real-time communication.
- **gRPC services** — High-performance RPC using HTTP/2 and Protocol Buffers.
- **Cloud-native development** — Designed for containers, Kubernetes, and serverless.

---

### Core Concepts / Features

1. ASP.NET Core (Overarching Concept)
2. ASP.NET Core Architecture (`Program.cs` and the Generic Host)
3. Middleware Pipeline
4. Dependency Injection
5. Configuration
6. Logging
7. Hosting: Kestrel, IIS, and HTTP.sys
8. ASP.NET Core Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated code examples, real-world cases, and references.

---

## 1. ASP.NET Core (Overarching Concept)

### Definitions

**Core Definition**
ASP.NET Core is a cross-platform, high-performance, open-source framework for building modern, cloud-enabled web applications.

**Technical Definition**
ASP.NET Core is built on the .NET runtime and provides a modular HTTP request pipeline, integrated dependency injection, environment-based configuration, and rich diagnostics. It ships with Kestrel as the default HTTP server and supports a variety of application models including MVC, Razor Pages, Minimal APIs, Blazor, SignalR, and gRPC.

**Beginner-Friendly Explanation**
ASP.NET Core is **the framework you use to build web apps and APIs with C#**. It handles the plumbing (HTTP, routing, DI, configuration) so you can focus on your business logic.

---

### Purposes

- To build **web applications** (MVC, Razor Pages, Blazor).
- To build **HTTP APIs** (Minimal APIs, controllers).
- To build **real-time applications** (SignalR).
- To build **high-performance RPC services** (gRPC).
- To provide a **unified, modular foundation** for all web workloads on .NET.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);
// register services
var app = builder.Build();
// configure middleware pipeline
app.Run();
```

**Component breakdown**

- `WebApplication.CreateBuilder(args)` — Creates the builder with defaults (configuration, logging, DI).
- `builder.Services` — The DI container for registering services.
- `builder.Build()` — Builds the application.
- `app.Use...` — Adds middleware.
- `app.Run()` — Starts the server.

#### Syntax Rules

- ASP.NET Core apps use top-level statements in `Program.cs`.
- The `WebApplicationBuilder` provides `Services`, `Configuration`, `Logging`, and `Environment`.
- Middleware is added via `Use...` extension methods.
- The app is started with `app.Run()`.

#### Constraints and Limitations

- The framework is opinionated about DI but allows third-party containers.
- Not all features are available in all .NET versions (e.g., Minimal APIs require .NET 6+).

---

### Annotated Code Examples and Expected Outputs

#### Example: Minimal ASP.NET Core App

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

**Expected Output (HTTP response)**
```
Hello, World!
```

**Why this output occurs**
`MapGet` registers a minimal API endpoint that responds to `GET /` with the string `"Hello, World!"`. Kestrel starts and listens for requests when `app.Run()` is called.

---

### Real-World Cases

- **Enterprise web apps** — MVC or Razor Pages for server-rendered HTML.
- **Public APIs** — Minimal APIs or controllers for JSON APIs.
- **Real-time dashboards** — SignalR for live updates.
- **Microservices** — Lightweight Minimal APIs with DI and configuration.

---

### References

- Overview of ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/refs/heads/guardrex-patch-2/aspnetcore/overview.md
- ASP.NET Core fundamentals overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/a99f04a765c23f00db49350b4ea2b4857b809081/aspnetcore/fundamentals/index.md

---

## 2. ASP.NET Core Architecture (`Program.cs` and the Generic Host)

### Definitions

**Core Definition**
The ASP.NET Core architecture is centered on the **`WebApplicationBuilder`** and **`WebApplication`** types, which configure services, build the application, and define the request pipeline in `Program.cs`.

**Technical Definition**
When `WebApplication.CreateBuilder(args)` is called, the framework creates a **generic host** with default configuration sources (appsettings.json, environment variables, command-line arguments), default logging providers (Console, Debug, EventSource, Windows EventLog), and a DI container pre-populated with framework services. Services are registered on `builder.Services`, and the pipeline is defined on the built `WebApplication` using middleware extension methods.

**Beginner-Friendly Explanation**
`Program.cs` is the **command center** of an ASP.NET Core app. You tell it what services you need (`builder.Services.Add...`), then you tell it how to handle requests (`app.Use...` and `app.Map...`).

---

### Purposes

- To **configure services** (DI registrations) before the app starts.
- To **define the middleware pipeline** that processes HTTP requests.
- To **load configuration** from multiple sources.
- To **set up logging** providers and filters.
- To **control application startup and shutdown**.

---

### Syntax Rules and Structure

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddControllers();
builder.Services.AddScoped<IMyService, MyService>();

var app = builder.Build();

// Configure pipeline
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Component breakdown**

- `builder.Services` — `IServiceCollection` for DI.
- `builder.Configuration` — `IConfiguration` for settings.
- `builder.Logging` — `ILoggingBuilder` for logging.
- `builder.Environment` — `IWebHostEnvironment` for environment info.
- `app` — `WebApplication` implementing `IApplicationBuilder` and `IEndpointRouteBuilder`.

#### Syntax Rules

- `Program.cs` uses top-level statements (no explicit `Main` method).
- Service registration happens before `builder.Build()`.
- Middleware configuration happens after `Build()`.
- The app is started with `app.Run()`.

#### Constraints and Limitations

- Services registered as `Scoped` cannot be resolved from the root provider.
- Configuration changes require a restart unless `reloadOnChange` is enabled.

---

### Annotated Code Examples and Expected Outputs

#### Example: Full `Program.cs` with Services and Middleware

```csharp
var builder = WebApplication.CreateBuilder(args);

// Services
builder.Services.AddControllers();
builder.Services.AddSingleton<IClock, SystemClock>();

var app = builder.Build();

// Middleware
app.UseExceptionHandler("/error");
app.UseHttpsRedirection();
app.UseRouting();
app.MapControllers();

app.Run();
```

**Expected Output (conceptual)**
```
// The app starts, registers controllers and IClock, configures exception handling,
// HTTPS redirection, routing, and controller endpoints.
```

**Why this structure is used**
It separates service configuration from pipeline configuration, following the ASP.NET Core convention.

---

### Real-World Cases

- **Web APIs** — `AddControllers()` and `MapControllers()`.
- **Minimal APIs** — `MapGet`, `MapPost`, etc.
- **Razor Pages** — `AddRazorPages()` and `MapRazorPages()`.
- **Blazor Server** — `AddServerSideBlazor()` and `MapBlazorHub()`.

---

### References

- ASP.NET Core fundamentals overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/a99f04a765c23f00db49350b4ea2b4857b809081/aspnetcore/fundamentals/index.md
- Program.cs — ASP.NET Core fundamentals — Microsoft Learn

---

## 3. Middleware Pipeline

### Definitions

**Core Definition**
Middleware is software assembled into a pipeline to handle HTTP requests and responses. Each middleware component can perform work before and after the next component and can choose whether to pass the request along.

**Technical Definition**
The ASP.NET Core request pipeline consists of a sequence of **request delegates** called one after another. Each delegate receives an `HttpContext` and a `RequestDelegate next`. Middleware is configured using `Use`, `Run`, and `Map` extension methods. `Use` adds a middleware that can call `next`; `Run` adds a **terminal middleware** that does not call `next`; `Map` branches the pipeline based on the request path. Middleware order is critical: exception handling should be early, authentication before authorization, and endpoint routing before endpoints.

**Beginner-Friendly Explanation**
The middleware pipeline is a **conveyor belt** for HTTP requests. Each middleware is a station on the belt. A request arrives, passes through each station in order, and each station can do something to the request or response. Some stations pass the request to the next; some (terminal middleware) stop the belt and send the response immediately.

---

### Purposes

- To **process HTTP requests** in a modular, composable way.
- To **short-circuit** requests (e.g., authentication failure returns 401 without invoking later middleware).
- To **branch** the pipeline based on path or other conditions.
- To **order cross-cutting concerns** correctly (exception handling → HTTPS → static files → routing → auth → endpoints).
- To **integrate framework features** (static files, authentication, CORS, compression).

---

### Syntax Rules and Structure

```csharp
// Inline middleware
app.Use(async (context, next) =>
{
    // before
    await next();
    // after
});

// Terminal middleware
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});

// Branching
app.Map("/branch", branch =>
{
    branch.Run(async context => await context.Response.WriteAsync("Branch"));
});
```

**Component breakdown**

- `Use(Func<HttpContext, RequestDelegate, Task>)` — Middleware that can call `next`.
- `Run(RequestDelegate)` — Terminal middleware; does not call `next`.
- `Map(string, Action<IApplicationBuilder>)` — Branches the pipeline.
- `MapGet`, `MapPost`, etc. — Endpoint routing (Minimal APIs).

#### Syntax Rules

- Middleware is added in order; the pipeline executes in the order added.
- Each middleware must call `next()` to pass the request along, unless it is terminal.
- `Run` delegates do not receive a `next` parameter.
- Middleware can be implemented as inline lambdas or as reusable classes.

#### Constraints and Limitations

- Incorrect ordering can break security (e.g., authorization before authentication).
- Terminal middleware prevents subsequent middleware from running.
- Middleware runs for every request, so performance-sensitive code should be efficient.

---

### Annotated Code Examples and Expected Outputs

#### Example: Ordered Middleware Pipeline

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler("/error");          // 1. catch exceptions
app.UseHsts();                              // 2. HSTS (production)
app.UseHttpsRedirection();                  // 3. redirect HTTP to HTTPS
app.UseStaticFiles();                       // 4. serve static files
app.UseRouting();                           // 5. match routes
app.UseAuthentication();                    // 6. authenticate
app.UseAuthorization();                     // 7. authorize
app.MapControllers();                       // 8. execute endpoints

app.Run();
```

**Expected Output (conceptual)**
```
// Requests flow through exception handling, HSTS, HTTPS redirection,
// static files, routing, authentication, authorization, and controllers.
```

**Why this order is used**
Microsoft’s recommended order ensures exceptions are caught early, security headers are applied, static files are served efficiently, and authentication precedes authorization.

---

#### Example: Custom Middleware Class

```csharp
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    public RequestTimingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        var sw = Stopwatch.StartNew();
        await _next(context);
        sw.Stop();
        context.Response.Headers["X-Elapsed-Ms"] = sw.ElapsedMilliseconds.ToString();
    }
}

// Registration
app.UseMiddleware<RequestTimingMiddleware>();
```

**Expected Output (conceptual)**
```
// Every response includes an X-Elapsed-Ms header with the request duration.
```

**Why this pattern is used**
Reusable middleware classes encapsulate logic and can be tested independently.

---

### Real-World Cases

- **Exception handling** — `UseExceptionHandler` catches unhandled exceptions.
- **HTTPS enforcement** — `UseHttpsRedirection` and `UseHsts`.
- **Static files** — `UseStaticFiles` serves CSS, JS, and images.
- **Authentication/authorization** — `UseAuthentication` and `UseAuthorization`.
- **Response compression** — `UseResponseCompression` reduces bandwidth.
- **CORS** — `UseCors` enables cross-origin requests.

---

### References

- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md
- Middleware order — ASP.NET Core middleware — Microsoft Learn (GitHub) — https://github.com

---

## 4. Dependency Injection

### Definitions

**Core Definition**
Dependency injection (DI) is a design pattern in which a class receives its dependencies from an external container rather than creating them itself. ASP.NET Core provides a **built-in DI container**.

**Technical Definition**
ASP.NET Core supports the DI software design pattern, a technique for achieving Inversion of Control (IoC) between classes and their dependencies. Dependencies are registered in an `IServiceCollection` (typically in `Program.cs`) with a lifetime of **Singleton**, **Scoped**, or **Transient**. Services are resolved via constructor injection, and the framework manages their creation and disposal. The built-in container is `Microsoft.Extensions.DependencyInjection`; third-party containers can be substituted.

**Beginner-Friendly Explanation**
Dependency injection means **“don’t build what you need—ask for it.”** Instead of `new DatabaseConnection()` inside a controller, you declare a constructor parameter of type `IDatabaseConnection`, and the DI container provides the right implementation. This makes code testable, loosely coupled, and easier to change.

---

### Purposes

- To **decouple** classes from their concrete dependencies.
- To **manage lifetimes** of services (singleton, scoped, transient).
- To **improve testability** by allowing mocks to be injected.
- To **centralize configuration** of service implementations.
- To **avoid resource leaks** by letting the container dispose services.

---

### Syntax Rules and Structure

```csharp
// Registration
builder.Services.AddSingleton<IMyService, MyService>();
builder.Services.AddScoped<IRepository, Repository>();
builder.Services.AddTransient<IEmailSender, EmailSender>();

// Resolution (constructor injection)
public class MyController : Controller
{
    private readonly IMyService _service;
    public MyController(IMyService service) => _service = service;
}
```

**Component breakdown**

- `AddSingleton` — One instance for the entire application lifetime.
- `AddScoped` — One instance per HTTP request (or per scope).
- `AddTransient` — A new instance every time it is requested.
- Constructor injection — The container supplies dependencies.

#### Syntax Rules

- Services are registered before `builder.Build()`.
- Constructor injection is the preferred resolution mechanism.
- Scoped services cannot be resolved from the root provider (use `CreateScope()` for background work).
- The container disposes `IDisposable` services it creates.

#### Constraints and Limitations

- Singleton services must be thread-safe.
- Scoped services should not be captured by singletons (captive dependency).
- The built-in container is intentionally simple; advanced scenarios may require a third-party container.

---

### Annotated Code Examples and Expected Outputs

#### Example: Three Lifetimes

```csharp
builder.Services.AddSingleton<ISingletonService, SingletonService>();
builder.Services.AddScoped<IScopedService, ScopedService>();
builder.Services.AddTransient<ITransientService, TransientService>();

var app = builder.Build();

app.MapGet("/", (ISingletonService s, IScopedService sc, ITransientService t) =>
    $"Singleton: {s.Id}, Scoped: {sc.Id}, Transient: {t.Id}");
```

**Expected Output (first request)**
```
Singleton: 1, Scoped: 1, Transient: 1
```

**Expected Output (second request)**
```
Singleton: 1, Scoped: 2, Transient: 2
```

**Why this output occurs**
The singleton ID stays the same across requests; the scoped ID changes per request; the transient ID changes on every resolution.

---

### Real-World Cases

- **DbContext** — Registered as scoped by EF Core.
- **Repositories and services** — Registered as scoped or transient.
- **Configuration and caches** — Registered as singleton.
- **HttpClient** — `AddHttpClient` manages lifetimes and handlers.

---

### References

- Dependency injection in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/refs/heads/refactorcaching/aspnetcore/fundamentals/dependency-injection.md
- Dependency injection in .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection

---

## 5. Configuration

### Definitions

**Core Definition**
Configuration is the mechanism by which an ASP.NET Core app reads settings from external sources such as JSON files, environment variables, and command-line arguments.

**Technical Definition**
ASP.NET Core configuration is built on **configuration providers**. The default providers, in increasing priority, are: `appsettings.json`, `appsettings.{Environment}.json`, environment variables, and command-line arguments. Providers added later override earlier ones. The `IConfiguration` interface provides access via keys (including hierarchical keys with `:`), and the **Options pattern** (`IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`) binds configuration sections to strongly typed classes.

**Beginner-Friendly Explanation**
Configuration is **where your app reads its settings**. You put defaults in `appsettings.json`, override them per environment in `appsettings.Development.json` or `appsettings.Production.json`, and override again with environment variables or command-line arguments in deployment. The Options pattern lets you access these settings as strongly typed C# objects.

---

### Purposes

- To **separate configuration** from code.
- To support **multiple environments** (Development, Staging, Production).
- To allow **deployment-time overrides** without rebuilding.
- To **bind configuration** to strongly typed options.
- To support **secrets management** (Secret Manager, Azure Key Vault).

---

### Syntax Rules and Structure

```json
// appsettings.json
{
  "Position": {
    "Title": "Editor",
    "Name": "Joe Smith"
  },
  "MyKey": "My appsettings.json Value"
}
```

```csharp
// Reading configuration
var title = builder.Configuration["Position:Title"];

// Options pattern
builder.Services.Configure<PositionOptions>(
    builder.Configuration.GetSection("Position"));
```

**Component breakdown**

- `appsettings.json` — Base settings.
- `appsettings.{Environment}.json` — Environment-specific overrides.
- Environment variables — `Logging__LogLevel__Default=Information` (double underscore for `:`).
- Command-line arguments — `--MyKey=value`.
- `IConfiguration` — Key-value access.
- `IOptions<T>` — Strongly typed binding.

#### Syntax Rules

- Providers are added in order; later providers override earlier ones.
- Hierarchical keys use `:` (JSON) or `__` (environment variables).
- `reloadOnChange: true` is enabled by default for JSON files.
- Secrets should never be committed to source control.

#### Constraints and Limitations

- Configuration is read-only at runtime unless a reloadable provider is used.
- Environment-specific files are optional; only `appsettings.json` is required.

---

### Annotated Code Examples and Expected Outputs

#### Example: Options Pattern with Environment Override

```csharp
public class PositionOptions
{
    public const string Section = "Position";
    public string Title { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
}

builder.Services.Configure<PositionOptions>(
    builder.Configuration.GetSection(PositionOptions.Section));

app.MapGet("/position", (IOptions<PositionOptions> options) =>
{
    var p = options.Value;
    return $"Title: {p.Title}, Name: {p.Name}";
});
```

**Expected Output (Development, with override in `appsettings.Development.json`)**
```
Title: Editor, Name: Joe Smith (Development override)
```

**Why this output occurs**
The `appsettings.Development.json` file overrides the base `appsettings.json` values when the environment is `Development`.

---

### Real-World Cases

- **Connection strings** — `ConnectionStrings:DefaultConnection`.
- **Feature flags** — `Features:NewDashboard=true`.
- **Logging levels** — `Logging:LogLevel:Default`.
- **Third-party API keys** — Stored in Secret Manager or Key Vault.

---

### References

- Configuration in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/configuration/index.md
- Configuration providers — Microsoft Learn (video2.skills-academy.com) — https://video2.skills-academy.com/ca-es/aspnet/core/fundamentals/configuration/

---

## 6. Logging

### Definitions

**Core Definition**
Logging is the framework-provided mechanism for recording application behavior and diagnosing problems using the `ILogger` API and configurable logging providers.

**Technical Definition**
ASP.NET Core supports **high-performance, structured logging** via `Microsoft.Extensions.Logging.ILogger`. Logs are written to **logging providers** (Console, Debug, EventSource, Windows EventLog, Application Insights, etc.). Multiple providers can be enabled simultaneously. Log levels (`Trace`, `Debug`, `Information`, `Warning`, `Error`, `Critical`) and categories allow fine-grained filtering, typically configured in `appsettings.json` under the `Logging` section. Structured logging captures properties (not just strings), making logs queryable.

**Beginner-Friendly Explanation**
Logging is **how your app tells you what it’s doing**. You inject `ILogger<T>` into your classes and call `LogInformation`, `LogWarning`, `LogError`, etc. The logs go to the console in development and to files, Event Log, or cloud services in production. Structured logging means the log entries carry named properties (like `OrderId`) that you can filter and search.

---

### Purposes

- To **diagnose problems** in development and production.
- To **monitor application behavior** and performance.
- To **audit** security and business events.
- To **feed observability platforms** (Application Insights, OpenTelemetry).
- To provide **structured, queryable** log data.

---

### Syntax Rules and Structure

```csharp
// Injection
public class MyService
{
    private readonly ILogger<MyService> _logger;
    public MyService(ILogger<MyService> logger) => _logger = logger;

    public void DoWork()
    {
        _logger.LogInformation("Work started at {Time}", DateTime.UtcNow);
    }
}
```

```json
// appsettings.json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft": "Warning",
    "Microsoft.Hosting.Lifetime": "Information"
  }
}
```

**Component breakdown**

- `ILogger<T>` — Generic logger with category `T`.
- `LogInformation`, `LogWarning`, `LogError` — Level-specific methods.
- Message templates — `{Property}` placeholders for structured logging.
- Providers — Console, Debug, EventSource, Windows EventLog, Application Insights.
- Configuration — `Logging:LogLevel` controls filtering.

#### Syntax Rules

- Use **message templates** with named properties, not string concatenation.
- Use the appropriate **log level** for the event.
- Configure providers in `Program.cs` or via `appsettings.json`.
- `builder.Logging.ClearProviders()` removes defaults; add providers explicitly.

#### Constraints and Limitations

- Logging every request at `Information` can be noisy in production.
- String interpolation in log messages (`$"..."`) prevents structured logging.
- Sensitive data should never be logged.

---

### Annotated Code Examples and Expected Outputs

#### Example: Structured Logging

```csharp
app.MapGet("/orders/{id}", (int id, ILogger<Program> logger) =>
{
    logger.LogInformation("Fetching order {OrderId}", id);
    return Results.Ok(new { Id = id, Total = 99.99m });
});
```

**Expected Output (console, development)**
```
info: Program[0]
      Fetching order 42
```

**Why this output occurs**
The `ILogger<Program>` writes to the Console provider; the `{OrderId}` property is captured structurally.

---

### Real-World Cases

- **Unhandled exception logging** — `UseExceptionHandler` logs exceptions.
- **Request logging** — `UseHttpLogging` logs HTTP requests and responses.
- **Business events** — `LogInformation("Order {OrderId} placed", id)`.
- **Security auditing** — `LogWarning("Failed login attempt for {User}", user)`.

---

### References

- Logging in .NET and ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/aspnet/Docs/main/aspnetcore/fundamentals/logging/index.md
- Logging in .NET and ASP.NET Core — Microsoft Learn (learn.microsoft.com) — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/

---

## 7. Hosting: Kestrel, IIS, and HTTP.sys

### Definitions

**Core Definition**
Hosting refers to the **HTTP server implementation** that listens for requests and the **hosting model** that determines how the ASP.NET Core app runs. The main servers are **Kestrel** (cross-platform, default), **IIS** (Windows, in-process or out-of-process), and **HTTP.sys** (Windows-only).

**Technical Definition**
ASP.NET Core ships with three server implementations:
- **Kestrel** — The default, cross-platform HTTP server. It provides the best performance and memory utilization but lacks some advanced features of HTTP.sys.
- **IIS HTTP Server** — An in-process server for IIS. When hosted in-process, the app runs in the same process as the IIS worker process. Out-of-process hosting uses Kestrel behind IIS.
- **HTTP.sys** — A Windows-only server based on the HTTP.sys kernel driver and HTTP Server API. It is recommended only when the app requires features not available in Kestrel (e.g., port sharing, Windows authentication without IIS).
Kestrel is recommended over HTTP.sys unless the app requires features not available in Kestrel.

**Beginner-Friendly Explanation**
Hosting is **where your app listens for requests**. Kestrel is the built-in server that runs everywhere. IIS is the Windows web server; you can run your app inside IIS (in-process, faster) or behind IIS (out-of-process, using Kestrel). HTTP.sys is another Windows-only server for special cases. For most apps, **Kestrel is the right choice**, often behind a reverse proxy like IIS, Nginx, or YARP.

---

### Purposes

- To **listen for HTTP requests** and pass them to the application pipeline.
- To provide **cross-platform hosting** (Kestrel).
- To integrate with **Windows infrastructure** (IIS, HTTP.sys).
- To support **different deployment models** (in-process, out-of-process, self-hosted).
- To optimize **performance** (Kestrel) or **feature compatibility** (IIS/HTTP.sys).

---

### Syntax Rules and Structure

```csharp
// Kestrel (default) — no extra configuration needed
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.Run();   // Kestrel is the default server
```

```csharp
// HTTP.sys
builder.WebHost.UseHttpSys(options =>
{
    options.Authentication.Schemes = AuthenticationSchemes.NTLM;
});
```

**Component breakdown**

- **Kestrel** — Default; cross-platform; configured via `Kestrel` section in `appsettings.json` or code.
- **IIS** — In-process (`<AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>`) or out-of-process; requires the ASP.NET Core Module.
- **HTTP.sys** — `UseHttpSys()`; Windows-only; supports kernel-mode authentication, port sharing.

#### Syntax Rules

- Kestrel is used by default when no other server is configured.
- IIS integration is configured in the project file and `Program.cs`.
- HTTP.sys requires `UseHttpSys()`.
- For production, Kestrel should typically run behind a reverse proxy (IIS, Nginx, Apache) for security and load balancing.

#### Constraints and Limitations

- Kestrel does not support Windows authentication natively; use IIS or HTTP.sys for that.
- HTTP.sys is Windows-only.
- In-process IIS hosting is faster but has version compatibility requirements.

---

### Annotated Code Examples and Expected Outputs

#### Example: Kestrel with HTTPS

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000);
    options.ListenAnyIP(5001, listen => listen.UseHttps());
});
```

**Expected Output (conceptual)**
```
// Kestrel listens on HTTP port 5000 and HTTPS port 5001.
```

**Why this configuration is used**
It explicitly configures Kestrel endpoints, which is common in containerized deployments.

---

### Real-World Cases

- **Cloud deployments** — Kestrel behind a load balancer or reverse proxy.
- **Windows enterprise** — IIS in-process hosting with Windows authentication.
- **Self-hosted services** — HTTP.sys for Windows services requiring port sharing.

---

### References

- Web server implementations in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/servers/index.md
- Kestrel — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel
- HTTP.sys — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/httpsys

---

## 8. ASP.NET Core Best Practices

### Definitions

**Core Definition**
ASP.NET Core best practices are the **guidelines for building secure, performant, reliable, and maintainable** ASP.NET Core applications.

**Technical Definition**
Best practices span several areas: **middleware ordering** (exception handling first, authentication before authorization); **DI lifetime management** (scoped services request-bound, no captive dependencies); **configuration security** (no secrets in source control, use Secret Manager/Key Vault); **response compression and caching** for performance; **static asset optimization** (`MapStaticAssets` in .NET 9+); and **security hardening** (HTTPS redirection, HSTS, authentication/authorization).

**Beginner-Friendly Explanation**
The best practices are the habits that keep ASP.NET Core apps **fast, safe, and easy to maintain**: order your middleware correctly, don’t put secrets in code, use the right DI lifetimes, compress and cache responses, and always use HTTPS.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Correct middleware order
app.UseExceptionHandler("/error");
app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

// 2. Correct DI lifetimes
builder.Services.AddScoped<IRepository, Repository>();   // per request
builder.Services.AddSingleton<ICache, Cache>();           // thread-safe

// 3. Secrets management
// Development: dotnet user-secrets
// Production: Azure Key Vault / environment variables

// 4. Response compression and caching
builder.Services.AddResponseCompression();
app.UseResponseCompression();
builder.Services.AddResponseCaching();
app.UseResponseCaching();

// 5. Static asset optimization (.NET 9+)
app.MapStaticAssets();
```

**Component breakdown**

- **Middleware order** — Security and correctness depend on it.
- **DI lifetimes** — Scoped for request-bound services; singleton only for thread-safe services.
- **Secrets** — Never commit; use Secret Manager in development and a secure store in production.
- **Compression/caching** — Reduce bandwidth and improve responsiveness.
- **Static assets** — `MapStaticAssets` optimizes delivery with compression, caching, and fingerprinting.

#### Rules and Constraints

- **Do** order middleware correctly (exception handler → HSTS → HTTPS → static files → routing → auth → authorization → endpoints).
- **Do** keep scoped services request-bound and avoid resolving them from singletons.
- **Do** use Secret Manager in development and Key Vault or environment variables in production.
- **Do** enable response compression and caching.
- **Do** use `MapStaticAssets` in .NET 9+ for static files.
- **Do not** commit secrets to source control.
- **Do not** use `UseStaticFiles` in .NET 9+ when `MapStaticAssets` is available.

---

### Annotated Code Examples and Expected Outputs

#### Example: Production-Ready `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddResponseCompression();
builder.Services.AddResponseCaching();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseResponseCompression();
app.UseResponseCaching();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output (conceptual)**
```
// Secure, compressed, cached responses with correct middleware ordering.
```

**Why this configuration is used**
It follows Microsoft’s recommended production patterns for security, performance, and middleware ordering.

---

### Real-World Cases

- **Public-facing APIs** — HTTPS redirection, HSTS, response compression.
- **Enterprise apps** — Secret Manager, Azure Key Vault, DI lifetime discipline.
- **High-traffic sites** — Response caching, static asset optimization, CDN.

---

### References

- ASP.NET Core best practices — Microsoft Learn (Japanese) — https://learn.microsoft.com/ja-jp/aspnet/core/performance/
- Middleware order — ASP.NET Core middleware — Microsoft Learn (GitHub) — https://github.com
- Configuration in ASP.NET Core — Secret Manager — Microsoft Learn
- Response compression in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `UseStaticFiles` | **Superseded by `MapStaticAssets`** in .NET 9+ for optimized static asset delivery. |
| `Startup.cs` | **Legacy**; replaced by top-level `Program.cs` with `WebApplicationBuilder` in .NET 6+. |
| `IHostingEnvironment` | **Obsolete**; use `IWebHostEnvironment`. |
| In-process IIS hosting | **Recommended** over out-of-process for performance. |
| HTTP.sys | **Windows-only**; recommended only when Kestrel lacks required features. |
| `AddResponseCompression` | **Stable**; recommended for production performance. |
| Secret Manager | **Development only**; not for production secrets. |

---

## References (Consolidated)

- Overview of ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/refs/heads/guardrex-patch-2/aspnetcore/overview.md
- ASP.NET Core fundamentals overview — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/a99f04a765c23f00db49350b4ea2b4857b809081/aspnetcore/fundamentals/index.md
- ASP.NET Core middleware — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/middleware/index.md
- Dependency injection in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/refs/heads/refactorcaching/aspnetcore/fundamentals/dependency-injection.md
- Configuration in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/configuration/index.md
- Configuration providers — Microsoft Learn (video2.skills-academy.com) — https://video2.skills-academy.com/ca-es/aspnet/core/fundamentals/configuration/
- Logging in .NET and ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/aspnet/Docs/main/aspnetcore/fundamentals/logging/index.md
- Web server implementations in ASP.NET Core — Microsoft Learn (GitHub) — https://raw.githubusercontent.com/dotnet/AspNetCore.Docs/main/aspnetcore/fundamentals/servers/index.md
- Kestrel — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel
- HTTP.sys — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/httpsys
- ASP.NET Core best practices — Microsoft Learn (Japanese) — https://learn.microsoft.com/ja-jp/aspnet/core/performance/
- Response compression in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- What's new in ASP.NET Core 9 — Microsoft Learn (Spanish) — https://learn.microsoft.com/es-es/aspnet/core/release-notes/aspnetcore-9.0