# ASP.NET Core MVC Action Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** (.NET 6–10). Action filters are the third stage of the MVC filter pipeline—after authorization and resource filters, and around the action method itself. They implement `IActionFilter` (synchronous) or `IAsyncActionFilter` (asynchronous) and can inspect or modify action arguments before execution and the result after execution. Action filters **do not apply to Razor Page handler methods** (use `IPageFilter`/`IAsyncPageFilter`); they do apply to the page model when configured globally. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action filters are components that run immediately before and after an action method is invoked, allowing inspection or modification of the action’s arguments and result. |
| **Technical Definition** | Action filters implement `IActionFilter` (synchronous) or `IAsyncActionFilter` (asynchronous). `OnActionExecuting`/`OnActionExecutionAsync` run **after model binding and resource filters** but **before the action method**. `OnActionExecuted`/`OnActionExecutionAsync`’s post-`next` code run **after the action method** but **before result filters**. Action filters can read and modify `ActionArguments`, short-circuit the action by setting `context.Result`, and inspect or replace the action’s result. They are the canonical filter type for argument validation, action-level logging, and result enrichment. |
| **Beginner-Friendly Explanation** | Action filters are **wrappers around a single action method**. They run right before your action code and right after it. They can look at the parameters the action is about to receive, and they can look at or change what the action returns—all without modifying the action itself. |

### Key Characteristics

- **Action-scoped** – Unlike resource filters, action filters wrap **only** the action method (and other action filters), not model binding or result execution.
- **Argument-aware** – `ActionExecutingContext.ActionArguments` exposes the bound action parameters, which can be inspected or modified.
- **Result-aware** – `ActionExecutedContext.Result` exposes the action’s return value, which can be inspected or replaced.
- **Short-circuit capable** – Setting `context.Result` in `OnActionExecuting` prevents the action from running.
- **Two interfaces** – `IActionFilter` (synchronous) and `IAsyncActionFilter` (asynchronous).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC project.
- Working knowledge of the **filter pipeline** (authorization → resource → action → exception → result).
- Familiarity with **model binding**, **controllers**, and **dependency injection**.

### Related Programming Areas

- **Resource filters** – Wrap model binding + action + result; broader scope than action filters.
- **Exception filters** – Handle exceptions thrown by the action (and by action filters).
- **Result filters** – Run after action filters, around result execution.
- **Model binding** – Populates `ActionArguments` before action filters run.
- **Dependency injection** – `ServiceFilter` and `TypeFilter` enable DI for action filters.

### Core Concepts / Features

1. **Action filters** – The concept, pipeline position, and use cases.
2. **`IActionFilter`** – The synchronous interface.
3. **`IAsyncActionFilter`** – The asynchronous interface.
4. **`ActionExecutingContext`** – The context for the “before” stage.
5. **`ActionExecutedContext`** – The context for the “after” stage.
6. **Action filter best practices** – Async, DI, ordering, and short-circuiting discipline.

---

## 1. Action Filters (The Concept and Pipeline Position)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action filters are MVC filters that execute immediately before and after an action method is invoked. |
| **Technical Definition** | Action filters implement `IActionFilter` or `IAsyncActionFilter`. `OnActionExecuting` runs after model binding and resource filters but before the action method. `OnActionExecuted` runs after the action method but before result filters. The `ActionExecutingContext` exposes `ActionArguments` (mutable) and a settable `Result` property for short-circuiting; the `ActionExecutedContext` exposes the action’s `Result` and `Exception` (if one occurred and was not yet handled). Action filters are the **innermost** of the “before” filters (authorization → resource → action) and the **first** of the “after” filters. |
| **Beginner-Friendly Explanation** | Action filters are **wrappers around your action method**. They run right before the action (after model binding has populated its arguments) and right after the action (before the result is executed). They are the best place for logic that is specifically about the action—its arguments and its result—rather than about the whole request. |

### Purposes

