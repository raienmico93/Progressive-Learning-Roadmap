# Error Handling Architecture — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Error handling architecture is the systematic design of how an application detects, propagates, processes, and responds to errors that occur during request processing — ensuring that failures are caught consistently, logged appropriately, and communicated to clients without crashing the application.

**Technical Definition:** Error handling refers to how Express catches and processes errors that occur both synchronously and asynchronously. Express provides a default error handler so you don't need to write your own to get started. A robust error handling architecture distinguishes between **operational errors** (expected runtime failures such as invalid input, network timeouts, or resource not found) and **programmer errors** (bugs in the code such as reading a property of undefined). It centralises error processing in a single middleware function with the signature `(err, req, res, next)`, ensuring consistent responses across the entire application.

**Beginner-Friendly Explanation:** Error handling architecture is like the safety systems in a building. Smoke detectors (error detection) catch problems early. Sprinklers (error handling middleware) respond automatically. Emergency exits (error propagation) guide people out safely. And the fire department (logging and monitoring) investigates what happened. Without a proper architecture, a small fire (error) can burn down the whole building (crash the server). With it, the building stays standing and the problem is contained.

### Key Characteristics

- **Dual nature:** Express handles synchronous errors automatically but requires explicit handling for asynchronous errors in Express 4.
- **Centralised processing:** A single error-handling middleware processes all errors, ensuring consistent responses.
- **Error classification:** Errors are categorised as operational (expected) or programmer (bugs) to determine the appropriate response.
- **Custom error classes:** Extending the native `Error` object with properties like `statusCode` and `isOperational` enables structured error handling.
- **Middleware signature:** Error handlers are identified by their four-argument signature `(err, req, res, next)`.
- **Order-dependent:** The error handler must be registered **after** all routes and other middleware.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v10+ for Express 4.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, classes, callbacks, and asynchronous programming.
- **Understanding of Express middleware:** `req`, `res`, `next`, and the middleware chain.

### Related Programming Areas

- **Middleware Order:** Error-handling middleware must be registered last.
- **Custom Error Classes:** Extending `Error` for domain-specific errors.
- **Logging and Observability:** Structured logging of errors for debugging and monitoring.
- **Validation:** Validation errors are a common source of operational errors.
- **Process Management:** Handling `uncaughtException` and `unhandledRejection` for graceful shutdown.

### Core Concepts

1. **Synchronous Errors** — catching instant runtime faults natively within Express.
2. **Asynchronous Errors** — managing broken Promises, unhandled async operations, Express 5 vs Express 4.
3. **Custom Error Classes** — extending `Error` to build specific domain types (AppError, HttpError).
4. **Centralised Error Middleware** — a single point of truth `(err, req, res, next)` for all pipeline failures.
5. **Error Propagation** — understanding how exceptions bubble up; `next(err)` vs `throw`.
6. **Operational vs. Programmer Errors** — distinguishing expected failures from unpredictable bugs.

---

## Core Concept 1: Synchronous Errors

### Definitions

**Core Definition:** Synchronous errors are errors that occur immediately during the execution of a function, throwing an exception that interrupts the normal flow of the program.

**Technical Definition:** Errors that occur in synchronous code inside route handlers and middleware require no extra work. If synchronous code throws an error, Express will catch and process it automatically. This is because Express wraps synchronous handler invocations in a try-catch block internally. The thrown error is automatically passed to the error-handling middleware without requiring an explicit `next(err)` call.

**Beginner-Friendly Explanation:** Synchronous errors are like dropping a glass on the floor — it breaks immediately, and everyone hears it. Express automatically notices the break (the error) and sends it to the clean-up crew (the error handler). You don't need to do anything special.

### Purposes

- To catch instant script compilation and execution runtime faults natively within Express.
- To allow developers to use `throw new Error()` naturally in synchronous code.
- To provide automatic error propagation without boilerplate.
- To ensure that synchronous failures do not crash the process silently.

### Syntax Rules and Structure

```js
app.get('/', (req, res) => {
  throw new Error('BROKEN'); // Express will catch this on its own
});
```

| Component | Breakdown |
|-----------|-----------|
| `throw new Error()` | Throws a synchronous exception. |
| Express behaviour | Automatically catches and forwards to error handler. |
| No `next(err)` needed | Synchronous throws are handled natively. |

**Rules:**
- Any synchronous `throw` inside a route handler or middleware is caught by Express.
- The error is automatically passed to the error-handling middleware.
- No `try/catch` or `next(err)` is required for synchronous code.

