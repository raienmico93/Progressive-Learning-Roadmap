# ASP.NET Core Security Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the **foundational security principles** that underpin every secure application: threat modeling, attack surface management, defense in depth, least privilege, and secure defaults. Security is not a feature you add at the end — it is a **property of the architecture** that must be designed in from the first line of code. The framework provides excellent primitives, but many are **opt‑in or environment‑sensitive**; a service can work perfectly in a demo while leaking stack traces, accepting cleartext HTTP, or missing authorization checks . The threat landscape is framed by the **OWASP API Security Top 10** and **STRIDE**, and the guidance here aligns with Microsoft's official security documentation and the ASP.NET Core security checklist. Each requested topic is treated independently — no sections are merged.


## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security is the discipline of protecting an application's **confidentiality**, **integrity**, and **availability** against threats by applying layered controls, minimizing exposed surface, granting only necessary permissions, and defaulting to safe configurations. |
| **Technical Definition** | ASP.NET Core security spans **transport** (HTTPS, HSTS, forwarded headers), **identity** (authentication, authorization, policies), **input safety** (validation, encoding, parameterization), **browser defenses** (CSP, `X-Content-Type-Options`, `SameSite` cookies), **secret management** (Secret Manager, Key Vault, Data Protection), and **operational hygiene** (rate limiting, dependency scanning, logging). The **STRIDE** threat model (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) provides a structured way to enumerate threats at each **trust boundary** . |
| **Beginner‑Friendly Explanation** | Security is the **set of locks, cameras, and guards around your application**. It is not one setting — it is a stack of protections. **Threat modeling** is figuring out who might attack and how before you write code. **Attack surface** is everything an attacker can touch — every endpoint, input field, and configuration knob. **Defense in depth** means layering controls so that if one fails, another catches it. **Least privilege** means giving each user, service, and process only the minimum access it needs. **Secure defaults** mean the application is safe **unless** you explicitly choose otherwise. |

### Key Characteristics

- **Layered** — No single control is sufficient; security is defense in depth .
- **Default‑deny** — A fallback authorization policy requires authentication for all endpoints unless explicitly marked `[AllowAnonymous]` .
- **Least‑privilege** — Users, services, database accounts, and managed identities receive only the permissions required for their function .
- **Fail‑closed** — When authorization is unavailable or ambiguous, access is denied, not granted .
- **Threat‑model driven** — STRIDE analysis at each trust boundary identifies threats before code is written .
- **Surface‑minimized** — Every endpoint, health probe, and auxiliary feature is an attack surface that must be justified and protected .

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **authentication**, **authorization**, and **dependency injection**.
- Familiarity with **HTTP**, **TLS**, **cookies**, and **claims-based identity**.
- Basic understanding of **OWASP Top 10** and **STRIDE**.

### Related Programming Areas

- **Authentication and authorization** — The identity and access control layers.
- **Middleware pipeline** — Transport security, headers, and CSRF protection are pipeline-level concerns.
- **Data protection** — Encrypting cookies, tokens, and sensitive payloads.
- **Dependency management** — Vulnerability scanning and package hygiene.
- **Logging and monitoring** — Detecting attacks and anomalies.

### Core Concepts / Features

1. **Security** — The overarching discipline and the CIA triad.
2. **Threat modeling** — STRIDE per trust boundary.
3. **Attack surface** — What is exposed, and how to minimize it.
4. **Defense in depth** — Layered controls.
5. **Least privilege** — Minimum access for every identity.
6. **Secure defaults** — Fail‑closed, opt‑in configuration.
7. **Security best practices** — The checklist that ties it all together.


## 1. Security (The Discipline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security is the practice of protecting an application and its data from unauthorized access, modification, or disruption by applying layered, principle‑driven controls throughout the software lifecycle. |
| **Technical Definition** | In ASP.NET Core, security is implemented through **middleware** (HTTPS redirection, HSTS, forwarded headers, CSRF, CORS, rate limiting), **authentication schemes** (cookies, JWT, OIDC), **authorization policies** (fallback policies, resource‑based checks), **data protection** (key management, encryption), and **input/output safety** (model validation, HTML encoding, parameterized queries). The discipline is **continuous** — from threat modeling at design time to dependency scanning in CI. |
| **Beginner‑Friendly Explanation** | Security is **everything you do to keep bad things from happening**. It starts before you write code (threat modeling), continues through design (least privilege, defense in depth), and is enforced by the framework (authentication, authorization, HTTPS). It is never “done” — new vulnerabilities appear, dependencies age, and configurations drift. Security is a **process**, not a checkbox. |

### Purposes

