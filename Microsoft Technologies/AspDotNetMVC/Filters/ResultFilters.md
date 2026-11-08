# ASP.NET Core MVC Result Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Result filters are the fifth stage of the MVC filter pipeline—after action filters and exception filters, and around the execution of an `IActionResult`. They implement `IResultFilter` (synchronous) or `IAsyncResultFilter` (asynchronous), and a special variant (`IAlwaysRunResultFilter`) runs even when the pipeline is short-circuited by authorization or resource filters. Result filters can inspect or replace the result, add headers, and cancel result execution. They run **before** the resource filters’ `OnResourceExecuted` code. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Result filters are MVC filters that run immediately before and after the execution of an `IActionResult`, allowing inspection or modification of the result and its response. |
| **Technical Definition** | Result filters implement `IResultFilter` (`OnResultExecuting`, `OnResultExecuted`) or `IAsyncResultFilter` (`OnResultExecutionAsync` with a `ResultExecutionDelegate next`). They run **after action filters** and **after exception filters** (when no exception is handled), and they are the innermost filters before the response is written to the client. `ResultExecutingContext` exposes `Result` (mutable) and `Cancel` (set to `true` to skip result execution). `ResultExecutedContext` exposes `Result`, `Canceled`, `Exception`, and `ExceptionHandled`. The special interface `IAlwaysRunResultFilter` ensures the filter runs even when the pipeline is short-circuited by an authorization or resource filter. |
| **Beginner-Friendly Explanation** | Result filters are the **final checkpoint before the response goes out**. They can look at or change what the action returned (the `IActionResult`), add response headers, or even cancel the result entirely. They are the last filters to run before the response is sent to the client. |

### Key Characteristics

- **“Around-result” filters** – `OnResultExecuting` runs before the result executes; `OnResultExecuted` runs after.
- **Innermost filters** – They run after action filters and exception filters, immediately before the response is written.
- **Result-aware** – They can inspect, replace, or cancel `context.Result`.
- **Three interfaces** – `IResultFilter` (sync), `IAsyncResultFilter` (async), and `IAlwaysRunResultFilter` (runs even on short-circuit).
- **Response-body limits** – `OnResultExecuted` runs after the response has started, so the body cannot be modified there (only headers already sent).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** (authorization → resource → action → exception → result).
- Familiarity with **`IActionResult`** and **response writing** in MVC.

### Related Programming Areas

- **Action filters** – Run before result filters, around the action method.
- **Exception filters** – Run before result filters, handling exceptions from the action.
- **Resource filters** – Their `OnResourceExecuted` code runs **after** result filters.
- **`IActionResult` execution** – The thing result filters wrap.
- **Response headers and caching** – Common targets of result filters.

### Core Concepts / Features

1. **Result filters** – The concept, pipeline position, and use cases.
2. **`IResultFilter`** – The synchronous interface.
3. **`IAsyncResultFilter`** – The asynchronous interface.
4. **`ResultExecutingContext`** – The context for the “before” stage.
5. **`ResultExecutedContext`** – The context for the “after” stage.
6. **Result filter best practices** – Async, cancellation, headers, and short-circuit awareness.

---

## 1. Result Filters (The Concept and Pipeline Position)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Result filters are MVC filters that execute immediately before and after an `IActionResult` is executed, allowing inspection, modification, or cancellation of the result. |
| **Technical Definition** | Result filters implement `IResultFilter` or `IAsyncResultFilter`. They run **after action filters** and **after exception filters** (when no exception is handled), and **before** the resource filters’ `OnResourceExecuted` code. `OnResultExecuting` runs before the result’s `ExecuteResultAsync` is called; `OnResultExecuted` runs after it completes. The `ResultExecutingContext` exposes a settable `Result` and a `Cancel` flag. The special `IAlwaysRunResultFilter` interface ensures execution even when the pipeline is short-circuited by an authorization or resource filter. |
| **Beginner-Friendly Explanation** | Result filters are the **last stop before the response leaves your app**. They wrap the moment when MVC turns your `IActionResult` into an actual HTTP response. You can inspect the result, replace it, add headers, or even cancel it—but once the response has started, you cannot change the body. |

### Purposes

