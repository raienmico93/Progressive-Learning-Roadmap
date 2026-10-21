# Express.js Custom Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Custom middleware is user-defined middleware written to perform application-specific tasks—such as authentication, logging, validation, or request transformation—that are not covered by Express's built-in or third-party middleware.

**Technical Definition**
A custom middleware is any function conforming to the Express middleware signature `(req, res, next)` (or `(err, req, res, next)` for error handlers) that is registered with `app.use()`, `router.use()`, or a route method. Custom middleware can read and modify `req` and `res`, attach data to the request for downstream handlers, terminate the request–response cycle, or call `next()` to delegate control. When packaged as a **middleware factory**—a function that accepts configuration options and returns a middleware function—custom middleware becomes reusable and configurable across routes and projects .

**Beginner-Friendly Explanation**
Custom middleware is middleware you write yourself. Express and npm give you many ready-made middleware, but every application has unique needs—like checking a specific API key, logging in your own format, or validating data against your own rules. You write a function that receives the request and response, does your custom logic, and then either sends a response or calls `next()` to move on .

---

### Key Characteristics

- **User-defined logic** — Custom middleware implements application-specific behavior that built-in and third-party middleware do not provide .
- **Single responsibility** — Well-designed custom middleware focuses on one task, making it composable and testable .
- **Factory-friendly** — The most reusable custom middleware is packaged as a factory function that accepts options and returns a configured middleware .
- **Attaches data to `req`** — Custom middleware commonly enriches the request with data (e.g., `req.user`, `req.requestId`) for downstream handlers .
- **Security-sensitive** — Custom middleware often handles authentication and validation, making correctness and input trust critical .

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- Understanding of the middleware signature `(req, res, next)` and the `next()` function.
- Familiarity with middleware ordering and the difference between application-level and router-level middleware.
- Basic knowledge of JavaScript closures (for factories) and Promises/`async`/`await` (for async middleware) .

---

### Related Programming Areas

- **Middleware architecture** — Custom middleware is one category alongside built-in, third-party, router-level, and error-handling middleware.
- **Authentication and authorization** — Custom guards (JWT verification, role checks) are among the most common custom middleware .
- **Validation** — Custom validation middleware (often factory-based) checks request bodies and parameters .
- **Logging and observability** — Custom loggers attach request IDs and measure response times .
- **Testing** — Custom middleware must be unit-testable in isolation using mock `req`/`res` objects or integration tests with Supertest .

---

## Core Concept 1: Custom Middleware

### Definitions

**Core Definition**
A custom middleware is a user-authored function that participates in the Express request–response pipeline to perform application-specific processing.

**Technical Definition**
A custom middleware is defined with the signature `(req, res, next)` and registered like any other middleware. It can (1) read from `req`—method, path, headers, body, params, query; (2) validate, transform, or authenticate; (3) optionally attach data to `req` for downstream handlers; and (4) call `next()` to continue, `next(err)` to signal an error, or send a response to terminate the cycle . Custom middleware should be small and focused on a single responsibility .

**Beginner-Friendly Explanation**
A custom middleware is just a function you write that runs during a request. It can look at the request, do something (like check a token or log the URL), maybe add information to the request for later, and then either send a response or say “next” to continue .

---

### Purposes

- To implement application-specific logic that built-in and third-party middleware do not provide.
- To attach verified data (e.g., `req.user`, `req.tenant`) to the request for downstream handlers .
- To enforce security checks such as authentication and authorization before routes execute .
- To perform custom logging, timing, or request enrichment .

---

### Syntax Rules and Structure

#### Complete General Syntax: Basic Custom Middleware

```javascript
const myMiddleware = (req, res, next) => {
  // 1. Read from req
  // 2. Validate, transform, or authenticate
  // 3. Optionally attach data to req: req.customData = ...
  // 4. Call next() or send a response
};
```

**Component Breakdown**

