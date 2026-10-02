# Express.js Security Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Express security middleware refers to a set of configurable middleware packages that sit between the incoming HTTP request and your application's route handlers, applying protective measures such as security headers, origin validation, request throttling, payload restrictions, and input sanitisation.

**Technical Definition:** Express security middleware operates within the Express middleware pipeline — a stack of functions with the signature `(req, res, next)` that execute sequentially for each request. Each middleware package (Helmet, cors, express-rate-limit, express-validator) intercepts requests or responses to enforce a specific security control, either by setting response headers, validating request metadata, limiting request frequency, constraining request body size, or sanitising input data before it reaches application logic.

**Beginner-Friendly Explanation:** Think of your Express application as a building with a single entrance. Security middleware are the security guards stationed at that entrance. One guard checks IDs (CORS), another makes sure visitors don't carry in dangerous packages (Helmet headers), another limits how often each person can enter (rate limiting), another checks that packages aren't too heavy (body limits), and another inspects and cleans everything that comes through the door (sanitisation). Each guard handles one specific job, and together they form a layered defence.

### Key Characteristics

- **Pipeline-based:** Middleware executes in the order it is registered; ordering is critical for security.
- **Composable:** Multiple middleware functions can be combined for defence-in-depth.
- **Configurable:** Each package offers extensive configuration options to match application requirements.
- **Framework-integrated:** Designed specifically for Express and Connect-style applications.
- **Layered defence:** No single middleware provides complete protection; each addresses a specific attack surface.

### Prerequisites

- **Node.js runtime** (v18 or higher recommended).
- **Express.js installed** (`npm install express`).
- **Basic understanding of Express middleware:** how `app.use()` and route handlers work.
- **Familiarity with HTTP headers:** request and response headers, status codes.
- **Understanding of web security concepts:** CORS, CSP, rate limiting, input validation.

### Related Programming Areas

- **HTTP security headers:** CSP, HSTS, X-Frame-Options, X-Content-Type-Options.
- **Web application firewalls (WAF):** Middleware-based request filtering.
- **API gateway security:** Rate limiting, authentication, and throttling at the edge.
- **Input validation and sanitisation:** Preventing injection and XSS attacks.
- **DevOps and deployment:** Reverse proxy configuration, trust proxy settings.

### Core Concepts

1. **Helmet** — configuring standard security headers, customising CSP, and understanding coverage gaps.
2. **CORS** — strict origin whitelisting, dynamic origins, and preflight OPTIONS handling.
3. **Rate Limiting & Throttling** — IP-based and user-based limits, Redis-backed distributed limiters, and `trust proxy` configuration.
4. **Request-Size & Body Limits** — payload constraints on `express.json()` and `express.urlencoded()`, slowloris and multipart attack mitigation.
5. **Data Sanitisation Middleware** — using `express-validator` for sanitising and casting raw input data before processing.

---

## Core Concept 1: Helmet

### Definitions

**Core Definition:** Helmet is an Express middleware that sets HTTP response headers to protect applications from well-known web vulnerabilities by configuring headers such as `Content-Security-Policy`, `Strict-Transport-Security`, and `X-Content-Type-Options`.

**Technical Definition:** Helmet is a collection of smaller middleware functions, each responsible for setting one or more HTTP response headers. When applied via `app.use(helmet())`, it sets 13 HTTP response headers by default, including `Content-Security-Policy`, `Cross-Origin-Opener-Policy`, `Cross-Origin-Resource-Policy`, `Origin-Agent-Cluster`, `Referrer-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `X-DNS-Prefetch-Control`, `X-Download-Options`, `X-Frame-Options`, `X-Permitted-Cross-Domain-Policies`, and `X-XSS-Protection`. Each header can be individually disabled or configured.

**Beginner-Friendly Explanation:** Helmet is like a set of automatic safety stickers that your server attaches to every response it sends. These stickers tell the browser "don't run scripts from untrusted sources," "only connect over HTTPS," and "don't let this page be embedded in another site." The browser reads these stickers and enforces the rules, protecting your users from common attacks.

### Purposes

- To set security-related HTTP response headers that instruct browsers to enforce protective policies.
- To mitigate cross-site scripting (XSS), clickjacking, MIME sniffing, and protocol downgrade attacks.
- To provide a baseline of security headers with minimal configuration effort.
- To allow granular customisation of Content Security Policy for applications with specific resource requirements.

### Sub-Feature 1.1: Standard Security Headers

#### Definitions

**Core Definition:** Standard security headers are the 13 HTTP response headers set by Helmet's default configuration, each addressing a specific class of browser-based attack.

**Technical Definition:** Helmet's default configuration sets `Content-Security-Policy`, `Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Resource-Policy: same-origin`, `Origin-Agent-Cluster: ?1`, `Referrer-Policy: no-referrer`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`, `X-Content-Type-Options: nosniff`, `X-DNS-Prefetch-Control: off`, `X-Download-Options: noopen`, `X-Frame-Options: SAMEORIGIN`, `X-Permitted-Cross-Domain-Policies: none`, and `X-XSS-Protection: 0`.

**Beginner-Friendly Explanation:** Each header is like a different safety rule. One says "only load resources from my own site" (CSP), another says "don't let anyone embed my page in a frame" (X-Frame-Options), and another says "only use HTTPS" (HSTS). Helmet applies all of them at once.

#### Purposes

- To provide immediate protection against common browser-based attacks without custom configuration.
- To establish a security baseline for all responses.
- To reduce the attack surface available to client-side exploits.

#### Syntax Rules and Structure

```js
import helmet from 'helmet';
app.use(helmet());
```

| Component | Breakdown |
|-----------|-----------|
| `helmet()` | Middleware factory; returns middleware with default headers. |
| `app.use()` | Registers middleware for all routes. |
| Default | Sets 13 security headers. |

**Constraints and Limitations:**
- Helmet's CSP default may break applications that use inline scripts or load resources from CDNs; CSP must be tuned for each application.
- Helmet performs very little validation on your CSP; rely on CSP checkers like CSP Evaluator.
- Helmet does not protect against all vulnerabilities — it addresses browser-side concerns, not server-side injection, authentication, or authorisation.

#### Annotated Code Example

