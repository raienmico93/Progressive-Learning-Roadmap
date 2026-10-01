# Express.js Custom Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Custom middleware functions are user-defined functions that have access to the request object (`req`), the response object (`res`), and the next middleware function (`next`) in the application's request-response cycle. They enable developers to extend Express's functionality with domain-specific logic.

**Technical Definition:** Middleware functions are functions that have access to the request object, the response object, and the next function in the application's request-response cycle. The next function is a function in the Express router which, when invoked, executes the middleware succeeding the current middleware. If the current middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function. Otherwise, the request will be left hanging. Custom middleware follows the same signature as built-in middleware but is authored by the developer to address specific application requirements.

**Beginner-Friendly Explanation:** Think of an Express application as an assembly line. Built-in middleware are the standard stations that come with the factory. Custom middleware are the specialised stations you build yourself — a station that logs every request, a station that checks ID badges, a station that cleans up the data before it moves forward. Each custom station does one specific job and then passes the work along the line.

### Key Characteristics

- **User-defined:** Written by the developer to address specific application needs.
- **Standard signature:** All middleware receive `(req, res, next)`.
- **Composable:** Custom middleware can be chained with built-in and third-party middleware.
- **Configurable:** Can accept options via factory functions that return middleware.
- **Reusable:** Can be exported from modules and applied across multiple routes or applications.
- **Order-dependent:** Execute in the order they are registered.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, closures, and asynchronous programming.
- **Understanding of Express middleware fundamentals:** req, res, next, and the middleware chain.

### Related Programming Areas

- **Authentication and authorisation:** Verifying identity and checking permissions.
- **Validation:** Sanitising and validating user input.
- **Logging and observability:** Recording request metadata for debugging and monitoring.
- **Error handling:** Catching and formatting errors before they reach the client.
- **Rate limiting:** Throttling requests based on client identity.

### Core Concepts

1. **Request Logging** — writing custom logs with timestamps, methods, URLs, and performance metrics.
2. **Authentication Checks** — verifying identity via sessions or JWTs.
3. **Authorization** — role-based access control and permission checks.
4. **Validation** — sanitising and validating user input with Joi, Zod, or Express-Validator.
5. **Request Transformation** — modifying data (normalising emails, adding request IDs).
6. **Response Processing** — intercepting or formatting responses before they are sent.
7. **Configurable Middleware** — factory functions that accept options and return middleware.

---

## Core Concept 1: Request Logging

### Definitions

**Core Definition:** Request logging middleware captures information about incoming HTTP requests — such as timestamps, HTTP methods, URLs, and response times — and writes it to a log destination for debugging, monitoring, and auditing.

**Technical Definition:** Logging middleware executes at the beginning of the middleware chain and typically attaches a listener to the response's `finish` event to measure the total request duration. It can use external logging libraries (Winston, Pino, Morgan) or write directly to the console. The middleware calls `next()` immediately after setting up the logging logic so that subsequent middleware and route handlers can execute.

**Beginner-Friendly Explanation:** Request logging is like a security camera that records every visitor to a building — when they arrived, what they did, and how long they stayed. This record is invaluable when something goes wrong and you need to trace what happened.

### Purposes

- To record request metadata (timestamp, HTTP method, URL) for debugging and auditing.
- To measure response times and identify performance bottlenecks.
- To integrate with external logging systems (Winston, Pino, Datadog).
- To provide a trail of activity for security monitoring.
- To correlate requests across distributed systems via request IDs.

### Syntax Rules and Structure

```js
function requestLogger(req, res, next) {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${new Date().toISOString()} ${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`);
  });
  next();
}
```

| Component | Breakdown |
|-----------|-----------|
| `req.method` | HTTP method (GET, POST, etc.). |
| `req.originalUrl` | Full URL including query string. |
| `res.statusCode` | HTTP status code set by the handler. |
| `res.on('finish')` | Event emitted when the response is fully sent. |
| `next()` | Passes control to the next middleware immediately. |

**Rules:**
- Call `next()` immediately after setting up the logging logic so the request proceeds.
- Use `res.on('finish')` to capture the final status code and duration.
- Use `req.originalUrl` (not `req.url`) to get the full URL, especially when routers rewrite `req.url`.
- For production, use a structured logging library instead of `console.log`.

**Constraints and Limitations:**
- Logging every request can generate significant I/O; consider sampling in high-traffic environments.
- Never log sensitive data (passwords, tokens, credit card numbers).
- Synchronous logging blocks the event loop; use asynchronous logging libraries.

### Annotated Code Example

```js
// request-logger.js
function requestLogger(req, res, next) {
  const start = Date.now();

  // Capture the final status and duration when the response finishes
  res.on('finish', () => {
    const duration = Date.now() - start;
    const logEntry = {
      timestamp: new Date().toISOString(),
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
      userAgent: req.get('User-Agent')
    };
    console.log(JSON.stringify(logEntry));
  });

  next(); // Proceed immediately
}

