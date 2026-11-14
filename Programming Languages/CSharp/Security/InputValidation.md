# Input Validation in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Input validation is the practice of verifying that data entering an application — from users, APIs, files, message queues, or other systems — conforms to expected structure, type, length, range, and format before it is processed or stored.

**Technical Definition**
Input validation in .NET is a defence-in-depth mechanism applied at **trust boundaries** — every point where data crosses from an untrusted source into the application. The platform provides two first-class declarative frameworks: **Data Annotations** (`System.ComponentModel.DataAnnotations`), which attach validation rules to model properties via attributes and are automatically enforced by ASP.NET Core's model binding and `[ApiController]` pipeline, and **FluentValidation**, a third-party library that expresses validation rules as strongly typed, testable classes separate from the model. Validation must occur **on the server** regardless of client-side checks, because client-side validation is trivially bypassed. Validation is distinct from **sanitisation**: validation rejects non-conforming input, while output encoding neutralises dangerous content for a specific sink (HTML, SQL, shell, path). Both are required; neither replaces the other.

**Beginner-Friendly Explanation**
Input validation is "checking what comes in before you use it." If someone types `"abc"` into an age field, validation rejects it before it reaches your database. In C#, you can attach rules directly to your model with attributes like `[Required]` and `[Range]`, or write them in a separate, testable class with FluentValidation. The golden rule: **never trust input just because it came from your own frontend** — always validate on the server.

---

### Key Characteristics

- **Boundary-first** — Validate at the trust boundary, not deep in the call stack; reject invalid data as early as possible.
- **Server-authoritative** — Client-side validation is a UX convenience; server-side validation is the security control.
- **Declarative vs. imperative** — Data Annotations are declarative attributes on the model; FluentValidation is imperative, fluent, and separate from the model.
- **Fail-closed** — Invalid input must result in rejection (400 Bad Request), never silent acceptance or best-effort coercion.
- **Allow-list preferred** — Prefer specifying what is allowed over enumerating what is forbidden.
- **Not sanitisation** — Validation checks structure; output encoding handles dangerous content for the target sink.
- **Testable and centralised** — FluentValidation rules are unit-testable classes; Data Annotations are less testable in isolation.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and model binding.
- Familiarity with the `System.ComponentModel.DataAnnotations` namespace.
- Understanding of dependency injection (for FluentValidation registration).
- Awareness of trust boundaries: HTTP requests, message queues, file uploads, and database reads.
- Basic knowledge of OWASP Top 10, particularly injection and cross-site scripting.

---

### Related Programming Areas

- **Security fundamentals** — Threat modeling and attack surface reduction.
- **Authentication and authorization** — Validation follows authentication; authorization decisions must not rely on unvalidated input.
- **Data access** — Parameterised queries and ORM mapping depend on validated input.
- **API design** — Request DTOs are the natural home for validation rules.
- **Testing** — FluentValidation validators are unit-testable; validation filters are integration-testable.

---

### Core Concepts / Features

1. Input Validation (Overarching Discipline)
2. Data Annotations
3. FluentValidation
4. Validation Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with "To"), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Input Validation (Overarching Discipline)

### Definitions

**Core Definition**
Input validation is the systematic verification of external data against expected constraints before the application acts on it.

**Technical Definition**
Input validation in .NET operates at multiple layers: **transport** (model binding rejects type mismatches), **model** (Data Annotations or FluentValidation enforce structure), **domain** (value objects and invariants enforce business rules), and **database** (constraints provide a last line of defence). The ASP.NET Core `[ApiController]` attribute automatically returns a 400 Bad Request when `ModelState.IsValid` is false, but only if the controller is decorated with `[ApiController]` or the action explicitly checks `ModelState`. Validation must be applied to **all** externally controlled inputs: query strings, route parameters, headers, body payloads, cookies, and file uploads. OWASP identifies improper input validation as a root cause of injection, cross-site scripting, and path traversal.

**Beginner-Friendly Explanation**
Input validation is "the bouncer at the door of your application." Every piece of data from outside must pass the checks before it is allowed in. You validate on the server, not just in the browser, and you validate everything — not just the fields you think are risky.

---

### Purposes

