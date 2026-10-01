# Security, Middleware, & Request Validation — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Security, middleware, and request validation in Node.js encompasses the practices, libraries, and architectural patterns used to protect web applications from malicious input, abuse, and common attack vectors such as injection, cross-site scripting, and denial-of-service.

**Technical Definition:** Request validation is the process of verifying that incoming HTTP request data (body, query parameters, route parameters, headers) conforms to an expected schema before it reaches business logic. Middleware is a function in the Express/Fastify/Koa pipeline that has access to the request object, response object, and the next middleware function, enabling cross-cutting concerns such as validation, rate limiting, CORS, and sanitisation to be applied declaratively. Security middleware forms a defensive layer that mitigates OWASP Top 10 risks including A01: Broken Access Control, A03: Injection, and A05: Security Misconfiguration.

**Beginner-Friendly Explanation:** Every time a user sends data to your server — filling out a form, uploading a file, or calling your API — you're receiving something from an untrusted source. Security middleware and request validation are like a bouncer at a club: they check IDs (validation), limit how many people enter at once (rate limiting), decide who's allowed in (CORS), and make sure nobody smuggles in anything dangerous (sanitisation). Without them, attackers can inject malicious code, overwhelm your server, or steal data.

### Key Characteristics

- **Defence in depth:** Multiple layers of protection — validation, sanitisation, rate limiting, and CORS — work together.
- **Schema-first validation:** Modern libraries (Zod, Joi, AJV) define expected data shapes declaratively, rejecting malformed requests at the boundary.
- **Payload size limits:** Body parsers enforce maximum request sizes to prevent memory exhaustion and denial-of-service.
- **Rate limiting algorithms:** Token bucket and sliding window algorithms control request throughput, with `X-RateLimit-*` headers communicating limits to clients.
- **CORS as controlled access:** CORS is a browser relaxation mechanism — it does not protect APIs; it selectively allows cross-origin reads.
- **Injection prevention by design:** Parameterised queries, schema validation, and context-aware encoding prevent SQL, NoSQL, and XSS attacks.
- **Centralised validation:** Validation should be a reusable middleware, not scattered `if (!req.body.x)` checks throughout route handlers.

### Prerequisites

- **HTTP fundamentals:** Request methods, status codes, headers, and bodies.
- **Express.js or Fastify fundamentals:** Middleware pipeline, routing, and error handling.
- **JavaScript/TypeScript:** Async/await, Promises, and schema definition.
- **Basic security concepts:** OWASP Top 10, injection attacks, and the same-origin policy.
- **Database familiarity:** SQL parameterised queries and NoSQL query patterns.

### Related Programming Areas

- **Advanced API Design:** Status codes, error formatting (RFC 7807), and response envelopes.
- **HTTP Server Architecture:** Middleware pipelines and request processing.
- **Database Querying:** Parameterised queries and ORM safety.
- **Authentication & Authorization:** JWT, sessions, and access control.
- **Logging & Monitoring:** Detecting and responding to security incidents.

### Core Concepts

1. **Comprehensive Request Validation** — payload size limits, request-body/query/param validation, and schema-based validation (Zod, Joi, AJV).
2. **API Security Essentials** — CORS, rate limiting, and data sanitisation (SQL, XSS, NoSQL injection prevention).

---

## Core Concept 1: Comprehensive Request Validation

### Sub-Feature 1.1: Payload Size Limits (Preventing DoS via Massive JSON Bodies)

#### Definitions

**Core Definition:** Payload size limits restrict the maximum amount of data accepted in an HTTP request body, preventing attackers from exhausting server memory by sending extremely large JSON or form-encoded payloads.

**Technical Definition:** Express's built-in body parsers (`express.json()` and `express.urlencoded()`) default to a 100kb limit. This limit can be configured globally or per-route. Without an explicit limit, or with an excessively large global limit, a few hundred concurrent large POST requests can pin the event loop and exhaust memory. CVE-2026-12590 (CVSS 3.7) describes a body-parser vulnerability where an invalid limit value (e.g., an unparseable string or `NaN`) causes `bytes.parse()` to return `null`, silently skipping the size check — fixed in body-parser 1.20.6 and 2.3.0.

