# Error Handling — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Error handling in Node.js is the systematic process of detecting, propagating, classifying, and responding to errors that occur during the execution of a server-side application, ensuring reliability, observability, and a safe client experience.

**Technical Definition:** Error handling encompasses the mechanisms by which JavaScript errors are caught (synchronously via `try/catch` or asynchronously via callbacks, Promises, and `async/await`), propagated to a centralised handler, classified by type (operational vs. programming), and transformed into structured HTTP responses. Express provides a default error handler and supports custom error-handling middleware with a four-argument signature `(err, req, res, next)`. Node.js exposes process-level events `'uncaughtException'` and `'unhandledRejection'` for errors that escape the application boundary.

**Beginner-Friendly Explanation:** Error handling is like a safety net for your application. When something goes wrong — a database query fails, a user sends bad data, or a bug causes a crash — the error handling system catches it, decides what to do, logs it for debugging, and sends a safe message back to the user. Without it, a single error could crash your entire server or expose sensitive information.

### Key Characteristics

- **Propagation paths:** Synchronous errors are caught by `try/catch`; asynchronous errors must be passed to `next()` or handled in a `.catch()`.
- **Centralised handling:** A single error-handling middleware processes all errors, avoiding scattered error logic.
- **Classification:** Errors are classified as operational (expected, recoverable) or programming (bugs, unrecoverable).
- **Process-level safety:** `'uncaughtException'` and `'unhandledRejection'` handlers prevent silent crashes and enable graceful recovery.
- **Standardised responses:** RFC 7807 Problem Details provides a machine-readable JSON format for error responses.
- **Environment awareness:** Stack traces are shown in development but hidden in production to prevent information leakage.
- **Observability integration:** APM tools (Sentry, Datadog) capture errors with context (request ID, user ID, release).

### Prerequisites

- **Node.js runtime:** Node.js 18+ recommended.
- **Express fundamentals:** Middleware, routing, and `next()`.
- **JavaScript error handling:** `try/catch`, `Error` class, and Promises.
- **HTTP fundamentals:** Status codes, headers, and response bodies.

### Related Programming Areas

- **Express.js:** Error-handling middleware and the `next(err)` pattern.
- **Node.js Process:** `'uncaughtException'` and `'unhandledRejection'` events.
- **Observability:** Sentry, Datadog, Pino, and structured logging.
- **API Design:** RFC 7807 Problem Details and standardised error formats.
- **Cluster and PM2:** Process recycling and crash recovery.

### Core Concepts

1. **Error Propagation** — synchronous catching, asynchronous forwarding, and boundary catches.
2. **Centralised Handlers** — global error middleware, environment-specific logging, and APM integration.
3. **Operational Errors** — database timeouts, client errors, and graceful degradation.
4. **Programming Errors** — type errors, memory leaks, and process crash recovery.
5. **Error Classification** — custom error classes, HTTP status mapping, and domain-specific codes.
6. **Structured Error Responses** — standardised JSON payloads, stack trace hiding, and i18n.

---

## Core Concept 1: Error Propagation

### Sub-Feature 1.1: Synchronous Error Catching

#### Definitions

**Core Definition:** Synchronous errors are exceptions thrown within synchronous code blocks and caught using `try/catch` or, in Express, automatically caught by the framework.

**Technical Definition:** Errors that occur in synchronous code inside route handlers and middleware require no extra work — if synchronous code throws an error, Express will catch and process it. In raw Node.js, synchronous errors must be caught with `try/catch` or they will propagate up the call stack and potentially crash the process.

**Beginner-Friendly Explanation:** Synchronous errors are like dropping a plate in the kitchen — everyone hears it immediately. In Express, the framework is always listening and catches the plate before it hits the floor. In raw Node.js, you need your own safety net (`try/catch`).

#### Purposes

- To catch and handle errors immediately where they occur.
- To prevent synchronous exceptions from crashing the process.
- To leverage Express's built-in synchronous error catching.

#### Syntax Rules and Structure

```js
// Raw Node.js: try/catch required
try {
  const data = JSON.parse('invalid json');
} catch (err) {
  console.error('Parse error:', err.message);
}
```
```js
// Express: automatic catching (no try/catch needed)
app.get('/data', (req, res) => {
  throw new Error('Something broke'); // Express catches this
});
```

**Constraints and Limitations:**
- `try/catch` does not catch asynchronous errors from callbacks or Promises unless awaited.
- Express automatically catches synchronous throws in route handlers and middleware.

#### Annotated Code Example

```javascript
// sync-errors.js
const express = require('express');
const app = express();

// Synchronous throw — Express catches automatically
app.get('/sync-error', (req, res) => {
  throw new Error('Synchronous failure');
});

// try/catch in raw Node.js
app.get('/safe', (req, res, next) => {
  try {
    const data = JSON.parse('{"invalid": json}');
    res.json(data);
  } catch (err) {
    next(err); // Forward to error handler
  }
});

// Error-handling middleware
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

app.listen(3000);
```

**Expected Output (for `GET /sync-error`):**
```json
{"error":"Synchronous failure"}
```

**Expected Output (for `GET /safe`):**
```json
{"error":"Unexpected token j in JSON at position 11"}
```

**Why this output:** The `throw` in `/sync-error` is caught by Express and passed to the error middleware. The `try/catch` in `/safe` catches the JSON parse error and forwards it via `next(err)`. Both produce structured error responses.

### Sub-Feature 1.2: Asynchronous Error Forwarding (Promises, Async/Await, `next(err)`)

#### Definitions

**Core Definition:** Asynchronous errors must be explicitly forwarded to Express via `next(err)` or caught with `try/catch` (for `async/await`) because Express cannot automatically catch errors outside the synchronous call stack.

**Technical Definition:** For errors returned from asynchronous functions invoked by route handlers and middleware, you must pass them to the `next()` function, where Express will catch and process them. Starting with Express 5, route handlers and middleware that return a Promise will automatically call `next(value)` when they reject or throw an error. In Express 4, async errors must be wrapped in `try/catch` or an async wrapper utility.

**Beginner-Friendly Explanation:** Asynchronous errors are like a delayed package delivery problem — the delivery happens after the customer has left the store. Express can't automatically know about it unless you tell the delivery person to call you (`next(err)`). In Express 5, the delivery person is trained to automatically call you.

#### Purposes

- To forward async errors to the centralised error handler.
- To prevent unhandled Promise rejections from crashing the process.
- To provide consistent error handling across sync and async code.

#### Syntax Rules and Structure

```javascript
// Express 4: manual try/catch
app.get('/data', async (req, res, next) => {
  try {
    const data = await fetchData();
    res.json(data);
  } catch (err) {
    next(err); // Forward to error handler
  }
});

// Express 5: automatic forwarding
app.get('/data', async (req, res) => {
  const data = await fetchData(); // Rejection → next(err) automatically
  res.json(data);
});

// Async wrapper for Express 4
const catchAsync = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

**Constraints and Limitations:**
- Express 4 does not catch async errors automatically; you must use `try/catch` or a wrapper.
- Unhandled Promise rejections can crash the process in Node.js 22+.
- Always return after `next(err)` to avoid executing further code.

#### Annotated Code Example

```javascript
// async-errors.js
const express = require('express');
const app = express();

// Async wrapper for Express 4
const catchAsync = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Simulated async operation that fails
async function fetchUser(id) {
  if (id === '0') throw new Error('User not found');
  return { id, name: 'Alice' };
}

