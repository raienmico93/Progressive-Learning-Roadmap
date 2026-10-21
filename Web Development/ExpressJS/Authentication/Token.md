# Express.js Token Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Token-based authentication is an authentication mechanism in which the server issues a signed, self-contained credential—a **token**—to a client after successful login, and the client presents that token on subsequent requests to prove its identity, without the server maintaining session state.

**Technical Definition**
In Express.js, token authentication typically uses JSON Web Tokens (JWTs) defined by RFC 7519. A JWT is a compact, URL-safe string composed of three Base64URL-encoded parts—a **header** specifying the signing algorithm and token type, a **payload** containing claims such as subject (`sub`), expiration (`exp`), issuer (`iss`), and audience (`aud`), and a **signature** that guarantees integrity. The client transmits the token in the `Authorization: Bearer <token>` HTTP header, and Express middleware verifies the signature and claims using a secret (HMAC) or public key (RSA/ECDSA) before attaching the decoded identity to `req.user`. Because verification is stateless and requires no session store lookup, token authentication scales horizontally and is well suited to APIs, microservices, and mobile clients.

**Beginner-Friendly Explanation**
Token authentication is like getting a stamped wristband at an event. After you show your ID once (log in), the server gives you a wristband (token) with your name and an expiry time stamped on it. On every later request, you show the wristband instead of your ID. The server checks the stamp to make sure the wristband is real and hasn’t expired—without needing to remember you in a database. The wristband itself is not secret: anyone can read what’s written on it, but they cannot forge the stamp.

---

### Key Characteristics

- **Stateless** — The server does not store session data; every request carries its own proof of identity.
- **Signed, not encrypted** — JWT payloads are readable by anyone; the signature ensures integrity, not confidentiality.
- **Short-lived by design** — Access tokens typically expire in 5–15 minutes, limiting the damage of theft.
- **Refresh-capable** — Long-lived refresh tokens obtain new access tokens without requiring the user to log in again.
- **Algorithm-sensitive** — The verification algorithm must be pinned server-side to prevent algorithm-confusion and `none`-algorithm attacks.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- The `jsonwebtoken` package installed (`npm install jsonwebtoken`).
- Understanding of Express middleware, the `(req, res, next)` signature, and HTTP headers.
- Basic familiarity with cryptographic signing concepts (HMAC, RSA, ECDSA).
- For refresh tokens: a persistence layer (database or Redis) to store token state.

---

### Related Programming Areas

- **Session authentication** — An alternative stateful approach; often compared with token-based auth.
- **OAuth 2.0 and OpenID Connect** — Standard protocols that use JWTs as access tokens and ID tokens.
- **Authorization** — Token claims (roles, scopes) drive access-control decisions.
- **API security** — Token handling is a critical part of API hardening.
- **Cryptography** — Key management, signing algorithms, and rotation.

---

## Core Concept 1: Token-Based Authentication

### Definitions

**Core Definition**
Token-based authentication is an authentication model in which the server issues a signed token to a client upon successful authentication, and the client uses that token to authenticate subsequent requests without the server maintaining session state.

**Technical Definition**
Upon successful credential verification, the server generates a token containing claims about the authenticated principal and signs it with a secret or private key. The client stores the token and sends it in the `Authorization` header as a Bearer token. On each request, Express middleware extracts the token, verifies its signature and claims (expiration, issuer, audience), and, on success, attaches the decoded identity to `req.user`. Because verification is purely cryptographic, the server does not need to consult a session store, enabling horizontal scalability. The trade-off is that stateless tokens cannot be revoked before expiration without a denylist or short expiration combined with refresh-token rotation.

**Beginner-Friendly Explanation**
Token authentication is a way for the server to remember you without actually remembering you. After login, the server gives you a signed “pass” that says who you are and when it expires. You show that pass on every request. The server checks the signature—like checking a hologram on a passport—and lets you in if it’s valid. It never has to look up your session in a database.

---

### Purposes

- To enable stateless, horizontally scalable authentication for APIs and microservices.
- To avoid the server-side session store lookup on every request.
- To support mobile, single-page, and third-party clients that handle tokens more naturally than cookies.
- To provide a portable, language-agnostic authentication mechanism.

---

### Syntax Rules and Structure

#### Complete General Syntax: Bearer Token Flow

```text
Client → POST /login (credentials)
Server → 200 { "accessToken": "<JWT>" }
Client → GET /protected  Authorization: Bearer <JWT>
Server → verifies signature and claims → 200 protected data
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Authorization: Bearer <token>` | Standard HTTP header carrying the token |
| Token verification | Signature + claim validation in middleware |
| `req.user` | Decoded identity attached for downstream handlers |

#### Syntax Rules

- The token must be transmitted over HTTPS in production.
- The `Authorization` header scheme is `Bearer` followed by a space and the token.
- Verification must occur in middleware before route handlers access protected resources.