- **To** inspect or modify **action arguments** before the action executes (e.g., normalizing inputs, validating cross-parameter rules).
- **To** **short-circuit the action** when preconditions fail (e.g., invalid arguments, business rule violations).
- **To** inspect or modify the **action result** after execution (e.g., wrapping JSON responses, adding headers to specific results).
- **To** log or measure **action-specific** information (e.g., which action ran, how long it took, what arguments it received).
- **To** implement **action-level** caching, idempotency checks, or feature flags.

### Syntax Rules and Structure

#### Pipeline Position

```text
Request → Authorization Filters → Resource Filters (before) → Model Binding
→ [Action Filters (before)] → Action Method → [Action Filters (after)]
→ Exception Filters → Result Filters → Resource Filters (after) → Response
```

**Component breakdown:**

| Stage | Filter Type | Runs relative to action filters |
|-------|-------------|---------------------------------|
| 1 | Authorization | Before (first) |
| 2 | Resource (before) | Before (wraps model binding) |
| 3 | Model binding | Before action filters |
| 4 | **Action (before)** | Immediately before the action |
| 5 | Action | The action method itself |
| 6 | **Action (after)** | Immediately after the action |
| 7 | Exception | After action filters (when exceptions occurred) |
| 8 | Result | After action filters |
| 9 | Resource (after) | After everything above |

**Syntax rules:**

- `OnActionExecuting` runs **after model binding** (so `ActionArguments` is populated) and **before the action**.
- `OnActionExecuted` runs **after the action** and **before result filters**.
- Setting `context.Result` in `OnActionExecuting` **short-circuits** the action and jumps to result filters.
- Action filters **do not wrap model binding**—use resource filters for that.

**Constraints and limitations:**

- **Action filters do not apply to Razor Page handler methods**—use `IPageFilter`/`IAsyncPageFilter` instead.
- **Action filters do not catch exceptions** from the action—use exception filters for that.
- **Exceptions thrown in action filters** are handled by exception filters **after** the action stage.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ArgumentLoggingActionFilter : IActionFilter
{
    private readonly ILogger<ArgumentLoggingActionFilter> _logger;

    public ArgumentLoggingActionFilter(ILogger<ArgumentLoggingActionFilter> logger)
        => _logger = logger;

    public void OnActionExecuting(ActionExecutingContext context)
    {
        foreach (var arg in context.ActionArguments)
        {
            _logger.LogInformation("Argument {Name} = {Value}",
                arg.Key, arg.Value ?? "(null)");
        }
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation("Action {Action} returned {Result}",
            context.ActionDescriptor.DisplayName,
            context.Result?.GetType().Name ?? "(null)");
    }
}

// Usage
[ServiceFilter(typeof(ArgumentLoggingActionFilter))]
public IActionResult Edit(int id, ProductInputModel input)
{
    // ...
    return View(input);
}
```

**Step-by-step explanation:**

1. The filter implements `IActionFilter` and injects `ILogger<T>` via constructor.
2. `OnActionExecuting` runs after model binding, iterating over `ActionArguments` (the bound `id` and `input`).
3. The action method `Edit` runs.
4. `OnActionExecuted` runs after the action, logging the result type.

**Expected Output:** Two log entries per request: one listing the action arguments, and one showing the result type returned by the action.

**Why this output?** `ActionExecutingContext.ActionArguments` exposes the bound parameters; `ActionExecutedContext.Result` exposes the action’s return value after execution.

### Real-World Cases

- **Audit logging:** Log which user called which action with which arguments.
- **Argument normalization:** Trim or standardize input arguments before the action processes them.
- **Result transformation:** Wrap action results in a standard envelope for APIs.
- **Action-level caching:** Check a cache in `OnActionExecuting` and short-circuit on a hit.

### References

- Filters in ASP.NET Core (Action filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters
- Action filter best practices – https://www.milanjovanovic.tech/blog/how-to-use-action-filters-in-aspnetcore

---

## 2. `IActionFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IActionFilter` is the synchronous interface for implementing action filters in ASP.NET Core MVC. |
| **Technical Definition** | `IActionFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares two methods: `void OnActionExecuting(ActionExecutingContext context)` and `void OnActionExecuted(ActionExecutedContext context)`. `OnActionExecuting` runs before the action; `OnActionExecuted` runs after. The interface extends `IFilterMetadata`. It is suitable for CPU-bound logic (argument validation, header manipulation) but not for asynchronous I/O. |
| **Beginner-Friendly Explanation** | `IActionFilter` is the **synchronous contract** for action filters. You implement two methods—one that runs before the action and one that runs after. It is fine for quick, synchronous work, but for anything involving I/O (database, cache, HTTP), use the async version. |

