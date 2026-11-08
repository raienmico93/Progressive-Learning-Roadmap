# ASP.NET Core Authorization: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and its authorization stack: `[Authorize]`, `[AllowAnonymous]`, roles, claims, policies, requirements, and handlers. Authorization is distinct from authentication: authentication determines **who** the user is; authorization determines **what** they are allowed to do. In ASP.NET Core, authorization is enforced by the `AuthorizationMiddleware` (`UseAuthorization`), which evaluates the `ClaimsPrincipal` populated by authentication against policies. **Policy-based authorization is the recommended approach**; role and claim checks are convenience shortcuts built on top of policies. **Minimal APIs and controllers are unified** under endpoint routing, so `RequireAuthorization()` works for both. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization is the process of determining whether an authenticated user is permitted to access a specific resource or perform a specific action. |
| **Technical Definition** | ASP.NET Core authorization is policy-based. The `AuthorizationMiddleware` invokes the `IAuthorizationService` for each endpoint with an authorization policy. A policy consists of one or more **requirements** (`IAuthorizationRequirement`), each evaluated by one or more **handlers** (`AuthorizationHandler<TRequirement>`). If any handler marks a requirement as succeeded (and none fail), the policy succeeds. The framework provides `[Authorize]`, `[AllowAnonymous]`, role checks, and claim checks as built-in conveniences that ultimately map to policies. |
| **Beginner-Friendly Explanation** | Authorization is the **"Are you allowed?"** step. After authentication tells the app **who** you are, authorization decides **what you can do**. Think of a nightclub: authentication is the ID check at the door; authorization is the bouncer deciding whether your name is on the VIP list. Policies are the VIP lists—rules that say "admins can access this," "users with a premium subscription can access that," or "anyone with a tenant claim matching the resource's tenant can access it." |

### Key Characteristics

- **Policy-based** – All authorization decisions are ultimately expressed as policies; roles and claims are conveniences layered on top.
- **Requirement/handler model** – A policy is a collection of requirements; each requirement is evaluated by one or more handlers.
- **Endpoint-aware** – `UseAuthorization()` must run between `UseRouting()` and endpoint mapping; analyzer **ASP0001** enforces this.
- **Fallback policy** – A fallback policy can require authentication for **all endpoints by default**, providing secure-by-default behavior.
- **Multiple schemes** – `[Authorize(AuthenticationSchemes = ...)]` selects which authentication scheme(s) to use for a given endpoint.
- **Fail-closed** – If no handler succeeds for a requirement, authorization fails (403 for authenticated users; 401 for unauthenticated).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **authentication** and **claims-based identity**.
- Familiarity with **middleware ordering** (`UseAuthentication` before `UseAuthorization`).
- Basic understanding of **dependency injection** (for handler registration).

### Related Programming Areas

- **Authentication** – Produces the `ClaimsPrincipal` that authorization evaluates.
- **Claims** – The attributes (roles, tenant, subscription) that policies check.
- **Middleware** – `UseAuthorization()` invokes the authorization middleware.
- **Minimal APIs** – `RequireAuthorization()` applies policies to endpoint handlers.
- **Resource-based authorization** – `IAuthorizationService.AuthorizeAsync` with a resource for per-item checks.

### Core Concepts / Features

1. **Authorization** – The overall policy-based system.
2. **`[Authorize]`** – The attribute that applies policies to controllers and actions.
3. **`[AllowAnonymous]`** – The attribute that bypasses authorization.
4. **Roles** – Role-based checks (`[Authorize(Roles = "...")]`).
5. **Claims** – Claim-based checks (`[Authorize(Policy = "...")]` or `RequireClaim`).
6. **Policies** – Named combinations of requirements.
7. **Requirements** – The assertions a policy evaluates.
8. **Handlers** – The code that evaluates requirements.
9. **Authorization best practices** – Secure defaults, fallback policies, resource-based checks, and testing.

---

## 1. Authorization (The Policy-Based System)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization is the process of evaluating whether an authenticated user is permitted to access a resource or perform an action. |
| **Technical Definition** | The `AuthorizationMiddleware` (`UseAuthorization`) evaluates the selected endpoint’s authorization metadata against `HttpContext.User`. The `IAuthorizationService` evaluates policies, which are collections of `IAuthorizationRequirement` objects. Each requirement is evaluated by one or more `AuthorizationHandler<TRequirement>` implementations. A policy succeeds if **all** requirements succeed. The framework’s `AuthorizeFilter` (MVC) and endpoint metadata (minimal APIs) feed into this pipeline. |
| **Beginner-Friendly Explanation** | Authorization is the **rulebook for access**. Each endpoint has rules attached to it—sometimes simple ("must be logged in"), sometimes complex ("must be an admin in the same tenant as the resource"). The authorization middleware checks the user against those rules before the endpoint executes. If the rules pass, the request continues; if not, the user gets a 401 or 403. |

