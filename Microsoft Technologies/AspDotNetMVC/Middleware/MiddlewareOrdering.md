# ASP.NET Core Middleware Ordering: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the minimal hosting model (`WebApplication`). Middleware ordering is the single most consequential design decision in the request pipeline: the order in which middleware are added defines request-processing order (forward) and response-processing order (reverse). Ordering bugs have **no compiler signal**, compile cleanly, and are often **security-relevant** (authorization bypass, information disclosure). Only one ordering rule is enforced by an analyzer today (ASP0001 for `UseAuthorization` placement); all other rules rely on developer knowledge. This cheat sheet treats each requested topic independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware ordering is the sequence in which middleware components are registered in the pipeline, which determines the order in which they process requests and (in reverse) responses. |
| **Technical Definition** | Middleware are invoked in the order they are added in `Program.cs` for requests, and in **reverse order** for responses. Each middleware wraps the next, forming a nested chain (`middleware1(middleware2(middleware3(terminal)))`). The order is critical for security, performance, and functionality: `UseAuthentication` must precede `UseAuthorization`; `UseCors` must precede `UseAuthorization`; `UseExceptionHandler` must be first to catch downstream exceptions; `UseAuthorization` must sit between `UseRouting` and `UseEndpoints` to be endpoint-aware. |
| **Beginner-Friendly Explanation** | Middleware order is **who checks you first**. If authorization runs before authentication, the system doesn’t yet know who you are, so authorization cannot work. If CORS runs after authorization, browser preflight requests are rejected before CORS headers are added. The order is not arbitrary—it follows the logical flow of inspecting, identifying, and then acting on a request. |

### Key Characteristics

- **Bidirectional** – Requests flow forward; responses flow backward through the same chain.
- **Order-sensitive** – Reordering two lines can silently break security or functionality.
- **Largely unenforced** – Only `UseAuthorization` placement is checked by analyzer ASP0001; other rules depend on developer knowledge.
- **Scenario-dependent** – Some orderings (caching vs. compression) are valid in multiple configurations; the choice depends on goals.
- **Auto-insertion in minimal hosting** – `WebApplication` auto-inserts `UseRouting`, `UseAuthentication`, and `UseAuthorization` if not called explicitly and the required services are detected.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware pipelines**, **authentication/authorization**, and **routing**.
- Familiarity with `Program.cs` (minimal hosting).

### Related Programming Areas

- **Authentication and authorization** – The most security-critical ordering pair.
- **CORS** – Must be placed between routing and authorization.
- **Exception handling** – Must wrap the entire downstream pipeline.
- **Endpoint routing** – `UseRouting` selects the endpoint; `UseAuthorization` and `UseCors` depend on it.
- **Response caching and compression** – Order determines whether compressed responses are cached, among other trade-offs.

### Core Concepts / Features

1. **Middleware ordering** – The concept and the canonical sequence.
2. **Order importance** – Why order matters (security, functionality, performance) with concrete traps.
3. **Order best practices** – Guidelines, canonical shape, and known pitfalls to avoid.

---

## 1. Middleware Ordering

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Middleware ordering is the arrangement of middleware components in a specific sequence within the request pipeline. |
| **Technical Definition** | The middleware pipeline is a nested chain: each middleware receives a `RequestDelegate next` pointing to the rest of the pipeline. Requests traverse the chain from the outermost (first registered) to the innermost (terminal), and responses traverse in reverse. The order is defined by the sequence of `app.Use...()` calls in `Program.cs`. In minimal hosting, `WebApplication` inserts routing, authentication, and authorization automatically if they are not explicitly called, but explicit calls give full control over placement. |
| **Beginner-Friendly Explanation** | Middleware ordering is **the sequence of checkpoints** a request passes through. The first checkpoint you add is the outermost layer; the last checkpoint is closest to your application code. When the response comes back, it passes through the checkpoints in the opposite order. |

### Purposes

- **To** ensure security features (authentication, authorization, CORS) execute in the correct logical sequence.
- **To** guarantee that exception handling wraps the widest possible scope of the pipeline.
- **To** allow performance optimisations (static files, caching, compression) to work as intended.
- **To** ensure endpoint-aware middleware can access routing metadata.
- **To** prevent silent security regressions that compile without errors.

### Syntax Rules and Structure

#### Canonical Order (Recommended Shape)

