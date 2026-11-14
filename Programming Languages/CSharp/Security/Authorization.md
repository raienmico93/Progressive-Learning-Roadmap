# Authorization Security in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Authorization security is the discipline of determining what an authenticated identity is permitted to do — which resources it can access and which operations it can perform — and enforcing those decisions consistently at every entry point in the application.

**Technical Definition**
Authorization in ASP.NET Core is a framework subsystem built on **requirements**, **handlers**, and **policies**. An authorization policy is a named set of one or more requirements that the framework evaluates to decide whether a user is allowed to access a resource. The evaluation is performed by `IAuthorizationHandler` implementations, which inspect the user's `ClaimsPrincipal` and optionally the resource being accessed. ASP.NET Core provides three complementary models: **role-based authorization** (membership in roles), **claims-based authorization** (presence and value of claims), and **policy-based authorization** (arbitrary requirements evaluated by handlers). These are applied declaratively via `[Authorize]` attributes or imperatively via `IAuthorizationService`, and are supported across MVC controllers, Razor Pages, Minimal APIs, and Blazor components. The **fallback policy** provides secure-by-default behaviour, requiring authentication for all endpoints that do not explicitly opt out.

**Beginner-Friendly Explanation**
Authorization is "deciding who is allowed to do what." Authentication tells you who the user is; authorization tells you what they can do. In ASP.NET Core, you attach `[Authorize]` attributes to controllers, pages, or endpoints to say "only these people can access this." You can express those rules in three ways: by **role** ("only Admins"), by **claim** ("only users with an EmployeeNumber"), or by **policy** ("only users whose department matches the resource and who are over 21"). The framework checks the rule on every request and returns 401 (not logged in) or 403 (logged in but not allowed) if it fails.

---

### Key Characteristics

- **Authentication-dependent** — Authorization always follows authentication; an unauthenticated user cannot be authorized. ASP.NET Core's `UseAuthorization` middleware sits after `UseAuthentication` in the pipeline.
- **Requirement-handler-policy architecture** — Every authorization decision is expressed as one or more requirements, evaluated by handlers, grouped into a named policy.
- **Layered models** — Role-based, claims-based, and policy-based authorization are not alternatives but layers; role and claim checks are themselves implemented as preconfigured policies.
- **Declarative and imperative** — `[Authorize]` attributes handle endpoint-level checks; `IAuthorizationService` handles resource-based checks that require the resource to be loaded first.
- **Secure-by-default capable** — A fallback policy can require authentication for every endpoint that does not explicitly opt out with `[AllowAnonymous]`.
- **Fail-closed** — Denied requests receive 401 or 403; there is no "default allow" when a policy is present and fails.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and dependency injection.
- Understanding of authentication concepts: `ClaimsPrincipal`, `ClaimsIdentity`, and `Claim`.
- Familiarity with the `Microsoft.AspNetCore.Authorization` namespace.
- Basic knowledge of ASP.NET Core middleware ordering (`UseAuthentication` before `UseAuthorization`).
- Awareness of OWASP Top 10 **A01: Broken Access Control**, the most critical web application security risk.

---

### Related Programming Areas

- **Authentication security** — Passwords, MFA, and token validation precede authorization.
- **Security fundamentals** — Least privilege and defense in depth.
- **ASP.NET Core Identity** — Role and claim management.
- **OpenID Connect / OAuth 2.0** — Scopes and claims in access tokens.
- **Resource-based authorization** — Per-object access checks.
- **Blazor authorization** — `AuthorizeView` and `[Authorize]` in Razor components.

---

### Core Concepts / Features

1. Role-Based Access Control (RBAC)
2. Claims-Based Authorization
3. Policy-Based Authorization
4. Authorization Security Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with "To"), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Role-Based Access Control (RBAC)

### Definitions

**Core Definition**
Role-based access control is an authorization model in which permissions are assigned to roles, and users are granted access by being assigned to those roles.

