# Express.js Middleware Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Middleware functions are functions that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application's request-response cycle. An Express application is essentially a series of middleware function calls. 

**Technical Definition:** Middleware functions execute sequentially in the order they are registered using `app.use()` or HTTP method functions like `app.get()`. Each middleware function can execute code, modify the request and response objects, end the request-response cycle, or call `next()` to pass control to the next middleware. If the current middleware does not end the cycle, it must call `next()` or the request will be left hanging. Express supports application-level, router-level, error-handling, built-in, and third-party middleware. 

**Beginner-Friendly Explanation:** Think of an Express application as an assembly line. Each request enters the line and passes through a series of stations (middleware functions). At each station, something happens: a logger records the request, an authenticator checks credentials, a parser reads the body. Each station either passes the request to the next station (by calling `next()`) or stops the line by sending a response back to the client. This assembly line is how Express processes every request.

### Key Characteristics

- **Sequential execution:** Middleware runs in the exact order it is registered. 
- **Access to req, res, and next:** Every middleware function receives these three arguments. 
- **Terminal or pass-through:** Middleware either ends the request-response cycle (by sending a response) or passes control via `next()`. 
- **Mutable objects:** Middleware can add custom properties to `req` (e.g., `req.user`) and modify `res`. 
- **Error propagation:** Passing an argument to `next(err)` skips all remaining non-error middleware and jumps to error-handling middleware. 
- **Express 5 native async support:** Route handlers and middleware returning a Promise automatically call `next(value)` on rejection. 

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, callbacks, and asynchronous programming.
- **Understanding of HTTP:** Requests, responses, and the request-response lifecycle.

### Related Programming Areas

- **Routing:** Middleware and route handlers share the same function signature.
- **Error handling:** Error-handling middleware has a 4-argument signature `(err, req, res, next)`.
- **Authentication:** Middleware verifies credentials and attaches user data to `req`.
- **Logging:** Middleware records request metadata for debugging and monitoring.
- **Body parsing:** Middleware like `express.json()` parses request bodies into `req.body`.
- **CORS:** Middleware sets cross-origin headers.

### Core Concepts

1. **What Middleware Is** — functions with access to req, res, and next.
2. **Middleware Execution Flow** — sequential execution until a response is sent.
3. **req** — the HTTP request object (mutations, custom properties).
4. **res** — the HTTP response object (methods and ending the cycle).
5. **next** — the callback to pass control; `next(err)` for errors.
6. **Middleware Chaining** — executing multiple functions for a route or globally.
7. **Asynchronous Middleware** — async/await in Express 5 vs. Express 4.

---

## Core Concept 1: What Middleware Is

### Definitions

**Core Definition:** Middleware functions are functions that have access to the request object, the response object, and the next middleware function in the application's request-response cycle. 

**Technical Definition:** An Express application is essentially a series of middleware function calls. Middleware functions can execute any code, make changes to the request and response objects, end the request-response cycle, or call the next middleware function in the stack. If the current middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function. 

**Beginner-Friendly Explanation:** Middleware is like a series of security checkpoints at an airport. Each checkpoint examines you (the request), and either lets you proceed to the next checkpoint (by calling `next()`) or stops you and sends you back (by ending the response). Each checkpoint has a specific job: one checks your ID, another scans your bags, another verifies your boarding pass.

### Purposes

- To execute any code during the request-response cycle.
- To make changes to the request and response objects.
- To end the request-response cycle.
- To call the next middleware function in the stack.
- To modularise request processing into reusable, composable units.

### Syntax Rules and Structure

```js
function middlewareName(req, res, next) {
  // Execute code
  // Modify req or res
  // Either: res.send(...) to end the cycle
  // Or:     next() to pass control
}
```

| Component | Breakdown |
|-----------|-----------|
| `req` | The HTTP request object. |
| `res` | The HTTP response object. |
| `next` | Callback to pass control to the next middleware. |
| Return | Middleware should not return a value; it either calls `next()` or ends the response. |

**Rules:**
- Middleware functions must accept `(req, res, next)` — three arguments.
- Error-handling middleware must accept `(err, req, res, next)` — four arguments.
- Middleware can be mounted globally with `app.use()`, on a path with `app.use('/path', ...)`, or on a specific route.
- Order matters: middleware executes in the order it is registered. 

