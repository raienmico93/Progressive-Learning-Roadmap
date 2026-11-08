# ASP.NET Core Common Vulnerabilities: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the most consequential web application vulnerabilities in the .NET ecosystem. It is framed by the **OWASP Top 10 (2021)** and the **OWASP API Security Top 10**, and draws on Microsoft's official security documentation and the .NET security community. The framework provides strong primitives for every vulnerability class covered here, but most protections are **opt-in or environment-sensitive**—a service can work perfectly in a demo while shipping SQL injection through `FromSqlRaw`, XSS through `Html.Raw`, or broken access control through a forgotten `[Authorize]` attribute. Each vulnerability is treated independently with a uniform structure—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A vulnerability is a weakness in an application's design, implementation, or configuration that an attacker can exploit to compromise confidentiality, integrity, or availability. |
| **Technical Definition** | Common web vulnerabilities in ASP.NET Core arise from **trust boundary violations**—untrusted data crossing into a context where it is interpreted as code (SQL, HTML, JavaScript, serialized types, authorization decisions) or from **missing security controls** (authorization checks, transport encryption, secret management). The OWASP Top 10 (2021) categorizes the highest-risk classes: Broken Access Control (A01), Cryptographic Failures (A02), Injection (A03), Insecure Design (A04), Security Misconfiguration (A05), Vulnerable and Outdated Components (A06), Identification and Authentication Failures (A07), Software and Data Integrity Failures (A08), Security Logging and Monitoring Failures (A09), and Server-Side Request Forgery (A10). |
| **Beginner-Friendly Explanation** | A vulnerability is a **way an attacker can make your app do something it shouldn't**. SQL injection lets them run their own database queries. XSS lets them run their own JavaScript in your users' browsers. CSRF tricks a logged-in user into performing an action they didn't intend. SSRF makes your server fetch internal resources on the attacker's behalf. Insecure deserialization lets them run code on your server. Broken access control lets them see data they shouldn't. The OWASP Top 10 is the industry's ranked list of the most dangerous vulnerabilities. |

### Key Characteristics

- **Trust-boundary driven** – Every vulnerability involves untrusted data crossing into a context where it is interpreted as code or used for a security decision.
- **Framework primitives are strong but opt-in** – ASP.NET Core provides parameterization, output encoding, antiforgery tokens, and fallback authorization policies, but developers must use them correctly.
- **Configuration-sensitive** – Security misconfiguration (detailed errors, missing headers, cleartext HTTP) is one of the most common causes of exposure.
- **Defense in depth required** – No single control is sufficient; parameterization, encoding, validation, authorization, and monitoring work together.
- **Measurable and testable** – Every vulnerability class has concrete mitigation patterns and verification techniques (SAST, DAST, unit tests).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **HTTP**, **middleware**, **dependency injection**, and **claims-based identity**.
- Familiarity with **C#**, **Razor**, **EF Core**, and **SQL**.
- Basic understanding of the **OWASP Top 10** and **STRIDE**.

### Related Programming Areas

- **Authentication and authorization** – Broken access control and authentication failures.
- **Data access** – SQL injection and insecure deserialization.
- **Output rendering** – XSS.
- **State-changing requests** – CSRF.
- **Outbound HTTP** – SSRF.
- **Configuration and headers** – Security misconfiguration and sensitive data exposure.

### Core Concepts / Features

1. **SQL Injection** – Injecting SQL through unsafe raw queries.
2. **XSS (Cross-Site Scripting)** – Injecting JavaScript through unencoded output.
3. **CSRF (Cross-Site Request Forgery)** – Forcing authenticated users to perform unwanted actions.
4. **SSRF (Server-Side Request Forgery)** – Making the server fetch attacker-controlled URLs.
5. **Insecure Deserialization** – Instantiating attacker-controlled types during deserialization.
6. **Broken Access Control** – Missing or flawed authorization checks.
7. **Sensitive Data Exposure** – Exposing data in transit or at rest without encryption.
8. **Security Misconfiguration** – Insecure defaults, missing headers, and exposed diagnostics.
9. **OWASP Top 10** – The industry-standard ranking of web vulnerability classes.
10. **Security Best Practices** – The baseline controls every ASP.NET Core app should implement.

---

## 1. SQL Injection

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | SQL injection is a vulnerability where an attacker can insert or manipulate SQL statements through user-controlled input, potentially reading, modifying, or deleting arbitrary data. |
| **Technical Definition** | SQL injection (CWE-89) occurs when user input is **concatenated or interpolated into a SQL string** that is executed by the database. EF Core's LINQ provider **parameterizes queries by default**—values are sent as `DbParameter` objects separate from the SQL text and cannot be interpreted as executable syntax. The vulnerability reappears when developers use **`FromSqlRaw` or `ExecuteSqlRaw` with string concatenation**, or when they use **`FromSqlInterpolated` incorrectly** (though the interpolation overload is safe when used as intended). |
| **Beginner-Friendly Explanation** | SQL injection is the **“inject your own SQL command”** attack. If your code builds a SQL query by gluing user input into a string—`"SELECT * FROM Users WHERE Name = '" + name + "'"`—an attacker can enter `' OR 1=1--` and retrieve every row, or worse, drop tables. EF Core's normal LINQ queries are safe because they send the value separately from the SQL. The danger is in the raw SQL escape hatches. |

### Purposes (Mitigation)

- **To** **eliminate string concatenation** from all SQL construction—use LINQ or parameterized raw SQL.
- **To** **use `FromSqlInterpolated`** instead of `FromSqlRaw` when building dynamic SQL with user input, because the interpolated overload parameterizes automatically.
- **To** **audit `FromSqlRaw` and `ExecuteSqlRaw` usage**—the only safe pattern is `{0}` placeholders with separate parameter values, never string building.
- **To** **validate and allowlist dynamic identifiers** (column names, sort directions) that cannot be parameterized.
- **To** **use least-privilege database accounts** so a successful injection cannot drop tables or access other schemas.

### Syntax Rules and Structure

#### Safe: LINQ (Parameterized by Default)

```csharp
var user = await _db.Users
    .Where(u => u.Email == email)
    .FirstOrDefaultAsync(ct);
```

#### Safe: `FromSqlInterpolated` (Automatic Parameterization)

```csharp
var users = await _db.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {name}")
    .ToListAsync(ct);
```

