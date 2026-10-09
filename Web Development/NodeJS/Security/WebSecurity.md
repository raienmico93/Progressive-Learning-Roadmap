# Web Security Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Web security fundamentals are the foundational practices, controls, and architectural principles that protect web applications, their users, and their data from unauthorized access, tampering, disclosure, and denial of service.

**Technical Definition:** Web application security encompasses the defensive controls applied across all layers of a web stack — from transport (TLS), to identity (authentication), to access control (authorization), to session management, to the principle of least privilege applied to database users, file system access, and network capabilities. It aligns with the OWASP Top 10, the OWASP Application Security Verification Standard (ASVS), and NIST guidelines. Security is a cross-cutting concern: it must be enforced at every layer, validated at every boundary, and never trusted from the client.

**Beginner-Friendly Explanation:** Think of web security as the security of a bank. **Transport security** is the armored truck that carries money (TLS/HTTPS). **Authentication security** is the ID check at the door (passwords, MFA). **Authorization security** is the access rules that determine which vaults you can open (permissions). **Session security** is the temporary badge that lets you move around after you've been verified (cookies, tokens). **Least privilege** is the principle that even the bank manager shouldn't have keys to every vault — everyone gets only what they need to do their job. If any one layer fails, the others must still hold.

### Key Characteristics

- **Defense in depth:** Multiple layers of security controls, so no single failure compromises the system.
- **Never trust the client:** All input must be validated and sanitized on the server.
- **Least privilege:** Every user, service, and process gets only the minimum access necessary.
- **Secure by default:** Security controls are enabled by default, not as opt-in.
- **Fail securely:** When errors occur, the system fails in a way that denies access, not grants it.
- **Auditability:** Every security-relevant action is logged and monitored.
- **Continuous improvement:** Security is a process, not a one-time configuration.

### Prerequisites

- **HTTP fundamentals:** Methods, headers, status codes, cookies, redirects.
- **TLS/HTTPS:** Certificates, cipher suites, HSTS, certificate pinning.
- **Authentication and authorization:** Passwords, MFA, RBAC, ABAC, sessions, tokens.
- **Cryptography basics:** Hashing, HMAC, symmetric/asymmetric encryption, CSPRNG.
- **Web application architecture:** Layers, boundaries, and trust zones.
- **Node.js fundamentals:** Express, NestJS, middleware, environment configuration.
- **OWASP Top 10:** The most critical web application security risks.

### Related Programming Areas

- **Authentication:** Password security, MFA, sessions, tokens.
- **Authorization:** RBAC, ABAC, ReBAC, multi-tenancy.
- **API security:** Input validation, rate limiting, CORS, CSRF.
- **Infrastructure security:** Firewalls, VPCs, IAM, secrets management.
- **Compliance:** PCI DSS, HIPAA, SOC 2, GDPR, ISO 27001.
- **DevSecOps:** SAST, DAST, dependency scanning, secret scanning.

### Core Concepts

1. **Authentication Security** — safeguarding user credentials and preventing brute-force cracking.
2. **Authorization Security** — ensuring robust privilege validation checks across all API access pathways.
3. **Session Security** — defending active login sessions from hijacking, fixation, and forgery.
4. **Transport Security** — enforcing TLS/SSL configurations, strict cipher suites, and HTTPS-only traffic redirection.
5. **Least Privilege Architecture** — restricting database users, file system access, and system network capabilities to the bare minimum.

---

## Core Concept 1: Authentication Security

### Definitions

**Core Definition:** Authentication security is the set of controls that protect user credentials and the authentication process from theft, brute-force, credential stuffing, and account takeover.

**Technical Definition:** Authentication security encompasses: (1) **credential storage** — hashing passwords with memory-hard algorithms (Argon2id, bcrypt, scrypt); (2) **credential transmission** — always over HTTPS, never in URLs; (3) **brute-force prevention** — rate limiting, account lockout, CAPTCHA, and exponential backoff; (4) **credential stuffing prevention** — breached password detection (HaveIBeenPwned), MFA enforcement; (5) **phishing resistance** — WebAuthn/passkeys, FIDO2; (6) **account enumeration prevention** — generic error messages, constant-time comparisons; (7) **MFA** — TOTP, push, hardware keys; (8) **password reset security** — cryptographically random, single-use, short-lived tokens. Authentication must be enforced at every entry point (login, registration, password reset, MFA, OAuth callbacks).

**Beginner-Friendly Explanation:** Authentication security is like the security of a bank's front door. The door (login) is reinforced, the lock (password hashing) is strong, and there's a guard who stops people from trying too many keys (rate limiting). If someone forgets their key, they go through a careful verification process (password reset) that can't be exploited. And for high-value accounts, there's a second lock (MFA) that requires a fingerprint or a code.

### Purposes

- To prevent unauthorized access to user accounts.
- To protect credentials from theft, cracking, and reuse.
- To resist brute-force, credential stuffing, and phishing attacks.
- To prevent account enumeration and information leakage.
- To ensure that authentication is enforced consistently across all entry points.
- To comply with security standards (OWASP ASVS, NIST 800-63B).

### Syntax Rules and Structure

#### Credential Storage Rules