**Constraints:**
- If `next()` is never called and the response is never ended, the request hangs indefinitely. 
- Middleware registered after a route handler will not execute for that route if the handler ends the cycle.

### Annotated Code Example

```js
// middleware-basic.js
const express = require('express');
const app = express();

// Middleware function 1: Logging
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();  // Pass control to the next middleware
}

// Middleware function 2: Authentication
function authenticate(req, res, next) {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).send('Unauthorized');  // Ends the cycle
  }
  req.user = { id: 1, name: 'Alice' };  // Mutate req
  next();  // Pass control
}

// Register middleware globally
app.use(logger);
app.use(authenticate);

// Route handler
app.get('/profile', (req, res) => {
  res.json({ user: req.user });  // Access the custom property
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `GET /profile` with `Authorization: Bearer token`):**
```
GET /profile
{"user":{"id":1,"name":"Alice"}}
```

**Expected Output (for `GET /profile` without token):**
```
GET /profile
HTTP/1.1 401 Unauthorized

Unauthorized
```

**Why this output:** The `logger` middleware runs first, logs the request method and URL, then calls `next()`. The `authenticate` middleware checks for the `Authorization` header. If present, it attaches a user object to `req` and calls `next()`. If absent, it sends a 401 response and does not call `next()`, terminating the cycle.

### Real-World Cases

- **Logging:** Recording request method, URL, and timestamp for every request.
- **Authentication:** Verifying JWT tokens before allowing access to protected routes.
- **Body parsing:** Parsing JSON or URL-encoded bodies into `req.body`.
- **CORS:** Setting cross-origin headers for API access from browsers.
- **Rate limiting:** Counting requests per IP and rejecting excess requests.

---

## Core Concept 2: Middleware Execution Flow

### Definitions

**Core Definition:** Middleware execution flow is the sequential, ordered processing of a request through the middleware stack until a response is sent or the cycle is terminated. 

**Technical Definition:** Express executes middleware functions in the order they are added using `app.use()` or HTTP method functions. When a request is made, Express processes middleware from top to bottom, invoking each middleware function sequentially. Each middleware must either end the request-response cycle or call `next()` to pass control to the next function. 

**Beginner-Friendly Explanation:** Imagine a relay race. The baton (the request) is passed from runner to runner (middleware function). Each runner either passes the baton to the next runner (calls `next()`) or stops the race by crossing the finish line (sending a response). The order of the runners is fixed — they run in the order they were assigned.

### Purposes

- To control the order in which request processing occurs.
- To ensure that dependencies (e.g., body parsing) run before handlers that need them.
- To provide a predictable, traceable request lifecycle.
- To enable conditional short-circuiting of the request (e.g., rejecting unauthorised requests early).

### Syntax Rules and Structure

```
Client Request → Middleware 1 → Middleware 2 → ... → Route Handler → Response
```

| Stage | Description |
|-------|-------------|
| Client Request | The incoming HTTP request. |
| Middleware 1 | First registered middleware; executes first. |
| Middleware 2 | Second registered middleware; executes after Middleware 1 calls `next()`. |
| Route Handler | The final function that sends the response. |
| Response | The HTTP response sent to the client. |

**Rules:**
- Middleware is executed in the order it is registered. 
- A middleware function must call `next()` to proceed to the next function. 
- If a middleware ends the response (e.g., `res.send()`), subsequent middleware and route handlers do not execute.
- Error-handling middleware (4 arguments) is skipped during normal flow and only invoked when `next(err)` is called. 

**Common Middleware Order Best Practices:** 

1. Request parsing middleware (e.g., `express.json()`)
2. Logging middleware
3. Authentication middleware
4. Authorisation middleware
5. Route handlers
6. Error-handling middleware (last)

### Annotated Code Example

```js
// middleware-flow.js
const express = require('express');
const app = express();

// Middleware 1: Logging
app.use((req, res, next) => {
  console.log('1. Logger: Request received');
  next();
});

// Middleware 2: Authentication
app.use((req, res, next) => {
  console.log('2. Auth: Checking credentials');
  req.user = { id: 1 };
  next();
});

// Middleware 3: Data fetching (simulated)
app.use((req, res, next) => {
  console.log('3. Data: Fetching user data');
  req.data = { name: 'Alice' };
  next();
});

// Route handler
app.get('/user', (req, res) => {
  console.log('4. Handler: Sending response');
  res.json({ user: req.user, data: req.data });
});