- **To** protect **confidentiality** — prevent unauthorized data disclosure.
- **To** preserve **integrity** — prevent unauthorized modification of data or code.
- **To** maintain **availability** — prevent denial‑of‑service and resource exhaustion.
- **To** **fail closed** — deny access when security decisions cannot be made.
- **To** **minimize the impact** of a breach through layered controls and least privilege.

### Syntax Rules and Structure

#### The Security Lifecycle

```text
Threat model → Design controls → Implement → Test → Deploy → Monitor → Iterate
```

**Component breakdown:**

| Phase | Activity | ASP.NET Core tooling |
|-------|----------|----------------------|
| **Threat model** | Identify assets, trust boundaries, and STRIDE threats. | STRIDE, OWASP Threat Dragon |
| **Design** | Apply least privilege, defense in depth, secure defaults. | Architecture review |
| **Implement** | Authentication, authorization, HTTPS, CSRF, headers. | Middleware, Identity, policies |
| **Test** | Security tests, penetration testing, SAST/DAST. | OWASP ZAP, Burp Suite |
| **Monitor** | Detect anomalies and attacks. | Logging, metrics, alerts |

**Syntax rules:**

- **Security must be designed in, not bolted on** —retrofitting security is expensive and error‑prone.
- **Every layer must fail closed** —an unavailable authorization decision must deny, not allow .
- **Secrets must never live in source code** —use Secret Manager in development and a production secret store .

**Constraints and limitations:**

- **The framework cannot fix a broken design** —if an endpoint lacks authorization, no middleware will save it.
- **Environment sensitivity** —HTTPS redirection, HSTS, and detailed errors behave differently in Development vs. Production.

### Annotated Complete Step‑by‑Step Code Example (Security Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Secure defaults: fallback policy denies by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// 2. HTTPS + HSTS outside Development
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}

// 3. Security headers
app.Use(async (ctx, next) =>
{
    var h = ctx.Response.Headers;
    h["Content-Security-Policy"] = "default-src 'self'; frame-ancestors 'none'";
    h["X-Content-Type-Options"] = "nosniff";
    h["Referrer-Policy"] = "no-referrer";
    await next();
});

app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Step‑by‑step explanation:**

1. The fallback policy requires authentication for all endpoints without explicit authorization metadata — forgotten `[Authorize]` attributes fail closed.
2. HSTS and HTTPS redirection enforce encrypted, non‑downgradeable transport outside Development.
3. Security headers mitigate XSS, clickjacking, and MIME sniffing.
4. Authentication precedes authorization in the pipeline.

**Expected Output:** Every endpoint requires authentication unless explicitly marked `[AllowAnonymous]`; HTTPS is enforced in production; security headers are present on every response.

**Why this output?** The fallback policy implements default‑deny; the middleware pipeline enforces transport and browser defenses.

### Real-World Cases

- **Public APIs:** Fallback policy + explicit `[AllowAnonymous]` on public endpoints; HTTPS + HSTS.
- **Enterprise apps:** Defense in depth across transport, identity, input, and data layers.
- **Regulated industries:** Continuous monitoring, audit logging, and least‑privilege database accounts.

### References

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- ASP.NET Core Security Checklist for Production – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security And Identity (GitHub) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md


## 2. Threat Modeling (STRIDE per Trust Boundary)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Threat modeling is the disciplined enumeration of what an attacker can send through each trust boundary, what they can learn from the response, and what mitigations the system applies. |
| **Technical Definition** | Threat modeling uses **STRIDE** — **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege — applied **per trust boundary** (HTTP request boundary, deserialization boundary, database boundary, outbound HTTP boundary). For each boundary, the team identifies concrete threats, the .NET mitigation, the OWASP API item it maps to, and the test that proves the mitigation works . |
| **Beginner‑Friendly Explanation** | Threat modeling is **“thinking like an attacker before writing code.”** You draw the system, mark the boundaries where untrusted data enters (HTTP requests, file uploads, deserialization), and for each boundary ask: “Could someone pretend to be someone else? Tamper with data? Repudiate an action? Steal information? Crash the system? Escalate privileges?” Then you write down how you will stop each one. |

### Purposes

- **To** identify threats **before** they become vulnerabilities in code.
- **To** prioritize security work by **risk** — not every threat is equally likely or impactful.
- **To** provide a **repeatable, documented method** (STRIDE per boundary) that lives in the repository, not a one‑off consultant engagement .
- **To** map threats to **concrete mitigations** (.NET features, OWASP controls) and **tests** that prove they work.
- **To** satisfy **compliance** requirements for secure design documentation.

### Syntax Rules and Structure

#### STRIDE Categories

