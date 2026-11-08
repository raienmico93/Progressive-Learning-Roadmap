# ASP.NET Core MVC Exception Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Exception filters are the fourth stage of the MVC filter pipeline—after action execution and action filters, and before result filters. They implement `IExceptionFilter` (synchronous) or `IAsyncExceptionFilter` (asynchronous), have only an “after” method (`OnException`/`OnExceptionAsync`), and handle **unhandled exceptions** from controller/page-model creation, model binding, action filters, and action methods. They do **not** catch exceptions from resource filters, result filters, or result execution. Official guidance strongly prefers **middleware** for global exception handling; filters are recommended only when error handling must differ per action (e.g., API JSON vs. view HTML). Exception filters cannot turn an exception into a “success”—only action filters can do that. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Exception filters are MVC filters that run after an action (or an action filter) throws an unhandled exception, allowing a common error-handling policy to be applied. |
| **Technical Definition** | Exception filters implement `IExceptionFilter` (synchronous, `OnException(ExceptionContext)`) or `IAsyncExceptionFilter` (asynchronous, `OnExceptionAsync(ExceptionContext)`). They execute **after action filters** and **before result filters**, and they handle unhandled exceptions from controller/page-model creation, model binding, action filters, and action methods. They do **not** catch exceptions from resource filters, result filters, or MVC result execution. To handle an exception, set `context.ExceptionHandled = true` or assign a non-null `context.Result`; this stops exception propagation. |
| **Beginner-Friendly Explanation** | Exception filters are the **safety net around your action**. If your action (or an action filter) throws an exception that nothing else has caught, the exception filter can catch it and return a friendly error response instead of letting the exception crash the request. However, for application-wide error handling, ASP.NET Core’s **exception-handling middleware** is the recommended tool—exception filters are for cases where different actions need different error responses. |

### Key Characteristics

- **“After-only” filters** – Exception filters have **no before event**; they only implement `OnException`/`OnExceptionAsync`.
- **Action-scoped exception handling** – They handle exceptions from controller creation, model binding, action filters, and actions—but **not** from resource filters, result filters, or result execution.
- **Handle by flag or result** – Set `ExceptionHandled = true` or assign `Result` to stop propagation.
- **Cannot convert to success** – Exception filters cannot turn an exception into a “success”; only action filters can do that.
- **Middleware preferred** – Official guidance recommends middleware for exception handling; use filters only where error handling differs by action.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** (authorization → resource → action → exception → result).
- Familiarity with **exception-handling middleware** (`UseExceptionHandler`) and **dependency injection**.

### Related Programming Areas

- **Exception-handling middleware** – The recommended approach for global error handling.
- **Action filters** – The only filter type that can convert an exception into a success.
- **Resource filters** – Exceptions from resource filters are **not** caught by exception filters.
- **Result filters** – Exceptions from result filters are **not** caught by exception filters.
- **Problem Details** – A standardized error response format often produced by exception handling.

### Core Concepts / Features

1. **Exception filters** – The concept, pipeline position, and when to use them.
2. **`IExceptionFilter`** – The synchronous interface.
3. **`IAsyncExceptionFilter`** – The asynchronous interface.
4. **`ExceptionContext`** – The context object passed to exception filters.
5. **Exception filter best practices** – Middleware preference, action-specific handling, and security.

---

## 1. Exception Filters (The Concept and Pipeline Position)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Exception filters are MVC filters that run after an action throws an unhandled exception, enabling a common error-handling policy to be applied. |
| **Technical Definition** | Exception filters implement `IExceptionFilter` or `IAsyncExceptionFilter`. They run **after action filters** and **before result filters**. They handle unhandled exceptions from controller/page-model creation, model binding, action filters, and action methods. They do **not** catch exceptions from resource filters, result filters, or MVC result execution. To handle an exception, set `ExceptionHandled = true` or assign `Result`; this stops propagation. Exception filters cannot convert an exception into a success—only action filters can. |
| **Beginner-Friendly Explanation** | Exception filters are **action-level safety nets**. They catch exceptions that happen during or around the action—but not exceptions from the broader pipeline (resource filters, result filters). They are useful when one action should return JSON errors and another should return an HTML error page. |

