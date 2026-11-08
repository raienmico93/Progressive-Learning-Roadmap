# ASP.NET Core API Security: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the security controls that protect HTTP APIs. The threat landscape is defined by the **OWASP API Security Top 10 (2023)** — notably **Broken Object Level Authorization (BOLA)** as the #1 risk — and by **RFC 9700 (BCP 240)** for OAuth 2.0 security. ASP.NET Core provides strong security primitives, but many are **opt-in or environment-sensitive**; a service can work perfectly in a demo while leaking stack traces, accepting cleartext HTTP, or missing authorization checks. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API security is the practice of protecting HTTP APIs from unauthorized access, data exposure, abuse, and exploitation through layered controls: authentication, authorization, transport security, input validation, rate limiting, and observability. |
| **Technical Definition** | API security in ASP.NET Core spans **transport** (HTTPS, HSTS, forwarded headers), **identity** (JWT bearer, cookies, OIDC), **access control** (policies, fallback policies, resource‑based checks), **abuse prevention** (rate limiting, request size limits), **input safety** (DTO validation, mass‑assignment protection), **header hardening** (CSP, `X-Content-Type-Options`, `Referrer-Policy`), and **secret management** (Data Protection, secure configuration). The OWASP API Top 10 (2023) frames the threat model: **BOLA** (API1) is the most common API flaw — an endpoint authenticates the caller but never verifies that the caller owns the specific object being requested. |
| **Beginner‑Friendly Explanation** | API security is the **set of locks, cameras, and guards around your API**. Authentication checks **who** is calling. Authorization checks **what** they are allowed to do — including whether they are allowed to touch **this specific order**, not just whether they are logged in. Rate limiting prevents someone from hammering your API. HTTPS and HSTS protect data in transit. Security headers tell browsers how to behave. Input validation stops malicious payloads. None of these alone is enough — security is layered. |

### Key Characteristics

- **Default‑deny posture** – A **fallback authorization policy** requires authentication for all endpoints unless explicitly marked `[AllowAnonymous]`.
- **BOLA‑aware** – Every object‑returning endpoint must verify **object‑level ownership**, not just authentication.
- **Transport‑hardened** – HTTPS redirection and HSTS enforce encrypted, non‑downgradeable traffic.
- **Header‑hardened** – CSP, `X-Content-Type-Options`, `Referrer-Policy`, and `X-Frame-Options` mitigate XSS, clickjacking, and MIME sniffing.
- **Abuse‑resistant** – Built‑in rate limiting (since .NET 7) with partitioned policies per IP, user, or API key.
- **Standards‑aligned** – OAuth 2.0 implementations should follow **RFC 9700**: PKCE everywhere, exact redirect URI matching, short‑lived tokens, refresh rotation.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core Web API project.
- Working knowledge of **middleware**, **authentication**, **authorization**, and **HTTP**.
- Familiarity with **JWT**, **OAuth 2.0/OIDC**, and **claims‑based identity**.
- Basic understanding of **OWASP API Top 10** risks (especially BOLA).

### Related Programming Areas

- **Authentication fundamentals** – Schemes, handlers, and the `ClaimsPrincipal`.
- **Authorization** – Policies, fallback policies, and resource‑based checks.
- **API controllers / Minimal APIs** – The endpoints being protected.
- **Rate limiting middleware** – Abuse prevention.
- **Data Protection** – Encrypting cookies, tokens, and sensitive payloads.
- **Security headers** – Browser‑side defenses.

### Core Concepts / Features

1. **API security** – The overall layered model and OWASP API Top 10 framing.
2. **Authentication** – JWT bearer, OIDC, and secure token validation.
3. **Authorization** – Fallback policies, policy‑based access, and BOLA prevention.
4. **Rate limiting** – Built‑in middleware with partitioned policies.
5. **API security best practices** – Transport, headers, secrets, and lifecycle.

---

