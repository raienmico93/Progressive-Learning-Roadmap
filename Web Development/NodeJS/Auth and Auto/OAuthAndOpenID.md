# OAuth 2.0 and OpenID Connect — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** OAuth 2.0 is an authorization framework that enables applications to obtain limited access to user accounts on an HTTP service, while OpenID Connect (OIDC) is an authentication layer built on top of OAuth 2.0 that adds identity verification and profile information.

**Technical Definition:** OAuth 2.0 (RFC 6749) is a delegation-based authorization framework in which a resource owner grants a client application access to protected resources via an authorization server, which issues access tokens. OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol that enables clients to verify the identity of the end user based on authentication performed by an authorization server, and to obtain basic profile information about the end user in an interoperable and REST-like manner. OIDC introduces the ID token (a JWT containing identity claims), the UserInfo endpoint, and standard scopes (`openid`, `profile`, `email`).

**Beginner-Friendly Explanation:** OAuth is like a valet key for your car. Instead of giving a valet your master key (your password), you give them a special key that only starts the engine and opens the trunk — nothing else. The valet can park your car, but can't open the glovebox or drive away permanently. OpenID Connect is like the valet also getting your name badge — it proves who you are, not just what you can do. Social login ("Sign in with Google") uses OIDC to verify your identity and OAuth to grant limited access to your Google data.

### Key Characteristics

- **Delegation, not sharing:** OAuth grants limited access without sharing credentials.
- **Authorization server:** Issues tokens after the resource owner grants consent.
- **Access tokens:** Grant access to specific scopes (API permissions).
- **Refresh tokens:** Enable long-term access without re-authentication.
- **ID tokens (OIDC):** Provide identity claims (name, email, picture).
- **PKCE:** Mandatory for public clients (SPAs, mobile apps) to prevent code interception.
- **Scopes and consent:** Granular permissions that users explicitly approve.
- **Standardised flows:** Authorization Code, Client Credentials, Device Authorization, and deprecated flows (Implicit, Password).

### Prerequisites

- **HTTP fundamentals:** Redirects, headers, status codes, cookies.
- **Cryptography basics:** JWT, JWS, JWKS, signing algorithms (RS256).
- **Token authentication:** Access tokens, refresh tokens, and JWT validation.
- **Session management:** Cookies, `HttpOnly`, `Secure`, `SameSite`.
- **Identity providers:** Auth0, Okta, Keycloak, AWS Cognito, Google, GitHub.
- **Node.js fundamentals:** Express or NestJS, Passport.js, openid-client.
- **PKCE:** Code verifier, code challenge, SHA-256 hashing.

### Related Programming Areas

- **Authentication:** OIDC provides identity; OAuth provides authorization.
- **Authorization:** Scopes define what the access token can do.
- **Single Sign-On (SSO):** OIDC enables SSO across multiple applications.
- **API gateways:** Validate access tokens before routing to backend services.
- **Microservices:** Tokens propagate identity and scopes across services.
- **Social login:** Google, GitHub, Apple, and Microsoft provide OIDC endpoints.

### Core Concepts

1. **Authorization Flows** — Authorization Code with PKCE, Client Credentials, and Device Authorization.
2. **Identity Providers** — integrating Auth0, Okta, Keycloak, or AWS Cognito.
3. **Access Tokens** — consuming delegated strings to access specific API scopes.
4. **ID Tokens** — consuming JWTs containing authenticated profile details.
5. **Refresh Tokens** — managing long-term offline user data access securely.
6. **Third-Party Authentication** — building social login with Google, GitHub, Apple, and Microsoft.
7. **Scopes and Consent Management** — defining granular API permissions and managing revocations.

---

## Core Concept 1: Authorization Flows

### Definitions

**Core Definition:** An authorization flow (or grant type) is the sequence of HTTP exchanges by which a client obtains tokens from an authorization server, depending on the client type and use case.

**Technical Definition:** OAuth 2.0 defines several grant types: **Authorization Code** (RFC 6749 §4.1) — the recommended flow for web apps and SPAs, exchanging a short-lived code for tokens; **Authorization Code with PKCE** (RFC 7636) — mandatory for public clients, using a code verifier/challenge to prevent code interception; **Client Credentials** (RFC 6749 §4.4) — for machine-to-machine (M2M) communication without a user; **Device Authorization** (RFC 8628) — for input-constrained devices (smart TVs, CLI tools) that display a code for the user to enter on another device; **Refresh Token** (RFC 6749 §1.5) — to obtain new access tokens; and **Implicit** (deprecated) and **Resource Owner Password** (deprecated) — no longer recommended.

**Beginner-Friendly Explanation:** Think of authorization flows as different ways to get a ticket. **Authorization Code** is like buying a ticket online — you get a receipt (code), and you exchange it for the actual ticket (token) at the counter. **PKCE** adds a secret handshake so nobody can steal your receipt. **Client Credentials** is like a robot buying a ticket for itself — no human involved. **Device Authorization** is like a smart TV showing you a code to enter on your phone, because the TV has no keyboard.

### Purposes

- To obtain access tokens with the appropriate scopes for the client type.
- To prevent token interception and replay attacks (PKCE).
- To enable machine-to-machine communication without a user (Client Credentials).
- To support input-constrained devices (Device Authorization).
- To obtain new access tokens without re-authentication (Refresh Token).
- To comply with OAuth 2.0 Security Best Current Practice (RFC 9700).

### Syntax Rules and Structure

#### Flow Comparison

| Flow | Client Type | User Involved | PKCE | Use Case |
|------|-------------|---------------|------|----------|
| **Authorization Code + PKCE** | Web, SPA, Mobile | ✅ Yes | ✅ Required for public | Login, user-delegated access |
| **Client Credentials** | Confidential (server) | ❌ No | ❌ N/A | M2M, service-to-service |
| **Device Authorization** | Public (input-constrained) | ✅ Yes (on another device) | ❌ N/A | Smart TVs, CLI tools |
| **Refresh Token** | Any | ❌ No | ❌ N/A | Long-term access |
| **Implicit** (deprecated) | SPA | ✅ Yes | ❌ N/A | ❌ Do not use |
| **Password** (deprecated) | Any | ✅ Yes | ❌ N/A | ❌ Do not use |

#### Authorization Code with PKCE Flow

```
1. Client generates code_verifier (random) and code_challenge = SHA256(code_verifier)
2. Client redirects user to /authorize?response_type=code&client_id=...&code_challenge=...
3. User authenticates and grants consent
4. Authorization server redirects to /callback?code=AUTH_CODE
5. Client exchanges code + code_verifier for tokens at /token
6. Authorization server validates code_verifier against code_challenge
7. Authorization server returns access_token, refresh_token, id_token
```

#### PKCE Implementation (Node.js)

