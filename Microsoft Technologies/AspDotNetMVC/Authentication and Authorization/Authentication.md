# ASP.NET Core Authentication Fundamentals: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10). Authentication is the process of determining a user's identity; authorization determines whether that identity has access to a resource. In ASP.NET Core, authentication is handled by the `IAuthenticationService`, which uses registered authentication handlers grouped into **schemes**. Configuring authentication does **not** automatically restrict access—a fallback authorization policy or explicit `[Authorize]` attributes are required. This cheat sheet treats each requested topic independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authentication is the process of determining a user's identity. |
| **Technical Definition** | In ASP.NET Core, authentication is handled by the `IAuthenticationService`, which is used by the authentication middleware (`UseAuthentication`). The service invokes registered **authentication handlers** to perform authentication-related actions: authenticating a user and responding when an unauthenticated user attempts to access a restricted resource. Successful authentication produces a `ClaimsPrincipal`, which is attached to `HttpContext.User` for downstream authorization decisions. |
| **Beginner-Friendly Explanation** | Authentication is the **"Who are you?"** step. When a user tries to access your app, authentication checks their credentials (password, token, cookie) and, if valid, builds a profile (the `ClaimsPrincipal`) that tells the rest of the app who they are. Without authentication, your app cannot identify users; without authorization, any identified user could perform administrative actions. |

### Key Characteristics

- **Claims-based identity** – The authenticated user is represented by a `ClaimsPrincipal` containing one or more `ClaimsIdentity` objects, each holding a list of `Claim` key-value pairs (e.g., `Name: Alice`, `Role: Admin`).
- **Scheme-driven** – Authentication handlers and their configuration are grouped into named **schemes**; the scheme determines which handler processes the request.
- **Middleware-integrated** – `UseAuthentication()` populates `HttpContext.User` by invoking the configured scheme; it must run **before** `UseAuthorization()`.
- **Extensible** – Supports cookies, JWT bearer, OAuth/OIDC, Windows, certificate authentication, and custom handlers.
- **Decoupled from authorization** – Authentication only identifies the user; it does **not** grant or deny access by itself.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **dependency injection**, and **service registration**.
- Familiarity with **HTTP**, **cookies**, and **tokens** (for JWT/OAuth scenarios).

### Related Programming Areas

- **Authorization** – Uses the `ClaimsPrincipal` produced by authentication to make permission decisions.
- **ASP.NET Core Identity** – A higher-level membership system that manages users, passwords, and claims; it calls `AddAuthentication` internally.
- **Dependency injection** – Authentication services and handlers are registered in `IServiceCollection`.
- **Middleware pipeline** – `UseAuthentication` and `UseAuthorization` have mandatory ordering.

### Core Concepts / Features

1. **Authentication** – The overall identity-determination process.
2. **Identity** – `ClaimsPrincipal`/`ClaimsIdentity`/`Claim` and the claims model.
3. **Credentials** – Passwords, tokens, cookies, certificates, and secure storage.
4. **Authentication schemes** – Named handler configurations (`AddAuthentication`, `AddScheme`).
5. **Authentication best practices** – Middleware ordering, fallback policies, secret management, and avoiding custom implementations.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. Authentication (The Process)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authentication is the process of determining a user's identity by validating provided credentials against a trusted store. |
| **Technical Definition** | Authentication is handled by `IAuthenticationService`, invoked by the authentication middleware (`UseAuthentication`). The service delegates to an **authentication handler** for the selected scheme. The handler's **Authenticate** action constructs the user's identity from the request context (e.g., cookies, JWT bearer tokens) and returns an `AuthenticateResult` containing an `AuthenticationTicket` if successful. If authentication fails, authorization middleware may invoke the handler's **Challenge** action to prompt the user (e.g., redirect to login or return `401` with `WWW-Authenticate`). |
| **Beginner-Friendly Explanation** | Authentication is the **checkpoint at the entrance of your app**. When a request arrives, the authentication middleware asks the configured handler: "Can you figure out who this is?" The handler reads the cookie or token, validates it, and if everything checks out, attaches a `ClaimsPrincipal` to `HttpContext.User`. From that point on, every other component knows who the user is—or knows the user is anonymous. |

### Purposes

- **To** establish the identity of the current user for every request.
- **To** populate `HttpContext.User` with a `ClaimsPrincipal` that downstream authorization can use.
- **To** provide a challenge mechanism (login redirect or `401`) when unauthenticated users access protected resources.
- **To** support multiple authentication mechanisms (cookies, tokens, external providers) through a unified scheme-based model.
- **To** decouple identity verification from permission decisions, allowing each concern to evolve independently.