```js
// helmet-basic.js
import express from 'express';
import helmet from 'helmet';

const app = express();

// Apply Helmet with all default headers
app.use(helmet());

app.get('/', (req, res) => {
  res.send('Hello, secure world!');
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Response Headers (for `GET /`):**
```
Content-Security-Policy: default-src 'self';base-uri 'self';font-src 'self' https: data:;form-action 'self';frame-ancestors 'self';img-src 'self' data:;object-src 'none';script-src 'self';script-src-attr 'none';style-src 'self' https: 'unsafe-inline';upgrade-insecure-requests
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 0
```

**Why this output:** Helmet's default configuration sets these headers on every response. The CSP restricts resources to the same origin, `X-Frame-Options` prevents clickjacking, `X-Content-Type-Options` prevents MIME sniffing, and `Strict-Transport-Security` enforces HTTPS for future requests.

#### Real-World Cases

- **Any Express application:** Helmet is the highest-value, lowest-effort security control and should be applied to every production application.
- **API servers:** Helmet's headers protect API responses even when no HTML is served.
- **Single-page applications:** CSP configuration must be tuned to allow the SPA's resource loading patterns.

---

### Sub-Feature 1.2: Customising Content Security Policy

#### Definitions

**Core Definition:** Content Security Policy (CSP) is a security header that instructs the browser on which sources of scripts, styles, images, and other resources are permitted to load on a page.

**Technical Definition:** The `Content-Security-Policy` header uses a directive-based syntax (e.g., `script-src 'self' https://cdn.example.com`). Helmet allows configuration through a `directives` object where keys are directive names in camelCase or kebab-case, and values are arrays of permitted sources. Directives can be set to `null` to disable them. A nonce (number used once) can be generated per request and included in the `script-src` directive for inline scripts.

**Beginner-Friendly Explanation:** CSP is like a guest list for your website's resources. You tell the browser exactly which scripts, styles, and images are allowed to load. Anything not on the list is blocked. This prevents attackers from injecting malicious scripts because the browser refuses to run them.

#### Purposes

- To mitigate cross-site scripting (XSS) by restricting the sources of executable scripts.
- To prevent data exfiltration by restricting where the page can send data (`connect-src`).
- To control which external resources (fonts, images, frames) can be loaded.
- To enforce HTTPS for all resource loads via `upgrade-insecure-requests`.

#### Syntax Rules and Structure

```js
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "example.com"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "cdn.example.com"],
      connectSrc: ["'self'", "api.example.com"],
      objectSrc: ["'none'"]
    }
  }
}));
```

| Component | Breakdown |
|-----------|-----------|
| `contentSecurityPolicy` | Helmet option for CSP configuration. |
| `directives` | Object mapping directive names to arrays of sources. |
| `useDefaults` | Set to `false` to disable merging with defaults. |
| Nonce | Function `(req, res) => \`'nonce-${res.locals.cspNonce}'\`` for inline scripts. |

**Constraints and Limitations:**
- CSP is complex; misconfiguration can break legitimate functionality.
- `'unsafe-inline'` for `style-src` weakens protection but is often necessary for CSS-in-JS.
- Nonces must be unique per request and unpredictable.

#### Annotated Code Example

```js
// helmet-csp.js — Custom CSP with nonce
import express from 'express';
import helmet from 'helmet';
import crypto from 'node:crypto';

const app = express();

// Generate a nonce per request
app.use((req, res, next) => {
  res.locals.cspNonce = crypto.randomBytes(32).toString('hex');
  next();
});

app.use(helmet({
  contentSecurityPolicy: {
    useDefaults: true,
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        (req, res) => `'nonce-${res.locals.cspNonce}'`
      ],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
      objectSrc: ["'none'"]
    }
  }
}));

app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <script nonce="${res.locals.cspNonce}">
          console.log('This inline script is allowed via nonce.');
        </script>
      </head>
      <body><h1>Hello, CSP!</h1></body>
    </html>
  `);
});

app.listen(3000, () => console.log('CSP server on 3000'));
```

**Expected Response Header:**
```
Content-Security-Policy: default-src 'self';script-src 'self' 'nonce-a1b2c3d4...';style-src 'self' 'unsafe-inline';img-src 'self' data: https:;connect-src 'self' https://api.example.com;object-src 'none';upgrade-insecure-requests
```

**Why this output:** The `scriptSrc` directive includes `'self'` (same-origin scripts) and a per-request nonce. The browser allows inline scripts only if their nonce attribute matches the header value. Injected scripts without the correct nonce are blocked.

#### Real-World Cases

- **Single-page applications:** CSP with nonces for inline hydration scripts.
- **E-commerce sites:** Restricting `connect-src` to payment gateway APIs.
- **Content-heavy sites:** Allowlisting CDN domains for images and fonts.

---

### Sub-Feature 1.3: What Helmet Doesn't Cover

#### Definitions

**Core Definition:** Helmet addresses browser-side security headers but does not provide protection against server-side vulnerabilities such as injection, authentication flaws, or authorisation bypasses.

**Technical Definition:** Helmet's scope is limited to HTTP response headers that control browser behaviour. It does not validate input, sanitise output, manage sessions, enforce authentication, prevent CSRF, mitigate SQL injection, or provide rate limiting. These must be handled by other middleware or application logic.

**Beginner-Friendly Explanation:** Helmet is like a seatbelt — it protects you in a crash but doesn't prevent the crash. You still need brakes, airbags, and careful driving (other security measures) to be fully protected.

#### Purposes

- To clarify the boundaries of Helmet's protection so developers do not rely on it alone.
- To encourage complementary security measures (rate limiting, validation, CSRF protection).
- To prevent a false sense of security from applying Helmet without other controls.

#### Syntax Rules and Structure

Helmet does not have syntax for what it doesn't do; instead, developers must add complementary middleware.

**Constraints and Limitations:**
- Helmet does not prevent SQL/NoSQL injection, XSS through unsanitised input, CSRF, brute force, or DoS.
- Helmet does not manage sessions, cookies, or authentication.
- Helmet's CSP requires tuning and can break applications if applied blindly.

#### Annotated Code Example

```js
// helmet-gaps.js — Helmet plus complementary middleware
import express from 'express';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import { body, validationResult } from 'express-validator';

const app = express();
app.use(express.json());

// ✅ Helmet: security headers
app.use(helmet());

// ✅ Rate limiting: brute force and DoS protection
app.use(rateLimit({ windowMs: 60000, max: 100 }));

