# Express.js Error-Handling Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Error-handling middleware in Express.js is a specialized category of middleware, identified by its four-argument signature `(err, req, res, next)`, that intercepts errors propagated through the middleware stack and determines the response sent to the client.

**Technical Definition**
Express defines error-handling middleware functions in the same way as other middleware functions, except that error-handling functions have four arguments instead of three: `(err, req, res, next)`. Express identifies error handlers by **arity** — the number of declared parameters. Error-handling middleware must be defined **at the very end of the middleware function stack**, after all other `app.use()` and route calls. When `next(err)` is called with any value except the string `'route'`, Express regards the current request as being in an error state and skips all remaining non-error-handling routing and middleware functions, proceeding directly to the nearest error handler. Multiple error handlers can be registered and are executed in registration order; each must call `next(err)` to pass the error onward unless it sends a response. Express includes a default error handler at the end of the stack, which writes the stack trace to the client in development but omits it when `NODE_ENV=production`.

**Beginner-Friendly Explanation**
Error-handling middleware is special code that wakes up only when something goes wrong. Normal middleware runs for every request; the error handler waits until an error is thrown or passed to `next(error)`. Then it takes over, logs the problem, and sends a safe response to the user. Express has a basic built-in catcher, but for real applications you write your own to control the message, log the error, and avoid leaking sensitive details.

---

### Key Characteristics

- **Arity-based identification** — Express recognizes error handlers solely by their four-argument signature; three-argument functions are treated as regular middleware and will never be invoked for errors.
- **Selective invocation** — Error handlers run only when an error has been propagated via `next(err)` or thrown in synchronous code (or rejected in Express 5 async code).
- **Order-dependent chaining** — Multiple error handlers execute in registration order, each calling `next(err)` to delegate to the next until one sends a response.
- **Final-stack placement** — Error-handling middleware must be registered after all routes and regular middleware to catch errors from the entire pipeline.
- **Environment-aware default behavior** — The default error handler includes stack traces in development but omits them in production (`NODE_ENV=production`).

---

### Prerequisites

- Node.js 18 or higher (required by Express 5).
- An Express application initialized with `const app = express()`.
- Understanding of the middleware signature `(req, res, next)` and the `next()` function.
- Familiarity with HTTP status codes and the difference between operational and programming errors.

---

### Related Programming Areas

- **Middleware** — Error handlers are a specialized middleware type with distinct arity and invocation rules.
- **Routing** — Errors thrown or forwarded from route handlers propagate to error middleware.
- **Async handlers** — Promise rejections and async throws must be routed to error handlers (automatic in Express 5, manual wrappers in Express 4).
- **Logging and monitoring** — Centralized error handlers are the natural place to log errors and emit metrics.
- **Security** — Error handlers prevent stack traces and internal details from leaking to clients.

---

## Core Concept 1: Error-Handling Middleware

### Definitions

**Core Definition**
Error-handling middleware is middleware that executes when an error is present in the request pipeline, receiving the error object as its first argument and responsible for producing an appropriate error response.

**Technical Definition**
Error-handling middleware functions are defined with the `(err, req, res, next)` signature and registered using `app.use()` or `router.use()`. They are invoked when `next(err)` is called, when synchronous code throws inside a handler, or when a Promise returned by a handler rejects (Express 5). Express skips all remaining non-error-handling middleware and routes once an error is passed to `next()`. Error handlers may log the error, transform it, set status codes, and send a response; if they do not respond, they must call `next(err)` to continue to the next error handler.

**Beginner-Friendly Explanation**
An error-handling middleware is a function that only wakes up when something goes wrong. Normal middleware runs for every request; the error handler waits until an error is thrown or passed to `next(error)`. Then it takes over, logs the problem, and sends a safe response to the user.

---

### Purposes

