# ASP.NET Core OAuth and OpenID Connect: A Comprehensive Cheat Sheet

> **Scope & Version Note:** This cheat sheet targets **ASP.NET Core** (.NET 6–10) and the standards that underpin its external authentication stack: **OAuth 2.0** (RFC 6749) and **OpenID Connect Core 1.0**. OAuth 2.0 is an **authorization** framework, not an authentication protocol; OpenID Connect (OIDC) adds a standards-based **identity layer** on top of it. ASP.NET Core’s `Microsoft.AspNetCore.Authentication.OpenIdConnect` package implements the OIDC client, while `Microsoft.AspNetCore.Authentication.OAuth` supports provider-specific OAuth 2.0 flows. Starting with **.NET 9**, OAuth 2.0 **Pushed Authorization Requests (PAR)** are used by default when the OIDC server supports them. Public OIDC/OAuth clients are **no longer recommended for web applications**; confidential clients using the authorization code flow with PKCE are the recommended default. Each requested topic is treated independently—no sections are merged.

---

## Topic Overview

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | OAuth 2.0 is an open standard for **delegated authorization** that lets a third-party application obtain limited access to an HTTP service, either on behalf of a resource owner or on its own behalf. OpenID Connect is a **simple identity layer on top of OAuth 2.0** that enables clients to verify the end-user’s identity and obtain basic profile information in an interoperable, REST-like manner. |
| **Technical Definition** | OAuth 2.0 (RFC 6749) defines four roles (resource owner, client, authorization server, resource server), four grant types (authorization code, implicit, resource owner password credentials, client credentials), and token-based access to protected resources. OpenID Connect Core 1.0 extends OAuth 2.0 with an **ID Token** (a signed JWT containing claims about the authentication event and the end-user), a **UserInfo endpoint**, and standardized authentication flows—most importantly the **authorization code flow**. In ASP.NET Core, external authentication is implemented through authentication handlers (`AddOpenIdConnect`, `AddOAuth`, provider-specific handlers such as `AddGoogle`/`AddMicrosoftAccount`) that orchestrate the redirect-based flow, validate tokens, and establish a local cookie session. |
| **Beginner-Friendly Explanation** | OAuth 2.0 is the **“valet key” for APIs**. It lets a user grant an app limited access to their data on another service **without sharing their password**. OpenID Connect is **OAuth 2.0 plus an ID card**: it adds a standard way for the app to learn **who the user is** (their identity), not just what they are allowed to access. In ASP.NET Core, you configure an OIDC handler, the user is redirected to an identity provider (Google, Microsoft Entra ID, Auth0, etc.) to sign in, and the app receives tokens that prove the user’s identity. |

### Key Characteristics

- **OAuth 2.0 is for authorization, not authentication** – Access tokens grant API access; they do not, by themselves, prove the user’s identity in a standard way. OIDC fills this gap with the **ID Token**.
- **Redirect-based flows** – The user is redirected to the identity provider’s authorization endpoint and back to the app via a registered redirect URI.
- **Token-centric** – Access tokens (and optionally refresh tokens) are issued by the authorization server; OIDC additionally issues an ID Token.
- **Claims-based** – Both access tokens and ID tokens carry claims; OIDC standardizes identity claims (`sub`, `name`, `email`, etc.).
- **Pluggable identity providers** – ASP.NET Core supports OIDC, OAuth 2.0, WS-Federation, and SAML2p through authentication handlers.
- **Security BCP driven** – RFC 9700 (BCP 240) mandates PKCE, exact redirect URI matching, and sender-constrained tokens for modern OAuth 2.0 deployments.

### Prerequisites

- .NET SDK (6.0+ recommended), ASP.NET Core project.
- Working knowledge of **authentication fundamentals**, **middleware**, and **claims-based identity**.
- Familiarity with **dependency injection** and service registration.
- An account with an identity provider (Microsoft Entra ID, Google, Auth0, Duende IdentityServer, Keycloak, etc.) and the ability to register a client application.

### Related Programming Areas

- **Authentication fundamentals** – Schemes, handlers, and the `ClaimsPrincipal`.
- **Cookie authentication** – OIDC handlers typically sign the user in to a local cookie session after successful external authentication.
- **JWT authentication** – ID tokens and access tokens are often JWTs; validation parameters overlap with JWT bearer configuration.
- **Authorization** – Policies and roles evaluate the claims established during external authentication.
- **Data Protection** – Protects the correlation cookie and state used during the redirect flow.
- **Backend for Frontend (BFF)** – A recommended architecture for SPAs that avoids exposing tokens to the browser.

### Core Concepts / Features

1. **OAuth** – The authorization framework, roles, and delegated access model.
2. **OAuth2** – The protocol: grant types, endpoints, tokens, and flows.
3. **OpenID Connect** – The identity layer: ID Token, UserInfo, authentication flows, and claims.
4. **Identity providers** – The entities that authenticate users and issue tokens (IdP/OP).
5. **External authentication** – ASP.NET Core handlers for OIDC and OAuth 2.0 providers.
6. **OAuth best practices** – RFC 9700 security requirements, PKCE, redirect URI matching, and token protection.

Each concept is elaborated below using a uniform structure: **Definitions**, **Purposes**, **Syntax Rules & Structure**, **Annotated Code Examples**, **Real-World Cases**, and **References**.

---

