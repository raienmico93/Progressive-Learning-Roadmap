# ASP.NET Core Built-in Middleware: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and its built-in middleware components. Examples use the minimal hosting model (`WebApplication`). Several middleware have mandatory ordering requirements for security and functionality. The `UseWelcomePage()` middleware is included for completeness but is **not recommended for production** and has limited support in modern hosting models. Each requested middleware is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Built-in middleware are pre-packaged components provided by ASP.NET Core that handle common cross-cutting concerns in the request pipeline. |
| **Technical Definition** | Built-in middleware are extension methods on `IApplicationBuilder` (or `WebApplication`) that add specific middleware components to the pipeline. Each component encapsulates a well-defined responsibility—routing, authentication, authorization, static file serving, error handling, compression, localization, and more—and is designed to be composed in a specific order. The order in which middleware are added defines the request processing order (and reverse order for responses) and is critical for security, performance, and functionality. |
| **Beginner-Friendly Explanation** | Built-in middleware are **ready-made processing stations** that ASP.NET Core provides so you don’t have to build common features from scratch. Need to serve static files? Add `UseStaticFiles()`. Need to check if users are logged in? Add `UseAuthentication()`. Each station has a job, and the order you arrange them in determines how requests flow through your app. |

### Key Characteristics

- **Purpose-specific** – Each middleware solves one well-defined problem (routing, auth, files, errors, etc.).
- **Order-sensitive** – Incorrect ordering can break security (auth bypass), functionality (CORS failures), or performance (unnecessary work).
- **Composable** – Middleware can be combined freely, as long as ordering constraints are respected.
- **Configurable** – Most middleware accept options objects for customization.
- **Environment-aware** – Some middleware (e.g., `UseExceptionHandler`, `UseHsts`) should only be used outside Development.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Basic understanding of the **middleware pipeline** and **`Program.cs`**.
- Familiarity with **HTTP** concepts (headers, cookies, status codes).

### Related Programming Areas

- **Middleware pipeline** – The chain through which requests and responses flow.
- **Routing** – `UseRouting` and `UseEndpoints` are the backbone of endpoint execution.
- **Authentication and authorization** – `UseAuthentication` and `UseAuthorization` enforce identity and permissions.
- **Error handling** – `UseExceptionHandler` and `UseStatusCodePages` manage failures.
- **Performance** – `UseResponseCompression` and `UseStaticFiles` optimise response delivery.

### Core Concepts / Features

1. **`UseRouting()`** – Matches requests to endpoints.
2. **`UseEndpoints()`** – Executes the selected endpoint.
3. **`UseAuthentication()`** – Identifies the user.
4. **`UseAuthorization()`** – Enforces permissions.
5. **`UseStaticFiles()`** – Serves static files and short-circuits.
6. **`UseCors()`** – Configures Cross-Origin Resource Sharing.
7. **`UseHttpsRedirection()`** – Redirects HTTP to HTTPS.
8. **`UseHsts()`** – Sends HTTP Strict Transport Security headers.
9. **`UseSession()`** – Establishes and maintains session state.
10. **`UseCookiePolicy()`** – Enforces cookie consent and security policies.
11. **`UseExceptionHandler()`** – Catches unhandled exceptions.
12. **`UseStatusCodePages()`** – Generates responses for error status codes without bodies.
13. **`UseResponseCompression()`** – Compresses responses (Gzip/Brotli).
14. **`UseRequestLocalization()`** – Sets culture based on request.
15. **`UseWelcomePage()`** – Serves a welcome page (not for production).
16. **Built-in middleware best practices** – Ordering, security, and performance guidelines.

---

## 1. `UseRouting()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseRouting()` adds middleware that matches an incoming HTTP request’s URL to an endpoint (controller action, Razor Page, minimal API handler, etc.). |
| **Technical Definition** | `UseRouting()` (in `Microsoft.AspNetCore.Builder`) adds the routing middleware to the pipeline. It parses the request path, matches it against registered endpoint data sources, and stores the selected `Endpoint` in `HttpContext`. Middleware added **between** `UseRouting` and `UseEndpoints` can inspect the selected endpoint via `HttpContext.GetEndpoint()` and use its metadata. In modern `WebApplication` hosting, `UseRouting` is added automatically if endpoints are configured, but apps can call it explicitly to control ordering. |
| **Beginner-Friendly Explanation** | `UseRouting()` is the **receptionist** that looks at the URL and decides which piece of code should handle the request. It doesn’t execute the code—it just figures out where the request should go. |

### Purposes

- **To** determine which endpoint (controller action, Razor Page, etc.) should handle each request.
- **To** populate endpoint metadata so downstream middleware (authorization, CORS, etc.) can make informed decisions.
- **To** enable URL generation and link building.
- **To** support attribute routing, conventional routing, and minimal API routing in a unified pipeline.

### Syntax Rules and Structure

```csharp
app.UseRouting();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseRouting()` | Adds routing middleware; typically called after static files and before auth/CORS. |

**Syntax rules:**

- Must be called **before** `UseEndpoints()` (and before endpoint-aware middleware such as `UseAuthorization`).
- In `WebApplication`, `UseRouting` is inserted automatically if endpoints are configured, but explicit calls are allowed for ordering control.
- `UseRouting` does **not** execute the endpoint—it only selects it.

**Constraints and limitations:**

- `UseRouting` alone does nothing observable unless `UseEndpoints` (or equivalent endpoint mapping) is also present.
- Middleware placed **before** `UseRouting` cannot access endpoint metadata.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();

app.UseStaticFiles();          // 1. Static files short-circuit
app.UseRouting();              // 2. Match request to an endpoint
app.UseAuthorization();        // 3. Endpoint metadata is now available
app.MapControllers();          // 4. Execute the endpoint

