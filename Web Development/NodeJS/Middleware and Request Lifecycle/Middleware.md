# Middleware Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Middleware in Node.js refers to functions that have access to the request object, the response object, and the next middleware function in the application's request-response cycle, enabling modular processing of HTTP requests.

**Technical Definition:** Express is a routing and middleware web framework with minimal functionality of its own: an Express application is essentially a series of middleware function calls. Middleware functions execute sequentially in the order they are registered, forming a pipeline that processes requests before they reach route handlers. Middleware can perform tasks such as parsing request bodies, authenticating users, logging requests, validating input, transforming data, and managing request-scoped context.

**Beginner-Friendly Explanation:** Middleware is like an assembly line in a factory. Each worker (middleware function) does something to the product (request) before passing it to the next worker. One worker parses the body, another checks authentication, another logs the request, and the final worker sends the response. The order matters — if you check authentication before parsing the body, you might process unauthenticated data.

### Key Characteristics

- **Sequential execution:** Middleware functions execute in the order they are registered.
- **Shared request/response objects:** Every middleware has access to `req`, `res`, and `next`.
- **Cross-cutting concerns:** Middleware handles logging, authentication, validation, and context management without cluttering route handlers.
- **Composable:** Multiple middleware functions can be chained, and each can modify `req` or `res` before passing control.
- **Ecosystem-rich:** The Express team maintains middleware for body parsing, cookies, sessions, CORS, logging, and more.

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **Express fundamentals:** Routing, `app.use()`, and route handlers.
- **HTTP fundamentals:** Request methods, headers, status codes, and bodies.
- **JavaScript async/await:** Promises and asynchronous middleware patterns.

### Related Programming Areas

- **Express.js:** The most common framework for middleware-based architecture.
- **Security:** Helmet, CORS, rate limiting, and CSRF protection.
- **Logging:** Morgan, Pino, and Winston.
- **Validation:** Zod, Joi, and AJV.
- **Context Management:** AsyncLocalStorage and request-scoped state.

### Core Concepts

1. **Request Preprocessing** — body parsing, cookie/session, CORS.
2. **Authentication** — JWT, OAuth2, session validation, RBAC.
3. **Logging** — request/response logging, performance profiling, correlation IDs.
4. **Validation** — schema validation, parameter sanitization, payload limits.
5. **Transformation** — header normalization, response formatting, DTO mapping.
6. **Context and State Management** — AsyncLocalStorage, injecting user state.

---

## Core Concept 1: Request Preprocessing

### Sub-Feature 1.1: Body Parsing (JSON, URL-encoded, Multipart/Form-data)

#### Definitions

**Core Definition:** Body parsing middleware reads the raw request stream and converts it into a structured JavaScript object available on `req.body`.

**Technical Definition:** Express's built-in `express.json()` and `express.urlencoded()` middleware parse JSON and URL-encoded bodies respectively. For multipart/form-data (file uploads), `multer` is the standard middleware. The `body-parser` package provides the underlying implementation for Express's built-in parsers.

**Beginner-Friendly Explanation:** When a client sends data in a POST request, it arrives as a stream of bytes. Body parsing middleware collects those bytes and turns them into a usable JavaScript object like `{ "name": "Alice" }`.

#### Purposes

- To convert raw request bodies into structured data.
- To enforce payload size limits.
- To reject malformed bodies early.
- To support JSON, URL-encoded, and multipart formats.

#### Syntax Rules and Structure

```javascript
// JSON body parser
app.use(express.json({ limit: '100kb' }));

// URL-encoded body parser
app.use(express.urlencoded({ extended: true, limit: '100kb' }));

// Multipart (file uploads)
const multer = require('multer');
const upload = multer({ limits: { fileSize: 10 * 1024 * 1024 } });
app.post('/upload', upload.single('file'), handler);
```

| Option | Description | Default |
|--------|-------------|---------|
| `limit` | Max body size. | `'100kb'` |
| `extended` | Use `qs` for rich objects/arrays. | `false` |
| `type` | Content-Type to match. | `'application/json'` |

**Constraints and Limitations:**
- `express.json()` must be registered before routes that read `req.body`.
- The default 100kb limit must be increased for large payloads (e.g., `limit: '10mb'`).
- Body parsing must occur before validation middleware.

#### Annotated Code Example

```javascript
// body-parsing.js
const express = require('express');
const multer = require('multer');
const app = express();

// JSON parser
app.use(express.json({ limit: '100kb' }));

// URL-encoded parser
app.use(express.urlencoded({ extended: true }));

// Multipart parser
const upload = multer({ limits: { fileSize: 5 * 1024 * 1024 } });

app.post('/api/users', (req, res) => {
  res.json({ received: req.body });
});

app.post('/api/upload', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file.originalname, size: req.file.size });
});

app.listen(3000);
```

**Expected Output (for `POST /api/users` with `{"name":"Alice"}`):**
```json
{"received":{"name":"Alice"}}
```

