# Authentication Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authentication is the process of verifying that a user, device, or system is who they claim to be — establishing identity through credentials and maintaining proof of that identity over time via sessions or tokens.

**Technical Definition:** Authentication (AuthN) is the security process of verifying the identity of a principal (user, service, or device) before granting access to resources. It encompasses identity establishment (unique identifiers), credential verification (passwords, cryptographic keys, biometrics), session management (server-side state tracking), token-based stateless verification (JWTs, opaque tokens), multi-factor authentication (combining knowledge, possession, and inherence factors), and passwordless mechanisms (WebAuthn, FIDO2, passkeys). Authentication is distinct from authorization (AuthZ), which determines what an authenticated principal is permitted to do.

**Beginner-Friendly Explanation:** Authentication is like showing your ID at the airport. You prove you are who you say you are by presenting something only you should have — a passport (credential). The security guard checks it (verification), and if everything matches, you're allowed through (authenticated). Once inside, you might get a wristband (session/token) that lets you move around without showing your passport again. Multi-factor authentication is like the guard asking for both your passport AND a fingerprint scan — two different proofs. Passwordless authentication is like using a fingerprint alone, with no password at all.

### Key Characteristics

- **Identity is foundational:** Every authenticated action is tied to a unique identifier (user ID, service account, device ID).
- **Credentials prove claims:** Passwords, keys, and biometrics are the evidence used to verify identity.
- **Sessions provide state:** Server-side sessions track authenticated users across requests.
- **Tokens enable statelessness:** JWTs and opaque tokens carry proof of identity without server-side storage.
- **MFA increases assurance:** Combining multiple factors (knowledge, possession, inherence) significantly reduces account compromise risk.
- **Passwordless is the future:** WebAuthn, FIDO2, and passkeys eliminate shared secrets, resisting phishing and credential stuffing.
- **Defense in depth:** Authentication is one layer; authorization, encryption, and monitoring are equally essential.

### Prerequisites

- **HTTP fundamentals:** Cookies, headers, status codes, and statelessness.
- **Cryptography basics:** Hashing, HMAC, symmetric/asymmetric encryption, digital signatures.
- **Web security concepts:** HTTPS, TLS, CSRF, XSS, and same-origin policy.
- **Session and cookie mechanics:** `Set-Cookie`, `HttpOnly`, `Secure`, `SameSite`.
- **Token standards:** JWT (RFC 7519), OAuth 2.0 (RFC 6749), OpenID Connect.
- **WebAuthn and FIDO2:** Public-key cryptography, authenticators, and relying parties.
- **Node.js fundamentals:** Express/NestJS, middleware, and async patterns.

### Related Programming Areas

- **Authorization (AuthZ):** RBAC, ABAC, and policy engines (Casbin, OPA).
- **Identity Providers (IdP):** Auth0, Okta, Keycloak, AWS Cognito.
- **Single Sign-On (SSO):** SAML 2.0, OpenID Connect, OAuth 2.0.
- **Session Stores:** Redis, Memcached, database-backed sessions.
- **Token Management:** JWT signing/verification, refresh token rotation, revocation lists.
- **Password Hashing:** bcrypt, scrypt, Argon2.
- **WebAuthn Libraries:** SimpleWebAuthn, @simplewebauthn/server, fido2-lib.

### Core Concepts

1. **Identity** — establishing unique user identifiers.
2. **Credentials** — verifying claims via passwords, keys, or biometrics.
3. **Sessions** — server-side tracking of an authenticated user's state.
4. **Tokens** — stateless, client-held cryptographic proof of identity.
5. **Multi-Factor Authentication (MFA)** — TOTP, SMS fallbacks, authenticator app setups.
6. **Passwordless Authentication & Passkeys** — WebAuthn, FIDO2 hardware keys, biometric device integration.

---

## Core Concept 1: Identity (Establishing Unique User Identifiers)

### Definitions

**Core Definition:** Identity is the set of attributes that uniquely distinguishes one user, service, or device from all others within a system, anchored by a unique identifier such as a user ID, email address, or username.

**Technical Definition:** In authentication systems, identity is represented by a unique, immutable identifier (typically a UUID, ULID, or database-generated primary key) assigned to each principal at registration. Human-readable identifiers (email, username, phone number) are used for login but may change over time; the immutable ID is the canonical reference. Identity may also include attributes (name, role, tenant) used for authorization and personalization. Federated identity allows a principal to be identified by an external Identity Provider (IdP) via protocols like OpenID Connect (OIDC).

**Beginner-Friendly Explanation:** Your identity is your "account" — the unique thing that distinguishes you from every other user. Your email might be your login name, but it can change; your internal user ID never changes. It's like your national ID number: your name, address, and phone number might change over time, but your ID number stays the same forever.

### Purposes

- To establish a stable, unique reference for every principal in the system.
- To decouple human-readable identifiers (email, username) from the immutable internal ID.
- To enable federation with external identity providers without losing internal consistency.
- To provide a foundation for authorization, auditing, and personalization.
- To support multiple authentication methods (password, OAuth, WebAuthn) linked to the same identity.

### Syntax Rules and Structure

#### Identity Model (TypeScript/Prisma)

```prisma
model User {
  id            String   @id @default(uuid())   // Immutable unique identifier
  email         String   @unique                 // Human-readable login identifier
  emailVerified Boolean  @default(false)
  username      String?  @unique
  displayName   String?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  // Authentication methods linked to this identity
  passwordHash  String?
  mfaEnabled    Boolean  @default(false)
  totpSecret    String?

  // Federated identities
  oauthAccounts OAuthAccount[]
  webauthnCreds WebAuthnCredential[]

  // Sessions and tokens
  sessions      Session[]
  refreshTokens RefreshToken[]
}

model OAuthAccount {
  id         String @id @default(uuid())
  userId     String
  provider   String // "google", "github", "microsoft"
  providerId String // External subject ID
  user       User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerId])
}
```

| Field | Breakdown |
|-------|-----------|
| `id` | Immutable UUID — the canonical identity. |
| `email` | Human-readable, mutable login identifier. |
| `username` | Optional alternate login identifier. |
| `emailVerified` | Whether email ownership has been confirmed. |
| `oauthAccounts` | Links federated identities to the internal identity. |

#### Syntax Rules