#### Constraints and Limitations

- Stateless tokens cannot be revoked instantly without additional server-side state (denylist or short expiry + refresh rotation).
- Token theft grants full access until expiration.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Minimal Token Authentication Middleware

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const JWT_SECRET = process.env.JWT_SECRET;

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing or malformed token' });
  }

  const token = authHeader.slice(7); // remove "Bearer "
  try {
    const payload = jwt.verify(token, JWT_SECRET, { algorithms: ['HS256'] });
    req.user = { id: payload.sub, role: payload.role };
    next();
  } catch (err) {
    next(err); // forward to centralized error handler
  }
}

app.get('/protected', authenticate, (req, res) => {
  res.json({ message: `Hello, user ${req.user.id}` });
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Header extraction** — The middleware reads the `Authorization` header and checks for the `Bearer` scheme.
2. **Verification** — `jwt.verify()` validates the signature against the secret and enforces an explicit `algorithms` allowlist (`['HS256']`).
3. **Identity attachment** — The decoded `sub` and `role` claims are attached to `req.user`.
4. **Error forwarding** — Invalid or expired tokens are forwarded to the centralized error handler.

**Expected Output**

```
GET /protected  (Authorization: Bearer <valid-token>)
→ 200 { "message": "Hello, user 42" }

GET /protected  (no header)
→ 401 { "error": "Missing or malformed token" }

GET /protected  (expired token)
→ 401 (error handler response)
```

**Why This Output Occurs**

- The middleware short-circuits unauthenticated requests and only calls `next()` after successful verification.

---

### Real-World Cases

- **REST APIs** — Stateless authentication for endpoints consumed by web, mobile, and third-party clients.
- **Microservices** — Each service verifies the token independently without shared session state.
- **Mobile applications** — Tokens are more natural to store and transmit than cookies.

---

## Core Concept 2: JWT (JSON Web Token)

### Definitions

**Core Definition**
A JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties, defined by RFC 7519, consisting of a header, a payload, and a signature.

**Technical Definition**
A JWT is a string of three Base64URL-encoded segments separated by dots: `header.payload.signature`. The **header** declares the token type (`JWT`) and signing algorithm (`alg`). The **payload** contains claims—registered claims such as `iss` (issuer), `sub` (subject), `aud` (audience), `exp` (expiration), `nbf` (not before), and `iat` (issued at), as well as custom claims. The **signature** is computed over the encoded header and payload using a secret (HMAC) or private key (RSA/ECDSA). The payload is **not encrypted**; it is only Base64URL-encoded, so anyone can read it, but no one can alter it without invalidating the signature.

**Beginner-Friendly Explanation**
A JWT is like a tamper-evident envelope. Inside the envelope is a note saying who you are and when the note expires. The envelope is sealed with a special stamp. Anyone can open the envelope and read the note, but if someone changes the note, the stamp no longer matches and the server knows it has been forged.

---

### Purposes

- To carry identity and authorization claims in a compact, URL-safe format.
- To enable stateless verification through cryptographic signatures.
- To support interoperable authentication across languages and platforms.
- To express time-bound access through `exp` and `nbf` claims.

---

### Syntax Rules and Structure

#### Complete General Syntax: JWT Structure

```text
<base64url(header)>.<base64url(payload)>.<base64url(signature)>
```

**Component Breakdown**

| Segment | Typical Contents |
|---------|------------------|
| Header | `{"alg":"RS256","typ":"JWT"}` |
| Payload | `{"sub":"42","role":"admin","exp":1735689600}` |
| Signature | Cryptographic signature over `header.payload` |

#### Registered Claims

| Claim | Meaning |
|-------|---------|
| `iss` | Issuer |
| `sub` | Subject (user ID) |
| `aud` | Audience (intended recipient) |
| `exp` | Expiration time (NumericDate) |
| `nbf` | Not before (NumericDate) |
| `iat` | Issued at (NumericDate) |

#### Syntax Rules

- The payload must not contain sensitive data (passwords, secrets, PII) because it is readable by anyone holding the token.
- `exp` should be short for access tokens (typically 5–15 minutes).
- The `alg` header must match the algorithm pinned by the server; never trust the token’s algorithm without an allowlist.

#### Constraints and Limitations

- JWTs are signed, not encrypted; confidentiality requires JWE (JSON Web Encryption), which is a separate specification.
- Large payloads increase request size; keep claims minimal.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Decoding a JWT Without Verification

```javascript
const jwt = require('jsonwebtoken');

const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
const decoded = jwt.decode(token);

console.log(decoded);
// { sub: '42', role: 'admin', exp: 1735689600, iat: 1735686000 }
```

**Expected Output**

```
{ sub: '42', role: 'admin', exp: 1735689600, iat: 1735686000 }
```

**Why This Output Occurs**

- `jwt.decode()` performs Base64URL decoding without verifying the signature. It must never be used as a substitute for `jwt.verify()` in authentication decisions.

---

### Real-World Cases

- **Access tokens** — Short-lived JWTs carrying user ID and scopes.
- **ID tokens** — OpenID Connect identity assertions.
- **Service-to-service tokens** — Machine-to-machine authentication with asymmetric signatures.

---

## Core Concept 3: `jsonwebtoken`

### Definitions

**Core Definition**
`jsonwebtoken` is the standard Node.js library for signing, verifying, and decoding JSON Web Tokens, implementing the JWT specification (RFC 7519).

**Technical Definition**
The library exposes `jwt.sign(payload, secretOrPrivateKey, options)` for token creation and `jwt.verify(token, secretOrPublicKey, options, callback)` for verification. Signing options include `algorithm` (default `HS256`), `expiresIn`, `notBefore`, `audience`, `issuer`, `subject`, `jwtid`, and `keyid`. Verification options include an explicit `algorithms` allowlist (required for security), `audience`, `issuer`, `ignoreExpiration`, and `clockTolerance`. As of version 9.0.0, the library rejects the `none` algorithm by default and requires explicit algorithm specification for verification. Historical vulnerabilities (CVE-2015-9235, CVE-2022-23540, CVE-2022-23541) stemmed from algorithm confusion and unsafe defaults; v9+ closes these when used correctly.

**Beginner-Friendly Explanation**
`jsonwebtoken` is the toolbox for working with JWTs in Node.js. It can create tokens (signing them) and check tokens (verifying them). It’s like a stamp and a magnifying glass: one to seal the envelope, one to check the seal.

---

### Purposes

- To generate signed JWTs with configurable claims and expiration.
- To verify token signatures and claims securely.
- To decode token payloads for inspection (without verification).
- To support both symmetric (HMAC) and asymmetric (RSA/ECDSA) signing.

---

### Syntax Rules and Structure

#### Complete General Syntax: `jwt.sign` and `jwt.verify`

```javascript
const jwt = require('jsonwebtoken');

// Sign
const token = jwt.sign(
  { sub: user.id, role: user.role },   // payload
  process.env.JWT_SECRET,               // secret or private key
  { expiresIn: '15m', algorithm: 'HS256', issuer: 'https://auth.example.com', audience: 'api.example.com' }
);

// Verify
jwt.verify(
  token,
  process.env.JWT_SECRET,
  { algorithms: ['HS256'], issuer: 'https://auth.example.com', audience: 'api.example.com' },
  (err, payload) => {
    if (err) { /* invalid or expired */ }
    // payload contains sub, role, iat, exp
  }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `payload` | Claims object; `sub` is the standard subject claim |
| `secretOrPrivateKey` | HMAC secret or PEM private key |
| `options.algorithm` | Signing algorithm (`HS256` default; prefer `RS256`/`ES256` for distributed systems) |
| `options.expiresIn` | Token lifetime (`'15m'`, `'7d'`) |
| `options.algorithms` (verify) | **Explicit allowlist**; prevents algorithm confusion |

#### Syntax Rules

- `jwt.verify()` **must** receive an explicit `algorithms` allowlist; never let the token header decide the algorithm.
- The `none` algorithm must never be allowed.
- Secrets must be high-entropy (at least 32 bytes for HS256) and stored in environment variables.
- Asymmetric algorithms (RS256/ES256) are recommended over HS256 in distributed systems because verification does not require sharing a secret.

#### Constraints and Limitations

- `jwt.verify()` throws on invalid tokens; use a callback or `try/catch` to handle errors gracefully.
- `jwt.decode()` does not verify signatures and must not be used for authentication.
- Version ≤8.5.1 allowed `none` algorithm fallback when a falsy secret was used; upgrade to v9+.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Signing and Verifying with Algorithm Allowlist

```javascript
const jwt = require('jsonwebtoken');

const SECRET = process.env.JWT_SECRET;

// Sign
const token = jwt.sign(
  { sub: 'user_42', role: 'editor' },
  SECRET,
  { algorithm: 'HS256', expiresIn: '15m', issuer: 'my-api', audience: 'my-client' }
);

// Verify with explicit allowlist
try {
  const payload = jwt.verify(token, SECRET, {
    algorithms: ['HS256'],
    issuer: 'my-api',
    audience: 'my-client'
  });
  console.log(payload); // { sub: 'user_42', role: 'editor', iat: ..., exp: ... }
} catch (err) {
  console.error('Invalid token:', err.message);
}
```

**Expected Output**

```
{ sub: 'user_42', role: 'editor', iat: 1735686000, exp: 1735686900 }
```

**Why This Output Occurs**

- The signature matches, the algorithm is in the allowlist, and the issuer/audience claims match the expected values.

---

#### Example 2: Rejecting the `none` Algorithm

```javascript
const jwt = require('jsonwebtoken');

// A token with alg: "none" and no signature
const noneToken = 'eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJoYWNrZXIifQ.';

try {
  jwt.verify(noneToken, SECRET, { algorithms: ['HS256'] });
} catch (err) {
  console.log(err.message); // "invalid algorithm"
}
```

**Expected Output**

```
invalid algorithm
```

**Why This Output Occurs**

- The explicit `algorithms: ['HS256']` allowlist rejects the `none` algorithm even though the token header claims it.

---

### Real-World Cases

- **API authentication** — Signing and verifying access tokens.
- **Refresh token rotation** — Generating new token pairs with unique `jti` claims.
- **Federated identity** — Verifying JWTs issued by external identity providers using JWKS public keys.

---

## Core Concept 4: Access Tokens

### Definitions

**Core Definition**
An access token is a short-lived credential, typically a JWT, that authorizes the bearer to access specific protected resources for a limited time.

**Technical Definition**
Access tokens are presented in the `Authorization: Bearer` header and verified by resource servers on every request. They carry claims that identify the subject and authorize specific actions (scopes or roles). Best practice sets their lifetime to **15 minutes or less** to minimize the window of opportunity if a token is stolen. Access tokens should **not** be stored in persistent client storage such as `localStorage`; `HttpOnly` cookies or in-memory storage are preferred.

**Beginner-Friendly Explanation**
An access token is like a day pass at a ski resort. It lets you ride the lifts, but it expires quickly—often in minutes. If someone steals it, they can only use it until it expires.

---

### Purposes

- To authorize API requests without re-presenting credentials.
- To limit the impact of token theft through short expiration.
- To carry scopes and roles for fine-grained authorization.
- To enable stateless verification on resource servers.

---

### Syntax Rules and Structure

#### Complete General Syntax: Access Token Lifetime

```javascript
const accessToken = jwt.sign(
  { sub: user.id, scope: 'read:profile write:profile' },
  process.env.JWT_SECRET,
  { expiresIn: '15m', algorithm: 'HS256' }
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `expiresIn: '15m'` | Short lifetime limits theft window |
| `scope` claim | Authorizes specific operations |
| `sub` claim | Identifies the user |

#### Syntax Rules

- Access tokens should expire in **15 minutes or less**.
- They should not be persisted in `localStorage`; use `HttpOnly` cookies or memory.
- Resource servers must verify the signature and claims on every request.

#### Constraints and Limitations

- Short expiration causes frequent re-authentication unless paired with refresh tokens.
- Access tokens cannot be revoked before expiration without a denylist.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Issuing a Short-Lived Access Token

```javascript
app.post('/login', async (req, res) => {
  const user = await verifyCredentials(req.body);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  const accessToken = jwt.sign(
    { sub: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m', algorithm: 'HS256' }
  );

  res.json({ accessToken, expiresIn: 900 });
});
```

**Expected Output**

```
POST /login  { "username": "alice", "password": "secret" }
→ 200 { "accessToken": "eyJ...", "expiresIn": 900 }
```

**Why This Output Occurs**

- The access token is signed with a 15-minute expiration, and the client is informed of the lifetime in seconds.

---

### Real-World Cases

- **API gateways** — Access tokens authorize requests to downstream services.
- **Single-page applications** — Short-lived tokens paired with refresh tokens.
- **Mobile apps** — Access tokens stored in secure platform storage.

---

## Core Concept 5: Refresh Tokens

### Definitions

**Core Definition**
A refresh token is a long-lived credential used to obtain new access tokens without requiring the user to re-authenticate, enabling persistent sessions while keeping access tokens short-lived.

**Technical Definition**
Refresh tokens are issued alongside access tokens at login. When an access token expires, the client sends the refresh token to a dedicated token endpoint, which validates the refresh token and issues a new access-token/refresh-token pair. Best practice requires **refresh-token rotation**: each refresh token is single-use, and using an old refresh token invalidates the entire session (rotation with reuse detection). Refresh tokens should be stored securely—ideally in `HttpOnly` cookies or server-side—and should be revocable on logout. OWASP recommends **sender-constrained** refresh tokens (DPoP or mTLS) or rotation to prevent replay. Token rotation reduces replay risk by more than 80% according to experimental analysis.

**Beginner-Friendly Explanation**
A refresh token is like a season pass. Your day pass (access token) expires quickly, but you can use the season pass (refresh token) to get a new day pass without going back to the ticket counter. The season pass is long-lived and valuable, so it must be stored very securely. Each time you use it, the resort gives you a new season pass and invalidates the old one—so if someone steals it, it won’t work after you’ve used it.

---

### Purposes

- To maintain long-lived sessions without compromising security through long-lived access tokens.
- To enable token rotation and revocation on logout.
- To reduce the frequency of user re-authentication.
- To detect and mitigate refresh-token replay attacks through rotation.

---

### Syntax Rules and Structure

#### Complete General Syntax: Refresh Token Endpoint with Rotation

```javascript
app.post('/token/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  // Verify refresh token signature and type
  const payload = jwt.verify(refreshToken, process.env.REFRESH_SECRET, {
    algorithms: ['HS256'],
    issuer: 'my-api'
  });

  // Check that the token is still valid in the store (rotation)
  const stored = await RefreshTokenStore.findValid(payload.jti);
  if (!stored) return res.status(401).json({ error: 'Refresh token revoked' });

  // Rotate: invalidate old token, issue new pair
  await RefreshTokenStore.revoke(payload.jti);
  const newAccessToken = jwt.sign(
    { sub: payload.sub },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );
  const newRefreshToken = jwt.sign(
    { sub: payload.sub, jti: crypto.randomUUID() },
    process.env.REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  res.json({ accessToken: newAccessToken, refreshToken: newRefreshToken });
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `refreshToken` | Long-lived token sent by the client |
| `jti` claim | Unique identifier for rotation and revocation |
| `RefreshTokenStore` | Persistence layer tracking valid tokens |
| Rotation | Old token revoked; new pair issued |

#### Syntax Rules

- Refresh tokens must have a **type** claim (`type: 'refresh'`) to prevent access tokens from being used as refresh tokens.
- Use a separate secret or key pair for refresh tokens.
- Implement **rotation with reuse detection**: if an old token is reused, revoke the entire session.
- Store refresh tokens securely (`HttpOnly` cookies or server-side).

#### Constraints and Limitations

- Refresh-token rotation adds storage complexity (tracking valid tokens).
- If a refresh token is stolen before rotation, the attacker can use it; rotation limits but does not eliminate the risk.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Refresh Token Rotation with Reuse Detection

```javascript
const refreshTokens = new Map(); // jti -> { userId, used: false }

app.post('/login', (req, res) => {
  const user = authenticate(req.body);
  const jti = crypto.randomUUID();
  refreshTokens.set(jti, { userId: user.id, used: false });

  const accessToken = jwt.sign({ sub: user.id }, SECRET, { expiresIn: '15m' });
  const refreshToken = jwt.sign({ sub: user.id, jti, type: 'refresh' }, REFRESH_SECRET, { expiresIn: '7d' });

  res.json({ accessToken, refreshToken });
});

app.post('/refresh', (req, res) => {
  const { refreshToken } = req.body;
  const payload = jwt.verify(refreshToken, REFRESH_SECRET, { algorithms: ['HS256'] });

  const record = refreshTokens.get(payload.jti);
  if (!record || record.used) {
    // Reuse detected: revoke all tokens for this user
    return res.status(401).json({ error: 'Refresh token reuse detected' });
  }

  record.used = true; // invalidate old token
  const newJti = crypto.randomUUID();
  refreshTokens.set(newJti, { userId: payload.sub, used: false });

  const newAccessToken = jwt.sign({ sub: payload.sub }, SECRET, { expiresIn: '15m' });
  const newRefreshToken = jwt.sign({ sub: payload.sub, jti: newJti, type: 'refresh' }, REFRESH_SECRET, { expiresIn: '7d' });

  res.json({ accessToken: newAccessToken, refreshToken: newRefreshToken });
});
```

**Expected Output**

```
POST /refresh  { "refreshToken": "<valid-old-token>" }
→ 200 { "accessToken": "<new>", "refreshToken": "<new>" }

POST /refresh  { "refreshToken": "<reused-old-token>" }
→ 401 { "error": "Refresh token reuse detected" }
```

**Why This Output Occurs**

- The old token is marked as used; reusing it triggers detection and rejection.

---

### Real-World Cases

- **Long-lived web sessions** — Users stay logged in across days without re-entering credentials.
- **Mobile applications** — Refresh tokens stored in secure platform storage.
- **OAuth 2.0 flows** — Refresh tokens obtain new access tokens from the authorization server.

---

## Core Concept 6: Token Storage

### Definitions

**Core Definition**
Token storage is the client-side mechanism used to persist access and refresh tokens between requests, with security implications that differ sharply between storage options.

**Technical Definition**
The two primary browser storage options are `localStorage`/`sessionStorage` and `HttpOnly` cookies. **`localStorage`** is readable by any JavaScript on the page, making it vulnerable to XSS token exfiltration; it is, however, immune to CSRF because tokens are not sent automatically. **`HttpOnly` cookies** are not readable by JavaScript, protecting against XSS-based theft, but reintroduce CSRF risk, which is mitigated with `SameSite=Lax/Strict` and anti-CSRF tokens. The prevailing 2026 guidance for browser SPAs is to store tokens in `HttpOnly`, `Secure`, `SameSite` cookies, often via a backend-for-frontend that keeps tokens server-side and exposes only a session cookie to the browser.

**Beginner-Friendly Explanation**
Token storage is where you keep your wristband. If you keep it in your pocket (localStorage), any pickpocket (XSS) can grab it. If you keep it in a locked locker that only the resort staff can open (HttpOnly cookie), pickpockets cannot reach it—but someone might trick you into using the locker for them (CSRF), so you need extra protections.

---

### Purposes

- To persist tokens across requests and page reloads.
- To balance protection against XSS and CSRF threats.
- To comply with security best practices for browser-based applications.
- To avoid exposing tokens to client-side scripts when possible.

---

### Syntax Rules and Structure

#### Complete General Syntax: Secure Cookie Storage

```javascript
app.post('/login', (req, res) => {
  // ... verify credentials ...
  const accessToken = jwt.sign({ sub: user.id }, SECRET, { expiresIn: '15m' });

  res.cookie('access_token', accessToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 15 * 60 * 1000,
    path: '/'
  });

  res.json({ message: 'Logged in' });
});
```

**Component Breakdown**

| Cookie Attribute | Security Effect |
|------------------|-----------------|
| `httpOnly` | Blocks JavaScript access; mitigates XSS theft |
| `secure` | Restricts to HTTPS |
| `sameSite: 'strict'` | Mitigates CSRF |
| `maxAge` | Aligns cookie lifetime with token expiration |

#### Syntax Rules

- **Never** store sensitive tokens in `localStorage` or `sessionStorage`; they are XSS-exfiltratable.
- Use `HttpOnly`, `Secure`, and `SameSite` cookies for browser-based token storage.
- Pair cookie storage with CSRF defenses (anti-CSRF tokens or `SameSite`).
- For mobile apps, use platform-specific secure storage (Keychain, Keystore).

#### Constraints and Limitations

- `HttpOnly` cookies are not accessible to JavaScript, so SPAs cannot read the token to attach it to headers; the cookie is sent automatically.
- `SameSite=Strict` can break cross-site flows such as OAuth redirects.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Setting a Secure `HttpOnly` Cookie

```javascript
app.post('/login', (req, res) => {
  const accessToken = jwt.sign({ sub: user.id }, SECRET, { expiresIn: '15m' });

  res.cookie('access_token', accessToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 15 * 60 * 1000
  });

  res.json({ message: 'Token stored in HttpOnly cookie' });
});
```

**Expected Output**

```
Set-Cookie: access_token=eyJ...; HttpOnly; Secure; SameSite=Strict; Max-Age=900; Path=/
```

**Why This Output Occurs**

- The cookie attributes enforce `HttpOnly`, `Secure`, and `SameSite`, protecting the token from JavaScript and cross-site transmission.

---

### Real-World Cases

- **SPAs with backend-for-frontend** — Tokens held server-side; browser receives only a session cookie.
- **Traditional web applications** — Server-rendered pages use `HttpOnly` session cookies.
- **Mobile apps** — Tokens stored in iOS Keychain or Android Keystore.

---

## Core Concept 7: Token Security

### Definitions

**Core Definition**
Token security is the set of controls that protect tokens from forgery, theft, replay, algorithm confusion, and information leakage throughout their lifecycle.

**Technical Definition**
Token security encompasses: **algorithm pinning** (explicit `algorithms` allowlist in `jwt.verify()`), **rejection of the `none` algorithm**, **short access-token lifetimes**, **refresh-token rotation**, **HTTPS-only transport**, **secure storage**, **avoiding sensitive data in payloads**, **key rotation**, and **rate limiting** on token-issuing endpoints. Historical `jsonwebtoken` vulnerabilities—CVE-2015-9235 (RS256→HS256 confusion), CVE-2022-23540 (`none` fallback), and CVE-2022-23541 (unsafe key callbacks)—were all rooted in trusting the token header’s algorithm instead of the server’s configuration; v9.0.0 closes these when an explicit allowlist is used. OWASP recommends `HttpOnly` + `Secure` + `SameSite` cookies, never `localStorage`, HTTPS-only, and signing-key rotation.

**Beginner-Friendly Explanation**
Token security is about making sure nobody can forge your wristband, steal it, or use it after it should have expired. The most important rule: the server decides which stamp to check, not the wristband. If the wristband says “no stamp needed,” the server must ignore that and check anyway.

---

### Purposes

- To prevent token forgery through algorithm confusion and `none`-algorithm attacks.
- To limit the damage of token theft through short expiration and rotation.
- To protect tokens from XSS and CSRF through secure storage.
- To ensure tokens are transmitted only over encrypted channels.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Verification

```javascript
jwt.verify(token, publicKey, {
  algorithms: ['RS256'],          // explicit allowlist
  issuer: 'https://auth.example.com',
  audience: 'api.example.com',
  clockTolerance: 5                // seconds
}, (err, payload) => {
  if (err) { /* reject */ }
  // trusted payload
});
```

**Component Breakdown**

| Control | Implementation |
|---------|----------------|
| Algorithm allowlist | `algorithms: ['RS256']` |
| Reject `none` | Never include `'none'` in the allowlist |
| Issuer/audience | `issuer`, `audience` options |
| Clock tolerance | `clockTolerance` for minor clock skew |

#### Syntax Rules

- **Always** pass an explicit `algorithms` allowlist to `jwt.verify()`.
- **Never** allow the `none` algorithm.
- Use **RS256 or ES256** in distributed systems; avoid sharing HMAC secrets across services.
- Serve tokens only over HTTPS.
- Rotate signing keys and maintain separate key sets per algorithm family.

#### Constraints and Limitations

- HS256 secrets must be at least 32 bytes and high-entropy; short secrets are brute-forceable.
- Algorithm confusion is only prevented when the allowlist is explicit; omitting it reintroduces the vulnerability.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Rejecting Algorithm Confusion

```javascript
const jwt = require('jsonwebtoken');

// Server expects RS256 but attacker sends an HS256 token
const maliciousToken = jwt.sign({ sub: 'attacker' }, publicKey, { algorithm: 'HS256' });

try {
  jwt.verify(maliciousToken, publicKey, { algorithms: ['RS256'] });
} catch (err) {
  console.log(err.message); // "invalid algorithm"
}
```

**Expected Output**

```
invalid algorithm
```

**Why This Output Occurs**

- The `algorithms: ['RS256']` allowlist rejects the HS256 token even though the signature would otherwise validate with the public key used as an HMAC secret.

---

### Real-World Cases

- **Financial APIs** — Algorithm pinning and short-lived tokens are mandatory.
- **Multi-service architectures** — Asymmetric keys prevent secret sharing and algorithm confusion.
- **Public-facing APIs** — HTTPS-only and `HttpOnly` cookies protect tokens in transit and at rest.

---

## Core Concept 8: Token Authentication Best Practices

### Definitions

**Core Definition**
Token authentication best practices are the design, implementation, and operational patterns that keep token-based authentication secure, scalable, and maintainable across the token lifecycle.

**Technical Definition**
Best practices include: issuing **short-lived access tokens** (≤15 minutes) and **long-lived refresh tokens** with **rotation and reuse detection**; pinning the verification algorithm with an explicit allowlist and rejecting `none`; storing tokens in `HttpOnly`, `Secure`, `SameSite` cookies (not `localStorage`); transmitting tokens only over HTTPS; using asymmetric algorithms (RS256/ES256) in distributed systems; keeping sensitive data out of JWT payloads; rate-limiting login and token-refresh endpoints; rotating signing keys; and implementing server-side revocation or denylists for logout and compromise scenarios.

**Beginner-Friendly Explanation**
Best practices are the habits that keep token authentication safe. Use short-lived passes, use a secure locker for the long-lived season pass, always check the stamp, never let the wristband choose the stamp, and always use the encrypted entrance.

---

### Purposes

- To minimize the attack window through short access-token lifetimes.
- To prevent algorithm-confusion and `none`-algorithm attacks.
- To protect tokens from XSS and CSRF through correct storage.
- To enable session revocation and refresh-token replay detection.
- To maintain observability and rate-limit abuse.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Token Lifecycle

```javascript
// 1. Issue short-lived access token
const accessToken = jwt.sign(
  { sub: user.id, type: 'access' },
  process.env.JWT_SECRET,
  { expiresIn: '15m', algorithm: 'HS256', issuer: 'my-api', audience: 'my-client' }
);

// 2. Issue refresh token with rotation ID
const refreshToken = jwt.sign(
  { sub: user.id, type: 'refresh', jti: crypto.randomUUID() },
  process.env.REFRESH_SECRET,
  { expiresIn: '7d', algorithm: 'HS256' }
);

// 3. Verify with algorithm allowlist
jwt.verify(token, SECRET, { algorithms: ['HS256'], issuer: 'my-api', audience: 'my-client' });

// 4. Store in HttpOnly cookie
res.cookie('access_token', accessToken, { httpOnly: true, secure: true, sameSite: 'strict' });
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Short access tokens | `expiresIn: '15m'` |
| Refresh rotation | `jti` + server-side store |
| Algorithm pinning | `algorithms: ['HS256']` |
| Secure storage | `HttpOnly` + `Secure` + `SameSite` |
| Type claim | `type: 'access'` / `'refresh'` to prevent cross-use |

#### Syntax Rules

- **Access tokens** expire in ≤15 minutes; **refresh tokens** are long-lived and rotated.
- **Algorithm allowlist** is mandatory in every `jwt.verify()` call.
- **Never** store tokens in `localStorage`; use `HttpOnly` cookies or server-side storage.
- **HTTPS-only** in production.
- **Rate-limit** `/login` and `/token/refresh` to slow brute force.
- **Separate secrets/keys** for access and refresh tokens.

#### Constraints and Limitations

- Refresh-token rotation requires persistent storage and adds complexity.
- Stateless JWTs cannot be revoked instantly; short expiry is the primary mitigation.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Hardened Token Pipeline

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const app = express();
app.use(express.json());

const ACCESS_SECRET = process.env.JWT_ACCESS_SECRET;
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;
const refreshStore = new Map(); // jti -> userId

// Login: issue short access + rotating refresh
app.post('/login', (req, res) => {
  const user = authenticate(req.body);
  const accessToken = jwt.sign({ sub: user.id, type: 'access' }, ACCESS_SECRET, { expiresIn: '15m', algorithm: 'HS256' });
  const jti = crypto.randomUUID();
  refreshStore.set(jti, user.id);
  const refreshToken = jwt.sign({ sub: user.id, type: 'refresh', jti }, REFRESH_SECRET, { expiresIn: '7d', algorithm: 'HS256' });

  res.cookie('access_token', accessToken, { httpOnly: true, secure: true, sameSite: 'strict', maxAge: 900000 });
  res.cookie('refresh_token', refreshToken, { httpOnly: true, secure: true, sameSite: 'strict', maxAge: 604800000 });
  res.json({ message: 'Logged in' });
});

// Refresh: rotate and detect reuse
app.post('/refresh', (req, res) => {
  const { refresh_token } = req.cookies;
  const payload = jwt.verify(refresh_token, REFRESH_SECRET, { algorithms: ['HS256'] });
  if (!refreshStore.has(payload.jti)) return res.status(401).json({ error: 'Revoked' });
  refreshStore.delete(payload.jti); // rotation
  const newJti = crypto.randomUUID();
  refreshStore.set(newJti, payload.sub);
  const newAccess = jwt.sign({ sub: payload.sub, type: 'access' }, ACCESS_SECRET, { expiresIn: '15m' });
  const newRefresh = jwt.sign({ sub: payload.sub, type: 'refresh', jti: newJti }, REFRESH_SECRET, { expiresIn: '7d' });
  res.cookie('access_token', newAccess, { httpOnly: true, secure: true, sameSite: 'strict' });
  res.cookie('refresh_token', newRefresh, { httpOnly: true, secure: true, sameSite: 'strict' });
  res.json({ message: 'Tokens rotated' });
});
```

**Expected Output**

```
POST /login  (valid credentials)
→ 200 { "message": "Logged in" }
Set-Cookie: access_token=...; HttpOnly; Secure; SameSite=Strict
Set-Cookie: refresh_token=...; HttpOnly; Secure; SameSite=Strict

POST /refresh  (valid refresh cookie)
→ 200 { "message": "Tokens rotated" }
```

**Why This Output Occurs**

- Access tokens are short-lived; refresh tokens are rotated and tracked server-side; both are stored in hardened cookies.

---

### Real-World Cases

- **Production APIs** — Short access tokens, rotating refresh tokens, algorithm pinning, and HTTPS.
- **Financial services** — Asymmetric keys, key rotation, and strict algorithm allowlists.
- **Multi-tenant SaaS** — Server-side refresh-token stores enable revocation per tenant or user.

---

## References

- JWT Authentication for Secure Node.js APIs (Sourcetrail, 2026) — https://www.sourcetrail.com/javascript/implement-jwt-authentication-in-node-js-apis-like-a-pro/
- Autenticazione JWT in Node.js: 12 Step (Shattered, 2026) — https://shattered.io/it/autenticazione-jwt-nodejs/
- `jsonwebtoken` npm Documentation (v9.0.3) — https://www.npmjs.com/package/jsonwebtoken
- JWT Security Resources (transilienceai/communitytools) — https://github.com/transilienceai/communitytools/blob/main/skills/authentication/reference/jwt_security_resources.md
- Single-Page Application (SPA) Security Guide (Safeguard, 2026) — https://safeguard.sh/resources/blog/single-page-application-security
- JWT Security: Common Flaws and How to Fix Them (Safeguard, 2026) — https://safeguard.sh/resources/blog/jwt-security-vulnerabilities-and-best-practices
- OWASP JWT Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- OWASP OAuth 2.0 Protocol Cheatsheet (refresh token rotation) — https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html
- RFC 7519: JSON Web Token (JWT) — https://datatracker.ietf.org/doc/html/rfc7519
- RFC 8725: JWT Best Current Practices — https://datatracker.ietf.org/doc/html/rfc8725
- CVE-2022-23540 / CVE-2022-23541 (`jsonwebtoken` algorithm issues) — https://github.com/advisories/GHSA-qwph-4952-7xr6