**Expected Output (for `POST /api/upload` with a file):**
```json
{"file":"photo.jpg","size":245760}
```

**Why this output:** `express.json()` parses the JSON body and populates `req.body`. `multer` parses the multipart form data and populates `req.file`.

---

### Sub-Feature 1.2: Cookie Parsing and Session Handling

#### Definitions

**Core Definition:** Cookie parsing middleware reads the `Cookie` header and populates `req.cookies`; session middleware establishes server-side or cookie-based sessions.

**Technical Definition:** `cookie-parser` parses the cookie header and populates `req.cookies` with an object keyed by cookie names. `cookie-session` establishes cookie-based sessions, while `express-session` provides server-side session storage.

**Beginner-Friendly Explanation:** Cookies are small pieces of data stored by the browser and sent with every request. Cookie parsing makes them accessible on `req.cookies`. Sessions use cookies to identify a user across multiple requests.

#### Purposes

- To read authentication tokens or session identifiers from cookies.
- To maintain user state across requests.
- To set and clear cookies securely.

#### Syntax Rules and Structure

```javascript
const cookieParser = require('cookie-parser');
const session = require('express-session');

app.use(cookieParser('secret'));
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: true, httpOnly: true, sameSite: 'strict' },
}));
```

**Constraints and Limitations:**
- `cookie-parser` must be registered before routes that read `req.cookies`.
- Session secrets must be stored in environment variables.
- `secure: true` requires HTTPS.

#### Annotated Code Example

```javascript
// cookie-session.js
const express = require('express');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const app = express();

app.use(cookieParser('secret'));
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: true,
  cookie: { httpOnly: true, sameSite: 'strict' },
}));

app.get('/login', (req, res) => {
  req.session.userId = 42;
  res.cookie('theme', 'dark', { httpOnly: true });
  res.json({ loggedIn: true });
});

app.get('/profile', (req, res) => {
  res.json({
    userId: req.session.userId,
    theme: req.cookies.theme,
  });
});

app.listen(3000);
```

**Expected Output (for `GET /login` then `GET /profile`):**
```json
{"userId":42,"theme":"dark"}
```

**Why this output:** The session middleware stores `userId` on the server and sets a session cookie. The cookie parser reads the `theme` cookie. Both are accessible in the profile route.

---

### Sub-Feature 1.3: CORS (Cross-Origin Resource Sharing) Configuration

#### Definitions

**Core Definition:** CORS middleware sets HTTP headers that tell browsers which origins are allowed to read responses from the API.

**Technical Definition:** The `cors` package enables cross-origin resource sharing with configurable origins, methods, headers, and credentials.

**Beginner-Friendly Explanation:** By default, browsers block JavaScript on one site from reading responses from another site. CORS middleware relaxes this restriction in a controlled way.

#### Purposes

- To allow legitimate cross-origin requests from trusted frontends.
- To handle preflight `OPTIONS` requests.
- To prevent unauthorized cross-origin access.

#### Syntax Rules and Structure

```javascript
const cors = require('cors');

app.use(cors({
  origin: ['https://app.example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

**Constraints and Limitations:**
- CORS is browser-enforced only; server-to-server calls ignore it.
- `origin: '*'` cannot be used with `credentials: true`.
- Reflecting the `Origin` header without an allowlist is a security risk.

#### Annotated Code Example

```javascript
// cors-config.js
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors({
  origin: ['https://app.example.com', 'https://admin.example.com'],
  credentials: true,
}));

app.get('/api/data', (req, res) => {
  res.json({ data: 'protected' });
});

app.listen(3000);
```

**Expected Output (for a request from `https://app.example.com`):**
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
```

**Why this output:** The CORS middleware checks the `Origin` header against the allowlist and sets the appropriate headers for allowed origins.

---

## Core Concept 2: Authentication

### Sub-Feature 2.1: Token Verification (JWT, OAuth2)

#### Definitions

**Core Definition:** Token verification middleware validates JSON Web Tokens (JWT) or OAuth2 access tokens and attaches the authenticated user to the request.

**Technical Definition:** JWT middleware extracts the token from the `Authorization` header, verifies its signature using a secret or public key, and decodes the payload. OAuth2 middleware validates access tokens against an authorization server or introspects them. The `@tundralibs/pact` library provides transport-agnostic authentication with JWT/HMAC tokens, OAuth2/OIDC login, and drop-in middleware for Express.

**Beginner-Friendly Explanation:** JWT is like a digitally signed ID card. The middleware checks the signature to make sure the card is genuine, then reads the user information from it.

#### Purposes

- To verify that requests come from authenticated users.
- To attach user identity to the request for downstream middleware.
- To reject requests with invalid or expired tokens.

#### Syntax Rules and Structure

```javascript
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
  const token = req.headers['authorization']?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
}