// Wrapped async route
app.get('/users/:id', catchAsync(async (req, res) => {
  const user = await fetchUser(req.params.id);
  res.json(user);
}));

// Error handler
app.use((err, req, res, next) => {
  res.status(404).json({ error: err.message });
});

app.listen(3000);
```

**Expected Output (for `GET /users/0`):**
```json
{"error":"User not found"}
```

**Expected Output (for `GET /users/42`):**
```json
{"id":"42","name":"Alice"}
```

**Why this output:** The `catchAsync` wrapper catches the rejected Promise and forwards it to `next(err)`. The error handler returns a 404 with the error message. Without the wrapper, the rejection would be unhandled.

### Sub-Feature 1.3: Boundary Catches and Unhandled Rejections

#### Definitions

**Core Definition:** Boundary catches are process-level handlers for errors that escape all application-level handling: `'uncaughtException'` and `'unhandledRejection'`.

**Technical Definition:** The `'uncaughtException'` event is emitted when an uncaught JavaScript exception bubbles all the way back to the event loop. The `'unhandledRejection'` event is emitted whenever a Promise is rejected and no error handler is attached within a turn of the event loop. `'uncaughtException'` is a crude mechanism intended only as a last resort — the correct use is to perform synchronous cleanup before shutting down.

**Beginner-Friendly Explanation:** Boundary catches are the last line of defence — like a fire alarm that goes off when all other safety measures have failed. They should not be used as a substitute for proper error handling, but they prevent the process from crashing silently.

#### Purposes

- To log fatal errors before the process exits.
- To perform synchronous cleanup of resources.
- To prevent silent crashes from unhandled rejections.
- To trigger alerts in monitoring systems.

#### Syntax Rules and Structure

```javascript
process.on('uncaughtException', (err, origin) => {
  console.error('Fatal error:', err);
  process.exit(1); // Graceful exit
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection:', reason);
  // Optionally exit or log
});
```

**Constraints and Limitations:**
- `'uncaughtException'` should not be used to resume normal operation; the application is in an undefined state.
- Only synchronous operations are safe in `'exit'` listeners.
- Node.js 22+ crashes on unhandled rejections by default.

#### Annotated Code Example

```javascript
// boundary-catches.js
const express = require('express');
const app = express();

// Process-level handlers
process.on('uncaughtException', (err, origin) => {
  console.error(`[${new Date().toISOString()}] UNCAUGHT: ${err.message} (origin: ${origin})`);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error(`[${new Date().toISOString()}] UNHANDLED REJECTION:`, reason);
});

app.get('/crash', (req, res) => {
  // Intentional unhandled rejection
  Promise.reject(new Error('Async failure'));
  res.json({ message: 'Rejection triggered' });
});

app.listen(3000);
```

**Expected Output (console):**
```
[2026-01-15T12:00:00.000Z] UNHANDLED REJECTION: Error: Async failure
```

**Why this output:** The `Promise.reject` in `/crash` is not handled, triggering the `'unhandledRejection'` event. The handler logs the error. In production, this handler would typically send the error to an APM tool and potentially trigger a graceful shutdown.

---

## Core Concept 2: Centralised Handlers

### Sub-Feature 2.1: Global Error-Catching Middleware

#### Definitions

**Core Definition:** Global error-handling middleware is a function with the four-argument signature `(err, req, res, next)` registered last in the middleware stack that catches all errors passed to `next(err)`.

**Technical Definition:** Error-handling middleware functions are defined in the same way as other middleware functions, except with four arguments instead of three: `(err, req, res, next)`. Express only treats a handler as an error handler when it has exactly four parameters. The middleware must be registered after all other middleware and routes.

**Beginner-Friendly Explanation:** Global error middleware is like a customer service desk at the exit of a store. No matter what went wrong inside, the customer service desk handles it politely and sends the customer on their way with a proper resolution.

#### Purposes

- To centralise error handling logic in one place.
- To provide consistent error responses across all endpoints.
- To log errors with request context.
- To hide internal details in production.

#### Syntax Rules and Structure

```javascript
app.use((err, req, res, next) => {
  // err: the error object
  // req: the request
  // res: the response
  // next: the next error handler
  const status = err.status || 500;
  res.status(status).json({
    error: err.message,
    code: err.code || 'INTERNAL_ERROR',
  });
});
```

**Constraints and Limitations:**
- The function must have exactly 4 arguments.
- Must be registered last.
- Calling `next(err)` inside an error handler passes to the next error handler.

#### Annotated Code Example

```javascript
// global-error-handler.js
const express = require('express');
const app = express();
app.use(express.json());

// Custom error class
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
  }
}

// Route that throws operational error
app.get('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  if (isNaN(id)) {
    throw new AppError('Invalid user ID', 400, 'INVALID_ID');
  }
  res.json({ userId: id });
});

// Route that throws programming error
app.get('/bug', (req, res) => {
  const obj = null;
  obj.nonexistentMethod(); // TypeError
});

// Global error handler
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  const response = {
    error: err.message,
    code: err.code || 'INTERNAL_ERROR',
  };

  // Hide stack traces in production
  if (process.env.NODE_ENV !== 'production') {
    response.stack = err.stack;
  }

  if (!err.isOperational) {
    console.error('PROGRAMMING ERROR:', err);
  }

  res.status(status).json(response);
});

app.listen(3000);
```

**Expected Output (for `GET /users/abc` in development):**
```json
{
  "error": "Invalid user ID",
  "code": "INVALID_ID",
  "stack": "AppError: Invalid user ID\n    at ..."
}
```

**Expected Output (for `GET /users/abc` in production):**
```json
{"error":"Invalid user ID","code":"INVALID_ID"}
```

**Expected Output (for `GET /bug` in production):**
```json
{"error":"Cannot read properties of null (reading 'nonexistentMethod')","code":"INTERNAL_ERROR"}
```

**Why this output:** The `AppError` is classified as operational and returns a 400 with a specific code. The TypeError is a programming error and returns a generic 500. Stack traces are included only in development.

### Sub-Feature 2.2: Environment-Specific Error Logging (Development vs. Production)

#### Definitions

**Core Definition:** Environment-specific logging adjusts the verbosity and content of error logs based on `NODE_ENV`, showing full stack traces and detailed context in development but suppressing sensitive information in production.

**Technical Definition:** When `NODE_ENV` is set to `development`, Express's default error handler writes the stack trace to the client. When `NODE_ENV` is set to `production`, the stack trace is not written — only the HTTP response code. Custom error handlers should follow the same principle: include `err.stack` in development responses, omit it in production, and always log the full error server-side.

**Beginner-Friendly Explanation:** In development, you want to see every detail of what went wrong. In production, you want to show a polite "something went wrong" message to users while logging the full details privately for your team.

#### Purposes

- To aid debugging in development with full stack traces.
- To prevent information leakage in production.
- To maintain detailed server-side logs while sending safe client responses.

#### Syntax Rules and Structure

```javascript
const isProduction = process.env.NODE_ENV === 'production';

app.use((err, req, res, next) => {
  // Always log full error server-side
  console.error(`[${req.method} ${req.url}]`, err);

  // Client response depends on environment
  const response = {
    error: err.message,
    statusCode: err.statusCode || 500,
  };

  if (!isProduction) {
    response.stack = err.stack;
  }

  res.status(err.statusCode || 500).json(response);
});
```

**Constraints and Limitations:**
- Never expose `err.stack` to clients in production.
- Always log the full error server-side for debugging.
- Use structured logging (JSON) for easier parsing in production.

#### Annotated Code Example

```javascript
// env-error-logging.js
const express = require('express');
const pino = require('pino');
const app = express();