## 1. OAuth (The Authorization Framework)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | OAuth is an open standard for **delegated authorization** that enables a third-party application to obtain limited access to an HTTP service on behalf of a resource owner, without sharing the resource owner’s credentials. |
| **Technical Definition** | OAuth 2.0 (RFC 6749) defines an authorization framework with four roles: **resource owner** (the user), **client** (the application requesting access), **authorization server** (issues tokens after authenticating the resource owner and obtaining authorization), and **resource server** (hosts the protected resources and accepts access tokens). The framework defines grant types (authorization code, implicit, resource owner password credentials, client credentials) that determine how the client obtains an access token. OAuth 2.0 replaced and obsoleted OAuth 1.0 (RFC 5849). |
| **Beginner-Friendly Explanation** | OAuth is the **“delegated access” system**. Imagine you want a third-party app to read your calendar without giving it your Google password. OAuth lets you tell Google: “Give this app a limited key that can only read my calendar.” Google issues a token to the app, and the app uses that token to access your calendar. You never share your password, and you can revoke the token at any time. |

### Purposes

- **To** enable third-party applications to access protected resources **without sharing user credentials**.
- **To** provide a standardized **delegation model** where resource owners can grant limited, scoped access to clients.
- **To** decouple **authorization** (what the client can do) from **authentication** (how the user proves who they are), allowing each concern to evolve independently.
- **To** support multiple client types (web, mobile, desktop, machine-to-machine) through different grant types.
- **To** serve as the foundation for **OpenID Connect**, which adds standardized identity to the OAuth 2.0 authorization framework.

### Syntax Rules and Structure

#### OAuth 2.0 Roles

```text
Resource Owner (User)
      │ grants authorization
      ▼
Authorization Server ──issues token──▶ Client (App)
                                          │ presents token
                                          ▼
                                    Resource Server (API)
```

**Component breakdown:**

| Role | Responsibility |
|------|----------------|
| **Resource Owner** | The entity capable of granting access to a protected resource (typically the end-user). |
| **Client** | The application requesting access to the resource on the resource owner’s behalf. |
| **Authorization Server** | Authenticates the resource owner and issues access tokens after obtaining authorization. |
| **Resource Server** | Hosts the protected resources and validates access tokens. |

#### OAuth 2.0 Grant Types

| Grant Type | Use Case | Security Status |
|------------|----------|-----------------|
| **Authorization Code** | Web apps, mobile apps, SPAs (with PKCE) | **Recommended** |
| **Implicit** | Legacy SPAs | **Deprecated** by RFC 9700 |
| **Resource Owner Password Credentials** | Legacy first-party apps | **Deprecated** by RFC 9700 |
| **Client Credentials** | Machine-to-machine | **Recommended** for service accounts |

**Syntax rules:**

- **OAuth 2.0 is an authorization framework, not an authentication protocol**—it does not define how to verify the user’s identity. Use **OpenID Connect** for authentication.
- **Access tokens** are bearer tokens by default; anyone holding the token can use it. Sender-constrained tokens (DPoP, mTLS) mitigate theft but add complexity.
- **The authorization code flow with PKCE** is the recommended grant type for interactive applications.

**Constraints and limitations:**

- **Implicit grant and resource owner password credentials are deprecated** by RFC 9700 and should not be used in new applications.
- **OAuth 2.0 alone cannot tell you who the user is**—it only tells you that the client has been granted access. OIDC’s ID Token provides identity.

### Annotated Complete Step-by-Step Code Example

```csharp
// Conceptual OAuth 2.0 client registration in ASP.NET Core
// (Provider-specific; OIDC is preferred for authentication)

builder.Services.AddAuthentication()
    .AddOAuth("ProviderName", options =>
    {
        options.ClientId = builder.Configuration["OAuth:ClientId"]!;
        options.ClientSecret = builder.Configuration["OAuth:ClientSecret"]!;
        options.AuthorizationEndpoint = "https://provider.com/oauth/authorize";
        options.TokenEndpoint = "https://provider.com/oauth/token";
        options.UserInformationEndpoint = "https://provider.com/oauth/userinfo";
        options.CallbackPath = "/signin-provider";
        options.Scope.Add("read:profile");
    });
```

**Step-by-step explanation:**

1. `AddOAuth` registers a generic OAuth 2.0 handler for a provider that does not have a dedicated ASP.NET Core handler.
2. `ClientId` and `ClientSecret` are the credentials issued by the provider when the client is registered.
3. `AuthorizationEndpoint` and `TokenEndpoint` are the provider’s OAuth 2.0 endpoints.
4. `UserInformationEndpoint` is used to retrieve the user’s profile after token acquisition.
5. `CallbackPath` must match the redirect URI registered with the provider.
6. `Scope` specifies the permissions the client requests.

**Expected Output:** When a user triggers a challenge, the handler redirects to the provider’s authorization endpoint. After the user consents, the provider redirects back to `/signin-provider`, and the handler exchanges the authorization code for an access token and retrieves the user’s profile.

**Why this output?** The OAuth 2.0 handler orchestrates the redirect-based authorization code flow, obtaining an access token that the application can use to call the provider’s API. **For authentication** (signing the user in), OIDC is preferred because it provides a standardized identity token.

### Real-World Cases

- **“Sign in with Google/GitHub”** – Third-party apps obtain limited access to a user’s profile and data without the user sharing credentials.
- **API-to-API access** – A background service uses the client credentials grant to obtain an access token for a protected API.
- **Delegated resource access** – A calendar app reads a user’s calendar events after the user grants permission via OAuth.