// Error-handling middleware (4 arguments)
app.use((err, req, res, next) => {
  console.error('Error:', err.message);
  res.status(500).send('Something broke!');
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /user`):**
```
1. Logger: Request received
2. Auth: Checking credentials
3. Data: Fetching user data
4. Handler: Sending response
```

**Expected Output (HTTP response):**
```json
{"user":{"id":1},"data":{"name":"Alice"}}
```

**Why this output:** The middleware functions execute in registration order. Each calls `next()` to pass control to the next function. The route handler is the final function in the chain and sends the response. The console output demonstrates the sequential execution flow.

### Real-World Cases

- **Request lifecycle tracing:** Logging the execution order to debug middleware issues.
- **Dependency management:** Ensuring `express.json()` runs before route handlers that read `req.body`.
- **Security layering:** Running authentication before authorisation before business logic.
- **Performance monitoring:** Timing each middleware to identify bottlenecks.

---

## Core Concept 3: req — The Request Object

### Definitions

**Core Definition:** The `req` object represents the HTTP request and contains properties for the request query string, parameters, body, HTTP headers, and other request metadata. 

**Technical Definition:** The `req` object is an enhanced version of Node's own request object and supports all built-in fields and methods. Middleware can mutate `req` by adding custom properties (e.g., `req.user`, `req.data`), which are then available to subsequent middleware and route handlers. 

**Beginner-Friendly Explanation:** `req` is like a folder containing everything about the incoming request: who sent it, what they want, and any data they included. Middleware can add notes to this folder (like "this user is authenticated") that later middleware can read.

### Purposes

- To provide access to request data (params, query, body, headers).
- To allow middleware to attach custom properties for downstream use.
- To enable request-scoped state sharing between middleware functions.
- To provide metadata for logging, authentication, and routing decisions.

### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  req.customProperty = 'value';  // Attach custom data
  next();
});

app.get('/route', (req, res) => {
  res.json({ value: req.customProperty });  // Access custom data
});
```

| Property | Description |
|----------|-------------|
| `req.params` | Route parameters (e.g., `:id`). |
| `req.query` | Query string parameters. |
| `req.body` | Parsed request body. |
| `req.headers` | Request headers. |
| `req.customProperty` | Custom property added by middleware. |

**Rules:**
- Custom properties should have descriptive names to avoid conflicts.
- `req` is shared across all middleware and handlers for a single request.
- Changes to `req` are scoped to that request only.

### Annotated Code Example

```js
// req-mutation.js
const express = require('express');
const app = express();

// Middleware that adds a custom property to req
app.use((req, res, next) => {
  req.requestTime = new Date().toISOString();
  req.userId = req.headers['x-user-id'] || 'anonymous';
  next();
});

app.get('/info', (req, res) => {
  res.json({
    requestTime: req.requestTime,
    userId: req.userId,
    method: req.method,
    path: req.path
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /info` with `X-User-Id: 42`):**
```json
{
  "requestTime": "2026-01-15T10:30:00.000Z",
  "userId": "42",
  "method": "GET",
  "path": "/info"
}
```

**Why this output:** The middleware adds `requestTime` and `userId` to the `req` object. The route handler reads these custom properties along with built-in properties like `req.method` and `req.path`.

### Real-World Cases

- **Authentication:** `req.user = decodedToken` after verifying a JWT.
- **Logging:** `req.requestId = generateId()` for request tracing.
- **Localisation:** `req.language = detectLanguage(req)` for i18n.
- **Database connections:** `req.db = getConnection()` for per-request connections.

---

## Core Concept 4: res — The Response Object

### Definitions

**Core Definition:** The `res` object represents the HTTP response that an Express app sends when it receives an HTTP request. 

**Technical Definition:** The `res` object is an enhanced version of Node's own response object and supports all built-in fields and methods. Middleware and route handlers use `res` methods to send responses, set headers, and end the request-response cycle. Common methods include `res.send()`, `res.json()`, `res.status()`, and `res.end()`. 

**Beginner-Friendly Explanation:** `res` is your toolkit for sending something back to the client. You can send text, JSON, HTML, or a file. You can set the status code, add headers, and then send the response. Once you send a response, the request-response cycle is complete.

### Purposes

- To send a response to the client (text, JSON, HTML, file, or stream).
- To set the HTTP status code and headers.
- To end the request-response cycle.
- To provide a consistent interface for response delivery.

### Syntax Rules and Structure

```js
res.send('Hello');          // Send text
res.json({ key: 'value' }); // Send JSON
res.status(404).send('Not Found'); // Set status and send
res.end();                  // End without body
```

| Method | Purpose |
|--------|---------|
| `res.send()` | Send a response of any type. |
| `res.json()` | Send a JSON response. |
| `res.status()` | Set the HTTP status code. |
| `res.end()` | End the response without data. |

**Rules:**
- Only one response can be sent per request.
- Calling a response method ends the cycle; no further middleware executes.
- `res.status()` returns `res` for chaining.

### Annotated Code Example

```js
// res-methods.js
const express = require('express');
const app = express();

app.get('/text', (req, res) => {
  res.send('Plain text response');
});

app.get('/json', (req, res) => {
  res.json({ message: 'JSON response', success: true });
});

app.get('/error', (req, res) => {
  res.status(500).json({ error: 'Something went wrong' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /text`):**