// ✅ Input validation: injection and XSS prevention
app.post('/login', [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).trim().escape()
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  res.json({ message: 'Validated' });
});

app.listen(3000, () => console.log('Layered security on 3000'));
```

**Why this output:** Helmet sets headers but does not validate input, limit requests, or sanitise data. The complementary middleware fills those gaps.

#### Real-World Cases

- **Security audits:** Auditors check that Helmet is present but also look for input validation, rate limiting, and session security.
- **Penetration testing:** Testers verify that CSP does not replace server-side validation.
- **Compliance:** Standards require multiple layers of defence, not just headers.

---

## Core Concept 2: CORS

### Definitions

**Core Definition:** Cross-Origin Resource Sharing (CORS) is a browser security mechanism that restricts which origins can read responses from a server, implemented in Express through the `cors` middleware package.

**Technical Definition:** The `cors` middleware sets `Access-Control-Allow-Origin` and related response headers that instruct browsers on which cross-origin requests are permitted. It supports static origin strings, arrays of origins, and dynamic origin validation functions. Preflight requests (OPTIONS) are handled automatically when the middleware is applied at the application level.

**Beginner-Friendly Explanation:** CORS is like a club bouncer who checks your ID against a guest list. When your browser tries to fetch data from another website, the browser asks that website's server "Is this website allowed to read my response?" The server's CORS headers answer yes or no. If no, the browser blocks the response.

### Purposes

- To control which origins are permitted to read responses from your API.
- To prevent malicious websites from reading sensitive data on behalf of authenticated users.
- To handle preflight OPTIONS requests for complex cross-origin requests.
- To support dynamic origin validation based on databases or configuration.

### Sub-Feature 2.1: Strict Origin Whitelisting

#### Definitions

**Core Definition:** Origin whitelisting is the practice of specifying an explicit list of trusted origins that are permitted to make cross-origin requests to the server.

**Technical Definition:** The `origin` option of the `cors` middleware accepts a string (single origin), an array of strings (multiple origins), a regular expression, or a boolean. When set to an array, the middleware checks the request's `Origin` header against the list and reflects it in the `Access-Control-Allow-Origin` header if permitted.

**Beginner-Friendly Explanation:** Instead of letting everyone into the club, you keep a written guest list. Only people whose names are on the list get in.

#### Purposes

- To restrict API access to known, trusted frontend applications.
- To prevent unauthorised websites from reading API responses.
- To comply with security policies that require explicit allowlisting.

#### Syntax Rules and Structure

```js
const cors = require('cors');
const whitelist = ['https://app.example.com', 'https://admin.example.com'];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || whitelist.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

| Component | Breakdown |
|-----------|-----------|
| `origin` | Function `(origin, callback) => callback(err, allowed)`. |
| `credentials` | Allow cookies and authentication headers. |
| `methods` | Allowed HTTP methods. |
| `allowedHeaders` | Allowed request headers. |

**Constraints and Limitations:**
- CORS does not block requests; it only controls whether browsers allow JavaScript to read responses. Non-browser clients (curl, Postman) ignore CORS entirely.
- `credentials: true` cannot be used with `origin: '*'`.
- Origins are case-sensitive and include the protocol and port.

#### Annotated Code Example

```js
// cors-whitelist.js
const express = require('express');
const cors = require('cors');
const app = express();

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com'
];

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (e.g., Postman, server-to-server)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);        // Origin allowed
    } else {
      callback(new Error('CORS: Origin not allowed'));  // Origin blocked
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

app.get('/api/data', (req, res) => {
  res.json({ message: 'CORS-enabled data' });
});

app.listen(3000, () => console.log('CORS server on 3000'));
```

**Expected Output (for `GET /api/data` from `https://app.example.com`):**
```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
{"message":"CORS-enabled data"}
```

**Expected Output (for `GET /api/data` from `https://evil.com`):**
```
No Access-Control-Allow-Origin header present; browser blocks the response.
```

**Why this output:** The origin validation function checks the `Origin` header against the whitelist. If the origin is allowed, the header is set; if not, the callback returns an error and no CORS headers are set, causing the browser to block the response.

#### Real-World Cases

- **SaaS APIs:** Restricting access to the company's own frontend applications.
- **Multi-tenant platforms:** Allowing each tenant's custom domain via a database-backed whitelist.
- **Microservices:** Allowing internal service origins while blocking external ones.

---

### Sub-Feature 2.2: Dynamic Origins and Preflight Handling

#### Definitions

**Core Definition:** Dynamic origin validation allows the CORS policy to be determined at runtime based on the request's origin, enabling database-driven or environment-aware configurations. Preflight handling addresses the browser's automatic OPTIONS request sent before "complex" cross-origin requests.

**Technical Definition:** The `cors` middleware supports a function passed to the `origin` option, which is called for each request with the origin string and a callback. This function can load allowed origins from a database, environment variables, or any other data source. Preflight requests (OPTIONS) are automatically handled when `cors()` is applied at the application level.

**Beginner-Friendly Explanation:** Dynamic origins are like a bouncer who checks a real-time database of approved guests rather than a printed list. Preflight handling is the bouncer's routine of asking "What do you want to do?" before letting someone in — for complex requests, the browser sends an OPTIONS request first to check permissions.

#### Purposes

- To support multi-tenant applications where allowed origins vary per tenant.
- To load CORS policies from environment configuration or databases.
- To automatically handle preflight OPTIONS requests for all routes.
- To customise CORS settings dynamically per request.

#### Syntax Rules and Structure

```js
// Dynamic origin from database
app.use(cors({
  origin: async (origin, callback) => {
    if (!origin) return callback(null, true);
    const allowed = await db.isOriginAllowed(origin);
    callback(null, allowed);
  },
  credentials: true
}));

// Preflight across all routes
app.options('*', cors());  // Express 4
app.options('/*splat', cors()); // Express 5
```

| Component | Breakdown |
|-----------|-----------|
| `origin` function | `(origin, callback) => callback(error, allowed)`. |
| `app.options('*', cors())` | Handles preflight for all routes. |
| Application-level `app.use(cors())` | Preflight handled automatically. |

**Constraints and Limitations:**
- In Express 5.x, wildcard routes must be named (e.g., `/*splat`), not `'*'`.
- Dynamic origin functions must call the callback asynchronously if they perform async operations.

#### Annotated Code Example

