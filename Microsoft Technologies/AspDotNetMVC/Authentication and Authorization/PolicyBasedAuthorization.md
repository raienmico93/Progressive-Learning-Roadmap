# ASP.NET Core Policy-Based Authorization: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and its policy-based authorization framework. Policy-based authorization is the foundational mechanism in ASP.NET Core—role-based and claims-based authorization are built on top of it as preconfigured policies. The framework's `AuthorizationMiddleware` evaluates policies against the `ClaimsPrincipal` produced by authentication. **Minimal APIs, MVC controllers, and Razor Pages** all share this authorization infrastructure through endpoint routing. Custom policy providers (`IAuthorizationPolicyProvider`) enable dynamic policy generation when static registration is insufficient. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Policy-based authorization is a model where access decisions are driven by named policies composed of one or more requirements, each evaluated by one or more handlers. |
| **Technical Definition** | An `AuthorizationPolicy` is a collection of `IAuthorizationRequirement` objects. Policies are registered via `AuthorizationBuilder.AddPolicy(name, policy)` or `AuthorizationOptions.AddPolicy`. The `IAuthorizationService` evaluates a policy by invoking all registered `AuthorizationHandler<TRequirement>` instances for each requirement. A policy succeeds if **every** requirement has at least one successful handler evaluation. Policies are applied via `[Authorize(Policy = "...")]` on controllers/actions or `RequireAuthorization("...")` on minimal API endpoints. Role-based and claims-based authorization are convenience wrappers that construct `RolesAuthorizationRequirement` and `ClaimsAuthorizationRequirement` respectively. |
| **Beginner-Friendly Explanation** | Policy-based authorization is the **rulebook system for access control**. Instead of scattering `if (user.IsInRole("Admin"))` checks across your code, you define named policies—"OnlyAdults", "SameTenant", "PremiumMember"—each expressing a complete rule. You then attach these policies to endpoints. When a request arrives, the framework checks the user against the policy's rules. If every rule passes, access is granted; otherwise, the user is forbidden (403) or challenged (401). |

### Key Characteristics

- **Requirement/handler model** – Policies are made of requirements; requirements are evaluated by handlers.
- **Composable** – Multiple requirements can be combined in a single policy; multiple policies can be combined on a single endpoint.
- **Testable** – Handlers and requirements can be unit tested independently of HTTP concerns.
- **Extensible** – Custom requirements and handlers express domain-specific rules (tenant isolation, account age, subscription tier).
- **Secure-by-default capable** – A fallback policy can require authentication for all endpoints without explicit authorization metadata.
- **Dynamic policy support** – `IAuthorizationPolicyProvider` generates policies at runtime for scenarios with many permission permutations.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **authentication** and **claims-based identity**.
- Familiarity with **middleware ordering** (`UseAuthentication` before `UseAuthorization`).
- Basic understanding of **dependency injection** (for handler registration).

### Related Programming Areas

- **Authentication** – Produces the `ClaimsPrincipal` that policies evaluate.
- **Claims** – The attributes (roles, tenant, subscription) that requirements check.
- **Middleware** – `UseAuthorization()` invokes the authorization middleware.
- **Minimal APIs** – `RequireAuthorization("policy")` applies policies to endpoint handlers.
- **Resource-based authorization** – `IAuthorizationService.AuthorizeAsync` with a resource for per-item checks.

### Core Concepts / Features

1. **Policy-based authorization** – The overall framework and evaluation model.
2. **Policy creation** – Registering named policies with requirements.
3. **Policy requirements** – `IAuthorizationRequirement` implementations carrying the rules.
4. **Policy handlers** – `AuthorizationHandler<TRequirement>` implementations evaluating requirements.
5. **Policy best practices** – Secure defaults, testability, and maintainability.

---