### References

- RFC 6749: The OAuth 2.0 Authorization Framework – https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.0 (IBM) – https://docs.verify.ibm.com
- OAuth 2.0 Part I (Microsoft Learn) – https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/dn829806(v=msdn.10)

---

## 2. OAuth2 (The Protocol)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | OAuth 2.0 is the second generation of the OAuth protocol—an authorization framework that uses access tokens to grant clients limited access to protected resources on behalf of a resource owner. |
| **Technical Definition** | OAuth 2.0 defines the **authorization code**, **implicit**, **resource owner password credentials**, and **client credentials** grant types; the **authorization endpoint**, **token endpoint**, and **redirection endpoint**; and the **access token** and **refresh token** artifacts. RFC 6749 is the core specification; RFC 6750 defines bearer token usage; RFC 7636 defines **PKCE** (Proof Key for Code Exchange), which is mandatory for public clients and recommended for confidential clients under RFC 9700. |
| **Beginner-Friendly Explanation** | OAuth 2.0 is the **actual protocol** that implements the OAuth idea. It defines the steps: the app sends you to the provider’s login page, you sign in and approve access, the provider sends the app a short code, the app exchanges that code for an access token, and the app uses the token to call the API. OAuth 2.0 also defines how tokens expire and how to refresh them. |

### Purposes

- **To** provide a concrete, interoperable protocol for delegated authorization across web, mobile, and machine-to-machine clients.
- **To** define **grant types** that match different client capabilities and security requirements.
- **To** standardize **token issuance, refresh, and revocation** through the authorization server.
- **To** support **scoped access** so clients request only the permissions they need.
- **To** enable **PKCE** and other extensions that harden the authorization code flow against interception and CSRF attacks.

### Syntax Rules and Structure

#### Authorization Code Flow (with PKCE)

```text
1. Client generates code_verifier; computes code_challenge = BASE64URL(SHA256(code_verifier))
2. Client redirects user to:
   GET /authorize?response_type=code
        &client_id=CLIENT_ID
        &redirect_uri=REDIRECT_URI
        &scope=SCOPE
        &state=STATE
        &code_challenge=CODE_CHALLENGE
        &code_challenge_method=S256
3. Authorization server authenticates user and redirects back:
   REDIRECT_URI?code=AUTH_CODE&state=STATE
4. Client exchanges code at token endpoint:
   POST /token
   grant_type=authorization_code
   &code=AUTH_CODE
   &redirect_uri=REDIRECT_URI
   &client_id=CLIENT_ID
   &code_verifier=CODE_VERIFIER
5. Token endpoint returns access_token, (refresh_token), token_type, expires_in
```

**Component breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `response_type=code` | Requests the authorization code grant. |
| `client_id` | Identifies the client registered with the authorization server. |
| `redirect_uri` | Where the authorization server sends the user after consent. Must **exactly match** a registered URI. |
| `scope` | The permissions requested. |
| `state` | An opaque value that the client uses to prevent CSRF; must be validated on callback. |
| `code_challenge` / `code_challenge_method` | PKCE parameters; `S256` is the recommended method. |
| `code_verifier` | The original random string; proves the client that initiated the flow is the same one exchanging the code. |

**Syntax rules:**

- **PKCE is mandatory for public clients** and recommended for confidential clients under RFC 9700.
- **Exact redirect URI matching** is required; pattern or prefix matching is not permitted.
- **The `state` parameter must be validated** on callback to prevent CSRF.
- **Authorization codes are single-use** and short-lived.

**Constraints and limitations:**

- **Implicit grant is deprecated** because it exposes access tokens in the URL fragment; it should not be used for new applications.
- **Resource owner password credentials are deprecated** because they require the client to handle the user’s password directly.
- **Bearer tokens can be replayed** if intercepted; sender-constrained tokens (DPoP) are recommended for high-security scenarios.

