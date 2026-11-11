# C# Logging — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Logging is the systematic recording of events that occur during the execution of an application. In .NET, logging is a **first-class framework feature** built on the `Microsoft.Extensions.Logging` abstraction, which provides a unified API (`ILogger`) for emitting log messages at varying severity levels to configurable destinations called **providers**. Logging is essential for diagnostics, auditing, monitoring, and understanding application behavior in development and production environments.

**Technical Definition**
The .NET logging framework is built around the **`ILogger<T>`** and **`ILoggerFactory`** abstractions in the `Microsoft.Extensions.Logging.Abstractions` package. Log messages are emitted at one of six levels—**Trace (0)**, **Debug (1)**, **Information (2)**, **Warning (3)**, **Error (4)**, and **Critical (5)**—with **None (6)** used only for filtering . Messages are written to **providers** (Console, Debug, EventSource, EventLog, Application Insights, Azure App Services) which are configured via code or `appsettings.json` . **Structured logging** uses message templates with named placeholders (`{OrderId}`) to preserve the association between names and values, enabling queryable log data . **`LoggerMessage`** source generation provides zero-allocation logging for hot paths . Third-party frameworks such as **Serilog** and **NLog** extend or replace the built-in logging with advanced features like sinks, destructuring, and XML-based routing .

**Beginner-Friendly Explanation**
Logging is **“writing down what your app is doing”** so you can figure out what went wrong later. Think of it as a **flight recorder** for your software. Normally, you might sprinkle `Console.WriteLine` calls throughout your code, but that gives you no levels, no filtering, no timestamps, and no structure. The .NET logging framework gives you a **standard API** (`ILogger<T>`) that writes messages to configurable **destinations** (console, files, cloud services) at different **severity levels** (Trace, Debug, Information, Warning, Error, Critical). You can control **which** messages are written and **where** without changing your code—just configuration. **Structured logging** goes further: instead of writing “Order 42 failed,” you write a template `"Order {OrderId} failed"` with the value `42`, and the logging system stores the ID as a searchable property. This makes logs **queryable** and **machine-readable**.

---

### Key Characteristics

- **Abstraction-first** — Application code depends on `ILogger<T>`, not concrete logging libraries; providers can be swapped without changing business logic .
- **Six severity levels** — Trace, Debug, Information, Warning, Error, Critical; None is a filtering value only .
- **Provider-based** — Messages are written to one or more providers (Console, Debug, EventSource, EventLog, Application Insights) .
- **Structured logging** — Message templates with named placeholders (`{Name}`) preserve property names and values .
- **Configuration-driven** — Log levels and providers are configured in `appsettings.json` and can be overridden per environment .
- **Zero-allocation options** — `[LoggerMessage]` source generation produces zero-allocation logging for hot paths .
- **Scoped context** — Log scopes attach contextual properties that propagate across async boundaries .
- **Third-party ecosystem** — Serilog and NLog provide advanced sinks, destructuring, and routing .

---

### Prerequisites

- Basic understanding of **C#** (classes, interfaces, dependency injection).
- Familiarity with **ASP.NET Core** or the **Generic Host** (`Program.cs`, `WebApplicationBuilder`).
- Knowledge of **configuration** (`appsettings.json`, environment variables).
- Awareness of **`IDisposable`** and async/await (for scopes and background logging).
- Basic understanding of **structured logging** concepts (helpful but not required).

---

### Related Programming Areas

- **Dependency injection** — `ILogger<T>` is resolved from DI.
- **Configuration** — Log levels and providers are configured via `appsettings.json`.
- **Observability** — Logs, metrics, and traces form the three pillars of observability.
- **Monitoring** — Application Insights, OpenTelemetry, and Seq consume structured logs.
- **Resilience** — Logging is critical for diagnosing retries, circuit breakers, and failures.

---

### Core Concepts / Features

