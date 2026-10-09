# Session Authentication — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Session authentication is a stateful authentication mechanism where the server creates and stores a session record after a user logs in, and the client holds only a session identifier (typically in a cookie) that references that server-side record on subsequent requests.

**Technical Definition:** Session authentication maintains server-side state keyed by a session identifier (session ID). Upon successful authentication, the server generates a cryptographically random session ID, stores session data (user ID, roles, timestamps, metadata) in a session store (in-memory, Redis, Memcached, or a relational database), and sends the session ID to the client via a `Set-Cookie` header. On each subsequent request, the client sends the session cookie, the server looks up the session record, and if valid, treats the request as authenticated. Session security depends on cookie flags (`HttpOnly`, `Secure`, `SameSite`), session ID entropy (≥128 bits), regeneration on privilege changes, expiration policies (absolute and idle timeouts), and revocation mechanisms.

**Beginner-Friendly Explanation:** Think of a session like a coat-check ticket at a restaurant. When you arrive (log in), the host takes your coat (your credentials), gives you a numbered ticket (session ID), and stores your coat in the back room (session store). Every time you want something, you show your ticket, and the staff retrieves your coat from the back. The staff controls everything — you just hold the ticket. If you leave (log out), the staff throws away your coat and your ticket becomes worthless. If you lose your ticket, nobody else can use it because it's just a random number that only the staff can match to a coat.

### Key Characteristics

- **Server-controlled state:** The server owns the session data; the client holds only an opaque identifier.
- **Immediate revocation:** Sessions can be invalidated instantly (logout, admin action, password change).
- **Cookie-based transport:** Session IDs are typically transported in cookies with strict security flags.
- **Cryptographic randomness:** Session IDs must be generated with a CSPRNG and have ≥128 bits of entropy.
- **Store flexibility:** Sessions can be stored in memory (development), Redis (production), Memcached, or relational databases.
- **Expiration policies:** Absolute timeouts (maximum lifetime) and idle timeouts (sliding window) control session duration.
- **Fixation and hijacking defenses:** Session ID regeneration and fingerprinting prevent common attacks.

### Prerequisites

- **HTTP fundamentals:** Statelessness, cookies, headers, status codes.
- **Cookie mechanics:** `Set-Cookie`, `Cookie`, `HttpOnly`, `Secure`, `SameSite`, `Domain`, `Path`, `Max-Age`.
- **Cryptography basics:** CSPRNG (`crypto.randomBytes`), hashing, HMAC.
- **Session store familiarity:** Redis, Memcached, or database-backed stores.
- **Web security concepts:** CSRF, XSS, session fixation, session hijacking, timing attacks.
- **Node.js fundamentals:** Express or NestJS, middleware, async patterns.
- **HTTPS/TLS:** All session cookies must be transmitted over HTTPS in production.

### Related Programming Areas

- **Authentication:** Sessions are created after credential verification (password, MFA, passkey).
- **Authorization:** Session data often includes roles and permissions used for access control.
- **CSRF Protection:** Session cookies require CSRF tokens (double-submit or synchronizer pattern).
- **Rate Limiting:** Login and session endpoints must be rate-limited to prevent brute-force.
- **Token Authentication:** Sessions and tokens are alternative approaches; hybrid designs exist.
- **Compliance:** PCI DSS, HIPAA, SOC 2, and GDPR mandate session security controls.

### Core Concepts

1. **Sessions** — storing authenticated state securely in server-side memory or distributed stores.
2. **Cookies** — managing browser storage triggers securely using `HttpOnly`, `Secure`, and `SameSite` flags.
3. **Session Storage** — configuring state stores like Redis, Memcached, or relational databases.
4. **Session Expiration** — implementing absolute timeouts vs. sliding window inactivity expirations.
5. **Session Revocation** — building mechanisms to terminate specific sessions or clear a user's active logins globally.
6. **Session Fixation & Hijacking Protections** — regenerating session IDs upon privilege changes and fingerprinting client attributes.

---

## Core Concept 1: Sessions (Server-Side State)

### Definitions

**Core Definition:** A session is a server-side record of an authenticated user's state, identified by a unique session ID, that persists across multiple HTTP requests.

**Technical Definition:** A session is created when a user successfully authenticates. The server generates a cryptographically random session ID (≥128 bits), stores a session record containing the user ID, roles, timestamps, and metadata in a session store, and sends the session ID to the client via a `Set-Cookie` header. The session record is the authoritative source of authentication state — the client's cookie is only a lookup key. Sessions are inherently stateful, requiring shared storage for horizontal scaling. Session data should be minimal (user ID, session metadata) — large objects should be loaded from the database on demand.

**Beginner-Friendly Explanation:** A session is the server's memory of who you are. When you log in, the server writes down "Alice is logged in, here's her user ID and when she logged in" and gives you a numbered ticket. On every request, you show the ticket, and the server looks up its notes. Unlike tokens, where the client carries all the proof, sessions keep the proof on the server — which means the server can revoke access instantly.

### Purposes

- To maintain authenticated state across stateless HTTP requests.
- To provide immediate session revocation (logout, admin action, password change).
- To store server-controlled session data that the client cannot tamper with.
- To enable session-based CSRF protection and session fingerprinting.
- To support single sign-on within a single domain (or across subdomains with shared cookies).

### Syntax Rules and Structure