## 1. Policy-Based Authorization (The Framework)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Policy-based authorization is the ASP.NET Core authorization model where access decisions are expressed as named policies—sets of requirements evaluated by handlers. |
| **Technical Definition** | An `AuthorizationPolicy` is an immutable collection of `IAuthorizationRequirement` objects. The `IAuthorizationService` evaluates a policy by invoking all registered `IAuthorizationHandler` instances. A policy succeeds if **all** requirements succeed. Policies are applied via `[Authorize(Policy = "...")]` (MVC/Razor Pages/Blazor) or `RequireAuthorization("...")` (minimal APIs). The `AuthorizationMiddleware` (`UseAuthorization`) evaluates the applicable policy for each request. The **fallback policy** applies to endpoints without explicit authorization metadata; the **default policy** applies when `[Authorize]` is used without a policy name. |
| **Beginner-Friendly Explanation** | Policy-based authorization is the **"named rulebook" system**. You create a rulebook called "AdultsOnly" that says "the user must be 18 or older." Then you put a sign on a page saying "AdultsOnly required." When someone tries to access that page, the framework reads the rulebook, checks the user against every rule, and decides whether to let them in. This keeps access rules consistent, testable, and separate from the pages themselves. |

### Purposes

- **To** centralize **complex authorization logic** in named, reusable policies.
- **To** decouple **authorization decisions** from controllers, pages, and endpoint handlers.
- **To** support **fine-grained access control** through requirements and handlers.
- **To** enable **secure-by-default** behavior via a fallback policy.
- **To** provide a **testable** authorization model—handlers can be unit tested in isolation.

### Syntax Rules and Structure

#### General Syntax (Registration)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorizationBuilder()
    .AddPolicy("PolicyName", policy =>
        policy.RequireAuthenticatedUser()
              .RequireClaim("claim_type", "value")
              .RequireRole("RoleName"));

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthorizationBuilder()` | Registers authorization services and returns an `AuthorizationBuilder`. |
| `.AddPolicy(name, configure)` | Adds a named policy with the specified requirements. |
| `RequireAuthenticatedUser()` | Adds an authentication requirement. |
| `RequireClaim(...)` | Adds a claims requirement. |
| `RequireRole(...)` | Adds a roles requirement. |
| `UseAuthorization()` | Adds the authorization middleware (after authentication). |

#### Applying a Policy

```csharp
// MVC controller/action
[Authorize(Policy = "PolicyName")]
public IActionResult Protected() => View();

// Minimal API endpoint
app.MapGet("/protected", () => "OK")
   .RequireAuthorization("PolicyName");
```

**Syntax rules:**

- **Policies must be registered before the app starts**—registration occurs during service configuration.
- **`UseAuthentication()` must precede `UseAuthorization()`**.
- **Multiple `[Authorize]` attributes combine (AND semantics)**; multiple requirements within one policy also combine with AND.
- **The fallback policy applies to endpoints without explicit authorization metadata**; `[AllowAnonymous]` opts out.

**Constraints and limitations:**

- **Authorization alone does not authenticate**—the user must be identified first.
- **Policies are evaluated per endpoint**—they do not filter data (use resource-based authorization for per-item checks).
- **Handler registration is scoped**—handlers resolving Scoped services must be registered as Scoped.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Fallback Policy (Secure by Default)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication().AddCookie();
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` and `AddCookie` register authentication; users are identified per request.
2. `SetFallbackPolicy` requires authentication for **all endpoints** without explicit authorization metadata.
3. `UseAuthentication()` populates `HttpContext.User`.
4. `UseAuthorization()` evaluates the fallback policy or the endpoint's explicit policy.
5. Endpoints are mapped **after** authorization.

**Expected Output:** An unauthenticated request to **any** endpoint returns a challenge (401/redirect), unless the endpoint has `[AllowAnonymous]`. This is **secure-by-default** behavior.

**Why this output?** The fallback policy is applied to endpoints without explicit authorization metadata, protecting newly added endpoints that developers forget to annotate.

#### Example 2: Named Policy with Multiple Requirements

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("EmployeeOver21", policy =>
        policy.RequireAuthenticatedUser()
              .RequireClaim("EmployeeNumber")
              .AddRequirements(new MinimumAgeRequirement(21)));