### Syntax Rules and Structure

#### General Syntax (Registration and Middleware)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Register authentication services with a default scheme
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* configure */ })
    .AddCookie(options => { /* configure */ });

// 2. Register authorization services
builder.Services.AddAuthorization();

var app = builder.Build();

// 3. Add authentication middleware (after UseRouting, before UseAuthorization)
app.UseAuthentication();

// 4. Add authorization middleware
app.UseAuthorization();

app.MapControllers();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthentication(defaultScheme)` | Registers `IAuthenticationService` and sets the default scheme. The parameter is the scheme to use when a specific scheme is not requested. |
| `.AddJwtBearer(...)` / `.AddCookie(...)` | Scheme-specific extension methods that call `AuthenticationBuilder.AddScheme` to register handlers. |
| `UseAuthentication()` | Adds middleware that invokes the configured schemes and populates `HttpContext.User`. |
| `UseAuthorization()` | Must be called **after** `UseAuthentication()`. |

#### Syntax Rules

- **`AddAuthentication` must be called before `UseAuthentication`**; the middleware uses the schemes registered at startup.
- **`UseAuthentication()` must precede `UseAuthorization()`**—authorization reads `HttpContext.User`, which authentication populates.
- **A default scheme is required** when multiple schemes are registered and no scheme is specified in `[Authorize]`; otherwise, an `InvalidOperationException` is thrown.
- **Scheme-specific extension methods** (`AddJwtBearer`, `AddCookie`, etc.) register handlers with appropriate options.

#### Constraints and Limitations

- **Authentication alone does not restrict access**—a fallback authorization policy or `[Authorize]` attributes are required to enforce it.
- **Calling `AddAuthentication` multiple times** is supported but requires care with options configuration; scheme registration conflicts (same scheme name) throw `InvalidOperationException`.
- **`UseAuthentication` does not refuse anyone**—it only identifies. Refusal (challenge/forbid) is handled by authorization middleware.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Cookie Authentication for a Web App

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register cookie authentication with a default scheme
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
        options.ExpireTimeSpan = TimeSpan.FromHours(1);
    });

builder.Services.AddAuthorization();
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();   // Populates HttpContext.User from cookie
app.UseAuthorization();    // Enforces [Authorize] policies
app.MapDefaultControllerRoute();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` registers the authentication service with `CookieAuthenticationDefaults.AuthenticationScheme` as the default scheme.
2. `AddCookie` registers the cookie handler and configures the login and access-denied paths.
3. `UseAuthentication()` runs on every request, invoking the cookie handler to validate the authentication cookie and populate `HttpContext.User`.
4. `UseAuthorization()` checks `[Authorize]` attributes and challenges unauthenticated users by redirecting to the login path.

**Expected Output:** An unauthenticated request to a `[Authorize]`-protected action is redirected to `/Account/Login`. An authenticated request with a valid cookie passes through to the action, and `User.Identity.IsAuthenticated` is `true`.

**Why this output?** The cookie handler validates the cookie and builds the `ClaimsPrincipal`; authorization middleware challenges unauthenticated users using the handler's challenge behavior.

### Real-World Cases

- **Consumer web applications:** Cookie authentication for browser-based sessions.
- **APIs consumed by SPAs:** JWT bearer authentication for JavaScript requests, often alongside cookies for server-rendered pages.
- **Enterprise applications:** Windows or certificate authentication for internal users.

### References

- Overview of ASP.NET Core authentication – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- ASP.NET Core security topics (Authentication vs. Authorization) – https://learn.microsoft.com/en-us/aspnet/core/security/

---

## 2. Identity (Claims-Based Identity Model)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Identity is the representation of a user as a set of claims—key-value statements about who the user is and what attributes they have. |
| **Technical Definition** | In ASP.NET Core, the authenticated user is represented by a `ClaimsPrincipal` attached to `HttpContext.User`. A `ClaimsPrincipal` contains one or more `ClaimsIdentity` objects; each `ClaimsIdentity` contains a list of `Claim` objects (name-value pairs). Common claims include `Name`, `Role`, and `Email`. The `ClaimsIdentity.IsInRole` method checks whether a `Role` claim exists in the collection. |
| **Beginner-Friendly Explanation** | Identity is the **profile card** built after authentication succeeds. It says things like "Name: Alice", "Role: Admin", "Email: alice@example.com". Each statement is a **claim**. All the claims together form the user's identity, which authorization uses to decide what Alice is allowed to do. |

