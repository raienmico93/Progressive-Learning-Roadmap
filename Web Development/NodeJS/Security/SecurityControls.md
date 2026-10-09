# Security Controls — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Security controls are the technical mechanisms, configurations, and practices applied to a web application to prevent, detect, and respond to security threats — forming the defensive layers between an attacker and the application's assets.

**Technical Definition:** Security controls implement the preventive, detective, and corrective measures defined by security frameworks (OWASP ASVS, NIST SP 800-53, CIS Controls). Preventive controls include input validation, output encoding, rate limiting, security headers, CORS configuration, request-size limits, secure cookies, and secret management. These controls are enforced at multiple layers: the edge (CDN, WAF, reverse proxy), the application (middleware, guards, validation pipes), and the infrastructure (secrets manager, TLS termination). Defense in depth requires that no single control is relied upon exclusively; each complements the others.

**Beginner-Friendly Explanation:** Think of security controls as the layers of protection around a castle. **Input validation** is the gatekeeper who checks everyone entering. **Output encoding** is the scribe who rewrites messages so they can't contain hidden instructions. **Rate limiting** is the guard who stops anyone from knocking on the door too many times. **Security headers** are the castle's defensive walls (CSP, HSTS). **CORS** is the list of friendly kingdoms allowed to send messengers. **Request-size limits** prevent someone from dumping a cart of rocks at the gate. **Secure cookies** are tamper-proof seals on important documents. **Secret management** is the vault where the keys to the castle are stored. Together, they make the castle extremely difficult to breach.

### Key Characteristics

- **Defense in depth:** Multiple overlapping controls, so no single failure compromises the system.
- **Layered enforcement:** Applied at the edge, application, and infrastructure layers.
- **Configurable:** Each control has parameters that must be tuned for the application.
- **Framework-supported:** Modern frameworks (Express, NestJS) provide built-in mechanisms.
- **Testable:** Each control can be verified with automated tests and security scanners.
- **Standards-aligned:** Implement OWASP ASVS, NIST, and CIS recommendations.

### Prerequisites

- **HTTP fundamentals:** Headers, methods, status codes, cookies.
- **Web security concepts:** XSS, CSRF, SQL injection, DoS.
- **Node.js fundamentals:** Express or NestJS, middleware, async patterns.
- **Validation libraries:** Joi, Zod, express-validator, class-validator.
- **Security headers:** CSP, HSTS, X-Frame-Options, Referrer-Policy.
- **CORS:** Same-origin policy, preflight requests, `Access-Control-*` headers.
- **Secrets management:** Environment variables, `.env` files, vaults.

### Related Programming Areas

- **Web security fundamentals:** The broader context for these controls.
- **Common web vulnerabilities:** The threats these controls mitigate.
- **Node.js-specific security:** Runtime-specific controls.
- **DevSecOps:** SAST, DAST, SCA, and secret scanning.
- **Compliance:** PCI DSS, HIPAA, SOC 2, GDPR, ISO 27001.

### Core Concepts

1. **Input Validation** — enforcing strict schema validation rules using Joi, Zod, or express-validator.
2. **Output Encoding** — escaping dynamic string inputs before parsing them into HTML or execution scripts.
3. **Rate Limiting** — throttling request thresholds using memory or distributed Redis stores.
4. **Security Headers** — implementing context-specific headers using Helmet (CSP, HSTS).
5. **CORS Configuration** — restricting API access paths to verified domain origins.
6. **Request-Size Limits** — configuring explicit body-parser caps to prevent memory flooding.
7. **Secure Cookies** — applying `HttpOnly`, `Secure`, and `SameSite` to all state trackers.
8. **Secret Management** — storing and loading environment variables securely using encrypted vaults, `.env.vault`, or cloud providers.

---

## Core Concept 1: Input Validation

### Definitions

**Core Definition:** Input validation is the process of verifying that all incoming data conforms to the expected format, type, range, and constraints before it is processed by the application.

**Technical Definition:** Input validation enforces a schema (allow-list) on all untrusted input — request bodies, query parameters, path parameters, headers, and cookies. Validation libraries (Zod, Joi, express-validator, class-validator) define schemas declaratively, reject invalid input with structured errors, and coerce types where appropriate. Validation must occur at the boundary (controller or middleware) before any business logic executes. Allow-list validation (accept only known-good) is preferred over deny-list validation (reject known-bad). Validation must be repeated server-side, even if client-side validation exists.

**Beginner-Friendly Explanation:** Input validation is like a bouncer at a club with a guest list. Only people whose names are on the list (valid data) get in. Anyone not on the list is turned away. The bouncer does not try to identify "bad" people — they only let in "good" ones. This is allow-list validation. It's much safer than trying to identify every possible bad person (deny-list).

### Purposes

- To prevent malformed data from reaching business logic.
- To prevent injection attacks (SQL, NoSQL, command, XSS).
- To enforce data integrity and business constraints.
- To provide clear error messages for invalid input.
- To prevent type confusion and unexpected data structures.

### Syntax Rules and Structure

#### Zod Schema

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase().trim(),
  name: z.string().min(1).max(100).trim(),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
}).strict(); // Reject unknown properties

type CreateUserDto = z.infer<typeof CreateUserSchema>;
```

#### Joi Schema

```typescript
import Joi from 'joi';

const createUserSchema = Joi.object({
  email: Joi.string().email().max(255).lowercase().trim().required(),
  name: Joi.string().min(1).max(100).trim().required(),
  age: Joi.number().integer().min(0).max(150).optional(),
  role: Joi.string().valid('user', 'editor', 'admin').default('user'),
}).options({ stripUnknown: true, abortEarly: false });
```

#### express-validator

```typescript
import { body, query, param, validationResult } from 'express-validator';

