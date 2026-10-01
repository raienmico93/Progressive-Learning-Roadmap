# Express.js Basic Routing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Routing in Express.js is the mechanism by which an application's endpoints (URIs) respond to client requests, defined by associating an HTTP method and a URL path with one or more handler functions.

**Technical Definition:** Express routing uses methods of the `app` object (or a `Router` instance) that correspond to HTTP verbs — `app.get()`, `app.post()`, `app.put()`, `app.delete()`, etc. — to register callback functions that are executed when an incoming request matches the specified route path and HTTP method. Express translates route path strings into regular expressions internally using the `path-to-regexp` library to match incoming request URLs. Query strings are not considered when performing route path matches.

**Beginner-Friendly Explanation:** Routing is like a receptionist at an office building. When someone walks in (a request), the receptionist looks at two things: what they want to do (the HTTP method) and where they want to go (the URL path). Based on those two pieces of information, the receptionist directs them to the right person (the handler function) who knows how to help. If no one matches, the visitor gets a "404 Not Found."

### Key Characteristics

- **Method-driven:** Routes are defined by HTTP methods (GET, POST, PUT, DELETE, PATCH, etc.).
- **Path-based matching:** Routes match URL paths using strings, string patterns, or regular expressions.
- **Middleware chaining:** Multiple handler functions can be chained for a single route, each acting as middleware.
- **Dynamic parameters:** URL segments can be captured as named parameters (`:id`) accessible via `req.params`.
- **Query string support:** Non-hierarchical key-value pairs appended to the URL are accessible via `req.query`.
- **`app.all()` for catch-all:** A special method handles all HTTP methods for a given path.
- **Router modularity:** Routes can be organised into `express.Router()` instances for modular applications.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, callbacks, and `async/await`.
- **Understanding of HTTP methods:** GET, POST, PUT, DELETE, PATCH.
- **Familiarity with the request–response cycle** in web applications.

### Related Programming Areas

- **Middleware:** Route handlers are a specialised form of middleware.
- **REST API design:** Routing is the foundation of RESTful API architecture.
- **Template engines:** Routes render views using engines like EJS or Pug.
- **Database integration:** Routes call database operations and return results.
- **Error handling:** Routes pass errors to Express's error-handling middleware.
- **Authentication:** Routes use middleware to verify user identity before processing.

### Core Concepts

1. **Route Definitions** — mapping HTTP endpoints to execution blocks.
2. **HTTP Methods** — CRUD verbs and `app.all()`.
3. **Route Paths** — string paths, string patterns, and regular expressions.
4. **Route Handlers** — chaining handlers and async/await.
5. **Route Parameters** — dynamic URL segments via `req.params`.
6. **Query Parameters** — URL query strings via `req.query`.

---

## Core Concept 1: Route Definitions

### Definitions

**Core Definition:** A route definition maps a specific HTTP method and URL path to one or more handler functions that execute when a matching request is received.

**Technical Definition:** A route is created using `app.METHOD(path, [callback...], callback)`, where `METHOD` is a lowercase HTTP verb (e.g., `get`, `post`). The `path` can be a string, string pattern, regular expression, or an array of those. The callback(s) are functions that receive `(req, res, next)` and are executed in order when the route matches.

**Beginner-Friendly Explanation:** A route definition is like a rule in a receptionist's manual: "If someone comes in asking for the customer list and they use the word GET, send them to the person who handles that." You're telling your server: "When a request with this method and this path arrives, run this code."

### Purposes

- To map unique HTTP endpoints to deterministic server execution blocks.
- To organise application logic into discrete, reusable units.
- To enable RESTful API design by associating CRUD operations with specific URLs.
- To provide a clear, declarative structure for handling client requests.

### Syntax Rules and Structure

#### General Syntax

```js
app.METHOD(path, handler);
app.METHOD(path, handler1, handler2, ...);
```

| Component | Breakdown |
|-----------|-----------|
| `app` | The Express application instance. |
| `METHOD` | Lowercase HTTP verb (`get`, `post`, `put`, `delete`, `patch`, `all`). |
| `path` | String, string pattern, RegExp, or array of paths. |
| `handler` | Function `(req, res, next) => {}` or `async (req, res, next) => {}`. |
| Multiple handlers | Executed in order; must call `next()` to proceed. |