### Annotated Complete Step-by-Step Code Example (PKCE in ASP.NET Core OIDC Handler)

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = builder.Configuration["OIDC:Authority"];
    options.ClientId = builder.Configuration["OIDC:ClientId"];
    options.ClientSecret = builder.Configuration["OIDC:ClientSecret"];
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.UsePkce = true;   // PKCE is enabled by default in modern handler
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
    options.MapInboundClaims = false;
});
```

**Step-by-step explanation:**

1. `AddOpenIdConnect` registers the OIDC handler, which implements the authorization code flow.
2. `ResponseType = Code` selects the authorization code flow (not implicit).
3. `UsePkce = true` enables PKCE; in modern ASP.NET Core versions, PKCE is enabled by default for the code flow.
4. `SaveTokens = true` stores the access and refresh tokens in the authentication properties so the app can call downstream APIs.
5. `GetClaimsFromUserInfoEndpoint = true` retrieves additional claims from the UserInfo endpoint.
6. `MapInboundClaims = false` preserves standard claim names (`sub`, `name`, etc.).

**Expected Output:** The user is redirected to the OIDC provider, signs in, and is redirected back to the app. The handler exchanges the authorization code (with PKCE) for tokens, validates the ID Token, and establishes a local cookie session.

**Why this output?** The OIDC handler implements the authorization code flow with PKCE, which is the recommended OAuth 2.0 flow for interactive applications. The ID Token provides identity; the access token is used for API calls.

### Real-World Cases

- **Web application sign-in:** OIDC authorization code flow with PKCE for browser-based apps.
- **Mobile app sign-in:** Authorization code flow with PKCE using a custom URI scheme or loopback redirect.
- **API access:** Access tokens obtained through the code flow are used to call downstream APIs.
- **Machine-to-machine:** Client credentials grant for background services.

### References

- RFC 6749: The OAuth 2.0 Authorization Framework – https://www.rfc-editor.org/rfc/rfc6749
- RFC 7636: Proof Key for Code Exchange (PKCE) – https://www.rfc-editor.org/rfc/rfc7636
- RFC 9700: Best Current Practice for OAuth 2.0 Security – https://www.rfc-editor.org/rfc/rfc9700
- OWASP OAuth 2.0 Protocol Cheatsheet – https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html

---

## 3. OpenID Connect (The Identity Layer)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | OpenID Connect (OIDC) is a **simple identity layer on top of OAuth 2.0** that enables clients to verify the identity of the end-user based on the authentication performed by an authorization server, and to obtain basic profile information about the end-user in an interoperable, REST-like manner. |
| **Technical Definition** | OIDC Core 1.0 extends OAuth 2.0 with an **ID Token**—a signed JWT containing claims about the authentication event and the end-user (`iss`, `sub`, `aud`, `exp`, `iat`, and optional profile claims)—and a **UserInfo endpoint** for retrieving additional claims. It standardizes the **authorization code flow** for authentication, defines **scopes** (`openid`, `profile`, `email`), and introduces the **OpenID Provider (OP)** role (an authorization server that implements OIDC) and the **Relying Party (RP)** (the client). |
| **Beginner-Friendly Explanation** | OpenID Connect is **OAuth 2.0 plus a standardized ID card**. OAuth 2.0 tells the app what the user is allowed to access; OIDC tells the app **who the user is**. After the user signs in at the identity provider, the app receives an **ID Token**—a signed document that says “this is Alice, her unique ID is 12345, and she authenticated at 10:00 AM.” The app can trust this token because it is signed by the provider. |

### Purposes

- **To** provide a **standardized authentication protocol** on top of OAuth 2.0, eliminating the need for provider-specific authentication logic.
- **To** deliver a **verifiable identity assertion** (the ID Token) that the client can validate and use to establish a local session.
- **To** standardize **claims** about the end-user (`sub`, `name`, `email`, `picture`, etc.) so clients can consume identity data uniformly across providers.
- **To** define interoperable **authentication flows** (authorization code flow, implicit flow, hybrid flow) and their security requirements.
- **To** support **discovery** via the `/.well-known/openid-configuration` document, enabling clients to configure themselves automatically.

### Syntax Rules and Structure

#### OIDC Authentication Request (Authorization Code Flow)

```text
GET /authorize?
  response_type=code
  &client_id=CLIENT_ID
  &redirect_uri=REDIRECT_URI
  &scope=openid%20profile%20email
  &state=STATE
  &nonce=NONCE
  &code_challenge=CODE_CHALLENGE
  &code_challenge_method=S256
```

**Component breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `scope=openid` | **Required**—requests an ID Token. |
| `scope=profile email` | Optional scopes for additional claims. |
| `nonce` | A random value that binds the ID Token to the client session; mitigates replay attacks. |
| `response_type=code` | Uses the authorization code flow (recommended). |
| `code_challenge` / `code_challenge_method` | PKCE parameters (recommended). |

#### ID Token Validation (Minimum Checks)

| Check | Description |
|-------|-------------|
| **Signature** | Verify the token is signed by the OP using the key from the JWKS endpoint. |
| **Issuer (`iss`)** | Must match the expected OP issuer identifier. |
| **Audience (`aud`)** | Must contain the client’s `client_id`. |
| **Expiration (`exp`)** | Must be in the future (with a small clock skew). |
| **Nonce** | Must match the `nonce` sent in the authentication request. |
| **Subject (`sub`)** | A locally unique, never-reassigned identifier for the user at the OP. |

**Syntax rules:**

- **The `openid` scope is mandatory** for an OIDC authentication request; without it, the response is a plain OAuth 2.0 response without an ID Token.
- **The `nonce` must be included** and validated to prevent ID Token replay.
- **ID Tokens must be validated** before the user is signed in; ASP.NET Core’s OIDC handler performs these validations automatically when configured correctly.
- **The UserInfo endpoint** (if available) can be called with the access token to retrieve additional claims.

**Constraints and limitations:**

- **OIDC is not a replacement for OAuth 2.0**—it is a layer on top; OAuth 2.0 remains the authorization framework.
- **The implicit flow is deprecated** by OAuth 2.0 security BCP and should not be used for new applications.
- **ID Tokens must not be used as access tokens**—they are intended for the client, not for calling APIs.

### Annotated Complete Step-by-Step Code Example (ASP.NET Core OIDC Configuration)

```csharp
// Program.cs
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    var oidc = builder.Configuration.GetSection("OpenIDConnectSettings");
    options.Authority = oidc["Authority"];          // e.g., https://login.microsoftonline.com/{tenant}/v2.0
    options.ClientId = oidc["ClientId"];
    options.ClientSecret = oidc["ClientSecret"];
    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.UsePkce = true;
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
    options.MapInboundClaims = false;
    options.TokenValidationParameters.NameClaimType = "name";
    options.TokenValidationParameters.RoleClaimType = "roles";
});