**Technical Definition**
In ASP.NET Core, a role is a claim of type `ClaimTypes.Role` attached to the user's identity. When a user's identity is created after authentication, the user may belong to one or more roles, reflecting the authorizations the user has to access data and perform operations. Roles are exposed to the developer through `ClaimsPrincipal.IsInRole` and checked declaratively with `[Authorize(Roles = "...")]` or `[Authorize(Roles = "Admin,User")]` for multiple roles (comma-separated, OR semantics). Role services must be registered by calling `AddRoles<IdentityRole>()` when configuring Identity. While roles are technically claims, Microsoft cautions that using roles to carry per-user claims can confuse the boundary between the user and their individual claims, and that role proliferation can make role claims too large for tokens used by SPAs.

**Beginner-Friendly Explanation**
Role-based access control is "putting users into groups and giving the groups permissions." A user in the "Administrator" role can access admin pages; a user in the "User" role cannot. In ASP.NET Core, you mark a controller or endpoint with `[Authorize(Roles = "Administrator")]` and the framework checks whether the logged-in user belongs to that role.

---

### Purposes

- To assign permissions to groups rather than to individual users.
- To simplify administration by managing role membership instead of per-user grants.
- To express coarse-grained access rules (admin vs. regular user) declaratively.
- To integrate with ASP.NET Core Identity's built-in role management.
- To provide a familiar, auditable model for access reviews.
- To serve as a foundation on which finer-grained claims and policies can be built.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Single role
[Authorize(Roles = "Administrator")]
public IActionResult AdminDashboard() => View();

// Multiple roles (OR semantics: any of the listed roles grants access)
[Authorize(Roles = "Administrator,Manager")]
public IActionResult Reports() => View();

// Policy that checks a role
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdministratorRole", policy =>
        policy.RequireRole("Administrator"));
});
```

**Component breakdown**

- `Roles` — Comma-separated list of role names; the user must be in **at least one** of them.
- `RequireRole("...")` — Policy-based role check; equivalent to `[Authorize(Roles = "...")]` but composable with other requirements.
- `AddRoles<IdentityRole>()` — Registers role services with ASP.NET Core Identity.
- `ClaimsPrincipal.IsInRole` — Imperative role check inside code.

#### Syntax Rules

- Role names are case-sensitive; use consistent casing across the application.
- Multiple roles in `Roles` are evaluated with **OR** semantics; to require **AND** semantics, use multiple `RequireRole` calls or a custom policy.
- Roles must be registered in Identity (`AddRoles<IdentityRole>()`) for the role store to function.
- Prefer policy-based role checks (`RequireRole`) when the rule must be composed with claims or other requirements.

#### Constraints and Limitations

- Roles are coarse-grained; they cannot express per-resource ownership without additional claims or handlers.
- Role proliferation can make tokens large and unwieldy, particularly for SPAs.
- Using roles to carry per-user claims conflates group membership with individual identity.
- Role names are strings; typos cause silent authorization failures.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Controller-Level Role Authorization

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[Authorize(Roles = "Administrator")]
public class AdminController : Controller
{
    public IActionResult Index() => View();

    [AllowAnonymous]
    public IActionResult Health() => Ok("OK");
}
```

**Expected Output (conceptual)**

```
GET /Admin/Index as Administrator: 200.
GET /Admin/Index as regular user: 403 Forbidden.
GET /Admin/Index without login: 401 Unauthorized.
GET /Admin/Health without login: 200 (AllowAnonymous overrides).
```

**Why this output occurs**

- The controller-level `[Authorize(Roles = "Administrator")]` applies to all actions.
- `[AllowAnonymous]` on `Health` explicitly opts that action out.
- Unauthenticated requests receive 401; authenticated users without the role receive 403.

---

#### Example 2: Policy-Based Role Check

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOrManager", policy =>
        policy.RequireRole("Administrator", "Manager"));
});