```csharp
var app = builder.Build();

// 1. Exception handling (first to catch all downstream errors)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
else
{
    app.UseDeveloperExceptionPage();
}

// 2. HTTPS redirection
app.UseHttpsRedirection();

// 3. Static files (short-circuits file requests)
app.UseStaticFiles();

// 4. Routing (selects the endpoint)
app.UseRouting();

// 5. CORS (after routing, before auth)
app.UseCors();

// 6. Authentication (identifies the user)
app.UseAuthentication();

// 7. Authorization (checks permissions; must be between UseRouting and endpoints)
app.UseAuthorization();

// 8. Session / endpoint-specific middleware
app.UseSession();

// 9. Endpoints (terminal execution)
app.MapControllers();
app.MapRazorPages();
```

**Component breakdown:**

| Position | Middleware | Reason |
|----------|------------|--------|
| 1 | Exception handler / HSTS | Must catch exceptions from all downstream middleware; HSTS adds security headers early. |
| 2 | HTTPS redirection | Redirects HTTP to HTTPS before sensitive processing. |
| 3 | Static files | Short-circuits file requests before routing/auth overhead. |
| 4 | Routing | Selects the endpoint; must precede `UseAuthorization` and `UseCors`. |
| 5 | CORS | Must be after `UseRouting` and **before** `UseAuthorization`. |
| 6 | Authentication | Identifies the user; must precede `UseAuthorization`. |
| 7 | Authorization | Checks permissions; must be **between** `UseRouting` and endpoints (ASP0001). |
| 8 | Session | After routing/cookie policy, before endpoints. |
| 9 | Endpoints | Terminal execution of controllers/pages. |

**Syntax rules:**

- `UseAuthentication()` must precede `UseAuthorization()`.
- `UseCors()` must be called after `UseRouting()` and before `UseAuthorization()`.
- `UseAuthorization()` should appear between `UseRouting()` and `UseEndpoints()` to be endpoint-aware.
- `UseExceptionHandler()` must be first so it catches exceptions from all downstream middleware.

**Constraints and limitations:**

- **Not every middleware appears in the canonical list** – output caching, rate limiting, and compression require scenario-specific placement.
- **Minimal hosting auto-insertion** can mask ordering issues; explicit calls are clearer.
- **Some orderings are valid in multiple configurations** (e.g., caching and compression) and depend on the desired trade-off.

### Annotated Code Example (Minimal API with Explicit Ordering)

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();
builder.Services.AddCors();
builder.Services.AddControllers();

var app = builder.Build();

app.UseExceptionHandler("/error");      // 1. Catch all downstream exceptions
app.UseHsts();                          // 2. Security header (production)
app.UseHttpsRedirection();              // 3. Redirect HTTP → HTTPS
app.UseStaticFiles();                   // 4. Short-circuit static files
app.UseRouting();                       // 5. Select endpoint
app.UseCors();                          // 6. CORS before auth
app.UseAuthentication();                // 7. Identify user
app.UseAuthorization();                 // 8. Check permissions
app.MapControllers();                   // 9. Execute endpoint