- **To** inspect or modify the `IActionResult` **before** it executes (e.g., wrap JSON responses in an envelope).
- **To** **add response headers** (caching headers, security headers, custom metadata) consistently across actions.
- **To** **cancel result execution** when a condition is not met (e.g., client disconnected, precondition failed).
- **To** **log or measure** result execution time and status codes.
- **To** run logic **even on short-circuited requests** using `IAlwaysRunResultFilter` (e.g., ensuring security headers are added to 401/403 responses).

### Syntax Rules and Structure

#### Pipeline Position

```text
Request → Authorization Filters → Resource Filters (before) → Model Binding
→ Action Filters (before) → Action → Action Filters (after) → Exception Filters
→ [Result Filters (before)] → Result Execution → [Result Filters (after)]
→ Resource Filters (after) → Response
```

**Component breakdown:**

| Stage | Filter Type | Runs relative to result filters |
|-------|-------------|----------------------------------|
| 1 | Authorization | Before (first) |
| 2 | Resource (before) | Before (wraps the whole pipeline) |
| 3 | Model binding | Before |
| 4 | Action filters | Before |
| 5 | Action | Before |
| 6 | Exception filters | Before (only when an exception occurs) |
| 7 | **Result (before)** | Immediately before result execution |
| 8 | Result execution | Inside result filters |
| 9 | **Result (after)** | Immediately after result execution |
| 10 | Resource (after) | After result filters |

**Syntax rules:**

- `OnResultExecuting` runs **after action filters and exception filters** and **before** the result’s `ExecuteResultAsync`.
- `OnResultExecuted` runs **after** the result has completed and **before** resource filters’ `OnResourceExecuted`.
- Setting `context.Result` in `OnResultExecuting` **replaces** the result that will be executed.
- Setting `context.Cancel = true` in `OnResultExecuting` **prevents** the result from executing (the response may be empty).
- **Response body cannot be modified** in `OnResultExecuted` because the response has started.

**Constraints and limitations:**

- **Short-circuited pipelines skip result filters** unless the filter implements `IAlwaysRunResultFilter`. If an authorization or resource filter short-circuits, ordinary result filters **do not run**.
- **Exception filter handled results** – When an exception filter handles an exception by setting `Result` and `ExceptionHandled = true`, result filters **do run** against the handled result.
- **Response body modification** after `OnResultExecuting` is limited to what the result itself writes; `OnResultExecuted` cannot modify the body.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ResultLoggingFilter : IResultFilter
{
    private readonly ILogger<ResultLoggingFilter> _logger;

    public ResultLoggingFilter(ILogger<ResultLoggingFilter> logger)
        => _logger = logger;

    public void OnResultExecuting(ResultExecutingContext context)
    {
        _logger.LogInformation("Executing result {Result} for {Action}",
            context.Result.GetType().Name,
            context.ActionDescriptor.DisplayName);
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        _logger.LogInformation("Executed result {Result}; canceled: {Canceled}",
            context.Result.GetType().Name,
            context.Canceled);
    }
}

// Usage
[ServiceFilter(typeof(ResultLoggingFilter))]
public IActionResult Index() => View();
```

**Step-by-step explanation:**

1. The filter implements `IResultFilter` and injects `ILogger<T>`.
2. `OnResultExecuting` logs the result type before execution.
3. The result (`ViewResult`) executes.
4. `OnResultExecuted` logs that execution completed and whether it was canceled.

**Expected Output:** Two log entries per request: “Executing result ViewResult…” and “Executed result ViewResult; canceled: False”.

**Why this output?** `IResultFilter` wraps the execution of the `IActionResult`, logging before and after.

### Real-World Cases

- **Response envelope:** Wrap JSON API responses in a standard `{ success, data }` structure.
- **Security headers:** Add `X-Content-Type-Options`, `X-Frame-Options`, etc. to every response.
- **Cache headers:** Set `Cache-Control` based on the result type.
- **Result timing:** Measure how long it takes to render a view or serialize JSON.

### References

- Filters in ASP.NET Core (Result filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters
- Result filters in ASP.NET Core MVC – https://www.milanjovanovic.tech/blog/how-to-use-action-filters-in-aspnetcore

---

## 2. `IResultFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IResultFilter` is the synchronous interface for implementing result filters in ASP.NET Core MVC. |
| **Technical Definition** | `IResultFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares two methods: `void OnResultExecuting(ResultExecutingContext context)` and `void OnResultExecuted(ResultExecutedContext context)`. It extends `IFilterMetadata`. `OnResultExecuting` runs before the result executes; `OnResultExecuted` runs after. The synchronous interface is suitable for CPU-bound logic (header addition, result inspection) but not asynchronous I/O. |
| **Beginner-Friendly Explanation** | `IResultFilter` is the **synchronous contract** for result filters. You implement two methods—one that runs before the result is executed and one that runs after. It is fine for quick, synchronous work like adding a header, but for anything involving I/O, use `IAsyncResultFilter`. |

