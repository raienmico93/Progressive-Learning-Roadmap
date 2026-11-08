# ASP.NET Core Secure Coding: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **secure coding** discipline — the day‑to‑day engineering practices that prevent vulnerabilities from entering the codebase. It complements the earlier "Common Vulnerabilities" cheat sheet: that one describes the *threats*; this one describes the *habits* that stop them. Secure coding is not a library you install — it is a set of defaults, patterns, and review disciplines applied consistently at every trust boundary: input validation, output encoding, parameterized queries, least privilege, secure defaults, disciplined error handling, safe logging, and secret management. Each requested topic is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secure coding is the practice of writing application code that is resistant to attack by design — validating untrusted input, encoding output for its destination context, parameterizing data access, enforcing least privilege, defaulting to safe configurations, failing safely, logging without leaking, and managing secrets correctly. |
| **Technical Definition** | Secure coding in ASP.NET Core applies the framework's security primitives — `ModelState` validation, Razor's automatic HTML encoding, EF Core parameterization, `[Authorize]`/fallback policies, `ProblemDetails`, `ILogger` with redaction, `IDataProtectionProvider`, and User Secrets/vaults — at every **trust boundary**. The guiding principles are **validate at the boundary**, **encode at the sink**, **parameterize at the data layer**, **deny by default**, **fail closed**, and **never log or commit secrets**. |
| **Beginner‑Friendly Explanation** | Secure coding is **writing code with the attacker in mind**. It is not about adding a security feature at the end — it is about every line of code doing the safe thing by default. Validate what users send you. Encode what you send back. Never glue user input into SQL. Give every account the minimum permissions. Make the app safe unless you explicitly change it. When something fails, fail safely and log the details where only you can see them. Keep secrets in a vault, not in code. |

### Key Characteristics

- **Boundary‑driven** — Every external input (HTTP request, file upload, message queue, database, external API) is untrusted until validated.
- **Sink‑aware** — Encoding depends on the destination: HTML, attribute, JavaScript, URL, CSS, SQL, or shell. There is no universal "escape" function.
- **Default‑deny** — Authorization, error responses, and configuration all start from the safe option.
- **Least privilege** — Users, services, database accounts, and containers receive only the permissions required for their function.
- **Fail closed** — When a security decision cannot be made, access is denied, not granted.
- **Non‑leaky observability** — Logs and error responses provide enough context to diagnose, but never enough to attack.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **dependency injection**, **model binding**, **EF Core**, and **claims‑based identity**.
- Familiarity with **HTTP**, **cookies**, **TLS**, and **SQL**.
- Basic understanding of the **OWASP Top 10** and **CWE Top 25**.

### Related Programming Areas

- **Authentication and authorization** — The identity layer and access control.
- **Data access** — EF Core, Dapper, and raw ADO.NET.
- **Razor views and APIs** — The output sinks where encoding matters.
- **Logging and monitoring** — Detecting attacks and diagnosing failures without leaking.
- **Configuration and secrets** — The settings that make or break security.
- **CI/CD** — Automated review, SAST, and dependency scanning.

### Core Concepts / Features

1. **Secure Coding** — The overarching discipline and mental model.
2. **Input Validation** — Reject malformed or malicious input at the boundary.
3. **Output Encoding** — Encode for the destination context so data is never executed as code.
4. **Parameterized Queries** — Separate SQL code from data.
5. **Least Privilege** — Minimum permissions for every identity.
6. **Secure Defaults** — Safe unless explicitly changed.
7. **Error Handling** — Fail closed, respond safely, and avoid leaking details.
8. **Logging** — Record security‑relevant events without recording secrets.
9. **Secret Management** — Keep credentials out of source and configuration files.
10. **Secure Coding Best Practices** — The review checklist that ties it all together.

---

## 1. Secure Coding (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secure coding is the systematic practice of writing code that resists attack by applying validation, encoding, parameterization, least privilege, secure defaults, safe error handling, and secret management at every trust boundary. |
| **Technical Definition** | Secure coding in ASP.NET Core combines **framework primitives** (`ModelState` validation, Razor encoding, EF Core parameterization, `[Authorize]` and fallback policies, `ProblemDetails`, `ILogger`, `IDataProtectionProvider`, User Secrets and vaults) with **engineering discipline** (code review, threat modeling, static analysis, dependency scanning, and security testing). The goal is to make the **secure path the easy path** and the insecure path **impossible or obvious**. |
| **Beginner‑Friendly Explanation** | Secure coding is the **"do the safe thing automatically"** habit. A secure codebase does not rely on developers remembering to add authorization or encode output — the framework defaults, the code patterns, and the review process all push toward safety. When a developer writes something dangerous, it should stand out in review and fail in tests. |

### Purposes

- **To** prevent vulnerabilities from being introduced in the first place, rather than fixing them in production.
- **To** make the secure implementation the path of least resistance, so safety does not depend on memory.
- **To** provide a consistent, reviewable set of patterns across the team and the codebase.
- **To** align with the framework's built‑in protections and the OWASP Top 10 mitigations.
- **To** reduce the cost of security — fixing a design flaw before merge is orders of magnitude cheaper than an incident.

### Syntax Rules and Structure

#### The Secure Coding Loop

```text
Identify trust boundary → Validate input → Authorize access → Process with parameterized data access
→ Encode output for destination → Fail safely → Log without secrets → Review
```

**Component breakdown:**

| Phase | Activity | ASP.NET Core primitive |
|-------|----------|------------------------|
| **Boundary** | Identify where untrusted data enters. | HTTP request, file upload, queue message |
| **Validate** | Reject malformed input. | `ModelState`, data annotations, FluentValidation |
| **Authorize** | Deny by default; check ownership. | `[Authorize]`, fallback policy, `IAuthorizationService` |
| **Data access** | Parameterize queries. | LINQ, `FromSqlInterpolated` |
| **Output** | Encode for the destination. | Razor `@`, `HtmlEncoder`, JSON serialization |
| **Fail** | Return safe errors; never leak. | `ProblemDetails`, `UseExceptionHandler` |
| **Log** | Record events without secrets. | `ILogger`, structured logging with redaction |
| **Review** | Check the pattern in code review. | PR checklist, SAST |

