# Common Vulnerabilities in C# — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Common vulnerabilities are recurring classes of weaknesses in software systems that attackers exploit to gain unauthorised access, modify or destroy data, disrupt service, or execute arbitrary code. In C# and .NET, these vulnerabilities arise not from the managed runtime itself — which eliminates whole classes of memory-safety bugs — but from how developers use (or misuse) the platform's APIs, configure the application, and trust input.

**Technical Definition**
Vulnerability classes in .NET map to the OWASP Top 10, a regularly updated consensus list of the most critical web application security risks. The **OWASP Top 10:2025** edition reordered and expanded the categories: **A01 Broken Access Control** remains first (and now incorporates SSRF), **A02 Security Misconfiguration** rose from fifth to second, **A03 Software Supply Chain Failures** widened from vulnerable components to encompass build pipelines and distribution, **A04 Cryptographic Failures** fell from second to fourth, **A05 Injection** fell from third to fifth, and **A10 Mishandling of Exceptional Conditions** is a new category. Each category encompasses specific CWEs (Common Weakness Enumerations) and is mitigated by a combination of framework features, secure coding patterns, and configuration hardening. .NET provides first-class defences — parameterised queries in EF Core and ADO.NET, automatic Razor output encoding, antiforgery tokens, the Data Protection API, `System.Text.Json` without type-name resolution, and ASP.NET Core's authorization pipeline — but the developer must use them correctly. Most .NET breaches do not come from CLR zero-days; they come from unaudited dependencies, insecure deserialisation, missing authorization checks, secrets in source control, and debug settings left on in production.

**Beginner-Friendly Explanation**
Common vulnerabilities are “the usual ways applications get hacked.” They include things like letting user input run as SQL (SQL injection), letting user input run as JavaScript in someone else's browser (XSS), tricking a logged-in user's browser into making a request they did not intend (CSRF), making the server fetch an internal URL on the attacker's behalf (SSRF), and deserialising attacker-controlled data into dangerous objects (insecure deserialisation). OWASP Top 10 is the industry's scoreboard of these risks. .NET gives you strong tools to prevent each one, but the defaults are tuned for developer convenience, not adversarial production traffic — so you have to actively harden them.

---

### Key Characteristics

- **Input-driven** — Most vulnerabilities begin with unvalidated, untrusted input reaching a dangerous sink (SQL, HTML, a URL, a deserialiser, a file path).
- **Configuration-dependent** — Many vulnerabilities (security misconfiguration, broken access control, sensitive data exposure) are caused by what is *not* configured, not by buggy code.
- **Framework-mitigable** — .NET provides strong primitives; the gap is almost always in how they are used.
- **Supply-chain-sensitive** — The NuGet dependency tree is part of the attack surface; vulnerable transitive packages are a leading cause of .NET breaches.
- **OWASP-classified** — The OWASP Top 10 provides a shared vocabulary and prioritisation framework for these risks.
- **Defence-in-depth required** — No single control is sufficient; validation, encoding, parameterisation, authorization, and monitoring must overlap.

---

### Prerequisites

- Working knowledge of C#, ASP.NET Core, and Entity Framework Core.
- Familiarity with HTTP, HTML, JavaScript, and SQL.
- Understanding of authentication, authorization, and sessions.
- Awareness of the OWASP Top 10 and its purpose.
- Basic knowledge of dependency management (NuGet, lock files).

---

### Related Programming Areas

- **Security fundamentals** — Threat modeling, attack surface, defense in depth.
- **Authentication and authorization security** — Identity and access control.
- **Input validation** — Boundary validation and encoding.
- **Data access** — Parameterised queries and ORM safety.
- **Cryptography** — Data Protection API, hashing, and encryption.
- **Supply chain security** — NuGet audit, SBOMs, and package provenance.

---

### Core Concepts / Features

1. SQL Injection
2. Cross-Site Scripting (XSS)
3. Cross-Site Request Forgery (CSRF)
4. Server-Side Request Forgery (SSRF)
5. Insecure Deserialization
6. Broken Access Control
7. Sensitive Data Exposure (Cryptographic Failures)
8. Security Misconfiguration
9. OWASP Top 10 (2025)
10. Security Best Practices

Each concept follows the same uniform structure: definitions (core, technical, beginner-friendly), purposes (each beginning with “To”), syntax rules and structure, annotated code examples with expected outputs, real-world cases, and references.

---

## 1. SQL Injection

### Definitions

**Core Definition**
SQL injection is a vulnerability in which attacker-controlled input is interpreted as SQL code by the database, allowing the attacker to read, modify, or delete data, bypass authentication, or execute administrative operations.

**Technical Definition**
SQL injection occurs when a program integrates a user-supplied string value into a SQL query such that the value can terminate the intended string and perform another SQL operation. In .NET, the risk exists wherever raw SQL is constructed with string concatenation or unsafe interpolation: `FromSqlRaw`, `ExecuteSqlRaw`, `ADO.NET` commands built with `+`, and dynamic `ORDER BY` or column-name clauses. **LINQ queries translated by EF Core are parameterised by default** — values become `DbParameter` objects, not part of the SQL text. `FromSql` and `FromSqlInterpolated` are safe because they automatically convert interpolated values to database parameters. The danger lies in the raw-SQL escape hatches and in dynamic identifiers (table names, column names) that cannot be parameterised and require manual allow-listing.

**Beginner-Friendly Explanation**
SQL injection is “letting user input become SQL code.” If your code does `"SELECT * FROM Users WHERE Name = '" + input + "'"` and the attacker types `' OR '1'='1`, the query returns every user. The fix is to use parameterised queries: the SQL text and the data travel separately to the database, so the data can never be interpreted as code. In EF Core, normal LINQ queries are safe by default; the risk is when you write raw SQL and concatenate strings.

---

### Purposes

- To prevent unauthorised data disclosure, modification, and deletion.
- To prevent authentication bypass through crafted login payloads.
- To protect against administrative operations executed through injection.
- To comply with OWASP A05:2025 and PCI-DSS requirements.
- To eliminate the most classic and still-prevalent web vulnerability.

---

### Syntax Rules and Structure

#### Safe vs. Unsafe Patterns

| Pattern | Safety | Reason |
|---|---|---|
| **EF Core LINQ** (`context.Users.Where(u => u.Name == name)`) | Safe | Translated to parameterised SQL |
| **`FromSql` / `FromSqlInterpolated`** | Safe | Interpolated values become `DbParameter` |
| **`FromSqlRaw` with concatenation** | **Unsafe** | User input becomes part of the SQL text |
| **`ExecuteSqlRaw` with concatenation** | **Unsafe** | Same risk for non-query commands |
| **ADO.NET with `SqlParameter`** | Safe | Explicit parameterisation |
| **Dynamic `ORDER BY` / column names** | **Unsafe without allow-list** | Identifiers cannot be parameterised |

#### General Syntax (EF Core — Safe)

```csharp
// LINQ: safe by default
var user = await context.Users
    .FirstOrDefaultAsync(u => u.Name == userInput);

// FromSqlInterpolated: safe
var users = await context.Users
    .FromSql($"SELECT * FROM Users WHERE Name = {userInput}")
    .ToListAsync();

// ADO.NET: safe with parameters
using var cmd = new SqlCommand("SELECT * FROM Users WHERE Name = @Name", connection);
cmd.Parameters.AddWithValue("@Name", userInput);
```

#### General Syntax (Unsafe — Do Not Use)

```csharp
// ❌ SQL injection
var sql = $"SELECT * FROM Users WHERE Name = '{userInput}'";
context.Users.FromSqlRaw(sql);

// ✅ Parameterised
context.Users.FromSqlInterpolated($"SELECT * FROM Users WHERE Name = {userInput}");
```

#### Syntax Rules

- Use LINQ queries wherever possible; EF Core parameterises them by default.
- When raw SQL is necessary, use `FromSqlInterpolated` or `FromSql` with interpolated strings; never concatenate user input.
- When raw SQL is necessary and you use `FromSqlRaw`, construct `DbParameter` objects explicitly.
- For dynamic identifiers (column names, sort directions), validate against an allow-list before building the SQL string.
- Enable parameterisation at the ADO.NET level when not using EF Core.

#### Constraints and Limitations