### Purposes

- **To** provide a structured, extensible representation of the authenticated user.
- **To** carry attributes (roles, email, tenant, permissions) that authorization can evaluate.
- **To** decouple identity data from a specific authentication mechanism—cookies, JWTs, and external providers all produce claims.
- **To** enable claims-based authorization policies that go beyond simple role checks.

### Syntax Rules and Structure

#### The Identity Object Model

```text
ClaimsPrincipal
  └── ClaimsIdentity (one or more)
        └── Claim (key-value pair)
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `ClaimsPrincipal` | The user object attached to `HttpContext.User`; exposes `Identities` and helper methods like `IsInRole`. |
| `ClaimsIdentity` | Represents a single identity (e.g., cookie-based, token-based) with a collection of claims and an authentication type. |
| `Claim` | A key-value pair describing the user (e.g., `new Claim(ClaimTypes.Role, "Admin")`). |

#### Creating a ClaimsPrincipal (Manual Example)

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "Alice"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim(ClaimTypes.Email, "alice@example.com")
};

var identity = new ClaimsIdentity(claims, "pwd");
var principal = new ClaimsPrincipal(identity);
```

**Syntax rules:**

- A `ClaimsPrincipal` can hold **multiple identities**; authorization evaluates all of them.
- `ClaimTypes` provides standard claim type constants (`Name`, `Role`, `Email`, etc.).
- `ClaimsIdentity.IsInRole("Admin")` checks for a `Role` claim with the value `"Admin"`.

**Constraints and limitations:**

- **Claims are not encrypted by default**—they are encoded in the cookie or token; do not store secrets in claims.
- **Role claims depend on the identity store**—social logins may not include role claims.

### Annotated Code Example

```csharp
public IActionResult Profile()
{
    var user = HttpContext.User;

    if (user.Identity?.IsAuthenticated == true)
    {
        var name = user.Identity.Name;
        var isAdmin = user.IsInRole("Admin");
        var email = user.FindFirst(ClaimTypes.Email)?.Value;

        return Content($"Name: {name}, Admin: {isAdmin}, Email: {email}");
    }

    return Content("Anonymous");
}
```

**Expected Output:** For an authenticated user with `Name = "Alice"`, `Role = "Admin"`, and `Email = "alice@example.com"`, the response is `"Name: Alice, Admin: True, Email: alice@example.com"`. For an anonymous request, the response is `"Anonymous"`.

**Why this output?** `HttpContext.User` is populated by the authentication middleware; `IsInRole` checks the `Role` claim, and `FindFirst` retrieves the email claim.

### Real-World Cases

- **Role-based access:** `[Authorize(Roles = "Admin")]` uses the `Role` claim.
- **Claims-based policies:** A policy requiring an `EmployeeId` claim or a minimum `Age` claim.
- **Multi-tenant apps:** A `TenantId` claim used to scope data access.

### References

- Claims-based identity in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/claims
- Policy-based authorization (ClaimsPrincipal) – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## 3. Credentials

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Credentials are the evidence a user provides to prove their identity—passwords, tokens, certificates, or cookies. |
| **Technical Definition** | Credentials are compared to trusted values stored in an operating system, database, app, or resource; when they match, authentication succeeds. In ASP.NET Core, credentials may be passwords (hashed by Identity using PBKDF2), JWT bearer tokens, authentication cookies, client certificates, or external OAuth/OIDC tokens. Secrets such as connection strings, API keys, and signing keys must be loaded from secure configuration providers, never embedded in source code. |
| **Beginner-Friendly Explanation** | Credentials are **what you use to prove who you are**: your password, a token, a certificate, or a cookie. The authentication handler checks these credentials against what the app trusts. If they match, you are authenticated. If not, you are challenged. |

### Purposes

- **To** provide verifiable evidence of identity to the authentication handler.
- **To** enable different authentication mechanisms (passwords, tokens, certificates) through different credential types.
- **To** protect user accounts by storing passwords as **salted hashes**, never plain text.
- **To** keep secrets (signing keys, connection strings) out of source code and load them from secure stores.

### Syntax Rules and Structure

#### Credential Types and Handling