- **Always use an immutable identifier** (UUID, ULID, or auto-incrementing integer) as the canonical ID.
- **Never expose sequential IDs** in URLs or APIs without authorization checks — use UUIDs or hashids.
- **Email addresses are mutable** — users may change them; use them for login, not for identity.
- **Federated identities must be linked** to the internal identity via a provider + providerId pair.
- **Support multiple identifiers per identity** — a user might log in via email, Google, or a passkey.
- **Enforce uniqueness** on email, username, and (provider, providerId).

#### Constraints and Limitations

- **Email changes require verification** — an unverified email change can lock a user out.
- **Account merging is complex** — when a user signs up with email and later with Google, the system must detect and merge identities.
- **UUIDs consume more storage** than integers but avoid enumeration attacks.
- **Federated identities depend on the IdP** — if the provider disappears, the user may lose access unless a fallback method exists.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: User Registration with Identity Establishment (NestJS + Prisma)

```typescript
// prisma/schema.prisma (excerpt)
model User {
  id            String   @id @default(uuid())
  email         String   @unique
  emailVerified Boolean  @default(false)
  username      String?  @unique
  passwordHash  String?
  createdAt     DateTime @default(now())
}

// users/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async register(dto: RegisterDto): Promise<User> {
    const normalizedEmail = dto.email.toLowerCase().trim();

    // Check for existing identity
    const existing = await this.prisma.user.findUnique({
      where: { email: normalizedEmail },
    });
    if (existing) {
      throw new ConflictException('Email already registered');
    }

    // Hash password before storage
    const passwordHash = await bcrypt.hash(dto.password, 12);

    // Create the identity with an immutable UUID
    const user = await this.prisma.user.create({
      data: {
        email: normalizedEmail,
        username: dto.username,
        passwordHash,
      },
    });

    // Send verification email (out of band)
    await this.emailService.sendVerification(user.id, user.email);

    return user;
  }
}
```

**Expected behaviour:** Calling `register()` with a new email creates a `User` row with a UUID `id`, a normalized email, and a bcrypt-hashed password. Calling it again with the same email throws `409 Conflict`.

**Why this works:** The `id` is the canonical identity — immutable and unique. Email is a mutable login identifier with a unique constraint. Password is hashed, never stored in plaintext.

### Real-World Cases

- **SaaS platforms:** Users have immutable UUIDs; email addresses change but the ID remains constant for billing, audit logs, and integrations.
- **Federated login:** A user signs in with Google; the system links the Google `sub` claim to the internal UUID.
- **Multi-tenant applications:** Identity is scoped per tenant; the same email may exist in different tenants with different UUIDs.
- **Enterprise SSO:** Identity is provided by the corporate IdP (Okta, Azure AD); the application stores the immutable `sub` claim.

---

## Core Concept 2: Credentials (Verifying Claims)

### Definitions

**Core Definition:** Credentials are the evidence a principal presents to prove their identity — typically a password, cryptographic key, biometric trait, or a combination thereof.

**Technical Definition:** Credentials fall into three categories (authentication factors): **knowledge** (something you know — password, PIN), **possession** (something you have — hardware key, TOTP code, phone), and **inherence** (something you are — fingerprint, face, voice). Credential verification involves comparing the presented credential against a stored representation, using constant-time comparisons to prevent timing attacks. Passwords must be hashed with a memory-hard function (bcrypt, scrypt, Argon2id) — never with fast hashes (MD5, SHA-1, SHA-256) alone. Asymmetric credentials (public/private keys) are verified using digital signatures, never by transmitting the private key.

**Beginner-Friendly Explanation:** Your credentials are the "proof" you present at the door. A password is a secret only you know. A hardware key is a physical object only you possess. A fingerprint is a trait only you have. The system checks your proof against what it stored when you registered. Passwords are never stored directly — they're run through a one-way scrambler (hash) so that even if the database leaks, attackers can't recover the original passwords.

### Purposes

- To verify that a principal is who they claim to be before granting access.
- To resist credential theft (phishing, database breaches, replay attacks).
- To support multiple authentication factors (knowledge, possession, inherence).
- To enable passwordless and key-based authentication.
- To provide a secure, standardised foundation for session and token issuance.

### Syntax Rules and Structure

#### Password Hashing (bcrypt)

```typescript
import bcrypt from 'bcrypt';

// Hashing (during registration)
const passwordHash = await bcrypt.hash(plaintextPassword, 12);

// Verification (during login)
const isValid = await bcrypt.compare(plaintextPassword, storedHash);
```

| Algorithm | Recommended For | Notes |
|-----------|-----------------|-------|
| **Argon2id** | New applications | Winner of Password Hashing Competition; memory-hard. |
| **scrypt** | Node.js built-in | Memory-hard; `crypto.scrypt()` is available natively. |
| **bcrypt** | Legacy and general use | Widely supported; cost factor ≥ 12. |
| **PBKDF2** | FIPS compliance | NIST-approved; use ≥ 600,000 iterations (SHA-256). |
| **MD5/SHA-1/SHA-256** | ❌ Never for passwords | Too fast; vulnerable to GPU brute-force. |

#### Password Policy (OWASP-Aligned)

```typescript
import { z } from 'zod';

export const passwordSchema = z
  .string()
  .min(12, 'Password must be at least 12 characters')
  .max(128, 'Password must be at most 128 characters')
  .refine((pwd) => !commonPasswords.has(pwd), 'Password is too common');
```

| Rule | Rationale |
|------|-----------|
| Minimum 12 characters (or 8 with MFA) | Length is the primary strength factor. |
| Maximum ≥ 64 characters | Allow passphrases; do not truncate. |
| Check against breach lists (HaveIBeenPwned) | Reject known-compromised passwords. |
| Allow all Unicode and spaces | Do not over-restrict character sets. |
| Do not require complexity rules | They encourage predictable patterns (`Password1!`). |
| Do not force periodic rotation | Rotation leads to weak variants (`Password2!`). |

#### Syntax Rules

- **Never store passwords in plaintext** — always hash with a memory-hard function.
- **Use a unique salt per password** — bcrypt, scrypt, and Argon2 handle salts automatically.
- **Use constant-time comparison** for secrets (`crypto.timingSafeEqual`).
- **Never log credentials** — redact passwords, tokens, and secrets from logs.
- **Enforce HTTPS** — credentials transmitted over HTTP can be intercepted.
- **Rate-limit login attempts** — prevent brute-force and credential-stuffing attacks.
- **Do not reveal whether the email or password was wrong** — return a generic "Invalid credentials" message.