const logger = pino({ level: process.env.LOG_LEVEL || 'info' });
const isProduction = process.env.NODE_ENV === 'production';

app.get('/error', (req, res) => {
  throw new Error('Database connection failed');
});

app.use((err, req, res, next) => {
  // Structured logging server-side
  logger.error({
    err: { message: err.message, stack: err.stack },
    req: { method: req.method, url: req.url, ip: req.ip },
  }, 'Request failed');

  // Client response
  const response = {
    error: isProduction ? 'Internal Server Error' : err.message,
  };

  if (!isProduction) {
    response.stack = err.stack;
    response.detail = 'Full error details available in development';
  }

  res.status(500).json(response);
});

app.listen(3000, () => {
  logger.info(`Server started (${isProduction ? 'production' : 'development'})`);
});
```

**Expected Output (development, `GET /error`):**
```json
{
  "error": "Database connection failed",
  "stack": "Error: Database connection failed\n    at ...",
  "detail": "Full error details available in development"
}
```

**Expected Output (production, `GET /error`):**
```json
{"error":"Internal Server Error"}
```

**Server log (both environments):**
```json
{"level":50,"time":1712345678901,"err":{"message":"Database connection failed","stack":"..."},"req":{"method":"GET","url":"/error","ip":"::1"},"msg":"Request failed"}
```

**Why this output:** In development, the client sees the actual error message and stack trace. In production, only a generic message is returned. The server log always includes the full error details, request metadata, and timestamp for debugging.

### Sub-Feature 2.3: Integration with External APM/Monitoring Tools (Sentry, Datadog)

#### Definitions

**Core Definition:** APM (Application Performance Monitoring) integration captures errors, traces, and metrics from the application and sends them to an external service (Sentry, Datadog) for aggregation, alerting, and analysis.

**Technical Definition:** Sentry and Datadog are complete platforms with their own agents and backends. Sentry excels at error monitoring and session replay. Datadog Error Tracking is ideal when logs, metrics, traces, and on-call workflow already live in Datadog. A useful path is: request middleware adds a correlation ID, the exception handler adds context, the capture API stores an event, and a triage view groups repeated failures.

**Beginner-Friendly Explanation:** APM tools are like a security camera system for your application. When something goes wrong, the tool records what happened, where, and for which user — and alerts your team if the problem is serious.

#### Purposes

- To aggregate and group similar errors across deployments.
- To provide stack traces with source map support.
- To correlate errors with user sessions and releases.
- To trigger alerts and on-call workflows.

#### Syntax Rules and Structure

**Sentry (Node.js):**
```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.APP_RELEASE,
});

// Capture exception in error handler
app.use((err, req, res, next) => {
  Sentry.captureException(err, {
    extra: { requestId: req.id, userId: req.user?.id },
  });
  res.status(500).json({ error: 'Internal Server Error' });
});
```

**Datadog (Node.js):**
```javascript
const tracer = require('dd-trace').init();

// Capture error in error handler
app.use((err, req, res, next) => {
  const span = tracer.scope().active();
  if (span) span.setTag('error', err);
  res.status(500).json({ error: 'Internal Server Error' });
});
```

**Constraints and Limitations:**
- APM tools add overhead; configure sampling rates appropriately.
- Source maps must be uploaded for readable stack traces.
- Never send sensitive data (passwords, tokens) to APM tools.

#### Annotated Code Example

```javascript
// apm-integration.js
const express = require('express');
const crypto = require('crypto');
const app = express();

// Sentry initialization (simulated)
const Sentry = {
  captureException(err, context) {
    console.log('[SENTRY] Capturing:', err.message, '| Context:', JSON.stringify(context));
  },
};

// Request ID middleware
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || crypto.randomUUID();
  res.setHeader('x-request-id', req.id);
  next();
});

app.get('/api/data', (req, res) => {
  throw new Error('Database timeout');
});

app.use((err, req, res, next) => {
  Sentry.captureException(err, {
    extra: {
      requestId: req.id,
      method: req.method,
      url: req.url,
      userId: req.user?.id,
    },
  });

  res.status(500).json({
    error: 'Internal Server Error',
    requestId: req.id,
  });
});

app.listen(3000);
```

**Expected Output (console):**
```
[SENTRY] Capturing: Database timeout | Context: {"extra":{"requestId":"a1b2c3d4-...","method":"GET","url":"/api/data"}}
```

**Response body:**
```json
{"error":"Internal Server Error","requestId":"a1b2c3d4-..."}
```

**Why this output:** The request ID middleware attaches a unique ID to each request. When an error occurs, `Sentry.captureException` sends the error and context to Sentry. The client receives a generic error message with the request ID for support correlation.

---

## Core Concept 3: Operational Errors

### Sub-Feature 3.1: Database Timeouts and Network Failures

#### Definitions

**Core Definition:** Operational errors are expected runtime problems — database timeouts, network failures, and external service unavailability — that are part of normal operation and can be handled gracefully.

**Technical Definition:** Operational errors are errors that are expected to happen in the normal course of operations. They are not bugs but rather conditions that the application must handle. Examples include database connection failures, network timeouts, and third-party API errors. These should be distinguished from programming errors using an `isOperational` flag.

**Beginner-Friendly Explanation:** Operational errors are like a traffic jam — they happen, they're annoying, but they're not a design flaw. You plan for them, detect them, and handle them gracefully. Programming errors are like a car with faulty brakes — a design flaw that needs fixing.

#### Purposes

- To handle transient failures with retries or fallbacks.
- To return appropriate HTTP status codes (503, 504) to clients.
- To prevent operational errors from being treated as bugs.
- To enable graceful degradation when external services are unavailable.

#### Syntax Rules and Structure

```javascript
class DatabaseError extends Error {
  constructor(message) {
    super(message);
    this.name = 'DatabaseError';
    this.isOperational = true;
    this.statusCode = 503;
  }
}

try {
  await db.query('SELECT ...');
} catch (err) {
  if (err.code === 'ECONNREFUSED') {
    throw new DatabaseError('Database unavailable');
  }
  throw err;
}
```

**Constraints and Limitations:**
- Always distinguish operational errors from programming errors.
- Retry logic should be bounded (e.g., 3 attempts with exponential backoff).
- Circuit breakers prevent cascading failures.

#### Annotated Code Example

```javascript
// operational-errors.js
const express = require('express');
const app = express();

class DatabaseError extends Error {
  constructor(message) {
    super(message);
    this.name = 'DatabaseError';
    this.isOperational = true;
    this.statusCode = 503;
    this.code = 'DATABASE_UNAVAILABLE';
  }
}

class TimeoutError extends Error {
  constructor(message) {
    super(message);
    this.name = 'TimeoutError';
    this.isOperational = true;
    this.statusCode = 504;
    this.code = 'GATEWAY_TIMEOUT';
  }
}

// Simulated database with retry logic
async function queryWithRetry(query, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      // Simulate failure
      if (Math.random() < 0.7) throw new Error('ECONNREFUSED');
      return { rows: [{ id: 1 }] };
    } catch (err) {
      if (attempt === maxRetries) {
        throw new DatabaseError(`Database failed after ${maxRetries} attempts`);
      }
      await new Promise(r => setTimeout(r, 100 * attempt));
    }
  }
}

