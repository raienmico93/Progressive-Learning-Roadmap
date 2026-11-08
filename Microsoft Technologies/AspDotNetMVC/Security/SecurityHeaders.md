# ASP.NET Core Security Headers: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **security response headers** that activate browser‑side defenses against XSS, clickjacking, MIME‑sniffing, referrer leakage, and feature‑API abuse. A critical framing note: **ASP.NET Core does not send a Content‑Security‑Policy by default**, and most security headers are entirely opt‑in. Headers must be added by **middleware** (custom or a library such as `NetEscapades.AspNetCore.SecurityHeaders`), registered **early in the pipeline** so static files and error pages inherit them. Several legacy headers (`X-XSS-Protection`, `Public-Key-Pins`) are **deprecated** and should be omitted. Each requested header is treated independently — no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security headers are HTTP response headers that instruct the browser to enforce security policies — restricting script execution, framing, MIME‑type interpretation, referrer leakage, and feature‑API access. |
| **Technical Definition** | Security headers are set on the HTTP response and interpreted by the browser’s security model. They include **Content‑Security‑Policy (CSP)** (script/style/resource allowlisting), **X‑Frame‑Options** and the CSP **`frame-ancestors`** directive (clickjacking protection), **X‑Content‑Type‑Options** (MIME‑sniffing prevention), **Referrer‑Policy** (referrer leakage control), and **Permissions‑Policy** (browser feature/API restriction). Additional headers — **Cross‑Origin‑Opener‑Policy (COOP)** , **Cross‑Origin‑Embedder‑Policy (COEP)** , and **Cross‑Origin‑Resource‑Policy (CORP)** — provide process isolation for advanced cross‑origin scenarios. |
| **Beginner‑Friendly Explanation** | Security headers are the **browser’s rulebook** for your site. They travel invisibly in every response and tell the browser: “Only run scripts from these sources,” “Never let another site frame this page,” “Don’t guess the content type — trust the declared one,” “Don’t leak our URLs to third parties,” and “Don’t let JavaScript access the camera unless we allow it.” Without them, the browser uses permissive defaults that attackers can exploit. |

### Key Characteristics

- **Browser‑enforced** — Headers are policies the browser applies; the server does not enforce them.
- **Opt‑in** — ASP.NET Core sends **no CSP** and only a limited default `X‑Frame-Options: SAMEORIGIN` on certain responses.
- **Defense‑in‑depth** — Headers complement, not replace, server‑side validation, encoding, and authorization.
- **Context‑dependent** — A strict CSP for one application may break another; headers must be tuned to the application’s actual resource requirements.
- **Deprecation‑prone** — `X-XSS-Protection` and HPKP are obsolete; CSP is the modern replacement.
- **Middleware‑dependent** — Header order in the pipeline matters; a misordered pipeline can silently drop headers.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **HTTP response headers**, and the **browser security model**.
- Familiarity with **CSP directives**, **framing**, and **MIME types**.
- Optional: `NetEscapades.AspNetCore.SecurityHeaders` NuGet package for a structured, testable header policy collection.

### Related Programming Areas

- **XSS prevention** — CSP is the strongest browser‑side defense; output encoding is the server‑side defense.
- **Clickjacking prevention** — `X-Frame-Options` and CSP `frame-ancestors`.
- **Transport security** — HSTS (covered separately) and HTTPS.
- **Cookie security** — `Secure`, `HttpOnly`, `SameSite` complement the header posture.
- **Cross‑origin isolation** — COOP, COEP, and CORP for advanced scenarios.

### Core Concepts / Features

1. **Security Headers** — The overall middleware‑based approach and header set.
2. **CSP (Content‑Security‑Policy)** — The most powerful and complex header.
3. **`X-Frame-Options`** — Clickjacking protection (and its CSP successor).
4. **`X-Content-Type-Options`** — MIME‑sniffing prevention.
5. **`Referrer-Policy`** — Referrer leakage control.
6. **`Permissions-Policy`** — Browser feature and API restriction.
7. **Security Header Best Practices** — Baseline values, rollout strategy, and verification.

---

