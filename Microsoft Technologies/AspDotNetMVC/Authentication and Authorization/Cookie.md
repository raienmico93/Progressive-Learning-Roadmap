# ASP.NET Core Cookie Authentication: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the built-in cookie authentication handler (`Microsoft.AspNetCore.Authentication.Cookies`). Cookie authentication stores an **encrypted and signed authentication ticket** in a browser cookie, using ASP.NET Core **Data Protection** for encryption. It can be used **without ASP.NET Core Identity** (as shown here) or configured through Identity's `ConfigureApplicationCookie`. Starting with **ASP.NET Core 10**, known API endpoints no longer redirect to login pages when using cookie authentication—they return **401/403** status codes instead. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Cookie authentication is an authentication mechanism where the user's identity is serialized into an encrypted and signed cookie that the browser sends with every subsequent request. |
| **Technical Definition** | Cookie authentication is implemented by `CookieAuthenticationHandler`, registered via `AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme).AddCookie(...)`. On sign-in, `HttpContext.SignInAsync` serializes a `ClaimsPrincipal` into an `AuthenticationTicket`, encrypts and signs it using ASP.NET Core Data Protection, and writes it to a cookie. On subsequent requests, `UseAuthentication()` invokes the handler, which decrypts the cookie and populates `HttpContext.User`. The handler also manages challenge (redirect to `LoginPath` or 401), forbid (redirect to `AccessDeniedPath` or 403), and sign-out (delete the cookie) behaviour. |
| **Beginner-Friendly Explanation** | Cookie authentication is the **classic "login cookie" mechanism**. When a user logs in successfully, the server creates an encrypted cookie that says "this is Alice, and here is what she is allowed to do." The browser stores that cookie and sends it back on every request. The server decrypts it, checks that it has not been tampered with, and attaches Alice's identity to the request. When she logs out, the cookie is deleted. |

### Key Characteristics

- **Claims-based** – The cookie carries a serialized `ClaimsPrincipal` (claims such as name, role, email).
- **Encrypted and signed** – Uses ASP.NET Core Data Protection; the client cannot read or forge the ticket.
- **Stateful on the client, stateless on the server by default** – The cookie is self-contained; the server does not need a session store unless `SessionStore` is configured.
- **Challenge/forbid aware** – Unauthenticated users are redirected to `LoginPath` (web) or receive 401 (API); authenticated but unauthorized users are redirected to `AccessDeniedPath` or receive 403.
- **Lifetime-controlled** – `ExpireTimeSpan` and `SlidingExpiration` control ticket lifetime; `IsPersistent` controls whether the cookie survives browser restarts.
- **Revocable with extra infrastructure** – By default, a valid cookie remains valid until expiry; server-side revocation requires `ITicketStore` or security-stamp validation.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **middleware**, **dependency injection**, and **claims-based identity**.
- Familiarity with `Program.cs` (minimal hosting) and **Data Protection** (for multi-server deployments).

### Related Programming Areas

- **Authentication fundamentals** – Schemes, handlers, and the `IAuthenticationService`.
- **Authorization** – `[Authorize]`, policies, and fallback policies use the `ClaimsPrincipal` produced by cookie authentication.
- **ASP.NET Core Identity** – A higher-level membership system that uses cookie authentication under the hood (`ConfigureApplicationCookie`).
- **Data Protection** – Encrypts and signs the authentication ticket; must be configured consistently in web farms.
- **Antiforgery** – Cookie authentication sends credentials automatically, so state-changing endpoints need CSRF protection.

### Core Concepts / Features

1. **Cookie authentication** – The overall mechanism, registration, and middleware integration.
2. **Cookie configuration** – `CookieAuthenticationOptions`, security attributes, lifetimes, and SameSite policy.
3. **Login** – Creating a `ClaimsPrincipal` and calling `SignInAsync`.
4. **Logout** – Calling `SignOutAsync`, server-side revocation, and security-stamp invalidation.
5. **Cookie authentication best practices** – Secure attributes, HTTPS, revocation, and claims hygiene.

---

## 1. Cookie Authentication (The Mechanism)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Cookie authentication is the process of issuing an encrypted cookie containing the user's identity on login, and validating that cookie on subsequent requests to restore the user's identity. |
| **Technical Definition** | `AddCookie` registers the `CookieAuthenticationHandler` with a scheme name (default: `"Cookies"`). `UseAuthentication()` invokes the handler's `AuthenticateAsync`, which reads the cookie, decrypts the ticket using Data Protection, and produces an `AuthenticateResult`. If successful, `HttpContext.User` is set to the ticket's `ClaimsPrincipal`. The handler's `ChallengeAsync` redirects to `LoginPath` (or returns 401 for API endpoints in .NET 10+); `ForbidAsync` redirects to `AccessDeniedPath` (or returns 403). `SignInAsync` creates the ticket; `SignOutAsync` deletes the cookie. |
| **Beginner-Friendly Explanation** | Cookie authentication is the **login-cookie system**. On login, the server bakes an encrypted "identity cookie." On every request, the cookie is presented, the server decrypts it, and the user is recognized. If the cookie is missing or invalid, the server challenges the user (login redirect or 401). |