#### Session Creation (Express + express-session)

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';
import { randomBytes } from 'node:crypto';

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',
    genid: () => randomBytes(32).toString('base64url'), // 256 bits
    cookie: {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      maxAge: 1000 * 60 * 60 * 24, // 24 hours
      path: '/',
    },
    rolling: true,
  }),
);
```

| Component | Breakdown |
|-----------|-----------|
| `store` | Session store (Redis, Memcached, database). |
| `secret` | Signs the session ID cookie. |
| `resave: false` | Do not save unchanged sessions. |
| `saveUninitialized: false` | Do not create sessions for anonymous users. |
| `genid` | Custom session ID generator (CSPRNG). |
| `name: '__Host-sid'` | `__Host-` prefix enforces Secure + no Domain + Path=/. |
| `rolling: true` | Refresh session expiration on activity. |

#### Session Record Structure

```typescript
interface SessionRecord {
  id: string;              // Session ID (also the key)
  userId: string;          // Authenticated user
  roles: string[];         // Cached roles for authorization
  createdAt: number;       // Absolute timeout tracking
  lastActivityAt: number;  // Idle timeout tracking
  ip: string;              // Client IP (fingerprinting)
  userAgent: string;       // Client User-Agent (fingerprinting)
  csrfToken: string;       // CSRF token (synchronizer pattern)
}
```

#### Syntax Rules

- **Session IDs must be generated with a CSPRNG** — `crypto.randomBytes(32)` (256 bits).
- **Session IDs must be opaque** — no user data encoded in the ID.
- **Session data must be minimal** — store `userId`, `roles`, timestamps, and metadata; not the full user object.
- **Sessions must be stored server-side** — never trust client-side session data.
- **Session stores must be shared** — in-memory stores do not scale horizontally.
- **Session secrets must be strong** — ≥256 bits, stored in environment variables or a secrets manager.
- **Session IDs must be rotated on login** — prevents fixation.

#### Constraints and Limitations

- **Stateful by design** — requires shared storage for horizontal scaling.
- **Cookie dependency** — sessions require cookie support; some API clients do not support cookies.
- **Cross-domain limitations** — cookies are scoped to a domain; cross-domain SSO requires additional mechanisms.
- **Session store availability** — if the store is down, all users are logged out.
- **Session store size** — sessions accumulate; expired sessions must be cleaned up.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Session Creation and Authentication (Express)

```typescript
// server.ts
import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';
import { randomBytes } from 'node:crypto';
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
    genid: () => randomBytes(32).toString('base64url'),
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 1000 * 60 * 60 * 24,
      path: '/',
    },
    rolling: true,
  }),
);

// --- Login: create session ---
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
      req.session.lastActivityAt = Date.now();
      req.session.ip = req.ip;
      req.session.userAgent = req.get('User-Agent') ?? '';

      res.json({ user: { id: user.id, email: user.email } });
    });
  } catch (err) {
    next(err);
  }
});

// --- Protected route ---
app.get('/me', requireAuth, (req, res) => {
  res.json({ userId: req.session.userId, roles: req.session.roles });
});

function requireAuth(req, res, next) {
  if (!req.session.userId) return res.status(401).json({ error: 'Unauthorized' });

  // Fingerprint check
  if (req.session.ip !== req.ip || req.session.userAgent !== req.get('User-Agent')) {
    req.session.destroy(() => {});
    return res.status(401).json({ error: 'Session invalid' });
  }

  next();
}

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `POST /auth/login` with valid credentials creates a session, stores it in Redis, and sets a `__Host-sid` cookie.
- `GET /me` returns the user data if the session is valid and the fingerprint matches.
- If the IP or User-Agent changes, the session is destroyed and `401` is returned.

**Why this works:** The session ID is generated with `randomBytes(32)` (256 bits). The session is stored in Redis (shared, scalable). The cookie is protected with `HttpOnly`, `Secure`, `SameSite=Lax`, and the `__Host-` prefix. The session ID is regenerated on login to prevent fixation. Fingerprinting detects session hijacking.

### Real-World Cases

- **Traditional web applications:** Server-rendered apps with session cookies.
- **Admin panels:** Sessions with short idle timeouts and IP binding.
- **Multi-tenant SaaS:** Sessions scoped per tenant; tenant ID stored in the session.
- **Banking applications:** Sessions with strict timeouts, step-up authentication, and device fingerprinting.

---

## Core Concept 2: Cookies (Secure Browser Storage)

### Definitions

**Core Definition:** Cookies are small pieces of data stored by the browser and automatically sent with every request to the same origin, used to transport session IDs securely when configured with appropriate flags.

**Technical Definition:** Cookies are set via the `Set-Cookie` response header and sent back via the `Cookie` request header. Security-critical attributes include: `HttpOnly` (inaccessible to JavaScript, mitigating XSS), `Secure` (sent only over HTTPS), `SameSite` (`Strict`, `Lax`, or `None`; mitigates CSRF), `Domain` (scope), `Path` (scope), `Max-Age`/`Expires` (lifetime), and the `__Host-` prefix (enforces Secure + no Domain + Path=/). Session cookies should use `HttpOnly; Secure; SameSite=Lax` (or `Strict` for high-security applications). The `__Host-` prefix provides the strongest cookie security by preventing subdomain attacks.

**Beginner-Friendly Explanation:** A cookie is like a name tag the browser wears. Every time the browser visits your site, it shows the name tag automatically. If the name tag is marked `HttpOnly`, JavaScript can't read it (protecting against XSS). If it's marked `Secure`, it's only shown over HTTPS. If it's marked `SameSite=Lax`, the browser won't show it when visiting from another site (protecting against CSRF). The `__Host-` prefix is like a tamper-proof seal — it guarantees the cookie was set by your exact domain and can't be overridden by a subdomain.

### Purposes

- To transport session IDs between the client and server automatically.
- To protect session cookies from XSS via `HttpOnly`.
- To protect session cookies from network interception via `Secure`.
- To protect session cookies from CSRF via `SameSite`.
- To scope cookies to specific domains and paths.
- To control cookie lifetime via `Max-Age` and `Expires`.