const createUserValidation = [
  body('email').isEmail().normalizeEmail().isLength({ max: 255 }),
  body('name').trim().isLength({ min: 1, max: 100 }),
  body('age').optional().isInt({ min: 0, max: 150 }).toInt(),
  body('role').optional().isIn(['user', 'editor', 'admin']),
];
```

#### Syntax Rules

- **Use allow-list validation** — accept only known-good values.
- **Validate at the boundary** — controller or middleware, before business logic.
- **Use strict schemas** — reject unknown properties (`strict()`, `stripUnknown: false`).
- **Coerce types explicitly** — convert strings to numbers, booleans.
- **Validate all input sources** — body, query, params, headers, cookies.
- **Use consistent error responses** — structured, non-leaking.
- **Repeat validation server-side** — never trust client-side validation.
- **Limit string lengths** — prevent buffer overflow and DoS.
- **Use `.trim()` and `.toLowerCase()`** — normalise input.
- **Test with malicious payloads** — SQL injection, XSS, prototype pollution.

#### Constraints and Limitations

- **Validation is not sanitisation** — validated input may still need encoding for output.
- **Complex schemas are hard to maintain** — balance strictness with usability.
- **Some attacks bypass validation** — business logic flaws, race conditions.
- **Validation errors can leak information** — use generic messages for sensitive fields.
- **Performance** — complex validation adds latency.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Comprehensive Input Validation (NestJS + Zod)

```typescript
// dto/create-user.dto.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email('Invalid email').max(255).toLowerCase().trim(),
  name: z.string().min(1, 'Name is required').max(100).trim(),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(['user', 'editor', 'admin']).default('user'),
  address: z.object({
    street: z.string().max(200).trim(),
    city: z.string().max(100).trim(),
    zip: z.string().regex(/^\d{5}$/, 'ZIP must be 5 digits'),
  }).optional(),
}).strict();

export type CreateUserDto = z.infer<typeof CreateUserSchema>;
```

```typescript
// pipes/zod-validation.pipe.ts
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';
import { ZodSchema } from 'zod';

@Injectable()
export class ZodValidationPipe implements PipeTransform {
  constructor(private schema: ZodSchema) {}

  transform(value: unknown) {
    const result = this.schema.safeParse(value);
    if (!result.success) {
      throw new BadRequestException({
        error: 'ValidationError',
        details: result.error.issues.map((issue) => ({
          field: issue.path.join('.'),
          message: issue.message,
        })),
      });
    }
    return result.data;
  }
}
```

```typescript
// users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @UsePipes(new ZodValidationPipe(CreateUserSchema))
  async create(@Body() dto: CreateUserDto): Promise<UserDto> {
    return this.usersService.create(dto);
  }

  @Get()
  async findAll(
    @Query('page', new ParseIntPipe({ optional: true })) page = 1,
    @Query('limit', new ParseIntPipe({ optional: true })) limit = 20,
  ) {
    if (limit > 100) throw new BadRequestException('Limit cannot exceed 100');
    return this.usersService.findAll({ page, limit });
  }
}
```

**Expected behaviour:**
- Valid input creates a user.
- Invalid email returns `400` with field-level errors.
- Unknown properties are rejected (strict schema).
- `limit > 100` returns `400`.

**Why this works:** Zod provides type-safe schema validation. The pipe applies the schema at the boundary. NestJS pipes handle path/query parameters. All input is validated before business logic.

### Real-World Cases

- **Registration forms:** Email format, password strength, name length.
- **API endpoints:** Query parameters, pagination, filtering.
- **File uploads:** MIME types, file sizes, file names.
- **Webhooks:** Payload signatures, schema validation.

---

## Core Concept 2: Output Encoding

### Definitions

**Core Definition:** Output encoding is the process of converting dynamic data into a safe representation for the target context (HTML, JavaScript, URL, CSS) before it is rendered or executed.

**Technical Definition:** Output encoding (also called escaping) transforms special characters into their safe equivalents — `<` becomes `&lt;`, `>` becomes `&gt;`, `"` becomes `&quot;`, `'` becomes `&#x27;`. Different contexts require different encodings: **HTML entity encoding** for HTML content, **JavaScript encoding** for script contexts, **URL encoding** for URL parameters, **CSS encoding** for style contexts, and **attribute encoding** for HTML attributes. Modern frameworks (React, Vue, Angular) auto-encode by default. Server-side templates must explicitly encode. Rich text requires sanitisation (DOMPurify) rather than encoding.

**Beginner-Friendly Explanation:** Output encoding is like translating a message into a language where dangerous words lose their power. If someone writes `<script>alert(1)</script>` in a comment, encoding turns it into `&lt;script&gt;alert(1)&lt;/script&gt;` — which displays as text instead of executing. The message is preserved, but its power to run code is neutralised.

### Purposes

- To prevent XSS by neutralising malicious scripts.
- To prevent HTML injection and defacement.
- To ensure that user-generated content is displayed safely.
- To prevent injection in JavaScript, URL, and CSS contexts.
- To comply with OWASP Top 10 (A03:2021 — Injection).

### Syntax Rules and Structure

#### Context-Specific Encoding

| Context | Encoding | Example |
|---------|----------|---------|
| **HTML content** | HTML entity | `<` → `&lt;` |
| **HTML attribute** | Attribute encoding | `"` → `&quot;` |
| **JavaScript** | JS encoding | `<` → `\u003c` |
| **URL** | Percent encoding | ` ` → `%20` |
| **CSS** | CSS encoding | `<` → `\3c` |
| **JSON** | JSON escaping | `"` → `\"` |

#### HTML Entity Encoding

```typescript
function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}
```

#### Framework Auto-Encoding

```tsx
// ✅ SAFE — React escapes by default
<div>{userInput}</div>

// ❌ VULNERABLE — dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE — Vue escapes by default
<div>{{ userInput }}</div>

// ❌ VULNERABLE — v-html
<div v-html="userInput"></div>
```

#### HTML Sanitisation (DOMPurify)

```typescript
import DOMPurify from 'isomorphic-dompurify';

// ✅ SAFE — sanitise before inserting HTML
const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
  ALLOWED_ATTR: ['href', 'title'],
  ALLOWED_URI_REGEXP: /^(?:https?|mailto):/i,
});
```

#### Syntax Rules

- **Encode for the specific context** — HTML, JavaScript, URL, CSS.
- **Use framework auto-encoding** — React, Vue, Angular.
- **Avoid `innerHTML`, `dangerouslySetInnerHTML`, `v-html`.**
- **Sanitise rich text with DOMPurify.**
- **Never encode for one context and use in another.**
- **Use `textContent` instead of `innerHTML`.**
- **Encode on output, not on input.**
- **Store raw data; encode at render time.**
- **Validate URLs before rendering links.**
- **Use CSP as defense in depth.**

#### Constraints and Limitations

- **Context is critical** — HTML encoding in JavaScript is ineffective.
- **Rich text editors require sanitisation** — encoding would break formatting.
- **Double encoding is a bug** — encoding twice produces incorrect output.
- **Some frameworks do not auto-encode** — server-side templates require explicit encoding.
- **DOM-based XSS bypasses server-side encoding.**
- **CSP can be bypassed if misconfigured.**

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Context Output Encoding (Express + EJS)