#### Safe: `FromSqlRaw` with Placeholders

```csharp
var users = await _db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = {0}", name)
    .ToListAsync(ct);
```

#### ❌ Vulnerable: String Concatenation

```csharp
// NEVER DO THIS — user input becomes executable SQL
var users = await _db.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'")
    .ToListAsync(ct);
```

**Syntax rules:**

- **LINQ queries are safe by default**—`Where`, `Select`, `OrderBy`, and `Include` all generate parameterized SQL.
- **`FromSqlInterpolated` is safe**—it decomposes the interpolated string into parameters.
- **`FromSqlRaw` is safe only when the SQL string is a constant** with `{0}` placeholders and the values are passed separately.
- **Dynamic SQL identifiers** (column names, table names, sort directions) must be **allowlisted**—they cannot be parameterized.

**Constraints and limitations:**

- **`FromSqlRaw` with string concatenation is the primary injection vector** in EF Core applications.
- **Stored procedure wrappers** that concatenate parameters are equally vulnerable.
- **`EF.Functions.Like`** with raw patterns is safe if the pattern is parameterized, but `%` and `_` wildcards from user input should be escaped.

### Annotated Complete Step-by-Step Code Example

```csharp
// ❌ VULNERABLE: concatenation into raw SQL
public async Task<List<User>> SearchVulnerable(string name)
    => await _db.Users
        .FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'")
        .ToListAsync();

// ✅ SAFE: parameterized raw SQL
public async Task<List<User>> SearchSafe(string name)
    => await _db.Users
        .FromSqlRaw("SELECT * FROM Users WHERE Name = {0}", name)
        .ToListAsync();

// ✅ SAFE: interpolated raw SQL (automatic parameters)
public async Task<List<User>> SearchSafeInterpolated(string name)
    => await _db.Users
        .FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {name}")
        .ToListAsync();
```

**Step-by-step explanation:**

1. The vulnerable version concatenates `name` directly into the SQL string—an attacker entering `' OR 1=1--` retrieves every user.
2. The safe version uses `{0}` as a placeholder and passes `name` as a separate parameter—the value cannot become executable SQL.
3. The interpolated version looks like string interpolation but is safe because EF Core decomposes it into parameters.

**Expected Output:** The vulnerable version returns all users for a malicious input; the safe versions return only the matching user (or none).

**Why this output?** Parameterized queries send values separately from the SQL text, so the database treats them as data, not code.

### Real-World Cases

- **Login bypass:** `' OR 1=1--` in a username field bypasses authentication.
- **Data exfiltration:** `UNION SELECT` attacks retrieve data from other tables.
- **Schema manipulation:** `; DROP TABLE Users--` destroys data when the database account has DDL permissions.

### References

- SQL Injection Prevention in C# with Entity Framework (safeguard.sh) – https://safeguard.sh/resources/blog/sql-injection-prevention-in-c-with-entity-frameworklinq
- SQL Injection Prevention in EF Core (.NET 2026 Guide) – https://safeguard.sh/resources/blog/sql-injection-prevention-in-ef-core
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html

---

## 2. XSS (Cross-Site Scripting)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | XSS is a vulnerability that allows an attacker to inject client-side scripts (usually JavaScript) into web pages viewed by other users, enabling cookie theft, session hijacking, DOM manipulation, and redirection. |
| **Technical Definition** | XSS occurs when an application takes user input and outputs it to a page **without validation, encoding, or escaping**. ASP.NET Core's Razor engine **automatically HTML-encodes all variable output**—the `@` directive applies HTML attribute encoding rules. XSS in ASP.NET Core therefore typically arises from **bypassing Razor encoding** (`Html.Raw`, `@Html.Raw()`), placing untrusted data inside **JavaScript contexts** without proper encoding, or **APIs returning raw markup** that client applications trust. |
| **Beginner-Friendly Explanation** | XSS is the **“inject your own JavaScript”** attack. If your page displays a comment without encoding it, an attacker can submit `<script>alert('hacked')</script>` and every visitor runs that script. The script can steal cookies, redirect users, or deface the page. Razor protects you by encoding output by default—but if you use `Html.Raw` or build JavaScript strings carelessly, you open the door. |

### Purposes (Mitigation)

- **To** **rely on Razor's automatic encoding**—use `@variable` and let the engine encode it.
- **To** **avoid `Html.Raw` with untrusted input**—encode or sanitize first using `HtmlEncoder.Default.Encode()`.
- **To** **use JavaScript encoding** when embedding data in scripts (`Json.Serialize` or `JavaScriptEncoder`).
- **To** **implement a Content-Security-Policy (CSP)** that restricts script sources and disables `unsafe-inline`.
- **To** **validate input** as a secondary defense—never as the primary XSS control.

### Syntax Rules and Structure

#### Safe: Razor `@` (Automatic HTML Encoding)

```cshtml
<p>@Model.UserComment</p>
```

#### Safe: `HtmlEncoder` for Explicit Encoding

```csharp
var safe = HtmlEncoder.Default.Encode(userInput);
```

#### Safe: JSON Serialization for JavaScript Contexts

```cshtml
<script>
    var data = @Html.Raw(JsonSerializer.Serialize(Model.Data));
</script>
```

#### ❌ Vulnerable: `Html.Raw` with Untrusted Input

```cshtml
@Html.Raw(Model.UserComment)   @* NEVER with untrusted input *@
```

**Syntax rules:**

- **The `@` directive HTML-encodes variable output**—`<` becomes `&lt;` and scripts are neutralized.
- **`Html.Raw()` bypasses encoding**—it is safe only for trusted, already-sanitized content.
- **JavaScript contexts require JavaScript encoding**—HTML encoding is insufficient inside `<script>` blocks.
- **CSP is a defense-in-depth control**—it blocks inline scripts even if injection occurs.

**Constraints and limitations:**

- **`Html.Raw` is the primary XSS vector in Razor applications**—audit every usage.
- **APIs returning HTML** must encode or sanitize output; client applications may trust API responses.
- **DOM-based XSS** occurs when client-side JavaScript writes untrusted data to the DOM without encoding—server-side encoding cannot prevent it.

### Annotated Complete Step-by-Step Code Example