## 1. API Security (The Layered Model)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API security is the end‑to‑end protection of API endpoints, data, and infrastructure against unauthorized access, abuse, and exploitation. |
| **Technical Definition** | API security in ASP.NET Core is implemented through **middleware ordering** and **configuration**: transport security first (`UseHsts`, `UseHttpsRedirection`), then header hardening, then `UseCors`, `UseRateLimiter`, `UseAuthentication`, and `UseAuthorization` — in that order. Authentication establishes identity; authorization enforces access; rate limiting blunts brute force and scraping; security headers constrain browser behaviour; input validation and DTO binding prevent injection and mass assignment. |
| **Beginner‑Friendly Explanation** | API security is **not one setting — it is a stack of protections**. The outer layer is transport (HTTPS). Inside, headers tell browsers what to trust. Then rate limiting slows attackers down. Authentication checks who is calling. Authorization checks what they can access — including per‑object ownership. Finally, input validation ensures the data itself is safe. If any layer is missing, the others compensate only partially. |

### Purposes

- **To** prevent **unauthorized access** through authentication and authorization.
- **To** prevent **data exposure** through object‑level authorization checks and DTO‑based responses (BOLA prevention).
- **To** prevent **abuse** through rate limiting and request size limits.
- **To** protect **data in transit** through HTTPS, HSTS, and forwarded headers.
- **To** protect **browsers and clients** through security headers (CSP, `X-Content-Type-Options`).
- **To** fail **closed** — a forgotten `[Authorize]` should deny, not allow.

### Syntax Rules and Structure

#### The Security Middleware Pipeline (Canonical Order)

```text
Request → UseHsts (production) → UseHttpsRedirection → UseForwardedHeaders
       → UseCors → UseRateLimiter → UseAuthentication → UseAuthorization
       → Endpoints
```

**Component breakdown:**

| Order | Middleware | Responsibility |
|-------|------------|----------------|
| 1 | `UseHsts` | Adds `Strict-Transport-Security` header (production only). |
| 2 | `UseHttpsRedirection` | Redirects HTTP → HTTPS. |
| 3 | `UseForwardedHeaders` | Restores correct scheme/IP behind a proxy. |
| 4 | `UseCors` | Applies explicit CORS policies (never `AllowAnyOrigin` in production). |
| 5 | `UseRateLimiter` | Enforces rate‑limit policies. |
| 6 | `UseAuthentication` | Populates `HttpContext.User`. |
| 7 | `UseAuthorization` | Enforces fallback + endpoint policies. |

**Syntax rules:**

- **`UseAuthentication()` must precede `UseAuthorization()`** —authorization reads the authenticated user.
- **`UseAuthorization()` must sit between `UseRouting()` and endpoint mapping** (ASP0001).
- **`UseRateLimiter()` should run before authentication** so abusive requests are throttled before expensive auth work.
- **`UseCors()` must run before `UseAuthorization()`** and after `UseRouting()`.

**Constraints and limitations:**

- **Middleware order is security‑critical** —reversing authentication and authorization allows unauthenticated requests to bypass policies.
- **HSTS must not run in Development** —it breaks `localhost` HTTPS.
- **`AllowAnyOrigin()` is unsafe in production** —it disables CORS protection.

### Annotated Complete Step‑by‑Step Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);