```typescript
import express from 'express';
import ejs from 'ejs';

const app = express();
app.set('view engine', 'ejs');

// ❌ VULNERABLE — unescaped output
app.get('/unsafe', (req, res) => {
  const name = req.query.name as string;
  res.send(`<h1>Hello, ${name}!</h1>`);
});

// ✅ SAFE — HTML entity encoding
function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}

app.get('/safe', (req, res) => {
  const name = escapeHtml(req.query.name as string);
  res.send(`<h1>Hello, ${name}!</h1>`);
});

// ✅ SAFE — EJS auto-escapes with <%= %>
app.get('/ejs', (req, res) => {
  res.render('profile', { name: req.query.name });
  // profile.ejs: <h1>Hello, <%= name %>!</h1>  (auto-escaped)
  // profile.ejs: <h1>Hello, <%- name %>!</h1>  (UNESCAPED — dangerous)
});

// ✅ SAFE — context-specific encoding
function escapeJs(unsafe: string): string {
  return unsafe.replace(/[^a-zA-Z0-9,._]/g, (char) => {
    return '\\u' + char.charCodeAt(0).toString(16).padStart(4, '0');
  });
}

function escapeUrl(unsafe: string): string {
  return encodeURIComponent(unsafe);
}

app.get('/contextual', (req, res) => {
  const data = req.query.data as string;
  res.send(`
    <script>
      const userData = "${escapeJs(data)}"; // JS context
      fetch("/api?data=${escapeUrl(data)}"); // URL context
    </script>
    <div>${escapeHtml(data)}</div> <!-- HTML context -->
  `);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:** `<script>alert(1)</script>` is neutralised in all contexts — displayed as text in HTML, escaped in JavaScript, and percent-encoded in URLs.

**Why this works:** Context-specific encoding ensures that special characters lose their meaning in each context. EJS auto-escapes with `<%= %>`. DOMPurify sanitises rich text.

### Real-World Cases

- **Comment systems:** Displaying user comments safely.
- **User profiles:** Displaying names, bios, and avatars.
- **Search results:** Reflecting search terms safely.
- **Email templates:** Encoding user data in HTML emails.

---

## Core Concept 3: Rate Limiting

### Definitions

**Core Definition:** Rate limiting is the practice of restricting the number of requests a client can make to an API within a specified time window, preventing abuse, brute-force, and denial of service.

**Technical Definition:** Rate limiting algorithms include **fixed window** (simple, but bursty at boundaries), **sliding window** (smoother, more accurate), **token bucket** (allows bursts up to a limit), and **leaky bucket** (smooths traffic). Rate limiting can be applied per-IP, per-user, per-API-key, or per-endpoint. Storage backends include in-memory (development only) and Redis (distributed, production). Rate limiting must return `429 Too Many Requests` with `Retry-After` and `X-RateLimit-*` headers. It complements, but does not replace, authentication and authorization.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer who says "you can only come in 10 times per minute." If you try to come in 11 times, you're turned away until the minute is up. This prevents someone from flooding the door (DoS) or trying a thousand keys in the lock (brute-force).

### Purposes

- To prevent denial of service (DoS) attacks.
- To prevent brute-force and credential stuffing.
- To prevent API abuse and scraping.
- To ensure fair usage across clients.
- To protect downstream services from overload.

### Syntax Rules and Structure

#### Rate Limiting Algorithms

| Algorithm | Pros | Cons |
|-----------|------|------|
| **Fixed Window** | Simple | Bursty at boundaries |
| **Sliding Window** | Smooth | More complex |
| **Token Bucket** | Allows bursts | Requires tuning |
| **Leaky Bucket** | Smooth output | Requires tuning |

#### express-rate-limit Configuration

```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { createClient } from 'redis';

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100,             // 100 requests per minute
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({ client: redisClient, prefix: 'rl:global:' }),
  message: { error: 'Too many requests, please try again later' },
});

// Strict limiter for authentication
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5,                    // 5 attempts per 15 minutes
  skipSuccessfulRequests: true,
  store: new RedisStore({ client: redisClient, prefix: 'rl:auth:' }),
  message: { error: 'Too many login attempts, please try again later' },
});

app.use(globalLimiter);
app.post('/auth/login', authLimiter, authController.login);
```

#### Response Headers

```
RateLimit-Limit: 100
RateLimit-Remaining: 42
RateLimit-Reset: 1712345678
Retry-After: 30
```

#### Syntax Rules

- **Use sliding window or token bucket** — fixed window is bursty.
- **Use Redis for distributed rate limiting** — in-memory does not scale.
- **Apply per-IP, per-user, and per-API-key** — different limits for different identities.
- **Return `429 Too Many Requests`** — with `Retry-After` and `X-RateLimit-*` headers.
- **Exempt trusted IPs** — internal services, health checks.
- **Log rate-limit violations** — for security monitoring.
- **Combine with authentication** — rate limit by user, not just IP.
- **Set stricter limits for authentication** — 5–10 attempts per 15 minutes.
- **Use `skipSuccessfulRequests`** — only count failures.
- **Test under load** — verify limits are enforced.

#### Constraints and Limitations

- **Distributed attacks bypass IP-based limits** — use user-based limits.
- **Redis is a single point of failure** — high availability is essential.
- **Rate limiting can break legitimate clients** — provide clear headers.
- **NAT and proxies** — many users share an IP.
- **Memory-based limits do not scale** — use Redis.
- **Rate limiting is not a substitute for authentication.**

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Tier Rate Limiting (Express + Redis)

```typescript
import express from 'express';
import rateLimit from 'express-rate-limit';
import slowDown from 'express-slow-down';
import RedisStore from 'rate-limit-redis';
import { createClient } from 'redis';

const app = express();
const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

// Tier 1: Global rate limit (100 requests per minute per IP)
const globalLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({ client: redisClient, prefix: 'rl:global:' }),
});

// Tier 2: Progressive slowdown (after 50 requests)
const globalSlowDown = slowDown({
  windowMs: 60 * 1000,
  delayAfter: 50,
  delayMs: (hits) => hits * 100,
  store: new RedisStore({ client: redisClient, prefix: 'sd:global:' }),
});

// Tier 3: Strict auth limit (5 attempts per 15 minutes)
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  standardHeaders: true,
  store: new RedisStore({ client: redisClient, prefix: 'rl:auth:' }),
});

// Tier 4: Per-user rate limit (1000 requests per hour)
const userLimiter = rateLimit({
  windowMs: 60 * 60 * 1000,
  max: 1000,
  keyGenerator: (req) => req.user?.id ?? req.ip!,
  store: new RedisStore({ client: redisClient, prefix: 'rl:user:' }),
});