#### Syntax Rules

- HTTP method names must be **lowercase** (`app.get()`, not `app.GET()`).
- The callback function receives the request object, response object, and (optionally) the `next` function.
- Multiple callbacks are treated equally and behave like middleware; call `next()` to pass control to the next callback.
- To route methods that translate to invalid JavaScript variable names, use bracket notation: `app['m-search']('/', ...)`.

#### Constraints and Limitations

- Express does not support HTTP method names that are not valid JavaScript identifiers without bracket notation.
- Route paths are matched in the order they are defined; the first match wins.
- By default, Express routing is **not case-sensitive**, so `/Home` and `/home` match the same route. To disable this, set `app.set('case sensitive routing', true)`.
- Trailing slashes are ignored by default (e.g., `/foo` and `/foo/` match the same route).

### Annotated Code Example

```js
// routes-basic.js
const express = require('express');
const app = express();

// Route for GET requests to the root path
app.get('/', (req, res) => {
  res.send('Home Page');                     // Sends "Home Page" to the client
});

// Route for GET requests to /about
app.get('/about', (req, res) => {
  res.send('About Page');
});

// Route for POST requests to /submit
app.post('/submit', (req, res) => {
  res.status(201).send('Submitted successfully');
});

// Route for DELETE requests to /items/1
app.delete('/items/1', (req, res) => {
  res.send('Item deleted');
});

// Start the server
app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

**Expected Output (when requesting `GET /`):**
```
Home Page
```

**Why this output:** The client sends a GET request to `/`. Express matches the first route definition (`app.get('/')`) and executes its handler, which sends the string `'Home Page'` as the response. The other routes are not executed because their paths do not match.

### Real-World Cases

- **Homepage and static pages:** `app.get('/', ...)` for the homepage, `app.get('/about', ...)` for an about page.
- **API endpoints:** `app.get('/api/users', ...)` to list users, `app.post('/api/users', ...)` to create a user.
- **Form submissions:** `app.post('/contact', ...)` to handle contact form submissions.
- **Resource deletion:** `app.delete('/api/posts/:id', ...)` to delete a blog post.

---

## Core Concept 2: HTTP Methods

### Definitions

**Core Definition:** HTTP methods (verbs) define the type of action a client wants to perform on a resource. Express provides routing methods that correspond to each HTTP method.

**Technical Definition:** Express supports routing methods corresponding to all HTTP request methods: `get`, `post`, `put`, `delete`, `patch`, `head`, `options`, `trace`, `connect`, and `all`. These are attached to the `app` object (or a `Router` instance). The special method `app.all()` loads middleware for **all** HTTP request methods at a given path.

**Beginner-Friendly Explanation:** HTTP methods are like verbs in a sentence. GET means "give me something," POST means "create something," PUT means "replace something," PATCH means "update part of something," and DELETE means "remove something." Express lets you tell the server what to do for each verb.

### Purposes

- To implement core CRUD (Create, Read, Update, Delete) operations in a RESTful API.
- To differentiate actions on the same URL path (e.g., GET `/users` vs. POST `/users`).
- To apply middleware to all HTTP methods on a path using `app.all()`.
- To handle CORS preflight `OPTIONS` requests.

### Sub-Feature 2.1: Core CRUD Verbs (GET, POST, PUT, DELETE, PATCH)

#### Definitions

**Core Definition:** The five primary HTTP methods used in RESTful APIs to perform CRUD operations on resources.

**Technical Definition:** `GET` retrieves data; `POST` creates a new resource; `PUT` replaces an entire resource; `PATCH` partially updates a resource; `DELETE` removes a resource. Express exposes each as a lowercase method on the `app` object.

**Beginner-Friendly Explanation:** These five verbs are like the five basic operations you can do with a contact in your phone: look them up (GET), add a new one (POST), replace all their info (PUT), change just their phone number (PATCH), or delete them (DELETE).

#### Syntax Rules and Structure

```js
app.get(path, handler);
app.post(path, handler);
app.put(path, handler);
app.patch(path, handler);
app.delete(path, handler);
```

| Component | Breakdown |
|-----------|-----------|
| `path` | The URL path to match. |
| `handler` | `(req, res) => {}` — the function to execute. |

#### Annotated Code Example

```js
// crud-routes.js
const express = require('express');
const app = express();