```typescript
import { randomBytes, createHash } from 'node:crypto';

// 1. Generate code verifier and challenge
function generatePkce(): { verifier: string; challenge: string } {
  const verifier = randomBytes(32).toString('base64url'); // 43-128 chars
  const challenge = createHash('sha256').update(verifier).digest('base64url');
  return { verifier, challenge };
}

// 2. Build authorization URL
function buildAuthorizationUrl(params: {
  authorizationEndpoint: string;
  clientId: string;
  redirectUri: string;
  scope: string;
  state: string;
  codeChallenge: string;
}): string {
  const url = new URL(params.authorizationEndpoint);
  url.searchParams.set('response_type', 'code');
  url.searchParams.set('client_id', params.clientId);
  url.searchParams.set('redirect_uri', params.redirectUri);
  url.searchParams.set('scope', params.scope);
  url.searchParams.set('state', params.state);
  url.searchParams.set('code_challenge', params.codeChallenge);
  url.searchParams.set('code_challenge_method', 'S256');
  return url.toString();
}

// 3. Exchange code for tokens
async function exchangeCode(params: {
  tokenEndpoint: string;
  clientId: string;
  clientSecret?: string;
  redirectUri: string;
  code: string;
  codeVerifier: string;
}) {
  const body = new URLSearchParams({
    grant_type: 'authorization_code',
    code: params.code,
    redirect_uri: params.redirectUri,
    client_id: params.clientId,
    code_verifier: params.codeVerifier,
  });

  if (params.clientSecret) body.set('client_secret', params.clientSecret);

  const response = await fetch(params.tokenEndpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body,
  });

  return response.json();
}
```

#### Client Credentials Flow

```typescript
async function clientCredentialsGrant(params: {
  tokenEndpoint: string;
  clientId: string;
  clientSecret: string;
  scope: string;
}) {
  const body = new URLSearchParams({
    grant_type: 'client_credentials',
    client_id: params.clientId,
    client_secret: params.clientSecret,
    scope: params.scope,
  });

  const response = await fetch(params.tokenEndpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body,
  });

  const { access_token } = await response.json();
  return access_token;
}
```

#### Device Authorization Flow

```typescript
async function deviceAuthorizationFlow(deviceAuthorizationEndpoint: string, clientId: string, scope: string) {
  // 1. Request device and user codes
  const response = await fetch(deviceAuthorizationEndpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({ client_id: clientId, scope }),
  });

  const { device_code, user_code, verification_uri, interval, expires_in } = await response.json();

  console.log(`Go to ${verification_uri} and enter code: ${user_code}`);

  // 2. Poll for tokens
  const deadline = Date.now() + expires_in * 1000;
  while (Date.now() < deadline) {
    await new Promise((r) => setTimeout(r, interval * 1000));

    const tokenResponse = await fetch(/* token endpoint */, {
      method: 'POST',
      body: new URLSearchParams({
        grant_type: 'urn:ietf:params:oauth:grant-type:device_code',
        device_code,
        client_id: clientId,
      }),
    });

    const tokens = await tokenResponse.json();
    if (tokens.access_token) return tokens;
    if (tokens.error !== 'authorization_pending') throw new Error(tokens.error);
  }

  throw new Error('Device authorization timed out');
}
```

#### Syntax Rules

- **Use Authorization Code with PKCE for all user-facing clients** — mandatory for public clients.
- **Use Client Credentials only for M2M** — never for user authentication.
- **Use Device Authorization for input-constrained devices.**
- **Always use `state`** — prevents CSRF in the redirect.
- **Always use PKCE for public clients** — SPAs, mobile, desktop.
- **Never use Implicit or Password flows** — deprecated and insecure.
- **Always use HTTPS** — tokens must never traverse plain HTTP.
- **Validate `state` on callback** — reject mismatches.
- **Use `nonce` with OIDC** — prevents ID token replay.

#### Constraints and Limitations

- **Authorization Code requires a backend** — or PKCE for public clients.
- **Client Credentials has no user context** — cannot access user data.
- **Device Authorization requires a second device** — not suitable for all scenarios.
- **Token endpoints must be highly available** — outages block all logins.
- **Redirect URIs must be pre-registered** — prevents open redirectors.
- **Deprecated flows still exist** — legacy systems may require migration.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Authorization Code with PKCE (Express)