| Rule | Implementation |
|------|----------------|
| **Hash with Argon2id** | 64 MiB memory, 3 iterations, 1 parallelism |
| **Unique salt per password** | Handled automatically by the algorithm |
| **Constant-time comparison** | `argon2.verify()`, `bcrypt.compare()` |
| **Never store plaintext** | Always hash before storage |
| **Never log passwords** | Redact from logs, error messages |
| **Never transmit in URLs** | Use POST bodies only |

#### Rate Limiting Rules

| Control | Value |
|---------|-------|
| **Per-IP rate limit** | 10–20 login attempts per minute |
| **Per-account rate limit** | 5–10 attempts per 15 minutes |
| **Account lockout** | Temporary lockout after N failures |
| **Exponential backoff** | Increase delay after each failure |
| **CAPTCHA** | After 3–5 failures |
| **Breached password check** | HaveIBeenPwned k-anonymity API |

#### Syntax Rules

- **Always hash passwords** with a memory-hard algorithm (Argon2id preferred).
- **Always use HTTPS** for authentication endpoints.
- **Always rate-limit** login, registration, and password reset.
- **Always use constant-time comparison** for credentials.
- **Always return generic error messages** ("Invalid credentials") to prevent enumeration.
- **Always require MFA** for sensitive accounts or operations.
- **Always validate password strength** — length ≥ 12, breach check.
- **Never reveal whether email or password was wrong.**
- **Never log credentials or tokens.**
- **Never trust client-side validation** — always re-validate server-side.

#### Constraints and Limitations

- **Passwords are inherently weak** — users reuse them, choose weak ones, fall for phishing.
- **Rate limiting can be bypassed** with distributed attacks (botnets).
- **Account lockout can be a DoS vector** — attackers can lock out legitimate users.
- **CAPTCHA hurts UX** — use only when necessary.
- **MFA adds friction** — balance security with usability.
- **Breach detection requires an external API** — network dependency.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Login with Rate Limiting, Breach Detection, and Constant-Time Comparison (Express)

```typescript
// auth.service.ts
import bcrypt from 'bcrypt';
import { createHash } from 'node:crypto';
import rateLimit from 'express-rate-limit';
import slowDown from 'express-slow-down';

// Rate limiting: 10 attempts per minute per IP
const loginLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  message: { error: 'Too many login attempts, please try again later' },
  standardHeaders: true,
  legacyHeaders: false,
});

// Progressive delay: slow down after 3 attempts
const loginSlowDown = slowDown({
  windowMs: 15 * 60 * 1000,
  delayAfter: 3,
  delayMs: (hits) => hits * 500, // 500ms, 1000ms, 1500ms, ...
});

// Per-account lockout (stored in Redis)
async function checkAccountLockout(email: string): Promise<boolean> {
  const key = `lockout:${email.toLowerCase()}`;
  const attempts = await redis.get(key);
  if (attempts && parseInt(attempts) >= 5) {
    const ttl = await redis.ttl(key);
    throw new Error(`Account locked. Try again in ${ttl} seconds.`);
  }
  return false;
}

async function recordFailedAttempt(email: string): Promise<void> {
  const key = `lockout:${email.toLowerCase()}`;
  await redis.incr(key);
  await redis.expire(key, 15 * 60); // 15-minute window
}

async function clearFailedAttempts(email: string): Promise<void> {
  await redis.del(`lockout:${email.toLowerCase()}`);
}

// Breach detection (HaveIBeenPwned k-anonymity)
async function isBreached(password: string): Promise<boolean> {
  const sha1 = createHash('sha1').update(password).digest('hex').toUpperCase();
  const prefix = sha1.slice(0, 5);
  const suffix = sha1.slice(5);

  const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`, {
    headers: { 'Add-Padding': 'true' },
  });
  const body = await response.text();
  return body.split('\n').some((line) => line.startsWith(suffix));
}