```js
// cors-dynamic.js — Dynamic origin + preflight
const express = require('express');
const cors = require('cors');
const app = express();

// Simulated database of allowed origins
const allowedOriginsDB = new Map([
  ['tenant1.example.com', 'https://tenant1.example.com'],
  ['tenant2.example.com', 'https://tenant2.example.com']
]);

const corsOptions = {
  origin: (origin, callback) => {
    if (!origin) return callback(null, true);

    // Check if origin is in the dynamic database
    const isAllowed = [...allowedOriginsDB.values()].includes(origin);
    callback(null, isAllowed);
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
};

// Apply CORS to all routes (preflight handled automatically)
app.use(cors(corsOptions));

// Explicit preflight handler for custom headers (optional)
app.options('/api/*', cors(corsOptions));

app.get('/api/tenant-data', (req, res) => {
  res.json({ tenant: 'data' });
});

app.listen(3000, () => console.log('Dynamic CORS on 3000'));
```

**Expected Output (for preflight `OPTIONS /api/tenant-data` from `https://tenant1.example.com`):**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://tenant1.example.com
Access-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
Access-Control-Allow-Headers: Content-Type,Authorization
Access-Control-Allow-Credentials: true
```

**Why this output:** The browser sends an OPTIONS request before the actual GET request because the request includes custom headers (`Authorization`). The CORS middleware automatically responds with the allowed methods and headers. The origin function validates `https://tenant1.example.com` against the database and allows it.

#### Real-World Cases

- **Multi-tenant SaaS:** Each tenant has a custom domain; allowed origins are loaded from the database.
- **Development vs. production:** Different allowed origins based on `NODE_ENV`.
- **API gateways:** Dynamic origin validation for partner integrations.

---

## Core Concept 3: Rate Limiting & Throttling

### Definitions

**Core Definition:** Rate limiting restricts the number of requests a client can make within a defined time window, protecting against brute force attacks, credential stuffing, and denial-of-service (DoS) attacks.

**Technical Definition:** The `express-rate-limit` middleware tracks request counts per key (typically IP address or authenticated user ID) within a sliding or fixed time window. When the count exceeds the configured maximum, subsequent requests receive a 429 (Too Many Requests) response. For distributed deployments behind load balancers, a Redis-backed store synchronises rate limit counters across multiple server instances.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer who lets in only 10 people per minute. If an attacker tries to send 100 login attempts in a minute, the bouncer blocks them after the 10th attempt, slowing down the attack to a manageable pace.

### Purposes

- To prevent brute force attacks on login, registration, and password reset endpoints.
- To mitigate credential stuffing campaigns that try leaked credentials across many accounts.
- To protect server resources from exhaustion by high-volume request floods.
- To distribute rate limit state across multiple server instances using Redis.

### Sub-Feature 3.1: IP-Based and User-Based Limits

#### Definitions

**Core Definition:** IP-based rate limiting counts requests per client IP address, while user-based rate limiting counts requests per authenticated user account.

**Technical Definition:** `express-rate-limit` defaults to `req.ip` as the key. A custom `keyGenerator` function can return `req.user.id` for authenticated requests, enabling per-user limits that are more resistant to IP rotation. Combining both layers (per-IP and per-user) provides defence against both distributed and targeted attacks.

**Beginner-Friendly Explanation:** IP-based limits are like limiting how many people can enter from the same street address. User-based limits are like limiting how many times each individual person can enter, regardless of which door they use.

#### Purposes

- To prevent a single IP address from overwhelming the server.
- To prevent a single user account from being targeted by brute force.
- To combine IP and user limits for comprehensive coverage.

#### Syntax Rules and Structure

```js
const rateLimit = require('express-rate-limit');

// IP-based limiter
const ipLimiter = rateLimit({
  windowMs: 60 * 1000,       // 1 minute
  max: 100,                   // 100 requests per IP per minute
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many requests from this IP' }
});

// User-based limiter (requires authentication middleware first)
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  keyGenerator: (req) => req.user?.id || req.ip,
  message: { error: 'Too many requests from this user' }
});
```

| Component | Breakdown |
|-----------|-----------|
| `windowMs` | Time window in milliseconds. |
| `max` | Maximum requests per window. |
| `keyGenerator` | Function returning the rate-limit key. |
| `standardHeaders` | Includes `RateLimit-*` headers. |

**Constraints and Limitations:**
- IP-based limits can block legitimate users behind NAT or shared proxies.
- User-based limits require authentication middleware to run first.
- In-memory stores do not work across multiple server processes.

#### Annotated Code Example

```js
// rate-limit-layers.js — Multi-layer rate limiting
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();
app.use(express.json());

// Layer 1: Global IP limiter
const globalLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 200,
  message: { error: 'Global rate limit exceeded' }
});

// Layer 2: Login-specific limiter (stricter)
const loginLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { error: 'Too many login attempts' }
});

// Layer 3: Per-account limiter
const accountAttempts = new Map();
function accountLimiter(req, res, next) {
  const email = req.body.email;
  if (!email) return next();

  const now = Date.now();
  const record = accountAttempts.get(email) || { count: 0, resetAt: now + 60000 };

  if (now > record.resetAt) {
    record.count = 0;
    record.resetAt = now + 60000;
  }

  record.count++;
  accountAttempts.set(email, record);

  if (record.count > 5) {
    return res.status(429).json({ error: 'Account temporarily locked' });
  }
  next();
}

app.use(globalLimiter);
app.post('/login', loginLimiter, accountLimiter, (req, res) => {
  res.json({ message: 'Login endpoint' });
});

app.listen(3000, () => console.log('Layered rate limiting on 3000'));
```

**Expected Output (for 6 rapid `POST /login` requests with the same email):**
```
{"error":"Account temporarily locked"}
```

**Why this output:** The per-account limiter increments a counter for the email. After 5 attempts within the 60-second window, the 6th request is rejected with a 429 status.

#### Real-World Cases

- **Login endpoints:** Per-IP and per-account limits prevent both distributed and targeted brute force.
- **Password reset:** Limiting reset requests to prevent email flooding and account enumeration.
- **Public APIs:** Per-IP limits protect against scraping and abuse.

---

### Sub-Feature 3.2: Distributed Rate Limiting with Redis

#### Definitions

**Core Definition:** Distributed rate limiting uses a shared data store (Redis) to synchronise rate limit counters across multiple server instances, ensuring consistent limits regardless of which server handles a request.

