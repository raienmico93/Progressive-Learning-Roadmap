# Token Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Token authentication is a stateless authentication mechanism in which the server issues a cryptographically signed or encrypted token to the client after successful login, and the client presents this token with every subsequent request to prove its identity — without the server needing to store session state.

**Technical Definition:** Token authentication uses self-contained credentials (typically JSON Web Tokens, JWTs) that encode claims about the authenticated principal. A JWT consists of three base64url-encoded parts: header (algorithm and token type), payload (claims such as `sub`, `iss`, `aud`, `exp`, `iat`, and custom data), and signature (cryptographic proof of integrity). Tokens are signed with HMAC (HS256/384/512) or asymmetric algorithms (RS256/384/512, ES256/384/512, EdDSA). Access tokens are short-lived and carried in the `Authorization: Bearer` header. Refresh tokens are long-lived, stored securely, and used to obtain new access tokens. Security depends on algorithm validation, audience/issuer checks, short TTLs, rotation with reuse detection, and revocation strategies (denylists, short TTLs, or distributed checks).

**Beginner-Friendly Explanation:** A token is like a signed concert ticket. The ticket says "Alice, seat 42, valid until 10 PM" and is stamped with the venue's unforgeable seal. Anyone can read the ticket, but only the venue can create a valid one. You carry the ticket yourself — the venue doesn't keep a guest list. If your ticket is stolen, the thief can use it until it expires, which is why tickets are short-lived. A **refresh token** is like a VIP pass that lets you get new tickets without standing in line again. And if you lose your VIP pass, you can cancel it (revocation) — but regular tickets can't be cancelled, so they must expire quickly.

### Key Characteristics

- **Stateless:** The server does not store token state — all information is in the token itself.
- **Self-contained:** Tokens carry claims (identity, roles, scopes, expiration) directly.
- **Cryptographically signed:** Tokens cannot be tampered with without invalidating the signature.
- **Short-lived access tokens:** Typical TTL is 5–15 minutes to limit the window of compromise.
- **Long-lived refresh tokens:** Used to obtain new access tokens without re-authentication.
- **Rotation with reuse detection:** Refresh tokens are rotated on each use; reuse triggers family revocation.
- **Revocation trade-off:** Stateless tokens cannot be revoked without a denylist or short TTLs.
- **Asymmetric vs. symmetric:** RS256/ES256 enable public-key verification by multiple services; HS256 requires shared secrets.

### Prerequisites

- **HTTP fundamentals:** Headers, status codes, and the `Authorization` header.
- **Cryptography basics:** Hashing, HMAC, digital signatures, public/private key pairs.
- **JWT structure:** Header, payload, signature, base64url encoding.
- **JOSE standards:** JWS (JSON Web Signature), JWE (JSON Web Encryption), JWK (JSON Web Key), JWA (JSON Web Algorithms).
- **OAuth 2.0 and OpenID Connect:** Token flows, scopes, and claims.
- **Node.js fundamentals:** Express or NestJS, middleware, async patterns.
- **Token libraries:** `jsonwebtoken`, `jose`, `@nestjs/jwt`, `fast-jwt`.

### Related Programming Areas

- **OAuth 2.0 / OpenID Connect:** Token issuance and validation in federated identity.
- **API Gateways:** Token validation at the edge (Kong, AWS API Gateway, NGINX).
- **Microservices:** JWTs propagate identity across service boundaries.
- **Session Authentication:** Tokens are an alternative to sessions; hybrid designs exist.
- **Authorization:** Claims in tokens drive RBAC/ABAC decisions.
- **Compliance:** PCI DSS, HIPAA, SOC 2 mandate token security controls.

### Core Concepts

1. **JWT Concepts** — parsing and validating JSON Web Token headers, payloads, and signatures.
2. **Access Tokens** — issuing short-lived state tokens for stateless API authorization.
3. **Refresh Tokens** — persisting long-lived tokens securely to acquire new access keys without re-authenticating.
4. **Token Expiration** — configuring safe time-to-live thresholds for tokens.
5. **Token Rotation** — invalidating previous refresh tokens and issuing pairs on every exchange to catch replay attacks.
6. **Token Revocation** — managing blacklists or using short lifetimes alongside distributed storage checks to invalidate stateless keys early.
7. **Cryptographic Signatures & Encryption** — understanding JWS vs. JWE, and symmetric vs. asymmetric key pairs like RS256.

---

## Core Concept 1: JWT Concepts (Parsing and Validating)

### Definitions

**Core Definition:** A JSON Web Token (JWT) is a compact, URL-safe, self-contained token that encodes claims as a JSON object and is cryptographically signed or encrypted.

**Technical Definition:** A JWT (RFC 7519) is a string composed of three base64url-encoded parts separated by dots: `header.payload.signature`. The header specifies the signing algorithm (`alg`) and token type (`typ`). The payload contains claims — registered (`iss`, `sub`, `aud`, `exp`, `nbf`, `iat`, `jti`), public, or private. The signature is computed over `base64url(header) + "." + base64url(payload)` using a secret (HMAC) or private key (RSA/ECDSA/EdDSA). Validation requires: (1) parsing the token; (2) verifying the signature with the correct key; (3) checking `alg` against an allowlist; (4) validating `iss`, `aud`, `exp`, `nbf`, and `iat`; and (5) checking claims against application rules.

**Beginner-Friendly Explanation:** A JWT is like a driver's license. The front (header) says what kind of document it is. The back (payload) contains your name, birthdate, and expiration date. The hologram (signature) proves it's genuine. When you present it, the verifier checks the hologram, confirms the expiration date is in the future, and confirms the issuing authority is one they trust. If any of these checks fail, the license is rejected.

### Purposes

- To provide a self-contained, tamper-evident credential for stateless authentication.
- To carry identity and authorization claims across services without a shared session store.
- To enable cross-domain authentication (tokens in headers, not cookies).
- To support fine-grained, time-limited access via `exp`, `nbf`, and `iat`.
- To enable signature verification by any party with the public key (asymmetric) or shared secret (symmetric).

### Syntax Rules and Structure

#### JWT Structure

```
header.payload.signature
```

| Part | Contents | Example |
|------|----------|---------|
| **Header** | `{ "alg": "RS256", "typ": "JWT", "kid": "key-1" }` | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImtleS0xIn0` |
| **Payload** | `{ "sub": "user-123", "role": "admin", "exp": 1712345678, "iat": 1712342078 }` | `eyJzdWIiOiJ1c2VyLTEyMyIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTcxMjM0NTY3OCwiaWF0IjoxNzEyMzQyMDc4fQ` |
| **Signature** | `RSASHA256(base64url(header) + "." + base64url(payload), privateKey)` | `SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c` |

#### Registered Claims

| Claim | Name | Purpose |
|-------|------|---------|
| `iss` | Issuer | Who created the token. |
| `sub` | Subject | Who the token is about (user ID). |
| `aud` | Audience | Who the token is for (API, service). |
| `exp` | Expiration Time | When the token expires (Unix timestamp). |
| `nbf` | Not Before | When the token becomes valid. |
| `iat` | Issued At | When the token was issued. |
| `jti` | JWT ID | Unique identifier for the token (replay prevention). |

#### JWT Signing (jsonwebtoken)

```typescript
import jwt from 'jsonwebtoken';
import { randomUUID } from 'node:crypto';