// Login endpoint
app.post('/auth/login', loginLimiter, loginSlowDown, async (req, res) => {
  const { email, password } = req.body;

  try {
    await checkAccountLockout(email);
  } catch (err) {
    return res.status(429).json({ error: 'Account temporarily locked' });
  }

  const user = await prisma.user.findUnique({
    where: { email: email.toLowerCase().trim() },
  });

  // Always perform a hash comparison (constant-time) to prevent timing attacks
  const dummyHash = '$2b$12$invalidhashplaceholderinvalidhashplaceholder';
  const hash = user?.passwordHash ?? dummyHash;
  const valid = await bcrypt.compare(password, hash).catch(() => false);

  if (!user || !valid) {
    await recordFailedAttempt(email);
    // Generic error — does not reveal whether email or password was wrong
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Check for breached password (optional — can be done at registration)
  if (await isBreached(password)) {
    return res.status(403).json({
      error: 'Password has been breached',
      message: 'Please reset your password',
    });
  }

  await clearFailedAttempts(email);

  // Create session or issue tokens
  const tokens = await issueTokens(user.id);
  res.json(tokens);
});
```

**Expected behaviour:**
- After 10 login attempts per minute from the same IP, requests are rejected with `429`.
- After 3 failed attempts for an account, subsequent attempts are progressively delayed.
- After 5 failed attempts for an account, the account is locked for 15 minutes.
- Unknown emails receive the same generic `401` response as wrong passwords.
- Breached passwords are rejected with `403` and a reset prompt.

**Why this works:** Multiple layers of protection (IP rate limiting, progressive delay, account lockout, constant-time comparison, breach detection) make brute-force and credential stuffing infeasible. Generic error messages prevent account enumeration.

### Real-World Cases

- **Banking:** Rate limiting, MFA, breach detection, and account lockout.
- **SaaS:** Progressive delays, CAPTCHA, and MFA for all users.
- **E-commerce:** Rate limiting and breach detection to prevent credential stuffing.
- **Enterprise:** SSO with MFA and conditional access policies.

---

## Core Concept 2: Authorization Security

### Definitions

**Core Definition:** Authorization security is the set of controls that ensure every request is validated against the user's permissions, at every API pathway, without exception.

**Technical Definition:** Authorization security requires: (1) **enforcement at every entry point** — REST endpoints, GraphQL resolvers, WebSocket handlers, gRPC methods, background jobs; (2) **deny by default** — if no rule explicitly grants access, deny it; (3) **centralised policy** — policies defined in one place and enforced consistently; (4) **resource-level checks** — ownership and tenant isolation; (5) **no client-side enforcement** — all checks on the server; (6) **audit logging** — every authorization decision logged; (7) **IDOR prevention** — never trust resource IDs from the client; (8) **privilege escalation prevention** — validate role assignments and scope changes. Common vulnerabilities include IDOR, missing function-level access control, and privilege escalation.

**Beginner-Friendly Explanation:** Authorization security is like a hotel with electronic locks. Every door (API endpoint) checks your key card (permissions) before opening. You can't just walk into any room because you know the room number (IDOR). The front desk (policy) decides which rooms your key opens. And if you try to use your key on a door you shouldn't access, the system logs it (audit). Even if you sneak past one door, the next door still checks your key — defense in depth.

### Purposes

- To ensure that every request is validated against the user's permissions.
- To prevent unauthorized access to resources and functions.
- To prevent privilege escalation and IDOR vulnerabilities.
- To enforce tenant isolation in multi-tenant applications.
- To provide audit trails for compliance.
- To implement least privilege across the application.

### Syntax Rules and Structure

#### Authorization Enforcement Points

| Entry Point | Enforcement |
|-------------|-------------|
| REST endpoints | Middleware / guards |
| GraphQL resolvers | Resolver-level guards |
| WebSocket handlers | Connection + message guards |
| gRPC methods | Interceptors |
| Background jobs | Job-level authorization |
| CLI commands | Command-level authorization |
| Admin tools | Separate, stricter authorization |

#### Authorization Rules

| Rule | Implementation |
|------|----------------|
| **Deny by default** | If no rule grants, deny |
| **Server-side only** | Never trust client-side checks |
| **Resource-level checks** | Load resource, check ownership |
| **Tenant isolation** | Always filter by `tenantId` |
| **Centralised policy** | Use a policy engine (Casbin, OPA) |
| **Audit logging** | Log every authorization decision |
| **Fail securely** | Errors deny access, not grant |

#### Syntax Rules

- **Enforce authorization at every entry point** — no exceptions.
- **Deny by default** — if no rule grants access, deny.
- **Validate resource ownership** — never trust resource IDs from the client.
- **Filter by tenant** — always include `tenantId` in queries.
- **Use a centralised policy engine** — avoid scattered authorization logic.
- **Log every authorization decision** — for audit and incident response.
- **Return 403, not 404** — or return 404 to hide resource existence.
- **Never trust client-side authorization** — always re-validate server-side.
- **Validate role assignments** — prevent privilege escalation.
- **Test authorization thoroughly** — include negative tests.

#### Constraints and Limitations

- **Authorization is application-specific** — no one-size-fits-all solution.
- **Centralised policy engines add complexity** — but improve consistency.
- **Resource-level checks add database queries** — performance impact.
- **Tenant isolation requires discipline** — a single missed filter leaks data.
- **Audit logging adds storage overhead** — but is essential for compliance.
- **Authorization bugs are common** — OWASP Top 10 includes Broken Access Control as #1.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comprehensive Authorization with Guards, Ownership, and Tenant Isolation (NestJS)

```typescript
// auth.guard.ts — authentication
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private readonly jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const req = context.switchToHttp().getRequest();
    const token = extractBearerToken(req);
    if (!token) throw new UnauthorizedException();

    try {
      const payload = await this.jwtService.verifyAsync(token, {
        algorithms: ['RS256'],
        issuer: 'https://auth.example.com',
        audience: 'https://api.example.com',
      });
      req.user = { id: payload.sub, roles: payload.roles, tenantId: payload.tenantId };
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }
}

// permissions.guard.ts — authorization
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly permissionsService: PermissionsService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const required = this.reflector.getAllAndOverride<string[]>('permissions', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!required || required.length === 0) return true;

    const req = context.switchToHttp().getRequest();
    const userId = req.user?.id;
    if (!userId) throw new UnauthorizedException();

    const hasAll = await Promise.all(
      required.map((p) => this.permissionsService.hasPermission(userId, p)),
    );

    if (!hasAll.every(Boolean)) {
      throw new ForbiddenException('Insufficient permissions');
    }
    return true;
  }
}

// tenant.guard.ts — tenant isolation
@Injectable()
export class TenantGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest();
    if (!req.user?.tenantId) throw new ForbiddenException('No tenant context');
    req.tenantId = req.user.tenantId;
    return true;
  }
}