#### Constraints and Limitations

- **Passwords are inherently weak** — users reuse them, choose weak ones, and fall for phishing.
- **Hashing is CPU-intensive** — high cost factors can be a DoS vector if not rate-limited.
- **Password managers are essential** — encourage users to use them.
- **Breach detection requires an external service** — HaveIBeenPwned API or a local breach list.
- **Biometrics are irrevocable** — if a fingerprint is compromised, it cannot be changed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Password Registration and Login (Express)

```typescript
// auth/password.service.ts
import bcrypt from 'bcrypt';
import { createHash, timingSafeEqual } from 'node:crypto';

export class PasswordService {
  private readonly COST_FACTOR = 12;

  async hash(password: string): Promise<string> {
    return bcrypt.hash(password, this.COST_FACTOR);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  // Constant-time comparison for non-hashed secrets (e.g., API keys)
  safeCompare(a: string, b: string): boolean {
    const bufA = Buffer.from(a);
    const bufB = Buffer.from(b);
    if (bufA.length !== bufB.length) return false;
    return timingSafeEqual(bufA, bufB);
  }
}
```

```typescript
// auth/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly passwordService: PasswordService,
    private readonly rateLimiter: RateLimiter,
  ) {}

  async register(email: string, password: string): Promise<User> {
    // Validate password against policy
    const result = passwordSchema.safeParse(password);
    if (!result.success) throw new BadRequestException(result.error.issues);

    // Check for breach (HaveIBeenPwned k-anonymity API)
    if (await this.isBreached(password)) {
      throw new BadRequestException('Password has appeared in a data breach');
    }

    const passwordHash = await this.passwordService.hash(password);

    return this.prisma.user.create({
      data: { email: email.toLowerCase().trim(), passwordHash },
    });
  }

  async login(email: string, password: string, ip: string): Promise<Session> {
    // Rate-limit by IP and email to prevent brute-force
    await this.rateLimiter.consume(`login:${ip}:${email}`, 10, 60_000);

    const user = await this.prisma.user.findUnique({
      where: { email: email.toLowerCase().trim() },
    });

    // Always perform a hash comparison, even if user not found,
    // to prevent user enumeration via timing.
    const hash = user?.passwordHash ?? '$2b$12$invalidhashplaceholder';
    const valid = await this.passwordService.verify(password, hash);

    if (!user || !valid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return this.sessionService.create(user.id);
  }

  private async isBreached(password: string): Promise<boolean> {
    const sha1 = createHash('sha1').update(password).digest('hex').toUpperCase();
    const prefix = sha1.slice(0, 5);
    const suffix = sha1.slice(5);

    const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
    const body = await response.text();
    return body.split('\n').some((line) => line.startsWith(suffix));
  }
}
```

**Expected behaviour:**
- `register()` with a weak or breached password throws `400 Bad Request`.
- `register()` with a valid password creates a user with a bcrypt hash (cost 12).
- `login()` with an unknown email still performs a hash comparison (timing-safe).
- `login()` after 10 failed attempts in 60 seconds throws a rate-limit error.

**Why this works:** Passwords are hashed with bcrypt at cost 12. Login always performs a hash comparison, even for unknown users, to prevent timing-based user enumeration. Rate limiting prevents brute-force. Breach detection uses the HaveIBeenPwned k-anonymity API (only the first 5 SHA-1 characters are sent).

### Real-World Cases

- **Consumer applications:** Password + bcrypt with cost 12, breach detection, and rate limiting.
- **Enterprise applications:** Password + enterprise SSO (SAML/OIDC) with fallback to local credentials.
- **APIs:** API keys or client secrets stored as hashes; never as plaintext.
- **Hardware-backed credentials:** FIDO2 keys and TPM-backed certificates.

---

## Core Concept 3: Sessions (Server-Side Tracking)

### Definitions

**Core Definition:** A session is a server-side record of an authenticated user's state, identified by a session ID stored in a cookie, that persists across multiple HTTP requests.

**Technical Definition:** HTTP is stateless; sessions add state by storing a session identifier (typically in an `HttpOnly` cookie) and associating it with server-side data (user ID, roles, expiration). Session stores include in-memory (development only), Redis, Memcached, and database-backed stores. Session cookies must be `HttpOnly`, `Secure` (in production), and `SameSite=Lax` or `Strict`. Session fixation attacks are prevented by regenerating the session ID after successful login. Sessions are invalidated on logout, password change, and (optionally) after a period of inactivity.

**Beginner-Friendly Explanation:** A session is like a coat-check ticket. When you log in, the server gives you a ticket (session ID stored in a cookie). On every subsequent request, you show the ticket, and the server looks up your coat (your authenticated state) in the back room. The server controls everything — you just hold the ticket. If you log out, the server tears up the ticket, and it's useless.

### Purposes

- To maintain authenticated state across stateless HTTP requests.
- To store server-controlled session data (user ID, roles, CSRF tokens).
- To enable immediate invalidation (logout, password change, admin revocation).
- To provide a secure alternative to client-held tokens for browser-based applications.
- To support session fixation prevention via ID regeneration.

### Syntax Rules and Structure

#### Session Cookie Configuration (Express + express-session)

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',            // __Host- prefix for extra security
    cookie: {
      httpOnly: true,
      secure: true,                 // HTTPS only
      sameSite: 'lax',
      maxAge: 1000 * 60 * 60 * 24,  // 24 hours
      domain: undefined,            // No domain (host-only cookie)
      path: '/',
    },
    rolling: true,                  // Reset maxAge on each request
  }),
);
```

| Option | Purpose |
|--------|---------|
| `store` | Session store (Redis, database); never use MemoryStore in production. |
| `secret` | Used to sign the session ID cookie. |
| `resave: false` | Do not save unchanged sessions. |
| `saveUninitialized: false` | Do not create sessions for anonymous users. |
| `name: '__Host-sid'` | `__Host-` prefix enforces Secure + no Domain + Path=/ . |
| `httpOnly` | Prevent JavaScript access (mitigates XSS). |
| `secure` | Send only over HTTPS. |
| `sameSite: 'lax'` | Mitigate CSRF for top-level navigations. |
| `rolling: true` | Refresh session expiration on activity. |

#### Syntax Rules

- **Always use a persistent session store in production** — `MemoryStore` leaks memory and does not scale.
- **Always set `HttpOnly` and `Secure`** on session cookies.
- **Use `SameSite=Lax` or `Strict`** to mitigate CSRF.
- **Use the `__Host-` cookie prefix** — enforces `Secure`, no `Domain`, and `Path=/`.
- **Regenerate the session ID after login** to prevent session fixation.
- **Invalidate sessions on logout, password change, and privilege escalation.**
- **Set an absolute and idle timeout** — sessions should not live forever.
- **Rotate the session secret periodically** and support multiple secrets for zero-downtime rotation.

#### Constraints and Limitations

- **Server-side storage scales horizontally only with a shared store** (Redis, database).
- **Session cookies are sent on every request** — they add overhead to every HTTP call.
- **Sessions do not work well for cross-domain APIs** — tokens are preferred for mobile/SPA clients.
- **Session fixation and CSRF are real risks** if the configuration is incorrect.
- **Server restarts without a persistent store invalidate all sessions.**

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Session Management with Redis (Express)

```typescript
// server.ts
import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';
import csrf from 'csurf';
import helmet from 'helmet';