### Purposes

- **To** enforce **access control** consistently across controllers, Razor Pages, and minimal APIs.
- **To** support **fine-grained, policy-based decisions** that go beyond simple role checks.
- **To** enable **secure-by-default** configurations via fallback policies.
- **To** provide **standardized responses** (401 for unauthenticated, 403 for forbidden) without ad-hoc logic in each action.
- **To** allow **resource-based** authorization (per-item checks) through `IAuthorizationService`.

### Syntax Rules and Structure

#### General Syntax (Middleware Registration)

```csharp
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();   // Between UseRouting and endpoint mapping
app.MapControllers();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthorization()` | Registers authorization services. |
| `UseAuthentication()` | Populates `HttpContext.User`. Must precede authorization. |
| `UseAuthorization()` | Evaluates policies for the selected endpoint. Must be between routing and endpoints. |

#### General Syntax (Fallback Policy)

```csharp
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SetFallbackPolicy` | Applies a policy to endpoints **without explicit authorization metadata**. |
| `RequireAuthenticatedUser()` | Requires an authenticated user. |

**Syntax rules:**

- **`UseAuthentication()` must precede `UseAuthorization()`**.
- **`UseAuthorization()` must be called between `UseRouting()` and endpoint mapping** (ASP0001).
- **`RequireAuthorization()`** on minimal API endpoints and `[Authorize]` on controllers both feed into the same authorization middleware.
- **The fallback policy** applies to endpoints without `[Authorize]`/`RequireAuthorization()`, but endpoints with `[AllowAnonymous]` opt out.

**Constraints and limitations:**

- **Authorization alone does not authenticate**—the user must be identified by authentication middleware first.
- **Authorization is not data filtering**—it controls access to endpoints, not which rows a user can see (use resource-based authorization or query filters for that).
- **Middleware ordering is security-critical**—reversed order leads to unauthenticated users bypassing policies.

### Annotated Complete Step-by-Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication()
    .AddCookie();
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();   // Populates User
app.UseAuthorization();    // Enforces fallback + endpoint policies
app.MapControllers();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` and `AddCookie` register authentication; the user is identified per request.
2. `AddAuthorizationBuilder().SetFallbackPolicy(...)` requires authentication for **all endpoints** without explicit authorization metadata.
3. `UseAuthentication()` populates `HttpContext.User`.
4. `UseAuthorization()` evaluates the fallback policy or the endpoint’s explicit policy.
5. Controllers are mapped **after** authorization.

**Expected Output:** An unauthenticated request to **any** endpoint returns a challenge (401/redirect to login), unless the endpoint has `[AllowAnonymous]`. Authenticated requests proceed if their policies pass.

**Why this output?** The fallback policy is applied to endpoints without explicit authorization metadata, providing secure-by-default behavior. Explicit policies on endpoints override the fallback.

### Real-World Cases

- **Public APIs with mixed access levels:** Fallback policy requires authentication globally; `[AllowAnonymous]` on public endpoints.
- **Multi-tenant SaaS:** Policies require a matching `tenant` claim between the user and the resource.
- **Enterprise apps:** Policies combine authentication requirements with role and claim checks for sensitive operations.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Minimal API authorization – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/security

---

## 2. `[Authorize]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[Authorize]` is an attribute that applies an authorization policy to a controller, action method, or Razor Page. |
| **Technical Definition** | `AuthorizeAttribute` implements `IAuthorizeData` and is consumed by the `AuthorizationMiddleware` (via `AuthorizeFilter` in MVC). When applied without arguments, it requires an authenticated user. It supports `Policy` (named policy), `Roles` (comma-separated role names), and `AuthenticationSchemes` (comma-separated scheme names). Multiple `[Authorize]` attributes on the same action are **combined**: all policies must pass. |
| **Beginner-Friendly Explanation** | `[Authorize]` is a **“members only” sign** on a controller or action. Put it on a controller to require login for everything inside it, or on a single action to protect just that action. You can add conditions—`[Authorize(Roles = "Admin")]` for admin-only access, or `[Authorize(Policy = "Over18")]` for a custom rule. |

### Purposes

- **To** require an authenticated user for a controller or action.
- **To** apply a **named policy** (`Policy = "..."`) to enforce complex rules.
- **To** require **specific roles** (`Roles = "Admin,Manager"`).
- **To** select specific **authentication schemes** (`AuthenticationSchemes = "Bearer"`) for APIs.
- **To** support **multiple attributes** that combine their requirements.

### Syntax Rules and Structure

```csharp
// Require authentication
[Authorize]
public class DashboardController : Controller { ... }

// Require a specific role
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly() => View();