app.MapGet("/reports", [Authorize(Policy = "AdminOrManager")] () => "Reports");
```

**Expected Output (conceptual)**

```
Administrator: 200.
Manager: 200.
Regular user: 403.
Unauthenticated: 401.
```

**Why this output occurs**

- `RequireRole` with multiple roles uses OR semantics: any listed role satisfies the requirement.
- The policy is applied to the Minimal API endpoint via `[Authorize(Policy = "...")]`.

---

### Real-World Cases

- **Administrative back offices** — Admin role required for user management, configuration, and audit pages.
- **Content management** — Editor, Reviewer, and Publisher roles control workflow transitions.
- **Multi-tenant SaaS** — Tenant-admin role scoped to a tenant; platform-admin role for cross-tenant operations.
- **Line-of-business applications** — Finance, HR, and Operations roles gate access to functional modules.

---

### References

- Role-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authorization/roles.md
- Introduction to authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction
- Simple authorization in ASP.NET Core MVC — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/mvc/security/authorization/simple.md

---

## 2. Claims-Based Authorization

### Definitions

**Core Definition**
Claims-based authorization is an authorization model in which access decisions are based on the presence and value of claims — name-value pairs that describe attributes of the subject.

**Technical Definition**
A claim is a name-value pair that represents what the subject (a user, an app or service, or a device) **is**, not what the subject can do. When an identity is created at sign-in, the identity provider assigns one or more claims to the user's identity. Claims-based authorization checks are declarative: they specify claims via policies that the current user must present to access the requested resource. In ASP.NET Core, a claim check is expressed as a policy with `RequireClaim("Type")` (presence only) or `RequireClaim("Type", "value1", "value2")` (presence with one of the specified values). Claims are also used for richer checks: date-of-birth claims for age requirements, department claims for data segregation, and scope claims for OAuth 2.0 API access.

**Beginner-Friendly Explanation**
Claims-based authorization is "checking facts about the user." A claim is a fact: "this user has an EmployeeNumber," "this user's department is Finance," "this user was born before 2000." Instead of checking a role name, you check these facts. For example, you can require that the user has an `EmployeeNumber` claim before they can access the employee directory, or that their `Department` claim is `"HR"` before they can see HR data.

---

### Purposes

- To express access rules based on user attributes rather than group membership.
- To support fine-grained rules that roles cannot express (e.g., age, department, clearance level).
- To consume claims issued by external identity providers (OIDC, Entra ID, Auth0).
- To compose multiple attribute checks into a single policy.
- To enforce OAuth 2.0 scope-based access to APIs.
- To decouple authorization logic from role administration.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
// Policy with a presence-only claim check
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("EmployeeOnly", policy =>
        policy.RequireClaim("EmployeeNumber"));

// Policy with claim value check
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("HRDepartment", policy =>
        policy.RequireClaim("Department", "HR"));
});

// Multiple allowed values
options.AddPolicy("FinanceOrAccounting", policy =>
    policy.RequireClaim("Department", "Finance", "Accounting"));
```

**Component breakdown**

- `RequireClaim("EmployeeNumber")` — Requires the claim to be present; any value satisfies it.
- `RequireClaim("Department", "HR")` — Requires the claim to be present with the value `"HR"`.
- `RequireClaim("Department", "Finance", "Accounting")` — Requires the claim to be present with one of the listed values.
- `[Authorize(Policy = "...")]` — Applies the policy to an endpoint.

#### Syntax Rules

- Claims are case-sensitive by type and value; ensure the identity provider and the policy use the same casing.
- The `RequireClaim` overload without values checks only for presence; it does not inspect the value.
- Multiple `RequireClaim` calls in a single policy are evaluated with **AND** semantics.
- Claims must be issued by a trusted party; never trust client-supplied claims without signature validation.

#### Constraints and Limitations

- Claims can become stale; the security stamp or token lifetime determines how often they are refreshed.
- Claims-based rules can be hard to audit when distributed across many policies; centralise policy definitions.
- Overly large claims increase token size and can exceed header limits.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Presence-Only Claim Check