const accessToken = jwt.sign(
  {
    sub: user.id,
    roles: user.roles,
    type: 'access',
  },
  privateKey,
  {
    algorithm: 'RS256',
    expiresIn: '15m',
    issuer: 'https://api.example.com',
    audience: 'https://app.example.com',
    jwtid: randomUUID(),
  },
);
```

#### JWT Verification (jsonwebtoken)

```typescript
import jwt from 'jsonwebtoken';

try {
  const payload = jwt.verify(token, publicKey, {
    algorithms: ['RS256'],              // Allowlist — prevents alg confusion
    issuer: 'https://api.example.com',
    audience: 'https://app.example.com',
    clockTolerance: 5,                  // 5-second leeway for clock skew
  });
  // payload is trusted
} catch (err) {
  if (err instanceof jwt.TokenExpiredError) {
    // Token has expired — client should refresh
  } else if (err instanceof jwt.JsonWebTokenError) {
    // Invalid signature, malformed token, or claim mismatch
  }
}
```

#### Syntax Rules

- **Always validate the algorithm** — pass an explicit `algorithms` allowlist to `verify()`.
- **Never accept `alg: none`** — reject unsigned tokens.
- **Always validate `iss` and `aud`** — prevent cross-service token reuse.
- **Always validate `exp` and `nbf`** — reject expired or not-yet-valid tokens.
- **Use `clockTolerance`** — allow a few seconds for clock skew between services.
- **Verify the signature before trusting any claim** — never read the payload first.
- **Use the `kid` header** to select the correct key during rotation.
- **Never store secrets in the payload** — payloads are base64url-encoded, not encrypted.
- **Validate custom claims** (roles, scopes) after signature verification.

#### Constraints and Limitations

- **JWTs cannot be revoked** without a denylist or short TTL.
- **Payloads are readable by anyone** — never include sensitive data.
- **Token size grows with claims** — large payloads increase request size.
- **Clock skew** can cause valid tokens to be rejected; use `clockTolerance`.
- **Algorithm confusion attacks** — always specify allowed algorithms.
- **Key management is complex** — asymmetric keys must be rotated and distributed securely.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: JWT Signing, Verification, and Claim Validation (Node.js)

```typescript
// token.service.ts
import jwt from 'jsonwebtoken';
import { randomUUID } from 'node:crypto';
import { readFileSync } from 'node:fs';

export class TokenService {
  private readonly privateKey: string;
  private readonly publicKey: string;
  private readonly issuer = 'https://api.example.com';
  private readonly audience = 'https://app.example.com';

  constructor() {
    this.privateKey = readFileSync('./keys/private.pem', 'utf8');
    this.publicKey = readFileSync('./keys/public.pem', 'utf8');
  }

  signAccessToken(userId: string, roles: string[]): string {
    return jwt.sign(
      { sub: userId, roles, type: 'access' },
      this.privateKey,
      {
        algorithm: 'RS256',
        expiresIn: '15m',
        issuer: this.issuer,
        audience: this.audience,
        jwtid: randomUUID(),
      },
    );
  }

