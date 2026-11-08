# ASP.NET Core MVC Filter Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (controllers with views and API controllers) and **Razor Pages**. Filters are the primary extension point for cross-cutting concerns in the MVC action-invocation pipeline. The five filter types and their execution order are stable; DI support via `ServiceFilter`/`TypeFilter` is consistent across .NET 6–10. Filters **cannot** be applied to Razor Page handler methods—only to the page model or globally. Filters do **not** work directly with Blazor components. Each requested topic is treated independently—no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filters are components that run code before or after specific stages of the ASP.NET Core MVC action-invocation pipeline, enabling cross-cutting concerns to be handled without duplicating code. |
| **Technical Definition** | Filters are executed within the **filter pipeline**, which runs **after** MVC selects the action to execute. Each filter type is executed at a different stage: authorization filters run first, followed by resource filters, action filters, exception filters, and result filters. Filters can be applied at three **scopes**: globally (for all controllers/actions), at the controller level, or at the action-method level. Filters implement synchronous or asynchronous interfaces (e.g., `IActionFilter` / `IAsyncActionFilter`) and can be attribute-based, DI-activated (`ServiceFilter` / `TypeFilter`), or registered globally. |
| **Beginner-Friendly Explanation** | Filters are **checkpoints around your controller actions**. Some run before the action (to check permissions, validate input, or log), some run after, and some wrap the action entirely (to catch exceptions or transform results). They let you handle concerns like error handling, caching, authorization, and logging in one reusable place instead of repeating the same code in every action. |

### Key Characteristics

- **Stage-specific** – Each filter type runs at a well-defined point in the pipeline.
- **Five types** – Authorization, Resource, Action, Exception, and Result filters.
- **Three scopes** – Global, controller, and action-method scopes determine the default execution order (global surrounds controller surrounds action).
- **Bidirectional** – “Before” code runs forward; “after” code runs in reverse order due to nesting.
- **DI-capable** – Filters can receive dependencies via `ServiceFilter` or `TypeFilter`, but only if constructed through DI.
- **Short-circuit capable** – Filters can set `context.Result` to skip the rest of the pipeline.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- Working knowledge of **middleware**, **routing**, and **controllers/actions**.
- Basic understanding of **dependency injection** and **C# attributes**.

### Related Programming Areas

- **Middleware** – Filters are MVC-specific; middleware is pipeline-wide. Filters run after routing and MVC action selection.
- **Dependency injection** – `ServiceFilter` and `TypeFilter` enable DI for filters.
- **Authorization** – The built-in `AuthorizeFilter` is an authorization filter.
- **Exception handling** – Exception filters handle exceptions from action execution.
- **Result transformation** – Result filters inspect or modify `IActionResult` before execution.

### Core Concepts / Features

1. **Filters** – The concept, pipeline, scopes, and execution order.
2. **Filter types** – Authorization, Resource, Action, Exception, and Result filters.
3. **Filter best practices** – DI, ordering, short-circuiting, and maintainability guidelines.


## 1. Filters (The Concept and Pipeline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filters are components that run code before or after specific stages of the MVC action-invocation pipeline. |
| **Technical Definition** | Filters execute within the **filter pipeline**, which runs after ASP.NET Core selects the action to execute. The pipeline is a nested chain: global filters surround controller filters, which surround action-method filters. “Before” code runs from outermost (global) to innermost (action); “after” code runs in reverse. Filters can be added via attributes (controller or action), globally via `options.Filters.Add<T>()`, or via DI-aware attributes (`ServiceFilter`, `TypeFilter`). |
| **Beginner-Friendly Explanation** | Filters are **layers of logic wrapped around your action**. The outermost layer (global filters) runs first on the way in and last on the way out. The innermost layer (action filters) runs closest to your action code. This nesting is why “after” code runs in reverse order. |

### Purposes

- **To** consolidate cross-cutting concerns (error handling, caching, authorization, logging) into reusable components.
- **To** run code before and after action execution without modifying the action itself.
- **To** short-circuit the pipeline when a request should not proceed (e.g., unauthorized, cached response).
- **To** transform action arguments or results in a centralized way.
- **To** enable DI-based filters for testable, service-backed cross-cutting logic.

### Syntax Rules and Structure

#### Filter Scopes