1. Logging (Overarching Concept)
2. `ILogger<T>`
3. Log Levels
4. Log Providers
5. Structured Logging
6. Serilog
7. NLog
8. Logging Best Practices

---

## 1. Logging (Overarching Concept)

### Definitions

**Core Definition**
Logging is the framework-provided mechanism for recording application events with severity, context, and destination control.

**Technical Definition**
The `Microsoft.Extensions.Logging` framework provides `ILogger`, `ILoggerFactory`, `ILoggerProvider`, and `ILogger<T>` abstractions. The `ILogger` interface exposes `Log<TState>` and `IsEnabled(LogLevel)`; convenience extension methods (`LogInformation`, `LogWarning`, etc.) wrap the core method. Providers implement `ILoggerProvider` and are registered via `ILoggerFactory.AddProvider`. The framework is used uniformly across ASP.NET Core, Worker Services, MAUI, and console applications.

**Beginner-Friendly Explanation**
Logging is **the standard way to write diagnostic messages in .NET**. It gives you levels, providers, filtering, and structure—all through a single, consistent API.

---

### Purposes

- To **diagnose problems** in development and production.
- To **audit** security and business events.
- To **monitor** application health and performance.
- To **feed observability platforms** with structured data.
- To **control log volume** via levels and filtering.

---

### Syntax Rules and Structure

```csharp
// In Program.cs (host builder)
var builder = WebApplication.CreateBuilder(args);
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();

// In a service
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

**Component breakdown**

- `builder.Logging` — Configures providers and filters.
- `AddConsole()`, `AddDebug()` — Registers built-in providers.
- `ILogger<T>` — Injected per class; category is the fully qualified type name .
- `LogInformation` — Convenience method for `LogLevel.Information`.

#### Syntax Rules

- `ILogger<T>` is equivalent to `ILoggerFactory.CreateLogger(typeof(T).FullName)` .
- Providers are registered at startup and are immutable afterward.
- Filters control which messages reach which providers.
- Log levels are ordered from lowest (Trace) to highest (Critical) severity .

#### Constraints and Limitations

- Direct use of `Console.WriteLine` bypasses levels, filtering, and structure.
- Logging every event at `Information` can be noisy and expensive.
- Sensitive data must never be logged at any level .

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Logging Setup

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Logging.ClearProviders();
builder.Logging.AddConsole();

var app = builder.Build();

app.MapGet("/", (ILogger<Program> logger) =>
{
    logger.LogInformation("Request received at {Timestamp}", DateTime.UtcNow);
    return "OK";
});

app.Run();
```

**Expected Output (console)**
```
info: Program[0]
      Request received at 11/11/2026 10:30:00
```

**Why this output occurs**
The `ILogger<Program>` writes to the Console provider at `Information` level; the `{Timestamp}` placeholder is captured as a structured property.

---

### Real-World Cases

- **Web APIs** — Request logging, error logging, performance timing.
- **Background services** — Job start/completion/failure logging.
- **Desktop apps** — Diagnostic logging for support.
- **Microservices** — Structured logs shipped to centralized platforms.

---

### References

- Logging in C# and .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/
- Logging in .NET and ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/

---

## 2. `ILogger<T>`

### Definitions

**Core Definition**
`ILogger<T>` is the generic logging interface whose category name is the fully qualified name of `T`.

**Technical Definition**
`Microsoft.Extensions.Logging.ILogger<T>` derives from `ILogger`. The type parameter `T` is used to set the **category** of the logger to `typeof(T).FullName`, enabling per-class filtering. `ILogger<T>` is registered in DI automatically by the host; injecting it requires no additional registration.

**Beginner-Friendly Explanation**
`ILogger<T>` is **the logger for your class**. You put your class name in the angle brackets, and the framework tags every message with that category so you can filter logs by class.

---

### Purposes

- To provide a **per-class logger** with a meaningful category.
- To enable **category-based filtering** (e.g., `Microsoft` → Warning).
- To avoid **manual category strings**.
- To integrate seamlessly with **dependency injection**.

