# ASP.NET Core MVC Resource Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Resource filters are the second stage of the MVC filter pipeline—after authorization filters and before model binding. They implement `IResourceFilter` or `IAsyncResourceFilter`, wrap the majority of the pipeline, and can short-circuit before model binding runs. Resource filters do not apply to Razor Page handler methods (use `IPageFilter`/`IAsyncPageFilter` instead). Each requested topic is treated independently—no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource filters are components that run after authorization and wrap most of the MVC filter pipeline, allowing code to execute before model binding and after the rest of the pipeline completes. |
| **Technical Definition** | Resource filters implement `IResourceFilter` (synchronous) or `IAsyncResourceFilter` (asynchronous). `OnResourceExecuting` runs **before model binding** and the rest of the filter pipeline; `OnResourceExecuted` runs **after the rest of the pipeline** has completed. Only authorization filters run before resource filters. Resource filters can short-circuit the entire downstream pipeline by setting `context.Result` in `OnResourceExecuting`. They are the canonical filter type for caching, large-upload protection, and any concern that must avoid the cost of model binding. |
| **Beginner-Friendly Explanation** | Resource filters are **outer wrappers** around almost everything in the MVC pipeline. They run before model binding (where the request body is parsed into your action’s parameters) and after the action and result have finished. This makes them perfect for caching: if a cached response exists, the filter can return it immediately and skip model binding, the action, and result execution entirely. |

### Key Characteristics

- **Second in the pipeline** – Only authorization filters run before resource filters.
- **Wraps model binding** – `OnResourceExecuting` runs before model binding; `OnResourceExecuted` runs after result execution.
- **Short-circuit capable** – Setting `context.Result` skips model binding, action filters, the action, and result filters.
- **Two interfaces** – `IResourceFilter` (synchronous) and `IAsyncResourceFilter` (asynchronous).
- **Not action filters** – Resource filters wrap far more of the pipeline than action filters, which only wrap the action method itself.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** (authorization → resource → action → exception → result).
- Familiarity with **model binding** and **dependency injection**.

### Related Programming Areas

- **Authorization filters** – The only filters that run before resource filters.
- **Action filters** – Run after model binding, unlike resource filters.
- **Model binding** – Resource filters can prevent it from reading form data.
- **Response caching** – The canonical use case for resource filters.
- **Exception filters** – Do **not** catch exceptions thrown from resource filters (they only wrap the action and action filters).

### Core Concepts / Features

1. **Resource filters** – The concept, pipeline position, and use cases.
2. **`IResourceFilter`** – The synchronous interface.
3. **`IAsyncResourceFilter`** – The asynchronous interface.
4. **Resource filter best practices** – Short-circuiting discipline, DI, and ordering.


## 1. Resource Filters (The Concept and Pipeline Position)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource filters are pipeline components that execute after authorization and wrap model binding, action execution, and result execution. |
| **Technical Definition** | Resource filters implement `IResourceFilter` or `IAsyncResourceFilter`. They are the **broadest-scoped filter type after authorization**: `OnResourceExecuting` runs before model binding, and `OnResourceExecuted` runs after result execution. Only authorization filters run before them. Setting `context.Result` in `OnResourceExecuting` short-circuits the entire downstream pipeline—model binding, action filters, the action, exception filters, and result filters all never run. |
| **Beginner-Friendly Explanation** | Resource filters are the **outer layer** around your action and everything that happens before and after it (except authorization). They are the first filters that can inspect the request in its raw form (before model binding) and the last filters to run after the response has been produced. |

### Purposes

- **To** short-circuit the pipeline **before model binding** for cached responses, early rejection, or large-upload protection.
- **To** wrap the majority of the MVC pipeline (model binding + action + result) with before-and-after logic.
- **To** prevent model binding from reading form data when it is unnecessary or unsafe (e.g., large file uploads).
- **To** implement response caching at the filter level, skipping all downstream work on a cache hit.
- **To** run cleanup or logging logic after the entire action-and-result pipeline has completed.

### Syntax Rules and Structure

#### Pipeline Position

