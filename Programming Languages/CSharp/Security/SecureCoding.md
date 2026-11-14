# Secure Coding in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Secure coding is the practice of writing software that is resilient to adversarial misuse — accepting only valid input, neutralising dangerous data for its destination, enforcing least privilege, failing safely, and protecting secrets — throughout the entire development lifecycle.

**Technical Definition**
Secure coding in C# and .NET is the disciplined application of language-level patterns, framework-provided primitives, and configuration hardening to eliminate vulnerability classes before they reach production. It spans **input validation** (validating structure, type, length, range, and format at trust boundaries), **output encoding** (context-aware encoding for HTML, JavaScript, URL, and CSS sinks), **parameterised queries** (keeping data separate from SQL, shell commands, and file paths), **least privilege** (minimal permissions for users, services, database accounts, and containers), **secure defaults** (HTTPS, authorization by default, safe serialisation, secure cookies), **error handling** (failing closed, suppressing internal details from clients), **logging** (capturing security events without leaking sensitive data), and **secret management** (runtime loading from managed vaults, never committing secrets to source). The .NET platform provides strong building blocks — EF Core's automatic parameterisation, Razor's default encoding, ASP.NET Core Identity's password hashing, the Data Protection API, and `System.Text.Json`'s safe deserialisation — but the defaults are tuned for developer convenience, not adversarial production traffic. Secure coding is therefore about using the right primitive for each sink and never bypassing the framework's built-in protections.

**Beginner-Friendly Explanation**
Secure coding is “writing code that cannot be tricked into doing harmful things.” In C#, that means: check everything that comes from outside before you use it, never let user input become SQL or JavaScript or a file path, give your app only the permissions it actually needs, never show users your internal error details, keep secrets out of your source code, and log the right things without logging passwords. The .NET platform gives you good tools, but you have to use them correctly — the framework cannot protect you if you bypass its safeguards.

---

### Key Characteristics

- **Sink-aware** — The correct defence depends on the destination: parameterisation for SQL, HTML encoding for HTML, JavaScript encoding for script blocks, canonicalisation for file paths.
- **Boundary-driven** — Validate at the trust boundary (HTTP, message queue, file, database), not deep in the call stack.
- **Framework-first** — Prefer the platform's safe APIs over hand-rolled logic; Razor encodes by default, EF Core parameterises by default, ASP.NET Core Identity hashes passwords correctly.
- **Fail-closed** — On error, deny access rather than grant it; suppress internal details from clients.
- **Least-privilege throughout** — Minimise permissions for users, services, database accounts, and containers.
- **Secret-aware** — Secrets are loaded at runtime from a managed store, never committed to source, never logged.
- **Measurement-driven** — Secure coding is verified by SAST, DAST, dependency auditing, and penetration testing, not assumed.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and Entity Framework Core.
- Familiarity with HTTP, HTML, JavaScript, and SQL.
- Understanding of authentication, authorization, and sessions.
- Awareness of OWASP Top 10 and common vulnerability classes.
- Basic knowledge of dependency management (NuGet, lock files) and CI/CD pipelines.

---

### Related Programming Areas

- **Common vulnerabilities** — SQL injection, XSS, CSRF, SSRF, insecure deserialisation.
- **Authentication and authorization security** — Identity and access control.
- **Input validation** — Data Annotations and FluentValidation.
- **Cryptography** — Data Protection API, hashing, and encryption.
- **Supply chain security** — NuGet Audit, SBOMs, and package provenance.

---

### Core Concepts / Features

1. Input Validation
2. Output Encoding
3. Parameterised Queries
4. Least Privilege
5. Secure Defaults
6. Error Handling
7. Logging
8. Secret Management
9. Secure Coding Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure with complete general syntaxes and component breakdowns, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. Input Validation

### Definitions

**Core Definition**
Input validation is the practice of verifying that data entering the application conforms to expected structure, type, length, range, and format before it is processed or stored.

**Technical Definition**
Input validation in .NET is applied at trust boundaries — every point where data crosses from an untrusted source into the application. The platform provides Data Annotations (`System.ComponentModel.DataAnnotations`) for declarative property-level validation and FluentValidation for imperative, testable rules. ASP.NET Core's `[ApiController]` attribute automatically returns 400 Bad Request when `ModelState.IsValid` is false. Validation is distinct from sanitisation: validation rejects non-conforming input, while output encoding neutralises dangerous content for a specific sink. Both are required; doing only one leaves a gap. The most common C# vulnerability root cause is trusting a value because it “came from our own frontend” — every value crossing a trust boundary must be treated as hostile until proven otherwise.

**Beginner-Friendly Explanation**
Input validation is “checking what comes in before you use it.” If someone types `"abc"` into an age field, validation rejects it before it reaches your database. Validate on the server, not just in the browser, and validate everything — not just the fields you think are risky.

---

### Purposes

- To prevent invalid, malformed, or malicious data from reaching business logic or storage.
- To protect against injection attacks by rejecting structurally invalid input.
- To enforce business invariants and data integrity at the boundary.
- To provide clear, actionable error messages to API consumers.
- To reduce the attack surface exposed to downstream systems.
- To satisfy OWASP ASVS and regulatory validation requirements.

---

### Syntax Rules and Structure

#### General Syntax (Data Annotations)

```csharp
public class CreateUserRequest
{
    [Required, StringLength(100, MinimumLength = 2)]
    public string Name { get; set; } = default!;

    [Required, EmailAddress, StringLength(255)]
    public string Email { get; set; } = default!;

    [Range(0, 150)]
    public int Age { get; set; }
}
```

#### General Syntax (Boundary Validation with `TryParse`)

```csharp
if (!long.TryParse(Request.Query["id"], out var id) || id <= 0)
    return Results.BadRequest("invalid id");
```

#### Syntax Rules