app.use(express.json()); // Parse JSON request bodies

let users = [{ id: 1, name: 'Alice' }];

// READ — list all users
app.get('/users', (req, res) => {
  res.json(users);                           // Returns the array as JSON
});

// CREATE — add a new user
app.post('/users', (req, res) => {
  const newUser = { id: users.length + 1, name: req.body.name };
  users.push(newUser);
  res.status(201).json(newUser);             // 201 Created
});

// UPDATE (replace) — replace a user entirely
app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).send('User not found');
  user.name = req.body.name;
  res.json(user);
});

// UPDATE (partial) — patch a user
app.patch('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).send('User not found');
  if (req.body.name) user.name = req.body.name;
  res.json(user);
});

// DELETE — remove a user
app.delete('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  users = users.filter(u => u.id !== id);
  res.status(204).send();                    // 204 No Content
});

app.listen(3000, () => console.log('CRUD API on port 3000'));
```

**Expected Output (for `POST /users` with `{ "name": "Bob" }`):**
```
HTTP/1.1 201 Created
Content-Type: application/json

{"id":2,"name":"Bob"}
```

**Why this output:** The POST route reads `req.body.name`, creates a new user object with the next ID, pushes it into the `users` array, and responds with status 201 and the new user as JSON.

#### Real-World Cases

- **Social media API:** GET `/posts` (list), POST `/posts` (create), PUT `/posts/:id` (edit), DELETE `/posts/:id` (remove).
- **E-commerce:** GET `/products`, POST `/cart`, PATCH `/cart/:itemId` (update quantity), DELETE `/cart/:itemId`.
- **Task manager:** GET `/tasks`, POST `/tasks`, PATCH `/tasks/:id` (mark complete), DELETE `/tasks/:id`.

---

### Sub-Feature 2.2: `app.all()` — Catch-All Methods and OPTIONS Preflight

#### Definitions

**Core Definition:** `app.all()` is a special routing method that matches **all** HTTP request methods for a given path.

**Technical Definition:** `app.all(path, callback...)` loads middleware functions at a path for all HTTP request methods. It is useful for applying authentication, logging, or CORS headers to every request on a path regardless of the verb used. Express also automatically responds to `OPTIONS` requests for routes defined with `app.get()`, but `app.all()` can be used to customise this behaviour.

**Beginner-Friendly Explanation:** `app.all()` is like a security guard at a building entrance who checks everyone's ID — whether they're coming in to deliver a package (POST), pick something up (GET), or remove furniture (DELETE). The check happens no matter what they're there to do.

#### Purposes

- To apply middleware (authentication, logging, CORS) to all HTTP methods on a path.
- To handle CORS preflight `OPTIONS` requests explicitly.
- To define catch-all routes for undefined methods on a specific path.
- To simplify code when the same pre-processing applies to multiple methods.

#### Syntax Rules and Structure

```js
app.all(path, handler);
app.all('*', handler); // Matches all paths and all methods
```

| Component | Breakdown |
|-----------|-----------|
| `path` | The path to match (`'*'` for all paths). |
| `handler` | Middleware or handler function. |
| `'*'` | Wildcard matching every path. |

**Constraints:**
- `app.all()` adds an `OPTIONS` handler by default, which can interfere with custom OPTIONS handling if not managed carefully.
- `app.all()` matches the **complete path**, whereas `app.use()` matches path prefixes.

#### Annotated Code Example

```js
// app-all-options.js
const express = require('express');
const app = express();

// app.all() — runs for every HTTP method on /secret
app.all('/secret', (req, res, next) => {
  console.log('Accessing the secret section...');
  next(); // Pass control to the next handler
});

app.get('/secret', (req, res) => {
  res.send('Secret GET handler');
});

app.post('/secret', (req, res) => {
  res.send('Secret POST handler');
});

// CORS preflight handling with app.options()
app.options('/api/data', (req, res) => {
  res.set('Access-Control-Allow-Origin', '*');
  res.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.set('Access-Control-Allow-Headers', 'Content-Type');
  res.sendStatus(204); // No Content
});