```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("EmployeeOnly", policy =>
        policy.RequireClaim("EmployeeNumber"));

[Authorize(Policy = "EmployeeOnly")]
public IActionResult EmployeeDirectory() => View();
```

**Expected Output (conceptual)**

```
User with EmployeeNumber claim: 200.
User without EmployeeNumber claim: 403.
Unauthenticated user: 401.
```

**Why this output occurs**

- `RequireClaim("EmployeeNumber")` checks only that the claim exists, regardless of its value.

---

#### Example 2: Claim Value Check with Multiple Allowed Values

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("FinanceOrAccounting", policy =>
        policy.RequireClaim("Department", "Finance", "Accounting"));
});

[Authorize(Policy = "FinanceOrAccounting")]
public IActionResult FinancialReports() => View();
```

**Expected Output (conceptual)**

```
User with Department=Finance: 200.
User with Department=Accounting: 200.
User with Department=Marketing: 403.
```

**Why this output occurs**

- The policy accepts any of the listed claim values.

---

#### Example 3: Date-of-Birth Claim for Age Requirement (Custom Handler)

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
    {
        var dobClaim = context.User.FindFirst(c => c.Type == ClaimTypes.DateOfBirth);
        if (dobClaim is null) return Task.CompletedTask;

        var dob = Convert.ToDateTime(dobClaim.Value);
        if (DateTime.Today.Year - dob.Year >= requirement.MinimumAge)
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}
```

**Expected Output (conceptual)**

```
User aged 25 with DOB claim and minimum age 21: 200.
User aged 18 with DOB claim and minimum age 21: 403.
User without DOB claim: 403.
```

**Why this output occurs**

- The custom handler evaluates the claim value against the requirement.
- The requirement carries the minimum age as data.

---

### Real-World Cases

- **Employee directories** — `EmployeeNumber` claim required for internal tools.
- **Departmental data segregation** — `Department` claim gates access to Finance, HR, and Engineering data.
- **Age-restricted content** — `DateOfBirth` claim evaluated against a minimum age.
- **OAuth 2.0 APIs** — `scope` claims control which API operations a client can invoke.
- **Clearance levels** — `Clearance` claim with values such as `Public`, `Secret`, `TopSecret`.

---

### References

- Claim-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- Claims-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/53d05ee9faf3cfe577c6180a0a4c6966d8396ffd/aspnetcore/security/authorization/claims.md
- Authorization based on Scopes and Claims — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/claims-scopes.md

---

## 3. Policy-Based Authorization

### Definitions

**Core Definition**
Policy-based authorization is an authorization model in which a named policy — a set of one or more requirements — is evaluated by the framework to decide whether a user may access a resource.

**Technical Definition**
An ASP.NET Core authorization policy is a named set of one or more authorization requirements that the framework evaluates to decide whether a user is allowed to access a resource. Each requirement implements `IAuthorizationRequirement` (an empty marker interface), and each requirement is evaluated by one or more `IAuthorizationHandler` implementations. Policies are registered with `AddPolicy` (or `AddAuthorizationBuilder().AddPolicy`) and applied with `[Authorize(Policy = "...")]`, `RequireAuthorization("...")`, or imperatively via `IAuthorizationService`. Role-based and claims-based authorization are themselves built on the policy infrastructure using preconfigured requirements and handlers. The **default policy** applies when `[Authorize]` is used without a policy name; the **fallback policy** applies to endpoints with no authorization metadata, enabling secure-by-default. For resource-based checks, `IAuthorizationService.AuthorizeAsync(user, resource, policy)` allows the handler to inspect both the user and the resource.

**Beginner-Friendly Explanation**
Policy-based authorization is "writing your own rule and giving it a name." Instead of just checking a role or a claim, you can write any rule you want — "the user must be over 21 and work in the same department as the document" — and register it as a named policy like `"CanEditDocument"`. Then you attach `[Authorize(Policy = "CanEditDocument")]` to your endpoint. The framework runs your rule on every request and allows or denies access.