## 1. Security Headers (The Middleware‑Based Approach)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security headers are a set of HTTP response headers added by the application (or reverse proxy) that instruct the browser to enforce security policies on the page. |
| **Technical Definition** | In ASP.NET Core, security headers are added via **custom middleware** (e.g., `app.Use(async (ctx, next) => { ctx.Response.Headers[...] = ...; await next(); })`) or a **library** such as `NetEscapades.AspNetCore.SecurityHeaders`, which provides a `HeaderPolicyCollection` and a `UseSecurityHeaders()` extension. The middleware must be registered **before** static files and endpoint routing so that all responses — including error pages and static assets — inherit the headers. |
| **Beginner‑Friendly Explanation** | Security headers are the **invisible instructions** your server sends with every response. You add them once, in middleware, and every page, API response, and static file gets them. The middleware is like a stamp that marks every outgoing response with the browser’s rulebook. |

### Purposes

- **To** activate browser‑side defenses that the server cannot enforce alone (CSP, framing, MIME‑sniffing).
- **To** provide a **consistent security posture** across all responses, including error pages and static files.
- **To** **reduce the attack surface** for XSS, clickjacking, and data injection.
- **To** satisfy **security audits and compliance** requirements.
- **To** centralize header configuration in one place, making it reviewable and testable.

### Syntax Rules and Structure

#### General Syntax (Custom Middleware)

```csharp
app.Use(async (ctx, next) =>
{
    var h = ctx.Response.Headers;
    h["Content-Security-Policy"] = "default-src 'self'; object-src 'none'; frame-ancestors 'none'";
    h["X-Content-Type-Options"] = "nosniff";
    h["Referrer-Policy"] = "no-referrer";
    await next();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ctx.Response.Headers` | The response header collection. |
| `h["Header-Name"] = "value"` | Sets (or overwrites) a header. |
| `await next()` | Continues the pipeline; the headers are sent with the response. |

#### General Syntax (NetEscapades Library)

```csharp
app.UseSecurityHeaders(policyCollection);
```

**Syntax rules:**

- **Register the middleware early** — before static files, routing, and endpoints.
- **Avoid duplicate headers** — multiple occurrences with conflicting values can disable protections.
- **Do not add deprecated headers** — omit `X-XSS-Protection` and HPKP.
- **Verify headers in production** — misordered middleware can silently drop them.

**Constraints and limitations:**