app.get('/api/data', (req, res) => {
  res.json({ message: 'Data retrieved' });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (when sending `DELETE /secret`):**
```
Accessing the secret section...
```

**Why this output:** `app.all('/secret')` matches the DELETE request because it handles all HTTP methods. The middleware logs the message and calls `next()`. Since no `app.delete('/secret')` route is defined, the request eventually falls through to Express's default 404 handler (not shown).

**Expected Output (for `OPTIONS /api/data`):**
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type
```

**Why this output:** The `app.options()` route intercepts the preflight OPTIONS request and responds with the appropriate CORS headers and a 204 status, allowing the browser to proceed with the actual request.

#### Real-World Cases

- **Authentication middleware:** `app.all('/admin/*', requireAuth)` to protect all admin routes regardless of HTTP method.
- **API versioning:** `app.all('/api/v1/*', logApiRequest)` to log all API calls.
- **CORS preflight:** `app.options('*', cors())` to handle preflight requests globally.
- **Maintenance mode:** `app.all('*', (req, res) => res.status(503).send('Maintenance'))` to block all traffic.

---

## Core Concept 3: Route Paths

### Definitions

**Core Definition:** A route path is the URL pattern that Express uses to match incoming request URLs.

**Technical Definition:** Route paths can be **strings**, **string patterns**, or **regular expressions**. Express uses the `path-to-regexp` library to convert string paths into regular expressions for matching. Query strings are not considered during path matching.

**Beginner-Friendly Explanation:** A route path is like a street address pattern. A simple string path is a specific address (`/about`). A string pattern is a range of addresses (`/users/:id` matches any user ID). A regular expression is a complex rule for matching addresses that follow a specific format.

### Purposes

- To match exact URL paths using literal strings.
- To match dynamic URL segments using named parameters.
- To match complex URL patterns using regular expressions.
- To validate URL structure before processing a request.

### Sub-Feature 3.1: String Paths

#### Definitions

**Core Definition:** String paths match request URLs exactly (or with optional trailing slashes) using literal characters.

**Technical Definition:** A string path like `'/about'` matches only the exact URL path `/about`. Express's default case-insensitive, trailing-slash-tolerant matching applies unless disabled.

**Beginner-Friendly Explanation:** A string path is like a specific house address: `/contact` means "only match requests for the contact page."

#### Syntax Rules and Structure

```js
app.get('/about', handler);
app.get('/users/list', handler);
```

| Component | Breakdown |
|-----------|-----------|
| `'/about'` | Literal string path. |
| `'/users/list'` | Nested literal path. |

#### Annotated Code Example

```js
// string-paths.js
const express = require('express');
const app = express();

app.get('/', (req, res) => res.send('Root'));
app.get('/about', (req, res) => res.send('About'));
app.get('/contact', (req, res) => res.send('Contact'));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /about`):**
```
About
```

**Why this output:** The path `'/about'` matches the literal URL path `/about`. Express compares the incoming URL path to the registered route paths and executes the first match.

---

### Sub-Feature 3.2: String Patterns

#### Definitions

**Core Definition:** String patterns use special characters (`:`, `?`, `+`, `*`, `()`) to create flexible route paths that match multiple URLs.

**Technical Definition:** Express supports pattern characters in string paths: `?` (optional), `+` (one or more), `*` (zero or more), and `()` (grouping). Route parameters are defined with a colon prefix (`:paramName`). These patterns are compiled by `path-to-regexp` into regular expressions.

**Beginner-Friendly Explanation:** String patterns are like using wildcards in a search. `/ab?cd` matches `acd` or `abcd`. `/users/:id` matches `/users/123` or `/users/alice` — the `:id` part is a placeholder for any value.

#### Syntax Rules and Structure

```js
app.get('/ab?cd', handler);     // Matches /acd and /abcd
app.get('/ab+cd', handler);     // Matches /abcd, /abbcd, /abbbcd, ...
app.get('/ab*cd', handler);     // Matches /abcd, /abxcd, /abRANDOMcd, ...
app.get('/ab(cd)?e', handler);  // Matches /abe and /abcde
```

| Pattern | Meaning |
|---------|---------|
| `?` | Optional preceding character. |
| `+` | One or more preceding character. |
| `*` | Zero or more of any character. |
| `()` | Groups characters. |

**Constraints:**
- In Express 5.x, wildcard `*` parameters must be **named** (e.g., `/*splat`), and `{}` is required for optional segments. Unnamed wildcards are not supported.

#### Annotated Code Example

```js
// string-patterns.js
const express = require('express');
const app = express();

app.get('/ab?cd', (req, res) => res.send('Matched ab?cd'));
app.get('/ab+cd', (req, res) => res.send('Matched ab+cd'));
app.get('/ab*cd', (req, res) => res.send('Matched ab*cd'));
app.get('/ab(cd)?e', (req, res) => res.send('Matched ab(cd)?e'));

app.listen(3000, () => console.log('Pattern server on 3000'));
```

**Expected Output (for `GET /abcd`):**
```
Matched ab?cd
```

**Why this output:** The route `/ab?cd` matches `/abcd` because `?` makes the `b` optional. Express checks routes in order, so the first matching route (`/ab?cd`) responds. If you requested `/abbcd`, the `/ab+cd` route would match instead.

---

### Sub-Feature 3.3: Regular Expression Paths

#### Definitions

**Core Definition:** Regular expression paths allow precise control over which URLs match a route by defining a custom pattern.

**Technical Definition:** Express accepts a RegExp object as the route path. Captured groups in the regex are available as `req.params[0]`, `req.params[1]`, etc. This is useful for very specific constraints, such as matching commit hashes or version numbers.

**Beginner-Friendly Explanation:** A regex path is like a very precise rule: "Match only URLs that look like a commit hash range, such as `/commits/71dbb9c..4c084f9`."

#### Syntax Rules and Structure

```js
app.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, (req, res) => {
  const from = req.params[0];
  const to = req.params[1] || 'HEAD';
  res.send(`Commit range ${from}..${to}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `/^\/commits\/(\w+)(?:\.\.(\w+))?$/` | RegExp matching `/commits/<hash>` or `/commits/<hash>..<hash>`. |
| `req.params[0]` | First captured group (from hash). |
| `req.params[1]` | Second captured group (to hash, optional). |

#### Annotated Code Example

```js
// regex-paths.js
const express = require('express');
const app = express();

app.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, (req, res) => {
  const from = req.params[0];                // First captured group
  const to = req.params[1] || 'HEAD';        // Second group or default
  res.send(`Commit range ${from}..${to}`);
});

app.listen(3000, () => console.log('Regex server on 3000'));
```

**Expected Output (for `GET /commits/71dbb9c`):**
```
Commit range 71dbb9c..HEAD
```

**Expected Output (for `GET /commits/71dbb9c..4c084f9`):**
```
Commit range 71dbb9c..4c084f9
```

**Why this output:** The regex captures the first hash as `req.params[0]`. The second hash is optional (`(?:\.\.(\w+))?`); if absent, `req.params[1]` is `undefined`, and the fallback `'HEAD'` is used.

#### Real-World Cases

- **API versioning:** `/^\/api\/v(\d+)\//` to capture the version number.
- **File extensions:** `/\.(png|jpg|jpeg)$/` to match image requests.
- **Date-based URLs:** `/^\/archive\/(\d{4})\/(\d{2})\/(\d{2})$/` to capture year, month, and day.

---

## Core Concept 4: Route Handlers

### Definitions

**Core Definition:** Route handlers are the callback functions that execute when a route matches an incoming request.

**Technical Definition:** A route handler is a function with the signature `(req, res, next)` that processes the request and sends a response. Multiple handlers can be provided for a single route; they are executed in order, and each must call `next()` to pass control to the next handler. Handlers can be synchronous or asynchronous (`async` functions).

**Beginner-Friendly Explanation:** Route handlers are the workers who actually do the job. When a request arrives at the right desk (route), the handler is the person who processes the request — looking up data, saving information, or sending back a response. You can have multiple workers in a chain, each doing a small part.

### Purposes

- To process incoming requests and generate responses.
- To chain multiple functions for a single route (pre-conditions, validation, then action).
- To handle synchronous and asynchronous operations cleanly.
- To reuse middleware functions across multiple routes.

### Sub-Feature 4.1: Chaining Multiple Handler Functions

#### Definitions

**Core Definition:** Multiple handler functions can be registered for a single route, forming a processing chain where each handler can modify the request/response or pass control to the next.

**Technical Definition:** `app.METHOD(path, handler1, handler2, ...)` registers multiple callbacks. All are treated equally and behave like middleware. The only exception is that a handler may call `next('route')` to bypass the remaining handlers for the current route and pass control to subsequent routes.

**Beginner-Friendly Explanation:** Chaining handlers is like an assembly line. Each worker does one small task — one checks if you're logged in, the next loads your data, the next validates it, and the final one sends the response.

#### Syntax Rules and Structure

```js
app.get('/user/:id',
  (req, res, next) => { /* pre-condition */ next(); },
  (req, res, next) => { /* validation */ next(); },
  (req, res) => { /* final response */ }
);
```

| Component | Breakdown |
|-----------|-----------|
| First handler | Often loads or validates data. |
| `next()` | Passes control to the next handler. |
| `next('route')` | Skips remaining handlers for this route. |
| Final handler | Sends the response (does not call `next()`). |

#### Annotated Code Example

```js
// chained-handlers.js
const express = require('express');
const app = express();