app.use(globalLimiter, globalSlowDown, userLimiter);
app.post('/auth/login', authLimiter, authController.login);
app.get('/api/data', apiController.getData);

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- After 100 requests per minute per IP, requests return `429`.
- After 50 requests, responses are progressively delayed.
- After 5 failed login attempts per 15 minutes, login is blocked.
- After 1000 requests per hour per user, the user is limited.

**Why this works:** Multiple tiers provide layered protection. Redis ensures distributed enforcement. Progressive delays slow down attackers before blocking them.

### Real-World Cases

- **Authentication:** 5–10 attempts per 15 minutes.
- **API endpoints:** 100–1000 requests per minute per user.
- **Public APIs:** 60 requests per minute per IP.
- **File uploads:** 10 uploads per hour per user.

---

## Core Concept 4: Security Headers

### Definitions

**Core Definition:** Security headers are HTTP response headers that instruct browsers to enforce security policies, such as blocking inline scripts (CSP), enforcing HTTPS (HSTS), and preventing clickjacking (X-Frame-Options).

**Technical Definition:** Key security headers include: **Content-Security-Policy (CSP)** — restricts sources of scripts, styles, images, and other resources; **Strict-Transport-Security (HSTS)** — forces HTTPS for future requests; **X-Content-Type-Options: nosniff** — prevents MIME type sniffing; **X-Frame-Options: DENY** — prevents clickjacking; **Referrer-Policy** — controls referrer information; **Permissions-Policy** — restricts browser features; **Cross-Origin-Opener-Policy (COOP)** and **Cross-Origin-Embedder-Policy (COEP)** — isolate the browsing context; **Cross-Origin-Resource-Policy (CORP)** — controls cross-origin resource sharing.

**Beginner-Friendly Explanation:** Security headers are like instructions you give to every visitor's browser: "Only load scripts from my website" (CSP), "Always use HTTPS" (HSTS), "Don't let anyone embed my site in a frame" (X-Frame-Options). The browser follows these instructions, which stops many attacks automatically.

### Purposes

- To prevent XSS (CSP).
- To prevent clickjacking (X-Frame-Options, CSP frame-ancestors).
- To enforce HTTPS (HSTS).
- To prevent MIME type confusion (X-Content-Type-Options).
- To control referrer leakage (Referrer-Policy).
- To restrict browser features (Permissions-Policy).
- To isolate the browsing context (COOP, COEP).

### Syntax Rules and Structure

#### Key Security Headers

| Header | Purpose | Recommended Value |
|--------|---------|-------------------|
| `Content-Security-Policy` | Restrict resource sources | `default-src 'self'; script-src 'self'; ...` |
| `Strict-Transport-Security` | Enforce HTTPS | `max-age=31536000; includeSubDomains; preload` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `X-Frame-Options` | Prevent clickjacking | `DENY` |
| `Referrer-Policy` | Control referrer | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Restrict features | `geolocation=(), microphone=(), camera=()` |
| `Cross-Origin-Opener-Policy` | Isolate context | `same-origin` |
| `Cross-Origin-Embedder-Policy` | Isolate context | `require-corp` |
| `Cross-Origin-Resource-Policy` | Control resource sharing | `same-origin` |

#### Helmet Configuration

```typescript
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"], // No 'unsafe-inline'
      styleSrc: ["'self'"],  // No 'unsafe-inline'
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
  permissionsPolicy: {
    features: {
      geolocation: [],
      microphone: [],
      camera: [],
      payment: [],
    },
  },
}));
```

#### Syntax Rules

- **Set CSP with no `unsafe-inline`** — use nonces or hashes if needed.
- **Set HSTS with `max-age=31536000; includeSubDomains; preload`.**
- **Set `X-Content-Type-Options: nosniff`.**
- **Set `X-Frame-Options: DENY`** (or CSP `frame-ancestors 'none'`).
- **Set `Referrer-Policy: strict-origin-when-cross-origin`.**
- **Set `Permissions-Policy`** — restrict features the app does not use.
- **Set COOP and COEP** — for cross-origin isolation.
- **Set CORP** — control resource sharing.
- **Test with securityheaders.com** — aim for A+.
- **Use Helmet** — it sets sensible defaults.

#### Constraints and Limitations

- **CSP can break legitimate functionality** — requires careful configuration.
- **CSP reports can be noisy** — use `report-uri` or `report-to`.
- **HSTS cannot be easily undone** — once set, browsers enforce it.
- **HSTS preload requires submission** — removal is slow.
- **Some headers are deprecated** — `X-XSS-Protection` is no longer recommended.
- **Headers must be set on all responses** — including errors.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Full Security Headers with Helmet (Express)

```typescript
import express from 'express';
import helmet from 'helmet';
import crypto from 'node:crypto';

const app = express();

// Generate a nonce for each request (for CSP)
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(16).toString('base64');
  next();
});

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      styleSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
  permissionsPolicy: {
    features: {
      geolocation: [],
      microphone: [],
      camera: [],
      payment: [],
    },
  },
}));

app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <script nonce="${res.locals.cspNonce}">
          console.log('Inline script with nonce');
        </script>
      </head>
      <body>
        <h1>Hello, world!</h1>
      </body>
    </html>
  `);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- All security headers are set on every response.
- Inline scripts are allowed only with the correct nonce.
- HTTPS is enforced (HSTS).
- Clickjacking is prevented (`frame-ancestors 'none'`).
- Referrer information is limited.

**Why this works:** Helmet sets sensible defaults. The nonce allows inline scripts without `unsafe-inline`. HSTS enforces HTTPS. CSP restricts resource sources. All headers are set on every response.

### Real-World Cases

- **All web applications:** Security headers are a baseline requirement.
- **Banking:** Strict CSP with nonces, HSTS preload.
- **E-commerce:** CSP, HSTS, and Permissions-Policy.
- **SaaS:** CSP, HSTS, COOP, COEP.

---

## Core Concept 5: CORS Configuration

### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts which origins can access a server's resources, configured via `Access-Control-Allow-*` response headers.

**Technical Definition:** CORS is enforced by browsers based on the same-origin policy. When a browser makes a cross-origin request, it sends an `Origin` header and (for non-simple requests) a preflight `OPTIONS` request. The server responds with `Access-Control-Allow-Origin` (specific origin or `*`), `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Allow-Credentials`, and `Access-Control-Max-Age`. Misconfigurations include `Access-Control-Allow-Origin: *` with credentials, reflecting the `Origin` header blindly, and overly permissive `Allow-Methods`/`Allow-Headers`.