### Syntax Rules and Structure

#### Cookie Attributes

| Attribute | Purpose | Recommended Value |
|-----------|---------|-------------------|
| `HttpOnly` | Block JavaScript access (XSS mitigation) | Always set |
| `Secure` | Send only over HTTPS | Always set (production) |
| `SameSite` | CSRF mitigation | `Lax` (default) or `Strict` |
| `Domain` | Cookie scope | Omit (host-only) |
| `Path` | Cookie scope | `/` |
| `Max-Age` | Lifetime in seconds | 86400 (24 hours) |
| `Expires` | Absolute expiry date | Redundant with `Max-Age` |
| `__Host-` prefix | Enforces Secure + no Domain + Path=/ | Use for session cookies |
| `__Secure-` prefix | Enforces Secure | Use when Domain is required |
| `Partitioned` | CHIPS (Cookies Having Independent Partitioned State) | For embedded contexts |

#### Setting Session Cookies (Express)

```typescript
res.cookie('__Host-sid', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
  maxAge: 1000 * 60 * 60 * 24,
  path: '/',
  // domain: undefined  // Omit to enforce host-only
});
```

#### Cookie Prefixes

| Prefix | Requirements | Security Benefit |
|--------|--------------|------------------|
| `__Host-` | Secure, no Domain, Path=/ | Strongest — protects against subdomain attacks |
| `__Secure-` | Secure | Protects against insecure origins |

#### Syntax Rules

- **Always set `HttpOnly`** on session cookies — prevents XSS theft.
- **Always set `Secure`** in production — prevents network interception.
- **Always set `SameSite=Lax` or `Strict`** — prevents CSRF.
- **Use the `__Host-` prefix** for session cookies — strongest protection.
- **Omit the `Domain` attribute** — host-only cookies are more secure.
- **Set `Path=/`** — ensures the cookie is sent for all paths (required for `__Host-`).
- **Set a reasonable `Max-Age`** — do not use session cookies (no expiry) for authenticated sessions.
- **Never store sensitive data in cookies** — only the session ID.
- **Use `SameSite=None; Secure`** only when cross-site is required (e.g., embedded widgets).

#### Constraints and Limitations

- **Cookie size limit** — ~4 KB per cookie; ~50 cookies per domain.
- **Cookie count limit** — browsers limit cookies per domain.
- **Cross-domain limitations** — cookies are scoped to a domain and its subdomains (unless `Domain` is set).
- **`SameSite=None` requires `Secure`** — browsers reject otherwise.
- **`SameSite=Lax` does not protect state-changing GET requests** — use `Strict` or CSRF tokens for sensitive operations.
- **Third-party cookie blocking** — modern browsers block third-party cookies; use CHIPS (`Partitioned`) if needed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Cookie Configuration with CSRF Protection (Express)

```typescript
// server.ts
import express from 'express';
import cookieParser from 'cookie-parser';
import csrf from 'csurf';
import helmet from 'helmet';
import crypto from 'node:crypto';

const app = express();
app.use(helmet());
app.use(express.json());
app.use(cookieParser());

// CSRF protection using the double-submit cookie pattern
const csrfProtection = csrf({
  cookie: {
    key: '__Host-csrf',
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    path: '/',
  },
});
app.use(csrfProtection);

// Endpoint to get a CSRF token
app.get('/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Protected endpoint
app.post('/transfer', (req, res) => {
  res.json({ message: 'Transfer completed' });
});

// CSRF error handler
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  next(err);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- `GET /csrf-token` returns a CSRF token and sets a `__Host-csrf` cookie.
- `POST /transfer` requires the CSRF token in the request (header or body).
- Without a valid CSRF token, `POST /transfer` returns `403 Forbidden`.

**Why this works:** The session cookie uses `HttpOnly`, `Secure`, `SameSite=Lax`, and the `__Host-` prefix. CSRF protection uses the double-submit cookie pattern (or synchronizer pattern with `csurf`). Helmet sets additional security headers (CSP, HSTS, X-Frame-Options).

### Real-World Cases

- **Banking applications:** Session cookies with `__Host-` prefix and CSRF tokens.
- **SaaS platforms:** Session cookies with `SameSite=Lax` and short `Max-Age`.
- **E-commerce:** Session cookies for cart persistence and authentication.
- **Embedded widgets:** `SameSite=None; Secure; Partitioned` (CHIPS) for third-party contexts.

---

## Core Concept 3: Session Storage (Redis, Memcached, Databases)

### Definitions

**Core Definition:** Session storage is the backend system that persists session records, enabling horizontal scaling, high availability, and immediate revocation.

**Technical Definition:** Session stores fall into four categories: (1) **in-memory** (`MemoryStore`) — development only, leaks memory, does not scale; (2) **Redis** — the de facto standard, with fast reads/writes, TTL support, pub/sub for revocation, and clustering; (3) **Memcached** — similar to Redis but without persistence or pub/sub; (4) **relational databases** — durable, transactional, but slower; suitable when session data must be audited or joined with other tables. Redis is the recommended default for production. Session records should have a TTL matching the session expiration, and the store should be configured for high availability (Redis Sentinel or Cluster).

**Beginner-Friendly Explanation:** A session store is like the back room where the restaurant keeps everyone's coats. An in-memory store is a tiny closet that only works for one restaurant (server). Redis is a giant shared warehouse that all restaurants in the chain can access — any server can look up any session. A relational database is like keeping the coats in a secure vault — durable but slower to retrieve.

### Purposes

- To enable horizontal scaling by sharing session state across multiple servers.
- To provide high availability and failover for session data.
- To support immediate session revocation via store operations.
- To enable TTL-based automatic session cleanup.
- To persist sessions across server restarts.

### Syntax Rules and Structure

#### Redis Session Store (Express + connect-redis)

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient({
  url: process.env.REDIS_URL,
  socket: {
    reconnectStrategy: (retries) => Math.min(retries * 50, 2000),
  },
});
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({
      client: redisClient,
      prefix: 'sess:',
      ttl: 86400, // 24 hours
    }),
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',
    cookie: { httpOnly: true, secure: true, sameSite: 'lax', maxAge: 86400000, path: '/' },
  }),
);
```