**Technical Definition:** The `rate-limit-redis` package provides a Redis-backed store for `express-rate-limit`. Each rate limit increment is executed as an atomic Redis command, ensuring that counters are accurate across all instances. A `prefix` option namespaces the keys for different limiters.

**Beginner-Friendly Explanation:** If you have multiple servers, each one needs to know how many requests a user has made to any of the other servers. Redis is like a shared notebook that all servers write to and read from, so they all agree on the request count.

#### Purposes

- To ensure consistent rate limiting across horizontally scaled deployments.
- To prevent attackers from bypassing limits by hitting different server instances.
- To centralise rate limit state for monitoring and analytics.

#### Syntax Rules and Structure

```js
const rateLimit = require('express-rate-limit');
const { RedisStore } = require('rate-limit-redis');
const { createClient } = require('redis');

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

const limiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args) => redisClient.sendCommand(args),
    prefix: 'rl:'
  }),
  windowMs: 60 * 1000,
  max: 100
});
```

| Component | Breakdown |
|-----------|-----------|
| `RedisStore` | Redis-backed store for rate limit data. |
| `sendCommand` | Function to send commands to Redis. |
| `prefix` | Key namespace to avoid collisions. |
| `windowMs` | Time window for the limit. |

**Constraints and Limitations:**
- Redis introduces a network dependency; handle connection failures gracefully.
- Redis must be secured (authentication, TLS) to prevent tampering.
- Key expiration must be managed to avoid memory bloat.

#### Annotated Code Example

```js
// rate-limit-redis.js — Distributed rate limiting
const express = require('express');
const rateLimit = require('express-rate-limit');
const { RedisStore } = require('rate-limit-redis');
const { createClient } = require('redis');

const app = express();
app.use(express.json());

(async () => {
  const redisClient = createClient({ url: 'redis://localhost:6379' });
  await redisClient.connect();

  const limiter = rateLimit({
    store: new RedisStore({
      sendCommand: (...args) => redisClient.sendCommand(args),
      prefix: 'rl:api:'
    }),
    windowMs: 60 * 1000,
    max: 100,
    standardHeaders: true,
    message: { error: 'Rate limit exceeded (distributed)' }
  });

  app.use('/api', limiter);

  app.get('/api/data', (req, res) => {
    res.json({ message: 'Distributed rate-limited data' });
  });

  app.listen(3000, () => console.log('Redis rate limiter on 3000'));
})();
```

**Expected Output (for 101 requests in a minute):**
```
{"error":"Rate limit exceeded (distributed)"}
```

**Why this output:** Each request increments a counter in Redis. When the counter exceeds 100 within the 60-second window, subsequent requests are rejected. Because Redis is shared, all server instances enforce the same limit.

#### Real-World Cases

- **Horizontally scaled APIs:** Multiple Node.js instances behind a load balancer.
- **Microservices:** Shared rate limit state across services.
- **Serverless deployments:** Redis-backed limits for stateless function instances.

---

### Sub-Feature 3.3: Trust Proxy Configuration

#### Definitions

**Core Definition:** The `trust proxy` setting tells Express to trust the `X-Forwarded-For` header set by reverse proxies, so that `req.ip` reflects the real client IP address rather than the proxy's IP.

**Technical Definition:** When an Express application runs behind a reverse proxy (Nginx, Heroku, AWS ELB, Cloudflare), the TCP connection originates from the proxy. Without `app.set('trust proxy', 1)`, `req.ip` returns the proxy's IP address, causing rate limiting to treat all users as a single client. The `trust proxy` setting accepts a number (number of proxies to trust), a boolean, or a function.

**Beginner-Friendly Explanation:** If you're behind a receptionist (proxy), the receptionist's phone number appears on caller ID instead of the actual caller's number. The `trust proxy` setting tells Express to look at the forwarding information the receptionist provides to find the real caller's number.

#### Purposes

- To ensure `req.ip` returns the real client IP address behind a reverse proxy.
- To make IP-based rate limiting effective in proxy deployments.
- To prevent all users from sharing a single rate limit bucket.

#### Syntax Rules and Structure

```js
app.set('trust proxy', 1);       // Trust one proxy (e.g., Nginx)
app.set('trust proxy', 'loopback'); // Trust loopback addresses
app.set('trust proxy', true);    // Trust all proxies (not recommended)
```

| Component | Breakdown |
|-----------|-----------|
| `1` | Number of proxies between client and server. |
| `'loopback'` | Trust only loopback addresses. |
| `true` | Trust all proxies (security risk). |

**Constraints and Limitations:**
- Setting `trust proxy` to `true` blindly trusts the `X-Forwarded-For` header, allowing attackers to spoof their IP.
- The correct number of proxies must be determined empirically.
- Some proxies include port numbers in the `X-Forwarded-For` header, requiring custom key generators.

#### Annotated Code Example

```js
// trust-proxy.js — Correct IP resolution behind a proxy
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// ✅ Trust exactly 1 proxy (e.g., Nginx, Cloudflare, Heroku router)
app.set('trust proxy', 1);

app.use(rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  keyGenerator: (req) => {
    // req.ip now returns the real client IP
    // Strip port number if present (e.g., Azure Application Gateway)
    return req.ip.replace(/:\d+[^:]*$/, '');
  }
}));

// Test endpoint to verify IP resolution
app.get('/ip', (req, res) => {
  res.json({ ip: req.ip });
});

app.listen(3000, () => console.log('Trust proxy server on 3000'));
```

**Expected Output (for `GET /ip` behind a proxy):**
```
{"ip":"203.0.113.42"}
```

**Why this output:** With `trust proxy` set to `1`, Express reads the `X-Forwarded-For` header and extracts the client IP. Without this setting, `req.ip` would return the proxy's IP (e.g., `10.0.0.1`).

#### Real-World Cases

- **Heroku deployments:** `app.set('trust proxy', 1)` is required for rate limiting to work correctly.
- **Cloudflare:** Trusting Cloudflare's IP range and extracting the real client IP.
- **Nginx reverse proxy:** Trusting the Nginx server as a single proxy.

---

## Core Concept 4: Request-Size & Body Limits

### Definitions

**Core Definition:** Request-size limits constrain the maximum size of HTTP request bodies, preventing attackers from exhausting server memory or CPU by sending excessively large payloads.