- Not all queries can be expressed in LINQ; raw SQL is sometimes necessary, but must be parameterised.
- Dynamic identifiers cannot be parameterised; allow-listing is the only safe mitigation.
- Stored procedures are safe only if they themselves do not build dynamic SQL from parameters.
- Second-order injection occurs when data stored safely is later concatenated into a query elsewhere; audit all query construction.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Vulnerable vs. Safe Login Query

```csharp
// ❌ VULNERABLE
public async Task<User?> LoginUnsafe(string username, string password)
{
    var sql = $"SELECT * FROM Users WHERE Username = '{username}' AND Password = '{password}'";
    return await _context.Users.FromSqlRaw(sql).FirstOrDefaultAsync();
}
// Input: username = "admin' --", password = "anything"
// SQL: SELECT * FROM Users WHERE Username = 'admin' --' AND Password = 'anything'
// The "--" comments out the password check; login succeeds without a password.

// ✅ SAFE
public async Task<User?> LoginSafe(string username, string password)
{
    return await _context.Users
        .FromSqlInterpolated($"SELECT * FROM Users WHERE Username = {username} AND Password = {password}")
        .FirstOrDefaultAsync();
}
```

**Expected Output (conceptual)**

```
Unsafe: attacker logs in as admin with any password.
Safe: the crafted input is treated as literal data; login fails.
```

**Why this output occurs**

- In the unsafe version, the `--` sequence terminates the SQL statement early, nullifying the password check.
- In the safe version, the entire input is bound as a parameter and never parsed as SQL.

---

#### Example 2: Dynamic Sort Column with Allow-List

```csharp
// ❌ VULNERABLE: user-controlled column name
var column = Request.Query["sort"];
var users = _context.Users.FromSqlRaw($"SELECT * FROM Users ORDER BY {column}").ToList();

// ✅ SAFE: allow-list
private static readonly HashSet<string> AllowedSortColumns = new(StringComparer.OrdinalIgnoreCase)
{
    "Name", "Email", "CreatedAt"
};

public async Task<List<User>> GetSortedUsers(string sortColumn)
{
    if (!AllowedSortColumns.Contains(sortColumn))
        sortColumn = "Name";

    // Use a switch expression to map to a known-safe column reference
    var query = sortColumn.ToLowerInvariant() switch
    {
        "email" => _context.Users.OrderBy(u => u.Email),
        "createdat" => _context.Users.OrderBy(u => u.CreatedAt),
        _ => _context.Users.OrderBy(u => u.Name)
    };
    return await query.ToListAsync();
}
```

**Expected Output (conceptual)**

```
Unsafe: attacker supplies "Name; DROP TABLE Users --" and the table is deleted.
Safe: the invalid column falls back to a known-safe default; injection is impossible.
```

**Why this output occurs**

- Column identifiers cannot be parameterised; the only safe approach is to map the user's choice to a known-safe expression.

---

### Real-World Cases

- **Login forms** — Classic `' OR '1'='1` bypass.
- **Search endpoints** — Concatenated filter values that expose the entire table.
- **Reporting** — User-controlled sort or filter expressions that execute arbitrary SQL.
- **Legacy ADO.NET code** — Pre-EF codebases with string-built commands.

---

### References

- SQL Queries — EF Core (parameterisation guidance) - https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- SQL Injection Prevention in EF Core (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/ef-core-sql-injection-prevention
- Preventing SQL Injection with Entity Framework — safeguard.sh - https://safeguard.sh/resources/blog/preventing-sql-injection-with-entity-framework
- SQL Injection Prevention in C# with Entity Framework — safeguard.sh - https://safeguard.sh/resources/blog/sql-injection-prevention-csharp-entity-framework

---

## 2. Cross-Site Scripting (XSS)

### Definitions

**Core Definition**
Cross-site scripting is a vulnerability that allows an attacker to inject client-side scripts (usually JavaScript) into web pages viewed by other users.

**Technical Definition**
XSS occurs when an application includes untrusted data in a web page without proper validation or encoding. There are three types: **stored XSS** (malicious script persisted in a database and served to every visitor), **reflected XSS** (malicious script reflected off the server via a crafted link or form submission), and **DOM-based XSS** (client-side JavaScript writes untrusted data into the DOM). In ASP.NET Core, **Razor encodes variable output by default**, so most XSS vulnerabilities arise from explicitly bypassing encoding — using `Html.Raw`, `HtmlString`, or returning unescaped markup from an API — or from placing untrusted data inside JavaScript contexts where HTML encoding is insufficient. The defence is **context-aware output encoding**: HTML encoding for HTML content, HTML attribute encoding for attributes, JavaScript encoding for script blocks, and URL encoding for query strings. A **Content Security Policy** (CSP) provides an additional layer that restricts which scripts can execute.

**Beginner-Friendly Explanation**
XSS is “letting user input run as JavaScript in someone else's browser.” If a comment field accepts `<script>alert('hacked')</script>` and the site displays it without encoding, every visitor runs that script. The script can steal cookies, redirect users, or change the page. Razor automatically encodes output, so `<` becomes `&lt;` and the browser displays it as text instead of executing it. XSS happens when developers disable that encoding — for example, by using `Html.Raw` on user-supplied content.

---

### Purposes

- To prevent session hijacking through cookie theft.
- To prevent defacement and content manipulation.
- To protect users from phishing redirects and credential theft.
- To prevent DOM-based attacks through client-side script injection.
- To satisfy OWASP A05:2025 and PCI-DSS requirements.
- To maintain user trust in the application's content integrity.

---

### Syntax Rules and Structure

#### Context-Aware Encoding

| Context | Encoding | .NET API |
|---|---|---|
| **HTML content** | HTML encoding | Razor `@variable` (automatic), `HtmlEncoder.Default.Encode()` |
| **HTML attributes** | Attribute encoding | Razor (automatic) |
| **JavaScript** | JavaScript encoding | `JavaScriptEncoder.Default.Encode()`, `JsonSerializer` |
| **URL query strings** | URL encoding | `Uri.EscapeDataString()`, `UrlEncoder.Default.Encode()` |
| **CSS** | CSS encoding | Avoid dynamic CSS; use classes |

#### General Syntax (Safe Razor)

```razor
@* Razor automatically HTML-encodes this *@
<p>@Model.UserComment</p>

@* Explicit encoding *@
@HtmlEncoder.Default.Encode(Model.UserComment)
```

#### General Syntax (Unsafe Razor — Do Not Use)

```razor
@* ❌ Bypasses encoding; XSS risk *@
<p>@Html.Raw(Model.UserComment)</p>

@* ❌ Untrusted data inside a script block *@
<script>
    var comment = '@Model.UserComment';
</script>
```

#### General Syntax (Safe JavaScript Context)

```csharp
// ✅ Serialize to JSON; the serializer encodes for JavaScript
<script>
    var comment = @Json.Serialize(Model.UserComment);
</script>
```

#### Content Security Policy

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Append("Content-Security-Policy",
        "default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';");
    await next();
});
```

**Component breakdown**

- `default-src 'self'` — Only load resources from the same origin.
- `script-src 'self'` — Only execute scripts from the same origin; no inline scripts.
- `object-src 'none'` — Block plugin content.
- `base-uri 'self'` — Prevent base-tag hijacking.

#### Syntax Rules

- Let Razor encode output by default; never use `Html.Raw` on untrusted data.
- Use `Json.Serialize` (or `JsonSerializer`) for data embedded in script blocks.
- Set a restrictive CSP that disables inline scripts and `eval`.
- Validate input at the boundary as an additional layer, but never rely on validation alone — encoding is the primary defence.
- Use `HttpOnly` cookies to reduce the impact of XSS-based cookie theft.

#### Constraints and Limitations

- HTML encoding does not protect JavaScript contexts, CSS contexts, or URL contexts; each requires its own encoding.
- CSP can break legitimate functionality if not tested; start in report-only mode.
- Client-side frameworks (React, Angular) escape by default, but `dangerouslySetInnerHTML` and equivalents reintroduce the risk.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Razor Auto-Encoding vs. `Html.Raw`

```razor
@* Model.Comment = "<script>alert('xss')</script>" *@

@* ✅ Razor auto-encodes *@
<p>@Model.Comment</p>
@* Renders: <p>&lt;script&gt;alert('xss')&lt;/script&gt;</p> *@