  verifyAccessToken(token: string): jwt.JwtPayload {
    return jwt.verify(token, this.publicKey, {
      algorithms: ['RS256'],
      issuer: this.issuer,
      audience: this.audience,
      clockTolerance: 5,
    }) as jwt.JwtPayload;
  }
}
```

```typescript
// auth.guard.ts — Express middleware
import { Request, Response, NextFunction } from 'express';

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing bearer token' });
  }

  const token = header.slice(7);

  try {
    const payload = tokenService.verifyAccessToken(token);
    req.user = { id: payload.sub!, roles: payload.roles as string[] };
    next();
  } catch (err) {
    if (err instanceof jwt.TokenExpiredError) {
      return res.status(401).json({ error: 'Token expired', code: 'TOKEN_EXPIRED' });
    }
    return res.status(401).json({ error: 'Invalid token', code: 'INVALID_TOKEN' });
  }
}
```

**Expected behaviour:**
- `signAccessToken()` produces an RS256-signed JWT with `sub`, `roles`, `exp`, `iss`, `aud`, and `jti`.
- `verifyAccessToken()` verifies the signature, algorithm, issuer, audience, and expiration.
- The middleware extracts the token from the `Authorization: Bearer` header, verifies it, and attaches the user to the request.
- Expired tokens return `401` with `TOKEN_EXPIRED`; invalid tokens return `401` with `INVALID_TOKEN`.

**Why this works:** The token is signed with a private key and verified with the public key — any service with the public key can verify it without the private key. The `algorithms` allowlist prevents algorithm confusion attacks. Issuer and audience checks prevent cross-service token reuse.

### Real-World Cases

- **Microservices:** Each service verifies JWTs with the public key, without contacting the auth service.
- **API gateways:** The gateway validates tokens at the edge before routing to backend services.
- **SPAs and mobile apps:** Access tokens in memory, refresh tokens in secure storage.
- **Federated identity:** OpenID Connect ID tokens carry identity claims from the IdP.

---

## Core Concept 2: Access Tokens

### Definitions

**Core Definition:** An access token is a short-lived credential that grants the bearer access to specific resources or APIs, typically carried in the `Authorization: Bearer` header.

**Technical Definition:** Access tokens are issued by an authorization server after successful authentication and represent the authorization granted to the client (scopes, roles, permissions). They may be opaque (random strings verified via introspection) or structured (JWTs verified locally). Access tokens are typically short-lived (5–15 minutes) to limit the impact of theft. In OAuth 2.0, access tokens are bound to scopes and may be audience-restricted. Best practice: use JWTs for stateless verification, include minimal claims (`sub`, `scope`, `aud`, `exp`), and never include sensitive data.

**Beginner-Friendly Explanation:** An access token is like a temporary key card for a building. It says "Alice, access to floors 1–3, valid for 15 minutes." You show it at every door, and the door checks it. Because it expires quickly, if someone steals it, they can only use it for a short time. And because it's scoped, it only opens the doors Alice is allowed to open.

### Purposes

- To grant stateless, scoped access to APIs without server-side sessions.
- To carry authorization claims (scopes, roles) for fine-grained access control.
- To enable local verification (JWTs) without introspection calls.
- To limit the impact of theft via short TTLs.
- To support audience restriction (tokens valid only for specific APIs).

### Syntax Rules and Structure

#### Access Token Payload (Minimal)

```json
{
  "sub": "user-123",
  "scope": "read:orders write:orders",
  "aud": "https://api.example.com",
  "iss": "https://auth.example.com",
  "exp": 1712345678,
  "iat": 1712342078,
  "jti": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

| Claim | Purpose |
|-------|---------|
| `sub` | User or service identifier. |
| `scope` | Space-separated permissions. |
| `aud` | Intended audience (API). |
| `iss` | Issuer (authorization server). |
| `exp` | Expiration (short TTL). |
| `jti` | Unique token ID (for revocation). |

#### Access Token Issuance (NestJS)

```typescript
@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    private readonly prisma: PrismaService,
    private readonly passwordService: PasswordService,
  ) {}

  async login(email: string, password: string): Promise<{ accessToken: string; refreshToken: string }> {
    const user = await this.prisma.user.findUnique({ where: { email } });
    if (!user || !(await this.passwordService.verify(password, user.passwordHash))) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const accessToken = await this.jwtService.signAsync(
      { sub: user.id, scope: user.scopes, type: 'access' },
      {
        algorithm: 'RS256',
        expiresIn: '15m',
        issuer: 'https://auth.example.com',
        audience: 'https://api.example.com',
        jwtid: randomUUID(),
      },
    );

    const refreshToken = await this.issueRefreshToken(user.id);

    return { accessToken, refreshToken };
  }
}
```

#### Syntax Rules

- **Keep access tokens short-lived** — 5–15 minutes is standard.
- **Use the `Authorization: Bearer <token>` header** — never in query strings.
- **Include only necessary claims** — `sub`, `scope`, `aud`, `exp`, `iat`, `jti`.
- **Scope tokens narrowly** — use OAuth 2.0 scopes for fine-grained access.
- **Restrict audience** — validate `aud` to prevent token reuse across APIs.
- **Never include sensitive data** — payloads are readable.
- **Use asymmetric signing (RS256/ES256)** for multi-service architectures.
- **Store access tokens in memory** (SPAs) or secure storage (mobile) — never `localStorage` for long-lived tokens.

#### Constraints and Limitations

- **Access tokens cannot be revoked** without a denylist or short TTL.
- **Short TTLs increase refresh frequency** — balance security with performance.
- **Token size grows with claims** — keep payloads minimal.
- **Bearer tokens are vulnerable to theft** — use HTTPS, `HttpOnly` cookies (for SPAs), or DPoP/mTLS for high-security scenarios.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Access Token Issuance and Scoped Authorization (Express)

```typescript
// auth.service.ts
import jwt from 'jsonwebtoken';
import { randomUUID } from 'node:crypto';

export class AuthService {
  async login(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user || !(await passwordService.verify(password, user.passwordHash))) {
      throw new Error('Invalid credentials');
    }

    const accessToken = jwt.sign(
      { sub: user.id, scope: user.scopes, type: 'access' },
      privateKey,
      {
        algorithm: 'RS256',
        expiresIn: '15m',
        issuer: 'https://auth.example.com',
        audience: 'https://api.example.com',
        jwtid: randomUUID(),
      },
    );

    return { accessToken };
  }
}
```

```typescript
// require-scope.middleware.ts
export function requireScope(scope: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const scopes = (req.user?.scope ?? '').split(' ');
    if (!scopes.includes(scope)) {
      return res.status(403).json({ error: `Missing scope: ${scope}` });
    }
    next();
  };
}

// Usage
app.get('/orders', requireAuth, requireScope('read:orders'), (req, res) => {
  res.json({ orders: [] });
});

app.post('/orders', requireAuth, requireScope('write:orders'), (req, res) => {
  res.status(201).json({ id: 'order-1' });
});
```

**Expected behaviour:**
- A user with `read:orders` scope can `GET /orders` but receives `403` on `POST /orders`.
- A user with `write:orders` scope can `POST /orders`.
- Expired tokens return `401` from the `requireAuth` middleware.

**Why this works:** The access token carries the user's scopes. The `requireScope` middleware checks the scopes after authentication, enforcing fine-grained authorization without a database lookup.

### Real-World Cases

- **API gateways:** Access tokens validated at the edge, scopes enforced per route.
- **Microservices:** Access tokens propagate identity and scopes across services.
- **Third-party APIs:** OAuth 2.0 access tokens with scoped permissions.
- **SPAs:** Access tokens in memory, refresh tokens in `HttpOnly` cookies.

---

## Core Concept 3: Refresh Tokens

### Definitions

**Core Definition:** A refresh token is a long-lived credential used to obtain new access tokens without requiring the user to re-authenticate.

**Technical Definition:** Refresh tokens are issued alongside access tokens during authentication. They are stored securely (HttpOnly cookies for browsers, secure storage for mobile) and exchanged at a dedicated endpoint (`/auth/refresh`) for new access tokens. Refresh tokens should be opaque (random strings) or JWTs with minimal claims. Best practice: store refresh tokens as hashes in a database, bind them to a session/family for reuse detection, rotate them on each use, and revoke them on logout or password change. Refresh tokens must never be sent to resource servers — only to the authorization server.

**Beginner-Friendly Explanation:** An access token is like a day pass to a gym — it expires quickly. A refresh token is like your membership card — it lasts much longer and lets you get a new day pass whenever you need one. You keep the membership card safe (in a secure place), and you only show it at the front desk (the refresh endpoint), never to the equipment (resource servers).

### Purposes

- To obtain new access tokens without re-authentication.
- To improve UX by reducing login frequency.
- To enable secure, long-lived sessions with short-lived access tokens.
- To support revocation via server-side storage.
- To detect token theft via rotation and reuse detection.

### Syntax Rules and Structure

#### Refresh Token Database Schema

```prisma
model RefreshToken {
  id          String   @id @default(uuid())
  userId      String
  tokenHash   String   @unique    // SHA-256 hash of the token
  family      String              // Token family for reuse detection
  expiresAt   DateTime
  revokedAt   DateTime?
  replacedBy  String?
  createdAt   DateTime @default(now())
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([family])
}
```

#### Refresh Token Issuance

```typescript
import { randomBytes, createHash } from 'node:crypto';

async function issueRefreshToken(userId: string, family?: string): Promise<string> {
  const token = randomBytes(32).toString('base64url'); // 256 bits
  const tokenHash = createHash('sha256').update(token).digest('hex');
  const tokenId = randomUUID();
  const familyId = family ?? tokenId;

  await prisma.refreshToken.create({
    data: {
      id: tokenId,
      userId,
      tokenHash,
      family: familyId,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
    },
  });

  return token;
}
```

#### Refresh Endpoint (Express)

```typescript
app.post('/auth/refresh', async (req, res, next) => {
  try {
    const refreshToken = req.cookies['__Host-refresh'];
    if (!refreshToken) return res.status(401).json({ error: 'Missing refresh token' });

    const tokenHash = createHash('sha256').update(refreshToken).digest('hex');
    const record = await prisma.refreshToken.findUnique({ where: { tokenHash } });

    if (!record || record.revokedAt || record.expiresAt < new Date()) {
      if (record) {
        // Reuse detected — revoke the entire family
        await prisma.refreshToken.updateMany({
          where: { family: record.family },
          data: { revokedAt: new Date() },
        });
      }
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    // Rotate: revoke old, issue new
    const newToken = await issueRefreshToken(record.userId, record.family);
    await prisma.refreshToken.update({
      where: { id: record.id },
      data: { revokedAt: new Date(), replacedBy: newToken },
    });

    // Issue new access token
    const accessToken = signAccessToken(record.userId);

    res.cookie('__Host-refresh', newToken, {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      maxAge: 7 * 24 * 60 * 60 * 1000,
      path: '/',
    });

    res.json({ accessToken });
  } catch (err) {
    next(err);
  }
});
```

#### Syntax Rules

- **Store refresh tokens as hashes** — never plaintext.
- **Use opaque tokens** (random strings) for refresh tokens — they don't need to be self-contained.
- **Bind refresh tokens to a family** — enables reuse detection.
- **Rotate on every use** — revoke the old token, issue a new one.
- **Detect reuse** — if a revoked token is presented, revoke the entire family.
- **Set a reasonable TTL** — 7–30 days is standard.
- **Store in HttpOnly cookies** (browsers) or secure storage (mobile).
- **Never send refresh tokens to resource servers** — only to the refresh endpoint.
- **Revoke on logout and password change.**
- **Rate-limit the refresh endpoint.**

#### Constraints and Limitations

- **Refresh tokens are long-lived** — theft has a larger impact than access token theft.
- **Rotation requires server-side storage** — not stateless.
- **Reuse detection can cause false positives** — network retries may trigger family revocation.
- **Cookie storage requires CSRF protection** — refresh endpoints must be CSRF-protected.
- **Mobile apps must use secure storage** — Keychain (iOS), Keystore (Android).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Refresh Token Flow with Rotation and Reuse Detection

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  async login(email: string, password: string) {
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user || !(await passwordService.verify(password, user.passwordHash))) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const accessToken = signAccessToken(user.id);
    const refreshToken = await issueRefreshToken(user.id);

    return { accessToken, refreshToken };
  }

  async refresh(refreshToken: string) {
    const tokenHash = createHash('sha256').update(refreshToken).digest('hex');
    const record = await prisma.refreshToken.findUnique({ where: { tokenHash } });

    if (!record || record.expiresAt < new Date()) {
      throw new UnauthorizedException('Invalid refresh token');
    }

    // Reuse detection: if already revoked, revoke the entire family
    if (record.revokedAt) {
      await prisma.refreshToken.updateMany({
        where: { family: record.family },
        data: { revokedAt: new Date() },
      });
      throw new UnauthorizedException('Refresh token reuse detected');
    }

    // Rotate
    const newRefreshToken = await issueRefreshToken(record.userId, record.family);
    await prisma.refreshToken.update({
      where: { id: record.id },
      data: { revokedAt: new Date(), replacedBy: newRefreshToken },
    });

    const accessToken = signAccessToken(record.userId);
    return { accessToken, refreshToken: newRefreshToken };
  }

  async logout(refreshToken: string) {
    const tokenHash = createHash('sha256').update(refreshToken).digest('hex');
    await prisma.refreshToken.updateMany({
      where: { tokenHash },
      data: { revokedAt: new Date() },
    });
  }
}
```

**Expected behaviour:**
- `login()` issues an access token and a refresh token.
- `refresh()` rotates the refresh token: revokes the old, issues a new one.
- If a revoked refresh token is presented, the entire family is revoked (reuse detection).
- `logout()` revokes the refresh token.

**Why this works:** Refresh tokens are stored as SHA-256 hashes. Rotation ensures each token is used once. Reuse detection revokes the entire family, mitigating token theft. The refresh token is bound to a family, enabling efficient bulk revocation.

### Real-World Cases

- **SPAs:** Refresh tokens in HttpOnly cookies, access tokens in memory.
- **Mobile apps:** Refresh tokens in Keychain/Keystore, access tokens in memory.
- **APIs:** Refresh tokens stored server-side with rotation and reuse detection.
- **Enterprise:** Refresh tokens with short TTLs and device binding.

---

## Core Concept 4: Token Expiration

### Definitions

**Core Definition:** Token expiration is the time-to-live (TTL) configured for access and refresh tokens, limiting the window during which a stolen token can be used.

**Technical Definition:** Token expiration is enforced via the `exp` (expiration time) claim in JWTs, or via server-side TTL for opaque tokens. Access tokens should expire quickly (5–15 minutes) to limit the impact of theft. Refresh tokens should expire in 7–30 days, balancing security with UX. The `nbf` (not before) and `iat` (issued at) claims provide additional temporal constraints. Clock skew between services must be accommodated with a `clockTolerance` (typically 5–60 seconds). Expiration must be validated server-side — never trust the client's clock. Sliding expiration (renewing the token on each use) can extend the effective lifetime but must be bounded by an absolute maximum.

**Beginner-Friendly Explanation:** Token expiration is like a parking meter. The meter gives you a fixed amount of time (TTL). When time runs out, you must feed the meter again (refresh). Short times mean a stolen ticket is useless quickly. Long times are convenient but riskier. The best balance is short access tokens (15 minutes) with longer refresh tokens (7 days) that you can cancel.

### Purposes

- To limit the window during which a stolen token can be used.
- To enforce re-authentication for sensitive operations.
- To balance security with user experience.
- To comply with security standards (OWASP, NIST).
- To reduce the impact of token leakage.

### Syntax Rules and Structure

#### Recommended TTLs

| Token Type | TTL | Rationale |
|------------|-----|-----------|
| **Access token** | 5–15 minutes | Limits theft impact; stateless verification. |
| **Refresh token** | 7–30 days | Balances UX with security; revocable. |
| **ID token** | 5–15 minutes | Should not outlive the session. |
| **Service token** | 1–24 hours | Long-running services; revocable. |
| **Password reset token** | 15–60 minutes | Single-use, short-lived. |
| **Email verification token** | 24 hours | One-time use. |

#### JWT Expiration Claims

```json
{
  "sub": "user-123",
  "iat": 1712342078,
  "nbf": 1712342078,
  "exp": 1712342978
}
```

| Claim | Meaning | Enforcement |
|-------|---------|-------------|
| `iat` | Issued At | Optional; used for audit. |
| `nbf` | Not Before | Reject if `now < nbf`. |
| `exp` | Expiration | Reject if `now >= exp`. |

#### Verification with Clock Tolerance

```typescript
jwt.verify(token, publicKey, {
  algorithms: ['RS256'],
  issuer: 'https://auth.example.com',
  audience: 'https://api.example.com',
  clockTolerance: 5, // 5 seconds
});
```

#### Syntax Rules

- **Keep access tokens short-lived** (5–15 minutes).
- **Keep refresh tokens longer** (7–30 days).
- **Always validate `exp` and `nbf`** — never trust the client's clock.
- **Use `clockTolerance`** — accommodate clock skew (5–60 seconds).
- **Set an absolute maximum session lifetime** — even with refresh tokens.
- **Rotate refresh tokens on each use** — combined with short access TTLs.
- **Implement step-up authentication** for sensitive operations.
- **Never extend expiration client-side** — always server-side.

#### Constraints and Limitations

- **Short TTLs increase refresh frequency** — more load on the refresh endpoint.
- **Clock skew causes false rejections** — use `clockTolerance`.
- **Long refresh TTLs increase theft impact** — combine with rotation and reuse detection.
- **Expiration does not revoke tokens** — expired tokens are still valid if the clock is manipulated.
- **Token expiration does not invalidate sessions** — sessions must be managed separately.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Configuring Safe TTLs (NestJS + JWT)

```typescript
// auth.module.ts
@Module({
  imports: [
    JwtModule.registerAsync({
      useFactory: () => ({
        privateKey: readFileSync('./keys/private.pem'),
        publicKey: readFileSync('./keys/public.pem'),
        signOptions: {
          algorithm: 'RS256',
          expiresIn: '15m',
          issuer: 'https://auth.example.com',
          audience: 'https://api.example.com',
        },
      }),
    }),
  ],
})
export class AuthModule {}
```

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  async issueTokens(userId: string) {
    const accessToken = await this.jwtService.signAsync(
      { sub: userId, type: 'access' },
      { expiresIn: '15m' },
    );

    const refreshToken = randomBytes(32).toString('base64url');
    const tokenHash = createHash('sha256').update(refreshToken).digest('hex');

    await prisma.refreshToken.create({
      data: {
        userId,
        tokenHash,
        family: randomUUID(),
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
      },
    });

    return { accessToken, refreshToken };
  }
}
```