### Purposes

- **To** provide a **synchronous extension point** around action execution.
- **To** allow inspection and modification of **action arguments** before the action runs.
- **To** enable **short-circuiting** by setting `context.Result` in `OnActionExecuting`.
- **To** support **post-action logic** such as result inspection or logging.
- **To** serve as the simplest, most direct way to wrap an action method.

### Syntax Rules and Structure

```csharp
public interface IActionFilter : IFilterMetadata
{
    void OnActionExecuting(ActionExecutingContext context);
    void OnActionExecuted(ActionExecutedContext context);
}
```

**Component breakdown:**

| Method | Runs | Context |
|--------|------|---------|
| `OnActionExecuting` | Before the action | `ActionExecutingContext` (set `Result` to short-circuit) |
| `OnActionExecuted` | After the action | `ActionExecutedContext` |

**Syntax rules:**

- Both methods must be implemented.
- `OnActionExecuting` runs **after model binding** (so `ActionArguments` is populated).
- `OnActionExecuted` runs **before result filters**.
- The filter must be public and implement `IActionFilter`.

**Constraints and limitations:**

- **Synchronous only**—use `IAsyncActionFilter` for asynchronous work.
- **Does not catch exceptions**—exceptions from the action propagate to exception filters.
- **Controller-level filters run before action-level filters** in the default scope order; use `IOrderedFilter.Order` to override.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ValidateModelStateFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(context.ModelState);
        }
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        // No post-action logic needed
    }
}
```

**Step-by-step explanation:**

1. The filter implements `IActionFilter`.
2. `OnActionExecuting` checks `ModelState.IsValid`.
3. If invalid, `context.Result` is set to `BadRequestObjectResult`—the action method is **not** invoked, and result filters execute against this result.
4. If valid, `OnActionExecuting` returns without setting `Result`, and the action runs.

**Expected Output:** Actions with invalid model state return HTTP 400 with the `ModelState` payload; the action method is skipped entirely.

**Why this output?** Setting `context.Result` in `OnActionExecuting` short-circuits the action, and result filters execute against the assigned result instead of the action’s return value.

### Real-World Cases

- **API model-state guard:** Return 400 automatically when `ModelState` is invalid.
- **Feature flags:** Short-circuit with 404 when a feature is disabled.
- **Argument normalization:** Modify `context.ActionArguments` before the action runs.

### References

- `IActionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iactionfilter
- Action filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters

---

## 3. `IAsyncActionFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAsyncActionFilter` is the asynchronous interface for implementing action filters that perform I/O-bound work. |
| **Technical Definition** | `IAsyncActionFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)`. The method runs around the action: code before `await next()` runs before the action, and code after `await next()` runs after. Setting `context.Result` before calling `next()` short-circuits the action and skips `next()`. The `ActionExecutionDelegate` returns an `ActionExecutedContext` after the action and any inner action filters have completed. |
| **Beginner-Friendly Explanation** | `IAsyncActionFilter` is the **asynchronous version** of the action filter contract. It has one method that receives a `next` delegate: you write “before” code, `await next()` to run the action, and then write “after” code. It is the recommended approach for anything that involves `await`—logging, database calls, cache reads, or HTTP requests. |

### Purposes

- **To** perform **asynchronous before/after action logic** without blocking threads.
- **To** support **I/O-bound work** in action filters (logging to a database, checking a distributed cache, calling an external service).
- **To** enable **short-circuiting** asynchronously when preconditions fail.
- **To** provide a **single-method** interface that naturally expresses the “around” semantics of action filters.
- **To** be the **recommended** action filter interface for modern ASP.NET Core applications.

### Syntax Rules and Structure