const app = express();
app.use(helmet());
app.use(express.json());

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 1000 * 60 * 60 * 24, // 24 hours
      path: '/',
    },
    rolling: true,
  }),
);

app.use(csrf());

// --- Login: regenerate session ID ---
app.post('/auth/login', async (req, res, next) => {
  try {
    const user = await authService.verifyCredentials(req.body.email, req.body.password);
    if (!user) return res.status(401).json({ error: 'Invalid credentials' });

    // Regenerate session ID to prevent fixation
    req.session.regenerate((err) => {
      if (err) return next(err);

      req.session.userId = user.id;
      req.session.roles = user.roles;
      req.session.createdAt = Date.now();

      res.json({ user: { id: user.id, email: user.email } });
    });
  } catch (err) {
    next(err);
  }
});

// --- Logout: destroy session ---
app.post('/auth/logout', (req, res, next) => {
  req.session.destroy((err) => {
    if (err) return next(err);
    res.clearCookie('__Host-sid', { path: '/' });
    res.status(204).end();
  });
});

// --- Protected route ---
app.get('/me', requireAuth, (req, res) => {
  res.json({ userId: req.session.userId, roles: req.session.roles });
});

function requireAuth(req, res, next) {
  if (!req.session.userId) return res.status(401).json({ error: 'Unauthorized' });
  next();
}

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `POST /auth/login` with valid credentials regenerates the session ID, stores `userId` and `roles`, and returns user data.
- `GET /me` returns the user data if the session is valid; otherwise returns `401`.
- `POST /auth/logout` destroys the session and clears the cookie.
- The session cookie has `HttpOnly`, `Secure`, and `SameSite=Lax` flags.

**Why this works:** Sessions are stored in Redis (shared, scalable). The session ID is regenerated on login to prevent fixation. The cookie is protected with `HttpOnly`, `Secure`, `SameSite=Lax`, and the `__Host-` prefix. CSRF protection is enabled via `csurf`.

### Real-World Cases

- **Traditional web applications:** Server-rendered apps with session cookies.
- **Admin panels:** Sessions with short idle timeouts and IP binding.
- **Multi-tenant SaaS:** Sessions scoped per tenant; tenant ID stored in the session.
- **Banking applications:** Sessions with strict timeouts, step-up authentication, and device fingerprinting.

---

## Core Concept 4: Tokens (Stateless, Client-Held Proof)

### Definitions

**Core Definition:** A token is a self-contained, cryptographically signed piece of data that a client presents to prove its identity without requiring the server to maintain session state.

**Technical Definition:** Tokens fall into two broad categories: **opaque tokens** (random strings whose meaning is stored server-side) and **structured tokens** (JWTs, which encode claims in a base64url-encoded JSON payload signed with HMAC or RSA/ECDSA). JWTs consist of three parts — header, payload, and signature — and are verified using the signing key. Access tokens are short-lived (5–15 minutes) and carried in the `Authorization: Bearer` header. Refresh tokens are long-lived, stored securely (HttpOnly cookie for browsers), and used to obtain new access tokens. Refresh token rotation and reuse detection are essential security measures.

**Beginner-Friendly Explanation:** A token is like a signed concert ticket. The ticket contains your seat number and the concert details (claims), and it's signed by the venue (server signature). Anyone can read the ticket, but only the venue can create a valid one. You carry the ticket yourself — the venue doesn't keep a list of every ticket holder. If your ticket is stolen, the thief can use it until it expires, which is why tickets are short-lived and why you have a "refresh ticket" to get new ones.

### Purposes

- To enable stateless authentication that scales horizontally without shared session storage.
- To authenticate API clients (mobile apps, SPAs, service-to-service).
- To support cross-domain authentication (tokens in headers, not cookies).
- To carry identity and authorization claims (user ID, roles, scopes).
- To support fine-grained, time-limited access.

### Syntax Rules and Structure

#### JWT Structure

```
header.payload.signature
```

| Part | Contents |
|------|----------|
| **Header** | `{ "alg": "RS256", "typ": "JWT", "kid": "key-1" }` |
| **Payload** | `{ "sub": "user-uuid", "role": "admin", "exp": 1712345678, "iat": 1712342078 }` |
| **Signature** | `RSASHA256(base64url(header) + "." + base64url(payload), privateKey)` |

#### JWT Signing and Verification (TypeScript)

```typescript
import jwt from 'jsonwebtoken';
import { randomUUID } from 'node:crypto';

const ACCESS_TOKEN_TTL = '15m';
const REFRESH_TOKEN_TTL = '7d';

export class TokenService {
  constructor(
    private readonly privateKey: string,
    private readonly publicKey: string,
  ) {}

  signAccessToken(userId: string, roles: string[], sessionId: string): string {
    return jwt.sign(
      {
        sub: userId,
        roles,
        sid: sessionId,   // Bind token to a session for revocation
        type: 'access',
      },
      this.privateKey,
      {
        algorithm: 'RS256',
        expiresIn: ACCESS_TOKEN_TTL,
        issuer: 'https://api.example.com',
        audience: 'https://app.example.com',
        jwtid: randomUUID(),
      },
    );
  }

  signRefreshToken(userId: string, tokenId: string): string {
    return jwt.sign(
      { sub: userId, type: 'refresh', jti: tokenId },
      this.privateKey,
      {
        algorithm: 'RS256',
        expiresIn: REFRESH_TOKEN_TTL,
        issuer: 'https://api.example.com',
      },
    );
  }

  verify(token: string): jwt.JwtPayload {
    return jwt.verify(token, this.publicKey, {
      algorithms: ['RS256'],   // Prevent algorithm confusion attacks
      issuer: 'https://api.example.com',
      audience: 'https://app.example.com',
    }) as jwt.JwtPayload;
  }
}
```