**Expected behaviour:**
- Access tokens expire after 15 minutes.
- Refresh tokens expire after 7 days.
- The refresh endpoint issues a new access token when the old one expires.

**Why this works:** Short access TTLs limit theft impact. Longer refresh TTLs balance UX. Rotation and reuse detection mitigate refresh token theft.

### Real-World Cases

- **Banking:** Access tokens 5 minutes, refresh tokens 24 hours, step-up for transfers.
- **E-commerce:** Access tokens 15 minutes, refresh tokens 30 days.
- **Enterprise SaaS:** Access tokens 1 hour, refresh tokens 7 days, absolute session 30 days.
- **IoT:** Access tokens 24 hours, refresh tokens 90 days with device binding.

---

## Core Concept 5: Token Rotation

### Definitions

**Core Definition:** Token rotation is the practice of issuing a new refresh token every time the old one is used, invalidating the previous token and detecting reuse as a signal of theft.

**Technical Definition:** Refresh token rotation replaces the refresh token on every exchange: the client sends the old refresh token, the server validates it, revokes it, and issues a new refresh token along with a new access token. If an attacker steals a refresh token and uses it before the legitimate client, the legitimate client's next refresh will fail — because the token is already revoked. This triggers **reuse detection**: the server revokes the entire token family, forcing re-authentication. Rotation with reuse detection is recommended by OAuth 2.0 Security Best Current Practice (RFC 9700) and is essential for high-security applications.