// Apply the policy
[Authorize(Policy = "EmployeeOver21")]
public IActionResult Sensitive() => View();
```

**Expected Output:** The user must be authenticated **and** have an `EmployeeNumber` claim **and** satisfy the `MinimumAgeRequirement` (21+). All three requirements must pass.

**Why this output?** The policy combines multiple requirements; the authorization middleware evaluates each requirement through its handlers and succeeds only if all pass.

### Real-World Cases

- **Multi-tenant SaaS:** A `SameTenant` policy requiring the user's tenant claim to match the resource's tenant.
- **Financial applications:** A `MinimumAge` policy requiring users to be 18+.
- **Enterprise APIs:** A `EmployeeOnly` policy requiring an `EmployeeNumber` claim.
- **Public APIs with mixed access:** Fallback policy requiring authentication, with `AllowAnonymous` on public endpoints.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/

---

## 2. Policy Creation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Policy creation is the process of registering named policies with one or more requirements that define the conditions for access. |
| **Technical Definition** | Policies are created via `AuthorizationBuilder.AddPolicy(name, configure)` (preferred in .NET 6+) or `AuthorizationOptions.AddPolicy(name, configure)`. The `configure` delegate receives an `AuthorizationPolicyBuilder`, which provides convenience methods (`RequireAuthenticatedUser`, `RequireClaim`, `RequireRole`) and `AddRequirements` for custom requirements. Policies can also be created dynamically via a custom `IAuthorizationPolicyProvider`. |
| **Beginner-Friendly Explanation** | Policy creation is **writing a named rulebook**. You say: "I want a policy called 'AdultsOnly' that requires the user to be 18 or older." You register it once in `Program.cs`, and then you can attach it to any endpoint by name. The policy builder provides shortcuts for common rules (authentication, roles, claims) and lets you add custom rules when you need something more specific. |

### Purposes

- **To** define **reusable access rules** that can be applied to multiple endpoints.
- **To** provide **named references** for policies in `[Authorize]` and `RequireAuthorization`.
- **To** combine **multiple requirements** into a single policy.
- **To** support both **convenience methods** (roles, claims) and **custom requirements** (domain-specific rules).
- **To** enable **dynamic policy generation** when static registration is impractical.

### Syntax Rules and Structure

#### General Syntax (Static Registration)

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)))
    .AddPolicy("EmployeeOnly", policy =>
        policy.RequireClaim("EmployeeNumber"))
    .AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
```

**Component breakdown:**

| Method | Purpose |
|--------|---------|
| `AddPolicy(name, configure)` | Registers a named policy. |
| `policy.Requirements.Add(...)` | Adds a custom requirement directly. |
| `policy.RequireClaim(...)` | Adds a claims requirement. |
| `policy.RequireRole(...)` | Adds a roles requirement. |
| `policy.RequireAuthenticatedUser()` | Adds an authentication requirement. |

#### General Syntax (Using `AuthorizationOptions`)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));
});
```

**Syntax rules:**

- **Policies must be registered before the container is built**.
- **Policy names are case-sensitive** and must be unique.
- **`AddPolicy` is additive**—calling it multiple times with the same name throws `InvalidOperationException`.
- **The `AuthorizationPolicyBuilder` can combine multiple requirements** with AND semantics.

**Constraints and limitations:**

- **Static registration is impractical for hundreds of policies** (e.g., per-resource permissions)—use `IAuthorizationPolicyProvider` for dynamic generation.
- **Policies registered with `AddPolicy` cannot be modified after registration**—they are immutable.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Policy with a Custom Requirement

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

builder.Services.AddAuthorizationBuilder()
    .AddPolicy("AtLeast21", policy =>
        policy.AddRequirements(new MinimumAgeRequirement(21)));

[Authorize(Policy = "AtLeast21")]
public IActionResult AdultsOnly() => View();
```

**Step-by-step explanation:**

1. `MinimumAgeRequirement` carries the minimum age (21) as data.
2. `AddPolicy` registers the "AtLeast21" policy with the requirement.
3. `[Authorize(Policy = "AtLeast21")]` applies the policy to the action.