// Usage
const express = require('express');
const app = express();

app.use(requestLogger);

app.get('/api/data', (req, res) => {
  res.json({ data: 'example' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/data`):**
```
{"timestamp":"2026-01-15T10:30:00.000Z","method":"GET","url":"/api/data","status":200,"duration":"3ms","ip":"::1","userAgent":"curl/8.0"}
```

**Why this output:** The middleware attaches a `finish` listener before calling `next()`. When the route handler sends the JSON response, the `finish` event fires, and the listener calculates the duration, captures the final status code (200), and logs the structured entry.

### Real-World Cases

- **Production monitoring:** Feeding request logs into ELK Stack (Elasticsearch, Logstash, Kibana) or Datadog for analysis.
- **Performance debugging:** Identifying slow endpoints by analysing response times.
- **Security auditing:** Tracking access to sensitive endpoints for compliance (HIPAA, GDPR).
- **Incident response:** Tracing the sequence of requests that led to a failure.

---

## Core Concept 2: Authentication Checks

### Definitions

**Core Definition:** Authentication middleware verifies the identity of the client making the request by validating credentials — such as JWT tokens, session cookies, or API keys — before allowing the request to proceed to protected routes.

**Technical Definition:** Authentication middleware extracts credentials from the request (typically the `Authorization` header or a cookie), validates them against a trusted source, and attaches the authenticated user's information to the request object (e.g., `req.user`). If validation fails, the middleware sends a 401 Unauthorized response and does not call `next()`.

**Beginner-Friendly Explanation:** Authentication middleware is like a bouncer at a club door. Before anyone enters, the bouncer checks their ID (credentials). If the ID is valid, the person is allowed in and gets a wristband (the user object attached to the request). If not, they are turned away.

### Purposes

- To verify client identity before allowing access to protected routes.
- To extract and decode JWT tokens from the `Authorization` header.
- To validate session cookies and load the associated user.
- To attach the authenticated user object to `req.user` for downstream use.
- To reject unauthenticated requests with a 401 status.

### Syntax Rules and Structure

```js
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  const token = authHeader.split(' ')[1];
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `req.headers.authorization` | The header containing the bearer token. |
| `jwt.verify()` | Validates the token signature and expiration. |
| `req.user` | Custom property holding the decoded user data. |
| `next()` | Proceeds only if authentication succeeds. |

**Rules:**
- Always use `Bearer ` prefix in the `Authorization` header convention.
- Store the JWT secret in an environment variable, never in source code.
- Attach the decoded payload to `req.user` for downstream middleware.
- Handle expired and invalid tokens with clear 401 responses.
- In Express 5, async middleware automatically forwards rejections to error handlers.

**Constraints and Limitations:**
- JWT tokens cannot be revoked without a blacklist or token versioning system.
- Session-based authentication requires server-side session storage.
- Authentication middleware should be mounted only on routes that require it.

### Annotated Code Example

```js
// auth-middleware.js
const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

const JWT_SECRET = process.env.JWT_SECRET || 'dev-secret';

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      error: 'Authentication required',
      message: 'Provide a Bearer token in the Authorization header'
    });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = { id: decoded.userId, role: decoded.role };
    next();
  } catch (err) {
    const message = err.name === 'TokenExpiredError'
      ? 'Token has expired'
      : 'Invalid token';
    return res.status(401).json({ error: message });
  }
}

// Protected route
app.get('/api/profile', authenticate, (req, res) => {
  res.json({ userId: req.user.id, role: req.user.role });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/profile` with a valid token):**
```json
{"userId":1,"role":"admin"}
```

**Expected Output (for `GET /api/profile` without token):**
```json
{"error":"Authentication required","message":"Provide a Bearer token in the Authorization header"}
```

**Why this output:** The middleware extracts the token from the `Authorization` header. If present, `jwt.verify()` validates the signature and expiration. On success, the decoded payload is attached to `req.user`, and `next()` is called. On failure, a 401 response is sent without calling `next()`.

### Real-World Cases

- **REST APIs:** Protecting endpoints that require a logged-in user.
- **Single-page applications:** Validating JWTs issued by the SPA's login flow.
- **Mobile apps:** Validating tokens from iOS/Android clients.
- **Microservices:** Verifying tokens issued by an identity provider (Auth0, Keycloak).

---

## Core Concept 3: Authorization (RBAC)

### Definitions

**Core Definition:** Authorization middleware determines what an authenticated user is allowed to do by checking their role or permissions against the requirements of the requested resource.

**Technical Definition:** Role-Based Access Control (RBAC) middleware runs after authentication and compares the user's role (stored in `req.user.role` or a permissions array) against an allow-list of roles required by the route. If the user's role is not permitted, the middleware sends a 403 Forbidden response. If permitted, it calls `next()`.

**Beginner-Friendly Explanation:** Authorization is like the difference between a building's front door and individual office doors. Authentication gets you into the building (you have an ID badge), but authorization determines which offices you can enter. A regular employee can enter the break room, but only the CEO can enter the executive suite.

### Purposes

- To enforce role-based access control (RBAC) across protected routes.
- To check permissions before allowing access to sensitive resources.
- To prevent privilege escalation by users with insufficient roles.
- To centralise authorization logic in reusable middleware.

### Syntax Rules and Structure

```js
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}
```

| Component | Breakdown |
|-----------|-----------|
| `req.user.role` | The authenticated user's role (set by auth middleware). |
| `roles` | Allow-list of roles that can access the route. |
| `403` | Status code for authenticated but unauthorized requests. |
| `next()` | Proceeds only if the role is permitted. |

**Rules:**
- Authorization middleware must run **after** authentication middleware.
- Use a 401 status when the user is not authenticated; use 403 when authenticated but lacking permissions.
- Define roles as constants to avoid typos.
- Consider a permissions-based system (more granular than roles) for complex applications.

**Constraints and Limitations:**
- Role hierarchies (e.g., admin inherits editor permissions) require additional logic.
- Roles stored in JWTs become stale if the user's role changes before the token expires.
- Overly broad roles can create security gaps.

### Annotated Code Example

```js
// rbac-middleware.js
const express = require('express');
const app = express();

// Simulated auth middleware (sets req.user)
app.use((req, res, next) => {
  const role = req.headers['x-user-role'] || 'guest';
  req.user = { id: 1, role };
  next();
});

// Role-checking middleware factory
function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Requires one of: ${roles.join(', ')}`,
        yourRole: req.user.role
      });
    }
    next();
  };
}