- To prevent invalid, malformed, or malicious data from reaching business logic or storage.
- To protect against injection attacks (SQL, shell, path traversal, XSS).
- To enforce business invariants and data integrity.
- To provide clear, actionable error messages to API consumers.
- To reduce the attack surface exposed to downstream systems.
- To satisfy OWASP ASVS and regulatory validation requirements.

---

### Syntax Rules and Structure

#### General Validation Layers

```
Layer 1: Transport   — Model binding rejects type mismatches (400 automatically with [ApiController]).
Layer 2: Model       — Data Annotations or FluentValidation enforce structure.
Layer 3: Domain      — Value objects and invariants enforce business rules.
Layer 4: Database    — Constraints (NOT NULL, CHECK, UNIQUE, FK) provide last-line defence.
```

**Component breakdown**

- **Transport** — Type coercion and binding.
- **Model** — Declarative or fluent validation.
- **Domain** — Constructor guards and value object invariants.
- **Database** — Schema constraints.

#### Syntax Rules

- Validate all externally controlled inputs, not just the body.
- Always validate on the server; client-side validation is advisory only.
- Reject invalid input with 400 Bad Request; do not silently coerce.
- Use allow-lists where possible; avoid deny-lists that can be bypassed.
- Validation is not sanitisation; encode output for the target sink separately.

#### Constraints and Limitations

- Validation rules must be maintained as the domain evolves; stale rules create false confidence.
- Validation adds CPU and latency; async validation (database uniqueness) requires careful performance design.
- Overly strict validation can reject legitimate inputs (e.g., international names, Unicode).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Boundary Validation with TryParse

```csharp
// ✅ Validate and constrain at the edge
if (!long.TryParse(Request.Query["id"], out var id) || id <= 0)
    return Results.BadRequest("invalid id");
```

**Expected Output (conceptual)**

```
id=abc: 400 Bad Request.
id=-1: 400 Bad Request.
id=42: proceeds to business logic.
```

**Why this output occurs**

- `TryParse` avoids exceptions and returns `false` on malformed input.
- The range check rejects non-positive values before they reach the domain.

---

#### Example 2: Server-Side Validation with Data Annotations

```csharp
public class CreateUserRequest
{
    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; } = default!;

    [Required]
    [EmailAddress]
    [StringLength(255)]
    public string Email { get; set; } = default!;

    [Range(0, 150)]
    public int Age { get; set; }
}
```

**Expected Output (conceptual)**

```
Missing Name: 400 with "Name is required".
Invalid email: 400 with format error.
Age=200: 400 with range error.
All valid: 200/201.
```

**Why this output occurs**

- `[ApiController]` inspects `ModelState` automatically and returns 400 with validation errors.

---

### Real-World Cases

- **Registration forms** — Name, email, and password constraints enforced server-side.
- **File uploads** — File name and content type validated to prevent path traversal.
- **Message queue consumers** — Payload schema validated before processing.
- **Configuration** — Startup validation via `IValidateOptions<T>` prevents misconfiguration.

---

### References

- C# Secure Coding Guide (2026) — Validate input at the boundary - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- Input Validation — Microsoft Threat Modeling Tool - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- OWASP Top 10 for .NET — Input validation - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling

---

## 2. Data Annotations

### Definitions

**Core Definition**
Data Annotations are a set of attributes in the `System.ComponentModel.DataAnnotations` namespace that declaratively attach validation rules to model properties and classes.

**Technical Definition**
Data Annotations provide built-in attributes such as `[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[EmailAddress]`, `[Url]`, `[Phone]`, and `[Compare]`, plus allow-list and deny-list attributes introduced in .NET 8: `[AllowedValues]` and `[DeniedValues]`. They are applied directly to model properties and are automatically evaluated by ASP.NET Core's model binding pipeline. `[ApiController]` triggers automatic 400 responses when validation fails. Custom rules can be implemented by subclassing `ValidationAttribute` or implementing `IValidatableObject` for cross-property checks. Data Annotations are simple, well-known, and require no extra packages, but they lack support for async validation, are harder to unit test in isolation, and can clutter models with attributes.

**Beginner-Friendly Explanation**
Data Annotations are "rules written as labels on your model properties." You put `[Required]` above a property to say it must have a value, `[Range(0, 150)]` to say it must be between 0 and 150, and `[EmailAddress]` to say it must look like an email. ASP.NET Core reads these labels and automatically rejects bad requests.

---

### Purposes