// Services
builder.Services.AddControllers();
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* see Authentication section */ });
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
builder.Services.AddRateLimiter(options => { /* see Rate Limiting section */ });
builder.Services.AddCors(options =>
{
    options.AddPolicy("Spa", policy =>
        policy.WithOrigins("https://spa.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod());
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}
else
{
    app.UseHttpsRedirection();
}

app.UseForwardedHeaders();   // When behind a proxy
app.UseCors("Spa");
app.UseRateLimiter();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

**Expected Output:** Unauthenticated requests to any endpoint return `401` unless explicitly marked `[AllowAnonymous]`; HTTPS is enforced; abusive request rates are throttled; CORS is restricted to the configured origin.

**Why this output?** The fallback policy enforces authentication by default; the pipeline order ensures authentication precedes authorization; rate limiting and CORS are applied before endpoint execution.

### Real-World Cases

- **Public APIs:** Fallback policy + explicit `[AllowAnonymous]` on public endpoints; rate limiting per API key.
- **Internal microservices:** JWT bearer authentication with strict issuer/audience validation; rate limiting per client.
- **Multi‑tenant SaaS:** BOLA prevention via resource‑based authorization; tenant claims checked per object.

### References

- ASP.NET Core Security Checklist (2026) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- OWASP API Security Top 10 (2023) – https://owasp.org/API-Security/editions/2023/en/0x11-t10/

---

## 2. Authentication (JWT Bearer and OIDC)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authentication is the process of verifying a caller’s identity — typically by validating a JWT bearer token or an OIDC ID token. |
| **Technical Definition** | JWT bearer authentication is configured via `AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(...)`. `TokenValidationParameters` must explicitly enable **issuer validation**, **audience validation**, **lifetime validation**, and **signing‑key validation** — the framework does not enable all of these by default. `ClockSkew` defaults to 5 minutes and should be tightened (e.g., 1 minute). `MapInboundClaims = false` preserves original claim names (`sub`, `roles`) instead of legacy URI mappings. |
| **Beginner‑Friendly Explanation** | Authentication is the **ID check** at the door. A JWT is a digitally signed ID card. The API checks the signature (is the card real?), the issuer (who issued it?), the audience (is it meant for this API?), and the expiry (is it still valid?). If anything fails, the request is rejected with `401`. |

### Purposes

- **To** establish **who** the caller is before any business logic runs.
- **To** reject **tampered, expired, or wrongly targeted tokens** with `401`.
- **To** populate `HttpContext.User` for authorization policies.
- **To** support **stateless API authentication** without server‑side sessions.
- **To** enable **delegated authorization** through OAuth 2.0/OIDC access tokens.

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.RequireHttpsMetadata = true;   // false only in development
        options.MapInboundClaims = false;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
            ClockSkew = TimeSpan.FromMinutes(1)
        };
    });
```

**Component breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `ValidateIssuer` | Verifies the token was issued by the expected authority. |
| `ValidateAudience` | Verifies the token is intended for this API. |
| `ValidateLifetime` | Rejects expired tokens. |
| `ValidateIssuerSigningKey` | Verifies the signature. |
| `ClockSkew` | Tolerance for clock differences (tighten from 5 min default). |
| `MapInboundClaims = false` | Preserves original claim names. |

**Syntax rules:**

- **All four validation flags must be explicitly set to `true`** —some are not enabled by default.
- **Secrets must come from secure configuration** —never hard‑code signing keys.
- **OIDC** should use `ResponseType = Code` with PKCE; Implicit and ROPC are deprecated by RFC 9700.

**Constraints and limitations:**

- **`RequireHttpsMetadata = false`** is only acceptable in development; production must fetch metadata over HTTPS.
- **Bearer tokens are replayable if stolen** —short lifetimes and refresh token rotation mitigate the window.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.RequireHttpsMetadata = true;
        options.MapInboundClaims = false;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
            ClockSkew = TimeSpan.FromMinutes(1)
        };
    });
```

**Expected Output:** A request with a valid JWT and correct issuer/audience is authenticated; an expired token, wrong audience, or invalid signature returns `401 Unauthorized`.

**Why this output?** All validation parameters are explicitly enabled; the token is rejected if any check fails.

### Real-World Cases

- **Public APIs:** JWT bearer with issuer/audience validation.
- **SPA backends:** OIDC with authorization code + PKCE.
- **Microservices:** Service‑to‑service tokens with strict audience scoping.

### References