### Purposes

- **To** provide a **synchronous extension point** around result execution.
- **To** allow **result inspection and replacement** before execution.
- **To** enable **header manipulation** and **cancellation** of result execution.
- **To** support **post-result logging** and diagnostics.
- **To** serve as the simplest, most direct way to wrap result execution.

### Syntax Rules and Structure

```csharp
public interface IResultFilter : IFilterMetadata
{
    void OnResultExecuting(ResultExecutingContext context);
    void OnResultExecuted(ResultExecutedContext context);
}
```

**Component breakdown:**

| Method | Runs | Context |
|--------|------|---------|
| `OnResultExecuting` | Before result execution | `ResultExecutingContext` (set `Result` or `Cancel`) |
| `OnResultExecuted` | After result execution | `ResultExecutedContext` |

**Syntax rules:**

- Both methods must be implemented.
- `OnResultExecuting` runs **after action filters** and **before** the result’s `ExecuteResultAsync`.
- `OnResultExecuted` runs **after** the result has executed and **before** resource filters’ `OnResourceExecuted`.
- The filter must be public and implement `IResultFilter`.

**Constraints and limitations:**

- **Synchronous only**—use `IAsyncResultFilter` for asynchronous work.
- **Does not run on short-circuited pipelines** unless it also implements `IAlwaysRunResultFilter`.
- **Cannot modify the response body** in `OnResultExecuted` (the response has started).

### Annotated Complete Step-by-Step Code Example

```csharp
public class SecurityHeadersResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        // Add headers before the result executes
        context.HttpContext.Response.Headers.Append(
            "X-Content-Type-Options", "nosniff");
        context.HttpContext.Response.Headers.Append(
            "X-Frame-Options", "DENY");
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // No post-result logic
    }
}

// Usage
[ServiceFilter(typeof(SecurityHeadersResultFilter))]
public IActionResult Index() => View();
```

**Step-by-step explanation:**

1. The filter implements `IResultFilter`.
2. `OnResultExecuting` adds two security headers to the response.
3. The result executes; the headers are included in the response.
4. `OnResultExecuted` does nothing.

**Expected Output:** Every response from the `Index` action includes `X-Content-Type-Options: nosniff` and `X-Frame-Options: DENY`.

**Why this output?** Headers added in `OnResultExecuting` are set before the result writes the response, so they appear in the final response.

### Real-World Cases

- **Security headers:** Add standard security headers to every response.
- **Cache headers:** Set `Cache-Control` based on the result type (e.g., no-cache for authenticated pages).
- **Result logging:** Log which result type was produced for each action.

### References

- `IResultFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iresultfilter
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters

---

## 3. `IAsyncResultFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAsyncResultFilter` is the asynchronous interface for implementing result filters that perform I/O-bound work around result execution. |
| **Technical Definition** | `IAsyncResultFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `Task OnResultExecutionAsync(ResultExecutingContext context, ResultExecutionDelegate next)`. Code before `await next()` runs before the result executes; code after `await next()` runs after. Setting `context.Cancel = true` before calling `next()` prevents the result from executing. `await next()` returns a `ResultExecutedContext`. |
| **Beginner-Friendly Explanation** | `IAsyncResultFilter` is the **asynchronous version** of the result filter contract. It has one method that receives a `next` delegate: you write “before result” code, `await next()` to execute the result, and then write “after result” code. Use it when you need to `await` something—like writing to a log database or reading a cache—around result execution. |

### Purposes

- **To** perform **asynchronous before/after result logic** without blocking threads.
- **To** support **I/O-bound work** (database logging, external telemetry) around result execution.
- **To** enable **asynchronous cancellation** of result execution when preconditions fail.
- **To** provide a **single-method** interface that naturally expresses the “around” semantics of result filters.
- **To** be the **recommended** result filter interface for modern applications with async dependencies.