// Require a named policy
[Authorize(Policy = "MinimumAge")]
public IActionResult AdultsOnly() => View();

// Select an authentication scheme
[Authorize(AuthenticationSchemes = "Bearer")]
public IActionResult ApiEndpoint() => Ok();

// Combine: both policies must pass
[Authorize(Policy = "EmployeeOnly")]
[Authorize(Policy = "MinimumAge")]
public IActionResult Sensitive() => View();
```

**Component breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `Policy` | The named policy to evaluate. |
| `Roles` | Comma-separated roles; the user must be in **at least one** (roles are OR-ed within a single attribute). |
| `AuthenticationSchemes` | Comma-separated schemes; overrides the default scheme for this endpoint. |

**Syntax rules:**

- **Multiple `[Authorize]` attributes combine** (AND semantics across policies).
- **Roles within a single attribute** are OR-ed—`[Authorize(Roles = "Admin,Manager")]` allows either role.
- **Without parameters**, `[Authorize]` only requires authentication.
- **`[Authorize]` is ignored when `[AllowAnonymous]` is also present** on the same action.

**Constraints and limitations:**

- **`[Authorize]` on a controller applies to all actions** in that controller; `[AllowAnonymous]` on specific actions overrides it.
- **Role checks are less flexible than policies**—prefer policies for anything beyond simple role membership.
- **Custom role names are case-sensitive** in the `IsInRole` check by default.

### Annotated Complete Step-by-Step Code Example

```csharp
[Authorize]   // Requires authentication for the whole controller
public class ReportsController : Controller
{
    public IActionResult Index() => View();   // Any authenticated user

    [Authorize(Roles = "Admin")]
    public IActionResult AdminReports() => View();   // Admin only

    [Authorize(Policy = "MinimumAge")]
    public IActionResult FinancialReports() => View();   // Custom policy

    [AllowAnonymous]
    public IActionResult PublicStats() => View();   // Bypasses the controller [Authorize]
}
```

**Expected Output:** An unauthenticated user is challenged for all actions except `PublicStats`. An authenticated non-admin user is forbidden from `AdminReports`. A user failing the `MinimumAge` policy is forbidden from `FinancialReports`.

**Why this output?** The controller-level `[Authorize]` requires authentication; action-level attributes add role and policy requirements; `[AllowAnonymous]` opts out of the controller requirement.

### Real-World Cases

- **Administrative areas:** `[Authorize(Roles = "Admin")]` on admin controllers.
- **API endpoints:** `[Authorize(AuthenticationSchemes = "Bearer")]` to require JWT authentication.
- **Mixed policies:** `[Authorize(Policy = "TenantMatch")]` combined with `[Authorize(Policy = "MinimumAge")]`.

### References

- AuthorizeAttribute class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute
- Policy-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## 3. `[AllowAnonymous]`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `[AllowAnonymous]` is an attribute that bypasses authorization for a controller, action method, or Razor Page. |
| **Technical Definition** | `AllowAnonymousAttribute` implements `IAllowAnonymous`. When present on an endpoint, the `AuthorizationMiddleware` skips authorization entirely—even if `[Authorize]` is present at a higher scope or the fallback policy is configured. It is the explicit opt-out mechanism for public endpoints. |
| **Beginner-Friendly Explanation** | `[AllowAnonymous]` is a **“public access” sign**. Even if a controller requires login, you can mark a specific action with `[AllowAnonymous]` to let anyone use it—like a public contact form or health check inside an otherwise-protected area. |

### Purposes

- **To** **opt out** of authorization for specific public endpoints.
- **To** override a controller-level `[Authorize]` attribute or the fallback policy on a per-action basis.
- **To** protect **health check, login, and registration endpoints** from requiring authentication.
- **To** make public API endpoints accessible without credentials while keeping the rest of the application locked down.

### Syntax Rules and Structure

```csharp
[AllowAnonymous]
public IActionResult Public() => View();

// On a Razor Page
[AllowAnonymous]
public class IndexModel : PageModel { ... }
```

**Syntax rules:**

- **`[AllowAnonymous]` wins over `[Authorize]`** on the same endpoint.
- **It must be applied to the exact endpoint** you want to be public—it does not affect other actions in the controller.
- **It bypasses the fallback policy**—an endpoint marked `[AllowAnonymous]` is accessible even if the fallback policy requires authentication.

**Constraints and limitations:**

- **Use sparingly**—every `[AllowAnonymous]` endpoint is a public surface. Review them during security audits.
- **It does not bypass authentication middleware**—it only affects authorization. Authentication still runs and populates `User` if credentials are provided.
- **It does not disable other middleware** (CORS, rate limiting, etc.).

### Annotated Complete Step-by-Step Code Example

```csharp
[Authorize]   // Everything in this controller requires login
public class AccountController : Controller
{
    [AllowAnonymous]
    public IActionResult Login() => View();   // Public