- ASP.NET Core JWT Authentication and Authorization – https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/csharp/web/aspnet-auth.md
- RFC 9700: Best Current Practice for OAuth 2.0 Security – https://datatracker.ietf.org/doc/rfc9700
- OAuth 2.0 Security Best Practices (2026) – https://safeguard.sh/resources/blog/oauth-2-security-best-practices

---

## 3. Authorization (Fallback Policies and BOLA Prevention)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authorization determines whether an authenticated caller is permitted to perform an action or access a specific resource. |
| **Technical Definition** | ASP.NET Core authorization is **policy‑based**: a policy is a set of requirements evaluated by handlers. A **fallback policy** (`AuthorizationOptions.FallbackPolicy`) applies to endpoints without explicit authorization metadata, providing **secure‑by‑default** behaviour. **BOLA (API1:2023)** occurs when an endpoint accepts an object ID but never verifies the caller owns that object — the fix is **resource‑based authorization** (`IAuthorizationService.AuthorizeAsync`) or per‑object checks, not just endpoint‑level `[Authorize]`. |
| **Beginner‑Friendly Explanation** | Authorization is the **permission check** after the ID check. A fallback policy says “everyone must be logged in unless I explicitly say otherwise.” But that is not enough: if `GET /orders/42` returns order 42 to any logged‑in user, that is **BOLA** — the API checked **who** you are but not **whether you own order 42**. Resource‑based authorization fixes this by comparing the caller’s identity to the resource’s owner or tenant. |

### Purposes

- **To** enforce **default‑deny** access — forgotten attributes fail closed, not open.
- **To** prevent **BOLA** — verify **object‑level ownership** for every ID‑based endpoint.
- **To** support **fine‑grained policies** (roles, claims, custom requirements).
- **To** prevent **mass assignment** — bind DTOs, never domain entities.
- **To** isolate **tenants** — verify the caller’s tenant claim matches the resource’s tenant.

### Syntax Rules and Structure

#### Fallback Policy (Global Default)

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `FallbackPolicy` | Applies to endpoints **without** explicit authorization metadata. |
| `RequireAuthenticatedUser()` | Requires a valid identity. |
| `[AllowAnonymous]` / `AllowAnonymous()` | Explicitly opts out for public endpoints. |

#### Policy‑Based Authorization

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
    options.AddPolicy("CanManageUsers", policy =>
        policy.RequireClaim("permission", "users.manage"));
});