**Constraints:**
- Synchronous throws only work for code executing in the same tick as the handler invocation.
- Asynchronous code (Promises, callbacks) requires explicit error handling.

### Annotated Code Example

```js
// synchronous-errors.js
const express = require('express');
const app = express();

// Synchronous throw — Express catches it automatically
app.get('/crash', (req, res) => {
  throw new Error('Something went wrong');
});

// Synchronous error in middleware
app.use((req, res, next) => {
  if (!req.headers.authorization) {
    throw new Error('No authorization header');
  }
  next();
});

// Error handler
app.use((err, req, res, next) => {
  console.error('Error caught:', err.message);
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /crash`):**
```
Error caught: Something went wrong
HTTP/1.1 500 Internal Server Error
{"error":"Something went wrong"}
```

**Why this output:** The route handler throws a synchronous error. Express automatically catches it and forwards it to the error-handling middleware. The error handler logs the message and sends a JSON response.

### Real-World Cases

- **Validation failures:** Throwing a synchronous error when input validation fails.
- **Programming bugs:** Null reference errors or undefined property access.
- **Configuration errors:** Missing environment variables detected at request time.

---

## Core Concept 2: Asynchronous Errors

### Definitions

**Core Definition:** Asynchronous errors are errors that occur in code that does not execute immediately — such as Promises, `async/await` functions, or callback-based operations — and require explicit handling to be forwarded to Express's error pipeline.

**Technical Definition:** For errors returned from asynchronous functions invoked by route handlers and middleware, you must pass them to the `next()` function, where Express will catch and process them. Starting with Express 5, route handlers and middleware that return a Promise will call `next(value)` automatically when they reject or throw an error. In Express 4, unhandled rejections in async middleware crash the process or leave the request hanging.

**Beginner-Friendly Explanation:** Asynchronous errors are like a delayed reaction. If you drop a glass but catch it mid-air, the break happens later — and Express might not notice if you don't tell it. In Express 5, Express has a net that catches everything. In Express 4, you have to shout "I dropped it!" (call `next(err)`) for Express to know.

### Purposes

- To manage broken Promises and unhandled async operations.
- To use Express 5's native async forwarding for automatic error propagation.
- To provide manual wrappers (like `express-async-errors`) for Express 4 compatibility.
- To prevent silent failures where async errors disappear without a response.

### Sub-Feature 2.1: Express 5 — Native Async Error Catching

#### Syntax Rules and Structure

```js
// Express 5 — This just works
app.get('/users', async (req, res) => {
  const users = await db.getUsers(); // If this rejects, error handler catches it
  res.json(users);
});
```

| Component | Breakdown |
|-----------|-----------|
| `async` handler | Returns a Promise. |
| Rejection | Automatically forwarded to `next(err)`. |
| No wrapper needed | Express 5 handles it natively. |

**Rules:**
- If the Promise rejects, `next(value)` is called with the rejected value.
- If no rejected value is provided, `next` is called with a default Error object.
- No `try/catch` or wrapper function is required.

---

### Sub-Feature 2.2: Express 4 — Manual Wrappers Required

#### Syntax Rules and Structure

```js
// Express 4 — DANGEROUS
app.get('/users', async (req, res) => {
  const users = await db.getUsers(); // Rejection = unhandled crash!
  res.json(users);
});
```

```js
// Express 4 — SAFE with wrapper
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users', asyncHandler(async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
}));
```

| Approach | Error Handling |
|----------|---------------|
| No wrapper | Unhandled rejection → crash or hang. |
| `try/catch` + `next(err)` | Explicit handling in every handler. |
| `asyncHandler` wrapper | Automatic `.catch(next)` for all async handlers. |
| `express-async-errors` | Monkey-patches Express to catch async errors globally. |

**Rules:**
- Express 4 does **not** catch rejected Promises automatically.
- Every async handler needs a `try/catch` block or a wrapper function.
- The wrapper catches rejections and forwards them to `next(err)`.

### Annotated Code Example

```js
// async-errors.js
const express = require('express');
const app = express();

// Simulated async database
const db = {
  getUsers: async () => {
    throw new Error('Database connection failed');
  }
};

// Express 5 — native async error catching
app.get('/users', async (req, res) => {
  const users = await db.getUsers(); // Rejection auto-forwarded
  res.json(users);
});

// Error handler
app.use((err, req, res, next) => {
  console.error('Async error caught:', err.message);
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users`):**
```
Async error caught: Database connection failed
HTTP/1.1 500 Internal Server Error
{"error":"Database connection failed"}
```