app.get('/protected', authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

**Constraints and Limitations:**
- JWT payloads are base64-encoded, not encrypted; do not store sensitive data.
- Token expiration must be checked (`exp` claim).
- Refresh token rotation is recommended for long-lived sessions.

#### Annotated Code Example

```javascript
// jwt-auth.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const SECRET = 'my-secret';

app.post('/login', (req, res) => {
  const token = jwt.sign({ userId: 42, role: 'admin' }, SECRET, { expiresIn: '1h' });
  res.json({ token });
});

function authenticate(req, res, next) {
  const token = req.headers['authorization']?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
}

app.get('/protected', authenticate, (req, res) => {
  res.json({ user: req.user });
});

app.listen(3000);
```

**Expected Output (for `GET /protected` with valid token):**
```json
{"user":{"userId":42,"role":"admin","iat":1712345678,"exp":1712349278}}
```

**Expected Output (for `GET /protected` without token):**
```json
{"error":"No token"}
```

**Why this output:** The `authenticate` middleware extracts and verifies the JWT, attaching the decoded payload to `req.user`. Invalid tokens return 403; missing tokens return 401.

---

### Sub-Feature 2.2: Session Validation

#### Definitions

**Core Definition:** Session validation middleware checks whether a request contains a valid session identifier and loads the associated user data.

**Technical Definition:** Session middleware (e.g., `express-session`) stores session data on the server and uses a cookie to identify the session. Validation middleware checks the session store for the user's data and attaches it to the request.

**Beginner-Friendly Explanation:** A session is like a coat check ticket. The server keeps your coat (user data) and gives you a ticket (session ID cookie). When you return with the ticket, the server retrieves your coat.

#### Purposes

- To maintain user login state across requests.
- To load user data from server-side storage.
- To invalidate sessions on logout.

#### Syntax Rules and Structure

```javascript
function requireSession(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  next();
}
```

**Constraints and Limitations:**
- Session stores must be shared across process instances in clustered deployments.
- Session cookies must be `HttpOnly`, `Secure`, and `SameSite`.

#### Annotated Code Example

```javascript
// session-validation.js
const express = require('express');
const session = require('express-session');
const app = express();

app.use(session({
  secret: 'secret',
  resave: false,
  saveUninitialized: false,
  cookie: { httpOnly: true, sameSite: 'strict' },
}));

app.post('/login', (req, res) => {
  req.session.userId = 42;
  res.json({ loggedIn: true });
});

function requireSession(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  next();
}

app.get('/dashboard', requireSession, (req, res) => {
  res.json({ userId: req.session.userId });
});

app.listen(3000);
```

**Expected Output (for `GET /dashboard` after login):**
```json
{"userId":42}
```

**Expected Output (for `GET /dashboard` without login):**
```json
{"error":"Not authenticated"}
```

**Why this output:** The session middleware loads the session from the cookie. `requireSession` checks for `userId` and rejects unauthenticated requests.

---

### Sub-Feature 2.3: Authorization and Role-Based Access Control (RBAC)

#### Definitions

**Core Definition:** RBAC middleware checks whether the authenticated user has the required role or permission to access a resource.

**Technical Definition:** After authentication, authorization middleware compares the user's role (stored in the JWT or session) against the roles allowed for the route. The `@tundralibs/pact` library supports RBAC over BigInt bitmask permissions.

**Beginner-Friendly Explanation:** Authentication tells you who the user is; authorization tells you what they're allowed to do. An admin can delete users; a regular user cannot.

#### Purposes

- To restrict access to resources based on user roles.
- To enforce the principle of least privilege.
- To prevent privilege escalation.

#### Syntax Rules and Structure

```javascript
function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

app.delete('/users/:id', authenticate, requireRole('admin'), handler);
```

**Constraints and Limitations:**
- Roles must be derived from the authenticated session, not from client-supplied data.
- Centralize authorization logic rather than scattering checks across handlers.

#### Annotated Code Example

```javascript
// rbac.js
const express = require('express');
const app = express();

// Simulated auth middleware
app.use((req, res, next) => {
  req.user = { id: 1, role: 'user' };
  next();
});

function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden: insufficient permissions' });
    }
    next();
  };
}

app.get('/admin', requireRole('admin'), (req, res) => {
  res.json({ admin: true });
});

app.get('/user', requireRole('user', 'admin'), (req, res) => {
  res.json({ user: true });
});

app.listen(3000);
```

**Expected Output (for `GET /admin` with role `user`):**
```json
{"error":"Forbidden: insufficient permissions"}
```

**Expected Output (for `GET /user` with role `user`):**
```json
{"user":true}
```

**Why this output:** The `requireRole` middleware checks the user's role against the allowed roles. Admin routes reject regular users; user routes accept both users and admins.

---

## Core Concept 3: Logging

### Sub-Feature 3.1: Request/Response Logging (HTTP Method, URL, Status Code)

#### Definitions

**Core Definition:** Logging middleware records details about incoming requests and outgoing responses, including method, URL, status code, and response time.

**Technical Definition:** Morgan is the standard HTTP request logger for Express, while Pino and Winston provide structured JSON logging. The `pino-http` middleware automatically logs request/response data with correlation IDs.

**Beginner-Friendly Explanation:** Logging middleware is like a security camera that records every visitor. It tracks who came in, what they did, and when they left.

#### Purposes

- To monitor API usage and detect anomalies.
- To debug issues by tracing request flows.
- To measure response times and identify slow endpoints.

#### Syntax Rules and Structure

```javascript
const morgan = require('morgan');
app.use(morgan('combined'));

// Or with Pino
const pino = require('pino');
const pinoHttp = require('pino-http');
app.use(pinoHttp({ logger: pino() }));
```

**Constraints and Limitations:**
- Logging must be registered early in the middleware stack to capture all requests.
- Sensitive data (passwords, tokens) must be redacted.

#### Annotated Code Example

```javascript
// logging.js
const express = require('express');
const morgan = require('morgan');
const app = express();

// Log all requests in combined format
app.use(morgan('combined'));

// Custom logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.url} ${res.statusCode} ${Date.now() - start}ms`);
  });
  next();
});