**Beginner-Friendly Explanation:** CORS is like a guest list for a party. When a browser from another website (origin) wants to talk to your API, your API checks the guest list: "Is this origin allowed?" If yes, the browser lets the request through. If no, the browser blocks the response. The server must explicitly allow each origin.

### Purposes

- To restrict API access to known, trusted origins.
- To prevent unauthorized cross-origin requests.
- To enable legitimate cross-origin communication (SPAs, microservices).
- To protect against CSRF (complementary to other controls).
- To comply with browser security policies.

### Syntax Rules and Structure

#### CORS Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Access-Control-Allow-Origin` | Allowed origins | `https://app.example.com` |
| `Access-Control-Allow-Methods` | Allowed methods | `GET, POST, PATCH, DELETE` |
| `Access-Control-Allow-Headers` | Allowed headers | `Content-Type, Authorization` |
| `Access-Control-Allow-Credentials` | Allow cookies | `true` |
| `Access-Control-Max-Age` | Preflight cache | `86400` |
| `Access-Control-Expose-Headers` | Headers exposed to JS | `X-Total-Count` |

#### cors Middleware Configuration

```typescript
import cors from 'cors';

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com',
  'https://staging.example.com',
];

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (mobile apps, curl, same-origin)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      return callback(null, true);
    }

    callback(new Error('Not allowed by CORS'));
  },
  methods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With', 'X-CSRF-Token'],
  exposedHeaders: ['X-Total-Count', 'X-Page-Count'],
  credentials: true,
  maxAge: 86400,
}));
```

#### Syntax Rules

- **Use an allow-list of origins** — never `*` with credentials.
- **Validate the `Origin` header** — do not reflect it blindly.
- **Set `credentials: true`** only if cookies are required.
- **Limit `Allow-Methods`** — only methods the API uses.
- **Limit `Allow-Headers`** — only headers the API uses.
- **Set `Max-Age`** — cache preflight requests.
- **Handle preflight `OPTIONS`** — respond with `204 No Content`.
- **Expose only necessary headers** — do not leak internal headers.
- **Test with `curl`** — verify CORS behaviour.
- **Log CORS rejections** — for security monitoring.

#### Constraints and Limitations

- **CORS is browser-enforced** — not a server-side control.
- **`*` cannot be used with credentials** — browsers reject it.
- **Subdomains require explicit listing** — `*.example.com` is not supported.
- **Preflight requests add latency** — `Max-Age` mitigates.
- **CORS does not protect against CSRF** — use CSRF tokens.
- **CORS misconfigurations are common** — test thoroughly.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Strict CORS Configuration (Express)

```typescript
import express from 'express';
import cors from 'cors';

const app = express();

const ALLOWED_ORIGINS = new Set([
  'https://app.example.com',
  'https://admin.example.com',
]);

app.use(cors({
  origin: (origin, callback) => {
    // Allow same-origin and non-browser requests
    if (!origin) return callback(null, true);

    // Allow-list check
    if (ALLOWED_ORIGINS.has(origin)) {
      return callback(null, true);
    }

    // Reject — do not throw (avoid leaking origin information)
    callback(null, false);
  },
  methods: ['GET', 'POST', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
  exposedHeaders: ['X-Total-Count'],
  credentials: true,
  maxAge: 86400,
  optionsSuccessStatus: 204,
}));

// Explicit preflight handler (optional)
app.options('*', cors());

app.get('/api/data', (req, res) => {
  res.json({ data: 'hello' });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- Requests from `https://app.example.com` are allowed.
- Requests from `https://evil.com` are rejected (browser blocks the response).
- Preflight `OPTIONS` requests return `204` with CORS headers.
- Credentials (cookies) are allowed for listed origins.

**Why this works:** The allow-list restricts origins. `credentials: true` allows cookies. The preflight is handled correctly. Rejected origins do not receive `Access-Control-Allow-Origin`.

### Real-World Cases

- **SPAs:** Allow the SPA origin to access the API.
- **Microservices:** Allow internal service origins.
- **Third-party APIs:** Allow registered partner origins.
- **Admin panels:** Allow admin origins with stricter limits.

---

## Core Concept 6: Request-Size Limits

### Definitions

**Core Definition:** Request-size limits cap the maximum size of incoming request bodies, preventing memory exhaustion and denial-of-service attacks.

**Technical Definition:** Body parsers (`express.json()`, `express.urlencoded()`, `body-parser`) accept a `limit` option that caps the request body size. Exceeding the limit causes the parser to return `413 Payload Too Large`. Limits should be set based on the expected use case: JSON APIs (100 KB–1 MB), file uploads (10 MB–100 MB), and form submissions (1 MB). Reverse proxies (Nginx, AWS ALB) and CDNs also enforce limits. Multiple layers of limits provide defense in depth.

**Beginner-Friendly Explanation:** Request-size limits are like a mailbox that only accepts letters up to a certain size. If someone tries to stuff a package into it, the mailbox rejects it. This prevents someone from sending a giant file that fills up your server's memory.

### Purposes

- To prevent memory exhaustion from large request bodies.
- To prevent denial of service (DoS) attacks.
- To enforce reasonable payload sizes.
- To protect downstream services from overload.
- To complement rate limiting.

### Syntax Rules and Structure

#### Body Parser Limits

```typescript
import express from 'express';

const app = express();

// JSON body limit (1 MB)
app.use(express.json({ limit: '1mb' }));

// URL-encoded body limit (1 MB)
app.use(express.urlencoded({ extended: true, limit: '1mb' }));

// Raw body limit (for webhooks, 100 KB)
app.use('/webhooks', express.raw({ type: 'application/json', limit: '100kb' }));

// Text body limit (for XML, 500 KB)
app.use('/xml', express.text({ type: 'application/xml', limit: '500kb' }));
```

#### File Upload Limits (Multer)

```typescript
import multer from 'multer';

const upload = multer({
  storage: multer.diskStorage({ destination: './uploads' }),
  limits: {
    fileSize: 10 * 1024 * 1024, // 10 MB
    files: 5,                     // Max 5 files
    fields: 20,                   // Max 20 non-file fields
    fieldSize: 1024 * 1024,       // 1 MB per field
  },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (allowed.includes(file.mimetype)) cb(null, true);
    else cb(new Error('Invalid file type'));
  },
});

app.post('/upload', upload.array('files', 5), (req, res) => {
  res.json({ files: req.files });
});
```

#### Syntax Rules