```csharp
public interface IAsyncActionFilter : IFilterMetadata
{
    Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `context` | `ActionExecutingContext`; set `Result` to short-circuit. |
| `next` | `ActionExecutionDelegate`; call `await next()` to run the action. |
| Return | `Task`; supports `async`/`await`. |

**Syntax rules:**

- Code before `await next()` runs **before the action**.
- Code after `await next()` runs **after the action**.
- Setting `context.Result` before `await next()` **short-circuits** the action and **`next()` should not be called**.
- `await next()` returns an `ActionExecutedContext` with the action’s result.

**Constraints and limitations:**

- **Do not call `next()` after short-circuiting**—assigning `context.Result` and then calling `next()` leads to the action running anyway, which defeats the short-circuit.
- **Exceptions from the action** propagate through `await next()`—handle them or let exception filters catch them.
- **Result filters run after** the action filters’ post-`next` code (when no exception is handled).

### Annotated Complete Step-by-Step Code Example

```csharp
public class TimingActionFilter : IAsyncActionFilter
{
    private readonly ILogger<TimingActionFilter> _logger;

    public TimingActionFilter(ILogger<TimingActionFilter> logger)
        => _logger = logger;

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        var sw = Stopwatch.StartNew();

        // Before the action
        _logger.LogInformation("Starting {Action}",
            context.ActionDescriptor.DisplayName);

        // Execute the action (and any inner action filters)
        var executed = await next();

        sw.Stop();
        _logger.LogInformation("Finished {Action} in {Elapsed}ms",
            executed.ActionDescriptor.DisplayName,
            sw.ElapsedMilliseconds);
    }
}

// Usage
[ServiceFilter(typeof(TimingActionFilter))]
public async Task<IActionResult> Index()
{
    await Task.Delay(50);
    return View();
}
```

**Step-by-step explanation:**

1. The filter implements `IAsyncActionFilter` and injects `ILogger<T>`.
2. `OnActionExecutionAsync` starts a stopwatch and logs before the action.
3. `await next()` runs the action (and any inner action filters).
4. After `next()` returns, the stopwatch is stopped and the duration is logged.

**Expected Output:** Two log entries per request: “Starting Index” and “Finished Index in Xms”.

**Why this output?** `await next()` invokes the action and returns the `ActionExecutedContext`; the code after `await next()` runs after the action completes.

### Real-World Cases

- **Distributed tracing:** Wrap each action with trace spans and timing.
- **Database audit:** Log action invocations to a database asynchronously.
- **Cache check:** Await a distributed cache before the action and short-circuit on a hit.
- **External calls:** Enrich the action result by calling an external service after execution.

### References

- `IAsyncActionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncactionfilter
- Action filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters

---

## 4. `ActionExecutingContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ActionExecutingContext` is the context object passed to action filters **before** the action method executes, exposing the bound arguments and allowing short-circuiting. |
| **Technical Definition** | `ActionExecutingContext` (in `Microsoft.AspNetCore.Mvc.Filters`) inherits from `FilterContext` and adds `ActionArguments` (an `IDictionary<string, object?>`) and a settable `Result` property. It provides access to `HttpContext`, `ActionDescriptor`, `ModelState`, `RouteData`, and `Filters`. Setting `Result` to a non-null `IActionResult` short-circuits the action and skips the `next()` delegate. |
| **Beginner-Friendly Explanation** | `ActionExecutingContext` is the **“before the action” envelope**. It gives you the arguments the action is about to receive, the model state (validation errors), and a `Result` slot you can fill in to stop the action from running. |

### Purposes

- **To** provide access to the **bound action arguments** so filters can inspect or modify them before the action executes.
- **To** expose **`ModelState`** for validation-based short-circuiting.
- **To** enable **short-circuiting** by setting `Result`.
- **To** give access to the **route data**, **action descriptor**, and **HTTP context** for contextual logic.
- **To** allow argument-level **normalization** (trimming, redacting, or transforming values).

### Syntax Rules and Structure