#### Redis Key Structure

| Key | Value | TTL |
|-----|-------|-----|
| `sess:<session-id>` | JSON-encoded session record | 86400s |
| `user_sessions:<user-id>` | Set of session IDs (for global revocation) | None (managed manually) |

#### Session Store Comparison

| Store | Speed | Persistence | Scalability | Revocation | Use Case |
|-------|-------|-------------|-------------|------------|----------|
| **MemoryStore** | Fastest | None | None | Immediate | Development only |
| **Redis** | Fast | Optional (RDB/AOF) | Excellent (Cluster) | Immediate | **Production default** |
| **Memcached** | Fast | None | Excellent | Immediate | Simple caching; no pub/sub |
| **PostgreSQL/MySQL** | Moderate | Strong | Good (read replicas) | Immediate | Audit requirements; joins |

#### Syntax Rules

- **Never use `MemoryStore` in production** — it leaks memory and does not scale.
- **Set a TTL on session keys** — matching the session's absolute expiration.
- **Use Redis with persistence** (RDB or AOF) if sessions must survive restarts.
- **Use Redis Cluster or Sentinel** for high availability.
- **Prefix keys** to avoid collisions (`sess:`, `user_sessions:`).
- **Store minimal data** — `userId`, `roles`, timestamps, fingerprint metadata.
- **Monitor store latency** — session lookups happen on every request.
- **Clean up expired sessions** — Redis TTL handles this automatically; databases require a cron job.

#### Constraints and Limitations

- **Redis is in-memory** — persistence is optional and must be configured.
- **Store downtime logs out all users** — high availability is essential.
- **Session data is not encrypted at rest by default** — use Redis encryption or an encrypted store.
- **Relational databases are slower** — session lookups add latency to every request.
- **Memcached has no persistence** — sessions are lost on restart.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Redis Session Store with Global Revocation (Express)

```typescript
// session-store.ts
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

export const redisClient = createClient({
  url: process.env.REDIS_URL,
  socket: { reconnectStrategy: (retries) => Math.min(retries * 50, 2000) },
});
await redisClient.connect();

export const sessionStore = new RedisStore({
  client: redisClient,
  prefix: 'sess:',
  ttl: 86400,
});

// Track sessions per user for global revocation
export async function trackUserSession(userId: string, sessionId: string): Promise<void> {
  await redisClient.sAdd(`user_sessions:${userId}`, sessionId);
  await redisClient.expire(`user_sessions:${userId}`, 86400 * 7);
}

export async function revokeAllUserSessions(userId: string): Promise<void> {
  const sessionIds = await redisClient.sMembers(`user_sessions:${userId}`);
  await Promise.all(sessionIds.map((id) => sessionStore.destroy(id)));
  await redisClient.del(`user_sessions:${userId}`);
}
```

```typescript
// server.ts
app.post('/auth/login', async (req, res, next) => {
  const user = await authService.verifyCredentials(req.body.email, req.body.password);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  req.session.regenerate(async (err) => {
    if (err) return next(err);

    req.session.userId = user.id;
    req.session.roles = user.roles;
    req.session.createdAt = Date.now();

    // Track session for global revocation
    await trackUserSession(user.id, req.sessionID);

    res.json({ user: { id: user.id, email: user.email } });
  });
});

// Admin endpoint: revoke all sessions for a user
app.post('/admin/users/:userId/revoke-sessions', requireAdmin, async (req, res) => {
  await revokeAllUserSessions(req.params.userId);
  res.json({ message: 'All sessions revoked' });
});
```

**Expected behaviour:**
- Login creates a session and adds its ID to the user's session set in Redis.
- `POST /admin/users/:userId/revoke-sessions` destroys all sessions for that user.
- The user is logged out from all devices immediately.

**Why this works:** The session store is Redis (shared, scalable). The `user_sessions:<userId>` set tracks all session IDs for a user, enabling global revocation. `sessionStore.destroy()` removes the session record, and the next request with that cookie fails authentication.

### Real-World Cases

- **Multi-server deployments:** Redis shared across all application servers.
- **High-security applications:** Redis with TLS and AOF persistence.
- **Audit-heavy applications:** PostgreSQL session store with session history.
- **Simple caching:** Memcached for session data without persistence requirements.

---

## Core Concept 4: Session Expiration (Absolute vs. Sliding)

### Definitions

**Core Definition:** Session expiration controls how long a session remains valid, using either an absolute timeout (maximum lifetime regardless of activity) or an idle timeout (sliding window based on inactivity).

**Technical Definition:** **Absolute timeout** sets a hard maximum lifetime for a session (e.g., 24 hours from creation), regardless of user activity. **Idle timeout** (sliding window) resets the expiration on each request, extending the session as long as the user remains active (e.g., 30 minutes of inactivity). Best practice combines both: an idle timeout for user convenience and an absolute timeout for security. OWASP recommends idle timeouts of 15–30 minutes for low-risk applications and 2–5 minutes for high-risk applications (banking), with absolute timeouts of 4–8 hours. Session expiration must be enforced server-side (not just via cookie `Max-Age`), and expired sessions must be destroyed in the store.

**Beginner-Friendly Explanation:** Imagine you're at a library. The **idle timeout** is like a librarian who says "if you leave your desk for more than 30 minutes, I'll pack up your books." The **absolute timeout** is like a rule that says "no matter how long you stay, you must leave after 8 hours." Combining both means you can work as long as you're actively there, but you can't stay forever.