// Simulated user database
const users = { 1: { name: 'Alice', role: 'admin' } };

// Pre-condition: load user from database
function loadUser(req, res, next) {
  const user = users[req.params.id];
  if (!user) return res.status(404).send('User not found');
  req.user = user;                           // Attach to request object
  next();                                    // Proceed to next handler
}

// Validation: check permissions
function requireAdmin(req, res, next) {
  if (req.user.role !== 'admin') {
    return res.status(403).send('Forbidden');
  }
  next();
}

// Final handler
app.get('/user/:id',
  loadUser,                                  // Step 1
  requireAdmin,                              // Step 2
  (req, res) => {                            // Step 3
    res.json({ message: `Hello, ${req.user.name}` });
  }
);

app.listen(3000, () => console.log('Chained handlers on 3000'));
```

**Expected Output (for `GET /user/1`):**
```
{"message":"Hello, Alice"}
```

**Why this output:** `loadUser` finds the user and attaches it to `req.user`, then calls `next()`. `requireAdmin` checks the role and calls `next()`. The final handler sends the JSON response. If the user did not exist, `loadUser` would have sent a 404 and never called `next()`, stopping the chain.

---

### Sub-Feature 4.2: Synchronous vs. Async/Await Handlers

#### Definitions

**Core Definition:** Synchronous handlers execute blocking code and send a response immediately. Asynchronous handlers use `async/await` to perform non-blocking operations and must handle errors explicitly.

**Technical Definition:** Express does **not** natively handle Promise rejections from `async` handlers. An unhandled rejection will not be passed to Express's error-handling middleware unless the handler is wrapped or uses `try/catch`. Synchronous handlers that throw an error are automatically caught by Express.

**Beginner-Friendly Explanation:** A synchronous handler is like a cashier who takes your order, makes your coffee, and hands it to you — all in one continuous action. An async handler is like a cashier who takes your order, gives you a buzzer, and serves other customers while your coffee is being made. If something goes wrong with the coffee machine (an error), the async handler needs a way to tell the manager (error middleware).

#### Syntax Rules and Structure

```js
// Synchronous handler
app.get('/sync', (req, res) => {
  const data = calculateSomething(); // Blocking
  res.json(data);
});

