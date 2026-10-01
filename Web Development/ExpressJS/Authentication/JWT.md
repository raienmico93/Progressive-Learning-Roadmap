# JWT Authentication (Stateless / Hybrid) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** JSON Web Token (JWT) authentication is a stateless authentication mechanism in which the server issues a cryptographically signed token containing user claims, which the client presents on every subsequent request; the server verifies the token's signature and claims without maintaining any server-side session state.

**Technical Definition:** A JWT is a compact, URL-safe string composed of three Base64URL-encoded parts separated by dots: `header.payload.signature`. The header declares the token type and signing algorithm; the payload contains claims (registered claims such as `iss`, `sub`, `exp`, `iat`, `jti`, plus custom claims); and the signature is computed over the first two parts using a secret (HMAC) or private key (RSA/ECDSA/EdDSA). The server verifies the signature and validates claims on every request. In a hybrid model, a short-lived access token (5–15 minutes) is paired with a long-lived refresh token (7–30 days) to balance statelessness with revocability.

**Beginner-Friendly Explanation:** A JWT is like a stamped passport. When you log in (apply for the passport), the server stamps it with a signature that's hard to forge. You carry the passport with you (the token), and every time you cross a border (make a request), the officer checks the stamp. If the stamp is valid and the passport hasn't expired, you're allowed through. The passport doesn't require the officer to call headquarters — the stamp is self-contained.

### Key Characteristics

- **Stateless:** The server stores no session data; the token contains all necessary information.
- **Self-contained:** Claims are embedded in the token and readable by the client.
- **Signed, not encrypted:** The payload is Base64URL-encoded, not encrypted — anyone can read it. Never put sensitive data in the payload.
- **Short-lived access tokens:** Typically 5–15 minutes, limiting the window of opportunity for token theft.
- **Refresh token rotation:** Each refresh token is single-use and replaced on each refresh, enabling theft detection.
- **Hybrid revocability:** Stateless access tokens combined with server-side refresh token tracking and Redis-backed denylists.

### Prerequisites

- **Node.js runtime** (v18 or higher; `crypto` key generation is stable).
- **Express.js installed:** `npm install express`.
- **A JWT library:** `npm install jsonwebtoken` (v9.0.3 is the current stable release, with no known vulnerabilities).
- **A Redis client:** `npm install ioredis` (for refresh token tracking and denylists).
- **Basic JavaScript knowledge:** Async/await, Promises, and middleware concepts.

### Related Programming Areas

- **Session-Based Authentication:** The stateful alternative to JWT.
- **Password Authentication:** The credential verification step that precedes token issuance.
- **Authorisation:** Role and permission checks that follow authentication.
- **OAuth 2.1:** The standardised framework that uses JWT as its token format.
- **Token Storage Security:** Cookie flags, HttpOnly, and Secure storage for web clients.

### Core Concepts

1. **Access Tokens** — short-lived, signed JWTs containing safe user claims.
2. **Refresh Tokens** — long-lived tokens stored securely to obtain new access tokens.
3. **Token Expiration & Verification** — validating `exp`, `iss`, and signatures on every request.
4. **Token Rotation (RTR)** — automatic refresh token rotation with token family reuse detection.
5. **Revocation Strategies** — Redis-backed denylists and Bloom filters for active revocation.
6. **Token Storage Security** — HttpOnly cookies vs. localStorage, and XSS/CSRF trade-offs.

---

## Core Concept 1: Access Tokens

### Definitions

**Core Definition:** An access token is a short-lived, cryptographically signed JWT that the client presents on every API request to prove its identity and authorisation to access protected resources.

**Technical Definition:** Access tokens are JWTs signed using a signing algorithm — either symmetric HMAC (HS256) or asymmetric (RS256, ES256, EdDSA) — and contain registered claims (`iss`, `sub`, `exp`, `iat`, `jti`) plus custom claims (user ID, roles, permissions). They are intentionally short-lived (5–15 minutes) to limit the damage from token theft. The signing algorithm must be validated against an allow-list on the server to prevent algorithm confusion attacks. The payload is Base64URL-encoded, not encrypted, so no sensitive data (passwords, credit card numbers) should be included.

**Beginner-Friendly Explanation:** An access token is like a temporary visitor badge. It gets you into the building (protected API), but it expires quickly. If someone steals it, they only have a few minutes to use it before it becomes useless. The badge contains only your name and role — no sensitive information.

### Purposes

- To emit short-lived JSON Web Tokens signed using modern cryptographic standard algorithms (e.g., asymmetric RS256/EdDSA or symmetric HS256).
- To contain safe user claims (user ID, roles, permissions) for stateless authorisation.
- To enable stateless verification without database lookups on every request.
- To limit the damage window from token theft through short expiration times.