// Apply
[Authorize(Policy = "AdminOnly")]
public IActionResult AdminOnly() => Ok();
```

#### BOLA Prevention (Resource‑Based)

```csharp
// In the action — verify ownership
var order = await _repo.FindAsync(id);
if (order is null) return NotFound();
if (order.CustomerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
    return Forbid();   // Authenticated but not the owner
return Ok(order);
```

**Syntax rules:**

- **Fallback policy must be set** for secure‑by‑default; otherwise, unannotated endpoints are public.
- **`RequireAuthorization()` without a policy name** uses the default policy, not the fallback policy.
- **Object‑level checks** must happen **after** the object is loaded — `[Authorize]` runs before model binding and cannot see the object.

**Constraints and limitations:**

- **Fallback policy does not apply to endpoints with any authorization metadata** —an endpoint with `[Authorize]` (even without a policy name) uses the default policy instead.
- **Minimal API `.AllowAnonymous()`** is the explicit opt‑out for public endpoints.
- **BOLA is a business‑logic flaw** —generic scanners rarely catch it; manual authorization matrices are required.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
});

app.MapControllers();

// Public endpoint
[AllowAnonymous]
[HttpGet("public")]
public IActionResult Public() => Ok("Anyone");

// Protected endpoint
[Authorize(Policy = "AdminOnly")]
[HttpGet("admin")]
public IActionResult Admin() => Ok("Admin only");

// BOLA-safe endpoint
[Authorize]
[HttpGet("{id}")]
public async Task<IActionResult> GetOrder(int id)
{
    var order = await _repo.FindAsync(id);
    if (order is null) return NotFound();
    if (order.CustomerId != User.FindFirstValue(ClaimTypes.NameIdentifier))
        return Forbid();
    return Ok(order);
}
```

**Expected Output:** Public endpoint allows anonymous; admin endpoint requires the Admin role; order endpoint requires authentication **and** ownership — a different authenticated user receives `403 Forbidden`.

**Why this output?** The fallback policy requires authentication by default; `[AllowAnonymous]` opts out; the per‑object check prevents BOLA.

### Real-World Cases

- **E‑commerce:** Customers can only access their own orders (BOLA prevention).
- **Multi‑tenant SaaS:** Tenant claims must match the resource’s tenant.
- **Admin APIs:** Role/policy‑based access with fallback authentication.

### References

- Policy-based authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Security blueprint (fallback policies, pipeline ordering) – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/0_security_blueprint.md
- OWASP API Security Top 10 (BOLA) – https://owasp.org/API-Security/editions/2023/en/0x11-t10/

---

## 4. Rate Limiting (Built‑in Middleware)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Rate limiting controls how many requests a client can make in a given time window, protecting the API from brute force, scraping, and resource exhaustion. |
| **Technical Definition** | ASP.NET Core provides **built‑in rate limiting middleware** (since .NET 7). `AddRateLimiter()` configures policies; `UseRateLimiter()` adds the middleware. **Partitioned rate limiters** divide traffic into buckets by IP address, user identity, API key, or endpoint path, so one client cannot exhaust the limit for everyone. Algorithms include **fixed window**, **sliding window**, **token bucket**, and **concurrency limiter** (capping in‑flight requests). |
| **Beginner‑Friendly Explanation** | Rate limiting is the **“too many requests”** guard. Without it, an attacker can hammer your login endpoint with password guesses or scrape your entire catalogue. With it, each client gets a budget — e.g., 100 requests per minute — and requests beyond that receive `429 Too Many Requests`. Partitioning ensures that one abusive client does not block everyone else. |

### Purposes

- **To** blunt **brute‑force attacks** on login and token endpoints.
- **To** prevent **scraping and resource exhaustion**.
- **To** enforce **tiered service limits** (free vs. premium API keys).
- **To** protect **expensive endpoints** (search, reports) from overload.
- **To** provide **fairness** through partitioned buckets.

### Syntax Rules and Structure

#### Global Fixed Window (Per IP)

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(
        httpContext => RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown",
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));
});

app.UseRateLimiter();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `GlobalLimiter` | Applies to all requests unless overridden. |
| `PartitionedRateLimiter` | Creates per‑key buckets. |
| `GetFixedWindowLimiter` | Fixed‑window algorithm. |
| `PermitLimit` | Requests allowed per window. |
| `Window` | Duration of the window. |

#### Named Policy (Per Endpoint)

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", opt =>
    {
        opt.PermitLimit = 50;
        opt.Window = TimeSpan.FromSeconds(10);
    });
});

app.MapGet("/api/search", () => "Results")
   .RequireRateLimiting("fixed");
```

**Syntax rules:**

- **`AddRateLimiter()` registers policies**; `UseRateLimiter()` adds the middleware.
- **Named policies are applied via `.RequireRateLimiting("name")`** or `[EnableRateLimiting("name")]`.
- **Partition keys** can be IP, user identity, API key, or endpoint path.

**Constraints and limitations:**

- **Rate limiting is per‑instance** —distributed deployments need a distributed store (e.g., Redis) or a gateway‑level limiter.
- **In‑memory counters reset on restart** —plan for this in ephemeral containers.
- **Too‑aggressive limits can block legitimate clients** —tune based on monitoring.

### Annotated Complete Step‑by‑Step Code Example

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(
        httpContext => RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: httpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown",
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1)
            }));

    options.AddFixedWindowLimiter("auth", opt =>
    {
        opt.PermitLimit = 10;
        opt.Window = TimeSpan.FromMinutes(1);
    });
});

app.UseRateLimiter();

app.MapPost("/api/auth/login", () => "Login")
   .RequireRateLimiting("auth");
```