@* ❌ Html.Raw bypasses encoding *@
<p>@Html.Raw(Model.Comment)</p>
@* Renders: <p><script>alert('xss')</script></p> — script executes *@
```

**Expected Output (conceptual)**

```
Auto-encoded: the browser displays the literal text <script>alert('xss')</script>.
Html.Raw: the browser executes the script.
```

**Why this output occurs**

- Razor HTML-encodes `<` to `&lt;`, so the browser treats the content as text.
- `Html.Raw` emits the string as-is, so the browser parses it as markup and executes the script.

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

- `Json.Serialize` produces a JavaScript-safe string literal.
- Direct interpolation places untrusted data inside a JavaScript string without escaping, allowing breakout.

---

### Real-World Cases

- **Comment sections** — Stored XSS through unsanitised comments.
- **Search results pages** — Reflected XSS via the search query string.
- **User profiles** — XSS through display names and bios.
- **API responses rendered as HTML** — Markup returned by an API and injected into a page without encoding.

---

### References

- Prevent Cross-Site Scripting (XSS) in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- CSRF and XSS Prevention in ASP.NET Core (2026) — safeguard.sh - https://safeguard.sh/resources/blog/csrf-xss-prevention-aspnet-core
- How do you prevent XSS in ASP.NET Core and C# web apps? — safeguard.sh - https://safeguard.sh/resources/blog/prevent-xss-aspnet-core
- Security cards — XSS encoding - https://github.com/…/security-cards

---

## 3. Cross-Site Request Forgery (CSRF)

### Definitions

**Core Definition**
Cross-site request forgery is a vulnerability in which an attacker tricks a logged-in user's browser into sending an unintended state-changing request to a vulnerable application.

**Technical Definition**
CSRF exploits the browser's automatic inclusion of credentials (cookies, HTTP authentication, client certificates, IP address, and Windows domain credentials) with every request to a domain. If the user is authenticated to `bank.com`, a malicious page at `evil.com` can cause the browser to submit a POST request to `bank.com/transfer` with the user's cookies attached. ASP.NET Core's primary defence is **antiforgery tokens**: a token is embedded in the page and validated on the server for unsafe HTTP methods (POST, PUT, DELETE, PATCH). The `[ValidateAntiForgeryToken]` attribute enforces validation on specific actions; `[AutoValidateAntiforgeryToken]` applies it globally to unsafe methods, which is the recommended approach. Additional layers include `SameSite` cookies (default `Lax` in modern browsers), custom request headers, and origin/referer validation.

**Beginner-Friendly Explanation**
CSRF is “tricking a logged-in user's browser into making a request they did not intend.” If you are logged into your bank and then visit a malicious site, that site can secretly submit a form to your bank using your existing session. The bank sees a valid cookie and processes the request. The defence is an antiforgery token: the bank's page includes a secret token, and the bank rejects any state-changing request that does not include it. The malicious site cannot read the token because of the same-origin policy.

---

### Purposes

- To prevent unauthorised state-changing operations on behalf of authenticated users.
- To protect password changes, email changes, fund transfers, and account deletions.
- To satisfy OWASP A01:2025 (broken access control) requirements.
- To complement `SameSite` cookies and CSP.
- To close a vulnerability that is invisible to input validation.

---

### Syntax Rules and Structure

#### General Syntax (Razor Form with Antiforgery)

```razor
<form asp-action="Transfer" method="post">
    @Html.AntiforgeryToken()
    <input name="amount" />
    <button type="submit">Transfer</button>
</form>
```

Razor form tag helpers automatically inject the antiforgery token when the form uses `asp-*` attributes.

#### General Syntax (Controller Validation)

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Transfer(TransferRequest request) { ... }
```

#### Global Validation (Recommended)

```csharp
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

#### Secure Cookie Configuration

```csharp
services.AddAntiforgery(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.HeaderName = "X-CSRF-TOKEN"; // for AJAX requests
});
```

#### Syntax Rules

- Apply `[AutoValidateAntiforgeryToken]` globally; it validates unsafe methods (POST, PUT, DELETE, PATCH) and ignores safe methods (GET, HEAD, OPTIONS, TRACE).
- Include the antiforgery token in every state-changing form; Razor tag helpers do this automatically.
- For AJAX requests, send the token in a custom header (`X-CSRF-TOKEN`) and configure `HeaderName`.
- Set `Cookie.SecurePolicy = Always` outside development.
- Do not rely solely on `SameSite` cookies; use antiforgery tokens as the primary defence.

#### Constraints and Limitations

- Antiforgery tokens protect against CSRF but not against XSS; an XSS vulnerability can read the token.
- APIs consumed by non-browser clients (mobile apps, third-party services) typically use token-based authentication (JWT) and are not vulnerable to classic CSRF, but must still enforce authorization.
- `[ValidateAntiForgeryToken]` on GET requests is unnecessary and can cause issues.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Protected vs. Unprotected POST Action

```csharp
// ❌ VULNERABLE: no antiforgery validation
[HttpPost]
public IActionResult DeleteAccount() { ... }

// ✅ SAFE: antiforgery validation
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult DeleteAccount() { ... }
```

**Expected Output (conceptual)**

```
Unprotected: a cross-site form submission succeeds; the account is deleted.
Protected: the request is rejected with 400 Bad Request because the token is missing or invalid.
```

**Why this output occurs**

- Without validation, the server cannot distinguish a legitimate form submission from a forged one.
- The antiforgery token proves the request originated from a page served by the application.

---

#### Example 2: AJAX Request with Header Token

```javascript
// Client: read the token from a meta tag and send it as a header
const token = document.querySelector('meta[name="csrf-token"]').content;
fetch('/api/transfer', {
    method: 'POST',
    headers: { 'X-CSRF-TOKEN': token, 'Content-Type': 'application/json' },
    body: JSON.stringify({ amount: 100 })
});
```

```csharp
// Server: configure the antiforgery header name
services.AddAntiforgery(options => options.HeaderName = "X-CSRF-TOKEN");
```

**Expected Output (conceptual)**

```
Request with valid header token: accepted.
Request without the header: 400 Bad Request.
```

**Why this output occurs**

- The token is validated from the custom header rather than the form body, supporting AJAX scenarios.

---

### Real-World Cases

- **Banking transfers** — CSRF could initiate unauthorised transfers.
- **Account settings** — Password and email changes must be protected.
- **Administrative actions** — User deletion, role changes, and configuration updates.
- **E-commerce** — Adding items to a cart or placing orders.

---

### References

- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- CSRF and XSS Prevention in ASP.NET Core (2026) — safeguard.sh - https://safeguard.sh/resources/blog/csrf-xss-prevention-aspnet-core
- ASP.NET Core Security Checklist (2026 Production Guide) — safeguard.sh - https://safeguard.sh/resources/blog/aspnet-core-security-checklist

---

## 4. Server-Side Request Forgery (SSRF)

### Definitions

**Core Definition**
Server-side request forgery is a vulnerability in which an attacker can coerce the server into making an HTTP (or other protocol) request to an arbitrary destination chosen by the attacker.

**Technical Definition**
SSRF occurs when an application fetches a remote resource using a user-supplied URL without validating the destination. The attacker can point the server at internal services (metadata endpoints, databases, admin panels), localhost, or private IP ranges, bypassing network perimeter controls. In .NET, the risk exists wherever `HttpClient`, `WebRequest`, or similar APIs are called with user-controlled URLs. The `HttpClient` itself does not validate destinations. Mitigation requires **allow-listing approved schemes, hosts, and IP ranges**, resolving DNS and validating the resolved IP against private-range blocklists, re-validating after redirects, and using a library or handler designed for SSRF defence. Microsoft provides the **AntiSSRF** library, which integrates with `HttpClient` via `AntiSSRFHandler` and validates URLs and resolved IPs against configurable policies.

**Beginner-Friendly Explanation**
SSRF is “making the server fetch a URL the attacker chooses.” If your application has a feature like “import from URL” and you pass the user's URL directly to `HttpClient`, the attacker can supply `http://169.254.169.254/` (the cloud metadata endpoint) and the server will fetch internal credentials. The fix is to allow-list approved domains and schemes, and to block private IP ranges.

---

### Purposes

- To prevent access to internal services and cloud metadata endpoints.
- To prevent port scanning and service discovery from the server's network position.
- To prevent circumvention of network perimeter controls.
- To satisfy OWASP A01:2025 (SSRF is now part of broken access control).
- To protect cloud credentials and internal APIs.

---

### Syntax Rules and Structure