- **Headers are not a substitute for server‑side security** — validation, encoding, authorization, and HTTPS remain essential.
- **Overly strict headers break functionality** — a CSP that blocks legitimate scripts will break the UI.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Security headers middleware — early in the pipeline
app.Use(async (ctx, next) =>
{
    var h = ctx.Response.Headers;
    h["X-Content-Type-Options"] = "nosniff";
    h["X-Frame-Options"] = "DENY";
    h["Referrer-Policy"] = "strict-origin-when-cross-origin";
    h["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()";
    h["Content-Security-Policy"] =
        "default-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'";
    await next();
});

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. The middleware is registered before static files and routing.
2. It sets the baseline security headers on every response.
3. `await next()` allows the rest of the pipeline to execute; the headers are included in the final response.

**Expected Output:** Every response — HTML pages, API JSON, static files, and error pages — includes the configured security headers.

**Why this output?** The middleware runs for every request and sets the headers before the response is generated.

### Real‑World Cases

- **Public web applications** — baseline headers on every page.
- **APIs** — at minimum `X-Content-Type-Options: nosniff` and a restrictive CSP for any HTML responses.
- **Reverse‑proxy deployments** — headers can be set at the proxy or in the app; consistency is key.

### References

- ASP.NET Core Security Checklist (headers section) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- NetEscapades.AspNetCore.SecurityHeaders (GitHub) – https://github.com/andrewlock/netescapades.aspnetcore.securityheaders
- Security Headers for ASP.NET Core 10 on IIS (ASPnix) – https://www.aspnix.com/posts/security-headers-aspnet-core-10-on-iis
- OWASP WSTG — Other HTTP Security Header Misconfigurations – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/

---

## 2. CSP (Content‑Security‑Policy)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Content‑Security‑Policy is an HTTP response header that allows the server to declare which sources of scripts, styles, images, fonts, and other resources the browser is allowed to load and execute. |
| **Technical Definition** | CSP is a **declarative allowlist** composed of directives (e.g., `default-src`, `script-src`, `style-src`, `img-src`, `connect-src`, `frame-ancestors`). It is the **primary browser‑side defense against XSS** and data‑injection attacks. The strongest deployments use **nonce‑based** or **hash‑based** script policies rather than `'unsafe-inline'`, which defeats most of CSP’s XSS value. A **per‑request nonce** is generated, stored in `HttpContext.Items`, included in the CSP header, and applied to legitimate inline scripts in Razor views. |
| **Beginner‑Friendly Explanation** | CSP is the **“guest list”** for your page’s resources. It tells the browser: “Only execute scripts from `self` and this CDN; only load images from `self` and `data:`; never allow this page to be framed.” An attacker who injects a script tag finds that the browser refuses to run it because the script’s source is not on the guest list. The strongest guest lists use a **nonce** — a one‑time password that changes every request — so only scripts your server explicitly marked are allowed. |

### Purposes

- **To** **prevent XSS** by blocking injected scripts that are not on the allowlist.
- **To** **prevent data injection** by restricting `connect-src`, `form-action`, and `base-uri`.
- **To** **prevent clickjacking** via `frame-ancestors` (the modern replacement for `X-Frame-Options`).
- **To** **provide a report‑only mode** (`Content-Security-Policy-Report-Only`) for safe rollout and violation monitoring.
- **To** **enforce least privilege** on the browser — only the resources the page actually needs are allowed.

### Syntax Rules and Structure

#### General Syntax (Starting CSP)

```text
default-src 'self';
object-src 'none';
base-uri 'self';
frame-ancestors 'none';
form-action 'self';
script-src 'self';
style-src 'self';
img-src 'self' data:;
```

**Component breakdown:**

| Directive | Purpose | Baseline Value |
|-----------|---------|----------------|
| `default-src` | Fallback for all resource types. | `'self'` |
| `object-src` | `<object>`, `<embed>`, `<applet>`. | `'none'` |
| `base-uri` | Restricts `<base>` tag URLs. | `'self'` |
| `frame-ancestors` | Which pages may frame this page. | `'none'` |
| `form-action` | Where forms may submit. | `'self'` |
| `script-src` | JavaScript sources. | `'self'` or nonce |
| `style-src` | CSS sources. | `'self'` or nonce |
| `img-src` | Image sources. | `'self' data:` |

#### Nonce‑Based CSP (Strongest XSS Defense)

```csharp
app.Use(async (context, next) =>
{
    var nonce = Convert.ToBase64String(
        System.Security.Cryptography.RandomNumberGenerator.GetBytes(16));
    context.Items["CspNonce"] = nonce;

    context.Response.Headers.Append("Content-Security-Policy",
        $"default-src 'self'; " +
        $"script-src 'self' 'nonce-{nonce}'; " +
        $"style-src 'self' 'nonce-{nonce}'; " +
        "object-src 'none'; frame-ancestors 'none'; base-uri 'self'");

    await next();
});
```

**Syntax rules:**

- **Generate a unique nonce per request** — never reuse a nonce across requests.
- **Apply the nonce to legitimate inline scripts** in Razor views (e.g., `<script nonce="@Context.Items["CspNonce"]">`).
- **Prefer nonce over `'unsafe-inline'`** — `'unsafe-inline'` defeats most of CSP’s XSS protection.
- **Start in Report‑Only mode** (`Content-Security-Policy-Report-Only`) to identify legitimate resources before enforcing.
- **Do not add external origins you do not need** — every allowed origin is a potential attack vector.

**Constraints and limitations:**

- **CSP is complex and application‑specific** — a policy that works for one app may break another.
- **`frame-ancestors` is ignored in `<meta>` tags** — CSP must be delivered as an HTTP header for framing protection.
- **CSP does not replace server‑side encoding** — it is a defense‑in‑depth layer, not a substitute.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    var nonce = Convert.ToBase64String(
        RandomNumberGenerator.GetBytes(16));
    context.Items["CspNonce"] = nonce;

    context.Response.Headers.Append("Content-Security-Policy",
        "default-src 'self'; " +
        $"script-src 'self' 'nonce-{nonce}' https://cdn.trusted.com; " +
        $"style-src 'self' 'nonce-{nonce}'; " +
        "img-src 'self' data:; " +
        "object-src 'none'; " +
        "frame-ancestors 'none'; " +
        "base-uri 'self'; " +
        "form-action 'self'");

    await next();
});

app.UseStaticFiles();
app.UseRouting();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. A cryptographically random nonce is generated for each request.
2. The nonce is stored in `HttpContext.Items` for Razor views to use.
3. The CSP header is appended with the nonce in `script-src` and `style-src`.
4. Legitimate inline scripts must include the same nonce to execute.

**Expected Output:** Injected scripts without the correct nonce are blocked by the browser; legitimate scripts with the nonce execute; the page cannot be framed by other sites.