| Step | Role |
|------|------|
| Read from `req` | Access method, path, headers, body, params, query . |
| Process | Validate, transform, authenticate, or log. |
| Attach to `req` | Pass data to downstream handlers (e.g., `req.user`) . |
| `next()` / response | Continue the chain or end the cycle. |

#### Syntax Rules

- A custom middleware **must** either call `next()` or send a response; otherwise the request hangs .
- Data attached to `req` (e.g., `req.user`) is available to all downstream middleware and route handlers .
- **Never** trust client-supplied data (`req.body`, `req.params`, `req.query`) without validation .
- For error handling, call `next(err)`; for async operations, use `try/catch` and `next(error)` in Express 4, or rely on Express 5's automatic forwarding .

#### Constraints and Limitations

- Custom middleware that does not call `next()` or respond leaves the request pending indefinitely .
- Calling `next()` after sending a response causes `ERR_HTTP_HEADERS_SENT` errors .
- Middleware that reads `req.user` must be registered **after** the middleware that sets `req.user` .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Custom Request Logger with Request ID

```javascript
// server/src/middleware/requestLogger.js
const requestLogger = (req, res, next) => {
  const start = Date.now();

  // Attach a unique ID for log correlation
  req.requestId = `${Date.now()}-${Math.random().toString(36).slice(2, 8)}`;

  // Log when the response finishes, so we capture the status code
  res.on('finish', () => {
    const duration = Date.now() - start;
    const logLine = [
      new Date().toISOString(),
      req.requestId,
      req.method,
      req.originalUrl,
      res.statusCode,
      `${duration}ms`,
      req.ip
    ].join(' | ');

    if (res.statusCode >= 400) {
      console.error(logLine);
    } else {
      console.log(logLine);
    }
  });

  next(); // continue to the next middleware or route
};

module.exports = requestLogger;
```

**Step-by-Step Explanation**

1. **Timing** — `start` captures the request start time.
2. **Request ID** — A unique `req.requestId` is attached for correlation .
3. **Finish listener** — `res.on('finish')` runs after the response is sent, capturing the status code and duration .
4. **Delegation** — `next()` passes control to the next middleware.

**Expected Output**

```
GET /api/users
→ Console: 2026-01-01T00:00:00.000Z | 1672531200000-a1b2c3 | GET | /api/users | 200 | 5ms | ::1
→ 200 (user data)
```

**Why This Output Occurs**

- The middleware attaches a request ID and logs a structured line when the response completes .

---

### Real-World Cases

- **JWT authentication guards** — Custom middleware verifies tokens and attaches `req.user` .
- **Role-based access checks** — Factory-generated middleware checks `req.user.role` against a required role .
- **Tenant resolution** — Custom middleware extracts a subdomain and attaches tenant configuration to `req.tenant` .

---

## Core Concept 2: Middleware Factories

### Definitions

**Core Definition**
A middleware factory is a higher-order function that accepts configuration options and returns a middleware function, enabling reusable, configurable middleware.

**Technical Definition**
A middleware factory is a function that takes configuration arguments, closes over them via JavaScript closures, and returns a `(req, res, next)` middleware function . The outer function runs once at wire-up time (when the middleware is registered), while the inner function runs on every request . This pattern is used by `express.static()`, `helmet()`, `cors()`, and other ecosystem middleware . Factories enable one implementation to serve many configurations without copy-paste drift .

**Beginner-Friendly Explanation**
A middleware factory is a function that makes middleware. Instead of writing a separate “admin checker” and a separate “editor checker,” you write one factory that takes the role as an argument and returns the right middleware. The factory “locks in” your options so the middleware remembers them .

---

### Purposes

- To create reusable middleware that can be configured differently for different routes .
- To keep expensive setup (compiling schemas, reading keys) in the outer scope, executed once, while the inner function runs per request .
- To avoid copy-paste drift between similar middleware .
- To make middleware trivially unit-testable by calling the factory and then invoking the returned function with mock `req`/`res` .