### Syntax Rules and Structure

#### Signing with HMAC (HS256)

```javascript
const jwt = require('jsonwebtoken');

const accessToken = jwt.sign(
  { userId: user.id, role: user.role },      // Payload (claims)
  process.env.JWT_SECRET,                     // Secret key
  {
    algorithm: 'HS256',
    expiresIn: '15m',                         // Short-lived
    issuer: 'myapp.example.com',              // iss claim
    audience: 'myapp-api',                    // aud claim
    jwtid: crypto.randomUUID()                // jti claim
  }
);
```

#### Signing with Asymmetric Keys (RS256/EdDSA)

```javascript
const fs = require('fs');
const jwt = require('jsonwebtoken');

// RS256 — RSA private key signs, public key verifies
const privateKey = fs.readFileSync('private.pem');
const publicKey = fs.readFileSync('public.pem');

const accessToken = jwt.sign(
  { userId: user.id, role: user.role },
  privateKey,
  { algorithm: 'RS256', expiresIn: '15m' }
);

// Verification with the public key
const decoded = jwt.verify(accessToken, publicKey, {
  algorithms: ['RS256']                      // Allow-list
});
```

| Algorithm | Key Type | Best For |
|-----------|----------|----------|
| HS256 | Symmetric secret | Single-service, internal APIs |
| RS256 | RSA key pair | Multi-service, distributed verification |
| ES256 | ECDSA key pair | Smaller keys than RSA, modern |
| EdDSA (Ed25519) | Ed25519 key pair | Fastest, smallest keys, timing-attack immune |

**Rules:**
- Use `expiresIn: '15m'` for access tokens — never longer than 15 minutes.
- Always validate the algorithm against an allow-list (`algorithms: ['RS256']`) — never trust the token's `alg` header.
- Never include sensitive data (passwords, credit card numbers) in the payload — it is readable by anyone who intercepts the token.
- Use asymmetric algorithms (RS256/EdDSA) when multiple services need to verify tokens independently.
- Include the `iss` and `aud` claims for additional validation.

**Constraints:**
- HS256 uses the same secret for signing and verification — any service that verifies can also sign.
- RSA keys are larger and slower than EdDSA (Ed25519).
- EdDSA (Ed25519) is supported natively in Node.js 18+ without external dependencies.

### Annotated Code Example

```javascript
// access-token.js
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

function generateAccessToken(user) {
  return jwt.sign(
    {
      sub: user.id,                     // Subject — user identifier
      role: user.role,                  // Custom claim — authorisation
      email: user.email                 // Custom claim — display only
    },
    process.env.JWT_SECRET,
    {
      algorithm: 'HS256',
      expiresIn: '15m',                  // 15-minute lifetime
      issuer: 'myapp.example.com',
      audience: 'myapp-api',
      jwtid: crypto.randomUUID()        // Unique token identifier
    }
  );
}

// Verification with algorithm allow-list
function verifyAccessToken(token) {
  try {
    return jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ['HS256'],
      issuer: 'myapp.example.com',
      audience: 'myapp-api'
    });
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      throw new Error('Access token expired');
    }
    if (err.name === 'JsonWebTokenError') {
      throw new Error('Invalid access token');
    }
    throw err;
  }
}
```

**Expected Output (decoded token):**
```json
{
  "sub": "usr_123",
  "role": "admin",
  "email": "alice@example.com",
  "iss": "myapp.example.com",
  "aud": "myapp-api",
  "exp": 1781172000,
  "iat": 1781171100,
  "jti": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Why this output:** The token contains the user's ID (`sub`), role (`role`), and email, along with registered claims. The signature guarantees integrity — any tampering invalidates the token. The 15-minute expiration ensures that stolen tokens become useless quickly.

### Real-World Cases

- **Microservices:** Access tokens verified independently by each service using the public key.
- **SPAs:** Short-lived access tokens refreshed silently by the client.
- **Mobile apps:** Access tokens stored in platform-secure storage (Keychain/Keystore).
- **API gateways:** Access tokens verified at the edge before forwarding to internal services.

---

## Core Concept 2: Refresh Tokens

### Definitions

**Core Definition:** A refresh token is a long-lived, cryptographically secure credential that allows the client to obtain a new access token without requiring the user to re-authenticate, and which is stored and tracked server-side to enable revocation.

**Technical Definition:** Unlike access tokens, refresh tokens are not JWTs that the server can verify statelessly — they are opaque, cryptographically random strings (typically 64+ bytes) stored in a database or Redis with the user ID, expiration, and a token family identifier. The refresh token is issued at login alongside the access token and stored client-side in a secure HttpOnly cookie. When the access token expires, the client sends the refresh token to a dedicated endpoint, which validates it against the store, revokes it, and issues a new access token and a new refresh token (rotation). Refresh tokens are hashed (SHA-256) before storage — plaintext refresh tokens are never stored.

**Beginner-Friendly Explanation:** A refresh token is like a long-term membership card. When your day pass (access token) expires, you show your membership card to get a new day pass — without having to fill out the entire registration form again. The membership card is kept securely at the front desk (server-side store), and if you lose it, it can be deactivated.

### Purposes

- To issue long-lived tokens stored securely to request fresh access tokens without requiring user re-authentication.
- To enable refresh token rotation — each refresh token is single-use and replaced on each refresh.
- To support token family tracking for reuse detection and theft mitigation.
- To store refresh tokens as hashes (SHA-256), never plaintext.

### Syntax Rules and Structure

#### Refresh Token Schema (Mongoose)

```javascript
const mongoose = require('mongoose');

const refreshTokenSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    index: true
  },
  tokenHash: {
    type: String,
    required: true
  },
  family: {
    type: String,
    required: true,
    index: true
  },
  expiresAt: {
    type: Date,
    required: true,
    index: { expires: 0 }                    // TTL index
  },
  revoked: {
    type: Boolean,
    default: false
  }
});
```

#### Issuing Refresh Tokens at Login

```javascript
const crypto = require('crypto');

async function issueRefreshToken(userId) {
  // Cryptographically secure random token (64 bytes)
  const plainToken = crypto.randomBytes(64).toString('hex');

  // Hash before storage — SHA-256 is sufficient for high-entropy tokens
  const tokenHash = crypto.createHash('sha256').update(plainToken).digest('hex');

  // Generate a family ID (one per login session)
  const family = crypto.randomUUID();

  await RefreshToken.create({
    userId,
    tokenHash,
    family,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
  });

  return plainToken; // Sent to client in HttpOnly cookie
}
```

| Component | Breakdown |
|-----------|-----------|
| `crypto.randomBytes(64)` | 512-bit cryptographically secure random token. |
| `sha256(token)` | Hash before storage — plaintext never stored. |
| `family` | Groups all tokens from one login session. |
| `expiresAt` | TTL for automatic cleanup. |

**Rules:**
- Generate refresh tokens with `crypto.randomBytes(64)` — minimum 256 bits of entropy.
- Store **only the SHA-256 hash** of the refresh token — never the plaintext.
- Track a `family` identifier for reuse detection.
- Set a TTL index on `expiresAt` for automatic cleanup.
- Send refresh tokens in **HttpOnly, Secure, SameSite** cookies — never in the response body.

**Constraints:**
- SHA-256 is sufficient for high-entropy tokens (unlike passwords, which require Argon2id).
- Refresh tokens are bearer credentials — possession is sufficient for access.
- Refresh token storage must be shared across server instances (Redis or database).

### Annotated Code Example

```javascript
// refresh-token.js
const crypto = require('crypto');
const RefreshToken = require('../models/RefreshToken');

async function generateRefreshToken(userId) {
  // 1. Generate cryptographically secure token
  const plainToken = crypto.randomBytes(64).toString('hex');

  // 2. Hash for storage (SHA-256 — fast and appropriate for high-entropy tokens)
  const tokenHash = crypto.createHash('sha256').update(plainToken).digest('hex');

  // 3. Generate family ID for reuse detection
  const family = crypto.randomUUID();

  // 4. Store in database with expiry
  await RefreshToken.create({
    userId,
    tokenHash,
    family,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  });

  // 5. Return plaintext (sent to client in HttpOnly cookie)
  return plainToken;
}