### Purposes

- To limit the window of opportunity for session hijacking.
- To free server resources by cleaning up inactive sessions.
- To comply with security standards (OWASP, PCI DSS, HIPAA).
- To balance security (short timeouts) with usability (long timeouts).
- To enforce re-authentication for sensitive operations.

### Syntax Rules and Structure

#### Absolute vs. Idle Timeout

| Timeout Type | Definition | Reset Trigger | Use Case |
|--------------|------------|---------------|----------|
| **Absolute** | Hard maximum lifetime | Never resets | All sessions |
| **Idle (sliding)** | Expires after inactivity | Each request | Active users |
| **Renewal** | Issue new session ID after privilege change | Login, MFA, password change | Security-critical |

#### Recommended Timeouts (OWASP)

| Application Type | Idle Timeout | Absolute Timeout |
|------------------|--------------|------------------|
| Low-risk (blogs, forums) | 30 minutes | 24 hours |
| Medium-risk (e-commerce) | 15–30 minutes | 8 hours |
| High-risk (banking, healthcare) | 2–5 minutes | 4 hours |

#### Implementation (Express)

```typescript
// session-config.ts
const IDLE_TIMEOUT_MS = 30 * 60 * 1000;       // 30 minutes
const ABSOLUTE_TIMEOUT_MS = 8 * 60 * 60 * 1000; // 8 hours

app.use(
  session({
    store: sessionStore,
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',
    rolling: true, // Enable sliding window (reset maxAge on each request)
    cookie: {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      maxAge: IDLE_TIMEOUT_MS,
      path: '/',
    },
  }),
);

// Middleware: enforce absolute timeout
function enforceAbsoluteTimeout(req, res, next) {
  if (!req.session.userId) return next();

  const age = Date.now() - req.session.createdAt;
  if (age > ABSOLUTE_TIMEOUT_MS) {
    req.session.destroy(() => {
      res.clearCookie('__Host-sid', { path: '/' });
      res.status(401).json({ error: 'Session expired' });
    });
    return;
  }

  next();
}

app.use(enforceAbsoluteTimeout);
```

#### Syntax Rules

- **Enforce timeouts server-side** — cookie `Max-Age` alone is insufficient (clients can tamper).
- **Combine idle and absolute timeouts** — idle for usability, absolute for security.
- **Use `rolling: true`** for sliding windows — resets `maxAge` on each request.
- **Regenerate the session ID** after a privilege change (login, MFA, password change).
- **Destroy expired sessions** in the store — do not rely on cookie expiration alone.
- **Notify the client** when a session expires — return `401` with a clear message.
- **Consider step-up authentication** for sensitive operations within a valid session.

#### Constraints and Limitations

- **Short idle timeouts frustrate users** — balance security with UX.
- **Sliding windows extend session lifetime indefinitely** — absolute timeouts are essential.
- **Server-side enforcement adds latency** — a store lookup is required on each request.
- **Clock skew** between servers can cause premature expiration.
- **Session expiration does not revoke tokens** — if using JWTs alongside sessions, both must be managed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Combined Absolute and Idle Timeouts (Express)

```typescript
// server.ts
const IDLE_TIMEOUT_MS = 30 * 60 * 1000;
const ABSOLUTE_TIMEOUT_MS = 8 * 60 * 60 * 1000;

app.use(
  session({
    store: sessionStore,
    secret: process.env.SESSION_SECRET!,
    resave: false,
    saveUninitialized: false,
    name: '__Host-sid',
    rolling: true,
    cookie: {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      maxAge: IDLE_TIMEOUT_MS,
      path: '/',
    },
  }),
);

app.use((req, res, next) => {
  if (!req.session.userId) return next();

  // Absolute timeout check
  if (Date.now() - req.session.createdAt > ABSOLUTE_TIMEOUT_MS) {
    req.session.destroy(() => {
      res.clearCookie('__Host-sid', { path: '/' });
      return res.status(401).json({ error: 'Session expired', code: 'ABSOLUTE_TIMEOUT' });
    });
    return;
  }

  // Update last activity (idle timeout is handled by rolling: true)
  req.session.lastActivityAt = Date.now();
  next();
});
```

**Expected behaviour:**
- A user who is active for 7 hours and 59 minutes stays logged in (idle timeout never triggers).
- At 8 hours, the session is destroyed regardless of activity (absolute timeout).
- A user who is inactive for 30 minutes is logged out (idle timeout via `rolling: true`).

**Why this works:** `rolling: true` resets the cookie `Max-Age` on each request, implementing the idle timeout. The middleware checks `createdAt` against `ABSOLUTE_TIMEOUT_MS` on each request, enforcing the absolute timeout. Both timeouts are enforced server-side.

### Real-World Cases

- **Banking:** Idle timeout of 5 minutes, absolute timeout of 4 hours.
- **E-commerce:** Idle timeout of 30 minutes, absolute timeout of 24 hours.
- **Healthcare:** Idle timeout of 15 minutes, absolute timeout of 8 hours.
- **Admin panels:** Idle timeout of 10 minutes, absolute timeout of 2 hours.

---

## Core Concept 5: Session Revocation

### Definitions

**Core Definition:** Session revocation is the process of invalidating an active session, either for a specific session (logout from one device) or globally (logout from all devices).

**Technical Definition:** Session revocation is straightforward with server-side sessions: the session record is deleted from the store, and the next request with that cookie fails authentication. Revocation mechanisms include: (1) **single-session revocation** — delete one session record (logout); (2) **global revocation** — delete all sessions for a user (password change, "log out everywhere"); (3) **privilege-based revocation** — delete sessions with lower privilege after escalation; (4) **admin-initiated revocation** — delete sessions for a specific user or all users. Redis supports efficient revocation via key deletion and sets (`user_sessions:<userId>`). Revocation must be immediate — the next request must fail authentication.