- To declaratively enforce simple property-level constraints.
- To provide automatic validation in ASP.NET Core with minimal code.
- To serve as a fallback when FluentValidation is not available or appropriate.
- To document the model's constraints alongside the code.
- To support custom reusable validation attributes.

---

### Syntax Rules and Structure

#### General Syntax

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateProductRequest
{
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(200, MinimumLength = 1)]
    public required string Name { get; set; }

    [Range(0.01, 1_000_000, ErrorMessage = "Price must be between {1} and {2}")]
    public decimal Price { get; set; }

    [RegularExpression(@"^[A-Z]{2,4}-\d{4,8}$", ErrorMessage = "SKU format: AA-0000 to AAAA-00000000")]
    public string? Sku { get; set; }

    [EmailAddress]
    public string? ContactEmail { get; set; }

    [AllowedValues("Draft", "Published")]
    public string Status { get; set; } = "Draft";
}
```

**Component breakdown**

- `[Required]` — Non-null, non-empty.
- `[StringLength]` — Minimum and maximum length.
- `[Range]` — Numeric or date range.
- `[RegularExpression]` — Pattern match.
- `[EmailAddress]`, `[Url]`, `[Phone]` — Format validation.
- `[AllowedValues]` / `[DeniedValues]` — Allow-list and deny-list (.NET 8+).

#### Custom Validation Attribute

```csharp
[AttributeUsage(AttributeTargets.Property | AttributeTargets.Parameter)]
public sealed class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is DateOnly date && date <= DateOnly.FromDateTime(DateTime.UtcNow))
            return new ValidationResult(ErrorMessage ?? "Date must be in the future",
                new[] { validationContext.MemberName! });
        return ValidationResult.Success;
    }
}
```

#### Cross-Property Validation with `IValidatableObject`

```csharp
public class CreateOrderRequest : IValidatableObject
{
    public DateOnly OrderDate { get; set; }
    public DateOnly? ShipByDate { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (ShipByDate.HasValue && ShipByDate.Value <= OrderDate)
            yield return new ValidationResult("Ship-by date must be after order date",
                new[] { nameof(ShipByDate) });
    }
}
```

#### Syntax Rules

- `[ApiController]` enables automatic 400 responses on invalid `ModelState`.
- Without `[ApiController]`, the action must explicitly check `ModelState.IsValid`.
- `[Required]` on non-nullable value types is redundant; the model binder will reject missing values.
- Custom attributes must return `ValidationResult.Success` or a `ValidationResult` with an error message.
- `IValidatableObject` is for model-specific cross-property rules; use a custom attribute when the rule is reusable across models.

#### Constraints and Limitations

- **No async validation** — Data Annotations cannot check database uniqueness asynchronously.
- **Poor testability** — Attributes are hard to unit test in isolation; FluentValidation is far easier.
- **Model clutter** — Many attributes on a model reduce readability.
- **Limited composition** — Complex conditional or cross-field rules require custom attributes or `IValidatableObject`.
- **Performance** — Data Annotations are slower than FluentValidation in some benchmarks (see comparison table).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Automatic Validation with `[ApiController]`

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateProductRequest request) => Ok();
}
```

**Expected Output (conceptual)**

```
Valid payload: 200 OK.
Missing Name: 400 with {"errors":{"Name":["Product name is required"]}}.
Price=-5: 400 with range error.
```

**Why this output occurs**

- `[ApiController]` automatically checks `ModelState` and returns 400 with validation errors.

---

#### Example 2: Custom Validation Attribute

```csharp
public sealed class CreateEventRequest
{
    [Required, StringLength(200)]
    public required string Title { get; set; }

    [FutureDate(ErrorMessage = "Event date must be in the future")]
    public DateOnly EventDate { get; set; }
}
```

**Expected Output (conceptual)**

```
Past date: 400 with "Event date must be in the future".
Future date: 200 OK.
```

**Why this output occurs**

- The custom `FutureDateAttribute` evaluates the date and returns a validation error.

---

### Real-World Cases

- **Simple CRUD APIs** — `[Required]`, `[StringLength]`, and `[Range]` cover most fields.
- **Blazor forms** — `DataAnnotationsValidator` reads the same attributes.
- **Configuration validation** — `.ValidateDataAnnotations()` on options classes.

---

### References