**Expected Output:** Each IP gets 100 requests/minute globally; the login endpoint gets 10 requests/minute per IP. Excess requests receive `429 Too Many Requests`.

**Why this output?** The global limiter partitions by IP; the named policy applies a stricter limit to the sensitive login endpoint.

### Real-World Cases

- **Login endpoints:** Strict limits to prevent credential stuffing.
- **Public APIs:** Per‑API‑key limits with higher quotas for premium tiers.
- **Search/report endpoints:** Concurrency limits to protect backend databases.

### References

- Rate limiting middleware in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit
- Rate limiting algorithms (fixed, sliding, token bucket, concurrency) – https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit#algorithms

---

## 5. API Security Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | API security best practices are the baseline controls that every production ASP.NET Core API should have — transport security, header hardening, secure authentication/authorization, rate limiting, input validation, and secret management. |
| **Technical Definition** | Best practices encompass **HTTPS + HSTS** (with forwarded headers behind proxies), **security headers** (CSP, `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`), **fallback authorization policies**, **BOLA prevention** via resource‑based checks, **rate limiting**, **DTO‑based input validation and mass‑assignment protection**, **secure secret storage** (no secrets in source/config), and **Data Protection** for cookies/tokens. |
| **Beginner‑Friendly Explanation** | Best practices are the **“do not ship without these”** list. Every API needs HTTPS, security headers, default‑deny authorization, rate limiting, input validation, and secrets stored safely. A service that works perfectly in a demo but lacks these is not ready for production. |

### Purposes

- **To** prevent **data breaches** through layered defenses.
- **To** avoid **shipping vulnerabilities** that are opt‑in or easily forgotten.
- **To** comply with **security standards** and regulatory requirements.
- **To** reduce **attack surface** through headers, rate limits, and validation.
- **To** fail **closed** — forgotten annotations deny access.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Transport** | `UseHsts()` + `UseHttpsRedirection()` outside Development; `UseForwardedHeaders()` behind proxies. | Allow HTTP in production; enable HSTS in Development. |
| **Headers** | CSP (nonce‑based), `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`, `X-Frame-Options: DENY`. | Rely on defaults — ASP.NET Core does not send CSP by default. |
| **Authorization** | Set `FallbackPolicy` to `RequireAuthenticatedUser()`; mark public endpoints `[AllowAnonymous]`. | Leave endpoints unannotated — they become public. |
| **BOLA** | Verify object ownership on every ID‑based endpoint (`order.CustomerId == user.Id`). | Assume authentication is enough — it is not. |
| **Rate limiting** | Apply global + per‑endpoint limits; strict limits on auth and mutation endpoints. | Leave login endpoints unlimited. |
| **Input validation** | Use DTOs with `[Required]`, `[StringLength]`, `[Range]`; check `ModelState.IsValid`. | Bind domain entities — mass assignment risk. |
| **Secrets** | Load from Azure Key Vault / User Secrets; never hard‑code. | Commit secrets to source or `appsettings.json`. |
| **Data Protection** | Use `IDataProtectionProvider` for cookies/tokens; persist keys in shared storage. | Rely on ephemeral local keys in web farms. |
| **CORS** | Explicit `WithOrigins("https://app.example.com")`. | `AllowAnyOrigin()` in production. |
| **Error responses** | `ProblemDetails` without stack traces in production. | Leak exception details. |

### Annotated Code Example (Best‑Practice Security Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// HTTPS + HSTS
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
    options.Preload = true;
});

// Authentication (JWT)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* validation parameters */ });