**Beginner-Friendly Explanation:** Imagine you have a special key that lets you get new keys. Every time you use it, it changes into a new key. If someone steals your key and uses it first, the next time you try to use it, it won't work — and the system will know something is wrong. That's rotation: it turns a stolen refresh token into a one-time use item, and it alerts you when theft occurs.

### Purposes

- To detect refresh token theft via reuse detection.
- To limit the window during which a stolen refresh token is usable.
- To comply with OAuth 2.0 Security Best Current Practice.
- To enable revocation of the entire token family when theft is detected.
- To improve security without compromising UX.

### Syntax Rules and Structure

#### Rotation Flow

```
1. Client sends refresh token A to /auth/refresh
2. Server validates A, revokes A, issues B (new refresh) + access token
3. Client stores B
4. If A is used again (replay), server detects A is revoked
5. Server revokes the entire family (A, B, and any descendants)
6. Client must re-authenticate
```

#### Rotation Implementation

```typescript
async function rotateRefreshToken(oldToken: string, family: string): Promise<string> {
  const newToken = randomBytes(32).toString('base64url');
  const newHash = createHash('sha256').update(newToken).digest('hex');
  const newId = randomUUID();

  await prisma.$transaction([
    prisma.refreshToken.update({
      where: { tokenHash: hash(oldToken) },
      data: { revokedAt: new Date(), replacedBy: newId },
    }),
    prisma.refreshToken.create({
      data: {
        id: newId,
        userId: oldRecord.userId,
        tokenHash: newHash,
        family,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    }),
  ]);

  return newToken;
}
```

#### Reuse Detection

```typescript
if (record.revokedAt) {
  // The token was already used — this is a replay
  await prisma.refreshToken.updateMany({
    where: { family: record.family },
    data: { revokedAt: new Date() },
  });
  throw new UnauthorizedException('Refresh token reuse detected');
}
```

#### Syntax Rules

- **Rotate on every refresh** — never reuse a refresh token.
- **Store refresh tokens as hashes** — never plaintext.
- **Bind tokens to a family** — enables bulk revocation.
- **Detect reuse** — revoke the entire family on replay.
- **Notify the user** when reuse is detected (email, in-app).
- **Rate-limit the refresh endpoint** — prevent abuse.
- **Set a maximum family lifetime** — even with rotation, families should expire.
- **Handle network retries gracefully** — retries may trigger false positives.

#### Constraints and Limitations

- **Rotation requires server-side storage** — not stateless.
- **Reuse detection can cause false positives** — network retries or client bugs may trigger family revocation.
- **Mobile apps must handle rotation correctly** — concurrent requests may cause race conditions.
- **Family lifetime must be bounded** — otherwise a family can live indefinitely.
- **Revocation is eventual** — distributed systems may have propagation delays.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Rotation with Reuse Detection (NestJS)