- To catch errors from route handlers and middleware before they crash the process or leave requests hanging.
- To centralize error processing, avoiding duplicated error-response logic across handlers.
- To log errors for debugging, monitoring, and alerting.
- To send consistent, user-friendly error responses with appropriate HTTP status codes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use((err, req, res, next) => {
  // handle the error
});
```

**Component Breakdown**

| Parameter | Role |
|-----------|------|
| `err` | The error object passed to `next(err)` or thrown |
| `req` | The request object (useful for logging context) |
| `res` | The response object used to send the error response |
| `next` | Must be declared to maintain the four-argument arity; call `next(err)` to delegate |

#### Syntax Rules

- Error handlers **must** have four arguments; otherwise Express treats them as regular middleware.
- Register error handlers after all routes and regular middleware.
- If the error handler does not send a response, it must call `next(err)` to pass the error to the next error handler.
- Calling `next()` with any value other than `'route'` signals an error state.

#### Constraints and Limitations

- Error handlers cannot recover the request into the normal flow; they can only respond or delegate to another error handler.
- `next('route')` is meaningful only in route handlers, not in error-handling middleware.
- The default error handler exposes stack traces unless `NODE_ENV=production`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic Error Handler

```javascript
const express = require('express');
const app = express();

app.get('/fail', (req, res, next) => {
  next(new Error('Something broke'));
});

// Error handler: four arguments, registered last
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: err.message });
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Route triggers error** — `next(new Error('Something broke'))` places the request in an error state.
2. **Error state** — Express skips all remaining regular middleware and routes.
3. **Error handler invoked** — The four-argument function receives the error and sends a `500` response.

**Expected Output**

```
GET /fail → 500 { "error": "Something broke" }
Console: Error: Something broke
```

**Why This Output Occurs**

- `next(err)` bypasses normal handling and invokes the error handler, which reads `err.message` and responds.

---

### Real-World Cases

- **API error responses** — Returning structured JSON errors with appropriate status codes.
- **Logging** — Capturing error stacks and request context for observability.
- **Fallback safety net** — Preventing unhandled errors from crashing the process.

---

## Core Concept 2: Four-Argument Signature

### Definitions

**Core Definition**
The four-argument signature `(err, req, res, next)` is the parameter pattern that uniquely identifies a function as error-handling middleware to Express.

**Technical Definition**
Express checks the **arity** (the number of declared parameters) of each middleware function to determine its role. Functions with exactly four parameters are registered as error handlers; functions with three parameters are regular middleware. The `next` parameter must remain in the signature even if unused, because removing it reduces arity to three and causes Express to treat the function as regular middleware. The error object is always the first argument, enabling error handlers to inspect properties such as `message`, `status`, and custom fields.

**Beginner-Friendly Explanation**
Express looks at how many parameters your function declares. Three means “normal middleware.” Four means “error handler.” The fourth parameter is always the error. If you write three parameters in an error handler, Express will never call it when an error occurs.

---

### Purposes

- To allow Express to distinguish error handlers from regular middleware without explicit configuration.
- To provide a consistent interface for receiving and processing errors.
- To ensure error handlers can access the error object and the request/response context.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
// Regular middleware (3 arguments)
function regular(req, res, next) { /* ... */ }

// Error-handling middleware (4 arguments)
function errorHandler(err, req, res, next) { /* ... */ }
```

**Component Breakdown**

| Signature | Role |
|-----------|------|
| `(req, res, next)` | Regular middleware or route handler |
| `(err, req, res, next)` | Error-handling middleware |

#### Syntax Rules

- Error handlers **must** declare all four parameters, even if `next` is unused.
- The parameter names are conventional but not mandatory; arity is what matters.
- Arrow functions and method shorthand must also declare four parameters to be recognized.

#### Constraints and Limitations

- Removing the `next` parameter to avoid lint warnings silently breaks error handling by reducing arity to three.
- A three-argument function registered with `app.use()` after routes will be treated as regular middleware and will not catch errors.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Correct vs. Incorrect Arity

```javascript
const express = require('express');
const app = express();

app.get('/fail', (req, res, next) => {
  next(new Error('Boom'));
});