app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

app.listen(3000);
```

**Expected Output:**
```
::1 - - [15/Jan/2026:12:00:00 +0000] "GET /api/users HTTP/1.1" 200 12 "-" "curl/7.68.0"
GET /api/users 200 3ms
```

**Why this output:** Morgan logs the request in combined format. The custom middleware measures response time and logs a concise summary.

---

### Sub-Feature 3.2: Performance Profiling and Execution Timing

#### Definitions

**Core Definition:** Performance profiling middleware measures the time taken to process a request, often breaking it down by middleware stage.

**Technical Definition:** Middleware can record `Date.now()` at the start of a request and calculate the elapsed time in `res.on('finish')`. The `response-time` package provides this functionality out of the box.

**Beginner-Friendly Explanation:** Performance profiling is like a stopwatch that tells you how long each part of the request took.

#### Purposes

- To identify slow middleware or handlers.
- To monitor API performance over time.
- To detect performance regressions.

#### Syntax Rules and Structure

```javascript
app.use((req, res, next) => {
  req.startTime = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    console.log(`${req.method} ${req.url} took ${duration}ms`);
  });
  next();
});
```

**Constraints and Limitations:**
- Timing includes network latency, not just server processing.
- High-resolution timing requires `process.hrtime.bigint()`.

#### Annotated Code Example

```javascript
// performance-timing.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  const start = process.hrtime.bigint();
  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - start) / 1e6;
    res.setHeader('X-Response-Time', `${duration.toFixed(2)}ms`);
    console.log(`${req.method} ${req.url}: ${duration.toFixed(2)}ms`);
  });
  next();
});

app.get('/slow', (req, res) => {
  setTimeout(() => res.json({ done: true }), 500);
});

app.listen(3000);
```

**Expected Output:**
```
GET /slow: 502.34ms
```

**Response header:** `X-Response-Time: 502.34ms`

**Why this output:** The middleware records the start time with `process.hrtime.bigint()` (nanosecond precision), calculates the duration on `'finish'`, and sets a response header.

---

### Sub-Feature 3.3: Correlation IDs for Distributed Tracing

#### Definitions

**Core Definition:** Correlation ID middleware generates or propagates a unique identifier for each request, enabling tracing across microservices and log aggregation.

**Technical Definition:** The middleware checks for an existing `X-Correlation-ID` or `X-Request-ID` header (from an upstream service); if absent, it generates a UUID. The ID is attached to the request context via `AsyncLocalStorage` and included in all log messages.

**Beginner-Friendly Explanation:** A correlation ID is like a tracking number for a package. Every service that touches the request logs the same number, so you can trace the entire journey.

#### Purposes

- To correlate logs across distributed services.
- To debug requests that span multiple services.
- To enable end-to-end tracing in observability platforms.

#### Syntax Rules and Structure

```javascript
const { AsyncLocalStorage } = require('async_hooks');
const { v4: uuidv4 } = require('uuid');

const asyncLocalStorage = new AsyncLocalStorage();

app.use((req, res, next) => {
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  res.setHeader('x-correlation-id', correlationId);
  asyncLocalStorage.run({ correlationId }, () => next());
});
```

**Constraints and Limitations:**
- `AsyncLocalStorage` requires Node.js 12.17+.
- Correlation IDs must be propagated to downstream services via headers.

#### Annotated Code Example

```javascript
// correlation-id.js
const express = require('express');
const { AsyncLocalStorage } = require('async_hooks');
const { v4: uuidv4 } = require('uuid');
const app = express();

const asyncLocalStorage = new AsyncLocalStorage();

app.use((req, res, next) => {
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  res.setHeader('x-correlation-id', correlationId);
  asyncLocalStorage.run({ correlationId }, () => {
    console.log(`[${correlationId}] ${req.method} ${req.url}`);
    next();
  });
});