```text
Request → Authorization Filters → [Resource Filters (before)] → Model Binding
→ Action Filters (before) → Action → Action Filters (after) → Exception Filters
→ Result Filters (before) → Result → Result Filters (after) → [Resource Filters (after)]
→ Response
```

**Component breakdown:**

| Stage | Filter Type | Runs relative to resource filters |
|-------|-------------|-----------------------------------|
| 1 | Authorization | **Before** resource filters |
| 2 | Resource (before) | Before model binding |
| 3 | Model binding | **Inside** resource filters |
| 4 | Action + action filters | Inside resource filters |
| 5 | Exception filters | Inside resource filters |
| 6 | Result + result filters | Inside resource filters |
| 7 | Resource (after) | After everything above |

**Syntax rules:**

- `OnResourceExecuting` runs **after authorization** and **before model binding**.
- `OnResourceExecuted` runs **after result execution**.
- Setting `context.Result` in `OnResourceExecuting` short-circuits the entire downstream pipeline.
- Resource filters **wrap** action filters, exception filters, and result filters.

**Constraints and limitations:**

- **Resource filters do not catch exceptions** from the action or downstream filters—exception filters handle those. Exceptions thrown from resource filters themselves propagate to middleware-level exception handling.
- **Razor Pages** use `IPageFilter` / `IAsyncPageFilter`, not resource filters.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ShortCircuitingResourceFilter : Attribute, IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        // Runs BEFORE model binding
        context.Result = new ContentResult
        {
            Content = nameof(ShortCircuitingResourceFilter)
        };
        // Pipeline is short-circuited: model binding, action, and result filters never run
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        // Runs AFTER result execution (never reached if short-circuited above)
    }
}

// Usage
[ShortCircuitingResourceFilter]
public IActionResult Index() => Content("This never runs.");
```

**Step-by-step explanation:**

1. The filter implements `IResourceFilter`.
2. `OnResourceExecuting` sets `context.Result` to a `ContentResult`—this short-circuits the pipeline.
3. Model binding, the `Index` action, action filters, and result filters **never execute**.
4. The `ContentResult` is returned directly to the client.

**Expected Output:** The client receives the content `"ShortCircuitingResourceFilter"`. The `Index` action body never runs.

**Why this output?** Setting `context.Result` in `OnResourceExecuting` bypasses the entire downstream pipeline, including model binding and the action method.

### Real-World Cases

- **Response caching:** Check a cache in `OnResourceExecuting`; if a hit exists, return it and skip model binding, the action, and result execution.
- **Large upload protection:** Reject requests with excessive `Content-Length` before model binding attempts to read the body into memory.
- **Maintenance mode:** Short-circuit all requests with a maintenance page before any application logic runs.

### References

- Filters in ASP.NET Core (Resource filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters
- Short-circuiting resource filter example – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#cancellation-and-short-circuiting


## 2. `IResourceFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IResourceFilter` is the synchronous interface for implementing resource filters in ASP.NET Core MVC. |
| **Technical Definition** | `IResourceFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares two methods: `void OnResourceExecuting(ResourceExecutingContext context)` and `void OnResourceExecuted(ResourceExecutedContext context)`. `OnResourceExecuting` runs before model binding and the rest of the filter pipeline; `OnResourceExecuted` runs after the rest of the pipeline has completed. Setting `context.Result` in `OnResourceExecuting` short-circuits the pipeline. |
| **Beginner-Friendly Explanation** | `IResourceFilter` is the **synchronous contract** for resource filters. You implement two methods: one that runs before model binding and one that runs after everything else finishes. If you set `context.Result` in the first method, the rest of the pipeline is skipped. |

### Purposes

- **To** provide a synchronous extension point for logic that must run before model binding and after result execution.
- **To** enable short-circuiting of the pipeline before expensive model binding occurs.
- **To** implement synchronous caching, early validation, or pipeline-wide logging.
- **To** wrap the action-and-result pipeline with before-and-after logic.

### Syntax Rules and Structure

```csharp
public interface IResourceFilter : IFilterMetadata
{
    void OnResourceExecuting(ResourceExecutingContext context);
    void OnResourceExecuted(ResourceExecutedContext context);
}
```