async function verifyRefreshToken(plainToken) {
  const tokenHash = crypto.createHash('sha256').update(plainToken).digest('hex');

  const stored = await RefreshToken.findOne({
    tokenHash,
    revoked: false,
    expiresAt: { $gt: new Date() }
  });

  return stored; // null if not found, expired, or revoked
}
```

**Expected Output (stored record):**
```json
{
  "userId": "usr_123",
  "tokenHash": "a1b2c3d4e5f6...",
  "family": "550e8400-e29b-41d4-a716-446655440000",
  "expiresAt": "2026-01-22T10:30:00.000Z",
  "revoked": false
}
```

**Why this output:** The plaintext refresh token is never stored — only its SHA-256 hash. The `family` identifier groups all refresh tokens from the same login session. The `expiresAt` TTL index ensures automatic deletion after 7 days.

### Real-World Cases

- **SPAs:** Refresh tokens in HttpOnly cookies; access tokens in memory.
- **Mobile apps:** Refresh tokens in Keychain (iOS) or Keystore (Android).
- **Multi-device sessions:** Each device gets its own token family.
- **Long-lived sessions:** Users stay logged in for days without re-entering credentials.

---

## Core Concept 3: Token Expiration & Verification

### Definitions

**Core Definition:** Token expiration and verification is the process of validating that a JWT has not expired (`exp`), was issued by a trusted issuer (`iss`), is intended for this audience (`aud`), and carries a valid cryptographic signature — all on every incoming request.

**Technical Definition:** The `jsonwebtoken` library's `verify()` method performs signature verification and automatically validates the `exp` and `nbf` (not before) claims. Additional validation includes `issuer`, `audience`, and `algorithms` allow-listing. The `clockTolerance` option accommodates minor clock skew between servers. Verification failures throw specific error types: `TokenExpiredError` (expired), `JsonWebTokenError` (invalid signature or malformed), and `NotBeforeError` (not yet valid). The server must handle each error type and respond with a 401 status. The algorithm must be validated against an allow-list — never trust the token's `alg` header.

**Beginner-Friendly Explanation:** Token verification is like a bouncer checking a passport. The bouncer checks the expiration date (is the passport still valid?), the issuing country (is it from a trusted country?), and the security features (is the passport forged?). If any check fails, you're turned away.

### Purposes

- To validate expiration bounds (`exp`), issuer claims (`iss`), and cryptographic signatures on every incoming API request.
- To prevent algorithm confusion attacks by allow-listing algorithms.
- To handle clock skew between distributed servers with `clockTolerance`.
- To distinguish between expired tokens (client should refresh) and invalid tokens (client should re-authenticate).

### Syntax Rules and Structure

```javascript
const jwt = require('jsonwebtoken');

function verifyAccessToken(token) {
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ['HS256'],                    // Allow-list
      issuer: 'myapp.example.com',              // Validate iss
      audience: 'myapp-api',                    // Validate aud
      clockTolerance: 30                        // 30 seconds skew
    });
    return { valid: true, payload: decoded };
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return { valid: false, reason: 'expired', expiredAt: err.expiredAt };
    }
    if (err.name === 'JsonWebTokenError') {
      return { valid: false, reason: 'invalid' };
    }
    if (err.name === 'NotBeforeError') {
      return { valid: false, reason: 'not_active', date: err.date };
    }
    throw err;
  }
}
```

| Claim | Purpose | Validation |
|-------|---------|------------|
| `exp` | Expiration time | Automatically validated by `verify()` |
| `nbf` | Not before | Automatically validated |
| `iss` | Issuer | Pass `issuer` option |
| `aud` | Audience | Pass `audience` option |
| `jti` | JWT ID | Manual check against denylist |

**Rules:**
- Always pass `algorithms` as an array to `jwt.verify()` — never omit it.
- Validate `issuer` and `audience` when tokens are issued by multiple services.
- Set `clockTolerance` to 30–60 seconds for distributed systems.
- Handle `TokenExpiredError` by triggering a token refresh, not by forcing re-login.
- Never use `jwt.decode()` as a substitute for `jwt.verify()` — `decode` does not verify the signature.

**Constraints:**
- `jsonwebtoken` v9 rejects tokens with `alg: none` by default.
- Clock skew beyond `clockTolerance` causes false expiration errors.
- Verification is CPU-intensive for RS256 — consider caching public keys (JWKS).

### Annotated Code Example

```javascript
// token-verification.js
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ['HS256'],
      issuer: 'myapp.example.com',
      audience: 'myapp-api',
      clockTolerance: 30
    });

    req.user = { id: decoded.sub, role: decoded.role };
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Token expired',
        code: 'TOKEN_EXPIRED',
        expiredAt: err.expiredAt
      });
    }

    return res.status(401).json({
      error: 'Invalid token',
      code: 'TOKEN_INVALID'
    });
  }
}
```

**Expected Output (for an expired token):**
```json
{
  "error": "Token expired",
  "code": "TOKEN_EXPIRED",
  "expiredAt": "2026-01-15T10:30:00.000Z"
}
```

**Expected Output (for a tampered token):**
```json
{
  "error": "Invalid token",
  "code": "TOKEN_INVALID"
}
```

**Why this output:** The `verify()` method checks the signature, `exp`, `iss`, and `aud`. An expired token throws `TokenExpiredError` — the client should refresh. A tampered token throws `JsonWebTokenError` — the client must re-authenticate.

### Real-World Cases

- **API middleware:** Every protected route uses the same verification middleware.
- **Microservices:** Each service verifies tokens independently using a shared public key.
- **Token refresh endpoints:** The refresh endpoint verifies the refresh token before issuing new tokens.
- **Multi-tenant APIs:** The `aud` claim scopes tokens to specific tenants.

---

## Core Concept 4: Token Rotation (RTR) & Token Families

### Definitions

**Core Definition:** Refresh Token Rotation (RTR) is a security mechanism in which each refresh token is single-use and replaced with a new refresh token on every refresh, with token families tracking the lineage to detect and respond to token theft.

**Technical Definition:** In RTR, a token family is created at login. Each rotation within that family updates a single pointer in Redis (or the database) indicating the currently valid token ID. When a refresh request arrives: if the presented token ID matches the family's current pointer, it is a legitimate rotation — issue a new token ID and update the pointer. If it does not match, the token was already superseded — this signals reuse, which is treated as theft. The safe response is to delete the entire family, revoking all tokens for that user's session lineage. The Redis footprint stays constant per session regardless of how many times it refreshes, as only the current valid token ID is stored. Overlap periods can account for leeway time between request and response before triggering automatic reuse detection.

**Beginner-Friendly Explanation:** RTR is like a security system that changes the locks every time you use your key. If someone copies your key (steals your refresh token) and tries to use it later, the lock has already been changed — and the system knows someone tried to use an old key. It immediately changes all the locks in the building (revokes the family), protecting your account.

### Purposes

- To implement automatic Refresh Token Rotation (RTR) with token family tracking.
- To detect token reuse anomalies and invalidate the entire token family proactively.
- To stop token theft by ensuring stolen refresh tokens become useless after one use.
- To maintain a constant Redis footprint per session regardless of rotation count.

### Syntax Rules and Structure

#### Redis Token Family Storage

```javascript
// Store the current valid token ID for a family
async function storeTokenFamily(familyId, tokenId, userId, ttl) {
  await redis.setex(
    `family:${familyId}`,
    ttl,
    JSON.stringify({ tokenId, userId })
  );
}