### Purposes

- **To** handle unhandled exceptions from controller creation, model binding, action filters, and action methods with a common policy.
- **To** return **action-specific error responses** (e.g., JSON for API actions, HTML for view actions).
- **To** log exceptions with **action-level context** (action name, arguments).
- **To** supplement—not replace—exception-handling middleware for scenarios where per-action error handling differs.
- **To** provide a **filter-level** exception handling mechanism that can be scoped globally, per controller, or per action.

### Syntax Rules and Structure

#### Pipeline Position

```text
Request → Authorization Filters → Resource Filters → Model Binding
→ Action Filters → Action → [Exception Filters] → Result Filters
→ Resource Filters (after) → Response
```

**Component breakdown:**

| Stage | Filter Type | Runs relative to exception filters |
|-------|-------------|------------------------------------|
| 1 | Authorization | Before |
| 2 | Resource | Before (exceptions **not** caught) |
| 3 | Model binding | Before (exceptions **caught**) |
| 4 | Action filters | Before (exceptions **caught**) |
| 5 | Action | Before (exceptions **caught**) |
| 6 | **Exception filters** | After the action (when an exception occurred) |
| 7 | Result filters | After (exceptions **not** caught) |

**Syntax rules:**

- Exception filters have **only an “after” event**—`OnException` or `OnExceptionAsync`.
- They run **after action filters** and **before result filters**.
- Set `ExceptionHandled = true` or assign `Result` to handle the exception.
- They **do not catch** exceptions from resource filters, result filters, or result execution.

**Constraints and limitations:**

- **Middleware is preferred** for global exception handling; exception filters are less flexible.
- **Cannot convert to success**—exception filters cannot turn an exception into a success result; only action filters can.
- **Exceptions from resource/result filters** are outside their scope.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ApiExceptionFilter : IExceptionFilter
{
    private readonly ILogger<ApiExceptionFilter> _logger;

    public ApiExceptionFilter(ILogger<ApiExceptionFilter> logger)
        => _logger = logger;

    public void OnException(ExceptionContext context)
    {
        // 1. Log the exception with action context
        _logger.LogError(context.Exception,
            "Unhandled exception in {Action}",
            context.ActionDescriptor.DisplayName);

        // 2. Handle the exception by assigning a result
        context.Result = new ObjectResult(new
        {
            Error = "An unexpected error occurred.",
            TraceId = context.HttpContext.TraceIdentifier
        })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };

        // 3. Mark as handled to stop propagation
        context.ExceptionHandled = true;
    }
}

// Usage: API controller
[TypeFilter(typeof(ApiExceptionFilter))]
public class ApiController : ControllerBase
{
    public IActionResult Fail() => throw new InvalidOperationException("Boom");
}
```

**Step-by-step explanation:**

1. The filter implements `IExceptionFilter` and injects `ILogger<T>`.
2. `OnException` runs after the action throws an unhandled exception.
3. The exception is logged with the action’s display name.
4. `context.Result` is assigned a structured JSON error response (500).
5. `context.ExceptionHandled = true` stops the exception from propagating further.

**Expected Output:** A request to `Fail()` returns HTTP 500 with a JSON body containing a generic error message and trace ID—the exception does not crash the request.

**Why this output?** The exception filter catches the unhandled exception and replaces it with a controlled result. Assigning `Result` and setting `ExceptionHandled` prevents the exception from reaching middleware-level handling.

### Real-World Cases

- **API endpoints:** Return JSON error payloads for API actions while view actions return HTML error pages.
- **Action-specific logging:** Log exceptions with the action name and arguments.
- **Partial error handling:** Handle known exceptions in specific controllers while letting unknown exceptions propagate to middleware.

### References

- Filters in ASP.NET Core (Exception filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling

---

## 2. `IExceptionFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IExceptionFilter` is the synchronous interface for implementing exception filters in ASP.NET Core MVC. |
| **Technical Definition** | `IExceptionFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `void OnException(ExceptionContext context)`. It extends `IFilterMetadata`. The `ExceptionContext` exposes the `Exception`, a settable `ExceptionHandled` flag, and a settable `Result` property. Setting `ExceptionHandled = true` or assigning `Result` handles the exception and stops propagation. |
| **Beginner-Friendly Explanation** | `IExceptionFilter` is the **synchronous contract** for exception filters. You implement one method—`OnException`—and inside it you decide how to respond to the unhandled exception. If you set the result or mark it handled, the exception is contained; otherwise, it propagates. |