```typescript
// auth.service.ts
async refresh(refreshToken: string) {
  const tokenHash = createHash('sha256').update(refreshToken).digest('hex');
  const record = await prisma.refreshToken.findUnique({ where: { tokenHash } });

  if (!record || record.expiresAt < new Date()) {
    throw new UnauthorizedException('Invalid refresh token');
  }

  if (record.revokedAt) {
    // Reuse detected — revoke the entire family
    await prisma.refreshToken.updateMany({
      where: { family: record.family },
      data: { revokedAt: new Date() },
    });
    await this.notificationService.sendSecurityAlert(record.userId, 'Refresh token reuse detected');
    throw new UnauthorizedException('Refresh token reuse detected');
  }

  // Rotate
  const newToken = randomBytes(32).toString('base64url');
  const newHash = createHash('sha256').update(newToken).digest('hex');
  const newId = randomUUID();

  await prisma.$transaction([
    prisma.refreshToken.update({
      where: { id: record.id },
      data: { revokedAt: new Date(), replacedBy: newId },
    }),
    prisma.refreshToken.create({
      data: {
        id: newId,
        userId: record.userId,
        tokenHash: newHash,
        family: record.family,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    }),
  ]);

  const accessToken = signAccessToken(record.userId);
  return { accessToken, refreshToken: newToken };
}
```

**Expected behaviour:**
- The first refresh rotates the token and returns a new pair.
- If a revoked token is used, the entire family is revoked, and a security alert is sent.
- The user must re-authenticate after reuse detection.

**Why this works:** Rotation ensures each refresh token is used once. Reuse detection revokes the entire family, mitigating theft. Security alerts inform the user.

### Real-World Cases

- **Banking:** Rotation with reuse detection for all refresh tokens.
- **SPAs:** Rotation with HttpOnly cookies; concurrent requests handled via a single-flight pattern.
- **Mobile apps:** Rotation with secure storage; retries handled gracefully.
- **Enterprise:** Rotation with device binding and security alerts.

---

## Core Concept 6: Token Revocation

### Definitions

**Core Definition:** Token revocation is the process of invalidating a token before its natural expiration, using denylists, short TTLs, or distributed storage checks.

**Technical Definition:** Stateless JWTs cannot be revoked without additional mechanisms. Revocation strategies include: (1) **short TTLs** — tokens expire quickly, limiting the revocation window; (2) **denylists** — revoked `jti` values stored in Redis with TTLs matching the token's `exp`; (3) **token versioning** — a `tokenVersion` claim compared against a server-side value; (4) **introspection** — OAuth 2.0 token introspection for opaque tokens; (5) **hybrid** — short TTLs plus denylists for high-risk events. Revocation must be checked on every request (for denylists) or at token refresh (for versioning). Denylists must be shared across all services (Redis) and cleaned up automatically (TTL).

**Beginner-Friendly Explanation:** Imagine a hotel key card. Normally, if you check out early, the card still works until its expiration date. To fix this, the hotel can either make cards expire quickly (short TTL) or keep a list of cancelled cards at every door (denylist). The list must be shared with all doors, and cards can be removed from the list once they expire naturally.

### Purposes

- To invalidate tokens after logout, password change, or security incidents.
- To enforce administrative actions (banning a user, revoking access).
- To comply with security standards requiring revocation.
- To limit the impact of stolen tokens.
- To support "log out everywhere" functionality.

### Syntax Rules and Structure

#### Revocation Strategies

| Strategy | Mechanism | Latency | Storage |
|----------|-----------|---------|---------|
| **Short TTL** | Tokens expire quickly | None (natural) | None |
| **Denylist** | Revoked `jti` values in Redis | Immediate | Redis (TTL = token exp) |
| **Token versioning** | `tokenVersion` claim compared to server | Refresh-time | Database |
| **Introspection** | OAuth 2.0 introspection endpoint | Per-request | Auth server |
| **Hybrid** | Short TTL + denylist for high-risk events | Near-immediate | Redis + DB |

#### Denylist Implementation (Redis)

```typescript
// token-revocation.service.ts
@Injectable()
export class TokenRevocationService {
  constructor(@Inject('REDIS_CLIENT') private readonly redis: Redis) {}

  async revoke(jti: string, exp: number): Promise<void> {
    const ttl = exp - Math.floor(Date.now() / 1000);
    if (ttl > 0) {
      await this.redis.set(`revoked:${jti}`, '1', 'EX', ttl);
    }
  }

  async isRevoked(jti: string): Promise<boolean> {
    return (await this.redis.exists(`revoked:${jti}`)) === 1;
  }

  async revokeAllForUser(userId: string): Promise<void> {
    // Option 1: Use a token version (requires DB)
    await prisma.user.update({
      where: { id: userId },
      data: { tokenVersion: { increment: 1 } },
    });

    // Option 2: Revoke all known jti values (requires tracking)
    const jtis = await this.redis.sMembers(`user_jtis:${userId}`);
    await Promise.all(jtis.map((jti) => this.revoke(jti, /* exp */)));
  }
}
```

#### Verification with Revocation Check

```typescript
async function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = extractBearerToken(req);
  if (!token) return res.status(401).json({ error: 'Missing token' });

  try {
    const payload = jwt.verify(token, publicKey, {
      algorithms: ['RS256'],
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
    });

    // Check denylist
    if (await tokenRevocationService.isRevoked(payload.jti)) {
      return res.status(401).json({ error: 'Token revoked' });
    }

    req.user = { id: payload.sub, roles: payload.roles };
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

#### Token Versioning

```typescript
// Include tokenVersion in JWT
const accessToken = jwt.sign(
  { sub: user.id, tokenVersion: user.tokenVersion, type: 'access' },
  privateKey,
  { algorithm: 'RS256', expiresIn: '15m' },
);