---

### Syntax Rules and Structure

#### Complete General Syntax: Factory Pattern

```javascript
function myFactory(options) {
  // setup (runs once)
  return function myMiddleware(req, res, next) {
    // per-request logic, with access to options via closure
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Outer function | Accepts options; runs once at registration . |
| Closure | Captures options and setup state. |
| Inner function | The actual middleware, runs per request . |

#### Syntax Rules

- The outer function should **return** the middleware function .
- Options are captured via closure and remain accessible in the inner function .
- Expensive setup should be done in the outer function, not per request .

#### Constraints and Limitations

- State stored in the factory scope (e.g., a `Map` for caching) is shared across all requests using that middleware instance .
- Factory options should be validated at setup time to fail fast.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Role-Checking Factory

```javascript
// Factory: returns a middleware configured for a specific role
function requireRole(role) {
  return (req, res, next) => {
    if (req.user?.role !== role) {
      return next(new Error(`Role '${role}' required`));
    }
    next();
  };
}

// Usage with different configurations
app.get('/admin', requireRole('admin'), (req, res) => {
  res.send('Admin area');
});

app.get('/editor', requireRole('editor'), (req, res) => {
  res.send('Editor area');
});
```

**Expected Output**

```
GET /admin  (req.user.role = 'admin')  → 200 "Admin area"
GET /admin  (req.user.role = 'user')   → Error (role check fails)
GET /editor (req.user.role = 'editor') → 200 "Editor area"
```

**Why This Output Occurs**

- The factory closes over `role`, so each returned middleware checks for its own configured role .

---

#### Example 2: Configurable Rate Limiter Factory

```javascript
function createRateLimiter({ windowMs, maxRequests }) {
  const requestCounts = new Map(); // state lives in factory scope

  return (req, res, next) => {
    const key = req.ip;
    const now = Date.now();
    let record = requestCounts.get(key);

    if (!record || now > record.resetAt) {
      record = { count: 0, resetAt: now + windowMs };
      requestCounts.set(key, record);
    }

    record.count++;

    if (record.count > maxRequests) {
      return res.status(429).json({ error: 'Too many requests' });
    }

    next();
  };
}