// Retrieve the current valid token for a family
async function getTokenFamily(familyId) {
  const data = await redis.get(`family:${familyId}`);
  return data ? JSON.parse(data) : null;
}
```

#### Rotation with Reuse Detection

```javascript
async function rotateRefreshToken(presentedTokenId, familyId, userId) {
  const current = await getTokenFamily(familyId);

  if (!current) {
    throw new Error('Token family not found — session expired');
  }

  if (current.tokenId !== presentedTokenId) {
    // REUSE DETECTED: presented token is not the current one
    await redis.del(`family:${familyId}`);  // Revoke entire family
    throw new Error('Refresh token reuse detected — all sessions revoked');
  }

  // Legitimate rotation — generate new token and update pointer
  const newTokenId = crypto.randomUUID();
  await storeTokenFamily(familyId, newTokenId, userId, 7 * 24 * 60 * 60);

  return newTokenId;
}
```

| Component | Breakdown |
|-----------|-----------|
| `family:${familyId}` | Redis key for the token family pointer. |
| `current.tokenId` | The currently valid token ID. |
| `presentedTokenId` | The token ID from the client's request. |
| `redis.del()` | Revokes the entire family on reuse detection. |

**Rules:**
- Every refresh must issue a **new** refresh token and invalidate the old one.
- The token family is created at login and persists across rotations.
- On reuse detection, revoke the **entire family** — not just the reused token.
- Set the Redis key TTL to match the refresh token's lifetime for automatic cleanup.
- Implement an overlap period (e.g., 10 seconds) to handle concurrent refresh requests from the same client.

**Constraints:**
- Concurrent refresh requests (e.g., from multiple tabs) can trigger false reuse detection — use an overlap period.
- The family pointer approach requires atomic Redis operations to prevent race conditions.
- Mobile clients with flaky networks may retry refresh requests — the overlap period handles this.

### Annotated Code Example

```javascript
// refresh-rotation.js
const crypto = require('crypto');
const redis = require('../redis');
const { generateAccessToken } = require('./access-token');

async function refreshTokens(presentedRefreshToken) {
  // 1. Hash the presented token to find the family
  const tokenHash = crypto.createHash('sha256')
    .update(presentedRefreshToken)
    .digest('hex');

  // 2. Look up the token record to get its family and ID
  const record = await RefreshToken.findOne({ tokenHash, revoked: false });

  if (!record) {
    // Token not found or already revoked — possible theft
    throw new Error('Invalid refresh token');
  }

  // 3. Check the family pointer in Redis
  const current = await getTokenFamily(record.family);

  if (!current) {
    // Family expired or revoked
    throw new Error('Session expired');
  }

  if (current.tokenId !== record._id.toString()) {
    // REUSE DETECTED — revoke entire family
    await redis.del(`family:${record.family}`);
    await RefreshToken.updateMany(
      { family: record.family },
      { revoked: true }
    );
    throw new Error('Refresh token reuse detected — all sessions revoked');
  }

  // 4. Legitimate rotation
  const newPlainToken = crypto.randomBytes(64).toString('hex');
  const newTokenHash = crypto.createHash('sha256')
    .update(newPlainToken)
    .digest('hex');

  const newRecord = await RefreshToken.create({
    userId: record.userId,
    tokenHash: newTokenHash,
    family: record.family,                     // Same family
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  });

  // Update the family pointer
  await storeTokenFamily(
    record.family,
    newRecord._id.toString(),
    record.userId,
    7 * 24 * 60 * 60
  );

  // 5. Revoke the old token
  record.revoked = true;
  await record.save();

  // 6. Issue new access token
  const accessToken = generateAccessToken({ id: record.userId });

  return { accessToken, refreshToken: newPlainToken };
}
```

**Expected Output (for a legitimate refresh):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "a1b2c3d4e5f6..."
}
```

