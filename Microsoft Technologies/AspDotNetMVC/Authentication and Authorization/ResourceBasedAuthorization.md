# ASP.NET Core Resource-Based Authorization: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **imperative authorization** model used for resource-based checks. Attribute-based `[Authorize]` evaluation occurs **before** data binding and before the action or page handler loads the resource, so it cannot make decisions that depend on the resource’s data. Resource-based authorization therefore requires **imperative authorization** via `IAuthorizationService`. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource-based authorization is an authorization approach where the access decision depends on **the specific resource being accessed**, requiring the resource to be loaded before authorization is evaluated. |
| **Technical Definition** | In ASP.NET Core, resource-based authorization is implemented through **imperative authorization**: the app injects `IAuthorizationService`, loads the resource, and calls `AuthorizeAsync(User, resource, policyNameOrRequirements)`. The service invokes `AuthorizationHandler<TRequirement, TResource>` implementations that receive both the requirement and the resource instance. Unlike attribute-based authorization, which runs before model binding and before the action executes, resource-based authorization runs **inside** the action or page handler after the resource is retrieved from the data store. |
| **Beginner-Friendly Explanation** | Sometimes you cannot decide access until you have the actual thing being accessed. For example, "only the author can edit this document"—you need the document first to know who the author is. `[Authorize]` runs too early, before the document is loaded. Resource-based authorization lets you load the document, then ask the authorization service: "Given this document and this user, can they edit it?" The service calls a handler that compares the user’s identity to the document’s properties and answers yes or no. |

### Key Characteristics

- **Imperative** – Authorization is invoked explicitly in code via `IAuthorizationService.AuthorizeAsync`, not declaratively via attributes.
- **Resource-aware** – Handlers receive the resource instance (`TResource`) alongside the requirement, enabling decisions based on resource properties (author, tenant, status, owner).
- **Late-bound** – Runs **after** the resource is loaded, inside the action/page handler.
- **Two overloads** – `AuthorizeAsync(user, resource, policyName)` and `AuthorizeAsync(user, resource, requirements)`.
- **Challenge vs. forbid** – The caller decides whether to return `ChallengeResult` (unauthenticated) or `ForbidResult` (authenticated but unauthorized).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core MVC or Razor Pages project.
- Working knowledge of **policy-based authorization** and **claims-based identity**.
- Familiarity with **dependency injection** and **async/await**.
- A data store or repository to load the resource before authorization.

### Related Programming Areas

- **Policy-based authorization** – Resource-based authorization uses policies and requirements, but evaluates them against a resource.
- **`IAuthorizationService`** – The service that orchestrates imperative authorization.
- **Authorization handlers** – `AuthorizationHandler<TRequirement, TResource>` evaluates requirements against resources.
- **`OperationAuthorizationRequirement`** – A built-in helper for CRUD-based resource authorization.
- **MVC and Razor Pages** – Both support imperative authorization inside controllers and page handlers.

### Core Concepts / Features

1. **Resource-based authorization** – The concept, why `[Authorize]` is insufficient, and the imperative model.
2. **`IAuthorizationService`** – The service interface, registration, and injection.
3. **`AuthorizeAsync()`** – The two overloads, return value, and usage patterns.
4. **Resource-based authorization best practices** – Challenge/forbid, handler design, and common pitfalls.

---

## 1. Resource-Based Authorization (The Concept)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource-based authorization is an access-control model where the decision depends on the specific resource instance being accessed, not just on the user’s identity or role. |
| **Technical Definition** | Because `[Authorize]` attributes are evaluated **before** model binding and before the action loads the resource, they cannot inspect resource data. Resource-based authorization must therefore be **imperative**: the action/page handler loads the resource, then calls `IAuthorizationService.AuthorizeAsync(User, resource, policyName)` or the requirements overload. The service evaluates the policy’s requirements using handlers that receive both the requirement and the resource, and returns an `AuthorizationResult` with `Succeeded` set to `true` or `false`. |
| **Beginner-Friendly Explanation** | Resource-based authorization is the **“check the actual item”** model. If the rule is “only the document’s author can edit it,” you first fetch the document, then ask: “Is this user the author of this document?” Attribute-based authorization cannot do this because it runs before you have the document. Resource-based authorization lets you make the decision **after** loading the resource, using the resource’s own data. |

### Purposes