// posts.controller.ts — layered guards
@Controller('posts')
@UseGuards(AuthGuard, TenantGuard, PermissionsGuard)
export class PostsController {
  @Get()
  @RequirePermissions('posts:read')
  async findAll(@CurrentTenant() tenantId: string) {
    return this.postsService.findAll(tenantId);
  }

  @Get(':id')
  @RequirePermissions('posts:read')
  async findById(@Param('id') id: string, @CurrentTenant() tenantId: string) {
    return this.postsService.findById(id, tenantId);
  }

  @Patch(':id')
  @RequirePermissions('posts:edit:own', 'posts:edit:any')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdatePostDto,
    @CurrentUser() user: AuthUser,
    @CurrentTenant() tenantId: string,
  ) {
    return this.postsService.update(id, dto, user, tenantId);
  }
}

// posts.service.ts — resource-level authorization
@Injectable()
export class PostsService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly permissionsService: PermissionsService,
  ) {}

  async findAll(tenantId: string) {
    return this.prisma.post.findMany({ where: { tenantId } });
  }

  async findById(id: string, tenantId: string) {
    const post = await this.prisma.post.findFirst({ where: { id, tenantId } });
    if (!post) throw new NotFoundException('Post not found');
    return post;
  }

  async update(id: string, dto: UpdatePostDto, user: AuthUser, tenantId: string) {
    const post = await this.prisma.post.findFirst({ where: { id, tenantId } });
    if (!post) throw new NotFoundException('Post not found');

    const canEditAny = await this.permissionsService.hasPermission(user.id, 'posts:edit:any');
    const canEditOwn = await this.permissionsService.hasPermission(user.id, 'posts:edit:own');
    const isOwner = post.authorId === user.id;

    if (!canEditAny && !(canEditOwn && isOwner)) {
      throw new ForbiddenException('Cannot edit this post');
    }

    return this.prisma.post.update({ where: { id }, data: dto });
  }
}
```

**Expected behaviour:**
- Every request passes through `AuthGuard` (authentication), `TenantGuard` (tenant isolation), and `PermissionsGuard` (permission check).
- `findAll()` filters by `tenantId` — users cannot see other tenants' posts.
- `update()` checks both permissions and ownership — admins can edit any post; authors can only edit their own.
- Requests without the required permissions return `403`.

**Why this works:** Authorization is layered (guards + service-level checks + tenant filtering). Deny by default. Resource-level checks prevent IDOR. Tenant isolation prevents cross-tenant access. Audit logging (not shown) would record every decision.

### Real-World Cases

- **SaaS platforms:** Tenant isolation, RBAC, and resource ownership.
- **Banking:** Fine-grained permissions, resource-level checks, and audit logging.
- **Healthcare:** Role-based access with patient-level ownership checks.
- **E-commerce:** Vendor isolation and order ownership.

---

## Core Concept 3: Session Security

### Definitions

**Core Definition:** Session security is the set of controls that protect active login sessions from hijacking, fixation, forgery, and other attacks that could allow an attacker to impersonate an authenticated user.

**Technical Definition:** Session security requires: (1) **secure session IDs** — cryptographically random, ≥128 bits; (2) **secure cookie flags** — `HttpOnly`, `Secure`, `SameSite`, `__Host-` prefix; (3) **session fixation prevention** — regenerate session IDs on login and privilege changes; (4) **session hijacking prevention** — fingerprinting (IP, User-Agent), short timeouts, HTTPS-only; (5) **session forgery prevention** — signed cookies, HMAC, or server-side storage; (6) **CSRF protection** — synchronizer tokens, double-submit cookies, `SameSite`; (7) **session expiration** — absolute and idle timeouts; (8) **revocation** — immediate invalidation on logout, password change, and admin action.

**Beginner-Friendly Explanation:** Session security is like protecting a temporary badge. The badge has a unique number (session ID), it's stored in a sealed envelope (`HttpOnly`), only shown over a secure line (`Secure`), and only used at the same building (`SameSite`). When you get a promotion (privilege change), you get a new badge (session regeneration). If someone steals your badge, the system notices because they're not in the same building (fingerprinting). And when you leave (logout), the badge is destroyed immediately.

### Purposes

- To protect active sessions from theft and impersonation.
- To prevent session fixation and hijacking.
- To ensure that session cookies cannot be read by JavaScript (XSS).
- To ensure that session cookies cannot be intercepted (network sniffing).
- To prevent CSRF attacks on state-changing operations.
- To enable immediate session revocation.

### Syntax Rules and Structure

#### Session Cookie Flags

| Flag | Purpose | Value |
|------|---------|-------|
| `HttpOnly` | Block JavaScript access | Always set |
| `Secure` | HTTPS only | Always set (production) |
| `SameSite` | CSRF mitigation | `Lax` or `Strict` |
| `__Host-` prefix | Strongest protection | Use for session cookies |
| `Max-Age` | Lifetime | 15 min – 24 hours |
| `Path` | Scope | `/` |

#### Session Security Rules

| Rule | Implementation |
|------|----------------|
| **Regenerate on login** | Prevent fixation |
| **Regenerate on privilege change** | Prevent escalation |
| **Use CSPRNG** | `crypto.randomBytes(32)` |
| **Fingerprint sessions** | IP, User-Agent, Accept-Language |
| **Absolute timeout** | 8 hours maximum |
| **Idle timeout** | 15–30 minutes |
| **Revoke on logout** | Destroy session record |
| **Revoke on password change** | All sessions |
| **CSRF protection** | Synchronizer token |
| **Session store** | Redis (shared, scalable) |

#### Syntax Rules

- **Use `HttpOnly`, `Secure`, `SameSite=Lax`** — minimum cookie protection.
- **Use the `__Host-` prefix** — strongest cookie protection.
- **Regenerate session IDs on login and privilege changes.**
- **Use CSPRNG for session IDs** — ≥128 bits.
- **Implement absolute and idle timeouts.**
- **Fingerprint sessions** — IP, User-Agent, Accept-Language.
- **Implement CSRF protection** — synchronizer tokens or double-submit.
- **Store sessions server-side** — never trust client-side session data.
- **Revoke sessions on logout, password change, and admin action.**
- **Use HTTPS only** — session cookies must never traverse HTTP.

#### Constraints and Limitations

- **Fingerprinting can cause false positives** — mobile IPs change frequently.
- **Session store is a single point of failure** — high availability is essential.
- **CSRF protection adds complexity** — but is mandatory for cookie-based sessions.
- **Session cookies are sent on every request** — overhead.
- **Cross-domain sessions are complex** — cookies are domain-scoped.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Session Management with Redis, CSRF, and Fingerprinting (Express)

```typescript
// server.ts
import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';
import { randomBytes } from 'node:crypto';
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
    genid: () => randomBytes(32).toString('base64url'),
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax',
      maxAge: 30 * 60 * 1000, // 30 minutes idle timeout
      path: '/',
    },
    rolling: true,
  }),
);