- Part 9, add validation to an ASP.NET Core MVC app — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- The Rich Set of Data Annotation and Validation Attributes in .NET — CodeMag - https://www.codemag.com/article/1810071/The-Rich-Set-of-Data-Annotation-and-Validation-Attributes-in-.NET
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- CustomValidationAttribute Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.customvalidationattribute

---

## 3. FluentValidation

### Definitions

**Core Definition**
FluentValidation is a .NET library for building strongly typed, testable validation rules using a fluent interface, separate from the model being validated.

**Technical Definition**
FluentValidation expresses rules in classes that inherit from `AbstractValidator<T>`, using methods such as `RuleFor`, `NotEmpty`, `Length`, `EmailAddress`, `Matches`, and `Must` (or `MustAsync` for async rules). Validators are registered with the dependency injection container via `AddValidatorsFromAssemblyContaining<T>()` and can be invoked manually or automatically through ASP.NET Core's validation pipeline. FluentValidation supports **conditional validation** (`When`/`Unless`), **cross-field validation** (`Equal`, `NotEqual`, custom `Must`), **collection validation** (`RuleForEach`), **nested validation** (`SetValidator`), **async rules** (`MustAsync`, `CustomAsync`), and **custom error messages** (`.WithMessage`). It is the recommended approach for complex, testable validation in ASP.NET Core applications.

**Beginner-Friendly Explanation**
FluentValidation is "writing your validation rules as a separate, readable class." Instead of stuffing attributes onto your model, you create a `CreateUserValidator` class where you write rules like `RuleFor(x => x.Email).NotEmpty().EmailAddress()`. This keeps your model clean, makes rules easy to test, and supports advanced scenarios like checking the database for duplicate emails.

---

### Purposes

- To express complex, conditional, and cross-field validation rules cleanly.
- To separate validation logic from the model, improving separation of concerns.
- To enable unit testing of validation rules in isolation.
- To support async validation (database uniqueness, external service checks).
- To provide rich, custom error messages and error codes.
- To replace repetitive, inconsistent validation scattered across controllers.

---

### Syntax Rules and Structure

#### Setup

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<Program>();
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddFluentValidationClientsideAdapters();
```

#### Basic Validator

```csharp
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(256);

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Matches("[A-Z]").WithMessage("Must contain uppercase")
            .Matches("[a-z]").WithMessage("Must contain lowercase")
            .Matches("[0-9]").WithMessage("Must contain digit");

        RuleFor(x => x.ConfirmPassword)
            .Equal(x => x.Password).WithMessage("Passwords do not match");

        RuleFor(x => x.Age)
            .InclusiveBetween(18, 120);
    }
}
```

#### Async Validation

```csharp
RuleFor(x => x.Email)
    .MustAsync(BeUniqueEmail)
    .WithMessage("Email already exists");

private async Task<bool> BeUniqueEmail(string email, CancellationToken ct)
    => !await _repository.ExistsAsync(u => u.Email == email, ct);
```

**Critical rule**: if a validator contains any async rule, you **must** call `ValidateAsync`, never `Validate`; otherwise FluentValidation throws `AsyncValidatorInvokedSynchronouslyException`.

#### Conditional Validation

```csharp
RuleFor(x => x.ShippingAddress)
    .NotEmpty()
    .When(x => x.RequiresShipping)
    .WithMessage("Shipping address is required when shipping is needed");
```

#### Collection and Nested Validation

```csharp
RuleForEach(x => x.Items).ChildRules(item =>
{
    item.RuleFor(x => x.ProductId).GreaterThan(0);
    item.RuleFor(x => x.Quantity).InclusiveBetween(1, 1000);
});
```

#### Syntax Rules

- Validators inherit from `AbstractValidator<T>`.
- Rules are defined in the constructor using `RuleFor`.
- Async rules require `ValidateAsync`; never mix sync and async invocation.
- Register validators with DI; `AddValidatorsFromAssemblyContaining` scans the assembly.
- Use `.WithMessage()` for user-friendly errors; generic messages are an anti-pattern.

#### Constraints and Limitations

- FluentValidation performs **worse** than Data Annotations in micro-benchmarks for simple rules (see comparison).
- Validators must be registered; a missing registration causes silent validation bypass.
- Async validation adds database round-trips; ensure appropriate indexes.
- Overly complex validator classes can become difficult to navigate; split by concern.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Full Validator with Cross-Field and Async Rules

```csharp
public class CreateUserValidator : AbstractValidator<CreateUserRequest>
{
    private readonly IUserRepository _repository;