| Category | Threat | Example | .NET Mitigation |
|----------|--------|---------|-----------------|
| **Spoofing** | Impersonating a user or service | Stolen JWT, session hijacking | Strong authentication, short token lifetimes, `HttpOnly` cookies |
| **Tampering** | Modifying data in transit or at rest | Parameter tampering in URL | Model validation, parameterized queries, TLS |
| **Repudiation** | Denying an action was performed | “I didn’t place that order” | Audit logging with tamper‑evident storage |
| **Information disclosure** | Leaking sensitive data | Stack traces, PII in logs | `ProblemDetails` without stack traces, structured logging redaction |
| **Denial of service** | Exhausting resources | Request flooding | Rate limiting, request size limits, timeouts |
| **Elevation of privilege** | Gaining unauthorized access | Missing `[Authorize]`, BOLA | Fallback policy, resource‑based authorization |

#### The Threat Modeling Artifact

```text
THREATMODEL.md lives in the repository:
- Trust boundaries (HTTP, deserialization, database, outbound HTTP)
- STRIDE table per boundary
- Mitigation per threat
- Test that proves the mitigation
```

**Syntax rules:**

- **Apply STRIDE per trust boundary**, not per endpoint — boundaries are where trust changes.
- **Every threat must have a mitigation and a test** —a threat without a test is an assumption.
- **Keep the threat model in the repository** —it evolves with the code, unlike a one‑time document.

**Constraints and limitations:**

- **Threat modeling is not a substitute for security testing** —it identifies threats; tests and scans prove mitigations work.
- **STRIDE is a thinking tool, not a checklist** —the goal is systematic coverage, not filling in all six categories mechanically.

### Annotated Complete Step‑by‑Step Code Example (STRIDE‑Driven Mitigation)

```csharp
// Threat: Spoofing (stolen JWT)
// Mitigation: short-lived tokens + refresh rotation
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ClockSkew = TimeSpan.FromSeconds(30)   // tight clock skew
        };
    });

// Threat: Elevation of privilege (missing [Authorize])
// Mitigation: fallback policy denies by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Threat: Information disclosure (stack traces)
// Mitigation: ProblemDetails without exception details
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
app.UseStatusCodePages();
```

**Step‑by‑step explanation:**

1. Tight token validation mitigates **Spoofing** through stolen or forged tokens.
2. The fallback policy mitigates **Elevation of privilege** by denying unannotated endpoints.
3. `ProblemDetails` mitigates **Information disclosure** by returning generic error shapes instead of stack traces.

**Expected Output:** A threat model document that maps each STRIDE category to a concrete mitigation and a test; the code implements the mitigations.

**Why this output?** Threat modeling turns abstract risks into specific, testable controls.

### Real-World Cases

- **Financial systems:** STRIDE per boundary for payment flows, with tamper‑evident audit logs for Repudiation.
- **Multi‑tenant SaaS:** Threat modeling the tenant boundary to prevent cross‑tenant data access (BOLA).
- **API gateways:** Threat modeling the outbound HTTP boundary to prevent SSRF and information disclosure.

### References

- STRIDE per boundary threat modeling (GitHub) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/…/06-homework.md
- C# Security Code Review (trust boundaries) – https://github.com/…/security-code-review.md
- OWASP Threat Modeling Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html


## 3. Attack Surface (What Is Exposed)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | The attack surface is the sum of all points where an attacker can interact with the application — every endpoint, input field, protocol, service, and configuration knob. |
| **Technical Definition** | In ASP.NET Core, the attack surface includes **HTTP endpoints** (controllers, minimal APIs, Razor Pages, SignalR hubs, gRPC services), **input vectors** (query strings, headers, cookies, request bodies, file uploads), **authentication mechanisms**, **auxiliary features** (health checks, metrics, Swagger UI, developer exception pages), and **infrastructure** (open ports, TLS configuration, reverse proxy). Minimizing surface means removing unnecessary endpoints, disabling unused features in production, and gating diagnostic surfaces. |
| **Beginner‑Friendly Explanation** | The attack surface is **everything an attacker can touch**. Every endpoint you expose is a door. Every input field is a window. Every health check, Swagger UI, and debug endpoint is another opening. The smaller the surface, the fewer ways in. You reduce it by removing what you do not need, locking what you must keep, and never exposing diagnostics in production. |

### Purposes

- **To** **reduce the number of entry points** an attacker can probe.
- **To** **eliminate unnecessary features** —Swagger UI, health checks, and developer exception pages in production.
- **To** **gate auxiliary endpoints** behind authentication or IP allowlists.
- **To** **minimize exposed metadata** —remove server headers, disable detailed errors.
- **To** **focus security effort** on the endpoints that genuinely need to be public.

### Syntax Rules and Structure