- **To** authorize access based on **resource properties** (author, owner, tenant, status, creation date) that are unknown until the resource is loaded.
- **To** handle **fine-grained, per-item** authorization that cannot be expressed as static policies on the endpoint.
- **To** support **imperative checks inside actions and page handlers** where the resource is already available.
- **To** enable **CRUD-based authorization** through `OperationAuthorizationRequirement`, using a single handler for create/read/update/delete.
- **To** complement—not replace—endpoint-level policies for coarse-grained access control.

### Syntax Rules and Structure

#### Why `[Authorize]` Is Insufficient

```text
Request → [Authorize] evaluation (before model binding)
       → Model binding
       → Action/Page handler runs
       → Resource is loaded
```

The `[Authorize]` attribute runs **before** the resource exists. Resource-based decisions must run **after** the resource is loaded, **inside** the action.

#### General Syntax (Imperative Check in a Controller)

```csharp
public class DocumentController : Controller
{
    private readonly IAuthorizationService _authorizationService;
    private readonly IDocumentRepository _documentRepository;

    public DocumentController(
        IAuthorizationService authorizationService,
        IDocumentRepository documentRepository)
    {
        _authorizationService = authorizationService;
        _documentRepository = documentRepository;
    }

    public async Task<IActionResult> Edit(Guid documentId)
    {
        var document = _documentRepository.Find(documentId);
        if (document is null) return NotFound();

        var result = await _authorizationService
            .AuthorizeAsync(User, document, "EditPolicy");

        if (result.Succeeded) return View(document);
        if (User.Identity?.IsAuthenticated == true) return Forbid();
        return Challenge();
    }
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IAuthorizationService` | Injected service that performs imperative authorization. |
| `_documentRepository.Find(id)` | Loads the resource before authorization. |
| `AuthorizeAsync(User, document, "EditPolicy")` | Evaluates the policy against the resource. |
| `result.Succeeded` | `true` if the user is authorized for this resource. |
| `Forbid()` / `Challenge()` | Returns 403 for authenticated-but-unauthorized, or 401/challenge for unauthenticated. |

**Syntax rules:**

- **Load the resource first**—`AuthorizeAsync` needs the resource instance.
- **Use `User` from `HttpContext`**—the `ClaimsPrincipal` populated by authentication.
- **Check `result.Succeeded`** and branch to `Forbid()` or `Challenge()` accordingly.
- **The resource can be any `object`**—a domain entity, a DTO, or a view model.

**Constraints and limitations:**

- **`[Authorize]` cannot be used for resource-specific decisions**—it runs too early.
- **Imperative checks are not automatically enforced**—you must call `AuthorizeAsync` in every action that needs the check.
- **Handlers must be registered**—the policy and its handlers must be configured in `Program.cs`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Edit Authorization for a Document

```csharp
[HttpGet]
public async Task<IActionResult> Edit(Guid documentId)
{
    // 1. Load the resource
    Document document = _documentRepository.Find(documentId);
    if (document is null) return NotFound();

    // 2. Imperative authorization
    var authorizationResult = await _authorizationService
        .AuthorizeAsync(User, document, "EditPolicy");

    // 3. Branch on the result
    if (authorizationResult.Succeeded)
    {
        return View(document);
    }

    if (User.Identity?.IsAuthenticated == true)
    {
        return Forbid();   // 403 — authenticated but not allowed
    }

    return Challenge();   // 401 / redirect to login
}
```

**Step-by-step explanation:**

1. The document is loaded from the repository **before** authorization runs.
2. `AuthorizeAsync` evaluates the `"EditPolicy"` policy against the document and the current user.
3. If `Succeeded` is `true`, the user is authorized and the view is returned.
4. If not, the code distinguishes between authenticated users (`Forbid`) and unauthenticated users (`Challenge`).

**Expected Output:** The author of the document sees the edit view. An authenticated non-author receives **403 Forbidden**. An unauthenticated user is challenged (redirect to login or 401).

**Why this output?** The handler for `"EditPolicy"` compares the user’s identity to the document’s `Author` property and succeeds only on a match.

#### Example 2: Razor Pages Imperative Check

```csharp
public async Task<IActionResult> OnGetAsync(Guid documentId)
{
    Document = _documentRepository.Find(documentId);
    if (Document is null) return NotFound();

    var result = await _authorizationService
        .AuthorizeAsync(User, Document, "EditPolicy");

    if (result.Succeeded) return Page();
    if (User.Identity?.IsAuthenticated == true) return Forbid();
    return Challenge();
}
```