- Validate at the boundary, not deep in the call stack.
- Prefer allow-lists over deny-lists; specify what is allowed rather than enumerating what is forbidden.
- Validate structure (type, length, range, format) as early as possible.
- Use `TryParse` instead of `Parse` to avoid exceptions on malformed input.
- `[ApiController]` automatically returns 400 when validation fails; without it, check `ModelState.IsValid` explicitly.
- Validation is not sanitisation; encode on output for the specific sink.

#### Constraints and Limitations

- Validation rules must be maintained as the domain evolves; stale rules create false confidence.
- Async validation (database uniqueness) adds latency; ensure indexes on checked columns.
- Overly strict validation can reject legitimate international input.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Boundary Validation with `TryParse`

```csharp
// ❌ Anti-pattern: parse-then-hope
var id = long.Parse(Request.Query["id"]); // throws or overflows on bad input

// ✅ Pattern: validate and constrain at the edge
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

- `TryParse` avoids exceptions and returns `false` on malformed input; the range check rejects non-positive values before they reach the domain.

---

#### Example 2: Data Annotations with `[ApiController]`

```csharp
[ApiController]
public class UsersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateUserRequest request) => Ok();
}
```

**Expected Output (conceptual)**

```
Missing Name: 400 with {"errors":{"Name":["The Name field is required."]}}.
Invalid email: 400 with format error.
Age=200: 400 with range error.
All valid: 200.
```

**Why this output occurs**

- `[ApiController]` inspects `ModelState` automatically and returns 400 with validation errors.

---

### Real-World Cases

- **Registration forms** — Name, email, and password constraints enforced server-side.
- **File uploads** — File names and content types validated to prevent path traversal.
- **Message queue consumers** — Payload schema validated before processing.
- **API query parameters** — Pagination and filter values constrained to safe ranges.

---

### References

- C# Secure Coding Guide (2026): Patterns and Anti-Patterns — safeguard.sh - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- ASP.NET Core Security Best Practices 2026 — Offensive360 - https://offensive360.com/blog/aspnet-core-security-best-practices/

---

## 2. Output Encoding

### Definitions

**Core Definition**
Output encoding is the process of transforming data so that it is treated as literal text by its destination sink, preventing it from being interpreted as executable code or markup.

**Technical Definition**
Output encoding is context-aware: HTML content requires HTML encoding, HTML attributes require attribute encoding, JavaScript contexts require JavaScript encoding, URL query strings require URL encoding, and CSS requires CSS encoding. In ASP.NET Core, Razor **HTML-encodes variable output by default**, so most XSS vulnerabilities arise from explicitly bypassing encoding with `Html.Raw`, `HtmlString`, or returning unescaped markup from an API. `System.Text.Json.JsonSerializer` (or `Json.Serialize` in Razor) produces JavaScript-safe output when embedding data in script blocks. A **Content Security Policy** (CSP) provides an additional layer that restricts which scripts can execute.

**Beginner-Friendly Explanation**
Output encoding is “making sure data is treated as text, not code, when it reaches the browser.” If a comment contains `<script>alert('xss')</script>` and you display it without encoding, the browser runs the script. Razor encodes it automatically, so the browser shows the literal text instead. Encoding must match the context: HTML encoding for HTML, JavaScript encoding for script blocks, and so on.

---

### Purposes

- To prevent cross-site scripting (XSS) through HTML, attribute, JavaScript, and URL contexts.
- To neutralise dangerous content in user-generated content.
- To protect against DOM-based attacks when embedding data in client-side scripts.
- To satisfy OWASP A05:2025 injection requirements.
- To complement CSP as a defence-in-depth layer.

---

### Syntax Rules and Structure

#### Context-Aware Encoding

| Context | Encoding | .NET API |
|---|---|---|
| **HTML content** | HTML encoding | Razor `@variable` (automatic), `HtmlEncoder.Default.Encode()` |
| **HTML attributes** | Attribute encoding | Razor (automatic) |
| **JavaScript** | JavaScript encoding | `Json.Serialize()`, `JavaScriptEncoder.Default.Encode()` |
| **URL query strings** | URL encoding | `Uri.EscapeDataString()` |

#### General Syntax (Safe Razor)

```razor
@* Razor automatically HTML-encodes this *@
<p>@Model.UserComment</p>

@* Safe JavaScript embedding *@
<script>
    var comment = @Json.Serialize(Model.UserComment);
</script>
```

#### General Syntax (Unsafe — Do Not Use)

```razor
@* ❌ Bypasses encoding; XSS risk *@
<p>@Html.Raw(Model.UserComment)</p>

@* ❌ Untrusted data inside a script string *@
<script>
    var comment = '@Model.UserComment';
</script>
```

#### Syntax Rules

- Let Razor encode output by default; never use `Html.Raw` on untrusted data.
- Use `Json.Serialize` for data embedded in script blocks.
- Set a restrictive CSP that disables inline scripts and `eval`.
- Use `HttpOnly` cookies to reduce the impact of XSS-based cookie theft.
- HTML encoding does not protect JavaScript, CSS, or URL contexts; each requires its own encoding.

#### Constraints and Limitations

- CSP can break legitimate functionality if not tested; start in report-only mode.
- Client-side frameworks escape by default, but `dangerouslySetInnerHTML` reintroduces the risk.
- Encoding alone does not address stored XSS if the data is later used in an unsafe context.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Razor Auto-Encoding vs. `Html.Raw`

```razor
@* Model.Comment = "<script>alert('xss')</script>" *@

@* ✅ Razor auto-encodes *@
<p>@Model.Comment</p>
@* Renders: &lt;script&gt;alert('xss')&lt;/script&gt; *@

@* ❌ Html.Raw bypasses encoding *@
<p>@Html.Raw(Model.Comment)</p>
@* Renders: <script>alert('xss')</script> — script executes *@
```

**Expected Output (conceptual)**

```
Auto-encoded: the browser displays the literal text.
Html.Raw: the browser executes the script.
```

**Why this output occurs**

- Razor HTML-encodes `<` to `&lt;`, so the browser treats the content as text. `Html.Raw` emits the string as-is, so the browser parses it as markup and executes the script.

---

#### Example 2: Safe JavaScript Embedding

```csharp
// ✅ Safe: JSON serializer encodes for JavaScript
<script>
    var comment = @Json.Serialize(Model.UserComment);