#### Attack Surface Reduction Checklist

| Surface | Risk | Mitigation |
|---------|------|------------|
| **Swagger UI** | Full API schema exposed | Gate to Development/Staging; `[Authorize]` in other environments |
| **Developer exception page** | Stack traces, source code | Development only |
| **Health checks** | Probe endpoint, potential info leak | Minimal response; restrict to internal network |
| **Server header** | Version disclosure | `AddServerHeader = false` |
| **Unused endpoints** | Forgotten attack surface | Remove or disable |
| **`MapIdentityApi`** | Adds all Identity endpoints, not configurable | Replace with custom login/refresh endpoints  |

**Syntax rules:**

- **Every endpoint must be justified** —if it is not needed, remove it.
- **Diagnostic surfaces must be environment‑gated** —Swagger UI, exception pages, and detailed errors belong in Development.
- **Auxiliary endpoints (health, metrics) should return minimal information** and be restricted to internal networks when possible.

**Constraints and limitations:**

- **Surface reduction is not a substitute for authorization** —a minimal surface still needs authentication and authorization on what remains.
- **Some surface is unavoidable** —the public API is the attack surface; the goal is to make it as small and well‑defended as possible.

### Annotated Complete Step‑by‑Step Code Example (Surface Reduction)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Disable the Server header
builder.WebHost.ConfigureKestrel(options =>
{
    options.AddServerHeader = false;
});

var app = builder.Build();

// Developer exception page only in Development
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

// Health check returns minimal information
app.MapHealthChecks("/health").AllowAnonymous();