// Async handler with try/catch
app.get('/async', async (req, res) => {
  try {
    const data = await fetchData();    // Non-blocking
    res.json(data);
  } catch (err) {
    res.status(500).send('Server error');
  }
});

// Async handler with wrapper
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/wrapped', asyncHandler(async (req, res) => {
  const data = await fetchData();
  res.json(data);
}));
```

| Approach | Error Handling |
|----------|---------------|
| Sync | Thrown errors caught by Express automatically. |
| Async + `try/catch` | Explicit error handling; send response directly. |
| Async + wrapper | Errors passed to Express error middleware via `next(err)`. |

**Constraints:**
- **Never** use async handlers without error handling; unhandled rejections can crash the process.
- Synchronous blocking operations in handlers block the entire event loop.

#### Annotated Code Example

```js
// async-handlers.js
const express = require('express');
const app = express();

// Async handler with try/catch
app.get('/users/:id', async (req, res) => {
  try {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/users/${req.params.id}`
    );
    if (!response.ok) throw new Error('User not found');
    const user = await response.json();
    res.json(user);
  } catch (err) {
    console.error(err.message);
    res.status(500).send('Failed to fetch user');
  }
});

// Async handler with wrapper (cleaner)
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/posts/:id', asyncHandler(async (req, res) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${req.params.id}`
  );
  const post = await response.json();
  res.json(post);
}));