**Component breakdown:**

| Method | Runs | Context |
|--------|------|---------|
| `OnResourceExecuting` | Before model binding | `ResourceExecutingContext` (set `Result` to short-circuit) |
| `OnResourceExecuted` | After result execution | `ResourceExecutedContext` |

**Syntax rules:**

- Both methods must be implemented.
- `OnResourceExecuting` can set `context.Result` to short-circuit.
- `OnResourceExecuted` runs even if the pipeline was **not** short-circuited (unless an exception propagates).
- The filter must be public and implement `IResourceFilter`.

**Constraints and limitations:**

- **Synchronous only**—use `IAsyncResourceFilter` for asynchronous work.
- **Does not catch exceptions**—exceptions from the action propagate to exception filters or middleware.
- **Short-circuiting must be done carefully**—if you set `context.Result`, you must **not** call `next` in the async version (see `IAsyncResourceFilter`).

### Annotated Complete Step-by-Step Code Example

```csharp
public class LoggingResourceFilter : IResourceFilter
{
    private readonly ILogger<LoggingResourceFilter> _logger;

    public LoggingResourceFilter(ILogger<LoggingResourceFilter> logger)
        => _logger = logger;

    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        _logger.LogInformation("Resource pipeline starting for {Path}",
            context.HttpContext.Request.Path);
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        _logger.LogInformation("Resource pipeline completed for {Path}",
            context.HttpContext.Request.Path);
    }
}
```

**Step-by-step explanation:**

1. The filter injects `ILogger<T>` via constructor (requires DI registration).
2. `OnResourceExecuting` logs before model binding.
3. `OnResourceExecuted` logs after the action and result have completed.

**Expected Output:** Two log entries per request: one before model binding and one after result execution.

**Why this output?** `IResourceFilter` wraps model binding, action execution, and result execution, so the “before” and “after” methods bracket that entire span.

### Real-World Cases

- **Pipeline timing:** Measure the duration from before model binding to after result execution.
- **Request logging:** Log request start and end around the full MVC pipeline.
- **Early validation:** Check headers or route data before model binding runs.

### References

- `IResourceFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iresourcefilter
- Resource filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters


## 3. `IAsyncResourceFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAsyncResourceFilter` is the asynchronous interface for implementing resource filters that perform I/O-bound work. |
| **Technical Definition** | `IAsyncResourceFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `Task OnResourceExecutionAsync(ResourceExecutingContext context, ResourceExecutionDelegate next)`. The method can `await` asynchronous operations (database queries, cache reads, HTTP calls) and must call `await next()` to execute the rest of the pipeline—**unless** it sets `context.Result` to a non-null value to short-circuit. |
| **Beginner-Friendly Explanation** | `IAsyncResourceFilter` is the **asynchronous version** of the resource filter contract. Use it when your resource filter needs to `await` something—like checking a Redis cache or querying a database—without blocking threads. It has a single method that receives a `next` delegate; you either call `await next()` to continue or set `context.Result` to stop. |

### Purposes

- **To** perform **asynchronous** before-model-binding work (cache reads, database checks) without blocking threads.
- **To** short-circuit the pipeline asynchronously when a cached response or early rejection is available.
- **To** wrap the pipeline with asynchronous logging, metrics, or cleanup.
- **To** integrate with services like `IMemoryCache` or `IDistributedCache` that require async access.

### Syntax Rules and Structure

```csharp
public interface IAsyncResourceFilter : IFilterMetadata
{
    Task OnResourceExecutionAsync(
        ResourceExecutingContext context,
        ResourceExecutionDelegate next);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context` | `ResourceExecutingContext`; set `Result` to short-circuit. |
| `next` | `ResourceExecutionDelegate`; call `await next()` to continue the pipeline. |
| Return | `Task`; supports `async`/`await`. |

**Syntax rules:**

- Call `await next()` to execute the rest of the pipeline (model binding, action, result).
- **Do not call `next()` after setting `context.Result`**—this throws `InvalidOperationException`: “If an IAsyncResourceFilter provides a result value by setting the Result property of ResourceExecutingContext to a non-null value, then it cannot call the next filter by invoking ResourceExecutionDelegate”.
- Use `async Task` for the implementation.

