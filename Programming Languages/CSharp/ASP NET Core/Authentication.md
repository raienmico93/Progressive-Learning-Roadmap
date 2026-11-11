# C# Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Authentication is the process of verifying that a user, service, or application is who or what it claims to be. In ASP.NET Core, authentication is handled by **authentication schemes**—middleware components that examine incoming requests for credentials (cookies, tokens, certificates) and construct a **`ClaimsPrincipal`** representing the authenticated identity .

**Technical Definition**
ASP.NET Core authentication is built around **`IAuthenticationScheme`**, **`IAuthenticationHandler`**, and **`AuthenticationService`**. When a request arrives, `UseAuthentication()` middleware invokes the default authentication scheme’s handler, which attempts to authenticate the request. On success, `HttpContext.User` is populated with a `ClaimsPrincipal` containing **claims** (name, role, permissions). Authentication schemes are registered via `AddAuthentication()` and specific handlers such as `AddCookie()`, `AddJwtBearer()`, and `AddOpenIdConnect()` . The framework supports multiple simultaneous schemes (e.g., cookies for UI, JWT for APIs) and explicit scheme selection via `[Authorize(AuthenticationSchemes = "...")]` .

**Beginner-Friendly Explanation**
Authentication is **“proving who you are.”** When you log into a website with a username and password, the server checks your credentials and, if valid, gives you a **cookie or a token**. On every subsequent request, you present that cookie or token, and the server knows it is still you. ASP.NET Core provides built-in “authentication schemes” for the most common patterns: **cookies** (for browser-based apps), **JWT bearer tokens** (for APIs), and **OpenID Connect** (for delegating login to an external provider like Microsoft, Google, or IdentityServer).

---

### Key Characteristics

- **Scheme-based** — Authentication is configured as named schemes; multiple schemes can coexist .
- **Claims-based identity** — Authenticated users are represented as a `ClaimsPrincipal` with claims (name, role, etc.).
- **Challenge and forbid** — Schemes can issue challenges (e.g., redirect to login) and forbid (403) responses.
- **Cookie vs. token** — Cookies are browser-friendly and automatic; tokens are stateless and cross-domain .
- **Delegated authentication** — OIDC/OAuth2 allow external providers to handle credential verification.
- **First-party identity** — ASP.NET Core Identity provides a complete user management system .

---

### Prerequisites

- Basic understanding of **ASP.NET Core** (`Program.cs`, middleware, DI).
- Familiarity with **HTTP** (headers, cookies, status codes).
- Knowledge of **claims** and **`ClaimsPrincipal`**.
- Awareness of **HTTPS** and secure cookie/token transmission.
- Understanding of **OAuth 2.0** and **OpenID Connect** concepts (helpful).

---

### Related Programming Areas

- **Authorization** — What an authenticated user is allowed to do.
- **ASP.NET Core Identity** — First-party user and password management.
- **OAuth 2.0 / OpenID Connect** — Delegated authentication and authorization.
- **JWT** — Token format commonly used for API authentication.
- **Azure AD / Microsoft Entra ID** — Cloud identity provider.

---

### Core Concepts / Features

1. Authentication (Overarching Concept)
2. Cookie Authentication
3. JWT Authentication
4. OAuth2
5. OpenID Connect (OIDC)
6. Azure AD (Microsoft Entra ID)
7. IdentityServer (Duende)
8. ASP.NET Core Identity
9. Authentication Best Practices

---

## 1. Authentication (Overarching Concept)

### Definitions

**Core Definition:** Authentication is the framework’s mechanism for verifying identity and populating `HttpContext.User`.

**Technical Definition:** Authentication is implemented via **authentication schemes**. A scheme pairs a name with an `AuthenticationHandler` that reads credentials from the request and produces an `AuthenticationTicket` (containing a `ClaimsPrincipal` and properties). The `AuthenticationMiddleware` invokes the default scheme’s handler and, on success, assigns the principal to `HttpContext.User`. Multiple schemes can be registered, and the default scheme is used unless another is explicitly requested .

**Beginner-Friendly Explanation:** Authentication is **the “who are you?” step**. The framework checks your cookie or token and, if valid, attaches your identity to the request.

---

### Purposes

