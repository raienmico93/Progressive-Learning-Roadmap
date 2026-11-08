# ASP.NET Core MVC Authorization Filters: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core MVC** and **Razor Pages** (.NET 6–10). Authorization filters are the first stage of the MVC filter pipeline. They implement `IAuthorizationFilter` or `IAsyncAuthorizationFilter`, have only a “before” method, and are the only filters that run before resource filters. Custom authorization filters are a **last resort**—the framework documentation explicitly recommends policy-based authorization over custom filters. Each requested topic is treated independently—no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization filters are components that run first in the MVC filter pipeline to determine whether the current user is permitted to access the requested action or Razor Page. |
| **Technical Definition** | Authorization filters implement `IAuthorizationFilter` (synchronous) or `IAsyncAuthorizationFilter` (asynchronous). They execute **first** in the filter pipeline, before resource filters, action filters, exception filters, and result filters. They have only a “before” method—`OnAuthorization` or `OnAuthorizationAsync`—with no post-processing method. The built-in `AuthorizeFilter` delegates to ASP.NET Core’s policy-based authorization system. Custom authorization filters require a custom authorization framework and are **not** recommended for standard policy enforcement. |
| **Beginner-Friendly Explanation** | Authorization filters are the **security gate** at the very start of the MVC pipeline. Before any model binding, action execution, or result processing happens, they check: “Is this user allowed to be here?” If not, they stop the request immediately by setting `context.Result`. If they do nothing, the request proceeds. |


## 1. Authorization Filters (The Concept and Pipeline Position)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization filters are the first filters executed in the MVC filter pipeline, responsible for controlling access to action methods. |
| **Technical Definition** | Authorization filters run **before all other filter types**—only they execute before resource filters. They run after routing has selected the action but before model binding, action execution, and result processing. They have a single “before” method and no “after” method. The built-in authorization filter **calls the authorization system but does not itself authorize requests**—it delegates to policy-based authorization. Custom authorization filters bypass the policy system and require a fully custom authorization framework; the official guidance is to **prefer configuring authorization policies or writing custom authorization policies over writing custom filters**. |
| **Beginner-Friendly Explanation** | Authorization filters are the **first checkpoint** in the MVC pipeline. They run before anything else—before model binding, before your action code, before the result is formed. Their only job is to answer “yes” or “no” to the access question. If the answer is “no,” they short-circuit the pipeline and return an error (401, 403, or a challenge). |

### Purposes

- **To** enforce access control **before any other filter or action logic runs**, preventing unauthorized requests from consuming resources.
- **To** **short-circuit the filter pipeline** when the user is not authorized, avoiding unnecessary model binding, action execution, and result processing.
- **To** integrate with ASP.NET Core’s policy-based authorization system through the built-in `AuthorizeFilter` (the recommended approach).
- **To** support **custom authorization logic** (e.g., API key validation, subscription checks) when policy-based authorization is genuinely insufficient.
- **To** provide a **single, early enforcement point** for access rules that must run before any data is read or processed.

### Syntax Rules and Structure

#### Filter Pipeline Position

```text
Request → [Authorization Filters] → Resource Filters → Model Binding →
Action Filters → Action Execution → Exception Filters → Result Filters →
Response
```

**Component breakdown:**

| Stage | Filter Type | Description |
|-------|-------------|-------------|
| 1 | **Authorization** | Runs first; access control |
| 2 | Resource | Runs after authorization; wraps most of the pipeline |
| 3 | Action | Runs around the action method |
| 4 | Exception | Handles action exceptions |
| 5 | Result | Runs around result execution |

**Syntax rules:**

- Authorization filters are **always first**—no other filter type runs before them.
- They have **only a “before” method** (`OnAuthorization` or `OnAuthorizationAsync`); there is no “after” method.
- Setting `context.Result` **short-circuits** the rest of the filter pipeline and the action.

**Constraints and limitations:**

- **Custom authorization filters require a custom authorization framework** and bypass the standard policy system.
- **Never throw exceptions in authorization filters**—exception filters will **not** catch them, and the exception will not be handled.
- **Policy-based authorization is the recommended alternative**—use `[Authorize]` with policies before considering a custom filter.

### Annotated Complete Step-by-Step Code Example