**Syntax rules:**

- **Every trust boundary gets validation** — a boundary without validation is a vulnerability waiting to happen.
- **Every output sink gets the correct encoding** — HTML encoding in HTML context, JavaScript encoding in script context.
- **Every endpoint gets authorization** — a fallback policy makes this the default.

**Constraints and limitations:**

- **Secure coding cannot fix a broken architecture** — if a service trusts another service without authentication, no amount of input validation helps.
- **Framework defaults can be overridden** — `Html.Raw`, `FromSqlRaw` with concatenation, and `[AllowAnonymous]` all opt out of safety; review must catch them.

### Annotated Complete Step‑by‑Step Code Example (Secure Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Deny by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// 2. Validate input via data annotations
builder.Services.AddControllersWithViews();

// 3. Safe errors
builder.Services.AddProblemDetails();

var app = builder.Build();

// 4. Fail safely
app.UseExceptionHandler();
app.UseStatusCodePages();

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. The fallback policy requires authentication for every endpoint without explicit metadata.
2. MVC's model validation enforces data annotations on DTOs.
3. `ProblemDetails` gives a consistent, non‑leaking error shape.
4. `UseExceptionHandler` and `UseStatusCodePages` prevent stack traces from reaching clients.

**Expected Output:** Unannotated endpoints are protected; invalid models return 400 with a structured error; unhandled exceptions return 500 with a generic `ProblemDetails` body.

**Why this output?** Each step applies a secure default, so the application is safe unless a developer explicitly opts out.

### Real‑World Cases

- **Public APIs:** Fallback policy, DTO validation, `ProblemDetails`, parameterized EF Core.
- **Internal microservices:** JWT authentication, least‑privilege service accounts, structured logging.
- **Regulated industries:** Secrets from a vault, audit logging without PII, tested authorization.

### References

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- OWASP Top 10 (2021) – https://owasp.org/Top10/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html

---

## 2. Input Validation

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Input validation is the practice of checking that data received from an untrusted source is well‑formed, within expected bounds, and of the expected type before it is used. |
| **Technical Definition** | ASP.NET Core validates input through **model binding** (type conversion errors populate `ModelState`) and **validation attributes** (`[Required]`, `[StringLength]`, `[Range]`, `[RegularExpression]`, `[EmailAddress]`, `[Compare]`) or **FluentValidation**. `ModelState.IsValid` gates processing; `[ApiController]` automatically returns **400 Bad Request** with `ValidationProblemDetails` for invalid models. Validation must occur **at the boundary** and must never be the only defense for injection or XSS — it is a **first line**, not a replacement for encoding and parameterization. |
| **Beginner‑Friendly Explanation** | Input validation is the **"check the package at the door"** habit. Before your code uses a value, it verifies the value is what it expects: a required field is present, a string is not too long, a number is in range, an email looks like an email. If the value fails, the request is rejected with a clear error. Validation catches mistakes early and reduces the attack surface, but it does **not** replace encoding or parameterization — those are separate layers. |

### Purposes

- **To** reject malformed or unexpected input before it reaches business logic or the database.
- **To** provide clear, actionable error messages to clients through `ModelState` and `ProblemDetails`.
- **To** enforce business and domain rules at the boundary (length, range, format, required).
- **To** reduce the surface for injection, XSS, and logic bugs by constraining input shape.
- **To** generate client-side validation automatically when using data annotations with jQuery Unobtrusive.

### Syntax Rules and Structure

#### Data Annotations on a DTO

```csharp
public class CreateProductDto
{
    [Required, StringLength(100, MinimumLength = 3)]
    public string Name { get; set; } = "";

    [Range(0.01, 1_000_000)]
    public decimal Price { get; set; }

    [EmailAddress]
    public string? ContactEmail { get; set; }

    [RegularExpression(@"^[A-Z]{2}-\d{4}$")]
    public string Sku { get; set; } = "";
}
```

#### Controller Validation Check

```csharp
[HttpPost]
public IActionResult Create(CreateProductDto input)
{
    if (!ModelState.IsValid) return BadRequest(ModelState);
    // ...
}
```

**Syntax rules:**

- **Validate the DTO, not the domain entity** — DTOs restrict which properties can be bound (mass‑assignment protection).
- **Use the most specific attribute** — `[EmailAddress]` over `[RegularExpression]` for emails, `[Range]` over `[RegularExpression]` for numbers.
- **`[ApiController]` handles 400 automatically** — the action does not need an explicit `ModelState.IsValid` check, though checking is fine for clarity.
- **Validation is a first line, not the only line** — encoding and parameterization are still required.

**Constraints and limitations:**