**Why this output?** The browser enforces the allowlist; only sources explicitly permitted (or scripts bearing the nonce) are executed.

### Real‑World Cases

- **SPAs with server‑rendered shells** — nonce‑based CSP for the shell and API‑driven client resources.
- **Content‑heavy sites** — CSP with `img-src` and `style-src` tuned to the CDN and font providers in use.
- **Blazor applications** — CSP with `wasm-unsafe-eval` for WebAssembly execution and `connect-src` for SignalR/HTTP.

### References

- ASP.NET Core Security Checklist (CSP guidance) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Implementing CSP with Kendo UI for ASP.NET Core (Telerik) – https://www.telerik.com/aspnet-mvc/documentation/knowledge-base/csp-compliance-kendo-ui-aspnet-core
- Content Security Policy for ASP.NET Core Blazor (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/blazor/security/content-security-policy
- OWASP WSTG — Security Header Misconfigurations – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/

---

## 3. `X-Frame-Options`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `X-Frame-Options` is an HTTP response header that instructs the browser whether the page may be rendered inside a `<frame>` or `<iframe>`, protecting against **clickjacking**. |
| **Technical Definition** | `X-Frame-Options` supports three values: **`DENY`** (no framing at all), **`SAMEORIGIN`** (framing only from the same origin), and **`ALLOW-FROM uri`** (deprecated, limited browser support). It is the **legacy predecessor** of the CSP **`frame-ancestors`** directive, which is the modern, more expressive replacement. ASP.NET Core emits `X-Frame-Options: SAMEORIGIN` by default in some contexts (e.g., antiforgery responses), and `SuppressXFrameOptionsHeader` can disable it. |
| **Beginner‑Friendly Explanation** | `X-Frame-Options` is the **“no embedding”** sign. It tells the browser: “Don’t let another site put this page in an iframe.” Without it, an attacker can overlay a transparent iframe of your bank page on their own page and trick users into clicking buttons they did not intend — that is clickjacking. `DENY` blocks all framing; `SAMEORIGIN` allows your own site to frame itself. |

### Purposes

- **To** **prevent clickjacking** — stop attackers from embedding your page in a hidden or transparent frame.
- **To** **protect sensitive actions** — forms, account settings, and payment pages must not be framed.
- **To** **provide defense in depth** — complement CSP `frame-ancestors` for browsers that support both.
- **To** **allow same‑origin framing** when the application legitimately uses iframes internally.

### Syntax Rules and Structure

```csharp
context.Response.Headers["X-Frame-Options"] = "DENY";
// or
context.Response.Headers["X-Frame-Options"] = "SAMEORIGIN";
```

**Component breakdown:**

| Value | Meaning |
|-------|---------|
| `DENY` | No framing by any site, including the same origin. |
| `SAMEORIGIN` | Framing allowed only by pages from the same origin. |
| `ALLOW-FROM uri` | Deprecated; limited support; do not use. |

**Syntax rules:**

- **Use `DENY` by default** — if the application never needs to be framed, deny all framing.
- **Use `SAMEORIGIN` only when the application frames itself** — e.g., an internal admin dashboard with iframe widgets.
- **Pair with CSP `frame-ancestors`** — `frame-ancestors 'none'` is the modern equivalent of `DENY`.
- **Do not use `ALLOW-FROM`** — it is deprecated and unsupported by modern browsers.

**Constraints and limitations:**

- **`X-Frame-Options` is not supported inside `<meta>` tags** — it must be an HTTP header.
- **CSP `frame-ancestors` overrides `X-Frame-Options`** in browsers that support both; keep the values consistent.
- **ASP.NET Core may emit `SAMEORIGIN` by default** in certain antiforgery contexts; audit the emitted header if you need `DENY`.

### Annotated Complete Step‑by‑Step Code Example

```csharp
app.Use(async (ctx, next) =>
{
    // Deny all framing — the safest default
    ctx.Response.Headers["X-Frame-Options"] = "DENY";
    await next();
});
```

**Expected Output:** The browser refuses to render the page inside any iframe; clickjacking attempts fail.

**Why this output?** The `DENY` value instructs the browser to block all framing.

### Real‑World Cases

- **Banking and payment pages** — `DENY` to prevent clickjacking.
- **Admin dashboards** — `DENY` or `SAMEORIGIN` if internal iframes are used.
- **Marketing pages with embedded widgets** — `SAMEORIGIN` plus CSP `frame-ancestors` for fine‑grained control.

### References

- X-Frame-Options — DENY vs SAMEORIGIN (C# Corner) – https://www.c-sharpcorner.com/article/clickjacking-protection-with-x-frame-options-and-csp-in-asp-net-core/
- OWASP WSTG — X-Frame-Options and frame-ancestors – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/
- ASP.NET Core antiforgery X-Frame-Options default – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery

---

## 4. `X-Content-Type-Options`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `X-Content-Type-Options: nosniff` is an HTTP response header that prevents browsers from **MIME‑type sniffing** — guessing a resource’s content type instead of trusting the declared `Content-Type`. |
| **Technical Definition** | Without `nosniff`, browsers may interpret a file declared as `text/plain` as JavaScript or CSS if the content resembles those types, enabling **MIME‑confusion attacks** and some XSS vectors. The header forces the browser to **respect the declared `Content-Type`** and refuse to execute resources whose types do not match the context. It is a simple, universally supported header with no configuration parameters. |
| **Beginner‑Friendly Explanation** | `X-Content-Type-Options: nosniff` is the **“don’t guess”** instruction. If your server says a file is `text/plain`, the browser must treat it as plain text — not decide “this looks like JavaScript, I’ll run it.” This blocks a class of attacks where an attacker uploads a file that is declared as an image but actually contains script, and the browser “sniffs” it as executable. |

### Purposes

- **To** **prevent MIME‑sniffing attacks** — stop browsers from executing resources that are not declared as executable.
- **To** **enforce correct `Content-Type` declarations** — the server must set accurate types, and the browser must respect them.
- **To** **reduce the impact of file‑upload vulnerabilities** — uploaded files cannot be reinterpreted as scripts.
- **To** **provide a low‑cost, high‑value baseline header** — no configuration, no compatibility issues.

### Syntax Rules and Structure

```csharp
context.Response.Headers["X-Content-Type-Options"] = "nosniff";
```

**Syntax rules:**

- **The only valid value is `nosniff`** — there are no parameters or alternatives.
- **Add it to every response** — it is safe for all content types and has no downside.
- **Ensure `Content-Type` headers are accurate** — `nosniff` is only effective if the server declares correct types.

**Constraints and limitations:**

- **`nosniff` does not sanitize content** — it prevents interpretation, not injection; output encoding is still required.
- **Some legacy browsers may ignore it** — modern browsers universally support it.

### Annotated Complete Step‑by‑Step Code Example

```csharp
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    await next();
});
```

**Expected Output:** Browsers refuse to MIME‑sniff responses; a file declared as `text/plain` containing `<script>` is not executed.

**Why this output?** The browser respects the declared `Content-Type` and does not second‑guess it.

### Real‑World Cases

- **File‑upload endpoints** — prevent uploaded files from being reinterpreted as scripts.
- **APIs serving JSON** — ensure JSON is never sniffed as HTML.
- **Static file hosting** — enforce correct types for CSS, JS, and images.

### References

- ASP.NET Core Security Checklist (nosniff) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- OWASP WSTG — X-Content-Type-Options – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/
- Security Headers for ASP.NET Core 10 on IIS (ASPnix) – https://www.aspnix.com/posts/security-headers-aspnet-core-10-on-iis

---

## 5. `Referrer-Policy`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Referrer-Policy` is an HTTP response header that controls how much **referrer information** (the URL of the current page) is sent with outgoing requests and navigation. |
| **Technical Definition** | The header supports values such as **`no-referrer`** (send nothing), **`strict-origin-when-cross-origin`** (send the origin for cross‑origin requests, full URL for same‑origin), **`same-origin`** (send full URL only for same‑origin), and **`no-referrer-when-downgrade`** (send full URL except when downgrading HTTPS→HTTP). The modern recommended default is **`strict-origin-when-cross-origin`**, which prevents path and query leakage to third parties while preserving analytics for same‑origin requests. |
| **Beginner‑Friendly Explanation** | `Referrer-Policy` is the **“how much of our address do we share”** control. When a user clicks a link from your site to another site, the browser normally sends the full URL of the page they came from — including sensitive path segments and query strings. `Referrer-Policy` limits this: `no-referrer` sends nothing; `strict-origin-when-cross-origin` sends only the domain (not the path) to other sites, and only over HTTPS. |

### Purposes

- **To** **prevent sensitive information leakage** — paths, query strings, and fragments should not be sent to third parties.
- **To** **preserve analytics for same‑origin navigation** — `strict-origin-when-cross-origin` still sends the full URL within your own site.
- **To** **protect user privacy** — prevent third parties from building a profile of the pages users visit.
- **To** **comply with privacy regulations** — minimize data sharing with third parties.

### Syntax Rules and Structure

```csharp
context.Response.Headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
```

**Component breakdown:**

| Value | Behavior |
|-------|----------|
| `no-referrer` | Never send the referrer. |
| `strict-origin-when-cross-origin` | Send origin for cross‑origin; full URL for same‑origin; nothing on downgrade. |
| `same-origin` | Send full URL only for same‑origin requests. |
| `no-referrer-when-downgrade` | Send full URL except when HTTPS→HTTP. |

**Syntax rules:**

- **Use `strict-origin-when-cross-origin` as the modern default** — it balances privacy with analytics.
- **Use `no-referrer` for highly sensitive applications** — banking, healthcare, and internal tools.
- **Avoid `unsafe-url`** — it sends the full URL even to third parties and downgrade scenarios.

**Constraints and limitations:**

- **`Referrer-Policy` is a hint, not a guarantee** — some older browsers may not honor it.
- **It does not prevent the `Referer` header from being sent entirely** — it controls how much is sent.

### Annotated Complete Step‑by‑Step Code Example

```csharp
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
    await next();
});
```

**Expected Output:** Cross‑origin requests send only the origin (e.g., `https://example.com`) as the referrer; same‑origin requests send the full URL.

**Why this output?** The browser applies the policy when constructing the `Referer` header for outgoing requests.

### Real‑World Cases

- **Public websites with third‑party analytics** — `strict-origin-when-cross-origin` preserves same‑origin analytics without leaking paths.
- **Healthcare and finance** — `no-referrer` to prevent any leakage.
- **Multi‑tenant SaaS** — `same-origin` to avoid cross‑tenant referrer leakage.

### References

- OWASP WSTG — Referrer-Policy – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/
- Referrer-Policy — strict-origin-when-cross-origin (Csharp Fix) – https://db.fluidattacks.com/
- Security Headers for ASP.NET Core 10 on IIS (ASPnix) – https://www.aspnix.com/posts/security-headers-aspnet-core-10-on-iis

---

## 6. `Permissions-Policy`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `Permissions-Policy` is an HTTP response header that allows the server to declare which **browser features and APIs** (camera, microphone, geolocation, payment, etc.) the page — and any embedded frames — are allowed to use. |
| **Technical Definition** | `Permissions-Policy` uses a directive syntax: `feature=(allowlist)`. An empty allowlist (`camera=()`) **disables** the feature for all origins. `feature=(self)` allows it for the current origin. `feature=*` allows it for all origins (not recommended). The header replaces the older, less standardized `Feature-Policy`. It is a **defense‑in‑depth** control that limits the damage if an attacker achieves script execution. |
| **Beginner‑Friendly Explanation** | `Permissions-Policy` is the **“which browser features can this page use”** switchboard. If your app does not need the camera, microphone, or geolocation, you turn them off. An attacker who manages to run script on your page cannot then ask the user for camera access, because the browser has been told the page is not allowed to use the camera. |

### Purposes

- **To** **disable unnecessary browser features** — reduce the attack surface for script‑based attacks.
- **To** **control feature access in embedded iframes** — the header applies to the page and its frames.
- **To** **enforce least privilege in the browser** — only the APIs the application genuinely needs are enabled.
- **To** **replace the deprecated `Feature-Policy`** header with the standardized `Permissions-Policy`.

### Syntax Rules and Structure

```csharp
context.Response.Headers["Permissions-Policy"] =
    "camera=(), microphone=(), geolocation=(), payment=(), usb=()";
```

**Component breakdown:**

| Directive | Effect |
|-----------|--------|
| `camera=()` | Camera disabled for all origins. |
| `microphone=()` | Microphone disabled. |
| `geolocation=()` | Geolocation disabled. |
| `payment=()` | Payment Request API disabled. |
| `usb=()` | WebUSB disabled. |
| `fullscreen=(self)` | Fullscreen allowed for same origin only. |

**Syntax rules:**

- **Disable what you do not need** — start with a deny‑all baseline and enable only required features.
- **Use `=()` to disable a feature** — an empty allowlist denies all access.
- **Use `=(self)` to allow only your own origin** — the safest allowlist for features the app needs.
- **Avoid `=*`** — it allows any origin, which is rarely appropriate.

**Constraints and limitations:**

- **Browser support varies** — some features may not be recognized; the header is ignored for unknown directives.
- **It does not prevent server‑side access to data** — it only controls browser APIs.
- **It is a defense‑in‑depth control** — not a substitute for authorization or input validation.

### Annotated Complete Step‑by‑Step Code Example

```csharp
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["Permissions-Policy"] =
        "camera=(), microphone=(), geolocation=(), payment=(), usb=(), fullscreen=(self)";
    await next();
});
```

**Expected Output:** The browser refuses camera, microphone, geolocation, payment, and USB access for the page; fullscreen is allowed only for the same origin.

**Why this output?** The browser enforces the feature allowlist declared in the header.

### Real-World Cases

- **Content sites** — disable camera, microphone, and geolocation entirely.
- **Video conferencing apps** — allow camera and microphone only for the app’s origin.
- **E‑commerce checkout** — allow payment API only for the checkout origin.

### References

- Permissions-Policy in ASP.NET Core middleware (LobeHub) – https://lobehub.com/skills/…
- NetEscapades.AspNetCore.SecurityHeaders — AddPermissionsPolicy – https://www.nuget.org/packages/NetEscapades.AspNetCore.SecurityHeaders
- OWASP WSTG — Permissions-Policy – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/

---

## 7. Security Header Best Practices (The Baseline Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security header best practices are the guidelines for selecting, configuring, rolling out, and verifying headers so that they provide real protection without breaking application functionality. |
| **Technical Definition** | Best practices encompass **baseline header values** (CSP, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`), **middleware placement** (early in the pipeline), **rollout strategy** (Report‑Only CSP first), **nonce‑based CSP** over `'unsafe-inline'`, **deprecation awareness** (`X-XSS-Protection`, HPKP), **duplicate‑header avoidance**, **production verification** (`curl -I`), and **cookie‑flag alignment**. |
| **Beginner‑Friendly Explanation** | These are the **“do not ship without these”** rules for security headers. The biggest mistakes are: forgetting to add CSP at all (ASP.NET Core does not send it by default), using `'unsafe-inline'` and thinking you have XSS protection, enabling HSTS on a site that still needs HTTP, adding duplicate headers, and never verifying that the headers actually reach production. |

### Purposes

- **To** provide a **baseline set** that mitigates the most common browser‑based attacks.
- **To** ensure headers are **actually delivered** in production — not dropped by misordered middleware or a reverse proxy.
- **To** **roll out CSP safely** using Report‑Only mode and nonces.
- **To** **avoid deprecated headers** that provide no benefit and may introduce risk.
- **To** **align headers with cookie flags** (`Secure`, `HttpOnly`, `SameSite`) for complete browser‑side defense.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **CSP** | Start with `default-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'`. | Use `'unsafe-inline'` — it defeats most of CSP’s XSS value. |
| **CSP rollout** | Use **Report‑Only** mode first; monitor violations; tighten gradually. | Enable a strict CSP in production without testing — it will break legitimate scripts. |
| **Nonce** | Generate a **per‑request nonce** for inline scripts and styles. | Reuse a static nonce — it becomes a predictable allowlist. |
| **`X-Frame-Options`** | Use `DENY` unless same‑origin framing is required; pair with CSP `frame-ancestors`. | Use `ALLOW-FROM` — deprecated and unsupported. |
| **`X-Content-Type-Options`** | Always set `nosniff`. | Omit it — MIME‑sniffing attacks are preventable. |
| **`Referrer-Policy`** | Use `strict-origin-when-cross-origin` as the default. | Use `unsafe-url` — it leaks full URLs to third parties. |
| **`Permissions-Policy`** | Disable camera, microphone, geolocation, payment, and USB unless required. | Allow `*` for features you do not need. |
| **Deprecated headers** | Omit `X-XSS-Protection` and HPKP. | Add them “just in case” — they are obsolete and can cause harm. |
| **Duplicates** | Set each header once, in one middleware or proxy layer. | Let the app and proxy both set the same header with different values. |
| **Verification** | Verify with `curl -I https://your-app` in staging and production. | Assume the headers are present without checking. |
| **Pipeline order** | Register security header middleware **before** static files and routing. | Register it late — error pages and static files will miss the headers. |

### Annotated Code Example (Best‑Practice Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    var nonce = Convert.ToBase64String(
        RandomNumberGenerator.GetBytes(16));
    context.Items["CspNonce"] = nonce;

    var h = context.Response.Headers;
    h["Content-Security-Policy"] =
        $"default-src 'self'; " +
        $"script-src 'self' 'nonce-{nonce}'; " +
        $"style-src 'self' 'nonce-{nonce}'; " +
        "object-src 'none'; " +
        "frame-ancestors 'none'; " +
        "base-uri 'self'; " +
        "form-action 'self'; " +
        "img-src 'self' data:";
    h["X-Frame-Options"] = "DENY";
    h["X-Content-Type-Options"] = "nosniff";
    h["Referrer-Policy"] = "strict-origin-when-cross-origin";
    h["Permissions-Policy"] =
        "camera=(), microphone=(), geolocation=(), payment=(), usb=()";

    await next();
});

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **CSP is nonce‑based** — the strongest XSS defense.
- **`frame-ancestors 'none'` and `X-Frame-Options: DENY`** — consistent clickjacking protection.
- **`nosniff`** — MIME‑sniffing prevention on every response.
- **`strict-origin-when-cross-origin`** — modern referrer default.
- **Permissions‑Policy** disables unnecessary features.
- **The middleware is early** — all responses inherit the headers.
- **No deprecated headers** — `X-XSS-Protection` is omitted.