**Beginner-Friendly Explanation:** Imagine a post office that accepts any size parcel. An attacker could send thousands of enormous parcels and fill the entire warehouse. Payload size limits are like saying "we only accept parcels up to 1 kg" — anything bigger is rejected at the door. The default limit is 100kb, which is fine for most APIs, but teams often increase it globally for one upload endpoint and forget to reduce it elsewhere.

#### Purposes

- To prevent memory exhaustion and denial-of-service from oversized payloads.
- To protect the event loop from being blocked by parsing massive JSON bodies.
- To enforce a predictable resource envelope per request.
- To comply with security audits and framework hardening requirements.

#### Syntax Rules and Structure

```javascript
// Global limit (applies to all routes)
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ extended: true, limit: '100kb' }));

// Route-specific limit (for file upload endpoints)
app.post('/upload', express.json({ limit: '50mb' }), uploadHandler);

// With explicit error handling
app.use((err, req, res, next) => {
  if (err.type === 'entity.too.large') {
    return res.status(413).json({
      error: 'Payload Too Large',
      message: `Request body exceeds the ${err.limit} limit`,
    });
  }
  next(err);
});
```

| Option | Description | Default |
|--------|-------------|---------|
| `limit` | Maximum body size (`'100kb'`, `'1mb'`, `'50mb'`). | `'100kb'` |
| `type` | Content-Type to match. | `'application/json'` |
| `strict` | Only accept arrays and objects. | `true` |

**Constraints and Limitations:**
- The `limit` option is parsed by the `bytes` library; invalid values (e.g., `NaN`) silently disable the check in vulnerable versions.
- A global limit of `'50mb'` exposes every route to DoS; apply large limits only at the route level.
- The 413 response should be handled by an error middleware for a consistent API response.

#### Annotated Code Example

```javascript
// payload-limit.js
const express = require('express');
const app = express();

// Global limit: 100kb for all JSON bodies
app.use(express.json({ limit: '100kb' }));

// Route-specific limit: 10mb for a specific upload endpoint
app.post('/api/upload', express.json({ limit: '10mb' }), (req, res) => {
  res.json({ received: true, size: JSON.stringify(req.body).length });
});

// Error handler for payload too large
app.use((err, req, res, next) => {
  if (err.type === 'entity.too.large') {
    return res.status(413).json({
      error: 'Payload Too Large',
      limit: err.limit,
    });
  }
  next(err);
});

app.listen(3000);
```

**Expected Output (for a 200kb JSON body):**
```http
HTTP/1.1 413 Payload Too Large

{"error":"Payload Too Large","limit":"100kb"}
```

**Expected Output (for a 5mb JSON body to `/api/upload`):**
```json
{"received":true,"size":5242880}
```

**Why this output:** The global `express.json({ limit: '100kb' })` rejects the 200kb body with a 413 response. The route-specific `express.json({ limit: '10mb' })` overrides the global limit for `/api/upload`, allowing the 5mb body to be processed.

#### Real-World Cases

- **API gateways:** Enforcing a strict global limit (e.g., 100kb) for JSON APIs.
- **File upload endpoints:** Applying a larger per-route limit (e.g., 10mb) only where needed.
- **Security audits:** Verifying that `body-parser` is not in `package.json` and that `express.json()` limits are explicit.

---

### Sub-Feature 1.2: Request-Body, Query-String, and Route-Parameter Validation

#### Definitions

**Core Definition:** Request-body, query-string, and route-parameter validation ensures that all incoming data matches expected types, formats, and constraints before reaching business logic, preventing injection, crashes, and inconsistent data.