    [AllowAnonymous]
    public IActionResult Register() => View();   // Public

    public IActionResult Profile() => View();   // Requires login
}
```

**Expected Output:** `Login` and `Register` are accessible without authentication; `Profile` requires a logged-in user.

**Why this output?** `[AllowAnonymous]` on `Login` and `Register` overrides the controller-level `[Authorize]`, making them public.

### Real-World Cases

- **Login and registration pages:** Public access even in fully secured apps.
- **Health checks:** `/health` and `/metrics` endpoints accessible without credentials.
- **Webhook endpoints:** Receiving calls from external services that cannot authenticate with cookies.

### References

- AllowAnonymousAttribute class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute
- Simple authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/simple

---

## 4. Roles

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Roles are named labels (e.g., “Admin”, “Manager”, “User”) assigned to users, used for coarse-grained authorization checks. |
| **Technical Definition** | Role membership is represented by role claims (`ClaimTypes.Role`) in the user’s `ClaimsPrincipal`. The `[Authorize(Roles = "...")]` attribute creates a `RolesAuthorizationRequirement` that is evaluated by `RolesAuthorizationHandler`, which checks whether the user’s `ClaimsPrincipal` has any of the specified role claims. Identity’s `UserClaimsPrincipalFactory<TUser, TRole>` includes role claims when `AddRoles<TRole>` is registered. |
| **Beginner-Friendly Explanation** | Roles are **job titles** for users. If you give Alice the “Admin” role, then `[Authorize(Roles = "Admin")]` lets her in. Roles are simple and convenient, but they are coarse-grained—if you need “admins in tenant X” or “users with premium subscriptions,” use claims and policies instead. |

### Purposes

- **To** provide **simple, coarse-grained** access control based on group membership.
- **To** support **`[Authorize(Roles = "...")]`** checks without custom policy code.
- **To** enable **role claims** in the `ClaimsPrincipal` for `User.IsInRole(...)` checks.
- **To** serve as a convenient baseline before moving to more granular policy-based authorization.

### Syntax Rules and Structure

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly() => View();

// Multiple roles (OR semantics)
[Authorize(Roles = "Admin,Manager")]
public IActionResult Elevated() => View();
```

**Syntax rules:**

- **Roles within a single attribute are OR-ed**—the user must have **at least one** of the listed roles.
- **Role names are case-sensitive** by default in `IsInRole` checks.
- **Multiple `[Authorize]` attributes with roles are AND-ed**—the user must satisfy each attribute’s role requirement.

**Constraints and limitations:**

- **Roles are coarse-grained**—they do not express conditions like “same tenant” or “subscription tier.”
- **Role claims must be present in the `ClaimsPrincipal`**—if the authentication scheme does not issue role claims, role checks will fail.
- **Role-based authorization is built on policies**—for complex scenarios, create a policy with a custom requirement instead of relying on role strings.

### Annotated Complete Step-by-Step Code Example

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Index() => View();
}

// Program.cs / Identity setup
builder.Services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();

// Assign a role
await _userManager.AddToRoleAsync(user, "Admin");
```

**Expected Output:** Only users with the `Admin` role claim can access `AdminController`; others receive 403 Forbidden.

**Why this output?** `AddRoles<IdentityRole>` makes the `ClaimsPrincipal` factory include role claims, and `[Authorize(Roles = "Admin")]` evaluates them.

### Real-World Cases

- **Admin areas:** `[Authorize(Roles = "Admin")]` for administrative pages.
- **Tiered access:** `[Authorize(Roles = "Premium,Enterprise")]` for premium features.
- **Back-office tools:** `[Authorize(Roles = "Staff")]` for internal tools.

### References

- Role-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles
- RolesAuthorizationRequirement – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.rolesauthorizationrequirement

---

## 5. Claims

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Claims are key-value assertions about a user (e.g., `tenant: acme`, `subscription: premium`, `age: 25`) that authorization can evaluate. |
| **Technical Definition** | Claims are stored in the user’s `ClaimsIdentity`. Policies can require claims (`RequireClaim`) or use them in custom requirements. The `ClaimsAuthorizationRequirement` evaluates whether the user has a specific claim type and (optionally) one of a set of allowed values. Claims are the foundation of fine-grained authorization—roles are just a specific claim type (`ClaimTypes.Role`). |
| **Beginner-Friendly Explanation** | Claims are **facts about the user** used for access decisions. “Alice’s tenant is acme” is a claim. “Bob’s subscription is premium” is another. Policies can say “only users with a tenant claim matching the resource’s tenant can access this” or “only users with a premium subscription can use this feature.” Claims give you **finer control than roles**. |

### Purposes

- **To** provide **fine-grained attributes** for authorization decisions (tenant, subscription, department, age).
- **To** support **policy-based authorization** via `RequireClaim` and custom requirements.
- **To** enable **resource-based authorization** where the user’s claims are compared to the resource’s properties.
- **To** replace or supplement role checks when roles are too coarse-grained.

### Syntax Rules and Structure

```csharp
// Require a specific claim type
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("EmployeeOnly", policy =>
        policy.RequireClaim("employee_id"));
});