- **Validation is not sanitization** — a valid string can still contain malicious HTML or SQL; encode and parameterize.
- **`[Required]` does not trim whitespace** — a whitespace‑only string passes `[Required]` unless a custom rule is added.
- **Client‑side validation is a UX optimization** — server‑side validation is authoritative.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public record CreateUserDto(
    [property: Required, StringLength(50, MinimumLength = 3)] string Username,
    [property: Required, EmailAddress] string Email,
    [property: Required, MinLength(12)] string Password);

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> Create(CreateUserDto input, CancellationToken ct)
    {
        // [ApiController] returns 400 automatically when ModelState is invalid
        var user = await _service.CreateAsync(input, ct);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }
}
```

**Step-by-step explanation:**

1. The DTO declares validation rules on each property.
2. Model binding enforces the rules; failures populate `ModelState`.
3. `[ApiController]` short‑circuits with `400 Bad Request` and `ValidationProblemDetails` when validation fails.
4. The action runs only for valid input.

**Expected Output:** An invalid email or short password returns `400 Bad Request` with a structured error listing the failed fields; valid input proceeds to create the user.

**Why this output?** Model validation at the boundary prevents invalid data from reaching the service layer.

### Real-World Cases

- **Registration forms:** Username length, email format, password strength.
- **Checkout forms:** Shipping address required fields, quantity ranges.
- **Search endpoints:** Query‑string length limits, page size bounds.

### References

- Model validation in ASP.NET Core MVC and Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- OWASP Input Validation Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html

---

## 3. Output Encoding

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Output encoding is the practice of converting data into a safe representation for the destination context — HTML, HTML attribute, JavaScript, URL, CSS, or SQL — so that it is displayed as data and never executed as code. |
| **Technical Definition** | Razor **HTML‑encodes all variable output** by default — `@variable` encodes `<`, `>`, `&`, `"`, and `'` into HTML entities. Contexts differ: HTML body requires HTML encoding; inside `<script>` requires JavaScript encoding; inside an attribute requires attribute encoding; inside a URL requires URL encoding. ASP.NET Core provides `HtmlEncoder`, `JavaScriptEncoder`, `UrlEncoder`, and `JsonSerializer` for each context. `Html.Raw` bypasses HTML encoding and is safe only for trusted, already‑sanitized content. |
| **Beginner‑Friendly Explanation** | Output encoding is the **"label the data so it is read, not run"** habit. When you put user data on a page, you must mark it as data. In HTML, `<` becomes `&lt;` so a script tag is displayed as text, not executed. Different contexts need different encodings — HTML is not JavaScript, and JavaScript is not a URL. Razor does HTML encoding automatically; your job is to avoid bypassing it (`Html.Raw`) and to use the right encoder for non‑HTML contexts. |

### Purposes

- **To** prevent XSS by ensuring data is never interpreted as executable markup or script.
- **To** produce correct output in each context — HTML, attribute, JavaScript, URL, CSS.
- **To** rely on framework encoding by default and only encode manually where the framework cannot help.
- **To** make `Html.Raw` and other bypasses rare, reviewed, and justified.
- **To** complement Content Security Policy (CSP) as a defense‑in‑depth control.

### Syntax Rules and Structure

#### HTML Context — Razor `@` (Automatic Encoding)

```cshtml
<p>@Model.Comment</p>
```

#### HTML Attribute Context — Razor `@` (Attribute Encoding)

```cshtml
<div title="@Model.Title">...</div>
```

#### JavaScript Context — JSON Serialization

```cshtml
<script>
    var data = @Html.Raw(JsonSerializer.Serialize(Model.Data));
</script>
```

#### Manual Encoding

```csharp
using System.Text.Encodings.Web;

var safeHtml = HtmlEncoder.Default.Encode(userInput);
var safeJs = JavaScriptEncoder.Default.Encode(userInput);
var safeUrl = UrlEncoder.Default.Encode(userInput);
```

#### ❌ Vulnerable: `Html.Raw` with Untrusted Input

```cshtml
@Html.Raw(Model.Comment)   @* NEVER with untrusted input *@
```

**Syntax rules:**

- **Razor's `@` encodes for the current context** — HTML body and attribute contexts are handled automatically.
- **JavaScript contexts require JSON serialization** — HTML encoding is insufficient inside `<script>`.
- **URL contexts require `UrlEncoder`** — do not concatenate raw input into URLs.
- **`Html.Raw` is an explicit opt‑out** — use it only for trusted, sanitized content.

**Constraints and limitations:**

- **HTML encoding is not JavaScript encoding** — using the wrong encoder leaves XSS holes.
- **Encoding does not sanitize** — if you intentionally render user HTML, you must use an HTML sanitizer (e.g., a library) in addition to encoding.
- **DOM‑based XSS** happens in client‑side JavaScript — server‑side encoding cannot prevent it.

### Annotated Complete Step‑by‑Step Code Example

```cshtml
@model CommentPageModel

@* Safe: Razor HTML-encodes the comment *@
<div class="comment">@Model.Comment</div>

@* Safe: attribute encoding *@
<a href="/users/@UrlEncoder.Default.Encode(Model.Username)">@Model.Username</a>

@* Safe: JSON encoding inside script *@
<script>
    var commentId = @JsonSerializer.Serialize(Model.CommentId);
</script>
```

**Step-by-step explanation:**

1. The comment is rendered with `@` — Razor HTML‑encodes it, so `<script>` becomes `&lt;script&gt;`.
2. The username in the URL is encoded with `UrlEncoder`.
3. The comment ID inside the script is serialized with `JsonSerializer`, producing a safe JavaScript literal.

**Expected Output:** Malicious HTML in the comment is displayed as text; the URL contains no injected characters; the script receives a properly encoded value.

**Why this output?** Each sink uses the encoder appropriate for its context.

### Real-World Cases

- **Comment sections:** User‑submitted text must be encoded before display.
- **User profiles:** Display names and bios are common XSS vectors.
- **APIs returning HTML fragments:** Must encode before returning.

### References