**Technical Definition:** Express does not validate request input by default. Handlers receive raw `req.body`, `req.query`, and `req.params` — strings where numbers were expected, missing fields, or unexpected types. Validation middleware intercepts requests before the handler, parses the data against a schema, and either passes the validated (and often coerced) data to `req.body`/`req.query`/`req.params` or returns a 400 Bad Request with detailed error messages.

**Beginner-Friendly Explanation:** Imagine a restaurant that accepts orders written on napkins. Without validation, the kitchen might receive "one thousand pizzas" when the customer meant "one pizza" — or "DROP TABLE orders" written in the special instructions. Validation middleware is like a waiter who reads the order back, checks it's on the menu, confirms the quantity is reasonable, and only then passes it to the kitchen.

#### Purposes

- To reject malformed requests at the boundary before they reach business logic.
- To coerce types (e.g., string `"42"` → integer `42` for route parameters).
- To provide consistent, structured error responses for invalid input.
- To reduce the attack surface for injection and logic bugs.

#### Syntax Rules and Structure

```javascript
// Generic validation middleware
function validate(schemas) {
  return (req, res, next) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.query) req.query = schemas.query.parse(req.query);
      if (schemas.params) req.params = schemas.params.parse(req.params);
      next();
    } catch (error) {
      return res.status(400).json({ error: 'Validation failed', details: error });
    }
  };
}

// Usage
app.post('/users', validate({ body: createUserSchema }), handler);
```

| Data Source | Access | Common Pitfalls |
|-------------|--------|----------------|
| Body | `req.body` | Only available after `express.json()` / `express.urlencoded()`. |
| Query | `req.query` | All values are strings; `?page=1` is `"1"`, not `1`. |
| Params | `req.params` | Always strings; `/:id` yields `"42"`, not `42`. |

**Constraints and Limitations:**
- Query parameters always arrive as strings; explicit coercion is required.
- Route parameters can be polluted if not validated (e.g., `../` path traversal).
- Validation must run before any business logic or database interaction.

#### Annotated Code Example

```javascript
// request-validation.js
const express = require('express');
const { z } = require('zod');
const app = express();
app.use(express.json());

// Schema for creating a user
const createUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  password: z.string().min(8),
  age: z.number().int().positive().optional(),
});

// Schema for query parameters (coerce strings to numbers)
const listUsersSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
});

// Schema for route parameters
const userParamsSchema = z.object({
  userId: z.string().uuid(),
});

function validate(schemas) {
  return (req, res, next) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.query) req.query = schemas.query.parse(req.query);
      if (schemas.params) req.params = schemas.params.parse(req.params);
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(400).json({
          error: 'Validation failed',
          details: error.errors.map(e => ({
            path: e.path.join('.'),
            message: e.message,
          })),
        });
      }
      next(error);
    }
  };
}

app.post('/users', validate({ body: createUserSchema }), (req, res) => {
  res.status(201).json({ message: 'User created', user: req.body });
});

app.get('/users', validate({ query: listUsersSchema }), (req, res) => {
  res.json({ page: req.query.page, limit: req.query.limit });
});

app.get('/users/:userId', validate({ params: userParamsSchema }), (req, res) => {
  res.json({ userId: req.params.userId });
});

app.listen(3000);
```

**Expected Output (for `POST /users` with valid body):**
```json
{"message":"User created","user":{"username":"alice","email":"alice@example.com","password":"secure123"}}
```

**Expected Output (for `POST /users` with `{"username":"al","email":"not-an-email"}`):**
```json
{
  "error": "Validation failed",
  "details": [
    { "path": "username", "message": "String must contain at least 3 character(s)" },
    { "path": "email", "message": "Invalid email" }
  ]
}
```

**Expected Output (for `GET /users?page=2&limit=50`):**
```json
{"page":2,"limit":50}
```

**Why this output:** The `validate` middleware parses each part of the request against the corresponding Zod schema. Zod's `z.coerce.number()` converts the string query parameters to numbers. Invalid input produces a structured 400 response with field-specific error messages. Valid input is assigned back to `req.body`/`req.query`/`req.params` and passed to the handler.