| Credential Type | Handler / Mechanism | Storage / Validation |
|-----------------|---------------------|----------------------|
| **Password** | `SignInManager` / `UserManager` | Hashed with PBKDF2 (HMAC-SHA256, 128-bit salt, 256-bit subkey, 10,000 iterations) |
| **JWT Bearer Token** | `AddJwtBearer` | Validated via `TokenValidationParameters` (issuer, audience, lifetime, signing key) |
| **Authentication Cookie** | `AddCookie` | Encrypted/signed ticket stored in a cookie |
| **Client Certificate** | Certificate authentication handler | Mutual TLS certificate validation |
| **External OAuth/OIDC** | `AddOAuth` / `AddOpenIdConnect` | Provider-issued tokens |

#### Secure Secret Loading

```csharp
builder.Services.AddAuthentication()
    .AddFacebook(options =>
    {
        options.AppId = builder.Configuration["Authentication:Facebook:AppId"]!;
        options.AppSecret = builder.Configuration["Authentication:Facebook:AppSecret"]!;
    });
```

**Syntax rules:**

- **Never store passwords in plain text**—always use a hashing mechanism (ASP.NET Core Identity's `PasswordHasher<TUser>` or a library like BCrypt/Argon2).
- **Load secrets from secure providers**—Environment Variables, User Secrets (development), Azure Key Vault, or other vaults—not `appsettings.json` committed to source control.
- **JWT signing keys** configured as symmetric keys must be Base64-encoded.

**Constraints and limitations:**

- **`KeyDerivation.Pbkdf2` directly** should **not** be used in new apps that support password sign-in; use `PasswordHasher<TUser>` instead.
- **Environment variables** are not recommended for sensitive data outside local development; use a real vault in production.
- **Bypassing `UserManager`** for password operations means passwords are not hashed, security stamps are not generated, and users cannot log in.

### Annotated Code Example (Password Hashing with ASP.NET Core Identity)

```csharp
public class AccountService
{
    private readonly UserManager<IdentityUser> _userManager;

    public AccountService(UserManager<IdentityUser> userManager)
        => _userManager = userManager;

    public async Task<IdentityResult> RegisterAsync(string email, string password)
    {
        var user = new IdentityUser { UserName = email, Email = email };
        // UserManager hashes the password with PBKDF2
        return await _userManager.CreateAsync(user, password);
    }
}
```

**Expected Output:** The password is stored as a PBKDF2 hash with a random salt; the plain-text password is never persisted. `CreateAsync` returns an `IdentityResult` indicating success or validation errors.

**Why this output?** `UserManager` delegates to `IPasswordHasher<TUser>`, which uses PBKDF2 with HMAC-SHA256, a 128-bit salt, a 256-bit subkey, and 10,000 iterations by default.

### Real-World Cases

- **User registration/login:** Passwords hashed by Identity; tokens validated by JWT bearer.
- **API-to-API calls:** Client credentials or certificates.
- **Secret management:** Azure Key Vault or User Secrets for OAuth app secrets and signing keys.

### References

- Hash passwords in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/consumer-apis/password-hashing
- Safe storage of app secrets in development – https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- OWASP Password Storage Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html

---

## 4. Authentication Schemes

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An authentication scheme is a named configuration that pairs an authentication handler with its options. |
| **Technical Definition** | A scheme is a name that corresponds to an authentication handler and the options for that handler instance. Schemes are registered by calling scheme-specific extension methods (e.g., `AddJwtBearer`, `AddCookie`) after `AddAuthentication`, which internally call `AuthenticationBuilder.AddScheme`. When a single scheme is registered, it automatically becomes the `DefaultScheme`; when multiple schemes are registered, a default must be specified or the scheme must be named in `[Authorize(AuthenticationSchemes = ...)]`. |
| **Beginner-Friendly Explanation** | A scheme is a **labeled authentication mechanism**. You can have a "Cookie" scheme and a "Bearer" scheme registered at the same time. The scheme label tells the authentication middleware which handler should process a given request. If you don't specify a scheme, the default one is used—so you only need to think about schemes when you have more than one. |

### Purposes

- **To** support multiple authentication mechanisms (cookies, JWT, OAuth, certificates) in the same application.
- **To** allow different endpoints to require different authentication methods (e.g., cookies for UI, JWT for API).
- **To** enable policy-based selection of schemes (e.g., using `AddPolicyScheme` to forward based on request characteristics).
- **To** provide a named reference for challenge and forbid behavior in authorization policies.

### Syntax Rules and Structure

#### Registering Multiple Schemes

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = "B2C_OR_AAD";
    options.DefaultChallengeScheme = "B2C_OR_AAD";
})
.AddJwtBearer("B2C", jwtOptions => { /* ... */ })
.AddJwtBearer("AAD", jwtOptions => { /* ... */ })
.AddPolicyScheme("B2C_OR_AAD", "B2C_OR_AAD", options =>
{
    options.ForwardDefaultSelector = context =>
    {
        // Select scheme based on token properties
        return "B2C";
    };
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthentication(options => ...)` | Configures default schemes (`DefaultScheme`, `DefaultChallengeScheme`, etc.). |
| `AddJwtBearer("name", ...)` | Registers a named JWT bearer scheme. Each scheme name must be unique. |
| `AddPolicyScheme("name", ...)` | Combines multiple schemes behind one logical name with forwarding logic. |

#### Selecting a Scheme in Authorization

```csharp
[Authorize(AuthenticationSchemes = "Bearer")]
public class ApiController : ControllerBase { ... }
```

**Syntax rules:**

- **Scheme names are unique**—registering the same name twice throws `InvalidOperationException`.
- **`[Authorize(AuthenticationSchemes = ...)]`** overrides the default scheme for that endpoint.
- **Policy schemes** (`AddPolicyScheme`) can forward to different handlers based on request context, enabling a single logical scheme to select among multiple physical schemes.

**Constraints and limitations:**

- **A default scheme is required when multiple schemes are registered**; without it, `[Authorize]` without an explicit scheme throws.
- **Multiple calls to `AddAuthentication`** are allowed, but options configuration can overwrite defaults; explicitly configure the properties you need.

### Annotated Code Example (Cookie + JWT for Mixed UI/API)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = CookieAuthenticationDefaults.AuthenticationScheme;
})
.AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options =>
{
    options.LoginPath = "/Account/Login";
})
.AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidAudience = builder.Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]!)),
        ClockSkew = TimeSpan.Zero
    };
});

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` sets the cookie scheme as the default for both authentication and challenge.
2. `AddCookie` registers the cookie handler with a login path.
3. `AddJwtBearer` registers the JWT bearer handler with full token validation (issuer, audience, lifetime, signing key).
4. Both schemes are available; the default is used unless a specific scheme is requested.

**Expected Output:** Browser requests without a valid cookie are redirected to `/Account/Login`. API requests with a valid JWT are authenticated via the Bearer scheme and can access `[Authorize]` endpoints. A controller can specify `[Authorize(AuthenticationSchemes = "Bearer")]` to require JWT even though cookies are the default.

**Why this output?** The default scheme handles unspecified requests; `[Authorize(AuthenticationSchemes = ...)]` explicitly selects the Bearer scheme for API endpoints.

### Real-World Cases

- **SPAs with server-rendered pages:** Cookies for the UI, JWT for API calls from JavaScript.
- **Multi-tenant or multi-provider APIs:** Two JWT schemes for different issuers, combined with a policy scheme that forwards based on the token's issuer.
- **MFA scenarios:** Two cookie handlers—one for basic identity, one for MFA-completed identity.

### References

- Authentication scheme – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/#authentication-scheme
- Authorize with a specific scheme – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/limitingidentitybyscheme
- Policy schemes in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/policy-schemes

---

## 5. Authentication Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Authentication best practices are guidelines for implementing authentication that is secure, maintainable, and aligned with ASP.NET Core's built-in protections. |
| **Technical Definition** | Best practices encompass **middleware ordering** (`UseAuthentication` before `UseAuthorization`), **fallback authorization policies** for secure-by-default behavior, **using ASP.NET Core Identity or standards-based providers** instead of custom authentication, **secure secret management** (no secrets in source code), and **avoiding custom authentication implementations** that introduce timing attacks and token-generation flaws. |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for authentication—they help you avoid security holes like leaking credentials, exposing endpoints by default, or building authentication from scratch (which is extremely hard to get right). |

### Purposes

- **To** prevent authentication bypass caused by incorrect middleware ordering.
- **To** enforce authentication **by default** using fallback policies, rather than relying on developers to remember `[Authorize]` on every endpoint.
- **To** avoid critical vulnerabilities (timing attacks, predictable tokens, insecure password reset flows) that custom authentication often introduces.
- **To** keep credentials and secrets out of source code and configuration files.
- **To** use battle-tested, standards-compliant implementations (ASP.NET Core Identity, JWT Bearer, OpenID Connect) rather than reinventing authentication.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Middleware order** | Call `UseAuthentication()` before `UseAuthorization()`. | Call `UseAuthorization()` before `UseAuthentication()`—authorization would see unauthenticated users. |
| **Fallback policy** | Configure `FallbackPolicy` with `RequireAuthenticatedUser()` for secure-by-default. | Assume configuring authentication automatically restricts access—it does not. |
| **Auth implementation** | Use ASP.NET Core Identity, JWT Bearer, OAuth 2.0/OpenID Connect via standard libraries. | Roll your own authentication (timing attacks, weak token generation, missing expiry validation). |
| **Secret management** | Load secrets from Environment Variables, User Secrets, or Azure Key Vault. | Embed connection strings, API keys, or signing keys in source code or `appsettings.json`. |
| **Password handling** | Use `UserManager` / `PasswordHasher<TUser>` for hashing and verification. | Bypass `UserManager`—passwords won't be hashed and security stamps won't be generated. |
| **Cookie security** | Set `HttpOnly`, `Secure`, and `SameSite=Strict` on authentication cookies. | Store sensitive data in cookies without encryption or security flags. |
| **HTTPS** | Enforce HTTPS and use HSTS in production. | Serve authentication endpoints over plain HTTP. |
| **Custom schemes** | Register unique scheme names; use `AddPolicyScheme` for multi-scheme forwarding. | Register the same scheme name twice (throws `InvalidOperationException`). |

### Annotated Code Example (Secure-by-Default Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Use ASP.NET Core Identity or a standards-based provider
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]!)),
            ClockSkew = TimeSpan.Zero
        };
    });

