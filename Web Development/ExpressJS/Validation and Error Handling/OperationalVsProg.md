# Operational vs Programming Errors — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Operational errors and programming errors are the two fundamental categories into which all runtime failures in a Node.js/Express application can be divided, distinguished by whether the failure is an expected consequence of normal operation (operational) or a defect in the code itself (programming).

**Technical Definition**
Operational errors represent runtime problems experienced by correctly written programs—such as invalid user input, network timeouts, database connection failures, or missing files. They are not bugs; they are expected failures that must be handled gracefully with appropriate responses and recovery strategies. Programming errors (also called programmer errors or catastrophic errors) are bugs in the program—such as reading a property of `undefined`, type errors, logic mistakes, or failing to handle an operational error. They represent defects that can only be fixed by changing the code. The distinction is critical because the two categories demand fundamentally different handling strategies: operational errors should be caught, logged, and translated into safe client responses, while programming errors should be allowed to crash the process (or be caught at the process level) so that a process manager can restart the application in a known-good state, with the bug subsequently fixed in code.

**Beginner-Friendly Explanation**
When something goes wrong in your app, it helps to ask: “Is this an expected problem, or is this a bug?” Expected problems—like a user typing a bad email address or a database being temporarily unreachable—are **operational errors**. You should catch these, log them, and send a friendly message to the user. Bugs—like trying to read a property of `undefined` or forgetting to handle an error—are **programming errors**. These mean your code is broken. You cannot “handle” them meaningfully; the best strategy is to let the process crash (or catch it at the top level), restart, and fix the code.

---

### Key Characteristics

- **Different root causes** — Operational errors arise from external conditions (user input, network, I/O); programming errors arise from defects in the code itself.
- **Different recoverability** — Operational errors are recoverable (retry, return a `4xx`, fall back); programming errors are not meaningfully recoverable at the point of failure because the application state may be corrupted.
- **Different response strategies** — Operational errors produce specific, safe client messages with appropriate HTTP status codes (`400`, `404`, `503`); programming errors produce generic `500` responses with full details logged server-side.
- **Different process-level behavior** — Operational errors should not crash the process; programming errors should trigger a graceful shutdown and restart.
- **The `isOperational` flag** — Custom error classes conventionally carry an `isOperational` boolean that encodes this classification, enabling the centralized error handler to branch appropriately.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- Understanding of Express error-handling middleware and the four-argument signature `(err, req, res, next)`.
- Familiarity with custom error classes and error propagation.
- Basic knowledge of HTTP status codes and process management (PM2, Docker restart policies, systemd).

---

### Related Programming Areas

- **Error handling middleware** — The centralized handler is where classification drives response behavior.
- **Custom error classes** — The `isOperational` flag is the primary vehicle for classification.
- **Process management** — Restarters (PM2, Docker, systemd) implement the “crash and restart” strategy for programming errors.
- **Observability** — Logging and alerting must distinguish the two categories to avoid noise and ensure bugs are surfaced.
- **API design** — Operational error contracts (status codes, error codes) are part of the public API; programming errors are not.

---

### Core Concepts / Features

1. **Operational errors** — Expected runtime failures from external conditions.
2. **Programming errors** — Bugs in the code.
3. **Error classification** — The process of determining which category an error belongs to.
4. **Error handling strategies** — The differentiated responses for each category.
5. **Error classification best practices** — Patterns for reliable classification and handling.

---

## Core Concept 1: Operational Errors

### Definitions

**Core Definition**
Operational errors are runtime problems experienced by correctly written programs—expected failures arising from external conditions rather than defects in the code—that must be handled gracefully to maintain application stability.

**Technical Definition**
Operational errors represent runtime problems whose results are expected and should be dealt with in a proper way. They do not mean the application itself has bugs. Examples include invalid input for an API endpoint, failed database connections, network timeouts, file-not-found errors, rate-limit exhaustion, and out-of-memory conditions from resource exhaustion. In Express, operational errors are typically represented by custom error classes carrying an `isOperational: true` flag and an appropriate `statusCode`. The centralized error handler responds with the specific status code and a safe message, and the process continues running.