```cshtml
@* Safe: Razor encodes automatically *@
<div>@Model.Comment</div>

@* ❌ Vulnerable: bypasses encoding *@
<div>@Html.Raw(Model.Comment)</div>

@* Safe: encode before Raw, or better — don't use Raw *@
<div>@Html.Raw(HtmlEncoder.Default.Encode(Model.Comment))</div>
```

**Step-by-step explanation:**

1. The first line uses `@Model.Comment`—Razor HTML-encodes it, so `<script>` is displayed as text.
2. The second line uses `Html.Raw`—the script executes if `Comment` contains malicious markup.
3. The third line encodes before `Raw`, but the encoding is redundant—the correct fix is to avoid `Raw` entirely for untrusted content.

**Expected Output:** The safe line displays `<script>alert('xss')</script>` as text; the vulnerable line executes the script.

**Why this output?** Razor's automatic encoding neutralizes HTML metacharacters; `Html.Raw` bypasses that protection.

### Real-World Cases

- **Comment sections:** Unencoded comments execute scripts for every visitor.
- **User profiles:** Display names or bios with `<script>` tags compromise sessions.
- **API responses:** APIs returning user-generated HTML without encoding expose client applications.

### References

- Prevent Cross-Site Scripting (XSS) in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- How do you prevent XSS in ASP.NET Core and C# web apps? (safeguard.sh) – https://safeguard.sh/resources/blog/prevent-xss-aspnet-core
- Security cards (HtmlEncoder guidance) – https://github.com/reware-labs/securitycards

---

## 3. CSRF (Cross-Site Request Forgery)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | CSRF is an attack where a malicious site causes a logged-in user's browser to send an authenticated request to a vulnerable application, performing an action the user did not intend. |
| **Technical Definition** | CSRF exploits the browser's automatic inclusion of **cookies** (including authentication cookies) with every request to a domain. ASP.NET Core provides **antiforgery tokens**—a hidden form field and a companion cookie that must match for unsafe HTTP methods (POST, PUT, DELETE, PATCH). The `[ValidateAntiForgeryToken]` filter enforces validation per action, `[AutoValidateAntiforgeryToken]` validates all unsafe methods by default, and `app.UseAntiforgery()` (in .NET 8+) wires the middleware. |
| **Beginner-Friendly Explanation** | CSRF is the **“trick the logged-in user”** attack. Imagine you're logged into your bank, then visit a malicious site that silently submits a transfer form to your bank. Your browser automatically sends your bank cookies, so the bank thinks the request came from you. Antiforgery tokens prevent this: the bank requires a secret token that only its own pages can generate, so the malicious site cannot forge a valid request. |

### Purposes (Mitigation)

- **To** **require antiforgery tokens** for all state-changing requests (POST, PUT, DELETE, PATCH).
- **To** **use `[AutoValidateAntiforgeryToken]`** broadly so POST actions are protected by default, rather than forgetting `[ValidateAntiForgeryToken]` on individual actions.
- **To** **enable the antiforgery middleware** (`app.UseAntiforgery()`) in .NET 8+.
- **To** **set `SameSite=Strict` or `Lax`** on authentication cookies as an additional layer.
- **To** **exempt only explicitly safe endpoints** (e.g., API endpoints using token authentication rather than cookies) with `[IgnoreAntiforgeryToken]`.

### Syntax Rules and Structure

#### Global Protection: `AutoValidateAntiforgeryToken`

```csharp
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

#### Per-Action Protection: `ValidateAntiForgeryToken`

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Transfer(TransferInput input) { ... }
```

#### Razor Form: Automatic Token Emission

```cshtml
<form asp-action="Transfer" method="post">
    <!-- Token is emitted automatically by the form tag helper -->
    <input type="text" name="Amount" />
    <button type="submit">Transfer</button>
</form>
```

#### ❌ Vulnerable: POST Without Antiforgery

```csharp
[HttpPost]   // No antiforgery validation — CSRF vulnerable
public IActionResult Transfer(TransferInput input) { ... }
```

**Syntax rules:**

- **`[ValidateAntiForgeryToken]`** requires a valid token on the decorated action—including GET requests, though GET should be safe and idempotent anyway.
- **`[AutoValidateAntiforgeryToken]`** validates all unsafe methods (POST, PUT, DELETE, PATCH) but skips GET, HEAD, OPTIONS, and TRACE.
- **Razor form tag helpers emit the token automatically**—`@Html.AntiForgeryToken()` is needed only for manual forms.

**Constraints and limitations:**

- **API endpoints using bearer tokens are not CSRF-vulnerable**—CSRF exploits cookie-based authentication; token-authenticated APIs do not need antiforgery.
- **Antiforgery tokens must be validated on the server**—client-side checks alone are useless.
- **AJAX requests** must include the token in a header (e.g., `RequestVerificationToken`) or form field.

### Annotated Complete Step-by-Step Code Example

```csharp
// Global antiforgery protection
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});

app.UseAntiforgery();   // .NET 8+

// Controller action — protected automatically by the global filter
[HttpPost]
public IActionResult Transfer(TransferInput input)
{
    // Antiforgery validation has already occurred
    return RedirectToAction("Success");
}
```

**Step-by-step explanation:**

1. The global `AutoValidateAntiforgeryToken` filter validates tokens on every unsafe HTTP method.
2. The `Transfer` action does not need an explicit `[ValidateAntiForgeryToken]` attribute—the global filter covers it.
3. The middleware ensures tokens are available and validated.

**Expected Output:** A POST request without a valid antiforgery token receives a 400 Bad Request; legitimate form submissions with tokens succeed.

**Why this output?** The global filter enforces token validation on all state-changing requests, closing the CSRF hole that a forgotten per-action attribute would leave.

### Real-World Cases

- **Bank transfers:** CSRF can trigger unauthorized transfers if forms lack antiforgery tokens.
- **Account settings:** Email or password changes via CSRF can lead to account takeover.
- **Admin actions:** CSRF on admin endpoints can compromise the entire application.

### References

- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Understanding Anti-Forgery in ASP.NET Core (Duende Software) – https://duendesoftware.com/blog/understanding-anti-forgery-in-aspnet-core
- ASP.NET Core Security Checklist (antiforgery section) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist

---