// 2. Secure-by-default: require authentication everywhere
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();

// 3. Correct middleware order
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Why this follows best practices:**

- **Standards-based JWT bearer authentication** with full validation (issuer, audience, lifetime, signing key) instead of custom token handling.
- **Fallback policy** requires authenticated users for all endpoints that do not explicitly opt out, providing secure-by-default behavior.
- **HTTPS redirection** ensures credentials are never sent over plain HTTP.
- **Correct ordering**—`UseAuthentication` precedes `UseAuthorization`.
- **Secrets are loaded from configuration** (which should be backed by a secure provider) rather than hard-coded.

### Real-World Cases

- **Public APIs with mixed endpoints:** Fallback policy requires authentication globally; `[AllowAnonymous]` on health-check and public endpoints.
- **Enterprise applications:** ASP.NET Core Identity with 2FA and account lockout, plus cookie authentication with security flags.
- **Secret rotation:** JWT signing keys loaded from Azure Key Vault, allowing rotation without code changes.

### References

- ASP.NET Core security topics – https://learn.microsoft.com/en-us/aspnet/core/security/
- Security cards: secret handling and fallback policies – https://github.com/reware-labs/securitycards
- ASP.NET Core Security Best Practices 2026 – https://offensive360.com/blog/aspnet-core-security-best-practices/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html