```csharp
// 1. Global scope
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<GlobalSampleActionFilter>();
});

// 2. Controller scope
[SampleActionFilter]
public class ControllerFiltersController : Controller { ... }

// 3. Action-method scope
[SampleActionFilter]
public IActionResult Index() { ... }
```

**Component breakdown:**

| Scope | Registration | Notes |
|-------|--------------|-------|
| Global | `options.Filters.Add<T>()` | Surrounds all controller and action filters. |
| Controller | Attribute on controller class | Surrounds action-method filters. |
| Action | Attribute on action method | Innermost scope; runs closest to the action. |

#### Execution Order (Action Filters Example)

| Sequence | Filter scope | Filter method |
|----------|--------------|---------------|
| 1 | Global | `OnActionExecuting` |
| 2 | Controller | `OnActionExecuting` |
| 3 | Action | `OnActionExecuting` |
| 4 | Action | `OnActionExecuted` |
| 5 | Controller | `OnActionExecuted` |
| 6 | Global | `OnActionExecuted` |

**Syntax rules:**

- **Scope determines default order**: global surrounds controller surrounds action.
- **After code runs in reverse order** of before code due to nesting.
- **`IOrderedFilter.Order`** overrides scope order: a lower `Order` value runs its before code earlier and its after code later.
- **Controller-level filters** set `Order` to `int.MinValue` by default and cannot be configured to run after method-level filters.

**Constraints and limitations:**

- **Filters cannot be applied to Razor Page handler methods**—only to the page model or globally.
- **Filters do not work directly with Blazor components**—only indirectly when the component is embedded in a page or view that uses filters.
- **Filter nesting can be deep**—use `IOrderedFilter` only when scope-based ordering is insufficient.

### Annotated Code Example (Execution Order)

```csharp
// Global filter
public class GlobalSampleActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
        => Console.WriteLine("Global: Before");
    public void OnActionExecuted(ActionExecutedContext context)
        => Console.WriteLine("Global: After");
}

// Controller with controller-level filter
[SampleActionFilter]
public class ControllerFiltersController : Controller
{
    public IActionResult Index()
    {
        Console.WriteLine("Action: Executing");
        return Content("Check the Console.");
    }
}

// Registration
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<GlobalSampleActionFilter>();
});
```

**Expected Output:**

```
Global: Before
Controller: Before
Action: Before
Action: Executing
Action: After
Controller: After
Global: After
```

**Why this output?** Global filters surround controller filters, which surround action filters. “After” code runs in reverse order because the filters are nested.

### Real-World Cases

- **Audit logging:** A global action filter logs every action execution.
- **Caching:** A resource filter short-circuits the pipeline for cached responses.
- **Error handling:** An exception filter catches action exceptions and returns a consistent error response.

### References