### Syntax Rules and Structure

```csharp
public interface IAsyncResultFilter : IFilterMetadata
{
    Task OnResultExecutionAsync(
        ResultExecutingContext context,
        ResultExecutionDelegate next);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context` | `ResultExecutingContext`; set `Result` or `Cancel`. |
| `next` | `ResultExecutionDelegate`; call `await next()` to execute the result. |
| Return | `Task`; supports `async`/`await`. |

**Syntax rules:**

- Code before `await next()` runs **before result execution**.
- Code after `await next()` runs **after result execution**.
- Setting `context.Cancel = true` before `await next()` prevents result execution.
- Setting `context.Result` before `await next()` replaces the result to execute.
- `await next()` returns a `ResultExecutedContext`.

**Constraints and limitations:**

- **`next()` must be called** unless you explicitly cancel (with `context.Cancel = true`). Unlike action filters, result filters do not have a short-circuit rule that forbids calling `next`; cancellation is expressed via `Cancel`.
- **Response body cannot be modified** after `await next()` if the response has started.
- **No automatic short-circuit**—setting `Result` replaces the result but does not skip `next()`.

### Annotated Complete Step-by-Step Code Example

```csharp
public class AsyncResultTimingFilter : IAsyncResultFilter
{
    private readonly ILogger<AsyncResultTimingFilter> _logger;

    public AsyncResultTimingFilter(ILogger<AsyncResultTimingFilter> logger)
        => _logger = logger;

    public async Task OnResultExecutionAsync(
        ResultExecutingContext context,
        ResultExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();

        // Before result execution
        _logger.LogInformation("Executing {Result}",
            context.Result.GetType().Name);

        // Execute the result
        var executed = await next();

        sw.Stop();
        _logger.LogInformation("Result executed in {Elapsed}ms; canceled: {Canceled}",
            sw.ElapsedMilliseconds,
            executed.Canceled);
    }
}

// Usage
[ServiceFilter(typeof(AsyncResultTimingFilter))]
public async Task<IActionResult> Index()
{
    await Task.Delay(10);
    return View();
}
```

**Step-by-step explanation:**

1. The filter implements `IAsyncResultFilter` and injects `ILogger<T>`.
2. `OnResultExecutionAsync` starts a stopwatch and logs the result type.
3. `await next()` executes the result (the `ViewResult` renders the view).
4. After `next()` returns, the duration and cancellation status are logged.

**Expected Output:** Two log entries: “Executing ViewResult” and “Result executed in Xms; canceled: False”.

**Why this output?** The code before `await next()` runs before result execution; the code after runs after.

### Real-World Cases

- **Async result logging:** Write result execution details to a database asynchronously.
- **Result telemetry:** Record result duration and type in Application Insights or OpenTelemetry.
- **Async cache invalidation:** Invalidate cache entries after specific result types execute.

### References