#### Refresh Token Rotation (Database Schema)

```prisma
model RefreshToken {
  id         String   @id @default(uuid())
  userId     String
  tokenHash  String   @unique   // Store hash, not the raw token
  family     String              // Token family for reuse detection
  expiresAt  DateTime
  revokedAt  DateTime?
  replacedBy String?             // ID of the token that replaced this one
  createdAt  DateTime @default(now())
  user       User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([family])
}
```

#### Syntax Rules

- **Use asymmetric algorithms (RS256, ES256)** for multi-service architectures.
- **Use symmetric algorithms (HS256)** only when the same service signs and verifies.
- **Never accept the `alg` from the token header** — always specify allowed algorithms.
- **Always validate `iss`, `aud`, `exp`, and `nbf`.**
- **Keep access tokens short-lived** (5–15 minutes).
- **Store refresh tokens securely** — HttpOnly cookies for browsers, secure storage for mobile.
- **Implement refresh token rotation** — issue a new refresh token on each use.
- **Detect refresh token reuse** — if a used token is presented again, revoke the entire token family.
- **Never store sensitive data in JWT payloads** — payloads are base64url-encoded, not encrypted.

#### Constraints and Limitations

- **JWTs cannot be revoked** without a denylist or short TTLs.
- **Token theft is possible** — XSS can steal tokens from `localStorage`; prefer HttpOnly cookies.
- **JWTs grow with claims** — large payloads increase request size.
- **Clock skew** can cause valid tokens to be rejected; allow a small leeway.
- **Algorithm confusion attacks** — always specify allowed algorithms during verification.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Access + Refresh Token Flow with Rotation (NestJS)

```typescript
// auth/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly tokenService: TokenService,
    private readonly passwordService: PasswordService,
  ) {}

  async login(email: string, password: string): Promise<LoginResult> {
    const user = await this.prisma.user.findUnique({ where: { email } });
    if (!user || !(await this.passwordService.verify(password, user.passwordHash))) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const tokenId = randomUUID();
    const accessToken = this.tokenService.signAccessToken(user.id, user.roles, tokenId);
    const refreshToken = this.tokenService.signRefreshToken(user.id, tokenId);

    // Store refresh token hash and family
    await this.prisma.refreshToken.create({
      data: {
        id: tokenId,
        userId: user.id,
        tokenHash: await bcrypt.hash(refreshToken, 10),
        family: tokenId,
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      },
    });

    return { accessToken, refreshToken };
  }

  async refresh(refreshToken: string): Promise<LoginResult> {
    // 1. Verify signature
    const payload = this.tokenService.verify(refreshToken);

    // 2. Look up the token record
    const record = await this.prisma.refreshToken.findUnique({
      where: { id: payload.jti },
    });

    // 3. Reuse detection: if already revoked, revoke the entire family
    if (!record || record.revokedAt) {
      if (record) {
        await this.prisma.refreshToken.updateMany({
          where: { family: record.family },
          data: { revokedAt: new Date() },
        });
      }
      throw new UnauthorizedException('Refresh token reuse detected');
    }

    // 4. Verify hash
    if (!(await bcrypt.compare(refreshToken, record.tokenHash))) {
      throw new UnauthorizedException('Invalid refresh token');
    }

    // 5. Rotate: revoke old, issue new
    const newTokenId = randomUUID();
    const newAccessToken = this.tokenService.signAccessToken(
      payload.sub!,
      payload.roles as string[],
      newTokenId,
    );
    const newRefreshToken = this.tokenService.signRefreshToken(payload.sub!, newTokenId);

    await this.prisma.$transaction([
      this.prisma.refreshToken.update({
        where: { id: record.id },
        data: { revokedAt: new Date(), replacedBy: newTokenId },
      }),
      this.prisma.refreshToken.create({
        data: {
          id: newTokenId,
          userId: record.userId,
          tokenHash: await bcrypt.hash(newRefreshToken, 10),
          family: record.family,   // Same family for reuse detection
          expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
        },
      }),
    ]);

    return { accessToken: newAccessToken, refreshToken: newRefreshToken };
  }
}
```

**Expected behaviour:**
- `login()` issues an access token (15 min) and a refresh token (7 days).
- `refresh()` verifies the refresh token, rotates it (revokes old, issues new), and returns a new pair.
- If a revoked refresh token is presented, the entire token family is revoked (reuse detection).
- Access tokens are verified statelessly — no database lookup required.

**Why this works:** Access tokens are short-lived and stateless. Refresh tokens are stored as hashes (never plaintext). Rotation ensures that stolen refresh tokens are usable only once. Reuse detection revokes the entire family, mitigating token theft.

### Real-World Cases

- **SPAs and mobile apps:** Access tokens in memory, refresh tokens in HttpOnly cookies.
- **API-to-API communication:** Service tokens with short TTLs and scoped permissions.
- **Microservices:** JWTs propagate identity across services without shared session storage.
- **Third-party integrations:** OAuth 2.0 access tokens with scoped permissions.

---

## Core Concept 5: Multi-Factor Authentication (MFA)

### Definitions

**Core Definition:** Multi-Factor Authentication (MFA) requires a user to present two or more distinct authentication factors — knowledge, possession, or inherence — to prove their identity.

**Technical Definition:** MFA strengthens authentication by combining factors from different categories. TOTP (Time-based One-Time Password, RFC 6238) is the most common possession factor: a shared secret is provisioned to an authenticator app, which generates a 6-digit code every 30 seconds using HMAC-SHA1. SMS-based OTP is a weaker fallback due to SIM-swapping and SS7 attacks. Push-based MFA (Duo, Okta Verify) is more secure and user-friendly. Recovery codes are essential to prevent lockout. MFA should be required for sensitive operations (login, password change, payment) and enforced via step-up authentication.