**Expected Output:** The same behavior as the controller example, applied to a Razor Page handler.

**Why this output?** Razor Pages use the same `IAuthorizationService` and `AuthorizeAsync` APIs; the only difference is that the check is in a page handler rather than a controller action.

### Real-World Cases

- **Document management:** Only the author can edit or delete a document.
- **Multi-tenant SaaS:** A user can access an order only if the order’s `TenantId` matches their `tenant` claim.
- **Healthcare records:** A clinician can view a patient’s record only if they are assigned to that patient.
- **E-commerce orders:** A customer can view an order only if they placed it.

### References

- Resource-based authorization in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based
- Resource-based authorization in ASP.NET Core (GitHub source) – https://github.com/dotnet/AspNetCore.Docs/blob/live/aspnetcore/security/authorization/resourcebased.md

---

## 2. `IAuthorizationService`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `IAuthorizationService` is the service that evaluates authorization policies and requirements, including resource-based checks. |
| **Technical Definition** | `IAuthorizationService` (in `Microsoft.AspNetCore.Authorization`) is registered in the DI container by the ASP.NET Core framework at startup. It exposes `AuthorizeAsync` overloads for evaluating a policy or a collection of requirements against a `ClaimsPrincipal` and an optional resource. The service resolves the appropriate `AuthorizationHandler<TRequirement, TResource>` instances and returns an `AuthorizationResult` indicating success or failure. |
| **Beginner-Friendly Explanation** | `IAuthorizationService` is the **authorization engine** you call when you need a decision. You inject it into your controller or page handler, hand it the user, the resource, and the policy name, and it runs all the relevant handlers to determine whether access should be granted. It is the imperative counterpart to the `[Authorize]` attribute. |

### Purposes

- **To** perform **imperative authorization** inside actions and page handlers.
- **To** evaluate a **policy name** against a user and a resource.
- **To** evaluate an **ad-hoc list of requirements** against a user and a resource.
- **To** return an `AuthorizationResult` that the caller can inspect.
- **To** support **resource-based** decisions through handlers that receive `TResource`.

### Syntax Rules and Structure

```csharp
public interface IAuthorizationService
{
    Task<AuthorizationResult> AuthorizeAsync(
        ClaimsPrincipal user, object? resource, string policyName);

    Task<AuthorizationResult> AuthorizeAsync(
        ClaimsPrincipal user, object? resource,
        IEnumerable<IAuthorizationRequirement> requirements);
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `IAuthorizationService` | The service interface (in `Microsoft.AspNetCore.Authorization`). |
| `AuthorizeAsync(user, resource, policyName)` | Evaluates a named policy against the resource. |
| `AuthorizeAsync(user, resource, requirements)` | Evaluates a custom collection of requirements. |
| `AuthorizationResult` | Contains `Succeeded` (bool) and, on failure, `FailureReasons`. |

**Syntax rules:**

- **`IAuthorizationService` is registered automatically** by the framework—do not register it manually.
- **Inject it via constructor** into controllers, page models, or services.
- **Pass `User` from `HttpContext`** as the `ClaimsPrincipal`.
- **`resource` can be `null`** for endpoint-level checks, but for resource-based checks pass the loaded resource.

**Constraints and limitations:**

- **`IAuthorizationService` does not load the resource**—the caller must load it first.
- **The service is stateless**—it resolves handlers per call and is safe to inject into Scoped services.
- **Handlers must be registered in DI**—otherwise, the service will not find them and the policy will fail.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Injecting and Using `IAuthorizationService`

```csharp
public class InvoiceController : Controller
{
    private readonly IAuthorizationService _authorizationService;
    private readonly IInvoiceRepository _invoiceRepository;

    public InvoiceController(
        IAuthorizationService authorizationService,
        IInvoiceRepository invoiceRepository)
    {
        _authorizationService = authorizationService;
        _invoiceRepository = invoiceRepository;
    }