app.Run();
```

**Expected Output:** A request to `/Home/Index` is matched to `HomeController.Index`; a request to `/css/site.css` is served by static files before routing.

**Why this output?** `UseRouting` selects the endpoint based on the URL; `MapControllers` (via `UseEndpoints`) executes it.

### Real-World Cases

- **MVC/Razor Pages:** Routing selects the controller action or page handler.
- **Minimal APIs:** Routing matches the URL to a lambda handler.
- **Mixed applications:** Routing enables both controllers and minimal APIs in one pipeline.

### References

- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- `UseRouting` and `UseEndpoints` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#routing

---

## 2. `UseEndpoints()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseEndpoints()` adds middleware that executes the endpoint selected by `UseRouting()`. |
| **Technical Definition** | `UseEndpoints()` (in `Microsoft.AspNetCore.Builder`) adds endpoint execution to the pipeline. It runs the `RequestDelegate` associated with the selected `Endpoint` (e.g., a controller action or minimal API handler). In `WebApplication`, endpoint execution is typically configured via `MapControllers()`, `MapRazorPages()`, or `MapGet`/`MapPost` without an explicit `UseEndpoints` call, but `UseEndpoints` remains available for explicit pipeline control. |
| **Beginner-Friendly Explanation** | `UseEndpoints()` is the **executor** that actually runs the code chosen by the receptionist (`UseRouting`). If routing says “this request goes to `HomeController.Index`,” `UseEndpoints` makes that method run. |

### Purposes

- **To** execute the endpoint selected by routing.
- **To** provide a terminal point for the middleware pipeline (endpoints are terminal).
- **To** support explicit control over where endpoint execution occurs in the pipeline.
- **To** enable both controller-based and minimal-API endpoints in one place.

### Syntax Rules and Structure

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapRazorPages();
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseEndpoints` | Adds endpoint execution middleware. |
| `endpoints` | An `IEndpointRouteBuilder` for mapping endpoints. |

**Syntax rules:**

- Must be called **after** `UseRouting` (and after any middleware that needs to inspect or modify the endpoint).
- `UseEndpoints` is terminal for matched endpoints; if no endpoint matches, it passes control forward (which may result in a 404).
- In `WebApplication`, explicit `UseEndpoints` is often unnecessary because `MapControllers`/`MapRazorPages` configure endpoint execution automatically.

**Constraints and limitations:**

- Placing `UseEndpoints` too early (before authorization, for example) means those middleware will never run for matched endpoints.
- `UseEndpoints` does **not** perform routing—it only executes the endpoint selected by `UseRouting`.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});

app.Run();
```

**Expected Output:** Matched controller actions execute only after authentication and authorization have run.

**Why this output?** `UseEndpoints` is placed after auth middleware, so the endpoint executes only after the request has been authenticated and authorized.

### Real-World Cases

- **Explicit pipeline control:** Applications that need fine-grained ordering of endpoint execution.
- **Migration from older hosting models:** `UseEndpoints` is the classic way to configure endpoint execution.
- **Minimal APIs:** `MapGet`/`MapPost` call `UseEndpoints` internally in `WebApplication`.

### References

- `UseEndpoints` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing#use-endpoints
- Minimal APIs quick reference (`UseEndpoints` behaviour) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis

---

## 3. `UseAuthentication()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseAuthentication()` adds middleware that identifies the current user by validating credentials (cookies, tokens, etc.). |
| **Technical Definition** | `UseAuthentication()` adds the `AuthenticationMiddleware`, which invokes the registered authentication handlers (e.g., cookie authentication, JWT bearer) to populate `HttpContext.User` with a `ClaimsPrincipal`. It must run **before** `UseAuthorization()` and **after** `UseRouting()`. In `WebApplication`, it is added automatically immediately after `UseRouting` if an `IAuthenticationSchemeProvider` is registered, but explicit calls are common for clarity and ordering control. |
| **Beginner-Friendly Explanation** | `UseAuthentication()` is the **ID check**. It looks at the request (cookies, tokens, headers) and figures out **who** is making the request. It doesn’t decide whether they are allowed—that comes next with authorization. |

### Purposes

- **To** establish the identity of the current user for each request.
- **To** populate `HttpContext.User` with claims, roles, and identity information.
- **To** support multiple authentication schemes (cookies, JWT, OAuth, etc.).
- **To** enable downstream middleware and endpoints to make identity-based decisions.

### Syntax Rules and Structure

```csharp
app.UseAuthentication();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseAuthentication()` | Adds authentication middleware; must be before `UseAuthorization()`. |

**Syntax rules:**

- Must be called **after** `UseRouting()` and **before** `UseAuthorization()`.
- Authentication services must be registered (e.g., `AddAuthentication`, `AddCookie`, `AddJwtBearer`).
- If `UseCors()` is used, it must run **before** `UseAuthentication()`.

**Constraints and limitations:**

- Without `UseAuthentication`, `HttpContext.User` is unauthenticated, and `UseAuthorization` cannot enforce identity-based rules.
- Authentication middleware does not challenge or forbid requests—that is authorization’s role.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseRouting();
app.UseCors();
app.UseAuthentication();   // Identifies the user
app.UseAuthorization();    // Checks permissions
app.MapControllers();

app.Run();
```

**Expected Output:** A request with a valid authentication cookie populates `HttpContext.User` with the user’s claims; a request without a cookie leaves `User` unauthenticated.

**Why this output?** The cookie authentication handler reads the cookie, validates it, and constructs the `ClaimsPrincipal`.

### Real-World Cases

- **Cookie authentication:** Standard for MVC/Razor Pages web apps.
- **JWT bearer authentication:** APIs consumed by SPAs or mobile apps.
- **Multiple schemes:** APIs that support both cookies and tokens.

### References

- Authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- `UseAuthentication` ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order

---

## 4. `UseAuthorization()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseAuthorization()` adds middleware that checks whether the authenticated user is permitted to access the selected endpoint. |
| **Technical Definition** | `UseAuthorization()` adds the `AuthorizationMiddleware`, which evaluates authorization policies (e.g., `[Authorize]` attributes, `RequireAuthorization()` on endpoints) against `HttpContext.User`. It must run **after** `UseAuthentication()` and **after** `UseRouting()` (so endpoint metadata is available). In `WebApplication`, it is added automatically immediately after authentication if an `IAuthorizationHandlerProvider` is registered. |
| **Beginner-Friendly Explanation** | `UseAuthorization()` is the **permission check**. Once authentication has told us **who** the user is, authorization decides whether that user is **allowed** to access the requested resource. |

### Purposes

- **To** enforce access control policies (roles, claims, custom policies).
- **To** prevent unauthorized users from reaching protected endpoints.
- **To** support both attribute-based (`[Authorize]`) and endpoint-based (`RequireAuthorization()`) policies.
- **To** integrate with authentication to form a complete security model.

### Syntax Rules and Structure

```csharp
app.UseAuthorization();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseAuthorization()` | Adds authorization middleware; must be after `UseAuthentication()`. |