**Constraints and limitations:**

- **Must not call `next` after short-circuiting**—the framework enforces this with an exception.
- **Exceptions from `OnResourceExecutionAsync`** are not caught by exception filters—they propagate to middleware-level exception handling.

### Annotated Complete Step-by-Step Code Example

```csharp
public class CachingResourceFilter : IAsyncResourceFilter
{
    private readonly IMemoryCache _cache;

    public CachingResourceFilter(IMemoryCache cache) => _cache = cache;

    public async Task OnResourceExecutionAsync(
        ResourceExecutingContext context,
        ResourceExecutionDelegate next)
    {
        var cacheKey = context.HttpContext.Request.Path.ToString();

        // 1. Try cache
        if (_cache.TryGetValue(cacheKey, out string? cachedContent))
        {
            // 2. Short-circuit — DO NOT call next()
            context.Result = new ContentResult { Content = cachedContent };
            return;
        }

        // 3. Execute the rest of the pipeline
        var executed = await next();

        // 4. Cache the result after execution
        if (executed.Result is ContentResult contentResult)
        {
            _cache.Set(cacheKey, contentResult.Content, TimeSpan.FromSeconds(60));
        }
    }
}
```

**Step-by-step explanation:**

1. The filter reads the cache using the request path as the key.
2. If a cached value exists, `context.Result` is set and the method **returns immediately**—`next()` is **not** called, avoiding the `InvalidOperationException`.
3. If no cached value exists, `await next()` executes model binding, the action, and result execution.
4. After execution, the result is cached for future requests.

**Expected Output:** On a cache hit, the cached content is returned and the action never runs. On a miss, the action runs and its result is cached.

**Why this output?** The async filter short-circuits by setting `context.Result` and returning without calling `next()`; on a miss, it executes the pipeline and caches the result afterward.

### Real-World Cases

- **Distributed caching:** Check Redis or SQL Server cache asynchronously before model binding.
- **Subscription checks:** Query a database asynchronously to validate a subscription before allowing the request into the pipeline.
- **API rate limiting:** Check a rate-limit store asynchronously and short-circuit with 429 if exceeded.

### References

- `IAsyncResourceFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresourcefilter
- Short-circuiting pitfall in async resource filters – https://stackoverflow.com/questions/67893373/-/67895078#67895078


## 4. Resource Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource filter best practices are guidelines for using resource filters correctly—especially around short-circuiting, asynchronous execution, and dependency injection. |
| **Technical Definition** | Best practices encompass **never calling `next()` after setting `context.Result`** in async resource filters, using `IAsyncResourceFilter` for any I/O-bound work, registering filters with `TypeFilterAttribute` or `ServiceFilterAttribute` for DI, keeping resource filters focused on concerns that genuinely require the before-model-binding timing, and understanding that exception filters do **not** catch resource filter exceptions. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for resource filters. The biggest “don’t” is calling `next()` after setting a result—that throws an error. The biggest “do” is using async filters for cache checks and database calls, and keeping resource filters focused on things that truly need to happen before model binding. |

### Purposes

- **To** avoid the `InvalidOperationException` caused by calling `next()` after short-circuiting.
- **To** use asynchronous I/O correctly without blocking threads.
- **To** enable dependency injection in resource filters.
- **To** keep resource filters focused on concerns that justify their early pipeline position.
- **To** prevent confusion about which filter type catches which exceptions.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Short-circuiting** | Set `context.Result` and **return immediately** (async) or skip `next` (sync). | Call `next()` after setting `context.Result`—throws `InvalidOperationException`. |
| **Async I/O** | Use `IAsyncResourceFilter` and `await` cache/database calls. | Use `IResourceFilter` for async work (blocks threads). |
| **DI** | Use `TypeFilterAttribute` or `ServiceFilterAttribute` to inject services. | Use `[new MyResourceFilter()]` attributes that bypass DI. |
| **Filter scope** | Apply resource filters only for concerns that must run **before model binding** (caching, early rejection). | Use resource filters for logic that can run after model binding (use action filters instead). |
| **Exception handling** | Handle resource filter exceptions with middleware (`UseExceptionHandler`). | Expect exception filters to catch exceptions thrown from resource filters—they do not. |
| **Razor Pages** | Use `IPageFilter` / `IAsyncPageFilter` for Razor Pages. | Apply MVC resource filters to Razor Page handlers. |

### Annotated Code Example (Best-Practice Async Resource Filter)

```csharp
public class SubscriptionCheckResourceFilter : IAsyncResourceFilter
{
    private readonly ISubscriptionService _subscriptions;