---

### Purposes

- To express arbitrary, business-specific authorization rules.
- To compose multiple requirements (role + claim + custom logic) into a single named policy.
- To centralise authorization logic and make it testable and auditable.
- To support resource-based checks where the decision depends on the resource being accessed.
- To implement secure-by-default via fallback policies.
- To replace scattered role checks with coherent, named policies.

---

### Syntax Rules and Structure

#### General Syntax (Simple Policy)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));
});
```

#### General Syntax (Custom Requirement and Handler)

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        if (context.User.HasClaim(c => c.Type == ClaimTypes.DateOfBirth &&
            DateTime.Parse(c.Value) <= DateTime.Today.AddYears(-requirement.MinimumAge)))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

// Register the handler
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();

// Apply the policy
[Authorize(Policy = "AtLeast21")]
public IActionResult AlcoholPurchase() => View();
```

**Component breakdown**

- `IAuthorizationRequirement` — Marker interface; carries any data the handler needs.
- `AuthorizationHandler<TRequirement>` — Evaluates the requirement against the user and context.
- `context.Succeed(requirement)` — Marks the requirement as satisfied.
- `AddSingleton<IAuthorizationHandler, ...>` — Registers the handler with DI.
- `[Authorize(Policy = "...")]` — Applies the policy.

#### Resource-Based Authorization Syntax

```csharp
public class DocumentAuthorizationHandler
    : AuthorizationHandler<SameAuthorRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameAuthorRequirement requirement,
        Document resource)
    {
        if (context.User.Identity?.Name == resource.Author)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}

// Imperative check
var result = await _authorizationService.AuthorizeAsync(
    User, document, "SameAuthorPolicy");
if (!result.Succeeded) return Forbid();
```

#### Fallback Policy (Secure by Default)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Opt out explicitly
[AllowAnonymous]
public IActionResult Public() => Ok();
```

#### Syntax Rules

- Every requirement must have at least one registered handler; an unhandled requirement causes the policy to fail.
- Handlers should call `context.Succeed(requirement)` only when the requirement is satisfied; otherwise leave it unfulfilled.
- Multiple requirements in a single policy are evaluated with **AND** semantics: all must succeed.
- The fallback policy applies to endpoints with no authorization metadata; `[AllowAnonymous]` overrides it.
- Resource-based handlers must implement `IAuthorizationHandler<TRequirement, TResource>` and the resource must be passed to `AuthorizeAsync`.

#### Constraints and Limitations

- Complex policies can become hard to reason about; keep requirements small and composable.
- Handlers are registered as singletons by default; do not inject scoped services directly without using `IServiceScopeFactory`.
- Resource-based authorization requires the resource to be loaded before the check, which may require a separate query.
- Policy names are strings; typos cause silent failures unless covered by tests.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Minimum Age Policy

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }
    public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
    {
        var dobClaim = context.User.FindFirst(ClaimTypes.DateOfBirth);
        if (dobClaim is not null &&
            DateTime.TryParse(dobClaim.Value, out var dob) &&
            dob <= DateTime.Today.AddYears(-requirement.MinimumAge))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));
});
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

**Expected Output (conceptual)**

```
User aged 25 with DOB claim: policy succeeds.
User aged 18 with DOB claim: policy fails.
User without DOB claim: policy fails.
```

**Why this output occurs**

- The handler evaluates the DOB claim against the required minimum age.
- Failure to satisfy the requirement results in a denied request.

---

#### Example 2: Resource-Based Document Authorization

```csharp
public class SameAuthorRequirement : IAuthorizationRequirement { }

public class DocumentAuthorizationHandler
    : AuthorizationHandler<SameAuthorRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameAuthorRequirement requirement,
        Document resource)
    {
        if (context.User.Identity?.Name == resource.Author)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}

// Usage in a controller
var document = await _repository.GetByIdAsync(id);
var result = await _authorizationService.AuthorizeAsync(
    User, document, "SameAuthorPolicy");