</script>

// ❌ Unsafe: string interpolation into a script block
<script>
    var comment = '@Model.UserComment';
</script>
```

**Expected Output (conceptual)**

```
Safe: the comment is encoded as a JSON string; quotes and angle brackets are escaped.
Unsafe: a comment containing '); alert(1); // breaks out of the string and executes.
```

**Why this output occurs**

- `Json.Serialize` produces a JavaScript-safe string literal; direct interpolation places untrusted data inside a JavaScript string without escaping.

---

### Real-World Cases

- **Comment sections** — Stored XSS through unsanitised comments.
- **User profiles** — XSS through display names and bios.
- **Search results pages** — Reflected XSS via the search query string.
- **API responses rendered as HTML** — Markup injected into a page without encoding.

---

### References

- C# Secure Coding Guide (2026) — Output encoding for the specific sink - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- ASP.NET Core Security Best Practices 2026 — XSS in Razor - https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP .NET Security Cheat Sheet — Output encoding - https://github.com/CyberSecAI/genai-sec-agents/blob/main/data/owasp_cache/dotnet-security.md

---

## 3. Parameterised Queries

### Definitions

**Core Definition**
Parameterised queries are a technique in which SQL text and data values are sent to the database separately, so that data can never be interpreted as SQL code.

**Technical Definition**
In .NET, parameterisation is provided automatically by **EF Core LINQ queries** (which translate to parameterised SQL) and by **ADO.NET `DbParameter`** objects. `FromSqlInterpolated` and `FromSql` in EF Core are safe because interpolated values become database parameters. The danger lies in `FromSqlRaw` and `ExecuteSqlRaw` when combined with string concatenation, and in dynamic identifiers (table names, column names, sort directions) that cannot be parameterised and require allow-listing. The same principle applies to shell commands (invoke the executable directly, pass arguments as a list) and file paths (canonicalise and verify containment).

**Beginner-Friendly Explanation**
Parameterised queries keep “the SQL” and “the data” separate. If your code builds SQL by gluing strings together, the attacker can inject SQL. If you use parameters, the database treats the input as data, never as code. In EF Core, normal LINQ queries are already parameterised; the risk is when you write raw SQL and concatenate strings.

---

### Purposes

- To prevent SQL injection through crafted input values.
- To keep data separate from code in SQL, shell, and file-path sinks.
- To protect against path traversal and shell injection.
- To satisfy OWASP A05:2025 injection requirements.
- To eliminate the most classic and still-prevalent web vulnerability.

---

### Syntax Rules and Structure

#### Safe vs. Unsafe Patterns

| Sink | Unsafe | Safe |
|---|---|---|
| **SQL** | `cmd.CommandText = $"SELECT * FROM Users WHERE Email = '{email}'"` | `cmd.CommandText = "SELECT * FROM Users WHERE Email = @email"; cmd.Parameters.AddWithValue("@email", email)` |
| **SQL (EF Core)** | `FromSqlRaw($"... {input}")` | `FromSqlInterpolated($"... {input}")` |
| **Shell** | `Process.Start("cmd.exe", $"/c {userInput}")` | `ProcessStartInfo` with `UseShellExecute = false` and `ArgumentList.Add(userInput)` |
| **File path** | `Path.Combine(baseDir, userPath)` without validation | `Path.GetFullPath` + containment check |

#### General Syntax (SQL Parameterisation)

```csharp
using var cmd = new SqlCommand("SELECT * FROM Users WHERE Email = @email", connection);
cmd.Parameters.AddWithValue("@email", email);
```

#### General Syntax (File Path Containment)

```csharp
var full = Path.GetFullPath(Path.Combine(baseDir, userPath));
if (!full.StartsWith(baseDir + Path.DirectorySeparatorChar, StringComparison.Ordinal))
    throw new UnauthorizedAccessException();
```

#### Syntax Rules

- Use EF Core LINQ wherever possible; it parameterises automatically.
- When raw SQL is necessary, use `FromSqlInterpolated` or explicit `DbParameter` objects.
- Never concatenate user input into SQL, shell commands, or file paths.
- Allow-list dynamic identifiers (column names, sort directions) before building SQL.
- Canonicalise file paths and verify they remain within the intended base directory.

#### Constraints and Limitations

- Dynamic identifiers cannot be parameterised; allow-listing is the only safe mitigation.
- Stored procedures are safe only if they do not build dynamic SQL from parameters.
- Second-order injection can occur when safely stored data is later concatenated into a query elsewhere.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: SQL Injection Prevention

```csharp
// ❌ Vulnerable: string interpolation
cmd.CommandText = $"SELECT * FROM Users WHERE Email = '{email}'";

// ✅ Safe: bound parameter
cmd.CommandText = "SELECT * FROM Users WHERE Email = @email";
cmd.Parameters.AddWithValue("@email", email);
```

**Expected Output (conceptual)**

```
Unsafe: attacker supplies ' OR '1'='1 and the query returns every user.
Safe: the input is bound as a parameter and never parsed as SQL.
```

**Why this output occurs**

- In the unsafe version, the input becomes part of the SQL text; in the safe version, it travels separately and is treated as data.

---

#### Example 2: Shell Command Without Shell Parsing

```csharp
// ❌ Vulnerable: shell injection
Process.Start("convert.exe", userSuppliedFilename);