- Prevent Cross-Site Scripting (XSS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- OWASP Cross Site Scripting Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- `HtmlEncoder` class – https://learn.microsoft.com/en-us/dotnet/api/system.text.encodings.web.htmlencoder

---

## 4. Parameterized Queries

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Parameterized queries separate SQL code from data by sending user values as parameters, so the database treats them as data and never as executable SQL syntax. |
| **Technical Definition** | EF Core's LINQ provider generates parameterized SQL automatically — `Where(u => u.Email == email)` sends `email` as a `DbParameter`, not as text in the SQL string. `FromSqlInterpolated` decomposes an interpolated string into parameters. `FromSqlRaw` is safe only when the SQL is a constant with `{0}` placeholders and the values are passed separately. ADO.NET `SqlCommand.Parameters` and Dapper's anonymous parameter objects provide the same protection outside EF Core. |
| **Beginner‑Friendly Explanation** | Parameterized queries are the **"keep the code and the data separate"** habit. Instead of building `"SELECT * FROM Users WHERE Email = '" + email + "'"`, you write `"SELECT * FROM Users WHERE Email = @Email"` and pass `email` as a parameter. The database treats `@Email` as a value, not as SQL, so an attacker cannot inject `' OR 1=1--`. EF Core does this automatically for LINQ; the danger is in raw SQL. |

### Purposes

- **To** prevent SQL injection by design, not by escaping or filtering.
- **To** leverage the database's prepared‑statement caching for performance.
- **To** avoid the fragility of manual escaping — parameterization is unambiguous.
- **To** enforce type safety between application values and database columns.
- **To** provide a clear, reviewable pattern that static analysis can verify.

### Syntax Rules and Structure

#### Safe: LINQ (Parameterized by Default)

```csharp
var user = await _db.Users
    .Where(u => u.Email == email)
    .FirstOrDefaultAsync(ct);
```

#### Safe: `FromSqlInterpolated`

```csharp
var users = await _db.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Email = {email}")
    .ToListAsync(ct);
```

#### Safe: `FromSqlRaw` with Placeholders

```csharp
var users = await _db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Email = {0}", email)
    .ToListAsync(ct);
```

#### Safe: ADO.NET Parameters

```csharp
using var cmd = new SqlCommand(
    "SELECT * FROM Users WHERE Email = @Email", connection);
cmd.Parameters.AddWithValue("@Email", email);
```

#### ❌ Vulnerable: String Concatenation

```csharp
var users = await _db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Email = '" + email + "'")
    .ToListAsync(ct);
```

**Syntax rules:**

- **LINQ is parameterized by default** — no action needed beyond not using raw SQL.
- **`FromSqlInterpolated` is safe** — the interpolation is decomposed into parameters.
- **`FromSqlRaw` must use `{0}` placeholders** — never concatenate values.
- **Dynamic identifiers** (column names, sort directions) cannot be parameterized — use an allowlist.

**Constraints and limitations:**

- **`FromSqlRaw` with concatenation is the primary EF Core injection vector** — audit every usage.
- **Stored procedures that build dynamic SQL internally** are equally vulnerable — review them.
- **`EF.Functions.Like`** is safe if the pattern is parameterized; escape `%` and `_` from user input if literal matching is intended.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public async Task<List<Order>> SearchOrdersAsync(
    string customerName, OrderStatus status, CancellationToken ct)
{
    return await _db.Orders
        .AsNoTracking()
        .Where(o => o.Customer.Name == customerName && o.Status == status)
        .ToListAsync(ct);
}
```

**Step-by-step explanation:**

1. The LINQ query expresses the filter with C# expressions.
2. EF Core translates `customerName` and `status` into `@p0` and `@p1` parameters.
3. The database receives the SQL text and the parameter values separately.

**Expected Output:** The query returns only orders matching the provided name and status; malicious input is treated as a literal value and matches nothing.

**Why this output?** The values are parameters, so they cannot alter the SQL structure.

### Real-World Cases

- **Login and search:** The most common injection targets.
- **Reporting queries:** Raw SQL with user‑supplied filters.
- **Stored procedures:** Parameterized calls to procedures that do not build dynamic SQL internally.

### References

- SQL Queries (EF Core) – https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- OWASP Query Parameterization Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html

---

## 5. Least Privilege

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Least privilege is the principle that every user, service, process, and database account should receive only the minimum permissions necessary to perform its function. |
| **Technical Definition** | Least privilege in ASP.NET Core applies at every layer: **users** receive the minimum roles and claims required; **API clients** receive the minimum scopes; **database accounts** use `db_datareader` and `db_datawriter` rather than `db_owner` or `sa`; **managed identities** receive precise RBAC roles rather than broad subscription‑level access; **containers** run as non‑root with a read‑only filesystem; and **CI/CD identities** can deploy but not read production secrets unless required. |
| **Beginner‑Friendly Explanation** | Least privilege is the **"minimum key for each lock"** habit. Your application's database account should not be able to drop tables — it only needs to read and write data. A user who needs to view reports should not be an admin. A container should not run as root. The principle limits the damage an attacker can do if they compromise one identity: a stolen low‑privilege credential cannot do much. |

### Purposes

- **To** limit the blast radius of a compromised identity — stolen credentials grant minimal access.
- **To** prevent accidental damage — a buggy service cannot drop production tables.
- **To** satisfy compliance requirements (PCI DSS, SOC 2, NIST 800‑53 AC‑6).
- **To** make each identity's purpose narrow and auditable.
- **To** reduce the attack surface of the application's own credentials.

### Syntax Rules and Structure

#### Database Least Privilege

```sql
-- Runtime account: read and write data only
CREATE USER app_user FROM LOGIN app_user;
ALTER ROLE db_datareader ADD MEMBER app_user;
ALTER ROLE db_datawriter ADD MEMBER app_user;

-- Migration account: schema changes only, used by the deployment pipeline
-- ALTER ROLE db_ddladmin ADD MEMBER migration_user;
```

#### Application Identity via Managed Identity

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        "Server=myserver.database.windows.net;Database=MyDb;" +
        "Authentication=Active Directory Managed Identity"));
```

#### Container Least Privilege

```dockerfile
# Run as a non-root user
USER app
```

#### ❌ Over‑Privileged

```csharp
// The app account is sa or db_owner — never do this
var connectionString = "Server=...;User Id=sa;Password=...";
```

**Syntax rules:**

- **The runtime database account must never be `sa` or `db_owner`** — grant only `db_datareader` and `db_datawriter`.
- **Schema changes use a separate migration identity** with `db_ddladmin`, invoked by the deployment pipeline.
- **Prefer managed identities over secrets** — they eliminate credentials and can be scoped precisely.
- **Containers should run as non‑root** and not allow privilege escalation.

**Constraints and limitations:**

- **Least privilege requires knowing what "minimum" is** — start restrictive and grant as evidence demands.
- **Overly restrictive permissions break functionality** — balance security with operability.
- **Role memberships are inherited** — audit group memberships, not just direct grants.

### Annotated Complete Step‑by‑Step Code Example

```csharp
// AppDbContext uses a least-privilege connection
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("AppReadWrite")));

// Migrations use a separate connection with DDL permissions, only in deployment
builder.Services.AddDbContext<MigrationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Migration")));
```

**Step-by-step explanation:**

1. The runtime `AppDbContext` connects with an account that can only read and write data.
2. A separate `MigrationDbContext` uses an account with `db_ddladmin` and is used only by the deployment pipeline.
3. A compromised runtime account cannot alter the schema or drop tables.

**Expected Output:** The application reads and writes data normally; schema changes are applied by the migration identity during deployment.

**Why this output?** Separating runtime and migration identities enforces least privilege at the database layer.

### Real-World Cases

- **Multi‑tenant SaaS:** Tenant‑scoped roles and claims; the runtime account cannot cross tenant boundaries.
- **Microservices:** Each service has its own database account with access only to its schema.
- **CI/CD pipelines:** Deployment identities can deploy but cannot read production secrets they do not need.

### References

- Security in ASP.NET Core (least privilege principles) – https://learn.microsoft.com/en-us/aspnet/core/security/
- OWASP Authorization Cheat Sheet (least privilege) – https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- NIST SP 800‑53 AC‑6 (Least Privilege) – https://csf.tools/reference/nist-sp-800-53/r5/ac/ac-6/

---

## 6. Secure Defaults

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secure defaults are the principle that the application's default configuration should be the most secure option — the developer must explicitly opt out of security, not opt in. |
| **Technical Definition** | In ASP.NET Core, secure defaults include a **fallback authorization policy** requiring authentication for all endpoints without explicit metadata; **HTTPS redirection and HSTS** outside Development; **antiforgery** enabled for cookie‑based apps; **detailed errors disabled** in production; **secure cookie attributes** (`Secure`, `HttpOnly`, `SameSite`); and **secret management** that requires explicit configuration rather than defaulting to insecure values. The default posture is **closed** — public endpoints must be explicitly marked `[AllowAnonymous]`. |
| **Beginner‑Friendly Explanation** | Secure defaults mean **"the door is locked unless you unlock it."** If a developer forgets to add `[Authorize]`, the endpoint should deny access, not allow everyone. If a deployment forgets to disable the developer exception page, the default should already be off in production. Secure defaults protect against human error, which is the most common cause of vulnerabilities. |

### Purposes

- **To** protect against forgotten security controls — a missing `[Authorize]` fails closed, not open.
- **To** reduce the attack surface by default — diagnostics, detailed errors, and Swagger UI are off unless enabled.
- **To** enforce secure transport by default — HTTPS and HSTS are on outside Development.
- **To** eliminate insecure fallback values — secrets do not default to weak or empty values.
- **To** make the secure path the easy path.

### Syntax Rules and Structure

#### Fallback Authorization Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

#### Environment‑Gated Diagnostics

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    app.UseExceptionHandler();
    app.UseStatusCodePages();
}
```

#### Secure Cookies

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.HttpOnly = true;
    options.Cookie.SameSite = SameSiteMode.Strict;
});
```