if (!result.Succeeded) return Forbid();
```

**Expected Output (conceptual)**

```
Author accessing their own document: allowed.
Different user accessing the document: 403 Forbidden.
Document not found: 404 (handled before authorization).
```

**Why this output occurs**

- The handler receives both the user and the resource and compares the author.
- The resource must be loaded before the check, which is why imperative authorization is used.

---

#### Example 3: Fallback Policy for Secure by Default

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

app.MapGet("/public", [AllowAnonymous] () => "Public");
app.MapGet("/private", () => "Private"); // no [Authorize] needed
```

**Expected Output (conceptual)**

```
GET /public without login: 200.
GET /private without login: 401.
GET /private with login: 200.
```

**Why this output occurs**

- The fallback policy requires authentication for all endpoints without explicit authorization metadata.
- `[AllowAnonymous]` overrides the fallback for the public endpoint.

---

### Real-World Cases

- **Document management** — Only the author can edit their document (resource-based policy).
- **Multi-tenant SaaS** — Policy checks that the user's tenant claim matches the resource's tenant.
- **Regulated industries** — Policies encoding clearance level, training completion, and manager approval.
- **API gateways** — Policy-based scope checks composed with role and claim requirements.

---

### References

- Policy-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Policy-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/policies.md
- Custom authorization policy providers in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/iauthorizationpolicyprovider.md
- Resource-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Resource-based authorization in ASP.NET Core MVC — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/resourcebased.md
- Minimal APIs quick reference — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/security

---

## 4. Authorization Security Best Practices

### Definitions

**Core Definition**
Authorization security best practices are empirically grounded guidelines for designing, implementing, and operating authorization systems that resist privilege escalation, broken access control, and insecure defaults.

**Technical Definition**
Best practices in ASP.NET Core authorization span **secure-by-default configuration** (fallback policy requiring authentication), **deny-by-default endpoint protection** (every externally facing endpoint must have `[Authorize]` or an equivalent), **least-privilege policy design** (requirements should grant only the permissions needed), **resource-level checks** (imperative authorization for per-object access), **centralised policy definitions** (avoid scattered role and claim checks), **thorough testing** (negative tests for 401 and 403), **defence in depth** (authorization at the gateway and at the service), and **auditability** (logging authorization failures). OWASP Top 10 **A01: Broken Access Control** is the most critical web application security risk, and ASP.NET Core's built-in authorization features exist specifically to address it.

**Beginner-Friendly Explanation**
Authorization best practices are "the habits that keep the wrong people out." They include: require login by default, protect every endpoint, give users the minimum permissions they need, check access to individual resources (not just the endpoint), write policies in one place, and test that denied requests actually return 401 or 403. Broken access control is the number one web security risk, and these habits are how you avoid it.

---

### Purposes

- To prevent broken access control (OWASP A01).
- To ensure every externally reachable endpoint is protected.
- To enforce least privilege consistently across the application.
- To make authorization decisions testable, auditable, and maintainable.
- To close gaps caused by forgotten `[Authorize]` attributes.
- To provide defence in depth against privilege escalation.

---

### Guidelines and Structure

#### Require Authentication by Default

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

- The fallback policy protects all endpoints that lack explicit authorization metadata.
- Opt out explicitly with `[AllowAnonymous]` only where truly required.

#### Protect Every Externally Facing Endpoint

```
✅ Every controller, Razor Page, Minimal API endpoint, and Blazor component that exposes data or functionality must have [Authorize] or a fallback policy.
❌ A controller with no authorization attributes treats every request the same way, applying no security checks.
```

- Review the endpoint inventory during code review.
- Use integration tests to assert that unauthenticated requests receive 401.

#### Prefer Policies Over Scattered Role and Claim Checks

```csharp
// ✅ Centralised policy
options.AddPolicy("CanEditOrders", policy =>
    policy.RequireClaim("permission", "orders:write"));

// ❌ Scattered string-based checks
if (User.IsInRole("Admin") || User.HasClaim("dept", "Sales")) { ... }
```