app.use(csrf({ cookie: { key: '__Host-csrf', httpOnly: true, secure: true, sameSite: 'lax' } }));

// Login: regenerate session ID
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

// Fingerprint check middleware
function requireAuth(req, res, next) {
  if (!req.session.userId) return res.status(401).json({ error: 'Unauthorized' });

  // Absolute timeout
  if (Date.now() - req.session.createdAt > 8 * 60 * 60 * 1000) {
    req.session.destroy(() => res.status(401).json({ error: 'Session expired' }));
    return;
  }

  // Fingerprint check
  if (
    req.session.ip !== req.ip ||
    req.session.userAgent !== req.get('User-Agent') ||
    req.session.acceptLanguage !== req.get('Accept-Language')
  ) {
    req.session.destroy(() => res.status(401).json({ error: 'Session invalid' }));
    return;
  }

  next();
}

// Logout: destroy session
app.post('/auth/logout', requireAuth, (req, res, next) => {
  req.session.destroy((err) => {
    if (err) return next(err);
    res.clearCookie('__Host-sid', { path: '/' });
    res.status(204).end();
  });
});

app.get('/me', requireAuth, (req, res) => {
  res.json({ userId: req.session.userId, roles: req.session.roles });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- Login regenerates the session ID, preventing fixation.
- Sessions expire after 30 minutes of inactivity (idle) or 8 hours total (absolute).
- Fingerprint mismatches destroy the session.
- CSRF tokens are required for state-changing requests.
- Logout destroys the session and clears the cookie.

**Why this works:** Multiple layers of session security — secure session IDs, cookie flags, regeneration, fingerprinting, timeouts, CSRF protection, and revocation.

### Real-World Cases

- **Banking:** Short idle timeouts, fingerprinting, MFA, and immediate revocation.
- **SaaS:** Redis-backed sessions, `SameSite=Lax`, and CSRF protection.
- **Healthcare:** Fingerprinting, audit logging, and HIPAA-compliant session management.
- **Enterprise:** SSO sessions with strict timeouts and revocation.

---

## Core Concept 4: Transport Security

### Definitions

**Core Definition:** Transport security is the set of controls that protect data in transit between clients and servers, primarily through TLS/HTTPS, strict cipher suites, and HTTPS-only redirection.

**Technical Definition:** Transport security requires: (1) **TLS 1.2+** (preferably TLS 1.3) with strong cipher suites; (2) **valid certificates** from a trusted CA; (3) **HSTS** (HTTP Strict Transport Security) with `max-age` ≥ 1 year, `includeSubDomains`, and `preload`; (4) **HTTPS-only** — all HTTP requests redirected to HTTPS (301/308); (5) **secure cookie flags** — `Secure` ensures cookies are only sent over HTTPS; (6) **certificate pinning** (for mobile apps); (7) **OCSP stapling** for certificate revocation; (8) **perfect forward secrecy** (ECDHE cipher suites). Transport security is the foundation of all other web security — without it, credentials, sessions, and tokens can be intercepted.

**Beginner-Friendly Explanation:** Transport security is like sending a letter in a locked, armored truck instead of a postcard. Anyone can read a postcard (HTTP), but the armored truck (HTTPS) keeps the contents private and tamper-proof. HSTS is like telling the post office "never accept a postcard from me — always use the armored truck." Cipher suites are the type of lock on the truck — some are strong, some are weak. Transport security is the foundation: without it, everything else is exposed.

### Purposes

- To protect data in transit from interception and tampering.
- To prevent man-in-the-middle (MITM) attacks.
- To ensure the authenticity of the server.
- To protect credentials, sessions, and tokens.
- To comply with security standards (PCI DSS, HIPAA, GDPR).
- To enable modern browser features (Service Workers, Geolocation, etc.).

### Syntax Rules and Structure

#### TLS Configuration

| Setting | Recommended Value |
|---------|-------------------|
| **TLS version** | TLS 1.3 (preferred), TLS 1.2 (minimum) |
| **Cipher suites** | `TLS_AES_256_GCM_SHA384`, `TLS_CHACHA20_POLY1305_SHA256` (TLS 1.3) |
| **Key exchange** | ECDHE (forward secrecy) |
| **Certificate** | Valid, from a trusted CA, ≥2048-bit RSA or ≥256-bit ECDSA |
| **OCSP stapling** | Enabled |
| **HSTS** | `max-age=31536000; includeSubDomains; preload` |

#### HSTS Header

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

| Directive | Purpose |
|-----------|---------|
| `max-age=31536000` | 1 year (recommended minimum) |
| `includeSubDomains` | Apply to all subdomains |
| `preload` | Submit to browser preload lists |

#### HTTPS Redirect (Express)

```typescript
// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (req.secure || req.headers['x-forwarded-proto'] === 'https') {
    return next();
  }
  res.redirect(301, `https://${req.headers.host}${req.url}`);
});