app.get('/api/data', async (req, res, next) => {
  try {
    const result = await queryWithRetry('SELECT * FROM data');
    res.json(result);
  } catch (err) {
    next(err);
  }
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
    });
  }
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```

**Expected Output (when database fails all retries):**
```json
{"error":"Database failed after 3 attempts","code":"DATABASE_UNAVAILABLE"}
```

**HTTP status:** 503 Service Unavailable

**Why this output:** The `queryWithRetry` function attempts the query three times with exponential backoff. If all attempts fail, it throws a `DatabaseError` with `isOperational: true` and `statusCode: 503`. The error handler recognises the operational error and returns the appropriate status code.

### Sub-Feature 3.2: Client Errors (4xx Series) and Resource Shortages

#### Definitions

**Core Definition:** Client errors (4xx) indicate that the request contains bad syntax or cannot be fulfilled, while resource shortages (5xx) indicate server-side failures such as memory exhaustion or connection pool depletion.

**Technical Definition:** HTTP 4xx status codes indicate client errors: 400 Bad Request (malformed syntax), 401 Unauthorized (missing authentication), 403 Forbidden (insufficient permissions), 404 Not Found (resource doesn't exist), 409 Conflict (state conflict), 422 Unprocessable Entity (validation failure). HTTP 5xx status codes indicate server errors: 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable (resource shortage), 504 Gateway Timeout.

**Beginner-Friendly Explanation:** 4xx errors are the client's fault — they sent something wrong. 5xx errors are the server's fault — something on our end is broken or overloaded. The distinction matters because clients can fix 4xx errors but can only retry 5xx errors.

#### Purposes

- To communicate the nature of the error to the client.
- To enable appropriate client-side handling (retry vs. fix request).
- To distinguish between user errors and server failures.
- To support standardised error handling across the API.

#### Syntax Rules and Structure

| Status Code | Meaning | Use Case |
|-------------|---------|----------|
| 400 | Bad Request | Malformed JSON, invalid syntax. |
| 401 | Unauthorized | Missing or invalid token. |
| 403 | Forbidden | Authenticated but insufficient permissions. |
| 404 | Not Found | Resource doesn't exist. |
| 409 | Conflict | Duplicate resource, state conflict. |
| 422 | Unprocessable Entity | Validation failed. |
| 429 | Too Many Requests | Rate limit exceeded. |
| 503 | Service Unavailable | Database down, resource shortage. |
| 504 | Gateway Timeout | Upstream timeout. |

**Constraints and Limitations:**
- 4xx errors should include actionable information (which field is invalid).
- 5xx errors should not expose internal details.
- 429 should include a `Retry-After` header.

#### Annotated Code Example

```javascript
// client-server-errors.js
const express = require('express');
const app = express();
app.use(express.json());

// 400 Bad Request
app.post('/users', (req, res) => {
  if (!req.body.name) {
    return res.status(400).json({
      error: 'Bad Request',
      detail: 'name is required',
      code: 'MISSING_FIELD',
    });
  }
  res.status(201).json({ id: 1, name: req.body.name });
});

// 401 Unauthorized
app.get('/protected', (req, res) => {
  if (!req.headers.authorization) {
    return res.status(401).json({
      error: 'Unauthorized',
      detail: 'Authorization header is required',
    });
  }
  res.json({ data: 'protected' });
});

// 503 Service Unavailable
app.get('/db-status', (req, res) => {
  const dbHealthy = false;
  if (!dbHealthy) {
    return res.status(503).json({
      error: 'Service Unavailable',
      detail: 'Database connection pool exhausted',
      retryAfter: 30,
    });
  }
  res.json({ status: 'healthy' });
});

app.listen(3000);
```

**Expected Output (for `POST /users` without name):**
```json
{"error":"Bad Request","detail":"name is required","code":"MISSING_FIELD"}
```

**Expected Output (for `GET /db-status`):**
```json
{"error":"Service Unavailable","detail":"Database connection pool exhausted","retryAfter":30}
```

**HTTP status:** 503 Service Unavailable

**Why this output:** The 400 error includes the specific field that's missing and a machine-readable code. The 503 error includes a `retryAfter` hint. Both follow the structured error response pattern.

### Sub-Feature 3.3: Graceful Degradation Strategies

#### Definitions

**Core Definition:** Graceful degradation is the practice of allowing an application to continue operating at reduced functionality when a non-critical service fails, rather than failing completely.

**Technical Definition:** Graceful degradation involves implementing fallbacks for non-critical dependencies. If the recommendation service is down, show popular items instead. If the database read fails, serve cached data. The key is to identify which services are critical (cannot fail) and which are non-critical (can degrade).

**Beginner-Friendly Explanation:** Graceful degradation is like a car with a flat tyre that can still drive on a temporary spare. The car isn't at full performance, but it keeps moving. In an application, if the image service is down, you show a placeholder; if the database is slow, you serve cached results.

#### Purposes

- To maintain core functionality when non-critical services fail.
- To improve user experience during partial outages.
- To prevent cascading failures across services.
- To buy time for recovery.

#### Syntax Rules and Structure

```javascript
async function getRecommendations(userId) {
  try {
    return await recommendationService.get(userId);
  } catch (err) {
    // Fallback: return popular items
    console.warn('Recommendation service failed, using fallback');
    return await getPopularItems();
  }
}
```

**Constraints and Limitations:**
- Fallback data must be appropriate (e.g., don't show stale prices for purchases).
- Degradation should be logged and alerted.
- Circuit breakers can automate the degradation decision.

#### Annotated Code Example

```javascript
// graceful-degradation.js
const express = require('express');
const app = express();

// Simulated services
const recommendationService = {
  get: async (userId) => { throw new Error('Service down'); },
};

const popularItems = [{ id: 1, name: 'Popular Item' }];

async function getRecommendations(userId) {
  try {
    return await recommendationService.get(userId);
  } catch (err) {
    console.warn('[DEGRADED] Recommendation service failed:', err.message);
    return { source: 'fallback', items: popularItems };
  }
}

app.get('/recommendations/:userId', async (req, res) => {
  const recommendations = await getRecommendations(req.params.userId);
  res.json(recommendations);
});

app.listen(3000);
```

**Expected Output (console):**
```
[DEGRADED] Recommendation service failed: Service down
```

**Response body:**
```json
{"source":"fallback","items":[{"id":1,"name":"Popular Item"}]}
```

**Why this output:** The `getRecommendations` function attempts to call the recommendation service. When it fails, it logs a warning and returns fallback data. The client receives a valid response with a `source` field indicating the data is from a fallback.

---

## Core Concept 4: Programming Errors

### Sub-Feature 4.1: Type Errors, Null Pointers, and Syntax Exceptions

#### Definitions

**Core Definition:** Programming errors are bugs in the code — type errors, null pointer dereferences, and syntax exceptions — that are not expected and cannot be handled gracefully by the application.

**Technical Definition:** Programming errors are errors that are not expected to happen in the normal course of operations. They are bugs in the code that need to be fixed. Examples include `TypeError` (calling a method on `undefined`), `ReferenceError` (using an undeclared variable), and `SyntaxError` (invalid JavaScript). These errors indicate that the application is in an undefined state and should be allowed to crash (after logging).

**Beginner-Friendly Explanation:** Programming errors are like a car with a broken engine — it's not a traffic jam, it's a mechanical failure. You can't fix it on the road; you need to take it to the mechanic. Similarly, programming errors need to be fixed in the code, not handled at runtime.

#### Purposes

- To identify and fix bugs in the code.
- To prevent silent failures that corrupt data.
- To trigger alerts for immediate investigation.
- To restart the process in a clean state.

#### Syntax Rules and Structure

| Error Type | Cause | Example |
|------------|-------|---------|
| `TypeError` | Calling method on null/undefined | `null.toString()` |
| `ReferenceError` | Using undeclared variable | `console.log(undefinedVar)` |
| `SyntaxError` | Invalid JavaScript | `JSON.parse('{bad}')` |
| `RangeError` | Value out of range | `new Array(-1)` |

**Constraints and Limitations:**
- Programming errors should not be caught and swallowed.
- The process should be restarted after a programming error.
- `'uncaughtException'` is the last resort for programming errors.

#### Annotated Code Example

```javascript
// programming-errors.js
const express = require('express');
const app = express();