// Authorization (fallback deny)
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Rate limiting
builder.Services.AddRateLimiter(options => { /* global + named policies */ });

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}
else
{
    app.UseHttpsRedirection();
}

// Security headers middleware
app.Use(async (ctx, next) =>
{
    var h = ctx.Response.Headers;
    h["Content-Security-Policy"] = "default-src 'self'; frame-ancestors 'none'";
    h["X-Content-Type-Options"] = "nosniff";
    h["Referrer-Policy"] = "no-referrer";
    await next();
});

app.UseForwardedHeaders();
app.UseRateLimiter();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **HTTPS + HSTS** enforce encrypted, non‑downgradeable transport.
- **Security headers** mitigate XSS, clickjacking, and MIME sniffing.
- **Fallback policy** makes authorization default‑deny.
- **Rate limiting** blunts brute force and abuse.
- **Secrets** should be loaded from secure providers (not shown here, but required).

### Real-World Cases

- **Public APIs:** All eight baseline controls; BOLA checks per object.
- **Internal microservices:** JWT + fallback policy + rate limiting; mTLS optional.
- **Regulated industries:** HSTS preload, CSP, audit logging, secure secret storage.

### References

- ASP.NET Core Security Best Practices 2026 – https://offensive360.com/blog/aspnet-core-security-best-practices/
- Security blueprint (ASP.NET Core 10) – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/0_security_blueprint.md
- OWASP API Security Top 10 (2023) – https://owasp.org/API-Security/editions/2023/en/0x11-t10/
- RFC 9700 OAuth 2.0 Security BCP – https://datatracker.ietf.org/doc/rfc9700

---

## Summary Table: API Security Controls at a Glance

| Control | Primary Role | Key API / Configuration | When to Use |
|---------|--------------|--------------------------|-------------|
| **Authentication** | Verify identity | `AddJwtBearer`, `TokenValidationParameters` | Every protected API |
| **Authorization** | Enforce access | `FallbackPolicy`, `[Authorize]`, resource‑based checks | Every protected API |
| **Rate limiting** | Prevent abuse | `AddRateLimiter`, `UseRateLimiter`, partitioned policies | Every public/exposed API |
| **HTTPS + HSTS** | Protect transport | `UseHsts`, `UseHttpsRedirection`, `UseForwardedHeaders` | Every production API |
| **Security headers** | Browser defenses | CSP, `X-Content-Type-Options`, `Referrer-Policy` | Every API serving browsers |
| **BOLA prevention** | Object‑level ownership | Resource‑based checks after loading the object | Every ID‑based endpoint |
| **Input validation** | Prevent injection/mass assignment | DTOs + `[Required]`, `ModelState.IsValid` | Every endpoint accepting input |
| **Secrets management** | Protect keys/passwords | Azure Key Vault, User Secrets | Every production deployment |

---

## References (Full List)

- ASP.NET Core Security Checklist (2026) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- ASP.NET Core Security Best Practices 2026 (Offensive360) – https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP API Security Top 10 (2023) – https://owasp.org/API-Security/editions/2023/en/0x11-t10/
- OWASP API Security Top 10: A Testing Primer – https://safeguard.sh/resources/blog/api-security-risks-fundamentals
- RFC 9700: Best Current Practice for OAuth 2.0 Security – https://datatracker.ietf.org/doc/rfc9700
- OAuth 2.0 Security Best Practices (2026): RFC 9700 Guide – https://safeguard.sh/resources/blog/oauth-2-security-best-practices
- Rate limiting middleware in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit
- Policy-based authorization in ASP.NET Core (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Security blueprint (ASP.NET Core 10) – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/0_security_blueprint.md
- ASP.NET Core JWT Authentication and Authorization – https://raw.githubusercontent.com/glennguilloux/llm-knowledge-base/refs/heads/main/csharp/web/aspnet-auth.md
- ASP.NET Core security basics (Tessl Labs) – https://tessl.io/registry/tessl-labs/aspnet-security-basics