// ✅ Safe: no shell, explicit argument list
var psi = new ProcessStartInfo("/usr/bin/convert") { UseShellExecute = false };
psi.ArgumentList.Add(userSuppliedFilename);
Process.Start(psi);
```

**Expected Output (conceptual)**

```
Unsafe: a filename containing ; rm -rf / executes additional commands.
Safe: the filename is treated as a single literal argument; no shell parsing occurs.
```

**Why this output occurs**

- `UseShellExecute = false` and `ArgumentList` prevent the shell from interpreting metacharacters.

---

### Real-World Cases

- **Login forms** — Classic `' OR '1'='1` bypass.
- **Search endpoints** — Concatenated filter values that expose the entire table.
- **File download endpoints** — Path traversal to read `/etc/passwd` or application configuration.
- **Report generation** — User-controlled sort or filter expressions that execute arbitrary SQL.

---

### References

- C# Secure Coding Guide (2026) — Never build SQL, shell, or paths from raw strings - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- ASP.NET Core Security Best Practices 2026 — SQL injection via EF Core - https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP .NET Security Cheat Sheet — Injection prevention - https://github.com/CyberSecAI/genai-sec-agents/blob/main/data/owasp_cache/dotnet-security.md

---

## 4. Least Privilege

### Definitions

**Core Definition**
The principle of least privilege states that every user, process, service, and component should operate with only the minimum permissions necessary to perform its function, and no more.

**Technical Definition**
Least privilege in .NET applies at multiple levels: **user accounts** should have only the roles and claims required; **application pool identities** should run as low-privilege virtual accounts; **database users** should have only the CRUD permissions needed on specific objects; **managed identities** and service principals should have narrowly scoped RBAC roles; **containers** should run as non-root with `allowPrivilegeEscalation: false`; and **authorization** should deny by default, requiring `[Authorize]` on all endpoints and using `[AllowAnonymous]` explicitly and sparingly. The principle limits the blast radius of a compromise: an attacker who breaches a low-privilege component can do far less damage than one who breaches an administrator.

**Beginner-Friendly Explanation**
Least privilege means “give everyone and everything the smallest key that still opens the doors they need.” Your web app should not run as an administrator, your database user should not be able to drop tables if it only needs to read and write rows, and your users should not be admins unless their job requires it.

---

### Purposes

- To limit the damage an attacker can do after compromising a component.
- To reduce the blast radius of a vulnerability.
- To satisfy compliance requirements (PCI-DSS, HIPAA, SOC 2).
- To prevent accidental misuse of privileges.
- To enforce separation of duties and least-privilege database access.
- To harden containers and cloud identities.

---

### Syntax Rules and Structure

#### Least-Privilege Checklist

```
✅ Run application pools under low-privilege virtual accounts.
✅ Use a dedicated database user with only required CRUD permissions.
✅ Apply [Authorize] by default; use [AllowAnonymous] explicitly and sparingly.
✅ Scope managed identities and service principals to specific resources.
✅ Run containers as non-root with allowPrivilegeEscalation: false.
✅ Do not grant db_owner or sysadmin to application accounts.
```

#### General Syntax (Authorization by Default)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

[AllowAnonymous]
public IActionResult Health() => Ok();
```

#### General Syntax (Least-Privilege Database User)

```sql
CREATE USER app_user WITH PASSWORD '...';
GRANT SELECT, INSERT, UPDATE ON Orders TO app_user;
-- No DELETE, no DDL, no db_owner
```

#### Syntax Rules

- Require authentication by default; opt out explicitly with `[AllowAnonymous]`.
- Grant database users only the permissions required for the application's function.
- Use managed identities with narrowly scoped roles in cloud environments.
- Run containers as non-root and disable privilege escalation.

#### Constraints and Limitations

- Least privilege can increase administrative overhead; automation and policy-as-code help.
- Overly restrictive permissions can break functionality; permissions must be tested.
- Privilege creep occurs over time; regular access reviews are required.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Default Authorization with Explicit Anonymous Exception

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

app.MapGet("/health", [AllowAnonymous] () => "OK");
app.MapGet("/orders", () => GetOrders()); // protected by fallback
```

**Expected Output (conceptual)**

```
GET /health anonymous: 200.
GET /orders anonymous: 401.
GET /orders authenticated: 200.
```

**Why this output occurs**

- The fallback policy requires authentication for all endpoints without explicit metadata; `[AllowAnonymous]` opts the health endpoint out.

---

#### Example 2: Low-Privilege Database User

```sql
CREATE USER app_user WITH PASSWORD '...';
GRANT SELECT, INSERT, UPDATE ON Orders TO app_user;
GRANT SELECT ON Products TO app_user;
```

**Expected Output (conceptual)**

```
The application can read and write orders and read products.
It cannot delete orders or alter the schema.
```

**Why this output occurs**

- The database user has only the permissions required for the application's function.

---

### Real-World Cases

- **Web applications** — Application pool identity running as `ApplicationPoolIdentity` rather than `LocalSystem`.
- **Cloud services** — Managed identities scoped to a single Key Vault or Storage Account.
- **Containers** — Non-root user with read-only filesystem where possible.
- **Multi-tenant SaaS** — Per-tenant database users or row-level security.

---

### References

- Security And Identity — Secure defaults and least privilege - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- Least Privilege checklist — raw.githubusercontent.com - https://github.com/…/least-privilege
- Code Checklist — Altinn Studio (container privileges) - https://docs.altinn.studio

---

## 5. Secure Defaults

### Definitions

**Core Definition**
Secure defaults are the configuration and behavioural choices that are safest out of the box, requiring deliberate, informed action to weaken them.

**Technical Definition**
Secure defaults in .NET and ASP.NET Core include **HTTPS redirection and HSTS** outside development, **authorization by default** via a fallback policy, **secure cookie flags** (`HttpOnly`, `Secure`, `SameSite`), **antiforgery protection**, **safe deserialisation** with `System.Text.Json` (no `TypeNameHandling`), **validated JWT parameters**, and **secrets kept out of source code**. .NET 8+ enables NuGet Audit by default, and modern templates enable HTTPS redirection and HSTS outside development. Secure defaults make the safe choice the path of least resistance, forcing developers to actively opt into less secure behaviour.

**Beginner-Friendly Explanation**
Secure defaults mean “the safest setting is the one you get if you don't change anything.” If the default is insecure, developers who don't read the docs will ship vulnerabilities. Secure defaults flip that: you have to actively choose to weaken security, which forces a conscious decision.

---

### Purposes

- To protect developers who do not configure security explicitly.
- To make the safe choice the path of least resistance.
- To reduce the prevalence of common misconfigurations.
- To force conscious decisions when weakening security.
- To align with compliance and framework requirements.

---

### Syntax Rules and Structure

#### Secure Defaults Checklist

```
✅ HTTPS redirection and HSTS enabled outside development.
✅ Antiforgery protection enabled for cookie-based apps.
✅ [Authorize] by default or a fallback authorization policy.
✅ Secure cookie flags: HttpOnly, Secure, SameSite.
✅ System.Text.Json without TypeNameHandling.
✅ NuGet Audit enabled (default in .NET 8+).
✅ No secrets in source code or appsettings.json.
✅ Detailed exception pages disabled in production.
```

#### General Syntax (Secure ASP.NET Core Defaults)

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}
app.UseHttpsRedirection();

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

#### Syntax Rules

- Do not disable secure defaults without understanding the implications and documenting the decision.
- Use the fallback policy to require authentication by default.
- Keep `TypeNameHandling` off in JSON serialisation; prefer `System.Text.Json`.
- Enable HTTPS redirection and HSTS outside development.

#### Constraints and Limitations

- Secure defaults can break compatibility with legacy clients or workflows.
- Some defaults are environment-specific (e.g., developer exception page in Development only).
- Secure defaults are a baseline, not a complete security strategy.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure by Default Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output (conceptual)**

```
HTTP requests redirected to HTTPS.
HSTS header sent outside development.
Detailed errors suppressed in production.
Authentication required by default.
```

**Why this output occurs**

- Each secure default is enabled explicitly (or is the template default); weakening any of them requires a deliberate code change.

---

#### Example 2: Safe Deserialisation Default

```csharp
// ✅ System.Text.Json does not resolve arbitrary types by name
var order = JsonSerializer.Deserialize<Order>(json);