// Require a specific claim value
options.AddPolicy("AcmeTenant", policy =>
    policy.RequireClaim("tenant", "acme"));

// Require one of several values
options.AddPolicy("PremiumOrEnterprise", policy =>
    policy.RequireClaim("subscription", "premium", "enterprise"));

// Check in code
[Authorize(Policy = "AcmeTenant")]
public IActionResult AcmeDashboard() => View();
```

**Syntax rules:**

- **`RequireClaim(type)`** requires the claim to be present with any value.
- **`RequireClaim(type, values...)`** requires the claim to be present with one of the specified values.
- **Claims are case-sensitive** in their values; claim types are compared as strings.
- **`[Authorize(Policy = "...")]`** applies the claim requirement to the endpoint.

**Constraints and limitations:**

- **Claims must be in the `ClaimsPrincipal`**—they must be issued by the authentication scheme or added by Identity/claims factory.
- **Claims are not encrypted**—do not store secrets in claims.
- **Large claim sets** can exceed cookie size; keep claims minimal.

### Annotated Complete Step-by-Step Code Example

```csharp
// 1. Configure the policy
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SameTenant", policy =>
        policy.RequireClaim("tenant"));
});

// 2. Apply the policy
[Authorize(Policy = "SameTenant")]
public IActionResult TenantDashboard() => View();

// 3. Add the claim at sign-in (Identity example)
await _userManager.AddClaimAsync(user, new Claim("tenant", "acme"));
```

**Expected Output:** Users with a `tenant` claim can access `TenantDashboard`; users without the claim receive 403.

**Why this output?** `RequireClaim("tenant")` requires the claim’s presence. For matching the claim to a specific resource’s tenant, use a custom requirement or resource-based authorization (see below).

### Real-World Cases

- **Multi-tenant SaaS:** Require a `tenant` claim; compare it to the resource’s tenant in a custom requirement.
- **Subscription tiers:** Require `subscription=premium` for premium features.
- **Department-based access:** Require `department=finance` for financial reports.

### References

- Claims-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- ClaimsAuthorizationRequirement – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.claims

---

## 6. Policies

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A policy is a named collection of requirements that together define a set of conditions a user must satisfy to access a resource. |
| **Technical Definition** | An `AuthorizationPolicy` is a collection of `IAuthorizationRequirement` objects. Policies are registered via `AuthorizationOptions.AddPolicy(name, policy)` or `AddPolicy` in `AuthorizationBuilder`. The `IAuthorizationService` evaluates the policy by running all registered handlers for each requirement; a policy succeeds if **all** requirements succeed. Policies can be applied via `[Authorize(Policy = "...")]`, `RequireAuthorization("...")`, or `IAuthorizationService.AuthorizeAsync`. |
| **Beginner-Friendly Explanation** | A policy is a **named rulebook** for access. You create a policy called "SameTenant" that says "the user must have a tenant claim matching the resource’s tenant," or a policy called "MinimumAge" that says "the user must be at least 18." Then you apply the policy with `[Authorize(Policy = "SameTenant")]`. Policies can combine multiple requirements—for example, "authenticated AND employee AND over 18." |

### Purposes

- **To** encapsulate **complex authorization logic** in a reusable, named rule.
- **To** combine **multiple requirements** (authentication, claims, custom checks) in a single policy.
- **To** decouple **authorization logic** from controllers and endpoints, improving testability.
- **To** support **resource-based** authorization through `IAuthorizationService`.
- **To** provide a **clear, discoverable authorization model** that can be documented and audited.

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthorization(options =>
{
    // Simple policy: require authentication
    options.AddPolicy("AuthenticatedUser", policy =>
        policy.RequireAuthenticatedUser());

    // Policy combining role and claim
    options.AddPolicy("ManagerInAcme", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("tenant", "acme"));

    // Policy with custom requirement
    options.AddPolicy("MinimumAge", policy =>
        policy.AddRequirements(new MinimumAgeRequirement(18)));
});

// Apply
[Authorize(Policy = "ManagerInAcme")]
public IActionResult Sensitive() => View();
```

**Component breakdown:**