    public CreateUserValidator(IUserRepository repository)
    {
        _repository = repository;

        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
            .MustAsync(BeUniqueEmail).WithMessage("Email already exists");

        RuleFor(x => x.Password)
            .NotEmpty().MinimumLength(12)
            .Matches("[A-Z]").WithMessage("Must contain uppercase")
            .Matches("[a-z]").WithMessage("Must contain lowercase")
            .Matches("[0-9]").WithMessage("Must contain digit")
            .Matches("[^a-zA-Z0-9]").WithMessage("Must contain special character");

        RuleFor(x => x.ConfirmPassword)
            .Equal(x => x.Password).WithMessage("Passwords do not match");
    }

    private async Task<bool> BeUniqueEmail(string email, CancellationToken ct)
        => !await _repository.ExistsAsync(u => u.Email == email, ct);
}
```

**Expected Output (conceptual)**

```
Duplicate email: 400 with "Email already exists".
Weak password: 400 with specific missing-character errors.
Mismatched passwords: 400 with "Passwords do not match".
All valid: 200/201.
```

**Why this output occurs**

- Async `MustAsync` checks uniqueness.
- `Equal` enforces cross-field consistency.
- Multiple `Matches` provide precise password policy feedback.

---

#### Example 2: Manual Validation in a Controller

```csharp
[ApiController]
public class UsersController : ControllerBase
{
    private readonly IValidator<CreateUserRequest> _validator;

    public UsersController(IValidator<CreateUserRequest> validator)
        => _validator = validator;

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CreateUserRequest request)
    {
        var result = await _validator.ValidateAsync(request);
        if (!result.IsValid)
            return BadRequest(result.Errors.Select(e => new
            {
                Field = e.PropertyName,
                Message = e.ErrorMessage
            }));

        // proceed
        return Ok();
    }
}
```

**Expected Output (conceptual)**

```
Invalid request: 400 with structured field-level errors.
Valid request: 200.
```

**Why this output occurs**

- Manual invocation gives full control over the error response shape.

---

### Real-World Cases

- **Registration and login** — Password policy, email uniqueness, and confirmation matching.
- **Order processing** — Conditional shipping address, collection item validation, and async stock checks.
- **Multi-tenant SaaS** — Tenant-scoped uniqueness and cross-field business rules.
- **API gateways** — Centralised validators registered once and applied across endpoints.

---

### References

- FluentValidation ASP.NET Core documentation — https://docs.fluentvalidation.net/en/latest/aspnet.html
- How to Validate Requests with FluentValidation in .NET — OneUptime - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-fluentvalidation-dotnet/README.md
- Asynchronous Validation — FluentValidation - https://docs.fluentvalidation.net/en/latest/async.html
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- ASP.NET Validation Quick Reference — claude-dev-suite - https://github.com/claude-dev-suite/claude-dev-suite/blob/main/skills/backend-frameworks/aspnet-validation/SKILL.md

---

## 4. Validation Best Practices

### Definitions

**Core Definition**
Validation best practices are empirically grounded guidelines for implementing input validation that is secure, maintainable, performant, and user-friendly.

**Technical Definition**
Best practices span **layer selection** (Data Annotations for simple rules, FluentValidation for complex/async rules, `IValidatableObject` for model-specific cross-property logic, `IValidateOptions<T>` for configuration), **boundary discipline** (validate at the trust boundary, not deep in the call stack), **server authority** (never rely on client-side validation), **error handling** (specific, actionable messages; no stack traces or internal details), **anti-pattern avoidance** (no validation in controllers, no client-only validation, no generic error messages), and **performance awareness** (Data Annotations are faster for simple rules; FluentValidation is slower but more capable).

**Beginner-Friendly Explanation**
Validation best practices are "the habits that keep validation secure and maintainable." They include: validate on the server, validate at the edge, use the right tool for the job, write specific error messages, and never skip validation just because the frontend checked it.

---

### Purposes

- To prevent validation bypass and injection attacks.
- To keep validation logic centralised, testable, and consistent.
- To provide a good developer and user experience.
- To avoid performance pitfalls from overly expensive validators.
- To ensure configuration is validated at startup rather than failing at runtime.
- To satisfy security review and compliance requirements.

---

### Guidelines and Structure

#### Choose the Right Tool

| Scenario | Recommended Approach |
|---|---|
| **Simple property constraints** | Data Annotations (`[Required]`, `[Range]`, `[StringLength]`) |
| **Complex conditional rules** | FluentValidation (`When`, `Unless`, cross-field) |
| **Async database checks** | FluentValidation (`MustAsync`) |
| **Model-specific cross-property rules** | `IValidatableObject` |
| **Reusable custom rule** | Custom `ValidationAttribute` subclass |
| **Configuration validation** | `IValidateOptions<T>` with `ValidateOnStart` |

**Source: validation-patterns decision tree**.

#### Validate at the Boundary, Not Deep in the Call Stack

```csharp
// ✅ Validate at the edge
if (!long.TryParse(id, out var validId) || validId <= 0)
    return BadRequest();

