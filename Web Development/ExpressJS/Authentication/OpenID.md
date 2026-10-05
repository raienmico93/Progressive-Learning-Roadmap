# OpenID Connect (OIDC) & Identity — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** OpenID Connect 1.0 (OIDC) is a simple identity layer built on top of the OAuth 2.0 protocol that enables clients to verify the identity of an end-user based on authentication performed by an authorization server, and to obtain basic profile information about the end-user in an interoperable, REST-like manner. 

**Technical Definition:** OIDC is an identity framework that provides authentication, authorization, and attribute transmission capability. It introduces the ID Token — a signed JWT containing claims about the authentication event and the end-user — and standardizes the UserInfo endpoint for retrieving additional identity claims. OIDC defines a discovery mechanism (`/.well-known/openid-configuration`) that allows clients to automatically configure themselves against an OpenID Provider. 

**Beginner-Friendly Explanation:** OIDC is like a standardized ID card system for the internet. When you log into a website using "Sign in with Google" or "Login with Microsoft," OIDC is the protocol working behind the scenes. It lets the website verify who you are without ever seeing your password, and it can fetch basic profile information about you (name, email, picture) in a consistent, secure format.

### Key Characteristics

- **Identity layer on OAuth 2.0:** OIDC adds authentication on top of OAuth 2.0's authorization framework. 
- **ID Token:** A signed JWT that contains claims about the authentication event and the user. 
- **Discovery:** Standardized metadata endpoint for automatic client configuration. 
- **UserInfo endpoint:** REST-like endpoint for retrieving additional identity claims. 
- **Claims-based:** Identity information is transmitted as claims (name-value pairs). 
- **Interoperable:** Works across web, mobile, and JavaScript clients. 
- **Cryptographically verifiable:** ID tokens are signed with the provider's private key and verified using public keys from the JWKS endpoint. 

### Prerequisites

- **Understanding of OAuth 2.0:** Authorization flows, tokens, and scopes.
- **Basic knowledge of JWTs:** Structure, signing, and verification.
- **Familiarity with HTTP:** Headers, status codes, and REST concepts.
- **Understanding of public-key cryptography:** RSA, SHA-256, and digital signatures.
- **Node.js environment** for code examples.

### Related Programming Areas

- **Authentication:** OIDC is the modern standard for federated identity.
- **Authorization:** OIDC extends OAuth 2.0's authorization with identity.
- **Single Sign-On (SSO):** OIDC enables SSO across multiple applications.
- **API security:** ID tokens and access tokens secure API access.
- **Identity Providers:** Google, Microsoft, Okta, Auth0, Keycloak all support OIDC.
- **Session management:** OIDC defines session synchronization mechanisms.

### Core Concepts

1. **Identity Providers (IdPs)** — Discovery endpoints and provider roles.
2. **ID Tokens** — JWT architecture, claims, and cryptographic verification.
3. **User Identity & UserInfo** — Profiling, claims mapping, and session syncing.

---

## Core Concept 1: Identity Providers (IdPs)

### Definitions

**Core Definition:** An Identity Provider (IdP) in OIDC is called an OpenID Provider (OP) — a service that authenticates users and issues ID tokens containing identity claims to relying parties.

**Technical Definition:** An OpenID Provider (OP) is an OAuth 2.0 authorization server that implements OIDC. The OP is responsible for managing users and their identities, issuing tokens, handling user administration, authenticating the user, vouching for the user's identity with the relying party, and revoking authenticated sessions and tokens. The OP publishes its configuration metadata at a well-known discovery endpoint. 

**Beginner-Friendly Explanation:** An Identity Provider is like a passport office. It verifies who you are and issues a document (the ID token) that other parties (websites and apps) can trust. Just as a passport office has a publicly known address and publishes information about how to verify its passports, an OIDC Identity Provider publishes its configuration at a standard discovery endpoint so clients can automatically configure themselves.

### Purposes

- To provide a trusted source of identity verification for federated authentication.
- To issue ID tokens containing verified identity claims.
- To publish configuration metadata for automatic client discovery.
- To manage user identities, sessions, and token lifecycle.

### Sub-Feature 1.1: Discovery Endpoints

#### Definitions

**Core Definition:** The OIDC discovery endpoint is a standardized URL (`.well-known/openid-configuration`) where an OpenID Provider publishes its configuration metadata in JSON format.

**Technical Definition:** OpenID Providers have metadata describing their configuration. The endpoint is located at `/.well-known/openid-configuration` relative to the issuer identifier after stripping any trailing slash. The metadata is formatted in JSON and includes endpoints, supported scopes, signing algorithms, and other configuration details. 

**Beginner-Friendly Explanation:** The discovery endpoint is like a business card that every OIDC provider hands out. It tells clients exactly where to find the authorization endpoint, the token endpoint, the user info endpoint, and the public keys for verifying tokens — all in a predictable location.