### Purposes

- **To** provide a **synchronous extension point** for handling action-level exceptions.
- **To** allow setting a custom error result (`Result`) for known exception types.
- **To** enable logging and diagnostics with action context.
- **To** support **action-specific** exception handling policies (e.g., different controllers return different error formats).
- **To** complement middleware-level exception handling for cases where per-action differentiation is needed.

### Syntax Rules and Structure

```csharp
public interface IExceptionFilter : IFilterMetadata
{
    void OnException(ExceptionContext context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `OnException` | The single method; receives `ExceptionContext`. |
| `ExceptionContext` | Exposes `Exception`, `ExceptionHandled`, `Result`, and inherited context. |
| `ExceptionHandled` | Set to `true` to stop propagation. |
| `Result` | Assign to provide a response. |

**Syntax rules:**

- Implement `OnException`; there is **no** “before” method.
- Set `ExceptionHandled = true` **or** assign `Result` to handle the exception.
- Leave both untouched to let the exception propagate.
- The filter must be public and implement `IExceptionFilter`.

**Constraints and limitations:**

- **Synchronous only**—use `IAsyncExceptionFilter` for asynchronous work.
- **Does not catch exceptions** from resource filters, result filters, or result execution.
- **Cannot convert to success**—only action filters can do that.

### Annotated Complete Step-by-Step Code Example

```csharp
public class DevelopmentExceptionFilter : IExceptionFilter
{
    private readonly IHostEnvironment _env;

    public DevelopmentExceptionFilter(IHostEnvironment env) => _env = env;

    public void OnException(ExceptionContext context)
    {
        // Only show details in Development
        if (!_env.IsDevelopment())
        {
            return;   // Let middleware handle it
        }

        context.Result = new ContentResult
        {
            Content = context.Exception.ToString()
        };
        context.ExceptionHandled = true;
    }
}

// Usage
[TypeFilter(typeof(DevelopmentExceptionFilter))]
public class DiagnosticsController : Controller
{
    public IActionResult Boom() => throw new Exception("Test exception");
}
```

**Step-by-step explanation:**

1. The filter injects `IHostEnvironment` to check the environment.
2. `OnException` runs when the action throws.
3. In non-development environments, the filter returns without handling—middleware handles the exception.
4. In Development, the exception details are returned as `ContentResult` and `ExceptionHandled` is set to `true`.

**Expected Output:** In Development, `Boom()` returns the exception’s string representation. In Production, the exception is not handled by this filter and propagates to middleware.

**Why this output?** The filter conditionally handles the exception based on the environment. Not setting `ExceptionHandled` lets the exception propagate.

### Real-World Cases

- **Development diagnostics:** Show detailed exception information only in Development.
- **Known business exceptions:** Return a specific error response for a domain exception.
- **Controller-level error policies:** Different controllers return different error formats.

### References

- `IExceptionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iexceptionfilter
- Exception filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters

---