**Expected Output:** Users whose claims indicate they are 21 or older can access `AdultsOnly`; others receive 403 Forbidden.

**Why this output?** The policy evaluates the `MinimumAgeRequirement` through its handler, which checks the user's birthdate claim.

#### Example 2: Policy with Multiple Requirements

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("ManagerInAcme", policy =>
        policy.RequireAuthenticatedUser()
              .RequireRole("Manager")
              .RequireClaim("tenant", "acme"));
```

**Expected Output:** The user must be authenticated **and** have the `Manager` role **and** have a `tenant` claim with value `"acme"`. All three conditions must be satisfied.

**Why this output?** `RequireAuthenticatedUser`, `RequireRole`, and `RequireClaim` add separate requirements that are all evaluated (AND semantics).

#### Example 3: Dynamic Policy Provider (Conceptual)

```csharp
public class PermissionPolicyProvider : IAuthorizationPolicyProvider
{
    public Task<AuthorizationPolicy> GetPolicyAsync(string policyName)
    {
        // policyName format: "Permission:Read:Documents"
        var parts = policyName.Split(':');
        var permission = parts[1];

        var policy = new AuthorizationPolicyBuilder()
            .AddRequirements(new PermissionRequirement(permission))
            .Build();

        return Task.FromResult(policy);
    }

    public Task<AuthorizationPolicy> GetDefaultPolicyAsync() =>
        Task.FromResult(new AuthorizationPolicyBuilder()
            .RequireAuthenticatedUser().Build());

    public Task<AuthorizationPolicy> GetFallbackPolicyAsync() =>
        Task.FromResult<AuthorizationPolicy>(null);
}

builder.Services.AddSingleton<IAuthorizationPolicyProvider, PermissionPolicyProvider>();
```

**Expected Output:** Policies are generated on demand based on the policy name—no need to register every permission combination upfront.

**Why this output?** The custom provider parses the policy name and constructs a policy with the appropriate requirement.

### Real-World Cases

- **Static policies for common rules:** `"AdminOnly"`, `"EmployeeOnly"`, `"AtLeast21"`.
- **Dynamic policies for fine-grained permissions:** `"Permission:Read:Documents"`, `"Permission:Write:Orders"`.
- **Multi-tenant policies:** `"SameTenant"` combined with resource-based authorization.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Custom authorization policy providers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider

---

## 3. Policy Requirements

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A policy requirement is a condition that a policy evaluates; a policy is a collection of requirements, all of which must be satisfied for access to be granted. |
| **Technical Definition** | `IAuthorizationRequirement` is an empty marker interface. Implementations carry the data needed for evaluation (e.g., `MinimumAge`, `TenantId`, `PermissionName`). Requirements are added to policies via `AuthorizationPolicyBuilder.AddRequirements` or convenience methods (`RequireClaim`, `RequireRole`, `RequireAuthenticatedUser`). Each requirement is evaluated by one or more `AuthorizationHandler<TRequirement>` implementations. |
| **Beginner-Friendly Explanation** | A requirement is a **single rule** that must be satisfied. "Must be 18 or older" is a requirement. "Must belong to the same tenant as the resource" is a requirement. "Must have an EmployeeNumber claim" is a requirement. A policy combines several requirements into a complete rulebook. Requirements carry the **data** the handler needs (e.g., the minimum age value), while handlers contain the **logic** that evaluates the rule. |

### Purposes

- **To** express **individual conditions** that a policy evaluates.
- **To** carry **data** needed by the handler (minimum age, tenant ID, permission name).
- **To** enable **reuse** of the same requirement across multiple policies.
- **To** support **unit testing** of authorization logic by testing requirements and handlers in isolation.
- **To** decouple the **definition** of a rule from its **evaluation**.

### Syntax Rules and Structure

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

// Usage in a policy
options.AddPolicy("AtLeast21", policy =>
    policy.AddRequirements(new MinimumAgeRequirement(21)));
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IAuthorizationRequirement` | Marker interface; no members. |
| Constructor parameters | Data the requirement carries (e.g., `minimumAge`). |
| Properties | Expose the data to handlers. |