**Beginner-Friendly Explanation:** Revocation is like cancelling a coat-check ticket. If you leave the restaurant, the staff throws away your coat and your ticket becomes useless. If you think someone stole your ticket, you can ask the staff to throw away all your coats — so whoever has the ticket can't get anything. Revocation is one of the biggest advantages of sessions over tokens: the server can cut off access instantly.

### Purposes

- To log out a user from one device (single-session revocation).
- To log out a user from all devices (global revocation).
- To invalidate sessions after a password change or security incident.
- To enforce administrative actions (banning a user, revoking access).
- To revoke sessions when a user's privileges are reduced.

### Syntax Rules and Structure

#### Revocation Operations

| Operation | Implementation |
|-----------|----------------|
| Single-session logout | `sessionStore.destroy(sessionId)` |
| Global logout | Delete all session IDs in `user_sessions:<userId>` |
| Password change | Global logout + revoke refresh tokens |
| Admin ban | Global logout + prevent new logins |
| Privilege reduction | Revoke sessions with higher privileges |

#### Global Revocation (Redis)

```typescript
// session-revocation.ts
export async function revokeSession(sessionId: string): Promise<void> {
  await sessionStore.destroy(sessionId);
}

export async function revokeAllUserSessions(userId: string): Promise<void> {
  const sessionIds = await redisClient.sMembers(`user_sessions:${userId}`);
  await Promise.all(sessionIds.map((id) => sessionStore.destroy(id)));
  await redisClient.del(`user_sessions:${userId}`);
}

export async function revokeAllSessions(): Promise<void> {
  // Dangerous — logs out all users
  await sessionStore.clear();
}
```

#### Revocation on Password Change

```typescript
async changePassword(userId: string, oldPassword: string, newPassword: string): Promise<void> {
  const user = await prisma.user.findUniqueOrThrow({ where: { id: userId } });

  // Verify old password
  if (!(await passwordService.verify(oldPassword, user.passwordHash))) {
    throw new UnauthorizedException('Invalid password');
  }

  // Validate new password
  const validation = await passwordValidator.validate(newPassword, user);
  if (!validation.valid) throw new BadRequestException(validation.errors);

  // Hash and update
  const passwordHash = await passwordService.hash(newPassword);
  await prisma.user.update({ where: { id: userId }, data: { passwordHash } });

  // Revoke all sessions globally
  await revokeAllUserSessions(userId);

  // Revoke all refresh tokens (if using JWT alongside sessions)
  await prisma.refreshToken.updateMany({
    where: { userId, revokedAt: null },
    data: { revokedAt: new Date() },
  });
}
```

#### Syntax Rules

- **Revocation must be immediate** — the next request with a revoked session must fail.
- **Revoke all sessions on password change** — prevents persistent access by attackers.
- **Track sessions per user** — use a `user_sessions:<userId>` set in Redis.
- **Clear the session cookie** on the client after revocation — `res.clearCookie()`.
- **Notify the user** when their sessions are revoked (email, in-app).
- **Log revocation events** — for audit and security monitoring.
- **Rate-limit revocation endpoints** — prevent abuse.
- **Consider step-up authentication** for high-risk operations (not full revocation).

#### Constraints and Limitations

- **Global revocation requires session tracking** — a set of session IDs per user.
- **Revocation does not affect JWTs** — if using tokens alongside sessions, both must be managed.
- **Session store downtime prevents revocation** — high availability is essential.
- **Revocation is not instantaneous across replicas** — eventual consistency may delay propagation.
- **Revocation does not prevent new logins** — pair with account suspension if needed.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Revocation System (NestJS + Redis)

```typescript
// auth/session.service.ts
@Injectable()
export class SessionService {
  constructor(
    @Inject('REDIS_CLIENT') private readonly redis: Redis,
    private readonly sessionStore: RedisStore,
  ) {}

  async trackSession(userId: string, sessionId: string): Promise<void> {
    await this.redis.sAdd(`user_sessions:${userId}`, sessionId);
    await this.redis.expire(`user_sessions:${userId}`, 7 * 24 * 60 * 60);
  }

  async revokeSession(sessionId: string): Promise<void> {
    await this.sessionStore.destroy(sessionId);
  }

  async revokeAllUserSessions(userId: string): Promise<number> {
    const sessionIds = await this.redis.sMembers(`user_sessions:${userId}`);
    await Promise.all(sessionIds.map((id) => this.sessionStore.destroy(id)));
    await this.redis.del(`user_sessions:${userId}`);
    return sessionIds.length;
  }

  async listUserSessions(userId: string): Promise<string[]> {
    return this.redis.sMembers(`user_sessions:${userId}`);
  }
}
```

```typescript
// auth/auth.controller.ts
@Controller('auth')
export class AuthController {
  constructor(
    private readonly sessionService: SessionService,
    private readonly authService: AuthService,
  ) {}

  @Post('logout')
  @HttpCode(204)
  async logout(@Req() req: Request): Promise<void> {
    await this.sessionService.revokeSession(req.sessionID);
    req.session.destroy(() => {});
  }

  @Post('logout-all')
  @HttpCode(204)
  async logoutAll(@Req() req: Request): Promise<void> {
    await this.sessionService.revokeAllUserSessions(req.session.userId);
    req.session.destroy(() => {});
  }

  @Get('sessions')
  async listSessions(@Req() req: Request): Promise<{ sessionId: string; current: boolean }[]> {
    const sessionIds = await this.sessionService.listUserSessions(req.session.userId);
    return sessionIds.map((id) => ({
      sessionId: id.slice(0, 8) + '...',
      current: id === req.sessionID,
    }));
  }

  @Delete('sessions/:sessionId')
  @HttpCode(204)
  async revokeSession(
    @Req() req: Request,
    @Param('sessionId') sessionId: string,
  ): Promise<void> {
    // Verify the session belongs to the user
    const userSessions = await this.sessionService.listUserSessions(req.session.userId);
    if (!userSessions.includes(sessionId)) {
      throw new ForbiddenException('Session does not belong to you');
    }
    await this.sessionService.revokeSession(sessionId);
  }
}
```