**Syntax rules:**

- Must be called **after** `UseAuthentication()` and **after** `UseRouting()`.
- Authorization services must be registered (`AddAuthorization()`).
- Placing `UseAuthorization` before `UseAuthentication` results in unauthenticated users being evaluated, breaking security.

**Constraints and limitations:**

- Without `UseAuthentication`, authorization cannot identify the user and will treat all requests as unauthenticated.
- Authorization middleware does not challenge (redirect to login) for APIs—it returns 401/403; for web apps, cookie authentication handles challenges.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();   // First: identify the user
app.UseAuthorization();    // Then: check permissions
app.MapControllers();

app.Run();
```

**Expected Output:** An authenticated user with the required role accesses a `[Authorize(Roles = "Admin")]` endpoint; an authenticated user without the role receives 403 Forbidden; an unauthenticated user receives a challenge (redirect to login).

**Why this output?** Authorization middleware evaluates the user’s claims against the endpoint’s policy and allows, forbids, or challenges accordingly.

### Real-World Cases

- **Role-based access:** Admin-only controllers or actions.
- **Policy-based access:** Custom requirements (e.g., minimum age, subscription tier).
- **API protection:** JWT-protected endpoints with scope requirements.

### References

- Authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/
- `UseAuthorization` ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order

---

## 5. `UseStaticFiles()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseStaticFiles()` adds middleware that serves static files (HTML, CSS, JavaScript, images) directly from the file system, short-circuiting the pipeline for file requests. |
| **Technical Definition** | `UseStaticFiles()` adds the `StaticFileMiddleware`, which looks for a file matching the request path in the web root (`wwwroot` by default) or configured file providers. If found, it serves the file and **does not call `next`** (terminal for file requests). Static assets in `wwwroot` are publicly accessible by default because `UseStaticFiles` is called **before** `UseAuthorization` in standard templates. |
| **Beginner-Friendly Explanation** | `UseStaticFiles()` is the **file server**. When a browser requests `/css/site.css` or `/images/logo.png`, this middleware finds the file on disk and sends it back immediately—without involving routing, authentication, or controllers. |

### Purposes

- **To** serve static assets (CSS, JS, images, fonts) efficiently without hitting the MVC pipeline.
- **To** short-circuit requests for files, reducing unnecessary processing.
- **To** support custom file providers (e.g., embedded resources, cloud storage).
- **To** enable directory browsing (with additional middleware).

### Syntax Rules and Structure

```csharp
app.UseStaticFiles();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseStaticFiles()` | Serves files from `wwwroot` by default. |
| Options overload | Allows custom `FileProvider`, request path, etc. |

**Syntax rules:**

- Typically called **before** `UseRouting()` and **before** `UseAuthorization()` so file requests are served without authentication overhead.
- Files are served from `wwwroot` by default; additional folders can be exposed with `UseStaticFiles(new StaticFileOptions { FileProvider = ... })`.
- `UseDefaultFiles()` (URL rewriter) is often called before `UseStaticFiles()` to serve default documents (e.g., `index.html`).

**Constraints and limitations:**

- **Public access:** Files served by `UseStaticFiles` are publicly accessible—do not place sensitive files in `wwwroot`.
- **No compression:** Static files served by this middleware are **not compressed** by `UseResponseCompression` if static files run first. Place `UseResponseCompression` before `UseStaticFiles` if compression is desired for static files.
- **No authorization:** Static files bypass authentication and authorization by default.

### Annotated Code Example

```csharp
var app = builder.Build();

app.UseResponseCompression();   // Compress responses (including static files)
app.UseStaticFiles();           // Serve files from wwwroot
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A request to `/css/site.css` is served directly from `wwwroot/css/site.css` and compressed if the client supports it; the request never reaches routing or authorization.

**Why this output?** `UseStaticFiles` finds the file, serves it, and does not call `next`—the pipeline is short-circuited.

### Real-World Cases

- **Web app assets:** CSS, JavaScript, images, and fonts.
- **SPA hosting:** Serving the SPA’s static files while routing API requests to controllers.
- **Documentation sites:** Serving static HTML documentation.

### References

- Static files in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files
- `UseStaticFiles` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files#serve-static-files

---

## 6. `UseCors()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseCors()` adds middleware that enables Cross-Origin Resource Sharing (CORS), allowing browsers to make requests to a different origin (domain, scheme, or port) than the one that served the page. |
| **Technical Definition** | `UseCors()` adds the `CorsMiddleware`, which evaluates CORS policies (named policies or default policy) and adds the appropriate `Access-Control-Allow-*` headers to responses. It must be called **after** `UseRouting()` and **before** `UseAuthorization()`; otherwise, preflight requests may be rejected or policies may not apply correctly. |
| **Beginner-Friendly Explanation** | `UseCors()` is the **cross-origin gatekeeper**. Browsers block JavaScript from calling APIs on a different domain unless the API explicitly allows it. CORS middleware adds the headers that tell the browser “yes, this origin is allowed.” |

### Purposes

- **To** enable SPAs, mobile web apps, and other cross-origin clients to call your API.
- **To** control which origins, methods, and headers are permitted.
- **To** handle CORS preflight (`OPTIONS`) requests automatically.
- **To** apply CORS policies globally, per endpoint, or per controller/action.

### Syntax Rules and Structure

```csharp
// Configure policy in services
builder.Services.AddCors(options =>
{
    options.AddPolicy("MyPolicy", policy =>
        policy.WithOrigins("https://example.com")
              .AllowAnyHeader()
              .AllowAnyMethod());
});

// Add middleware
app.UseCors("MyPolicy");   // or app.UseCors(); for default policy
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddCors` | Registers CORS policies. |
| `UseCors` | Adds CORS middleware; must be after `UseRouting` and before `UseAuthorization`. |

**Syntax rules:**

- `UseCors()` must be called **after** `UseRouting()` and **before** `UseAuthorization()`.
- `UseCors()` should generally be called **after** `UseStaticFiles()`.
- When using endpoint routing, `UseCors` must be before `UseAuthorization` and after `UseRouting`.

**Constraints and limitations:**

- **Order matters:** Placing `UseCors` after `UseAuthorization` causes preflight requests to be rejected by authorization before CORS headers are added.
- **Not a security boundary:** CORS is enforced by browsers; it does not prevent server-to-server requests.
- **`[EnableCors]` attributes** on controllers/actions require the CORS middleware to be in the pipeline.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpa", policy =>
        policy.WithOrigins("https://my-spa.com")
              .AllowAnyHeader()
              .AllowAnyMethod());
});