- Centralised policies are easier to audit, test, and change.
- Name policies after business capabilities (`CanEditOrders`), not implementation details.

#### Apply Resource-Level Checks Where Needed

```csharp
var result = await _authorizationService.AuthorizeAsync(User, resource, "ResourcePolicy");
if (!result.Succeeded) return Forbid();
```

- Attribute-based `[Authorize]` cannot inspect a resource that is loaded after attribute evaluation.
- Use imperative resource-based authorization for per-object decisions.

#### Deny by Default in Handlers

```csharp
protected override Task HandleRequirementAsync(
    AuthorizationHandlerContext context, MyRequirement requirement)
{
    if (/* condition is explicitly satisfied */)
        context.Succeed(requirement);
    // Do NOT call context.Succeed by default
    return Task.CompletedTask;
}
```

- Handlers should succeed only when the requirement is demonstrably met.
- Never default to `Succeed` when the condition is uncertain.

#### Use Consistent Deny Semantics

- Unauthenticated request to a protected endpoint: **401 Unauthorized**.
- Authenticated request that fails authorization: **403 Forbidden**.
- Do not leak resource existence by returning 404 for unauthorized access unless that is the deliberate design.

#### Test Negative Paths

```csharp
[Fact]
public async Task AdminEndpoint_WithoutRole_Returns403()
{
    var client = factory.CreateClient();
    // authenticate as non-admin
    var response = await client.GetAsync("/admin");
    Assert.Equal(HttpStatusCode.Forbidden, response.StatusCode);
}
```

- Every protected endpoint should have a test asserting 401 for anonymous and 403 for insufficient privileges.
- Broken access control often hides in untested paths.

#### Centralise and Review Policy Definitions

- Register all policies in a single `AddAuthorization` block or a policy provider.
- Review policies during security reviews and threat modeling.
- Use custom `IAuthorizationPolicyProvider` when policies are dynamic or numerous.

#### Defence in Depth

- Authorize at the API gateway and again at the service.
- Do not rely on client-side UI hiding as an authorization control; always enforce on the server.
- Log authorization failures for monitoring and incident response.

#### Audit and Log

- Log authentication and authorization failures with sufficient context (user, endpoint, policy) without logging sensitive data.
- Alert on spikes in 403 responses, which may indicate enumeration or privilege escalation attempts.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure-by-Default Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* validation */ });

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/public", [AllowAnonymous] () => "Public");
app.MapGet("/private", () => "Private"); // protected by fallback
app.MapGet("/admin", [Authorize(Policy = "AdminOnly")] () => "Admin");