- **Set explicit limits** — never rely on defaults.
- **Use different limits for different routes** — JSON vs. file uploads.
- **Set limits on the reverse proxy** — Nginx `client_max_body_size`.
- **Set limits on the CDN** — Cloudflare, AWS CloudFront.
- **Return `413 Payload Too Large`** — with a clear error.
- **Validate `Content-Length`** — reject early if it exceeds the limit.
- **Limit nested structures** — prevent deeply nested JSON.
- **Limit array lengths** — prevent huge arrays.
- **Test with large payloads** — verify limits are enforced.
- **Log oversized requests** — for security monitoring.

#### Constraints and Limitations

- **Limits must be tuned** — too low breaks legitimate use cases.
- **`Content-Length` can be spoofed** — the parser must enforce the limit.
- **Streaming requests bypass limits** — use streaming parsers with limits.
- **Multipart parsing is complex** — use `busboy` with limits.
- **Compressed bodies** — decompression can expand beyond the limit.
- **Proxy limits may differ** — align them.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Multi-Layer Request-Size Limits (Express + Nginx)

```typescript
// server.ts
import express from 'express';
import multer from 'multer';

const app = express();

// Global JSON limit (1 MB)
app.use(express.json({ limit: '1mb' }));
app.use(express.urlencoded({ extended: true, limit: '1mb' }));

// Webhook-specific limit (100 KB)
app.use('/webhooks', express.raw({ type: 'application/json', limit: '100kb' }));

// File upload limit (10 MB per file, 5 files)
const upload = multer({
  storage: multer.diskStorage({ destination: './uploads' }),
  limits: {
    fileSize: 10 * 1024 * 1024,
    files: 5,
    fields: 20,
    fieldSize: 1024 * 1024,
  },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'application/pdf'];
    if (allowed.includes(file.mimetype)) cb(null, true);
    else cb(new Error('Invalid file type'));
  },
});

app.post('/upload', upload.array('files', 5), (req, res) => {
  res.json({ files: req.files });
});

// Error handler for 413
app.use((err, req, res, next) => {
  if (err.type === 'entity.too.large') {
    return res.status(413).json({ error: 'Payload too large' });
  }
  if (err.code === 'LIMIT_FILE_SIZE') {
    return res.status(413).json({ error: 'File too large' });
  }
  if (err.code === 'LIMIT_FILE_COUNT') {
    return res.status(413).json({ error: 'Too many files' });
  }
  next(err);
});

app.listen(3000, () => console.log('Server on port 3000'));
```

```nginx
# nginx.conf — reverse proxy limits
http {
  client_max_body_size 10M;        # Global limit
  client_body_buffer_size 128k;    # Buffer size
  client_body_timeout 30s;         # Body timeout

  server {
    listen 443 ssl;
    server_name api.example.com;

    location /upload {
      client_max_body_size 10M;    # Upload limit
      proxy_pass http://localhost:3000;
    }

    location /webhooks {
      client_max_body_size 100k;   # Webhook limit
      proxy_pass http://localhost:3000;
    }

    location /api {
      client_max_body_size 1M;     # API limit
      proxy_pass http://localhost:3000;
    }
  }
}
```

**Expected behaviour:**
- JSON requests over 1 MB return `413`.
- Webhook requests over 100 KB return `413`.
- File uploads over 10 MB return `413`.
- Nginx rejects oversized requests before they reach the application.

**Why this works:** Multiple layers — Nginx (proxy), Express (parser), and Multer (upload). Each layer enforces limits appropriate for its context. Defense in depth.

### Real-World Cases

- **JSON APIs:** 100 KB–1 MB limits.
- **File uploads:** 10 MB–100 MB limits.
- **Webhooks:** 100 KB–1 MB limits.
- **GraphQL:** Limit query complexity and depth.

---

## Core Concept 7: Secure Cookies

### Definitions

**Core Definition:** Secure cookies are HTTP cookies configured with security attributes (`HttpOnly`, `Secure`, `SameSite`, `__Host-` prefix) that prevent theft, tampering, and cross-site abuse.

**Technical Definition:** Cookie security attributes include: **`HttpOnly`** — prevents JavaScript access (mitigates XSS); **`Secure`** — sends only over HTTPS; **`SameSite`** — `Strict`, `Lax`, or `None` (mitigates CSRF); **`Domain`** — scope; **`Path`** — scope; **`Max-Age`/`Expires`** — lifetime; **`__Host-` prefix** — enforces `Secure`, no `Domain`, `Path=/`; **`__Secure-` prefix** — enforces `Secure`. Session cookies should use `HttpOnly; Secure; SameSite=Lax` (or `Strict` for high-security apps). The `__Host-` prefix provides the strongest protection.

**Beginner-Friendly Explanation:** Cookies are like name tags the browser wears. Secure cookies are name tags with special protections: `HttpOnly` means JavaScript can't read them (protecting against XSS); `Secure` means they're only shown over HTTPS (protecting against sniffing); `SameSite` means they're only shown when visiting from the same site (protecting against CSRF). The `__Host-` prefix is like a tamper-proof seal.

### Purposes

- To protect session cookies from XSS (HttpOnly).
- To protect session cookies from network sniffing (Secure).
- To protect session cookies from CSRF (SameSite).
- To prevent subdomain attacks (`__Host-` prefix).
- To control cookie lifetime and scope.

### Syntax Rules and Structure

#### Cookie Attributes

| Attribute | Purpose | Recommended Value |
|-----------|---------|-------------------|
| `HttpOnly` | Block JavaScript access | Always set |
| `Secure` | HTTPS only | Always set (production) |
| `SameSite` | CSRF mitigation | `Lax` or `Strict` |
| `Domain` | Cookie scope | Omit (host-only) |
| `Path` | Cookie scope | `/` |
| `Max-Age` | Lifetime | 15 min – 24 hours |
| `__Host-` prefix | Strongest protection | Use for session cookies |
| `Partitioned` | CHIPS | For embedded contexts |

#### Express Cookie Configuration

```typescript
res.cookie('__Host-sid', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
  path: '/',
  // domain: undefined  // Omit for host-only
});
```

#### Session Cookie Configuration

```typescript
import session from 'express-session';
import RedisStore from 'connect-redis';

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  name: '__Host-sid',
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 1000 * 60 * 60 * 24,
    path: '/',
  },
  rolling: true,
}));
```

#### Syntax Rules

- **Always set `HttpOnly`** — prevents XSS cookie theft.
- **Always set `Secure`** in production — prevents network sniffing.
- **Always set `SameSite=Lax` or `Strict`** — prevents CSRF.
- **Use the `__Host-` prefix** — strongest protection.
- **Omit `Domain`** — host-only cookies are more secure.
- **Set `Path=/`** — required for `__Host-` prefix.
- **Set a reasonable `Max-Age`** — do not use session cookies for auth.
- **Never store sensitive data** — only the session ID.
- **Use `SameSite=None; Secure`** only for cross-site.
- **Test cookie flags** — verify in browser DevTools.