- To **verify identity** for every incoming request.
- To **populate `HttpContext.User`** with claims.
- To **support multiple authentication mechanisms** in one app.
- To **enable authorization** based on verified identity.
- To **integrate with external identity providers**.

---

### Syntax Rules and Structure

```csharp
// Program.cs
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options => { /* ... */ });

app.UseAuthentication();
app.UseAuthorization();
```

**Component breakdown**

- `AddAuthentication()` — Registers authentication services.
- `DefaultScheme` — The scheme used by `UseAuthentication()`.
- `AddCookie()`, `AddJwtBearer()`, `AddOpenIdConnect()` — Scheme handlers.
- `UseAuthentication()` — Adds the authentication middleware .

#### Syntax Rules

- `UseAuthentication()` must be called before `UseAuthorization()` and before endpoints .
- The default scheme is used unless `[Authorize(AuthenticationSchemes = "...")]` overrides it .
- Schemes can be combined (e.g., cookies for UI, JWT for APIs) .

#### Constraints and Limitations

- Authentication does not perform authorization; both are required for secured endpoints.
- Incorrect middleware order breaks authentication.

---

### Annotated Code Examples and Expected Outputs

#### Example: Cookie + JWT Schemes

```csharp
builder.Services.AddAuthentication()
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
    })
    .AddJwtBearer(options =>
    {
        options.Authority = "https://localhost:5001";
        options.Audience = "https://localhost:5000";
    });
```

**Expected Output (conceptual)**
```
// Browser requests use cookies; API requests with Bearer tokens use JWT.
```

**Why this structure is used:** It supports both browser-based and API-based clients in the same application .

---

### Real-World Cases

- **Web apps** — Cookie authentication for browser sessions.
- **APIs** — JWT bearer authentication for stateless clients.
- **Hybrid apps** — Cookies for UI, tokens for JavaScript API calls .

---

### References

- Overview of ASP.NET Core Authentication — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- Choose an identity management solution — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/how-to-choose-identity-solution

---

## 2. Cookie Authentication

### Definitions

**Core Definition:** Cookie authentication stores the user’s identity in an encrypted cookie that is automatically sent by the browser on every request.

**Technical Definition:** `CookieAuthenticationHandler` creates an encrypted `AuthenticationTicket`, serializes it into a cookie, and sets it on the response. On subsequent requests, the handler decrypts the cookie, validates it, and reconstructs the `ClaimsPrincipal`. Cookies are configured with `LoginPath`, `AccessDeniedPath`, `ExpireTimeSpan`, `SlidingExpiration`, and security flags (`HttpOnly`, `Secure`, `SameSite`) .

**Beginner-Friendly Explanation:** Cookie authentication is **the browser’s “wristband.”** After login, the server gives the browser a secure cookie; the browser sends it back automatically, and the server knows who you are.

---

### Purposes

- To **maintain user sessions** in browser-based applications.
- To **avoid sending credentials** on every request.
- To provide **automatic** authentication by the browser.
- To support **sliding expiration** and **persistent sessions**.
- To integrate with **ASP.NET Core Identity** and external login providers.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/Forbidden";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
        options.SlidingExpiration = true;
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
    });
```

**Component breakdown**

- `AddCookie()` — Registers the cookie scheme.
- `LoginPath` — Redirect target for unauthenticated users.
- `ExpireTimeSpan` — Cookie lifetime.
- `SlidingExpiration` — Renews the cookie on activity.
- `HttpOnly` — Prevents JavaScript access.
- `SecurePolicy` — Requires HTTPS.
- `SameSite` — CSRF protection.

#### Syntax Rules

- Cookie authentication requires `UseAuthentication()` and `UseAuthorization()`.
- Cookies are encrypted and signed using the **Data Protection** system.
- `LoginPath` is used when a challenge is issued.

#### Constraints and Limitations

- Cookies are domain-bound; not suitable for cross-domain APIs .
- Cookies must be protected with HTTPS and `SameSite` to prevent theft and CSRF.
- Data Protection keys must be persisted in multi-instance deployments .

---

### Annotated Code Examples and Expected Outputs

#### Example: Sign-In and Sign-Out

```csharp
// Sign in
var claims = new List<Claim>
{
    new(ClaimTypes.Name, "alice"),
    new(ClaimTypes.Role, "Admin")
};
var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(identity));