**Beginner-Friendly Explanation:** MFA is like requiring both a key and a fingerprint to open a safe. A password alone is "something you know" — if someone learns it, they're in. MFA adds "something you have" (your phone generating a code) or "something you are" (your fingerprint). Even if your password is stolen, the attacker can't get in without the second factor.

### Purposes

- To significantly reduce the risk of account compromise from stolen passwords.
- To protect against credential stuffing, phishing, and brute-force attacks.
- To meet compliance requirements (PCI DSS, HIPAA, SOC 2, GDPR).
- To provide step-up authentication for sensitive operations.
- To enable secure account recovery when the primary factor is unavailable.

### Syntax Rules and Structure

#### TOTP Setup (otplib + qrcode)

```typescript
import { authenticator } from 'otplib';
import QRCode from 'qrcode';

export class TotpService {
  // 1. Generate a secret during MFA enrollment
  generateSecret(): string {
    return authenticator.generateSecret(); // Base32-encoded
  }

  // 2. Generate the otpauth:// URI for the QR code
  generateUri(secret: string, email: string): string {
    return authenticator.keyuri(email, 'MyApp', secret);
  }

  // 3. Render the QR code as a data URL
  async generateQrCode(uri: string): Promise<string> {
    return QRCode.toDataURL(uri);
  }

  // 4. Verify a TOTP code
  verify(code: string, secret: string): boolean {
    return authenticator.verify({ token: code, secret });
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `authenticator.generateSecret()` | Generates a base32 secret (default 20 bytes). |
| `authenticator.keyuri()` | Builds the `otpauth://totp/...` URI. |
| `QRCode.toDataURL()` | Renders the URI as a QR code for scanning. |
| `authenticator.verify()` | Verifies a 6-digit code against the secret. |
| `window` | Optional tolerance (default 0; 1 allows ±30 seconds). |

#### TOTP Enrollment Flow

```typescript
// auth/mfa.service.ts
@Injectable()
export class MfaService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly totp: TotpService,
    private readonly encryption: EncryptionService,
  ) {}

  async beginEnrollment(userId: string): Promise<{ qrCode: string; secret: string }> {
    const user = await this.prisma.user.findUniqueOrThrow({ where: { id: userId } });
    const secret = this.totp.generateSecret();
    const uri = this.totp.generateUri(secret, user.email);
    const qrCode = await this.totp.generateQrCode(uri);

    // Store secret temporarily (encrypted) until verified
    await this.prisma.user.update({
      where: { id: userId },
      data: {
        totpSecret: this.encryption.encrypt(secret),
        mfaEnabled: false, // Not enabled until verified
      },
    });

    return { qrCode, secret };
  }

  async confirmEnrollment(userId: string, code: string): Promise<{ recoveryCodes: string[] }> {
    const user = await this.prisma.user.findUniqueOrThrow({ where: { id: userId } });
    const secret = this.encryption.decrypt(user.totpSecret!);

    if (!this.totp.verify(code, secret)) {
      throw new BadRequestException('Invalid TOTP code');
    }

    // Generate recovery codes
    const recoveryCodes = Array.from({ length: 10 }, () =>
      randomBytes(8).toString('hex'),
    );

    await this.prisma.user.update({
      where: { id: userId },
      data: {
        mfaEnabled: true,
        recoveryCodes: recoveryCodes.map((c) => bcrypt.hashSync(c, 10)),
      },
    });

    return { recoveryCodes };
  }
}
```

#### Syntax Rules

- **Store TOTP secrets encrypted at rest** — never plaintext.
- **Use `window: 1`** to allow for clock skew (±30 seconds).
- **Rate-limit TOTP verification attempts** — prevent brute-force of 6-digit codes.
- **Generate recovery codes** during enrollment — 8–10 single-use codes.
- **Hash recovery codes** before storing them.
- **Require MFA for sensitive operations** (password change, email change, payment).
- **Prefer TOTP over SMS** — SMS is vulnerable to SIM-swapping and SS7 attacks.
- **Use push-based MFA** (Duo, Okta Verify) for the best UX and security.
- **Bind MFA to the user, not the session** — MFA status persists across sessions.

#### Constraints and Limitations

- **TOTP requires a shared secret** — if the secret leaks, the factor is compromised.
- **Clock drift** can cause valid codes to be rejected; use `window: 1`.
- **SMS is insecure** — NIST no longer recommends SMS as a primary MFA factor.
- **Recovery codes must be single-use** and stored securely.
- **MFA enrollment requires a fallback** — if the user loses their device, recovery codes or a support process is needed.
- **Step-up authentication adds friction** — balance security with UX.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full TOTP MFA Flow (NestJS)

```typescript
// auth/mfa.controller.ts
@Controller('auth/mfa')
export class MfaController {
  constructor(private readonly mfaService: MfaService) {}

  @Post('enroll')
  @UseGuards(JwtAuthGuard)
  async enroll(@CurrentUser() user: AuthUser): Promise<{ qrCode: string; secret: string }> {
    return this.mfaService.beginEnrollment(user.id);
  }

  @Post('confirm')
  @UseGuards(JwtAuthGuard)
  async confirm(
    @CurrentUser() user: AuthUser,
    @Body() dto: ConfirmMfaDto,
  ): Promise<{ recoveryCodes: string[] }> {
    return this.mfaService.confirmEnrollment(user.id, dto.code);
  }

  @Post('verify')
  @HttpCode(200)
  @UseGuards(JwtAuthGuard)
  async verify(
    @CurrentUser() user: AuthUser,
    @Body() dto: VerifyMfaDto,
  ): Promise<{ accessToken: string }> {
    return this.mfaService.verifyChallenge(user.id, dto.code);
  }
}
```

```typescript
// auth/mfa.service.ts (continued)
async verifyChallenge(userId: string, code: string): Promise<{ accessToken: string }> {
  await this.rateLimiter.consume(`mfa:${userId}`, 5, 60_000);

  const user = await this.prisma.user.findUniqueOrThrow({ where: { id: userId } });
  if (!user.mfaEnabled || !user.totpSecret) {
    throw new BadRequestException('MFA is not enabled');
  }

  const secret = this.encryption.decrypt(user.totpSecret);
  const isValid = this.totp.verify(code, secret);

  if (!isValid) {
    // Check recovery codes
    const isRecovery = user.recoveryCodes.some((hash) => bcrypt.compareSync(code, hash));
    if (!isRecovery) throw new UnauthorizedException('Invalid MFA code');

    // Consume the recovery code
    await this.prisma.user.update({
      where: { id: userId },
      data: {
        recoveryCodes: user.recoveryCodes.filter((hash) => !bcrypt.compareSync(code, hash)),
      },
    });
  }

  // Issue a full access token (MFA satisfied)
  return { accessToken: this.tokenService.signAccessToken(user.id, user.roles, randomUUID()) };
}
```