```csharp
public class ActionExecutingContext : FilterContext
{
    public IDictionary<string, object?> ActionArguments { get; set; }
    public IActionResult? Result { get; set; }
    // Inherited from FilterContext:
    // HttpContext, ActionDescriptor, ModelState, RouteData, Filters
}
```

**Component breakdown:**

| Property | Type | Purpose |
|----------|------|---------|
| `ActionArguments` | `IDictionary<string, object?>` | The bound action parameters (mutable). |
| `Result` | `IActionResult?` | Set to short-circuit the action. |
| `HttpContext` | `HttpContext` | The request context. |
| `ActionDescriptor` | `ActionDescriptor` | Metadata about the action. |
| `ModelState` | `ModelStateDictionary` | Validation state. |
| `RouteData` | `RouteData` | Route values. |

**Syntax rules:**

- `ActionArguments` is populated **after model binding**; modifications to its values affect the action.
- Setting `Result` to a non-null value **short-circuits** the action and skips `next()`.
- `ModelState.IsValid` reflects validation results from model binding and validation attributes.

**Constraints and limitations:**

- **Modifying `ActionArguments`** only affects the action’s parameters—not the original HTTP request.
- **Setting `Result`** does not throw; it simply prevents the action from running.

### Annotated Complete Step-by-Step Code Example

```csharp
public class NormalizeEmailActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (context.ActionArguments.TryGetValue("email", out var raw) &&
            raw is string email)
        {
            // Normalize to lowercase and trim
            context.ActionArguments["email"] = email.Trim().ToLowerInvariant();
        }
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage
[ServiceFilter(typeof(NormalizeEmailActionFilter))]
public IActionResult Register(string email)
{
    // email is already normalized to lowercase/trimmed
    return Ok(email);
}
```

**Step-by-step explanation:**

1. The filter implements `IActionFilter`.
2. `OnActionExecuting` retrieves the `email` argument from `ActionArguments`.
3. It normalizes the value (trim + lowercase) and **reassigns** it in the dictionary.
4. The action method receives the normalized value.

**Expected Output:** A request with `email="  John@Example.COM  "` results in the action receiving `"john@example.com"`.

**Why this output?** Modifying `ActionArguments` before the action runs changes the value the action receives.

### Real-World Cases

- **Input normalization:** Trim and case-normalize emails, usernames, or codes.
- **Secret redaction:** Replace sensitive values with redacted placeholders before logging.
- **Validation short-circuit:** Set `Result` when `ModelState.IsValid` is `false`.

### References

- `ActionExecutingContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutingcontext
- Model state validation and filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation

---

## 5. `ActionExecutedContext`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `ActionExecutedContext` is the context object passed to action filters **after** the action method executes, exposing the result, exception, and cancellation state. |
| **Technical Definition** | `ActionExecutedContext` (in `Microsoft.AspNetCore.Mvc.Filters`) inherits from `FilterContext` and adds `Result` (the action’s `IActionResult`), `Exception` (the unhandled exception, if any), `ExceptionDispatchInfo`, and `Canceled`. It is returned by `await next()` in `IAsyncActionFilter` and passed to `OnActionExecuted` in `IActionFilter`. Modifying `Result` replaces the action’s return value; setting `ExceptionHandled` (via the base `FilterContext`) suppresses the exception. |
| **Beginner-Friendly Explanation** | `ActionExecutedContext` is the **“after the action” envelope**. It gives you what the action returned (or the exception it threw), so you can log it, transform it, or replace it before the result is executed. |

### Purposes

- **To** expose the action’s **return value** (`Result`) for inspection or replacement.
- **To** expose any **unhandled exception** thrown by the action for logging or custom handling.
- **To** indicate whether the action was **canceled**.
- **To** allow **result transformation** before result filters run.
- **To** support **exception handling** within action filters (though exception filters are typically the better place for that).

### Syntax Rules and Structure

```csharp
public class ActionExecutedContext : FilterContext
{
    public IActionResult? Result { get; set; }
    public Exception? Exception { get; }
    public ExceptionDispatchInfo? ExceptionDispatchInfo { get; }
    public bool Canceled { get; }
    // Inherited: HttpContext, ActionDescriptor, ModelState, RouteData, Filters
}
```

**Component breakdown:**

| Property | Type | Purpose |
|----------|------|---------|
| `Result` | `IActionResult?` | The action’s return value (mutable). |
| `Exception` | `Exception?` | The unhandled exception, if any. |
| `ExceptionDispatchInfo` | `ExceptionDispatchInfo?` | Preserves the original stack trace. |
| `Canceled` | `bool` | Whether the action was canceled (e.g., by a short-circuit). |

**Syntax rules:**

- `Result` can be **replaced** to transform the action’s output.
- `Exception` is non-null when the action (or an inner action filter) threw an unhandled exception.
- Handling the exception (marking it as handled) is typically done in **exception filters**, not action filters.

**Constraints and limitations:**

- **Result filters run after** `OnActionExecuted`, so modifying `Result` here affects what result filters see.
- **Handling exceptions here** is possible but not the recommended pattern—use exception filters for cross-cutting exception handling.

### Annotated Complete Step-by-Step Code Example

```csharp
public class WrapJsonResultActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.Result is ObjectResult objectResult &&
            objectResult.Value is not null)
        {
            // Wrap the result in a standard envelope
            context.Result = new ObjectResult(new
            {
                Success = true,
                Data = objectResult.Value
            })
            {
                StatusCode = objectResult.StatusCode
            };
        }
    }
}