var app = builder.Build();

app.UseRouting();
app.UseCors("AllowSpa");       // Must be after routing, before auth
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A browser request from `https://my-spa.com` to `/api/products` receives the appropriate `Access-Control-Allow-Origin` headers. A preflight `OPTIONS` request is handled by the CORS middleware before authorization runs.

**Why this output?** CORS middleware evaluates the policy and adds headers; placing it before authorization ensures preflight requests are not blocked.

### Real-World Cases

- **SPA backends:** React, Angular, or Vue apps calling an ASP.NET Core API on a different port or domain.
- **Public APIs:** Allowing specific third-party origins to call your API.
- **Multi-tenant SaaS:** Different customer domains calling a shared API.

### References

- Enable cross-origin requests (CORS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cors
- `UseCors` ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order

---

## 7. `UseHttpsRedirection()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseHttpsRedirection()` adds middleware that redirects HTTP requests to HTTPS. |
| **Technical Definition** | `UseHttpsRedirection()` adds the `HttpsRedirectionMiddleware`, which issues a `307 Temporary Redirect` (or `308 Permanent Redirect` when configured) to the HTTPS version of the request URL. It is typically placed early in the pipeline so all requests are upgraded to HTTPS before any sensitive processing occurs. |
| **Beginner-Friendly Explanation** | `UseHttpsRedirection()` is the **“use the secure door”** sign. If someone tries to access your site over plain HTTP, this middleware automatically sends them to the HTTPS version. |

### Purposes

- **To** ensure all traffic uses HTTPS, protecting data in transit.
- **To** redirect legacy HTTP bookmarks and links to the secure version.
- **To** comply with security best practices and regulatory requirements.
- **To** work alongside HSTS (`UseHsts`) for comprehensive HTTPS enforcement.

### Syntax Rules and Structure

```csharp
app.UseHttpsRedirection();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseHttpsRedirection()` | Adds HTTPS redirection middleware. |

**Syntax rules:**

- Typically placed **before** `UseStaticFiles()` and `UseRouting()` so all requests are redirected early.
- Requires an HTTPS port to be configured (via `HTTPS_PORT` environment variable, `ASPNETCORE_HTTPS_PORTS`, or `UseHttps` in Kestrel configuration).

**Constraints and limitations:**

- **Development certificates:** HTTPS redirection may require a trusted development certificate (`dotnet dev-certs https --trust`).
- **Not a substitute for HSTS:** Redirection protects users who type `http://`; HSTS protects against downgrade attacks after the first visit.
- **API projects** may not include `UseHttpsRedirection` by default because APIs are often consumed over HTTPS directly.

### Annotated Code Example

```csharp
var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();              // HSTS for production
}
app.UseHttpsRedirection();      // Redirect HTTP to HTTPS
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A request to `http://example.com` receives a `307` redirect to `https://example.com`.

**Why this output?** The HTTPS redirection middleware intercepts the HTTP request and issues a redirect to the HTTPS scheme.

### Real-World Cases

- **Public websites:** All HTTP traffic is upgraded to HTTPS.
- **API endpoints:** Enforce secure communication for sensitive data.
- **Compliance:** Meet security requirements for data-in-transit protection.

### References

- Enforce HTTPS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl
- `UseHttpsRedirection` – https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl#https-redirection-middleware

---

## 8. `UseHsts()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseHsts()` adds middleware that sends the `Strict-Transport-Security` (HSTS) header, instructing browsers to only access the site over HTTPS for a specified period. |
| **Technical Definition** | `UseHsts()` adds the `HstsMiddleware`, which adds the `Strict-Transport-Security` response header with configurable `max-age`, `includeSubDomains`, and `preload` directives. HSTS prevents protocol downgrade attacks and cookie hijacking by telling browsers to never use HTTP for the site. **HSTS should only be enabled in production**, not in Development, because it is a browser-level instruction that is difficult to undo. |
| **Beginner-Friendly Explanation** | `UseHsts()` is the **“remember to always use HTTPS”** instruction. Once a browser sees the HSTS header, it will automatically use HTTPS for your site on every future visit—even if the user types `http://`. This makes your site more secure, but it should only be enabled in production because it is hard to reverse. |

### Purposes

- **To** protect against protocol downgrade attacks and cookie hijacking.
- **To** instruct browsers to always use HTTPS for the site.
- **To** complement `UseHttpsRedirection` with a browser-level enforcement mechanism.
- **To** support `includeSubDomains` and `preload` for comprehensive HTTPS enforcement.

### Syntax Rules and Structure

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseHsts()` | Adds HSTS middleware; typically wrapped in a production-only check. |

**Syntax rules:**

- **Must only be enabled in production** (not Development) because it is a long-lived browser instruction.
- Placed **before** `UseHttpsRedirection()` in the pipeline.
- Can be configured via `HstsOptions` (max-age, includeSubDomains, preload).

**Constraints and limitations:**

- **Difficult to undo:** Once a browser receives HSTS, it will refuse HTTP for the configured duration.
- **HTTPS required:** HSTS is only sent over HTTPS responses; if the site is not HTTPS, HSTS is not effective.
- **API projects** often do not enable HSTS because APIs are typically not accessed directly by browsers.

### Annotated Code Example

```csharp
var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();              // Only in production
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** In production, responses over HTTPS include the `Strict-Transport-Security` header. In Development, the header is absent.

**Why this output?** HSTS is only added when the environment is not Development, preventing accidental permanent HSTS in local development.

### Real-World Cases

- **Public-facing websites:** Enforce HTTPS for all visitors after the first visit.
- **E-commerce and banking:** Protect sensitive transactions from downgrade attacks.
- **Compliance:** Meet security standards that require HSTS.

### References

- Enforce HTTPS in ASP.NET Core (HSTS) – https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl#http-strict-transport-security-protocol-hsts
- `UseHsts` – https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl#http-strict-transport-security-protocol-hsts

---

## 9. `UseSession()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseSession()` adds middleware that establishes and maintains session state, allowing data to be stored across multiple requests from the same user. |
| **Technical Definition** | `UseSession()` adds the `SessionMiddleware`, which manages a session cookie and stores session data (in-memory, distributed cache, or SQL Server). It must be called **after** `UseRouting()` and **after** `UseCookiePolicy()`, and **before** MVC/Razor Pages middleware (endpoints). Session state requires `AddSession()` (or `AddDistributedMemoryCache()` with `AddSession()`) in the services configuration. |
| **Beginner-Friendly Explanation** | `UseSession()` gives your app **short-term memory** about a user across multiple requests. For example, a shopping cart can be stored in session so the user doesn’t lose it when navigating between pages. |

### Purposes

- **To** store user-specific data across multiple HTTP requests (e.g., shopping cart, wizard state).
- **To** provide a simple state management mechanism without database round-trips.
- **To** support temporary data that does not need to persist beyond the session lifetime.
- **To** integrate with authentication for session-based scenarios.

### Syntax Rules and Structure

```csharp
// Services
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

// Middleware
app.UseSession();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddSession` | Registers session services. |
| `UseSession` | Adds session middleware; must be after `UseRouting` and before endpoints. |

**Syntax rules:**

- Must be called **after** `UseRouting()` and **after** `UseCookiePolicy()` (if used), and **before** `MapRazorPages()`/`MapControllers()`.
- `HttpContext.Session` is only available **after** `UseSession` has run.
- Session data is stored in a cache; `AddDistributedMemoryCache` is required for in-memory storage.

**Constraints and limitations:**

- **Not for sensitive data:** Session data is stored on the server but identified by a cookie; do not store secrets unless the session store is secured.
- **Scalability:** In-memory session does not work across multiple servers; use a distributed cache (Redis, SQL Server) for web farms.
- **No session before middleware:** Accessing `HttpContext.Session` before `UseSession` throws `InvalidOperationException`.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseRouting();
app.UseSession();              // After routing, before endpoints
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** `HttpContext.Session` is available in controllers and views; a session cookie is set on the first request, and session values persist across requests.

**Why this output?** Session middleware manages the session cookie and makes the `ISession` available for storing and retrieving data.

### Real-World Cases

- **Shopping carts:** Temporary cart state across pages.
- **Multi-step wizards:** Preserve form data between steps.
- **User preferences:** Temporary UI settings (theme, language) before persistence.

### References

- Session and state management in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/app-state#session-state
- `UseSession` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/app-state#configure-session-state

---

## 10. `UseCookiePolicy()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseCookiePolicy()` adds middleware that enforces cookie consent and security policies, such as `Secure`, `SameSite`, and `HttpOnly` flags. |
| **Technical Definition** | `UseCookiePolicy()` adds the `CookiePolicyMiddleware`, which evaluates `CookiePolicyOptions` and applies policy decisions to cookies written by the application. It is commonly used for GDPR compliance (cookie consent) and to enforce minimum security standards for cookie fields. It must be placed **before** authentication and authorization middleware. |
| **Beginner-Friendly Explanation** | `UseCookiePolicy()` is the **cookie rule enforcer**. It ensures that cookies meet your security and privacy standards—for example, that they are marked `Secure` and `SameSite`, and that users have consented to non-essential cookies where required by law. |

### Purposes

- **To** enforce cookie security flags (`Secure`, `HttpOnly`, `SameSite`).
- **To** implement cookie consent for GDPR and similar regulations.
- **To** apply consistent cookie policies across the application.
- **To** integrate with authentication cookies for enhanced security.

### Syntax Rules and Structure

```csharp
builder.Services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = SameSiteMode.None;
    options.Secure = CookieSecurePolicy.Always;
});

app.UseCookiePolicy();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `CookiePolicyOptions` | Configures consent and security settings. |
| `UseCookiePolicy()` | Adds cookie policy middleware; must be before auth. |

**Syntax rules:**

- Must be called **before** `UseAuthentication()` and `UseAuthorization()`.
- `CheckConsentNeeded` determines whether user consent is required for non-essential cookies.
- `MinimumSameSitePolicy` and `Secure` enforce security flags.

**Constraints and limitations:**

- **Cookie policy middleware must run before authentication** to affect authentication cookies.
- **GDPR compliance** requires more than cookie policy—consent UI and data processing agreements are also needed.
- **Not a substitute for secure cookie configuration** in authentication options.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = SameSiteMode.None;
    options.Secure = CookieSecurePolicy.Always;
});

var app = builder.Build();

app.UseCookiePolicy();         // Before authentication
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** Cookies written by the application respect the configured policy (e.g., `Secure`, `SameSite`), and non-essential cookies require consent.

**Why this output?** Cookie policy middleware intercepts cookie writes and applies the configured policy decisions.

### Real-World Cases

- **GDPR compliance:** Cookie consent banners and policy enforcement.
- **Security hardening:** Enforcing `Secure` and `SameSite` on all cookies.
- **Multi-tenant SaaS:** Consistent cookie policies across tenants.

### References

- EU General Data Protection Regulation (GDPR) support in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/gdpr
- `UseCookiePolicy` – https://learn.microsoft.com/en-us/aspnet/core/security/gdpr#cookie-policy-middleware

---

## 11. `UseExceptionHandler()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseExceptionHandler()` adds middleware that catches unhandled exceptions from downstream middleware and re-executes the request against an error handling path. |
| **Technical Definition** | `UseExceptionHandler()` adds the `ExceptionHandlerMiddleware`, which catches unhandled exceptions, logs them, and re-executes the request pipeline at the specified error path (e.g., `/Error`). It is intended for **production** environments; in Development, `UseDeveloperExceptionPage` provides detailed error information instead. |
| **Beginner-Friendly Explanation** | `UseExceptionHandler()` is the **safety net**. If any downstream code throws an unhandled exception, this middleware catches it and redirects the request to a friendly error page, so users see a helpful message instead of a crash. |

### Purposes

- **To** catch unhandled exceptions and prevent raw error details from leaking to clients.
- **To** redirect users to a friendly error page.
- **To** log exceptions for diagnostics.
- **To** provide a consistent error-handling experience across the application.

### Syntax Rules and Structure

```csharp
app.UseExceptionHandler("/Error");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseExceptionHandler("/Error")` | Catches exceptions and re-executes the pipeline at `/Error`. |

**Syntax rules:**

- Should be placed **first** in the pipeline so it can catch exceptions from all downstream middleware.
- The error path (e.g., `/Error`) must be a valid endpoint (controller action, Razor Page, or minimal API handler).
- Can also be configured with a lambda for custom error handling.

**Constraints and limitations:**

- **Does not catch exceptions thrown by the error page itself** (avoid infinite loops).
- **Re-executes the pipeline**—the error page must be accessible without authentication in some scenarios.
- **Should not be used in Development**; use `UseDeveloperExceptionPage` for detailed diagnostics.

### Annotated Code Example

```csharp
var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
}
else
{
    app.UseDeveloperExceptionPage();
}