| Method | Purpose |
|--------|---------|
| `RequireAuthenticatedUser()` | Requires an authenticated user. |
| `RequireRole(...)` | Requires at least one of the listed roles. |
| `RequireClaim(...)` | Requires a claim (optionally with specific values). |
| `AddRequirements(...)` | Adds custom `IAuthorizationRequirement` instances. |
| `RequireAssertion(...)` | Adds an inline lambda requirement (use sparingly for simple checks). |

**Syntax rules:**

- **Policies must be registered before the app starts**—`AddAuthorization` is called during service registration.
- **`RequireRole` and `RequireClaim` are additive**—all requirements must pass.
- **`AddRequirements`** supports custom requirements evaluated by handlers.
- **`RequireAssertion`** is a lightweight way to express simple conditions without a handler.

**Constraints and limitations:**

- **`RequireAssertion` should not be used for complex logic**—prefer a custom requirement and handler for testability.
- **Policies do not filter data**—they only control endpoint access.
- **Handler registration is scoped**—handlers are resolved per request and can use Scoped services.

### Annotated Complete Step-by-Step Code Example

```csharp
// Requirement
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

// Handler
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var birthDate = context.User.FindFirst(c => c.Type == ClaimTypes.DateOfBirth);
        if (birthDate is null) return Task.CompletedTask;

        var age = DateTime.Today.Year - DateTime.Parse(birthDate.Value).Year;
        if (age >= requirement.MinimumAge) context.Succeed(requirement);
        return Task.CompletedTask;
    }
}

// Registration
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("MinimumAge", policy =>
        policy.AddRequirements(new MinimumAgeRequirement(18)));
});
builder.Services.AddScoped<IAuthorizationHandler, MinimumAgeHandler>();

// Apply
[Authorize(Policy = "MinimumAge")]
public IActionResult AdultsOnly() => View();
```

**Expected Output:** Users with a `DateOfBirth` claim indicating they are 18 or older can access `AdultsOnly`; others receive 403.

**Why this output?** The policy’s requirement is evaluated by `MinimumAgeHandler`, which checks the claim and calls `context.Succeed(requirement)` if the condition is met.

### Real-World Cases

- **Multi-tenant isolation:** A `SameTenantRequirement` that compares the user’s `tenant` claim to the resource’s tenant.
- **Minimum age:** A policy requiring the user to be over 18.
- **Combined conditions:** A policy requiring employee status and a department claim.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- IAuthorizationRequirement interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement

---

## 7. Requirements

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A requirement is a condition that a policy evaluates; a policy is a collection of requirements, all of which must be satisfied. |
| **Technical Definition** | `IAuthorizationRequirement` is a marker interface. Implementations carry the data needed for evaluation (e.g., `MinimumAge`, `SameTenant`). Requirements are added to policies via `AuthorizationPolicyBuilder.AddRequirements` or the convenience methods (`RequireClaim`, `RequireRole`, `RequireAuthenticatedUser`). Each requirement is evaluated by one or more `AuthorizationHandler<TRequirement>` implementations. |
| **Beginner-Friendly Explanation** | A requirement is a **single rule** that must be satisfied. “Must be 18” is a requirement. “Must have a tenant claim” is another. A policy combines several requirements into a complete rulebook. Requirements are the *what*; handlers are the *how*. |

### Purposes

- **To** express **individual conditions** that a policy evaluates.
- **To** carry **data** needed by the handler (e.g., minimum age, tenant ID).
- **To** enable **reuse** across policies—the same requirement can be part of multiple policies.
- **To** support **unit testing** of authorization logic by testing requirements and handlers in isolation.

### Syntax Rules and Structure

```csharp
public class SameTenantRequirement : IAuthorizationRequirement
{
    public SameTenantRequirement() { }
}

// Usage in a policy
options.AddPolicy("SameTenant", policy =>
    policy.AddRequirements(new SameTenantRequirement()));
```

**Syntax rules:**

- **`IAuthorizationRequirement` has no members**—it is a marker interface.
- **Requirements can carry data** (constructor parameters, properties) used by handlers.
- **Requirements are evaluated by handlers**—one or more handlers per requirement.

**Constraints and limitations:**

- **A requirement without a handler fails**—if no handler succeeds for a requirement, the policy fails.
- **Requirements should be simple**—complex logic belongs in the handler.

### Annotated Complete Step-by-Step Code Example