**Expected Output (for a reused token):**
```json
{
  "error": "Refresh token reuse detected — all sessions revoked"
}
```

**Why this output:** The legitimate rotation updates the family pointer and issues new tokens. The reused token does not match the family pointer — the system detects theft, revokes the entire family, and returns an error. The attacker's stolen token is now useless.

### Real-World Cases

- **Banking apps:** RTR with strict reuse detection for high-value accounts.
- **OAuth 2.1:** RTR is recommended for all OAuth 2.1 public clients.
- **Multi-tab SPAs:** Overlap periods handle concurrent refresh requests.
- **Mobile apps:** Retry logic with overlap periods accommodates flaky networks.

---

## Core Concept 5: Revocation Strategies

### Definitions

**Core Definition:** Revocation strategies are the mechanisms by which a stateless JWT — which remains valid until its `exp` claim — can be invalidated before its natural expiration, using server-side storage such as Redis-backed denylists or Bloom filters.

**Technical Definition:** JWT revocation is challenging because verification is stateless. The standard approach is a **denylist** (blacklist) stored in Redis: when a token must be revoked (logout, password change, theft detection), its `jti` (JWT ID) is added to the denylist with a TTL equal to the token's remaining lifetime. On every request, the server checks the denylist before accepting the token. Redis's `SET` with `EX` provides automatic cleanup — revoked entries expire when the token would have expired anyway. **Bloom filters** offer a memory-efficient alternative for large-scale denylists: they can definitively say "not revoked" (no false negatives) but may have false positives (a valid token may be flagged as revoked). For most applications, the Redis denylist with TTL is simpler and sufficient.

**Beginner-Friendly Explanation:** A denylist is like a list of cancelled tickets at a venue. When a ticket is cancelled, its number goes on the list. At the door, the staff checks the list before letting anyone in. The list automatically clears itself when the ticket would have expired anyway.

### Purposes

- To handle stateless token invalidation challenges by employing hybrid storage techniques like Redis-backed denylists or Bloom filters.
- To revoke access tokens immediately on logout, password change, or theft detection.
- To use Redis TTL for automatic cleanup of expired denylist entries.
- To use Bloom filters for memory-efficient revocation at scale.

### Syntax Rules and Structure

#### Redis Denylist (Recommended)

```javascript
const redis = require('../redis');

async function revokeToken(jti, expiresAt) {
  const ttl = Math.floor((expiresAt * 1000 - Date.now()) / 1000);
  if (ttl > 0) {
    await redis.setex(`denylist:${jti}`, ttl, '1');
  }
}

async function isTokenRevoked(jti) {
  return (await redis.exists(`denylist:${jti}`)) === 1;
}
```

#### Middleware Check

```javascript
async function authenticateToken(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Authentication required' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ['HS256']
    });

    // Check denylist
    if (await isTokenRevoked(decoded.jti)) {
      return res.status(401).json({ error: 'Token revoked' });
    }

    req.user = { id: decoded.sub, role: decoded.role };
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

| Strategy | Memory | False Positives | Use Case |
|----------|--------|----------------|----------|
| Redis denylist | O(revoked tokens) | None | Most applications |
| Bloom filter | O(revoked tokens) / 10 | Possible | Very large denylists |
| Version claim | O(1) per user | None | Password change revocation |

**Rules:**
- Set the denylist entry TTL to the token's remaining lifetime — Redis cleans up automatically.
- Use the `jti` claim as the denylist key — never the full token.
- For password changes, increment a `tokenVersion` claim and store the version per user.
- Bloom filters are only worthwhile when the denylist grows beyond millions of entries.

**Constraints:**
- The denylist introduces a database lookup on every request — Redis makes this fast but not free.
- In distributed systems, all instances must share the same Redis instance.
- Bloom filters may produce false positives — a valid token may be rejected.

### Annotated Code Example

```javascript
// revocation.js
const redis = require('../redis');

// Revoke a token by adding its jti to the denylist
async function revokeAccessToken(decodedToken) {
  const ttl = decodedToken.exp - Math.floor(Date.now() / 1000);
  if (ttl <= 0) return; // Already expired

  await redis.setex(`denylist:${decodedToken.jti}`, ttl, '1');
}