// ❌ Newtonsoft.Json with TypeNameHandling.All can instantiate arbitrary types
// var settings = new JsonSerializerSettings { TypeNameHandling = TypeNameHandling.All };
```

**Expected Output (conceptual)**

```
System.Text.Json: deserialises only the specified type; no type-resolution RCE.
Newtonsoft with TypeNameHandling.All: attacker-controlled type names may execute code.
```

**Why this output occurs**

- `System.Text.Json` uses compile-time type information, not attacker-supplied type names.

---

### Real-World Cases

- **ASP.NET Core templates** — HTTPS redirection and HSTS enabled by default.
- **.NET 8+ NuGet Audit** — Vulnerability warnings during `dotnet restore`.
- **JWT authentication** — Issuer, audience, and signature validation are required.
- **Cookie authentication** — `HttpOnly`, `Secure`, and `SameSite` flags set by default.

---

### References

- .NET Security Best Practices (2026 Guide) — secure defaults and deserialisation - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- ASP.NET Core Security Best Practices 2026 — secure defaults - https://offensive360.com/blog/aspnet-core-security-best-practices/
- Security And Identity — Secure defaults - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md

---

## 6. Error Handling

### Definitions

**Core Definition**
Secure error handling is the practice of responding to errors in a way that fails safely, does not expose internal implementation details to clients, and preserves diagnostic information for developers.

**Technical Definition**
In ASP.NET Core, secure error handling means using a **global exception handler** (`UseExceptionHandler`) to catch unhandled exceptions, return **generic error responses** to clients (no stack traces, no internal paths, no database schema details), and log the full exception details internally. Error responses should use consistent **Problem Details** contracts with appropriate HTTP status codes: **4xx for client errors**, **5xx for server errors**. The developer exception page must never be active in production. Error handling must **fail closed**: on error, deny access rather than grant it.

**Beginner-Friendly Explanation**
Secure error handling is “don't show your users your dirty laundry.” When something goes wrong, the user should see a generic message (“Something went wrong”), while you log the full details privately for debugging. If you show a stack trace to the internet, you are telling attackers exactly how your system is built.

---

### Purposes

- To prevent information leakage through stack traces and internal details.
- To provide a consistent, user-friendly error contract.
- To preserve full diagnostic information for developers.
- To fail closed on errors rather than granting unintended access.
- To satisfy OWASP error-handling and logging requirements.
- To support incident response through structured internal logging.

---

### Syntax Rules and Structure

#### General Syntax (Global Exception Handler)

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    // or a custom middleware that returns Problem Details
}

app.MapGet("/error", () => Results.Problem(
    statusCode: 500,
    title: "An unexpected error occurred"));
```

#### General Syntax (Problem Details)

```csharp
return Results.Problem(
    statusCode: StatusCodes.Status400BadRequest,
    title: "Invalid request",
    detail: "The 'age' field must be between 0 and 150.");
```

#### Syntax Rules

- Use `UseExceptionHandler` outside development; never use `UseDeveloperExceptionPage` in production.
- Return generic error messages to clients; log full details internally.
- Use 4xx for client errors and 5xx for server errors.
- Use Problem Details for a consistent error contract.
- Do not swallow exceptions silently; wrap or propagate them with logging.

#### Constraints and Limitations

- Overly generic errors can frustrate legitimate users; balance security with usability.
- Problem Details should not include sensitive internal data.
- Logging exceptions must not log sensitive request data (passwords, tokens).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Development vs. Production Error Handling

```csharp
// ❌ Development settings leak stack traces
app.UseDeveloperExceptionPage();

// ✅ Production: generic error response
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}
```

**Expected Output (conceptual)**

```
Development page: stack trace with file paths and source code.
Production handler: generic 500 response with no internal details.
```

**Why this output occurs**

- `UseDeveloperExceptionPage` is a development-only tool; it must never be active in production.

---

#### Example 2: Problem Details for a Validation Error

```csharp
return Results.Problem(
    statusCode: StatusCodes.Status400BadRequest,
    title: "Validation failed",
    detail: "One or more fields are invalid.");
```

**Expected Output (conceptual)**