**Expected behaviour:**
- `POST /auth/logout` revokes the current session.
- `POST /auth/logout-all` revokes all sessions for the user.
- `GET /auth/sessions` lists all active sessions (masked IDs).
- `DELETE /auth/sessions/:sessionId` revokes a specific session (with ownership check).

**Why this works:** Session IDs are tracked per user in Redis. Revocation destroys the session record in the store, so the next request with that cookie fails authentication. The ownership check prevents users from revoking other users' sessions.

### Real-World Cases

- **Banking:** "Log out everywhere" after a password change or suspicious activity.
- **E-commerce:** "Manage devices" page listing active sessions with individual revocation.
- **Enterprise:** Admin-initiated revocation for departing employees.
- **Healthcare:** Automatic revocation on privilege change or role transition.

---

## Core Concept 6: Session Fixation & Hijacking Protections

### Definitions

**Core Definition:** Session fixation is an attack where an attacker sets a victim's session ID before login, then uses that known ID after the victim authenticates. Session hijacking is the theft of a valid session ID to impersonate the victim. Protections include session ID regeneration on privilege changes and client fingerprinting.

**Technical Definition:** **Session fixation** occurs when an application accepts a session ID from the client (e.g., via URL parameter or cookie) without regenerating it upon authentication. The attacker sets a known session ID, tricks the victim into using it, and after the victim logs in, the attacker uses the same ID to access the authenticated session. **Mitigation:** regenerate the session ID upon successful authentication, MFA completion, password change, and privilege escalation. **Session hijacking** occurs when an attacker steals a session ID via XSS, network sniffing, or physical access. **Mitigations:** `HttpOnly` cookies (XSS), `Secure` cookies (network sniffing), `SameSite` cookies (CSRF), session fingerprinting (IP + User-Agent binding), short timeouts, and immediate revocation.

**Beginner-Friendly Explanation:** Session fixation is like an attacker giving you a pre-printed coat-check ticket before you arrive at the restaurant. You hand it in when you check your coat, and now the attacker knows your ticket number — so they can pick up your coat. The fix: when you check in, the restaurant gives you a brand-new ticket and throws away the old one. Session hijacking is like someone stealing your ticket. The fixes: keep the ticket in a sealed envelope (`HttpOnly`), only show it over a secure line (`Secure`), and check that the person holding the ticket looks the same as when they checked in (fingerprinting).

### Purposes

- To prevent session fixation attacks by regenerating session IDs on privilege changes.
- To detect and prevent session hijacking via client fingerprinting.
- To protect session cookies from XSS, network sniffing, and CSRF.
- To limit the window of opportunity for stolen sessions.
- To comply with OWASP and NIST session management requirements.

### Syntax Rules and Structure

#### Session ID Regeneration

```typescript
// Regenerate on login
req.session.regenerate((err) => {
  if (err) return next(err);
  req.session.userId = user.id;
  req.session.roles = user.roles;
  res.json({ user: { id: user.id } });
});

// Regenerate on privilege change (e.g., MFA completion)
req.session.regenerate((err) => {
  if (err) return next(err);
  req.session.userId = user.id;
  req.session.mfaVerified = true;
  req.session.roles = user.roles;
  res.json({ mfaVerified: true });
});
```

| Event | Regenerate Session ID? |
|-------|-----------------------|
| Login | ✅ Yes |
| MFA completion | ✅ Yes |
| Password change | ✅ Yes |
| Privilege escalation | ✅ Yes |
| Role change | ✅ Yes |
| Logout | Destroy session |

#### Session Fingerprinting

```typescript
interface SessionFingerprint {
  ip: string;
  userAgent: string;
  acceptLanguage: string;
}

function createFingerprint(req: Request): SessionFingerprint {
  return {
    ip: req.ip ?? '',
    userAgent: req.get('User-Agent') ?? '',
    acceptLanguage: req.get('Accept-Language') ?? '',
  };
}

function verifyFingerprint(req: Request, session: SessionRecord): boolean {
  const current = createFingerprint(req);

  // Strict: all attributes must match
  if (session.ip !== current.ip) return false;
  if (session.userAgent !== current.userAgent) return false;
  if (session.acceptLanguage !== current.acceptLanguage) return false;

  return true;
}
```

#### Syntax Rules

- **Always regenerate the session ID on login** — prevents fixation.
- **Always regenerate on privilege change** — MFA, password change, role change.
- **Destroy the old session record** after regeneration — do not leave it in the store.
- **Use `HttpOnly`, `Secure`, `SameSite`** — prevents XSS, network sniffing, and CSRF.
- **Use the `__Host-` prefix** — prevents subdomain-based fixation.
- **Implement fingerprinting** — bind sessions to IP, User-Agent, and Accept-Language.
- **Handle fingerprint mismatches gracefully** — destroy the session and require re-authentication.
- **Log fingerprint mismatches** — for security monitoring.
- **Do not rely solely on IP** — mobile users change IPs frequently; combine with User-Agent.
- **Consider geolocation and device fingerprinting** for high-security applications.

#### Constraints and Limitations