builder.Services.AddAuthorizationBuilder()
    .SetFallbackPolicy(new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build());

var app = builder.Build();

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapRazorPages();

app.Run();
```

**Step-by-step explanation:**

1. `AddAuthentication` sets the cookie scheme as the default for local sessions and the OIDC scheme as the default challenge.
2. `AddCookie` establishes the local session after external authentication.
3. `AddOpenIdConnect` configures the OIDC handler with the provider’s authority, client credentials, and redirect URI.
4. `ResponseType = Code` selects the authorization code flow.
5. `UsePkce = true` enables PKCE.
6. `SaveTokens = true` stores tokens for downstream API calls.
7. `GetClaimsFromUserInfoEndpoint = true` retrieves additional claims from the UserInfo endpoint.
8. `MapInboundClaims = false` preserves standard claim names.
9. The fallback policy requires authentication for all endpoints by default.
10. Middleware order: `UseAuthentication()` before `UseAuthorization()`.

**Expected Output:** An unauthenticated request to a protected page triggers a challenge, redirecting the user to the OIDC provider’s login page. After successful authentication, the user is redirected back, the ID Token is validated, a local cookie session is established, and the user is signed in.

**Why this output?** The OIDC handler orchestrates the authorization code flow, validates the ID Token, and signs the user in to the local cookie scheme—combining external identity with a local session.

### Real-World Cases

- **“Sign in with Microsoft/Google/Auth0”** – OIDC provides a standardized way to authenticate users across providers.
- **Enterprise SSO** – Employees sign in with their corporate identity provider (Microsoft Entra ID, Okta, Ping) via OIDC.
- **Consumer apps** – Social login with Google, Apple, or Facebook using OIDC (where supported) or OAuth 2.0 with provider-specific user-info endpoints.

### References

- OpenID Connect Core 1.0 – https://openid.net/specs/openid-connect-core-1_0.html
- Configure OpenID Connect Web (UI) authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- OpenID Connect (OIDC) overview (Oracle) – https://docs.oracle.com/en/cloud/paas/identity-cloud/rest-api/opc-rest-api-oidc.html

---

## 4. Identity Providers

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | An identity provider (IdP), also called an **OpenID Provider (OP)** in OIDC, is a trusted system that authenticates users and issues identity assertions or tokens (ID Tokens, access tokens) about them to relying parties. |
| **Technical Definition** | In OIDC, the **OpenID Provider (OP)** is an authorization server that implements OIDC—it authenticates the end-user, obtains authorization, and returns an ID Token and (usually) an access token. The OP publishes its configuration at `/.well-known/openid-configuration`, including its authorization, token, UserInfo, and JWKS endpoints. The client (Relying Party) trusts the OP’s signing keys and issuer identifier. |
| **Beginner-Friendly Explanation** | An identity provider is the **service that knows who you are**—Google, Microsoft, your company’s login system, or a specialized service like Auth0 or Okta. When you click “Sign in with Google,” Google is the identity provider. It authenticates you and tells the app “this is Alice, and here is proof.” The app trusts Google because it has agreed to trust Google’s signatures. |

### Purposes

- **To** centralize **user authentication** in a trusted system, reducing the burden on individual applications to manage credentials.
- **To** issue **verifiable identity assertions** (ID Tokens) that relying parties can validate and trust.
- **To** enable **single sign-on (SSO)** across multiple applications that trust the same IdP.
- **To** support **federated identity**—users can bring their existing accounts from a provider instead of creating new credentials for each app.
- **To** offload **security-sensitive operations** (password storage, MFA, account recovery) to a specialized provider.

### Syntax Rules and Structure

#### OIDC Discovery Document

```text
GET https://{issuer}/.well-known/openid-configuration