**Beginner-Friendly Explanation**
Operational errors are the “normal problems” of running a web application. A user submits a form with a missing field, a third-party API is slow, a file has been deleted—these are not your code's fault, but you still need to handle them. You catch them, tell the user what went wrong (in a safe way), and keep the server running.

---

### Purposes

- To represent expected failures with structured metadata (status code, error code, details) for accurate client responses.
- To enable graceful degradation and recovery without crashing the process.
- To distinguish “user mistakes” and “external failures” from actual bugs, so that monitoring and alerting are not polluted by normal events.
- To provide safe, actionable feedback to clients while preserving application stability.

---

### Syntax Rules and Structure

#### Complete General Syntax: Operational Error Class

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = true; // marks as operational
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `extends Error` | Inherits standard error behavior |
| `statusCode` | HTTP status for the client response |
| `isOperational` | Classification flag (`true` for operational errors) |
| `Error.captureStackTrace` | Clean stack trace for logging |

#### Syntax Rules

- Operational errors **must** set `isOperational = true` so the central handler can distinguish them from bugs.
- Operational errors **must** carry a meaningful `statusCode` (e.g., `400`, `404`, `409`, `503`).
- The `message` must be safe to expose to clients; it should not contain internal paths or secrets.
- Operational errors should be **thrown or passed to `next(err)`**, not allowed to crash the process.

#### Constraints and Limitations