- **IP fingerprinting breaks on mobile networks** — IPs change frequently (carrier NAT, Wi-Fi switching).
- **User-Agent fingerprinting can be spoofed** — attackers can copy the victim's User-Agent.
- **Fingerprinting may cause false positives** — legitimate users may be logged out.
- **Regeneration does not help if the attacker has the new ID** — XSS or malware can steal it.
- **Session ID regeneration adds latency** — one extra store operation.
- **Fingerprinting is not a silver bullet** — combine with other controls (MFA, short timeouts, revocation).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Fixation and Hijacking Protection (Express)

```typescript
// server.ts
import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';
import { randomBytes } from 'node:crypto';
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
    genid: () => randomBytes(32).toString('base64url'),
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 30 * 60 * 1000,
      path: '/',
    },
    rolling: true,
  }),
);

// --- Login: regenerate session ID ---
app.post('/auth/login', async (req, res, next) => {
  const user = await authService.verifyCredentials(req.body.email, req.body.password);
  if (!user) return res.status(401).json({ error: 'Invalid credentials' });

  req.session.regenerate((err) => {
    if (err) return next(err);

    req.session.userId = user.id;
    req.session.roles = user.roles;
    req.session.createdAt = Date.now();
    req.session.ip = req.ip;
    req.session.userAgent = req.get('User-Agent') ?? '';
    req.session.acceptLanguage = req.get('Accept-Language') ?? '';

    res.json({ user: { id: user.id, email: user.email } });
  });
});

// --- MFA: regenerate session ID again ---
app.post('/auth/mfa/verify', requireAuth, async (req, res, next) => {
  const valid = await mfaService.verify(req.session.userId, req.body.code);
  if (!valid) return res.status(401).json({ error: 'Invalid MFA code' });

  req.session.regenerate((err) => {
    if (err) return next(err);

    req.session.userId = req.session.userId; // Preserve
    req.session.mfaVerified = true;
    req.session.createdAt = Date.now();
    req.session.ip = req.ip;
    req.session.userAgent = req.get('User-Agent') ?? '';

    res.json({ mfaVerified: true });
  });
});

// --- Fingerprint verification middleware ---
function requireAuth(req, res, next) {
  if (!req.session.userId) return res.status(401).json({ error: 'Unauthorized' });

  // Fingerprint check
  const currentIp = req.ip;
  const currentUserAgent = req.get('User-Agent') ?? '';
  const currentAcceptLanguage = req.get('Accept-Language') ?? '';

  if (
    req.session.ip !== currentIp ||
    req.session.userAgent !== currentUserAgent ||
    req.session.acceptLanguage !== currentAcceptLanguage
  ) {
    // Log the mismatch for security monitoring
    logger.warn(`Session fingerprint mismatch for user ${req.session.userId}`, {
      sessionIp: req.session.ip,
      currentIp,
      sessionUserAgent: req.session.userAgent,
      currentUserAgent,
    });

    req.session.destroy(() => {
      res.clearCookie('__Host-sid', { path: '/' });
      return res.status(401).json({ error: 'Session invalid', code: 'FINGERPRINT_MISMATCH' });
    });
    return;
  }

  next();
}

app.get('/me', requireAuth, (req, res) => {
  res.json({ userId: req.session.userId, mfaVerified: req.session.mfaVerified });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- Login regenerates the session ID and stores the fingerprint.
- MFA verification regenerates the session ID again.
- Subsequent requests verify the fingerprint; mismatches destroy the session.
- Fingerprint mismatches are logged for security monitoring.

**Why this works:** Session ID regeneration on login and MFA prevents fixation. Fingerprinting binds the session to the client's IP, User-Agent, and Accept-Language, detecting hijacking. `HttpOnly`, `Secure`, `SameSite`, and the `__Host-` prefix protect the cookie from XSS, network sniffing, CSRF, and subdomain attacks.

### Real-World Cases

- **Banking:** Fingerprinting + MFA + session regeneration on every privilege change.
- **E-commerce:** Fingerprinting (User-Agent only, not IP) + session regeneration on login.
- **Enterprise:** Fingerprinting + short timeouts + admin-initiated revocation.
- **Healthcare:** Fingerprinting + MFA + audit logging for HIPAA compliance.

---

## References

- OWASP Cheat Sheet Series — Session Management — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cookie Theft Mitigation — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- NIST SP 800-63B — Digital Identity Guidelines: Authentication and Lifecycle Management — https://pages.nist.gov/800-63-3/sp800-63b.html
- RFC 6265 — HTTP State Management Mechanism (Cookies) — https://www.rfc-editor.org/rfc/rfc6265
- RFC 6265bis — Cookies: HTTP State Management Mechanism (Draft) — https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis
- MDN Web Docs — Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN Web Docs — SameSite Cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- MDN Web Docs — Cookie Prefixes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#cookie_prefixes
- Chrome Developers — Cookies Having Independent Partitioned State (CHIPS) — https://developer.chrome.com/docs/privacy-sandbox/chips/
- express-session Documentation — https://github.com/expressjs/session
- connect-redis Documentation — https://github.com/tj/connect-redis
- Redis Documentation — SET with TTL — https://redis.io/commands/set/
- Redis Documentation — Sets — https://redis.io/docs/data-types/sets/
- OWASP — Session Fixation — https://owasp.org/www-community/attacks/Session_fixation
- OWASP — Session Hijacking — https://owasp.org/www-community/attacks/Session_hijacking_attack
- PortSwigger — Session Fixation — https://portswigger.net/web-security/authentication/other-mechanisms
- Auth0 — Session Management Best Practices — https://auth0.com/docs/manage-users/sessions
- Helmet Documentation — https://helmetjs.github.io/
- csurf Documentation — https://github.com/expressjs/csurf