```
Plain text response
```

**Expected Output (for `GET /json`):**
```json
{"message":"JSON response","success":true}
```

**Expected Output (for `GET /error`):**
```
HTTP/1.1 500 Internal Server Error
{"error":"Something went wrong"}
```

**Why this output:** Each route handler uses a different `res` method to send the appropriate response. `res.send()` sends plain text, `res.json()` serialises the object to JSON, and `res.status(500).json()` sets the status code and sends a JSON error response.

### Real-World Cases

- **REST APIs:** `res.json()` to return data as JSON.
- **HTML pages:** `res.send()` to return rendered HTML.
- **File downloads:** `res.download()` to send a file as an attachment.
- **Redirects:** `res.redirect()` to redirect the client to another URL.

---

## Core Concept 5: next — The Middleware Callback

### Definitions

**Core Definition:** `next` is a callback function that, when called, passes control to the next middleware function in the stack. 

**Technical Definition:** The `next` function is a function in the Express router which, when invoked, executes the middleware succeeding the current middleware. If the current middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function. Otherwise, the request will be left hanging. Passing an argument to `next(err)` (except the string `'route'`) causes Express to regard the current request as an error and skip all remaining non-error middleware, jumping to the error-handling middleware. 

**Beginner-Friendly Explanation:** `next` is like a baton in a relay race. When a middleware function is done with its work, it hands the baton to the next function by calling `next()`. If something goes wrong, it can throw the baton to the error handler by calling `next(error)`.

### Purposes

- To pass control to the next middleware function.
- To propagate errors to error-handling middleware via `next(err)`.
- To skip remaining route handlers via `next('route')`.
- To exit the current router via `next('router')`.

### Sub-Feature 5.1: `next()` — Continue to Next Middleware

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  // Do something
  next();  // Pass control to the next middleware
});
```

**Rules:**
- Call `next()` when your middleware is done and the request should continue.
- Do not call `next()` if you have ended the response.

---

### Sub-Feature 5.2: `next(err)` — Pass to Error Handler

#### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  if (errorCondition) {
    return next(new Error('Something went wrong'));
  }
  next();
});
```

| Argument | Effect |
|----------|--------|
| `next()` | Continue to the next non-error middleware. |
| `next(err)` | Skip to error-handling middleware. |
| `next('route')` | Skip remaining handlers for the current route. |
| `next('router')` | Exit the current router. |

**Rules:**
- When `next(err)` is called with any value except `'route'`, Express skips all remaining non-error middleware and invokes the first error-handling middleware. 
- Error-handling middleware must have four arguments: `(err, req, res, next)`.
- If no error-handling middleware is defined, Express uses its default error handler.

### Annotated Code Example

```js
// next-errors.js
const express = require('express');
const app = express();

// Middleware that passes an error
app.get('/error', (req, res, next) => {
  const err = new Error('Database connection failed');
  err.status = 500;
  next(err);  // Jump to error handler
});

// Normal route
app.get('/ok', (req, res) => {
  res.send('All good');
});

// Error-handling middleware (4 arguments)
app.use((err, req, res, next) => {
  console.error('Error caught:', err.message);
  res.status(err.status || 500).json({
    error: err.message
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /ok`):**
```
All good
```