// Verify tokenVersion on each request
const user = await prisma.user.findUnique({ where: { id: payload.sub } });
if (user.tokenVersion !== payload.tokenVersion) {
  return res.status(401).json({ error: 'Token revoked' });
}
```

#### Syntax Rules

- **Use short TTLs as the first line of defense** — limits the revocation window.
- **Use denylists for high-risk events** — logout, password change, admin revocation.
- **Store denylisted `jti` values with TTL equal to the token's remaining lifetime.**
- **Share the denylist across all services** — Redis is the standard choice.
- **Use token versioning for global revocation** — increment on password change.
- **Check revocation on every request** (denylist) or at refresh (versioning).
- **Clean up expired denylist entries automatically** — TTL handles this.
- **Log revocation events** — for audit and monitoring.
- **Notify the user** when their tokens are revoked.

#### Constraints and Limitations

- **Denylists add a per-request lookup** — Redis latency (~1ms) on every request.
- **Denylists grow with revocations** — TTL handles cleanup, but memory must be sized.
- **Token versioning requires a database lookup** — negates some stateless benefits.
- **Introspection adds network latency** — not suitable for high-throughput APIs.
- **Revocation is not instantaneous across replicas** — eventual consistency.
- **Denylists must be highly available** — if Redis is down, revocations fail.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Hybrid Revocation (Short TTL + Denylist) (NestJS + Redis)

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    private readonly redis: Redis,
    private readonly prisma: PrismaService,
  ) {}

  async logout(userId: string, jti: string, exp: number): Promise<void> {
    const ttl = exp - Math.floor(Date.now() / 1000);
    if (ttl > 0) {
      await this.redis.set(`revoked:${jti}`, '1', 'EX', ttl);
    }
  }

  async logoutAll(userId: string): Promise<void> {
    // Increment tokenVersion — invalidates all existing tokens
    await this.prisma.user.update({
      where: { id: userId },
      data: { tokenVersion: { increment: 1 } },
    });
  }

  async isRevoked(jti: string): Promise<boolean> {
    return (await this.redis.exists(`revoked:${jti}`)) === 1;
  }
}
```

```typescript
// auth.guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private readonly jwtService: JwtService,
    private readonly authService: AuthService,
    private readonly prisma: PrismaService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const req = context.switchToHttp().getRequest();
    const token = extractBearerToken(req);
    if (!token) throw new UnauthorizedException('Missing token');

    const payload = await this.jwtService.verifyAsync(token, {
      algorithms: ['RS256'],
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
    });

    // Denylist check
    if (await this.authService.isRevoked(payload.jti)) {
      throw new UnauthorizedException('Token revoked');
    }

    // Token version check
    const user = await this.prisma.user.findUnique({
      where: { id: payload.sub },
      select: { tokenVersion: true },
    });
    if (!user || user.tokenVersion !== payload.tokenVersion) {
      throw new UnauthorizedException('Token revoked');
    }

    req.user = { id: payload.sub, roles: payload.roles };
    return true;
  }
}
```

**Expected behaviour:**
- `logout()` adds the token's `jti` to the denylist with TTL matching the token's remaining lifetime.
- `logoutAll()` increments `tokenVersion`, invalidating all existing tokens.
- The guard checks both the denylist and token version on every request.
- Revoked tokens return `401`.

**Why this works:** Short TTLs limit the revocation window. Denylists handle per-token revocation (logout). Token versioning handles global revocation (password change, "log out everywhere"). The hybrid approach balances stateless benefits with revocation capability.

### Real-World Cases

- **Banking:** Denylist + token versioning for immediate revocation.
- **E-commerce:** Denylist for logout; short TTLs for the rest.
- **Enterprise:** Token versioning for global revocation on password change.
- **High-security APIs:** Introspection for opaque tokens.

---

## Core Concept 7: Cryptographic Signatures & Encryption (JWS vs. JWE)

### Definitions

**Core Definition:** JWS (JSON Web Signature) provides integrity and authenticity by signing the token; JWE (JSON Web Encryption) provides confidentiality by encrypting the token. Symmetric algorithms use a shared secret; asymmetric algorithms use a public/private key pair.

**Technical Definition:** **JWS** (RFC 7515) produces a compact token with three parts (`header.payload.signature`). The signature ensures integrity and authenticity but does not hide the payload. **JWE** (RFC 7516) produces a token with five parts (`header.encrypted_key.iv.ciphertext.tag`). The payload is encrypted, providing confidentiality. **JWA** (RFC 7518) defines algorithms: HMAC (HS256/384/512) for symmetric signing; RSA (RS256/384/512), ECDSA (ES256/384/512), and EdDSA (Ed25519) for asymmetric signing; RSA-OAEP, ECDH-ES, and A256GCMKW for encryption. **Symmetric** algorithms (HS256) use a shared secret — fast but require secure key distribution. **Asymmetric** algorithms (RS256, ES256) use a key pair — the private key signs, the public key verifies, enabling public verification without sharing secrets.

**Beginner-Friendly Explanation:** A **JWS** is like a postcard with a wax seal — anyone can read the message, but the seal proves it's genuine. A **JWE** is like a sealed envelope — the message is hidden, and only the recipient can open it. **Symmetric** signing (HS256) is like a shared secret handshake — both sides must know the same secret. **Asymmetric** signing (RS256) is like a notary's stamp — the notary (private key) stamps the document, and anyone can verify the stamp (public key) without being able to forge it.

### Purposes

- To ensure token integrity (JWS) — detect tampering.
- To ensure token authenticity (JWS) — verify the issuer.
- To ensure token confidentiality (JWE) — protect sensitive claims.
- To enable public verification (asymmetric) — any service can verify without the private key.
- To choose the right algorithm based on architecture (single service vs. multiple services).
- To comply with standards (FAPI, OpenID Connect, OAuth 2.0).

### Syntax Rules and Structure

#### JWS vs. JWE

| Aspect | JWS | JWE |
|--------|-----|-----|
| **Parts** | 3 (`header.payload.signature`) | 5 (`header.encrypted_key.iv.ciphertext.tag`) |
| **Confidentiality** | ❌ No | ✅ Yes |
| **Integrity** | ✅ Yes | ✅ Yes |
| **Authenticity** | ✅ Yes | ✅ Yes |
| **Size** | Smaller | Larger |
| **Use Case** | Access tokens, ID tokens | Sensitive claims, PII |

#### Algorithm Comparison

| Algorithm | Type | Key | Use Case |
|-----------|------|-----|----------|
| **HS256** | Symmetric | Shared secret | Single service; simple setup |
| **RS256** | Asymmetric | RSA key pair | Multi-service; public verification |
| **ES256** | Asymmetric | ECDSA key pair | Smaller keys; modern |
| **EdDSA** | Asymmetric | Ed25519 | Fastest; modern |
| **PS256** | Asymmetric | RSA-PSS | Enhanced RSA security |
| **RSA-OAEP** | Asymmetric | RSA key pair | JWE encryption |
| **ECDH-ES** | Asymmetric | ECDH key pair | JWE encryption |
| **A256GCM** | Symmetric | Shared secret | JWE content encryption |

#### Symmetric Signing (HS256)

```typescript
import jwt from 'jsonwebtoken';

const token = jwt.sign({ sub: 'user-123' }, process.env.JWT_SECRET!, {
  algorithm: 'HS256',
  expiresIn: '15m',
});

const payload = jwt.verify(token, process.env.JWT_SECRET!, {
  algorithms: ['HS256'],
});
```

#### Asymmetric Signing (RS256)

```typescript
import jwt from 'jsonwebtoken';
import { readFileSync } from 'node:fs';

const privateKey = readFileSync('./keys/private.pem');
const publicKey = readFileSync('./keys/public.pem');

const token = jwt.sign({ sub: 'user-123' }, privateKey, {
  algorithm: 'RS256',
  expiresIn: '15m',
});

const payload = jwt.verify(token, publicKey, {
  algorithms: ['RS256'],
});
```