app.Run();
```

**Step-by-step explanation:**

1. `AddServerHeader = false` removes the Kestrel server version header.
2. The developer exception page and Swagger UI are gated to Development.
3. The health check is minimal and anonymous (internal network restriction is an infrastructure concern).

**Expected Output:** Production responses do not leak server version, stack traces, or API documentation; only the public API and a minimal health probe are exposed.

**Why this output?** Each surface is either removed or gated; what remains is intentional and minimal.

### Real-World Cases

- **Public APIs:** Swagger UI removed in production; developer exception page never enabled.
- **Internal microservices:** Health checks restricted to the internal network; no Swagger UI.
- **Legacy migrations:** Audit all endpoints and remove abandoned ones.

### References

- Minimal API `MapIdentityApi` surface concern – https://stackoverflow.com/questions/…/disable-access-to-aspnet-core-8-web-api-identity-methods
- Routing and endpoint groups (`MapGroup`) – https://learn.microsoft.com/cs-cz/aspnet/core/fundamentals/routing
- Server header hardening – https://github.com/jposluns/secureconfig/…

---

## 4. Defense in Depth (Layered Controls)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Defense in depth is the practice of layering multiple independent security controls so that if one fails, another prevents or detects the attack. |
| **Technical Definition** | In ASP.NET Core, defense in depth spans **network** (TLS, firewalls, private endpoints), **transport** (HTTPS, HSTS, forwarded headers), **browser** (CSP, `X-Content-Type-Options`, `SameSite`), **identity** (authentication, authorization, fallback policy), **input** (model validation, parameterization, encoding), **data** (encryption at rest, Data Protection), and **operations** (rate limiting, logging, monitoring). No single layer is trusted to be perfect; each assumes the others may fail . |
| **Beginner‑Friendly Explanation** | Defense in depth is the **“castle with a moat, a wall, a gate, and guards”** approach. If the moat fails, the wall stops the attacker. If the wall is breached, the gate slows them down. If the gate is breached, the guards catch them. In software: HTTPS protects data in transit; authentication checks identity; authorization checks permissions; input validation blocks injection; CSP blocks XSS. If one layer misses something, another catches it. |

### Purposes

- **To** **prevent single points of failure** in security —no single control is sufficient.
- **To** **detect and contain** attacks that bypass outer layers.
- **To** **increase attacker cost** —each layer adds time and effort.
- **To** **protect against both known and unknown threats** —different layers target different attack classes.
- **To** **satisfy compliance** requirements for layered security.

### Syntax Rules and Structure

#### The Layers (Request Path)

```text
Network → TLS → HSTS → Forwarded Headers → CORS → Rate Limiter → Authentication → Authorization → Input Validation → Output Encoding
```

**Component breakdown:**

| Layer | Control | Failure Mitigation |
|-------|---------|-------------------|
| **Network** | Firewall, private endpoint | TLS prevents eavesdropping |
| **Transport** | HTTPS, HSTS | Browser refuses HTTP after first visit |
| **Browser** | CSP, `X-Content-Type-Options` | XSS payload blocked |
| **Identity** | Authentication, fallback policy | Unauthorized request denied |
| **Input** | Validation, parameterization | Injection blocked |
| **Data** | Encryption at rest | Stolen backup unreadable |
| **Operations** | Rate limiting, logging | Brute force throttled, attack logged |

**Syntax rules:**

- **Every layer must be independently configured** —relying on one layer to compensate for another’s absence is fragile.
- **Fail closed** —when a layer cannot make a decision, deny access .
- **Defense in depth includes detection** —logging and monitoring are layers too.

**Constraints and limitations:**

- **Layers add complexity** —too many overlapping controls can create maintenance burden; each layer must justify its cost.
- **Defense in depth is not an excuse for missing authorization** —the authorization layer must still be correct.

### Annotated Complete Step‑by‑Step Code Example (Layered Pipeline)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* validation parameters */ });
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser().Build();
});
builder.Services.AddRateLimiter(options => { /* policies */ });

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();                      // Layer 2: browser forces HTTPS
    app.UseHttpsRedirection();          // Layer 1: transport
}

app.UseForwardedHeaders();              // Layer 0: correct scheme behind proxy
app.UseRateLimiter();                   // Layer 3: abuse prevention
app.UseAuthentication();                // Layer 4: identity
app.UseAuthorization();                 // Layer 5: permissions
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. Forwarded headers restore the correct scheme behind a proxy.
2. HSTS and HTTPS redirection enforce encrypted transport.
3. Rate limiting blunts brute force and scraping.
4. Authentication identifies the caller.
5. Authorization enforces access control.

**Expected Output:** A request passes through each layer in order; failure at any layer stops the request before the next.

**Why this output?** Each layer independently reduces risk; an attacker must bypass all of them to succeed.

### Real-World Cases

- **Financial APIs:** TLS + HSTS + JWT + fallback policy + rate limiting + CSP + parameterized queries.
- **Multi‑tenant SaaS:** Tenant isolation at the authorization layer + encryption at rest + audit logging.
- **E‑commerce checkout:** HTTPS + antiforgery + PCI‑compliant payment tokenization + fraud detection.

### References

- Security in .NET: Designing Across Every Layer (LinkedIn) – https://www.linkedin.com/posts/gopikumar_chandran_security-in-net-designing-across-every-layer
- Defense-in-depth file upload pipeline (NuGet) – https://www.nuget.org/packages?q=defense-in-depth
- ASP.NET Core Security Checklist (layered controls) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist


## 5. Least Privilege (Minimum Access)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Least privilege is the principle that every user, service, process, and database account should receive only the minimum permissions necessary to perform its function — nothing more. |
| **Technical Definition** | In ASP.NET Core, least privilege applies at every identity layer: **users** receive the minimum roles/claims required; **application services** use scoped identities with only the necessary scopes; **database accounts** use roles like `db_datareader` and `db_datawriter` instead of `db_owner` ; **managed identities** eliminate credentials entirely and receive precise RBAC roles ; **containers** run as non‑root users without privilege escalation . |
| **Beginner‑Friendly Explanation** | Least privilege is **“give everyone the key to their own room, not the master key.”** The application’s database account should not be able to drop tables — it only needs to read and write data. A user who needs to view reports should not have admin access. A container should not run as root. The principle limits the **blast radius** of a compromised identity: if an attacker steals a low‑privilege credential, they cannot do much damage. |

### Purposes

- **To** **limit the blast radius** of a compromised identity —stolen credentials grant minimal access.
- **To** **prevent accidental damage** —a misconfigured service cannot drop production tables.
- **To** **satisfy compliance** requirements (PCI DSS, SOC 2, least‑privilege mandates).
- **To** **reduce the attack surface of identities** —fewer permissions means fewer exploitable paths.
- **To** **enable accountability** —each identity has a narrow, auditable purpose.

### Syntax Rules and Structure

#### Least‑Privilege Checklist

| Identity | Over‑privileged (avoid) | Least privilege (use) |
|----------|-------------------------|------------------------|
| **Database account** | `db_owner`, `sa` | `db_datareader`, `db_datawriter`; `db_ddladmin` only for migrations  |
| **Azure service auth** | Connection strings with passwords | **Managed identity** with precise RBAC roles  |
| **User roles** | `Admin` for everyone | Minimum role/claim per function |
| **Container** | Root user | Non‑root user, `allowPrivilegeEscalation: false`  |
| **API scopes** | `*` or broad scopes | Minimum scopes required |

**Syntax rules:**

- **The application database account must never be `sa` or `db_owner`** —use `db_datareader` and `db_datawriter` for the runtime account; a separate migration account may have `db_ddladmin` .
- **Prefer managed identities over secrets** —they eliminate credential storage and can be scoped precisely .
- **Validate resource ownership** —an ID in the URL is not enough; check that the user owns the resource (BOLA prevention) .

**Constraints and limitations:**

- **Least privilege requires knowing what “minimum” is** —start restrictive and grant permissions as evidence demands.
- **Overly restrictive permissions break functionality** —balance security with operability.

### Annotated Complete Step‑by‑Step Code Example (Least‑Privilege Database Access)

```csharp
// ❌ Over-privileged: the app uses sa or db_owner
// "Server=...;Database=...;User Id=sa;Password=..."