app.get('/api/data', (req, res) => {
  const { correlationId } = asyncLocalStorage.getStore();
  console.log(`[${correlationId}] Fetching data`);
  res.json({ correlationId });
});

app.listen(3000);
```

**Expected Output:**
```
[a1b2c3d4-...] GET /api/data
[a1b2c3d4-...] Fetching data
```

**Response body:**
```json
{"correlationId":"a1b2c3d4-..."}
```

**Why this output:** The middleware generates a correlation ID (or uses the incoming one), stores it in `AsyncLocalStorage`, and sets it as a response header. All logs within the request scope include the same ID.

---

## Core Concept 4: Validation

### Sub-Feature 4.1: Schema Validation (Zod, Joi, Built-in Decorators)

#### Definitions

**Core Definition:** Schema validation middleware verifies that request data matches a declared schema before it reaches business logic.

**Technical Definition:** Zod, Joi, and AJV are popular schema validation libraries. Zod is TypeScript-first with static type inference. Joi is mature with extensive built-in rules. AJV implements JSON Schema.

**Beginner-Friendly Explanation:** A schema is like a form with specific fields and rules. The validator checks that the data matches the form before processing it.

#### Purposes

- To reject malformed requests at the boundary.
- To provide consistent error messages.
- To enable TypeScript type inference.

#### Syntax Rules and Structure

```javascript
const { z } = require('zod');

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ errors: result.error.errors });
    }
    req.body = result.data;
    next();
  };
}
```

**Constraints and Limitations:**
- Validation must run after body parsing.
- Query parameters are strings; use `z.coerce` for type coercion.

#### Annotated Code Example

```javascript
// zod-validation.js
const express = require('express');
const { z } = require('zod');
const app = express();
app.use(express.json());

const createUserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  password: z.string().min(8),
});

function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.error.errors.map(e => ({
          path: e.path.join('.'),
          message: e.message,
        })),
      });
    }
    req.body = result.data;
    next();
  };
}

app.post('/users', validate(createUserSchema), (req, res) => {
  res.status(201).json({ created: true, user: req.body });
});

app.listen(3000);
```

**Expected Output (for valid body):**
```json
{"created":true,"user":{"username":"alice","email":"alice@example.com","password":"secure123"}}
```

**Expected Output (for invalid body):**
```json
{"error":"Validation failed","details":[{"path":"email","message":"Invalid email"}]}
```

**Why this output:** Zod validates the body against the schema. Invalid data returns structured error messages. Valid data is assigned to `req.body` and passed to the handler.

---

### Sub-Feature 4.2: Query Parameter and URL Parameter Sanitization

#### Definitions

**Core Definition:** Parameter sanitization ensures that URL and query parameters are safe and conform to expected types before use.

**Technical Definition:** Route parameters and query strings are always strings. Sanitization middleware coerces types (e.g., `parseInt`), validates formats (e.g., UUID), and rejects unsafe patterns (e.g., path traversal).

**Beginner-Friendly Explanation:** URL parameters come from users and can be malicious. Sanitization makes sure `?id=123` is a number, not `?id=DROP TABLE`.

#### Purposes

- To prevent injection attacks via URL parameters.
- To ensure numeric parameters are valid integers.
- To validate UUIDs and slugs.

#### Syntax Rules and Structure

```javascript
function sanitizeId(req, res, next) {
  const id = parseInt(req.params.id, 10);
  if (isNaN(id) || id < 1) {
    return res.status(400).json({ error: 'Invalid ID' });
  }
  req.params.id = id;
  next();
}
```

**Constraints and Limitations:**
- Always validate route parameters before database queries.
- Use `z.coerce.number()` for query parameter coercion.

#### Annotated Code Example

```javascript
// param-sanitization.js
const express = require('express');
const { z } = require('zod');
const app = express();

const paramsSchema = z.object({
  id: z.coerce.number().int().positive(),
});

const querySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
});

app.get('/users/:id',
  (req, res, next) => {
    const result = paramsSchema.safeParse(req.params);
    if (!result.success) return res.status(400).json({ error: 'Invalid ID' });
    req.params = result.data;
    next();
  },
  (req, res, next) => {
    const result = querySchema.safeParse(req.query);
    if (!result.success) return res.status(400).json({ error: 'Invalid query' });
    req.query = result.data;
    next();
  },
  (req, res) => {
    res.json({ userId: req.params.id, page: req.query.page, limit: req.query.limit });
  }
);