#### JWE Encryption (jose)

```typescript
import { SignJWT, EncryptJWT, jwtDecrypt, compactDecrypt } from 'jose';

// JWS (signed)
const jws = await new SignJWT({ sub: 'user-123' })
  .setProtectedHeader({ alg: 'ES256' })
  .setIssuedAt()
  .setExpirationTime('15m')
  .sign(privateKey);

// JWE (encrypted)
const jwe = await new EncryptJWT({ sub: 'user-123', ssn: '123-45-6789' })
  .setProtectedHeader({ alg: 'RSA-OAEP', enc: 'A256GCM' })
  .setIssuedAt()
  .setExpirationTime('15m')
  .encrypt(publicKey);

// Decrypt
const { payload } = await jwtDecrypt(jwe, privateKey);
```

#### Syntax Rules

- **Use JWS for access tokens** — integrity and authenticity are sufficient.
- **Use JWE for sensitive claims** — PII, SSN, financial data.
- **Prefer asymmetric (RS256, ES256, EdDSA)** for multi-service architectures.
- **Use symmetric (HS256)** only when the same service signs and verifies.
- **Never use `alg: none`** — reject unsigned tokens.
- **Always specify `algorithms` in verification** — prevent algorithm confusion.
- **Rotate keys periodically** — use `kid` header for key selection.
- **Store private keys securely** — HSM, KMS, or secrets manager.
- **Publish public keys via JWKS** — `/.well-known/jwks.json`.

#### Constraints and Limitations

- **JWS does not hide the payload** — never include sensitive data.
- **JWE is larger and slower** — use only when confidentiality is required.
- **Asymmetric signing is slower** than symmetric — but the public key can verify.
- **Key management is complex** — rotation, distribution, and revocation.
- **JWKS endpoints must be highly available** — if the JWKS is down, verification fails.
- **Algorithm confusion attacks** — always specify allowed algorithms.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: RS256 with JWKS Endpoint (NestJS)

```typescript
// token.service.ts
@Injectable()
export class TokenService {
  private readonly privateKey: KeyLike;
  private readonly publicKey: KeyLike;
  private readonly kid = 'key-2026-01';

  constructor() {
    this.privateKey = createPrivateKey(readFileSync('./keys/private.pem'));
    this.publicKey = createPublicKey(readFileSync('./keys/public.pem'));
  }

  async signAccessToken(userId: string, roles: string[]): Promise<string> {
    return new SignJWT({ sub: userId, roles, type: 'access' })
      .setProtectedHeader({ alg: 'RS256', kid: this.kid, typ: 'JWT' })
      .setIssuedAt()
      .setIssuer('https://auth.example.com')
      .setAudience('https://api.example.com')
      .setExpirationTime('15m')
      .setJti(randomUUID())
      .sign(this.privateKey);
  }

  async verifyAccessToken(token: string) {
    return jwtVerify(token, this.publicKey, {
      algorithms: ['RS256'],
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
      clockTolerance: 5,
    });
  }
}
```

```typescript
// jwks.controller.ts — Publish public keys
@Controller('.well-known')
export class JwksController {
  @Get('jwks.json')
  getJwks() {
    const jwk = exportJWK(this.publicKey);
    return {
      keys: [
        {
          ...jwk,
          kid: 'key-2026-01',
          use: 'sig',
          alg: 'RS256',
        },
      ],
    };
  }
}
```

**Expected behaviour:**
- `signAccessToken()` produces an RS256-signed JWT with the `kid` header.
- `GET /.well-known/jwks.json` returns the public key in JWK format.
- Other services fetch the JWKS and verify tokens locally without the private key.

**Why this works:** RS256 enables public verification — any service with the public key can verify tokens without the private key. The `kid` header enables key rotation. JWKS provides a standard way to distribute public keys.

### Real-World Cases

- **Multi-service architectures:** RS256 with JWKS for public verification.
- **Single-service APIs:** HS256 with a shared secret for simplicity.
- **OpenID Connect:** ID tokens are JWS-signed (RS256) by the IdP.
- **Financial APIs (FAPI):** JWE + JWS for high-security scenarios.
- **Sensitive claims:** JWE for PII, SSN, and financial data.

---

## References

- RFC 7519 — JSON Web Token (JWT) — https://www.rfc-editor.org/rfc/rfc7519
- RFC 7515 — JSON Web Signature (JWS) — https://www.rfc-editor.org/rfc/rfc7515
- RFC 7516 — JSON Web Encryption (JWE) — https://www.rfc-editor.org/rfc/rfc7516
- RFC 7517 — JSON Web Key (JWK) — https://www.rfc-editor.org/rfc/rfc7517
- RFC 7518 — JSON Web Algorithms (JWA) — https://www.rfc-editor.org/rfc/rfc7518
- RFC 9700 — Best Current Practice for OAuth 2.0 Security — https://www.rfc-editor.org/rfc/rfc9700
- RFC 6749 — The OAuth 2.0 Authorization Framework — https://www.rfc-editor.org/rfc/rfc6749
- RFC 7662 — OAuth 2.0 Token Introspection — https://www.rfc-editor.org/rfc/rfc7662
- RFC 7009 — OAuth 2.0 Token Revocation — https://www.rfc-editor.org/rfc/rfc7009
- OpenID Connect Core 1.0 — https://openid.net/specs/openid-connect-core-1_0.html
- OWASP Cheat Sheet Series — JSON Web Token for Java — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Authentication — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP — JWT Algorithm Confusion — https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens
- Auth0 — Refresh Token Rotation — https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation
- Auth0 — JWT Best Practices — https://auth0.com/docs/secure/tokens/json-web-tokens
- jsonwebtoken — npm package — https://www.npmjs.com/package/jsonwebtoken
- jose — npm package — https://www.npmjs.com/package/jose
- @nestjs/jwt — npm package — https://www.npmjs.com/package/@nestjs/jwt
- fast-jwt — npm package — https://www.npmjs.com/package/fast-jwt
- NIST SP 800-63B — Digital Identity Guidelines: Authentication and Lifecycle Management — https://pages.nist.gov/800-63-3/sp800-63b.html
- OAuth 2.0 Security Best Current Practice — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics
- IANA — JSON Web Token Claims Registry — https://www.iana.org/assignments/jwt/jwt.xhtml
- IANA — JSON Web Signature and Encryption Algorithms — https://www.iana.org/assignments/jose/jose.xhtml
- JWT.io — Debugger and Library Directory — https://jwt.io/
- PortSwigger — JWT Attacks — https://portswigger.net/web-security/jwt