#### General Syntax (Unsafe)

```csharp
// ❌ VULNERABLE: user-controlled URL
var url = Request.Query["url"];
var client = new HttpClient();
var response = await client.GetAsync(url);
```

#### General Syntax (Allow-List Validation)

```csharp
private static readonly HashSet<string> AllowedHosts = new(StringComparer.OrdinalIgnoreCase)
{
    "api.example.com", "cdn.example.com"
};

public async Task<HttpResponseMessage> SafeFetchAsync(string url)
{
    if (!Uri.TryCreate(url, UriKind.Absolute, out var uri))
        throw new ArgumentException("Invalid URL");

    if (uri.Scheme != Uri.UriSchemeHttps)
        throw new ArgumentException("Only HTTPS is allowed");

    if (!AllowedHosts.Contains(uri.Host))
        throw new ArgumentException("Host not allowed");

    var ip = await Dns.GetHostAddressesAsync(uri.Host);
    if (ip.Any(IsPrivateAddress))
        throw new ArgumentException("Private address not allowed");

    return await _client.GetAsync(uri);
}

private static bool IsPrivateAddress(IPAddress ip)
    => IPAddress.IsLoopback(ip)
        || ip.ToString().StartsWith("10.")
        || ip.ToString().StartsWith("192.168.")
        || ip.ToString().StartsWith("172.16.");
```

#### Microsoft AntiSSRF Library

```csharp
builder.Services.AddHttpClient("safe")
    .AddHttpMessageHandler(() => new AntiSSRFHandler(new AntiSSRFPolicy
    {
        AllowedSchemes = { "https" },
        AllowedHosts = { "api.example.com" }
    }));
```

**Component breakdown**

- `AllowedSchemes` — Restrict to `https`; block `file`, `ldap`, `gopher`, etc.
- `AllowedHosts` — Allow-list approved domains.
- `IP validation` — Block loopback, private ranges, link-local, and metadata IPs.
- `Re-validation` — Check again after redirects.

#### Syntax Rules

- Never pass user-controlled URLs directly to `HttpClient`.
- Allow-list schemes and hosts; deny everything else.
- Resolve DNS and validate the resolved IP against private-range blocklists.
- Re-validate after HTTP redirects; an allow-listed host can redirect to an internal address.
- Use the Microsoft AntiSSRF library for defence in depth.

#### Constraints and Limitations

- DNS rebinding can change the resolved IP after validation; re-validate at connection time.
- Allow-listing can break legitimate use cases; design the feature around a fixed set of trusted destinations.
- SSRF can occur through non-HTTP protocols (FTP, LDAP, file); validate the scheme strictly.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Blocking the Cloud Metadata Endpoint

```csharp
// ❌ Vulnerable: attacker supplies http://169.254.169.254/latest/meta-data/
var response = await client.GetAsync(userUrl);

// ✅ Safe: block private and link-local addresses
if (IPAddress.TryParse(uri.Host, out var ip) && IsPrivateAddress(ip))
    throw new SecurityException("SSRF attempt blocked");
```

**Expected Output (conceptual)**

```
Vulnerable: the server returns cloud instance credentials.
Safe: the request is rejected with a security exception.
```

**Why this output occurs**

- The metadata endpoint is a link-local address; blocking private ranges prevents access.

---

#### Example 2: Allow-List with DNS Resolution

```csharp
var allowed = new[] { "api.trusted.com" };
if (!allowed.Contains(uri.Host, StringComparer.OrdinalIgnoreCase))
    throw new SecurityException("Host not allowed");

var addresses = await Dns.GetHostAddressesAsync(uri.Host);
if (addresses.Any(a => IPAddress.IsLoopback(a) || IsPrivate(a)))
    throw new SecurityException("Resolved to private address");
```

**Expected Output (conceptual)**

```
Attacker supplies http://api.trusted.com that resolves to 127.0.0.1: blocked.
Legitimate request to the real api.trusted.com public IP: allowed.
```

**Why this output occurs**

- DNS resolution is checked against private ranges, preventing an allow-listed hostname from pointing to localhost.

---

### Real-World Cases

- **URL import features** — “Import from URL” or “Fetch image from URL.”
- **Webhook testing** — Sending test payloads to a user-supplied URL.
- **PDF/image generation** — Fetching remote resources to embed in a document.
- **Cloud metadata** — The 169.254.169.254 endpoint is a prime SSRF target for credential theft.

---

### References

- Server-side request forgery — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/12-server-side-request-forgery
- Microsoft AntiSSRF Library for .NET — GitHub - https://github.com/microsoft/AntiSSRF
- SSRF Prevention in .NET — AntiSSRF library documentation - https://microsoft.github.io/AntiSSRF/

---

## 5. Insecure Deserialization

### Definitions

**Core Definition**
Insecure deserialization is a vulnerability in which attacker-controlled serialized data is deserialised into objects, allowing the attacker to control which types are instantiated and trigger dangerous behaviour.

**Technical Definition**
In .NET, insecure deserialization occurs when a deserialiser reconstructs arbitrary types from untrusted data. The classic vector is `BinaryFormatter`, which Microsoft has removed from .NET 9 because it cannot be made secure. The same risk exists in `Newtonsoft.Json` when `TypeNameHandling` is set to `All`, `Auto`, or `Objects`: the JSON can contain a `$type` field that names a CLR type to instantiate, enabling “gadget chain” attacks that escalate to remote code execution (RCE). `System.Text.Json` is safe by default because it binds to a concrete type specified by the caller and does not resolve arbitrary type names; polymorphism requires explicit `[JsonDerivedType]` attributes. The fix is to avoid type-aware deserialisation entirely, use `System.Text.Json` with concrete types, and if polymorphism is genuinely required, restrict it to an explicit allow-list of known-safe types.

**Beginner-Friendly Explanation**
Insecure deserialization is “letting attacker-controlled data choose which code to run.” If your app deserialises data and the data itself says “instantiate this type,” an attacker can name a dangerous type that executes commands when constructed. The fix is to use a deserialiser that does not resolve type names from the data — like `System.Text.Json` — and to bind to a specific, known type.

---

### Purposes

- To prevent remote code execution through crafted serialized payloads.
- To prevent file writes, SSRF, and other side effects triggered by gadget chains.
- To eliminate the use of `BinaryFormatter` and unsafe JSON settings.
- To satisfy OWASP A08:2025 (software and data integrity failures).
- To protect APIs, message queues, and cache deserialisation paths.

---

### Syntax Rules and Structure

#### Dangerous vs. Safe Deserialisers

| Deserialiser | Status | Reason |
|---|---|---|
| **`BinaryFormatter`** | Removed in .NET 9 | Inherently unsafe; RCE risk |
| **`SoapFormatter` / `NetDataContractSerializer`** | Removed | Same lineage as `BinaryFormatter` |
| **`Newtonsoft.Json` with `TypeNameHandling.All/Auto/Objects`** | **Unsafe** | Resolves attacker-supplied `$type` |
| **`Newtonsoft.Json` with `TypeNameHandling.None`** | Safe | Default; no type resolution |
| **`System.Text.Json`** | Safe | Binds to the caller-specified type |
| **`XmlSerializer` / `DataContractSerializer`** | Safe | No arbitrary type resolution |

#### General Syntax (Safe — System.Text.Json)

```csharp
var order = JsonSerializer.Deserialize<Order>(json);
```

#### General Syntax (Polymorphism with Allow-List)

```csharp
[JsonDerivedType(typeof(CreditCardPayment), "credit")]
[JsonDerivedType(typeof(BankTransferPayment), "bank")]
public abstract class Payment { ... }

var payment = JsonSerializer.Deserialize<Payment>(json);
```

#### General Syntax (Unsafe — Do Not Use)

```csharp
// ❌ BinaryFormatter (removed in .NET 9)
var formatter = new BinaryFormatter();
var obj = formatter.Deserialize(stream);

// ❌ Newtonsoft.Json with TypeNameHandling
var settings = new JsonSerializerSettings { TypeNameHandling = TypeNameHandling.All };
var obj = JsonConvert.DeserializeObject<BaseType>(json, settings);
```

#### Syntax Rules