**Syntax rules:**

- **The fallback policy is the single most important secure default** — it makes authorization default‑deny.
- **Diagnostics must be environment‑gated** — never in production by default.
- **Cookies should set `Secure`, `HttpOnly`, and `SameSite`** — the framework defaults vary by cookie type.

**Constraints and limitations:**

- **Secure defaults can break existing functionality** — endpoints that relied on being public will now deny access; this is intended.
- **The fallback policy does not apply to endpoints with `[Authorize]`** — those use the default policy instead.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Default deny
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser().Build();
});

// Secure cookies
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.HttpOnly = true;
    options.Cookie.SameSite = SameSiteMode.Strict;
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. The fallback policy protects all endpoints without explicit metadata.
2. Cookies are hardened with `Secure`, `HttpOnly`, and `SameSite=Strict`.
3. HTTPS and HSTS are enforced outside Development.
4. Authentication precedes authorization.

**Expected Output:** A new endpoint added without `[AllowAnonymous]` is automatically protected; cookies are not accessible to JavaScript and are only sent over HTTPS.

**Why this output?** The defaults are secure; developers must explicitly opt out for public endpoints.

### Real-World Cases

- **Public APIs:** Fallback policy plus explicit `[AllowAnonymous]` on truly public endpoints.
- **MVC/Razor Pages:** Secure cookies and antiforgery by default.
- **Microservices:** HTTPS and HSTS by default; health checks explicitly anonymous.

### References

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- Policy-based authorization (fallback policy) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- OWASP Secure Defaults (CIS Controls) – https://www.cisecurity.org/controls

---

## 7. Error Handling

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Error handling is the practice of responding to failures safely — returning controlled, non‑leaking error responses to clients, logging details server‑side, and ensuring that security decisions fail closed. |
| **Technical Definition** | ASP.NET Core error handling includes `UseExceptionHandler` (catches unhandled exceptions and re‑executes the pipeline at an error endpoint), `UseStatusCodePages` (generates bodies for error status codes), `ProblemDetails` (a standardized, RFC 9457 error shape), and the `[ApiController]` automatic 400 response for invalid models. The principle is **fail safely**: return 500 with a generic message and a correlation ID, log the exception with full detail server‑side, and never send stack traces, SQL errors, or internal paths to clients. Authorization decisions must **fail closed** — if the authorization service is unavailable, deny access. |
| **Beginner‑Friendly Explanation** | Error handling is the **"fail gracefully without giving away the blueprint"** habit. When something goes wrong, the user should see a friendly error message, not a stack trace with your file paths and SQL. The details go into the logs where only you can see them. And if the security system itself fails — for example, the authorization service is down — the safe behavior is to deny access, not to let everyone in. |

### Purposes

- **To** prevent information disclosure through stack traces, SQL errors, and internal paths.
- **To** provide a consistent, machine‑readable error contract (`ProblemDetails`).
- **To** log the full detail server‑side for diagnosis without exposing it to clients.
- **To** fail closed when security decisions cannot be made.
- **To** give clients a correlation ID so support can connect their report to the server logs.

### Syntax Rules and Structure

#### `ProblemDetails` and Exception Handling

```csharp
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
app.UseStatusCodePages();
```