// HSTS header
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  next();
});
```

#### TLS Configuration (Node.js HTTPS Server)

```typescript
import https from 'node:https';
import { readFileSync } from 'node:fs';

const server = https.createServer({
  key: readFileSync('./certs/private.key'),
  cert: readFileSync('./certs/certificate.crt'),
  ca: readFileSync('./certs/ca-bundle.crt'),
  minVersion: 'TLSv1.2',
  maxVersion: 'TLSv1.3',
  ciphers: [
    'TLS_AES_256_GCM_SHA384',
    'TLS_CHACHA20_POLY1305_SHA256',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES128-GCM-SHA256',
  ].join(':'),
  honorCipherOrder: true,
}, app);
```

#### Syntax Rules

- **Use TLS 1.3** (or TLS 1.2 minimum) — never TLS 1.0/1.1.
- **Use strong cipher suites** — AES-GCM or ChaCha20-Poly1305.
- **Use ECDHE for forward secrecy.**
- **Use valid certificates** from a trusted CA.
- **Enable HSTS** with `max-age` ≥ 1 year, `includeSubDomains`, `preload`.
- **Redirect all HTTP to HTTPS** with 301/308.
- **Set the `Secure` flag on all cookies.**
- **Use OCSP stapling** for certificate revocation.
- **Test with SSL Labs** — aim for an A+ rating.
- **Monitor certificate expiration** — automate renewal (Let's Encrypt, ACME).

#### Constraints and Limitations

- **TLS adds latency** — mitigated by TLS 1.3 and session resumption.
- **Certificate management is complex** — automation is essential.
- **HSTS cannot be easily undone** — once a browser sees it, it enforces HTTPS.
- **HSTS `preload` requires submission** — and removal is slow.
- **Mixed content** — HTTPS pages must not load HTTP resources.
- **TLS termination at load balancers** — requires careful configuration.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Transport Security Configuration (Express + Helmet)

```typescript
// server.ts
import express from 'express';
import helmet from 'helmet';
import https from 'node:https';
import { readFileSync } from 'node:fs';

const app = express();

// Helmet: sets security headers including HSTS
app.use(helmet({
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true,
  },
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      frameAncestors: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
    },
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
}));

// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (req.secure || req.headers['x-forwarded-proto'] === 'https') {
    return next();
  }
  res.redirect(301, `https://${req.headers.host}${req.url}`);
});

// Cookies: Secure flag
app.use((req, res, next) => {
  const originalCookie = res.cookie.bind(res);
  res.cookie = (name, value, options = {}) => {
    return originalCookie(name, value, {
      ...options,
      secure: true,
      httpOnly: true,
      sameSite: 'lax',
    });
  };
  next();
});

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'Secure connection' });
});

// HTTPS server
const httpsServer = https.createServer({
  key: readFileSync('./certs/private.key'),
  cert: readFileSync('./certs/certificate.crt'),
  minVersion: 'TLSv1.2',
  maxVersion: 'TLSv1.3',
  honorCipherOrder: true,
}, app);

httpsServer.listen(443, () => console.log('HTTPS server on port 443'));