// CORRECT: four arguments
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

// INCORRECT: three arguments (will NOT catch errors)
// app.use((err, req, res) => {
//   res.status(500).json({ error: err.message });
// });

app.listen(3000);
```

**Expected Output (Correct)**

```
GET /fail → 500 { "error": "Boom" }
```

**Expected Output (Incorrect)**

```
GET /fail → Express default error handler (stack trace in development)
```

**Why This Output Occurs**

- The four-argument function is recognized as an error handler; the three-argument version is treated as regular middleware and is skipped during error propagation.

---

### Real-World Cases

- **Linting configurations** — Some ESLint rules warn about unused `next`; the parameter must still be retained for correct arity.
- **Centralized error middleware** — The four-argument signature is the foundation of application-wide error handling.
- **Testing** — Error handlers can be tested in isolation by invoking them with a mock error object.

---

## Core Concept 3: Error Handling Order

### Definitions

**Core Definition**
Error handling order is the principle that error-handling middleware must be registered after all routes and regular middleware, and that multiple error handlers execute in the order they are registered.

**Technical Definition**
Express evaluates middleware and routes in the order they are added. Error handlers must be placed at the very end of the middleware function stack, after all `app.use()` and route calls. If multiple error handlers are registered, they are called in registration order; each must call `next(err)` to pass the error to the next error handler. A common pattern is to register a catch-all 404 handler (which creates a `404` error and calls `next(err)`) before the final error handler, so unmatched routes are converted into errors and handled uniformly.

**Beginner-Friendly Explanation**
Order matters. Error handlers go last—after all your routes. If you put an error handler before your routes, it will never catch errors from those routes. If you have several error handlers, they run in the order you wrote them, and each one must pass the error along if it does not respond.

---

### Purposes

- To ensure error handlers can catch errors from every route and middleware registered before them.
- To allow multiple error handlers to process errors in a defined sequence (e.g., logging, then client response).
- To convert unmatched routes into 404 errors that flow through the same error pipeline.
- To prevent errors from reaching the default handler unintentionally.

---

### Syntax Rules and Structure

#### Complete General Syntax: Recommended Order

```javascript
// 1. Routes and regular middleware
app.use('/api', apiRouter);
app.get('/', homeHandler);

// 2. Catch-all 404 handler (converts unmatched routes to errors)
app.use((req, res, next) => {
  const err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// 3. Error handlers (in order)
app.use(logErrors);
app.use(clientErrorHandler);
app.use(errorHandler);
```

**Component Breakdown**

| Order | Handler Type | Purpose |
|-------|-------------|---------|
| 1 | Routes | Business logic |
| 2 | 404 catch-all | Converts unmatched routes to `404` errors |
| 3a | `logErrors` | Logs the error, then `next(err)` |
| 3b | `clientErrorHandler` | Handles XHR errors or delegates |
| 3c | `errorHandler` | Final response |

#### Syntax Rules

- Error handlers **must** be registered after all routes and regular middleware.
- Multiple error handlers execute in registration order; each must call `next(err)` unless responding.
- A 404 catch-all should be placed after routes but before error handlers.

#### Constraints and Limitations

- Placing an error handler before a route means errors from that route will not reach it.
- The default error handler is always last; custom handlers registered after it will not run unless the default is bypassed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Chained Error Handlers with 404 Catch-All

```javascript
const express = require('express');
const app = express();

app.get('/ok', (req, res) => res.send('OK'));

// 404 catch-all
app.use((req, res, next) => {
  const err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// Error handler 1: logging
app.use((err, req, res, next) => {
  console.log(`[ERROR] ${err.status || 500}: ${err.message}`);
  next(err); // pass to next error handler
});

// Error handler 2: final response
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ error: err.message });
});