---

### Syntax Rules and Structure

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    public OrderService(ILogger<OrderService> logger) => _logger = logger;
}
```

**Component breakdown**

- `ILogger<OrderService>` — Generic logger.
- Category — `MyApp.Services.OrderService` (fully qualified name) .
- `_logger` — Field for reuse.

#### Syntax Rules

- `ILogger<T>` is registered by the host; no manual registration needed.
- The category is the fully qualified type name.
- Use `ILoggerFactory.CreateLogger("CustomCategory")` for custom categories .

#### Constraints and Limitations

- The category is fixed at construction; use `ILoggerFactory` for dynamic categories.
- `ILogger<T>` is thread-safe and can be stored in a readonly field.

---

### Annotated Code Examples and Expected Outputs

#### Example: Injecting `ILogger<T>`

```csharp
public class PaymentService
{
    private readonly ILogger<PaymentService> _logger;
    public PaymentService(ILogger<PaymentService> logger) => _logger = logger;

    public void Process(decimal amount)
    {
        _logger.LogInformation("Processing payment of {Amount}", amount);
    }
}
```

**Expected Output (console)**
```
info: MyApp.PaymentService[0]
      Processing payment of 99.99
```

**Why this output occurs**
The category is `MyApp.PaymentService`; the amount is captured as a structured property.

---

### Real-World Cases

- **Every service class** — Inject `ILogger<T>` where logging is needed.
- **Controllers** — `ILogger<HomeController>`.
- **Middleware** — `ILogger<MyMiddleware>`.

---

### References

- `ILogger<T>` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger-1
- Logging in C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/

---

## 3. Log Levels

### Definitions

**Core Definition**
Log levels are **severity categories** that determine the importance and verbosity of log messages.

**Technical Definition**
The `LogLevel` enum defines six emitted levels: **Trace (0)**, **Debug (1)**, **Information (2)**, **Warning (3)**, **Error (4)**, **Critical (5)**, plus **None (6)** for filtering only . Each level has a convenience extension method (`LogTrace`, `LogDebug`, `LogInformation`, `LogWarning`, `LogError`, `LogCritical`). Filters are configured per provider and per category.

**Beginner-Friendly Explanation**
Log levels are **how serious a message is**. Trace is the chattiest, Critical is the most urgent. You set a minimum level, and messages below it are ignored.

---

### Level Reference Table

| Level | Value | Method | Suggested Usage |
|---|---|---|---|
| **Trace** | 0 | `LogTrace` | Most detailed messages; may contain sensitive data; disabled by default; not for production  |
| **Debug** | 1 | `LogDebug` | Debugging and development; high volume; use with caution in production  |
| **Information** | 2 | `LogInformation` | General flow; may have long-term value  |
| **Warning** | 3 | `LogWarning` | Abnormal or unexpected events; typically not app-failing  |
| **Error** | 4 | `LogError` | Errors and exceptions that cannot be handled; failure in the current operation  |
| **Critical** | 5 | `LogCritical` | Failures requiring immediate attention (data loss, out of disk)  |
| **None** | 6 | — | Filtering value only; no messages written  |

---

### Purposes

- To **control verbosity** without changing code.
- To **prioritize** important messages.
- To **filter** logs by category and provider.
- To **align logging** with operational needs (development vs. production) .

---

### Syntax Rules and Structure

```csharp
_logger.LogTrace("Trace detail");
_logger.LogDebug("Debug detail");
_logger.LogInformation("Info detail");
_logger.LogWarning("Warning detail");
_logger.LogError(ex, "Error detail");
_logger.LogCritical("Critical detail");
```

**Component breakdown**

- `Log{Level}` — Convenience extension methods.
- `LogLevel` — Enum value passed to the core `Log` method.

#### Syntax Rules

- Lower values are more verbose; higher values are more severe .
- Filters compare the message level against the configured minimum.
- `None` cannot be emitted; it is only for filtering .

#### Constraints and Limitations

- `Trace` and `Debug` can produce **high volume** and may contain sensitive data .
- Logging at the wrong level (e.g., `Information` for everything) creates noise .

---

### Annotated Code Examples and Expected Outputs

#### Example: Level Filtering

```csharp
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "MyApp.Services": "Information"
    }
  }
}
```

**Expected Output (console)**

- `LogInformation` from `MyApp.Services` → **written**.
- `LogDebug` from `MyApp.Services` → **suppressed** (below `Information`).
- `LogWarning` from any class → **written** (default `Warning`).

**Why this output occurs**
The `Default` minimum is `Warning`; the `MyApp.Services` category overrides it to `Information` .

---

### Real-World Cases

- **Production** — `Warning` or `Error` minimum to reduce volume .
- **Development** — `Debug` or `Trace` for detailed diagnostics .
- **Security auditing** — `Warning` or `Error` for failures; `Information` for access events.

---

### References

- Log levels — Logging in C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/#log-level
- `LogLevel` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.loglevel

---

## 4. Log Providers

### Definitions

**Core Definition**
A log provider is a **destination** that receives log messages from the logging framework and writes them to a specific medium (console, file, event log, cloud service).

**Technical Definition**
A provider implements `ILoggerProvider` and is registered via `ILoggerFactory.AddProvider` or builder extension methods (`AddConsole`, `AddDebug`, `AddEventSourceLogger`, `AddEventLog`, `AddApplicationInsights`). Each provider has its own alias (`Console`, `Debug`, `EventSource`, `EventLog`, `AzureAppServicesFile`, `AzureAppServicesBlob`, `ApplicationInsights`) usable in `appsettings.json` . Filters are configured per provider under `Logging.{ProviderName}.LogLevel` .

**Beginner-Friendly Explanation**
A provider is **where the logs go**. The Console provider writes to the terminal; the Debug provider writes to the debugger; Application Insights sends logs to Azure.

---

### Built-in Providers

| Provider | Alias | Destination |
|---|---|---|
| **Console** | `Console` | Standard output  |
| **Debug** | `Debug` | `System.Diagnostics.Debug`  |
| **EventSource** | `EventSource` | Cross-platform ETW/EventPipe  |
| **EventLog** | `EventLog` | Windows Event Log (Windows only)  |
| **Azure App Services File** | `AzureAppServicesFile` | App Service file system  |
| **Azure App Services Blob** | `AzureAppServicesBlob` | Azure Blob Storage  |
| **Application Insights** | `ApplicationInsights` | Azure Monitor  |

---

### Purposes

- To **route logs** to the right destination.
- To **combine providers** (console + cloud + file).
- To **configure provider-specific settings** (e.g., `IncludeScopes`) .
- To **control costs** by filtering high-volume providers .

---

### Syntax Rules and Structure

```csharp
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddApplicationInsights();
```

```json
{
  "Logging": {
    "LogLevel": { "Default": "Information" },
    "Console": {
      "IncludeScopes": true,
      "LogLevel": { "Microsoft": "Warning" }
    }
  }
}
```

**Component breakdown**

- `ClearProviders()` — Removes default providers.
- `Add{Provider}()` — Registers a provider.
- `Logging.{Provider}.LogLevel` — Provider-specific filtering .

#### Syntax Rules

- Providers are registered at startup; adding them later is not supported.
- `Logging.{Provider}.LogLevel` overrides `Logging.LogLevel` .
- `IncludeScopes` controls whether log scopes are included .

#### Constraints and Limitations

- Some providers are **platform-specific** (EventLog is Windows-only).
- Multiple providers can produce **duplicate output** if configured carelessly.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Providers with Per-Provider Filtering

```csharp
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
```

```json
{
  "Logging": {
    "LogLevel": { "Default": "Information" },
    "Console": { "LogLevel": { "Microsoft": "Error" } },
    "Debug": { "LogLevel": { "Default": "Debug" } }
  }
}
```

**Expected Output**

- `Microsoft.*` messages at `Warning` → **not written to Console**, but **written to Debug** (minimum `Debug`).
- `MyApp.*` messages at `Information` → **written to both**.

**Why this output occurs**
Each provider has its own filter; `Console` suppresses `Microsoft` below `Error`, while `Debug` uses `Debug` minimum .

---

### Real-World Cases

- **Development** — Console + Debug.
- **Production** — Application Insights + Azure App Services File.
- **Windows services** — EventLog.
- **Cross-platform diagnostics** — EventSource.

---

### References

- Logging providers — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/#logging-providers
- Configure logging — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/

---

## 5. Structured Logging

### Definitions

**Core Definition**
Structured logging is the practice of emitting logs with **named properties** (via message templates) rather than pre-formatted strings, enabling machine-readable, queryable log data.

**Technical Definition**
Structured logging uses **message templates** with named placeholders (`"Order {OrderId} failed"`) instead of string interpolation (`$"Order {orderId} failed"`). The logging framework captures the template and the argument values separately, preserving the association between names and values . This enables downstream systems (Serilog, Seq, Elasticsearch) to index and query by property name. String interpolation **bypasses** structured logging because the values are embedded in the string before the logger sees them .

**Beginner-Friendly Explanation**
Structured logging is **“logging data, not just text.”** Instead of a log line that says `"Order 42 failed"`, you emit `"Order {OrderId} failed"` with `42`, and the logging system stores `OrderId=42` as a searchable field.

---

### Purposes

- To make logs **queryable** by property name.
- To preserve **type information** and values.
- To enable **correlation** across services (trace IDs, user IDs).
- To support **dashboards and alerts** based on log properties.
- To reduce **parsing errors** in log analysis tools.

---

### Syntax Rules and Structure

```csharp
// GOOD: structured template
_logger.LogInformation("Order {OrderId} failed for customer {CustomerId}",
    orderId, customerId);