// Error-handling middleware (must have 4 arguments)
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send('Something broke!');
});

app.listen(3000, () => console.log('Async handlers on 3000'));
```

**Expected Output (for `GET /users/1`):**
```
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  ...
}
```

**Why this output:** The `async` handler fetches data from an external API. `await` pauses execution until the fetch completes. The JSON response is sent to the client. If the fetch fails, the `catch` block sends a 500 error.

#### Real-World Cases

- **Database queries:** `await db.query('SELECT * FROM users')` in an async handler.
- **External API calls:** `await fetch('https://api.example.com/data')` in an async handler.
- **File I/O:** `await fs.promises.readFile('data.json')` in an async handler.
- **Authentication:** Sync middleware checks a JWT token and calls `next()` or sends 401.

---

## Core Concept 5: Route Parameters

### Definitions

**Core Definition:** Route parameters are named URL segments that capture values from specific positions in the URL path, making them accessible in the handler via `req.params`.

**Technical Definition:** Route parameters are defined with a colon prefix (`:paramName`) in the route path. Express captures the value of that URL segment and stores it in the `req.params` object with the parameter name as the key. For example, `/users/:userId` matches `/users/123` and makes `req.params.userId` equal `"123"` (always a string).

**Beginner-Friendly Explanation:** Route parameters are like fill-in-the-blank questions in a URL. `/users/:id` is a template where `:id` is the blank. When someone visits `/users/42`, Express fills in the blank with `42` and tells your code: "The user ID is 42."

### Purposes

- To identify dynamic URL variables and capture URL segment state.
- To create RESTful URLs that identify specific resources (`/users/123`, `/posts/456`).
- To avoid query strings for hierarchical resource identification.
- To enable reusable route definitions for multiple resources.

### Syntax Rules and Structure

```js
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  res.send(`User ${userId}, Book ${bookId}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `:userId` | Named parameter; captures the URL segment. |
| `req.params.userId` | The captured value (always a string). |
| `:bookId` | Second parameter. |

**Constraints:**
- Route parameter values are **always strings**; convert to numbers if needed (`parseInt(req.params.id)`).
- Route parameters must appear in the order defined in the path.
- In Express 5.x, wildcard parameters must be **named** (e.g., `/*splat`), and `{}` is required for optional segments.
- A parameter name must be a valid JavaScript identifier.

### Annotated Code Example

```js
// route-params.js
const express = require('express');
const app = express();

// Single parameter
app.get('/users/:userId', (req, res) => {
  res.send(`User profile for ID: ${req.params.userId}`);
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Parameter with regex constraint (numeric only)
app.get('/items/:id(\\d+)', (req, res) => {
  res.send(`Item ID: ${req.params.id}`);
});

// Optional parameter (Express 5.x syntax)
app.get('/blog{/:year}', (req, res) => {
  const year = req.params.year || 'all';
  res.send(`Blog year: ${year}`);
});

app.listen(3000, () => console.log('Params server on 3000'));
```

**Expected Output (for `GET /users/42`):**
```
User profile for ID: 42
```

**Expected Output (for `GET /users/42/posts/7`):**
```
{"userId":"42","postId":"7"}
```

**Expected Output (for `GET /items/abc`):**
```
404 Not Found
```

**Why this output:** The route `/items/:id(\\d+)` includes a regular expression constraint `\\d+` that only matches digits. `abc` does not match, so Express falls through to the 404 handler.

### Real-World Cases

- **User profiles:** `/users/:userId` to display a specific user's profile.
- **Blog posts:** `/posts/:postId/comments/:commentId` to identify a specific comment.
- **E-commerce:** `/products/:category/:productId` to identify a product within a category.
- **Versioned APIs:** `/api/v:version/users` to capture the API version.

---

## Core Concept 6: Query Parameters

### Definitions

**Core Definition:** Query parameters are non-hierarchical key-value pairs appended to the end of a URL after a `?` character, used to filter, sort, or customise the response without changing the URL path.

**Technical Definition:** Express parses the query string from the URL and exposes it as an object on `req.query`. Each key-value pair in the query string becomes a property on this object. For example, `/search?q=books&page=2` yields `req.query = { q: 'books', page: '2' }`. Query parameters are **not** considered when matching route paths.

**Beginner-Friendly Explanation:** Query parameters are like the options you tick on a search form. The main URL (`/search`) is where you're going, and the query string (`?q=books&page=2`) is the set of filters you're applying. Express automatically collects these filters and puts them in an easy-to-read object.

### Purposes

- To capture non-hierarchical key-value pairs appended to the end of request paths.
- To filter, sort, or paginate results without changing the URL structure.
- To pass optional configuration to a route handler.
- To support search functionality with user-provided terms.

### Syntax Rules and Structure

```js
app.get('/search', (req, res) => {
  const { q, page, limit } = req.query;
  res.json({ query: q, page, limit });
});
```

| Component | Breakdown |
|-----------|-----------|
| `req.query` | Object containing parsed query parameters. |
| `q` | Example parameter name. |
| `page` | Example parameter name. |
| Values | Always strings unless explicitly parsed. |

**Constraints:**
- Query parameter values are always strings (or arrays of strings for repeated keys).
- Query strings are **not** considered when matching route paths — `/search` matches both `/search` and `/search?q=books`.
- Express uses the `qs` library by default for parsing query strings.
- No validation is performed automatically; handle type conversion and validation in the handler.

### Annotated Code Example

```js
// query-params.js
const express = require('express');
const app = express();

// Search endpoint with query parameters
app.get('/search', (req, res) => {
  const q = req.query.q || '';               // Search term (default empty)
  const page = parseInt(req.query.page) || 1; // Page number (default 1)
  const limit = parseInt(req.query.limit) || 10; // Items per page

  res.json({
    query: q,
    page,
    limit,
    results: `Showing ${limit} results for "${q}" on page ${page}`
  });
});

// Filtering with multiple query parameters
app.get('/products', (req, res) => {
  const { category, minPrice, maxPrice, sort } = req.query;
  res.json({
    filters: { category, minPrice, maxPrice },
    sort: sort || 'default'
  });
});

// Repeated keys become an array
app.get('/tags', (req, res) => {
  const tags = req.query.tag;                // ?tag=js&tag=node
  res.json({ tags: Array.isArray(tags) ? tags : [tags].filter(Boolean) });
});

app.listen(3000, () => console.log('Query params on 3000'));
```

**Expected Output (for `GET /search?q=express&page=2&limit=5`):**
```
{
  "query": "express",
  "page": 2,
  "limit": 5,
  "results": "Showing 5 results for \"express\" on page 2"
}
```

**Expected Output (for `GET /tags?tag=js&tag=node`):**
```
{"tags":["js","node"]}
```

**Why this output:** Express parses `?tag=js&tag=node` and, because the key `tag` appears twice, `req.query.tag` becomes an array `['js', 'node']`. The handler checks if it's an array and returns it directly.

### Real-World Cases

- **Search engines:** `/search?q=node.js&page=3` to paginate search results.
- **E-commerce filtering:** `/products?category=electronics&minPrice=100&maxPrice=500` to filter products.
- **API pagination:** `/users?page=2&limit=20` to fetch the second page of users.
- **Sorting:** `/posts?sort=date_desc&author=alice` to sort and filter posts.
- **Analytics:** `/events?startDate=2026-01-01&endDate=2026-01-31` to filter date ranges.

---

## References

- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Express.js 5.x API Reference — https://expressjs.com/en/5x/api.html
- Express.js Application Object — https://expressjs.com/en/5x/api/application.html
- Express.js Router Class — https://expressjs.com/en/5x/api/router.html
- Express.js Request Object — https://expressjs.com/en/5x/api/request.html
- Express.js Response Object — https://expressjs.com/en/5x/api/response.html
- Express.js Basic Routing — https://expressjs.com/en/starter/basic-routing.html
- Express.js Middleware Guide — https://expressjs.com/en/guide/using-middleware.html
- Express.js Error Handling Guide — https://expressjs.com/en/guide/error-handling.html
- path-to-regexp Documentation — https://github.com/pillarjs/path-to-regexp
- MDN HTTP Methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN Query String — https://developer.mozilla.org/en-US/docs/Web/API/URL/search
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- Express.js CORS Middleware — https://expressjs.com/en/resources/middleware/cors.html