### Real-World Cases

- **Public web apps** — the full baseline set with nonce‑based CSP.
- **APIs** — `nosniff`, restrictive CSP for any HTML, and `Permissions-Policy`.
- **Reverse‑proxy deployments** — headers set at the proxy or app, but **not both** with conflicting values.

### References

- ASP.NET Core Security Checklist (headers and cookies) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security Headers for ASP.NET Core 10 on IIS (ASPnix) – https://www.aspnix.com/posts/security-headers-aspnet-core-10-on-iis
- OWASP WSTG — Security Header Misconfigurations – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/
- NetEscapades.AspNetCore.SecurityHeaders (defaults and customisation) – https://github.com/andrewlock/netescapades.aspnetcore.securityheaders

---

## Summary Table: Security Headers at a Glance

| Header | Primary Role | Recommended Baseline | When to Customize |
|--------|--------------|----------------------|-------------------|
| **CSP** | XSS and injection prevention | `default-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'` + nonce for scripts/styles | When the app uses CDNs, inline scripts, or WebAssembly |
| **`X-Frame-Options`** | Clickjacking protection (legacy) | `DENY` | `SAMEORIGIN` if the app frames itself |
| **`X-Content-Type-Options`** | MIME‑sniffing prevention | `nosniff` | Never — always use it |
| **`Referrer-Policy`** | Referrer leakage control | `strict-origin-when-cross-origin` | `no-referrer` for highly sensitive apps |
| **`Permissions-Policy`** | Browser feature restriction | `camera=(), microphone=(), geolocation=(), payment=(), usb=()` | Enable features only when the app requires them |
| **Best Practices** | Baseline and rollout | Report‑Only CSP first; nonce over `'unsafe-inline'`; verify in production | Every deployment |