// Different configurations for different routes
const apiLimiter = createRateLimiter({ windowMs: 60000, maxRequests: 100 });
const authLimiter = createRateLimiter({ windowMs: 300000, maxRequests: 5 });
```

**Expected Output**

```
GET /api/data  (101st request within 1 minute)  → 429 { "error": "Too many requests" }
POST /auth/login  (6th request within 5 minutes) → 429 { "error": "Too many requests" }
```

**Why This Output Occurs**

- Each factory call creates an independent rate-limiting state, so `apiLimiter` and `authLimiter` do not share counters .

---

### Real-World Cases

- **Schema validation** — `validate(schema)` factories that accept a Joi or Zod schema and return middleware .
- **Cache middleware** — `cache({ ttlMs, keyFn })` factories with per-instance `Map` state .
- **Timeout middleware** — A factory accepting a `ms` parameter so different route groups enforce different maximum response times .

---

## Core Concept 3: Middleware Options

### Definitions

**Core Definition**
Middleware options are the configuration values passed to a middleware factory that control the middleware's behavior, scope, and error handling.

**Technical Definition**
Options are typically passed as an object to the factory function. They may include values such as time windows, maximum counts, role names, header names, validation schemas, or callback functions. The factory closes over these options, and the returned middleware uses them on every request . Options should be validated at setup time to fail fast, and default values should be provided for optional settings .

**Beginner-Friendly Explanation**
Options are the settings you give a middleware factory. If you are making a rate limiter, options might be “how many requests” and “over how long.” The factory uses these settings to build the middleware .

---

### Purposes

- To customize middleware behavior for different routes or environments without duplicating code .
- To provide sensible defaults while allowing overrides .
- To centralize configuration in one place, improving maintainability .
- To enable environment-specific behavior (e.g., stricter limits in production).

---

### Syntax Rules and Structure

#### Complete General Syntax: Options Object with Defaults

```javascript
function myFactory({
  optionA = 'defaultA',
  optionB = 100,
  callback = () => {}
} = {}) {
  return (req, res, next) => {
    // use optionA, optionB, callback
  };
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| Destructuring with defaults | Provides fallback values for omitted options . |
| `= {}` | Allows the factory to be called without any options. |
| Closure | Captures validated options for the inner middleware . |

#### Syntax Rules

- Provide defaults for optional options; require essential options explicitly .
- Validate option types and ranges at setup time .
- Document the options contract for reuse across the codebase .

#### Constraints and Limitations

- Options captured by closure are fixed at factory call time; changing them later requires a new middleware instance.
- Overly broad options objects can make middleware harder to understand; keep options focused .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Header-Requirement Factory with Options

```javascript
function requireHeader(name, { message = `Missing ${name}`, status = 400 } = {}) {
  return (req, res, next) => {
    if (!req.get(name)) {
      return res.status(status).json({ error: message });
    }
    next();
  };
}

// Usage
app.use(requireHeader('x-request-id'));
app.use('/api', requireHeader('x-api-key', { status: 401, message: 'API key required' }));
```

**Expected Output**

```
GET /  (no x-request-id)  → 400 { "error": "Missing x-request-id" }
GET /  (with x-request-id) → passes to next middleware
GET /api  (no x-api-key)  → 401 { "error": "API key required" }
```

**Why This Output Occurs**

- The factory uses the `name`, `message`, and `status` options to build different header-checking middleware .

---

### Real-World Cases

- **CORS configuration** — `cors({ origin, methods, credentials })` uses options to control access .
- **Session configuration** — `session({ secret, cookie, store })` configures session behavior.
- **Validation factories** — `validate(schema, { abortEarly: false })` controls validation behavior .

---

## Core Concept 4: Middleware Configuration

### Definitions

**Core Definition**
Middleware configuration is the practice of supplying options and environment-specific settings to middleware factories, and of wiring middleware instances into the application at the correct positions in the stack.

**Technical Definition**
Middleware configuration encompasses: choosing option values per environment (development, staging, production); creating separate middleware instances for different route groups (e.g., API limiter vs. auth limiter); registering middleware in the correct order (security first, error handlers last); and ensuring configuration values (secrets, keys, origins) are supplied via environment variables rather than hard-coded. The factory pattern makes configuration explicit and testable .

**Beginner-Friendly Explanation**
Configuration is how you set up your middleware for different situations. You might want a strict rate limit on login routes and a loose one on public APIs. You configure each middleware instance differently and place them where they belong in the pipeline .

---

### Purposes

- To adapt middleware behavior to different environments and route groups .
- To keep secrets and environment-specific values out of source code.
- To ensure middleware is registered in the correct order for security and functionality .
- To make configuration explicit, documented, and testable .

---

### Syntax Rules and Structure

#### Complete General Syntax: Environment-Aware Configuration

```javascript
const config = {
  rateLimit: process.env.NODE_ENV === 'production'
    ? { windowMs: 60000, maxRequests: 100 }
    : { windowMs: 60000, maxRequests: 1000 }
};

app.use(createRateLimiter(config.rateLimit));
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Environment branching | Different option values per `NODE_ENV` . |
| Separate instances | `apiLimiter`, `authLimiter` configured independently . |
| Correct ordering | Security middleware first, error handlers last . |

#### Syntax Rules

- Supply secrets via `process.env`, never hard-code them .
- Create separate middleware instances for route groups with different requirements .
- Register error handlers last; register parsers before routes that need `req.body` .

#### Constraints and Limitations

- Middleware order cannot be changed at runtime; configuration must be set at wire-up time.
- Shared state in factory scope is per-process; distributed deployments may need external stores.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Environment-Aware Rate Limiting

```javascript
const isProduction = process.env.NODE_ENV === 'production';

const apiLimiter = createRateLimiter({
  windowMs: 60000,
  maxRequests: isProduction ? 100 : 1000
});

const authLimiter = createRateLimiter({
  windowMs: 300000,
  maxRequests: isProduction ? 5 : 50
});

app.use('/api', apiLimiter);
app.use('/auth', authLimiter);
```

**Expected Output**

```
# Production
GET /api/data  (101st request/minute)  → 429
POST /auth/login (6th request/5min)    → 429

# Development
GET /api/data  (101st request/minute)  → 200 (limit is 1000)
```

**Why This Output Occurs**

- The factory options differ by environment, so the same middleware behaves differently in production and development .

---

### Real-World Cases

- **Multi-environment deployments** — Strict limits in production, relaxed limits in development.
- **Multi-tenant SaaS** — Different rate limits or validation rules per tenant tier.
- **API versioning** — Different middleware configurations for `/v1` and `/v2` .

---

## Core Concept 5: Middleware Testing

### Definitions

**Core Definition**
Middleware testing is the practice of verifying that custom middleware behaves correctly in isolation and in integration with routes, covering success paths, failure paths, and edge cases.

**Technical Definition**
Middleware can be tested in two ways: **unit tests** invoke the middleware function directly with mock `req`, `res`, and `next` objects, asserting on calls to `next()`, status codes, and response payloads ; **integration tests** use Supertest to make HTTP requests against an Express app instance, verifying the middleware's behavior in the full pipeline . Factory-based middleware is especially easy to unit test: call the factory with options, then invoke the returned function with fakes . Separating the app configuration (`app.js`) from server startup (`server.js`) is essential for testability .

**Beginner-Friendly Explanation**
Testing middleware means checking that it does the right thing. You can test it by itself (unit test) by giving it fake request and response objects, or you can test it in a real Express app (integration test) by making HTTP requests and checking the responses .

---

### Purposes

- To catch bugs in authentication, validation, and error handling before deployment .
- To ensure middleware behaves correctly for edge cases (missing headers, invalid input, errors) .
- To make refactoring safe by locking in expected behavior .
- To document middleware behavior through executable tests .

---

### Syntax Rules and Structure

#### Complete General Syntax: Unit Testing a Middleware

```javascript
const middleware = require('./middleware');

it('calls next() when the header is present', () => {
  const req = { get: jest.fn().mockReturnValue('value') };
  const res = {};
  const next = jest.fn();

  middleware(req, res, next);

  expect(next).toHaveBeenCalledWith(); // success path
});

it('responds with 400 when the header is missing', () => {
  const req = { get: jest.fn().mockReturnValue(undefined) };
  const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };
  const next = jest.fn();

  middleware(req, res, next);

  expect(res.status).toHaveBeenCalledWith(400);
  expect(next).not.toHaveBeenCalled();
});
```

**Component Breakdown**

| Test | Assertion |
|------|-----------|
| Success path | `next` is called; response is not sent . |
| Failure path | Status code and response body are set; `next` is not called . |

#### Complete General Syntax: Integration Testing with Supertest

```javascript
const request = require('supertest');
const app = require('../app');