    public async Task<IActionResult> View(int invoiceId)
    {
        var invoice = await _invoiceRepository.GetAsync(invoiceId);
        if (invoice is null) return NotFound();

        var result = await _authorizationService
            .AuthorizeAsync(User, invoice, "InvoiceReadPolicy");

        if (!result.Succeeded)
            return User.Identity?.IsAuthenticated == true
                ? Forbid()
                : Challenge();

        return View(invoice);
    }
}
```

**Step-by-step explanation:**

1. `IAuthorizationService` is injected via the constructor (registered automatically by the framework).
2. The invoice is loaded from the repository.
3. `AuthorizeAsync` evaluates `"InvoiceReadPolicy"` against the invoice and the current user.
4. If not authorized, the action returns `Forbid()` or `Challenge()` depending on authentication status.

**Expected Output:** An authorized user sees the invoice; an authenticated but unauthorized user receives 403; an unauthenticated user is challenged.

**Why this output?** The `"InvoiceReadPolicy"` handler compares the invoice’s owner to the user’s identity and succeeds only on a match.

#### Example 2: Using the Requirements Overload

```csharp
var requirements = new IAuthorizationRequirement[]
{
    new SameAuthorRequirement(),
    new DocumentStatusRequirement(DocumentStatus.Draft)
};

var result = await _authorizationService
    .AuthorizeAsync(User, document, requirements);
```

**Expected Output:** The document is accessible only if the user is the author **and** the document is in `Draft` status.

**Why this output?** Both requirements are evaluated against the resource; the policy succeeds only if both succeed.

### Real-World Cases

- **Invoice access:** A customer can view only their own invoices.
- **Document status:** Only draft documents can be edited by the author; published documents are read-only.
- **Tenant isolation:** A user’s `tenant` claim must match the resource’s `TenantId`.

### References

- IAuthorizationService interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice
- Resource-based authorization (imperative authorization) – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based

---

## 3. `AuthorizeAsync()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `AuthorizeAsync()` is the method on `IAuthorizationService` that evaluates a policy or requirements against a user and a resource, returning an `AuthorizationResult`. |
| **Technical Definition** | `AuthorizeAsync` has two overloads: one accepting a **policy name** and one accepting a collection of **requirements**. Both accept a `ClaimsPrincipal` and an optional `object resource`. The method resolves the appropriate `AuthorizationHandler<TRequirement, TResource>` instances, invokes `HandleRequirementAsync` for each, and returns an `AuthorizationResult` whose `Succeeded` property indicates whether **all** requirements were satisfied. |
| **Beginner-Friendly Explanation** | `AuthorizeAsync()` is the **actual call you make** when you need an authorization decision. You hand it the user, the resource, and either a policy name or a list of requirements. It runs the handlers and tells you "yes" or "no." It is asynchronous because handlers may perform I/O (database lookups, external service calls). |

### Purposes

- **To** evaluate a **named policy** against a user and a resource.
- **To** evaluate an **ad-hoc set of requirements** without registering a named policy.
- **To** return an `AuthorizationResult` for branching logic in the caller.
- **To** support **resource-based** handlers that receive `TResource`.
- **To** enable **CRUD-based** authorization through `OperationAuthorizationRequirement`.

### Syntax Rules and Structure

#### Overload 1: Policy Name

```csharp
Task<AuthorizationResult> AuthorizeAsync(
    ClaimsPrincipal user,
    object? resource,
    string policyName);
```

#### Overload 2: Requirements

```csharp
Task<AuthorizationResult> AuthorizeAsync(
    ClaimsPrincipal user,
    object? resource,
    IEnumerable<IAuthorizationRequirement> requirements);
```

**Component breakdown:**

| Parameter | Description |
|-----------|-------------|
| `user` | The `ClaimsPrincipal` (typically `HttpContext.User`). |
| `resource` | The resource instance, or `null` for endpoint-level checks. |
| `policyName` | The name of a registered policy (overload 1). |
| `requirements` | A collection of `IAuthorizationRequirement` instances (overload 2). |

**Syntax rules:**

- **Both overloads are asynchronous**—use `await`.
- **`resource` can be `null`** if the check does not depend on a resource.
- **The result’s `Succeeded` property** is the primary decision indicator.
- **On failure, `AuthorizationResult.FailureReasons`** contains the reasons (useful for logging, not typically returned to the client).

**Constraints and limitations:**

- **`AuthorizeAsync` does not throw on authorization failure**—it returns a failed result; the caller must handle it.
- **The requirements overload bypasses named policies**—use it for ad-hoc checks, but prefer named policies for reuse and testability.
- **Handler exceptions propagate**—if a handler throws, `AuthorizeAsync` fails with an exception.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Policy Name Overload

```csharp
var result = await _authorizationService
    .AuthorizeAsync(User, document, "EditPolicy");

if (result.Succeeded)
{
    return View(document);
}
```