app.listen(3000);
```

**Expected Output (for `GET /users/42?page=2&limit=50`):**
```json
{"userId":42,"page":2,"limit":50}
```

**Expected Output (for `GET /users/abc`):**
```json
{"error":"Invalid ID"}
```

**Why this output:** Zod coerces the string parameters to numbers and validates constraints. Invalid parameters are rejected with 400.

---

### Sub-Feature 4.3: Request Payload Constraints (Size Limits)

#### Definitions

**Core Definition:** Payload size limit middleware rejects requests with bodies exceeding a configured maximum size, preventing denial-of-service.

**Technical Definition:** `express.json({ limit: '100kb' })` enforces a 100kb default limit. Requests exceeding the limit receive a 413 Payload Too Large error. Body size limits should be set at the parser level.

**Beginner-Friendly Explanation:** A payload limit is like a bouncer who only lets in people of a certain size. Anything bigger is turned away.

#### Purposes

- To prevent memory exhaustion from oversized payloads.
- To protect the event loop from parsing massive bodies.
- To enforce a predictable resource envelope.

#### Syntax Rules and Structure

```javascript
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ extended: true, limit: '100kb' }));

// Route-specific higher limit
app.post('/upload', express.json({ limit: '50mb' }), handler);
```

**Constraints and Limitations:**
- Global limits should be conservative; route-specific overrides for upload endpoints.
- The 413 response should be handled by error middleware.

#### Annotated Code Example

```javascript
// payload-limit.js
const express = require('express');
const app = express();

app.use(express.json({ limit: '100kb' }));

app.post('/api/data', (req, res) => {
  res.json({ received: true, size: JSON.stringify(req.body).length });
});

app.use((err, req, res, next) => {
  if (err.type === 'entity.too.large') {
    return res.status(413).json({ error: 'Payload Too Large', limit: err.limit });
  }
  next(err);
});

app.listen(3000);
```

**Expected Output (for a 200kb body):**
```http
HTTP/1.1 413 Payload Too Large

{"error":"Payload Too Large","limit":"100kb"}
```

**Why this output:** The parser rejects the oversized body before it reaches the route handler. The error middleware formats the 413 response.

---

## Core Concept 5: Transformation

### Sub-Feature 5.1: Header Normalization

#### Definitions

**Core Definition:** Header normalization middleware standardizes HTTP header names to a consistent case format (e.g., `Content-Type` instead of `content-type`).

**Technical Definition:** The `@middy/http-header-normalizer` middleware normalizes HTTP header names to their canonical format, useful when clients do not use canonical names. For Express, custom middleware can normalize headers to lowercase or title case.

**Beginner-Friendly Explanation:** Different clients send headers in different cases. Normalization makes them consistent so your code can reliably check `Content-Type` without worrying about `content-type`.

#### Purposes

- To ensure consistent header access regardless of client formatting.
- To simplify header-based logic.
- To comply with HTTP conventions.

#### Syntax Rules and Structure

```javascript
app.use((req, res, next) => {
  const normalized = {};
  for (const [key, value] of Object.entries(req.headers)) {
    normalized[key.toLowerCase()] = value;
  }
  req.headers = normalized;
  next();
});
```

**Constraints and Limitations:**
- Node.js already lowercases incoming header names.
- Normalization is primarily useful for outgoing requests or API Gateway payloads.

#### Annotated Code Example

```javascript
// header-normalization.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  // Normalize to lowercase
  const normalized = {};
  for (const [key, value] of Object.entries(req.headers)) {
    normalized[key.toLowerCase()] = value;
  }
  req.headers = normalized;
  next();
});

app.get('/api/data', (req, res) => {
  // Always safe to access lowercase
  const contentType = req.headers['content-type'];
  res.json({ contentType });
});

app.listen(3000);
```

**Expected Output (for a request with `Content-Type: application/json`):**
```json
{"contentType":"application/json"}
```

**Why this output:** The middleware normalizes all header names to lowercase, so `req.headers['content-type']` always works regardless of the original case.

---

### Sub-Feature 5.2: Response Formatting and Data Serialization

#### Definitions

**Core Definition:** Response formatting middleware intercepts and transforms the response body before it is sent to the client, ensuring a consistent envelope structure.

**Technical Definition:** Express does not have a built-in response body hook, but middleware can override `res.json` or `res.send` to wrap the payload. The `express-response-middleware` package provides this capability.

**Beginner-Friendly Explanation:** Response formatting is like wrapping every product in the same box before shipping. The client always knows what to expect.

#### Purposes

- To provide a consistent response envelope (`{ data, meta, error }`).
- To include metadata (pagination, timestamps).
- To standardize error responses.

#### Syntax Rules and Structure

```javascript
app.use((req, res, next) => {
  const originalJson = res.json.bind(res);
  res.json = (body) => {
    return originalJson({
      data: body,
      meta: { timestamp: new Date().toISOString() },
      error: null,
    });
  };
  next();
});
```

**Constraints and Limitations:**
- Overriding `res.json` affects all responses; use selectively.
- Error responses should use a separate formatting path.

#### Annotated Code Example

```javascript
// response-formatting.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  const originalJson = res.json.bind(res);
  res.json = (body) => {
    if (body.error) return originalJson(body);
    return originalJson({
      data: body,
      meta: { timestamp: new Date().toISOString(), requestId: req.headers['x-request-id'] },
      error: null,
    });
  };
  next();
});