// BAD: string interpolation bypasses structure
_logger.LogInformation($"Order {orderId} failed for customer {customerId}");
```

**Component breakdown**

- `{OrderId}` — Named placeholder captured as a property .
- `orderId` — Argument value passed separately.
- `$"..."` — Interpolation embeds values in the string (anti-pattern) .

#### Syntax Rules

- Use named placeholders in the template, not interpolation .
- Pass exception objects via the `Exception` overload, not `exception.ToString()` .
- Placeholder names should be meaningful and consistent.

#### Constraints and Limitations

- Interpolation loses structure; avoid it in log calls .
- Destructuring complex objects requires framework support (e.g., Serilog `{@Order}`) .

---

### Annotated Code Examples and Expected Outputs

#### Example: Structured vs. Interpolated

```csharp
var orderId = 42;
_logger.LogInformation("Order {OrderId} processed", orderId);
// Output: Order 42 processed (OrderId = 42 captured structurally)

_logger.LogInformation($"Order {orderId} processed");
// Output: Order 42 processed (no property captured)
```

**Expected Output (structured log)**
```json
{
  "Message": "Order 42 processed",
  "OrderId": 42
}
```

**Why this output occurs**
The template placeholder `{OrderId}` is captured as a property; interpolation produces only the formatted string .

---

### Real-World Cases

- **Microservices** — Correlation IDs, tenant IDs, request paths.
- **E-commerce** — Order IDs, customer IDs, payment IDs.
- **Security** — User IDs, IP addresses, action types.

---

### References

- Structured logging — Microsoft.Extensions.Logging guidance — GitHub — https://github.com/dotnet/dotnet/blob/066acc5a2c27f57b7f23003e9192d95e65005d52/src/runtime/.github/instructions/extensions-logging.instructions.md
- Message templates — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/

---

## 6. Serilog

### Definitions

**Core Definition**
Serilog is a **structured logging framework** for .NET that records named template properties, supports object destructuring, and routes logs to **sinks** (destinations).

**Technical Definition**
Serilog replaces or supplements `Microsoft.Extensions.Logging` with its own API (`ILogger` from `Serilog`). It supports **message templates** (`{UserId}`), **destructuring** (`{@Order}`), **enrichment** (`LogContext`), and **sinks** (Console, File, Seq, Elasticsearch, Application Insights). Integration packages (`Serilog.AspNetCore`) register Serilog as the provider for `Microsoft.Extensions.Logging`, so application code can still depend on `ILogger<T>` while Serilog handles formatting and delivery .

**Beginner-Friendly Explanation**
Serilog is **a powerful logging library** that specializes in structured data. It can turn objects into JSON-like properties and send logs to many destinations (files, databases, cloud services) through **sinks**.

---

### Purposes

- To **destructure objects** into structured properties.
- To **enrich logs** with contextual data (`LogContext`).
- To **route logs** to specialized sinks (Seq, Elasticsearch, files).
- To **replace** the built-in logging with a richer pipeline.
- To support **JSON output** natively .

---

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Host.UseSerilog((ctx, cfg) =>
    cfg.ReadFrom.Configuration(ctx.Configuration)
       .Enrich.FromLogContext()
       .WriteTo.Console()
       .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day));
```