### Purposes

- **To** persist a user's identity across multiple HTTP requests without requiring re-authentication on every request.
- **To** provide a simple, browser-friendly authentication mechanism that requires no JavaScript and no token storage in the client.
- **To** integrate seamlessly with MVC, Razor Pages, and minimal APIs through the standard authentication middleware.
- **To** support challenge and forbid behaviours that differ between web pages (redirect) and APIs (status codes).
- **To** enable server-side revocation when combined with `ITicketStore` or security-stamp validation.

### Syntax Rules and Structure

#### General Syntax (Registration and Middleware)

```csharp
using Microsoft.AspNetCore.Authentication.Cookies;

var builder = WebApplication.CreateBuilder(args);

// 1. Register cookie authentication
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
        options.SlidingExpiration = true;
    });

builder.Services.AddAuthorization();
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();   // Must precede UseAuthorization
app.UseAuthorization();
app.MapDefaultControllerRoute();

app.Run();
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthentication(scheme)` | Registers the authentication service and sets the default scheme to `"Cookies"`. |
| `.AddCookie(options => ...)` | Registers the cookie handler and configures `CookieAuthenticationOptions`. |
| `UseAuthentication()` | Adds middleware that invokes the cookie handler and populates `HttpContext.User`. |
| `UseAuthorization()` | Enforces `[Authorize]` policies; must run **after** `UseAuthentication()`. |

#### Syntax Rules

- **`UseAuthentication()` must be called before `UseAuthorization()`** and **after `UseRouting()`** (if routing is explicit).
- **`UseAuthentication()` and `UseAuthorization()` must be called before endpoint mapping** (`MapControllers`, `MapRazorPages`, etc.).
- **A default scheme is required**; `CookieAuthenticationDefaults.AuthenticationScheme` has the value `"Cookies"`.
- **The authentication cookie's `IsEssential` property defaults to `true`**, so it is allowed even when a visitor has not consented to data collection.

#### Constraints and Limitations

- **Cookie authentication alone does not restrict access**—a fallback authorization policy or `[Authorize]` attributes are required.
- **API endpoints in .NET 10+ return 401/403** instead of redirecting to the login page.
- **Data Protection keys must be shared** across servers in a web farm; otherwise, cookies issued by one server cannot be decrypted by another.
- **Cookie size limits** – Browsers limit cookies to roughly 4 KB; large claim sets must be reduced or moved to `SessionStore`.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Registering Cookie Authentication and Protecting an Action

```csharp
// Program.cs
using Microsoft.AspNetCore.Authentication.Cookies;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
    });

builder.Services.AddAuthorization();
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();   // Populates HttpContext.User from the cookie
app.UseAuthorization();    // Enforces [Authorize]
app.MapDefaultControllerRoute();

app.Run();
```

```csharp
// HomeController.cs
public class HomeController : Controller
{
    public IActionResult Public() => Content("Anyone can see this.");

    [Authorize]
    public IActionResult Private() =>
        Content($"Hello, {User.Identity?.Name ?? "anonymous"}!");
}
```

**Step-by-step explanation:**

1. `AddAuthentication` sets `"Cookies"` as the default scheme.
2. `AddCookie` registers the cookie handler with login and access-denied paths.
3. `UseAuthentication()` runs on every request, decrypts the cookie (if present), and sets `HttpContext.User`.
4. `UseAuthorization()` enforces `[Authorize]` on `Private`.
5. An unauthenticated request to `Private` is challenged: the cookie handler redirects to `/Account/Login`.

**Expected Output:** `Public` returns `"Anyone can see this."` without authentication. `Private` returns `"Hello, Alice!"` for an authenticated user with a valid cookie; an unauthenticated request is redirected to `/Account/Login`.

**Why this output?** `UseAuthentication` populates `User` from the cookie; `UseAuthorization` challenges unauthenticated users using the cookie handler's challenge behaviour.

#### Example 2: Creating the Cookie on Login (ClaimsPrincipal + SignInAsync)