**Why this output:** The async handler awaits `db.getUsers()`, which rejects. In Express 5, the rejection is automatically forwarded to `next(err)`, which triggers the error-handling middleware. The error handler logs the message and sends a JSON response.

### Real-World Cases

- **Database queries:** `await db.query()` in an async route handler.
- **External API calls:** `await fetch('https://api.example.com')` in middleware.
- **File I/O:** `await fs.promises.readFile()` in a route handler.

---

## Core Concept 3: Custom Error Classes

### Definitions

**Core Definition:** Custom error classes are user-defined classes that extend the native JavaScript `Error` object, adding properties such as `statusCode` and `isOperational` to carry structured metadata about the error.

**Technical Definition:** Custom error classes extend `Error` (e.g., `AppError`) with properties for `statusCode` and `isOperational`. The `isOperational` flag distinguishes expected operational errors (which can be handled gracefully) from programmer errors (which indicate bugs). The `Error.captureStackTrace` method is used to exclude the constructor from the stack trace.

**Beginner-Friendly Explanation:** Custom error classes are like labelled containers for different types of problems. Instead of throwing a generic "Error," you throw a "NotFoundError" or "ValidationError" — each with its own label (status code) and instructions for how to handle it.

### Purposes

- To extend the native JavaScript `Error` object to build specific domain types (e.g., `AppError`, `HttpError`).
- To carry HTTP status codes and error codes alongside error messages.
- To distinguish operational errors from programmer errors via the `isOperational` flag.
- To enable the error-handling middleware to respond appropriately based on error type.

### Syntax Rules and Structure

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode || 500;
    this.isOperational = true; // Indicates this is a known operational error
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(message, details) {
    super(message, 422);
    this.details = details;
  }
}
```

| Component | Breakdown |
|-----------|-----------|
| `extends Error` | Inherits from the native Error object. |
| `super(message)` | Calls the parent constructor with the message. |
| `this.statusCode` | HTTP status code for the response. |
| `this.isOperational` | Flag distinguishing expected errors from bugs. |
| `Error.captureStackTrace` | Excludes the constructor from the stack trace. |

**Rules:**
- Custom error classes should extend `Error` and call `super(message)`.
- The `statusCode` should be a valid HTTP status code.
- The `isOperational` flag should be `true` for expected errors and `false` (or absent) for bugs.
- Use `Error.captureStackTrace(this, this.constructor)` to keep stack traces clean.

**Constraints:**
- Custom error classes must be instantiated with `new`.
- The `instanceof` check should be used to identify error types in the error handler.

### Annotated Code Example

```js
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
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

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

module.exports = { AppError, NotFoundError, UnauthorizedError };
```

```js
// services/user.service.js
const { NotFoundError } = require('../errors/AppError');

class UserService {
  async findById(id) {
    const user = await db.findById(id);
    if (!user) throw new NotFoundError('User');
    return user;
  }
}
```

```js
// middleware/errorHandler.js
const { AppError } = require('../errors/AppError');

function errorHandler(err, req, res, next) {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: err.message
    });
  }
  console.error('PROGRAMMER ERROR:', err);
  res.status(500).json({
    success: false,
    error: 'Internal server error'
  });
}
```

**Expected Output (for a missing user):**
```json
{
  "success": false,
  "error": "User not found"
}
```

**Why this output:** The `NotFoundError` extends `AppError` with a 404 status code and `isOperational: true`. The error handler checks the `isOperational` flag and responds with the appropriate status code and message.

### Real-World Cases

- **REST APIs:** `NotFoundError` for 404 responses, `ValidationError` for 422 responses.
- **Authentication:** `UnauthorizedError` for 401 responses, `ForbiddenError` for 403 responses.
- **E-commerce:** `OutOfStockError` for 409 responses, `PaymentFailedError` for 402 responses.

---

## Core Concept 4: Centralised Error Middleware

### Definitions

**Core Definition:** Centralised error middleware is a single Express middleware function with the signature `(err, req, res, next)` that processes all errors from the entire application pipeline uniformly.

**Technical Definition:** Error-handling middleware functions are defined in the same way as other middleware functions, except error-handling functions have four arguments instead of three: `(err, req, res, next)`. This middleware must be registered **after** all other `app.use()` and route definitions. It differentiates between operational errors (400/404) and programmer errors (500), sending structured JSON responses.

**Beginner-Friendly Explanation:** Centralised error middleware is like a hospital's emergency room. No matter what kind of injury (error) comes in — a broken bone (validation error), a heart attack (database failure), or a paper cut (404) — the ER (error handler) assesses it and provides the appropriate treatment. Instead of each department handling its own emergencies, there's one place for all of them.

### Purposes

- To build a single point of truth endpoint `(err, req, res, next)` to process all pipeline failures uniformly.
- To ensure consistent error responses across all endpoints.
- To differentiate between operational errors and programmer errors.
- To log errors centrally for debugging and monitoring.
- To prevent unhandled errors from crashing the application.

### Syntax Rules and Structure

```js
function errorHandler(err, req, res, next) {
  // Log the error
  console.error(err.stack);

  // Operational errors (expected)
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: err.message
    });
  }

  // Programmer errors (bugs)
  res.status(500).json({
    success: false,
    error: 'Internal server error'
  });
}