#### Custom Error Response with Correlation ID

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        var traceId = Activity.Current?.Id ?? context.TraceIdentifier;

        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/problem+json";

        await context.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = 500,
            Title = "An unexpected error occurred.",
            Extensions = { ["traceId"] = traceId }
        });
    });
});
```

**Syntax rules:**

- **Never return `exception.ToString()` or `StackTrace` to clients** — log it instead.
- **Use `ProblemDetails`** for a standardized error shape that clients and tools understand.
- **Include a correlation ID** — `HttpContext.TraceIdentifier` or `Activity.Current?.Id`.
- **Fail closed** — authorization middleware denies when it cannot evaluate a policy.

**Constraints and limitations:**

- **`UseDeveloperExceptionPage` must never be enabled in production** — it leaks source code and configuration.
- **`ProblemDetails` must not include internal exception messages** — map to generic titles.
- **Error endpoints must not themselves throw** — keep them simple.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class OrdersController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderDto input, CancellationToken ct)
    {
        try
        {
            var order = await _service.CreateAsync(input, ct);
            return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
        }
        catch (InvalidOperationException ex)
        {
            _logger.LogWarning(ex, "Order creation failed");
            return Problem(
                title: "Order could not be created.",
                statusCode: StatusCodes.Status409Conflict);
        }
    }
}
```

**Step-by-step explanation:**

1. Known exceptions are caught and mapped to a controlled `ProblemDetails` response.
2. The full exception is logged server‑side with `_logger.LogWarning`.
3. The client receives only a generic title and status — no stack trace, no internal details.

**Expected Output:** The client receives `409 Conflict` with a `ProblemDetails` body; the server logs contain the exception detail.

**Why this output?** Error responses are safe and consistent; diagnostics remain server‑side.

### Real-World Cases

- **API endpoints:** Consistent `ProblemDetails` for 400, 404, 409, and 500.
- **Web apps:** Friendly error pages in production; developer pages in Development.
- **Authorization:** Fail closed when policies cannot be evaluated.

### References

- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- OWASP Error Handling Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html

---

## 8. Logging

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Logging is the practice of recording application events for diagnosis, auditing, and security monitoring — without recording sensitive data that would create a new exposure. |
| **Technical Definition** | ASP.NET Core uses `ILogger<T>` with structured logging (message templates and named properties) and configurable providers (console, file, Application Insights, OpenTelemetry). **Structured logging** enables querying by property (e.g., `UserId`, `TraceId`) rather than parsing strings. Security‑relevant events — authentication failures, authorization denials, input validation failures, and unhandled exceptions — should be logged with sufficient context but **without secrets, tokens, passwords, or full PII**. Log levels should be environment‑appropriate (Information in production, Debug in Development). |
| **Beginner‑Friendly Explanation** | Logging is the **"security camera"** for your app. It records what happened so you can investigate incidents and diagnose problems. But like a camera, it must not point at secrets — never log passwords, tokens, or full credit card numbers. Log **structured** events so you can search them, and log **security‑relevant** events (failed logins, authorization denials) so you can detect attacks. |

### Purposes

- **To** diagnose failures with enough context to reproduce and fix them.
- **To** detect attacks — repeated failed logins, authorization denials, suspicious input.
- **To** satisfy audit and compliance requirements (who did what, when).
- **To** correlate events across services using trace IDs.
- **To** avoid creating a new vulnerability by logging sensitive data.

### Syntax Rules and Structure

#### Structured Logging with `ILogger<T>`

```csharp
public class LoginService
{
    private readonly ILogger<LoginService> _logger;

    public LoginService(ILogger<LoginService> logger) => _logger = logger;

    public async Task<bool> AuthenticateAsync(string email, string password)
    {
        var user = await _users.FindByEmailAsync(email);
        if (user is null || !await _users.CheckPasswordAsync(user, password))
        {
            _logger.LogWarning(
                "Authentication failed for {Email} from {RemoteIp}",
                email, _httpContextAccessor.HttpContext?.Connection.RemoteIpAddress);
            return false;
        }

        _logger.LogInformation("User {UserId} authenticated", user.Id);
        return true;
    }
}
```

#### ❌ Vulnerable: Logging Secrets

```csharp
_logger.LogInformation("Login attempt: {Email} {Password}", email, password);
```

**Syntax rules:**

- **Use structured message templates** — `{Email}`, `{UserId}` become queryable properties.
- **Log security events at Warning or higher** — failed logins, authorization denials.
- **Never log passwords, tokens, API keys, or full PII** — redact or omit.
- **Use a correlation ID** (`TraceId`, `RequestId`) to connect logs across services.

**Constraints and limitations:**

- **Logs are a common data‑leak vector** — they are copied to monitoring systems and often have broader access than the database.
- **High‑volume logging is a denial‑of‑service risk** — rate limit or sample if necessary.
- **Log injection** — untrusted input in log messages can forge entries; structured logging mitigates this.

### Annotated Complete Step‑by‑Step Code Example

```csharp
public class PaymentController : ControllerBase
{
    private readonly ILogger<PaymentController> _logger;

    [HttpPost]
    public async Task<IActionResult> Pay(PaymentInput input, CancellationToken ct)
    {
        _logger.LogInformation(
            "Payment initiated for order {OrderId} by user {UserId}",
            input.OrderId, User.FindFirstValue(ClaimTypes.NameIdentifier));

        try
        {
            var result = await _payments.ProcessAsync(input, ct);
            _logger.LogInformation(
                "Payment {PaymentId} succeeded for order {OrderId}",
                result.PaymentId, input.OrderId);
            return Ok(result);
        }
        catch (PaymentDeclinedException ex)
        {
            _logger.LogWarning(ex,
                "Payment declined for order {OrderId}", input.OrderId);
            return Problem("Payment declined.", statusCode: 402);
        }
    }
}
```

**Step-by-step explanation:**

1. The initiation log includes the order ID and user ID — no card numbers.
2. The success log includes the payment ID — a safe reference.
3. The decline log includes the exception and order ID — sufficient for diagnosis, no sensitive data.

**Expected Output:** Structured log entries with queryable properties; no card numbers, tokens, or PII in the logs.

**Why this output?** Structured logging provides context; redaction prevents the log from becoming a data‑breach vector.