app.listen(3000);
```

**Expected Output**

```
GET /ok        → 200 "OK"
GET /missing   → 404 { "error": "Not Found" }
Console: [ERROR] 404: Not Found
```

**Why This Output Occurs**

- The 404 catch-all converts unmatched routes into errors; the logging handler logs and delegates; the final handler sends the response.

---

### Real-World Cases

- **API gateways** — A chain of handlers for logging, metrics, and client response.
- **Content-negotiated error responses** — Different error handlers for HTML, JSON, and XML clients.
- **Unmatched route handling** — A 404 catch-all ensures all requests receive consistent error treatment.

---

## Core Concept 4: Multiple Error Handlers

### Definitions

**Core Definition**
Multiple error handlers are two or more error-handling middleware functions registered in sequence, each capable of inspecting, transforming, logging, or responding to an error before delegating to the next handler.

**Technical Definition**
For organizational purposes, you can define several error-handling middleware functions, much as you would with regular middleware functions. They are executed in the order they are registered (in `app.use()` statements). Each error handler must call `next(err)` to pass the error to the next error handler in the chain, unless it chooses to send a response, in which case the chain terminates. This pattern enables separation of concerns: one handler may log the error, another may format the response for a specific content type, and a final handler may send the response.

**Beginner-Friendly Explanation**
You can write more than one error handler. They run one after another, like a relay race. Each handler can do something with the error—like log it—and then pass it to the next handler. The last handler in the chain usually sends the response to the user.

---

### Purposes

- To separate error-processing concerns (logging, formatting, responding) into distinct, reusable middleware functions.
- To allow different error handlers to address specific error types or scenarios (e.g., validation errors vs. authentication errors).
- To enable a final catch-all handler while still allowing specialized handlers to act first.
- To maintain modularity and testability in error-handling logic.

---

### Syntax Rules and Structure

#### Complete General Syntax: Multiple Error Handlers

```javascript
app.use(logErrors);          // 1st: log the error
app.use(formatErrors);       // 2nd: format for response
app.use(sendErrorResponse);  // 3rd: send the response
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `logErrors` | Logs error details; calls `next(err)` |
| `formatErrors` | Transforms the error; calls `next(err)` |
| `sendErrorResponse` | Sends the final response; does not call `next()` |

#### Syntax Rules

- Every error handler must declare the four-argument signature `(err, req, res, next)`.
- Each handler must call `next(err)` to delegate unless it sends a response.
- Registration order determines execution order.
- A handler that does not call `next(err)` and does not send a response leaves the request hanging.

#### Constraints and Limitations

- If an error handler calls `next(err)` after sending a response, it can trigger “headers already sent” errors.
- Calling `next(err)` multiple times can trigger the default error handler erroneously.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Logging, Formatting, and Responding

```javascript
const express = require('express');
const app = express();

// Handler 1: log all errors
app.use((err, req, res, next) => {
  console.error(`[LOG] ${req.method} ${req.path}: ${err.message}`);
  next(err);
});

// Handler 2: add a timestamp to the error
app.use((err, req, res, next) => {
  err.timestamp = new Date().toISOString();
  next(err);
});

// Handler 3: send the final response
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    error: err.message,
    timestamp: err.timestamp
  });
});

app.listen(3000);
```

**Expected Output**

```
GET /fail (with error "Boom")
→ Console: [LOG] GET /fail: Boom
→ 500 { "error": "Boom", "timestamp": "2026-10-21T12:00:00.000Z" }
```

**Why This Output Occurs**

- The first handler logs, the second enriches the error object, and the third sends the response using the enriched error.

---

### Real-World Cases

- **Logging and monitoring** — A logging handler runs first, then a response handler sends the client response.
- **Content negotiation** — One handler formats errors for JSON, another for HTML.
- **Error classification** — A handler for validation errors and another for authentication errors.

---

## Core Concept 5: Centralized Error Handling

### Definitions

**Core Definition**
Centralized error handling is the practice of consolidating error processing into a small number of dedicated middleware functions, typically one final handler that formats and sends all error responses.