---

## References (Full List)

- ASP.NET Core Security Checklist for Production (safeguard.sh) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security Headers for ASP.NET Core 10 on IIS (ASPnix) – https://www.aspnix.com/posts/security-headers-aspnet-core-10-on-iis
- NetCoreApplicationTemplate — security-headers.md (AsiBackbone) – https://github.com/AsiBackbone/NetCoreApplicationTemplate/blob/main/docs/articles/security-headers.md
- NetEscapades.AspNetCore.SecurityHeaders (GitHub) – https://github.com/andrewlock/netescapades.aspnetcore.securityheaders
- OWASP WSTG — Other HTTP Security Header Misconfigurations – https://wstg.owasp.org/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations/
- Implementing CSP with Kendo UI for ASP.NET Core (Telerik) – https://www.telerik.com/aspnet-mvc/documentation/knowledge-base/csp-compliance-kendo-ui-aspnet-core
- Permissions-Policy in ASP.NET Core middleware (LobeHub) – https://lobehub.com/skills/
- X-Frame-Options — DENY vs SAMEORIGIN (C# Corner) – https://www.c-sharpcorner.com/article/clickjacking-protection-with-x-frame-options-and-csp-in-asp-net-core/
- Referrer-Policy — strict-origin-when-cross-origin (Csharp Fix) – https://db.fluidattacks.com/
- ASP.NET Core antiforgery `SuppressXFrameOptionsHeader` (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery
- Content Security Policy for ASP.NET Core Blazor (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/blazor/security/content-security-policy