// ✅ Least privilege: a dedicated user with only data permissions
// "Server=...;Database=...;User Id=app_user;Password=..."

// SQL: create a least-privilege database user
// CREATE USER app_user FROM LOGIN app_user;
// ALTER ROLE db_datareader ADD MEMBER app_user;
// ALTER ROLE db_datawriter ADD MEMBER app_user;
// -- Do NOT add to db_owner or db_ddladmin for the runtime account

// ✅ Azure SQL with Managed Identity (no password)
// "Server=myserver.database.windows.net;Database=MyDb;
//  Authentication=Active Directory Managed Identity"
```

**Step-by-step explanation:**

1. The runtime database account is a dedicated user with only `db_datareader` and `db_datawriter`.
2. Migrations use a separate account with `db_ddladmin` —the runtime account cannot alter schema.
3. Managed identity eliminates the password entirely and receives only the RBAC roles it needs.

**Expected Output:** A compromised application account cannot drop tables, alter schema, or access other databases.

**Why this output?** The runtime account has only the permissions required to read and write application data.

### Real-World Cases

- **Azure SQL + App Service:** Managed identity with `db_datareader`/`db_datawriter`; no connection‑string password .
- **Multi‑tenant SaaS:** Tenant‑scoped roles and claims; the app account cannot cross tenant boundaries.
- **Containerised workloads:** Non‑root user, read‑only filesystem, no privilege escalation .

### References

- Connection Security (least‑privilege database roles) – https://github.com/…/connection-security.md
- Azure Managed Identity least‑privilege database roles – https://www.nuget.org/packages/Wicksipedia.Aspire.Hosting.Azure.SqlRoles
- Code Checklist (container least privilege) – https://docs.altinn.studio/…/code-checklist


## 6. Secure Defaults (Fail Closed, Opt In)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Secure defaults are the principle that the application’s default configuration should be the most secure option — the developer must explicitly opt **out** of security, not opt in. |
| **Technical Definition** | In ASP.NET Core, secure defaults include the **fallback authorization policy** requiring authentication for all endpoints without explicit metadata, **HTTPS redirection and HSTS** outside Development, **antiforgery** enabled for cookie‑based apps, **detailed errors disabled** in production, **cookies with `Secure`/`HttpOnly`/`SameSite`**, and **secret management** that requires explicit configuration rather than defaulting to insecure values. The default posture is **closed** — public endpoints must be explicitly marked `[AllowAnonymous]` . |
| **Beginner‑Friendly Explanation** | Secure defaults mean **“the door is locked unless you unlock it”** —not the other way around. If a developer forgets to add `[Authorize]`, the endpoint should deny access, not allow everyone. If a production deployment forgets to disable the developer exception page, the default should be that it is already off. Secure defaults protect against human error, which is the most common cause of vulnerabilities. |

### Purposes

- **To** **protect against forgotten security controls** —a missing `[Authorize]` fails closed, not open.
- **To** **reduce the attack surface by default** —diagnostic endpoints, detailed errors, and Swagger UI are off unless explicitly enabled.
- **To** **enforce secure transport** —HTTPS and HSTS are on by default outside Development.
- **To** **eliminate insecure fallback values** —secrets and connection strings do not default to weak or empty values .
- **To** **make the secure path the easy path** —developers do not need to remember to enable security.

### Syntax Rules and Structure

#### The Secure‑by‑Default Posture

| Control | Insecure default (avoid) | Secure default (use) |
|---------|--------------------------|----------------------|
| **Authorization** | No fallback policy (endpoints public) | Fallback policy requires authentication  |
| **HTTPS** | No redirection | `UseHsts` + `UseHttpsRedirection` outside Development |
| **Errors** | Detailed exception page everywhere | Developer page only in Development  |
| **Cookies** | No `Secure`/`HttpOnly`/`SameSite` | All three attributes set  |
| **Antiforgery** | Disabled or forgotten | Enabled for cookie‑based apps  |
| **Secrets** | Plain‑text config | Secret Manager (dev) / vault (prod)  |

**Syntax rules:**

- **The fallback policy is the single most important secure default** —it ensures every endpoint is protected unless explicitly opened.
- **Public endpoints must be marked `[AllowAnonymous]` deliberately** —not by omission.
- **Detailed errors and Swagger UI must be gated by environment** —never enabled in production by default.

**Constraints and limitations:**

- **Secure defaults can break existing functionality** —an application that relied on public endpoints will suddenly deny access; this is the intended behaviour.
- **The fallback policy does not apply to endpoints with any authorization metadata** —an endpoint with `[Authorize]` (even without a policy) uses the default policy, not the fallback.

### Annotated Complete Step‑by‑Step Code Example (Secure‑by‑Default Configuration)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Secure default #1: deny by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Secure default #2: secure cookies
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.HttpOnly = true;
    options.Cookie.SameSite = SameSiteMode.Strict;
});

// Secure default #3: antiforgery
builder.Services.AddAntiforgery();

var app = builder.Build();

// Secure default #4: HTTPS/HSTS outside Development
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}

app.UseAuthentication();
app.UseAuthorization();
app.UseAntiforgery();
app.MapControllers();
app.Run();
```