```csharp
public class LoginModel : PageModel
{
    [BindProperty]
    public string Email { get; set; } = "";
    [BindProperty]
    public string Password { get; set; } = "";

    public async Task<IActionResult> OnPostAsync()
    {
        // Validate credentials against your store
        var user = await AuthenticateUserAsync(Email, Password);
        if (user is null)
        {
            ModelState.AddModelError("", "Invalid login attempt.");
            return Page();
        }

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, user.Email),
            new Claim("FullName", user.FullName),
            new Claim(ClaimTypes.Role, "Administrator")
        };

        var identity = new ClaimsIdentity(
            claims, CookieAuthenticationDefaults.AuthenticationScheme);
        var principal = new ClaimsPrincipal(identity);

        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            principal);

        return LocalRedirect("/");
    }
}
```

**Expected Output:** On successful login, the response includes a `Set-Cookie` header with the encrypted authentication cookie. Subsequent requests carry the cookie, and `User.Identity.Name` is the user's email.

**Why this output?** `SignInAsync` serializes the `ClaimsPrincipal`, encrypts it with Data Protection, and appends the cookie to the response.

### Real-World Cases

- **Consumer web applications:** Cookie authentication for browser sessions with a login page and a "Remember Me" option.
- **Internal dashboards:** Cookie-based sign-in with role claims for admin sections.
- **Mixed MVC + API apps:** Cookies for the UI, with API endpoints returning 401/403 instead of redirects (ASP.NET Core 10+).

### References

- Use cookie authentication without ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie
- CookieAuthenticationDefaults.AuthenticationScheme – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationdefaults.authenticationscheme
- Authentication middleware order (UseAuthentication before UseAuthorization) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/bd52e44969a1fcf29a259d5a18b4c472dbf3ee13/curriculum/week-06-auth-and-identity/lecture-notes/01-cookies-jwt-and-the-identity-system.md

---

## 2. Cookie Configuration

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Cookie configuration is the set of options that control the authentication cookie's name, security attributes, lifetime, SameSite policy, and validation behaviour. |
| **Technical Definition** | Configuration is performed through the `CookieAuthenticationOptions` class, passed to `AddCookie(options => ...)`. Key properties include `LoginPath`, `LogoutPath`, `AccessDeniedPath`, `ExpireTimeSpan`, `SlidingExpiration`, `Cookie.Name`, `Cookie.HttpOnly`, `Cookie.SecurePolicy`, `Cookie.SameSite`, `Cookie.IsEssential`, `EventsType`, and `SessionStore`. **`Cookie.Expiration` is unsupported and triggers `OptionsValidationException`**; use `ExpireTimeSpan` or `AuthenticationProperties.ExpiresUtc` instead. |
| **Beginner-Friendly Explanation** | Cookie configuration is **tuning the cookie**. You decide how long it lives, whether it survives a browser restart, whether JavaScript can read it, whether it is only sent over HTTPS, and what happens when the user is not logged in. These settings determine the security and usability of the login session. |

### Purposes

- **To** control the authentication cookie's **lifetime** (`ExpireTimeSpan`, `SlidingExpiration`) and whether it persists across browser sessions (`IsPersistent`).
- **To** enforce **security attributes** (`HttpOnly`, `SecurePolicy`, `SameSite`) that protect the cookie from theft and cross-site attacks.
- **To** define **redirect paths** for challenge (`LoginPath`) and forbid (`AccessDeniedPath`) scenarios.
- **To** specify a **custom cookie name** (e.g., `__Host-AuthCookie`) for clarity and cookie-prefix security.
- **To** enable **server-side validation** via `EventsType` (`OnValidatePrincipal`) and **server-side ticket storage** via `SessionStore`.

### Syntax Rules and Structure

#### General Syntax (Full Configuration Example)

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/Forbidden";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
        options.SlidingExpiration = true;

        options.Cookie.Name = "__Host-AuthCookie";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
        options.Cookie.IsEssential = true;

        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });
```

**Component breakdown:**

| Option | Purpose |
|--------|---------|
| `LoginPath` | Redirect target for challenge (default: `/Account/Login`). |
| `LogoutPath` | Redirect target after sign-out (if configured). |
| `AccessDeniedPath` | Redirect target for forbid (authenticated but unauthorized). |
| `ExpireTimeSpan` | How long the authentication ticket remains valid. |
| `SlidingExpiration` | Whether activity renews the ticket (activity renews, not an absolute limit). |
| `Cookie.Name` | Cookie name; use `__Host-` prefix for host-bound security. |
| `Cookie.HttpOnly` | Prevents JavaScript access to the cookie. |
| `Cookie.SecurePolicy` | `Always` ensures the cookie is only sent over HTTPS. |
| `Cookie.SameSite` | `Lax` (default via cookie policy), `Strict`, or `None`. |
| `Cookie.IsEssential` | Allows the cookie even without GDPR consent (default `true` for auth cookies). |
| `EventsType` | Enables custom `CookieAuthenticationEvents` (e.g., `OnValidatePrincipal`). |
| `SessionStore` | `ITicketStore` for server-side ticket storage and revocation. |

#### Syntax Rules

- **`Cookie.Expiration` is unsupported**—setting it throws `OptionsValidationException`; use `ExpireTimeSpan` or `AuthenticationProperties.ExpiresUtc`.
- **`ExpireTimeSpan` controls the ticket lifetime**, not the browser's cookie deletion time; the browser may keep the cookie longer, but the server rejects it after expiry.
- **`SlidingExpiration = true`** renews the ticket when more than half the `ExpireTimeSpan` has elapsed; it is **not** an absolute session limit.
- **`Cookie.SameSite`** defaults to `Lax` through the cookie policy middleware; `Strict` is more secure but breaks OAuth2/cross-origin flows.
- **`Cookie.Name` with `__Host-` prefix** requires `Secure`, `Path=/`, and no `Domain`.

#### Constraints and Limitations

- **`Cookie.Expiration` is deprecated/unsupported**—do not use it.
- **`SlidingExpiration` does not cap session length**—a continuously active user can stay signed in indefinitely unless an absolute `ExpiresUtc` is set.
- **`SameSite=None` requires `Secure`**—browsers reject `SameSite=None` cookies that are not marked `Secure`.
- **`Cookie.SecurePolicy = Always`** will break local HTTP development unless HTTPS is enabled.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Security-Focused Configuration

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.Name = "__Host-AuthCookie";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
        options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
        options.SlidingExpiration = true;
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
    });
```

**Step-by-step explanation:**

1. `__Host-AuthCookie` binds the cookie to the exact host and requires `Secure` and `Path=/`.
2. `HttpOnly = true` prevents JavaScript from reading the cookie (mitigates XSS-based cookie theft).
3. `SecurePolicy = Always` ensures the cookie is only sent over HTTPS.
4. `SameSite = Lax` allows top-level navigation while blocking most cross-site POST requests.
5. `ExpireTimeSpan = 30 minutes` with `SlidingExpiration = true` renews the ticket on activity.

**Expected Output:** The authentication cookie is named `__Host-AuthCookie`, is marked `Secure` and `HttpOnly`, uses `SameSite=Lax`, and expires 30 minutes after the last renewal.

**Why this output?** These options are applied to the cookie written by `SignInAsync` and validated on every request.

#### Example 2: Persistent vs. Session Cookie

```csharp
// Session cookie (default): deleted when the browser closes
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal);

// Persistent cookie ("Remember Me"): survives browser restart
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties { IsPersistent = true });
```

**Expected Output:** The first cookie is session-based; the second persists across browser restarts (subject to `ExpireTimeSpan`).

**Why this output?** `IsPersistent = true` tells the handler to set an absolute cookie lifetime matching the ticket lifetime; otherwise, the cookie is session-based.

#### Example 3: Absolute Expiration with `ExpiresUtc`

```csharp
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTimeOffset.UtcNow.AddMinutes(20)
    });
```

**Expected Output:** The cookie expires at exactly 20 minutes, overriding any `ExpireTimeSpan` and ignoring sliding expiration.

**Why this output?** `ExpiresUtc` sets an absolute expiration on the ticket and cookie; it overrides `ExpireTimeSpan`.

### Real-World Cases

- **High-security apps:** `__Host-` prefix, `SecurePolicy.Always`, `HttpOnly`, short `ExpireTimeSpan`, and `SameSite=Strict` where cross-origin flows are not needed.
- **"Remember Me" flows:** `IsPersistent = true` with a longer `ExpireTimeSpan`.
- **Multi-server deployments:** Shared Data Protection key ring plus `SessionStore` for revocation.

### References

- CookieAuthenticationOptions class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions
- Configure cookie authentication options and session lifetimes – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/session-management.md
- Cookie.Expiration unsupported (OptionsValidationException) – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/api-contract-misuse.md
- Cookie policy middleware (SameSite matrix) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#cookie-policy-middleware
- OWASP DotNet Security Cheat Sheet (cookie attributes) – https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md

---