// ❌ Hoping deep code handles it
var entity = _repository.Get(long.Parse(id)); // throws on bad input
```

**Source: C# Secure Coding Guide (2026)**.

#### Always Validate on the Server

```csharp
// ✅ Server-side validation is the security control
[ApiController]
public class UsersController : ControllerBase { ... }

// ❌ Client-side only — trivially bypassed
```

**Source: ASP.NET Validation Quick Reference — anti-patterns**.

#### Use Specific, Actionable Error Messages

```csharp
// ✅ Specific
.WithMessage("Email must contain at least one uppercase letter")

// ❌ Generic
.WithMessage("Invalid input")
```

**Source: ASP.NET Validation Quick Reference — anti-patterns**.

#### Never Put Validation in Controllers

```csharp
// ❌ Repetitive, inconsistent, hard to test
if (string.IsNullOrEmpty(request.Name)) return BadRequest();

// ✅ Use FluentValidation + filter or [ApiController]
```

**Source: ASP.NET Validation Quick Reference — anti-patterns**.

#### Handle Async Validation Correctly

- Use `MustAsync` / `CustomAsync` for database checks.
- Always call `ValidateAsync` when any async rule exists; `Validate` throws `AsyncValidatorInvokedSynchronouslyException`.
- Ensure database columns used in async checks are indexed.

**Source: FluentValidation async documentation**.

#### Validate Configuration at Startup

```csharp
builder.Services.AddOptions<DatabaseOptions>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

- Misconfiguration should fail fast at startup, not on the first request.

**Source: validation-patterns — `IValidateOptions<T>`**.

#### Avoid Dangerous Regular Expressions

- Use built-in attributes (`[EmailAddress]`, `[Url]`) instead of hand-rolled regex when possible.
- Set a `RegexOptions` timeout when regex must touch untrusted input to prevent ReDoS.
- Prefer bounded quantifiers.

**Source: C# Secure Coding Guide (2026); OWASP guidance**.

#### Validate Collections and Nested Objects

- Use `RuleForEach` and `ChildRules` (FluentValidation) to validate each item.
- Missing nested validation is a common source of invalid data reaching the database.

**Source: FluentValidation collection validation examples**.

#### Performance Awareness

- Data Annotations are faster for simple rules; FluentValidation is 2–5× slower but far more capable.
- For hot paths with simple rules, Data Annotations may be the better choice.
- Do not use async validation unless the rule genuinely requires I/O.

**Source: CSharpEssentials validation benchmarks**.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure Boundary Validation

```csharp
app.MapGet("/users/{id}", (string id) =>
{
    if (!long.TryParse(id, out var userId) || userId <= 0)
        return Results.BadRequest("Invalid user id");

    return Results.Ok(userId);
});
```

**Expected Output (conceptual)**

```
/users/abc: 400.
/users/-1: 400.
/users/42: 200.
```

**Why this output occurs**

- `TryParse` and the range check reject invalid input at the boundary before any business logic runs.

---

#### Example 2: Centralised FluentValidation with Automatic 400

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<Program>();
builder.Services.AddFluentValidationAutoValidation();