#### Real-World Cases

- **User registration:** Validating email format, password strength, and username constraints.
- **Pagination:** Coercing `page` and `limit` query strings to integers with bounds.
- **Resource identification:** Validating UUID or numeric IDs in route parameters.

---

### Sub-Feature 1.3: Schema-Based Validation Using Zod, Joi, and AJV

#### Definitions

**Core Definition:** Schema-based validation uses declarative schema libraries — Zod, Joi, or AJV — to define the expected shape of request data and automatically validate incoming requests against those schemas.

**Technical Definition:** Zod is a TypeScript-first schema declaration and validation library with static type inference, making it the default choice for new projects. Joi is a mature, feature-rich validation library with extensive built-in rules and custom error messages, widely used in production for flexible schema validation. AJV (Another JSON Schema Validator) implements the JSON Schema standard and is used in performance-critical or OpenAPI-driven validation scenarios. All three integrate with Express via middleware.

**Beginner-Friendly Explanation:** A schema is like a blueprint for your data. Zod, Joi, and AJV are three different tools for drawing that blueprint. Zod is the newest and works beautifully with TypeScript. Joi has been around the longest and has the most features. AJV is the fastest and follows a strict standard (JSON Schema). Choosing between them depends on your project's priorities: type safety (Zod), flexibility (Joi), or standards compliance and speed (AJV).

#### Purposes

- To define the expected shape of request data declaratively.
- To provide type inference (Zod) or JSON Schema compliance (AJV).
- To generate consistent error messages for invalid requests.
- To decouple validation logic from route handlers.

#### Syntax Rules and Structure

**Zod:**
```javascript
const schema = z.object({
  name: z.string().min(1),
  age: z.number().int().positive().optional(),
  email: z.string().email(),
});
const result = schema.safeParse(req.body);
if (!result.success) return res.status(400).json(result.error);
```

**Joi:**
```javascript
const schema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
});
const { error, value } = schema.validate(req.body, { abortEarly: false });
if (error) return res.status(400).json(error.details);
```

**AJV:**
```javascript
const ajv = new Ajv();
const validate = ajv.compile({
  type: 'object',
  required: ['name'],
  properties: { name: { type: 'string', minLength: 1 } },
});
if (!validate(req.body)) return res.status(400).json(validate.errors);
```

| Library | Schema Format | Type Inference | Best For |
|---------|--------------|----------------|----------|
| Zod | TypeScript-first | Excellent | TypeScript projects, DX |
| Joi | Joi DSL | Limited | Production APIs, flexible rules |
| AJV | JSON Schema | Via `json-schema-to-ts` | OpenAPI, performance-critical |

**Constraints and Limitations:**
- Joi is larger and has a steeper learning curve than Zod.
- AJV requires JSON Schema authoring, which is more verbose.
- Zod's error messages can be less user-friendly without customisation.

#### Annotated Code Example

```javascript
// joi-validation.js
const express = require('express');
const Joi = require('joi');
const app = express();
app.use(express.json());

// Define schema
const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  username: Joi.string().alphanum().min(3).max(30).required(),
  age: Joi.number().integer().min(18).max(120).optional(),
});

// Validation middleware factory
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,   // Return all errors
      stripUnknown: true,  // Remove unknown properties
    });
    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message,
      }));
      return res.status(400).json({ success: false, errors });
    }
    req.body = value;
    next();
  };
}

app.post('/api/users', validate(createUserSchema), (req, res) => {
  res.status(201).json({ success: true, user: req.body });
});

app.listen(3000);
```

**Expected Output (for valid body):**
```json
{"success":true,"user":{"email":"alice@example.com","password":"secure123","username":"alice"}}
```

**Expected Output (for invalid body with short password):**
```json
{
  "success": false,
  "errors": [
    { "field": "password", "message": "\"password\" length must be at least 8 characters long" }
  ]
}
```