## 3. Login

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Login is the process of validating a user's credentials and, on success, creating an authentication cookie that establishes the user's identity for subsequent requests. |
| **Technical Definition** | Login is implemented by constructing a `ClaimsPrincipal` (with one or more `ClaimsIdentity` objects containing claims), then calling `HttpContext.SignInAsync(scheme, principal, authProperties)`. `SignInAsync` serializes the principal into an `AuthenticationTicket`, encrypts and signs it using Data Protection, and writes it to the response as an authentication cookie. The credentials themselves must be validated against a trusted store **before** calling `SignInAsync`. |
| **Beginner-Friendly Explanation** | Login is the **"prove who you are, then get your cookie"** step. The user submits a username and password. Your code checks them against the database. If they match, you build the user's identity (name, roles, email), and `SignInAsync` turns that identity into an encrypted cookie that the browser stores. |

### Purposes

- **To** validate user credentials against a trusted store (database, external provider).
- **To** create a `ClaimsPrincipal` containing the claims that authorization will use.
- **To** issue an encrypted, signed authentication cookie via `SignInAsync`.
- **To** support optional "Remember Me" persistence and absolute expiration.
- **To** redirect the user to the originally requested URL after successful login.

### Syntax Rules and Structure

#### General Syntax (Login Handler)

```csharp
public async Task<IActionResult> OnPostAsync(string? returnUrl = null)
{
    if (!ModelState.IsValid) return Page();

    var user = await AuthenticateUserAsync(Input.Email, Input.Password);
    if (user is null)
    {
        ModelState.AddModelError("", "Invalid login attempt.");
        return Page();
    }

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, user.Email),
        new Claim(ClaimTypes.Role, "User")
    };

    var identity = new ClaimsIdentity(
        claims, CookieAuthenticationDefaults.AuthenticationScheme);

    await HttpContext.SignInAsync(
        CookieAuthenticationDefaults.AuthenticationScheme,
        new ClaimsPrincipal(identity),
        new AuthenticationProperties { IsPersistent = false });

    return LocalRedirect(returnUrl ?? "/");
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AuthenticateUserAsync` | Validates credentials against your store (not shown; implement securely). |
| `Claim` list | The user's attributes (name, role, etc.). |
| `ClaimsIdentity` | Wraps the claims with the authentication type (the scheme name). |
| `ClaimsPrincipal` | The user object passed to `SignInAsync`. |
| `AuthenticationProperties` | Optional: `IsPersistent`, `ExpiresUtc`, `AllowRefresh`, `RedirectUri`. |
| `SignInAsync` | Encrypts the ticket and writes the cookie. |

#### Syntax Rules

- **Credentials must be validated before `SignInAsync`**—`SignInAsync` does not check passwords.
- **The `ClaimsIdentity` authentication type** should match the scheme name (conventionally `CookieAuthenticationDefaults.AuthenticationScheme`).
- **`AuthenticationProperties.IsPersistent = true`** creates a persistent cookie; `ExpiresUtc` sets an absolute expiry.
- **`LocalRedirect`** should be used for return URLs to prevent open redirects.

#### Constraints and Limitations

- **Do not store sensitive data in claims**—the cookie is encrypted, but claims are still client-side data; avoid secrets, passwords, or large payloads.
- **Cookie size limits**—large claim sets can exceed the ~4 KB browser limit; reduce claims or use `SessionStore`.
- **`SignInAsync` requires the authentication middleware to be registered** (`AddAuthentication`/`AddCookie`); otherwise, it throws.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Login with Claims and Redirect

```csharp
public class LoginModel : PageModel
{
    [BindProperty] public string Email { get; set; } = "";
    [BindProperty] public string Password { get; set; } = "";

    public async Task<IActionResult> OnPostAsync(string? returnUrl = null)
    {
        if (!ModelState.IsValid) return Page();

        var user = await _userService.ValidateAsync(Email, Password);
        if (user is null)
        {
            ModelState.AddModelError("", "Invalid login attempt.");
            return Page();
        }

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, user.Email),
            new Claim(ClaimTypes.Role, user.IsAdmin ? "Admin" : "User")
        };

        var identity = new ClaimsIdentity(
            claims, CookieAuthenticationDefaults.AuthenticationScheme);

        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            new ClaimsPrincipal(identity));

        return LocalRedirect(returnUrl ?? "/");
    }
}
```

**Step-by-step explanation:**

1. Model validation ensures the form is complete.
2. `ValidateAsync` checks the credentials against the store.
3. On failure, an error is added and the page is re-rendered.
4. On success, claims are built and wrapped in a `ClaimsIdentity`/`ClaimsPrincipal`.
5. `SignInAsync` encrypts the ticket and writes the cookie.
6. The user is redirected to the return URL or the home page.

**Expected Output:** A valid login returns a redirect to `/` with a `Set-Cookie` header containing the encrypted authentication cookie. An invalid login re-renders the page with “Invalid login attempt.”