**Technical Definition:** Express's built-in body parsers (`express.json()` and `express.urlencoded()`) accept a `limit` option that specifies the maximum request body size as a string (e.g., `'100kb'`, `'10mb'`) or number of bytes. The default limit is 100KB. Without a limit, an attacker can send a multi-gigabyte JSON payload, causing memory exhaustion or excessive CPU consumption during parsing.

**Beginner-Friendly Explanation:** Request-size limits are like a mailbox that only accepts letters under a certain weight. If someone tries to stuff a package into the mailbox, it's rejected before the post office has to handle it.

### Purposes

- To prevent memory exhaustion from large request bodies.
- To mitigate denial-of-service attacks that exploit unbounded parsing.
- To enforce application-specific payload constraints (e.g., API endpoints that only expect small JSON objects).
- To reduce the attack surface for slowloris and multipart form attacks.

### Sub-Feature 4.1: Configuring Payload Limits

#### Definitions

**Core Definition:** Payload limits are the `limit` option values passed to `express.json()` and `express.urlencoded()` that cap the maximum request body size.

**Technical Definition:** The `limit` option accepts a string parsed by the `bytes` library (e.g., `'100kb'`, `'1mb'`) or a number of bytes. When a request exceeds the limit, the body parser returns a 413 (Payload Too Large) error. Different routes can have different limits by applying body parser middleware at the route level.

**Beginner-Friendly Explanation:** You set a different weight limit for different types of mail. A standard letter is limited to 100KB, but a bulk import endpoint might allow 10MB. Anything heavier is rejected.

#### Purposes

- To enforce strict payload constraints based on endpoint requirements.
- To prevent large-payload DoS attacks.
- To provide clear error responses when payloads exceed limits.

#### Syntax Rules and Structure

```js
// Global limit
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ limit: '100kb', extended: true }));

// Route-specific limit
app.post('/bulk-import', express.json({ limit: '10mb' }), handler);
```

| Component | Breakdown |
|-----------|-----------|
| `limit` | Maximum body size (string or number). |
| `extended` | Use `qs` for rich objects/arrays in URL-encoded bodies. |
| `type` | Content-Type to parse (default: `application/json`). |

**Constraints and Limitations:**
- The default limit is 100KB; applications that expect larger payloads must increase it.
- Setting a very large limit reintroduces the DoS risk.
- Multipart form data (file uploads) is not handled by `express.json()` or `express.urlencoded()`; use `multer` with its own limits.

#### Annotated Code Example

```js
// body-limits.js — Strict payload constraints
const express = require('express');
const app = express();

// Global JSON limit: 100KB for most routes
app.use(express.json({ limit: '100kb' }));

// URL-encoded limit: 50KB
app.use(express.urlencoded({ limit: '50kb', extended: true }));

// Standard endpoint uses global limit
app.post('/api/comment', (req, res) => {
  res.json({ received: req.body });
});

// Bulk endpoint uses a higher limit
app.post('/api/bulk-import', express.json({ limit: '10mb' }), (req, res) => {
  res.json({ imported: req.body.items?.length || 0 });
});

// Error handler for payload too large
app.use((err, req, res, next) => {
  if (err.type === 'entity.too.large') {
    return res.status(413).json({ error: 'Payload too large' });
  }
  next(err);
});

app.listen(3000, () => console.log('Body limits on 3000'));
```

**Expected Output (for `POST /api/comment` with a 200KB JSON body):**
```
{"error":"Payload too large"}
```

**Expected Output (for `POST /api/bulk-import` with a 5MB JSON body):**
```
{"imported":5000}
```

**Why this output:** The global `express.json({ limit: '100kb' })` rejects the 200KB body with a 413 error. The bulk import route uses its own `express.json({ limit: '10mb' })`, so the 5MB body is accepted.

#### Real-World Cases

- **Comment APIs:** 100KB limit prevents abuse while accommodating long comments.
- **Bulk data import:** 10MB limit for CSV-to-JSON conversion endpoints.
- **Webhook receivers:** Small limits (10KB) for webhook payloads.
- **File upload metadata:** 1MB limit for JSON metadata accompanying file uploads.

---

### Sub-Feature 4.2: Mitigating Slowloris and Multipart Attacks

#### Definitions

**Core Definition:** Slowloris attacks hold connections open by sending partial HTTP requests slowly, exhausting the server's connection pool. Multipart form attacks exploit unbounded multipart parsing to consume memory or CPU.

**Technical Definition:** Slowloris is mitigated at the reverse proxy or server level (e.g., Nginx `client_body_timeout`, Node.js `server.headersTimeout`). Multipart attacks are mitigated by setting file size and field count limits in `multer` or by using a reverse proxy to enforce maximum request sizes.

**Beginner-Friendly Explanation:** Slowloris is like a customer who orders coffee one word at a time, holding up the queue. Multipart attacks are like sending a package with thousands of tiny boxes inside, overwhelming the unpacking process.

#### Purposes

- To prevent connection pool exhaustion from slow, partial requests.
- To limit resource consumption during multipart form parsing.
- To enforce maximum file sizes for uploads.
- To reject maliciously crafted multipart payloads before processing.

#### Syntax Rules and Structure

```js
// Node.js HTTP server timeouts
const server = app.listen(3000);
server.headersTimeout = 20000;      // 20 seconds
server.requestTimeout = 30000;      // 30 seconds
server.keepAliveTimeout = 5000;     // 5 seconds

// Multer limits for file uploads
const multer = require('multer');
const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024,      // 5MB per file
    files: 5,                        // Maximum 5 files
    fields: 20,                      // Maximum 20 non-file fields
    fieldSize: 1024 * 1024           // 1MB per field
  }
});
```

| Component | Breakdown |
|-----------|-----------|
| `headersTimeout` | Maximum time to receive headers. |
| `requestTimeout` | Maximum time to receive the entire request. |
| `fileSize` | Maximum file size in bytes. |
| `files` | Maximum number of files. |

**Constraints and Limitations:**
- Slowloris is best mitigated at the reverse proxy level (Nginx, Cloudflare).
- Node.js timeouts apply globally; route-specific timeouts require custom middleware.
- Multer limits apply per request; combine with body limits for complete coverage.

#### Annotated Code Example

