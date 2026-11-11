# C# Authorization — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
**Authorization** is the process of determining whether an authenticated user (or service) is **allowed to perform a specific action or access a specific resource**. Authentication answers *"Who are you?"*; authorization answers *"What are you allowed to do?"* In ASP.NET Core, authorization is handled by middleware that evaluates **policies** against the current `ClaimsPrincipal` and either allows the request to continue or rejects it with `401 Unauthorized` / `403 Forbidden`.

**Technical Definition**
ASP.NET Core authorization is built on **policies**, which combine one or more **requirements**. A policy is evaluated by one or more **handlers**, each of which decides whether the current user satisfies the requirement. The `[Authorize]` attribute triggers the authorization middleware for the decorated controller, action, or Razor Page. `[AllowAnonymous]` bypasses authorization. Authorization is applied via `UseAuthorization()` **after** `UseAuthentication()` and **before** endpoints. Policies are registered with `AddAuthorization(options => options.AddPolicy(...))` and can be applied by name (`[Authorize(Policy = "AtLeast21")]`), by role (`[Authorize(Roles = "Admin")]`), or by scheme.

**Beginner-Friendly Explanation**
Authorization is the **bouncer at the door**. Authentication already checked your ID; now the bouncer checks whether your name is on the VIP list, whether you have the right wristband, or whether you meet the dress code. In ASP.NET Core, the "VIP list" is a **policy** —a set of rules like "must be at least 21" or "must have the Admin role." You attach policies to controllers and actions with `[Authorize]`, and the framework enforces them before your code runs.

---

### Key Characteristics

- **Policy-based** — Modern authorization centers on policies composed of requirements and handlers.
- **Declarative and imperative** — Apply `[Authorize]` declaratively, or call `IAuthorizationService.AuthorizeAsync` imperatively.
- **Role- and claim-based** — Roles and claims are convenience shortcuts over policy evaluation.
- **Fail-closed** — Endpoints without `[AllowAnonymous]` require authentication by default when `FallbackPolicy` is configured.
- **Composable** — Policies combine requirements with AND; handlers decide how each requirement is satisfied.
- **Resource-based** — `AuthorizeAsync(user, resource, policy)` supports per-resource checks.
- **Scheme-aware** — Multiple authentication schemes can be used with different policies.

---

### Prerequisites

- Understanding of **authentication** (`ClaimsPrincipal`, cookies, JWT).
- Familiarity with **ASP.NET Core middleware** (`UseAuthentication`, `UseAuthorization`).
- Knowledge of **claims** and **roles**.
- Awareness of **dependency injection** (handlers are registered as services).
- Basic understanding of **`IAuthorizationService`** and **`AuthorizationHandler<TRequirement>`**.

---

### Related Programming Areas

- **Authentication** — Identity must be established before authorization runs.
- **Claims and identity** — Claims are the data authorization evaluates.
- **ASP.NET Core Identity** — Provides role and claim stores.
- **API security** — Policies protect endpoints and resources.
- **Resource-based authorization** — Per-document or per-entity access checks.

---

### Core Concepts / Features

1. Authorization (Overarching Concept)
2. `[Authorize]`
3. Policies
4. Requirements
5. Handlers
6. Role-Based Authorization
7. Claims-Based Authorization
8. Policy-Based Authorization
9. Authorization Best Practices

---

## 1. Authorization (Overarching Concept)

### Definitions

**Core Definition**
Authorization determines whether an authenticated user may access a resource or perform an action.

**Technical Definition**
ASP.NET Core authorization is provided by the `Microsoft.AspNetCore.Authorization` framework. It evaluates **policies** against the `ClaimsPrincipal` produced by authentication. The authorization middleware (`UseAuthorization`) runs after authentication and before endpoints. The `[Authorize]` attribute marks endpoints that require authorization; `[AllowAnonymous]` exempts them. Policy evaluation is delegated to registered **handlers**.

**Beginner-Friendly Explanation**
Authorization is **"are you allowed?"** The framework checks your policies and either lets the request through or returns 401/403.

---

### Purposes

- To **protect resources** from unauthorized access.
- To **enforce business rules** (age, ownership, subscription tier).
- To centralize access control in **policies and handlers**.
- To support **role- and claim-based** rules.
- To enable **resource-specific** authorization.

---

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));
});