// Routes with different role requirements
app.get('/api/content', requireRole('user', 'editor', 'admin'), (req, res) => {
  res.json({ content: 'Public content', accessedBy: req.user.role });
});

app.post('/api/content', requireRole('editor', 'admin'), (req, res) => {
  res.json({ message: 'Content created', accessedBy: req.user.role });
});

app.delete('/api/content/:id', requireRole('admin'), (req, res) => {
  res.json({ message: 'Content deleted', accessedBy: req.user.role });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/content` with `X-User-Role: user`):**
```json
{"content":"Public content","accessedBy":"user"}
```

**Expected Output (for `DELETE /api/content/1` with `X-User-Role: user`):**
```json
{"error":"Forbidden","message":"Requires one of: admin","yourRole":"user"}
```

**Why this output:** The `requireRole` factory returns middleware configured with the allowed roles. When a user with role `user` attempts to access the admin-only DELETE endpoint, the middleware rejects the request with 403 and a descriptive message.

### Real-World Cases

- **Content management systems:** Editors can create posts; admins can delete them.
- **E-commerce admin panels:** Only admins can access revenue analytics.
- **SaaS platforms:** Free-tier users have read-only access; paid users can write.
- **Healthcare systems:** Only doctors can access patient records; nurses can view schedules.

---

## Core Concept 4: Validation

### Definitions

**Core Definition:** Validation middleware sanitises and validates incoming request data (body, query, params) against a predefined schema, rejecting malformed or malicious input before it reaches the route handler.

**Technical Definition:** Validation middleware uses a schema definition (from Joi, Zod, or express-validator) to validate and sanitise `req.body`, `req.query`, and `req.params`. If validation fails, the middleware returns a 422 Unprocessable Entity or 400 Bad Request response with detailed error information. If validation succeeds, the sanitised data is attached to the request (e.g., `req.valid`) for use by the handler.

**Beginner-Friendly Explanation:** Validation middleware is like a quality control inspector on an assembly line. Before any product (request data) moves to the next station, the inspector checks it against a specification. If it meets the spec, it passes through. If not, it's rejected with a detailed report of what went wrong.

### Purposes

- To sanitise and validate user input before it reaches business logic.
- To prevent injection attacks (SQL injection, XSS) by rejecting malicious input.
- To enforce data integrity by ensuring required fields are present and correctly typed.
- To provide consistent, structured error responses for invalid input.

### Sub-Feature 4.1: Validation with Zod

```js
const { z } = require('zod');

const CreateUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(1).max(100),
  age: z.number().int().positive().optional()
});