- Not every error that “feels” operational is operational; misclassifying a bug as operational exposes internal details and hides defects.
- Operational errors that are not handled become programming errors (failing to handle an expected failure is itself a defect).

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Operational Errors in a Centralized Handler

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await findUser(req.params.id);
    if (!user) {
      throw new AppError('User not found', 404); // operational
    }
    res.json(user);
  } catch (err) {
    next(err);
  }
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: { message: err.message, code: err.code || 'OPERATIONAL_ERROR' }
    });
  }
  // programming error handling below
});
```

**Expected Output**

```
GET /users/99
→ 404 { "error": { "message": "User not found", "code": "OPERATIONAL_ERROR" } }
```

**Why This Output Occurs**

- The `AppError` is marked `isOperational: true`, so the handler exposes its specific message and status code.

---

### Real-World Cases

- **Invalid user input** — Returning `400 Bad Request` with field-level validation details.
- **Missing resources** — Returning `404 Not Found` when a database lookup returns `null`.
- **External service failures** — Returning `503 Service Unavailable` with a `Retry-After` header when a dependency is down.
- **Rate limiting** — Returning `429 Too Many Requests`.

---

## Core Concept 2: Programming Errors

### Definitions

**Core Definition**
Programming errors (also called programmer errors, bugs, or catastrophic errors) are defects in the application code that cause unexpected failures and cannot be meaningfully recovered from at the point of occurrence because the application state may be corrupted.

**Technical Definition**
Programmer errors represent unexpected bugs in poorly written code. They mean the code itself has some issues to solve and was coded wrong. Examples include attempting to read a property of `undefined`, type errors, calling an async function without awaiting or returning it, leaking database connections, and failing to handle an operational error. Because the application state is uncertain after a programming error, the recommended strategy is to let the process crash (or gracefully shut it down) so that a process manager can restart it in a known-good state. The bug must then be fixed in code; restarting alone does not solve it.

**Beginner-Friendly Explanation**
Programming errors are bugs—mistakes in your code. If your code tries to read a property of `undefined` or forgets to handle an error, that is a bug. You cannot “fix” it at runtime by catching it and continuing, because the application might be in a broken state. The right approach is to let it crash, restart the process, and then fix the code.

---

### Purposes

- To identify defects that require code changes rather than runtime recovery.
- To prevent corrupted application state from producing unpredictable behavior on subsequent requests.
- To surface bugs loudly in monitoring and alerting so they can be prioritized and fixed.
- To justify the “crash and restart” strategy that restores a known-good state.

---

### Syntax Rules and Structure

#### Complete General Syntax: Programming Error Detection and Handling

```javascript
// A programming error: unhandled operational error or bug
app.get('/bug', (req, res, next) => {
  throw new Error('Cannot read properties of undefined'); // programming error
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({ error: { message: err.message } });
  }

  // Programming error: log full details, send generic response
  console.error({ message: err.message, stack: err.stack, path: req.path });
  res.status(500).json({ error: { message: 'Internal Server Error', code: 'INTERNAL_ERROR' } });
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| No `isOperational` flag (or `isOperational: false`) | Identifies the error as a programming error |
| `console.error` / logger | Captures full details server-side |
| Generic `500` response | Prevents information leakage |

#### Syntax Rules

- Programming errors must **not** expose their `message` or `stack` to clients; a generic `500` message is returned.
- Full details must be logged server-side for debugging and alerting.
- At the process level, programming errors should trigger a **graceful shutdown** and restart, not be swallowed.
- `process.on('uncaughtException')` and `process.on('unhandledRejection')` handlers should log the error and exit with a non-zero code.

#### Constraints and Limitations

- Catching a programming error and continuing to serve requests can lead to unpredictable behavior and data corruption.
- Restarting without fixing the code simply reproduces the crash.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Programming Error Masked in the Response

```javascript
app.get('/bug', (req, res, next) => {
  throw new Error('Cannot read properties of undefined (reading "name")');
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({ error: { message: err.message } });
  }

  console.error({
    level: 'error',
    message: err.message,
    stack: err.stack,
    path: req.path
  });

  res.status(500).json({
    error: { message: 'Internal Server Error', code: 'INTERNAL_ERROR' }
  });
});
```

**Expected Output**

```
GET /bug
→ 500 { "error": { "message": "Internal Server Error", "code": "INTERNAL_ERROR" } }
Console: { "level": "error", "message": "Cannot read properties of undefined...", "stack": "...", "path": "/bug" }
```

**Why This Output Occurs**

- The error lacks `isOperational: true`, so the handler treats it as a programming error, logs full details server-side, and sends a generic message to the client.

---

### Real-World Cases

- **`undefined` property access** — A common bug in unvalidated data handling.
- **Type errors** — Passing a string where a number is expected.
- **Unhandled promise rejections** — Floating promises that escape Express 5 auto-catching.
- **Resource leaks** — Unclosed database connections or file handles.

---

## Core Concept 3: Error Classification

### Definitions

**Core Definition**
Error classification is the process of determining whether a given error is operational (expected, recoverable) or a programming error (a bug), typically by inspecting the error object's properties—most commonly the `isOperational` flag—or by the context in which the error arose.

**Technical Definition**
In practice, classification is implemented through a custom base error class (`AppError`) that sets `isOperational: true` by default, with specific subclasses (`NotFoundError`, `ValidationError`) inheriting that flag. Errors that do not carry the flag—native `Error` objects, type errors, or errors thrown by third-party libraries without classification—are treated as programming errors by default. Some libraries (e.g., `http-errors`) create errors with `statusCode` and `expose` properties that can inform classification. The centralized error handler branches on `isOperational` to determine whether to expose the message and status code or to send a generic `500` response.

**Beginner-Friendly Explanation**
Classification is how you decide “Is this a normal problem or a bug?” The simplest way is to use a custom error class: when you throw a `NotFoundError` or `ValidationError`, you are explicitly saying “this is operational.” Everything else—random errors from libraries, type errors, `undefined` access—is treated as a bug unless you deliberately wrap it in an operational error class.

---

### Purposes

- To ensure that expected failures receive appropriate, specific responses while bugs are masked.
- To drive differentiated logging (warnings for operational errors, errors/alerts for bugs).
- To prevent information leakage from bugs while preserving useful client feedback for operational failures.
- To enable reliable process-level behavior (continue for operational, restart for programming).

---

### Syntax Rules and Structure

#### Complete General Syntax: Classification via `isOperational`

```javascript
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

// In the central handler:
const isOperational = err.isOperational === true;
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `isOperational = true` | Default for `AppError` subclasses |
| `isOperational = false` | Explicitly marks a catastrophic/programming error |
| `err.isOperational === true` | Classification check in the handler |

#### Syntax Rules

- Operational errors **must** carry `isOperational: true`; absence of the flag (or `false`) defaults to programming error.
- Native `Error` instances and errors from third-party libraries are **unclassified** and should be treated as programming errors unless deliberately wrapped.
- The classification check should be explicit (`=== true`) to avoid truthy coercion surprises.

#### Constraints and Limitations

- Libraries may throw errors that are “operational” in nature (e.g., a database connection error) but lack the flag; wrapping or mapping them is necessary.
- Misclassification is the most common source of error-handling bugs: a programming error treated as operational leaks details; an operational error treated as programming hides useful feedback.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Classifying a Third-Party Database Error

```javascript
app.get('/data', async (req, res, next) => {
  try {
    const rows = await db.query('SELECT * FROM users');
    res.json(rows);
  } catch (err) {
    // Wrap known database errors as operational
    if (err.code === 'ECONNREFUSED' || err.code === 'ETIMEDOUT') {
      return next(new AppError('Database unavailable', 503));
    }
    next(err); // unknown errors remain programming errors
  }
});
```

**Expected Output**

```
GET /data  (database down)
→ 503 { "error": { "message": "Database unavailable", "code": "OPERATIONAL_ERROR" } }

GET /data  (unexpected query bug)
→ 500 { "error": { "message": "Internal Server Error", "code": "INTERNAL_ERROR" } }
```

**Why This Output Occurs**

- Known database connectivity errors are wrapped in `AppError` (operational); unknown errors are forwarded as-is and treated as programming errors.

---

### Real-World Cases

- **Database driver errors** — Mapping `ECONNREFUSED` and `ETIMEDOUT` to `503 Service Unavailable`.
- **HTTP client errors** — Mapping `404` and `429` from third-party APIs to operational errors.
- **File system errors** — Mapping `ENOENT` to a `404` for user-facing file requests.

---

## Core Concept 4: Error Handling Strategies

### Definitions

**Core Definition**
Error handling strategies are the differentiated responses applied to operational errors versus programming errors—graceful recovery and specific client feedback for the former, masking and process restart for the latter.

**Technical Definition**
For operational errors, the strategy is to catch the error, log it (at a `warn` level for `4xx`, `error` for `5xx`), and send a safe, specific response with the appropriate status code and error code. For programming errors, the strategy is to log the full stack and error object at `error`/`fatal` level, send a generic `500` response (without stack traces), and—at the process level—allow the process to crash or gracefully exit so that a restarter can bring it back. Node.js best practices recommend distinguishing the two categories centrally and handling them differently: never let an operational error crash the process unnecessarily, and never keep the process running after an unknown catastrophic error.

**Beginner-Friendly Explanation**
For operational errors: catch them, tell the user what went wrong, and keep the server running. For programming errors: log everything, send a vague “something went wrong,” and let the server restart so it can come back clean. The bug then gets fixed in code.

---

### Purposes

- To maximize uptime by not crashing on expected failures.
- To surface bugs loudly so they are fixed rather than hidden.
- To prevent data corruption and unpredictable behavior from continuing after a bug.
- To provide appropriate client experiences for each error type.

---

### Syntax Rules and Structure

#### Complete General Syntax: Differentiated Strategy

```javascript
app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  const level = err.isOperational ? (status >= 500 ? 'error' : 'warn') : 'fatal';

  logger[level]({ status, message: err.message, stack: err.stack, path: req.path });

  res.status(status).json({
    error: { message, code: err.isOperational ? err.code : 'INTERNAL_ERROR' }
  });

  if (!err.isOperational) {
    // Optional: trigger graceful shutdown after responding
    server.close(() => process.exit(1));
  }
});
```

**Component Breakdown**

| Strategy | Operational | Programming |
|----------|-------------|-------------|
| Log level | `warn` (4xx) / `error` (5xx) | `fatal` |
| Response message | Specific, safe | Generic |
| Status code | `err.statusCode` | `500` |
| Process behavior | Continue | Graceful exit + restart |

#### Syntax Rules

- Operational errors **must not** crash the process; the server continues serving.
- Programming errors **should** trigger a graceful shutdown after the response is sent (or let the process crash).
- Log levels must differentiate the two categories.
- Process-level handlers (`uncaughtException`, `unhandledRejection`) must log and exit, not swallow.

#### Constraints and Limitations

- In multi-tenant or long-running request scenarios, immediate exit may interrupt in-flight requests; graceful shutdown (stop accepting new connections, finish in-flight requests) is preferred.
- The `isOperational` classification must be reliable for this strategy to work.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Differentiated Handling in the Central Handler

```javascript
const express = require('express');
const app = express();

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