it('returns 401 without authentication', async () => {
  const response = await request(app).get('/api/protected').expect(401);
  expect(response.body.error).toBe('Unauthorized');
});
```

#### Syntax Rules

- Separate `app.js` (configuration) from `server.js` (listening) so tests can import the app without opening a port .
- Use mock `req`/`res` objects for unit tests; use Supertest for integration tests .
- Test both success and failure paths for every middleware .
- For factories, call the factory with options before invoking the middleware .

#### Constraints and Limitations

- Unit tests with mocks may not catch ordering or integration issues; combine with integration tests .
- Integration tests are slower but verify the full pipeline.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Unit Test for a Factory-Based Role Middleware

```javascript
// middleware/requireRole.js
function requireRole(role) {
  return (req, res, next) => {
    if (req.user?.role !== role) {
      return res.status(403).json({ error: `Role '${role}' required` });
    }
    next();
  };
}

// __tests__/requireRole.test.js
const requireRole = require('../middleware/requireRole');

describe('requireRole', () => {
  it('calls next when the role matches', () => {
    const middleware = requireRole('admin');
    const req = { user: { role: 'admin' } };
    const res = {};
    const next = jest.fn();

    middleware(req, res, next);

    expect(next).toHaveBeenCalled();
  });

  it('responds 403 when the role does not match', () => {
    const middleware = requireRole('admin');
    const req = { user: { role: 'user' } };
    const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };
    const next = jest.fn();

    middleware(req, res, next);

    expect(res.status).toHaveBeenCalledWith(403);
    expect(next).not.toHaveBeenCalled();
  });
});
```

**Expected Output**

```
PASS  __tests__/requireRole.test.js
  requireRole
    ✓ calls next when the role matches
    ✓ responds 403 when the role does not match