**Syntax rules:**

- **Requirements can be empty markers** (no data) when the handler relies entirely on `HttpContext` or claims.
- **Requirements can carry data** when the rule is parameterized (e.g., minimum age, tenant ID).
- **Requirements are added to policies** via `AddRequirements` or convenience methods.
- **Each requirement must have at least one handler**—otherwise, the policy fails.

**Constraints and limitations:**

- **A requirement without a handler fails**—the policy cannot succeed for that requirement.
- **Requirements should be simple data carriers**—complex logic belongs in the handler.
- **The same requirement type can be used in multiple policies**—handlers evaluate it the same way each time.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Empty Marker Requirement

```csharp
public class SameTenantRequirement : IAuthorizationRequirement
{
    // No data — the handler compares the user's tenant claim
    // to the resource's tenant ID
}

public class SameTenantHandler
    : AuthorizationHandler<SameTenantRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameTenantRequirement requirement,
        Document resource)
    {
        var tenant = context.User.FindFirst("tenant")?.Value;
        if (tenant == resource.TenantId)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

**Expected Output:** A user can access a `Document` only if their `tenant` claim matches the document's `TenantId`.

**Why this output?** The `SameTenantRequirement` carries no data; the handler compares the user's claim to the resource's property. This is a **resource-based** authorization check.

#### Example 2: Data-Carrying Requirement

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

public class MinimumAgeHandler
    : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var birthDate = context.User.FindFirst(ClaimTypes.DateOfBirth);
        if (birthDate is null) return Task.CompletedTask;

        var age = DateTime.Today.Year - DateTime.Parse(birthDate.Value).Year;
        if (age >= requirement.MinimumAge)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

**Expected Output:** Users 21 or older pass the requirement; others do not.

**Why this output?** The requirement carries `MinimumAge = 21`; the handler reads it and compares it to the user's computed age.

### Real-World Cases

- **Minimum age requirements:** Parameterized with the minimum age value.
- **Tenant isolation requirements:** Compare the user's tenant claim to the resource's tenant.
- **Permission requirements:** Carry a permission name (e.g., `"Read"`, `"Write"`) checked against a permission store.
- **Account age requirements:** Require the account to be at least N days old.

### References

- IAuthorizationRequirement interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement
- Policy-based authorization in ASP.NET Core (Requirements) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## 4. Policy Handlers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A policy handler is a class that evaluates one or more requirements and calls `context.Succeed(requirement)` or `context.Fail()` to record the outcome. |
| **Technical Definition** | `AuthorizationHandler<TRequirement>` is the base class for handlers evaluating a single requirement type. `AuthorizationHandler<TRequirement, TResource>` evaluates a requirement against a resource (for resource-based authorization). Handlers are registered in DI (typically as Scoped) and resolved by the `IAuthorizationService`. A handler can evaluate multiple requirements; the policy succeeds only if every requirement has at least one successful handler (and no handler calls `Fail()` for it). |
| **Beginner-Friendly Explanation** | A handler is the **logic that checks a rule**. If the requirement is "must be 18," the handler is the code that reads the user's birthdate claim, computes their age, and says "yes" or "no." Handlers are where the real work happens—requirements are the labels, handlers are the implementations. Handlers can use dependency injection to access databases, configuration, or other services. |

### Purposes

- **To** implement the **evaluation logic** for a requirement.
- **To** access the `HttpContext`, `ClaimsPrincipal`, resource, and DI services to make decisions.
- **To** support **multiple handlers per requirement**, enabling different sources of truth.
- **To** enable **resource-based** authorization by evaluating the requirement against a specific resource.
- **To** make authorization logic **testable** and **reusable** across policies.

### Syntax Rules and Structure

```csharp
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        if (/* condition */)
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