**Expected Output (for `GET /error`):**
```
Error caught: Database connection failed
HTTP/1.1 500 Internal Server Error
{"error":"Database connection failed"}
```

**Why this output:** The `/error` route creates an error and calls `next(err)`. Express skips all remaining non-error middleware and jumps directly to the error-handling middleware (the 4-argument function). The error handler logs the message and sends a JSON response.

### Real-World Cases

- **Database errors:** `next(err)` when a query fails.
- **Validation errors:** `next(validationError)` when input is invalid.
- **Authentication errors:** `next(new Error('Unauthorized'))` when a token is invalid.
- **Route skipping:** `next('route')` to bypass a route handler when a condition is not met.

---

## Core Concept 6: Middleware Chaining

### Definitions

**Core Definition:** Middleware chaining is the practice of executing multiple middleware functions sequentially for a single route or globally. 

**Technical Definition:** You can load a series of middleware functions together by passing them as separate arguments to `app.use()` or `app.METHOD()`. This creates a sub-stack of the middleware system at a mount point. Each function in the chain receives `(req, res, next)` and must call `next()` to proceed to the next function. 

**Beginner-Friendly Explanation:** Middleware chaining is like an assembly line with multiple stations. Each station does one specific task — one validates input, one checks permissions, one fetches data — and then passes the work to the next station. You can attach multiple stations to a single route or to the entire application.

### Purposes

- To execute multiple middleware functions sequentially for a single route.
- To reuse common middleware (logging, authentication) across multiple routes.
- To compose complex request processing from simple, focused functions.
- To apply middleware globally, on a path, or on a specific route.

### Syntax Rules and Structure

```js
// Chaining multiple middleware functions
app.get('/route',
  middleware1,
  middleware2,
  middleware3,
  (req, res) => {
    res.send('Response');
  }
);

// Chaining with an array
app.get('/route', [middleware1, middleware2], handler);

// Global chaining
app.use(middleware1);
app.use(middleware2);
```

| Component | Breakdown |
|-----------|-----------|
| `middleware1`, `middleware2` | Functions with signature `(req, res, next)`. |
| Array | Middleware functions can be grouped in an array. |
| `app.use()` | Registers middleware globally or on a path. |

**Rules:**
- Middleware functions execute in the order they are listed.
- Each function must call `next()` to proceed (unless it ends the response).
- The final function in the chain is typically the route handler.
- Arrays can be used to group middleware for reusability.

### Annotated Code Example

```js
// middleware-chaining.js
const express = require('express');
const app = express();

// Middleware 1: Validate API key
function validateApiKey(req, res, next) {
  if (!req.headers['x-api-key']) {
    return res.status(401).json({ error: 'API key required' });
  }
  next();
}

// Middleware 2: Check rate limit
function checkRateLimit(req, res, next) {
  console.log('Rate limit checked');
  next();
}

// Middleware 3: Log request
function logRequest(req, res, next) {
  console.log(`${req.method} ${req.path}`);
  next();
}

// Route with chained middleware
app.get('/api/data',
  validateApiKey,
  checkRateLimit,
  logRequest,
  (req, res) => {
    res.json({ data: 'protected resource' });
  }
);

// Reusable middleware array
const commonMiddleware = [logRequest, checkRateLimit];

app.get('/api/public', commonMiddleware, (req, res) => {
  res.json({ data: 'public resource' });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /api/data` with `X-API-Key: abc123`):**
```
Rate limit checked
GET /api/data
{"data":"protected resource"}
```

**Expected Output (for `GET /api/data` without API key):**
```json
{"error":"API key required"}
```

**Why this output:** The chained middleware functions execute in order. `validateApiKey` checks for the API key; if missing, it ends the cycle with a 401 response. If present, `checkRateLimit` and `logRequest` run, then the final handler sends the JSON response.

### Real-World Cases

- **API authentication:** `validateToken` → `checkPermissions` → `handler`.
- **Form processing:** `parseBody` → `validateInput` → `sanitizeData` → `handler`.
- **Logging and analytics:** `logRequest` → `trackMetrics` → `handler`.
- **Reusable middleware:** Arrays of common middleware applied to multiple routes.

---

## Core Concept 7: Asynchronous Middleware

### Definitions

**Core Definition:** Asynchronous middleware handles async operations (database queries, API calls, file I/O) using `async/await` or Promises. Express 5 natively supports async middleware by automatically forwarding rejections to error handlers; Express 4 requires explicit try/catch or wrapper functions. 