app.UseHsts();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** In production, an unhandled exception redirects the request to `/Error`, where a friendly error page is displayed. In Development, the detailed developer exception page is shown.

**Why this output?** Exception handler middleware catches the exception and re-executes the pipeline against the `/Error` path.

### Real-World Cases

- **Production error handling:** Friendly error pages without stack traces.
- **API error responses:** Consistent JSON error payloads.
- **Logging:** Capturing exceptions for monitoring and diagnostics.

### References

- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- `UseExceptionHandler` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#exception-handler-page

---

## 12. `UseStatusCodePages()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseStatusCodePages()` adds middleware that generates a response for HTTP status codes that do not have a body (e.g., 404, 500). |
| **Technical Definition** | `UseStatusCodePages()` adds the `StatusCodePagesMiddleware`, which checks the response status code between 400 and 599 and, if no body has been written, generates a text or HTML response. It is commonly used to provide user-friendly error pages for 404 (Not Found) and 500 (Internal Server Error) responses. |
| **Beginner-Friendly Explanation** | `UseStatusCodePages()` is the **missing response filler**. When something goes wrong (like a page not found), some responses have a status code but no body. This middleware adds a helpful message so the user sees something meaningful instead of a blank page. |

### Purposes

- **To** provide friendly error messages for status codes without bodies.
- **To** avoid blank pages for 404 and 500 responses.
- **To** support custom status code pages (HTML, JSON, or plain text).
- **To** complement `UseExceptionHandler` (which handles exceptions, not status codes).