```js
// slowloris-multipart.js — Timeouts and upload limits
const express = require('express');
const multer = require('multer');
const app = express();

// Configure multer with strict limits
const upload = multer({
  storage: multer.memoryStorage(),
  limits: {
    fileSize: 5 * 1024 * 1024,   // 5MB
    files: 3,
    fields: 10,
    fieldSize: 512 * 1024        // 512KB
  },
  fileFilter: (req, file, cb) => {
    // Only allow images
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('Only images allowed'));
    }
  }
});

app.post('/upload', upload.array('photos', 3), (req, res) => {
  res.json({
    files: req.files?.length || 0,
    sizes: req.files?.map(f => f.size)
  });
});

// Error handler for multer errors
app.use((err, req, res, next) => {
  if (err.code === 'LIMIT_FILE_SIZE') {
    return res.status(413).json({ error: 'File too large (max 5MB)' });
  }
  if (err.code === 'LIMIT_FILE_COUNT') {
    return res.status(400).json({ error: 'Too many files' });
  }
  next(err);
});

const server = app.listen(3000);
server.headersTimeout = 20000;
server.requestTimeout = 30000;

console.log('Upload server on 3000');
```

**Expected Output (for uploading a 10MB file):**
```
{"error":"File too large (max 5MB)"}
```

**Expected Output (for uploading 4 files):**
```
{"error":"Too many files"}
```

**Why this output:** Multer enforces the `fileSize` and `files` limits during parsing. When a limit is exceeded, it emits an error with a specific code, which the error handler maps to a user-friendly response.

#### Real-World Cases

- **Image upload APIs:** 5MB file size limit, image MIME type filter.
- **Document management:** 10MB limit, PDF/DOCX MIME type filter.
- **Avatar uploads:** 2MB limit, single file, image filter.

---

## Core Concept 5: Data Sanitisation Middleware

### Definitions

**Core Definition:** Data sanitisation middleware inspects, cleans, and transforms raw request data before it reaches application logic, preventing injection attacks and ensuring data consistency.

**Technical Definition:** `express-validator` provides a validation chain API and a schema-based API for defining sanitisation and validation rules on `req.body`, `req.query`, `req.params`, `req.cookies`, and `req.headers`. Sanitizers such as `trim()`, `escape()`, `normalizeEmail()`, `toInt()`, and `toBoolean()` transform input data. The `validationResult()` function collects errors, allowing the handler to reject invalid requests.

**Beginner-Friendly Explanation:** Sanitisation is like washing vegetables before cooking. The vegetables (user input) might carry dirt or bacteria (malicious code). Washing them (sanitising) removes the harmful parts, and checking them (validating) ensures they're the right type and quality before they go into the pot (application logic).

### Purposes

- To strip dangerous HTML characters from user input, preventing stored XSS.
- To normalise data formats (email addresses, phone numbers, dates) for consistency.
- To cast string inputs to the correct type (integer, boolean, array).
- To validate that input meets application requirements before processing.

### Sub-Feature 5.1: Validation Chains and Sanitisation

#### Definitions

**Core Definition:** A validation chain is a sequence of validators and sanitizers applied to a specific field, executed in the order they are chained.

**Technical Definition:** `express-validator`'s `body()`, `query()`, `param()`, and `check()` functions create validation chains. Each chain can include validators (`isEmail()`, `isLength()`) and sanitizers (`trim()`, `escape()`). The `validationResult()` function checks whether any validators failed. Sanitizers run before validators in the same chain, allowing you to clean input before checking it.

**Beginner-Friendly Explanation:** A validation chain is like an assembly line for a single piece of data. First, the data is trimmed (whitespace removed), then escaped (dangerous characters removed), then checked for length, and finally checked for format. Each step happens in order.

#### Purposes

- To sanitise input before validation, ensuring validators see clean data.
- To validate multiple fields in a single chain.
- To collect all validation errors for a structured error response.
- To cast string inputs to the correct type.

#### Syntax Rules and Structure

```js
const { body, validationResult } = require('express-validator');

app.post('/user', [
  body('email').trim().isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }).trim().escape(),
  body('age').optional().isInt({ min: 0 }).toInt()
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  // Process validated and sanitised data
});
```

| Component | Breakdown |
|-----------|-----------|
| `body('email')` | Creates a chain for `req.body.email`. |
| `.trim()` | Sanitizer: removes whitespace. |
| `.isEmail()` | Validator: checks email format. |
| `.normalizeEmail()` | Sanitizer: normalises email format. |
| `.toInt()` | Sanitizer: converts to integer. |

**Constraints and Limitations:**
- Sanitizers and validators run in the order they are chained; sanitizers should run before validators for cleaning.
- `escape()` replaces HTML characters with entities; use it for XSS prevention but not for data that will be stored as plain text.
- `normalizeEmail()` may change the email address (e.g., lowercasing); ensure this is acceptable.

#### Annotated Code Example

```js
// sanitize-chain.js — Validation chain with sanitisation
const express = require('express');
const { body, query, param, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

app.post('/users/:id/comments', [
  // Sanitise and validate URL parameter
  param('id').isInt().toInt(),

  // Sanitise and validate body fields
  body('comment')
    .trim()
    .isLength({ min: 1, max: 1000 })
    .withMessage('Comment must be 1-1000 characters')
    .escape()  // Prevents stored XSS
    .withMessage('Invalid characters in comment'),

  body('rating')
    .optional()
    .isInt({ min: 1, max: 5 })
    .toInt()
    .withMessage('Rating must be 1-5'),

  body('notifyOnReply')
    .optional()
    .isBoolean()
    .toBoolean()
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  // Sanitised data is now available
  res.json({
    userId: req.params.id,           // Integer (cast by toInt)
    comment: req.body.comment,       // Trimmed and HTML-escaped
    rating: req.body.rating,         // Integer or undefined
    notify: req.body.notifyOnReply   // Boolean or undefined
  });
});

app.listen(3000, () => console.log('Sanitisation server on 3000'));
```

**Expected Output (for `POST /users/42/comments` with body `{"comment":" <script>alert(1)</script> ","rating":"5","notifyOnReply":"true"}`):**
```
{
  "userId": 42,
  "comment": "&lt;script&gt;alert(1)&lt;/script&gt;",
  "rating": 5,
  "notify": true
}
```

**Why this output:** The `comment` field is trimmed (whitespace removed), then `escape()` converts `<` to `&lt;` and `>` to `&gt;`, preventing the script tag from executing. The `rating` is validated as an integer between 1 and 5 and cast to a number. `notifyOnReply` is cast to a boolean.