```csharp
_logger.LogInformation("Order {@Order} processed", order);   // destructuring
```

**Component breakdown**

- `UseSerilog` — Replaces the default logging provider.
- `ReadFrom.Configuration` — Reads Serilog settings from `appsettings.json`.
- `Enrich.FromLogContext` — Adds contextual properties.
- `WriteTo.Console` / `WriteTo.File` — Sinks .
- `{@Order}` — Destructures the object .

#### Syntax Rules

- Serilog is configured via code or `appsettings.json` (`"Serilog"` section) .
- Destructuring uses `@` prefix.
- Sinks are added via `WriteTo.{Sink}`.

#### Constraints and Limitations

- Serilog has its **own API**; mixing `Serilog.ILogger` and `Microsoft.Extensions.Logging.ILogger` requires care.
- Sink performance varies; test with representative workloads .

---

### Annotated Code Examples and Expected Outputs

#### Example: Destructuring an Object

```csharp
public record Order(int Id, decimal Total);

_logger.LogInformation("Order {@Order} processed", new Order(42, 99.99m));
```

**Expected Output (Serilog JSON)**
```json
{
  "Order": { "Id": 42, "Total": 99.99 }
}
```

**Why this output occurs**
`{@Order}` tells Serilog to destructure the object into its properties .