Response (abbreviated):
{
  "issuer": "https://{issuer}",
  "authorization_endpoint": "https://{issuer}/authorize",
  "token_endpoint": "https://{issuer}/token",
  "userinfo_endpoint": "https://{issuer}/userinfo",
  "jwks_uri": "https://{issuer}/.well-known/jwks.json",
  "scopes_supported": ["openid", "profile", "email"],
  "response_types_supported": ["code"],
  "code_challenge_methods_supported": ["S256"]
}
```

**Component breakdown:**

| Field | Purpose |
|-------|---------|
| `issuer` | The unique identifier of the OP; must match the `iss` claim in tokens. |
| `authorization_endpoint` | Where the client redirects the user to authenticate. |
| `token_endpoint` | Where the client exchanges the authorization code for tokens. |
| `userinfo_endpoint` | Where the client retrieves additional user claims. |
| `jwks_uri` | The JSON Web Key Set endpoint for signature verification. |
| `code_challenge_methods_supported` | Indicates PKCE support (`S256` recommended). |

**Syntax rules:**

- **The `Authority` in ASP.NET Core’s OIDC options** should be the issuer identifier; the handler appends `/.well-known/openid-configuration` to discover the endpoints.
- **The `issuer` in tokens must exactly match** the discovered issuer; mismatches must be rejected.
- **The JWKS endpoint** provides the public keys used to verify token signatures; key rotation is handled by the handler.

**Constraints and limitations:**

- **Not all providers fully implement OIDC**; some only support OAuth 2.0 with proprietary user-info endpoints. In those cases, use the generic OAuth handler.
- **Provider-specific quirks** (claim names, endpoints, PKCE support) may require configuration adjustments.

### Annotated Code Example (Configuring an OIDC Identity Provider)

```csharp
.AddOpenIdConnect(options =>
{
    options.Authority = "https://login.microsoftonline.com/{tenant}/v2.0";
    options.ClientId = "your-client-id";
    options.ClientSecret = "your-client-secret";
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.UsePkce = true;
    options.Scope.Add("openid");
    options.Scope.Add("profile");
    options.Scope.Add("email");
    options.SaveTokens = true;
});
```

**Expected Output:** The handler discovers the provider’s endpoints via `/.well-known/openid-configuration`, redirects the user to the `authorization_endpoint`, and validates the ID Token using the provider’s `jwks_uri`.

**Why this output?** The `Authority` tells the handler which OP to trust; discovery provides the endpoints and keys; the scopes request the ID Token and profile claims.

### Real-World Cases

- **Microsoft Entra ID** – Enterprise and consumer identity provider with OIDC support.
- **Google Identity** – Consumer identity provider; OIDC for authentication, OAuth 2.0 for API access.
- **Auth0 / Okta** – Identity platforms that provide OIDC and OAuth 2.0 as a service.
- **Duende IdentityServer / Keycloak** – Self-hosted OIDC/OAuth 2.0 providers.

### References

- OpenID Connect Core 1.0 – https://openid.net/specs/openid-connect-core-1_0.html
- Compare Microsoft identity providers (IdP vs OP) – https://learn.microsoft.com/en-us/training/modules/compare-microsoft-identity-providers/
- Identity Provider (IdP) definition (Zenodo) – https://zenodo.org/record/龙泉驿

---

## 5. External Authentication (ASP.NET Core Implementation)

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | External authentication in ASP.NET Core is the ability to delegate user authentication to an external identity provider (OIDC or OAuth 2.0) and establish a local session based on the external identity. |
| **Technical Definition** | ASP.NET Core provides authentication handlers for OIDC (`AddOpenIdConnect`), generic OAuth 2.0 (`AddOAuth`), and provider-specific OAuth 2.0 providers (`AddGoogle`, `AddFacebook`, `AddMicrosoftAccount`, `AddTwitter`). These handlers implement the redirect-based flow, validate tokens, and sign the user in to a **local cookie scheme** (`SignInScheme`). The external login is then linked to a local user account, typically through ASP.NET Core Identity’s `SignInManager` or a custom user store. |
| **Beginner-Friendly Explanation** | External authentication is the **“Sign in with…” button**. Instead of making users create a new password for your app, you let them sign in with an account they already have (Google, Microsoft, etc.). ASP.NET Core handles the redirects and token validation, and then creates a local session for the user. |

### Purposes

- **To** let users sign in with **existing credentials** from a trusted provider, improving usability and reducing password fatigue.
- **To** offload **credential management** (password storage, MFA, recovery) to the identity provider.
- **To** support **multiple providers** in the same application, giving users a choice of identity providers.
- **To** establish a **local session** (cookie) after external authentication, so the user does not need to re-authenticate on every request.
- **To** link external identities to **local user accounts**, enabling authorization and personalization within the application.

### Syntax Rules and Structure

#### General Syntax (OIDC External Authentication)

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = Configuration["OIDC:Authority"];
    options.ClientId = Configuration["OIDC:ClientId"];
    options.ClientSecret = Configuration["OIDC:ClientSecret"];
    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;
});
```

**Component breakdown:**

| Component | Description |
|-----------|-------------|
| `DefaultScheme` | The cookie scheme used for local sessions. |
| `DefaultChallengeScheme` | The OIDC scheme used to challenge unauthenticated users. |
| `AddCookie()` | Registers the local cookie handler. |
| `AddOpenIdConnect()` | Registers the OIDC handler. |
| `SignInScheme` | The scheme used to sign the user in after external authentication (the cookie scheme). |
| `SaveTokens` | Stores access and refresh tokens in the authentication properties. |

#### General Syntax (Multiple Providers)

```csharp
builder.Services.AddAuthentication()
    .AddGoogle(options => { /* ... */ })
    .AddFacebook(options => { /* ... */ })
    .AddMicrosoftAccount(options => { /* ... */ });
```

**Syntax rules:**

- **A local cookie scheme is required** to persist the session after external authentication.
- **The callback path** (e.g., `/signin-oidc`) must be registered with the provider as a valid redirect URI.
- **Secrets must be stored securely** (User Secrets in development, Azure Key Vault in production).
- **Forwarded headers middleware** must be configured if the app is behind a proxy, so redirect URIs use the correct scheme and host.

**Constraints and limitations:**

- **Public clients are no longer recommended** for web applications; use confidential clients with a client secret or client assertion.
- **Provider-specific configuration** may be required (e.g., claim mapping, PKCE support, UserInfo endpoint availability).
- **External login does not automatically create a local account**—you must link the external identity to a local user (e.g., via ASP.NET Core Identity or a custom mapping).

### Annotated Complete Step-by-Step Code Example (OIDC External Login with Local Linking)