```
HTTP 400 with a Problem Details JSON body: status, title, detail.
No stack trace, no internal paths, no database schema.
```

**Why this output occurs**

- Problem Details provides a standard, consistent error contract without leaking internals.

---

### Real-World Cases

- **APIs** — Consistent Problem Details responses for client and server errors.
- **Web applications** — Generic error pages with correlation IDs for support.
- **Authentication flows** — Generic “invalid credentials” messages to prevent account enumeration.
- **Payment processing** — Fail-closed error handling that does not reveal internal state.

---

### References

- Secure Error Handling for C# Applications — Pluralsight - https://www.pluralsight.com/labs/aws/secure-error-handling-for-c-applications
- RESTful API best practices — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- How to Build a Global Exception Handler in ASP.NET Core — raw.githubusercontent.com - https://raw.githubusercontent.com/…/global-exception-handler

---

## 7. Logging

### Definitions

**Core Definition**
Security logging is the practice of recording security-relevant events — authentication, authorization, input validation failures, and anomalies — in a structured, queryable form that supports detection, investigation, and compliance.

**Technical Definition**
In .NET, logging is provided by `Microsoft.Extensions.Logging` (`ILogger<T>`, `LoggerFactory`). Security logging should capture **authentication failures**, **authorization failures**, **input validation failures**, and **anomalies** with sufficient user context for identifying suspicious accounts. Logs must **never contain sensitive data** — credentials, tokens, PII, or request/response bodies — at any level, including Debug/Trace. Structured logging with message templates (`{Name}`) is preferred over string interpolation. Logs must be protected and monitored; collecting data without reviewing it is useless.

**Beginner-Friendly Explanation**
Security logging is “keeping a record of the important security events so you can detect attacks and investigate incidents.” Log who logged in, who failed to log in, who tried to access something they were not allowed to see, and any suspicious activity. But never log passwords, tokens, or personal data — those logs become a new security risk.

---

### Purposes

- To detect breaches and suspicious activity in close to real time.
- To support digital forensics and post-mortem analysis after an incident.
- To satisfy compliance and audit requirements.
- To identify malicious accounts and privilege escalation attempts.
- To provide accountability through audit trails.
- To enable alerting on security anomalies.

---

### Syntax Rules and Structure

#### General Syntax (Structured Security Logging)

```csharp
_logger.LogWarning("Authentication failed for {User} from {IPAddress}",
    username, ipAddress);

_logger.LogWarning("Authorization failed for {User} on {Endpoint}",
    user.Identity?.Name, endpoint);
```

#### General Syntax (Source-Generated Logging — `[LoggerMessage]`)

```csharp
[LoggerMessage(Level = LogLevel.Warning,
    Message = "Authentication failed for {User} from {IPAddress}")]
private static partial void LogAuthFailure(
    ILogger logger, string user, string ipAddress);
```

#### Syntax Rules

- Log authentication failures, authorization failures, and input validation failures.
- Use structured logging with template placeholders (`{Name}`), never string interpolation.
- Never log credentials, tokens, PII, or request/response bodies at any level.
- Wrap or propagate exceptions with logging; do not swallow silently.
- Protect log storage and monitor logs with alerting rules.

#### Constraints and Limitations

- Logging every request can create noise and storage cost; focus on security-relevant events.
- Logs must be protected; an unprotected log store is itself a vulnerability.
- Over-logging sensitive data is a common compliance failure.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Logging Security Events Without Sensitive Data

```csharp
// ✅ Log the event, not the password
_logger.LogWarning("Login failed for {Email}", email);

// ❌ Never log the password or token
// _logger.LogInformation("Login attempt: {Email} {Password}", email, password);
```

**Expected Output (conceptual)**

```
Log entry: "Login failed for alice@example.com"
No password, no token, no sensitive data in the log.
```

**Why this output occurs**

- The structured message captures the security event while excluding the sensitive value.

---

#### Example 2: Source-Generated Security Logging

```csharp
public partial class AuthService
{
    [LoggerMessage(Level = LogLevel.Warning,
        Message = "Authorization failed for {User} on {Endpoint}")]
    private static partial void LogAuthzFailure(
        ILogger logger, string user, string endpoint);

    public void CheckAccess(string user, string endpoint)
    {
        // ...
        LogAuthzFailure(_logger, user, endpoint);
    }
}
```

**Expected Output (conceptual)**

```
Structured log entry: "Authorization failed for alice on /admin"
Queryable by User and Endpoint.
```

**Why this output occurs**

- `[LoggerMessage]` generates zero-allocation structured logging code.

---

### Real-World Cases

- **Authentication systems** — Log failed logins and lockouts for detection of credential stuffing.
- **Authorization** — Log 403 responses for privilege escalation attempts.
- **Input validation** — Log rejected inputs to detect injection probing.
- **Compliance** — Audit trails for access to sensitive data.

---

### References

- Security logging and monitoring — Microsoft Learn (OWASP Top 10 for .NET) - https://github.com/microsoftdocs/learn/blob/5e8fe32a89d7c9bb6b315f4c74ef202f5616215c/learn-pr/aspnetcore/owasp-top-10-for-dotnet-developers/includes/11-security-logging-monitoring.md
- Microsoft.Extensions.Logging — Folder-Specific Guidance — dotnet/runtime - https://github.com/dotnet/runtime/blob/deae2e32bf0e018e09d64597b7c59b62b73bb3ce/.github/instructions/extensions-logging.instructions.md
- Secure Error Handling for C# Applications — Pluralsight - https://www.pluralsight.com/labs/aws/secure-error-handling-for-c-applications

---

## 8. Secret Management

### Definitions

**Core Definition**
Secret management is the practice of storing, accessing, rotating, and auditing sensitive values — connection strings, API keys, passwords, certificates — in a secure, controlled manner that keeps them out of source code and plaintext configuration.