**Technical Definition**
Centralized error handling uses a custom error class (such as `AppError`) that carries a `statusCode` and an `isOperational` flag to distinguish expected operational errors (e.g., `404`, `401`, validation failures) from programming errors (bugs). A single global error handler inspects these properties: operational errors receive their specific status code and a safe message, while programming errors receive a generic `500` response, with the full stack logged server-side. This pattern also wraps async handlers (or relies on Express 5 automatic forwarding) to ensure all errors reach the centralized handler.

**Beginner-Friendly Explanation**
Centralized error handling means you write the error-response logic once, in one place. Instead of every route deciding what to do when something fails, routes just pass the error along. A single handler looks at the error, decides whether it is a user mistake (like a wrong ID) or a bug, sends the right status code, and logs the details for developers.

---

### Purposes

- To avoid duplicating error-response logic across every route handler.
- To distinguish operational errors from programming errors and respond appropriately.
- To prevent sensitive internal details from leaking to clients.
- To provide a single integration point for logging, monitoring, and alerting.

---

### Syntax Rules and Structure

#### Complete General Syntax: `AppError` and Global Handler

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Global error handler
app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';

  console.error({ status, message: err.message, stack: err.stack, path: req.path });

  res.status(status).json({ error: message });
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `AppError` | Custom error class with `statusCode` and `isOperational` |
| `isOperational` | Distinguishes expected errors from bugs |
| Global handler | Formats operational errors; hides programming error details |

#### Syntax Rules

- Operational errors should carry an appropriate `statusCode` (e.g., `404`, `401`, `400`).
- Programming errors should return a generic `500` and be logged with full stack traces.
- Async routes must forward rejections to the centralized handler (Express 5 automatic or Express 4 wrapper).
- The centralized handler must be registered last to catch all errors.

#### Constraints and Limitations

- Over-classifying errors as operational can mask bugs; programming errors should remain `500`.
- Centralized handlers cannot recover the request into normal flow; they can only respond or delegate.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Centralized Handler with Operational Error

```javascript
const express = require('express');
const app = express();

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

app.get('/user/:id', (req, res, next) => {
  if (req.params.id !== '1') {
    return next(new AppError('User not found', 404));
  }
  res.json({ id: 1, name: 'Alice' });
});

app.get('/bug', (req, res, next) => {
  throw new Error('Unexpected bug');
});

app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';
  console.error(err.stack);
  res.status(status).json({ error: message });
});

app.listen(3000);
```

**Expected Output**

```
GET /user/1  → 200 { "id": 1, "name": "Alice" }
GET /user/2  → 404 { "error": "User not found" }
GET /bug     → 500 { "error": "Internal Server Error" }
Console: Error: Unexpected bug (stack trace)
```

**Why This Output Occurs**

- `AppError` is operational, so the handler sends its `404` message; the unexpected bug is not operational, so the handler logs it and sends a generic `500`.

---

### Real-World Cases

- **REST APIs** — Consistent JSON error contracts with correct status codes.
- **Multi-tenant SaaS** — Operational errors reveal tenant-specific messages; bugs are hidden.
- **Security-sensitive applications** — Stack traces never reach clients in production.

---

## Core Concept 6: Error Handling Best Practices

### Definitions

**Core Definition**
Error-handling best practices are the design and implementation patterns that keep error responses secure, consistent, observable, and maintainable across an Express application.

**Technical Definition**
Best practices include: registering error handlers last with four-argument signatures; using a custom `AppError` class to distinguish operational from programming errors; logging full error details server-side while sending safe, generic messages to clients; relying on Express 5 automatic promise rejection forwarding (or async wrappers in Express 4); setting `NODE_ENV=production` to suppress stack traces; and using appropriate HTTP status codes (`400`, `401`, `403`, `404`, `500`) rather than returning `200` with error payloads.

**Beginner-Friendly Explanation**
Best practices are the habits that keep error handling safe and useful. Log the full error for yourself, but send only a safe message to the user. Use the right status code. Make sure async errors actually reach the handler. And never let a stack trace leak in production.