```csharp
public class SameTenantRequirement : IAuthorizationRequirement
{
    public SameTenantRequirement() { }
}

public class SameTenantHandler : AuthorizationHandler<SameTenantRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameTenantRequirement requirement,
        Document resource)
    {
        var tenantClaim = context.User.FindFirst("tenant")?.Value;
        if (tenantClaim is not null && tenantClaim == resource.TenantId)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

**Expected Output:** A user can access a `Document` only if their `tenant` claim matches the document’s `TenantId`.

**Why this output?** The handler receives the requirement and the resource, compares the user’s tenant claim to the resource’s tenant, and succeeds only on a match.

### Real-World Cases

- **Resource-based authorization:** Same-tenant requirements for documents, orders, or projects.
- **Minimum age:** A requirement carrying a minimum age value.
- **Subscription tier:** A requirement carrying the minimum required tier.

### References

- IAuthorizationRequirement interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement
- Resource-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased

---

## 8. Handlers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A handler is a class that evaluates one or more requirements and calls `context.Succeed(requirement)` or `context.Fail()` to record the outcome. |
| **Technical Definition** | `AuthorizationHandler<TRequirement>` is the base class for handlers evaluating a single requirement type. `AuthorizationHandler<TRequirement, TResource>` evaluates a requirement against a resource. Handlers are registered in DI (typically as Scoped) and resolved by the `AuthorizationMiddleware`/`IAuthorizationService`. A handler can evaluate multiple requirements; the policy succeeds only if every requirement has at least one successful handler (and no handler calls `Fail()` for it). |
| **Beginner-Friendly Explanation** | A handler is the **logic that checks a rule**. If the requirement is “must be 18,” the handler is the code that reads the user’s birthdate claim, computes their age, and says “yes” or “no.” Handlers are where the real work of authorization happens—requirements are the labels, handlers are the implementations. |

### Purposes

- **To** implement the **evaluation logic** for a requirement.
- **To** access the `HttpContext`, `ClaimsPrincipal`, resource, and services to make decisions.
- **To** support **multiple handlers per requirement**, enabling extensibility and different sources of truth.
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
| `context.Fail()` | Marks the requirement as failed (optional). |
| Registration | `AddScoped<IAuthorizationHandler, THandler>()`. |

**Syntax rules:**

- **Handlers must be registered in DI**—either individually (`AddScoped`) or via `AddAuthorization` conventions.
- **Handlers can call `context.Succeed` for multiple requirements** they evaluate.
- **A handler that does not call `Succeed` is neutral**—another handler may still succeed the requirement.
- **Handlers should be scoped** if they depend on scoped services (e.g., `DbContext`).

**Constraints and limitations:**

- **Handlers run for all requirements they implement**—not just one policy. Be careful when a handler evaluates multiple requirements.
- **`context.Fail()` prevents success even if other handlers succeed**—use with caution; typically, simply not calling `Succeed` is enough to let another handler succeed.
- **Handlers must not throw**—an exception in a handler causes authorization to fail with an error.

### Annotated Complete Step-by-Step Code Example

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

// Resource-based authorization in a controller
public class DocumentsController : Controller
{
    private readonly IAuthorizationService _auth;

    public DocumentsController(IAuthorizationService auth) => _auth = auth;

    public async Task<IActionResult> Edit(int id)
    {
        var document = await _repo.GetAsync(id);
        var result = await _auth.AuthorizeAsync(User, document, "SameTenant");
        if (!result.Succeeded) return Forbid();

        return View(document);
    }
}
```

**Expected Output:** A user whose `tenant` claim matches the document’s tenant can edit it; others receive 403 Forbidden.

**Why this output?** `AuthorizeAsync` evaluates the `SameTenant` policy against the resource, invoking the `DocumentAuthorizationHandler` which compares the claims and succeeds or not.

### Real-World Cases

- **Resource-based authorization:** Per-item checks for documents, orders, or projects.
- **Multi-handler scenarios:** One handler checks a database-backed permission; another checks a feature flag.
- **Testable authorization:** Handlers can be unit tested with mock `AuthorizationHandlerContext` instances.

### References

- Resource-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- AuthorizationHandler<TRequirement> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1
- Custom authorization policy providers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider

---

## 9. Authorization Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization best practices are guidelines for configuring and implementing authorization that is secure, maintainable, and testable. |
| **Technical Definition** | Best practices encompass **secure-by-default fallback policies**, **policy-based (not role-based) fine-grained authorization**, **resource-based authorization for per-item checks**, **correct middleware ordering**, **avoiding client-side authorization enforcement**, **testing authorization logic**, and **minimizing `[AllowAnonymous]` surfaces**. |
| **Beginner-Friendly Explanation** | Authorization best practices are the **“do’s and don’ts”**—they help you avoid leaving endpoints accidentally public, relying on roles for complex rules, trusting the client to hide UI, and shipping authorization logic that is impossible to test. |

### Purposes