**Technical Definition**
In .NET, secret management spans **development** and **production**. In development, the **Secret Manager** (`dotnet user-secrets`) stores values in the user profile, outside the repository; it is explicitly **not for production** and stores values unencrypted. In production, secrets should be loaded at runtime from a **managed vault** — Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault — using a narrow managed identity. `appsettings.json` and `web.config` are the wrong place for secrets because they end up in Git. Secret access should be audited, rotated, and scoped to the minimum required permissions.

**Beginner-Friendly Explanation**
Secret management is “keeping your passwords and keys out of your code.” If you put a connection string in `appsettings.json` and commit it to Git, anyone with repository access can steal your database. Instead, use Secret Manager for local development and a managed vault (like Azure Key Vault) in production. The application loads the secret at runtime; the secret never appears in source control.

---

### Purposes

- To prevent secrets from being exposed through source control or configuration files.
- To centralise secret access, rotation, and auditing.
- To scope secret access to the minimum required identity.
- To satisfy compliance requirements for credential management.
- To enable secure rotation without redeploying the application.

---

### Syntax Rules and Structure

#### General Syntax (Development — Secret Manager)

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Server=...;Database=...;"
```

#### General Syntax (Production — Azure Key Vault)

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{keyVaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
```

#### General Syntax (Reading Secrets via `IOptions`)

```csharp
public class PaymentOptions
{
    public string ApiKey { get; set; } = "";
}

builder.Services.Configure<PaymentOptions>(
    builder.Configuration.GetSection("Payments"));
```

#### Syntax Rules

- Never commit secrets to source control; use Secret Manager in development.
- Use a managed vault in production with a narrow managed identity.
- Do not store secrets in `appsettings.json` or `web.config`.
- Audit secret access and rotate secrets regularly.
- Do not log secrets; masking and redaction must be applied in logging pipelines.

#### Constraints and Limitations

- Secret Manager stores values unencrypted in the user profile; it is for development only.
- Key Vault access requires network connectivity and appropriate RBAC permissions.
- Secret rotation must be coordinated with applications that cache secrets.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Development vs. Production Secrets