app.get('/bug/type', (req, res) => {
  const obj = null;
  obj.method(); // TypeError: Cannot read properties of null
});

app.get('/bug/ref', (req, res) => {
  console.log(undefinedVariable); // ReferenceError
});

app.get('/bug/range', (req, res) => {
  const arr = new Array(-1); // RangeError: Invalid array length
});

// Error handler classifies programming errors
app.use((err, req, res, next) => {
  const isProgrammingError = !err.isOperational;

  if (isProgrammingError) {
    console.error('PROGRAMMING ERROR (crash-worthy):', err);
    // In production, trigger graceful shutdown
  }

  res.status(500).json({
    error: 'Internal Server Error',
    code: isProgrammingError ? 'PROGRAMMING_ERROR' : err.code,
  });
});

app.listen(3000);
```

**Expected Output (for `GET /bug/type`):**
```json
{"error":"Internal Server Error","code":"PROGRAMMING_ERROR"}
```

**Server log:**
```
PROGRAMMING ERROR (crash-worthy): TypeError: Cannot read properties of null (reading 'method')
    at ...
```

**Why this output:** The `TypeError` is not marked as operational, so it's classified as a programming error. The client receives a generic 500 response, while the server logs the full error for investigation.

### Sub-Feature 4.2: Memory Leaks Within the Middleware Chain

#### Definitions

**Core Definition:** Memory leaks in the middleware chain occur when middleware accumulates references to objects (listeners, closures, caches) that are never released, causing memory usage to grow over time.

**Technical Definition:** Common memory leak sources in Express middleware include: event listeners added but never removed, closures capturing large objects, unbounded caches, and `AsyncLocalStorage` stores holding references. The `Express` framework's own middleware (e.g., `express-session` with MemoryStore) can also leak if not configured properly.

**Beginner-Friendly Explanation:** A memory leak is like a slow water leak in a pipe — it's not obvious at first, but over time it floods the basement. In middleware, leaks happen when you keep adding things (listeners, caches) without ever cleaning up.

#### Purposes

- To prevent gradual memory exhaustion in long-running processes.
- To ensure stable performance over time.
- To identify and fix leak sources.

#### Syntax Rules and Structure

```javascript
// ❌ Leak: listener added on every request
app.use((req, res, next) => {
  process.on('exit', () => { /* ... */ }); // Never removed!
  next();
});

// ✅ Fixed: add listener once
process.on('exit', () => { /* ... */ });

// ❌ Leak: unbounded cache
const cache = {};
app.use((req, res, next) => {
  cache[req.url] = req; // Grows forever
  next();
});

// ✅ Fixed: use LRU cache with max size
const LRU = require('lru-cache');
const cache = new LRU({ max: 500 });
```

**Constraints and Limitations:**
- Use `process.memoryUsage()` to monitor heap growth.
- Heap snapshots can identify retained objects.
- Event listeners should be removed when no longer needed.

#### Annotated Code Example

```javascript
// memory-leak.js
const express = require('express');
const app = express();

// Track listener count
let listenerCount = 0;

// ❌ Leaking middleware: adds listener on every request
app.use((req, res, next) => {
  const listener = () => {};
  process.on('warning', listener);
  listenerCount++;
  // listener is never removed!
  next();
});

app.get('/status', (req, res) => {
  res.json({
    listenerCount,
    heapUsed: `${(process.memoryUsage().heapUsed / 1024 / 1024).toFixed(2)}MB`,
  });
});

app.listen(3000);
```

**Expected Output (after 1000 requests):**
```json
{"listenerCount":1000,"heapUsed":"45.23MB"}
```

**Why this output:** Each request adds a new `'warning'` listener that is never removed. The `listenerCount` grows linearly with the number of requests, and `heapUsed` increases over time. This demonstrates a classic middleware memory leak.

### Sub-Feature 4.3: Process Crash and Recovery Strategies (Cluster Recycling)

#### Definitions

**Core Definition:** Process crash and recovery strategies use process managers (PM2, systemd, Node.js cluster) to automatically restart crashed workers, ensuring high availability.

**Technical Definition:** In a Node.js cluster, if a worker process crashes or is killed, it will auto-boot up again with a new instance. The `exit` event is extremely useful for restarting a worker following a crash: when a crash is detected, `fork()` is called again to replace the crashed worker. PM2 in cluster mode forks worker processes without the application using cluster directly, respawns crashes, and restarts on memory thresholds.

**Beginner-Friendly Explanation:** Process recycling is like having a spare tire that automatically replaces a flat one. When a worker process crashes, the master process immediately spawns a new one so the application keeps running.

#### Purposes

- To maintain high availability despite crashes.
- To recover from programming errors without manual intervention.
- To enable zero-downtime deployments via gradual worker reloading.
- To enforce memory limits and restart workers when exceeded.

#### Syntax Rules and Structure

**Node.js cluster:**
```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Respawning...`);
    cluster.fork();
  });
} else {
  require('./app');
}
```

**PM2 configuration:**
```json
{
  "apps": [{
    "name": "api",
    "script": "app.js",
    "instances": "max",
    "exec_mode": "cluster",
    "max_memory_restart": "500M",
    "exp_backoff_restart_delay": 100
  }]
}
```

**Constraints and Limitations:**
- Crashed workers lose in-flight requests.
- Graceful shutdown is preferred over hard crashes.
- PM2 must be set up as a systemd unit to auto-resurrect on boot.

#### Annotated Code Example