- `IAsyncResultFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters

---

## 4. `ResultExecutingContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ResultExecutingContext` is the context object passed to result filters **before** the `IActionResult` executes, exposing the result and a cancellation flag. |
| **Technical Definition** | `ResultExecutingContext` (in `Microsoft.AspNetCore.Mvc.Filters`) inherits from `FilterContext` and adds `Result` (a settable `IActionResult`) and `Cancel` (a boolean flag). Setting `Cancel = true` prevents the result from executing; setting `Result` replaces the result. It also inherits `HttpContext`, `ActionDescriptor`, `ModelState`, `RouteData`, and `Filters` from `FilterContext`. |
| **Beginner-Friendly Explanation** | `ResultExecutingContext` is the **“before the result” envelope**. It gives you the result that’s about to be executed and two levers: one to **replace** the result and one to **cancel** its execution entirely. |

### Purposes

- **To** expose the `IActionResult` **before** it executes, so filters can inspect or replace it.
- **To** provide a **`Cancel` flag** that prevents the result from executing.
- **To** give access to the **HTTP response** for adding headers before the result writes.
- **To** enable **result transformation** (wrapping JSON, changing status codes) before execution.
- **To** support **conditional cancellation** (e.g., client disconnected, precondition failed).

### Syntax Rules and Structure

```csharp
public class ResultExecutingContext : FilterContext
{
    public IActionResult Result { get; set; }
    public bool Cancel { get; set; }
    // Inherited: HttpContext, ActionDescriptor, ModelState, RouteData, Filters
}
```

**Component breakdown:**

| Property | Type | Purpose |
|----------|------|---------|
| `Result` | `IActionResult` | The result to execute (mutable). |
| `Cancel` | `bool` | Set to `true` to skip execution. |
| `HttpContext` | `HttpContext` | The request context. |
| `ActionDescriptor` | `ActionDescriptor` | Metadata about the action. |

**Syntax rules:**

- Setting `Result` to a new `IActionResult` **replaces** the result that will execute.
- Setting `Cancel = true` **prevents** the result from executing; the response may be empty or left as-is.
- Headers added to `HttpContext.Response` here are included in the response.
- The original `Result` is still accessible if you store it before replacing.

**Constraints and limitations:**

- **Cancellation does not automatically set a status code**—you must set it explicitly if a specific status is desired.
- **Replacing the result** does not skip `next()`; the new result is executed by the pipeline.

### Annotated Complete Step-by-Step Code Example

```csharp
public class EnvelopeResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.Result is ObjectResult objectResult &&
            objectResult.Value is not null)
        {
            // Replace the result with an envelope
            context.Result = new ObjectResult(new
            {
                Success = true,
                Data = objectResult.Value
            })
            {
                StatusCode = objectResult.StatusCode
            };
        }
        else if (context.Result is StatusCodeResult statusResult &&
                 statusResult.StatusCode == 404)
        {
            // Cancel and return a custom 404 body
            context.HttpContext.Response.StatusCode = 404;
            context.Result = new ObjectResult(new
            {
                Success = false,
                Error = "Resource not found"
            })
            {
                StatusCode = 404
            };
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}

// Usage
[ServiceFilter(typeof(EnvelopeResultFilter))]
public IActionResult Get() => Ok(new { Id = 1, Name = "Widget" });
```

**Step-by-step explanation:**

1. The filter implements `IResultFilter`.
2. `OnResultExecuting` checks whether the result is an `ObjectResult` with a value.
3. If so, it **replaces** the result with a wrapped envelope.
4. If the result is a 404 `StatusCodeResult`, it replaces it with a structured 404 error body.

**Expected Output:** A successful action returns `{ "Success": true, "Data": { "Id": 1, "Name": "Widget" } }`. A 404 returns `{ "Success": false, "Error": "Resource not found" }`.

**Why this output?** The filter replaces `context.Result` in `OnResultExecuting`, so the pipeline executes the new result instead of the original.

### Real-World Cases

- **Response envelopes:** Wrap API results in a consistent structure.
- **Status code transformation:** Convert 204 No Content to 200 with an empty object for clients that expect a body.
- **Conditional cancellation:** Cancel result execution if the client disconnected (check `HttpContext.RequestAborted`).

### References

- `ResultExecutingContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.resultexecutingcontext
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters

---

## 5. `ResultExecutedContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ResultExecutedContext` is the context object passed to result filters **after** the `IActionResult` executes, exposing the result, cancellation status, and any exception. |
| **Technical Definition** | `ResultExecutedContext` (in `Microsoft.AspNetCore.Mvc.Filters`) inherits from `FilterContext` and adds `Result` (the executed `IActionResult`), `Canceled` (whether execution was canceled), `Exception` (any unhandled exception from result execution), `ExceptionDispatchInfo`, and `ExceptionHandled`. Modifying `Result` here does not re-execute the result—the response has already started. |
| **Beginner-Friendly Explanation** | `ResultExecutedContext` is the **“after the result” envelope**. It tells you what result was executed, whether it was canceled, and whether an exception occurred. Because the response has already started, you cannot change the body—but you can inspect, log, or react to what happened. |

### Purposes

- **To** expose the **executed result** for logging or inspection.
- **To** report whether the result was **canceled** by an earlier filter.
- **To** expose any **exception** thrown during result execution.
- **To** support **post-result logging** (status code, result type, duration).
- **To** enable **cleanup logic** that runs after the response has been written.

### Syntax Rules and Structure

```csharp
public class ResultExecutedContext : FilterContext
{
    public IActionResult Result { get; }
    public bool Canceled { get; }
    public Exception? Exception { get; }
    public ExceptionDispatchInfo? ExceptionDispatchInfo { get; }
    public bool ExceptionHandled { get; set; }
    // Inherited: HttpContext, ActionDescriptor, ModelState, RouteData, Filters
}
```