## 4. SSRF (Server-Side Request Forgery)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | SSRF is a vulnerability where an attacker can make the server perform HTTP requests to attacker-controlled URLs, potentially accessing internal services, cloud metadata endpoints, or other resources behind the firewall. |
| **Technical Definition** | SSRF (OWASP A10:2021) occurs when an application fetches a URL supplied by user input without validating the destination. Attackers exploit this to reach internal networks (`http://169.254.169.254/` for cloud metadata, `http://localhost:8080/admin`), scan internal ports, or bypass network access controls. Microsoft provides the **AntiSSRF library** for .NET, which integrates with `HttpClient` via `AntiSSRFHandler` and validates URLs against policies (IP allowlists, domain allowlists, protocol restrictions, DNS rebinding protection, redirect re‑validation). |
| **Beginner-Friendly Explanation** | SSRF is the **“make the server fetch something it shouldn't”** attack. If your app has a feature like “fetch this image URL” or “import from this webhook,” an attacker can supply `http://169.254.169.254/latest/meta-data/` (the AWS metadata endpoint) or `http://internal-db:5432` and trick your server into making the request from inside your network. The fix is to validate every outbound URL against a strict allowlist of trusted destinations. |

### Purposes (Mitigation)

- **To** **validate every outbound URL** against a strict allowlist of trusted domains, IP ranges, and protocols.
- **To** **block access to private IP ranges** (10.x, 172.16–31.x, 192.168.x, 169.254.x) and link-local addresses.
- **To** **re‑validate on redirects**—an attacker can use a trusted URL that redirects to an internal address.
- **To** **disable automatic redirects** when fetching untrusted URLs.
- **To** **use the Microsoft AntiSSRF library** for `HttpClient`-based outbound requests.

### Syntax Rules and Structure

#### Using Microsoft AntiSSRF with `HttpClient`

```csharp
var handler = new AntiSSRFHandler(new AntiSSRFPolicy
{
    AllowedDomains = new[] { "api.trusted.com" },
    BlockPrivateNetworks = true,
    BlockLinkLocalAddresses = true
});

var client = new HttpClient(handler);
```

#### Allowlist Validation Pattern

```csharp
var uri = new Uri(userSuppliedUrl);
if (!trustedHosts.Contains(uri.Host))
    return BadRequest("Destination not allowed");
```

**Syntax rules:**

- **Allowlists are the only reliable SSRF control**—blocklists of “bad” IPs are incomplete and bypassable.
- **Validate the resolved IP**, not just the hostname—DNS rebinding can make a trusted hostname resolve to an internal IP after validation.
- **Re‑validate after redirects**—a trusted URL can redirect to an internal address.

**Constraints and limitations:**

- **URL parsing is surprisingly hard**—`http://127.0.0.1`, `http://[::1]`, and `http://0x7f000001` all point to localhost.
- **DNS rebinding** can bypass hostname allowlists; resolve and validate the IP just before the request.
- **Outbound HTTP from the server should be treated as untrusted**—validate at the boundary, not inside business logic.

### Annotated Complete Step-by-Step Code Example

```csharp
public async Task<IActionResult> FetchImage(string url)
{
    var uri = new Uri(url);

    // Allowlist: only trusted image hosts
    if (!uri.Host.EndsWith("images.example.com"))
        return BadRequest("Host not allowed");

    // Block private IPs (defence in depth)
    if (IsPrivateIp(uri.Host))
        return BadRequest("Private addresses not allowed");

    // Use a handler that re-validates on redirects
    var client = _httpClientFactory.CreateClient("ssrf-safe");
    var response = await client.GetAsync(uri);
    return File(await response.Content.ReadAsByteArrayAsync(), "image/png");
}
```

**Step-by-step explanation:**

1. The hostname is validated against a strict allowlist.
2. Private IP ranges are blocked as a second layer.
3. The HTTP client uses a handler that re-validates on redirects (the AntiSSRF library provides this).

**Expected Output:** Requests to `images.example.com` succeed; requests to `169.254.169.254` or internal hosts are blocked.

**Why this output?** The allowlist and private-IP check prevent the server from fetching internal resources on the attacker's behalf.

### Real-World Cases

- **Image import features:** “Fetch image from URL” can reach cloud metadata endpoints.
- **Webhook testing tools:** Outbound calls can scan internal ports.
- **PDF/screenshot generators:** Headless browsers fetching user-supplied URLs can access internal services.

### References

- Microsoft AntiSSRF Library for .NET (GitHub) – https://github.com/microsoft/antissrf
- Microsoft AntiSSRF open-source library (Help Net Security) – https://www.helpnetsecurity.com/2026/06/17/microsoft-antissrf/
- OWASP SSRF Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html

---

## 5. Insecure Deserialization

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Insecure deserialization is a vulnerability where an attacker can supply serialized data that, when deserialized, instantiates attacker-chosen types, potentially leading to remote code execution (RCE). |
| **Technical Definition** | The attack exploits **type-aware deserialization**—when the serialized data specifies which .NET type to instantiate. `BinaryFormatter` was the classic vector; it was **removed from the runtime in .NET 9**. The modern risk is **Newtonsoft.Json with `TypeNameHandling` enabled** (`All`, `Auto`, `Objects`)—the JSON can name a `$type` field and instantiate a “gadget” type whose deserialization triggers dangerous behaviour. The fix is to **never enable `TypeNameHandling` on untrusted input**, prefer **`System.Text.Json`** (which does not resolve arbitrary types by name), and use explicit DTOs with allowlisted types. |
| **Beginner-Friendly Explanation** | Insecure deserialization is the **“send a malicious object”** attack. When your app deserializes data, it can be tricked into creating objects of types it never intended—and some of those objects have side effects (like running commands) when constructed. `BinaryFormatter` was the worst offender and is now gone, but the same risk remains if you use `Newtonsoft.Json` with type-name handling enabled. The fix: use `System.Text.Json`, bind to explicit DTOs, and never let the input choose the type. |

### Purposes (Mitigation)

- **To** **never use `BinaryFormatter`**—it is removed in .NET 9 and should not be reintroduced via compatibility packages.
- **To** **never set `TypeNameHandling` to `All`, `Auto`, or `Objects`** on untrusted input with `Newtonsoft.Json`.
- **To** **prefer `System.Text.Json`**, which does not resolve arbitrary types by name by default.
- **To** **bind to explicit DTOs**—never deserialize into `object` or an unconstrained base type.
- **To** **use a custom `ISerializationBinder`** with a strict allowlist if polymorphism is genuinely required.