// Sign out
await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
```

**Expected Output (conceptual)**
```
// Set-Cookie header issued on sign-in; cookie cleared on sign-out.
```

**Why this pattern is used:** `SignInAsync` creates the encrypted ticket and sets the cookie; `SignOutAsync` deletes it.

---

### Real-World Cases

- **MVC / Razor Pages apps** — Session management for logged-in users.
- **ASP.NET Core Identity** — Uses cookie authentication by default.
- **SSO with OIDC** — Cookie stores the local session after OIDC login .

---

### References

- Use cookie authentication without ASP.NET Core Identity — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie
- Cookie authentication handler — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie

---

## 3. JWT Authentication

### Definitions

**Core Definition:** JWT (JSON Web Token) authentication uses a signed, self-contained token to authenticate requests, typically in APIs.

**Technical Definition:** JWT bearer authentication is implemented by `JwtBearerHandler` (package `Microsoft.AspNetCore.Authentication.JwtBearer`). The handler extracts the token from the `Authorization: Bearer <token>` header, validates the signature using the issuer’s public key, checks expiration, issuer, and audience, and constructs the `ClaimsPrincipal` . Tokens are stateless—no server-side session is stored.

**Beginner-Friendly Explanation:** JWT authentication is **a stamped passport**. The server issues a signed token containing your claims; you present it on every request, and the server verifies the stamp without storing session state.

---

### Purposes

- To **authenticate APIs** statelessly.
- To **avoid server-side session storage**.
- To enable **cross-domain** and **mobile** clients.
- To support **distributed systems** where any node can validate the token.
- To integrate with **OAuth 2.0 / OIDC** providers that issue JWTs .

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://identity.example.com";
        options.Audience = "https://api.example.com";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });
```

**Component breakdown**

- `AddJwtBearer()` — Registers the JWT handler.
- `Authority` — The token issuer (OIDC discovery endpoint).
- `Audience` — The intended recipient of the token.
- `TokenValidationParameters` — Validation rules.

#### Syntax Rules

- Tokens are read from the `Authorization` header by default.
- `Authority` enables automatic key discovery via OIDC metadata .
- `Audience` must match the `aud` claim.

#### Constraints and Limitations

- Tokens cannot be revoked before expiration unless additional infrastructure exists.
- Signing keys must be protected and rotated.
- Large claims increase token size.

---

### Annotated Code Examples and Expected Outputs

#### Example: Securing an Endpoint with JWT

```csharp
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok(new[] { "Widget", "Gadget" });
}
```

**Expected Output**

- Request with valid `Bearer` token → `200 OK` with data.
- Request without token → `401 Unauthorized`.

**Why this output occurs:** The `[Authorize]` attribute triggers the JWT handler to validate the token before the action executes.

---

### Real-World Cases

- **REST APIs** — Stateless authentication for SPAs, mobile apps, and services.
- **Microservices** — Tokens validated locally by each service.
- **API gateways** — JWT validation at the edge.

---

### References

- Configure JWT bearer authentication in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication
- JWT bearer authentication — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/

---

## 4. OAuth2

### Definitions

**Core Definition:** OAuth 2.0 is an **authorization framework** that allows an application to obtain limited access to a protected resource on behalf of a user or itself.

**Technical Definition:** OAuth 2.0 defines **flows** (Authorization Code, Client Credentials, Device Code, etc.) that issue **access tokens** scoped to specific permissions. ASP.NET Core uses OAuth 2.0 via handlers such as `AddOAuth()` and as the foundation for OpenID Connect. The **authorization code flow with PKCE** is the recommended flow for interactive clients . OAuth 2.0 is about **delegated authorization**, not authentication directly; OIDC adds the authentication layer on top .

**Beginner-Friendly Explanation:** OAuth 2.0 is **“letting an app do something on your behalf without giving it your password.”** It issues a scoped access token that the app uses to call APIs.

---

### Purposes

- To **grant scoped access** to APIs without sharing passwords.
- To support **third-party integrations** (social login, cloud APIs).
- To enable **delegated authorization** for SPAs and mobile apps.
- To provide **token-based** access to protected resources.
- To serve as the foundation for **OpenID Connect**.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication()
    .AddOAuth("Provider", options =>
    {
        options.ClientId = "client-id";
        options.ClientSecret = "client-secret";
        options.AuthorizationEndpoint = "https://provider.com/oauth/authorize";
        options.TokenEndpoint = "https://provider.com/oauth/token";
        options.CallbackPath = "/signin-provider";
        options.Scope.Add("read");
    });
```

**Component breakdown**

- `AddOAuth()` — Registers a generic OAuth 2.0 handler.
- `ClientId`, `ClientSecret` — App credentials.
- `AuthorizationEndpoint`, `TokenEndpoint` — Provider URLs.
- `Scope` — Requested permissions.

#### Syntax Rules

- The **authorization code + PKCE** flow is recommended for interactive clients .
- Access tokens are bearer tokens; protect them in transit.
- Refresh tokens (if issued) require secure storage.

#### Constraints and Limitations

- OAuth 2.0 alone does not authenticate the user; combine with OIDC for identity.
- Misconfigured redirect URIs or scopes can cause security vulnerabilities.

---

### Annotated Code Examples and Expected Outputs

#### Example: OAuth 2.0 Client Configuration

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = "Provider";
})
.AddCookie()
.AddOAuth("Provider", options =>
{
    options.ClientId = Configuration["OAuth:ClientId"]!;
    options.ClientSecret = Configuration["OAuth:ClientSecret"]!;
    options.CallbackPath = "/signin-oauth";
    options.AuthorizationEndpoint = "https://provider.com/authorize";
    options.TokenEndpoint = "https://provider.com/token";
    options.UserInformationEndpoint = "https://provider.com/userinfo";
    options.Scope.Add("openid");
});
```

**Expected Output (conceptual)**
```
// Unauthenticated users are challenged to the provider’s authorization endpoint.
// After consent, the app receives an access token and can call the provider’s API.
```

**Why this pattern is used:** The cookie scheme maintains the local session; the OAuth scheme handles the external authorization flow .

---

### Real-World Cases

- **Social login** — Google, Facebook, GitHub (often via OIDC) .
- **API access** — Granting an app permission to read a user’s data.
- **Service-to-service** — Client credentials flow for machine-to-machine access.

---

### References

- OAuth 2.0 and OpenID Connect in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- OAuth 2.0 — RFC 6749 — https://datatracker.ietf.org/doc/html/rfc6749

---

## 5. OpenID Connect (OIDC)

### Definitions

**Core Definition:** OpenID Connect is an **authentication layer** built on top of OAuth 2.0 that verifies the identity of the end-user and returns an **ID token** with claims about the user.

**Technical Definition:** OIDC adds an **ID token** (a JWT) to the OAuth 2.0 flow. ASP.NET Core’s `OpenIdConnectHandler` (package `Microsoft.AspNetCore.Authentication.OpenIdConnect`) implements the **authorization code flow with PKCE**, validates the ID token, and creates the `ClaimsPrincipal` . It uses the provider’s **discovery document** (`.well-known/openid-configuration`) to obtain endpoints and keys. The recommended pattern combines cookie authentication (for the local session) with OIDC (for the login challenge) .

**Beginner-Friendly Explanation:** OIDC is **“Login with Microsoft/Google/IdentityServer.”** It proves who the user is and returns their name, email, and other claims, while OAuth 2.0 handles the authorization tokens.

---

### Purposes

- To **delegate authentication** to a trusted identity provider.
- To obtain **standardized identity claims** (name, email, roles).
- To support **single sign-on (SSO)** across applications.
- To avoid managing passwords directly.
- To integrate with **Azure AD, IdentityServer, and social providers**.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = "https://identity.example.com";
    options.ClientId = "web-client";
    options.ClientSecret = "secret";
    options.ResponseType = "code";
    options.UsePkce = true;
    options.Scope.Add("openid");
    options.Scope.Add("profile");
    options.SaveTokens = true;
});
```

**Component breakdown**

- `AddOpenIdConnect()` — Registers the OIDC handler.
- `Authority` — The OIDC provider URL.
- `ClientId`, `ClientSecret` — App credentials.
- `ResponseType = "code"` — Authorization code flow.
- `UsePkce = true` — PKCE protection .
- `SaveTokens` — Stores tokens in the authentication properties.

#### Syntax Rules

- OIDC uses the **authorization code flow with PKCE** for web apps .
- The default challenge scheme should be OIDC; the default scheme should be cookies .
- `SaveTokens = true` allows later API calls with the access token.

#### Constraints and Limitations

- Public clients (without a secret) should use PKCE; confidential clients may also use a client secret .
- OIDC requires network access to the provider’s discovery endpoint.

---

### Annotated Code Examples and Expected Outputs

#### Example: OIDC Login Flow

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/", (HttpContext ctx) =>
    ctx.User.Identity?.IsAuthenticated == true
        ? $"Hello, {ctx.User.Identity.Name}"
        : "Not signed in");
```

**Expected Output**

- Unauthenticated request → challenge redirect to the OIDC provider.
- After login → cookie set; subsequent requests show `Hello, <name>`.

**Why this output occurs:** `UseAuthentication()` invokes the cookie handler; if no cookie is present, the OIDC challenge redirects to the provider .

---

### Real-World Cases

- **Enterprise SSO** — Azure AD / Entra ID, Okta, Auth0.
- **Social login** — Google, Microsoft Account, Facebook .
- **IdentityServer** — Self-hosted OIDC provider for internal apps .

---

### References

- Configure OpenID Connect web authentication in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- OpenID Connect Core — https://openid.net/specs/openid-connect-core-1_0.html

---

## 6. Azure AD (Microsoft Entra ID)

### Definitions

**Core Definition:** Azure Active Directory (now **Microsoft Entra ID**) is Microsoft’s cloud identity provider, supporting OIDC and OAuth 2.0 for authentication and authorization.

**Technical Definition:** Azure AD integrates with ASP.NET Core via the **Microsoft Identity Web** library (`Microsoft.Identity.Web`), which configures OIDC/JWT bearer authentication using `AddMicrosoftIdentityWebApp()` or `AddMicrosoftIdentityWebApi()`. Configuration is bound from the `AzureAd` section (Tenant ID, Client ID, scopes) . The library simplifies token acquisition, validation, and Microsoft Graph access.

**Beginner-Friendly Explanation:** Azure AD is **Microsoft’s cloud login system**. It lets users sign in with their Microsoft work or personal accounts, and your app receives their identity claims.

---

### Purposes

- To **authenticate Microsoft work/school accounts**.
- To enable **SSO** across Microsoft 365 and custom apps.
- To obtain **Microsoft Graph tokens** for accessing Microsoft data.
- To leverage **conditional access and MFA** managed by Azure.
- To integrate with **Azure AD B2C** for customer identities.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(
        builder.Configuration.GetSection("AzureAd"));

// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "tenant-id",
    "ClientId": "client-id",
    "CallbackPath": "/signin-oidc"
  }
}
```

**Component breakdown**

- `AddMicrosoftIdentityWebApp()` — Configures OIDC for a web app.
- `AzureAd` section — Tenant ID, Client ID, and instance.
- `CallbackPath` — Redirect URI after login.

#### Syntax Rules

- Requires `Microsoft.Identity.Web` package .
- Tenant ID identifies the directory; Client ID identifies the app registration.
- Scopes are configured for API access (e.g., Microsoft Graph).

#### Constraints and Limitations

- Requires an Azure AD tenant and app registration.
- Network access to `login.microsoftonline.com` is required.

---

### Annotated Code Examples and Expected Outputs

#### Example: Azure AD Web App Authentication

```csharp
builder.Services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(builder.Configuration.GetSection("AzureAd"));

app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/me", (HttpContext ctx) =>
    ctx.User.Claims.Select(c => new { c.Type, c.Value }));
```

**Expected Output (conceptual)**
```
// After login, returns claims such as name, email, and object ID from Azure AD.
```

**Why this pattern is used:** The library configures OIDC with Azure AD’s endpoints and validates the ID token automatically .

---

### Real-World Cases

- **Enterprise apps** — Internal dashboards using corporate credentials.
- **Microsoft Graph integration** — Accessing Outlook, Teams, or SharePoint data.
- **B2C scenarios** — Customer-facing apps with Azure AD B2C .

---

### References

- Microsoft identity platform and Microsoft Entra ID — Microsoft Learn — https://learn.microsoft.com/en-us/entra/identity-platform/
- Microsoft Identity Web — Microsoft Learn — https://learn.microsoft.com/en-us/entra/msal/dotnet/microsoft-identity-web/

---

## 7. IdentityServer (Duende)

### Definitions

**Core Definition:** Duende IdentityServer is a **self-hosted OIDC and OAuth 2.0 provider** for ASP.NET Core that issues tokens and manages authentication for multiple applications.

**Technical Definition:** Duende IdentityServer (the commercial successor to IdentityServer4) implements OIDC, OAuth 2.x, and SAML. It is added to an ASP.NET Core app via `AddIdentityServer()` and `UseIdentityServer()` . It supports authorization code + PKCE, client credentials, refresh tokens, device flow, PAR, and FAPI 2.0 . It integrates with **ASP.NET Core Identity** for user management via `Duende.IdentityServer.AspNetIdentity` .

**Beginner-Friendly Explanation:** IdentityServer is **your own private login server**. You run it alongside your apps, and it handles login, token issuance, and SSO for all of them.

---

### Purposes

- To **centralize authentication** for multiple apps.
- To provide **SSO** across internal applications.
- To issue **OIDC and OAuth 2.0 tokens** for APIs.
- To support **federation** with external providers.
- To enable **fine-grained token customization**.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddIdentityServer()
    .AddInMemoryClients(Config.Clients)
    .AddInMemoryIdentityResources(Config.IdentityResources)
    .AddInMemoryApiScopes(Config.ApiScopes)
    .AddAspNetIdentity<ApplicationUser>();

app.UseIdentityServer();
```

**Component breakdown**

- `AddIdentityServer()` — Registers IdentityServer services.
- `AddInMemoryClients` — Defines OAuth clients.
- `AddInMemoryApiScopes` — Defines API scopes.
- `AddAspNetIdentity<T>()` — Connects ASP.NET Core Identity for user storage.
- `UseIdentityServer()` — Adds the middleware .

#### Syntax Rules

- IdentityServer is typically hosted as a separate ASP.NET Core app.
- Clients and scopes can be stored in memory, EF Core, or custom stores .
- ASP.NET Core Identity integration requires the `Duende.IdentityServer.AspNetIdentity` package .

#### Constraints and Limitations

- **Production use may require a paid license** .
- Requires careful key management and data protection configuration.

---

### Annotated Code Examples and Expected Outputs

#### Example: IdentityServer Host Configuration

```csharp
// Program.cs of the IdentityServer host
builder.Services.AddIdentityServer()
    .AddInMemoryClients(new[]
    {
        new Client
        {
            ClientId = "web",
            ClientSecrets = { new Secret("secret".Sha256()) },
            AllowedGrantTypes = GrantTypes.Code,
            RedirectUris = { "https://localhost:5002/signin-oidc" },
            AllowedScopes = { "openid", "profile", "api1" }
        }
    })
    .AddInMemoryApiScopes(new[] { new ApiScope("api1") })
    .AddAspNetIdentity<ApplicationUser>();

app.UseIdentityServer();
```

**Expected Output (conceptual)**
```
// IdentityServer exposes OIDC discovery, authorization, and token endpoints.
// Clients authenticate users and receive tokens for api1.
```

**Why this pattern is used:** It defines a code-flow client and API scope, connecting to ASP.NET Core Identity for user authentication .

---

### Real-World Cases

- **Internal SSO** — One login for multiple internal apps.
- **API gateways** — IdentityServer issues tokens for downstream APIs.
- **Federated identity** — Acting as a gateway to external IdPs .

---

### References

- Duende IdentityServer Documentation — https://docs.duendesoftware.com/identityserver/
- ASP.NET Identity Integration — Duende — https://docs.duendesoftware.com/identityserver/aspnet-identity/

---

## 8. ASP.NET Core Identity

### Definitions

**Core Definition:** ASP.NET Core Identity is a **complete membership system** for managing users, passwords, roles, claims, and external logins.

**Technical Definition:** ASP.NET Core Identity (`Microsoft.AspNetCore.Identity`) provides APIs and UI for user registration, login, password hashing, email confirmation, two-factor authentication, account lockout, and role management . It uses EF Core (typically SQL Server) to persist user data and integrates with cookie authentication by default. It is **distinct from the Microsoft identity platform** (Azure AD) .

**Beginner-Friendly Explanation:** ASP.NET Core Identity is **a ready-made user database and login system**. It stores usernames, hashed passwords, and roles, and handles registration, login, and account management for you.

---

### Purposes

- To manage **first-party user accounts** (registration, login, logout).
- To securely **hash and store passwords**.
- To support **roles, claims, and policies**.
- To enable **email confirmation and password reset**.
- To provide **two-factor authentication (2FA)** and **account lockout**.

---

### Syntax Rules and Structure

```csharp
builder.Services.AddDefaultIdentity<IdentityUser>(options =>
    {
        options.SignIn.RequireConfirmedAccount = true;
        options.Password.RequiredLength = 8;
    })
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();

app.UseAuthentication();
app.UseAuthorization();
```

**Component breakdown**

- `AddDefaultIdentity<T>()` — Configures Identity with default UI and cookie authentication .
- `AddRoles<TRole>()` — Enables role management.
- `AddEntityFrameworkStores<T>()` — Persists Identity data with EF Core.
- Identity options — Password policy, lockout, sign-in requirements .

#### Syntax Rules

- Identity requires a `DbContext` derived from `IdentityDbContext`.
- Passwords are hashed automatically; never store plain text .
- Account lockout should be enabled to prevent brute-force attacks .

#### Constraints and Limitations

- Identity UI is a Razor Class Library; scaffold only the pages you need .
- Not designed for stateless APIs; use JWT/OIDC for APIs .

---

### Annotated Code Examples and Expected Outputs

#### Example: User Registration and Login

```csharp
// Register
var user = new IdentityUser { UserName = "alice@example.com", Email = "alice@example.com" };
var result = await _userManager.CreateAsync(user, "P@ssw0rd!");
if (result.Succeeded) await _signInManager.SignInAsync(user, isPersistent: false);

// Login
var result = await _signInManager.PasswordSignInAsync(
    "alice@example.com", "P@ssw0rd!", isPersistent: false, lockoutOnFailure: true);
```

**Expected Output (conceptual)**
```
// User created with hashed password; sign-in cookie issued on success.
```

**Why this pattern is used:** `UserManager` handles hashing and persistence; `SignInManager` issues the authentication cookie .

---

### Real-World Cases

- **Web apps with user accounts** — Registration, login, profile management.
- **Admin dashboards** — Role-based access control.
- **Customer portals** — Email confirmation and password reset.

---

### References

- Introduction to Identity on ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- Configure ASP.NET Core Identity — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration

---

## 9. Authentication Best Practices

### Definitions

**Core Definition:** Authentication best practices are the guidelines for implementing secure, maintainable, and standards-compliant authentication.

**Technical Definition:** Best practices encompass using **secure flows** (authorization code + PKCE), **keeping secrets out of source control**, **enforcing HTTPS**, **using ASP.NET Core Identity for first-party accounts**, **preferring standards-based providers for APIs**, **validating tokens strictly**, **persisting data protection keys**, and **applying antiforgery protection** for cookie-based apps .

**Beginner-Friendly Explanation:** The best practices are: **use HTTPS**, **don’t hard-code secrets**, **use established libraries**, **validate tokens properly**, and **keep your keys safe**.

---

### Guidelines and Structure

#### General Guidelines

```csharp
// 1. Use the most secure flow (code + PKCE for OIDC)
options.ResponseType = "code";
options.UsePkce = true;

// 2. Keep secrets out of source control
// Use Secret Manager in development, Key Vault in production

// 3. Enforce HTTPS and HSTS
app.UseHttpsRedirection();
app.UseHsts();

// 4. Use ASP.NET Core Identity for first-party accounts
builder.Services.AddDefaultIdentity<IdentityUser>();

// 5. Validate JWT tokens strictly
options.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidateAudience = true,
    ValidateLifetime = true,
    ValidateIssuerSigningKey = true
};

// 6. Persist data protection keys
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\keys\"));

// 7. Apply antiforgery for cookie-based apps
app.UseAntiforgery();   // .NET 8+
```