- Never use `BinaryFormatter`; it is removed in .NET 9 and cannot be made secure.
- Use `System.Text.Json` with concrete types; avoid type-name resolution.
- If polymorphism is required, use `[JsonDerivedType]` with an explicit discriminator, not attacker-supplied type names.
- If `Newtonsoft.Json` is unavoidable, keep `TypeNameHandling` at `None` or use a custom `SerializationBinder` with an allow-list.
- Validate deserialised data after parsing and limit input size.

#### Constraints and Limitations

- `System.Text.Json` polymorphism requires explicit configuration; it does not support arbitrary type discovery.
- Legacy compatibility packages for `BinaryFormatter` exist but are unsupported and unsafe.
- Third-party libraries may use unsafe deserialisation internally; audit dependencies.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: BinaryFormatter RCE

```csharp
// ❌ VULNERABLE
var bytes = Convert.FromBase64String(userInput);
var formatter = new BinaryFormatter();
var obj = formatter.Deserialize(new MemoryStream(bytes));
```

**Expected Output (conceptual)**

```
.NET 8: SYSLIB0011 warning; deserialization succeeds and may execute attacker-chosen gadget code.
.NET 9: NotSupportedException; BinaryFormatter is removed.
```

**Why this output occurs**

- `BinaryFormatter` reconstructs arbitrary types from the stream; a crafted payload can trigger command execution through gadget chains.

---

#### Example 2: Safe Polymorphism with System.Text.Json

```csharp
[JsonPolymorphic(TypeDiscriminatorPropertyName = "$type")]
[JsonDerivedType(typeof(CreditCardPayment), "credit")]
[JsonDerivedType(typeof(BankTransferPayment), "bank")]
public abstract class Payment { ... }

var payment = JsonSerializer.Deserialize<Payment>(json);
```

**Expected Output (conceptual)**

```
Only CreditCardPayment and BankTransferPayment can be instantiated.
An attacker-supplied "$type" of "System.Diagnostics.Process" is rejected.
```

**Why this output occurs**

- The allow-list of derived types is explicit; the deserialiser does not resolve arbitrary type names.

---

### Real-World Cases

- **ViewState deserialisation** — Legacy ASP.NET view-state formatters.
- **Message queue consumers** — Deserialising messages from untrusted producers.
- **Cache deserialisation** — Deserialising cached objects that may have been tampered with.
- **API payloads** — Accepting `object` or polymorphic base types without allow-listing.

---

### References

- Insecure Deserialization in .NET (2026): BinaryFormatter & JSON — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-insecure-deserialization
- How do you deserialize data safely in C# and .NET? — safeguard.sh - https://safeguard.sh/resources/blog/safe-deserialization-csharp-dotnet
- Do not deserialize untrusted data (.NET binary formatters) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-security-guide
- Serialization and Networking Breaking Changes (.NET 9) — GitHub - https://github.com/dotnet/docs/blob/main/docs/core/compatibility/serialization/9.0/binaryformatter-removed.md

---

## 6. Broken Access Control

### Definitions

**Core Definition**
Broken access control is a vulnerability in which users can act outside their intended permissions — accessing data, performing operations, or reaching endpoints they should not be allowed to reach.

**Technical Definition**
Broken access control is the **#1 risk in the OWASP Top 10:2025** (A01). It encompasses horizontal privilege escalation (accessing another user's resources by changing an identifier), vertical privilege escalation (accessing admin functionality as a regular user), missing authorization on endpoints, insecure direct object references (IDOR), CORS misconfiguration, and forced browsing. In ASP.NET Core, the framework provides `[Authorize]`, policy-based authorization, claims-based authorization, and resource-based authorization, but none of them apply automatically unless configured. A controller or Minimal API endpoint without authorization metadata treats every request the same way, applying no security checks. The **fallback policy** can require authentication for every endpoint that does not explicitly opt out, enabling secure-by-default behaviour. SSRF has been folded into A01 in the 2025 edition because both involve bypassing access controls to reach unintended resources.

**Beginner-Friendly Explanation**
Broken access control is “users being able to do things they should not be allowed to do.” If an API endpoint returns any user's data when given an ID, and you do not check that the caller owns that data, an attacker can simply change the ID and read someone else's information. The fix is to require authorization on every endpoint, check ownership for resource-specific operations, and deny by default.

---

### Purposes

- To prevent unauthorised data disclosure and modification.
- To prevent privilege escalation (horizontal and vertical).
- To enforce least privilege at every endpoint.
- To satisfy OWASP A01:2025, the most critical web application risk.
- To close IDOR and forced-browsing vulnerabilities.

---

### Syntax Rules and Structure

#### General Syntax (Secure by Default)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Explicit opt-out
[AllowAnonymous]
public IActionResult Public() => Ok();
```

#### General Syntax (Resource-Based Authorization)

```csharp
var document = await _repository.GetByIdAsync(id);
var result = await _authorizationService.AuthorizeAsync(User, document, "DocumentOwnerPolicy");
if (!result.Succeeded) return Forbid();
```

#### General Syntax (Endpoint Authorization — Minimal API)

```csharp
app.MapGet("/orders/{id}", [Authorize] (int id) => ...);
app.MapGet("/admin", [Authorize(Roles = "Admin")] () => ...);
```

#### Syntax Rules

- Apply `[Authorize]` to all externally facing endpoints; use `[AllowAnonymous]` only where explicitly required.
- Use a fallback policy to require authentication by default.
- For resource-specific access, load the resource and check ownership/tenant with `IAuthorizationService`.
- Deny by default in authorization handlers; call `context.Succeed` only when the condition is demonstrably met.
- Use UUIDs instead of sequential IDs to make enumeration harder (defence in depth, not a substitute for authorization).
- Log and alert on 403 responses; spikes may indicate enumeration or escalation attempts.

#### Constraints and Limitations

- `[Authorize]` alone does not check resource ownership; IDOR requires explicit resource-based checks.
- Client-side UI hiding (e.g., hiding admin buttons) is not authorization; the server must enforce.
- CORS misconfiguration can expose APIs to unauthorised origins; validate allowed origins.
- Path traversal (`/api/users/../admin`) can bypass route-based authorization if paths are not canonicalised.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: IDOR — Vulnerable vs. Safe

```csharp
// ❌ VULNERABLE: no ownership check
[HttpGet("/api/documents/{id}")]
public async Task<IActionResult> GetDocument(int id)
{
    var doc = await _repository.GetByIdAsync(id);
    return Ok(doc); // any authenticated user can read any document
}