### Syntax Rules and Structure

#### Safe: `System.Text.Json` with a DTO

```csharp
var dto = JsonSerializer.Deserialize<CreateOrderDto>(json);
```

#### Safe: `Newtonsoft.Json` with `TypeNameHandling.None`

```csharp
var settings = new JsonSerializerSettings
{
    TypeNameHandling = TypeNameHandling.None   // Default; safe
};
var dto = JsonConvert.DeserializeObject<CreateOrderDto>(json, settings);
```

#### ❌ Vulnerable: `TypeNameHandling.All`

```csharp
var settings = new JsonSerializerSettings
{
    TypeNameHandling = TypeNameHandling.All   // NEVER with untrusted input
};
var obj = JsonConvert.DeserializeObject<BaseType>(untrustedJson, settings);
```

**Syntax rules:**

- **`System.Text.Json` does not support polymorphic deserialization by default**—it is safe for untrusted input.
- **`TypeNameHandling.None`** is the safe default for `Newtonsoft.Json`—only change it with a custom binder and allowlist.
- **Explicit DTOs** constrain the type and properties that can be deserialized.

**Constraints and limitations:**

- **`BinaryFormatter` is removed but legacy code may still reference it**—treat removal as a security task.
- **Polymorphic deserialization with `System.Text.Json`** requires opt-in `[JsonDerivedType]` attributes—safe because the types are explicitly declared, not attacker-chosen.

### Annotated Complete Step-by-Step Code Example

```csharp
// ❌ VULNERABLE: attacker controls the type
var settings = new JsonSerializerSettings
{
    TypeNameHandling = TypeNameHandling.All
};
var obj = JsonConvert.DeserializeObject<object>(untrustedJson, settings);

// ✅ SAFE: explicit DTO, default settings
var dto = JsonSerializer.Deserialize<CreateOrderDto>(untrustedJson);
```

**Step-by-step explanation:**

1. The vulnerable version deserializes into `object` with `TypeNameHandling.All`—the JSON's `$type` field chooses the type.
2. The safe version deserializes into a concrete DTO with `System.Text.Json`—the type is fixed and the properties are constrained.

**Expected Output:** The vulnerable version can instantiate a gadget type from attacker-controlled JSON; the safe version rejects or ignores unrecognized fields.

**Why this output?** `TypeNameHandling.All` trusts the input to name the type; explicit DTOs and `System.Text.Json` do not.

### Real-World Cases

- **API endpoints accepting JSON:** Deserializing into `object` with `TypeNameHandling` enabled is a direct RCE path.
- **Session state:** Deserializing session data from an untrusted source.
- **ViewState:** Legacy ASP.NET view-state deserialization (related formatters share the same risk).

### References

- Insecure Deserialization in .NET (2026): BinaryFormatter & JSON (safeguard.sh) – https://safeguard.sh/resources/blog/dotnet-insecure-deserialization
- .NET JSON.NET Deserialization RCE (Invicti) – https://www.invicti.com/blog/web-security/json-net-deserialization-vulnerability/
- SOTA-skills security rules (System.Text.Json preference) – https://github.com/martinholovsky/SOTA-skills

---