// Usage
[ServiceFilter(typeof(WrapJsonResultActionFilter))]
public IActionResult Get() => Ok(new { Id = 1, Name = "Widget" });
```

**Step-by-step explanation:**

1. The filter implements `IActionFilter`.
2. `OnActionExecuting` does nothing (no short-circuit).
3. The action returns `Ok(...)`, producing an `ObjectResult`.
4. `OnActionExecuted` inspects `context.Result`; if it is an `ObjectResult` with a value, the filter **replaces** it with a wrapped envelope.

**Expected Output:** The client receives `{ "Success": true, "Data": { "Id": 1, "Name": "Widget" } }` instead of the raw `{ "Id": 1, "Name": "Widget" }`.

**Why this output?** The filter replaces `context.Result` in `OnActionExecuted`, so result filters and the response use the new value.

### Real-World Cases

- **Response envelopes:** Wrap API responses in a standard `{ success, data, errors }` structure.
- **Header injection:** Add headers based on the action’s result.
- **Result logging:** Log the type and status code of the action’s result.

### References

- `ActionExecutedContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutedcontext
- Result transformation – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters

---

## 6. Action Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Action filter best practices are guidelines for implementing action filters that are correct, testable, and maintainable. |
| **Technical Definition** | Best practices encompass using `IAsyncActionFilter` for I/O-bound work, registering filters via `ServiceFilter` or `TypeFilter` for DI, **not calling `next()` after short-circuiting**, using `IOrderedFilter.Order` when ordering across scopes matters, keeping filters focused on a single concern, and understanding the distinction between action filters (around the action) and resource filters (around the whole pipeline). |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for action filters—they help you avoid blocking threads, silently broken short-circuits, untestable filter logic, and misplaced concerns that belong in a different filter type. |

### Purposes

- **To** avoid blocking threads by using `IAsyncActionFilter` for I/O.
- **To** ensure short-circuiting actually works (no `next()` after `context.Result`).
- **To** make filters testable through DI.
- **To** keep filters focused and reusable.
- **To** place concerns in the correct filter type (resource vs. action vs. exception vs. result).

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Async** | Use `IAsyncActionFilter` for any `await`-based work (logging, cache, database). | Use `IActionFilter` for I/O (blocks threads). |
| **Short-circuiting** | Set `context.Result` and **do not call `next()`**. | Set `context.Result` and still call `next()`—the action runs anyway. |
| **DI** | Register with `ServiceFilter` or `TypeFilter`. | Use `[new MyFilter()]` attributes that bypass DI. |
| **Ordering** | Rely on scope for the default order (global → controller → action); use `Order` sparingly. | Assume controller-level filters run after action-level filters. |
| **Filter placement** | Use resource filters for caching/early rejection; action filters for argument/result logic. | Put model binding concerns in action filters (model binding already ran). |
| **Result modification** | Modify `context.Result` in `OnActionExecuted` (or after `next()`). | Modify the response body directly in action filters—use result filters for that. |
| **Exceptions** | Use exception filters for unhandled exception handling. | Rely on action filters for cross-cutting exception handling. |
| **Testing** | Test filters in isolation with mock `ActionExecutingContext`/`ActionExecutedContext`. | Only test through full integration tests. |