```csharp
// A simple custom authorization filter (NOT recommended for standard policies)
public class MinimumAgeAuthorizationFilter : IAuthorizationFilter
{
    private readonly int _minimumAge;

    public MinimumAgeAuthorizationFilter(int minimumAge)
        => _minimumAge = minimumAge;

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var user = context.HttpContext.User;

        if (!user.Identity?.IsAuthenticated ?? true)
        {
            // Short-circuit: challenge unauthenticated users
            context.Result = new ChallengeResult();
            return;
        }

        var ageClaim = user.FindFirst("Age")?.Value;
        if (!int.TryParse(ageClaim, out var age) || age < _minimumAge)
        {
            // Short-circuit: forbid authenticated but underage users
            context.Result = new ForbidResult();
        }
    }
}

// Registration as a global filter
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new MinimumAgeAuthorizationFilter(18));
});
```

**Step-by-step explanation:**

1. The filter implements `IAuthorizationFilter` with a constructor parameter for the minimum age.
2. `OnAuthorization` runs **first** in the pipeline.
3. If the user is not authenticated, `context.Result` is set to `ChallengeResult`—the pipeline stops and the authentication handler issues a challenge.
4. If the user is authenticated but underage, `context.Result` is set to `ForbidResult`—the pipeline stops and 403 is returned.
5. If neither condition triggers, the filter returns without setting `context.Result`, and the request proceeds to resource filters, model binding, and the action.

**Expected Output:** An unauthenticated request receives a challenge (typically a redirect to login or 401). An authenticated user under 18 receives 403 Forbidden. An authenticated user 18 or older proceeds to the action normally.

**Why this output?** Authorization filters run before any other filter or action logic. Setting `context.Result` short-circuits the pipeline, and not setting it allows the request to continue.

### Real-World Cases

- **API key validation:** Reject requests missing a required `X-Api-Key` header before any processing occurs.
- **Subscription-level access:** Check whether a user’s subscription tier permits access to a premium feature.
- **Time-based access:** Restrict access to actions during business hours or maintenance windows.
- **Fallback policy enforcement:** Configure a global fallback policy that requires authentication for all endpoints without explicit `[Authorize]`.

### References

- Filters in ASP.NET Core (Authorization filters) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#authorization-filters
- IAuthorizationFilter interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iauthorizationfilter