function validate(schema) {
  return (req, res, next) => {
    const parsed = schema.safeParse(req.body);
    if (!parsed.success) {
      return res.status(422).json({
        error: 'Validation failed',
        details: parsed.error.flatten()
      });
    }
    req.valid = parsed.data;
    next();
  };
}
```

### Sub-Feature 4.2: Validation with Joi

```js
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  name: Joi.string().min(1).max(100).required()
});

function validateJoi(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });
    if (error) {
      return res.status(422).json({
        error: 'Validation failed',
        details: error.details.map(d => d.message)
      });
    }
    req.valid = value;
    next();
  };
}
```

### Annotated Code Example

```js
// validation-middleware.js
const express = require('express');
const { z } = require('zod');
const app = express();
app.use(express.json());

// Zod schema
const CreateUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  name: z.string().min(1, 'Name is required').max(100)
});

// Generic validation middleware factory
function validate(schema) {
  return (req, res, next) => {
    const parsed = schema.safeParse(req.body);
    if (!parsed.success) {
      return res.status(422).json({
        error: 'Validation failed',
        details: parsed.error.flatten().fieldErrors
      });
    }
    req.valid = parsed.data; // Sanitised data
    next();
  };
}

app.post('/api/users', validate(CreateUserSchema), (req, res) => {
  // req.valid is typed and sanitised
  res.status(201).json({
    message: 'User created',
    user: req.valid
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with valid data):**
```json
{
  "message": "User created",
  "user": {
    "email": "alice@example.com",
    "password": "securepass123",
    "name": "Alice"
  }
}
```

**Expected Output (for `POST /api/users` with invalid email and short password):**
```json
{
  "error": "Validation failed",
  "details": {
    "email": ["Invalid email format"],
    "password": ["Password must be at least 8 characters"]
  }
}
```

**Why this output:** The Zod schema validates the request body. If validation fails, the middleware returns 422 with field-level error messages. If validation succeeds, the sanitised data is attached to `req.valid` and the handler uses it.

### Real-World Cases

- **User registration:** Validating email format and password strength.
- **E-commerce orders:** Validating product IDs, quantities, and shipping addresses.
- **API gateways:** Validating all incoming requests before routing to services.
- **Form submissions:** Ensuring required fields are present and correctly formatted.

---

## Core Concept 5: Request Transformation

### Definitions

**Core Definition:** Request transformation middleware modifies the incoming request data — such as normalising email addresses, trimming whitespace, or adding unique request identifiers — before it reaches the route handler.

**Technical Definition:** Transformation middleware runs early in the middleware chain and mutates `req.body`, `req.query`, `req.params`, or adds custom properties to `req`. It does not validate or reject data; its purpose is to normalise and enrich the request so that downstream handlers receive consistent, clean data.

**Beginner-Friendly Explanation:** Request transformation is like a translator and organiser at a front desk. Before a visitor (the request) meets with the manager (the route handler), the front desk translates their request into a standard format and attaches a visitor badge (request ID). This ensures the manager always gets consistent, well-organised information.

### Purposes

- To modify data (e.g., normalising emails, trimming whitespace).
- To add unique request IDs via UUID for tracing.
- To attach metadata (timestamps, client information) to the request.
- To standardise data formats before validation or processing.

### Syntax Rules and Structure

```js
const crypto = require('crypto');

function requestTransformer(req, res, next) {
  // Normalise email
  if (req.body.email) {
    req.body.email = req.body.email.trim().toLowerCase();
  }

  // Add unique request ID
  req.requestId = req.get('x-request-id') || crypto.randomUUID();
  res.set('x-request-id', req.requestId);

  next();
}
```

| Component | Breakdown |
|-----------|-----------|
| `req.body.email` | The email field to normalise. |
| `req.get('x-request-id')` | Reads an incoming request ID header. |
| `crypto.randomUUID()` | Generates a UUID v4 if no ID is provided. |
| `res.set()` | Sets the request ID in the response headers. |

**Rules:**
- Transformations should be idempotent (running them twice has no additional effect).
- Preserve existing request IDs from upstream services.
- Use `crypto.randomUUID()` (Node.js v14.17+) for UUID generation.
- Register transformation middleware early, before validation and route handlers.

**Constraints and Limitations:**
- Modifying `req.body` requires body-parsing middleware to run first.
- Be careful not to overwrite existing data unintentionally.
- Avoid heavy transformations (e.g., database lookups) in transformation middleware.

### Annotated Code Example

```js
// request-transformer.js
const express = require('express');
const crypto = require('crypto');
const app = express();
app.use(express.json());

function requestTransformer(req, res, next) {
  // 1. Normalise email
  if (req.body && req.body.email) {
    req.body.email = req.body.email.trim().toLowerCase();
  }

  // 2. Trim all string fields in body
  if (req.body) {
    for (const key of Object.keys(req.body)) {
      if (typeof req.body[key] === 'string') {
        req.body[key] = req.body[key].trim();
      }
    }
  }

  // 3. Add unique request ID
  req.requestId = req.get('x-request-id') || crypto.randomUUID();
  res.set('x-request-id', req.requestId);

  // 4. Add request timestamp
  req.requestTime = new Date().toISOString();

  next();
}

app.use(requestTransformer);

app.post('/api/users', (req, res) => {
  res.json({
    requestId: req.requestId,
    requestTime: req.requestTime,
    user: req.body
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with `{ "email": "  Alice@Example.COM  ", "name": "  Alice  " }`):**
```json
{
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "requestTime": "2026-01-15T10:30:00.000Z",
  "user": {
    "email": "alice@example.com",
    "name": "Alice"
  }
}
```

**Why this output:** The middleware normalises the email to lowercase and trims whitespace from the name. It adds a UUID request ID and a timestamp to the request object. The route handler returns the transformed data along with the metadata.

### Real-World Cases

- **User registration:** Normalising emails before storing them in the database.
- **API tracing:** Adding request IDs to correlate logs across microservices.
- **Data imports:** Trimming and standardising CSV field values.
- **Internationalisation:** Normalising phone numbers and addresses to a standard format.

---

## Core Concept 6: Response Processing

### Definitions

**Core Definition:** Response processing middleware intercepts the outgoing response — modifying its body, headers, or status code — before it is sent to the client.

**Technical Definition:** Response processing middleware wraps the response's send methods (`res.send`, `res.json`, `res.end`) or listens for the `finish` event to inspect and modify the response. It can standardise response envelopes, add metadata (timestamps, request IDs), redact sensitive fields, or transform data formats.

**Beginner-Friendly Explanation:** Response processing middleware is like a quality control inspector at the end of an assembly line. Before the product (response) is shipped to the customer, the inspector wraps it in standard packaging (response envelope), adds a label (metadata), and removes anything that shouldn't be there (redaction).

### Purposes

- To intercept and format responses before they are sent to the client.
- To add a consistent response envelope (data, meta, error) across all endpoints.
- To add metadata such as timestamps, request IDs, and version numbers.
- To redact or remove sensitive fields from responses.

### Syntax Rules and Structure

```js
function responseFormatter(req, res, next) {
  const originalJson = res.json.bind(res);

  res.json = (body) => {
    const formatted = {
      success: res.statusCode < 400,
      statusCode: res.statusCode,
      data: body,
      meta: {
        timestamp: new Date().toISOString(),
        requestId: req.requestId
      }
    };
    return originalJson(formatted);
  };

  next();
}
```

| Component | Breakdown |
|-----------|-----------|
| `res.json.bind(res)` | Preserves the original `this` context. |
| `res.json = (body) => {...}` | Overrides the original method. |
| `originalJson(formatted)` | Calls the original method with the formatted body. |

**Rules:**
- Always preserve a reference to the original method before overriding it.
- Use `.bind(res)` to ensure the original method has the correct context.
- Response processing middleware must be registered before route handlers.
- Test carefully to ensure the override does not break other response methods.

**Constraints and Limitations:**
- Overriding `res.json` affects all routes that use it; document this behaviour.
- Some libraries expect specific response shapes; ensure compatibility.
- Streaming responses (`res.write`/`res.end`) are not intercepted by `res.json` overrides.

### Annotated Code Example

```js
// response-formatter.js
const express = require('express');
const app = express();

function responseFormatter(req, res, next) {
  const originalJson = res.json.bind(res);

  res.json = (body) => {
    const isError = res.statusCode >= 400;
    const formatted = {
      success: !isError,
      statusCode: res.statusCode,
      ...(isError ? { error: body } : { data: body }),
      meta: {
        timestamp: new Date().toISOString(),
        path: req.originalUrl
      }
    };
    return originalJson(formatted);
  };

  next();
}

app.use(responseFormatter);

app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.get('/api/error', (req, res) => {
  res.status(500).json({ message: 'Something went wrong' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/users`):**
```json
{
  "success": true,
  "statusCode": 200,
  "data": [{ "id": 1, "name": "Alice" }],
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "path": "/api/users"
  }
}
```

**Expected Output (for `GET /api/error`):**
```json
{
  "success": false,
  "statusCode": 500,
  "error": { "message": "Something went wrong" },
  "meta": {
    "timestamp": "2026-01-15T10:30:00.000Z",
    "path": "/api/error"
  }
}
```

**Why this output:** The middleware overrides `res.json` to wrap the response body in a standard envelope. For success responses (status < 400), the body is placed in `data`. For error responses (status >= 400), the body is placed in `error`. Metadata is added to both.

### Real-World Cases

- **API standardisation:** Ensuring all endpoints return the same envelope structure.
- **Response redaction:** Removing sensitive fields (password hashes, internal IDs) before sending.
- **Audit logging:** Adding timestamps and request IDs to every response for traceability.
- **Multi-tenant APIs:** Adding tenant-specific metadata to responses.

---

## Core Concept 7: Configurable Middleware (Factory Functions)

### Definitions

**Core Definition:** Configurable middleware is created by a factory function that accepts an options object and returns a middleware function. This pattern allows the same middleware logic to be reused with different configurations.

**Technical Definition:** A middleware factory is a higher-order function that closes over the configuration options and returns a middleware function with the standard `(req, res, next)` signature. The options are resolved once when the factory is called, and the returned middleware uses them for every request. This is the pattern used by Express's own built-in middleware (e.g., `express.json({ limit: '1mb' })`).

**Beginner-Friendly Explanation:** A configurable middleware factory is like a vending machine factory. You give the factory a set of specifications (options), and it builds you a vending machine (middleware) that operates according to those specifications. You can build multiple machines with different configurations — one for drinks, one for snacks — but they all share the same underlying mechanism.

### Purposes

- To create reusable middleware that can be configured for different routes.
- To avoid hardcoding values (limits, roles, messages) into middleware functions.
- To follow the Express convention for third-party middleware.
- To enable testing with different configurations.

### Syntax Rules and Structure

```js
function createMiddleware(options) {
  const { setting1, setting2, ...rest } = options;

  return (req, res, next) => {
    // Use setting1, setting2, rest
    next();
  };
}

// Usage
app.use(createMiddleware({ setting1: 'value', setting2: 100 }));
```

| Component | Breakdown |
|-----------|-----------|
| `createMiddleware(options)` | The factory function; accepts configuration. |
| `options` | Configuration object with defaults. |
| Returned function | The actual middleware with `(req, res, next)` signature. |

**Rules:**
- The factory function is called once during application setup.
- The returned middleware function is called for every request.
- Use destructuring with default values for optional configuration.
- State that should persist across requests (e.g., rate-limit counters) should be stored in the factory's closure.

**Constraints and Limitations:**
- Configuration is resolved at setup time; changing it requires restarting the application.
- Shared state in the closure applies to all routes that use the same middleware instance.
- Debugging can be harder because the middleware is created dynamically.

### Annotated Code Example

```js
// configurable-rate-limiter.js
const express = require('express');
const app = express();

function createRateLimiter(options) {
  const {
    windowMs = 60000,           // Default: 1 minute
    maxRequests = 100,          // Default: 100 requests
    message = 'Too many requests',
    keyGenerator = (req) => req.ip || 'unknown'
  } = options;

  const requestCounts = new Map(); // Shared state across requests

  return (req, res, next) => {
    const key = keyGenerator(req);
    const now = Date.now();
    let record = requestCounts.get(key);

    if (!record || now > record.resetAt) {
      record = { count: 0, resetAt: now + windowMs };
      requestCounts.set(key, record);
    }

    record.count++;

    res.setHeader('X-RateLimit-Limit', maxRequests);
    res.setHeader('X-RateLimit-Remaining', Math.max(0, maxRequests - record.count));

    if (record.count > maxRequests) {
      return res.status(429).json({ error: message });
    }

    next();
  };
}

// Different configurations for different routes
const apiLimiter = createRateLimiter({ windowMs: 60000, maxRequests: 100 });
const authLimiter = createRateLimiter({
  windowMs: 300000,
  maxRequests: 5,
  message: 'Too many login attempts. Try again later.',
  keyGenerator: (req) => `auth:${req.ip}`
});

app.use('/api/', apiLimiter);
app.use('/auth/', authLimiter);

app.get('/api/data', (req, res) => res.json({ data: 'ok' }));
app.post('/auth/login', (req, res) => res.json({ token: 'abc' }));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for the 6th request to `/auth/login` within 5 minutes):**
```json
{"error":"Too many login attempts. Try again later."}
```

**Expected Output (for a request to `/api/data`):**
```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
```

**Why this output:** The `createRateLimiter` factory accepts configuration and returns a middleware function. The `authLimiter` instance uses a different window, limit, message, and key generator than the `apiLimiter` instance. The shared state (the `requestCounts` Map) persists across requests within each middleware instance.

### Real-Aworld Cases

- **Rate limiting:** Different limits for API endpoints vs. authentication endpoints.
- **CORS:** Different allowed origins for different routes.
- **Caching:** Different TTL values for different resource types.
- **Logging:** Different log levels or formats for different environments.

---

## References

- Express.js — Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- Express.js — Writing Middleware — https://expressjs.com/en/guide/writing-middleware.html
- Express.js — Error Handling — https://expressjs.com/en/guide/error-handling.html
- Express.js 5.x — Express Object — https://expressjs.com/en/5x/api/express/
- Express.js — Middleware Modules — https://expressjs.com/en/resources/middleware.html
- Request Logging — CIS 526 Textbook — https://textbooks.cs.ksu.edu/cis526/x-examples/01-express-starter/05-request-log/tele.html
- JWT Authentication for Secure Node.js APIs — https://www.sourcetrail.com/javascript/implement-jwt-authentication-in-node-js-apis-like-a-pro/
- How to Implement Role-Based Access Control in a Node.js REST API with JWT — https://www.freecodecamp.org/news/role-based-access-control-nodejs-rest-api-jwt
- Validazione Input in Node.js: Zod, Joi, express-validator — https://shattered.io/it/validazione-input-nodejs/
- 8 Node.js Middleware Tricks That Simplified My Backend — https://medium.com/@bhagyarana80/8-node-js-middleware-tricks-that-simplified-my-backend-e86b7c49373b
- responseinterceptor — npm — https://www.npmjs.com/package/responseinterceptor
- How to Build Composable Middleware in Express — https://oneuptime.com/blog/post/2026-01-27-composable-middleware-express/view
- express-response-middleware — npm — https://www.npmjs.com/package/express-response-middleware
- express-mung — npm — https://www.npmjs.com/package/express-mung
- rbac-express-auth — npm — https://www.npmjs.com/package/rbac-express-auth
- simple-request-id — npm — https://www.npmjs.com/package/simple-request-id
- Morgan — npm — https://www.npmjs.com/package/morgan
- Winston — npm — https://www.npmjs.com/package/winston
- Zod — npm — https://www.npmjs.com/package/zod
- Joi — npm — https://www.npmjs.com/package/joi
- express-validator — npm — https://www.npmjs.com/package/express-validator