```csharp
// Program.cs — OIDC with local cookie
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = Configuration["OIDC:Authority"];
    options.ClientId = Configuration["OIDC:ClientId"];
    options.ClientSecret = Configuration["OIDC:ClientSecret"];
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
});

// ExternalLogin callback page model (simplified)
[AllowAnonymous]
public class ExternalLoginModel : PageModel
{
    private readonly SignInManager<IdentityUser> _signInManager;

    public ExternalLoginModel(SignInManager<IdentityUser> signInManager)
        => _signInManager = signInManager;

    public async Task<IActionResult> OnGetCallbackAsync(string? returnUrl = null)
    {
        var info = await _signInManager.GetExternalLoginInfoAsync();
        if (info is null) return RedirectToPage("./Login");

        var result = await _signInManager.ExternalLoginSignInAsync(
            info.LoginProvider, info.ProviderKey, isPersistent: false);

        if (result.Succeeded) return LocalRedirect(returnUrl ?? "/");

        // Link to local account or prompt for registration
        return RedirectToPage("./RegisterExternal", new { returnUrl });
    }
}
```

**Step-by-step explanation:**

1. The OIDC handler challenges the user and redirects to the provider.
2. After authentication, the provider redirects back to the app’s callback path.
3. `GetExternalLoginInfoAsync()` retrieves the external login information (provider, key, and claims).
4. `ExternalLoginSignInAsync` attempts to sign in a user who already has a linked external login.
5. If no local account is linked, the user is redirected to a registration page to create one or link the external login.

**Expected Output:** A new user who signs in with Google is prompted to create a local account (or link the external login); a returning user with a linked login is signed in directly.

**Why this output?** External authentication establishes the external identity; the application must decide how to map it to a local user for authorization and personalization.

### Real-World Cases

- **Consumer apps with social login:** Google, Facebook, Microsoft account sign-in alongside local accounts.
- **Enterprise SSO:** Microsoft Entra ID, Okta, or Ping Identity for employee sign-in.
- **Multi-tenant SaaS:** Each tenant configures its own identity provider; the app uses OIDC to authenticate users against the tenant’s IdP.

### References

- External provider authentication in ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/
- Configure OpenID Connect Web (UI) authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- Using external login providers with Identity in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/
- Sign-in with External Identity Providers (IdentityServer) – https://docs.identityserver.com

---

## 6. OAuth Best Practices

### Definitions

| Type | Definition |
|------|------------|
| **Core Definition** | OAuth best practices are the security requirements and recommendations defined in **RFC 9700 (BCP 240)** and related standards for deploying OAuth 2.0 and OIDC securely. |
| **Technical Definition** | RFC 9700 updates and extends RFCs 6749, 6750, and 6819 with practical security guidance. It mandates **PKCE for public clients** (and recommends it for confidential clients), **exact redirect URI matching**, **CSRF prevention** (via `state` or PKCE), **token replay prevention** (sender-constrained tokens), and **deprecates the implicit grant and resource owner password credentials**. ASP.NET Core’s OIDC handler implements many of these requirements by default; developers must configure the handler correctly and avoid insecure overrides. |
| **Beginner-Friendly Explanation** | OAuth best practices are the **“rules of the road”** for using OAuth safely. The biggest rules are: always use PKCE, never use the old implicit flow, make sure redirect URIs match exactly, validate the `state` parameter, and don’t let tokens be replayed. Following these rules prevents attackers from stealing tokens or hijacking the login flow. |

### Purposes

- **To** prevent **authorization code interception and injection** attacks through PKCE and exact redirect URI matching.
- **To** prevent **CSRF attacks** on the redirect flow through `state` validation.
- **To** prevent **token replay** through short token lifetimes and sender-constrained tokens (DPoP, mTLS).
- **To** deprecate and eliminate **insecure grant types** (implicit, resource owner password credentials).
- **To** ensure **tokens are bound to the correct client and context** through audience validation and issuer validation.

### Guidelines Checklist

| Area | Do | Don't |
|------|----|-------|
| **Grant type** | Use the **authorization code flow** for interactive clients. | Use the implicit grant or resource owner password credentials—both are deprecated. |
| **PKCE** | Enable PKCE (`UsePkce = true`); **mandatory** for public clients, recommended for confidential clients. | Omit PKCE for public clients. |
| **Redirect URI** | Register exact redirect URIs; use **exact string matching** on the authorization server. | Use wildcard or prefix matching. |
| **State parameter** | Include and validate `state` to prevent CSRF. | Skip `state` validation. |
| **Token binding** | Prefer **sender-constrained tokens** (DPoP, mTLS) for high-security APIs. | Accept plain bearer tokens without additional protections in high-risk environments. |
| **Token lifetime** | Keep access tokens **short-lived** (minutes). | Issue long-lived access tokens. |
| **Refresh tokens** | Use rotation with reuse detection; store securely server-side. | Allow refresh token reuse. |
| **Client authentication** | Use **confidential clients** with client secrets or assertions for web apps. | Use public clients for web applications. |
| **Scope** | Request **minimal scopes**; avoid `offline_access` unless refresh tokens are needed. | Request excessive or unnecessary scopes. |
| **Secrets** | Store client secrets in **Azure Key Vault** or User Secrets; never in source code. | Commit secrets to configuration files or source control. |
| **HTTPS** | Use HTTPS for all OAuth/OIDC endpoints and redirect URIs. | Allow plain HTTP for token or authorization endpoints. |
| **Validation** | Validate the ID Token’s **signature, issuer, audience, expiry, and nonce**. | Trust ID Tokens without validation. |

### Annotated Code Example (Security-Hardened OIDC Configuration)