app.UseAuthentication();
app.UseAuthorization();
```

```csharp
[Authorize(Policy = "AtLeast21")]
public IActionResult Restricted() => View();
```

**Component breakdown**

- `AddAuthorization()` — Registers authorization services and policies.
- `UseAuthorization()` — Adds the authorization middleware.
- `[Authorize]` — Requires authorization for the endpoint.

#### Syntax Rules

- `UseAuthentication()` must precede `UseAuthorization()`.
- `UseAuthorization()` must precede endpoints (`MapControllers`, `MapRazorPages`, etc.).
- `[AllowAnonymous]` overrides `[Authorize]`.

#### Constraints and Limitations

- Authorization without authentication results in `401 Unauthorized`.
- Failed authorization results in `403 Forbidden` (for authenticated users).
- Incorrect middleware order disables authorization.

---

### Annotated Code Examples and Expected Outputs

#### Example: Basic Authorization Setup

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/public", () => "Anyone can see this");
app.MapGet("/private", () => "Only authenticated users")
   .RequireAuthorization();

app.Run();
```

**Expected Output**

- `GET /public` → `200 OK` with `"Anyone can see this"`.
- `GET /private` without cookie → `401 Unauthorized`.
- `GET /private` with valid cookie → `200 OK`.

**Why this output occurs:** `RequireAuthorization()` applies the default policy, which requires an authenticated user.

---

### Real-World Cases

- **Admin dashboards** — Restricted to users with the `Admin` role.
- **APIs** — JWT-protected endpoints.
- **Document management** — Per-document ownership checks.
- **Subscription tiers** — Feature access based on claims.

---

### References

- Introduction to authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction

---

## 2. `[Authorize]`

### Definitions

**Core Definition**
`[Authorize]` is an attribute that marks a controller, action, or Razor Page as requiring authorization.

**Technical Definition**
`Microsoft.AspNetCore.Authorization.AuthorizeAttribute` triggers the authorization middleware. Without parameters, it applies the **default policy** (authenticated user). It supports `Policy = "..."`, `Roles = "..."`, and `AuthenticationSchemes = "..."`. `[AllowAnonymous]` overrides `[Authorize]` at the same or more specific scope.

**Beginner-Friendly Explanation**
`[Authorize]` is **the "members only" sign** on a controller or action. The framework checks the policy before running your code.

---

### Purposes

- To **protect endpoints** declaratively.
- To apply **policies**, **roles**, or **schemes** to controllers and actions.
- To override authorization at the action level.
- To integrate with **MVC**, **Razor Pages**, and **Minimal APIs** (via `RequireAuthorization`).

---

### Syntax Rules and Structure

```csharp
[Authorize]                                  // default policy (authenticated)
[Authorize(Roles = "Admin,Manager")]        // roles
[Authorize(Policy = "AtLeast21")]           // policy
[Authorize(AuthenticationSchemes = "Bearer")] // scheme
[AllowAnonymous]                             // bypass
```

**Component breakdown**

- `Roles` — Comma-separated roles; user must be in at least one.
- `Policy` — Named policy to evaluate.
- `AuthenticationSchemes` — Scheme(s) used to authenticate.
- `[AllowAnonymous]` — Exempts the endpoint.

#### Syntax Rules

- Controller-level `[Authorize]` applies to all actions.
- Action-level `[Authorize]` adds to or overrides controller-level rules.
- `[AllowAnonymous]` on an action overrides controller-level `[Authorize]`.

#### Constraints and Limitations

- `[Authorize]` without authentication configured results in `401`.
- Role checks are simple string matches; policies are preferred for complex rules.

---

### Annotated Code Examples and Expected Outputs

#### Example: Controller with Mixed Authorization

```csharp
[Authorize]
public class AccountController : Controller
{
    public IActionResult Profile() => View();          // requires login

    [AllowAnonymous]
    public IActionResult Login() => View();            // public

    [Authorize(Roles = "Admin")]
    public IActionResult AdminPanel() => View();       // admin only
}
```

**Expected Output**

- `Profile` without login → `401`/redirect to login.
- `Login` → accessible to anyone.
- `AdminPanel` for non-admin → `403 Forbidden`.

**Why this output occurs:** The controller-level `[Authorize]` protects all actions; `[AllowAnonymous]` exempts `Login`; `[Authorize(Roles="Admin")]` restricts `AdminPanel`.

---

### Real-World Cases

- **MVC controllers** — Protect entire areas or specific actions.
- **Razor Pages** — `[Authorize]` on the PageModel.
- **Minimal APIs** — `.RequireAuthorization("PolicyName")`.