```typescript
// server.ts
import express from 'express';
import session from 'express-session';
import { randomBytes, createHash } from 'node:crypto';

const app = express();
app.use(session({ secret: process.env.SESSION_SECRET!, resave: false, saveUninitialized: false }));

const CONFIG = {
  authorizationEndpoint: 'https://auth.example.com/authorize',
  tokenEndpoint: 'https://auth.example.com/oauth/token',
  clientId: process.env.OAUTH_CLIENT_ID!,
  redirectUri: 'https://app.example.com/callback',
  scope: 'openid profile email read:posts',
};

// Step 1: Start the flow
app.get('/login', (req, res) => {
  const verifier = randomBytes(32).toString('base64url');
  const challenge = createHash('sha256').update(verifier).digest('base64url');
  const state = randomBytes(16).toString('base64url');

  // Store in session for later verification
  req.session.pkce = { verifier, state };

  const url = new URL(CONFIG.authorizationEndpoint);
  url.searchParams.set('response_type', 'code');
  url.searchParams.set('client_id', CONFIG.clientId);
  url.searchParams.set('redirect_uri', CONFIG.redirectUri);
  url.searchParams.set('scope', CONFIG.scope);
  url.searchParams.set('state', state);
  url.searchParams.set('code_challenge', challenge);
  url.searchParams.set('code_challenge_method', 'S256');

  res.redirect(url.toString());
});

// Step 2: Handle callback
app.get('/callback', async (req, res) => {
  const { code, state } = req.query;

  // Verify state (CSRF protection)
  if (state !== req.session.pkce?.state) {
    return res.status(400).send('Invalid state');
  }

  // Exchange code for tokens
  const response = await fetch(CONFIG.tokenEndpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code: code as string,
      redirect_uri: CONFIG.redirectUri,
      client_id: CONFIG.clientId,
      code_verifier: req.session.pkce.verifier,
    }),
  });

  const tokens = await response.json();

  // Store tokens in session (or issue your own session)
  req.session.tokens = tokens;
  delete req.session.pkce;

  res.redirect('/dashboard');
});

app.get('/dashboard', (req, res) => {
  if (!req.session.tokens) return res.redirect('/login');
  res.json({ accessToken: req.session.tokens.access_token });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `GET /login` redirects to the authorization server with a PKCE challenge.
- After authentication, the server redirects to `/callback` with an authorization code.
- The callback exchanges the code for tokens using the PKCE verifier.
- The tokens are stored in the session.

**Why this works:** PKCE prevents code interception. The `state` parameter prevents CSRF. The `code_verifier` proves the same client that started the flow is exchanging the code. All communication is over HTTPS.

### Real-World Cases

- **Web applications:** Authorization Code with PKCE (server-side).
- **SPAs:** Authorization Code with PKCE (browser-side, no client secret).
- **Mobile apps:** Authorization Code with PKCE (system browser).
- **M2M:** Client Credentials.
- **Smart TVs:** Device Authorization.
- **CLI tools:** Device Authorization or Authorization Code with PKCE (loopback).

---

## Core Concept 2: Identity Providers (Auth0, Okta, Keycloak, AWS Cognito)

### Definitions

**Core Definition:** An Identity Provider (IdP) is a service that manages user identities, authenticates users, and issues tokens via OAuth 2.0 / OpenID Connect.

**Technical Definition:** An IdP (also called an authorization server in OAuth terms) implements OAuth 2.0 and OpenID Connect endpoints: `/authorize` (authorization endpoint), `/oauth/token` (token endpoint), `/userinfo` (UserInfo endpoint), `/.well-known/openid-configuration` (discovery document), and `/.well-known/jwks.json` (JSON Web Key Set). Managed IdPs (Auth0, Okta, AWS Cognito, Azure AD B2C) handle user registration, login, MFA, password reset, social login, and token issuance. Self-hosted IdPs (Keycloak, Ory Hydra, Authentik) provide similar features with full control. Integration involves registering a client (client ID, client secret, redirect URIs), configuring scopes, and validating tokens.

**Beginner-Friendly Explanation:** An IdP is like a passport office. Instead of every app issuing its own IDs, apps trust a central authority (the IdP) to verify identities and issue passports (tokens). When you log into an app with "Sign in with Google," Google is the IdP — it authenticates you and tells the app "This is Alice, here's her profile." The app trusts Google, so it doesn't need to verify Alice itself.

### Purposes

- To centralise user identity management across multiple applications.
- To reduce the burden of implementing authentication from scratch.
- To enable Single Sign-On (SSO) across applications.
- To provide MFA, password reset, and social login out of the box.
- To support enterprise federation (SAML, LDAP, OIDC).
- To ensure compliance with security standards.

### Syntax Rules and Structure

#### IdP Comparison

| IdP | Type | Best For | Key Features |
|-----|------|----------|--------------|
| **Auth0** | Managed | Startups, SMBs | Social login, MFA, rules/actions, extensibility |
| **Okta** | Managed | Enterprises | SSO, SCIM, lifecycle management, federation |
| **Keycloak** | Self-hosted | Full control, on-prem | Open source, LDAP, SAML, OIDC, custom themes |
| **AWS Cognito** | Managed (AWS) | AWS-native apps | User pools, identity pools, Lambda triggers |
| **Azure AD B2C** | Managed (Microsoft) | Microsoft ecosystem | Custom policies, social login, MFA |
| **Ory Hydra** | Self-hosted | OAuth 2.0 / OIDC only | Headless, high performance |

#### OIDC Discovery Document

```json
{
  "issuer": "https://auth.example.com/",
  "authorization_endpoint": "https://auth.example.com/authorize",
  "token_endpoint": "https://auth.example.com/oauth/token",
  "userinfo_endpoint": "https://auth.example.com/userinfo",
  "jwks_uri": "https://auth.example.com/.well-known/jwks.json",
  "response_types_supported": ["code"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email"],
  "code_challenge_methods_supported": ["S256"]
}
```

#### Dynamic Discovery and Client Registration (openid-client)

```typescript
import { Issuer } from 'openid-client';

// Discover the IdP's configuration
const issuer = await Issuer.discover('https://auth.example.com');

const client = new issuer.Client({
  client_id: process.env.OAUTH_CLIENT_ID!,
  client_secret: process.env.OAUTH_CLIENT_SECRET!,
  redirect_uris: ['https://app.example.com/callback'],
  response_types: ['code'],
});

// Build authorization URL
const authUrl = client.authorizationUrl({
  scope: 'openid profile email',
  state: 'random-state',
  code_challenge: 'pkce-challenge',
  code_challenge_method: 'S256',
});

// Exchange code for tokens
const tokenSet = await client.callback(
  'https://app.example.com/callback',
  { code: 'auth-code', state: 'random-state' },
  { state: 'random-state', code_verifier: 'pkce-verifier' },
);

// Access tokens and ID token claims
console.log(tokenSet.access_token);
console.log(tokenSet.id_token);
console.log(tokenSet.claims());
```

#### Syntax Rules

- **Use OIDC discovery** — `/.well-known/openid-configuration` for endpoint configuration.
- **Register clients with the IdP** — get `client_id` and `client_secret`.
- **Configure redirect URIs exactly** — no wildcards in production.
- **Use the `openid` scope** — required for OIDC.
- **Validate tokens using the IdP's JWKS** — `/.well-known/jwks.json`.
- **Validate `iss`, `aud`, `exp`, `iat`, `nonce`** — per OIDC spec.
- **Support multiple IdPs** if required — use a library like `openid-client` or Passport.js.
- **Handle IdP errors gracefully** — `error`, `error_description` in redirects.
- **Cache JWKS** — with periodic refresh (e.g., every hour).

#### Constraints and Limitations

- **IdP outages block all logins** — high availability is essential.
- **Vendor lock-in:** Migrating from one IdP to another requires planning.
- **Cost:** Managed IdPs charge per monthly active user (MAU).
- **Customisation limits:** Managed IdPs may not support all custom flows.
- **Data residency:** Some IdPs store data in specific regions.
- **Compliance:** Verify the IdP meets your compliance requirements (SOC 2, HIPAA, GDPR).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: OIDC Integration with Auth0 (Express + openid-client)

```typescript
// auth.ts
import { Issuer, generators } from 'openid-client';
import express from 'express';
import session from 'express-session';

const app = express();
app.use(session({ secret: process.env.SESSION_SECRET!, resave: false, saveUninitialized: false }));

// Discover Auth0 configuration
const issuer = await Issuer.discover(`https://${process.env.AUTH0_DOMAIN}`);

const client = new issuer.Client({
  client_id: process.env.AUTH0_CLIENT_ID!,
  client_secret: process.env.AUTH0_CLIENT_SECRET!,
  redirect_uris: [`${process.env.APP_URL}/callback`],
  response_types: ['code'],
});

app.get('/login', (req, res) => {
  const codeVerifier = generators.codeVerifier();
  const codeChallenge = generators.codeChallenge(codeVerifier);
  const state = generators.state();
  const nonce = generators.nonce();

  req.session.oidc = { codeVerifier, state, nonce };

  const url = client.authorizationUrl({
    scope: 'openid profile email',
    state,
    nonce,
    code_challenge: codeChallenge,
    code_challenge_method: 'S256',
  });

  res.redirect(url);
});

app.get('/callback', async (req, res) => {
  const { codeVerifier, state, nonce } = req.session.oidc;

  const tokenSet = await client.callback(
    `${process.env.APP_URL}/callback`,
    client.callbackParams(req),
    { state, nonce, code_verifier: codeVerifier },
  );

  // Fetch user info
  const userinfo = await client.userinfo(tokenSet.access_token!);

  req.session.tokens = tokenSet;
  req.session.user = userinfo;

  res.redirect('/profile');
});

app.get('/profile', (req, res) => {
  if (!req.session.user) return res.redirect('/login');
  res.json(req.session.user);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `GET /login` redirects to Auth0 with PKCE and nonce.
- After authentication, Auth0 redirects to `/callback` with a code.
- The callback exchanges the code for tokens, fetches user info, and stores it in the session.
- `GET /profile` returns the user's profile.

**Why this works:** `Issuer.discover()` fetches the OIDC configuration. `generators` creates PKCE and nonce values. `client.callback()` validates the state, nonce, and code verifier. `client.userinfo()` fetches the user's profile using the access token.

### Real-World Cases

- **Startups:** Auth0 for fast time-to-market with social login and MFA.
- **Enterprises:** Okta for SSO, SCIM, and lifecycle management.
- **Self-hosted:** Keycloak for full control and on-prem deployments.
- **AWS-native:** Cognito for integration with AWS services.
- **Microsoft ecosystem:** Azure AD B2C for Microsoft-centric apps.

---

## Core Concept 3: Access Tokens

### Definitions

**Core Definition:** An access token is a credential issued by the authorization server that grants the client access to specific resources (APIs) with specific scopes.

**Technical Definition:** Access tokens are opaque strings or JWTs that represent the authorization granted to the client. In OAuth 2.0, access tokens are bound to scopes and audiences. They are presented to resource servers via the `Authorization: Bearer` header. Resource servers validate access tokens by: (1) verifying the JWT signature (if JWT); (2) checking `exp`, `iss`, `aud`, and `scope`; or (3) calling the introspection endpoint (if opaque). Access tokens should be short-lived (5–60 minutes). They must never be used for authentication — use ID tokens for that.

**Beginner-Friendly Explanation:** An access token is like a hotel key card. It opens specific doors (scopes) in a specific hotel (audience) for a limited time (expiration). You show it at each door, and the door checks it. The card doesn't tell you who the person is — that's what the ID token (your driver's license) is for. Access tokens are for access, not identity.

### Purposes

- To grant the client limited access to specific resources.
- To carry scopes that define what the token can do.
- To enable stateless verification at resource servers (JWT).
- To support audience restriction (tokens valid only for specific APIs).
- To limit the impact of theft via short TTLs.

### Syntax Rules and Structure

#### Access Token Validation (Node.js)

```typescript
import { jwtVerify, createRemoteJWKSet } from 'jose';

const JWKS = createRemoteJWKSet(
  new URL('https://auth.example.com/.well-known/jwks.json'),
);

async function validateAccessToken(token: string) {
  const { payload } = await jwtVerify(token, JWKS, {
    issuer: 'https://auth.example.com',
    audience: 'https://api.example.com',
    algorithms: ['RS256'],
    clockTolerance: 5,
  });

  // Validate scopes
  const scopes = (payload.scope as string ?? '').split(' ');
  if (!scopes.includes('read:posts')) {
    throw new Error('Missing required scope');
  }

  return payload;
}
```

#### Scope Enforcement Middleware

```typescript
function requireScope(scope: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const scopes = (req.user?.scope ?? '').split(' ');
    if (!scopes.includes(scope)) {
      return res.status(403).json({ error: 'Insufficient scope', required: scope });
    }
    next();
  };
}

app.get('/posts', requireAuth, requireScope('read:posts'), (req, res) => {
  res.json({ posts: [] });
});
```

#### Introspection (Opaque Tokens)

```typescript
async function introspectToken(token: string) {
  const response = await fetch('https://auth.example.com/oauth/introspect', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      Authorization: `Basic ${Buffer.from(`${clientId}:${clientSecret}`).toString('base64')}`,
    },
    body: new URLSearchParams({ token }),
  });

  const result = await response.json();
  if (!result.active) throw new Error('Token is not active');
  return result;
}
```

#### Syntax Rules

- **Use the `Authorization: Bearer <token>` header** — never query strings.
- **Validate `exp`, `iss`, `aud`** — per RFC 9068 (JWT Profile for OAuth 2.0 Access Tokens).
- **Validate scopes** — ensure the token has the required permissions.
- **Use short TTLs** — 5–60 minutes.
- **Cache JWKS** — with periodic refresh.
- **Never use access tokens for authentication** — use ID tokens.
- **Never log access tokens** — redact them from logs.
- **Use HTTPS** — always.

#### Constraints and Limitations

- **Access tokens cannot be revoked** without introspection or short TTLs.
- **Opaque tokens require introspection** — adds latency.
- **JWT size grows with claims** — keep payloads minimal.
- **Scope design is critical** — overly broad scopes violate least privilege.
- **Audience validation is often skipped** — a common security mistake.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Access Token Validation and Scope Enforcement (Express + jose)

```typescript
import express from 'express';
import { jwtVerify, createRemoteJWKSet } from 'jose';

const app = express();
const JWKS = createRemoteJWKSet(new URL('https://auth.example.com/.well-known/jwks.json'));

async function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) return res.status(401).json({ error: 'Missing token' });

  try {
    const { payload } = await jwtVerify(header.slice(7), JWKS, {
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
      algorithms: ['RS256'],
    });
    req.user = {
      id: payload.sub,
      scopes: (payload.scope ?? '').split(' '),
    };
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

function requireScope(scope) {
  return (req, res, next) => {
    if (!req.user.scopes.includes(scope)) {
      return res.status(403).json({ error: 'Insufficient scope' });
    }
    next();
  };
}

app.get('/posts', requireAuth, requireScope('read:posts'), (req, res) => {
  res.json({ posts: [] });
});

app.post('/posts', requireAuth, requireScope('write:posts'), (req, res) => {
  res.status(201).json({ id: 'post-1' });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:** Requests without a valid token return `401`. Requests with a token missing the required scope return `403`. Requests with the correct scope succeed.

**Why this works:** The middleware validates the JWT signature, issuer, and audience, then checks the scopes. This enforces least privilege at the API layer.

### Real-World Cases

- **API gateways:** Validate access tokens at the edge.
- **Microservices:** Access tokens propagate identity and scopes.
- **Third-party APIs:** Scoped access tokens for delegated access.
- **M2M:** Client Credentials access tokens for service-to-service calls.

---

## Core Concept 4: ID Tokens

### Definitions

**Core Definition:** An ID token is a JWT issued by the OpenID Connect provider that contains claims about the authenticated user (identity), used by the client to verify the user's identity.

**Technical Definition:** The ID token is a JWT (per OIDC Core 1.0) containing mandatory claims: `iss` (issuer), `sub` (subject — the user's unique identifier), `aud` (audience — the client ID), `exp` (expiration), `iat` (issued at), and `nonce` (when requested). Optional claims include `auth_time`, `acr`, `amr`, `azp`, and profile claims (`name`, `email`, `picture`, `locale`) when the corresponding scopes (`profile`, `email`) are requested. The ID token is signed (JWS) or encrypted (JWE) and must be validated by the client: verify the signature, `iss`, `aud`, `exp`, `iat`, and `nonce`. The ID token is for the client's consumption — it must never be sent to resource servers.

**Beginner-Friendly Explanation:** An ID token is like a passport. It says "This is Alice, born on this date, from this country" and is signed by the passport authority (the IdP). The app reads it to know who you are. But unlike a passport, you don't show it to every service — the app reads it once, then uses your session or an access token for subsequent requests.

### Purposes

- To verify the user's identity after authentication.
- To obtain profile information (name, email, picture) for personalisation.
- To enable Single Sign-On (SSO) across applications.
- To support session establishment (the client creates a session after validating the ID token).
- To comply with OpenID Connect standards.

### Syntax Rules and Structure

#### ID Token Claims

| Claim | Required | Description |
|-------|----------|-------------|
| `iss` | ✅ | Issuer (IdP URL) |
| `sub` | ✅ | Subject (user ID) |
| `aud` | ✅ | Audience (client ID) |
| `exp` | ✅ | Expiration time |
| `iat` | ✅ | Issued at |
| `nonce` | ✅ (if requested) | Replay protection |
| `auth_time` | Optional | When authentication occurred |
| `acr` | Optional | Authentication context class |
| `amr` | Optional | Authentication methods |
| `name` | Optional (`profile`) | Full name |
| `email` | Optional (`email`) | Email address |
| `picture` | Optional (`profile`) | Profile picture URL |

#### ID Token Validation (jose)

```typescript
import { jwtVerify, createRemoteJWKSet } from 'jose';

const JWKS = createRemoteJWKSet(new URL('https://auth.example.com/.well-known/jwks.json'));

async function validateIdToken(idToken: string, expectedNonce: string, clientId: string) {
  const { payload } = await jwtVerify(idToken, JWKS, {
    issuer: 'https://auth.example.com',
    audience: clientId,
    algorithms: ['RS256'],
    clockTolerance: 5,
  });

  // Verify nonce (replay protection)
  if (payload.nonce !== expectedNonce) {
    throw new Error('Invalid nonce');
  }

  // Verify auth_time (if max_age was requested)
  // if (payload.auth_time < Date.now() / 1000 - maxAge) throw ...

  return payload;
}
```

#### Using ID Token Claims

```typescript
async function handleCallback(code: string, state: string, nonce: string) {
  const tokenSet = await client.callback(redirectUri, { code, state }, { state, nonce });

  const claims = tokenSet.claims();
  // claims.sub  — user ID
  // claims.email — email (if 'email' scope)
  // claims.name — name (if 'profile' scope)

  // Create a session
  req.session.userId = claims.sub;
  req.session.email = claims.email;
  req.session.name = claims.name;
}
```

#### Syntax Rules

- **Validate the signature** using the IdP's JWKS.
- **Validate `iss`, `aud`, `exp`, `iat`** — all mandatory.
- **Validate `nonce`** — if you sent one.
- **Use the ID token only for the client** — never send it to resource servers.
- **Do not use the ID token for authorization** — use access tokens.
- **Request only necessary scopes** — `openid profile email`.
- **Store the ID token securely** — or derive a session from its claims.
- **Do not trust unvalidated claims** — always validate first.

#### Constraints and Limitations

- **ID tokens expire** — typically 1 hour or less.
- **ID tokens cannot be revoked** — use short TTLs.
- **ID tokens may be large** — profile claims increase size.
- **ID tokens are not for API access** — use access tokens.
- **Nonce validation is mandatory** — prevents replay.
- **`sub` is not always the email** — use `sub` as the canonical identifier.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: ID Token Validation and Session Creation (Express + jose)

```typescript
import express from 'express';
import session from 'express-session';
import { jwtVerify, createRemoteJWKSet } from 'jose';
import { generators, Issuer } from 'openid-client';

const app = express();
app.use(session({ secret: process.env.SESSION_SECRET!, resave: false, saveUninitialized: false }));

const issuer = await Issuer.discover('https://auth.example.com');
const client = new issuer.Client({
  client_id: process.env.CLIENT_ID!,
  client_secret: process.env.CLIENT_SECRET!,
  redirect_uris: ['https://app.example.com/callback'],
  response_types: ['code'],
});

const JWKS = createRemoteJWKSet(new URL('https://auth.example.com/.well-known/jwks.json'));

app.get('/login', (req, res) => {
  const codeVerifier = generators.codeVerifier();
  const codeChallenge = generators.codeChallenge(codeVerifier);
  const state = generators.state();
  const nonce = generators.nonce();

  req.session.oidc = { codeVerifier, state, nonce };

  const url = client.authorizationUrl({
    scope: 'openid profile email',
    state,
    nonce,
    code_challenge: codeChallenge,
    code_challenge_method: 'S256',
  });

  res.redirect(url);
});

app.get('/callback', async (req, res) => {
  const { codeVerifier, state, nonce } = req.session.oidc;

  const tokenSet = await client.callback(
    'https://app.example.com/callback',
    client.callbackParams(req),
    { state, nonce, code_verifier: codeVerifier },
  );

  // Validate ID token explicitly
  const { payload } = await jwtVerify(tokenSet.id_token!, JWKS, {
    issuer: 'https://auth.example.com',
    audience: process.env.CLIENT_ID!,
    algorithms: ['RS256'],
  });

  if (payload.nonce !== nonce) {
    return res.status(400).send('Invalid nonce');
  }

  // Create a local session from ID token claims
  req.session.user = {
    id: payload.sub,
    email: payload.email,
    name: payload.name,
    picture: payload.picture,
  };

  delete req.session.oidc;
  res.redirect('/profile');
});

app.get('/profile', (req, res) => {
  if (!req.session.user) return res.redirect('/login');
  res.json(req.session.user);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:** After authentication, the callback validates the ID token (signature, issuer, audience, nonce), extracts claims, and creates a local session. The session is used for subsequent requests.

**Why this works:** The ID token is validated using the IdP's JWKS. The nonce prevents replay. The claims are used to create a session — the ID token itself is not stored for API access.

### Real-World Cases

- **SSO:** ID tokens enable SSO across multiple applications.
- **Social login:** Google, GitHub, Apple, and Microsoft issue ID tokens.
- **Enterprise:** Okta, Azure AD, and Keycloak issue ID tokens with enterprise claims.
- **Consumer apps:** Profile information (name, email, picture) for personalisation.

---

## Core Concept 5: Refresh Tokens

### Definitions

**Core Definition:** A refresh token is a long-lived credential issued alongside the access token that allows the client to obtain new access tokens without requiring the user to re-authenticate.

**Technical Definition:** Refresh tokens (RFC 6749 §1.5) are issued to confidential clients (and to public clients with rotation) to enable long-term access. They are exchanged at the token endpoint using the `grant_type=refresh_token`. Refresh tokens should be stored securely (HttpOnly cookies for browsers, secure storage for mobile), rotated on each use, and bound to a client. OAuth 2.0 Security Best Current Practice (RFC 9700) mandates rotation for public clients and recommends it for confidential clients. Refresh tokens may be revoked via the revocation endpoint (RFC 7009). For offline access, the `offline_access` scope must be requested.

**Beginner-Friendly Explanation:** A refresh token is like a renewable library card. Instead of showing your ID every time you want a book (re-authenticating), you show your library card (refresh token) to get a new borrowing slip (access token). The library card lasts longer but can be cancelled if lost. If someone steals it, the library will notice when both you and the thief try to use it — and cancel all your cards.

### Purposes

- To obtain new access tokens without user interaction.
- To support long-running sessions with short-lived access tokens.
- To enable offline access to user data (with `offline_access` scope).
- To reduce friction for users (no repeated logins).
- To support mobile and SPA applications.

### Syntax Rules and Structure

#### Refresh Token Request

```typescript
async function refreshAccessToken(refreshToken: string) {
  const response = await fetch('https://auth.example.com/oauth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'refresh_token',
      refresh_token: refreshToken,
      client_id: process.env.CLIENT_ID!,
      client_secret: process.env.CLIENT_SECRET!, // For confidential clients
    }),
  });

  return response.json();
}
```

#### Refresh Token Rotation

```
1. Client sends refresh token A to /oauth/token
2. Server validates A, revokes A, issues B + new access token
3. Client stores B
4. If A is used again, server detects reuse and revokes the entire family
5. Client must re-authenticate
```

#### Revocation

```typescript
async function revokeRefreshToken(refreshToken: string) {
  await fetch('https://auth.example.com/oauth/revoke', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      Authorization: `Basic ${Buffer.from(`${clientId}:${clientSecret}`).toString('base64')}`,
    },
    body: new URLSearchParams({
      token: refreshToken,
      token_type_hint: 'refresh_token',
    }),
  });
}
```

#### Syntax Rules

- **Request `offline_access` scope** — for refresh tokens.
- **Store refresh tokens securely** — HttpOnly cookies (browser), Keychain/Keystore (mobile).
- **Rotate on every use** — revoke the old token, issue a new one.
- **Detect reuse** — revoke the entire family on replay.
- **Bind to the client** — refresh tokens should be client-specific.
- **Set a reasonable TTL** — 7–30 days.
- **Revoke on logout and password change.**
- **Never send refresh tokens to resource servers** — only to the token endpoint.
- **Use HTTPS** — always.

#### Constraints and Limitations

- **Refresh tokens are long-lived** — theft has a larger impact.
- **Rotation requires server-side storage** — not stateless.
- **Reuse detection can cause false positives** — network retries.
- **Confidential clients may not need rotation** — but it's recommended.
- **Revocation endpoint must be supported by the IdP** — check the discovery document.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Refresh Token with Rotation (Express)

```typescript
// server.ts
app.get('/callback', async (req, res) => {
  const tokenSet = await client.callback(
    redirectUri,
    client.callbackParams(req),
    { state, nonce, code_verifier: codeVerifier },
  );

  // Store tokens
  req.session.accessToken = tokenSet.access_token;
  req.session.refreshToken = tokenSet.refresh_token;
  req.session.expiresAt = Date.now() + (tokenSet.expires_in ?? 3600) * 1000;

  res.redirect('/dashboard');
});

// Middleware: refresh access token if expired
async function ensureValidToken(req, res, next) {
  if (!req.session.refreshToken) return res.redirect('/login');

  if (Date.now() < req.session.expiresAt - 60000) {
    return next(); // Still valid
  }

  try {
    const tokenSet = await client.refresh(req.session.refreshToken);

    req.session.accessToken = tokenSet.access_token;
    req.session.refreshToken = tokenSet.refresh_token ?? req.session.refreshToken;
    req.session.expiresAt = Date.now() + (tokenSet.expires_in ?? 3600) * 1000;

    next();
  } catch (err) {
    // Refresh failed — user must re-authenticate
    req.session.destroy(() => res.redirect('/login'));
  }
}

app.get('/dashboard', ensureValidToken, (req, res) => {
  res.json({ message: 'Dashboard', accessToken: req.session.accessToken });
});
```

**Expected behaviour:** When the access token expires, the middleware uses the refresh token to obtain a new one. If the refresh token is rotated, the new one is stored. If refresh fails, the user is redirected to login.

**Why this works:** Refresh tokens enable long-term access without re-authentication. Rotation (if supported by the IdP) limits theft impact. The middleware transparently refreshes tokens before they expire.

### Real-World Cases

- **SPAs:** Refresh tokens in HttpOnly cookies, access tokens in memory.
- **Mobile apps:** Refresh tokens in Keychain/Keystore.
- **Offline access:** `offline_access` scope for background data sync.
- **Enterprise:** Refresh tokens for long-running sessions with short access TTLs.

---

## Core Concept 6: Third-Party Authentication (Social Login)

### Definitions

**Core Definition:** Third-party authentication (social login) allows users to authenticate using an external identity provider (Google, GitHub, Apple, Microsoft) via OAuth 2.0 / OpenID Connect.

**Technical Definition:** Social login is implemented using OIDC (for Google, Apple, Microsoft) or OAuth 2.0 with a custom user info endpoint (for GitHub). Each provider has its own authorization endpoint, token endpoint, JWKS, and user info endpoint. The client registers an application with the provider, obtains a client ID and secret, and configures redirect URIs. The OIDC flow is standard (Authorization Code with PKCE), but provider-specific details (scopes, claims, PKCE support) vary. Account linking is required: the provider's `sub` (or `id`) must be linked to the internal user ID. Multiple social providers should be supported, with a fallback to email/password.

**Beginner-Friendly Explanation:** Social login is like using your Google account as a universal key. Instead of creating a new username and password for every app, you tell the app "Ask Google who I am." Google verifies you and tells the app "This is Alice." The app trusts Google, so it lets you in. You can use the same Google account for dozens of apps without creating new credentials.

### Purposes

- To reduce friction during registration and login.
- To increase conversion rates (fewer abandoned signups).
- To leverage the provider's security (MFA, breach detection).
- To obtain verified email addresses and profile information.
- To enable account linking across providers.

### Syntax Rules and Structure

#### Provider Comparison

| Provider | Protocol | Scopes | Notable Claims |
|----------|----------|--------|----------------|
| **Google** | OIDC | `openid`, `profile`, `email` | `sub`, `email`, `name`, `picture` |
| **GitHub** | OAuth 2.0 | `read:user`, `user:email` | `id`, `login`, `email`, `avatar_url` |
| **Apple** | OIDC | `name`, `email` | `sub`, `email` (private relay) |
| **Microsoft** | OIDC | `openid`, `profile`, `email` | `sub`, `email`, `name`, `picture` |

#### Google OIDC Configuration

```typescript
const googleIssuer = await Issuer.discover('https://accounts.google.com');
const googleClient = new googleIssuer.Client({
  client_id: process.env.GOOGLE_CLIENT_ID!,
  client_secret: process.env.GOOGLE_CLIENT_SECRET!,
  redirect_uris: ['https://app.example.com/auth/google/callback'],
  response_types: ['code'],
});
```

#### GitHub OAuth 2.0 Configuration

```typescript
const githubConfig = {
  authorizationEndpoint: 'https://github.com/login/oauth/authorize',
  tokenEndpoint: 'https://github.com/login/oauth/access_token',
  userInfoEndpoint: 'https://api.github.com/user',
  clientId: process.env.GITHUB_CLIENT_ID!,
  clientSecret: process.env.GITHUB_CLIENT_SECRET!,
  redirectUri: 'https://app.example.com/auth/github/callback',
  scope: 'read:user user:email',
};

async function fetchGitHubUser(accessToken: string) {
  const response = await fetch(githubConfig.userInfoEndpoint, {
    headers: { Authorization: `Bearer ${accessToken}`, 'User-Agent': 'MyApp' },
  });
  return response.json();
}
```

#### Account Linking

```typescript
async function linkOrCreateUser(provider: string, providerId: string, profile: any) {
  // Check for existing link
  const existing = await prisma.oAuthAccount.findUnique({
    where: { provider_providerId: { provider, providerId } },
    include: { user: true },
  });
  if (existing) return existing.user;

  // Check for existing user with same email
  const email = profile.email?.toLowerCase();
  if (email) {
    const userByEmail = await prisma.user.findUnique({ where: { email } });
    if (userByEmail) {
      await prisma.oAuthAccount.create({
        data: { userId: userByEmail.id, provider, providerId },
      });
      return userByEmail;
    }
  }

  // Create new user
  const user = await prisma.user.create({
    data: {
      email,
      name: profile.name,
      emailVerified: true,
      oauthAccounts: {
        create: { provider, providerId },
      },
    },
  });
  return user;
}
```

#### Syntax Rules

- **Use OIDC for providers that support it** (Google, Apple, Microsoft).
- **Use OAuth 2.0 + user info endpoint** for providers without OIDC (GitHub).
- **Always use PKCE** — even for confidential clients.
- **Validate the ID token** — signature, `iss`, `aud`, `exp`, `nonce`.
- **Link accounts by `sub` (or `id`)** — never by email alone (emails can change).
- **Handle Apple's private relay email** — users may hide their real email.
- **Support multiple providers** — let users choose.
- **Provide a fallback** — email/password for users without social accounts.
- **Handle account linking conflicts** — if the email already exists, link or ask the user.

#### Constraints and Limitations

- **Provider-specific quirks** — Apple's name is only sent on first login; GitHub's email requires a separate API call.
- **Email may not be verified** — some providers do not verify email ownership.
- **Account linking complexity** — users may have multiple accounts with different providers.
- **Provider outages block logins** — provide a fallback.
- **Privacy concerns** — users may not want to share data with the provider.
- **Apple's private relay** — `@privaterelay.appleid.com` emails require special handling.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Provider Social Login (Express + Passport.js)

```typescript
import express from 'express';
import passport from 'passport';
import { Strategy as GoogleStrategy } from 'passport-google-oauth20';
import { Strategy as GitHubStrategy } from 'passport-github2';
import session from 'express-session';

const app = express();
app.use(session({ secret: process.env.SESSION_SECRET!, resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

passport.serializeUser((user: any, done) => done(null, user.id));
passport.deserializeUser(async (id: string, done) => {
  const user = await prisma.user.findUnique({ where: { id } });
  done(null, user);
});

// Google
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID!,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
  callbackURL: 'https://app.example.com/auth/google/callback',
}, async (accessToken, refreshToken, profile, done) => {
  try {
    const user = await linkOrCreateUser('google', profile.id, {
      email: profile.emails?.[0]?.value,
      name: profile.displayName,
      picture: profile.photos?.[0]?.value,
    });
    done(null, user);
  } catch (err) {
    done(err);
  }
}));

// GitHub
passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID!,
  clientSecret: process.env.GITHUB_CLIENT_SECRET!,
  callbackURL: 'https://app.example.com/auth/github/callback',
  scope: ['read:user', 'user:email'],
}, async (accessToken, refreshToken, profile, done) => {
  try {
    // Fetch primary email (GitHub requires a separate call)
    const emailResponse = await fetch('https://api.github.com/user/emails', {
      headers: { Authorization: `Bearer ${accessToken}`, 'User-Agent': 'MyApp' },
    });
    const emails = await emailResponse.json();
    const primaryEmail = emails.find((e: any) => e.primary)?.email;

    const user = await linkOrCreateUser('github', profile.id, {
      email: primaryEmail,
      name: profile.displayName ?? profile.username,
      picture: profile.photos?.[0]?.value,
    });
    done(null, user);
  } catch (err) {
    done(err);
  }
}));

// Routes
app.get('/auth/google', passport.authenticate('google', { scope: ['openid', 'profile', 'email'] }));
app.get('/auth/google/callback', passport.authenticate('google', { failureRedirect: '/login' }), (req, res) => {
  res.redirect('/profile');
});

app.get('/auth/github', passport.authenticate('github'));
app.get('/auth/github/callback', passport.authenticate('github', { failureRedirect: '/login' }), (req, res) => {
  res.redirect('/profile');
});

app.get('/profile', (req, res) => {
  if (!req.user) return res.redirect('/login');
  res.json(req.user);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:** Users can log in with Google or GitHub. Accounts are linked by provider ID (or email if the user already exists). The session is established after successful authentication.

**Why this works:** Passport.js abstracts provider-specific details. Account linking ensures that users with the same email are unified. The session is established after successful authentication.

### Real-World Cases

- **Consumer apps:** Google and Apple for frictionless signup.
- **Developer tools:** GitHub for developer-friendly login.
- **Enterprise:** Microsoft for Microsoft 365 integration.
- **E-commerce:** Google, Facebook, and Apple for checkout.
- **SaaS:** Multiple providers to accommodate user preferences.

---

## Core Concept 7: Scopes and Consent Management

### Definitions

**Core Definition:** Scopes are space-separated strings that define the specific permissions the client is requesting, and consent is the user's explicit approval of those permissions.

**Technical Definition:** Scopes (RFC 6749 §3.3) are requested by the client during authorization and granted by the authorization server after user consent. OIDC defines standard scopes: `openid` (required), `profile`, `email`, `address`, `phone`, and `offline_access`. Custom scopes define API-specific permissions (`read:posts`, `write:posts`). Consent is managed by the authorization server: the user is shown the requested scopes and must approve them. Consent may be remembered (to avoid repeated prompts) and revoked (via the IdP's dashboard or a revocation API). The `scope` claim in the access token reflects the granted scopes.

**Beginner-Friendly Explanation:** Scopes are like the checkboxes on a permission form. When you sign in with Google, the app says "I want to see your name, email, and profile picture." You check the boxes (consent) and click "Allow." The app gets a token that only allows those things. If you later change your mind, you can revoke access from your Google account settings. The app's token stops working for those scopes.

### Purposes

- To define granular permissions for API access.
- To obtain explicit user consent for data access.
- To comply with privacy regulations (GDPR, CCPA).
- To limit the blast radius of token theft (least privilege).
- To enable users to revoke access to their data.
- To support incremental authorization (requesting scopes as needed).

### Syntax Rules and Structure

#### Standard Scopes

| Scope | Purpose | Claims |
|-------|---------|--------|
| `openid` | OIDC authentication | `sub` |
| `profile` | Profile information | `name`, `picture`, `locale` |
| `email` | Email address | `email`, `email_verified` |
| `address` | Postal address | `address` |
| `phone` | Phone number | `phone_number`, `phone_number_verified` |
| `offline_access` | Refresh token | — |

#### Custom API Scopes

| Scope | Permission |
|-------|------------|
| `read:posts` | Read posts |
| `write:posts` | Create/update posts |
| `delete:posts` | Delete posts |
| `read:users` | Read user data |
| `admin` | Full admin access |

#### Consent Screen (Authorization URL)

```
https://auth.example.com/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://app.example.com/callback&
  scope=openid%20profile%20email%20read:posts&
  state=RANDOM&
  code_challenge=PKCE_CHALLENGE&
  code_challenge_method=S256
```

#### Scope Validation (Resource Server)

```typescript
function requireScope(scope: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const granted = (req.user?.scope ?? '').split(' ');
    if (!granted.includes(scope)) {
      return res.status(403).json({
        error: 'insufficient_scope',
        error_description: `Required scope: ${scope}`,
      });
    }
    next();
  };
}

app.get('/posts', requireAuth, requireScope('read:posts'), (req, res) => {
  res.json({ posts: [] });
});
```

#### Consent Revocation (IdP Dashboard)

```
User → Google Account → Security → Third-party apps with account access
→ Select app → Remove access
→ App's tokens are revoked
```

#### Syntax Rules

- **Request only necessary scopes** — least privilege.
- **Use standard OIDC scopes** for identity (`openid profile email`).
- **Define custom scopes** for API-specific permissions (`read:posts`).
- **Document every scope** — its purpose and implications.
- **Request incremental consent** — request scopes as needed.
- **Validate scopes at the resource server** — never trust the client.
- **Support scope downgrade** — grant only a subset of requested scopes.
- **Provide a consent screen** — show what's being requested.
- **Enable revocation** — users must be able to revoke access.
- **Log consent** — for audit and compliance.

#### Constraints and Limitations

- **Scope explosion:** Too many scopes become unmanageable.
- **Consent fatigue:** Users may click "Allow" without reading.
- **Revocation is not instantaneous:** Tokens may remain valid until expiry.
- **Scope granularity varies:** Some IdPs have coarse scopes.
- **No standard for custom scopes:** Each API defines its own.
- **Incremental consent is not always supported:** Some IdPs require all scopes upfront.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Scope-Based Access Control (Express + OAuth 2.0)

```typescript
// server.ts
import express from 'express';
import { jwtVerify, createRemoteJWKSet } from 'jose';

const app = express();
const JWKS = createRemoteJWKSet(new URL('https://auth.example.com/.well-known/jwks.json'));

async function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) return res.status(401).json({ error: 'Missing token' });

  try {
    const { payload } = await jwtVerify(header.slice(7), JWKS, {
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
      algorithms: ['RS256'],
    });
    req.user = {
      id: payload.sub,
      scopes: (payload.scope ?? '').split(' '),
    };
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

function requireScope(...required: string[]) {
  return (req, res, next) => {
    const granted = req.user.scopes;
    const hasAll = required.every((s) => granted.includes(s));
    if (!hasAll) {
      return res.status(403).json({
        error: 'insufficient_scope',
        required,
        granted,
      });
    }
    next();
  };
}

// Public endpoint — no scope required
app.get('/health', (req, res) => res.json({ status: 'ok' }));

// Read-only endpoint
app.get('/posts', requireAuth, requireScope('read:posts'), (req, res) => {
  res.json({ posts: [] });
});

// Write endpoint
app.post('/posts', requireAuth, requireScope('write:posts'), (req, res) => {
  res.status(201).json({ id: 'post-1' });
});

// Admin endpoint
app.delete('/posts/:id', requireAuth, requireScope('delete:posts', 'admin'), (req, res) => {
  res.status(204).end();
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `/health` is public.
- `/posts` (GET) requires `read:posts`.
- `/posts` (POST) requires `write:posts`.
- `/posts/:id` (DELETE) requires both `delete:posts` and `admin`.
- Requests with insufficient scopes return `403` with `insufficient_scope`.

**Why this works:** Scopes are validated at the resource server. The `requireScope` middleware checks the granted scopes against the required scopes. The error response follows the OAuth 2.0 Bearer Token Usage spec (RFC 6750).

### Real-World Cases

- **Google APIs:** Scopes like `https://www.googleapis.com/auth/gmail.readonly`.
- **GitHub:** Scopes like `repo`, `read:user`, `user:email`.
- **Stripe:** Scopes like `read_write`, `read_only`.
- **Slack:** Scopes like `channels:read`, `chat:write`.
- **Enterprise:** Custom scopes for internal APIs (`read:employees`, `write:payroll`).

---

## References

- RFC 6749 — The OAuth 2.0 Authorization Framework — https://www.rfc-editor.org/rfc/rfc6749
- RFC 6750 — The OAuth 2.0 Authorization Framework: Bearer Token Usage — https://www.rfc-editor.org/rfc/rfc6750
- RFC 7636 — Proof Key for Code Exchange (PKCE) — https://www.rfc-editor.org/rfc/rfc7636
- RFC 8628 — OAuth 2.0 Device Authorization Grant — https://www.rfc-editor.org/rfc/rfc8628
- RFC 7009 — OAuth 2.0 Token Revocation — https://www.rfc-editor.org/rfc/rfc7009
- RFC 7662 — OAuth 2.0 Token Introspection — https://www.rfc-editor.org/rfc/rfc7662
- RFC 9068 — JWT Profile for OAuth 2.0 Access Tokens — https://www.rfc-editor.org/rfc/rfc9068
- RFC 9700 — Best Current Practice for OAuth 2.0 Security — https://www.rfc-editor.org/rfc/rfc9700
- OpenID Connect Core 1.0 — https://openid.net/specs/openid-connect-core-1_0.html
- OpenID Connect Discovery 1.0 — https://openid.net/specs/openid-connect-discovery-1_0.html
- OpenID Connect Session Management 1.0 — https://openid.net/specs/openid-connect-session-1_0.html
- OAuth 2.0 Security Best Current Practice — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics
- OAuth 2.0 for Browser-Based Apps — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps
- Auth0 Documentation — https://auth0.com/docs
- Okta Documentation — https://developer.okta.com/docs/
- Keycloak Documentation — https://www.keycloak.org/documentation
- AWS Cognito Documentation — https://docs.aws.amazon.com/cognito/
- openid-client — npm package — https://www.npmjs.com/package/openid-client
- Passport.js Documentation — https://www.passportjs.org/docs/
- jose — npm package — https://www.npmjs.com/package/jose
- Google Identity — OIDC Documentation — https://developers.google.com/identity/openid-connect/openid-connect
- GitHub OAuth Documentation — https://docs.github.com/en/apps/oauth-apps
- Sign in with Apple — https://developer.apple.com/sign-in-with-apple/
- Microsoft Identity Platform — https://learn.microsoft.com/en-us/entra/identity-platform/
- OWASP Cheat Sheet Series — OAuth 2.0 — https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html
- OWASP — OAuth 2.0 Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html