app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.listen(3000);
```

**Expected Output:**
```json
{
  "data": [{ "id": 1, "name": "Alice" }],
  "meta": { "timestamp": "2026-01-15T12:00:00.000Z", "requestId": null },
  "error": null
}
```

**Why this output:** The middleware overrides `res.json` to wrap the response in a standard envelope. Error responses (if present) are passed through unchanged.

---

### Sub-Feature 5.3: DTO (Data Transfer Object) Mapping

#### Definitions

**Core Definition:** DTO mapping middleware transforms incoming request data into a structured DTO instance and serializes outgoing responses from entity objects into DTOs.

**Technical Definition:** DTOs standardise payload shapes as they cross HTTP boundaries. NestJS provides `ClassSerializerInterceptor` with `class-transformer` to apply DTO rules declaratively. In Express, DTO mapping can be implemented manually or with libraries like `class-transformer`.

**Beginner-Friendly Explanation:** A DTO is like a shipping manifest. It specifies exactly what data goes in and what data comes out, keeping internal details (like passwords) out of API responses.

#### Purposes

- To decouple internal models from API contracts.
- To prevent leaking sensitive fields.
- To standardise request and response shapes.

#### Syntax Rules and Structure

```javascript
class UserDTO {
  constructor(user) {
    this.id = user.id;
    this.name = user.name;
    // email and password are NOT included
  }
}

app.get('/users/:id', async (req, res) => {
  const user = await db.findUser(req.params.id);
  res.json(new UserDTO(user));
});
```

**Constraints and Limitations:**
- DTO mapping should happen at the boundary, not in business logic.
- Use `class-transformer` for automated mapping with decorators.

#### Annotated Code Example

```javascript
// dto-mapping.js
const express = require('express');
const app = express();
app.use(express.json());

// DTO for user responses (excludes password)
class UserResponseDTO {
  constructor(user) {
    this.id = user.id;
    this.name = user.name;
    this.email = user.email;
  }
}

// DTO for user creation
class CreateUserDTO {
  constructor(body) {
    this.name = body.name;
    this.email = body.email;
    this.password = body.password; // Will be hashed before storage
  }
}

// Simulated database
const users = [{ id: 1, name: 'Alice', email: 'alice@example.com', password: 'secret' }];

app.get('/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(new UserResponseDTO(user));
});

app.post('/users', (req, res) => {
  const dto = new CreateUserDTO(req.body);
  const user = { id: users.length + 1, ...dto };
  users.push(user);
  res.status(201).json(new UserResponseDTO(user));
});

app.listen(3000);
```

**Expected Output (for `GET /users/1`):**
```json
{"id":1,"name":"Alice","email":"alice@example.com"}
```
*(Password is excluded.)*

**Expected Output (for `POST /users` with `{"name":"Bob","email":"bob@example.com","password":"secret"}`):**
```json
{"id":2,"name":"Bob","email":"bob@example.com"}
```

**Why this output:** `UserResponseDTO` excludes the password field. `CreateUserDTO` structures the incoming request. The DTO mapping ensures sensitive data is never sent to clients.

---

## Core Concept 6: Context and State Management

### Sub-Feature 6.1: Request-Scoped Storage (AsyncLocalStorage)

#### Definitions

**Core Definition:** `AsyncLocalStorage` is a Node.js API that provides a way to store data that is scoped to the current asynchronous execution context, accessible anywhere in the call chain without explicitly passing it through function arguments.

**Technical Definition:** `AsyncLocalStorage` (from `async_hooks`) allows storing and retrieving data across asynchronous operations. Middleware wraps the `next()` function in `asyncLocalStorage.run(context, () => next())`, making the context available in all downstream middleware and handlers.

**Beginner-Friendly Explanation:** `AsyncLocalStorage` is like a backpack that follows you through every function call in a request. Any function can reach into the backpack to get the request ID or user data without needing it passed as a parameter.

#### Purposes

- To store request-scoped data (correlation IDs, user state) accessible anywhere.
- To avoid threading `req` through every function.
- To enable contextual logging.

#### Syntax Rules and Structure

```javascript
const { AsyncLocalStorage } = require('async_hooks');
const asyncLocalStorage = new AsyncLocalStorage();

app.use((req, res, next) => {
  asyncLocalStorage.run({ requestId: uuidv4() }, () => next());
});
```

**Constraints and Limitations:**
- Requires Node.js 12.17+.
- Cannot be used with callback-based APIs that escape the async context.
- Memory leaks can occur if the store holds large objects.

#### Annotated Code Example

```javascript
// async-local-storage.js
const express = require('express');
const { AsyncLocalStorage } = require('async_hooks');
const { v4: uuidv4 } = require('uuid');
const app = express();

const asyncLocalStorage = new AsyncLocalStorage();

app.use((req, res, next) => {
  const requestId = req.headers['x-request-id'] || uuidv4();
  asyncLocalStorage.run({ requestId }, () => next());
});

function getRequestId() {
  return asyncLocalStorage.getStore()?.requestId;
}