**Why this output:** Joi validates the body against the schema. `abortEarly: false` collects all errors rather than stopping at the first one. `stripUnknown: true` removes any properties not defined in the schema, preventing property injection. The validated and cleaned value is assigned back to `req.body`.

#### Real-World Cases

- **Zod + TypeScript:** Full-stack TypeScript projects where type inference is critical.
- **Joi in production:** E-commerce APIs with complex business rules and custom error messages.
- **AJV + OpenAPI:** APIs with OpenAPI specifications where the schema is already defined.

---

## Core Concept 2: API Security Essentials

### Sub-Feature 2.1: Cross-Origin Resource Sharing (CORS)

#### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser mechanism that uses HTTP headers to selectively allow a web page from one origin to read responses from another origin, relaxing the same-origin policy in a controlled way.

**Technical Definition:** The same-origin policy blocks cross-origin JavaScript from reading responses by default. CORS is the controlled hole punched in that policy: the server sends `Access-Control-Allow-Origin` (and related headers) to tell the browser which origins may read its responses. For requests with side effects — a PUT, a JSON `Content-Type`, or an `Authorization` header — the browser first sends an `OPTIONS` preflight request asking permission. If the preflight response does not approve the method and headers, the real request never happens.

**Beginner-Friendly Explanation:** CORS is like a nightclub with a guest list. By default, the bouncer (browser) only lets in people from the same neighbourhood (same origin). CORS headers are the guest list that says "also let in people from these specific other neighbourhoods." You cannot just say "let everyone in" (`*`) if you're checking IDs (credentials), and if you're vague about who's allowed, you've effectively let everyone in — which is a security hole.

#### Purposes

- To allow legitimate cross-origin requests from trusted frontends.
- To protect against cross-site request forgery (CSRF) and data theft.
- To handle preflight `OPTIONS` requests correctly.
- To enforce an explicit allowlist of trusted origins.

#### Syntax Rules and Structure

```javascript
const cors = require('cors');

const allowlist = new Set([
  'https://app.example.com',
  'https://admin.example.com',
]);

app.use(cors({
  origin(origin, callback) {
    if (!origin) return callback(null, false);
    callback(null, allowlist.has(origin));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600,
}));
```

| Option | Description | Recommended |
|--------|-------------|-------------|
| `origin` | Allowed origins (allowlist, not `*`). | Exact-match `Set` |
| `credentials` | Allow cookies/auth headers. | `true` only when needed |
| `methods` | Allowed HTTP methods. | Explicit list |
| `allowedHeaders` | Allowed request headers. | Explicit list |
| `maxAge` | Preflight cache duration (seconds). | 600 |

**Constraints and Limitations:**
- CORS is enforced by the browser only; `curl`, server-to-server calls, and attackers ignore it completely.
- A blocked response is often still executed server-side; CORS never prevented the write.
- `Access-Control-Allow-Origin: *` cannot be used with credentials.
- Reflecting the `Origin` header back without an allowlist is a common misconfiguration.

#### Annotated Code Example

```javascript
// cors-setup.js
const express = require('express');
const cors = require('cors');
const app = express();

const allowlist = new Set([
  'https://app.example.com',
  'https://admin.example.com',
]);

app.use(cors({
  origin(origin, callback) {
    // Non-browser clients (no Origin header) pass through
    if (!origin) return callback(null, false);
    callback(null, allowlist.has(origin));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600,
}));

app.get('/api/data', (req, res) => {
  res.json({ data: 'protected' });
});

app.listen(3000);
```

**Expected Output (for a request from `https://app.example.com`):**
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Content-Type: application/json