app.get('/users/:id', (req, res, next) => {
  if (req.params.id !== '1') {
    throw new AppError('User not found', 404); // operational
  }
  res.json({ id: 1, name: 'Alice' });
});

app.get('/bug', (req, res, next) => {
  throw new Error('Cannot read properties of undefined'); // programming
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    console.warn(`[WARN] ${err.statusCode} ${err.message}`);
    return res.status(err.statusCode).json({ error: { message: err.message } });
  }

  console.error(`[FATAL] ${err.message}`, err.stack);
  res.status(500).json({ error: { message: 'Internal Server Error' } });
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1  → 200 { "id": 1, "name": "Alice" }
GET /users/2  → 404 { "error": { "message": "User not found" } }
Console: [WARN] 404 User not found

GET /bug     → 500 { "error": { "message": "Internal Server Error" } }
Console: [FATAL] Cannot read properties of undefined (stack trace)
```

**Why This Output Occurs**

- The operational error receives its specific status and message with a `warn` log; the programming error receives a generic `500` with a `fatal` log and full stack trace.

---

### Real-World Cases

- **API gateways** — Operational errors return `4xx`/`503`; bugs return `500` and trigger alerts.
- **Graceful shutdown** — After a programming error, the process stops accepting new connections and exits after finishing in-flight requests.
- **Process managers** — PM2, Docker restart policies, and systemd implement the “crash and restart” strategy.

---

## Core Concept 5: Error Classification Best Practices

### Definitions

**Core Definition**
Error classification best practices are the design and implementation patterns that ensure reliable classification of operational versus programming errors, correct differentiated handling, and safe, observable behavior across the application.

**Technical Definition**
Best practices include: using a custom base `AppError` class with `isOperational: true` and specific `statusCode` values; treating unclassified errors (native `Error`, library errors) as programming errors by default; wrapping known external failures (database connectivity, HTTP client errors) in operational error classes; distinguishing log levels (`warn` for operational `4xx`, `error`/`fatal` for `5xx` and bugs); implementing process-level handlers (`uncaughtException`, `unhandledRejection`) that log and exit rather than swallow; and documenting the classification and error contract for API clients.

**Beginner-Friendly Explanation**
Best practices are the habits that keep error classification reliable. Always use your own error class for expected problems. Treat everything else as a bug unless you deliberately wrap it. Log bugs loudly and operational errors quietly. And make sure your process restarts after a bug instead of limping along broken.

---

### Purposes

- To prevent misclassification that either leaks information or hides bugs.
- To ensure operational errors do not crash the process and programming errors do not continue in a corrupted state.
- To keep logs and alerts meaningful by separating expected events from defects.
- To provide a reliable API error contract for clients.

---

### Syntax Rules and Structure

#### Complete General Syntax: Best-Practice Classification Pipeline

```javascript
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Central handler
app.use((err, req, res, next) => {
  const isOperational = err.isOperational === true;
  const status = isOperational ? err.statusCode : 500;
  const level = isOperational ? (status >= 500 ? 'error' : 'warn') : 'fatal';

  logger[level]({ status, message: err.message, stack: err.stack, path: req.path });

  res.status(status).json({
    error: {
      message: isOperational ? err.message : 'Internal Server Error',
      code: isOperational ? err.code : 'INTERNAL_ERROR'
    }
  });

  if (!isOperational) server.close(() => process.exit(1));
});

// Process-level safety nets
process.on('uncaughtException', (err) => {
  logger.fatal({ err }, 'Uncaught exception — shutting down');
  process.exit(1);
});

process.on('unhandledRejection', (reason) => {
  logger.error({ err: reason }, 'Unhandled rejection');
  throw reason; // convert to uncaughtException
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Custom base class | `AppError` with `isOperational: true` |
| Explicit check | `err.isOperational === true` |
| Differentiated logging | `warn` / `error` / `fatal` |
| Generic `500` for bugs | No message/stack leakage |
| Graceful shutdown | `server.close` + `process.exit(1)` |
| Process-level handlers | Log and exit, never swallow |

#### Syntax Rules

- **Always** use `isOperational: true` for expected failures; **never** set it on bugs.
- **Treat unclassified errors as programming errors** by default.
- **Wrap known external failures** in operational error classes.
- **Log programming errors at `fatal` level** and trigger restart.
- **Do not swallow** `uncaughtException`; log and exit.
- **Document** the operational error contract (status codes, error codes) for clients.

#### Constraints and Limitations

- Graceful shutdown adds complexity (tracking in-flight requests); simple `process.exit(1)` is acceptable for many applications.
- Some frameworks or libraries may classify their errors differently; mapping is required.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Classification Pipeline

```javascript
const express = require('express');
const app = express();

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
  }
}

app.get('/users/:id', (req, res, next) => {
  if (req.params.id !== '1') throw new NotFoundError('User');
  res.json({ id: 1 });
});

app.get('/bug', (req, res, next) => {
  throw new Error('Unexpected bug');
});

app.use((err, req, res, next) => {
  if (err.isOperational) {
    console.warn(`[WARN] ${err.statusCode} ${err.message}`);
    return res.status(err.statusCode).json({ error: { message: err.message } });
  }

  console.error(`[FATAL] ${err.message}`, err.stack);
  res.status(500).json({ error: { message: 'Internal Server Error' } });
});

app.listen(3000);
```

**Expected Output**

```
GET /users/1  → 200 { "id": 1 }
GET /users/2  → 404 { "error": { "message": "User not found" } }
Console: [WARN] 404 User not found

GET /bug     → 500 { "error": { "message": "Internal Server Error" } }
Console: [FATAL] Unexpected bug (stack trace)
```

**Why This Output Occurs**

- The `NotFoundError` is operational and produces a specific `404`; the unclassified `Error` is treated as a programming error and masked as `500`.

---

### Real-World Cases

- **Production APIs** — Operational errors return documented `4xx`/`503` responses; bugs trigger alerts and restarts.
- **Microservices** — Each service classifies its errors and exposes operational contracts; bugs are surfaced to the platform.
- **Security-sensitive systems** — Stack traces never reach clients; bugs are logged with full context.

---

## References

- How to handle errors in Express · CoreUI — https://coreui.io/answers/how-to-handle-errors-in-express/
- Express 错误处理模式完全指南 · Guance — https://www.guance.com/learn/articles/error-handling-express
- Express Error Basics · Compile-N-Run (GitHub) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/9-express-error-handling/0-express-error-basics.mdx
- Distinguish catastrophic errors from operational errors · Node.js Best Practices (goldbergyoni) — https://github.com/goldbergyoni/nodebestpractices
- Best Practices for Node.js Error-handling · Toptal — https://www.toptal.com/developers/nodejs/node-js-error-handling
- Express Error Handling Patterns That Scale · Bugsly — https://bugsly.dev/blog/express-error-handling-patterns
- Operational vs. Programmer Errors · Enki Curriculum (GitHub) — https://raw.githubusercontent.com/enkidevs/curriculum/b7edab1f57a46b9157803e400dde7018236ba3ca/javascript/node/operational-vs-programmer-errors/operational-vs-programmer-errors.md
- express-unified-response · npm — https://www.npmjs.com/package/express-unified-response
- @point-hub/express-error-handler · npm — https://www.npmjs.com/package/@point-hub/express-error-handler
- oops-error · npm — https://www.npmjs.com/package/oops-error