// Logout — revoke both access and refresh tokens
app.post('/auth/logout', authenticateToken, async (req, res) => {
  const decoded = jwt.decode(req.headers.authorization.split(' ')[1]);

  // Revoke access token
  await revokeAccessToken(decoded);

  // Revoke refresh token family
  const record = await RefreshToken.findOne({ userId: req.user.id });
  if (record) {
    await redis.del(`family:${record.family}`);
    await RefreshToken.updateMany(
      { family: record.family },
      { revoked: true }
    );
  }

  res.json({ success: true });
});
```

**Expected Output (logout response):**
```json
{ "success": true }
```

**Expected Output (subsequent request with revoked token):**
```json
{ "error": "Token revoked" }
```

**Why this output:** The logout handler adds the access token's `jti` to the Redis denylist with a TTL matching the token's remaining lifetime. The refresh token family is also revoked. The next request with the revoked token is rejected by the denylist check.

### Real-World Cases

- **Logout:** Immediately revoke access and refresh tokens.
- **Password change:** Revoke all tokens issued before the password change.
- **Theft detection:** Revoke the entire token family when reuse is detected.
- **Admin revocation:** Administrators can revoke a user's tokens from a management console.

---

## Core Concept 6: Token Storage Security

### Definitions

**Core Definition:** Token storage security is the practice of storing JWTs and refresh tokens in locations that are resistant to cross-site scripting (XSS) and cross-site request forgery (CSRF) attacks, with HttpOnly, Secure, and SameSite cookies being the recommended approach for web applications.

**Technical Definition:** Where a token is stored determines which attack classes can compromise it. **localStorage** and **sessionStorage** are accessible by any JavaScript running on the origin — a single XSS vulnerability allows an attacker to read the token and exfiltrate it. **HttpOnly cookies** are not readable by JavaScript, blocking XSS token theft entirely. **Secure cookies** are only transmitted over HTTPS, preventing man-in-the-middle interception. **SameSite cookies** control whether the cookie is attached to cross-site requests, mitigating CSRF. The **`__Host-` prefix** enforces the strongest browser scope: Secure, no Domain attribute, and `Path=/`. For native apps, platform-secure storage (iOS Keychain, Android Keystore) provides encryption at rest and isolation. The recommended pattern for SPAs is to store the access token in memory (never persisted) and the refresh token in an HttpOnly, Secure, SameSite cookie.

**Beginner-Friendly Explanation:** Storing tokens in localStorage is like leaving your house key under the doormat — anyone who gets inside (XSS) can grab it. Storing them in HttpOnly cookies is like putting the key in a locked safe that only the browser can open — JavaScript can't reach it. The trade-off is that cookies are automatically sent with requests, which opens CSRF risks — mitigated by SameSite attributes.

### Purposes

- To mitigate cross-site scripting (XSS) and data theft by storing tokens in secure HttpOnly cookies instead of localStorage or sessionStorage.
- To prevent man-in-the-middle interception with Secure cookies (HTTPS only).
- To mitigate CSRF with SameSite attributes.
- To use the `__Host-` prefix for the strongest cookie scope enforcement.
- To store access tokens in memory and refresh tokens in HttpOnly cookies for SPAs.

### Syntax Rules and Structure

#### Secure Cookie Configuration (Express)

```javascript
app.use(session({
  secret: process.env.SESSION_SECRET,
  name: '__Host-session',
  cookie: {
    httpOnly: true,                    // Blocks XSS
    secure: true,                      // HTTPS only
    sameSite: 'lax',                   // CSRF mitigation
    path: '/',                         // Required for __Host- prefix
    maxAge: 15 * 60 * 1000             // 15 minutes
  }
}));
```

#### Token Storage Comparison

| Storage | XSS Readable | CSRF Risk | Best For |
|---------|-------------|-----------|----------|
| localStorage | ✅ Yes (vulnerable) | ❌ None (not auto-sent) | Non-sensitive, short-lived tokens |
| sessionStorage | ✅ Yes (vulnerable) | ❌ None | Same as localStorage |
| HttpOnly cookie | ❌ No | ✅ Yes (mitigated by SameSite) | Refresh tokens, sensitive sessions |
| Memory (JS variable) | ✅ Yes (but not persisted) | ❌ None | Access tokens in SPAs |
| Keychain/Keystore | ❌ No (native) | ❌ No | Mobile apps |

**Rules:**
- **Never** store refresh tokens in localStorage or sessionStorage.
- Store refresh tokens in **HttpOnly, Secure, SameSite** cookies.
- Store access tokens in **memory** in SPAs — never persist them.
- Use the **`__Host-` prefix** when the architecture does not require sharing across subdomains.
- For native apps, use **iOS Keychain** or **Android Keystore**.
- Set `secure: true` in production (use `'auto'` to match the connection).
- Set `app.set('trust proxy', 1)` when behind a reverse proxy.

**Constraints:**
- HttpOnly cookies introduce CSRF risk — mitigate with SameSite and CSRF tokens.
- The `__Host-` prefix requires `Secure`, `Path=/`, and no `Domain` attribute.
- localStorage is acceptable only for non-sensitive, short-lived data.

### Annotated Code Example

```javascript
// token-storage.js
const express = require('express');
const jwt = require('jsonwebtoken');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser());