### Real-World Cases

- **Authentication monitoring:** Log failed logins with email and IP; alert on patterns.
- **Authorization auditing:** Log denials with user, resource, and policy.
- **Incident response:** Correlate logs across services using trace IDs.

### References

- Logging in .NET Core and ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/
- OWASP Logging Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- Structured logging with `ILogger` – https://learn.microsoft.com/en-us/dotnet/core/extensions/logging

---

## 9. Secret Management

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secret management is the practice of storing, accessing, and rotating sensitive values — connection strings, API keys, signing keys, passwords — outside source code and configuration files, using dedicated secret stores. |
| **Technical Definition** | ASP.NET Core loads configuration from multiple providers in order; later providers override earlier ones. **User Secrets** (`dotnet user-secrets`) stores development secrets outside the repository. In production, secrets should come from a **vault** — Azure Key Vault, AWS Secrets Manager, HashiCorp Vault — via the configuration provider or a managed identity. **Data Protection keys** used for cookies and tokens must also be persisted securely and shared across instances. Secrets must never appear in source control, build logs, or client‑side code. |
| **Beginner‑Friendly Explanation** | Secret management is the **"keep the keys in a safe, not on the doormat"** habit. Connection strings, API keys, and signing keys must never be committed to source code or written in `appsettings.json` that goes into the repository. In development, use User Secrets. In production, use a vault (Azure Key Vault) and access it with a managed identity so there is no password to steal. Rotate secrets regularly and never log them. |

### Purposes

- **To** keep secrets out of source control, where they are visible to anyone with repository access.
- **To** centralize secret storage and rotation in a vault with access control and audit logging.
- **To** eliminate hard‑coded credentials from configuration files and build scripts.
- **To** use managed identities so there is no secret to steal for service‑to‑service authentication.
- **To** persist Data Protection keys securely so cookies and tokens survive restarts and scale‑out.

### Syntax Rules and Structure

#### User Secrets (Development)

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Server=...;Database=...;..."
```

```csharp
var builder = WebApplication.CreateBuilder(args);
// User Secrets are loaded automatically in Development
```

#### Azure Key Vault (Production)

```csharp
var builder = WebApplication.CreateBuilder(args);

if (!builder.Environment.IsDevelopment())
{
    var keyVaultUri = new Uri(builder.Configuration["KeyVault:Uri"]!);
    builder.Configuration.AddAzureKeyVault(
        keyVaultUri,
        new DefaultAzureCredential());
}
```

#### Data Protection Key Persistence

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(blobUri)
    .ProtectKeysWithAzureKeyVault(keyUri, credential);
```

#### ❌ Vulnerable: Secret in `appsettings.json` Committed to Source

```json
{
  "ConnectionStrings": {
    "Default": "Server=prod;Database=prod;User Id=app;Password=P@ssw0rd!"
  }
}
```

**Syntax rules:**

- **Development secrets use User Secrets** — never committed to source.
- **Production secrets use a vault** — accessed via managed identity when possible.
- **Data Protection keys must be persisted** in multi‑instance deployments.
- **Never log or return secrets** — redact them in errors and logs.

**Constraints and limitations:**

- **Environment variables are better than source code but visible in process listings and crash dumps** — use a vault for production.
- **Managed identity requires Azure hosting** — other environments use workload identity or vault credentials.
- **Secret rotation requires application restarts or reload support** — plan for it.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Development: User Secrets (loaded automatically)
// Production: Azure Key Vault via managed identity
if (!builder.Environment.IsDevelopment())
{
    builder.Configuration.AddAzureKeyVault(
        new Uri(builder.Configuration["KeyVault:Uri"]!),
        new DefaultAzureCredential());
}

// Data Protection keys in a shared, encrypted store
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(builder.Configuration["DataProtection:BlobUri"]!)
    .ProtectKeysWithAzureKeyVault(
        new Uri(builder.Configuration["DataProtection:KeyUri"]!),
        new DefaultAzureCredential());