**Technical Definition:** Starting with Express 5, route handlers and middleware that return a Promise will call `next(value)` automatically when they reject or throw an error. If no rejected value is provided, `next` will be called with a default Error object provided by the Express router. In Express 4, Express does not catch errors from async functions automatically — every async route handler needs a `try-catch` block, or a wrapper function that handles this. 

**Beginner-Friendly Explanation:** Asynchronous middleware is middleware that does work that takes time — like waiting for a database to respond. In Express 5, if something goes wrong, the error is automatically passed to your error handler. In Express 4, you have to catch the error yourself and pass it along, or the request will hang.

### Sub-Feature 7.1: Express 5 — Native Async Support

#### Syntax Rules and Structure

```js
app.get('/users', async (req, res) => {
  const users = await db.getUsers();  // If this rejects, error handler catches it
  res.json(users);
});
```

| Feature | Description |
|---------|-------------|
| Native `async` support | Rejections auto-forwarded to error middleware. |
| No wrapper needed | Simply `throw` or let the Promise reject. |
| Error handling | Errors are caught and passed to `next(err)` automatically. |

**Rules:**
- Express 5 automatically calls `next(value)` when a Promise rejects or throws an error. 
- If no rejected value is provided, `next` is called with a default Error object. 
- You can still use `try/catch` for custom error handling.

---

### Sub-Feature 7.2: Express 4 — Wrapper Required

#### Syntax Rules and Structure

```js
// Wrapper function for async error handling
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users', asyncHandler(async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
}));
```

**Rules:**
- Express 4 does not catch errors from async functions automatically. 
- Every async route handler needs a `try-catch` block or a wrapper function.
- The wrapper catches rejections and forwards them to `next(err)`.

### Annotated Code Example

```js
// async-middleware.js
const express = require('express');
const app = express();

// Simulated async database
const db = {
  getUsers: async () => {
    return [{ id: 1, name: 'Alice' }];
  },
  fail: async () => {
    throw new Error('Database connection failed');
  }
};

// Express 5: Native async support (no wrapper needed)
app.get('/users', async (req, res) => {
  const users = await db.getUsers();
  res.json(users);
});

app.get('/fail', async (req, res) => {
  await db.fail();  // Error is automatically forwarded
  res.send('This will not execute');
});

// Error-handling middleware
app.use((err, req, res, next) => {
  console.error('Error:', err.message);
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users`):**
```json
[{"id":1,"name":"Alice"}]
```

**Expected Output (for `GET /fail`):**
```
Error: Database connection failed
HTTP/1.1 500 Internal Server Error
{"error":"Database connection failed"}
```

**Why this output:** In Express 5, the async handler for `/fail` throws an error. Express automatically catches it and forwards it to the error-handling middleware. The error handler logs the message and sends a JSON response. No wrapper function is needed.

### Real-World Cases

- **Database queries:** `await db.query()` in an async route handler.
- **External API calls:** `await fetch('https://api.example.com')` in middleware.
- **File I/O:** `await fs.promises.readFile()` in a route handler.
- **Authentication:** `await verifyToken(token)` in auth middleware.

---

## References

- Express.js — Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- Express.js 5.x — Error Handling — https://expressjs.com/en/guide/error-handling.html
- Express.js 4.x — Error Handling — https://expressjs.com/en/4x/guide/error-handling.html
- Express.js — Writing Middleware — https://expressjs.com/en/guide/writing-middleware.html
- Express.js 5.x API — req — https://expressjs.com/en/5x/api.html#req
- Express.js 5.x API — res — https://expressjs.com/en/5x/api.html#res
- Express.js 5.x API — app.use() — https://expressjs.com/en/5x/api.html#app.use
- Express.js 5.x API — app.METHOD() — https://expressjs.com/en/5x/api.html#app.METHOD
- Express.js — Router-level Middleware — https://expressjs.com/en/guide/using-middleware.html#middleware.router
- Express.js — Error-handling Middleware — https://expressjs.com/en/guide/using-middleware.html#middleware.error-handling
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- MDN — Express/Node Introduction — https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction
- async-express-error (npm) — https://www.npmjs.com/package/async-express-error
- express-handler-async (npm) — https://www.npmjs.com/package/express-handler-async