// ✅ SAFE: ownership check
[HttpGet("/api/documents/{id}")]
[Authorize]
public async Task<IActionResult> GetDocument(int id)
{
    var doc = await _repository.GetByIdAsync(id);
    if (doc.OwnerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
        return Forbid();
    return Ok(doc);
}
```

**Expected Output (conceptual)**

```
Vulnerable: User A changes the ID to User B's document and reads it.
Safe: User A receives 403 Forbidden for User B's document.
```

**Why this output occurs**

- The safe version compares the document's owner to the authenticated user's identifier.
- Without this check, the endpoint only verifies that the caller is authenticated, not that they own the resource.

---

#### Example 2: Fallback Policy with Explicit Anonymous Exception

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

- The fallback policy requires authentication for all endpoints without explicit metadata.
- `[AllowAnonymous]` opts the health endpoint out.

---

### Real-World Cases

- **User profile APIs** — Changing `/users/123` to `/users/124` to read another user's data.
- **Admin panels** — Forced browsing to `/admin` by a regular user.
- **Document management** — Accessing another tenant's documents by ID.
- **API enumeration** — Discovering undocumented endpoints via fuzzing.

---

### References

- Broken access control — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/3-broken-access-control
- OWASP Top 10:2025 — Detailed Security Checklist — GitHub - https://github.com/alirezarezvani/claude-skills/blob/main/engineering-team/skills/security-pen-testing/references/owasp_top_10_checklist.md
- OWASP Top 10 2025: the ranking, the changes and the 2026 data — Patrowl - https://patrowl.io/blog/owasp-top-10-2025

---

## 7. Sensitive Data Exposure (Cryptographic Failures)

### Definitions

**Core Definition**
Sensitive data exposure is the unintentional disclosure of confidential information — passwords, personal data, financial records, health information, API keys, and tokens — to unauthorised parties.

**Technical Definition**
In the OWASP Top 10:2025, this category is **A04 Cryptographic Failures** (previously “Sensitive Data Exposure” in 2017 and “Cryptographic Failures” in 2021). It encompasses weak or missing encryption at rest and in transit, improper key management, use of deprecated algorithms (MD5, SHA-1, DES), hard-coded secrets, verbose error messages that leak internal details, and insufficient protection of data in logs and backups. In .NET, the **ASP.NET Core Data Protection API** provides a secure, key-management-aware framework for encrypting sensitive data before storage. The `System.Security.Cryptography` namespace provides vetted algorithms (AES-GCM, RSA-OAEP, SHA-256+). Secrets should be stored in a managed vault (Azure Key Vault, AWS Secrets Manager) in production and in Secret Manager during development — never in source code or `appsettings.json`.

**Beginner-Friendly Explanation**
Sensitive data exposure is “private data leaking out.” It happens when passwords are stored in plaintext, when data is sent over HTTP instead of HTTPS, when encryption keys are hard-coded, or when error messages reveal internal details. In .NET, you should use HTTPS everywhere, store passwords with adaptive hashing, encrypt sensitive data at rest with the Data Protection API, and keep secrets out of your code.

---

### Purposes

- To protect personal, financial, and health data from disclosure.
- To prevent credential theft through plaintext storage.
- To satisfy PCI-DSS, HIPAA, GDPR, and SOC 2 requirements.
- To protect API keys, connection strings, and tokens.
- To prevent information leakage through error messages and logs.

---

### Syntax Rules and Structure

#### General Syntax (Encryption at Rest with Data Protection)

```csharp
public class TokenService
{
    private readonly IDataProtector _protector;
    public TokenService(IDataProtectionProvider provider)
        => _protector = provider.CreateProtector("Tokens");

    public string Protect(string plaintext) => _protector.Protect(plaintext);
    public string Unprotect(string ciphertext) => _protector.Unprotect(ciphertext);
}
```

#### General Syntax (Key Management)

```csharp
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(connectionString, containerName, blobName)
    .ProtectKeysWithAzureKeyVault(keyVaultUri, credential);
```

#### General Syntax (Secrets in Development vs. Production)

```bash
# Development: Secret Manager
dotnet user-secrets set "ConnectionStrings:Default" "..."

# Production: environment variables or managed vault
builder.Configuration.AddAzureKeyVault(new Uri(vaultUri), new DefaultAzureCredential());
```

#### Syntax Rules

- Enforce HTTPS everywhere; use HSTS outside development.
- Use the Data Protection API for reversible encryption of sensitive data at rest.
- Use adaptive hashing (Argon2id, bcrypt ≥ 12, PBKDF2 with OWASP iterations) for passwords; never store plaintext.
- Store secrets in Secret Manager (development) and a managed vault (production); never commit them to source control.
- Suppress detailed error pages in production; use `UseExceptionHandler`.
- Do not log secrets, tokens, or sensitive PII.

#### Constraints and Limitations

- Encryption keys must be stored separately from the encrypted data; key rotation must be planned.
- The Data Protection API is designed for ASP.NET Core applications; other .NET app types may need a different key store.
- `appsettings.json` is not a secure secret store; use it only for non-sensitive configuration.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Encrypting Sensitive Data at Rest

```csharp
public class PatientService
{
    private readonly IDataProtector _protector;
    public PatientService(IDataProtectionProvider provider)
        => _protector = provider.CreateProtector("PatientData");

    public string EncryptSsn(string ssn) => _protector.Protect(ssn);
    public string DecryptSsn(string ciphertext) => _protector.Unprotect(ciphertext);
}
```

**Expected Output (conceptual)**

```
Plaintext SSN is encrypted before storage.
The ciphertext cannot be decrypted without the Data Protection keys.
```

**Why this output occurs**

- The Data Protection API uses authenticated encryption with managed keys.
- The protector is scoped to a purpose (“PatientData”), preventing cross-purpose decryption.

---

#### Example 2: Secret Management

```csharp
// ❌ NEVER commit secrets
// appsettings.json: "ConnectionStrings": { "Default": "Server=...;Password=..." }

// ✅ Development: Secret Manager
// ✅ Production: managed vault
builder.Configuration.AddAzureKeyVault(new Uri(vaultUri), new DefaultAzureCredential());
```

**Expected Output (conceptual)**

```
Secrets are loaded at runtime from a secure store; none are present in the repository.
```

**Why this output occurs**

- Secret Manager keeps development secrets outside the repository.
- Key Vault provides audited, access-controlled secret storage in production.

---

### Real-World Cases

- **Healthcare** — Encrypting patient records at rest and in transit.
- **Financial services** — Protecting cardholder data per PCI-DSS.
- **SaaS platforms** — Storing API keys and connection strings in a vault.
- **All applications** — HTTPS, HSTS, secure cookies, and suppressed error details.

---

### References

- Data Protection & Encryption for ASP.NET Core — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/main/skills/data-protection/SKILL.md
- OWASP Top 10:2025 — Cryptographic Failures (A04) — OWASP - https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/
- ASP.NET Core Security Checklist (2026 Production Guide) — safeguard.sh - https://safeguard.sh/resources/blog/aspnet-core-security-checklist

---

## 8. Security Misconfiguration

### Definitions

**Core Definition**
Security misconfiguration is a vulnerability caused by insecure default settings, incomplete configuration, verbose error messages, unnecessary features enabled, or missing hardening across the application stack.

**Technical Definition**
Security misconfiguration rose from **#5 in 2021 to #2 in the OWASP Top 10:2025** (A02), because misconfigurations appeared more frequently in the data for this cycle — 3.00% of the applications tested had one or more of the 16 CWEs in this category. It encompasses default accounts and passwords, enabled debugging features in production, detailed error pages, missing security headers, permissive CORS, unnecessary HTTP methods, directory listing, and unhardened cloud or container settings. In ASP.NET Core, misconfiguration includes failing to enable HTTPS redirection and HSTS, leaving the developer exception page active in production, not setting security headers, using default cookie settings without `HttpOnly`/`Secure`/`SameSite`, and not configuring host filtering or forwarded headers correctly.

**Beginner-Friendly Explanation**
Security misconfiguration is “leaving the default, insecure settings in place.” Examples: the app shows detailed error pages to users, debug mode is on in production, HTTPS is not enforced, security headers are missing, or CORS allows any origin. The fix is to harden every layer deliberately — use production-safe defaults, disable what you do not need, and set security headers.

---

### Purposes

- To eliminate insecure defaults that attackers rely on.
- To reduce the attack surface exposed by unnecessary features.
- To prevent information leakage through verbose errors.
- To satisfy OWASP A02:2025 requirements.
- To ensure consistent hardening across environments.

---

### Syntax Rules and Structure

#### General Hardening Checklist

```
✅ Enforce HTTPS redirection and HSTS outside development.
✅ Disable developer exception page in production; use UseExceptionHandler.
✅ Set security headers (CSP, X-Content-Type-Options, X-Frame-Options, Referrer-Policy).
✅ Configure host filtering to prevent host-header attacks.
✅ Do not enable forwarded headers middleware without a trusted proxy.
✅ Set secure cookie flags (HttpOnly, Secure, SameSite).
✅ Disable unused HTTP methods and endpoints.
✅ Use a restrictive CORS policy; never AllowAnyOrigin with credentials.
```

#### General Syntax (Security Headers)

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Append("X-Content-Type-Options", "nosniff");
    context.Response.Headers.Append("X-Frame-Options", "DENY");
    context.Response.Headers.Append("Referrer-Policy", "no-referrer");
    context.Response.Headers.Append("Content-Security-Policy",
        "default-src 'self'; script-src 'self'");
    await next();
});
```

#### General Syntax (Host Filtering)

```csharp
builder.Services.AddHostFiltering(options =>
{
    options.AllowedHosts = new[] { "example.com", "www.example.com" };
});
```

#### Syntax Rules

- Harden every layer: framework, web server, database, container, and cloud.
- Use environment-specific configuration; never ship development settings to production.
- Review security headers and CORS policies during code review.
- Automate configuration checks with analyzers and security testing.

#### Constraints and Limitations

- Security headers can break legitimate functionality if not tested; start in report-only mode for CSP.
- Host filtering must account for legitimate proxy and load-balancer hostnames.
- Forwarded headers must only be enabled when a trusted proxy is in front of the application.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Production Exception Handling

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

#### Example 2: Security Headers Middleware

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["X-Frame-Options"] = "DENY";
    context.Response.Headers["Content-Security-Policy"] =
        "default-src 'self'; script-src 'self'; object-src 'none'";
    await next();
});
```

**Expected Output (conceptual)**

```
Browser enforces no-sniff, blocks framing, and restricts scripts to same-origin.
```

**Why this output occurs**

- Each header closes a specific browser-level attack vector.

---

### Real-World Cases

- **Cloud deployments** — Public storage buckets, overly permissive IAM roles, and default VPC settings.
- **Containers** — Running as root, unnecessary capabilities, and secrets in image layers.
- **Web servers** — Directory listing enabled, default error pages, and unused modules loaded.
- **APIs** — Permissive CORS, verbose error payloads, and enabled Swagger in production.

---

### References

- ASP.NET Core Security Checklist (2026 Production Guide) — safeguard.sh - https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- ASP.NET Core Kestrel web server security considerations — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/security
- OWASP Top 10:2025 — Security Misconfiguration (A02) — OWASP - https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/
- ASP.NET Core Security Best Practices 2026 — Offensive360 - https://offensive360.com/blog/aspnet-core-security-best-practices

---

## 9. OWASP Top 10 (2025)

### Definitions

**Core Definition**
The OWASP Top 10 is a standard awareness document for developers and web application security, representing the most critical security risks to web applications, updated periodically based on data from security firms and the community.

**Technical Definition**
The **OWASP Top 10:2025** edition is the current list. It reordered several categories and introduced a new one. The 2025 list is: **A01 Broken Access Control** (now includes SSRF), **A02 Security Misconfiguration** (up from #5), **A03 Software Supply Chain Failures** (widened from vulnerable components to include build pipelines and distribution), **A04 Cryptographic Failures** (down from #2), **A05 Injection** (down from #3), **A06 Insecure Design**, **A07 Authentication Failures**, **A08 Software or Data Integrity Failures**, **A09 Security Logging and Alerting Failures**, and **A10 Mishandling of Exceptional Conditions** (new). Each category maps to specific CWEs and provides guidance on prevention. The list is not exhaustive — it is a starting point for prioritisation, not a complete security program.

**Beginner-Friendly Explanation**
OWASP Top 10 is “the industry's most-wanted list of web security risks.” It tells you which vulnerabilities are most common and most dangerous right now. The 2025 edition puts broken access control at #1 (the same as 2021), moves security misconfiguration up to #2, and adds a new category for mishandling exceptional conditions. If you fix the items on this list, you eliminate the majority of real-world attack surface.

---

### Purposes

- To provide a shared vocabulary for security discussions.
- To prioritise security effort based on real-world data.
- To guide secure coding standards and security testing.
- To satisfy compliance and audit requirements that reference OWASP.
- To track the evolution of the threat landscape over time.

---

### Syntax Rules and Structure

#### OWASP Top 10:2025 Categories

| # | Category | 2021 Equivalent | Key Change |
|---|---|---|---|
| **A01** | Broken Access Control | A01 + A10 (SSRF) | SSRF folded into A01 |
| **A02** | Security Misconfiguration | A05 | Rose from #5 to #2 |
| **A03** | Software Supply Chain Failures | A06 (widened) | Now includes build pipelines and distribution |
| **A04** | Cryptographic Failures | A02 | Fell from #2 to #4 |
| **A05** | Injection | A03 | Fell from #3 to #5 |
| **A06** | Insecure Design | A04 | Unchanged position |
| **A07** | Authentication Failures | A07 | Unchanged position |
| **A08** | Software or Data Integrity Failures | A08 | Unchanged position |
| **A09** | Security Logging and Alerting Failures | A09 | Unchanged position |
| **A10** | Mishandling of Exceptional Conditions | New | New category |

*Source: OWASP Top 10:2025 checklist* 

#### Syntax Rules

- Use the Top 10 as a starting point; it does not cover every vulnerability class.
- Map your security controls to each category and verify coverage.
- Track changes between editions; the 2025 list reordered and expanded categories significantly.
- Combine the Top 10 with framework-specific guidance (OWASP ASVS, .NET security cheat sheets).

#### Constraints and Limitations

- The Top 10 is awareness-focused, not a complete standard; OWASP ASVS provides deeper verification requirements.
- Category names and rankings change between editions; update threat models and training accordingly.
- The list reflects web application risks; other domains (mobile, API, cloud) have dedicated OWASP lists.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Mapping .NET Controls to OWASP Top 10:2025

```
A01 Broken Access Control → [Authorize] + fallback policy + resource-based checks
A02 Security Misconfiguration → HTTPS/HSTS + security headers + host filtering
A03 Software Supply Chain Failures → NuGet Audit + SBOM + package source mapping
A04 Cryptographic Failures → Data Protection API + adaptive password hashing
A05 Injection → EF Core LINQ + parameterised raw SQL + output encoding
A06 Insecure Design → Threat modeling + secure design principles
A07 Authentication Failures → MFA + secure token validation + lockout
A08 Software/Data Integrity Failures → System.Text.Json + signed artifacts
A09 Logging & Alerting Failures → Audit logs + 403 monitoring + alerts
A10 Mishandling Exceptional Conditions → Fail securely + generic error pages
```

**Expected Output (conceptual)**

```
Each OWASP category has a corresponding .NET control mapped to it.
```

**Why this output occurs**

- The mapping provides a practical checklist for .NET developers.

---

### Real-World Cases

- **Security training** — OWASP Top 10 is the standard curriculum for developer security awareness.
- **Compliance** — PCI-DSS, SOC 2, and ISO 27001 reference OWASP guidance.
- **Code review** — Reviewers use the Top 10 as a mental checklist.
- **Penetration testing** — Testers structure findings around OWASP categories.

---

### References

- OWASP Top 10:2025 — Official — https://top10.owasp.org
- OWASP Top 10:2025 — Detailed Security Checklist — GitHub - https://github.com/alirezarezvani/claude-skills/blob/main/engineering-team/skills/security-pen-testing/references/owasp_top_10_checklist.md
- OWASP Top Ten Vulnerabilities: The 2025 List Explained — safeguard.sh - https://safeguard.sh/resources/blog/owasp-top-ten-2025
- OWASP Top 10 2025: the ranking, the changes and the 2026 data — Patrowl - https://patrowl.io/blog/owasp-top-10-2025

---

## 10. Security Best Practices

### Definitions

**Core Definition**
Security best practices are empirically grounded guidelines for designing, coding, configuring, and operating .NET applications to minimise vulnerability to the common attack classes described in this cheat sheet.

**Technical Definition**
Best practices in .NET security span **input handling** (validate at boundaries, parameterise queries, encode output), **deserialisation** (avoid `BinaryFormatter`, avoid `TypeNameHandling`, use `System.Text.Json`), **authorization** (deny by default, `[Authorize]` everywhere, resource-based checks), **secrets management** (Secret Manager in development, managed vault in production), **cryptography** (Data Protection API, adaptive hashing, never roll your own), **configuration hardening** (HTTPS/HSTS, security headers, host filtering, generic error pages), **dependency management** (NuGet Audit, SBOMs, package source mapping), and **security testing** (SAST, DAST, dependency scanning, penetration testing).

**Beginner-Friendly Explanation**
Security best practices are “the checklist that keeps your app safe from the common attacks.” They include: parameterise your SQL, encode your output, protect state-changing requests with antiforgery tokens, validate URLs before fetching them, never deserialise untrusted data into arbitrary types, require authorization on every endpoint, keep secrets out of Git, use HTTPS, harden your configuration, and audit your dependencies. Following these prevents the vulnerabilities on the OWASP Top 10.

---

### Purposes

- To prevent the common vulnerability classes described in this cheat sheet.
- To establish a secure baseline for every .NET application.
- To satisfy OWASP, NIST, and regulatory requirements.
- To integrate security into the development lifecycle.
- To reduce the blast radius of a compromise.

---

### Guidelines and Structure

#### Injection Prevention

```
✅ Use EF Core LINQ; it parameterises by default.
✅ Use FromSqlInterpolated or explicit DbParameter for raw SQL.
✅ Allow-list dynamic identifiers (columns, sort directions).
❌ Never concatenate user input into SQL.
```

#### XSS Prevention

```
✅ Let Razor encode output; avoid Html.Raw on untrusted data.
✅ Use Json.Serialize for data embedded in script blocks.
✅ Set a restrictive Content Security Policy.
✅ Use HttpOnly cookies.
```

#### CSRF Prevention

```
✅ Apply [AutoValidateAntiforgeryToken] globally.
✅ Include tokens in all state-changing forms.
✅ Configure SameSite cookies and secure cookie flags.
```

#### SSRF Prevention

```
✅ Allow-list schemes and hosts.
✅ Validate resolved IPs against private ranges.
✅ Use the Microsoft AntiSSRF library.
```

#### Deserialisation Safety

```
✅ Use System.Text.Json with concrete types.
✅ Use [JsonDerivedType] allow-lists for polymorphism.
❌ Never use BinaryFormatter (removed in .NET 9).
❌ Keep Newtonsoft.Json TypeNameHandling at None.
```

#### Broken Access Control Prevention

```
✅ Require authorization by default (fallback policy).
✅ Apply [Authorize] to every externally facing endpoint.
✅ Use resource-based authorization for per-object checks.
✅ Deny by default in handlers.
```

#### Sensitive Data Protection

```
✅ Enforce HTTPS and HSTS.
✅ Encrypt sensitive data at rest with the Data Protection API.
✅ Use adaptive password hashing (Argon2id, bcrypt ≥ 12, PBKDF2 with OWASP iterations).
✅ Store secrets in Secret Manager (dev) and a managed vault (prod).
✅ Suppress detailed errors in production.
```

#### Configuration Hardening

```
✅ Disable developer exception page in production.
✅ Set security headers (CSP, X-Content-Type-Options, X-Frame-Options).
✅ Configure host filtering.
✅ Use restrictive CORS; never AllowAnyOrigin with credentials.
```

#### Supply Chain Security

```
✅ Enable NuGet Audit (default in .NET 8+; transitive in .NET 9+).
✅ Treat audit warnings as errors in CI.
✅ Generate an SBOM on every build.
✅ Use package source mapping.
```

#### Security Testing

```
✅ Run SAST (Roslyn security analyzers, CodeQL, SonarQube).
✅ Run DAST against running applications.
✅ Scan dependencies and containers.
✅ Conduct penetration testing for critical applications.
```

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Secure-by-Default ASP.NET Core Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Authorization: deny by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Antiforgery
builder.Services.AddAntiforgery(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});

// Data protection with persistent keys
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(connectionString, containerName, blobName);

var app = builder.Build();

// HTTPS and HSTS
app.UseHttpsRedirection();
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseExceptionHandler("/error");
}

// Security headers
app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["X-Frame-Options"] = "DENY";
    context.Response.Headers["Content-Security-Policy"] =
        "default-src 'self'; script-src 'self'";
    await next();
});

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
Security headers present on every response.
HTTPS enforced; HSTS sent outside development.
```