```javascript
// cluster-recycling.js
const cluster = require('node:cluster');
const os = require('node:os');
const http = require('node:http');

if (cluster.isPrimary) {
  const numWorkers = os.availableParallelism();
  console.log(`Primary ${process.pid} starting ${numWorkers} workers`);

  for (let i = 0; i < numWorkers; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (code: ${code}, signal: ${signal})`);
    console.log('Respawning...');
    cluster.fork();
  });
} else {
  const server = http.createServer((req, res) => {
    if (req.url === '/crash') {
      process.exit(1); // Simulate crash
    }
    res.end(`Worker ${process.pid} says hello\n`);
  });

  server.listen(3000, () => {
    console.log(`Worker ${process.pid} listening on port 3000`);
  });
}
```

**Expected Output (console after visiting `/crash`):**
```
Primary 12345 starting 8 workers
Worker 12346 listening on port 3000
Worker 12347 listening on port 3000
...
Worker 12346 died (code: 1, signal: null)
Respawning...
Worker 12350 listening on port 3000
```

**Why this output:** When the worker handling `/crash` calls `process.exit(1)`, the primary process receives the `'exit'` event and immediately forks a new worker. The replacement worker starts listening on the same port, maintaining service availability.

---

## Core Concept 5: Error Classification

### Sub-Feature 5.1: Custom Application Error Classes (Extending Base Error)

#### Definitions

**Core Definition:** Custom error classes extend the built-in `Error` class to add structured properties such as `statusCode`, `code`, and `isOperational`, enabling precise classification and handling.

**Technical Definition:** Custom error classes in JavaScript extend the built-in `Error` class. The `AppError` class pattern includes `name`, `httpCode`, `description`, `isOperational`, and an optional `errors` array for validation details. The `isOperational` flag distinguishes between expected errors and bugs.

**Beginner-Friendly Explanation:** Custom error classes are like labelled boxes in a warehouse. Instead of a generic "error" box, you have "ValidationError," "NotFoundError," and "DatabaseError" — each with its own handling instructions.

#### Purposes

- To identify the error type from the error name.
- To attach structured metadata (status code, error code).
- To distinguish operational errors from programming errors.
- To enable precise error handling in middleware.

#### Syntax Rules and Structure

```javascript
class AppError extends Error {
  constructor(message, statusCode, code, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(errors) {
    super('Validation failed', 422, 'VALIDATION_ERROR');
    this.errors = errors;
  }
}
```

**Constraints and Limitations:**
- Must call `super()` before accessing `this`.
- `Error.captureStackTrace` is V8-specific but widely supported.
- `instanceof` checks work across the inheritance chain.

#### Annotated Code Example

```javascript
// custom-errors.js
class AppError extends Error {
  constructor(message, statusCode, code, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = isOperational;
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(errors) {
    super('Validation failed', 422, 'VALIDATION_ERROR');
    this.errors = errors;
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Authentication required') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

// Usage
const express = require('express');
const app = express();
app.use(express.json());

app.get('/users/:id', (req, res) => {
  const id = parseInt(req.params.id, 10);
  if (isNaN(id)) throw new ValidationError([{ field: 'id', message: 'Must be a number' }]);
  if (id === 999) throw new NotFoundError('User');
  res.json({ id, name: 'Alice' });
});

app.use((err, req, res, next) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
      ...(err.errors && { errors: err.errors }),
    });
  }
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```

**Expected Output (for `GET /users/abc`):**
```json
{"error":"Validation failed","code":"VALIDATION_ERROR","errors":[{"field":"id","message":"Must be a number"}]}
```

**Expected Output (for `GET /users/999`):**
```json
{"error":"User not found","code":"NOT_FOUND"}
```

**Why this output:** `ValidationError` and `NotFoundError` extend `AppError`. The error handler uses `instanceof AppError` to identify operational errors and returns the appropriate status code and structured payload.

### Sub-Feature 5.2: HTTP Status Code Mapping

#### Definitions

**Core Definition:** HTTP status code mapping associates each custom error class with an appropriate HTTP status code, ensuring that errors are communicated to clients with the correct semantics.

**Technical Definition:** The `statusCode` property on custom error classes maps directly to HTTP response status codes. Operational errors (4xx) indicate client issues; programming errors (5xx) indicate server issues. The mapping should follow HTTP semantics: 400 for bad input, 401 for missing auth, 403 for insufficient permissions, 404 for missing resources, 409 for conflicts, 422 for validation failures.

**Beginner-Friendly Explanation:** Status code mapping is like labelling the severity of an issue. A "file not found" is a 404 (client should check the URL). A "database down" is a 503 (server needs to recover).

#### Purposes

- To communicate error semantics precisely.
- To enable client-side handling based on status code.
- To follow HTTP standards and conventions.
- To support API documentation and client SDK generation.

#### Syntax Rules and Structure

| Error Class | Status Code | When to Use |
|-------------|-------------|-------------|
| `ValidationError` | 422 | Request body fails schema validation. |
| `BadRequestError` | 400 | Malformed request syntax. |
| `UnauthorizedError` | 401 | Missing or invalid credentials. |
| `ForbiddenError` | 403 | Authenticated but insufficient permissions. |
| `NotFoundError` | 404 | Resource does not exist. |
| `ConflictError` | 409 | State conflict (duplicate, version mismatch). |
| `RateLimitError` | 429 | Too many requests. |
| `InternalError` | 500 | Programming error. |
| `ServiceUnavailableError` | 503 | Dependency down. |

**Constraints and Limitations:**
- Status codes must be semantically accurate.
- 4xx codes should not be used for server failures.
- 5xx codes should not be used for client input errors.

#### Annotated Code Example

```javascript
// status-mapping.js
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
  }
}

const ErrorTypes = {
  BadRequest: (msg) => new AppError(msg, 400, 'BAD_REQUEST'),
  Unauthorized: (msg) => new AppError(msg, 401, 'UNAUTHORIZED'),
  Forbidden: (msg) => new AppError(msg, 403, 'FORBIDDEN'),
  NotFound: (msg) => new AppError(msg, 404, 'NOT_FOUND'),
  Conflict: (msg) => new AppError(msg, 409, 'CONFLICT'),
  Validation: (msg) => new AppError(msg, 422, 'VALIDATION_ERROR'),
  RateLimit: (msg) => new AppError(msg, 429, 'RATE_LIMIT'),
  Internal: (msg) => new AppError(msg, 500, 'INTERNAL_ERROR', false),
  ServiceUnavailable: (msg) => new AppError(msg, 503, 'SERVICE_UNAVAILABLE'),
};

const express = require('express');
const app = express();

app.get('/error/:type', (req, res, next) => {
  const type = req.params.type;
  const errorFn = ErrorTypes[type] || ErrorTypes.Internal;
  next(errorFn(`Simulated ${type} error`));
});

app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    error: err.message,
    code: err.code,
  });
});

app.listen(3000);
```

**Expected Output (for `GET /error/NotFound`):**
```json
{"error":"Simulated NotFound error","code":"NOT_FOUND"}
```

**HTTP status:** 404 Not Found

**Expected Output (for `GET /error/Validation`):**
```json
{"error":"Simulated Validation error","code":"VALIDATION_ERROR"}
```

**HTTP status:** 422 Unprocessable Entity

**Why this output:** Each error type maps to a specific HTTP status code and domain-specific error code. The `ErrorTypes` factory provides a convenient way to create typed errors with the correct status codes.

### Sub-Feature 5.3: Domain-Specific Error Codes for Frontend Consumption

#### Definitions

**Core Definition:** Domain-specific error codes are machine-readable identifiers (e.g., `INSUFFICIENT_FUNDS`, `ACCOUNT_LOCKED`) that enable the frontend to handle errors programmatically without parsing human-readable messages.

**Technical Definition:** Error codes should be stable, unique, and documented. They complement the HTTP status code by providing finer-grained error classification. For example, a 422 validation error might have codes `INVALID_EMAIL`, `PASSWORD_TOO_SHORT`, and `USERNAME_TAKEN`. The frontend can display field-specific messages or trigger specific UI flows based on the code.

**Beginner-Friendly Explanation:** Error codes are like error numbers on a car dashboard — "P0301" tells a mechanic exactly what's wrong, while "engine problem" is too vague. Similarly, `INSUFFICIENT_FUNDS` tells the frontend exactly what to display.

#### Purposes

- To enable frontend handling without parsing messages.
- To support internationalisation (codes can be mapped to translated messages).
- To provide stable identifiers that don't change with message wording.
- To enable analytics on error types.

#### Syntax Rules and Structure

```javascript
class AppError extends Error {
  constructor(message, statusCode, code, details = {}) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;       // Machine-readable
    this.details = details; // Optional structured data
    this.isOperational = true;
  }
}