## 3. `IAsyncExceptionFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAsyncExceptionFilter` is the asynchronous interface for implementing exception filters that perform I/O-bound work. |
| **Technical Definition** | `IAsyncExceptionFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `Task OnExceptionAsync(ExceptionContext context)`. It extends `IFilterMetadata`. Unlike the synchronous interface, it can `await` operations such as logging to a database, calling an external error-tracking service, or reading configuration asynchronously. The same handling rules apply: set `ExceptionHandled = true` or assign `Result` to stop propagation. |
| **Beginner-Friendly Explanation** | `IAsyncExceptionFilter` is the **asynchronous version** of the exception filter contract. Use it when handling the exception requires `await`—for example, logging to a database or sending the error to an external monitoring service—without blocking threads. |

### Purposes

- **To** support **asynchronous exception handling** (database logging, external error reporting, async configuration reads).
- **To** avoid blocking threads when handling exceptions that require I/O.
- **To** integrate with asynchronous logging and telemetry services.
- **To** provide a **non-blocking** alternative to `IExceptionFilter` for modern asynchronous applications.

### Syntax Rules and Structure

```csharp
public interface IAsyncExceptionFilter : IFilterMetadata
{
    Task OnExceptionAsync(ExceptionContext context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `OnExceptionAsync` | The single asynchronous method. |
| `ExceptionContext` | Same context as the synchronous interface. |
| `Task` | Return type; supports `async`/`await`. |

**Syntax rules:**

- Implement `OnExceptionAsync`; there is **no** “before” method.
- Use `async Task` for the implementation.
- Set `ExceptionHandled = true` **or** assign `Result` to handle the exception.
- Leave both untouched to let the exception propagate.

**Constraints and limitations:**

- **Same exception scope** as the synchronous version—does not catch resource/result filter exceptions.
- **Cannot convert to success**—only action filters can.
- **Middleware is still preferred** for global exception handling.

### Annotated Complete Step-by-Step Code Example

```csharp
public class AsyncAuditExceptionFilter : IAsyncExceptionFilter
{
    private readonly IAuditService _audit;

    public AsyncAuditExceptionFilter(IAuditService audit) => _audit = audit;

    public async Task OnExceptionAsync(ExceptionContext context)
    {
        // Await asynchronous audit logging
        await _audit.LogExceptionAsync(
            context.HttpContext,
            context.Exception);

        context.Result = new StatusCodeResult(
            StatusCodes.Status500InternalServerError);

        context.ExceptionHandled = true;
    }
}

// Usage
[TypeFilter(typeof(AsyncAuditExceptionFilter))]
public class OrdersController : Controller
{
    public IActionResult Create() => throw new InvalidOperationException();
}
```

**Step-by-step explanation:**

1. The filter implements `IAsyncExceptionFilter` and injects `IAuditService`.
2. `OnExceptionAsync` awaits `LogExceptionAsync` to record the exception asynchronously.
3. A 500 status code result is assigned.
4. `ExceptionHandled` is set to `true` to stop propagation.

**Expected Output:** The exception is logged asynchronously and the client receives a 500 response without the exception propagating to middleware.

**Why this output?** The async filter performs I/O without blocking and then handles the exception by assigning a result and setting the handled flag.

### Real-World Cases

- **Database audit logging:** Log exceptions to a database asynchronously.
- **External error reporting:** Send exceptions to Sentry, Application Insights, or a custom error service.
- **Async telemetry:** Record exception metrics without blocking the request thread.

### References

- `IAsyncExceptionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncexceptionfilter
- Exception filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters

---

## 4. `ExceptionContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ExceptionContext` is the context object passed to exception filters, exposing the unhandled exception, a handled flag, and a settable result. |
| **Technical Definition** | `ExceptionContext` (in `Microsoft.AspNetCore.Mvc.Filters`) inherits from `FilterContext` and adds `Exception` (the unhandled exception), `ExceptionHandled` (a boolean flag), and `Result` (a settable `IActionResult`). It also inherits `HttpContext`, `ActionDescriptor`, `ModelState`, `RouteData`, and `Filters` from `FilterContext`. Setting `ExceptionHandled = true` or assigning `Result` handles the exception and stops propagation. |
| **Beginner-Friendly Explanation** | `ExceptionContext` is the **“exception envelope”**. It gives you the exception that was thrown, a switch to mark it as handled, and a slot to put the response you want to send back. |

### Purposes

- **To** expose the **unhandled exception** for logging, inspection, and transformation.
- **To** provide a **`ExceptionHandled` flag** that stops exception propagation when set to `true`.
- **To** allow assigning a **`Result`** (an `IActionResult`) to produce a custom error response.
- **To** give access to **request context** (`HttpContext`, `ActionDescriptor`, `RouteData`) for contextual error handling.
- **To** enable **action-specific** error responses based on the action or controller.

### Syntax Rules and Structure

```csharp
public class ExceptionContext : FilterContext
{
    public Exception? Exception { get; set; }
    public bool ExceptionHandled { get; set; }
    public IActionResult? Result { get; set; }
    // Inherited: HttpContext, ActionDescriptor, ModelState, RouteData, Filters
}
```

**Component breakdown:**

| Property | Type | Purpose |
|----------|------|---------|
| `Exception` | `Exception?` | The unhandled exception. |
| `ExceptionHandled` | `bool` | Set to `true` to stop propagation. |
| `Result` | `IActionResult?` | Assign to provide a response. |
| `HttpContext` | `HttpContext` | The request context. |
| `ActionDescriptor` | `ActionDescriptor` | Metadata about the action. |

**Syntax rules:**

- **Setting `ExceptionHandled = true`** or **assigning `Result`** handles the exception.
- Setting `Exception` to `null` also effectively handles the exception (used in action filters, not typically in exception filters).
- Both `ExceptionHandled` and `Result` can be set together (assign the result and mark handled).

**Constraints and limitations:**

- **Cannot convert to success**—the result is still an error response; only action filters can turn an exception into a success.
- **Exceptions from resource/result filters** are not surfaced here.

### Annotated Complete Step-by-Step Code Example

```csharp
public class DomainExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        if (context.Exception is DomainException domainEx)
        {
            context.Result = new ObjectResult(new
            {
                Error = domainEx.Message,
                Code = domainEx.Code
            })
            {
                StatusCode = StatusCodes.Status400BadRequest
            };
            context.ExceptionHandled = true;
        }
        else
        {
            // Unknown exception — let middleware handle it
            // Do not set ExceptionHandled or Result
        }
    }
}
```

**Step-by-step explanation:**

1. The filter checks whether the exception is a known `DomainException`.
2. If it is, a structured 400 response is assigned to `Result`.
3. `ExceptionHandled` is set to `true` to stop propagation.
4. If the exception is not a `DomainException`, the filter does nothing—the exception propagates to middleware.

**Expected Output:** A `DomainException` returns a 400 JSON response with the domain error code; any other exception propagates to middleware-level handling.

**Why this output?** The filter selectively handles known exception types and leaves unknown exceptions for the broader middleware handler.

### Real-World Cases

- **Domain exceptions:** Return 400 with business error codes for known domain failures.
- **Validation exceptions:** Return 422 with validation details.
- **Not-found exceptions:** Return 404 with a resource identifier.

### References

- `ExceptionContext` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext
- `ExceptionContext.Exception` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext.exception
- `ExceptionContext.ExceptionHandled` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext.exceptionhandled

---

## 5. Exception Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Exception filter best practices are guidelines for using exception filters safely, securely, and in alignment with ASP.NET Core’s recommended error-handling architecture. |
| **Technical Definition** | Best practices encompass **preferring exception-handling middleware** for global error handling, using exception filters only where error handling must differ by action, **not leaking sensitive exception details** in production, logging with sufficient context, understanding the scope limitations (resource/result filter exceptions are not caught), and using `IAsyncExceptionFilter` for I/O-bound handling. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for exception filters—they help you avoid duplicating middleware, leaking stack traces to users, and expecting filters to catch exceptions they cannot see. |

### Purposes

- **To** use the right tool for the job (middleware for global handling, filters for action-specific handling).
- **To** prevent **information disclosure** by not returning exception details in production.
- **To** ensure exceptions are logged with enough context for diagnosis.
- **To** avoid blocking threads by using async filters for I/O.
- **To** understand the scope boundaries of exception filters (what they can and cannot catch).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Global vs. action-specific** | Use **middleware** (`UseExceptionHandler`) for global exception handling. | Rely solely on exception filters for application-wide error handling. |
| **Action-specific differentiation** | Use exception filters when API actions return JSON and view actions return HTML. | Use filters when a single global policy is sufficient. |
| **Information disclosure** | Return generic error messages in production; log details server-side. | Return `exception.ToString()` or stack traces to production clients. |
| **Logging context** | Log with `HttpContext.TraceIdentifier`, action name, and user info. | Log only the exception message without context. |
| **Async I/O** | Use `IAsyncExceptionFilter` for database or external service logging. | Use `IExceptionFilter` for I/O (blocks threads). |
| **Scope awareness** | Remember filters do **not** catch resource/result filter exceptions. | Expect filters to handle exceptions from the entire pipeline. |
| **Exception conversion** | Let action filters convert exceptions to success if needed. | Try to convert exceptions to success in exception filters—it cannot be done. |
| **Handling flag** | Set `ExceptionHandled = true` **or** assign `Result` to stop propagation. | Set neither and expect the exception to be handled. |

### Annotated Code Example (Best-Practice Exception Filter Setup)

```csharp
// ✅ Best practice: Middleware for global handling
app.UseExceptionHandler("/error");