**Expected behaviour:**
- `POST /auth/mfa/enroll` returns a QR code and secret for the authenticator app.
- `POST /auth/mfa/confirm` verifies the first TOTP code and returns recovery codes.
- `POST /auth/mfa/verify` verifies a TOTP code (or recovery code) and issues an access token.
- Failed attempts are rate-limited (5 per minute).
- Recovery codes are single-use and consumed on use.

**Why this works:** TOTP secrets are encrypted at rest. Verification is rate-limited. Recovery codes provide a fallback. The challenge flow issues a full access token only after MFA is satisfied.

### Real-World Cases

- **Banking:** MFA required for login and every transaction above a threshold.
- **Enterprise SaaS:** TOTP or push-based MFA enforced by policy for all users.
- **Healthcare:** MFA required for accessing patient records (HIPAA compliance).
- **Cloud providers:** AWS, GCP, and Azure require MFA for privileged operations.

---

## Core Concept 6: Passwordless Authentication & Passkeys

### Definitions

**Core Definition:** Passwordless authentication eliminates passwords entirely, using cryptographic keys, biometrics, or one-time codes to authenticate users. Passkeys are a specific implementation of WebAuthn that sync across devices via the user's cloud account.

**Technical Definition:** WebAuthn (Web Authentication API, W3C standard) enables servers to register and authenticate users using public-key cryptography. During registration, the authenticator (hardware key, platform authenticator, or biometric sensor) generates a key pair; the public key is sent to the server, and the private key never leaves the authenticator. During authentication, the server sends a challenge; the authenticator signs it with the private key; the server verifies the signature with the stored public key. FIDO2 is the umbrella standard combining WebAuthn (client-side) and CTAP2 (authenticator-side). Passkeys are discoverable WebAuthn credentials that sync across devices via iCloud Keychain, Google Password Manager, or Windows Hello. Passkeys are phishing-resistant because the credential is bound to the origin (relying party ID).

**Beginner-Friendly Explanation:** Passwords are secrets you share with a website. Passkeys are the opposite: you never share a secret. Instead, your device generates a unique key pair for each website. The public key goes to the website; the private key stays on your device (or in your cloud keychain). When you log in, the website sends a challenge, your device signs it with the private key (often after a fingerprint or face scan), and the website verifies the signature. There's no password to steal, no code to phish, and no secret to reuse.

### Purposes

- To eliminate passwords and their associated risks (reuse, phishing, breaches).
- To provide phishing-resistant authentication (credentials are bound to the origin).
- To improve user experience (biometric login, no passwords to remember).
- To meet modern security standards (FIDO2, WebAuthn, NIST 800-63B).
- To enable cross-device authentication via passkey syncing.

### Syntax Rules and Structure

#### WebAuthn Registration (Server-Side with SimpleWebAuthn)

```typescript
import {
  generateRegistrationOptions,
  verifyRegistrationResponse,
  generateAuthenticationOptions,
  verifyAuthenticationResponse,
} from '@simplewebauthn/server';

// 1. Generate registration options
async function beginRegistration(userId: string) {
  const user = await prisma.user.findUniqueOrThrow({ where: { id: userId } });
  const userPasskeys = await prisma.webAuthnCredential.findMany({ where: { userId } });

  const options = await generateRegistrationOptions({
    rpName: 'MyApp',
    rpID: 'example.com',                // Relying Party ID (domain)
    userID: Buffer.from(user.id),
    userName: user.email,
    attestationType: 'none',            // 'none' for most use cases
    excludeCredentials: userPasskeys.map((pk) => ({
      id: pk.credentialId,
      transports: pk.transports,
    })),
    authenticatorSelection: {
      residentKey: 'required',          // Discoverable credential (passkey)
      userVerification: 'preferred',    // Require biometric/PIN if available
    },
  });

  // Store challenge in session for verification
  await prisma.user.update({
    where: { id: userId },
    data: { currentChallenge: options.challenge },
  });

  return options;
}

// 2. Verify registration response
async function finishRegistration(userId: string, response: RegistrationResponseJSON) {
  const user = await prisma.user.findUniqueOrThrow({ where: { id: userId } });

  const verification = await verifyRegistrationResponse({
    response,
    expectedChallenge: user.currentChallenge!,
    expectedOrigin: 'https://example.com',
    expectedRPID: 'example.com',
  });

  if (!verification.verified || !verification.registrationInfo) {
    throw new BadRequestException('Registration failed');
  }

  const { credential, credentialDeviceType, credentialBackedUp } = verification.registrationInfo;

  await prisma.webAuthnCredential.create({
    data: {
      userId,
      credentialId: credential.id,
      publicKey: Buffer.from(credential.publicKey),
      counter: credential.counter,
      transports: response.response.transports ?? [],
      deviceType: credentialDeviceType,
      backedUp: credentialBackedUp,
    },
  });

  return { verified: true };
}
```

#### WebAuthn Authentication

```typescript
// 3. Generate authentication options
async function beginAuthentication(email: string) {
  const user = await prisma.user.findUnique({ where: { email } });
  if (!user) throw new UnauthorizedException();

  const userPasskeys = await prisma.webAuthnCredential.findMany({ where: { userId: user.id } });

  const options = await generateAuthenticationOptions({
    rpID: 'example.com',
    allowCredentials: userPasskeys.map((pk) => ({
      id: pk.credentialId,
      transports: pk.transports,
    })),
    userVerification: 'preferred',
  });

  await prisma.user.update({
    where: { id: user.id },
    data: { currentChallenge: options.challenge },
  });

  return options;
}

// 4. Verify authentication response
async function finishAuthentication(email: string, response: AuthenticationResponseJSON) {
  const user = await prisma.user.findUniqueOrThrow({ where: { email } });
  const credential = await prisma.webAuthnCredential.findUniqueOrThrow({
    where: { credentialId: response.id },
  });

  const verification = await verifyAuthenticationResponse({
    response,
    expectedChallenge: user.currentChallenge!,
    expectedOrigin: 'https://example.com',
    expectedRPID: 'example.com',
    credential: {
      id: credential.credentialId,
      publicKey: credential.publicKey,
      counter: credential.counter,
    },
  });

  if (!verification.verified) throw new UnauthorizedException();

  // Update counter to prevent replay attacks
  await prisma.webAuthnCredential.update({
    where: { id: credential.id },
    data: { counter: verification.authenticationInfo.newCounter },
  });

  return { verified: true, userId: user.id };
}
```