- **To** ensure **secure-by-default** behavior so new endpoints are not accidentally public.
- **To** use **policies and claims** for complex authorization instead of ad-hoc role strings.
- **To** enforce **resource-level checks** for multi-tenant or user-owned data.
- **To** keep **authorization logic testable** and **decoupled from controllers**.
- **To** avoid **information disclosure** by returning 401/403 consistently.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Secure by default** | Configure a fallback policy requiring authentication. | Rely on developers to remember `[Authorize]` on every endpoint. |
| **Policy over roles** | Use policies with requirements and handlers for complex rules. | Encode complex authorization in role strings or inline checks. |
| **Resource-based checks** | Use `IAuthorizationService.AuthorizeAsync(User, resource, policy)` for per-item checks. | Assume endpoint-level authorization covers per-item access. |
| **Middleware order** | `UseAuthentication()` before `UseAuthorization()`, between `UseRouting` and endpoints. | Place `UseAuthorization` before `UseAuthentication` or after endpoint mapping. |
| **`[AllowAnonymous]`** | Use sparingly; audit public endpoints regularly. | Mark entire controllers anonymous for convenience. |
| **401 vs. 403** | Return 401 for unauthenticated, 403 for authenticated-but-forbidden. | Return 404 or 200 to hide authorization failures—it can be confusing and leak information indirectly. |
| **Testing** | Unit test handlers and policies; integration test endpoint authorization. | Test only the happy path or only the UI. |
| **Client-side** | Treat client-side role/claim checks as UX only. | Rely on client-side hiding of UI elements for security. |
| **Handler lifetime** | Register handlers as Scoped when they depend on Scoped services. | Register handlers as Singleton and capture Scoped services. |
| **Policy providers** | Use `IAuthorizationPolicyProvider` for dynamic policies (e.g., per-permission). | Hard-code every policy in `AddAuthorization`. |

### Annotated Code Example (Best-Practice Authorization Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build())
    .AddPolicy("SameTenant", policy =>
        policy.AddRequirements(new SameTenantRequirement()))
    .AddPolicy("MinimumAge", policy =>
        policy.AddRequirements(new MinimumAgeRequirement(18)));

builder.Services.AddScoped<IAuthorizationHandler, SameTenantHandler>();
builder.Services.AddScoped<IAuthorizationHandler, MinimumAgeHandler>();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

// Public endpoints opt out explicitly
app.MapGet("/health", () => "OK").AllowAnonymous();
```

**Why this follows best practices:**

- **Fallback policy** requires authentication for all endpoints by default.
- **Policies** encapsulate complex rules (`SameTenant`, `MinimumAge`) with dedicated requirements and handlers.
- **Handlers are Scoped**, allowing them to depend on per-request services.
- **`[AllowAnonymous]`/`AllowAnonymous()`** is used explicitly for public endpoints.
- **Middleware order** is correct—authentication before authorization.
- **Resource-based checks** (shown in the handler example) complement endpoint-level policies for per-item access.

### Real-World Cases

- **Multi-tenant SaaS:** Fallback policy + same-tenant resource-based checks.
- **Financial applications:** Minimum-age and employee-status policies; strict 401/403 responses.
- **Public APIs:** Fallback policy with explicit `AllowAnonymous` on health and documentation endpoints.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Resource-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Security cards: fallback policies and authorization rules – https://github.com/reware-labs/securitycards
- Custom authorization policy providers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider

---

## Summary Table: Authorization at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Authorization** | Enforce access rules | `AddAuthorization`, `UseAuthorization` | Every secured application |
| **`[Authorize]`** | Apply policies to controllers/actions | `[Authorize]`, `[Authorize(Policy = "...")]`, `[Authorize(Roles = "...")]` | Protecting endpoints |
| **`[AllowAnonymous]`** | Bypass authorization | `[AllowAnonymous]`, `.AllowAnonymous()` | Public endpoints (login, health checks) |
| **Roles** | Coarse-grained group membership | `[Authorize(Roles = "...")]`, `RequireRole` | Simple access levels |
| **Claims** | Fine-grained user attributes | `RequireClaim`, claims in policies | Tenant, subscription, department |
| **Policies** | Named rulebooks | `AddPolicy`, `[Authorize(Policy = "...")]`, `RequireAuthorization("...")` | Complex, reusable rules |
| **Requirements** | Individual conditions | `IAuthorizationRequirement` | Data-carrying rules |
| **Handlers** | Evaluate requirements | `AuthorizationHandler<TRequirement>` | Implementing authorization logic |
| **Best practices** | Secure, testable authorization | Fallback policy, resource-based checks, correct ordering | Every production app |

---

## References (Full List)

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Simple authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/simple
- Role-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles
- Claims-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- Resource-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Minimal API authorization – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/security
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- AuthorizeAttribute class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute
- AllowAnonymousAttribute class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute
- IAuthorizationRequirement interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement
- AuthorizationHandler<TRequirement> class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1
- RolesAuthorizationRequirement – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.rolesauthorizationrequirement
- ClaimsAuthorizationRequirement – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.claims
- Custom authorization policy providers – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider
- Security cards: fallback policies and authorization rules – https://github.com/reware-labs/securitycards