app.get('/api/data', (req, res) => {
  // Accessible without req
  console.log(`[${getRequestId()}] Processing request`);
  res.json({ requestId: getRequestId() });
});

app.listen(3000);
```

**Expected Output:**
```
[a1b2c3d4-...] Processing request
```

**Response body:**
```json
{"requestId":"a1b2c3d4-..."}
```

**Why this output:** The middleware stores the `requestId` in `AsyncLocalStorage`. The `getRequestId()` function retrieves it from the store without needing `req` passed in.

---

### Sub-Feature 6.2: Injecting User State and Database Instances into Request Context

#### Definitions

**Core Definition:** Middleware can inject authenticated user data and database connections into the request context, making them available to all downstream handlers.

**Technical Definition:** After authentication, middleware attaches `req.user` with the decoded token or session data. Database connections can be attached as `req.db` or stored in `AsyncLocalStorage` for access without threading.

**Beginner-Friendly Explanation:** Injection is like handing a waiter your coat check ticket and your menu at the same time. The waiter now has both the user information and the tools needed to serve the request.

#### Purposes

- To provide authenticated user data to route handlers.
- To share database connections or transactions across middleware.
- To decouple services from request objects.

#### Syntax Rules and Structure

```javascript
app.use(async (req, res, next) => {
  req.db = await pool.connect();
  res.on('finish', () => req.db.release());
  next();
});

app.use(authenticate); // Attaches req.user
```

**Constraints and Limitations:**
- Database connections must be released on response finish.
- User state must be derived from verified tokens, not client input.

#### Annotated Code Example

```javascript
// context-injection.js
const express = require('express');
const { AsyncLocalStorage } = require('async_hooks');
const app = express();

const asyncLocalStorage = new AsyncLocalStorage();

// Simulated database
const db = { query: async (sql) => ({ rows: [{ id: 1 }] }) };

// Middleware: inject database and user into context
app.use((req, res, next) => {
  const context = {
    db,
    user: { id: 42, name: 'Alice' },
  };
  asyncLocalStorage.run(context, () => next());
});

// Service function — no req parameter needed
async function getUsers() {
  const { db } = asyncLocalStorage.getStore();
  return db.query('SELECT * FROM users');
}

app.get('/users', async (req, res) => {
  const users = await getUsers();
  const { user } = asyncLocalStorage.getStore();
  res.json({ requestedBy: user.name, users: users.rows });
});

app.listen(3000);
```

**Expected Output:**
```json
{"requestedBy":"Alice","users":[{"id":1}]}
```

**Why this output:** The middleware injects the database and user into `AsyncLocalStorage`. The `getUsers()` service accesses the database without receiving `req` as a parameter.

---

## References

- Express Middleware — https://expressjs.com/en/resources/middleware.html
- Express Body Parser — https://expressjs.com/en/resources/middleware/body-parser.html
- Express Cookie Parser — https://expressjs.com/en/resources/middleware/cookie-parser.html
- Express CORS — https://expressjs.com/en/resources/middleware/cors.html
- Express Session — https://expressjs.com/en/resources/middleware/session.html
- Helmet.js — https://helmet.js.org/
- Pino — https://getpino.io/
- Morgan — https://github.com/expressjs/morgan
- Zod — https://zod.dev/
- Joi — https://joi.dev/api/
- AJV — https://ajv.js.org/
- AsyncLocalStorage — https://nodejs.org/api/async_context.html#class-asynclocalstorage
- express-rate-limit — https://github.com/express-rate-limit/express-rate-limit
- @tundralibs/pact — https://jsr.io/@tundralibs/pact
- Middy HTTP Header Normalizer — https://middy.js.org/docs/middlewares/http-header-normalizer
- FreeCodeCamp — How to Assign Unique IDs to Express API Requests for Tracing — https://www.freecodecamp.org/news/how-to-assign-unique-ids-to-express-api-requests-for-tracing/
- Safeguard.sh — Broken Access Control in Express.js — https://safeguard.sh/resources/blog/broken-access-control-express-nodejs
- Safeguard.sh — Express Node.js Security Hardening Guide — https://safeguard.sh/resources/blog/express-js-security-hardening-guide
- Safeguard.sh — Securing Express.js Applications (Express 5) — 2026 Playbook — https://safeguard.sh/resources/blog/securing-expressjs-applications-express-5
- OpenReplay — Safe User Input Handling in Node.js — https://blog.openreplay.com/safe-user-input-handling-in-node-js
- NestJS Serialization — https://docs.nestjs.com/techniques/serialization
- Toptal — Node.js REST API Tutorial: Architecture, DTOs, and TypeScript — https://www.toptal.com/nodejs/nodejs-rest-api-tutorial
- Express Middleware Order Best Practices — https://raw.githubusercontent.com/expressjs/expressjs.com/34836ff41fcc44038d0f79c9f1908cf90f753880/en/guide/using-middleware.md