### Syntax Rules and Structure

```csharp
app.UseStatusCodePages();
// or with a format string:
app.UseStatusCodePages("text/plain", "Status code: {0}");
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseStatusCodePages()` | Adds default status code pages. |
| Format string overload | Customizes the response body. |

**Syntax rules:**

- Place **before** `UseStaticFiles()` and endpoint middleware so it can observe their status codes.
- Does **not** catch exceptions—use `UseExceptionHandler` for that.
- `UseStatusCodePagesWithReExecute` re-executes the pipeline for status code pages.

**Constraints and limitations:**

- **Not typically used in production** because the default message is not user-friendly; customize it for production use.
- **Does not replace `UseExceptionHandler`**—status codes and exceptions are different failure modes.

### Annotated Code Example

```csharp
var app = builder.Build();

app.UseStatusCodePages("text/plain", "Oops! Status code: {0}");
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A request to a non-existent path (`/missing`) receives a `404` response with the body “Oops! Status code: 404”.

**Why this output?** Status code pages middleware detects the 404 status and no body, then generates the formatted text response.

### Real-World Cases

- **User-friendly 404 pages:** Custom “Page not found” messages.
- **API error responses:** JSON error bodies for status codes.
- **Development diagnostics:** Quick status code information during development.

### References

- Handle errors in ASP.NET Core (Status Code Pages) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#usestatuscodepages
- `UseStatusCodePages` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling#usestatuscodepages

---

## 13. `UseResponseCompression()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseResponseCompression()` adds middleware that compresses HTTP responses (Gzip, Brotli, etc.) to reduce bandwidth and improve load times. |
| **Technical Definition** | `UseResponseCompression()` adds the `ResponseCompressionMiddleware`, which compresses responses based on the `Accept-Encoding` request header and configured providers (Brotli, Gzip). It must be called **before** any middleware that generates compressible responses (e.g., static files, MVC) and requires `AddResponseCompression()` in the services configuration. |
| **Beginner-Friendly Explanation** | `UseResponseCompression()` is the **data shrinker**. It compresses responses before sending them to the browser, making them smaller and faster to download. The browser decompresses them automatically. |

### Purposes

- **To** reduce bandwidth usage and improve page load times.
- **To** support Brotli and Gzip compression.
- **To** compress dynamic responses (MVC views, API JSON) and static files.
- **To** improve performance for users on slow connections.

### Syntax Rules and Structure

```csharp
// Services
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

// Middleware
app.UseResponseCompression();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddResponseCompression` | Registers compression services and providers. |
| `UseResponseCompression()` | Adds compression middleware; must be before other middleware. |

**Syntax rules:**

- Must be called **before** any middleware that compresses responses (e.g., `UseStaticFiles`, `MapControllers`).
- `EnableForHttps` is `false` by default for security (BREACH attack); set to `true` only if necessary.
- The client must send `Accept-Encoding` for compression to occur.

**Constraints and limitations:**

- **Order matters:** Placing `UseResponseCompression` after `UseStaticFiles` means static files are not compressed.
- **Not all content types are compressed by default**; configure `MimeTypes` for additional types.
- **BREACH attack risk** when compressing HTTPS responses containing secrets; disable `EnableForHttps` unless necessary.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

var app = builder.Build();

app.UseResponseCompression();   // Before static files and endpoints
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** Responses to requests with `Accept-Encoding: gzip` or `br` are compressed; the browser decompresses them transparently. Static files and MVC responses are both compressed.

**Why this output?** Compression middleware wraps the response stream and compresses data based on the client’s accepted encodings.

### Real-World Cases

- **High-traffic sites:** Reduce bandwidth costs and improve load times.
- **Mobile users:** Faster downloads on limited connections.
- **APIs:** Compress JSON responses for large payloads.

### References

- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- `UseResponseCompression` – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression#configure-response-compression-middleware

---

## 14. `UseRequestLocalization()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseRequestLocalization()` adds middleware that determines the current culture and UI culture for each request based on request data (query string, cookies, `Accept-Language` header, etc.). |
| **Technical Definition** | `UseRequestLocalization()` adds the `RequestLocalizationMiddleware`, which evaluates a list of `RequestCultureProvider` instances (query string, cookie, `Accept-Language` header, route data) to determine the request’s culture. It sets `CultureInfo.CurrentCulture` and `CultureInfo.CurrentUICulture` and adds the `Content-Language` response header. It must be placed **after** `UseRouting()` if the `RouteDataRequestCultureProvider` is used. |
| **Beginner-Friendly Explanation** | `UseRequestLocalization()` is the **language detector**. It looks at the request (browser language, cookie, query string) and sets the app’s language and culture for that request, so dates, numbers, and text are displayed in the user’s preferred format. |

### Purposes

- **To** support multi-language and multi-culture applications.
- **To** automatically detect the user’s preferred culture from request data.
- **To** format dates, numbers, and currencies according to the current culture.
- **To** serve localized content based on the request culture.

### Syntax Rules and Structure

```csharp
builder.Services.AddLocalization(options => options.ResourcesPath = "Resources");