#### Real-World Cases

- **Comment systems:** Sanitising comments to prevent stored XSS.
- **User registration:** Normalising email addresses and validating password strength.
- **Product reviews:** Casting ratings to integers and trimming review text.
- **Search filters:** Sanitising query parameters before applying to database queries.

---

### Sub-Feature 5.2: Schema-Based Validation and Sanitisation

#### Definitions

**Core Definition:** Schema-based validation uses a declarative object that defines validation and sanitisation rules for all fields in one place, rather than building separate chains for each field.

**Technical Definition:** `express-validator`'s `checkSchema()` function accepts a schema object where keys are field paths (e.g., `email`, `addresses.*.postalCode`) and values are objects defining validators, sanitizers, locations (`body`, `query`, `params`), error messages, and options. The schema can include wildcards for nested fields and conditional validation.

**Beginner-Friendly Explanation:** Instead of writing separate assembly lines for each field, you write a recipe card that describes all the ingredients and their requirements at once. The card says "email must be a valid email," "password must be at least 8 characters," and "age must be a number."

#### Purposes

- To centralise validation and sanitisation rules for maintainability.
- To support complex nested data structures with wildcard paths.
- To enable reusable schemas across multiple routes.
- To define optional fields and conditional validation.

#### Syntax Rules and Structure

```js
const { checkSchema } = require('express-validator');

const userSchema = {
  email: {
    in: ['body'],
    trim: true,
    isEmail: { errorMessage: 'Invalid email' },
    normalizeEmail: true
  },
  password: {
    in: ['body'],
    isLength: {
      errorMessage: 'Password must be at least 8 characters',
      options: { min: 8 }
    }
  },
  'addresses.*.postalCode': {
    optional: { options: { nullable: true } },
    isPostalCode: { options: 'US' }
  }
};

app.post('/user', checkSchema(userSchema), handler);
```

| Component | Breakdown |
|-----------|-----------|
| `in` | Locations to check (`body`, `query`, `params`). |
| `errorMessage` | Custom error message. |
| `optional` | Field is not required. |
| Wildcards | `addresses.*.postalCode` validates every nested postal code. |

**Constraints and Limitations:**
- Schema syntax differs from chain syntax; mixing them in one route can be confusing.
- Wildcard validation applies to all matching fields but does not enforce array length limits.
- Custom validators receive `(value, { req, location, path })` and must return a boolean.

#### Annotated Code Example

```js
// schema-validation.js — Schema-based validation and sanitisation
const express = require('express');
const { checkSchema, validationResult } = require('express-validator');
const app = express();
app.use(express.json());

const registrationSchema = {
  username: {
    in: ['body'],
    trim: true,
    isLength: {
      errorMessage: 'Username must be 3-20 characters',
      options: { min: 3, max: 20 }
    },
    matches: {
      errorMessage: 'Username must be alphanumeric',
      options: /^[a-zA-Z0-9_]+$/
    }
  },
  email: {
    in: ['body'],
    trim: true,
    isEmail: { errorMessage: 'Invalid email address' },
    normalizeEmail: true
  },
  password: {
    in: ['body'],
    isLength: {
      errorMessage: 'Password must be at least 8 characters',
      options: { min: 8 }
    }
  },
  age: {
    in: ['body'],
    optional: { options: { nullable: true } },
    isInt: { errorMessage: 'Age must be a number', options: { min: 18, max: 120 } },
    toInt: true
  },
  'interests.*': {
    in: ['body'],
    trim: true,
    isLength: { options: { max: 50 } }
  }
};

app.post('/register', checkSchema(registrationSchema), (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  res.json({
    username: req.body.username,
    email: req.body.email,
    age: req.body.age,
    interests: req.body.interests
  });
});

app.listen(3000, () => console.log('Schema validation on 3000'));
```

**Expected Output (for a valid registration):**
```
{
  "username": "alice_99",
  "email": "alice@example.com",
  "age": 25,
  "interests": ["coding", "hiking"]
}
```

**Expected Output (for invalid data):**
```
{
  "errors": [
    { "msg": "Username must be 3-20 characters", "param": "username", "location": "body" },
    { "msg": "Invalid email address", "param": "email", "location": "body" }
  ]
}
```

**Why this output:** The schema defines all validation and sanitisation rules in one object. Valid data passes and is returned with sanitised values (trimmed username, normalised email, integer age). Invalid data produces a structured error array with field names and messages.

#### Real-World Cases

- **User registration forms:** Validating and sanitising all fields in one schema.
- **Product catalogues:** Validating nested category and pricing data.
- **Multi-step forms:** Reusing schemas across multiple route handlers.
- **API gateways:** Centralising input validation for multiple microservices.

---

## References

- Helmet.js Official Documentation — https://helmet.js.org/
- Helmet.js GitHub Repository — https://github.com/helmetjs/helmet
- Helmet Content Security Policy README — https://github.com/helmetjs/helmet/tree/main/middlewares/content-security-policy
- Express.js CORS Middleware — https://expressjs.com/en/resources/middleware/cors.html
- CORS npm Package (GitHub) — https://github.com/expressjs/cors
- MDN CORS Guide — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- express-rate-limit Documentation — https://express-rate-limit.mintlify.app/
- express-rate-limit Troubleshooting Proxy Issues — https://express-rate-limit.mintlify.app/guides/troubleshooting-proxy-issues
- Express.js Behind Proxies Guide — https://expressjs.com/en/guide/behind-proxies.html
- rate-limit-redis npm Package — https://www.npmjs.com/package/rate-limit-redis
- express-validator Documentation — https://express-validator.github.io/docs/
- express-validator Schema Validation — https://express-validator.github.io/docs/schema-validation/
- express-validator Sanitization Middlewares — https://express-validator.github.io/docs/sanitization-middlewares/
- Express.js Body-Parser API (express.json) — https://expressjs.com/en/5x/api.html#express.json
- CVE-2025-67731 (Servify Express JSON body limit) — https://nvd.nist.gov/vuln/detail/CVE-2025-67731
- OWASP Secure Headers Project — https://owasp.org/www-project-secure-headers/
- MDN Content Security Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP
- Node.js HTTP Server Timeouts — https://nodejs.org/api/http.html#serverheadertimeout
- Multer npm Package — https://www.npmjs.com/package/multer