// Register LAST
app.use(errorHandler);
```

| Component | Breakdown |
|-----------|-----------|
| `(err, req, res, next)` | Four arguments — Express detects error middleware by arity. |
| `err.isOperational` | Flag distinguishing expected errors from bugs. |
| `err.statusCode` | HTTP status code attached to the error. |
| `app.use(errorHandler)` | Must be registered after all routes. |

**Rules:**
- Error-handling middleware **must** have exactly four arguments.
- It must be registered **after** all other `app.use()` and route definitions.
- If registered too early, route errors will not be caught.
- The handler should not expose stack traces in production responses.

**Constraints:**
- Error handlers should distinguish between operational and programmer errors.
- Multiple error handlers can be defined for different error types.
- If an error handler calls `next(err)`, it passes to the next error handler.

### Annotated Code Example

```js
// app.js
const express = require('express');
const app = express();
const { AppError } = require('./errors/AppError');

// Routes
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userService.findById(req.params.id);
    if (!user) throw new AppError('User not found', 404);
    res.json({ data: user });
  } catch (error) {
    next(error); // Delegate to global handler
  }
});

app.get('/crash', (req, res) => {
  throw new Error('Unexpected bug'); // Programmer error
});

// Centralised error handler — MUST be last
app.use((err, req, res, next) => {
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    timestamp: new Date().toISOString()
  });

  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: err.message
    });
  }

  res.status(500).json({
    success: false,
    error: 'Internal server error'
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users/999`):**
```json
{
  "success": false,
  "error": "User not found"
}
```

**Expected Output (for `GET /crash`):**
```
console.error output:
{ message: 'Unexpected bug', stack: '...', url: '/crash', method: 'GET', timestamp: '...' }

HTTP Response:
{"success": false, "error": "Internal server error"}
```

**Why this output:** The error handler checks `err.isOperational`. For the operational `AppError` (user not found), it returns the specific message. For the programmer error (unexpected bug), it logs the full stack and returns a generic 500 response to avoid leaking internal details.

### Real-World Cases

- **REST APIs:** Consistent error responses across all endpoints.
- **Multi-service architectures:** Centralised error formatting for microservices.
- **Production applications:** Logging errors for debugging while hiding details from clients.

---

## Core Concept 5: Error Propagation

### Definitions

**Core Definition:** Error propagation is the mechanism by which errors travel through the middleware chain — either by throwing (for synchronous code) or by passing to `next(err)` (for asynchronous code) — until they reach the error-handling middleware.

**Technical Definition:** In general, Express follows the way of passing errors rather than throwing it, for any errors in the program you can pass the error object to `next`. Per the docs, `throw` and `next(err)` basically do the same thing in synchronous code. However, `return next(err)` instead of `throw err` allows asynchronous code to raise an exception and still have it caught by the error handling pipeline.

**Beginner-Friendly Explanation:** Error propagation is like a relay race where the baton is the error. In synchronous code, you can simply throw the baton and Express will catch it. In asynchronous code, you have to hand the baton to the next runner (call `next(err)`) — if you just throw it into the air, Express might not see it.

### Purposes

- To understand how exceptions bubble up through middleware chains.
- To know when to use `next(err)` vs. throwing directly.
- To ensure that errors are always caught by the error-handling middleware.
- To avoid silent failures where errors disappear without a response.

### Syntax Rules and Structure

```js
// Synchronous: throw works
app.get('/sync', (req, res) => {
  throw new Error('Sync error'); // Express catches it
});

// Asynchronous: next(err) required
app.get('/async', async (req, res, next) => {
  try {
    await someAsyncOperation();
    res.json({ ok: true });
  } catch (err) {
    next(err); // Pass to error handler
  }
});

// Or with Express 5: throw works
app.get('/async5', async (req, res) => {
  throw new Error('Async error'); // Express 5 catches it
});
```

| Context | Mechanism | Express Catches? |
|---------|-----------|------------------|
| Synchronous handler | `throw new Error()` | ✅ Yes |
| Synchronous middleware | `throw new Error()` | ✅ Yes |
| Async (Express 4) | `throw new Error()` | ❌ No |
| Async (Express 4) | `next(err)` | ✅ Yes |
| Async (Express 5) | `throw new Error()` | ✅ Yes |

**Rules:**
- Synchronous code: `throw` works and is equivalent to `next(err)`.
- Asynchronous code (Express 4): `next(err)` is required — `throw` will not be caught.
- Asynchronous code (Express 5): `throw` works natively.
- `next('route')` skips to the next route handler, not the error handler.

**Constraints:**
- Throwing inside a callback (e.g., `fs.readFile`) will not be caught by Express.
- Always use `return next(err)` to prevent further execution after passing the error.

### Annotated Code Example

```js
// propagation.js
const express = require('express');
const app = express();

// Synchronous throw
app.get('/sync-error', (req, res) => {
  throw new Error('Synchronous failure');
});

// Async with next(err)
app.get('/async-error', async (req, res, next) => {
  try {
    await Promise.reject(new Error('Async failure'));
  } catch (err) {
    next(err);
  }
});

// Callback-based error
app.get('/callback-error', (req, res, next) => {
  setTimeout(() => {
    try {
      throw new Error('Callback failure');
    } catch (err) {
      next(err);
    }
  }, 100);
});

// Error handler
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /sync-error`):**
```json
{"error":"Synchronous failure"}
```

**Expected Output (for `GET /async-error`):**
```json
{"error":"Async failure"}
```

**Expected Output (for `GET /callback-error`):**
```json
{"error":"Callback failure"}
```

**Why this output:** The synchronous route throws, and Express catches it. The async route catches the rejection and passes it to `next(err)`. The callback route wraps the throw in a try/catch and passes the error to `next(err)`. All three errors reach the error handler.

### Real-World Cases

- **Database operations:** `try/catch` around `await db.query()` with `next(err)`.
- **External API calls:** Catching fetch errors and passing to `next(err)`.
- **File I/O:** Wrapping `fs.readFile` callbacks in try/catch with `next(err)`.

---

## Core Concept 6: Operational vs. Programmer Errors

### Definitions

**Core Definition:** Operational errors are expected runtime failures that correctly-written programs should handle; programmer errors are bugs in the code that indicate the program itself is broken.

**Technical Definition:** Operational errors represent runtime problems whose results are expected and should be dealt with in a proper way. Operational errors don't mean the application itself has bugs, but developers need to handle them thoughtfully. Programmer errors represent unexpected bugs in poorly written code. They mean the code itself has some issues to solve and was coded wrong. A good example of a programmer error is trying to read a property of `undefined`.

**Beginner-Friendly Explanation:** Operational errors are like a customer ordering a dish that's sold out — it's an expected problem, and the restaurant handles it politely. Programmer errors are like the chef forgetting to turn on the stove — it's a mistake in the kitchen that needs to be fixed. The restaurant can handle a sold-out dish gracefully, but a broken stove requires turning everything off and fixing the problem.

### Purposes

- To distinguish between expected operational failures (e.g., resource not found, invalid validation) and unpredictable programmer bugs (e.g., undefined reference crashes).
- To determine the appropriate response: operational errors get specific status codes; programmer errors get a generic 500.
- To guide recovery strategy: operational errors can be retried or handled; programmer errors require a restart.
- To improve debugging by logging programmer errors with full stack traces.

### Syntax Rules and Structure

| Category | Examples | Response | Recovery |
|----------|----------|----------|----------|
| Operational | Invalid input, 404, timeout, DB connection failure | Specific status code (400, 404, etc.) | Retry, graceful degradation |
| Programmer | `undefined` property access, type errors, logic bugs | Generic 500 | Crash and restart |

```js
// Error handler distinguishing types
app.use((err, req, res, next) => {
  if (err.isOperational) {
    // Operational: send specific message
    return res.status(err.statusCode).json({ error: err.message });
  }

  // Programmer: log full stack, send generic message
  console.error('PROGRAMMER ERROR:', err);
  res.status(500).json({ error: 'Internal server error' });
});
```

**Rules:**
- Operational errors should be instantiated with `AppError` and `isOperational: true`.
- Programmer errors should be allowed to bubble up to the error handler.
- The error handler should differentiate based on the `isOperational` flag.
- Best practice for programmer errors is to crash immediately and let a restarter restart the application.

**Constraints:**
- Not all errors fall neatly into one category; some are ambiguous.
- Unhandled operational errors can become programmer errors if not handled properly.

### Annotated Code Example

```js
// operational-vs-programmer.js
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

// Operational error
app.get('/user/:id', async (req, res, next) => {
  const user = await db.findById(req.params.id);
  if (!user) {
    return next(new AppError('User not found', 404));
  }
  res.json(user);
});

// Programmer error
app.get('/bug', (req, res) => {
  const obj = undefined;
  console.log(obj.property); // TypeError: Cannot read property of undefined
});

// Error handler
app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message,
      type: 'operational'
    });
  }

  console.error('PROGRAMMER ERROR:', err.stack);
  res.status(500).json({
    error: 'Internal server error',
    type: 'programmer'
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /user/999` — operational):**
```json
{"error":"User not found","type":"operational"}
```

**Expected Output (for `GET /bug` — programmer):**
```
PROGRAMMER ERROR: TypeError: Cannot read property 'property' of undefined
    at ... (stack trace)

HTTP Response:
{"error":"Internal server error","type":"programmer"}
```

**Why this output:** The operational error (user not found) has `isOperational: true` and a 404 status code, so the error handler responds with the specific message. The programmer error (TypeError) has no `isOperational` flag, so the handler logs the full stack trace and returns a generic 500 response to avoid leaking internal details.

### Real-World Cases

- **E-commerce:** `OutOfStockError` (operational) vs. a bug in the pricing calculation (programmer).
- **Authentication:** `InvalidCredentialsError` (operational) vs. a missing `req.user` assignment (programmer).
- **File uploads:** `FileTooLargeError` (operational) vs. a bug in the upload middleware (programmer).

---

## References

- Express.js — Error Handling Guide — https://expressjs.com/en/guide/error-handling.html
- Express.js 4.x — Error Handling — https://expressjs.com/en/4x/guide/error-handling.html
- Express 5 Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- MDN — Express/Node Introduction — https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction
- Node.js — Errors — https://nodejs.org/api/errors.html
- Joyent — Node.js Error Handling Best Practices — https://www.joyent.com/node-js/production/design/errors
- Stack Overflow — throw Error vs next(error) — https://stackoverflow.com/questions/27794750/
- Stack Overflow — Error handling in ExpressJS middlewares — https://stackoverflow.com/questions/74908911/
- OneUptime — How to Handle Error Handling Properly in Express — https://oneuptime.com/blog/post/2026-02-02-error-handling-express/view
- OneUptime — How to Implement Error Handling in Express — https://oneuptime.com/blog/post/2026-01-26-error-handling-express/view
- CoreUI — How to handle errors globally in Node.js — https://coreui.io/answers/how-to-handle-errors-globally-in-nodejs/
- DEV Community — Express Middleware Patterns: Composition, Error Handling, and Auth — https://dev.to
- TRAE-Skills — Error Handling (Express) — https://github.com/HighMark-31/TRAE-Skills
- GitHub — WalletWise Issue #184: Centralized Error Handling — https://github.com/SoumyaMishra-7/WalletWise/issues/184
- GitHub — Typed-Error Middleware with Express — https://github.com
- @point-hub/express-error-handler — npm — https://www.npmjs.com/package/@point-hub/express-error-handler
- express-error-toolkit — npm — https://www.npmjs.com/package/express-error-toolkit
- error-cure — npm — https://www.npmjs.com/package/error-cure
- oops-error — npm — https://www.npmjs.com/package/oops-error
- @minikit/terror — npm — https://www.npmjs.com/package/@minikit/terror
- Node.js Best Practices — Distinguish Operational vs Programmer Errors — https://github.com/goldbergyoni/nodebestpractices
- Stackify — Node.js Error Handling Best Practices — https://stackify.com/node-js-error-handling/