**Component breakdown:**

| Property | Type | Purpose |
|----------|------|---------|
| `Result` | `IActionResult` | The result that was executed (read-only). |
| `Canceled` | `bool` | Whether execution was canceled. |
| `Exception` | `Exception?` | Any exception from result execution. |
| `ExceptionHandled` | `bool` | Set to `true` to mark the exception as handled. |

**Syntax rules:**

- `Result` is **read-only** in the executed context—replacing it does not re-execute the result.
- `Canceled` is `true` if an earlier filter set `ResultExecutingContext.Cancel = true`.
- `Exception` is non-null if the result threw during execution.
- **Cannot modify the response body**—the response has already started.

**Constraints and limitations:**

- **Response body modification is not possible** in `OnResultExecuted`—the response has been written.
- **Exception handling here** is limited—result execution exceptions are usually handled at the middleware level.
- **`Result` is read-only**—it is for inspection, not replacement.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ResultAuditFilter : IResultFilter
{
    private readonly ILogger<ResultAuditFilter> _logger;

    public ResultAuditFilter(ILogger<ResultAuditFilter> logger)
        => _logger = logger;

    public void OnResultExecuting(ResultExecutingContext context) { }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        var statusCode = context.HttpContext.Response.StatusCode;
        var resultType = context.Result.GetType().Name;

        _logger.LogInformation(
            "Result {ResultType} executed with status {StatusCode}; canceled: {Canceled}; exception: {HasException}",
            resultType,
            statusCode,
            context.Canceled,
            context.Exception != null);
    }
}

// Usage
[ServiceFilter(typeof(ResultAuditFilter))]
public IActionResult Get() => Ok(new { Id = 1 });
```

**Step-by-step explanation:**

1. The filter implements `IResultFilter`.
2. `OnResultExecuting` does nothing.
3. The result (`ObjectResult`) executes and writes the JSON response.
4. `OnResultExecuted` reads the status code and result type and logs them.

**Expected Output:** A log entry: “Result ObjectResult executed with status 200; canceled: False; exception: False”.

**Why this output?** `OnResultExecuted` runs after the result has executed, giving access to the final status code and result type.

### Real-World Cases

- **Audit logging:** Log the result type, status code, and duration for every request.
- **Exception detection:** Detect and log exceptions thrown during result execution (e.g., view rendering errors).
- **Cancellation tracking:** Log when a result was canceled by an earlier filter.

### References

- `ResultExecutedContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.resultexecutedcontext
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters

---

## 6. Result Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Result filter best practices are guidelines for implementing result filters that are correct, testable, and maintainable. |
| **Technical Definition** | Best practices encompass using `IAsyncResultFilter` for I/O-bound work, implementing `IAlwaysRunResultFilter` when headers must be added even to short-circuited responses, using `Cancel` carefully and setting an explicit status code, **not attempting to modify the response body** in `OnResultExecuted`, and understanding that ordinary result filters **do not run** when the pipeline is short-circuited by authorization or resource filters. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for result filters—they help you avoid silent failures (filters not running on 401s), broken responses (modifying the body too late), and untestable filter logic. |

### Purposes

- **To** ensure filters run in all expected scenarios (including short-circuited requests) using `IAlwaysRunResultFilter`.
- **To** avoid attempting response-body modifications after the response has started.
- **To** use async filters for I/O-bound work.
- **To** cancel result execution deliberately and set explicit status codes.
- **To** keep filters focused on result-level concerns (headers, envelopes, logging).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Short-circuit awareness** | Implement `IAlwaysRunResultFilter` when headers must be added even to 401/403 responses. | Assume ordinary result filters run when authorization or resource filters short-circuit. |
| **Async** | Use `IAsyncResultFilter` for any `await`-based work. | Use `IResultFilter` for I/O (blocks threads). |
| **Cancellation** | Set `context.Cancel = true` and explicitly set `HttpContext.Response.StatusCode`. | Cancel without setting a status code (response may be empty with 200). |
| **Response body** | Modify headers and replace the result **before** execution (`OnResultExecuting`). | Attempt to modify the response body in `OnResultExecuted` (response has started). |
| **Result replacement** | Replace `context.Result` in `OnResultExecuting` to transform the response. | Replace `context.Result` in `OnResultExecuted` (no effect; it is read-only). |
| **DI** | Use `ServiceFilter` or `TypeFilter` for dependency injection. | Use `[new MyFilter()]` attributes that bypass DI. |
| **Ordering** | Use `IOrderedFilter.Order` only when specific ordering across scopes is required. | Rely on arbitrary ordering between global, controller, and action filters. |
| **Exception handling** | Let middleware handle exceptions from result execution. | Attempt complex exception recovery in result filters. |