---

### References

- Simple authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/simple
- `AuthorizeAttribute` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute

---

## 3. Policies

### Definitions

**Core Definition**
A policy is a **named collection of requirements** that must be satisfied for authorization to succeed.

**Technical Definition**
An `AuthorizationPolicy` consists of **authentication requirements** (schemes), **requirements** (e.g., `MinimumAgeRequirement`), and a **combine strategy** (`RequireAll`, `RequireAny`, `RequireUnique`). Policies are registered by name with `AddPolicy("Name", builder => builder.RequireClaim(...))` or constructed directly with `AuthorizationPolicyBuilder`. The default policy requires an authenticated user.

**Beginner-Friendly Explanation**
A policy is **a rulebook**—"must be at least 21," "must have Admin role," "must be in the UK." You give the rulebook a name and attach it to endpoints.

---

### Purposes

- To **centralize access rules** under a name.
- To **combine multiple requirements**.
- To **reuse** authorization logic across endpoints.
- To support **complex business rules**.
- To **decouple** authorization logic from controllers.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));

    options.AddPolicy("EmployeeOnly", policy =>
        policy.RequireClaim("EmployeeNumber"));

    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
});
```

**Component breakdown**

- `AddPolicy(name, builder)` — Registers a named policy.
- `Requirements.Add(...)` — Adds a custom requirement.
- `RequireClaim`, `RequireRole`, `RequireAssertion` — Built-in requirement helpers.

#### Syntax Rules

- Policies are registered at startup.
- Multiple requirements are combined with AND by default.
- `RequireAssertion` provides a lambda-based policy.

#### Constraints and Limitations

- Policy names must be unique.
- Requirements are evaluated by handlers; unregistered handlers fail.

---

### Annotated Code Examples and Expected Outputs

#### Example: Multiple Policies

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
    options.AddPolicy("UKResident", p => p.RequireClaim("Country", "UK"));
    options.AddPolicy("AdultUK", p =>
    {
        p.RequireClaim("Country", "UK");
        p.Requirements.Add(new MinimumAgeRequirement(18));
    });
});
```

**Expected Output (conceptual)**
```
// Endpoints using [Authorize(Policy="AdultUK")] require both claims and age.
```

**Why this pattern is used:** Combining requirements in a named policy makes complex rules reusable.

---

### Real-World Cases

- **Subscription tiers** — `PremiumOnly` policy requiring a `Subscription` claim.
- **Regional access** — `UKResident` policy.
- **Department-level access** — `HRDepartment` policy.

---

### References

- Policy-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## 4. Requirements

### Definitions

**Core Definition**
A requirement is a **rule that a policy can evaluate** against the user.

**Technical Definition**
An `IAuthorizationRequirement` is a marker interface for requirement classes. Requirements carry data (e.g., `MinimumAgeRequirement(21)`). They are evaluated by **handlers** (`AuthorizationHandler<TRequirement>`). A requirement can be satisfied by one or more handlers.

**Beginner-Friendly Explanation**
A requirement is **one condition**—"age ≥ 21," "country = UK." Policies combine requirements.

---

### Purposes

- To **encapsulate individual rules**.
- To carry **rule-specific data**.
- To enable **reusable** handlers.
- To support **complex, multi-step** evaluation.
- To decouple **rules** from **evaluation logic**.

---

### Syntax Rules and Structure

```csharp
public sealed class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}
```

**Component breakdown**

- `: IAuthorizationRequirement` — Marker interface.
- Properties — Carry rule data.

#### Syntax Rules

- Requirements are simple data classes.
- They are registered within policies.
- They are evaluated by handlers.

#### Constraints and Limitations

- Requirements do not contain logic; handlers do.
- A requirement must have at least one handler to be evaluated.

---

### Annotated Code Examples and Expected Outputs

#### Example: Custom Requirement

```csharp
public sealed class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}
```

**Expected Output (conceptual)**
```
// Requirement carries the minimum age; a handler evaluates it.
```

**Why this pattern is used:** Requirements are data; handlers are logic.

---

### Real-World Cases

- **Age restrictions** — `MinimumAgeRequirement`.
- **Region restrictions** — `CountryRequirement`.
- **Ownership** — `DocumentOwnerRequirement`.

---

### References

- Requirements — Policy-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies#requirements

---

## 5. Handlers

### Definitions