---

### Real-World Cases

- **Log aggregation** — Seq, Elasticsearch, Datadog sinks.
- **Audit trails** — Enrichment with user and tenant context.
- **Structured JSON** — Native JSON formatting for cloud platforms.

---

### References

- Choose a .NET logging setup — Serilog — https://www.guance.com/learn/articles/dotnet-logging-libraries
- Serilog integration — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/

---

## 7. NLog

### Definitions

**Core Definition**
NLog is a **logging framework** that separates **targets** (destinations) from **rules** (routing and filtering), supporting XML-based configuration and automatic reload.

**Technical Definition**
NLog uses **targets** (File, Console, Database, Network) and **rules** that route log events by logger name and severity. Configuration can be XML (`nlog.config`) or code; `autoReload` refreshes rules when the config file changes . `JsonLayout` preserves event properties for downstream analysis . NLog integrates with `Microsoft.Extensions.Logging` via `NLog.Extensions.Logging`.

**Beginner-Friendly Explanation**
NLog is **a flexible logging library** that lets you define **where** logs go (targets) and **which** logs go where (rules) in XML configuration.

---

### Purposes

- To **route logs** by logger name and severity .
- To **reload configuration** without restarting.
- To **suppress unwanted events** via rules.
- To **support multiple targets** (file, database, network).
- To produce **JSON layouts** for structured analysis .