## 6. Broken Access Control

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Broken access control is a vulnerability where users can access resources or perform actions outside their intended permissions—OWASP Top 10 #1 (A01:2021). |
| **Technical Definition** | Broken access control includes **missing authorization checks** (an endpoint without `[Authorize]`), **insecure direct object references** (IDOR/BOLA—`GET /orders/42` returns another user's order), **broken function-level authorization** (BFLA—a regular user can access an admin endpoint), and **privilege escalation** (changing a role or tenant claim). ASP.NET Core provides `[Authorize]`, `[AllowAnonymous]`, policy-based authorization, and fallback policies, but **forgetting an attribute is silent**—the endpoint simply becomes public. The **fallback policy** is the critical defense: it requires authentication for all endpoints without explicit authorization metadata. |
| **Beginner-Friendly Explanation** | Broken access control is the **“you can see data you shouldn't”** vulnerability. It happens when an endpoint lacks an authorization check, or when the check is too weak—for example, checking that a user is logged in but not checking that they own the specific order they are requesting. The fix is **default-deny**: require authentication everywhere unless explicitly public, and verify **object-level ownership** on every ID-based endpoint. |

### Purposes (Mitigation)

- **To** **configure a fallback authorization policy** requiring authentication for all endpoints without explicit metadata.
- **To** **apply `[Authorize]`** explicitly to controllers and actions, and use `[AllowAnonymous]` sparingly and deliberately.
- **To** **verify object-level ownership** (BOLA prevention)—compare the user's ID/tenant to the resource's owner before returning it.
- **To** **use policy-based authorization** for role and claim checks, not ad-hoc `if` statements.
- **To** **audit for missing authorization** in code review and with static analysis (Semgrep rule for missing `[Authorize]`).

### Syntax Rules and Structure

#### Fallback Policy (Default Deny)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

#### Explicit `[Authorize]` and `[AllowAnonymous]`

```csharp
[Authorize]
public class OrdersController : Controller
{
    [AllowAnonymous]
    public IActionResult Health() => Ok();

    public IActionResult GetOrder(int id)
    {
        var order = _repo.Find(id);
        if (order.CustomerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
            return Forbid();   // BOLA prevention
        return Ok(order);
    }
}
```

#### ❌ Vulnerable: No Authorization Attribute

```csharp
public class AccountController : Controller
{
    public IActionResult GetCitizenTaxId() => Ok(taxId);   // Public!
}
```

**Syntax rules:**

- **The fallback policy applies to endpoints without any authorization metadata**—`[Authorize]` (even without a policy) uses the default policy, not the fallback.
- **`[AllowAnonymous]`** overrides `[Authorize]`—use it only for genuinely public endpoints.
- **Object-level checks** must run after loading the resource—`[Authorize]` cannot see the resource because it runs before model binding.

**Constraints and limitations:**

- **The fallback policy does not protect endpoints with `[Authorize]`**—it only covers unannotated endpoints.
- **Role checks are coarse-grained**—use policies and claims for fine-grained access.
- **BOLA is a business-logic flaw**—scanners rarely catch it; manual authorization matrices are required.

### Annotated Complete Step-by-Step Code Example

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

[Authorize]
public class OrdersController : Controller
{
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _repo.FindAsync(id);
        if (order is null) return NotFound();
        if (order.CustomerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
            return Forbid();   // Authenticated but not the owner
        return Ok(order);
    }
}
```

**Step-by-step explanation:**

1. The fallback policy requires authentication for all endpoints without explicit metadata.
2. `[Authorize]` on the controller ensures authentication for all actions.
3. The action loads the order and verifies that the current user is the owner—returning `403 Forbidden` if not.

**Expected Output:** An unauthenticated request receives 401; an authenticated user requesting another user's order receives 403; the owner receives 200 with the order.

**Why this output?** The fallback policy provides default-deny; the object-level check prevents BOLA.

### Real-World Cases

- **E-commerce:** `GET /api/orders/42` returns order 42 to any logged-in user without an ownership check.
- **Admin panels:** A regular user accesses `/admin/users` because the endpoint lacks `[Authorize(Roles = "Admin")]`.
- **Multi-tenant SaaS:** A user changes the `tenantId` in a request and accesses another tenant's data.

### References

- Broken access control (Microsoft Learn training) – https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/3-broken-access-control
- ASP.NET Core Security Checklist (fallback policy) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- OWASP Top 10: A01 Broken Access Control – https://owasp.org/Top10/A01_2021-Broken_Access_Control/

---

## 7. Sensitive Data Exposure (Cryptographic Failures)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Sensitive data exposure (renamed **Cryptographic Failures** in OWASP Top 10 2021) is the failure to adequately protect sensitive data—passwords, PII, financial data, tokens—in transit and at rest. |
| **Technical Definition** | Cryptographic failures occur when data is transmitted over **cleartext HTTP**, stored **unencrypted** in databases or configuration files, protected with **weak or misconfigured cryptography**, or when **secrets are committed to source control**. ASP.NET Core provides **TLS enforcement** (`UseHttpsRedirection`, `UseHsts`), the **Data Protection API** for encrypting cookies and tokens, and integration with secret stores (Azure Key Vault, User Secrets). |
| **Beginner-Friendly Explanation** | Sensitive data exposure is the **“data leaks because it wasn't protected”** vulnerability. If your app sends passwords over HTTP, anyone on the network can read them. If your database stores credit card numbers unencrypted, a breach exposes them. If you hard-code API keys in source code, anyone with repository access has them. The fixes are straightforward but often forgotten: HTTPS everywhere, encryption at rest, and a secret vault. |

### Purposes (Mitigation)

- **To** **enforce HTTPS everywhere** with `UseHttpsRedirection` and `UseHsts` in production.
- **To** **encrypt sensitive data at rest** using the **Data Protection API** or database-level encryption (TDE, Always Encrypted).
- **To** **never store passwords in plain text**—use ASP.NET Core Identity's `PasswordHasher<T>` (PBKDF2).
- **To** **keep secrets out of source code**—use User Secrets in development and a managed vault (Azure Key Vault) in production.
- **To** **use TLS 1.2+** and avoid deprecated protocols and cipher suites.

### Syntax Rules and Structure

#### HTTPS and HSTS

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}
```

#### Data Protection API

```csharp
var protector = _dataProtectionProvider.CreateProtector("MyApp.Secrets");
var encrypted = protector.Protect(sensitiveValue);
var decrypted = protector.Unprotect(encrypted);
```

#### ❌ Vulnerable: Plaintext Secret in Configuration

```json
{ "ConnectionStrings": { "Default": "Server=...;Password=P@ssw0rd!" } }
```

**Syntax rules:**

- **`UseHsts()` must run outside Development**—HSTS is difficult to undo and should not be enabled on localhost.
- **`UseHttpsRedirection()`** issues 307/308 redirects from HTTP to HTTPS.
- **Data Protection keys must be persisted** in multi-instance deployments—ephemeral keys invalidate cookies and tokens across restarts.

**Constraints and limitations:**

- **HTTPS termination at a proxy requires `UseForwardedHeaders()`**—otherwise `Request.IsHttps` is false and redirects loop.
- **Data Protection is not a substitute for database encryption**—use both for defense in depth.
- **Secrets in environment variables are better than source code but not the long-term answer**—use a vault.

### Annotated Complete Step-by-Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Secrets from a vault (production) or User Secrets (development)
var connectionString = builder.Configuration.GetConnectionString("Default");

// Data Protection for encrypting sensitive fields
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(blobUri)
    .ProtectKeysWithAzureKeyVault(keyUri, credential);

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}
```

**Step-by-step explanation:**

1. The connection string is loaded from configuration (backed by a vault in production).
2. Data Protection keys are persisted to Azure Blob Storage and encrypted with Key Vault—cookies and tokens survive restarts and are shared across instances.
3. HTTPS and HSTS are enforced outside Development.

**Expected Output:** Sensitive data is encrypted in transit (TLS) and at rest (Data Protection); secrets never appear in source code.

**Why this output?** Transport encryption and at-rest encryption are independent layers; secrets are managed by a vault rather than configuration files.

### Real-World Cases

- **Login forms:** Passwords transmitted over HTTP are intercepted.
- **Database backups:** Unencrypted backups expose PII when stolen.
- **CI pipelines:** Secrets in environment variables leak in build logs and artifacts.

### References

- Key encryption at rest in Windows and Azure using ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/implementation/key-encryption-at-rest
- Data Protection & Encryption (dotnet-skills) – https://github.com/wshaddix/dotnet-skills
- ASP.NET Core Security Checklist (transport section) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist

---

## 8. Security Misconfiguration

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security misconfiguration is the failure to properly configure security controls—default credentials, missing headers, verbose errors, unnecessary features, and insecure defaults—creating exploitable weaknesses. |
| **Technical Definition** | Security misconfiguration (OWASP A05:2021) includes **missing security headers** (CSP, `X-Content-Type-Options`, `Referrer-Policy`), **exposed diagnostic endpoints** (developer exception page, Swagger UI in production), **default or weak credentials**, **unnecessary features enabled** (directory browsing, detailed errors), and **insecure default settings** (no fallback authorization policy, permissive CORS). The fix is a **hardened production checklist** with environment-gated features and explicit security headers. |
| **Beginner-Friendly Explanation** | Security misconfiguration is the **“you left the door unlocked”** vulnerability. Your app is not broken—it is configured insecurely. The developer exception page is on in production (leaking stack traces). Swagger UI is public (exposing your entire API). Security headers are missing (allowing XSS). CORS is wide open (allowing any origin). The fix is a checklist: turn off diagnostics in production, add security headers, and lock down CORS. |

### Purposes (Mitigation)

- **To** **add security headers** (CSP, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `X-Frame-Options`) via middleware.
- **To** **gate diagnostic features** (developer exception page, Swagger UI) to Development only.
- **To** **disable the Server header** and other version-disclosing headers.
- **To** **restrict CORS** to trusted origins—never `AllowAnyOrigin()` in production.
- **To** **enforce a fallback authorization policy** so unannotated endpoints fail closed.

### Syntax Rules and Structure

#### Security Headers Middleware

```csharp
app.Use(async (ctx, next) =>
{
    var h = ctx.Response.Headers;
    h["Content-Security-Policy"] = "default-src 'self'; frame-ancestors 'none'";
    h["X-Content-Type-Options"] = "nosniff";
    h["Referrer-Policy"] = "no-referrer";
    await next();
});
```

#### Environment-Gated Diagnostics

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

**Syntax rules:**

- **`UseDeveloperExceptionPage` and `UseSwaggerUI` must be inside `IsDevelopment()`**—never in production.
- **Security headers must be added by middleware**—ASP.NET Core does not send CSP by default.
- **CORS must use `WithOrigins(...)`**—`AllowAnyOrigin` combined with credentials is unsafe.

**Constraints and limitations:**

- **Middleware ordering matters**—headers added after the response starts may not be sent.
- **CSP requires careful tuning**—test in Report-Only mode before enforcing.

### Annotated Complete Step-by-Step Code Example

```csharp
var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
    app.UseExceptionHandler("/Error");
}
else
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    ctx.Response.Headers["Content-Security-Policy"] =
        "default-src 'self'; frame-ancestors 'none'";
    await next();
});
```

**Step-by-step explanation:**

1. Diagnostics are gated to Development.
2. HTTPS and HSTS are enforced outside Development.
3. Security headers are added to every response.

**Expected Output:** Production responses do not include stack traces or Swagger UI; security headers are present on every response.

**Why this output?** The environment gate prevents diagnostic exposure; the middleware adds the headers.

### Real-World Cases

- **Stack trace leakage:** Developer exception page in production reveals source code and configuration.
- **Public Swagger UI:** Exposes the entire API schema to attackers.
- **Missing CSP:** Injected scripts execute because the browser has no restrictions.

### References

- ASP.NET Core Security Checklist (headers, environment gating) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security blueprint (fallback policy, headers) – https://github.com/reware-labs/securitycards
- Security Headers Every ASP.NET Core App Should Have (C# Corner) – https://www.c-sharpcorner.com/article/security-headers-every-aspnet-core-app-should-have/

---

## 9. OWASP Top 10 (2021) — Mapping to ASP.NET Core

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The OWASP Top 10 is a standard awareness document for developers and web application security, representing the most critical security risks to web applications. |
| **Technical Definition** | The **2021 edition** ranks: **A01 Broken Access Control**, **A02 Cryptographic Failures**, **A03 Injection**, **A04 Insecure Design**, **A05 Security Misconfiguration**, **A06 Vulnerable and Outdated Components**, **A07 Identification and Authentication Failures**, **A08 Software and Data Integrity Failures**, **A09 Security Logging and Monitoring Failures**, and **A10 Server-Side Request Forgery (SSRF)**. Several categories map directly to ASP.NET Core controls covered in this cheat sheet. |
| **Beginner-Friendly Explanation** | The OWASP Top 10 is the **“most wanted list”** for web vulnerabilities. It tells you which classes of bugs are most common and most damaging. Every ASP.NET Core developer should know the list and the framework controls that mitigate each item. |

### Mapping OWASP Top 10 to ASP.NET Core Controls

| OWASP Top 10 (2021) | ASP.NET Core Control | Section in This Cheat Sheet |
|----------------------|----------------------|------------------------------|
| **A01 Broken Access Control** | Fallback policy, `[Authorize]`, resource-based checks | §6 Broken Access Control |
| **A02 Cryptographic Failures** | HTTPS/HSTS, Data Protection, secret vault | §7 Sensitive Data Exposure |
| **A03 Injection** | LINQ parameterization, `FromSqlInterpolated` | §1 SQL Injection |
| **A04 Insecure Design** | Threat modeling, least privilege | §10 Security Best Practices |
| **A05 Security Misconfiguration** | Headers, environment gating, fallback policy | §8 Security Misconfiguration |
| **A06 Vulnerable Components** | `dotnet list package --vulnerable`, NuGet audit | §10 Security Best Practices |
| **A07 Authentication Failures** | ASP.NET Core Identity, secure cookies, lockout | §10 Security Best Practices |
| **A08 Integrity Failures** | Safe deserialization, signed packages | §5 Insecure Deserialization |
| **A09 Logging & Monitoring** | Structured logging, alerting | §10 Security Best Practices |
| **A10 SSRF** | AntiSSRF library, URL allowlists | §4 SSRF |

### References

- OWASP Top 10 (2021) – https://owasp.org/Top10/
- OWASP Top 10 September Update (owasptopten.org) – https://www.owasptopten.org/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html

---

## 10. Security Best Practices (The Baseline Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security best practices are the proven, baseline controls that every production ASP.NET Core application should implement to prevent the vulnerabilities covered in this cheat sheet. |
| **Technical Definition** | The baseline spans **transport** (HTTPS, HSTS, forwarded headers), **headers** (CSP, `X-Content-Type-Options`, `Referrer-Policy`), **authentication** (strongest available flow, short tokens, secure cookies), **authorization** (fallback policy, resource-based checks), **CSRF** (antiforgery on unsafe methods), **secrets** (vault, no source code), **Data Protection** (persisted keys), **rate limiting**, and **dependency hygiene** (`dotnet list package --vulnerable`). |

### Purposes

- **To** **prevent the most common vulnerabilities**—missing authorization, injection, XSS, CSRF, and misconfiguration.
- **To** **fail closed**—the default posture is deny, not allow.
- **To** **defend in depth**—no single control is sufficient.
- **To** **satisfy compliance** requirements (OWASP, PCI DSS, SOC 2).
- **To** **provide a reviewable checklist** for code reviews and pre‑deployment gates.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Transport** | `UseHsts()` + `UseHttpsRedirection()` outside Development; `UseForwardedHeaders()` behind proxies. | Allow HTTP in production; enable HSTS in Development. |
| **Headers** | CSP (nonce-based), `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`. | Rely on defaults—ASP.NET Core does not send CSP by default. |
| **Authorization** | Fallback policy requiring authentication; `[AllowAnonymous]` explicitly. | Leave endpoints unannotated—they become public. |
| **BOLA** | Verify object ownership on every ID-based endpoint. | Assume authentication is enough—it is not. |
| **CSRF** | `UseAntiforgery()`; `[AutoValidateAntiforgeryToken]` globally. | Disable antiforgery globally; opt out only per endpoint when justified. |
| **SQL Injection** | LINQ or `FromSqlInterpolated`; never concatenate user input into SQL. | Use `FromSqlRaw` with string building. |
| **XSS** | Rely on Razor encoding; avoid `Html.Raw` with untrusted input. | Bypass encoding with `Html.Raw`. |
| **Deserialization** | `System.Text.Json` with explicit DTOs; `TypeNameHandling.None`. | Use `BinaryFormatter` or `TypeNameHandling.All`. |
| **SSRF** | Allowlist outbound URLs; block private IPs; use AntiSSRF. | Fetch arbitrary user-supplied URLs. |
| **Secrets** | User Secrets (dev), vault (prod); never source code. | Commit secrets to source control. |
| **Dependencies** | `dotnet list package --vulnerable` in CI; keep packages patched. | Ignore vulnerability warnings. |

### Annotated Code Example (Best-Practice Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Authorization: default deny
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser().Build();
});

// Antiforgery + secure cookies
builder.Services.AddAntiforgery();
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

// Security headers
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    ctx.Response.Headers["Content-Security-Policy"] =
        "default-src 'self'; frame-ancestors 'none'";
    await next();
});

app.UseAuthentication();
app.UseAuthorization();
app.UseAntiforgery();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **Fallback policy** makes authorization default-deny.
- **HTTPS/HSTS** enforce encrypted transport.
- **Security headers** mitigate XSS and clickjacking.
- **Antiforgery** protects state-changing requests.
- **Secure cookies** prevent theft via XSS.
- **Secrets** are loaded from configuration (backed by a vault in production).

### References

- ASP.NET Core Security Checklist for Production – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security And Identity (GitHub) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html

---

## Summary Table: Common Vulnerabilities at a Glance

| Vulnerability | OWASP Category | Primary ASP.NET Core Mitigation | Key Section |
|---------------|----------------|--------------------------------|-------------|
| **SQL Injection** | A03 Injection | LINQ parameterization; `FromSqlInterpolated`; no string concatenation | §1 |
| **XSS** | A03 Injection | Razor automatic encoding; avoid `Html.Raw`; CSP | §2 |
| **CSRF** | A01 Broken Access Control (adjacent) | Antiforgery tokens; `[AutoValidateAntiforgeryToken]` | §3 |
| **SSRF** | A10 SSRF | URL allowlists; AntiSSRF library; block private IPs | §4 |
| **Insecure Deserialization** | A08 Integrity Failures | `System.Text.Json`; `TypeNameHandling.None`; explicit DTOs | §5 |
| **Broken Access Control** | A01 Broken Access Control | Fallback policy; `[Authorize]`; object-level ownership checks | §6 |
| **Sensitive Data Exposure** | A02 Cryptographic Failures | HTTPS/HSTS; Data Protection; secret vault | §7 |
| **Security Misconfiguration** | A05 Security Misconfiguration | Security headers; environment gating; fallback policy | §8 |
| **OWASP Top 10** | All categories | Mapped to controls in §1–§8 | §9 |
| **Security Best Practices** | All categories | Baseline checklist | §10 |

---

## References (Full List)

- SQL Injection Prevention in C# with Entity Framework (safeguard.sh) – https://safeguard.sh/resources/blog/sql-injection-prevention-in-c-with-entity-frameworklinq
- SQL Injection Prevention in EF Core (.NET 2026 Guide) – https://safeguard.sh/resources/blog/sql-injection-prevention-in-ef-core
- Prevent Cross-Site Scripting (XSS) in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- How do you prevent XSS in ASP.NET Core and C# web apps? (safeguard.sh) – https://safeguard.sh/resources/blog/prevent-xss-aspnet-core
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Understanding Anti-Forgery in ASP.NET Core (Duende Software) – https://duendesoftware.com/blog/understanding-anti-forgery-in-aspnet-core
- Microsoft AntiSSRF Library for .NET (GitHub) – https://github.com/microsoft/antissrf
- Microsoft AntiSSRF open-source library (Help Net Security) – https://www.helpnetsecurity.com/2026/06/17/microsoft-antissrf/
- Insecure Deserialization in .NET (2026): BinaryFormatter & JSON (safeguard.sh) – https://safeguard.sh/resources/blog/dotnet-insecure-deserialization
- .NET JSON.NET Deserialization RCE (Invicti) – https://www.invicti.com/blog/web-security/json-net-deserialization-vulnerability/
- Broken access control (Microsoft Learn training) – https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/3-broken-access-control
- ASP.NET Core Security Checklist for Production (safeguard.sh) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- OWASP Top 10 (2021) – https://owasp.org/Top10/
- OWASP Top 10 September Update (owasptopten.org) – https://www.owasptopten.org/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html
- Key encryption at rest in Windows and Azure using ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/implementation/key-encryption-at-rest
- Data Protection & Encryption (dotnet-skills) – https://github.com/wshaddix/dotnet-skills
- Security blueprint (GitHub) – https://github.com/reware-labs/securitycards
- Security And Identity (GitHub) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- Security Headers Every ASP.NET Core App Should Have (C# Corner) – https://www.c-sharpcorner.com/article/security-headers-every-aspnet-core-app-should-have/
- SOTA-skills security rules (System.Text.Json preference) – https://github.com/martinholovsky/SOTA-skills
- OWASP SSRF Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP SQL Injection Prevention Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP Top 10: A01 Broken Access Control – https://owasp.org/Top10/A01_2021-Broken_Access_Control/