```

**Why This Output Occurs**

- The factory is called with `'admin'`, and the returned middleware is tested with matching and non-matching roles .

---

#### Example 2: Integration Test with Supertest

```javascript
// app.js
const express = require('express');
const app = express();
app.use(require('./middleware/requireHeader')('x-api-key'));
app.get('/api/data', (req, res) => res.json({ ok: true }));
module.exports = app;

// __tests__/app.test.js
const request = require('supertest');
const app = require('../app');

it('returns 400 without the required header', async () => {
  const response = await request(app).get('/api/data').expect(400);
  expect(response.body.error).toContain('x-api-key');
});

it('returns 200 with the required header', async () => {
  await request(app).get('/api/data').set('x-api-key', 'secret').expect(200);
});
```

**Expected Output**

```
PASS  __tests__/app.test.js
  ✓ returns 400 without the required header
  ✓ returns 200 with the required header
```

**Why This Output Occurs**

- Supertest sends real HTTP requests to the app, verifying the middleware in the full pipeline .

---

### Real-World Cases

- **Authentication middleware** — Unit-testing JWT verification and `req.user` attachment.
- **Validation factories** — Testing that invalid input produces `400` responses .
- **Rate limiters** — Integration-testing that the `429` response is returned after the limit is exceeded.

---

## Core Concept 6: Custom Middleware Best Practices

### Definitions

**Core Definition**
Custom middleware best practices are the design, security, and testing patterns that keep user-authored middleware focused, reusable, secure, and maintainable.

**Technical Definition**
Best practices include: writing **small, single-responsibility** middleware ; using **factory functions** for configurability and reuse ; attaching data to `req` rather than relying on globals; **validating all client-supplied input** and never trusting `req.body`, `req.params`, or `req.query` ; handling asynchronous errors correctly (Express 5 automatic forwarding or Express 4 wrappers) ; separating app configuration from server startup for testability ; and registering error handlers last.

**Beginner-Friendly Explanation**
Best practices are the habits that keep custom middleware safe and useful. Write one job per middleware, make it configurable with a factory, validate everything from the client, handle errors properly, and test it thoroughly .

---

### Purposes

- To prevent security vulnerabilities caused by trusting unvalidated client data .
- To maximize reuse and reduce duplication through factories .
- To ensure errors from asynchronous middleware reach centralized error handlers .
- To make middleware easy to test, reason about, and maintain .

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Custom Middleware

```javascript
// Factory with options, validation, and error forwarding
function requireHeader(name, { status = 400 } = {}) {
  return (req, res, next) => {
    const value = req.get(name);
    if (!value) {
      return res.status(status).json({ error: `Missing ${name}` });
    }
    req.headers[name] = value; // attach verified data if needed
    next();
  };
}
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Single responsibility | One middleware does one thing . |
| Factory | Configurable via options . |
| Input trust | Validate before using . |
| Error handling | `next(err)` or response; no hanging . |