// Login — set refresh token in HttpOnly cookie
app.post('/auth/login', async (req, res) => {
  const user = await verifyCredentials(req.body);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  const accessToken = jwt.sign(
    { sub: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = await generateRefreshToken(user.id);

  // Access token in response body (client stores in memory)
  // Refresh token in HttpOnly cookie (not readable by JS)
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    path: '/auth/refresh',
    maxAge: 7 * 24 * 60 * 60 * 1000
  });

  res.json({ accessToken });
});

// Refresh — read refresh token from cookie
app.post('/auth/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token required' });
  }

  const result = await refreshTokens(refreshToken);
  res.json(result);
});
```

**Expected Output (login response headers):**
```
Set-Cookie: refreshToken=abc123...; Path=/auth/refresh; HttpOnly; Secure; SameSite=Strict; Max-Age=604800

Response body:
{ "accessToken": "eyJhbGciOiJIUzI1NiIs..." }
```

**Why this output:** The refresh token is set in an HttpOnly cookie scoped to the `/auth/refresh` path — JavaScript cannot read it, and it is only sent to the refresh endpoint. The access token is returned in the response body and stored in memory by the client. An XSS attack cannot steal the refresh token because it is HttpOnly.

### Real-World Cases

- **SPAs:** Access token in memory, refresh token in HttpOnly cookie.
- **Server-rendered apps:** Session cookies with HttpOnly and Secure flags.
- **Mobile apps:** Tokens in Keychain (iOS) or Keystore (Android).
- **BFF architecture:** All tokens handled server-side; the browser only sees session cookies.

---

## References

- RFC 7519 — JSON Web Token (JWT) — https://datatracker.ietf.org/doc/html/rfc7519
- RFC 8725 — JWT Best Current Practices — https://datatracker.ietf.org/doc/html/rfc8725
- OWASP — JSON Web Token for Java Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- OWASP — JWT for Java Cheat Sheet (Authentication) — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- shattered.io — Authentification JWT en Node.js : 12 étapes [2026] — https://shattered.io/fr/authentification-jwt-nodejs/
- shattered.io — Autenticazione JWT in Node.js: 12 Step [2026] — https://shattered.io/it/autenticazione-jwt-nodejs/
- DEV Community — Detecting Refresh Token Reuse with Redis (Working Code Included) — https://dev.to/polasamyeng/detecting-refresh-token-reuse-with-redis-working-code-included-fhd
- Auth0 — Configure Refresh Token Rotation — https://auth0.com/docs/secure/tokens/refresh-tokens/configure-refresh-token-rotation
- GitHub — ifindev/secure-authentication — https://github.com/ifindev/secure-authentication
- Duende Software — Best Practices When Using JWTs With Web and Mobile Apps — https://duendesoftware.com/learn/best-practices-using-jwts-with-web-and-mobile-apps
- Safeguard.sh — Single-Page Application (SPA) Security Guide (2026) — https://safeguard.sh/resources/blog/spa-security-guide
- Safeguard.sh — Secure Token Storage for SPAs: Avoiding XSS Theft — https://safeguard.sh/resources/blog/secure-token-storage-spa
- npm — jsonwebtoken — https://www.npmjs.com/package/jsonwebtoken
- npm — @purecore-br/jwt — https://www.npmjs.com/package/@purecore-br/jwt
- OneUptime — How to Handle JWT Token Validation — https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-jwt-token-validation/README.md
- Stack Overflow — JWT Blacklist with Redis — https://stackoverflow.com/questions/37563485/jwt-blacklist-with-redis
- GitHub — Zero-Trust Token Revocation List (TRL) with Redis Bloom Filters — https://github.com/nupurmadaan04/SOUL_SENSE_EXAM/issues/1101
- npm — jwt-blacklist (Bloom filter) — https://www.npmjs.com/package/jwt-blacklist
- Seven Square Technologies — How to Build a Distributed Token Blacklist Service in Node.js with Redis — https://www.sevensquaretech.com/how-to-build-a-distributed-token-blacklist-service-in-node-js-with-redis/