{"data":"protected"}
```

**Expected Output (for a request from `https://evil.com`):**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{"data":"protected"}
```
*(The browser blocks the JavaScript from reading the response, but the request still executed server-side.)*

**Why this output:** The `origin` callback checks against the allowlist. Allowed origins receive the `Access-Control-Allow-Origin` header. Non-allowed origins receive no CORS headers, so the browser blocks JavaScript access — but the server still processed the request. This illustrates why CORS is not a substitute for authentication.

#### Real-World Cases

- **SPA frontends:** Allowing a React app on a different subdomain to call the API.
- **Multi-tenant applications:** Allowing tenant-specific origins.
- **Preventing CSRF:** Using `credentials: true` with an explicit allowlist and `SameSite` cookies.

---

### Sub-Feature 2.2: Rate Limiting — Sliding Window and Token Bucket Algorithms

#### Definitions

**Core Definition:** Rate limiting restricts the number of requests a client can make within a time window, protecting APIs from abuse, brute-force attacks, and denial-of-service.

**Technical Definition:** Two common algorithms are **sliding window**, which tracks request timestamps in a rolling window for accurate limits with smooth distribution, and **token bucket**, which maintains a bucket of tokens that refill at a fixed rate, allowing bursts up to the bucket capacity. The `express-rate-limit` middleware implements both, with `standardHeaders: true` emitting `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` headers (formerly `X-RateLimit-*`). In production, the default in-memory store must be replaced with Redis for distributed deployments.

**Beginner-Friendly Explanation:** Rate limiting is like a bouncer who lets in 10 people per minute. If 50 people arrive, 40 wait outside. The sliding window algorithm checks how many people entered in the last 60 seconds; the token bucket algorithm gives the bouncer a bucket of 10 tokens that refill one per second — allowing a burst of 10 at once but then limiting the rate.

#### Purposes

- To prevent brute-force attacks on login and OTP endpoints.
- To protect against denial-of-service from request floods.
- To enforce fair usage tiers (free vs. pro vs. enterprise).
- To communicate limits to clients via standard headers.

#### Syntax Rules and Structure

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                    // 100 requests per window
  standardHeaders: true,       // RateLimit-* headers
  legacyHeaders: false,        // Disable X-RateLimit-* headers
  message: { error: 'Too many requests, try again later' },
});

app.use('/api/', limiter);
```

| Option | Description | Default |
|--------|-------------|---------|
| `windowMs` | Window duration in milliseconds. | 60000 |
| `max` | Max requests per window. | 5 |
| `standardHeaders` | Emit `RateLimit-*` headers. | `false` |
| `legacyHeaders` | Emit `X-RateLimit-*` headers. | `true` |
| `store` | Storage backend (MemoryStore, RedisStore). | MemoryStore |

**Response headers:**
| Header | Description |
|--------|-------------|
| `RateLimit-Limit` | Max requests allowed in window. |
| `RateLimit-Remaining` | Requests remaining in window. |
| `RateLimit-Reset` | Seconds until window resets. |
| `Retry-After` | Seconds to wait before retrying (on 429). |

**Constraints and Limitations:**
- The default MemoryStore does not share state across process instances; use Redis for multi-pod deployments.
- `req.ip` may be the load balancer IP if `app.set('trust proxy')` is not configured correctly.
- Rate limiting by IP alone can affect users behind shared NAT.

#### Annotated Code Example

```javascript
// rate-limit.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// Strict limiter for authentication endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                     // 5 attempts per window
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many login attempts, try again in 15 minutes' },
});

// General API limiter
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minute
  max: 100,              // 100 requests per minute
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', apiLimiter);
app.post('/api/auth/login', authLimiter, (req, res) => {
  res.json({ token: 'jwt-token' });
});

app.listen(3000);
```

**Expected Output (for the 6th login attempt within 15 minutes):**
```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 5
RateLimit-Remaining: 0
RateLimit-Reset: 847
Retry-After: 847

{"error":"Too many login attempts, try again in 15 minutes"}
```

**Expected Output (for the 50th API request within 1 minute):**
```http
HTTP/1.1 200 OK
RateLimit-Limit: 100
RateLimit-Remaining: 50
RateLimit-Reset: 30
```