app.Run();
```

**Expected Output:** Requests are authenticated, authorized, and routed correctly; static files are served before any auth overhead; exceptions from any downstream middleware are caught by the handler.

**Why this output?** Each middleware executes in the intended logical order, with CORS before auth and auth before authorization.

### Real-World Cases

- **Web APIs with JWT:** `UseAuthentication` → `UseAuthorization` ensures tokens are validated before policies are evaluated.
- **SPAs with CORS:** `UseCors` between routing and auth prevents preflight failures.
- **Public sites with static assets:** `UseStaticFiles` before routing avoids unnecessary authentication work for CSS/JS requests.

### References

- ASP.NET Core Middleware (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Middleware order (canonical list) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order

---

## 2. Order Importance

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Order importance is the degree to which the placement of a middleware affects the correctness, security, and performance of the application. |
| **Technical Definition** | Certain middleware have **mandatory relative ordering** enforced by framework behaviour or analyzers: `UseAuthentication` before `UseAuthorization`; `UseCors` after `UseRouting` and before `UseAuthorization`; `UseAuthorization` between `UseRouting` and `UseEndpoints` (ASP0001); `UseExceptionHandler` before any middleware that can throw. Other orderings are **trap-prone**: output caching before authentication can cache authenticated responses and replay them to other users; exception handlers placed after authorization re-execute requests without re-running authorization, potentially bypassing policies. |
| **Beginner-Friendly Explanation** | Order importance is **how badly things break if you get the sequence wrong**. Some mistakes cause obvious errors (CORS preflight fails), while others are silent and dangerous (one user’s authenticated data cached and served to another user). The most dangerous ordering bugs are the ones that compile cleanly and produce no error. |

### Purposes

- **To** understand which orderings are security-critical versus merely functional.
- **To** prioritise review and testing of high-impact orderings (auth, CORS, exception handling).
- **To** avoid silent security regressions that are difficult to detect in testing.
- **To** make informed trade-offs for scenario-dependent orderings (caching, compression).

### Syntax Rules and Structure (Mandatory vs. Trap-Prone Orderings)

| Ordering | Status | Consequence of Violation |
|----------|--------|--------------------------|
| `UseAuthentication` before `UseAuthorization` | **Mandatory** | Authorization sees unauthenticated users; protected endpoints may be incorrectly allowed or denied. |
| `UseCors` after `UseRouting`, before `UseAuthorization` | **Mandatory** | Preflight requests rejected by authorization before CORS headers are added; CORS configuration silently ineffective. |
| `UseAuthorization` between `UseRouting` and `UseEndpoints` | **Enforced by ASP0001** | Authorization not endpoint-aware; fallback policy applied to all requests, or policies ignored. |
| `UseExceptionHandler` first | **Mandatory** | Exceptions from middleware registered before the handler are not caught; error pages may not work. |
| `UseOutputCache` **after** auth for authenticated apps | **Trap** | Authenticated responses cached and replayed to unauthenticated users (information disclosure). |
| `UseExceptionHandler` **before** `UseRouting`/auth | **Trap (re-execution)** | If placed after auth, re-executed error requests do not re-run authorization for the error endpoint, potentially bypassing `[Authorize]`. |
| `UseResponseCompression` before `UseStaticFiles` | **Performance** | Static files are not compressed if compression runs after static files. |

### Annotated Code Example (Trap: Output Caching Before Authentication)

```csharp
// ❌ DANGEROUS: Output cache before authentication
app.UseRouting();
app.UseOutputCache();      // Runs before auth
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output (dangerous):** An authenticated user’s response is cached and may be replayed to unauthenticated users, leaking private data.

**Why this output?** The default output-cache policy checks `HttpContext.User?.Identity?.IsAuthenticated` and the `Authorization` header. When `UseOutputCache` runs before `UseAuthentication`, `IsAuthenticated` is still `false`, and cookie authentication sets no `Authorization` header—both guards pass, so the response is cached without considering the user’s identity.

**Correct order:**

```csharp
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.UseOutputCache();      // After auth for authenticated apps
```

### Real-World Cases

- **Financial dashboards:** Output caching before auth can expose one user’s account data to another.
- **Multi-tenant SaaS:** CORS misordering causes preflight failures for tenant-specific domains.
- **APIs with custom error pages:** Exception handlers after auth can bypass `[Authorize]` on error endpoints during re-execution.

### References

- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Known middleware-ordering traps (output caching, diagnostics re-execution) – https://github.com/dotnet/aspnetcore/issues/67268
- Enable cross-origin requests (CORS) – https://learn.microsoft.com/en-us/aspnet/core/security/cors

---

## 3. Order Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Order best practices are guidelines for arranging middleware to maximise security, correctness, and clarity while avoiding known traps. |
| **Technical Definition** | Best practices encompass following the canonical shape unless there is a concrete reason to deviate, placing error/diagnostics middleware early (before routing/auth), keeping proxy/header processing before auth and redirects, avoiding random custom middleware between auth and authorization, and documenting intentional deviations. The canonical shape should be the default; deviations require justification and testing. |
| **Beginner-Friendly Explanation** | Order best practices are the **“default arrangement”** you should use unless you have a specific reason to change it. Start with the recommended shape, understand **why** each piece is where it is, and only move things when you can explain the trade-off. |

### Purposes

- **To** provide a safe, well-understood default pipeline shape.
- **To** prevent security regressions from accidental reordering.
- **To** make pipelines predictable and maintainable across teams.
- **To** ensure error handling, auth, and CORS work correctly without extensive debugging.
- **To** guide intentional deviations (e.g., proxy headers, rate limiting) to the correct positions.

### Guidelines Checklist (Canonical Shape with Deviations)