// Usage
throw new AppError(
  'Insufficient funds',
  422,
  'INSUFFICIENT_FUNDS',
  { balance: 30, required: 50 }
);
```

**Constraints and Limitations:**
- Error codes must be documented in the API spec.
- Codes should be uppercase snake_case by convention.
- Avoid changing existing codes (breaking change for frontend).

#### Annotated Code Example

```javascript
// domain-error-codes.js
const express = require('express');
const app = express();
app.use(express.json());

// Domain error codes
const ErrorCodes = {
  INSUFFICIENT_FUNDS: 'INSUFFICIENT_FUNDS',
  ACCOUNT_LOCKED: 'ACCOUNT_LOCKED',
  INVALID_CURRENCY: 'INVALID_CURRENCY',
  DAILY_LIMIT_EXCEEDED: 'DAILY_LIMIT_EXCEEDED',
};

class PaymentError extends Error {
  constructor(message, statusCode, code, details = {}) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
    this.isOperational = true;
  }
}

app.post('/payments', (req, res, next) => {
  const { amount, currency } = req.body;

  if (currency !== 'USD') {
    return next(new PaymentError(
      'Invalid currency',
      422,
      ErrorCodes.INVALID_CURRENCY,
      { supported: ['USD'] }
    ));
  }

  if (amount > 1000) {
    return next(new PaymentError(
      'Insufficient funds',
      422,
      ErrorCodes.INSUFFICIENT_FUNDS,
      { balance: 500, required: amount }
    ));
  }

  res.json({ success: true, amount });
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.code,
      ...(Object.keys(err.details).length && { details: err.details }),
    });
  }
  res.status(500).json({ error: 'Internal Server Error' });
});

app.listen(3000);
```

**Expected Output (for `POST /payments` with `{"amount":1500,"currency":"USD"}`):**
```json
{
  "error": "Insufficient funds",
  "code": "INSUFFICIENT_FUNDS",
  "details": { "balance": 500, "required": 1500 }
}
```

**Frontend handling:**
```javascript
// Frontend can switch on the code
if (response.code === 'INSUFFICIENT_FUNDS') {
  showModal(`You need $${response.details.required - response.details.balance} more`);
}
```

**Why this output:** The `PaymentError` includes a domain-specific code and structured details. The frontend can handle the error programmatically without parsing the human-readable message.

---

## Core Concept 6: Structured Error Responses

### Sub-Feature 6.1: Standardized JSON Error Payloads

#### Definitions

**Core Definition:** A standardized JSON error payload is a consistent response structure used for all API errors, ensuring clients can reliably parse and handle errors without endpoint-specific logic.

**Technical Definition:** RFC 7807 (Problem Details for HTTP APIs) defines a standard format for carrying machine-readable error details in HTTP responses, using the media type `application/problem+json`. The Problem Details JSON Object has five canonical members: `type` (URI reference identifying the problem type), `title` (short human-readable summary), `status` (HTTP status code), `detail` (human-readable explanation), and `instance` (URI reference for the specific occurrence). Extension members may be added for additional context.

**Beginner-Friendly Explanation:** A standardized error payload is like a uniform complaint form. Whether the issue is a missing file, a validation error, or a server crash, the form has the same fields in the same places. The client always knows where to find the error message and how to interpret it.

#### Purposes

- To provide a consistent structure for all error responses.
- To enable machine-readable error parsing.
- To avoid the proliferation of proprietary error schemas.
- To support API documentation and client SDK generation.

#### Syntax Rules and Structure

```json
{
  "type": "https://api.example.com/problems/insufficient-credit",
  "title": "Insufficient Credit",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc"
}
```

| Member | Required | Description |
|--------|----------|-------------|
| `type` | No (defaults to `about:blank`) | URI reference identifying the problem type. |
| `title` | No | Short, human-readable summary. |
| `status` | No | HTTP status code. |
| `detail` | No | Human-readable explanation specific to this occurrence. |
| `instance` | No | URI reference identifying the specific occurrence. |
| Extensions | No | Arbitrary additional fields. |

**Constraints and Limitations:**
- The `status` member duplicates the HTTP status code.
- Extension members should not leak sensitive implementation details.
- The media type must be `application/problem+json`.

#### Annotated Code Example

```javascript
// rfc7807-problem.js
const express = require('express');
const app = express();

class ProblemDetails extends Error {
  constructor({ type, title, status, detail, instance, ...extensions }) {
    super(detail);
    this.type = type || 'about:blank';
    this.title = title;
    this.status = status;
    this.detail = detail;
    this.instance = instance;
    Object.assign(this, extensions);
  }

  toJSON() {
    const { name, message, stack, ...problem } = this;
    return problem;
  }
}

app.get('/api/users/:id', (req, res, next) => {
  const id = parseInt(req.params.id, 10);
  if (id === 999) {
    return next(new ProblemDetails({
      type: 'https://api.example.com/problems/user-not-found',
      title: 'User Not Found',
      status: 404,
      detail: `No user exists with ID ${id}.`,
      instance: `/api/users/${id}`,
      userId: id,
    }));
  }
  res.json({ id, name: 'Alice' });
});

app.use((err, req, res, next) => {
  if (err instanceof ProblemDetails) {
    return res
      .status(err.status)
      .type('application/problem+json')
      .json(err);
  }
  res.status(500).type('application/problem+json').json({
    type: 'about:blank',
    title: 'Internal Server Error',
    status: 500,
  });
});

app.listen(3000);
```

**Expected Output (for `GET /api/users/999`):**
```http
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "https://api.example.com/problems/user-not-found",
  "title": "User Not Found",
  "status": 404,
  "detail": "No user exists with ID 999.",
  "instance": "/api/users/999",
  "userId": 999
}
```

**Why this output:** The `ProblemDetails` class implements RFC 7807. The error includes the five canonical members plus an extension member (`userId`). The `Content-Type` is `application/problem+json`. Clients can parse the `type` URI to determine the error category.

### Sub-Feature 6.2: Hiding Internal Stack Traces in Production

#### Definitions

**Core Definition:** Hiding stack traces in production means excluding the `err.stack` property from error responses when `NODE_ENV=production`, preventing attackers from learning about internal file paths, library versions, and code structure.

**Technical Definition:** When `NODE_ENV` is set to `production`, Express's default error handler does not write the stack trace to the client — only the HTTP response code is included. Custom error handlers must explicitly check the environment and omit `err.stack` in production.

**Beginner-Friendly Explanation:** Stack traces are like a map of your building's internal wiring. Useful for electricians (developers) but dangerous in the hands of burglars (attackers). In production, you keep the map in a secure location (server logs) and only tell visitors "the lights are out" (generic error).

#### Purposes

- To prevent information leakage about internal code structure.
- To avoid revealing library versions that may have known vulnerabilities.
- To comply with security best practices.
- To provide a clean, professional error experience.

#### Syntax Rules and Structure

```javascript
const isProduction = process.env.NODE_ENV === 'production';