    public SubscriptionCheckResourceFilter(ISubscriptionService subscriptions)
        => _subscriptions = subscriptions;

    public async Task OnResourceExecutionAsync(
        ResourceExecutingContext context,
        ResourceExecutionDelegate next)
    {
        var user = context.HttpContext.User;
        if (!user.Identity?.IsAuthenticated ?? true)
        {
            context.Result = new ChallengeResult();
            return;   // ✅ Short-circuit — do NOT call next()
        }

        var isActive = await _subscriptions.IsActiveAsync(user);
        if (!isActive)
        {
            context.Result = new ForbidResult();
            return;   // ✅ Short-circuit — do NOT call next()
        }

        await next();   // ✅ Only call next() when not short-circuiting
    }
}

// Attribute with DI
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class RequireActiveSubscriptionAttribute : TypeFilterAttribute
{
    public RequireActiveSubscriptionAttribute()
        : base(typeof(SubscriptionCheckResourceFilter))
    {
        Order = int.MinValue + 1;   // After authorization, before other filters
    }
}

// Usage
[RequireActiveSubscription]
public IActionResult PremiumContent() => View();
```

**Why this follows best practices:**

- **`IAsyncResourceFilter`** is used because the subscription check performs I/O.
- **`return;` after setting `context.Result`** prevents the `InvalidOperationException` caused by calling `next()` after short-circuiting.
- **`TypeFilterAttribute`** enables DI for `ISubscriptionService`.
- **`Order = int.MinValue + 1`** ensures the filter runs after authorization (`int.MinValue`) but before other filters.
- **The filter is focused** on a concern that genuinely requires pre-model-binding execution.

### Real-World Cases

- **Cache-first APIs:** Check a distributed cache asynchronously and short-circuit on a hit.
- **Subscription-gated content:** Validate a user’s subscription status before allowing expensive model binding and action execution.
- **Large upload rejection:** Check `Content-Length` and short-circuit before model binding reads the body into memory.

### References

- Resource filters (official guidance) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters
- Short-circuiting and calling `next()` – https://stackoverflow.com/questions/67893373/-/67895078#67895078
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute


## Summary Table: Resource Filters at a Glance

| Concept | Primary Role | Key Interface / Method | When to Use |
|---------|--------------|------------------------|-------------|
| **Resource filters** | Wrap model binding + action + result | `IResourceFilter` / `IAsyncResourceFilter` | Caching, early rejection, large-upload protection |
| **`IResourceFilter`** | Synchronous resource filter | `OnResourceExecuting` / `OnResourceExecuted` | Simple synchronous before/after logic |
| **`IAsyncResourceFilter`** | Asynchronous resource filter | `OnResourceExecutionAsync(context, next)` | Cache reads, database checks, any I/O-bound work |
| **Best practices** | Correct short-circuiting and DI | Return after `context.Result`; use `TypeFilterAttribute` | Every resource filter implementation |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Resource filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#resource-filters
- `IResourceFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iresourcefilter
- `IAsyncResourceFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresourcefilter
- Short-circuiting and cancellation – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#cancellation-and-short-circuiting
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute
- Short-circuiting pitfall in async resource filters – https://stackoverflow.com/questions/67893373/-/67895078#67895078
- Resource filters overview (DEV Community) – https://dev.to/rhuturaj_takle/filters-in-aspnet-core-mvc-kco
- Resource filter use cases and best practices (LinkedIn) – https://www.linkedin.com/posts/hemil-shah_dotnet-aspnetcore-webapi-activity-7490784937122652161-to7a