**Expected Output:** `result.Succeeded` is `true` if the user passes the `"EditPolicy"` requirements for the document.

**Why this output?** The policy’s requirements are evaluated by handlers that receive the document resource.

#### Example 2: Requirements Overload with CRUD Operations

```csharp
public static class DocumentOperations
{
    public static OperationAuthorizationRequirement Create =
        new() { Name = nameof(Create) };
    public static OperationAuthorizationRequirement Read =
        new() { Name = nameof(Read) };
    public static OperationAuthorizationRequirement Update =
        new() { Name = nameof(Update) };
    public static OperationAuthorizationRequirement Delete =
        new() { Name = nameof(Delete) };
}

// In the action
var result = await _authorizationService
    .AuthorizeAsync(User, document, new[] { DocumentOperations.Update });
```

**Expected Output:** The user is authorized to update the document only if the CRUD handler succeeds for the `Update` operation.

**Why this output?** `OperationAuthorizationRequirement` carries the operation name (`"Update"`); the handler checks it along with the resource and the user’s identity.

#### Example 3: Challenge and Forbid with `AuthorizeAsync`

```csharp
if ((await _authorizationService
        .AuthorizeAsync(User, document, "ReadPolicy")).Succeeded)
{
    return View(document);
}
else if (User.Identity?.IsAuthenticated == true)
{
    return new ForbidResult();
}
else
{
    return new ChallengeResult();
}
```

**Expected Output:** Authorized users see the document; authenticated but unauthorized users receive 403; unauthenticated users receive a challenge.

**Why this output?** The caller explicitly distinguishes between `ForbidResult` (403) and `ChallengeResult` (401/redirect) based on authentication status.

### Real-World Cases

- **CRUD authorization:** Use `OperationAuthorizationRequirement` with `AuthorizeAsync` to check create/read/update/delete permissions per resource.
- **Ad-hoc checks:** Evaluate a requirement list directly without registering a named policy.
- **Multi-requirement checks:** Combine `SameAuthorRequirement` and `DocumentStatusRequirement` in a single `AuthorizeAsync` call.

### References

- IAuthorizationService.AuthorizeAsync overloads – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice.authorizeasync
- Operational requirements (CRUD) – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#operational-requirements
- Challenge and forbid with an operational resource handler – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#challenge-and-forbid-with-an-operational-resource-handler

---

## 4. Resource-Based Authorization Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Resource-based authorization best practices are guidelines for implementing imperative authorization that is secure, maintainable, and correctly integrated with the MVC/Razor Pages request pipeline. |
| **Technical Definition** | Best practices encompass **loading the resource before authorization**, **distinguishing `ForbidResult` from `ChallengeResult`**, **using `OperationAuthorizationRequirement` for CRUD**, **registering handlers with the correct lifetime**, **keeping the resource class free of authorization logic**, **using policies for reusable rules**, and **returning `NotFound` when the resource does not exist** (rather than leaking its existence through a 403). |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for resource-based authorization—they help you avoid authorizing before the resource is loaded, confusing “not logged in” with “not allowed,” and putting authorization logic in the wrong place. |

### Purposes

- **To** ensure authorization runs **after** the resource is available.
- **To** return the **correct status code** (401 vs. 403 vs. 404) for each situation.
- **To** keep authorization logic **testable and reusable** through policies and handlers.
- **To** avoid **leaking resource existence** to unauthorized users.
- **To** support **CRUD operations** with a single handler rather than four separate handlers.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Resource loading** | Load the resource **before** calling `AuthorizeAsync`. | Call `AuthorizeAsync` with a `null` or placeholder resource. |
| **Not found** | Return `NotFound()` if the resource does not exist **before** authorization. | Authorize a null resource and return 403—it leaks less but is semantically wrong. |
| **Challenge vs. forbid** | Return `Challenge()` for unauthenticated users; return `Forbid()` for authenticated-but-unauthorized users. | Return 403 for unauthenticated users (confuses the client). |
| **CRUD** | Use `OperationAuthorizationRequirement` with a single handler. | Create four separate handlers for Create/Read/Update/Delete. |
| **Handler lifetime** | Register handlers as **Scoped** if they depend on Scoped services (e.g., `DbContext`). | Register handlers as Singleton and capture Scoped services. |
| **Resource class** | Keep the resource class as a **data carrier**—no authorization logic. | Put `CanEdit(user)` methods on the resource class. |
| **Policy reuse** | Register named policies for rules used in multiple actions. | Inline ad-hoc requirement lists in every action. |
| **Failure reasons** | Use `AuthorizationResult.FailureReasons` for **logging**, not for returning to clients. | Return detailed failure reasons to the client—they can leak information. |