var app = builder.Build();
```

**Step-by-step explanation:**

1. In development, User Secrets provides connection strings without committing them.
2. In production, Key Vault is added as a configuration source; `DefaultAzureCredential` uses managed identity.
3. Data Protection keys are persisted to Blob Storage and encrypted with Key Vault — cookies and tokens survive restarts and are shared across instances.

**Expected Output:** Secrets are never in source code; production secrets are retrieved from Key Vault; Data Protection keys are shared and encrypted.

**Why this output?** Each environment uses the appropriate secret store, and key material is protected at rest.

### Real-World Cases

- **Multi‑instance deployments:** Shared Data Protection key ring in Redis or Blob Storage.
- **CI/CD:** Deployment pipelines retrieve secrets from the vault at deploy time, never from source.
- **Secret rotation:** Key Vault versioning allows rotation without application downtime.

### References

- Safe storage of app secrets in development – https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Azure Key Vault configuration provider – https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
- Data Protection key management – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview
- OWASP Secrets Management Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

---

## 10. Secure Coding Best Practices (The Review Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secure coding best practices are the reviewable checklist that ties validation, encoding, parameterization, least privilege, secure defaults, error handling, logging, and secret management into a consistent engineering discipline. |
| **Technical Definition** | The checklist covers **boundary validation** (DTOs, data annotations, FluentValidation), **output encoding** (Razor `@`, context‑specific encoders, avoid `Html.Raw`), **parameterized data access** (LINQ, `FromSqlInterpolated`, no `FromSqlRaw` concatenation), **authorization** (fallback policy, `[Authorize]`, object‑level checks), **least privilege** (database roles, managed identities, non‑root containers), **secure defaults** (HTTPS, HSTS, secure cookies, diagnostics gated), **error handling** (`ProblemDetails`, no stack traces), **logging** (structured, no secrets), and **secret management** (User Secrets, vault, persisted Data Protection keys). |
| **Beginner‑Friendly Explanation** | This is the **"before you merge" list**. Every pull request should be checked against it: Is input validated? Is output encoded? Are queries parameterized? Is the endpoint authorized? Does the database account have only the permissions it needs? Are diagnostics off in production? Do errors leak? Do logs leak? Are secrets in a vault? If all nine are yes, the change is much less likely to introduce a vulnerability. |

### Purposes

- **To** provide a **repeatable review artifact** that catches the most common secure‑coding mistakes.
- **To** make security a **team habit**, not a specialist's responsibility.
- **To** align code review with the OWASP Top 10 and the framework's security primitives.
- **To** catch issues before they reach production, where remediation is far more expensive.
- **To** support onboarding — new developers learn the patterns by reviewing them.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Input validation** | Validate DTOs with data annotations; check `ModelState`. | Bind domain entities directly; trust client validation. |
| **Output encoding** | Rely on Razor `@`; use context‑specific encoders. | Use `Html.Raw` with untrusted data. |
| **Parameterized queries** | LINQ or `FromSqlInterpolated`; `{0}` placeholders in `FromSqlRaw`. | Concatenate user input into SQL. |
| **Authorization** | Fallback policy; `[Authorize]`; object‑level ownership checks. | Leave endpoints unannotated; assume authentication is enough. |
| **Least privilege** | `db_datareader`/`db_datawriter`; managed identities; non‑root containers. | Use `sa`/`db_owner` at runtime. |
| **Secure defaults** | HTTPS/HSTS; secure cookies; diagnostics gated to Development. | Ship Swagger UI or developer exceptions to production. |
| **Error handling** | `ProblemDetails`; correlation ID; log details server‑side. | Return stack traces or SQL errors to clients. |
| **Logging** | Structured events; security‑relevant warnings; redaction. | Log passwords, tokens, or full PII. |
| **Secrets** | User Secrets (dev); vault (prod); persisted Data Protection keys. | Commit secrets to source or `appsettings.json`. |
| **Dependencies** | `dotnet list package --vulnerable` in CI; patch promptly. | Ignore vulnerability warnings. |

### Annotated Code Example (Review‑Ready Controller)

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _service;
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(IOrderService service, ILogger<OrdersController> logger)
    {
        _service = service;
        _logger = logger;
    }

    [HttpPost]
    public async Task<ActionResult<OrderDto>> Create(
        CreateOrderDto input, CancellationToken ct)
    {
        // Input validated by data annotations + [ApiController]
        var order = await _service.CreateAsync(input, ct);

        _logger.LogInformation(
            "Order {OrderId} created by {UserId}",
            order.Id, User.FindFirstValue(ClaimTypes.NameIdentifier));

        return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<OrderDto>> GetById(int id, CancellationToken ct)
    {
        var order = await _service.FindAsync(id, ct);
        if (order is null) return NotFound();

        // Object-level authorization (BOLA prevention)
        if (order.CustomerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
            return Forbid();

        return Ok(order);
    }
}
```

**Why this follows best practices:**

- **Input validation** is enforced by the DTO and `[ApiController]`.
- **Authorization** is applied at the controller level and checked at the object level.
- **Parameterized data access** is used via the service layer (EF Core LINQ).
- **Logging** includes IDs but no sensitive data.
- **Error handling** uses `NotFound` and `Forbid` — no leaked details.
- **Secure defaults** (fallback policy) protect any endpoint without explicit `[Authorize]`.

### Real-World Cases

- **Code review gates:** Every PR is checked against the nine areas.
- **Onboarding:** New developers review the checklist and the annotated example.
- **CI:** SAST tools (Semgrep, CodeQL) and dependency scanning enforce parts of the checklist automatically.

### References

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- OWASP Top 10 (2021) – https://owasp.org/Top10/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html
- OWASP Application Security Verification Standard (ASVS) – https://owasp.org/www-project-application-security-verification-standard/

---

## Summary Table: Secure Coding at a Glance

| Practice | Primary Role | Key Tool / Syntax | When to Apply |
|----------|--------------|-------------------|---------------|
| **Secure coding** | The overall discipline | Validation + encoding + parameterization + defaults | Every line of code |
| **Input validation** | Reject bad input at the boundary | DTOs, data annotations, `ModelState` | Every request body and query |
| **Output encoding** | Prevent XSS | Razor `@`, `HtmlEncoder`, `JsonSerializer` | Every output sink |
| **Parameterized queries** | Prevent SQL injection | LINQ, `FromSqlInterpolated` | Every database call |
| **Least privilege** | Limit blast radius | `db_datareader`/`db_datawriter`, managed identities | Every identity |
| **Secure defaults** | Safe unless changed | Fallback policy, HTTPS/HSTS, secure cookies | Every endpoint and deployment |
| **Error handling** | Fail safely without leaking | `ProblemDetails`, `UseExceptionHandler` | Every failure path |
| **Logging** | Diagnose and detect without leaking | `ILogger<T>` structured templates, redaction | Every security event |
| **Secret management** | Keep credentials out of code | User Secrets, Key Vault, Data Protection | Every secret |
| **Best practices** | The review checklist | The nine‑area checklist | Every pull request |

---

## References (Full List)

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- Model validation in ASP.NET Core MVC and Razor Pages – https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
- Part 9, add validation to an ASP.NET Core MVC app – https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/validation
- Prevent Cross-Site Scripting (XSS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- SQL Queries (EF Core) – https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- Logging in .NET Core and ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/
- Safe storage of app secrets in development – https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Azure Key Vault configuration provider – https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
- Data Protection key management – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview
- `HtmlEncoder` class – https://learn.microsoft.com/en-us/dotnet/api/system.text.encodings.web.htmlencoder
- Problem Details for HTTP APIs (RFC 9457) – https://www.rfc-editor.org/rfc/rfc9457
- OWASP Top 10 (2021) – https://owasp.org/Top10/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html
- OWASP Input Validation Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Cross Site Scripting Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Query Parameterization Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Authorization Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP Error Handling Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- OWASP Logging Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- OWASP Secrets Management Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- OWASP Application Security Verification Standard (ASVS) – https://owasp.org/www-project-application-security-verification-standard/
- NIST SP 800‑53 AC‑6 (Least Privilege) – https://csf.tools/reference/nist-sp-800-53/r5/ac/ac-6/