**Why this output?** `SignInAsync` creates the cookie only after credentials are validated; `LocalRedirect` prevents open-redirect attacks.

#### Example 2: Persistent "Remember Me" Cookie

```csharp
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTimeOffset.UtcNow.AddDays(14)
    });
```

**Expected Output:** The cookie persists across browser restarts and expires at exactly 14 days, regardless of activity.

**Why this output?** `IsPersistent = true` makes the cookie non-session; `ExpiresUtc` sets an absolute expiration that overrides `ExpireTimeSpan`.

### Real-World Cases

- **Standard login page:** Email/password validation followed by cookie issuance.
- **"Remember Me" checkbox:** Sets `IsPersistent = true` and a longer `ExpiresUtc`.
- **SSO callback:** After external authentication, build claims and call `SignInAsync` to issue the local cookie.

### References

- Create an authentication cookie (SignInAsync) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#create-an-authentication-cookie
- AuthenticationProperties (IsPersistent, ExpiresUtc) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationproperties
- Persistent cookies – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#persistent-cookies
- Absolute cookie expiration – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#absolute-cookie-expiration

---

## 4. Logout

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Logout is the process of removing the authentication cookie and, when configured, invalidating the server-side ticket so the user is no longer authenticated. |
| **Technical Definition** | Logout is implemented by calling `HttpContext.SignOutAsync(scheme)`, which instructs the cookie handler to issue a `Set-Cookie` header that expires (deletes) the authentication cookie. By default, the server does not track issued cookies, so a stolen or copied cookie remains valid until its ticket expires. **Server-side revocation** requires `CookieAuthenticationOptions.SessionStore` (`ITicketStore`) or security-stamp validation via `OnValidatePrincipal`. |
| **Beginner-Friendly Explanation** | Logout is **deleting the login cookie**. When the user clicks "Log out," the server tells the browser to delete the cookie. However, if someone copied the cookie before logout, it may still work until it expires—unless you use a server-side store or security stamps to revoke it. |

### Purposes

- **To** delete the authentication cookie from the browser so subsequent requests are anonymous.
- **To** provide a clean sign-out experience that redirects the user to a public page or the login page.
- **To** invalidate server-side sessions when `SessionStore` or security-stamp validation is configured.
- **To** support "sign out everywhere" scenarios by revoking all tickets for a user.

### Syntax Rules and Structure

#### General Syntax (Sign-Out Handler)

```csharp
public async Task<IActionResult> OnGetAsync()
{
    await HttpContext.SignOutAsync(
        CookieAuthenticationDefaults.AuthenticationScheme);

    return RedirectToPage("/Account/Login");
}
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `SignOutAsync(scheme)` | Deletes the authentication cookie and, if configured, invokes server-side revocation. |
| `RedirectToPage` / `LocalRedirect` | Sends the user to a public page after logout. |

#### Syntax Rules

- **`SignOutAsync` deletes the cookie** by sending an expired `Set-Cookie` header.
- **If a non-default scheme is used**, pass the scheme name to `SignOutAsync`; otherwise, the default scheme is used.
- **`SignOutAsync` should be followed by a redirect** so the browser performs a new request without the cookie.

#### Constraints and Limitations

- **Deleting the cookie does not invalidate a copied ticket**—the server is not notified when the browser deletes the cookie.
- **Server-side revocation requires additional configuration** (`SessionStore` or `OnValidatePrincipal` with security stamps).
- **Session-based cookies may persist until the browser closes** if `SignOutAsync` is not called; always call `SignOutAsync` explicitly.

### Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Logout

```csharp
public class LogoutModel : PageModel
{
    public async Task<IActionResult> OnGetAsync()
    {
        await HttpContext.SignOutAsync(
            CookieAuthenticationDefaults.AuthenticationScheme);

        return RedirectToPage("/Index");
    }
}
```

**Step-by-step explanation:**

1. `SignOutAsync` tells the cookie handler to expire the authentication cookie.
2. The response includes a `Set-Cookie` header that deletes the cookie in the browser.
3. The user is redirected to a public page.

**Expected Output:** The authentication cookie is deleted; subsequent requests are anonymous. The user lands on `/Index`.

**Why this output?** `SignOutAsync` issues the deletion header; the redirect ensures the browser makes a clean request without the cookie.

#### Example 2: Server-Side Revocation with `SessionStore`

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.SessionStore = sp.GetRequiredService<ITicketStore>();
        options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
    });

// On logout, the SessionStore entry is also removed
await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
```

**Expected Output:** The cookie is deleted and the server-side ticket entry is removed, so a copied cookie can no longer be validated.