app.use((err, req, res, next) => {
  // Always log full error server-side
  console.error(err.stack);

  const response = {
    error: isProduction ? 'Internal Server Error' : err.message,
  };

  // Only include stack in development
  if (!isProduction) {
    response.stack = err.stack;
  }

  res.status(err.statusCode || 500).json(response);
});
```

**Constraints and Limitations:**
- The full stack trace must still be logged server-side.
- Even error messages can leak information; use generic messages for 5xx errors.
- Source maps should be uploaded to APM tools for production stack traces.

#### Annotated Code Example

```javascript
// production-stack-hiding.js
const express = require('express');
const app = express();

const isProduction = process.env.NODE_ENV === 'production';

app.get('/api/data', (req, res) => {
  // Simulate an internal error
  const config = null;
  config.database.host; // TypeError
});

app.use((err, req, res, next) => {
  // Log full error server-side (always)
  console.error(`[${new Date().toISOString()}] ERROR:`, err.stack);

  // Build client response
  const response = {
    error: isProduction ? 'An unexpected error occurred' : err.message,
    statusCode: err.statusCode || 500,
  };

  // Development: include stack trace
  if (!isProduction) {
    response.stack = err.stack;
    response.hint = 'Full stack trace available in development mode';
  }

  res.status(err.statusCode || 500).json(response);
});

app.listen(3000, () => {
  console.log(`Server running in ${isProduction ? 'PRODUCTION' : 'DEVELOPMENT'} mode`);
});
```

**Expected Output (development, `GET /api/data`):**
```json
{
  "error": "Cannot read properties of null (reading 'database')",
  "statusCode": 500,
  "stack": "TypeError: Cannot read properties of null (reading 'database')\n    at /app/server.js:12:10\n    ...",
  "hint": "Full stack trace available in development mode"
}
```

**Expected Output (production, `GET /api/data`):**
```json
{"error":"An unexpected error occurred","statusCode":500}
```

**Server log (both environments):**
```
[2026-01-15T12:00:00.000Z] ERROR: TypeError: Cannot read properties of null (reading 'database')
    at /app/server.js:12:10
    ...
```

**Why this output:** In development, the client sees the actual error message, stack trace, and a hint. In production, only a generic message is returned. The server log always contains the full stack trace for debugging.

### Sub-Feature 6.3: Localization (i18n) of Error Messages

#### Definitions

**Core Definition:** Localisation (i18n) of error messages means returning error messages in the user's preferred language, based on the `Accept-Language` header or a user profile setting.

**Technical Definition:** Internationalisation (i18n) involves separating error messages from error codes. The server returns a machine-readable error code (e.g., `INVALID_EMAIL`), and the client (or server, if messages are localised server-side) maps the code to a translated message. The `Accept-Language` header (RFC 9110) indicates the client's language preferences.

**Beginner-Friendly Explanation:** Localisation is like having a multilingual customer service team. A French user gets error messages in French; a Japanese user gets them in Japanese. The error code stays the same; only the message changes.

#### Purposes

- To provide error messages in the user's language.
- To support international applications and user bases.
- To separate error semantics (code) from presentation (message).
- To enable client-side translation.

#### Syntax Rules and Structure

```javascript
const messages = {
  en: {
    INVALID_EMAIL: 'Invalid email address',
    PASSWORD_TOO_SHORT: 'Password must be at least 8 characters',
  },
  fr: {
    INVALID_EMAIL: 'Adresse e-mail invalide',
    PASSWORD_TOO_SHORT: 'Le mot de passe doit contenir au moins 8 caractères',
  },
  es: {
    INVALID_EMAIL: 'Dirección de correo electrónico no válida',
    PASSWORD_TOO_SHORT: 'La contraseña debe tener al menos 8 caracteres',
  },
};

function getMessage(code, lang) {
  const locale = messages[lang] || messages.en;
  return locale[code] || messages.en[code] || code;
}
```

**Constraints and Limitations:**
- Server-side localisation requires maintaining translations for all supported languages.
- Client-side localisation is often preferred (the client already has the translation infrastructure).
- The `Accept-Language` header can be complex; use a library like `negotiator`.

#### Annotated Code Example

```javascript
// i18n-errors.js
const express = require('express');
const app = express();
app.use(express.json());

const messages = {
  en: {
    INVALID_EMAIL: 'Invalid email address',
    PASSWORD_TOO_SHORT: 'Password must be at least 8 characters',
  },
  fr: {
    INVALID_EMAIL: 'Adresse e-mail invalide',
    PASSWORD_TOO_SHORT: 'Le mot de passe doit contenir au moins 8 caractères',
  },
  es: {
    INVALID_EMAIL: 'Dirección de correo electrónico no válida',
    PASSWORD_TOO_SHORT: 'La contraseña debe tener al menos 8 caracteres',
  },
};

function getLocale(req) {
  const acceptLang = req.headers['accept-language'] || 'en';
  // Parse Accept-Language: "fr-FR,fr;q=0.9,en;q=0.8"
  const langs = acceptLang.split(',').map(l => l.split(';')[0].trim().split('-')[0]);
  return langs[0] || 'en';
}

function getMessage(code, locale) {
  const lang = messages[locale] ? locale : 'en';
  return messages[lang][code] || messages.en[code] || code;
}

app.post('/register', (req, res) => {
  const locale = getLocale(req);
  const errors = [];

  if (!req.body.email?.includes('@')) {
    errors.push({
      code: 'INVALID_EMAIL',
      message: getMessage('INVALID_EMAIL', locale),
    });
  }

  if (req.body.password?.length < 8) {
    errors.push({
      code: 'PASSWORD_TOO_SHORT',
      message: getMessage('PASSWORD_TOO_SHORT', locale),
    });
  }

  if (errors.length > 0) {
    return res.status(422).json({
      error: 'Validation failed',
      code: 'VALIDATION_ERROR',
      errors,
    });
  }

  res.status(201).json({ registered: true });
});

app.listen(3000);
```

**Expected Output (with `Accept-Language: fr`):**
```json
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "errors": [
    { "code": "INVALID_EMAIL", "message": "Adresse e-mail invalide" },
    { "code": "PASSWORD_TOO_SHORT", "message": "Le mot de passe doit contenir au moins 8 caractères" }
  ]
}
```

**Expected Output (with `Accept-Language: en`):**
```json
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "errors": [
    { "code": "INVALID_EMAIL", "message": "Invalid email address" },
    { "code": "PASSWORD_TOO_SHORT", "message": "Password must be at least 8 characters" }
  ]
}
```

**Why this output:** The `getLocale` function parses the `Accept-Language` header and selects the appropriate language. The `getMessage` function maps error codes to localised messages. The response includes both the error code (for programmatic handling) and the localised message (for display).

---

## References

- Express.js — Error Handling — https://expressjs.com/en/guide/error-handling.html
- Node.js Documentation — Process — https://nodejs.org/api/process.html
- RFC 7807 — Problem Details for HTTP APIs — https://datatracker.ietf.org/doc/html/rfc7807
- Node.js Cluster — https://nodejs.org/api/cluster.html
- Sentry Node.js — https://docs.sentry.io/platforms/node/
- Datadog Node.js — https://docs.datadoghq.com/tracing/trace_collection/
- Heroku — Best Practices in Error Handling — https://www.heroku.com/codeish-podcasts/51-best-practices-in-error-handling
- Building a Simple and Effective Error-Handling System in Node.js — https://tsecurity.de/de/2462968/
- Express Error Inbox: Node.js API Setup — https://dev.to/judsonrhodes1569/
- TRAE-Skills — Error Handling (Express) — https://github.com/HighMark-31/TRAE-Skills
- MDN Web Docs — HTTP Status Codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status