app.Run();
```

**Expected Output (conceptual)**

```
GET /public anonymous: 200.
GET /private anonymous: 401.
GET /private authenticated: 200.
GET /admin non-admin: 403.
GET /admin admin: 200.
```

**Why this output occurs**

- The fallback policy requires authentication for all endpoints without explicit metadata.
- `[AllowAnonymous]` explicitly opts out the public endpoint.
- The admin endpoint additionally requires the `AdminOnly` policy.

---

#### Example 2: Negative Authorization Tests

```csharp
public class AdminEndpointTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public AdminEndpointTests(WebApplicationFactory<Program> factory)
        => _client = factory.CreateClient();

    [Fact]
    public async Task Admin_Anonymous_Returns401()
    {
        var response = await _client.GetAsync("/admin");
        Assert.Equal(HttpStatusCode.Unauthorized, response.StatusCode);
    }

    [Fact]
    public async Task Admin_NonAdmin_Returns403()
    {
        // configure client with a non-admin token
        var response = await _client.GetAsync("/admin");
        Assert.Equal(HttpStatusCode.Forbidden, response.StatusCode);
    }
}
```

**Expected Output**

```
Passed: Admin_Anonymous_Returns401
Passed: Admin_NonAdmin_Returns403
```

**Why this output occurs**

- The tests verify that the endpoint is protected and that the policy denies insufficient privileges.
- These negative tests catch broken access control regressions.

---

### Real-World Cases

- **API gateways** — Fallback authentication policy plus per-route policies.
- **Multi-tenant SaaS** — Tenant-scoped policies and resource-based checks that compare tenant claims.
- **Financial services** — Least-privilege policies, audit logging of 403s, and periodic access reviews.
- **Healthcare** — Defence-in-depth authorization at the gateway and service, with resource-level checks for patient data.

---

### References

- Introduction to authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction
- Broken access control — OWASP Top 10 for .NET developers — microsoftdocs/learn - https://github.com/microsoftdocs/learn/blob/cc1c54baa188e56e8b3a703944ad8c5cb457d534/learn-pr/aspnetcore/owasp-top-10-for-dotnet-developers/includes/3-broken-access-control.md
- Policy-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Resource-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Code checklist — Altinn Studio - https://docs.altinn.studio
- OWASP API Security — Broken function level authorization - https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/

---

## Summary Table: Authorization Models at a Glance

| Model | Question It Answers | ASP.NET Core Mechanism | When to Use |
|---|---|---|---|
| **Role-based** | "Is the user in this group?" | `[Authorize(Roles = "...")]`, `RequireRole(...)` | Coarse-grained access (admin vs. user) |
| **Claims-based** | "Does the user have this fact?" | `RequireClaim("Type")`, `RequireClaim("Type", "value")` | Attribute-based access (department, age, scope) |
| **Policy-based** | "Does the user satisfy this named rule?" | `AddPolicy("Name", ...)`, custom requirements and handlers | Business-specific rules, composition, resource-level checks |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **Fallback policy** | Available in ASP.NET Core 3.0+; recommended for secure-by-default |
| **`AddAuthorizationBuilder()`** | Introduced in .NET 8; modern replacement for `AddAuthorization(options => ...)` in some templates |
| **`IAuthorizationPolicyProvider`** | Dynamic policy generation; useful when policies are numerous or data-driven |
| **Resource-based authorization** | Requires imperative `IAuthorizationService.AuthorizeAsync`; attributes cannot inspect resources loaded after evaluation |
| **Role claim size** | Large role sets can exceed token size limits, especially for SPAs |
| **Client-side UI hiding** | Not an authorization control; always enforce on the server |
| **`[AllowAnonymous]` overuse** | Expands the unauthenticated attack surface; audit regularly |
| **Policy name typos** | Cause silent failures; cover policies with integration tests |

---

## References (Consolidated)

- Introduction to authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction
- Role-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/57fea1b959863e300f7d7926113bb9385c39909e/aspnetcore/security/authorization/roles.md
- Claim-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims
- Claims-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/53d05ee9faf3cfe577c6180a0a4c6966d8396ffd/aspnetcore/security/authorization/claims.md
- Policy-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Policy-based authorization in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/policies.md
- Custom authorization policy providers in ASP.NET Core — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/iauthorizationpolicyprovider.md
- Resource-based authorization in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased
- Resource-based authorization in ASP.NET Core MVC — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/resourcebased.md
- Minimal APIs quick reference — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/security
- Simple authorization in ASP.NET Core MVC — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/mvc/security/authorization/simple.md
- Broken access control — OWASP Top 10 for .NET developers — microsoftdocs/learn - https://github.com/microsoftdocs/learn/blob/cc1c54baa188e56e8b3a703944ad8c5cb457d534/learn-pr/aspnetcore/owasp-top-10-for-dotnet-developers/includes/3-broken-access-control.md
- Authorization based on Scopes and Claims — dotnet/aspnetcore.docs - https://github.com/dotnet/aspnetcore.docs/blob/main/aspnetcore/security/authorization/claims-scopes.md
- Code checklist — Altinn Studio - https://docs.altinn.studio
- OWASP API Security — Broken function level authorization - https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/