---

### Syntax Rules and Structure

```xml
<nlog autoReload="true">
  <targets>
    <target name="file" xsi:type="File" fileName="app.log"
            layout="${longdate}|${level}|${logger}|${message}" />
  </targets>
  <rules>
    <logger name="*" minlevel="Info" writeTo="file" />
    <logger name="Microsoft.*" maxlevel="Debug" final="true" />
  </rules>
</nlog>
```

**Component breakdown**

- `<targets>` — Define destinations.
- `<rules>` — Route events to targets by name and level.
- `autoReload` — Reloads XML changes automatically .
- `maxlevel` — Suppresses verbose framework logs.

#### Syntax Rules

- Rules are evaluated in order; `final="true"` stops further evaluation.
- `minlevel` and `maxlevel` control severity filters.
- Targets can use layouts (text, JSON).

#### Constraints and Limitations

- XML configuration is powerful but can be complex.
- `autoReload` is not a universal promise for every external setting .

---

### Annotated Code Examples and Expected Outputs

#### Example: NLog Rule Suppressing Framework Logs

```xml
<rules>
  <logger name="Microsoft.*" maxlevel="Debug" final="true" />
  <logger name="*" minlevel="Info" writeTo="file" />
</rules>
```

**Expected Output**

- `Microsoft.*` at `Debug` or below → **suppressed**.
- `MyApp.*` at `Info` → **written to file**.

**Why this output occurs**
The first rule matches `Microsoft.*` and suppresses events up to `Debug`, then stops evaluation .

---

### Real-World Cases

- **Windows services** — File and EventLog targets.
- **Enterprise apps** — XML configuration with auto-reload.
- **Multi-target logging** — File + database + network simultaneously.

---

### References

- Choose a .NET logging setup — NLog — https://www.guance.com/learn/articles/dotnet-logging-libraries
- NLog documentation — https://nlog-project.org/

---

## 8. Logging Best Practices

### Definitions

**Core Definition**
Logging best practices are the guidelines for emitting logs that are **useful, secure, performant, and maintainable**.

**Technical Definition**
Best practices derive from Microsoft's logging guidance: **use structured logging** with named templates ; **never log sensitive data** (credentials, tokens, PII) at any level ; **use appropriate log levels** and configure levels in configuration, not code ; **use `[LoggerMessage]` source generation** for zero-allocation hot paths ; **log exceptions via the `Exception` parameter**, not `ToString()` ; **use `IsEnabled` guards** for expensive message construction ; and **include correlation IDs** for distributed tracing .

**Beginner-Friendly Explanation**
The best practices are: **log structured data**, **don't log secrets**, **pick the right level**, **use the fast path for hot loops**, and **include a correlation ID**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Structured logging (no interpolation)
_logger.LogInformation("Order {OrderId} failed", orderId);

// 2. No sensitive data
// NEVER: _logger.LogInformation("Password: {Password}", password);

// 3. Appropriate level
_logger.LogWarning("Retry {Attempt} for {OrderId}", attempt, orderId);

// 4. Source-generated logging for hot paths
[LoggerMessage(EventId = 1001, Level = LogLevel.Information,
    Message = "Order {OrderId} processed")]
private static partial void LogOrderProcessed(ILogger logger, int orderId);

// 5. Exception as parameter
_logger.LogError(ex, "Order {OrderId} failed", orderId);

// 6. IsEnabled guard for expensive messages
if (_logger.IsEnabled(LogLevel.Debug))
    _logger.LogDebug("Payload: {Payload}", SerializeLargeObject());

// 7. Correlation IDs
using (_logger.BeginScope("CorrelationId: {CorrelationId}", correlationId))
    _logger.LogInformation("Processing request");