#### Syntax Rules and Structure

**Discovery URL Construction:**
```
{issuer}/.well-known/openid-configuration
```

| Component | Breakdown |
|-----------|-----------|
| `issuer` | The OP's issuer identifier (HTTPS URL, case-sensitive). |
| `/.well-known/openid-configuration` | Standard path appended to the issuer. |

**Key Discovery Metadata Fields:**
| Field | Required | Description |
|-------|----------|-------------|
| `issuer` | Yes | The OP's issuer identifier. |
| `authorization_endpoint` | Yes | URL for authorization requests. |
| `token_endpoint` | Yes | URL for token requests. |
| `userinfo_endpoint` | Recommended | URL for UserInfo requests. |
| `jwks_uri` | Yes | URL for the JSON Web Key Set. |
| `scopes_supported` | Recommended | Available scopes. |
| `response_types_supported` | Yes | Supported response types. |
| `id_token_signing_alg_values_supported` | Yes | Supported signing algorithms. |
| `claims_supported` | Recommended | Available claims. |

#### Constraints and Limitations

- The discovery endpoint must be served over HTTPS.
- The issuer identifier must match the `iss` claim in issued tokens.
- Not all providers implement every recommended field.
- The metadata document may be cached, but should be refreshed periodically.

#### Annotated Code Example

```js
// discovery-endpoint.js
const express = require('express');
const app = express();

// Simulated discovery endpoint
app.get('/.well-known/openid-configuration', (req, res) => {
  res.json({
    issuer: 'https://auth.example.com',
    authorization_endpoint: 'https://auth.example.com/authorize',
    token_endpoint: 'https://auth.example.com/token',
    userinfo_endpoint: 'https://auth.example.com/userinfo',
    jwks_uri: 'https://auth.example.com/.well-known/jwks.json',
    scopes_supported: ['openid', 'profile', 'email', 'offline_access'],
    response_types_supported: ['code', 'code id_token'],
    id_token_signing_alg_values_supported: ['RS256', 'ES256'],
    claims_supported: ['sub', 'iss', 'auth_time', 'name', 'email', 'picture'],
    end_session_endpoint: 'https://auth.example.com/logout'
  });
});

app.listen(3000, () => console.log('Discovery endpoint on 3000'));
```

**Expected Output (for `GET /.well-known/openid-configuration`):**
```json
{
  "issuer": "https://auth.example.com",
  "authorization_endpoint": "https://auth.example.com/authorize",
  "token_endpoint": "https://auth.example.com/token",
  "userinfo_endpoint": "https://auth.example.com/userinfo",
  "jwks_uri": "https://auth.example.com/.well-known/jwks.json",
  "scopes_supported": ["openid", "profile", "email", "offline_access"],
  "response_types_supported": ["code", "code id_token"],
  "id_token_signing_alg_values_supported": ["RS256", "ES256"],
  "claims_supported": ["sub", "iss", "auth_time", "name", "email", "picture"],
  "end_session_endpoint": "https://auth.example.com/logout"
}
```

**Why this output:** The discovery endpoint returns a JSON document containing all the metadata a client needs to configure itself: where to send authorization requests, where to exchange codes for tokens, where to verify token signatures, and what scopes and claims are available. Clients can fetch this document once and use it to drive their OIDC integration.

#### Real-World Cases

- **Google OIDC Discovery:** `https://accounts.google.com/.well-known/openid-configuration`
- **Microsoft Entra ID:** `https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration`
- **Auth0:** `https://{domain}/.well-known/openid-configuration`
- **Keycloak:** `https://{host}/realms/{realm}/.well-known/openid-configuration`

---

### Sub-Feature 1.2: OIDC Roles and Actors

#### Definitions

**Core Definition:** OIDC defines several actors: the OpenID Provider (OP), the Relying Party (RP), the End-User, and the User Agent.

**Technical Definition:** The OIDC actors include: the **End-User** (the human who authenticates), the **Relying Party (RP)** or **Client** (the application requesting authentication), the **OpenID Provider (OP)** (the authorization server that authenticates the user), and the **User Agent** (the browser or app through which the user interacts). The RP validates tokens issued by the OP and manages locally relevant user attributes. 