**Why this output occurs**

- Each best practice is configured explicitly; the application is secure by default.

---

### Real-World Cases

- **Enterprise .NET applications** — Layered security controls across the OWASP Top 10.
- **Open-source libraries** — NuGet Audit, signed packages, and safe deserialisation.
- **Cloud-native services** — Managed identities, Key Vault, and container hardening.
- **Regulated industries** — Documented alignment with OWASP ASVS and NIST.

---

### References

- OWASP Top 10:2025 — https://top10.owasp.org
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- C# Secure Coding Guide (2026) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-secure-coding-guide
- ASP.NET Core Security Checklist (2026 Production Guide) — safeguard.sh - https://safeguard.sh/resources/blog/aspnet-core-security-checklist

---

## Summary Table: Vulnerabilities and .NET Mitigations

| Vulnerability | OWASP 2025 | Primary .NET Mitigation |
|---|---|---|
| **SQL Injection** | A05 Injection | EF Core LINQ / `FromSqlInterpolated` / `DbParameter` |
| **XSS** | A05 Injection | Razor auto-encoding / `Json.Serialize` / CSP |
| **CSRF** | A01 Broken Access Control | `[AutoValidateAntiforgeryToken]` / `SameSite` cookies |
| **SSRF** | A01 Broken Access Control | Allow-list / Microsoft AntiSSRF / IP validation |
| **Insecure Deserialization** | A08 Software/Data Integrity | `System.Text.Json` / `[JsonDerivedType]` / no `BinaryFormatter` |
| **Broken Access Control** | A01 | Fallback policy / `[Authorize]` / resource-based checks |
| **Sensitive Data Exposure** | A04 Cryptographic Failures | Data Protection API / HTTPS / adaptive hashing / vault |
| **Security Misconfiguration** | A02 | HSTS / security headers / host filtering / generic errors |

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| **`BinaryFormatter`** | Removed in .NET 9; cannot be made secure; do not use |
| **`Newtonsoft.Json` `TypeNameHandling.All/Auto/Objects`** | Unsafe deserialisation RCE; keep at `None` |
| **`FromSqlRaw` with concatenation** | SQL injection risk; use `FromSqlInterpolated` |
| **`Html.Raw` on untrusted data** | XSS risk; let Razor encode |
| **`UseDeveloperExceptionPage` in production** | Information disclosure; use `UseExceptionHandler` |
| **`AllowAnyOrigin` with credentials in CORS** | Insecure combination; use specific origins |
| **OWASP Top 10:2021** | Superseded by the 2025 edition; update mappings |
| **`[ValidateAntiForgeryToken]` on GET** | Unnecessary; use `[AutoValidateAntiforgeryToken]` for unsafe methods |
| **Missing fallback authorization policy** | Endpoints without metadata are anonymously accessible |
| **NuGet Audit** | Default in .NET 8+; transitive in .NET 9+; treat warnings as errors |