## 2. `IAuthorizationFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAuthorizationFilter` is the synchronous interface for implementing custom authorization filters in ASP.NET Core MVC. |
| **Technical Definition** | `IAuthorizationFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `void OnAuthorization(AuthorizationFilterContext context)`. It extends `IFilterMetadata`. The `AuthorizationFilterContext` provides access to `HttpContext` (including `User`), `ActionDescriptor`, `Filters`, and a settable `Result` property. Setting `Result` short-circuits the remainder of the filter pipeline. The built-in `AuthorizeFilter` implements this interface. |
| **Beginner-Friendly Explanation** | `IAuthorizationFilter` is the **synchronous contract** for writing an authorization filter. You implement one method, `OnAuthorization`, and inside it you check the user and either let the request continue or set a result to stop it. |

### Purposes

- **To** provide a **synchronous extension point** for custom access-control logic.
- **To** allow inspection of the user, request headers, route data, and other request information before any other filter runs.
- **To** enable **short-circuiting** of the pipeline by setting `context.Result`.
- **To** support simple, CPU-bound authorization checks that do not require asynchronous I/O.
- **To** serve as the base interface for the built-in `AuthorizeFilter` and other framework authorization components.

### Syntax Rules and Structure

```csharp
public interface IAuthorizationFilter : IFilterMetadata
{
    void OnAuthorization(AuthorizationFilterContext context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `OnAuthorization` | The single method; receives `AuthorizationFilterContext`. |
| `AuthorizationFilterContext` | Provides `HttpContext`, `ActionDescriptor`, `Filters`, and settable `Result`. |
| `Result` | Setting this short-circuits the pipeline. |

**Syntax rules:**

- Implement `OnAuthorization`; there is **no** `OnAuthorizationExecuted` or equivalent.
- Set `context.Result` to a non-null `IActionResult` to short-circuit.
- Leave `context.Result` null to allow the request to proceed.
- The filter must be **public** and implement `IAuthorizationFilter`.

**Constraints and limitations:**

- **Synchronous only**—cannot use `await` inside `OnAuthorization`; use `IAsyncAuthorizationFilter` for asynchronous work.
- **Do not throw exceptions**—exception filters do not catch exceptions from authorization filters.
- **Custom filters bypass policy-based authorization**—prefer policies unless you genuinely need a custom filter.

### Annotated Complete Step-by-Step Code Example

```csharp
public class ApiKeyAuthorizationFilter : IAuthorizationFilter
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<ApiKeyAuthorizationFilter> _logger;

    public ApiKeyAuthorizationFilter(
        IConfiguration configuration,
        ILogger<ApiKeyAuthorizationFilter> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        // 1. Check for the API key header
        if (!context.HttpContext.Request.Headers
            .TryGetValue("X-Api-Key", out var providedKey))
        {
            _logger.LogWarning("API key missing from request to {Path}",
                context.HttpContext.Request.Path);

            // 2. Short-circuit with 401
            context.Result = new UnauthorizedObjectResult(new
            {
                Error = "API key is required",
                Code = "MISSING_API_KEY"
            });
            return;
        }

        // 3. Validate the key
        var validKey = _configuration["ApiKey"];
        if (providedKey != validKey)
        {
            _logger.LogWarning("Invalid API key for {Path}",
                context.HttpContext.Request.Path);

            // 4. Short-circuit with 401
            context.Result = new UnauthorizedObjectResult(new
            {
                Error = "Invalid API key",
                Code = "INVALID_API_KEY"
            });
        }
        // 5. If valid, do nothing—request proceeds
    }
}
```

**Step-by-step explanation:**

1. The filter injects `IConfiguration` and `ILogger<T>` via constructor (requires DI registration, see `TypeFilter` below).
2. `OnAuthorization` checks for the `X-Api-Key` header.
3. If missing, `context.Result` is set to `UnauthorizedObjectResult` with a structured error payload—the pipeline stops immediately.
4. If the key is present but invalid, the same short-circuit occurs with a different error code.
5. If the key is valid, the method returns without setting `context.Result`, allowing the request to proceed.

**Expected Output:** Requests without a valid API key receive a 401 response with a JSON error body. Requests with a valid key proceed to the action normally.

**Why this output?** `IAuthorizationFilter` runs first and has the authority to terminate the pipeline by setting `context.Result`. The `return` statements ensure no further processing occurs after a short-circuit.

### Real-World Cases

- **Webhook endpoints:** Validate API keys before accepting webhook payloads.
- **Internal service-to-service calls:** Check shared secrets or tokens in headers.
- **Feature-gated endpoints:** Verify a feature flag or license key before allowing access.

### References

- `IAuthorizationFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iauthorizationfilter
- Custom filter example (API key) – https://oneuptime.com/blog/post/2026-01-27-custom-filters-aspnet-core/view#1


## 3. `IAsyncAuthorizationFilter`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAsyncAuthorizationFilter` is the asynchronous interface for implementing custom authorization filters that perform I/O-bound work. |
| **Technical Definition** | `IAsyncAuthorizationFilter` (in `Microsoft.AspNetCore.Mvc.Filters`) declares a single method: `Task OnAuthorizationAsync(AuthorizationFilterContext context)`. It extends `IFilterMetadata`. Unlike the synchronous interface, it can `await` operations such as database queries, HTTP calls, or claims transformation. The framework recommends it when authorization logic depends on live data (e.g., checking whether an API key is actively billed). |
| **Beginner-Friendly Explanation** | `IAsyncAuthorizationFilter` is the **asynchronous version** of the authorization filter contract. Use it when your authorization check needs to `await` something—like querying a database to see if a subscription is still active—so you don’t block the server’s threads. |

### Purposes

- **To** support **asynchronous authorization logic** that performs I/O (database lookups, HTTP calls, cache reads) without blocking threads.
- **To** enable **dynamic, data-driven authorization** checks that cannot be expressed as static policies.
- **To** integrate with `IAuthorizationService` for policy evaluation in a custom filter.
- **To** provide a **non-blocking** alternative to `IAuthorizationFilter` for modern asynchronous applications.

### Syntax Rules and Structure

```csharp
public interface IAsyncAuthorizationFilter : IFilterMetadata
{
    Task OnAuthorizationAsync(AuthorizationFilterContext context);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `OnAuthorizationAsync` | The single asynchronous method. |
| `AuthorizationFilterContext` | Same context as the synchronous interface. |
| `Task` | Return type; supports `await`. |

**Syntax rules:**

- Implement `OnAuthorizationAsync`; there is no post-processing method.
- Use `async Task` for the implementation.
- Set `context.Result` to short-circuit; leave it null to proceed.
- Can `await` `IAuthorizationService.AuthorizeAsync` for policy evaluation.

**Constraints and limitations:**

- **Do not throw exceptions**—the same restriction as the synchronous version applies.
- **Async filters are slightly heavier** than synchronous ones; use them only when I/O is genuinely required.
- **`TypeFilterAttribute` with `IsReusable`** should not be used with filters that depend on services with a lifetime other than Singleton.

### Annotated Complete Step-by-Step Code Example

```csharp
public class PermissionFilter : IAsyncAuthorizationFilter
{
    private readonly IAuthorizationService _authService;
    private readonly PermissionRequirement _requirement;

    public PermissionFilter(
        IAuthorizationService authService,
        PermissionRequirement requirement)
    {
        _authService = authService;
        _requirement = requirement;
    }

    public async Task OnAuthorizationAsync(
        AuthorizationFilterContext context)
    {
        // 1. Call the authorization service asynchronously
        var result = await _authService.AuthorizeAsync(
            context.HttpContext.User,
            resource: null,
            _requirement);

        // 2. Short-circuit if not authorized
        if (!result.Succeeded)
        {
            context.Result = new ChallengeResult();
        }
    }
}

// Attribute wrapper for easy application
public class AuthorizePermissionAttribute : TypeFilterAttribute
{
    public AuthorizePermissionAttribute(params Permission[] permissions)
        : base(typeof(PermissionFilter))
    {
        Arguments = new[] { new PermissionRequirement(permissions) };
        Order = int.MinValue;   // Run as early as possible
    }
}

// Usage
[AuthorizePermission(Permission.Foo, Permission.Bar)]
public IActionResult Index() => View();
```

**Step-by-step explanation:**

1. The filter injects `IAuthorizationService` and a `PermissionRequirement` via constructor.
2. `OnAuthorizationAsync` awaits `AuthorizeAsync`, which may perform policy evaluation and I/O.
3. If authorization fails, `context.Result` is set to `ChallengeResult`, short-circuiting the pipeline.
4. If authorization succeeds, no result is set, and the request proceeds.
5. The `AuthorizePermissionAttribute` wraps the filter in a `TypeFilterAttribute`, passing constructor arguments and setting `Order = int.MinValue` to ensure it runs first.

**Expected Output:** A user without the required permissions receives a challenge (401 or redirect to login). A user with the required permissions proceeds to the `Index` action.

**Why this output?** The async filter awaits the authorization service and short-circuits when the policy does not succeed. The `TypeFilterAttribute` enables DI and constructor arguments while preserving the filter’s early execution order.

### Real-World Cases

- **Subscription validation:** Check a user’s subscription status against a live database before granting access to premium content.
- **Dynamic policy evaluation:** Evaluate complex, data-dependent policies that cannot be expressed as static `[Authorize(Policy = "...")]` rules.
- **Claims enrichment:** Await an external service to enrich the user’s claims before making an authorization decision.

### References

- `IAsyncAuthorizationFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncauthorizationfilter
- Async authorization filter with `IAuthorizationService` – https://stackoverflow.com/revisions/a6f5d7ff-4a1c-4ecf-ac20-f52ac6d4bf7a/view-source


## 4. Authorization Filter Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization filter best practices are guidelines for implementing access control that is secure, maintainable, and aligned with ASP.NET Core’s recommended authorization architecture. |
| **Technical Definition** | Best practices encompass preferring **policy-based authorization** over custom filters, **never throwing exceptions** inside authorization filters, using **`TypeFilterAttribute` or `ServiceFilterAttribute`** for DI, setting **`Order = int.MinValue`** when early execution is required, and **short-circuiting** with appropriate challenge/forbid results. |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for authorization filters—they help you avoid security holes, unhandled exceptions, and filters that fight against the framework’s built-in authorization system. |

### Purposes

- **To** avoid reimplementing authorization logic that the framework already provides through policies and handlers.
- **To** prevent unhandled exceptions that crash the request and bypass error-handling middleware.
- **To** ensure filters are **testable and maintainable** through proper DI registration.
- **To** guarantee that authorization filters run **before all other filters**, preserving their security role.
- **To** use the correct short-circuit result (`ChallengeResult` vs. `ForbidResult`) for the situation.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Authorization approach** | Prefer **policy-based authorization** (`[Authorize(Policy = "...")]`, `IAuthorizationHandler`) over custom filters. | Write a custom authorization filter for rules that policies can express. |
| **Exceptions** | **Never throw** inside an authorization filter; set `context.Result` to a challenge or forbid result. | Throw exceptions—exception filters will **not** catch them. |
| **DI and registration** | Use `TypeFilterAttribute` or `ServiceFilterAttribute` to enable constructor injection. | Use `new MyFilter()` in attributes, which bypasses DI. |
| **Execution order** | Set `Order = int.MinValue` on the attribute to ensure the filter runs before other filters. | Assume the filter will run before action filters without explicit ordering. |
| **Short-circuiting** | Use `ChallengeResult` for unauthenticated users and `ForbidResult` for authenticated but unauthorized users. | Return a generic 400 or 500 when a 401 or 403 is appropriate. |
| **Fallback policy** | Configure `AuthorizationOptions.FallbackPolicy` to enforce baseline authorization globally. | Rely on `[Authorize]` being present on every endpoint manually. |
| **Razor Pages** | Use `IPageFilter` / `IAsyncPageFilter` for page-specific authorization logic. | Apply MVC action filters to Razor Page handlers (not supported). |

### Annotated Code Example (Best-Practice Authorization Filter)

```csharp
// ✅ BEST PRACTICE: Async filter with DI, early order, no exceptions
public class ActiveSubscriptionFilter : IAsyncAuthorizationFilter
{
    private readonly ISubscriptionService _subscriptions;

    public ActiveSubscriptionFilter(ISubscriptionService subscriptions)
        => _subscriptions = subscriptions;

    public async Task OnAuthorizationAsync(
        AuthorizationFilterContext context)
    {
        var user = context.HttpContext.User;
        if (!user.Identity?.IsAuthenticated ?? true)
        {
            context.Result = new ChallengeResult();   // 401 / redirect
            return;
        }

        var isActive = await _subscriptions.IsActiveAsync(user);
        if (!isActive)
        {
            context.Result = new ForbidResult();      // 403
        }
    }
}

// Attribute wrapper with early order and DI
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class RequireActiveSubscriptionAttribute : TypeFilterAttribute
{
    public RequireActiveSubscriptionAttribute()
        : base(typeof(ActiveSubscriptionFilter))
    {
        Order = int.MinValue;   // Run before all other filters
    }
}

// Usage
[RequireActiveSubscription]
public IActionResult PremiumContent() => View();
```

**Why this follows best practices:**

- **Async interface** (`IAsyncAuthorizationFilter`) is used because the subscription check performs I/O.
- **`TypeFilterAttribute`** enables DI for `ISubscriptionService` without manual service registration.
- **`Order = int.MinValue`** ensures the filter runs before resource, action, and result filters.
- **No exceptions are thrown**—the filter sets `ChallengeResult` or `ForbidResult` to short-circuit appropriately.
- **The recommended alternative**—policy-based authorization—should be considered first; this filter is shown for scenarios where a genuinely custom, data-driven check is required.

### Real-World Cases

- **SaaS premium features:** Check a live subscription status before granting access to premium endpoints.
- **Multi-tenant isolation:** Verify that the authenticated user belongs to the tenant identified in the route.
- **Maintenance mode:** Block access to all non-admin users during scheduled maintenance.

### References

- Filters in ASP.NET Core (Authorization filters and best practices) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#authorization-filters
- Security cards: FallbackPolicy and authorization rules – https://github.com/reware-labs/securitycards
- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies


## Summary Table: Authorization Filters at a Glance

| Concept | Primary Role | Key Interface / Method | When to Use |
|---------|--------------|------------------------|-------------|
| **Authorization filters** | First-stage access control | `IAuthorizationFilter` / `IAsyncAuthorizationFilter` | When policy-based authorization is insufficient |
| **`IAuthorizationFilter`** | Synchronous authorization contract | `OnAuthorization(AuthorizationFilterContext)` | Simple, CPU-bound access checks |
| **`IAsyncAuthorizationFilter`** | Asynchronous authorization contract | `OnAuthorizationAsync(AuthorizationFilterContext)` | I/O-bound access checks (database, HTTP) |
| **Best practices** | Security and maintainability | Prefer policies, no exceptions, `TypeFilterAttribute`, `Order = int.MinValue` | Every custom authorization filter |

---

## References (Full List)

- Filters in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters
- Authorization filters (filter pipeline) – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#authorization-filters
- `IAuthorizationFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iauthorizationfilter
- `IAsyncAuthorizationFilter` interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncauthorizationfilter
- `AuthorizationFilterContext` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.filters.authorizationfiltercontext
- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Custom filter example (API key) – https://oneuptime.com/blog/post/2026-01-27-custom-filters-aspnet-core/view#1
- Async authorization filter with `IAuthorizationService` – https://stackoverflow.com/revisions/a6f5d7ff-4a1c-4ecf-ac20-f52ac6d4bf7a/view-source
- Security cards: FallbackPolicy and authorization rules – https://github.com/reware-labs/securitycards
- `TypeFilterAttribute` DI and `IsReusable` guidance – https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#typefilterattribute