**Core Definition**
A handler evaluates a requirement against the user and determines whether the requirement is satisfied.

**Technical Definition**
`AuthorizationHandler<TRequirement>` implements `IAuthorizationHandler`. It overrides `HandleRequirementAsync(AuthorizationHandlerContext, TRequirement)` and calls `context.Succeed(requirement)` when the requirement is met. Handlers are registered in DI (typically as `Scoped` or `Singleton`). Multiple handlers can evaluate the same requirement.

**Beginner-Friendly Explanation**
A handler is **the judge** that reads a requirement ("age ≥ 21") and checks the user's claims to decide if it is satisfied.

---

### Purposes

- To **evaluate requirements** against the user.
- To **encapsulate evaluation logic**.
- To support **multiple handlers** per requirement.
- To enable **resource-based** checks (`AuthorizationHandler<TRequirement, TResource>`).

---

### Syntax Rules and Structure

```csharp
public sealed class MinimumAgeHandler
    : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        if (context.User.HasClaim(c => c.Type == "DateOfBirth"))
        {
            var dob = DateTime.Parse(context.User.FindFirst("DateOfBirth")!.Value);
            if (dob.AddYears(requirement.MinimumAge) <= DateTime.Today)
                context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

// Register
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

**Component breakdown**

- `AuthorizationHandler<TRequirement>` — Base class.
- `HandleRequirementAsync` — Evaluation method.
- `context.Succeed(requirement)` — Marks the requirement as satisfied.
- DI registration — Required for the handler to run.

#### Syntax Rules

- Handlers must be registered in DI.
- `context.Succeed` indicates satisfaction.
- `context.Fail` (rare) explicitly fails.
- Multiple handlers can succeed independently.

#### Constraints and Limitations

- All requirements in a policy must succeed (AND by default).
- Unregistered handlers are never invoked.

---

### Annotated Code Examples and Expected Outputs

#### Example: Handler Success and Failure

```csharp
public sealed class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
    {
        var dobClaim = context.User.FindFirst("DateOfBirth");
        if (dobClaim is null) return Task.CompletedTask;   // fail (no succeed)

        var dob = DateTime.Parse(dobClaim.Value);
        if (dob.AddYears(requirement.MinimumAge) <= DateTime.Today)
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```

**Expected Output (conceptual)**
```
// User meets age: handler succeeds → 200 OK.
// User does not meet age: handler does not succeed → 403 Forbidden.
```

**Why this output occurs:** Only `context.Succeed` satisfies the requirement; otherwise, authorization fails.

---

### Real-World Cases

- **Age-based access** — Date-of-birth handler.
- **Ownership checks** — Resource-based handler.
- **Tenant isolation** — Tenant claim handler.

---

### References

- Handlers — Policy-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies#authorization-handlers

---

## 6. Role-Based Authorization

### Definitions

**Core Definition**
Role-based authorization grants or denies access based on the user’s **role claims**.

**Technical Definition**
Roles are claims of type `ClaimTypes.Role`. The `[Authorize(Roles = "Admin,Manager")]` attribute requires the user to have at least one of the listed roles. For Razor Pages, `[Authorize(Roles = "...")]` is applied to the PageModel. Roles are a simple string-matching convenience over policy evaluation.

**Beginner-Friendly Explanation**
Role-based authorization is **"are you in this group?"**—Admin, Manager, Customer.

---

### Purposes

- To **grant access by group membership**.
- To implement **simple, coarse-grained rules**.
- To integrate with **ASP.NET Core Identity** roles.
- To compose roles with policies.

---

### Syntax Rules and Structure

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminPanel() => View();

[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports() => View();
```

```csharp
// Policy equivalent
options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
```

**Component breakdown**

- `Roles = "Admin,Manager"` — User must have at least one role.
- `RequireRole("Admin")` — Policy-based equivalent.

#### Syntax Rules

- Roles are comma-separated in the attribute.
- The user must have at least one listed role.
- Roles are claims of type `ClaimTypes.Role`.

#### Constraints and Limitations

- Role checks are string-based; use policies for complex rules.
- Role names are case-sensitive by default.

---

### Annotated Code Examples and Expected Outputs

#### Example: Role Restriction

```csharp
[Authorize(Roles = "Admin")]
[ApiController]
[Route("api/[controller]")]
public class AdminController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Admin data");
}
```

**Expected Output**

- User with `Admin` role → `200 OK` with `"Admin data"`.
- Authenticated user without `Admin` → `403 Forbidden`.
- Unauthenticated user → `401 Unauthorized`.

**Why this output occurs:** The `Roles` property requires the `Admin` role claim.

---

### Real-World Cases

- **Admin panels** — `Admin` role.
- **Management areas** — `Manager` role.
- **Support tools** — `Support` role.

---

### References

- Role-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles

---

## 7. Claims-Based Authorization

### Definitions

**Core Definition**
Claims-based authorization grants access based on **specific claims** (key-value assertions) about the user.

**Technical Definition**
Claims are `Claim` objects (Type + Value) inside the user's identity. Policies can `RequireClaim(type)`, `RequireClaim(type, value)`, or `RequireClaim(type, allowedValues)`. Claims are more flexible than roles because they carry arbitrary data (department, country, subscription tier, date of birth).

**Beginner-Friendly Explanation**
Claims-based authorization is **"do you have this fact about you?"**—a claim like `Country=UK` or `Subscription=Premium`.

---

### Purposes

- To **authorize based on user attributes**.
- To support **fine-grained rules** (country, department, tier).
- To carry **richer information** than roles.
- To enable **dynamic, data-driven** policies.

---

### Syntax Rules and Structure

```csharp
options.AddPolicy("UKResident", p => p.RequireClaim("Country", "UK"));
options.AddPolicy("EmployeeOnly", p => p.RequireClaim("EmployeeNumber"));
options.AddPolicy("Tiered", p =>
    p.RequireClaim("Subscription", "Premium", "Enterprise"));
```

**Component breakdown**

- `RequireClaim("Country", "UK")` — Claim type + allowed values.
- `RequireClaim("EmployeeNumber")` — Claim type only (any value).

#### Syntax Rules

- Claim types are strings; constants are recommended.
- Multiple allowed values are OR-ed.
- Multiple `RequireClaim` calls are AND-ed.

#### Constraints and Limitations

- Claims must be issued at authentication time or loaded dynamically.
- Claim types must be consistent across issuers.

---

### Annotated Code Examples and Expected Outputs

#### Example: Claim-Based Policy

```csharp
options.AddPolicy("UKResident", p =>
    p.RequireClaim("Country", "UK"));

[Authorize(Policy = "UKResident")]
public IActionResult UkOnly() => View();
```

**Expected Output**

- User with `Country=UK` claim → `200 OK`.
- User without the claim → `403 Forbidden`.

**Why this output occurs:** `RequireClaim` checks the claim type and value.

---

### Real-World Cases

- **Geographic restrictions** — Country claims.
- **Subscription tiers** — Subscription claims.
- **Department access** — Department claims.

---

### References

- Claims-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims

---

## 8. Policy-Based Authorization

### Definitions

**Core Definition**
Policy-based authorization evaluates a **named set of requirements** against the user using registered handlers.

**Technical Definition**
Policies combine requirements and handlers. They can be applied declaratively (`[Authorize(Policy = "...")]`) or imperatively (`IAuthorizationService.AuthorizeAsync`). Resource-based authorization uses `AuthorizeAsync(user, resource, policy)`. Policies are the **recommended approach** for complex authorization because they are reusable, testable, and centralized.

**Beginner-Friendly Explanation**
Policy-based authorization is **the full rulebook**: it can combine roles, claims, and custom logic into a single named rule you attach to endpoints.

---

### Purposes

- To **centralize complex rules**.
- To **combine roles, claims, and custom logic**.
- To support **resource-based** authorization.
- To **test** authorization logic in isolation.
- To **reuse** rules across endpoints.

---

### Syntax Rules and Structure

```csharp
// Declarative
[Authorize(Policy = "AtLeast21")]
public IActionResult Restricted() => View();

// Imperative
var result = await _authorizationService.AuthorizeAsync(User, "AtLeast21");
if (!result.Succeeded) return Forbid();

// Resource-based
var result = await _authorizationService.AuthorizeAsync(User, document, "DocumentOwner");
```

**Component breakdown**

- `AuthorizeAsync(user, policyName)` — Imperative check.
- `AuthorizeAsync(user, resource, policyName)` — Resource-based check.
- `result.Succeeded` — Boolean outcome.

#### Syntax Rules

- Policies are registered by name.
- Resource-based handlers implement `AuthorizationHandler<TRequirement, TResource>`.
- `Forbid()` returns `403`; `Challenge()` returns `401`.

#### Constraints and Limitations

- Imperative checks must be performed manually in code.
- Resource-based handlers require the resource type.

---

### Annotated Code Examples and Expected Outputs

#### Example: Resource-Based Authorization

```csharp
public sealed class DocumentOwnerRequirement : IAuthorizationRequirement { }

public sealed class DocumentOwnerHandler
    : AuthorizationHandler<DocumentOwnerRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        DocumentOwnerRequirement requirement,
        Document resource)
    {
        if (resource.OwnerId == context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}

// Usage
var result = await _authorizationService.AuthorizeAsync(User, document, "DocumentOwner");
if (!result.Succeeded) return Forbid();
```

**Expected Output**

- Owner accessing document → allowed.
- Non-owner → `403 Forbidden`.

**Why this pattern is used:** The handler evaluates the requirement against the specific resource, not just the user.

---

### Real-World Cases

- **Document management** — Owner-only access.
- **Multi-tenant SaaS** — Tenant-specific resources.
- **E-commerce** — Order ownership checks.

---

### References

- Policy-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Resource-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased

---

## 9. Authorization Best Practices

### Definitions

**Core Definition**
Authorization best practices are the guidelines for implementing secure, maintainable, and testable access control.

**Technical Definition**
Best practices derive from Microsoft's authorization guidance: **use policies over roles** for complex rules; **prefer policy-based authorization** over imperative checks; **enforce HTTPS**; **apply `[Authorize]` by default** with a fallback policy; **use `[AllowAnonymous]` sparingly**; **test policies and handlers**; **avoid hard-coded role strings**; **use resource-based authorization** for per-resource access; and **keep handlers small and focused**.

**Beginner-Friendly Explanation**
The best practices are: **use policies**, **protect by default**, **test your rules**, **don't hard-code roles**, and **check per-resource access**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Fallback policy (secure by default)
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// 2. Policies over inline roles
options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));