### Annotated Code Example (Best-Practice Resource-Based Authorization)

```csharp
public class DocumentController : Controller
{
    private readonly IAuthorizationService _auth;
    private readonly IDocumentRepository _repo;

    public DocumentController(IAuthorizationService auth, IDocumentRepository repo)
    {
        _auth = auth;
        _repo = repo;
    }

    public async Task<IActionResult> Edit(Guid documentId)
    {
        // 1. Load resource first
        var document = await _repo.FindAsync(documentId);
        if (document is null) return NotFound();

        // 2. Imperative check with a named policy
        var result = await _auth.AuthorizeAsync(User, document, "EditPolicy");

        // 3. Branch: success, forbid, or challenge
        if (result.Succeeded) return View(document);
        if (User.Identity?.IsAuthenticated == true) return Forbid();
        return Challenge();
    }

    public async Task<IActionResult> Delete(Guid documentId)
    {
        var document = await _repo.FindAsync(documentId);
        if (document is null) return NotFound();

        var result = await _auth.AuthorizeAsync(
            User, document, new[] { DocumentOperations.Delete });

        if (result.Succeeded) { /* delete */ return RedirectToAction("Index"); }
        if (User.Identity?.IsAuthenticated == true) return Forbid();
        return Challenge();
    }
}

// Handler registration
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("EditPolicy", policy =>
        policy.Requirements.Add(new SameAuthorRequirement()));

builder.Services.AddScoped<IAuthorizationHandler, DocumentAuthorizationHandler>();
```

**Why this follows best practices:**

- **The resource is loaded before authorization**—the handler can inspect its properties.
- **`NotFound()` is returned before authorization**—the app does not leak the resource’s existence through a 403.
- **`Forbid()` and `Challenge()` are used correctly**—403 for authenticated users, challenge for unauthenticated users.
- **A named policy is used for a reusable rule** (`"EditPolicy"`).
- **`OperationAuthorizationRequirement` is used for CRUD**—a single handler can handle create/read/update/delete.
- **The handler is registered as Scoped**—it can depend on per-request services if needed.
- **The resource class remains a data carrier**—no authorization logic lives inside it.

### Real-World Cases

- **Document collaboration platforms:** Authors can edit, reviewers can comment, others can only read—all decided per-document.
- **Multi-tenant APIs:** A user can access an order only if the order’s `TenantId` matches their `tenant` claim.
- **Healthcare systems:** Clinicians can view patient records only if they are on the patient’s care team.

### References

- Resource-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based
- Challenge and forbid with an operational resource handler – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#challenge-and-forbid-with-an-operational-resource-handler
- Operational requirements (CRUD) – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#operational-requirements
- Policy-based authorization (handlers and requirements) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## Summary Table: Resource-Based Authorization at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Resource-based authorization** | Authorize based on resource properties | Imperative check inside the action/handler | Rules that depend on the loaded resource (author, tenant, owner, status) |
| **`IAuthorizationService`** | The authorization engine | Injected via constructor | Every imperative authorization check |
| **`AuthorizeAsync()`** | Evaluate a policy or requirements against a resource | `AuthorizeAsync(User, resource, policyNameOrRequirements)` | Inside actions/page handlers after loading the resource |
| **Best practices** | Correct, secure, maintainable checks | Load first, `NotFound` before auth, `Forbid`/`Challenge`, `OperationAuthorizationRequirement` | Every resource-based authorization implementation |

---

## References (Full List)

- Resource-based authorization in ASP.NET Core MVC – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based
- Resource-based authorization in ASP.NET Core (GitHub source) – https://github.com/dotnet/AspNetCore.Docs/blob/live/aspnetcore/security/authorization/resourcebased.md
- IAuthorizationService interface – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice
- IAuthorizationService.AuthorizeAsync overloads – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice.authorizeasync
- Operational requirements (CRUD) – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#operational-requirements
- Challenge and forbid with an operational resource handler – https://learn.microsoft.com/en-us/aspnet/core/mvc/security/authorization/resource-based#challenge-and-forbid-with-an-operational-resource-handler
- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Create an ASP.NET Core app with user data protected by authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/secure-data