// Registration
builder.Services.AddScoped<IAuthorizationHandler, MinimumAgeHandler>();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AuthorizationHandler<TRequirement>` | Base class for handlers. |
| `HandleRequirementAsync` | Override; evaluate the requirement. |
| `context.Succeed(requirement)` | Marks the requirement as satisfied. |
| `context.Fail()` | Marks the requirement as failed (optional; use sparingly). |
| Registration | `AddScoped<IAuthorizationHandler, THandler>()`. |

**Syntax rules:**

- **Handlers must be registered in DI**—individually (`AddScoped`) or via `AddAuthorization` conventions.
- **Handlers can call `context.Succeed` for multiple requirements** they evaluate.
- **A handler that does not call `Succeed` is neutral**—another handler may still succeed the requirement.
- **Handlers should be Scoped** if they depend on Scoped services (e.g., `DbContext`).

**Constraints and limitations:**

- **Handlers run for all requirements they implement**—be careful when a handler evaluates multiple requirements.
- **`context.Fail()` prevents success even if other handlers succeed**—use only when failure is definitive.
- **Handlers must not throw**—an exception in a handler causes authorization to fail with an error.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Handler with DI (Database-Backed Permission Check)

```csharp
public class PermissionRequirement : IAuthorizationRequirement
{
    public string Permission { get; }
    public PermissionRequirement(string permission) => Permission = permission;
}

public class PermissionHandler : AuthorizationHandler<PermissionRequirement>
{
    private readonly IPermissionService _permissions;

    public PermissionHandler(IPermissionService permissions)
        => _permissions = permissions;

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PermissionRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId is null) return;

        var hasPermission = await _permissions.HasPermissionAsync(
            userId, requirement.Permission);

        if (hasPermission)
            context.Succeed(requirement);
    }
}