#### Constraints and Limitations

- **`SameSite=None` requires `Secure`** — browsers reject otherwise.
- **`SameSite=Lax` does not protect state-changing GET** — use `Strict` or CSRF tokens.
- **Cookie size limit** — ~4 KB per cookie.
- **Cookie count limit** — ~50 per domain.
- **Third-party cookies are blocked** — use CHIPS (`Partitioned`).
- **`__Host-` prefix requires exact conditions** — `Secure`, no `Domain`, `Path=/`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Secure Cookie Configuration (Express + Session)

```typescript
import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import cookieParser from 'cookie-parser';
import { createClient } from 'redis';
import { randomBytes } from 'node:crypto';

const app = express();
app.use(express.json());
app.use(cookieParser());

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

// Session cookie (strongest protection)
app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
  name: '__Host-sid', // __Host- prefix
  genid: () => randomBytes(32).toString('base64url'),
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    maxAge: 1000 * 60 * 60 * 24,
    path: '/',
    // domain: undefined — omitted for __Host-
  },
  rolling: true,
}));

// CSRF cookie (double-submit)
app.use((req, res, next) => {
  if (!req.cookies['__Host-csrf']) {
    const token = randomBytes(32).toString('base64url');
    res.cookie('__Host-csrf', token, {
      httpOnly: true,
      secure: true,
      sameSite: 'lax',
      path: '/',
      maxAge: 1000 * 60 * 60 * 24,
    });
  }
  next();
});

// Non-sensitive preference cookie (readable by JS)
app.post('/preferences', (req, res) => {
  res.cookie('theme', req.body.theme, {
    httpOnly: false,        // JS needs to read this
    secure: true,
    sameSite: 'lax',
    path: '/',
    maxAge: 1000 * 60 * 60 * 24 * 365,
  });
  res.json({ success: true });
});

// Clear cookie on logout
app.post('/auth/logout', (req, res) => {
  req.session.destroy(() => {
    res.clearCookie('__Host-sid', { path: '/' });
    res.status(204).end();
  });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected behaviour:**
- Session cookies use `__Host-sid` with `HttpOnly; Secure; SameSite=Lax`.
- CSRF cookies use `__Host-csrf` with the same flags.
- Preference cookies are readable by JavaScript (`HttpOnly: false`).
- Logout clears the session cookie.

**Why this works:** Different cookies have different security requirements. Session and CSRF cookies use the strongest flags. Preference cookies are readable by JavaScript but still `Secure` and `SameSite`.

### Real-World Cases

- **Banking:** `__Host-` prefix, `Strict` SameSite, short `Max-Age`.
- **SaaS:** `__Host-` prefix, `Lax` SameSite, rolling expiration.
- **E-commerce:** `__Host-` prefix, `Lax` SameSite, persistent cart cookies.
- **Embedded widgets:** `SameSite=None; Secure; Partitioned`.

---

## Core Concept 8: Secret Management

### Definitions

**Core Definition:** Secret management is the practice of securely storing, accessing, rotating, and auditing sensitive credentials (API keys, database passwords, tokens) using dedicated tools and processes.

**Technical Definition:** Secret management encompasses: **storage** (encrypted vaults, HSMs, cloud secret managers), **access** (least-privilege IAM, short-lived credentials), **rotation** (automated rotation, versioning), **audit** (access logs, anomaly detection), and **injection** (environment variables, mounted files, SDKs). Tools include HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, Doppler, and `.env.vault` (from dotenv). Secrets must never be committed to version control, logged, or included in client-side bundles. `.env` files are acceptable for local development but must be `.gitignore`d. Production secrets should come from a vault or cloud provider.

**Beginner-Friendly Explanation:** Secret management is like a bank vault for your passwords. Instead of writing passwords on sticky notes (`.env` files) or leaving them in your desk drawer (source code), you store them in a vault that requires authentication to access. The vault logs every access, rotates passwords automatically, and only gives each service the secrets it needs. Even if an attacker compromises your application, they cannot easily extract the secrets.

### Purposes

- To prevent secrets from leaking into version control, logs, or client bundles.
- To enforce least-privilege access to secrets.
- To enable automated secret rotation.
- To provide audit trails for compliance.
- To support multiple environments (dev, staging, production).
- To enable dynamic, short-lived credentials.

### Syntax Rules and Structure

#### Secret Management Comparison

| Tool | Type | Best For |
|------|------|----------|
| **HashiCorp Vault** | Self-hosted / Cloud | Full control, dynamic secrets |
| **AWS Secrets Manager** | Managed (AWS) | AWS-native apps |
| **Azure Key Vault** | Managed (Azure) | Azure-native apps |
| **GCP Secret Manager** | Managed (GCP) | GCP-native apps |
| **Doppler** | Managed | Multi-cloud, developer-friendly |
| **`.env.vault`** | File-based | Simple deployments |

#### .gitignore Configuration

```gitignore
# .gitignore
.env
.env.local
.env.*.local
.env.vault
*.pem
*.key
*.p12
secrets/
config/secrets.json
```

#### dotenv-vault (Simple Approach)

```bash
# Install
npm install dotenv-vault

# Login
npx dotenv-vault login

# Push secrets to the vault
npx dotenv-vault push

# Pull secrets
npx dotenv-vault pull

# Build .env.vault
npx dotenv-vault build
```

```typescript
// config.ts — load from .env.vault
import 'dotenv-vault/config';

// Secrets are now in process.env
const config = {
  databaseUrl: process.env.DATABASE_URL!,
  jwtSecret: process.env.JWT_SECRET!,
  stripeKey: process.env.STRIPE_SECRET_KEY!,
};
```

#### AWS Secrets Manager (Production Approach)

```typescript
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'us-east-1' });

async function getSecret(secretId: string): Promise<Record<string, string>> {
  const response = await client.send(new GetSecretValueCommand({ SecretId: secretId }));

  if (response.SecretString) {
    return JSON.parse(response.SecretString);
  }

  if (response.SecretBinary) {
    return JSON.parse(Buffer.from(response.SecretBinary).toString('utf8'));
  }

  throw new Error('Secret not found');
}

// Load at startup
const secrets = await getSecret('myapp/production');