- Filters in ASP.NET Core (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Filter scopes and order of execution – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filter-scopes-and-order-of-execution


## 2. Filter Types

### 2.1 Authorization Filters

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization filters run first in the filter pipeline and determine whether the current user is authorized for the request. |
| **Technical Definition** | Authorization filters implement `IAuthorizationFilter` or `IAsyncAuthorizationFilter`. They run **first**, before any other filter type, and have only a “before” method—no “after” method. The built-in `AuthorizeFilter` calls the authorization system. Authorization filters can short-circuit the pipeline if the request is not authorized. |
| **Beginner-Friendly Explanation** | Authorization filters are the **security gate**. They check whether the user is allowed to access the requested action. If not, they stop the request immediately—no other filters or the action itself will run. |

#### Purposes

- **To** enforce access control before any other filter or action logic runs.
- **To** short-circuit the pipeline for unauthorized requests, preventing unnecessary work.
- **To** integrate with ASP.NET Core’s policy-based authorization system.
- **To** support custom authorization logic when attribute-based policies are insufficient.

#### Syntax Rules and Structure

```csharp
public class MyAuthorizationFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        if (!context.HttpContext.User.Identity.IsAuthenticated)
        {
            context.Result = new UnauthorizedResult();
        }
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IAuthorizationFilter` | Interface with `OnAuthorization`. |
| `context.Result` | Setting this short-circuits the pipeline. |
| No “after” method | Authorization filters run before only. |

**Syntax rules:**

- Runs **before** resource, action, exception, and result filters.
- Only a “before” method exists—no post-processing.
- Setting `context.Result` short-circuits the request.

**Constraints and limitations:**

- **Do not throw exceptions in authorization filters**—exception filters will not handle them. Issue a challenge instead.
- **Prefer policy-based authorization** over custom authorization filters when possible; custom filters require a custom authorization framework.

#### Annotated Code Example

```csharp
public class ApiKeyAuthorizationFilter : IAuthorizationFilter
{
    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var apiKey = context.HttpContext.Request.Headers["X-Api-Key"];
        if (string.IsNullOrEmpty(apiKey))
        {
            context.Result = new UnauthorizedResult();
        }
    }
}
```

**Expected Output:** Requests without the `X-Api-Key` header receive a 401 Unauthorized response; the action never executes.

**Why this output?** The filter sets `context.Result`, which short-circuits the pipeline before any other filter or action runs.

#### Real-World Cases

- **API key validation:** Reject requests missing a required header.
- **Custom policy checks:** Enforce domain-specific access rules beyond roles.
- **Tenant isolation:** Verify the request belongs to the correct tenant.

#### References

- Authorization filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#authorization-filters


### 2.2 Resource Filters

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource filters run after authorization and wrap most of the filter pipeline, executing code before model binding and after the rest of the pipeline completes. |
| **Technical Definition** | Resource filters implement `IResourceFilter` or `IAsyncResourceFilter`. `OnResourceExecuting` runs **before** model binding and the rest of the filter pipeline; `OnResourceExecuted` runs **after** the rest of the pipeline has completed. Only authorization filters run before resource filters. Resource filters are useful for short-circuiting large portions of the pipeline (e.g., caching) and for preventing model binding from reading form data (e.g., large file uploads). |
| **Beginner-Friendly Explanation** | Resource filters are **outer wrappers** around most of the pipeline. They run before model binding and after everything else. They are ideal for caching (skip the whole pipeline on a cache hit) or for blocking model binding when it would be wasteful. |

#### Purposes

- **To** short-circuit the pipeline before model binding for cached responses or early rejection.
- **To** prevent model binding from reading form data when it is unnecessary or unsafe.
- **To** wrap the majority of the filter pipeline with before-and-after logic.
- **To** implement response caching at the filter level.

#### Syntax Rules and Structure

```csharp
public class MyResourceFilter : IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // Before model binding and rest of pipeline
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // After rest of pipeline completes
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IResourceFilter` | Synchronous interface. |
| `OnResourceExecuting` | Runs before model binding. |
| `OnResourceExecuted` | Runs after the rest of the pipeline. |

**Syntax rules:**

- Runs **after** authorization filters, **before** model binding.
- `OnResourceExecuting` can set `context.Result` to short-circuit.
- `OnResourceExecuted` runs after the rest of the pipeline completes.

**Constraints and limitations:**

- **Only authorization filters run before resource filters**—no other filter type precedes them.
- **Short-circuiting here** prevents model binding and all inner filters from running.

#### Annotated Code Example

```csharp
public class CacheResourceFilter : IResourceFilter
{
    private static readonly Dictionary<string, IActionResult> _cache = new();

    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        var path = context.HttpContext.Request.Path.ToString();
        if (_cache.TryGetValue(path, out var cached))
        {
            context.Result = cached;   // Short-circuit
        }
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        var path = context.HttpContext.Request.Path.ToString();
        if (context.Result is IActionResult result)
        {
            _cache[path] = result;
        }
    }
}
```

**Expected Output:** On a cache hit, the action and inner filters are skipped; the cached result is returned directly. On a miss, the action runs and the result is cached.

**Why this output?** `OnResourceExecuting` short-circuits by setting `context.Result`; `OnResourceExecuted` captures the result for future requests.

#### Real-World Cases

- **Response caching:** Skip the entire action pipeline for cached responses.
- **Large upload protection:** Prevent model binding from reading form data when it would exhaust memory.
- **Early request validation:** Reject malformed requests before model binding.

#### References

- Resource filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters


### 2.3 Action Filters

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action filters run immediately before and after an action method is called, and can change the arguments passed to the action and the result returned from it. |
| **Technical Definition** | Action filters implement `IActionFilter` (synchronous) or `IAsyncActionFilter` (asynchronous). They run **around** the action method: `OnActionExecuting` runs before the action, and `OnActionExecuted` runs after. Action filters can inspect and modify `ActionArguments` (the action’s parameters) and `context.Result` (the action’s return value). Setting `context.Result` in `OnActionExecuting` short-circuits the action. |
| **Beginner-Friendly Explanation** | Action filters are the **closest wrappers around your action method**. They run right before and right after the action, and they can inspect or change the action’s input arguments and its return value. |

#### Purposes

- **To** validate or transform action arguments before the action executes.
- **To** inspect or modify the action result after execution.
- **To** short-circuit the action when preconditions are not met.
- **To** implement logging, timing, or auditing around specific actions.

#### Syntax Rules and Structure

```csharp
public class MyActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        // Read/modify context.ActionArguments
        // Set context.Result to short-circuit
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // Inspect/modify context.Result
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IActionFilter` | Synchronous interface. |
| `OnActionExecuting` | Runs before the action. |
| `OnActionExecuted` | Runs after the action. |
| `ActionArguments` | The action’s parameters (mutable). |
| `Result` | The action’s return value (mutable). |

**Syntax rules:**

- `OnActionExecuting` runs after resource filters and before the action.
- Setting `context.Result` in `OnActionExecuting` short-circuits the action.
- `OnActionExecuted` runs after the action, before result filters.

**Constraints and limitations:**

- **Action filters do not apply to Razor Pages**—Razor Pages use `IPageFilter` / `IAsyncPageFilter` instead.
- **Async filters** should implement `IAsyncActionFilter` to avoid blocking.

#### Annotated Code Example (Async Action Filter with DI)

```csharp
public class LogActionFilter : IAsyncActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger) => _logger = logger;

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        _logger.LogInformation("Before: {Action}",
            context.ActionDescriptor.DisplayName);
        var executed = await next();   // Call the action
        _logger.LogInformation("After: {Action}",
            executed.ActionDescriptor.DisplayName);
    }
}

// Registration with DI
builder.Services.AddScoped<LogActionFilter>();

// Usage
[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index() => View();
```

**Expected Output:** The logger records a message before and after the `Index` action executes.

**Why this output?** `ServiceFilter` resolves the filter from DI, enabling `ILogger<T>` to be constructor-injected. `await next()` invokes the action and the rest of the pipeline.

#### Real-World Cases

- **Audit logging:** Log which user called which action with which arguments.
- **Argument validation:** Reject invalid action parameters before the action runs.
- **Result enrichment:** Add headers or transform the action result.

#### References

- Action filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters
- `IAsyncActionFilter` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncactionfilter


### 2.4 Exception Filters

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Exception filters catch unhandled exceptions thrown by action execution and the filters that run before and after it, allowing a consistent error response to be produced. |
| **Technical Definition** | Exception filters implement `IExceptionFilter` or `IAsyncExceptionFilter`. They run **after** action filters and **before** result filters. An exception filter can handle an exception by setting `context.ExceptionHandled = true` and assigning `context.Result`. Exception filters do **not** catch exceptions from resource filters, model binding, or authorization filters—those must be handled by middleware or other mechanisms. |
| **Beginner-Friendly Explanation** | Exception filters are the **safety net around the action**. If the action (or an action filter) throws an exception, the exception filter can catch it and return a friendly error response instead of letting the exception bubble up. |

#### Purposes

- **To** handle exceptions from action execution and produce consistent error responses.
- **To** log exceptions with action-specific context.
- **To** transform exceptions into appropriate `IActionResult` types (e.g., JSON error payloads).
- **To** avoid repeating try/catch blocks in every action.

#### Syntax Rules and Structure

```csharp
public class MyExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        // Log context.Exception
        context.ExceptionHandled = true;
        context.Result = new ObjectResult(new { error = "..." })
        {
            StatusCode = 500
        };
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IExceptionFilter` | Interface with `OnException`. |
| `context.Exception` | The thrown exception. |
| `context.ExceptionHandled` | Set to `true` to mark the exception as handled. |
| `context.Result` | The response to return. |

**Syntax rules:**

- Runs **after** action filters, **before** result filters.
- Setting `ExceptionHandled = true` prevents the exception from propagating.
- **Does not catch** exceptions from resource filters, model binding, or authorization filters.

**Constraints and limitations:**

- **Authorization filter exceptions** are not handled by exception filters—do not throw in authorization filters; issue a challenge instead.
- **Middleware exceptions** require `UseExceptionHandler`, not exception filters.

#### Annotated Code Example

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;

    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
        => _logger = logger;

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");
        context.ExceptionHandled = true;
        context.Result = new ObjectResult(new
        {
            error = "An unexpected error occurred.",
            traceId = context.HttpContext.TraceIdentifier
        })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };
    }
}