#### Syntax Rules

- **Never** set `req.user` from `req.body`; always derive identity from a verified source (e.g., JWT) .
- Use `try/catch` with `next(error)` in Express 4 async middleware; rely on Express 5 automatic forwarding .
- Keep middleware small and focused; combine responsibilities makes reuse impossible .
- Separate `app.js` from `server.js` for testability .
- Register error-handling middleware last with four arguments .

#### Constraints and Limitations

- Over-broad middleware can create security blind spots; keep scope narrow.
- Stateful factories (e.g., in-memory caches) do not work across multiple processes; use external stores for distributed deployments.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Secure JWT Middleware Factory

```javascript
const jwt = require('jsonwebtoken');

function protect(secret) {
  return async (req, res, next) => {
    try {
      const authHeader = req.headers.authorization;
      if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
      }

      const token = authHeader.slice(7);
      const decoded = jwt.verify(token, secret);

      // Attach verified identity — never from req.body
      req.user = { id: decoded.sub, role: decoded.role };
      next();
    } catch (error) {
      next(error); // forward to centralized error handler
    }
  };
}

// Usage
app.use('/api', protect(process.env.JWT_SECRET));
```

**Expected Output**

```
GET /api/data  (valid Bearer token)  → passes to route, req.user set
GET /api/data  (no token)            → 401 { "error": "No token provided" }
GET /api/data  (invalid token)       → forwarded to error handler
```

**Why This Output Occurs**

- The factory closes over the secret; the middleware verifies the token and attaches a trusted identity .

---

### Real-World Cases

- **Production authentication** — JWT factories with environment-specific secrets.
- **Multi-tenant validation** — Factories that accept a schema and return validation middleware .
- **Observability middleware** — Request ID and timing middleware attached globally .

---

## References

- Writing middleware for use in Express apps · Express.js — https://expressjs.com/en/guide/writing-middleware.html
- Express.js Middleware Architecture Deep Dive (Grizzly Peak Software) — https://www.grizzlypeaksoftware.com/library/expressjs-middleware-architecture-deep-dive-p1qj1q08
- Writing Custom Middleware in Express (StackLesson) — https://www.stacklesson.com/mern-stack-tutorial/mern-middleware/ch07-lesson-04-writing-custom-middleware/
- How to Build Composable Middleware in Express (OneUptime) — https://oneuptime.com/blog/post/2026-01-27-composable-middleware-express/view
- Reusable, Configurable Middleware — Express (soundscode) — https://soundscode.com/express/ex-adv-reusable-middleware
- Writing Custom Middleware — Express (soundscode) — https://soundscode.com/express/ex-custom-middleware
- Express Middleware Patterns (LobeHub) — https://lobehub.com/skills/bradtaylorsf-alphaagent-team-express-middleware
- Express Error Handling (AppSignal Blog) — https://blog.appsignal.com/2025/07/16/express-for-node-error-handling-and-tracking-done-right.html
- Express API Error Handling (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/10-express-rest-api/8-express-api-error-handling.mdx
- Express.js Patterns and Best Practices (Shuffle-and-Sync) — https://github.com/Shuffle-and-Sync/reimagined-guacamole/blob/main/docs/guides/EXPRESS_PATTERNS.md
- Writing Unit Tests for an Express.js Application (tsecurity.de) — https://tsecurity.de/de/2516083/IT+Programmierung/Writing+Unit+Tests+for+an+Express.js+Application:+A+Beginner-Friendly+Guide/
- How to Write Tests for Express with Jest (OneUptime) — https://github.com/OneUptime/blog/blob/master/posts/2026-02-02-express-jest-testing/README.md
- Express 5 Error Handling (automatic async forwarding) — https://expressjs.com/en/guide/error-handling.html