```csharp
.AddOpenIdConnect(options =>
{
    options.Authority = "https://{your-authority}";
    options.ClientId = "your-client-id";
    options.ClientSecret = "your-client-secret";
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.UsePkce = true;                    // RFC 9700 requirement
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
    options.MapInboundClaims = false;
    options.TokenValidationParameters.ValidateIssuer = true;
    options.TokenValidationParameters.ValidateAudience = true;
    options.TokenValidationParameters.ValidateLifetime = true;
    options.TokenValidationParameters.ClockSkew = TimeSpan.FromSeconds(30);
    options.Scope.Clear();
    options.Scope.Add("openid");
    options.Scope.Add("profile");
    options.Scope.Add("email");
});
```

**Why this follows best practices:**

- **PKCE is enabled**—required by RFC 9700 for public clients and recommended for confidential clients.
- **The authorization code flow** is used—the implicit grant is deprecated.
- **ID Token validation** is explicit—issuer, audience, lifetime, and clock skew are configured.
- **Minimal scopes** are requested—`openid` is required; `profile` and `email` are added only as needed.
- **`MapInboundClaims = false`** preserves standard claim names.
- **Secrets are loaded from configuration** (backed by a vault in production).

### Real-World Cases

- **High-security financial APIs:** PKCE, exact redirect URI matching, sender-constrained tokens (DPoP), short access-token lifetimes, and refresh token rotation.
- **Enterprise SSO deployments:** Confidential clients with client secrets stored in Azure Key Vault; ID Token validation with strict issuer and audience checks.
- **SPAs with BFF pattern:** The backend-for-frontend handles OAuth flows, stores tokens server-side, and issues secure cookies to the SPA, avoiding token exposure in the browser.

### References

- RFC 9700: Best Current Practice for OAuth 2.0 Security – https://www.rfc-editor.org/rfc/rfc9700
- OAuth 2.0 Security Best Practices (2026): RFC 9700 Guide – https://safeguard.sh/resources/blog/oauth-2-0-security-best-practices
- OWASP OAuth 2.0 Protocol Cheatsheet – https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html
- ASP.NET Core OIDC configuration (PKCE, exact redirect, secrets) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- RFC 7636: Proof Key for Code Exchange (PKCE) – https://www.rfc-editor.org/rfc/rfc7636

---

## Summary Table: OAuth and OpenID Connect at a Glance

| Concept | Primary Role | Key Standard / API | When to Use |
|---------|--------------|---------------------|-------------|
| **OAuth** | Delegated authorization framework | RFC 6749; roles: resource owner, client, authorization server, resource server | When an app needs limited access to a user’s resources without their password |
| **OAuth2** | Concrete protocol for delegated authorization | Grant types: authorization code (PKCE), client credentials; RFC 6749, 7636, 9700 | Interactive apps, SPAs, mobile apps, machine-to-machine |
| **OpenID Connect** | Identity layer on top of OAuth 2.0 | ID Token, UserInfo endpoint, `openid` scope; OIDC Core 1.0 | When the app needs to **authenticate** users and know who they are |
| **Identity providers** | Trusted systems that authenticate users and issue tokens | OIDC discovery (`/.well-known/openid-configuration`), JWKS | Whenever external authentication is delegated |
| **External authentication** | ASP.NET Core handlers for OIDC and OAuth 2.0 | `AddOpenIdConnect`, `AddOAuth`, `AddGoogle`, etc.; local cookie session | “Sign in with…” scenarios, SSO, federated identity |
| **OAuth best practices** | Security requirements for OAuth 2.0 deployments | RFC 9700: PKCE, exact redirect URI matching, state validation, token replay prevention | Every production OAuth/OIDC deployment |

---

## References (Full List)

- RFC 6749: The OAuth 2.0 Authorization Framework – https://www.rfc-editor.org/rfc/rfc6749
- RFC 7636: Proof Key for Code Exchange (PKCE) – https://www.rfc-editor.org/rfc/rfc7636
- RFC 9700: Best Current Practice for OAuth 2.0 Security – https://www.rfc-editor.org/rfc/rfc9700
- OpenID Connect Core 1.0 – https://openid.net/specs/openid-connect-core-1_0.html
- Configure OpenID Connect Web (UI) authentication in ASP.NET Core – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- External provider authentication in ASP.NET Core Identity – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/
- OWASP OAuth 2.0 Protocol Cheatsheet – https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html
- OAuth 2.0 Security Best Practices (2026): RFC 9700 Guide – https://safeguard.sh/resources/blog/oauth-2-0-security-best-practices
- Compare Microsoft identity providers (IdP vs OP) – https://learn.microsoft.com/en-us/training/modules/compare-microsoft-identity-providers/
- Identity Provider (IdP) definition (Zenodo) – https://zenodo.org/record/龙泉驿
- Sign-in with External Identity Providers (IdentityServer) – https://docs.identityserver.com
- ASP.NET Core OIDC configuration (PKCE, exact redirect, secrets) – https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-oidc-web-authentication
- OpenID Connect (OIDC) overview (Oracle) – https://docs.oracle.com/en/cloud/paas/identity-cloud/rest-api/opc-rest-api-oidc.html
- OAuth 2.0 (IBM) – https://docs.verify.ibm.com
- OAuth 2.0 Part I (Microsoft Learn) – https://learn.microsoft.com/en-us/previous-versions/dotnet/articles/dn829806(v=msdn.10)