// HTTP redirect server
const httpApp = express();
httpApp.use((req, res) => {
  res.redirect(301, `https://${req.headers.host}${req.url}`);
});
httpApp.listen(80, () => console.log('HTTP redirect server on port 80'));
```

**Expected behaviour:**
- All HTTP requests are redirected to HTTPS.
- HSTS is set with `max-age=31536000; includeSubDomains; preload`.
- All cookies are set with `Secure`, `HttpOnly`, and `SameSite=Lax`.
- Only TLS 1.2 and 1.3 are allowed, with strong cipher suites.
- Security headers (CSP, X-Frame-Options, etc.) are set by Helmet.

**Why this works:** Multiple layers of transport security — TLS configuration, HSTS, HTTPS redirect, secure cookies, and security headers. The result is an A+ rating on SSL Labs.

### Real-World Cases

- **Banking:** TLS 1.3, HSTS preload, and certificate pinning.
- **E-commerce:** TLS 1.2+, HSTS, and secure cookies for PCI DSS.
- **Healthcare:** TLS 1.2+, HSTS, and audit logging for HIPAA.
- **SaaS:** Let's Encrypt automation, HSTS, and CSP.

---

## Core Concept 5: Least Privilege Architecture

### Definitions

**Core Definition:** Least privilege is the principle that every user, service, process, and system component should have only the minimum permissions necessary to perform its function — nothing more.

**Technical Definition:** Least privilege applies to: (1) **database users** — separate users for read/write/admin, with minimal grants; (2) **file system access** — application runs as a non-root user with restricted file permissions; (3) **network capabilities** — firewalls, security groups, and network policies that allow only necessary traffic; (4) **cloud IAM** — roles and policies scoped to specific resources and actions; (5) **service accounts** — each service has its own account with minimal permissions; (6) **containers** — non-root users, read-only file systems, dropped capabilities; (7) **secrets management** — secrets stored in vaults, not in code or environment variables. Least privilege limits the blast radius of a compromise: if an attacker gains access to one component, they cannot pivot to others.

**Beginner-Friendly Explanation:** Least privilege is like giving each employee a key to only the doors they need. The janitor doesn't need the CEO's office key. The CEO doesn't need the server room key. If someone steals the janitor's key, they can only open the janitor's doors — not the whole building. In software, this means the web server can't drop tables, the application can't read /etc/shadow, and the database can't make outbound network calls.

### Purposes

- To limit the blast radius of a compromise.
- To prevent privilege escalation and lateral movement.
- To comply with security standards (PCI DSS, HIPAA, SOC 2).
- To reduce the attack surface.
- To enable fine-grained audit and monitoring.
- To enforce separation of duties.

### Syntax Rules and Structure

#### Database Least Privilege

| Role | Permissions | Use Case |
|------|-------------|----------|
| **app_read** | `SELECT` on application tables | Read-only queries |
| **app_write** | `SELECT`, `INSERT`, `UPDATE`, `DELETE` on application tables | Application operations |
| **app_migrate** | `ALTER`, `CREATE`, `DROP` on application schema | Migrations only |
| **app_admin** | All privileges | Administrative tasks only |
| **reporting** | `SELECT` on views only | Analytics |

#### PostgreSQL Least Privilege

```sql
-- Create application user with minimal privileges
CREATE USER app_write WITH PASSWORD 'strong-password';
GRANT CONNECT ON DATABASE myapp TO app_write;
GRANT USAGE ON SCHEMA public TO app_write;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_write;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_write;

-- Revoke default public privileges
REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON DATABASE myapp FROM PUBLIC;

-- Create read-only user for reporting
CREATE USER app_read WITH PASSWORD 'strong-password';
GRANT CONNECT ON DATABASE myapp TO app_read;
GRANT USAGE ON SCHEMA public TO app_read;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_read;

-- Create migration user (used only during deployment)
CREATE USER app_migrate WITH PASSWORD 'strong-password';
GRANT ALL PRIVILEGES ON DATABASE myapp TO app_migrate;

-- Row-level security (defense in depth)
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON posts
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

#### File System Least Privilege

```dockerfile
# Dockerfile — run as non-root user
FROM node:20-alpine

# Create a non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY --chown=nodejs:nodejs . .

# Drop privileges
USER nodejs

# Read-only file system (when possible)
# docker run --read-only --tmpfs /tmp my-app

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

#### Cloud IAM Least Privilege (AWS)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-app-uploads/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail"
      ],
      "Resource": "arn:aws:ses:us-east-1:123456789012:identity/example.com"
    }
  ]
}
```

#### Syntax Rules

- **Use separate database users** for read, write, migrate, and admin.
- **Grant only the minimum privileges** — no `GRANT ALL` for application users.
- **Revoke public privileges** — `REVOKE ALL FROM PUBLIC`.
- **Run applications as non-root** — never run Node.js as root.
- **Use read-only file systems** — where possible (containers).
- **Drop Linux capabilities** — `--cap-drop=ALL`, add only what's needed.
- **Scope IAM policies** to specific resources and actions.
- **Use separate service accounts** for each service.
- **Store secrets in a vault** — not in code or environment variables.
- **Rotate credentials regularly.**
- **Audit permissions regularly** — remove unused grants.

#### Constraints and Limitations

- **Least privilege requires discipline** — it's easy to over-grant "just to make it work."
- **Debugging is harder** — permission errors may be opaque.
- **Migration complexity** — transitioning from over-privileged to least-privileged takes time.
- **Operational overhead** — more users, roles, and policies to manage.
- **Some frameworks require elevated privileges** — e.g., migrations need DDL.
- **Secrets management adds complexity** — vaults, rotation, and access control.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Least Privilege Database Access (Node.js + PostgreSQL)

```typescript
// db.ts — separate connections for read and write
import { Pool } from 'pg';

// Write pool — uses app_write user
export const writePool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_WRITE_USER,       // app_write
  password: process.env.DB_WRITE_PASSWORD,
  max: 20,
});

// Read pool — uses app_read user
export const readPool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_READ_USER,        // app_read
  password: process.env.DB_READ_PASSWORD,
  max: 50,                                // More read connections
});

// Repository using the appropriate pool
export class UserRepository {
  async findById(id: string): Promise<User | null> {
    const result = await readPool.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] ?? null;
  }

  async create(data: CreateUserData): Promise<User> {
    const result = await writePool.query(
      'INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *',
      [data.email, data.name],
    );
    return result.rows[0];
  }

  async update(id: string, data: UpdateUserData): Promise<User> {
    const result = await writePool.query(
      'UPDATE users SET name = $1 WHERE id = $2 RETURNING *',
      [data.name, id],
    );
    return result.rows[0];
  }
}
```