**Beginner-Friendly Explanation:** Think of a hotel check-in. You (End-User) walk up to the front desk (OP) with your ID. The front desk verifies your identity and gives you a key card (ID token). You then use that key card to access your room (the RP's services). The RP trusts the front desk's verification and gives you access based on the claims in your key card.

#### Purposes

- To define the trust relationships between parties in federated authentication.
- To clarify which party is responsible for which aspect of authentication.
- To establish the basis for token issuance and validation.

#### Syntax Rules and Structure

| Actor | Role | Responsibilities |
|-------|------|-----------------|
| End-User | Resource Owner | Authenticates with the OP. |
| Relying Party (RP) | Client | Validates tokens, manages local user attributes. |
| OpenID Provider (OP) | Authorization Server | Authenticates users, issues tokens, vouches for identity. |
| User Agent | Browser/App | Handles redirects, may store cookies/session info. |

#### Annotated Code Example

```js
// oidc-roles.js
// Simulated RP-side token validation flow

const express = require('express');
const app = express();

// RP: Relying Party server
app.get('/callback', (req, res) => {
  const { code, state } = req.query;

  // RP validates state (CSRF protection)
  if (state !== req.session?.oauthState) {
    return res.status(400).json({ error: 'Invalid state' });
  }

  // RP exchanges code for tokens at the OP
  // RP validates ID token signature using OP's JWKS
  // RP extracts claims from the ID token

  res.json({
    message: 'User authenticated',
    actor: 'Relying Party',
    validated: {
      state: true,
      code_received: !!code
    }
  });
});

app.listen(3000, () => console.log('RP server on 3000'));
```

**Expected Output:**
```json
{
  "message": "User authenticated",
  "actor": "Relying Party",
  "validated": { "state": true, "code_received": true }
}
```

**Why this output:** The RP receives the authorization code from the OP (via the user agent), validates the state parameter, and then exchanges the code for tokens. The RP is the "validating party" — it trusts the OP's authentication and uses the ID token claims to establish the user's identity locally.

#### Real-World Cases

- **"Sign in with Google":** Google is the OP; your application is the RP.
- **Enterprise SSO:** Microsoft Entra ID is the OP; internal apps are RPs.
- **Social login:** Facebook, GitHub, or Apple as OPs for third-party apps.

---

## Core Concept 2: ID Tokens

### Definitions

**Core Definition:** An ID Token is a signed JSON Web Token (JWT) that contains claims about the authentication event and the authenticated end-user, issued by the OpenID Provider to the Relying Party.

**Technical Definition:** The ID Token is a JWT that MUST contain the following claims: `iss` (issuer), `sub` (subject), `aud` (audience), `exp` (expiration), and `iat` (issued at). It MAY contain additional claims such as `auth_time`, `nonce`, `acr`, `amr`, `azp`, and standard profile claims (name, email, picture). The ID Token is signed by the OP using a private key; the RP verifies the signature using the OP's public key from the JWKS endpoint. 

**Beginner-Friendly Explanation:** An ID Token is like a digitally signed ID card. It contains information about who you are (your name, email, etc.) and when you logged in. The signature ensures that nobody can tamper with the card — if someone changes your name on the card, the signature won't match, and the RP will know the card is forged.

### Purposes

- To convey the result of authentication to the Relying Party.
- To provide verified identity claims about the end-user.
- To enable the RP to establish a local session for the user.
- To serve as the primary artifact of OIDC authentication.

### Sub-Feature 2.1: JWT Architecture

#### Definitions

**Core Definition:** The ID Token is structured as a JWT — a compact, URL-safe string with three Base64URL-encoded parts: header, payload, and signature.

**Technical Definition:** A JWT consists of three parts separated by dots: the header (containing the signing algorithm and key ID), the payload (containing the claims), and the signature (created by signing the header and payload with the OP's private key). The header and payload are Base64URL-encoded JSON objects. The signature is computed over the string `header.payload` using the algorithm specified in the header. 

**Beginner-Friendly Explanation:** A JWT is like a sealed envelope with three sections. The first section (header) says "this envelope was sealed using this method." The second section (payload) contains the actual message — who you are and when you logged in. The third section (signature) is the wax seal that proves nobody tampered with the envelope.

#### Syntax Rules and Structure

**JWT Structure:**
```
header.payload.signature
```

| Part | Content | Encoding |
|------|---------|----------|
| Header | Algorithm, key ID | Base64URL |
| Payload | Claims (iss, sub, aud, exp, iat, etc.) | Base64URL |
| Signature | Cryptographic signature | Base64URL |

**Standard ID Token Header:**
```json
{
  "alg": "RS256",
  "kid": "a1b2c3d4e5f6g7h8i9j0",
  "typ": "JWT"
}
```

| Field | Description |
|-------|-------------|
| `alg` | Signing algorithm (RS256, ES256, etc.). |
| `kid` | Key ID — identifies which key signed the token. |
| `typ` | Token type (JWT). |

#### Constraints and Limitations

- The `alg` field must be validated against the expected algorithm; never trust the token's declared algorithm without verification. 
- The `kid` is a lookup hint, not an instruction — the RP must verify that the key comes from the trusted JWKS endpoint. 
- ID tokens MUST be signed; unsigned tokens are not acceptable.

### Sub-Feature 2.2: Standard Claims

#### Definitions

**Core Definition:** Standard claims are the registered JWT claim names defined by the JWT, OIDC, and OAuth specifications that carry specific, well-understood semantics.

**Technical Definition:** The ID Token contains a set of required and optional claims. Required claims include `iss`, `sub`, `aud`, `exp`, and `iat`. Optional claims include `auth_time`, `nonce`, `acr`, `amr`, `azp`, and profile claims defined by OIDC Core (name, family_name, given_name, email, picture, etc.). 

**Beginner-Friendly Explanation:** Standard claims are like the standard fields on a driver's license: name, date of birth, license number. Everyone understands what these fields mean, so different systems can read them consistently. Custom claims are like extra fields your particular state adds — they might be useful, but they're not universal.

#### Syntax Rules and Structure

| Claim | Full Name | Required | Description |
|-------|-----------|----------|-------------|
| `iss` | Issuer | Yes | OP's issuer identifier. |
| `sub` | Subject | Yes | Unique user identifier (never reassigned). |
| `aud` | Audience | Yes | Client ID(s) the token is intended for. |
| `exp` | Expiration | Yes | Time after which the token is invalid. |
| `iat` | Issued At | Yes | Time when the token was issued. |
| `auth_time` | Authentication Time | No | When the user actually authenticated. |
| `nonce` | Nonce | No | Value to prevent replay attacks. |
| `acr` | Authentication Context Class | No | Assurance level of authentication. |
| `amr` | Authentication Methods | No | Methods used for authentication. |
| `azp` | Authorized Party | No | Client that requested the token. |
| `name` | Full Name | No | User's full name. |
| `email` | Email | No | User's email address. |
| `email_verified` | Email Verified | No | Whether email is verified. |
| `picture` | Picture | No | URL to user's profile picture. |

**Custom Claims:**
Custom claims are additional claims defined by the OP that are not part of the OIDC standard. They are used to convey application-specific or domain-specific identity information. 

#### Syntax Rules

- Claims in the ID Token MUST be validated by the RP before use.
- `sub` MUST be locally unique and never reassigned within the issuer. 
- `aud` MUST contain the RP's `client_id`. 
- `exp` MUST be checked; expired tokens MUST be rejected. 
- `nonce` MUST be verified if it was included in the authentication request. 

#### Constraints and Limitations

- ID tokens are not suitable for authorization decisions; they convey authentication, not permission.
- Custom claims must be documented and agreed upon between the OP and RP.
- Claim values are not encrypted by default; sensitive data should be transmitted via other means.

#### Annotated Code Example

```js
// id-token-structure.js
const jwt = require('jsonwebtoken');

// Simulated ID Token (in production, this comes from the OP)
const idToken = jwt.sign(
  {
    iss: 'https://auth.example.com',
    sub: 'user-12345',
    aud: 'my-client-id',
    exp: Math.floor(Date.now() / 1000) + 3600,
    iat: Math.floor(Date.now() / 1000),
    auth_time: Math.floor(Date.now() / 1000) - 10,
    name: 'Alice Johnson',
    email: 'alice@example.com',
    email_verified: true,
    picture: 'https://example.com/alice.jpg',
    'https://example.com/roles': ['admin', 'user']  // Custom claim
  },
  'private-key-placeholder',
  { algorithm: 'RS256', keyid: 'key-1' }
);

// Decode and inspect (without verification)
const decoded = jwt.decode(idToken, { complete: true });
console.log('Header:', JSON.stringify(decoded.header, null, 2));
console.log('Payload:', JSON.stringify(decoded.payload, null, 2));

module.exports = { idToken };
```

**Expected Output:**
```
Header: {
  "alg": "RS256",
  "kid": "key-1",
  "typ": "JWT"
}
Payload: {
  "iss": "https://auth.example.com",
  "sub": "user-12345",
  "aud": "my-client-id",
  "exp": 1737000000,
  "iat": 1736996400,
  "auth_time": 1736996390,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "email_verified": true,
  "picture": "https://example.com/alice.jpg",
  "https://example.com/roles": ["admin", "user"]
}
```

**Why this output:** The ID Token header identifies the signing algorithm (`RS256`) and key ID (`kid: key-1`). The payload contains the required claims (`iss`, `sub`, `aud`, `exp`, `iat`) plus standard profile claims (`name`, `email`, `picture`) and a custom claim (`https://example.com/roles`). The RP validates these claims after verifying the signature.

#### Real-World Cases

- **Google ID Tokens:** Contain `email`, `name`, `picture`, and `email_verified` claims.
- **Microsoft Entra ID Tokens:** Contain `oid` (object ID), `tid` (tenant ID), and `roles` claims.
- **Auth0 ID Tokens:** Support custom claims via Actions and Rules.
- **Keycloak ID Tokens:** Include realm roles and client roles as custom claims.

---

### Sub-Feature 2.3: Cryptographic Verification

#### Definitions

**Core Definition:** Cryptographic verification is the process of validating the ID Token's signature using the OpenID Provider's public key to ensure the token is authentic and has not been tampered with.

**Technical Definition:** The RP verifies the ID Token by fetching the OP's JSON Web Key Set (JWKS) from the `jwks_uri` specified in the discovery document. Each key in the JWKS is tagged with a `kid`. The RP extracts the `kid` from the ID Token header, looks up the matching key in the JWKS, and uses that public key to verify the signature. The RP MUST also verify the `iss`, `aud`, `exp`, and `nonce` claims. 

**Beginner-Friendly Explanation:** Verifying an ID Token is like checking a wax seal on a letter. The OP has a unique stamp (private key) that only it can use. The RP has a publicly available copy of the stamp (public key). If the seal on the letter matches the public stamp, the letter is authentic. The RP must also check that the letter is addressed to it (aud), hasn't expired (exp), and came from the right office (iss).

#### Purposes

- To ensure the ID Token is authentic and issued by the claimed OP.
- To detect tampering with the token's header or payload.
- To validate that the token is intended for this specific RP.
- To prevent replay attacks using the nonce and expiration claims.

#### Syntax Rules and Structure

**Verification Steps:**
1. Fetch the JWKS from the `jwks_uri` in the discovery document.
2. Extract the `kid` from the ID Token header.
3. Find the matching key in the JWKS.
4. Verify the signature using the public key and the algorithm from the header.
5. Validate the `iss` claim matches the expected issuer.
6. Validate the `aud` claim contains the RP's `client_id`.
7. Validate the `exp` claim is in the future.
8. Validate the `nonce` claim matches the one sent in the authentication request.

#### Syntax Rules

- The `algorithms` list MUST be explicitly specified in the verification call to prevent algorithm confusion attacks. 
- The `issuer` and `audience` MUST be verified on every token. 
- The `kid` is a lookup hint, not a security decision — the key must come from the trusted JWKS endpoint. 
- The `jwks_uri` MUST be fetched over HTTPS.

#### Constraints and Limitations

- Key rotation requires the RP to handle `kid` changes gracefully.
- Caching the JWKS is necessary to avoid hitting the OP on every request; cache duration should balance freshness and performance. 
- The `jwks-rsa` library is the standard Node.js client for JWKS lookup. 

#### Annotated Code Example

```js
// id-token-verification.js
const express = require('express');
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();

// Configure JWKS client with caching and rate limiting
const client = jwksClient({
  jwksUri: 'https://your-tenant.auth0.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 600000,        // 10 minutes
  rateLimit: true,
  jwksRequestsPerMinute: 10,
  timeout: 30000
});

// Key retrieval function
function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    if (err) return callback(err);
    callback(null, key.getPublicKey());
  });
}

// Verify ID Token
app.post('/verify', (req, res) => {
  const { idToken } = req.body;

  jwt.verify(
    idToken,
    getKey,
    {
      algorithms: ['RS256'],                          // Pin allowed algorithms
      issuer: 'https://your-tenant.auth0.com/',       // Verify issuer
      audience: 'api://my-service'                    // Verify audience
    },
    (err, payload) => {
      if (err) {
        return res.status(401).json({
          error: 'Token verification failed',
          message: err.message
        });
      }

      res.json({
        verified: true,
        claims: {
          sub: payload.sub,
          email: payload.email,
          name: payload.name
        }
      });
    }
  );
});

app.listen(3000, () => console.log('Verification server on 3000'));
```

**Expected Output (for a valid ID Token):**
```json
{
  "verified": true,
  "claims": {
    "sub": "auth0|12345",
    "email": "alice@example.com",
    "name": "Alice Johnson"
  }
}
```

**Expected Output (for an expired ID Token):**
```json
{
  "error": "Token verification failed",
  "message": "jwt expired"
}
```

**Why this output:** The `jwks-rsa` client fetches the OP's public keys from the JWKS endpoint and caches them. The `jwt.verify` function retrieves the key matching the `kid` in the token header, verifies the signature using RS256, and validates the `iss` and `aud` claims. If the signature is invalid or the token is expired, verification fails and an error is returned. 

#### Real-World Cases

- **Auth0:** Uses `jwks-rsa` with `express-jwt` for API authentication.
- **Keycloak:** JWTs verified against the realm's JWKS endpoint at `/realms/{realm}/protocol/openid-connect/certs`.
- **Google:** Public keys at `https://www.googleapis.com/oauth2/v3/certs`.
- **Azure AD B2C:** Uses RSA-256 public keys from the OpenID Connect metadata endpoint.

---

## Core Concept 3: User Identity & UserInfo

### Definitions

**Core Definition:** The UserInfo endpoint is an OAuth 2.0 protected resource that returns claims about the authenticated end-user when presented with a valid access token.

**Technical Definition:** The UserInfo Endpoint is part of the OpenID Connect standard and is designed to return claims about the authenticated user. It is accessed with the access token obtained during the authentication flow. The response is a JSON object containing claims about the user. The `sub` claim in the UserInfo response MUST exactly match the `sub` claim in the ID Token; otherwise, the UserInfo response must not be used. 

**Beginner-Friendly Explanation:** The UserInfo endpoint is like a "profile lookup" service. After you've proven who you are (via the ID Token), you can ask the OP for additional details about yourself — your full name, email, address, phone number — using a separate access token. This keeps the ID Token small and lets you fetch profile data only when needed.

### Purposes

- To retrieve additional identity claims beyond those in the ID Token.
- To keep the ID Token minimal (often containing only `sub`).
- To profile users with standard and custom claims.
- To enable session synchronization and periodic claim refresh.

### Sub-Feature 3.1: UserInfo Request and Response

#### Syntax Rules and Structure

**UserInfo Request:**
```
GET /userinfo
Authorization: Bearer {access_token}
```

| Component | Breakdown |
|-----------|-----------|
| `GET /userinfo` | The UserInfo endpoint URL from discovery. |
| `Authorization: Bearer` | The access token obtained from the token endpoint. |

**UserInfo Response:**
```json
{
  "sub": "user-12345",
  "name": "Alice Johnson",
  "given_name": "Alice",
  "family_name": "Johnson",
  "email": "alice@example.com",
  "email_verified": true,
  "picture": "https://example.com/alice.jpg",
  "locale": "en-US",
  "updated_at": 1736996400
}
```

| Claim | Description |
|-------|-------------|
| `sub` | MUST match the ID Token `sub`. |
| `name` | Full name. |
| `email` | Email address. |
| `picture` | Profile picture URL. |
| `locale` | User's locale. |
| `updated_at` | Last profile update time. |

#### Syntax Rules

- The UserInfo endpoint MUST require a valid access token.
- The `sub` claim in the UserInfo response MUST exactly match the `sub` from the ID Token. 
- The response is a JSON object; the `Content-Type` is `application/json`.
- The UserInfo response MAY include custom claims defined by the OP.
- If the UserInfo response is signed, it MUST contain `iss` and `aud` claims. 

#### Constraints and Limitations

- Not all IdPs support the UserInfo endpoint; some return all claims in the ID Token.
- The UserInfo endpoint may require the `openid` scope and additional scopes for specific claims.
- The `sub` match check is critical for security — a mismatch means the response is invalid. 

#### Annotated Code Example

```js
// userinfo-endpoint.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

// Simulated UserInfo endpoint
app.get('/userinfo', (req, res) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401)
      .set('WWW-Authenticate', 'Bearer realm="userinfo"')
      .json({ error: 'invalid_token' });
  }

  const accessToken = authHeader.split(' ')[1];

  // In production: verify access token signature and expiry
  try {
    const decoded = jwt.verify(accessToken, 'secret-key');
    // Look up user by sub
    const user = {
      sub: decoded.sub,
      name: 'Alice Johnson',
      given_name: 'Alice',
      family_name: 'Johnson',
      email: 'alice@example.com',
      email_verified: true,
      picture: 'https://example.com/alice.jpg',
      locale: 'en-US',
      updated_at: 1736996400
    };
    res.json(user);
  } catch (err) {
    res.status(401).json({ error: 'invalid_token' });
  }
});

app.listen(3000, () => console.log('UserInfo endpoint on 3000'));
```

**Expected Output (for `GET /userinfo` with valid Bearer token):**
```json
{
  "sub": "user-12345",
  "name": "Alice Johnson",
  "given_name": "Alice",
  "family_name": "Johnson",
  "email": "alice@example.com",
  "email_verified": true,
  "picture": "https://example.com/alice.jpg",
  "locale": "en-US",
  "updated_at": 1736996400
}
```

**Why this output:** The UserInfo endpoint requires a valid access token in the `Authorization: Bearer` header. After verifying the token, it looks up the user by `sub` and returns the profile claims. The RP then merges these claims with the ID Token claims to build a complete user profile.

#### Real-World Cases

- **Microsoft identity platform:** UserInfo endpoint returns `name`, `sub`, and `email` when consented. 
- **Google:** UserInfo endpoint returns profile, email, and address claims based on granted scopes.
- **Auth0:** UserInfo endpoint returns claims from the ID token and custom claims.
- **FusionAuth:** UserInfo endpoint returns custom claims defined by the IdP. 

---

### Sub-Feature 3.2: Claims Mapping

#### Definitions

**Core Definition:** Claims mapping is the process of translating IdP-specific claims into the application's internal user model, ensuring consistent identity representation across different providers.

**Technical Definition:** Different IdPs may use different claim names or structures for the same conceptual information. Claims mapping normalizes these into a canonical form used by the RP. For example, `sub` is always the unique identifier, but `oid` (Microsoft), `user_id` (custom), or `uid` may need to be mapped to `sub` in some contexts. Role claims may be named `roles`, `groups`, `realm_access.roles`, or custom URIs. 

**Beginner-Friendly Explanation:** Claims mapping is like translating different languages into a common one. Google might call your full name "name," Microsoft might call it "displayName," and Okta might call it "firstName + lastName." Claims mapping ensures your application understands all of them as the same concept: "the user's full name."

### Purposes

- To normalize identity data from different IdPs into a consistent internal model.
- To map IdP roles and groups to application-specific permissions.
- To handle custom claims that carry domain-specific information.
- To enable multi-IdP support with a single internal user model.

### Syntax Rules and Structure

**Mapping Table:**

| Internal Field | Common IdP Claim Names |
|---------------|----------------------|
| `userId` | `sub`, `oid`, `uid` |
| `email` | `email`, `mail`, `preferred_username` |
| `name` | `name`, `displayName`, `cn` |
| `roles` | `roles`, `groups`, `realm_access.roles`, custom URI |
| `department` | `department`, `dept`, custom claim |

#### Syntax Rules

- Always use `sub` as the primary unique identifier; it is stable and never reassigned.
- Map role claims to application permissions in a dedicated mapping layer.
- Handle missing claims gracefully with defaults or fallbacks.
- Document all claim mappings in the integration specification.

#### Constraints and Limitations

- Claim names are case-sensitive and may vary between IdPs.
- Custom claims may use URI namespaces (e.g., `https://example.com/roles`).
- Some IdPs encode roles as arrays; others as space-delimited strings.

#### Annotated Code Example

```js
// claims-mapping.js
const express = require('express');
const app = express();

// IdP-specific claims (from ID Token and UserInfo)
const idpClaims = {
  sub: 'auth0|12345',
  email: 'alice@example.com',
  name: 'Alice Johnson',
  'https://example.com/roles': ['admin', 'user'],
  'https://example.com/department': 'Engineering'
};

// Mapping function: IdP claims → internal user model
function mapClaims(idpClaims) {
  return {
    userId: idpClaims.sub,                              // Primary identifier
    email: idpClaims.email || null,
    displayName: idpClaims.name || idpClaims.email,
    roles: idpClaims['https://example.com/roles'] ||
           idpClaims.roles ||
           idpClaims.groups || [],
    department: idpClaims['https://example.com/department'] ||
                idpClaims.department || null,
    isAdmin: (idpClaims['https://example.com/roles'] || [])
              .includes('admin')
  };
}

app.get('/user/profile', (req, res) => {
  const internalUser = mapClaims(idpClaims);
  res.json(internalUser);
});

app.listen(3000, () => console.log('Claims mapping on 3000'));
```

**Expected Output:**
```json
{
  "userId": "auth0|12345",
  "email": "alice@example.com",
  "displayName": "Alice Johnson",
  "roles": ["admin", "user"],
  "department": "Engineering",
  "isAdmin": true
}
```

**Why this output:** The mapping function translates IdP-specific claim names (including custom URI claims) into a consistent internal model. The `sub` claim becomes `userId`, the custom roles claim becomes `roles`, and a derived `isAdmin` boolean is computed. This internal model is used throughout the application, decoupling it from IdP-specific claim names.

#### Real-World Cases

- **Multi-IdP SSO:** Supporting Google, Microsoft, and Okta with a single internal user model.
- **Enterprise RBAC:** Mapping IdP groups to application roles.
- **SaaS platforms:** Normalizing user data across different customer IdPs.

---

### Sub-Feature 3.3: Session Syncing

#### Definitions

**Core Definition:** Session syncing is the practice of periodically refreshing user claims from the IdP to ensure that role changes, profile updates, and permission revocations propagate to the application without requiring the user to re-login.

**Technical Definition:** OIDC claim-to-role mapping typically happens only at login time. To keep claims current, the RP can store the refresh token and periodically use it to re-fetch UserInfo and re-sync claim mappings. This ensures that role changes in the IdP propagate to the application. Additionally, the OIDC `end_session_endpoint` allows the RP to notify the IdP of logout, terminating the SSO session. 

**Beginner-Friendly Explanation:** Session syncing is like checking for updates to your ID card. If your role changes at the IdP — you get promoted, or your permissions are revoked — the application needs to know about it. Instead of waiting for you to log out and back in, the application periodically asks the IdP "has anything changed?" using the refresh token.

### Purposes

- To propagate IdP role changes without requiring re-login.
- To keep user profile data current.
- To enforce permission revocations promptly.
- To synchronize sessions across multiple applications.

### Syntax Rules and Structure

**Session Sync Flow:**
1. Store refresh token and ID token server-side.
2. Periodically (or on access token expiry) use refresh token to get new tokens.
3. Re-fetch UserInfo with the new access token.
4. Re-map claims and update the local session.
5. Optionally, revoke old tokens.

#### Syntax Rules

- Store refresh tokens securely (server-side, encrypted).
- Use the refresh token rotation pattern to detect token theft.
- Handle refresh failures by invalidating the local session.
- Use `end_session_endpoint` for logout to terminate the IdP SSO session.

#### Constraints and Limitations

- Not all IdPs support refresh tokens or UserInfo re-fetch.
- Frequent syncing increases load on the IdP.
- Role changes may take time to propagate through caches.

#### Annotated Code Example

```js
// session-sync.js
const express = require('express');
const app = express();

// Simulated session store
const sessions = new Map();

// Periodic session sync (runs every 15 minutes)
async function syncSession(sessionId) {
  const session = sessions.get(sessionId);
  if (!session) return;

  try {
    // Step 1: Refresh access token using refresh token
    const tokenResponse = await fetch('https://auth.example.com/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: session.refreshToken,
        client_id: 'my-client',
        client_secret: 'my-secret'
      })
    });

    const tokens = await tokenResponse.json();
    if (!tokens.access_token) throw new Error('Refresh failed');

    // Step 2: Re-fetch UserInfo with new access token
    const userInfoResponse = await fetch('https://auth.example.com/userinfo', {
      headers: { Authorization: `Bearer ${tokens.access_token}` }
    });
    const userInfo = await userInfoResponse.json();

    // Step 3: Verify sub matches ID token sub
    if (userInfo.sub !== session.sub) {
      throw new Error('UserInfo sub does not match ID token sub');
    }

    // Step 4: Re-map claims and update session
    session.claims = {
      ...session.claims,
      ...userInfo
    };
    session.refreshToken = tokens.refresh_token || session.refreshToken;
    session.lastSyncedAt = new Date();

    sessions.set(sessionId, session);
    console.log(`Session ${sessionId} synced at ${session.lastSyncedAt}`);
  } catch (err) {
    console.error(`Sync failed for ${sessionId}:`, err.message);
    // On failure, invalidate the session
    sessions.delete(sessionId);
  }
}

// Run sync every 15 minutes
setInterval(() => {
  for (const [id] of sessions) {
    syncSession(id);
  }
}, 15 * 60 * 1000);

app.listen(3000, () => console.log('Session sync on 3000'));
```

**Expected Output (after a successful sync):**
```
Session abc123 synced at 2026-01-15T10:45:00.000Z
```

**Expected Output (after a failed sync — session invalidated):**
```
Sync failed for abc123: UserInfo sub does not match ID token sub
```

**Why this output:** The sync function uses the stored refresh token to obtain a new access token, then re-fetches UserInfo. It verifies that the `sub` in the UserInfo response matches the original ID token `sub` — if not, the session is invalidated for security. On success, the session claims are updated with the latest profile data and role mappings.

#### Real-World Cases

- **Enterprise SSO:** Syncing group memberships from Microsoft Entra ID every 15 minutes.
- **SaaS platforms:** Propagating role changes from Okta without requiring re-login.
- **Financial services:** Enforcing permission revocations within minutes of an HR change.
- **Healthcare:** Syncing clinician credentials and permissions across systems.

---

## References

- OpenID Connect Core 1.0 (incorporating errata set 3) — https://www.openid.net/specs/openid-connect-core-1_0-36.txt
- OpenID Connect Discovery 1.0 — https://openid.net/specs/openid-connect-discovery-1_0.html
- OpenID Connect Core 1.0 (Final) — https://openid.net/specs/openid-connect-core-1_0.html
- RFC 7519 — JSON Web Token (JWT) — https://www.rfc-editor.org/rfc/rfc7519
- RFC 7517 — JSON Web Key (JWK) — https://www.rfc-editor.org/rfc/rfc7517
- RFC 8414 — OAuth 2.0 Authorization Server Metadata — https://www.rfc-editor.org/rfc/rfc8414
- jwks-rsa npm Documentation — https://www.npmjs.com/package/jwks-rsa
- jwks-rsa: Verifying JWTs Against a JWKS Endpoint Safely — https://safeguard.sh/resources/blog/jwks-rsa-npm-jwt-verification-guide
- Microsoft identity platform UserInfo endpoint — https://learn.microsoft.com/en-us/entra/identity-platform/userinfo
- Microsoft Entra ID OpenID Connect documentation — https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc
- Google OpenID Connect Discovery — https://accounts.google.com/.well-known/openid-configuration
- Understanding the basics of OIDC — https://docs.developer.singpass.gov.sg/docs/introduction/understanding-the-basics-of-oidc
- OpenID Connect Authentication (OneUptime Blog) — https://github.com/OneUptime/blog/tree/master/posts/2026-02-02-openid-connect-authentication
- Claims Lifecycle (Duende Software) — https://docs.duendesoftware.com
- OpenID Connect for Identity Assurance — https://openid.net/specs/openid-connect-4-identity-assurance-1_0.html
- OIDC Token Validation: How to Verify ID Tokens — https://securityboulevard.com