**Step-by-step explanation:**

1. The fallback policy requires authentication for all endpoints without explicit metadata.
2. Cookies are `Secure`, `HttpOnly`, and `SameSite=Strict` by default.
3. Antiforgery is registered and enabled for state‑changing requests.
4. HTTPS and HSTS are enforced outside Development.

**Expected Output:** A new endpoint added without `[AllowAnonymous]` is automatically protected; cookies are hardened; state‑changing requests require antiforgery tokens.

**Why this output?** Secure defaults make the secure path the default; developers must explicitly opt out.

### Real-World Cases

- **Public APIs:** Fallback policy + explicit `[AllowAnonymous]` on truly public endpoints.
- **MVC/Razor Pages apps:** Secure cookies + antiforgery by default.
- **Microservices:** HTTPS/HSTS by default; health checks explicitly anonymous.

### References

- Secure-by-Default ASP.NET Core Configuration (GitHub) – https://github.com/AsiBackbone/Learning/blob/…/secure-by-default-configuration.md
- Should Authorization Fail Open, Fail Closed, or Defer? (GitHub) – https://github.com/AsiBackbone/Learning/blob/…/fail-open-fail-closed.md
- ASP.NET Core Security Checklist (fallback policy) – https://safeguard.sh/resources/blog/aspnet-core-security-checklist


## 7. Security Best Practices (The Baseline Checklist)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Security best practices are the concrete, proven controls that every ASP.NET Core application should implement before reaching production. |
| **Technical Definition** | The baseline covers **transport** (HTTPS, HSTS, forwarded headers), **headers** (CSP, `X-Content-Type-Options`, `Referrer-Policy`), **authentication** (strongest available flow, short token lifetimes, secure cookies), **authorization** (fallback policy, resource‑based checks, `[AllowAnonymous]` sparingly), **CSRF/antiforgery**, **secrets** (Secret Manager, vault, no source‑code secrets), **Data Protection** (persisted keys in multi‑instance deployments), **rate limiting**, and **dependency hygiene** . |
| **Beginner‑Friendly Explanation** | These are the **“do not ship without these”** items —the security baseline that every production app must have. They are not advanced or exotic; they are the fundamentals that prevent the most common and most damaging vulnerabilities. |

### Purposes

- **To** prevent **the most common vulnerabilities** —missing authorization, insecure transport, leaked secrets.
- **To** provide a **reviewable checklist** for code reviews and pre‑deployment gates.
- **To** align with **Microsoft’s official guidance** and industry standards (OWASP, NIST).
- **To** **fail closed** and **default secure** across every layer.
- **To** **monitor and detect** attacks through logging and rate limiting.

### Guidelines Checklist

| Area | Do | Don’t |
|------|----|-------|
| **Transport** | `UseHsts()` + `UseHttpsRedirection()` outside Development; `UseForwardedHeaders()` behind proxies . | Allow HTTP in production; enable HSTS in Development. |
| **Headers** | CSP (nonce‑based), `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer` . | Rely on defaults —ASP.NET Core does not send CSP by default. |
| **Authorization** | Fallback policy requiring authentication; `[AllowAnonymous]` explicitly and sparingly . | Leave endpoints unannotated —they become public. |
| **BOLA** | Verify object ownership on every ID‑based endpoint . | Assume authentication is enough —it is not. |
| **CSRF** | `UseAntiforgery()`; validate tokens on unsafe methods . | Disable antiforgery globally; opt out per endpoint only when justified. |
| **Cookies** | `Secure`, `HttpOnly`, `SameSite=Strict` . | Store auth cookies accessible to JavaScript. |
| **Secrets** | Secret Manager (dev), production vault; never in source . | Commit secrets to source control. |
| **Data Protection** | Persist keys outside ephemeral storage for multi‑instance deployments . | Rely on local ephemeral keys in web farms. |
| **Rate limiting** | Apply global + per‑endpoint limits . | Leave login endpoints unlimited. |
| **Monitoring** | Structured logging with redaction; alert on anomalies. | Log sensitive data (tokens, PII). |