### Annotated Code Example (Best-Practice Action Filter)

```csharp
public class IdempotencyActionFilter : IAsyncActionFilter
{
    private readonly IIdempotencyStore _store;

    public IdempotencyActionFilter(IIdempotencyStore store)
        => _store = store;

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        var key = context.HttpContext.Request.Headers["Idempotency-Key"]
            .FirstOrDefault();

        if (string.IsNullOrEmpty(key))
        {
            // No idempotency key — proceed normally
            await next();
            return;
        }

        // Check if this request was already processed
        var cached = await _store.GetAsync(key);
        if (cached is not null)
        {
            // ✅ Short-circuit — do NOT call next()
            context.Result = cached;
            return;
        }

        // Execute the action
        var executed = await next();

        // Store the result for future duplicate requests
        if (executed.Result is not null)
        {
            await _store.StoreAsync(key, executed.Result);
        }
    }
}

// Attribute with DI
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class IdempotentAttribute : TypeFilterAttribute
{
    public IdempotentAttribute()
        : base(typeof(IdempotencyActionFilter))
    {
    }
}

// Usage
[Idempotent]
public async Task<IActionResult> CreateOrder(OrderInput input)
{
    // ...
    return Ok(orderId);
}
```

**Why this follows best practices:**

- **`IAsyncActionFilter`** is used because the store call is async.
- **`return;` after setting `context.Result`** ensures `next()` is **not** called—the action is truly short-circuited.
- **`TypeFilterAttribute`** enables DI for `IIdempotencyStore`.
- **Single responsibility**—the filter only handles idempotency, not unrelated concerns.
- **The filter uses `OnActionExecuted` semantics via the post-`next` code** to store results after the action.

### Real-World Cases

- **Idempotency:** Prevent duplicate order creation when the client retries.
- **Audit logging:** Async logging to a database or external service.
- **Feature flags:** Short-circuit actions when a feature is disabled.
- **A/B testing:** Modify action arguments based on the user’s cohort.

### References

- Filters in ASP.NET Core (Action filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters
- Action filter best practices – https://www.milanjovanovic.tech/blog/how-to-use-action-filters-in-aspnetcore
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute

---

## Summary Table: Action Filters at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|------------------|-------------|
| **Action filters** | Wrap the action method | `IActionFilter` / `IAsyncActionFilter` | Argument and result logic around a specific action |
| **`IActionFilter`** | Synchronous action filter | `OnActionExecuting` / `OnActionExecuted` | Simple synchronous before/after logic |
| **`IAsyncActionFilter`** | Asynchronous action filter | `OnActionExecutionAsync(context, next)` | Logging, cache, database, any async work |
| **`ActionExecutingContext`** | Context before the action | `ActionArguments`, `Result` (settable) | Argument inspection/modification, short-circuiting |
| **`ActionExecutedContext`** | Context after the action | `Result`, `Exception` | Result transformation, post-action logging |
| **Best practices** | Correctness and maintainability | `IAsyncActionFilter`, `return` after short-circuit, `TypeFilterAttribute` | Every action filter implementation |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Action filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#action-filters
- `IActionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iactionfilter
- `IAsyncActionFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncactionfilter
- `ActionExecutingContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutingcontext
- `ActionExecutedContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.actionexecutedcontext
- Filter DI with `TypeFilterAttribute` – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute
- Action filter best practices – https://www.milanjovanovic.tech/blog/how-to-use-action-filters-in-aspnetcore
- Model state validation and filters – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Cancellation and short-circuiting – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#cancellation-and-short-circuiting