**Why this output?** With `SessionStore`, the cookie contains only a session key; the actual ticket is stored server-side. Signing out removes the server-side entry, revoking the ticket.

#### Example 3: Security-Stamp Validation (Sign Out Everywhere)

```csharp
public class CustomCookieEvents : CookieAuthenticationEvents
{
    private readonly UserManager<IdentityUser> _userManager;

    public CustomCookieEvents(UserManager<IdentityUser> userManager)
        => _userManager = userManager;

    public override async Task ValidatePrincipal(
        CookieValidatePrincipalContext context)
    {
        var user = await _userManager.GetUserAsync(context.Principal!);
        if (user is null || !await _userManager.VerifySecurityStampAsync(user))
        {
            context.RejectPrincipal();
            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

**Expected Output:** If the security stamp has changed (e.g., after a password change), the cookie is rejected and the user is signed out.

**Why this output?** `OnValidatePrincipal` runs on each request (subject to the validator interval) and rejects cookies whose security stamp no longer matches.

### Real-World Cases

- **Standard logout button:** `SignOutAsync` followed by a redirect.
- **"Sign out everywhere":** `UpdateSecurityStampAsync` to invalidate all existing cookies, combined with security-stamp validation.
- **Session revocation:** `ITicketStore` to remove server-side tickets on logout or administrative action.

### References

- Sign out (SignOutAsync) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#sign-out
- React to back-end changes (ValidatePrincipal) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#react-to-back-end-changes
- SessionStore / ITicketStore for revocation – https://github.com/dotnet/aspnetcore/issues/67794
- Security stamp validation – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#react-to-back-end-changes

---

## 5. Cookie Authentication Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | Cookie authentication best practices are guidelines for configuring and using cookie authentication securely, maintainably, and in alignment with ASP.NET Core’s built-in protections. |
| **Technical Definition** | Best practices encompass **HTTPS enforcement** (`SecurePolicy.Always`), **HttpOnly cookies**, **appropriate SameSite policy**, **`__Host-` cookie prefix**, **short session lifetimes with absolute limits**, **server-side revocation** via `SessionStore` or security stamps, **minimal claims**, **antiforgery protection**, **correct middleware ordering**, and **fallback authorization policies**. |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for cookie authentication—they help you avoid stolen cookies, cross-site attacks, sessions that never expire, and the false sense of security that comes from configuring authentication without enforcing authorization. |

### Purposes

- **To** protect the authentication cookie from interception (`Secure`), script access (`HttpOnly`), and cross-site requests (`SameSite`).
- **To** bound session lifetime with both **idle** (`ExpireTimeSpan`) and **absolute** (`ExpiresUtc`) limits.
- **To** enable **revocation** so compromised or copied cookies stop working before natural expiry.
- **To** keep the cookie small and free of sensitive data.
- **To** enforce authentication by default with a fallback policy and protect state-changing endpoints with antiforgery tokens.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **HTTPS** | Set `Cookie.SecurePolicy = CookieSecurePolicy.Always`; enforce HTTPS/HSTS in production. | Allow the authentication cookie over plain HTTP. |
| **HttpOnly** | Set `Cookie.HttpOnly = true`. | Store authentication cookies accessible to JavaScript. |
| **SameSite** | Use `SameSiteMode.Lax` for most apps; `Strict` if no cross-origin flows are needed. | Use `SameSite=None` without `Secure` (browsers reject it). |
| **Cookie name** | Use `__Host-` prefix (e.g., `__Host-AuthCookie`). | Use a generic name without the prefix where host-binding matters. |
| **Lifetime** | Set a short `ExpireTimeSpan` and, for high-security apps, an absolute `ExpiresUtc`. | Rely on sliding expiration alone—it does not cap session length. |
| **Revocation** | Use `SessionStore` (`ITicketStore`) or security-stamp validation for immediate revocation. | Assume `SignOutAsync` invalidates copied cookies. |
| **Claims** | Store only necessary claims; keep the cookie under ~4 KB. | Store secrets, large payloads, or sensitive data in claims. |
| **Middleware order** | `UseAuthentication()` before `UseAuthorization()`, after `UseRouting()`. | Call `UseAuthorization()` before `UseAuthentication()`. |
| **Authorization** | Use a fallback policy requiring authentication; mark public endpoints `[AllowAnonymous]`. | Assume cookie authentication restricts access by itself. |
| **Antiforgery** | Protect state-changing endpoints with antiforgery tokens. | Exempt endpoints from antiforgery without verifying why. |
| **Version awareness** | In .NET 10+, API endpoints return 401/403 instead of redirecting. | Assume all unauthenticated requests redirect to login. |

### Annotated Code Example (Best-Practice Setup)

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.Name = "__Host-AuthCookie";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
        options.Cookie.IsEssential = true;

        options.ExpireTimeSpan = TimeSpan.FromMinutes(30);
        options.SlidingExpiration = true;

        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";

        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

builder.Services.AddScoped<CustomCookieAuthenticationEvents>();
builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Why this follows best practices:**

- **`__Host-` prefix** binds the cookie to the exact host and requires `Secure` and `Path=/`.
- **`HttpOnly` and `SecurePolicy.Always`** protect the cookie from script access and interception.
- **`SameSite=Lax`** blocks most cross-site POSTs while allowing normal navigation.
- **Short `ExpireTimeSpan` with sliding expiration** limits idle sessions.
- **`EventsType`** enables security-stamp validation for revocation.
- **Fallback policy** requires authentication for all endpoints by default.
- **Correct middleware order**—authentication before authorization.

### Real-World Cases

- **Public-facing web apps:** `__Host-` cookie, `Secure`, `HttpOnly`, `SameSite=Lax`, 30-minute idle timeout, fallback policy.
- **High-security admin panels:** `SameSite=Strict`, absolute `ExpiresUtc`, `SessionStore` for revocation, security-stamp validation.
- **API endpoints in .NET 10+:** Cookie authentication returns 401/403 instead of redirecting, so API clients are not confused by HTML login redirects.

### References

- OWASP DotNet Security Cheat Sheet (authentication cookies) – https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Security cards: cookie security policies and session stores – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/session-management.md
- Security cards: cookie lifetime using ExpireTimeSpan – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/api-contract-misuse.md
- Cookie policy middleware (SameSite) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#cookie-policy-middleware
- API endpoint authentication behavior in ASP.NET Core 10 – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/api-endpoint-auth
- Fallback authorization policy – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies

---

## Summary Table: Cookie Authentication at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **Cookie authentication** | Persist identity across requests | `AddAuthentication("Cookies").AddCookie()` | Browser-based sessions |
| **Cookie configuration** | Security, lifetime, and behaviour | `CookieAuthenticationOptions` (`ExpireTimeSpan`, `Cookie.HttpOnly`, `SecurePolicy`, `SameSite`) | Every cookie authentication setup |
| **Login** | Validate credentials and issue cookie | `SignInAsync(scheme, principal, authProperties)` | After successful credential validation |
| **Logout** | Delete cookie and revoke ticket | `SignOutAsync(scheme)`; `SessionStore` or security stamps for revocation | On user sign-out or "sign out everywhere" |
| **Best practices** | Secure, revocable, bounded sessions | `__Host-` prefix, `HttpOnly`, `Secure`, `SameSite`, fallback policy, antiforgery | Every production deployment |

---

## References (Full List)

- Use cookie authentication without ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie
- CookieAuthenticationOptions class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions
- CookieAuthenticationDefaults.AuthenticationScheme – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationdefaults.authenticationscheme
- AuthenticationProperties class – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationproperties
- SignInAsync – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationhttpcontextextensions.signinasync
- SignOutAsync – https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationhttpcontextextensions.signoutasync
- Cookie policy middleware (SameSite matrix) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#cookie-policy-middleware
- Persistent cookies – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#persistent-cookies
- Absolute cookie expiration – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#absolute-cookie-expiration
- React to back-end changes (ValidatePrincipal) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie#react-to-back-end-changes
- API endpoint authentication behavior in ASP.NET Core 10 – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/api-endpoint-auth
- OWASP DotNet Security Cheat Sheet – https://github.com/owasp/cheatsheetseries/blob/757270a26306e5edda9957bb7f86cfcde03c30f9/cheatsheets/DotNet_Security_Cheat_Sheet.md
- Security cards: session management (cookie security, SessionStore, security stamps) – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/session-management.md
- Security cards: cookie lifetime using ExpireTimeSpan – https://github.com/reware-labs/securitycards/blob/e711b0a76b826ad6ef6459708d9e49fc4573f179/data/c%23/asp-net/v10-0-10/api-contract-misuse.md
- SessionStore / ITicketStore for revocation – https://github.com/dotnet/aspnetcore/issues/67794
- Cookie name encoding removed (security) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie
- Fallback authorization policy – https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies
- Authentication middleware order (UseAuthentication before UseAuthorization) – https://github.com/CODECRUNCHWORLDWIDE/C9-CRUNCH-SHARP/blob/bd52e44969a1fcf29a259d5a18b4c472dbf3ee13/curriculum/week-06-auth-and-identity/lecture-notes/01-cookies-jwt-and-the-identity-system.md