var app = builder.Build();

var supportedCultures = new[] { "en-US", "fr-FR", "es-ES" };
app.UseRequestLocalization(new RequestLocalizationOptions
{
    DefaultRequestCulture = new RequestCulture("en-US"),
    SupportedCultures = supportedCultures,
    SupportedUICultures = supportedCultures,
    RequestCultureProviders = new List<IRequestCultureProvider>
    {
        new QueryStringRequestCultureProvider(),
        new CookieRequestCultureProvider(),
        new AcceptLanguageHeaderRequestCultureProvider()
    }
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddLocalization` | Registers localization services. |
| `UseRequestLocalization` | Adds middleware; configures supported cultures and providers. |

**Syntax rules:**

- Must be placed **after** `UseRouting()` if using `RouteDataRequestCultureProvider`.
- Providers are evaluated in order; the first one that returns a culture is used.
- `SupportedCultures` must include the cultures the app supports.

**Constraints and limitations:**

- **Culture detection order matters**—place more specific providers (query string) before general ones (`Accept-Language`).
- **Resource files** must be created for each supported culture.
- **Routing-based culture** requires `UseRequestLocalization` to run after routing.

### Annotated Code Example

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddLocalization(options => options.ResourcesPath = "Resources");
builder.Services.AddControllersWithViews();

var app = builder.Build();

var supportedCultures = new[] { "en-US", "fr-FR" };
app.UseRequestLocalization(new RequestLocalizationOptions
{
    DefaultRequestCulture = new RequestCulture("en-US"),
    SupportedCultures = supportedCultures,
    SupportedUICultures = supportedCultures
});

app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A request with `Accept-Language: fr-FR` sets the culture to French, and the `Content-Language` header is set to `fr-FR`. Dates and numbers are formatted according to French culture.

**Why this output?** The localization middleware evaluates the `Accept-Language` header, determines the culture, and sets `CurrentCulture`/`CurrentUICulture` for the request.

### Real-World Cases

- **International websites:** Serving content in multiple languages.
- **Global APIs:** Formatting responses according to client locale.
- **E-commerce:** Displaying prices in local currencies and formats.

### References

- Globalization and localization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization
- `UseRequestLocalization` – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization#configure-localization-middleware

---

## 15. `UseWelcomePage()`

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | `UseWelcomePage()` adds middleware that serves a simple welcome page for the application’s root URL (`/`). |
| **Technical Definition** | `UseWelcomePage()` adds the `WelcomePageMiddleware`, which returns a default ASP.NET Core welcome page for the root path. It is intended for **development and testing** only and is **not recommended for production** because it provides no real application content. |
| **Beginner-Friendly Explanation** | `UseWelcomePage()` is a **placeholder homepage**. It shows a simple “Welcome to ASP.NET Core” page when you first create an app, so you know the app is running. You replace it with real content before going to production. |

### Purposes

- **To** provide a quick visual confirmation that the application is running.
- **To** serve as a temporary homepage during early development.
- **To** demonstrate basic middleware functionality in tutorials.

### Syntax Rules and Structure

```csharp
app.UseWelcomePage();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `UseWelcomePage()` | Adds welcome page middleware for the root URL. |

**Syntax rules:**

- Serves the welcome page for `/` only.
- Can be configured with a specific path: `app.UseWelcomePage("/welcome")`.
- **Should not be used in production**; it provides no real content.

**Constraints and limitations:**

- **Not for production**—it is a placeholder and should be removed or replaced.
- **Overrides other root handling**—if `UseWelcomePage` is first, it short-circuits the root URL.
- **Limited support in modern hosting**—`WebApplication` templates do not include it by default.

### Annotated Code Example

```csharp
var app = builder.Build();

app.UseWelcomePage();   // Temporary homepage (development only)

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Expected Output:** A request to `/` shows the default ASP.NET Core welcome page. Other requests flow through the rest of the pipeline.

**Why this output?** Welcome page middleware short-circuits requests to `/` and returns the built-in welcome page.

### Real-World Cases

- **Learning and prototyping:** Quick confirmation that the app runs.
- **Tutorials:** Demonstrating middleware in a minimal setup.
- **Not recommended for production**—replace with real homepage content.

### References

- `UseWelcomePage` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.welcomepageextensions.usewelcomepage
- ASP.NET Core middleware (legacy welcome page) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/

---

## 16. Built-in Middleware Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Best practices for built-in middleware ensure they are ordered correctly, configured securely, and used only where appropriate. |
| **Technical Definition** | Best practices encompass the canonical middleware order (exception handling first, static files before routing, CORS before auth, auth before authorization), environment-specific middleware (`UseHsts`, `UseExceptionHandler` outside Development), performance considerations (compression placement, static file short-circuiting), and security configurations (cookie policy, HSTS, HTTPS redirection). |
| **Beginner-Friendly Explanation** | Best practices are the **“do’s and don’ts”** for built-in middleware—they help you avoid security holes, broken CORS, uncompressed assets, and middleware that never runs because it was placed after a terminal component. |

### Purposes

- **To** prevent security vulnerabilities (auth bypass, missing HTTPS, insecure cookies).
- **To** ensure functionality (CORS, session, localization) works as intended.
- **To** optimise performance (compression, static file short-circuiting).
- **To** keep middleware configuration maintainable and predictable.

### Guidelines Checklist (Canonical Order)

| Position | Middleware | Reason |
|----------|------------|--------|
| 1 | `UseExceptionHandler` / `UseDeveloperExceptionPage` | Must catch exceptions from all downstream middleware. |
| 2 | `UseHsts` (production only) | Adds security headers early. |
| 3 | `UseHttpsRedirection` | Redirects HTTP to HTTPS before any processing. |
| 4 | `UseStaticFiles` | Short-circuits file requests before routing/auth overhead. |
| 5 | `UseCookiePolicy` | Enforces cookie policies before authentication. |
| 6 | `UseRouting` | Selects the endpoint; must precede endpoint-aware middleware. |
| 7 | `UseCors` | Must be after routing, before auth. |
| 8 | `UseAuthentication` | Identifies the user; must precede authorization. |
| 9 | `UseAuthorization` | Checks permissions; must follow authentication. |
| 10 | `UseSession` | After routing/cookie policy, before endpoints. |
| 11 | `UseResponseCompression` | Before middleware that generates compressible responses. |
| 12 | `UseRequestLocalization` | After routing (if using route-based culture). |
| 13 | Endpoints (`MapControllers`, `MapRazorPages`) | Terminal execution. |

**Additional best practices:**

| Area | Do | Don’t |
|------|----|-------|
| **Exception handling** | Place first so it catches all downstream exceptions. | Place after middleware that may throw. |
| **HTTPS** | Use `UseHttpsRedirection` and `UseHsts` in production. | Enable HSTS in Development. |
| **CORS** | Place `UseCors` after `UseRouting` and before `UseAuthentication`/`UseAuthorization`. | Place `UseCors` after authorization (preflight requests fail). |
| **Compression** | Place `UseResponseCompression` before `UseStaticFiles` and endpoints. | Place after static files (static files not compressed). |
| **Static files** | Place before routing and authorization. | Place after routing (unnecessary processing). |
| **Session** | Place after routing and cookie policy, before endpoints. | Access `HttpContext.Session` before `UseSession`. |
| **Cookie policy** | Place before authentication to affect auth cookies. | Place after authentication (policy not applied to auth cookies). |
| **Localization** | Place after routing if using route-based culture. | Place before routing (route data not available). |
| **Welcome page** | Use only in development/prototyping. | Use in production. |

### Annotated Code Example (Canonical Best-Practice Pipeline)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Services
builder.Services.AddControllersWithViews();
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme).AddCookie();
builder.Services.AddAuthorization();
builder.Services.AddCors(options =>
{
    options.AddPolicy("Spa", policy =>
        policy.WithOrigins("https://spa.example.com").AllowAnyHeader().AllowAnyMethod());
});
builder.Services.AddSession();
builder.Services.AddResponseCompression();
builder.Services.AddLocalization();

var app = builder.Build();

// Middleware pipeline (canonical order)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
else
{
    app.UseDeveloperExceptionPage();
}

app.UseHttpsRedirection();
app.UseResponseCompression();      // Before static files and endpoints
app.UseStaticFiles();               // Short-circuit files
app.UseCookiePolicy();              // Before authentication
app.UseRouting();                   // Select endpoint
app.UseCors("Spa");                 // After routing, before auth
app.UseAuthentication();            // Identify user
app.UseAuthorization();             // Check permissions
app.UseSession();                   // After routing, before endpoints
app.UseRequestLocalization();       // After routing
app.MapControllers();

app.Run();
```

**Why this follows best practices:**

- **Exception handling is first** so it catches all downstream exceptions.
- **HSTS and HTTPS redirection** are production-only and placed early.
- **Compression is before static files** so both static and dynamic responses are compressed.
- **Static files are before routing** to short-circuit file requests.
- **Cookie policy is before authentication** so auth cookies respect the policy.
- **CORS is after routing and before authentication** so preflight requests work.
- **Authentication is before authorization** so the user is identified before permissions are checked.
- **Session is after routing and cookie policy, before endpoints**.
- **Localization is after routing** to support route-based culture.

### References

- ASP.NET Core middleware order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Response compression ordering – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- Session middleware ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/app-state#configure-session-state
- CORS ordering – https://learn.microsoft.com/en-us/aspnet/core/security/cors
- Authentication and authorization ordering – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Cookie policy ordering – https://learn.microsoft.com/en-us/aspnet/core/security/gdpr#cookie-policy-middleware

---

## Summary Table: Built-in Middleware at a Glance

| Middleware | Primary Role | Key Ordering Constraint | Environment |
|------------|--------------|--------------------------|-------------|
| `UseRouting()` | Matches request to endpoint | Before `UseEndpoints` and endpoint-aware middleware | All |
| `UseEndpoints()` | Executes selected endpoint | After `UseRouting` and auth middleware | All |
| `UseAuthentication()` | Identifies the user | After `UseRouting`, before `UseAuthorization` | All |
| `UseAuthorization()` | Checks permissions | After `UseAuthentication` | All |
| `UseStaticFiles()` | Serves static files (short-circuits) | Before `UseRouting` | All |
| `UseCors()` | Enables cross-origin requests | After `UseRouting`, before `UseAuthorization` | When SPAs/APIs are used |
| `UseHttpsRedirection()` | Redirects HTTP to HTTPS | Early in pipeline | All (production) |
| `UseHsts()` | Enforces HTTPS via browser instruction | After exception handling, before redirection | Production only |
| `UseSession()` | Manages session state | After `UseRouting`/`UseCookiePolicy`, before endpoints | When session state is needed |
| `UseCookiePolicy()` | Enforces cookie consent/security | Before `UseAuthentication` | When GDPR/security requires |
| `UseExceptionHandler()` | Catches unhandled exceptions | First in pipeline | Production (use DeveloperExceptionPage in Development) |
| `UseStatusCodePages()` | Generates bodies for error status codes | Before static files/endpoints | Typically Development or customised for production |
| `UseResponseCompression()` | Compresses responses | Before static files and endpoints | When performance optimisation is needed |
| `UseRequestLocalization()` | Sets culture based on request | After `UseRouting` (for route-based culture) | Multi-language apps |
| `UseWelcomePage()` | Serves a placeholder homepage | Early (short-circuits root) | Development/prototyping only |

---

## References (Full List)

- ASP.NET Core Middleware (official documentation) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/
- ASP.NET Core middleware order – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/#middleware-order
- Routing in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/routing
- Authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- Authorization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/
- Static files in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files
- Enable cross-origin requests (CORS) in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/cors
- Enforce HTTPS in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl
- Session and state management in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/app-state#session-state
- EU General Data Protection Regulation (GDPR) support in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/gdpr
- Handle errors in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/error-handling
- Response compression in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/performance/response-compression
- Globalization and localization in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization
- `UseWelcomePage` – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.welcomepageextensions.usewelcomepage