### Annotated Code Example (Best-Practice Result Filter)

```csharp
// ✅ Best practice: Runs even on short-circuited responses
public class SecurityHeadersAlwaysRunFilter : IAlwaysRunResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers.Append(
            "X-Content-Type-Options", "nosniff");
        context.HttpContext.Response.Headers.Append(
            "X-Frame-Options", "DENY");
        context.HttpContext.Response.Headers.Append(
            "Referrer-Policy", "no-referrer");
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // No post-execution logic
    }
}

// Register globally
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<SecurityHeadersAlwaysRunFilter>();
});

// ✅ Async filter for result timing with I/O
public class AsyncResultTelemetryFilter : IAsyncResultFilter
{
    private readonly ITelemetryService _telemetry;

    public AsyncResultTelemetryFilter(ITelemetryService telemetry)
        => _telemetry = telemetry;

    public async Task OnResultExecutionAsync(
        ResultExecutingContext context,
        ResultExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();
        await next();
        sw.Stop();

        await _telemetry.TrackResultAsync(
            context.ActionDescriptor.DisplayName,
            context.Result.GetType().Name,
            sw.ElapsedMilliseconds);
    }
}
```

**Why this follows best practices:**

- **`IAlwaysRunResultFilter`** ensures security headers are added even when authorization short-circuits the pipeline (e.g., 401/403 responses).
- **Headers are added in `OnResultExecuting`** before the response starts, so they are included in the final response.
- **`IAsyncResultFilter`** is used for the telemetry filter because `_telemetry.TrackResultAsync` performs I/O.
- **`await next()`** is called to execute the result, and the post-execution code runs after.
- **No response-body modification** is attempted in `OnResultExecuted`.

### Real-World Cases

- **Security headers on all responses:** `IAlwaysRunResultFilter` ensures headers are present even on 401/403/404 responses produced by short-circuits.
- **API response envelopes:** `OnResultExecuting` replaces `ObjectResult` values with a standard envelope.
- **Result telemetry:** Async filters track duration and result type without blocking threads.
- **Cache header policies:** Different result types receive different `Cache-Control` headers.

### References

- Filters in ASP.NET Core (Result filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters
- `IAlwaysRunResultFilter` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.ialwaysrunresultfilter
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute

---

## Summary Table: Result Filters at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|------------------|-------------|
| **Result filters** | Wrap `IActionResult` execution | `IResultFilter` / `IAsyncResultFilter` / `IAlwaysRunResultFilter` | Headers, envelopes, result logging |
| **`IResultFilter`** | Synchronous result filter | `OnResultExecuting` / `OnResultExecuted` | Simple synchronous before/after logic |
| **`IAsyncResultFilter`** | Asynchronous result filter | `OnResultExecutionAsync(context, next)` | Async logging, telemetry, cache |
| **`ResultExecutingContext`** | Context before result execution | `Result` (settable), `Cancel` | Replace or cancel the result |
| **`ResultExecutedContext`** | Context after result execution | `Result` (read-only), `Canceled`, `Exception` | Post-result logging and inspection |
| **Best practices** | Correctness and short-circuit awareness | `IAlwaysRunResultFilter`, `Cancel` with explicit status, async for I/O | Every result filter implementation |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters
- `IResultFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iresultfilter
- `IAsyncResultFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter
- `IAlwaysRunResultFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.ialwaysrunresultfilter
- `ResultExecutingContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.resultexecutingcontext
- `ResultExecutedContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.resultexecutedcontext
- `ResultExecutingContext.Cancel` property – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.resultexecutingcontext.cancel
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute
- Action filter best practices (related filter guidance) – https://www.milanjovanovic.tech/blog/how-to-use-action-filters-in-aspnetcore