---

## References (Consolidated)

- OWASP Top 10:2025 — https://top10.owasp.org
- OWASP Top 10:2025 — Detailed Security Checklist — GitHub - https://github.com/alirezarezvani/claude-skills/blob/main/engineering-team/skills/security-pen-testing/references/owasp_top_10_checklist.md
- OWASP Top Ten Vulnerabilities: The 2025 List Explained — safeguard.sh - https://safeguard.sh/resources/blog/owasp-top-ten-2025
- OWASP Top 10 2025: the ranking, the changes and the 2026 data — Patrowl - https://patrowl.io/blog/owasp-top-10-2025
- SQL Queries — EF Core — Microsoft Learn - https://learn.microsoft.com/en-us/ef/core/querying/sql-queries
- SQL Injection Prevention in EF Core (.NET 2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/ef-core-sql-injection-prevention
- Preventing SQL Injection with Entity Framework — safeguard.sh - https://safeguard.sh/resources/blog/preventing-sql-injection-with-entity-framework
- SQL Injection Prevention in C# with Entity Framework — safeguard.sh - https://safeguard.sh/resources/blog/sql-injection-prevention-csharp-entity-framework
- Prevent Cross-Site Scripting (XSS) in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting
- CSRF and XSS Prevention in ASP.NET Core (2026) — safeguard.sh - https://safeguard.sh/resources/blog/csrf-xss-prevention-aspnet-core
- How do you prevent XSS in ASP.NET Core and C# web apps? — safeguard.sh - https://safeguard.sh/resources/blog/prevent-xss-aspnet-core
- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Server-side request forgery — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/12-server-side-request-forgery
- Microsoft AntiSSRF Library for .NET — GitHub - https://github.com/microsoft/AntiSSRF
- Insecure Deserialization in .NET (2026): BinaryFormatter & JSON — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-insecure-deserialization
- How do you deserialize data safely in C# and .NET? — safeguard.sh - https://safeguard.sh/resources/blog/safe-deserialization-csharp-dotnet
- Do not deserialize untrusted data (.NET binary formatters) — Microsoft Learn - https://learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-security-guide
- Broken access control — Microsoft Learn - https://learn.microsoft.com/en-us/training/modules/owasp-top-10-for-dotnet-developers/3-broken-access-control
- Data Protection & Encryption for ASP.NET Core — dotnet-skills - https://github.com/wshaddix/dotnet-skills/blob/main/skills/data-protection/SKILL.md
- ASP.NET Core Security Checklist (2026 Production Guide) — safeguard.sh - https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- ASP.NET Core Kestrel web server security considerations — Microsoft Learn - https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/security
- .NET Security Best Practices (2026 Guide) — safeguard.sh - https://safeguard.sh/resources/blog/dotnet-security-best-practices
- C# Secure Coding Guide (2026) — safeguard.sh - https://safeguard.sh/resources/blog/csharp-secure-coding-guide