// Global registration
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<GlobalExceptionFilter>();
});
```

**Expected Output:** Any unhandled exception from an action returns a 500 JSON response with a generic error message and trace ID.

**Why this output?** The exception filter catches the exception, sets `ExceptionHandled = true`, and assigns a structured error result.

#### Real-World Cases

- **API error responses:** Consistent JSON error payloads for all actions.
- **Logging with context:** Capture the action name and arguments when an exception occurs.
- **Graceful degradation:** Return cached or fallback data when a service fails.

#### References

- Exception filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters


### 2.5 Result Filters

#### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Result filters run immediately before and after the execution of an `IActionResult`, allowing inspection or modification of the result before it is written to the response. |
| **Technical Definition** | Result filters implement `IResultFilter` or `IAsyncResultFilter`. `OnResultExecuting` runs before the result is executed; `OnResultExecuted` runs after. Result filters can modify `context.Result` or add response headers. They run **after** action filters and **after** exception filters (if no exception occurred), and they are the innermost filters before the response is written. |
| **Beginner-Friendly Explanation** | Result filters are the **final checkpoint before the response goes out**. They can inspect or change the result (for example, add a header or transform a JSON response) right before it is sent to the client. |

#### Purposes

- **To** inspect or modify the `IActionResult` before it is executed.
- **To** add response headers (e.g., caching headers, custom metadata).
- **To** transform the result format (e.g., wrap JSON responses in an envelope).
- **To** log or measure result execution time.

#### Syntax Rules and Structure

```csharp
public class MyResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        // Modify context.Result or add headers
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // After the result has been executed
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IResultFilter` | Synchronous interface. |
| `OnResultExecuting` | Runs before result execution. |
| `OnResultExecuted` | Runs after result execution. |
| `context.Result` | The action result (mutable). |

**Syntax rules:**

- Runs **after** action filters and **after** exception filters (when no exception is handled).
- `OnResultExecuting` can replace `context.Result` or add headers.
- `OnResultExecuted` runs after the response has started.

**Constraints and limitations:**

- **Cannot modify the response body** in `OnResultExecuted` if the response has already started.
- **Result filters do not run** if an exception filter sets `ExceptionHandled = true` and a new result is assigned—result filters for the original result are skipped.

#### Annotated Code Example

```csharp
public class SecurityHeadersResultFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers.Append(
            "X-Content-Type-Options", "nosniff");
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // Nothing to do after
    }
}
```

**Expected Output:** Every response includes the `X-Content-Type-Options: nosniff` header.

**Why this output?** The result filter adds the header in `OnResultExecuting`, before the result writes the response.

#### Real-World Cases

- **Security headers:** Add `X-Frame-Options`, `Content-Security-Policy`, etc. to all responses.
- **Cache headers:** Set `Cache-Control` based on the action or result.
- **Response envelopes:** Wrap API responses in a standard `{ data, meta }` structure.

#### References

- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters


## 3. Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Filter best practices are guidelines for choosing, ordering, implementing, and registering filters to ensure correctness, testability, and maintainability. |
| **Technical Definition** | Best practices encompass using the correct filter type for each concern, preferring async interfaces (`IAsyncActionFilter`, etc.) over synchronous ones, using `ServiceFilter`/`TypeFilter` for DI, avoiding short-circuiting in inappropriate stages, respecting execution order (`IOrderedFilter` when needed), and not throwing in authorization filters. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for filters—they help you avoid confusing ordering bugs, untestable filters, and exceptions that bypass your error-handling logic. |

### Purposes

- **To** choose the right filter type for each cross-cutting concern.
- **To** ensure filters are testable and maintainable through DI.
- **To** avoid subtle ordering and short-circuiting bugs.
- **To** keep the filter pipeline predictable and performant.
- **To** integrate filters correctly with exception handling and authorization.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Filter type selection** | Use authorization filters for access control, resource filters for caching/short-circuiting, action filters for action-level logic, exception filters for action exceptions, result filters for result transformation. | Use action filters for authorization or exception filters for middleware-level errors. |
| **Async** | Implement `IAsyncActionFilter`, `IAsyncResultFilter`, etc. | Use synchronous filters for I/O-bound work (blocks threads). |
| **DI** | Use `ServiceFilter` or `TypeFilter` for filters that need services. | Use `[Authorize]`-style attributes with constructor dependencies without DI. |
| **Authorization filters** | Issue a challenge for unauthenticated requests. | Throw exceptions—exception filters will not catch them. |
| **Ordering** | Rely on scope for default order; use `IOrderedFilter.Order` only when necessary. | Assume controller-level filters can run after action-level filters. |
| **Short-circuiting** | Short-circuit deliberately in resource filters (caching) or action filters (validation). | Short-circuit accidentally by setting `context.Result` without understanding the consequences. |
| **Razor Pages** | Use `IPageFilter` / `IAsyncPageFilter` for page-specific logic. | Apply action filters to Razor Page handler methods (not supported). |
| **Global filters** | Register global filters for truly cross-cutting concerns (logging, error handling). | Register highly specific filters globally (unnecessary overhead). |

### Annotated Code Example (Best-Practice Filter Setup)

```csharp
// Async action filter with DI via ServiceFilter
public class AuditActionFilter : IAsyncActionFilter
{
    private readonly IAuditService _audit;