// Registration
builder.Services.AddScoped<IPermissionService, PermissionService>();
builder.Services.AddScoped<IAuthorizationHandler, PermissionHandler>();
```

**Expected Output:** The handler queries the permission service to determine whether the user has the required permission. If so, the requirement succeeds.

**Why this output?** The handler uses constructor injection to access `IPermissionService`; it evaluates the requirement by checking the database.

#### Example 2: Resource-Based Handler

```csharp
public class DocumentAuthorizationHandler
    : AuthorizationHandler<SameTenantRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameTenantRequirement requirement,
        Document resource)
    {
        var tenant = context.User.FindFirst("tenant")?.Value;
        if (tenant == resource.TenantId)
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

// Usage in a controller
var result = await _auth.AuthorizeAsync(User, document, "SameTenant");
if (!result.Succeeded) return Forbid();
```

**Expected Output:** The user can access the document only if their `tenant` claim matches the document's `TenantId`.

**Why this output?** The handler receives both the requirement and the resource, enabling per-item authorization checks.

### Real-World Cases

- **Database-backed permissions:** A handler queries a permission table to check fine-grained permissions.
- **Feature flags:** A handler checks whether a feature is enabled for the user's subscription tier.
- **Resource ownership:** A handler compares the user's ID to the resource's owner ID.
- **Time-based access:** A handler checks whether the current time is within allowed hours.

### References

- AuthorizationHandler<TRequirement> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1
- Resource-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Dependency injection in requirement handlers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## 5. Policy Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Policy best practices are guidelines for creating, registering, and evaluating authorization policies that are secure, maintainable, and testable. |
| **Technical Definition** | Best practices encompass **secure-by-default fallback policies**, **policy-based (not role-based) fine-grained authorization**, **resource-based authorization for per-item checks**, **Scoped handler registration for DI**, **avoiding `context.Fail()` unless definitive**, **testing requirements and handlers in isolation**, and **using `IAuthorizationPolicyProvider` for dynamic policies**. |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for policy-based authorization—they help you avoid leaving endpoints accidentally public, creating policies that are impossible to test, and writing handlers that break when services change. |

### Purposes

- **To** ensure **secure-by-default** behavior so new endpoints are not accidentally public.
- **To** keep authorization logic **testable** and **decoupled from controllers**.
- **To** use the right **handler lifetime** (Scoped for DI-dependent handlers).
- **To** avoid **common pitfalls** like `Fail()` misuse and missing handler registration.
- **To** support **dynamic policies** when static registration is impractical.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Secure by default** | Configure a fallback policy requiring authentication. | Rely on developers to remember `[Authorize]` on every endpoint. |
| **Policy over roles** | Use policies with requirements and handlers for complex rules. | Encode complex authorization in role strings. |
| **Handler registration** | Register handlers as Scoped when they depend on Scoped services. | Register handlers as Singleton and capture Scoped services. |
| **`context.Fail()`** | Use `Fail()` only when the requirement is definitively unmet. | Call `Fail()` casually—it prevents success even if another handler succeeds. |
| **Testing** | Unit test handlers with mock `AuthorizationHandlerContext` instances. | Test only through full integration tests. |
| **Resource-based checks** | Use `IAuthorizationService.AuthorizeAsync(User, resource, policy)` for per-item checks. | Assume endpoint-level policies cover per-item access. |
| **Dynamic policies** | Use `IAuthorizationPolicyProvider` for many permission permutations. | Register hundreds of static policies. |
| **Policy naming** | Use clear, descriptive names (`"AtLeast21"`, `"SameTenant"`). | Use vague names (`"Policy1"`, `"Check"`). |
| **Handler simplicity** | Keep handlers focused on one requirement type. | Create "god handlers" that evaluate many unrelated requirements. |

### Annotated Code Example (Best-Practice Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Secure-by-default fallback policy
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build())
    .AddPolicy("SameTenant", policy =>
        policy.AddRequirements(new SameTenantRequirement()))
    .AddPolicy("MinimumAge", policy =>
        policy.AddRequirements(new MinimumAgeRequirement(18)));

// 2. Scoped handler registration for DI
builder.Services.AddScoped<IAuthorizationHandler, SameTenantHandler>();
builder.Services.AddScoped<IAuthorizationHandler, MinimumAgeHandler>();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

// 3. Public endpoints opt out explicitly
app.MapGet("/health", () => "OK").AllowAnonymous();
```

**Why this follows best practices:**

- **Fallback policy** requires authentication for all endpoints by default.
- **Policies** encapsulate complex rules (`SameTenant`, `MinimumAge`) with dedicated requirements.
- **Handlers are Scoped**, allowing them to depend on per-request services.
- **`[AllowAnonymous]`** is used explicitly for public endpoints.
- **Policy names are clear and descriptive**.
- **Resource-based checks** (shown in the handler examples) complement endpoint-level policies for per-item access.

### Real-World Cases

- **Multi-tenant SaaS:** Fallback policy + same-tenant resource-based checks.
- **Financial applications:** Minimum-age and employee-status policies; strict 401/403 responses.
- **Permission-based APIs:** Dynamic policy provider generating policies from permission names.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Resource-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Custom authorization policy providers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider
- Security cards: fallback policies and authorization rules – https://github.com/reware-labs/securitycards

---

## Summary Table: Policy-Based Authorization at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Policy-based authorization** | The framework for named access rules | `AddPolicy`, `[Authorize(Policy = "...")]`, `RequireAuthorization("...")` | Every secured application |
| **Policy creation** | Registering named policies | `AddAuthorizationBuilder().AddPolicy(...)` | Defining reusable access rules |
| **Policy requirements** | Individual conditions | `IAuthorizationRequirement` | Expressing rules (age, tenant, permission) |
| **Policy handlers** | Evaluating requirements | `AuthorizationHandler<TRequirement>` | Implementing authorization logic |
| **Best practices** | Secure, testable policies | Fallback policy, Scoped handlers, resource-based checks | Every production authorization setup |

---

## References (Full List)

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Custom authorization policy providers in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider
- Resource-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Role-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles
- Claims-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- AuthorizationHandler<TRequirement> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1
- IAuthorizationRequirement interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Security cards: fallback policies and authorization rules – https://github.com/reware-labs/securitycards
- Policy-based authorization in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Custom authorization policies with IAuthorizationRequirementData – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationrequirementdata
- Dependency injection in requirement handlers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies