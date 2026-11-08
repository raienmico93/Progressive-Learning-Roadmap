# ASP.NET Core JWT Authentication: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) using the `Microsoft.AspNetCore.Authentication.JwtBearer` package. JWT (JSON Web Token) Bearer authentication is the standard for APIs. Starting with ASP.NET Core 8+, the modern `Microsoft.IdentityModel.JsonWebTokens` library (`JsonWebTokenHandler`) is used by default; the legacy `System.IdentityModel.Tokens.Jwt` is officially deprecated. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | JWT (JSON Web Token) is a compact, URL-safe token format used to securely transmit information between systems as a signed statement about a user or service. |
| **Technical Definition** | A JWT is a Base64URL-encoded string with three parts separated by dots: `header.payload.signature`. The **header** specifies the signing algorithm; the **payload** contains **claims** (key-value assertions about the subject); the **signature** ensures integrity and authenticity. In ASP.NET Core, `JwtBearerHandler` validates incoming JWTs and extracts the user's identity from the claims, populating `HttpContext.User` for authorization. |
| **Beginner-Friendly Explanation** | A JWT is like a **digitally signed ID card** for APIs. When a user logs in, the identity provider issues a JWT that says "this is Alice, these are her roles, and this is when the card expires." The client sends this token with every API request. The API checks the signature (to ensure the card isn't forged) and the claims (to know who Alice is and what she can do). |

### Key Characteristics

- **Self-contained** – The token carries identity and authorization information; the API can validate it without a database lookup (though revocation requires additional infrastructure).
- **Signed, not encrypted** – The payload is Base64URL-encoded and readable by anyone; the signature prevents tampering but does **not** provide confidentiality. Never put secrets or sensitive PII in a JWT.
- **Stateless** – By default, no server-side session is stored; the token is valid until it expires. Revocation requires a deliberate strategy (denylist, short lifetimes, key rotation).
- **Bearer token** – The token is sent in the `Authorization: Bearer <token>` header. Anyone holding the token can use it; possession is the proof.
- **Claims-based** – The payload contains standard claims (`iss`, `aud`, `exp`, `sub`, `jti`) and custom claims (`role`, `tenant`, etc.).

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **authentication fundamentals**, **claims-based identity**, and **middleware**.
- Familiarity with **dependency injection** and service registration.
- An identity provider (Azure Entra ID, Duende IdentityServer, Auth0, WorkOS, or a custom issuer).

### Related Programming Areas

- **Authentication fundamentals** – Schemes, handlers, and the `ClaimsPrincipal`.
- **Authorization** – Policies and roles evaluate the claims in the JWT.
- **Cookie authentication** – Often used alongside JWT for mixed UI/API applications.
- **Refresh tokens** – Enable long-lived sessions without long-lived access tokens.
- **Key rotation and JWKS** – Critical for production JWT validation and zero-downtime key changes.

### Core Concepts / Features

1. **JWT** – Structure, claims, signing, and security properties.
2. **JWT authentication** – `AddJwtBearer`, `TokenValidationParameters`, and middleware integration.
3. **Access tokens** – Short-lived tokens used to call APIs.
4. **Refresh tokens** – Long-lived tokens used to obtain new access tokens, with rotation and reuse detection.
5. **JWT best practices** – Validation strictness, claim hygiene, revocation, and key management.

---

## 1. JWT (The Token Format)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A JWT is a compact, three-part token (`header.payload.signature`) that encodes claims about a subject and is protected by a cryptographic signature. |
| **Technical Definition** | A JWT consists of: **Header** – metadata including the signing algorithm (`alg`) and token type (`typ`); **Payload** – a JSON object of **claims** (registered claims like `iss`, `aud`, `exp`, `sub`, `jti`, and custom claims like `role`); **Signature** – the Base64URL-encoded hash of `header.payload` signed with a secret (HS256) or private key (RS256/ES256). The signature provides **integrity and authenticity**, not confidentiality. |
| **Beginner-Friendly Explanation** | A JWT is like a **tamper-evident envelope** containing a message. The message (payload) is readable by anyone, but the seal (signature) proves who sent it and that nobody changed it. Because the message is readable, you must never put secrets inside—only information you are comfortable sharing. |

### Purposes

- **To** transmit identity and authorization information between systems in a **stateless, self-contained** format.
- **To** enable APIs to validate the token **without a database lookup** (in the common case), improving scalability.
- **To** support **delegated authorization**, where a user's identity is carried from one service to another.
- **To** provide a standard, interoperable token format accepted across platforms and identity providers.

### Syntax Rules and Structure

```text
header.payload.signature
```

**Component breakdown:**

| Part | Content | Example (decoded) |
|------|---------|-------------------|
| **Header** | `alg` (algorithm), `typ` (token type), `kid` (key ID for rotation) | `{"alg":"RS256","typ":"JWT","kid":"2024-key"}` |
| **Payload** | Claims: `iss` (issuer), `aud` (audience), `exp` (expiry), `sub` (subject), `jti` (token ID), `iat` (issued at), custom claims | `{"sub":"alice","role":"Admin","tenant":"acme","exp":1735689600}` |
| **Signature** | Base64URL-encoded signature of `header.payload` | — |

**Syntax rules:**

- **Claims are JSON key-value pairs** in the payload.
- **Standard claims** (`iss`, `aud`, `exp`, `sub`, `jti`, `iat`) should follow the JWT specification.
- **Custom claims** (e.g., `role`, `tenant`, `permissions`) carry application-specific data.
- The signature is computed over the **exact Base64URL-encoded** header and payload.

**Constraints and limitations:**

- **The payload is not encrypted**—anyone can decode it. Never put passwords, secrets, or high-risk PII in a JWT payload.
- **Token size** – JWTs are sent with every request; keep claims minimal to avoid header-size limits.
- **No built-in revocation** – a valid JWT remains valid until expiry unless a revocation strategy is implemented.

### Annotated Code Example (Creating a JWT)

```csharp
using Microsoft.IdentityModel.JsonWebTokens;
using Microsoft.IdentityModel.Tokens;

public string CreateToken(string userId, string role, string issuer, string audience, SecurityKey signingKey)
{
    var handler = new JsonWebTokenHandler();
    var descriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(ClaimTypes.Role, role),
            new Claim("tenant", "acme")
        }),
        Issuer = issuer,
        Audience = audience,
        Expires = DateTime.UtcNow.AddMinutes(15),
        SigningCredentials = new SigningCredentials(signingKey, SecurityAlgorithms.RsaSha256)
    };
    return handler.CreateToken(descriptor);
}
```

**Expected Output:** A signed JWT string in the format `header.payload.signature` that encodes the user ID, role, tenant, issuer, audience, and a 15-minute expiry.

**Why this output?** `JsonWebTokenHandler` constructs the header and payload, then signs them with the provided `SigningCredentials`. The result is a compact, URL-safe token.

### Real-World Cases

- **API access tokens:** Short-lived JWTs issued by an identity provider and sent by clients in the `Authorization` header.
- **Service-to-service authentication:** Client credentials flow issues application access tokens as JWTs.
- **Delegated user authorization:** Access tokens carry the user's identity and scopes to downstream APIs.

### References

- JSON Web Token (JWT) – https://en.wikipedia.org/wiki/JSON_Web_Token
- JWT claims registry – https://www.iana.org/assignments/jwt/jwt.xhtml#claims
- How to handle JWT in .NET (WorkOS) – https://webflow.workos.com/blog/how-to-handle-jwt-in-net

---

## 2. JWT Authentication (Validation Pipeline)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | JWT authentication is the process of validating an incoming JWT and using its claims to establish the caller's identity. |
| **Technical Definition** | `AddJwtBearer` registers `JwtBearerHandler` with the `"Bearer"` scheme. On each request, `UseAuthentication()` invokes the handler, which extracts the token from the `Authorization: Bearer` header, validates its signature, issuer, audience, and lifetime using `TokenValidationParameters`, and creates a `ClaimsPrincipal` from the token's claims. The principal is attached to `HttpContext.User`. |
| **Beginner-Friendly Explanation** | JWT authentication is the **API's ID check**. The handler reads the token from the request header, checks the signature (is it real?), the issuer (who made it?), the audience (is it meant for this API?), and the expiry (is it still valid?). If everything passes, it builds the user's identity from the claims. If anything fails, the request is rejected with **401 Unauthorized**. |

### Purposes

- **To** validate incoming JWTs against a trusted issuer and signing key.
- **To** reject tampered, expired, or wrongly targeted tokens with a **401** response.
- **To** populate `HttpContext.User` with claims so authorization policies can evaluate permissions.
- **To** support multiple issuers or audiences through explicit configuration.
- **To** integrate with OpenID Connect / OAuth 2.0 providers for standards-based token validation.

### Syntax Rules and Structure

#### General Syntax (Basic Configuration)

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://{your-authority}";
        options.Audience = "https://{your-audience}";
    });
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `AddAuthentication(scheme)` | Sets `"Bearer"` as the default authentication scheme. |
| `AddJwtBearer(options => ...)` | Registers the JWT bearer handler. |
| `Authority` | The token issuer's metadata endpoint (for automatic key discovery via JWKS). |
| `Audience` | The intended audience of the token (usually the API's identifier). |

#### General Syntax (Explicit Validation)

```csharp
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
        ValidIssuer = Configuration["Jwt:Issuer"],
        ValidAudience = Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(Configuration["Jwt:SecretKey"]!)),
        ClockSkew = TimeSpan.FromSeconds(30)
    };
});
```

**Syntax rules:**

- **`ValidateIssuer` and `ValidateAudience`** should be `true`; the framework does not explicitly enable them by default, so set them explicitly.
- **`ValidateLifetime`** must be `true` to reject expired tokens.
- **`ValidateIssuerSigningKey`** must be `true` to verify the signature.
- **`ClockSkew`** defaults to **5 minutes**; tighten it (e.g., 30 seconds) to avoid accepting expired tokens for too long.
- **`MapInboundClaims = false`** prevents ASP.NET Core from rewriting standard claim names (`sub`) into legacy URIs, so claim names stay predictable.
- **`RequireHttpsMetadata = true`** ensures metadata is fetched over HTTPS.

**Constraints and limitations:**

- **JWT authentication does not restrict access by itself**—a fallback policy or `[Authorize]` attributes are required.
- **The API should return 401/403, not redirect** to the identity provider; the client is responsible for acquiring tokens.
- **Key management is critical**—using a weak secret or failing to rotate keys compromises the entire trust boundary.

### Annotated Complete Step-by-Step Code Example

```csharp
// Program.cs
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

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
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]!)),
            ClockSkew = TimeSpan.FromSeconds(30)
        };
    });

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();   // Validates JWT and populates User
app.UseAuthorization();    // Enforces [Authorize]
app.MapControllers();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` sets the default scheme to `"Bearer"`.
2. `AddJwtBearer` configures the handler with explicit validation: issuer, audience, lifetime, and signing key are all checked.
3. `MapInboundClaims = false` keeps claim names as-is (`sub`, not the legacy URI).
4. `ClockSkew = 30 seconds` limits the window during which expired tokens are still accepted.
5. `UseAuthentication()` runs on every request, validates the token, and sets `HttpContext.User`.
6. `UseAuthorization()` enforces `[Authorize]` on protected endpoints.

**Expected Output:** A request with a valid JWT and correct issuer/audience is authenticated; an invalid signature, wrong audience, or expired token returns **401 Unauthorized**.

**Why this output?** The handler validates every configured aspect of the token. If any check fails, `AuthenticateResult` is unsuccessful, and authorization middleware returns 401.

### Real-World Cases

- **API consumed by SPAs or mobile apps:** JWT bearer authentication with an identity provider (Auth0, Entra ID, IdentityServer).
- **Microservices:** Each service validates JWTs issued by a central authority, using `Authority` for automatic key discovery.
- **Multi-tenant APIs:** Two JWT schemes for different issuers, with a policy scheme that forwards based on the token's issuer.

### References

- Configure JWT bearer authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication
- Policy schemes in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/policy-schemes
- TokenValidationParameters (Microsoft.IdentityModel.Tokens) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters

---

## 3. Access Tokens

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An access token is a credential that a client presents to an API to access protected resources. |
| **Technical Definition** | Access tokens are strings used by a client app to make requests to an API. They can vary in format, may be encrypted, and are intended **solely for the API**—they should never be read or interpreted by the client (web UI, mobile app). They are typically sent in the `Authorization: Bearer` header. In OAuth 2.0, access tokens are issued by an authorization server after successful authentication/authorization. |
| **Beginner-Friendly Explanation** | An access token is the **key to the API**. After a user logs in, the identity provider gives the client an access token. The client shows this token to the API on every request. The API checks the token and, if valid, grants access. Access tokens are **short-lived** on purpose—if stolen, they expire quickly. |

### Purposes

- **To** authorize API requests by proving the caller's identity and permissions.
- **To** enable **delegated access**—a token issued to a user can be used by a client to act on that user's behalf.
- **To** support **application access**—client credentials tokens represent the application itself, not a user.
- **To** carry scopes and claims that the API uses for fine-grained authorization.

### Syntax Rules and Structure

#### Access Token Characteristics

| Property | Recommended Value / Behaviour |
|----------|-------------------------------|
| **Lifetime** | **Short**—15 minutes or less is common; never longer than an hour. |
| **Format** | JWT (most common) or opaque (reference) token. |
| **Transport** | `Authorization: Bearer <token>` header. |
| **Audience (`aud`)** | The API's identifier—must match what the API expects. |
| **Scopes / Claims** | Minimal permissions needed for the request. |
| **Storage (client)** | **Never** in `localStorage` or `sessionStorage`; use `HttpOnly`, `Secure`, `SameSite` cookies or in-memory storage. |

**Syntax rules:**

- The API **must validate** the token's signature, issuer, audience, and expiry on every request.
- **Delegated access tokens** (user tokens) should be used whenever a user is involved; **application tokens** (client credentials) should be used only for machine-to-machine scenarios.
- The client **must not** inspect or rely on the contents of the access token; it is opaque to the client by contract.

**Constraints and limitations:**

- **Revocation is not built-in**—short lifetimes limit the damage window. For immediate revocation, combine with a denylist or token versioning.
- **Token theft**—a stolen bearer token grants access until expiry; sender-constrained tokens (e.g., DPoP, mTLS) can mitigate this, but add complexity.
- **Audience validation** is critical—accepting a token meant for a different API is a serious vulnerability.

### Annotated Code Example (Issuing a Short-Lived Access Token)

```csharp
var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.Id),
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
        new Claim(ClaimTypes.Role, "User"),
        new Claim("scope", "api.read api.write")
    }),
    Issuer = issuer,
    Audience = audience,
    Expires = DateTime.UtcNow.AddMinutes(15),   // Short lifetime
    SigningCredentials = new SigningCredentials(signingKey, SecurityAlgorithms.RsaSha256)
};
```

**Expected Output:** A JWT with a 15-minute expiry, a unique `jti` (token ID for revocation), the user's role, and the granted scopes.

**Why this output?** The short expiry limits the window of token misuse; the `jti` enables token-level revocation if needed; the role and scope claims support API authorization.

### Real-World Cases

- **SPA + API:** The SPA obtains an access token via OIDC and sends it to the API in the `Authorization` header.
- **Mobile app + backend:** The app stores the access token in secure platform storage and attaches it to API requests.
- **Service-to-service:** A background service uses the client credentials flow to obtain an application access token.

### References

- Access tokens (Microsoft Learn) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication#access-tokens
- OWASP JWT security recommendations – https://github.com/Equilateral-AI/EquilateralAgents-Community-Standards/blob/main/patterns/api-authentication-security.yaml

---

## 4. Refresh Tokens

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | A refresh token is a long-lived credential used to obtain a new access token without requiring the user to re-authenticate. |
| **Technical Definition** | Refresh tokens are issued alongside access tokens by the authorization server. When the access token expires, the client presents the refresh token to the token endpoint, which validates it and returns a **new access token** (and typically a **new refresh token**). **Refresh token rotation** issues a new refresh token on each use and invalidates the old one; **reuse detection** revokes the entire token family if a consumed token is presented again. |
| **Beginner-Friendly Explanation** | A refresh token is a **long-term pass** that lets the client get a new short-term access token when the old one expires. Instead of forcing the user to log in every 15 minutes, the client silently exchanges the refresh token for a new access token. **Rotation** means each refresh token can be used only once—if an attacker tries to reuse an old one, the system detects it and revokes the entire session. |

### Purposes

- **To** enable **long-lived sessions** without long-lived access tokens, balancing security and usability.
- **To** allow the client to obtain new access tokens **without user interaction** when the access token expires.
- **To** detect **token theft** through rotation and reuse detection—if a consumed refresh token is replayed, the system revokes the token family.
- **To** support **revocation**—deleting or marking the refresh token as revoked prevents further access token issuance.

### Syntax Rules and Structure

#### Refresh Token Flow

```text
1. Login → Authorization server returns { access_token, refresh_token }
2. API call with access_token → 401 (expired)
3. POST /token with refresh_token → new { access_token, refresh_token }
4. Old refresh_token is marked as used/revoked
5. If old refresh_token is reused → revoke entire token family
```

**Component breakdown:**

| Step | Action |
|------|--------|
| 1 | Issue access + refresh tokens on login. |
| 2 | Access token expires; API returns 401. |
| 3 | Client sends refresh token to token endpoint. |
| 4 | Server validates refresh token, rotates it (issues new one, marks old as used). |
| 5 | Reuse of the old token triggers family-wide revocation. |

**Syntax rules:**

- **Refresh tokens must be stored securely**—server-side (database/cache) or in `HttpOnly`/`Secure` cookies; never in `localStorage`.
- **Rotation** should be enforced: each refresh token is **single-use**; a consumed token cannot be reused.
- **Reuse detection** must revoke the entire token family (all tokens issued from the same login).
- **Bind refresh tokens** to device/IP metadata where possible to detect anomalies.
- **Rate-limit** the refresh endpoint to prevent abuse.

**Constraints and limitations:**

- **Refresh tokens are high-value**—if stolen, they can be used to mint access tokens until revoked. Rotation and reuse detection are essential.
- **Server-side storage required** for revocation and rotation; stateless refresh tokens cannot be revoked.
- **Short refresh token lifetime** (e.g., 30 days) limits the window of misuse if a token is compromised without reuse detection.

### Annotated Code Example (Refresh Token Rotation with Reuse Detection)

```csharp
public async Task<TokenPair> RefreshAsync(string refreshToken)
{
    var stored = await _store.FindAsync(refreshToken);

    if (stored is null || stored.IsRevoked)
    {
        // Reuse detected — revoke entire family
        await _store.RevokeFamilyAsync(stored?.FamilyId);
        throw new SecurityTokenException("Invalid refresh token.");
    }

    // Mark old token as used/revoked
    stored.IsRevoked = true;
    stored.IsUsed = true;

    // Issue new pair
    var newAccess = CreateAccessToken(stored.UserId);
    var newRefresh = CreateRefreshToken(stored.UserId, stored.FamilyId);

    await _store.SaveAsync(newRefresh);
    return new TokenPair(newAccess, newRefresh);
}
```

**Expected Output:** A valid refresh token returns a new access/refresh pair; the old refresh token is invalidated. Reuse of a revoked token throws an exception and revokes the entire family.

**Why this output?** Rotation ensures each refresh token is single-use; reuse detection triggers family-wide revocation, mitigating token theft.

### Real-World Cases

- **Mobile apps:** Long-lived sessions with silent token refresh; refresh tokens stored in secure platform storage (Keychain/Keystore).
- **SPAs:** Refresh tokens stored in `HttpOnly` cookies; rotation and reuse detection protect against XSS token theft.
- **Multi-device sessions:** Each device gets its own refresh token family; revocation of one family does not affect other devices.

### References

- Refresh token rotation and reuse detection (dotnet-auth-kit) – https://github.com/jamesgober/dotnet-auth-kit
- Refresh token rotation with reuse detection (C# Corner) – https://www.c-sharpcorner.com/article/production-ready-authentication-jwt-and-secure-refresh-token-rotation/
- JWT Done Right: refresh token design – https://systemweakness.com/jwt-done-right-15-production-mistakes-every-asp-net-core-developer-should-avoid-net-9-guide-f6dc59df9be8

---

## 5. JWT Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | JWT best practices are guidelines for issuing, validating, and managing JWTs that are secure, performant, and maintainable in production. |
| **Technical Definition** | Best practices encompass **strict validation** (signature, issuer, audience, lifetime, with tightened `ClockSkew`), **short access-token lifetimes** with refresh-token rotation, **secure key management** (asymmetric algorithms, JWKS, key rotation), **revocation strategies** (denylist, token versioning, short lifetimes), **claim hygiene** (no secrets or sensitive PII), and **client-side storage** that resists XSS and CSRF. |
| **Beginner-Friendly Explanation** | Best practices are the **"do's and don'ts"** for JWT—they help you avoid the most common production mistakes: accepting tokens meant for other APIs, letting expired tokens live too long, storing secrets in the payload, and treating a valid token as proof of authorization (it only proves identity). |

### Purposes

- **To** prevent token forgery, replay, and cross-audience attacks through strict validation.
- **To** limit the blast radius of a stolen token via short lifetimes and revocation.
- **To** protect signing keys from compromise through rotation and secure storage.
- **To** keep the token payload free of sensitive data that could be exposed.
- **To** ensure client-side storage does not expose tokens to XSS or CSRF.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Validation strictness** | Set `ValidateIssuer`, `ValidateAudience`, `ValidateLifetime`, and `ValidateIssuerSigningKey` to `true` explicitly. | Rely on defaults—some validation flags are not enabled by default. |
| **Clock skew** | Set `ClockSkew` to 30 seconds or less. | Leave the default 5-minute window—expired tokens remain valid too long. |
| **Claim mapping** | Set `MapInboundClaims = false` so claim names stay predictable. | Allow legacy claim-name rewriting that breaks `sub` lookups. |
| **Access token lifetime** | Keep it short (15 minutes or less). | Issue long-lived access tokens—theft grants prolonged access. |
| **Refresh tokens** | Use rotation with reuse detection; store securely server-side. | Allow refresh token reuse—if stolen, it can mint access tokens indefinitely. |
| **Signing algorithm** | Use **asymmetric** algorithms (RS256, ES256) for distributed systems. | Use HS256 with a shared secret across services—any service can forge tokens. |
| **Key management** | Use JWKS for key distribution and rotation; support `kid`-aware validators. | Hard-code signing keys or embed them in source code. |
| **Payload** | Store only minimal, non-sensitive claims. | Put passwords, secrets, or high-risk PII in the payload. |
| **Client storage** | Store tokens in `HttpOnly`, `Secure`, `SameSite` cookies or in-memory. | Use `localStorage` or `sessionStorage`—XSS can exfiltrate tokens. |
| **Revocation** | Combine short lifetimes with a denylist or token versioning for immediate revocation. | Assume stateless JWTs can be revoked without additional infrastructure. |
| **Authorization** | Use policy-based authorization with claims; never treat a claim as a permission without a policy. | Assume authentication equals authorization—a valid token only proves identity. |
| **Algorithm allowlist** | Pin the expected algorithm; never accept `alg: none` or auto-detect. | Allow the token header to dictate the verification algorithm. |

### Annotated Code Example (Production-Ready JWT Validation Configuration)

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
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]!)),
            ClockSkew = TimeSpan.FromSeconds(30),
            ValidAlgorithms = new[] { SecurityAlgorithms.RsaSha256 }
        };
    });

builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());
```

**Why this follows best practices:**

- **All validation flags are explicit**—issuer, audience, lifetime, and signing key are all checked.
- **`ClockSkew` is 30 seconds**—the default 5-minute window is tightened.
- **`MapInboundClaims = false`** keeps claim names predictable.
- **`ValidAlgorithms`** pins the expected algorithm, preventing algorithm-confusion attacks.
- **Fallback policy** requires authentication for all endpoints by default.
- **Secrets are loaded from configuration** (backed by a vault in production), not hard-coded.

### Real-World Cases

- **High-security APIs:** Explicit validation, short access-token lifetimes, refresh-token rotation, and JWKS-based key rotation.
- **Multi-tenant SaaS:** Audience validation ensures tokens issued for one tenant's API cannot be used against another tenant's API.
- **SPAs with BFF pattern:** Access tokens stored in `HttpOnly` cookies, refreshed via a backend-for-frontend, avoiding token exposure to JavaScript.

### References

- JWT security best practices (OWASP-aligned) – https://github.com/transilienceai/communitytools/blob/main/skills/authentication/reference/jwt_security_resources.md
- JWT Done Right: production mistakes – https://systemweakness.com/jwt-done-right-15-production-mistakes-every-asp-net-core-developer-should-avoid-net-9-guide-f6dc59df9be8
- How to handle JWT in .NET (WorkOS) – https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html

---

## Summary Table: JWT Authentication at a Glance

| Concept | Primary Role | Key API / Syntax | When to Use |
|---------|--------------|-------------------|-------------|
| **JWT** | Self-contained token format | `header.payload.signature`; claims | Transmitting identity/authorization across services |
| **JWT authentication** | Validate token and populate `User` | `AddJwtBearer`, `TokenValidationParameters`, `UseAuthentication()` | Every API protected by JWTs |
| **Access tokens** | Short-lived API credentials | JWT with `aud`, `exp`, `scope`; sent in `Authorization` header | Authorizing individual API requests |
| **Refresh tokens** | Long-lived token to obtain new access tokens | Rotation with reuse detection; server-side storage | Long-lived sessions without long-lived access tokens |
| **Best practices** | Secure, performant, maintainable JWT usage | Strict validation, short lifetimes, rotation, key rotation, claim hygiene | Every production JWT deployment |

---

## References (Full List)

- Configure JWT bearer authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication
- JSON Web Token (JWT) – https://en.wikipedia.org/wiki/JSON_Web_Token
- JWT claims registry – https://www.iana.org/assignments/jwt/jwt.xhtml#claims
- TokenValidationParameters (Microsoft.IdentityModel.Tokens) – https://learn.microsoft.com/en-us/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters
- Policy schemes in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/policy-schemes
- How to handle JWT in .NET (WorkOS) – https://webflow.workos.com/blog/how-to-handle-jwt-in-net
- dotnet-auth-kit (refresh token rotation, reuse detection, key rotation) – https://github.com/jamesgober/dotnet-auth-kit
- JWT Done Right: 15 Production Mistakes – https://systemweakness.com/jwt-done-right-15-production-mistakes-every-asp-net-core-developer-should-avoid-net-9-guide-f6dc59df9be8
- JWT security best practices (OWASP-aligned) – https://github.com/transilienceai/communitytools/blob/main/skills/authentication/reference/jwt_security_resources.md
- Refresh token rotation with reuse detection (C# Corner) – https://www.c-sharpcorner.com/article/production-ready-authentication-jwt-and-secure-refresh-token-rotation/
- OWASP DotNet Security Cheat Sheet – https://cheatsheetseries.owasp.org/cheatsheets/DotNet_Security_Cheat_Sheet.html
- API Security Best Practices 2026 (JWT algorithm pitfalls) – https://offensive360.com/blog/api-security-best-practices/