export const config = {
  databaseUrl: secrets.DATABASE_URL,
  jwtSecret: secrets.JWT_SECRET,
  stripeKey: secrets.STRIPE_SECRET_KEY,
};
```

#### Syntax Rules

- **Never commit secrets** — add `.env` to `.gitignore`.
- **Use a vault for production** — AWS Secrets Manager, Vault, etc.
- **Use `.env` for local development only.**
- **Use `.env.example`** — with placeholder values.
- **Rotate secrets regularly** — and immediately after exposure.
- **Use short-lived credentials** — dynamic secrets (Vault).
- **Enforce least privilege** — each service gets only its secrets.
- **Audit secret access** — log every read.
- **Redact secrets in logs** — use a redacting logger.
- **Use secret scanning** — TruffleHog, Gitleaks, GitHub Secret Scanning.
- **Encrypt secrets at rest** — vaults do this automatically.
- **Inject secrets at runtime** — environment variables or mounted files.

#### Constraints and Limitations

- **Vaults add operational complexity** — but are essential for production.
- **Network dependency** — if the vault is down, the app may fail to start.
- **Rotation requires coordination** — services must reload secrets.
- **Environment variables are visible** to `ps` and child processes.
- **Secrets in memory** can be dumped if the process is compromised.
- **Cost** — managed vaults charge per secret and per API call.
- **Local development** — `.env` files are still common but risky.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Production Secret Management (AWS Secrets Manager + Node.js)

```typescript
// config/secrets.ts
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import { z } from 'zod';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION });

const SecretsSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  REDIS_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  SMTP_PASSWORD: z.string().min(8),
});

type Secrets = z.infer<typeof SecretsSchema>;

let cachedSecrets: Secrets | null = null;
let cacheExpiry = 0;
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

export async function getSecrets(): Promise<Secrets> {
  // Return cached secrets if still valid
  if (cachedSecrets && Date.now() < cacheExpiry) {
    return cachedSecrets;
  }

  const response = await client.send(
    new GetSecretValueCommand({ SecretId: `myapp/${process.env.NODE_ENV}` }),
  );

  const raw = JSON.parse(response.SecretString!);
  const result = SecretsSchema.safeParse(raw);

  if (!result.success) {
    throw new Error(`Invalid secrets: ${result.error.issues.map((i) => i.message).join(', ')}`);
  }

  cachedSecrets = result.data;
  cacheExpiry = Date.now() + CACHE_TTL;

  return cachedSecrets;
}

// Force refresh (called after rotation)
export function invalidateSecretsCache(): void {
  cachedSecrets = null;
  cacheExpiry = 0;
}
```

```typescript
// main.ts — bootstrap with secrets
import { getSecrets } from './config/secrets';
import { PrismaClient } from '@prisma/client';
import Redis from 'ioredis';

async function bootstrap() {
  // Load secrets before starting the app
  const secrets = await getSecrets();

  const prisma = new PrismaClient({
    datasources: { db: { url: secrets.DATABASE_URL } },
  });

  const redis = new Redis(secrets.REDIS_URL);

  const app = await NestFactory.create(AppModule);
  app.use(helmet());
  app.useGlobalPipes(new ValidationPipe());

  await app.listen(3000);
  console.log('Server started with secrets from AWS Secrets Manager');
}

bootstrap().catch((err) => {
  console.error('Failed to start:', err);
  process.exit(1);
});
```

```typescript
// logger.ts — redacting logger
import pino from 'pino';

export const logger = pino({
  redact: {
    paths: [
      'password',
      'token',
      'secret',
      'apiKey',
      'authorization',
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.token',
      '*.secret',
      'DATABASE_URL',
      'JWT_SECRET',
      'STRIPE_SECRET_KEY',
    ],
    censor: '[REDACTED]',
  },
});
```

**Expected behaviour:**
- Secrets are loaded from AWS Secrets Manager at startup.
- Secrets are validated with Zod.
- Secrets are cached for 5 minutes.
- Secrets are redacted from logs.
- Missing or invalid secrets cause the app to exit.

**Why this works:** Secrets are stored in a vault (not in code or environment variables). Access is audited. Rotation is supported (cache invalidation). Logging is redacted. Validation ensures all secrets are present.

### Real-World Cases

- **Banking:** HashiCorp Vault with dynamic secrets and HSM backing.
- **SaaS:** AWS Secrets Manager with automatic rotation.
- **Multi-cloud:** Doppler for unified secret management.
- **Small teams:** `.env.vault` for simple, secure deployments.
- **Enterprise:** Azure Key Vault with managed identities.

---

## References

- OWASP Cheat Sheet Series — Input Validation — https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Cross Site Scripting Prevention — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Denial of Service — https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html
- OWASP Cheat Sheet Series — HTTP Headers — https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Content Security Policy — https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
- OWASP Cheat Sheet Series — CORS — https://cheatsheetseries.owasp.org/cheatsheets/CORS_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Session Management — https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series — Secrets Management — https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- OWASP Application Security Verification Standard (ASVS) — https://owasp.org/www-project-application-security-verification-standard/
- NIST SP 800-53 — Security and Privacy Controls — https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final
- CIS Controls — https://www.cisecurity.org/controls
- MDN Web Docs — HTTP Security Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers
- MDN Web Docs — Content Security Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- MDN Web Docs — Strict-Transport-Security — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
- MDN Web Docs — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- MDN Web Docs — Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- Zod Documentation — https://zod.dev/
- Joi Documentation — https://joi.dev/
- express-validator Documentation — https://express-validator.github.io/docs/
- express-rate-limit — npm package — https://www.npmjs.com/package/express-rate-limit
- express-slow-down — npm package — https://www.npmjs.com/package/express-slow-down
- rate-limit-redis — npm package — https://www.npmjs.com/package/rate-limit-redis
- Helmet Documentation — https://helmetjs.github.io/
- cors — npm package — https://www.npmjs.com/package/cors
- multer — npm package — https://www.npmjs.com/package/multer
- DOMPurify — npm package — https://www.npmjs.com/package/dompurify
- dotenv-vault — npm package — https://www.npmjs.com/package/dotenv-vault
- HashiCorp Vault — https://www.vaultproject.io/
- AWS Secrets Manager — https://aws.amazon.com/secrets-manager/
- Azure Key Vault — https://azure.microsoft.com/en-us/products/key-vault
- GCP Secret Manager — https://cloud.google.com/secret-manager
- Doppler — https://www.doppler.com/
- TruffleHog — https://github.com/trufflesecurity/trufflehog
- Gitleaks — https://github.com/gitleaks/gitleaks
- GitHub Secret Scanning — https://docs.github.com/en/code-security/secret-scanning
- Pino — Redacting Logger — https://getpino.io/#/docs/redaction
- Security Headers — https://securityheaders.com/
- HSTS Preload — https://hstspreload.org/