```typescript
// migrations/run.ts — uses app_migrate user (separate)
import { Pool } from 'pg';

const migrationPool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_MIGRATE_USER,     // app_migrate
  password: process.env.DB_MIGRATE_PASSWORD,
});

async function runMigrations() {
  const migrations = await loadMigrations();
  for (const migration of migrations) {
    await migrationPool.query(migration.sql);
  }
  await migrationPool.end();
}

runMigrations();
```

**Expected behaviour:**
- Read queries use `app_read` (SELECT only).
- Write queries use `app_write` (SELECT, INSERT, UPDATE, DELETE).
- Migrations use `app_migrate` (ALTER, CREATE, DROP) — only during deployment.
- If the application is compromised, the attacker cannot DROP tables or read other databases.

**Why this works:** Separate database users with minimal privileges. The application cannot perform DDL operations. Migrations run with a separate, more privileged user. This limits the blast radius of a compromise.

#### Example 2: Least Privilege Container (Docker)

```dockerfile
# Dockerfile — hardened Node.js container
FROM node:20-alpine

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

WORKDIR /app

# Install dependencies as root (build stage)
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy application code with correct ownership
COPY --chown=nodejs:nodejs . .

# Drop privileges
USER nodejs

# Use dumb-init to handle signals
ENTRYPOINT ["dumb-init", "--"]

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

```yaml
# docker-compose.yml — security-hardened container
services:
  app:
    build: .
    read_only: true                    # Read-only file system
    tmpfs:
      - /tmp                           # Writable temp directory
    cap_drop:
      - ALL                            # Drop all Linux capabilities
    cap_add:
      - NET_BIND_SERVICE               # Only if binding to port < 1024
    security_opt:
      - no-new-privileges:true         # Prevent privilege escalation
    user: "1001:1001"                  # Run as non-root
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - app-network

secrets:
  db_password:
    external: true

networks:
  app-network:
    driver: bridge
```

**Expected behaviour:**
- The container runs as a non-root user (`nodejs`, UID 1001).
- The file system is read-only; only `/tmp` is writable.
- All Linux capabilities are dropped.
- Privilege escalation is prevented (`no-new-privileges`).
- Secrets are mounted from a vault, not environment variables.

**Why this works:** Multiple layers of least privilege — non-root user, read-only file system, dropped capabilities, no privilege escalation, and secrets management. If the container is compromised, the attacker has minimal access.

### Real-World Cases

- **Banking:** Separate database users for read/write/migrate, non-root containers, and IAM roles.
- **Healthcare:** Row-level security, non-root containers, and audit logging.
- **E-commerce:** Scoped IAM policies, read replicas for reads, and secrets in vaults.
- **SaaS:** Per-tenant database schemas, non-root containers, and scoped service accounts.
- **Enterprise:** Zero-trust networking, IAM roles, and continuous permission audits.

---

## References

- OWASP Top 10 — https://owasp.org/www-project-top-ten/
- OWASP Application Security Verification Standard (ASVS) — https://owasp.org/www-project-application-security-verification-standard/
- OWASP Cheat Sheet Series — Authentication — https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Authorization — https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Session Management — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Transport Layer Security — https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Password Storage — https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross-Site Request Forgery Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- NIST SP 800-63B — Digital Identity Guidelines — https://pages.nist.gov/800-63-3/sp800-63b.html
- NIST SP 800-52 Rev. 2 — Guidelines for TLS — https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final
- NIST SP 800-53 — Security and Privacy Controls — https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final
- RFC 8446 — TLS 1.3 — https://www.rfc-editor.org/rfc/rfc8446
- RFC 6797 — HTTP Strict Transport Security (HSTS) — https://www.rfc-editor.org/rfc/rfc6797
- RFC 6265 — HTTP State Management Mechanism (Cookies) — https://www.rfc-editor.org/rfc/rfc6265
- RFC 9700 — Best Current Practice for OAuth 2.0 Security — https://www.rfc-editor.org/rfc/rfc9700
- MDN Web Docs — Transport Layer Security — https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security
- MDN Web Docs — Strict-Transport-Security — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- SSL Labs — SSL Server Test — https://www.ssllabs.com/ssltest/
- HSTS Preload — https://hstspreload.org/
- Helmet Documentation — https://helmetjs.github.io/
- express-rate-limit — npm package — https://www.npmjs.com/package/express-rate-limit
- express-slow-down — npm package — https://www.npmjs.com/package/express-slow-down
- HaveIBeenPwned — Pwned Passwords API — https://haveibeenpwned.com/API/v3#PwnedPasswords
- PostgreSQL Documentation — Row-Level Security — https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- PostgreSQL Documentation — GRANT — https://www.postgresql.org/docs/current/sql-grant.html
- Docker Documentation — Security — https://docs.docker.com/engine/security/
- AWS — IAM Best Practices — https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- AWS — Least Privilege — https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege
- CIS Benchmarks — https://www.cisecurity.org/cis-benchmarks
- OWASP — Insecure Direct Object Reference (IDOR) — https://owasp.org/www-project-top-ten/2017/A4_2017-Insecure_Direct_Object_References
- OWASP — Broken Access Control — https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- PortSwigger — Web Security Academy — https://portswigger.net/web-security