    public AuditActionFilter(IAuditService audit) => _audit = audit;

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        await _audit.LogAsync("Before", context.ActionDescriptor.DisplayName);
        var executed = await next();
        await _audit.LogAsync("After", executed.ActionDescriptor.DisplayName);
    }
}

// Registration
builder.Services.AddScoped<AuditActionFilter>();
builder.Services.AddScoped<IAuditService, AuditService>();

// Usage
[ServiceFilter(typeof(AuditActionFilter))]
public IActionResult Index() => View();
```

**Why this follows best practices:**

- **Async interface** (`IAsyncActionFilter`) avoids blocking threads.
- **`ServiceFilter`** enables constructor injection of `IAuditService`.
- **Single responsibility**—the filter only handles auditing.
- **`await next()`** properly invokes the action and downstream pipeline.

### Real-World Cases

- **Global exception handling:** A global exception filter returns consistent error responses.
- **Per-action caching:** A resource filter short-circuits expensive actions on cache hits.
- **DI-backed audit logging:** An async action filter logs actions with a scoped audit service.

### References

- Filters in ASP.NET Core (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Dependency injection for filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#dependency-injection
- Filter scopes and order of execution – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filter-scopes-and-order-of-execution


## Summary Table: Filter Types at a Glance

| Filter Type | Interface(s) | When It Runs | Can Short-Circuit? | Typical Use |
|-------------|--------------|--------------|---------------------|-------------|
| **Authorization** | `IAuthorizationFilter`, `IAsyncAuthorizationFilter` | First; before all other filters | Yes (set `Result`) | Access control, API keys |
| **Resource** | `IResourceFilter`, `IAsyncResourceFilter` | After authorization; before model binding | Yes (set `Result`) | Caching, large-upload protection |
| **Action** | `IActionFilter`, `IAsyncActionFilter` | Immediately before/after the action | Yes (set `Result`) | Argument validation, logging |
| **Exception** | `IExceptionFilter`, `IAsyncExceptionFilter` | After action filters; before result filters | Yes (`ExceptionHandled = true`) | Error handling, logging |
| **Result** | `IResultFilter`, `IAsyncResultFilter` | Before/after result execution | Yes (replace `Result`) | Headers, response transformation |

---

## References (Full List)

- Filters in ASP.NET Core (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Filter scopes and order of execution – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#filter-scopes-and-order-of-execution
- Authorization filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#authorization-filters
- Resource filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters
- Action filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters
- Exception filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#exception-filters
- Result filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#result-filters
- Dependency injection for filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#dependency-injection
- `IAsyncActionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncactionfilter
- `IOrderedFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iorderedfilter