**Why this output:** The auth limiter allows 5 attempts per 15 minutes. The 6th attempt is rejected with a 429 response and `Retry-After` header. The API limiter allows 100 requests per minute; the 50th request succeeds with 50 remaining. The `standardHeaders: true` ensures the `RateLimit-*` headers are emitted.

#### Real-World Cases

- **Login endpoints:** 5 attempts per 15 minutes to prevent brute-force attacks.
- **Public APIs:** 100 requests per minute per IP for general abuse prevention.
- **Tiered access:** 100/hour (free), 1,000/hour (pro), 10,000/hour (enterprise).

---

### Sub-Feature 2.3: Data Sanitisation — SQL, XSS, and NoSQL Injection Prevention

#### Definitions

**Core Definition:** Data sanitisation is the process of cleaning or encoding user input to prevent it from being interpreted as executable code in SQL queries, HTML output, or NoSQL queries.

**Technical Definition:** SQL injection occurs when untrusted input is concatenated directly into SQL statements, allowing attackers to modify query logic. Prevention uses parameterised queries (prepared statements) where user input is passed as bound parameters, never as SQL text. NoSQL injection (e.g., MongoDB) occurs when unvalidated operator expressions like `$where` or query operators are passed from user input; prevention uses schema validation and operator sanitisation. XSS occurs when user input is rendered into HTML without encoding; prevention uses context-aware output encoding and HTML sanitisation libraries.

**Beginner-Friendly Explanation:** Sanitisation is like washing your hands before cooking. You don't want dirt (malicious code) to get into the food (your database or web page). Parameterised queries are like using a recipe card with blanks — the chef fills in the ingredients, but the instructions never change. Output encoding is like putting a glass cover over a painting — you can still see it, but nobody can touch it.

#### Purposes

- To prevent attackers from reading, modifying, or deleting database data.
- To prevent malicious scripts from executing in users' browsers (XSS).
- To prevent authentication bypass via NoSQL operator injection.
- To comply with OWASP Top 10 and security audit requirements.

#### Syntax Rules and Structure

**SQL injection prevention (parameterised queries):**
```javascript
// ❌ Vulnerable: string concatenation
const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;

// ✅ Safe: parameterised query
const result = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [req.body.email]
);
```

**NoSQL injection prevention (MongoDB):**
```javascript
// ❌ Vulnerable: passing user input directly
const user = await User.findOne({ email: req.body.email });

// ✅ Safe: validate input type and sanitise
if (typeof req.body.email !== 'string') {
  return res.status(400).json({ error: 'Invalid email' });
}
const user = await User.findOne({ email: req.body.email });
```

**XSS prevention (output encoding):**
```javascript
// ❌ Vulnerable: rendering raw user input
res.send(`<h1>Hello, ${req.query.name}</h1>`);

// ✅ Safe: encode output
const escapeHtml = require('escape-html');
res.send(`<h1>Hello, ${escapeHtml(req.query.name)}</h1>`);
```

| Attack | Vector | Prevention |
|--------|--------|------------|
| SQL Injection | Concatenated SQL | Parameterised queries. |
| NoSQL Injection | Query operators (`$gt`, `$where`) | Schema validation, type checks. |
| XSS | Unsanitised HTML output | Output encoding, CSP, `escape-html`. |
| Command Injection | `exec()` with user input | `execFile()` with argument arrays. |
| Path Traversal | `../` in file paths | `path.normalize()`, allowlist validation. |

**Constraints and Limitations:**
- Parameterised queries are supported by all major SQL drivers and ORMs.
- NoSQL injection via `$where` requires server-side JavaScript execution to be disabled.
- XSS prevention requires context-aware encoding (HTML, attribute, JavaScript, URL contexts).

#### Annotated Code Example