[ApiController]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateUserRequest request) => Ok();
}
```

**Expected Output (conceptual)**

```
Invalid request: 400 with field-level errors automatically.
Valid request: 200.
```

**Why this output occurs**

- Auto-validation plugs into ASP.NET Core's pipeline.
- `[ApiController]` returns 400 automatically when validation fails.

---

### Real-World Cases

- **Financial APIs** — Validate monetary amounts, account numbers, and currency codes at the boundary.
- **Healthcare systems** — Validate patient identifiers and date ranges with cross-field rules.
- **E-commerce** — Validate order quantities, shipping addresses, and payment details with conditional rules.
- **Multi-tenant SaaS** — Validate tenant-scoped uniqueness asynchronously.

---

### References

- C# Secure Coding Guide (2026) — Input validation patterns - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- ASP.NET Validation Quick Reference — claude-dev-suite - https://github.com/claude-dev-suite/claude-dev-suite/blob/main/skills/backend-frameworks/aspnet-validation/SKILL.md
- FluentValidation ASP.NET Core documentation — https://docs.fluentvalidation.net/en/latest/aspnet.html
- Asynchronous Validation — FluentValidation - https://docs.fluentvalidation.net/en/latest/async.html
- CSharpEssentials.Validation benchmarks — https://github.com/senrecep/csharpessentials/blob/main/benchmarks/results/validation/net9.0/results/
- Input Validation — Microsoft Threat Modeling Tool - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling

---

## Summary Table: Data Annotations vs. FluentValidation

| Feature | Data Annotations | FluentValidation |
|---|---|---|
| **Setup** | Built-in; no packages | NuGet packages required |
| **Simple rules** | Good | Good |
| **Complex conditions** | Limited | Excellent |
| **Cross-field validation** | `IValidatableObject` or custom attribute | Built-in (`Equal`, `Must`) |
| **Async validation** | Not supported | Built-in (`MustAsync`) |
| **Testability** | Hard | Easy (unit-test validators) |
| **Reusable rules** | Custom attributes | First-class |
| **Performance (simple)** | Faster (~1,383 ns for invalid; ~1,064 ns for valid) | Slower (~3,062 ns for invalid; ~1,854 ns for valid) |
| **Model cleanliness** | Attributes clutter model | Separate validator class |
| **Best for** | Simple property-level constraints | Complex, conditional, async, testable rules |

*Performance figures from CSharpEssentials.Validation benchmarks on .NET 9*.

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **Client-side-only validation** | Trivially bypassed; server-side validation is mandatory |
| **Validation in controllers** | Anti-pattern; use FluentValidation + filter or `[ApiController]` |
| **`Validate` with async rules** | Throws `AsyncValidatorInvokedSynchronouslyException`; use `ValidateAsync` |
| **Generic error messages** | Poor UX; use specific `.WithMessage()` |
| **Missing validator registration** | Silent validation bypass; verify `AddValidatorsFromAssemblyContaining` |
| **`[AllowedValues]` / `[DeniedValues]`** | Introduced in .NET 8; version-specific |
| **`IValidateOptions<T>` with `ValidateOnStart`** | Prevents runtime misconfiguration; recommended for configuration |
| **Regex without timeout** | ReDoS risk; set a timeout or use built-in attributes |
| **`BinaryFormatter`** | Obsolete; use `System.Text.Json` |

---

## References (Consolidated)

- C# Secure Coding Guide (2026) — Validate input at the boundary - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- ASP.NET Validation Quick Reference — claude-dev-suite - https://github.com/claude-dev-suite/claude-dev-suite/blob/main/skills/backend-frameworks/aspnet-validation/SKILL.md
- FluentValidation ASP.NET Core documentation — https://docs.fluentvalidation.net/en/latest/aspnet.html
- How to Validate Requests with FluentValidation in .NET — OneUptime - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-25-fluentvalidation-dotnet/README.md
- Asynchronous Validation — FluentValidation - https://docs.fluentvalidation.net/en/latest/async.html
- CustomValidationAttribute Class — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.customvalidationattribute
- The Rich Set of Data Annotation and Validation Attributes in .NET — CodeMag - https://www.codemag.com/article/1810071/The-Rich-Set-of-Data-Annotation-and-Validation-Attributes-in-.NET
- Part 9, add validation to an ASP.NET Core MVC app — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- Input Validation — Microsoft Threat Modeling Tool - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling
- CSharpEssentials.Validation benchmarks — https://github.com/senrecep/csharpessentials/blob/main/benchmarks/results/validation/net9.0/results/
- OWASP Top 10 for .NET — Input validation - https://learn.microsoft.com/en-us/training/modules/design-solutions-secure-applications/4-evaluate-application-threats-threat-modeling