### Annotated Code Example (Best‑Practice Security Baseline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Authentication (strongest flow available)
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* strict validation */ });

// Authorization (default deny)
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser().Build();
});

// Antiforgery + secure cookies + rate limiting
builder.Services.AddAntiforgery();
builder.Services.AddRateLimiter(options => { /* policies */ });

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

// Security headers
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
app.UseAntiforgery();
app.MapControllers();
app.Run();
```

**Why this follows best practices:**

- **Fallback policy** makes authorization default‑deny.
- **HTTPS/HSTS** enforce encrypted, non‑downgradeable transport.
- **Security headers** mitigate XSS and clickjacking.
- **Antiforgery** protects state‑changing requests.
- **Rate limiting** blunts abuse.
- **Secrets** are loaded from configuration (backed by a vault in production).

### Real-World Cases

- **Public APIs:** All baseline controls; BOLA checks per object.
- **Internal microservices:** JWT + fallback policy + rate limiting; mTLS optional.
- **Regulated industries:** HSTS preload, CSP, audit logging, least‑privilege database accounts.

### References

- ASP.NET Core Security Checklist for Production – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security And Identity (GitHub) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/


## Summary Table: Security Fundamentals at a Glance

| Concept | Primary Role | Key Tool / Practice | When to Use |
|---------|--------------|---------------------|-------------|
| **Security** | Protect confidentiality, integrity, availability | Layered controls, fail closed, continuous monitoring | Every application, every layer |
| **Threat modeling** | Identify threats before coding | STRIDE per trust boundary, documented in `THREATMODEL.md` | Design phase and every significant change |
| **Attack surface** | Minimize what attackers can touch | Remove unused endpoints, gate diagnostics, disable server header | Every production deployment |
| **Defense in depth** | Layer independent controls | TLS + HSTS + CSP + authN/authZ + validation + rate limiting | Every request path |
| **Least privilege** | Minimum access for every identity | `db_datareader`/`db_datawriter`, managed identities, non‑root containers | Every user, service, and database account |
| **Secure defaults** | Safe unless explicitly changed | Fallback policy, HTTPS by default, secure cookies, no detailed errors | Every new endpoint and deployment |
| **Best practices** | The baseline checklist | HTTPS, CSP, fallback policy, antiforgery, secrets vault, rate limiting | Every production release |

---

## References (Full List)

- ASP.NET Core security topics (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/
- ASP.NET Core Security Checklist for Production – https://safeguard.sh/resources/blog/aspnet-core-security-checklist
- Security And Identity (GitHub) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- Security And Identity (alternate version) – https://github.com/midudev/autoskills/blob/2dbab03b41988327e5c714d3fb233ebbce915e8c/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- STRIDE per boundary threat modeling (GitHub) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/…/06-homework.md
- C# Security Code Review (trust boundaries) – https://github.com/…/security-code-review.md
- OWASP Threat Modeling Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- Secure-by-Default ASP.NET Core Configuration (GitHub) – https://github.com/AsiBackbone/Learning/blob/…/secure-by-default-configuration.md
- Should Authorization Fail Open, Fail Closed, or Defer? (GitHub) – https://github.com/AsiBackbone/Learning/blob/…/fail-open-fail-closed.md
- Connection Security (least‑privilege database roles) – https://github.com/…/connection-security.md
- Azure Managed Identity least‑privilege database roles – https://www.nuget.org/packages/Wicksipedia.Aspire.Hosting.Azure.SqlRoles
- Code Checklist (container least privilege) – https://docs.altinn.studio/…/code-checklist
- Defense-in-depth file upload pipeline (NuGet) – https://www.nuget.org/packages?q=defense-in-depth
- Security in .NET: Designing Across Every Layer (LinkedIn) – https://www.linkedin.com/posts/gopikumar_chandran_security-in-net-designing-across-every-layer
- Minimal API `MapIdentityApi` surface concern – https://stackoverflow.com/questions/…/disable-access-to-aspnet-core-8-web-api-identity-methods
- Routing and endpoint groups (`MapGroup`) – https://learn.microsoft.com/cs-cz/aspnet/core/fundamentals/routing
- ASP.NET Core 10 security overview (Tencent Cloud) – https://cloud.tencent.cn/developer/article/2487064