#### Syntax Rules

- **`rpID` must match the domain** — `example.com` covers `app.example.com` but not `other.com`.
- **Challenges must be random, single-use, and stored server-side** — prevents replay.
- **Store the public key and credential ID** — never the private key (it never leaves the authenticator).
- **Update the signature counter** on each authentication — detects cloned authenticators.
- **Use `residentKey: 'required'`** for passkeys (discoverable credentials).
- **Use `userVerification: 'preferred'`** — require biometric/PIN where available.
- **Provide fallback authentication** — not all devices support WebAuthn.
- **Support multiple passkeys per user** — users may have multiple devices.
- **Allow passkey deletion** — users must be able to revoke lost devices.

#### Constraints and Limitations

- **WebAuthn requires HTTPS** (except on `localhost`).
- **Not all browsers or devices support WebAuthn** — fallback methods are essential.
- **Passkey syncing depends on the platform** — iCloud Keychain, Google Password Manager, etc.
- **Cross-device authentication** requires either a synced passkey or a hybrid transport (QR code + Bluetooth).
- **Account recovery is challenging** — if all passkeys are lost, recovery requires a fallback.
- **Enterprise environments** may restrict passkey syncing (managed vs. unmanaged devices).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Passkey Registration and Authentication (NestJS + SimpleWebAuthn)

```typescript
// auth/webauthn.controller.ts
@Controller('auth/webauthn')
export class WebAuthnController {
  constructor(private readonly webAuthnService: WebAuthnService) {}

  @Post('register/begin')
  @UseGuards(JwtAuthGuard)
  async beginRegistration(@CurrentUser() user: AuthUser) {
    return this.webAuthnService.beginRegistration(user.id);
  }

  @Post('register/finish')
  @UseGuards(JwtAuthGuard)
  async finishRegistration(
    @CurrentUser() user: AuthUser,
    @Body() body: RegistrationResponseJSON,
  ) {
    return this.webAuthnService.finishRegistration(user.id, body);
  }

  @Post('login/begin')
  async beginLogin(@Body() dto: BeginLoginDto) {
    return this.webAuthnService.beginAuthentication(dto.email);
  }

  @Post('login/finish')
  async finishLogin(@Body() dto: FinishLoginDto) {
    return this.webAuthnService.finishAuthentication(dto.email, dto.response);
  }
}
```

```typescript
// prisma/schema.prisma (excerpt)
model WebAuthnCredential {
  id           String   @id @default(uuid())
  userId       String
  credentialId String   @unique
  publicKey    Bytes
  counter      BigInt
  transports   String[]
  deviceType   String
  backedUp     Boolean
  createdAt    DateTime @default(now())
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}
```

**Expected behaviour:**
- `POST /auth/webauthn/register/begin` returns registration options (challenge, rp, user).
- The browser calls `navigator.credentials.create()` with these options.
- `POST /auth/webauthn/register/finish` verifies the response and stores the public key.
- `POST /auth/webauthn/login/begin` returns authentication options.
- The browser calls `navigator.credentials.get()`.
- `POST /auth/webauthn/login/finish` verifies the signature and issues a session or token.

**Why this works:** The challenge is stored server-side and verified against the response. The public key is stored; the private key never leaves the authenticator. The counter is updated to detect cloned authenticators. Passkeys are discoverable (resident keys) and sync across devices via the platform.

### Real-World Cases

- **Consumer apps:** Passkeys for login on iOS, Android, and web (Google, Apple, Microsoft support).
- **Enterprise:** FIDO2 hardware keys (YubiKey) for privileged access.
- **Banking:** Passkeys for high-value transactions.
- **E-commerce:** Passkeys for frictionless checkout (Shopify, Stripe support passkeys).

---

## References

- NIST SP 800-63B — Digital Identity Guidelines: Authentication and Lifecycle Management — https://pages.nist.gov/800-63-3/sp800-63b.html
- OWASP Cheat Sheet Series — Authentication — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Password Storage — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Session Management — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Multifactor Authentication — https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html
- W3C — Web Authentication: An API for accessing Public Key Credentials (WebAuthn) — https://www.w3.org/TR/webauthn-2/
- FIDO Alliance — FIDO2: WebAuthn & CTAP — https://fidoalliance.org/fido2/
- RFC 6238 — TOTP: Time-Based One-Time Password Algorithm — https://www.rfc-editor.org/rfc/rfc6238
- RFC 4226 — HOTP: An HMAC-Based One-Time Password Algorithm — https://www.rfc-editor.org/rfc/rfc4226
- RFC 7519 — JSON Web Token (JWT) — https://www.rfc-editor.org/rfc/rfc7519
- RFC 6749 — The OAuth 2.0 Authorization Framework — https://www.rfc-editor.org/rfc/rfc6749
- OpenID Connect Core 1.0 — https://openid.net/specs/openid-connect-core-1_0.html
- SimpleWebAuthn Documentation — https://simplewebauthn.dev/
- Passkeys.dev — https://passkeys.dev/
- Auth0 — Passwordless Authentication — https://auth0.com/passwordless
- Auth0 — Multi-Factor Authentication — https://auth0.com/docs/secure/multi-factor-authentication
- HaveIBeenPwned — Pwned Passwords API — https://haveibeenpwned.com/API/v3#PwnedPasswords
- OWASP — Credential Stuffing Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
- OWASP — Forgot Password Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
- Microsoft — Passkeys (FIDO2) in Microsoft Entra ID — https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passkeys-fido2
- Node.js Documentation — `crypto.timingSafeEqual()` — https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b
- Node.js Documentation — `crypto.scrypt()` — https://nodejs.org/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback
- Argon2 — Password Hashing Competition Winner — https://www.argon2.com/
- bcrypt — npm package — https://www.npmjs.com/package/bcrypt
- otplib — npm package — https://www.npmjs.com/package/otplib