---

## Summary Table: Authentication Fundamentals at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Authentication** | Determine user identity | `AddAuthentication`, `UseAuthentication` | Every secured application |
| **Identity** | `ClaimsPrincipal` representing the user | `Claim`, `ClaimsIdentity`, `ClaimsPrincipal` | Whenever authorization needs user attributes |
| **Credentials** | Proof of identity (password, token, certificate) | `UserManager`, `AddJwtBearer`, `AddCookie` | Every login and token validation |
| **Authentication schemes** | Named handler configurations | `AddJwtBearer("name")`, `[Authorize(AuthenticationSchemes = ...)]` | Multiple authentication mechanisms in one app |
| **Best practices** | Secure-by-default, standards-based, correctly ordered | Fallback policy, `UseAuthentication` before `UseAuthorization`, secret vaults | Every production authentication setup |

---

## References (Full List)

- Overview of ASP.NET Core authentication – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- ASP.NET Core security topics – https://learn.microsoft.com/en-us/aspnet/core/security/
- Authentication scheme – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/#authentication-scheme
- Authorize with a specific scheme – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/limitingidentitybyscheme
- Policy schemes in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/policy-schemes
- Claims-based identity in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/claims
- Policy-based authorization – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Hash passwords in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/consumer-apis/password-hashing
- Safe storage of app secrets in development – https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- OWASP Password Storage Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html
- Security cards: secret handling and fallback policies – https://github.com/reware-labs/securitycards
- ASP.NET Core Security Best Practices 2026 – https://offensive360.com/blog/aspnet-core-security-best-practices/
- Minimal APIs quick reference (`UseAuthentication` placement) – https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
- Authentication middleware order (UseAuthentication before UseAuthorization) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/bd52e44969a1fcf29a259d5a18b4c472dbf3ee13/curriculum/week-06-auth-and-identity/lecture-notes/01-cookies-jwt-and-the-identity-system.md