```javascript
// sanitisation.js
const express = require('express');
const escapeHtml = require('escape-html');
const { Pool } = require('pg');
const app = express();
app.use(express.json());

const pool = new Pool();

// SQL injection prevention (parameterised query)
app.get('/users/search', async (req, res) => {
  const { email } = req.query;

  // Validate type
  if (typeof email !== 'string') {
    return res.status(400).json({ error: 'Invalid email parameter' });
  }

  // ✅ Parameterised query — user input is a bound parameter
  const result = await pool.query(
    'SELECT id, name, email FROM users WHERE email = $1',
    [email]
  );

  res.json({ data: result.rows });
});

// XSS prevention (output encoding)
app.get('/greet', (req, res) => {
  const name = req.query.name || 'World';

  // ✅ Encode output before rendering into HTML
  const safeName = escapeHtml(name);
  res.type('html').send(`<h1>Hello, ${safeName}</h1>`);
});

// NoSQL injection prevention (MongoDB — type validation)
app.post('/login', async (req, res) => {
  const { email, password } = req.body;

  // ✅ Validate that inputs are strings
  if (typeof email !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid credentials format' });
  }

  // ✅ Use string values only (no operator objects)
  const user = await db.collection('users').findOne({ email });
  // ... validate password ...
  res.json({ authenticated: !!user });
});

app.listen(3000);
```

**Expected Output (for `GET /greet?name=<script>alert('xss')</script>`):**
```html
<h1>Hello, &lt;script&gt;alert(&#39;xss&#39;)&lt;/script&gt;</h1>
```
*(The script tags are encoded and rendered as text, not executed.)*

**Expected Output (for SQL injection attempt via `?email=' OR '1'='1`):**
```json
{"data":[]}
```
*(The parameterised query treats the entire input as a literal string, returning no results instead of all users.)*

**Expected Output (for NoSQL injection attempt with `{"email":{"$gt":""}}`):**
```json
{"error":"Invalid credentials format"}
```
*(The type check rejects the object value before it reaches the database.)*

**Why this output:** Parameterised queries prevent SQL injection by separating SQL logic from data. `escapeHtml` encodes XSS payloads into safe HTML entities. Type validation rejects NoSQL operator objects before they can be interpreted by MongoDB.

#### Real-World Cases

- **Authentication:** Parameterised queries for login and registration.
- **Search APIs:** Parameterised queries for user-supplied search terms.
- **User-generated content:** Output encoding for comments, profiles, and messages.
- **MongoDB applications:** Type validation and operator sanitisation for query parameters.

---

## References

- Zod Documentation — https://zod.dev/
- Joi Documentation — https://joi.dev/api/
- AJV — JSON Schema Validator — https://ajv.js.org/
- Express.js Security Best Practices — https://expressjs.com/en/advanced/best-practice-security.html
- express-rate-limit — GitHub — https://github.com/express-rate-limit/express-rate-limit
- MDN Web Docs — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- OWASP — SQL Injection Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- OWASP — XSS Prevention Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
- OWASP — NoSQL Injection — https://owasp.org/www-community/attacks/NoSQL_injection
- CVE-2026-12590 — body-parser Payload Limit Bypass — https://docs.devguard.org/
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- Safeguard.sh — Express.js Security Middleware Audit — https://safeguard.sh/resources/blog/express-js-security-middleware-audit
- Safeguard.sh — Why We Use CORS in Node.js: Configuration Without Foot-Guns — https://safeguard.sh/resources/blog/cors-in-nodejs-explained
- Steve Kinney — Using Zod with Express — https://stevekinney.com/courses/full-stack-typescript/using-zod-with-express
- Steve Kinney — Validating Path and Query Parameters with Middleware — https://stevekinney.com/courses/full-stack-typescript/validating-query-and-path-parameters
- CoreUI — How to use Joi for validation in Node.js — https://coreui.io/answers/how-to-use-joi-for-validation-in-nodejs/
- KodeKloud — Demo Input Validation — https://notes.kodekloud.com/docs/Claude-Code-For-Beginners/Security-Auditing-with-Claude-Code/Demo-Input-Validation
- Skill Gallery — api-rate-limiting — https://www.skill-gallery.jp/en/skills/secondsky/api-rate-limiting