```

**Component breakdown**

- **Structured templates** — `{OrderId}` placeholders .
- **No sensitive data** — Credentials, tokens, PII are forbidden .
- **Right level** — Trace/Debug for dev; Information for flow; Warning/Error for failures .
- **`[LoggerMessage]`** — Zero-allocation source-generated logging .
- **Exception parameter** — Use the `Exception` overload .
- **`IsEnabled`** — Avoid expensive construction when the level is disabled .
- **Scopes** — Correlation IDs propagate across async boundaries .

#### Rules and Constraints

- **Do** use structured logging with named placeholders.
- **Do** configure log levels in `appsettings.json` .
- **Do** use `[LoggerMessage]` for hot paths .
- **Do** include correlation IDs in distributed systems .
- **Do not** log sensitive data at any level .
- **Do not** use string interpolation in log calls .
- **Do not** log everything at `Information` .

---

### Annotated Code Examples and Expected Outputs

#### Example: Best-Practice Logging

```csharp
public partial class OrderService
{
    private readonly ILogger<OrderService> _logger;
    public OrderService(ILogger<OrderService> logger) => _logger = logger;

    [LoggerMessage(EventId = 1001, Level = LogLevel.Information,
        Message = "Order {OrderId} processed for {CustomerId}")]
    private static partial void LogOrderProcessed(
        ILogger logger, int orderId, int customerId);

    public void Process(Order order)
    {
        LogOrderProcessed(_logger, order.Id, order.CustomerId);
    }
}
```

**Expected Output (structured)**
```json
{
  "EventId": 1001,
  "Level": "Information",
  "Message": "Order 42 processed for 7",
  "OrderId": 42,
  "CustomerId": 7
}
```

**Why this pattern is used**
It combines structured logging, source generation (zero allocation), and a meaningful event ID .

---

### Real-World Cases

- **High-throughput APIs** — Source-generated logging for per-request paths.
- **Distributed systems** — Correlation IDs across services .
- **Security-sensitive apps** — Redaction of PII and tokens .

---

### References

- Logging in C# — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/
- Microsoft.Extensions.Logging — Folder-Specific Guidance — GitHub — https://github.com/dotnet/dotnet/blob/066acc5a2c27f57b7f23003e9192d95e65005d52/src/runtime/.github/instructions/extensions-logging.instructions.md
- ASP.NET Core logging best practices — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| `Console.WriteLine` for logging | **Not recommended**; no levels, filtering, or structure. |
| String interpolation in log messages | **Anti-pattern**; bypasses structured logging . |
| Logging sensitive data | **Unsafe**; never log credentials, tokens, PII . |
| `Exception.ToString()` in log templates | **Anti-pattern**; use the `Exception` overload . |
| `[LoggerMessage]` source generation | **Recommended** for hot paths; .NET 6+ . |
| Serilog `UseSerilog` | **Integration package**; version-specific. |
| NLog `autoReload` | **Configuration-dependent**; not a universal promise . |
| `ILogger<T>` category | **Fully qualified type name**; fixed at construction . |

---

## References (Consolidated)

- Logging in C# and .NET — Microsoft Learn — https://learn.microsoft.com/en-us/dotnet/core/extensions/logging/
- Logging in .NET and ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/
- `ILogger<T>` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger-1
- `LogLevel` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.loglevel
- Microsoft.Extensions.Logging — Folder-Specific Guidance — GitHub — https://github.com/dotnet/dotnet/blob/066acc5a2c27f57b7f23003e9192d95e65005d52/src/runtime/.github/instructions/extensions-logging.instructions.md
- Choose a .NET logging setup — https://www.guance.com/learn/articles/dotnet-logging-libraries
- Structured logging guidance — GitHub — https://github.com/dotnet/dotnet/blob/066acc5a2c27f57b7f23003e9192d95e65005d52/src/runtime/.github/instructions/extensions-logging.instructions.md
- Logging providers — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/#logging-providers
- Configure logging — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/
- Serilog integration — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/
- NLog documentation — https://nlog-project.org/

---