// ✅ Exception filter only for action-specific differentiation
public class ApiExceptionFilter : IAsyncExceptionFilter
{
    private readonly ILogger<ApiExceptionFilter> _logger;
    private readonly IHostEnvironment _env;

    public ApiExceptionFilter(
        ILogger<ApiExceptionFilter> logger,
        IHostEnvironment env)
    {
        _logger = logger;
        _env = env;
    }

    public async Task OnExceptionAsync(ExceptionContext context)
    {
        _logger.LogError(context.Exception,
            "Unhandled exception in {Action} (TraceId: {TraceId})",
            context.ActionDescriptor.DisplayName,
            context.HttpContext.TraceIdentifier);

        // Sanitize error details in production
        var message = _env.IsDevelopment()
            ? context.Exception.ToString()
            : "An unexpected error occurred.";

        context.Result = new ObjectResult(new
        {
            Error = message,
            TraceId = context.HttpContext.TraceIdentifier
        })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };

        context.ExceptionHandled = true;
        await Task.CompletedTask;
    }
}

// Apply only to API controllers/actions
[TypeFilter(typeof(ApiExceptionFilter))]
public class ApiController : ControllerBase { /* ... */ }
```

**Why this follows best practices:**

- **Middleware handles global exceptions**; the filter is applied only where API-specific JSON error responses are needed.
- **Exception details are sanitized** in non-development environments—no stack traces leak to clients.
- **Logging includes the trace ID** and action name for diagnostics.
- **`IAsyncExceptionFilter`** is used for consistency with async logging patterns.
- **`ExceptionHandled = true`** and **`Result`** are both set to properly handle and respond.

### Real-World Cases

- **Mixed MVC + API applications:** Middleware handles global errors; an exception filter on API controllers returns JSON while view controllers use the middleware’s HTML error page.
- **Domain-driven design:** A filter handles known domain exceptions with appropriate status codes, while unknown exceptions propagate to middleware.
- **High-security environments:** Filters sanitize error responses and log details only server-side, preventing information disclosure.

### References

- Filters in ASP.NET Core (Exception filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- Exception filters: middleware preferred – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Security cards: sanitize error details – https://github.com/reware-labs/securitycards

---

## Summary Table: Exception Filters at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|------------------|-------------|
| **Exception filters** | Handle action-level unhandled exceptions | `IExceptionFilter` / `IAsyncExceptionFilter` | When error handling differs by action |
| **`IExceptionFilter`** | Synchronous exception filter | `OnException(ExceptionContext)` | Simple synchronous exception handling |
| **`IAsyncExceptionFilter`** | Asynchronous exception filter | `OnExceptionAsync(ExceptionContext)` | Async logging or external error reporting |
| **`ExceptionContext`** | Context for exception filters | `Exception`, `ExceptionHandled`, `Result` | Inspecting and handling exceptions |
| **Best practices** | Middleware first, sanitize, log, async for I/O | `UseExceptionHandler`, generic messages, trace IDs | Every exception filter implementation |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Exception filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- `IExceptionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iexceptionfilter
- `IAsyncExceptionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncexceptionfilter
- `ExceptionContext` class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext
- `ExceptionContext.Exception` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext.exception
- `ExceptionContext.ExceptionHandled` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.exceptioncontext.exceptionhandled
- Security cards: sanitize error details – https://github.com/reware-labs/securitycards