**Component breakdown**

- **Secure flow** — Code + PKCE for OIDC .
- **Secrets management** — User Secrets / Key Vault .
- **HTTPS** — Prevents credential theft .
- **Identity** — For first-party accounts .
- **Token validation** — Strict validation of issuer, audience, lifetime, and signature.
- **Data protection** — Persist keys for multi-instance deployments .
- **Antiforgery** — CSRF protection for cookies .

#### Rules and Constraints

- **Do** use the authorization code flow with PKCE for interactive clients.
- **Do** store secrets in a secure store, never in `appsettings.json`.
- **Do** enforce HTTPS and HSTS.
- **Do** use ASP.NET Core Identity for first-party user accounts.
- **Do** validate JWT tokens strictly.
- **Do** persist data protection keys in multi-instance deployments.
- **Do not** use implicit flow or resource owner password flow for new apps.
- **Do not** trust client-side state or tokens without validation.

---

### Annotated Code Examples and Expected Outputs

#### Example: Secure Authentication Pipeline

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie(options =>
{
    options.Cookie.HttpOnly = true;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Lax;
})
.AddOpenIdConnect(options =>
{
    options.Authority = "https://identity.example.com";
    options.ClientId = Configuration["Oidc:ClientId"]!;
    options.ClientSecret = Configuration["Oidc:ClientSecret"]!;
    options.ResponseType = "code";
    options.UsePkce = true;
    options.SaveTokens = true;
});

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
```

**Expected Output (conceptual)**
```
// HTTPS enforced; OIDC code+PKCE flow; secure cookies; strict token validation.
```

**Why this configuration is used:** It combines the recommended flow, secure cookie settings, and correct middleware ordering .

---

### Real-World Cases

- **Public-facing web apps** — OIDC + cookies + HTTPS.
- **APIs** — JWT bearer with strict validation.
- **Enterprise SSO** — IdentityServer or Azure AD with PKCE.

---

### References

- Security and Identity — ASP.NET Core guidance — https://github.com/midudev/autoskills/blob/6fd754766ce993bea63773415dd8a5f52d8f119f/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- ASP.NET Core Security Best Practices 2026 — https://offensive360.com/aspnet-core-security-best-practices/
- Choose an identity management solution — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/how-to-choose-identity-solution

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---|---|
| Implicit flow / Resource Owner Password flow | **Not recommended**; use authorization code + PKCE . |
| IdentityServer4 | **Legacy**; Duende IdentityServer is the successor . |
| ASP.NET Core Identity for APIs | **Not ideal**; use JWT/OIDC for stateless APIs . |
| `AddDefaultIdentity` | Available since **ASP.NET Core 2.1** . |
| OIDC PAR | Enabled by default from **.NET 9** if supported . |
| Data Protection keys | Must be persisted for multi-instance deployments . |
| Secrets in `appsettings.json` | **Unsafe**; use Secret Manager/Key Vault . |

---

## References (Consolidated)

- Overview of ASP.NET Core Authentication — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/
- Choose an identity management solution — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/how-to-choose-identity-solution
- Use cookie authentication without ASP.NET Core Identity — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie
- Configure JWT bearer authentication in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication
- Configure OpenID Connect web authentication in ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- Introduction to Identity on ASP.NET Core — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity
- Configure ASP.NET Core Identity — Microsoft Learn — https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration
- Duende IdentityServer Documentation — https://docs.duendesoftware.com/identityserver/
- ASP.NET Identity Integration — Duende — https://docs.duendesoftware.com/identityserver/aspnet-identity/
- Security and Identity — ASP.NET Core guidance — https://github.com/midudev/autoskills/blob/6fd754766ce993bea63773415dd8a5f52d8f119f/packages/autoskills/skills-registry/aspnet-core/references/security-and-identity.md
- ASP.NET Core Security Best Practices 2026 — https://offensive360.com/aspnet-core-security-best-practices/
- OAuth 2.0 — RFC 6749 — https://datatracker.ietf.org/doc/html/rfc6749
- OpenID Connect Core — https://openid.net/specs/openid-connect-core-1_0.html

---