```bash
# Development: Secret Manager
dotnet user-secrets set "ConnectionStrings:Default" "Server=localhost;..."

# Production: Azure Key Vault
# Configure the Key Vault provider in Program.cs
```

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{keyVaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
```

**Expected Output (conceptual)**

```
Development: secrets loaded from the user profile, outside the repository.
Production: secrets loaded from Key Vault at runtime; no secrets in source or appsettings.
```

**Why this output occurs**

- Secret Manager keeps development secrets outside Git; Key Vault provides audited, access-controlled storage in production.

---

#### Example 2: Narrow Managed Identity

```csharp
// The app's managed identity is granted only "Key Vault Secrets User" on the specific vault.
builder.Configuration.AddAzureKeyVault(
    new Uri(vaultUri),
    new DefaultAzureCredential());
```

**Expected Output (conceptual)**

```
The application can read secrets from the vault but cannot manage the vault or access other resources.
```

**Why this output occurs**

- The managed identity is scoped to the minimum required role, following least privilege.

---

### Real-World Cases

- **Web APIs** — Connection strings and API keys loaded from Key Vault.
- **CI/CD pipelines** — Secrets injected as environment variables from a vault, not stored in pipeline YAML.
- **Multi-tenant SaaS** — Per-tenant secrets stored in a vault with tenant-scoped access.
- **Compliance** — Audited secret access for SOC 2 and ISO 27001.

---

### References

- Are .NET user secrets safe to use in production? — safeguard.sh - https://safeguard.sh/resources/blog/are-dotnet-user-secrets-safe
- .NET Security Best Practices (2026 Guide) — secrets management - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- Azure Key Vault configuration provider — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
- Secrets — Altinn Studio - https://docs.altinn.studio

---

## 9. Secure Coding Best Practices

### Definitions

**Core Definition**
Secure coding best practices are empirically grounded guidelines for writing C# code that resists the common vulnerability classes described in this cheat sheet.

**Technical Definition**
Best practices span **sink-aware defence** (parameterisation for SQL, encoding for HTML/JS, canonicalisation for paths), **framework-first development** (prefer safe built-in APIs over hand-rolled logic), **boundary validation** (validate at trust boundaries, not deep in the call stack), **least privilege** (minimal permissions everywhere), **secure defaults** (HTTPS, authorization by default, safe deserialisation), **fail-closed error handling** (generic client errors, full internal logs), **secret hygiene** (runtime loading from managed vaults), **security logging** (capture events, never log secrets), **dependency auditing** (NuGet Audit, SBOMs), and **automated verification** (SAST, DAST, analyzers).

**Beginner-Friendly Explanation**
Secure coding best practices are “the habits that keep your code safe.” They include: never trust input, always use parameters for SQL, let Razor encode output, run with the fewest permissions, don't show users your stack traces, never log passwords, keep secrets out of Git, and audit your NuGet packages. Following these prevents the vulnerabilities on the OWASP Top 10.

---

### Purposes

- To prevent the common vulnerability classes described in this cheat sheet.
- To establish a secure baseline for every .NET application.
- To satisfy OWASP, NIST, and regulatory requirements.
- To integrate security into the development lifecycle.
- To reduce the blast radius of a compromise.

---

### Guidelines and Structure

#### Sink-Aware Defence

```
✅ Parameterise SQL (EF Core LINQ / DbParameter).
✅ Encode output for HTML, JavaScript, URL, and CSS contexts.
✅ Canonicalise and contain file paths.
✅ Avoid shell parsing; use explicit argument lists.
```

#### Framework-First

```
✅ Prefer Razor auto-encoding over Html.Raw.
✅ Prefer System.Text.Json over BinaryFormatter.
✅ Prefer RandomNumberGenerator over System.Random for tokens.
✅ Prefer CryptographicOperations.FixedTimeEquals over == for secret comparison.
```

#### Boundary Validation

```
✅ Validate at the trust boundary with TryParse and model validation.
✅ Prefer allow-lists over deny-lists.
✅ Validate structure (type, length, range, format) early.
```

#### Least Privilege

```
✅ Require authorization by default (fallback policy).
✅ Use least-privilege database users and managed identities.
✅ Run containers as non-root with allowPrivilegeEscalation: false.
```

#### Secure Defaults

```
✅ HTTPS and HSTS outside development.
✅ Disable developer exception page in production.
✅ Keep TypeNameHandling at None; use System.Text.Json.
✅ Enable NuGet Audit and treat warnings as errors.
```

#### Error Handling and Logging

```
✅ Fail closed; return generic errors to clients.
✅ Log authentication, authorization, and validation failures.
✅ Never log credentials, tokens, PII, or request/response bodies.
✅ Protect log storage and monitor with alerts.
```

#### Secret Management

```
✅ Use Secret Manager in development.
✅ Use a managed vault in production with a narrow identity.
✅ Never commit secrets to source control.
```

#### Automated Verification

```
✅ Run SAST (Roslyn analyzers, CodeQL, SonarQube).
✅ Run DAST against running applications.
✅ Audit NuGet dependencies; treat warnings as errors.
✅ Use OWASP ZAP or equivalent for penetration testing.
```

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure-by-Default ASP.NET Core Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/public", [AllowAnonymous] () => "Public");
app.MapGet("/private", () => "Private"); // protected by fallback
app.Run();
```

**Expected Output (conceptual)**

```
Unauthenticated request to /private: 401.
Authenticated request to /private: 200.
Security headers present; HTTPS enforced; HSTS sent outside development.
```

**Why this output occurs**

- Each best practice is configured explicitly; the application is secure by default.

---

### Real-World Cases

- **Enterprise .NET applications** — Layered security controls across the OWASP Top 10.
- **Open-source libraries** — NuGet Audit, safe deserialisation, and signed packages.
- **Cloud-native services** — Managed identities, Key Vault, and container hardening.
- **Regulated industries** — Documented alignment with OWASP ASVS and NIST.

---

### References

- C# Secure Coding Guide (2026): Patterns and Anti-Patterns — safeguard.sh - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- ASP.NET Core Security Best Practices 2026 — Offensive360 - https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP .NET Security Cheat Sheet — CyberSecAI - https://github.com/CyberSecAI/genai-sec-agents/blob/main/data/owasp_cache/dotnet-security.md

---

## Summary Table: Secure Coding Controls

| Concern | Primary Control | .NET Primitive |
|---|---|---|
| **Input validation** | Validate at the boundary with allow-lists | `TryParse`, Data Annotations, FluentValidation |
| **Output encoding** | Context-aware encoding | Razor auto-encoding, `Json.Serialize` |
| **SQL injection** | Parameterised queries | EF Core LINQ, `DbParameter`, `FromSqlInterpolated` |
| **Shell injection** | No shell; explicit arguments | `ProcessStartInfo.ArgumentList` |
| **Path traversal** | Canonicalise and contain | `Path.GetFullPath` + containment check |
| **Least privilege** | Deny by default | Fallback policy, low-privilege identities |
| **Secure defaults** | HTTPS, authorization, safe serialisation | `UseHsts`, `FallbackPolicy`, `System.Text.Json` |
| **Error handling** | Fail closed; generic client errors | `UseExceptionHandler`, Problem Details |
| **Logging** | Structured security events, no secrets | `ILogger`, `[LoggerMessage]` |
| **Secret management** | Runtime loading from managed vault | Secret Manager (dev), Key Vault (prod) |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`BinaryFormatter`** | Removed in .NET 9; cannot be made secure; do not use |
| **`Newtonsoft.Json` `TypeNameHandling.All/Auto/Objects`** | Unsafe deserialisation RCE; keep at `None` |
| **`FromSqlRaw` with concatenation** | SQL injection risk; use `FromSqlInterpolated` |
| **`Html.Raw` on untrusted data** | XSS risk; let Razor encode |
| **`UseDeveloperExceptionPage` in production** | Information disclosure; use `UseExceptionHandler` |
| **Secret Manager in production** | Unencrypted; development only |
| **`System.Random` for tokens** | Not cryptographically secure; use `RandomNumberGenerator` |
| **`==` for secret comparison** | Timing side channel; use `CryptographicOperations.FixedTimeEquals` |
| **NuGet Audit** | Default in .NET 8+; treat warnings as errors in CI |
| **OWASP Top 10:2021** | Superseded by the 2025 edition; update mappings |

---

## References (Consolidated)

- C# Secure Coding Guide (2026): Patterns and Anti-Patterns — safeguard.sh - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- ASP.NET Core Security Best Practices 2026 — Offensive360 - https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP .NET Security Cheat Sheet — CyberSecAI - https://github.com/CyberSecAI/genai-sec-agents/blob/main/data/owasp_cache/dotnet-security.md
- Security And Identity — Secure defaults and least privilege - https://github.com/midudev/autoskills/blob/93d4a7a94aec211e5b322c451d8ef01d5e383e0a/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- Are .NET user secrets safe to use in production? — safeguard.sh - https://safeguard.sh/resources/blog/are-dotnet-user-secrets-safe
- Azure Key Vault configuration provider — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
- Secrets — Altinn Studio - https://docs.altinn.studio
- Secure Error Handling for C# Applications — Pluralsight - https://www.pluralsight.com/labs/aws/secure-error-handling-for-c-applications
- RESTful API best practices — Azure Architecture Center - https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Security logging and monitoring — Microsoft Learn (OWASP Top 10 for .NET) - https://github.com/microsoftdocs/learn/blob/5e8fe32a89d7c9bb6b315f4c74ef202f5616215c/learn-pr/aspnetcore/owasp-top-10-for-dotnet-developers/includes/11-security-logging-monitoring.md
- Microsoft.Extensions.Logging — Folder-Specific Guidance — dotnet/runtime - https://github.com/dotnet/runtime/blob/deae2e32bf0e018e09d64597b7c59b62b73bb3ce/.github/instructions/extensions-logging.instructions.md
- Validation patterns — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/master/skills/validation-patterns/SKILL.md
- Code Checklist — Altinn Studio - https://docs.altinn.studio