// 3. Resource-based checks
var result = await _authorizationService.AuthorizeAsync(User, doc, "DocumentOwner");

// 4. Test policies and handlers
// Use unit tests with AuthorizationHandlerContext.

// 5. HTTPS enforcement
app.UseHttpsRedirection();
```

**Component breakdown**

- **Fallback policy** — Protects endpoints by default.
- **Policies** — Centralized, reusable rules.
- **Resource-based** — Per-resource checks.
- **Testing** — Unit-test handlers with `AuthorizationHandlerContext`.
- **HTTPS** — Prevents credential interception.

#### Rules and Constraints

- **Do** use policies for complex rules.
- **Do** configure a fallback policy to secure by default.
- **Do** use resource-based authorization for per-resource access.
- **Do** test policies and handlers.
- **Do not** hard-code role strings across the codebase.
- **Do not** use `[AllowAnonymous]` broadly.
- **Do not** rely on client-side checks for security.

---

### Annotated Code Examples and Expected Outputs

#### Example: Secure-by-Default Configuration

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();

    options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
});

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

**Expected Output (conceptual)**
```
// All endpoints require authentication by default.
// Endpoints with [AllowAnonymous] remain public.
// [Authorize(Policy = "AdminOnly")] restricts to admins.
```

**Why this configuration is used:** The fallback policy protects new endpoints automatically, preventing accidental exposure.

---

### Real-World Cases

- **Enterprise APIs** — Fallback policy + explicit policies.
- **Document systems** — Resource-based authorization.
- **Multi-tenant SaaS** — Tenant claim policies.

---

### References

- Introduction to authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction
- Policy-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Role-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles
- Claims-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- Resource-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Role-based checks everywhere | **Discouraged**; prefer policies for complex rules. |
| `[AllowAnonymous]` broadly | **Risky**; use only for public endpoints. |
| Fallback policy | **Recommended** for secure-by-default apps. |
| Imperative `AuthorizeAsync` | **Supported**; use for resource-based checks. |
| Resource-based handlers | **Recommended** for per-resource access. |
| `IAuthorizationHandler` registration | **Required**; unregistered handlers are never invoked. |
| Client-side authorization | **Not a security boundary**; always enforce server-side. |

---

## References (Consolidated)

- Introduction to authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction
- Simple authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/simple
- Policy-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Role-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles
- Claims-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- Resource-based authorization in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- `AuthorizeAttribute` — .NET API — https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute
- Requirements — Policy-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies#requirements
- Handlers — Policy-based authorization — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies#authorization-handlers

---