---

### Purposes

- To prevent sensitive information (stack traces, internal paths) from leaking to clients.
- To ensure all errors—synchronous, asynchronous, and from third-party APIs—are caught and handled consistently.
- To provide clients with actionable, correctly typed error responses.
- To make errors observable through structured logging and monitoring.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Error Pipeline

```javascript
// 1. Ensure production mode suppresses stack traces
// NODE_ENV=production in deployment

// 2. Centralized handler with safe responses
app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';

  // Log full details server-side
  console.error({ status, message: err.message, stack: err.stack, path: req.path });

  res.status(status).json({ error: message });
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Hide internal details | Generic `500` for non-operational errors |
| Log server-side | Full stack and request context logged, not sent |
| Correct status codes | Use `404`, `401`, `400` for operational errors |
| Production mode | `NODE_ENV=production` suppresses default stack traces |

#### Syntax Rules

- Never expose stack traces or internal error messages in production responses.
- Use appropriate HTTP status codes; do not return `200` for errors.
- Ensure async errors are forwarded to the centralized handler (Express 5 automatic; Express 4 wrapper).
- Log errors with enough context (route, status, request ID) to diagnose issues.

#### Constraints and Limitations

- The default error handler includes stack traces in development; `NODE_ENV=production` must be set before deployment.
- Logging libraries may need configuration to avoid logging sensitive request data.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Production-Ready Error Handler

```javascript
const express = require('express');
const app = express();

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

app.get('/user/:id', async (req, res, next) => {
  try {
    const user = await findUser(req.params.id);
    if (!user) throw new AppError('User not found', 404);
    res.json(user);
  } catch (err) {
    next(err);
  }
});

app.use((err, req, res, next) => {
  const status = err.isOperational ? err.statusCode : 500;
  const message = err.isOperational ? err.message : 'Internal Server Error';

  console.error({
    status,
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });

  res.status(status).json({ error: message });
});

app.listen(3000);
```

**Expected Output**

```
GET /user/1   → 200 (user data)
GET /user/99  → 404 { "error": "User not found" }
GET /crash    → 500 { "error": "Internal Server Error" }
```

**Why This Output Occurs**

- Operational errors keep their status and message; programming errors are replaced with a generic `500` message while full details are logged server-side.

---

### Real-World Cases

- **Production APIs** — Safe error responses, full server-side logging, and monitoring integration.
- **Security audits** — No stack traces or internal paths exposed to clients.
- **Multi-environment deployments** — Detailed errors in development, safe errors in production.

---

## References

- Error Handling · Express.js — https://expressjs.com/en/guide/error-handling/
- Error Handling · Express.js 5.x — https://expressjs.com/en/5x/guide/error-handling/
- Advanced Error Handling (raw documentation) — https://raw.githubusercontent.com/devfikdark/nodeJS-Professionals/master/book_pages/chapter3/section3.8.md
- Express for Node Error Handling and Tracking Done Right · AppSignal Blog — https://blog.appsignal.com/2025/07/16/express-for-node-error-handling-and-tracking-done-right.html
- How to handle errors in Express · CoreUI — https://coreui.io/answers/how-to-handle-errors-in-express/
- Error Handling · Express.js 5.x (Japanese) — https://expressjs.com/ja/guide/error-handling/
- Express 错误处理模式完全指南 · Guance — https://www.guance.com/learn/articles/error-handling-express
- Implement Global Error Handling Middleware & Custom Error Classes · GitHub Issue — https://github.com/Bernard065/hakika/issues/1
- Express Error Middleware · GitHub raw — https://raw.githubusercontent.com/devfikdark/nodeJS-Professionals/master/book_pages/chapter3/section3.8.md
- Distinguish operational vs programmer errors · Node.js Best Practices — https://raw.githubusercontent.com/goldbergyoni/nodebestpractices/master/sections/errorhandling/operationalvsprogrammererror.md