| Position | Middleware | Notes |
|----------|------------|-------|
| 1 | Forwarded headers (if behind proxy) | Must run before auth, redirects, and link generation. |
| 2 | Exception handling / HSTS (non-Development) | Error handling first; HSTS for production HTTPS. |
| 3 | HTTPS redirection | Before static files and routing. |
| 4 | Static files | Short-circuit file requests. |
| 5 | Routing (when explicit) | Required for endpoint-aware middleware. |
| 6 | CORS (when endpoints require it) | After routing, before auth. |
| 7 | Authentication | Before authorization. |
| 8 | Authorization | After auth, between routing and endpoints. |
| 9 | Endpoint-specific middleware | Rate limiting, session, output caching (after auth for authenticated apps). |
| 10 | Endpoint mapping | `MapControllers`, `MapRazorPages`, etc.. |

**Additional ordering rules:**

- **Call `UseAuthentication()` before `UseAuthorization()`** — this is the single most important ordering rule.
- **Keep proxy/header processing before auth, redirects, and link generation**.
- **Do not insert custom middleware randomly between auth and authorization** without a concrete reason.
- **Register error/diagnostics middleware early** (before routing/auth) so re-execution re-runs authorization for the error endpoint.
- **Place output caching after authentication/authorization** for authenticated apps to prevent caching private responses.

### Annotated Code Example (Best-Practice Pipeline with Deviations)

```csharp
var app = builder.Build();

// 1. Proxy headers (if behind load balancer)
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

// 2. Error handling (first to catch all downstream errors)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
else
{
    app.UseDeveloperExceptionPage();
}

// 3. HTTPS redirection
app.UseHttpsRedirection();

// 4. Static files (short-circuit)
app.UseStaticFiles();

// 5. Routing
app.UseRouting();

// 6. CORS (before auth)
app.UseCors();

// 7. Authentication
app.UseAuthentication();

// 8. Authorization (between routing and endpoints)
app.UseAuthorization();

// 9. Output caching (after auth for authenticated apps)
app.UseOutputCache();

// 10. Endpoint mapping
app.MapControllers();
app.MapRazorPages();

app.Run();
```

**Why this follows best practices:**

- **Forwarded headers are first** so downstream middleware see the correct scheme, host, and client IP.
- **Error handling is early** so re-executed error requests pass through authorization.
- **CORS is before authentication** so preflight requests are not blocked.
- **Authentication precedes authorization** so the user is identified before policies are evaluated.
- **Output caching is after auth** so authenticated responses are not cached and replayed to other users.
- **Custom middleware is not inserted randomly** between auth and authorization.

### Real-World Cases

- **Behind a reverse proxy (Nginx, Azure Front Door):** `UseForwardedHeaders` must run first so `UseHttpsRedirection` and auth see the correct scheme.
- **Authenticated APIs with output caching:** Output caching must be after auth to avoid cross-user data leakage.
- **Multi-tenant apps with CORS:** CORS must be between routing and auth; moving it later breaks preflight requests.

### References

- Program And Pipeline (middleware order guidance) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/program-and-pipeline.md
- ASP.NET Core middleware order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Known middleware-ordering traps – https://github.com/dotnet/aspnetcore/issues/67268
- Enable cross-origin requests (CORS) – https://learn.microsoft.com/en-us/aspnet/core/security/cors

---

## Summary Table: Middleware Ordering at a Glance

| Concept | Primary Role | Key Insight | When to Use |
|---------|--------------|-------------|-------------|
| **Middleware ordering** | The sequence of middleware in the pipeline | Requests flow forward; responses flow backward | Every ASP.NET Core app |
| **Order importance** | Why sequence matters | Auth before authorization; CORS before auth; error handling first; output caching after auth | When reviewing or debugging pipelines |
| **Order best practices** | Safe default shape and deviations | Start with canonical shape; deviate only with a concrete reason; document deviations | Every production pipeline |

---

## References (Full List)

- ASP.NET Core Middleware (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- Middleware order (canonical list) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- ASP0001: Authorization middleware is incorrectly configured – https://learn.microsoft.com/en-us/aspnet/core/diagnostics/asp0001
- Enable cross-origin requests (CORS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cors
- Known middleware-ordering traps (output caching, diagnostics re-execution) – https://github.com/dotnet/aspnetcore/issues/67268
- Program And Pipeline (middleware order guidance) – https://github.com/midudev/autoskills/blob/077b6ea6d747b3f392314bc47e7bb5285f1159b1/packages/autoskills/skills-registry/aspnet-core/references/program-and-pipeline.md
- Minimal APIs quick reference (auto-insertion of auth middleware) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- `UseAuthentication` and `UseAuthorization` ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order