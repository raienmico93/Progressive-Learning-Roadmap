# Express.js Request Properties — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The Express.js request object (`req`) is an enhanced version of Node.js's native `http.IncomingMessage` that represents the incoming HTTP request and exposes properties for the URL, query string, route parameters, body, headers, cookies, and connection metadata .

**Technical Definition**
Express extends Node's `http.IncomingMessage.prototype` and attaches additional properties and methods during the request–response cycle. The `req` object is created per request, carries all client-supplied data, and is passed through the middleware chain and into route handlers. Express properties are either **getters** (computed on access, such as `req.fresh` and `req.secure`) or **populated values** (such as `req.body`, `req.cookies`, and `req.params`) that depend on middleware or the router's matching process. In Express 5, several properties changed behavior: `req.query` became a getter-only property, `req.param(name)` was removed, and unmatched optional parameters are omitted from `req.params` entirely rather than set to `undefined` .

**Beginner-Friendly Explanation**
Every time a request arrives, Express creates a `req` object that packages everything the client sent: the URL, the body, the headers, the cookies, and details like the client's IP address. Your middleware and route handlers read from this object to understand what the client wants. Some properties are filled in by middleware (like `req.body` and `req.cookies`), while others are computed automatically by Express (like `req.fresh` and `req.secure`).

---

### Key Characteristics

- **Inherited from Node.js** — `req` inherits `req.headers`, `req.url`, `req.method`, and other fields from `http.IncomingMessage`; Express adds `req.params`, `req.query`, `req.body`, `req.cookies`, `req.fresh`, `req.route`, and more .
- **Middleware-dependent** — `req.body` requires body-parsing middleware; `req.cookies` and `req.signedCookies` require `cookie-parser` .
- **Router-dependent** — `req.params` is populated only when a route with parameters matches; `req.route` is available only inside a matched route handler .
- **Proxy-aware** — `req.ip`, `req.ips`, `req.protocol`, `req.hostname`, and `req.secure` respect the `trust proxy` setting .
- **Express 5 compatibility** — `req.query` is getter-only and cannot be reassigned; `req.param()` is removed .

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- For `req.body`: `express.json()` or `express.urlencoded()` registered as middleware.
- For `req.cookies` / `req.signedCookies`: `cookie-parser` middleware registered with a secret.
- For `req.ip` / `req.ips` / `req.protocol` / `req.hostname`: correctly configured `trust proxy` setting when behind a reverse proxy.
- Basic understanding of HTTP, URL structure, and Express middleware.

---

### Related Programming Areas

- **Routing** — `req.params` and `req.route` are populated by the router.
- **Body parsing** — `req.body` depends on `express.json()` or `express.urlencoded()`.
- **Cookie handling** — `req.cookies` and `req.signedCookies` depend on `cookie-parser`.
- **Caching** — `req.fresh` and `req.stale` implement conditional request semantics.
- **Security and observability** — `req.ip`, `req.secure`, and `req.protocol` inform logging, rate limiting, and access control.

---

## Core Concept 1: Route and Query Parameters

### Definitions

**Core Definition**
`req.params` contains named route parameters extracted from the URL path, while `req.query` contains the parsed query string parameters from the URL.

**Technical Definition**
`req.params` is an object whose keys are the named segments declared in the route path (e.g., `:id` in `/users/:id`). In Express 5, wildcard parameters produce arrays, optional parameters that are not present are omitted entirely, and regex capture groups appear as numeric keys (`req.params[0]`) or named groups (`req.params.path`) . `req.query` is the result of the configured query parser; its shape depends on the `query parser` application setting . Both objects are based on user-controlled input and must be validated .

**Beginner-Friendly Explanation**
`req.params` gives you the dynamic parts of the URL path—like the `42` in `/users/42`. `req.query` gives you the `?key=value` pairs after the question mark. Both come directly from the client and cannot be trusted until validated.

---

### Purposes

- To extract resource identifiers from the URL path for database lookups.
- To read optional filtering, sorting, and pagination parameters from the query string.
- To provide a structured interface for route handlers to access URL data without parsing strings manually.

---

### Syntax Rules and Structure

```javascript
// Route: /users/:userId/books/:bookId
req.params.userId;   // "42"
req.params.bookId;   // "101"

// URL: /search?q=node&page=2
req.query.q;         // "node"
req.query.page;      // "2" (always a string)
```

**Component Breakdown**

| Property | Source | Type | Notes |
|----------|--------|------|-------|
| `req.params` | Route path | Object (or `Object.create(null)`) | Wildcards → arrays; regex groups → numeric keys  |
| `req.query` | Query string | Object | Shape depends on `query parser` setting  |

#### Syntax Rules

- `req.params` values are always strings; numeric conversion must be explicit.
- `req.query` values are strings or arrays of strings; type coercion is required for numbers and booleans.
- Both are untrusted user input and must be validated before use.
- In Express 5, `req.query` is getter-only and cannot be reassigned .

#### Constraints and Limitations

- `req.params` is reset if modified inside middleware; use `app.param()` for transformation .
- `req.query` cannot be mutated in Express 5; attach validated query data to a separate property if needed.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Route and Query Parameter Access

```javascript
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  const { format } = req.query;

  res.json({
    userId,
    bookId,
    format: format || 'json'
  });
});
```

**Expected Output**

```
GET /users/42/books/101?format=xml
→ 200 { "userId": "42", "bookId": "101", "format": "xml" }
```

**Why This Output Occurs**

- `req.params` captures the path segments; `req.query` captures the query string parameter.

---

### Real-World Cases

- **REST resource identifiers** — `/users/:id`, `/orders/:orderId/items/:itemId`.
- **Search and pagination** — `?q=node&page=2&limit=20`.
- **Filtering** — `?category=books&sort=price`.

---

## Core Concept 2: Request Body

### Definitions

**Core Definition**
`req.body` contains key–value pairs of data submitted in the request body, populated by body-parsing middleware.

**Technical Definition**
By default, `req.body` is `undefined`; it is populated when middleware such as `express.json()` or `express.urlencoded()` parses the request payload . The shape of `req.body` depends entirely on the parser and the client's `Content-Type`. Because the shape is based on user-controlled input, every property and value is untrusted and must be validated before use .

**Beginner-Friendly Explanation**
`req.body` is the data the client sends in a POST or PUT request—like a form submission or a JSON payload. Express does not parse it automatically; you must add a body parser. Once parsed, you can read `req.body.email`, `req.body.password`, and so on, but you must validate everything.

---

### Purposes

- To read JSON payloads from API clients.
- To read HTML form submissions encoded as `application/x-www-form-urlencoded`.
- To process file metadata or multipart form fields when combined with appropriate parsers.

---

### Syntax Rules and Structure

```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.post('/users', (req, res) => {
  const { email, password } = req.body;
  // validate before use
});
```

**Component Breakdown**

| Parser | Content-Type | Result |
|--------|-------------|--------|
| `express.json()` | `application/json` | Object or array |
| `express.urlencoded()` | `application/x-www-form-urlencoded` | Object of fields |

#### Syntax Rules

- Register body-parsing middleware **before** routes that read `req.body`.
- Validate every property of `req.body`; never trust its shape .
- Use `express.json()` and `express.urlencoded()` instead of the deprecated `body-parser` package.

#### Constraints and Limitations

- `req.body` is `undefined` if no body parser is registered or the content type does not match.
- Body size limits must be configured deliberately to prevent denial-of-service.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: JSON Body Parsing

```javascript
app.use(express.json());

app.post('/login', (req, res) => {
  const { email, password } = req.body;
  if (typeof email !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  res.json({ email });
});
```

**Expected Output**

```
POST /login  { "email": "alice@example.com", "password": "secret" }
→ 200 { "email": "alice@example.com" }

POST /login  { "email": 123 }
→ 400 { "error": "Invalid input" }
```

**Why This Output Occurs**

- `express.json()` parses the JSON body; the handler validates types before using the data.

---

### Real-World Cases

- **REST APIs** — JSON payloads for create and update operations.
- **Form submissions** — URL-encoded login and registration forms.
- **Webhooks** — JSON event payloads from third-party services.

---

## Core Concept 3: Headers and Cookies

### Definitions

**Core Definition**
`req.headers` is an object containing the request headers, and `req.cookies` / `req.signedCookies` contain parsed cookie values made available by `cookie-parser`.

**Technical Definition**
`req.headers` is inherited from Node's `http.IncomingMessage`; header names are lowercased, and the object is a plain key–value map . `req.cookies` is populated by `cookie-parser` and defaults to `{}` if no cookies are present . `req.signedCookies` contains signed cookies that have been verified and are ready for use; signed cookies are separated from unsigned cookies to prevent spoofing .

**Beginner-Friendly Explanation**
Headers are the metadata attached to every request—like `Authorization` or `Content-Type`. Cookies are small pieces of data the browser sends automatically. `cookie-parser` turns the raw `Cookie` header into readable objects, and `req.signedCookies` gives you verified cookies that cannot be tampered with.

---

### Purposes

- To read authentication tokens and API keys from the `Authorization` header.
- To inspect `Content-Type`, `Accept`, and other protocol headers.
- To read session identifiers and user preferences from cookies.
- To verify signed cookies for tamper-resistant state.

---

### Syntax Rules and Structure

```javascript
app.use(cookieParser(process.env.COOKIE_SECRET));

req.headers['content-type'];     // "application/json"
req.cookies.sessionId;           // unsigned cookie
req.signedCookies.preferences;   // verified signed cookie
```

**Component Breakdown**

| Property | Source | Notes |
|----------|--------|-------|
| `req.headers` | Node.js `http.IncomingMessage` | Keys are lowercase  |
| `req.cookies` | `cookie-parser` | Defaults to `{}`  |
| `req.signedCookies` | `cookie-parser` with secret | Verified, ready for use  |

#### Syntax Rules

- Header names are case-insensitive in HTTP but appear lowercase in `req.headers`; use `req.get()` for case-insensitive access.
- Register `cookie-parser` with a secret to populate `req.signedCookies`.
- Treat cookies as untrusted input; signed cookies are verified but not encrypted .

#### Constraints and Limitations

- `req.cookies` is `{}` if no cookies are present; always guard against missing values.
- Signing does not hide cookie contents; it only prevents tampering .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Headers and Signed Cookies

```javascript
app.use(cookieParser('my-secret'));

app.get('/profile', (req, res) => {
  const token = req.headers.authorization;
  const prefs = req.signedCookies.prefs;

  res.json({ hasToken: !!token, prefs: prefs || null });
});
```

**Expected Output**

```
GET /profile  (Authorization: Bearer abc, Cookie: prefs=s%3A...)
→ 200 { "hasToken": true, "prefs": { "theme": "dark" } }
```

**Why This Output Occurs**

- `req.headers.authorization` reads the header; `req.signedCookies.prefs` returns the verified cookie value.

---

### Real-World Cases

- **Authentication** — Reading Bearer tokens from headers.
- **User preferences** — Storing theme or language in signed cookies.
- **Session management** — Reading session IDs from cookies.

---

## Core Concept 4: URL and Routing Metadata

### Definitions

**Core Definition**
`req.method`, `req.url`, `req.originalUrl`, `req.path`, and `req.route` describe the HTTP method and the URL as seen by Express, including the effects of router mounting.

**Technical Definition**
`req.method` is the HTTP verb (e.g., `"GET"`, `"POST"`) . `req.url` is inherited from Node.js and is rewritten by `app.use()` to strip the mount point . `req.originalUrl` retains the original request URL and is a combination of `req.baseUrl` and `req.url` . `req.path` contains only the path portion of the URL and excludes the mount point when called from middleware . `req.route` contains the currently matched route object and is available only inside a route handler .

**Beginner-Friendly Explanation**
`req.url` is the URL the router currently sees—which may have had a mount prefix stripped. `req.originalUrl` is the full URL the client requested. `req.path` is just the path part. `req.route` tells you which route pattern matched.

---

### Purposes

- To log the full original URL while still allowing routers to rewrite `req.url`.
- To determine the current path for routing or conditional logic.
- To inspect the matched route for debugging or metrics.

---

### Syntax Rules and Structure

```javascript
app.use('/api', (req, res, next) => {
  console.log(req.originalUrl); // "/api/users"
  console.log(req.url);         // "/users" (mount point stripped)
  console.log(req.path);        // "/users"
  next();
});
```

**Component Breakdown**

| Property | Meaning | Mount-point behavior |
|----------|---------|----------------------|
| `req.method` | HTTP verb | Unchanged |
| `req.url` | Current URL | Mount point stripped  |
| `req.originalUrl` | Original URL | Unchanged  |
| `req.path` | Path only | Mount point excluded  |
| `req.route` | Matched route object | Available in route handlers  |

#### Syntax Rules

- Prefer `req.originalUrl` for logging; `req.url` is mutable by routers.
- Use `req.route` only inside a route handler; it is not available in general middleware.
- `req.path` does not include the query string.

#### Constraints and Limitations

- `req.url` can be rewritten by `app.use()` and other middleware; do not rely on it for the original URL.
- `req.route` is `undefined` outside a matched route handler.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: URL Logging Inside a Router

```javascript
app.use('/api', (req, res, next) => {
  console.log({
    originalUrl: req.originalUrl,
    url: req.url,
    path: req.path
  });
  next();
});

app.get('/api/users', (req, res) => {
  res.json({ route: req.route.path });
});
```

**Expected Output**

```
GET /api/users
→ Console: { originalUrl: '/api/users', url: '/users', path: '/users' }
→ 200 { "route": "/api/users" }
```

**Why This Output Occurs**

- `req.originalUrl` retains the full URL; `req.url` and `req.path` reflect the mount-point stripping; `req.route.path` shows the matched route pattern.

---

### Real-World Cases

- **Request logging** — Logging `req.originalUrl` for accurate access logs.
- **Conditional routing** — Using `req.path` for path-based logic.
- **Metrics** — Using `req.route.path` for route-level metrics without high-cardinality IDs.

---

## Core Concept 5: Host, IP, Protocol, and Network Metadata

### Definitions

**Core Definition**
`req.hostname`, `req.ip`, `req.ips`, `req.protocol`, `req.secure`, and `req.subdomains` expose network-level information about the request, with values influenced by the `trust proxy` setting.

**Technical Definition**
`req.hostname` derives the hostname from the `Host` header; when `trust proxy` is enabled, it uses `X-Forwarded-Host` . `req.ip` contains the remote IP address, using `X-Forwarded-For` when `trust proxy` is enabled . `req.ips` contains the array of addresses from `X-Forwarded-For` when `trust proxy` is enabled, otherwise it is an empty array . `req.protocol` is `"http"` or `"https"`, using `X-Forwarded-Proto` when `trust proxy` is enabled . `req.secure` is `true` when `req.protocol === "https"` . `req.subdomains` is an array of subdomains, determined by the `subdomain offset` setting (default `2`) .

**Beginner-Friendly Explanation**
These properties tell you where the request came from and how it arrived. `req.ip` is the client's address, `req.protocol` is `http` or `https`, and `req.secure` is `true` only for HTTPS. When your app is behind a proxy or load balancer, you must enable `trust proxy` so Express reads the forwarded headers instead of the proxy's address.

---

### Purposes

- To log the real client IP address behind a proxy.
- To enforce HTTPS by checking `req.secure`.
- To build tenant-aware logic using `req.subdomains`.
- To generate absolute URLs using the correct protocol and hostname.

---

### Syntax Rules and Structure

```javascript
app.set('trust proxy', 1); // behind a single reverse proxy

req.hostname;   // "api.example.com"
req.ip;         // "203.0.113.42"
req.ips;        // ["203.0.113.42", "10.0.0.1"]
req.protocol;   // "https"
req.secure;     // true
req.subdomains; // ["api"] for "api.example.com"
```

**Component Breakdown**

| Property | Source | Trust-proxy effect |
|----------|--------|-------------------|
| `req.hostname` | `Host` or `X-Forwarded-Host` | Uses forwarded header  |
| `req.ip` | Socket or `X-Forwarded-For` | Uses forwarded header  |
| `req.ips` | `X-Forwarded-For` | Populated only when trusted  |
| `req.protocol` | Socket or `X-Forwarded-Proto` | Uses forwarded header  |
| `req.secure` | Derived from `req.protocol` | Equivalent to `protocol === "https"`  |
| `req.subdomains` | `req.hostname` | Uses `subdomain offset`  |

#### Syntax Rules

- Configure `trust proxy` correctly when behind a reverse proxy; otherwise, `req.ip` will be the proxy's address and `req.secure` may be `false`.
- Use `req.secure` for HTTPS checks rather than comparing `req.protocol` manually.
- `req.subdomains` is ordered from the most specific subdomain to the least specific.

#### Constraints and Limitations

- Misconfigured `trust proxy` allows clients to spoof `X-Forwarded-For` and `X-Forwarded-Proto`.
- `req.ips` is empty when `trust proxy` is `false`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Trust Proxy and Client IP

```javascript
app.set('trust proxy', 1);

app.get('/info', (req, res) => {
  res.json({
    ip: req.ip,
    ips: req.ips,
    protocol: req.protocol,
    secure: req.secure,
    hostname: req.hostname,
    subdomains: req.subdomains
  });
});
```

**Expected Output**

```
GET /info
X-Forwarded-For: 203.0.113.42
X-Forwarded-Proto: https
Host: api.example.com
→ 200 {
  "ip": "203.0.113.42",
  "ips": ["203.0.113.42"],
  "protocol": "https",
  "secure": true,
  "hostname": "api.example.com",
  "subdomains": ["api"]
}
```

**Why This Output Occurs**

- With `trust proxy` enabled, Express reads the forwarded headers and derives the client-facing values.

---

### Real-World Cases

- **Rate limiting** — Using `req.ip` behind a proxy.
- **HTTPS enforcement** — Redirecting when `req.secure` is `false`.
- **Multi-tenant routing** — Using `req.subdomains[0]` as the tenant identifier.

---

## Core Concept 6: Conditional Request and Client Metadata

### Definitions

**Core Definition**
`req.fresh`, `req.stale`, and `req.xhr` provide conditional-request and client-type metadata used for caching and legacy AJAX detection.

**Technical Definition**
`req.fresh` returns `true` when the response is still fresh in the client's cache, based on `Last-Modified` and/or `ETag` matching; it returns `false` when the client sends `Cache-Control: no-cache` or when the cache is stale . `req.stale` is the logical opposite of `req.fresh` . `req.xhr` is `true` when the `X-Requested-With` header equals `"XMLHttpRequest"`, indicating a legacy AJAX request .

**Beginner-Friendly Explanation**
`req.fresh` tells you whether the client's cached copy is still valid, so you can send `304 Not Modified` instead of the full response. `req.stale` is simply the opposite. `req.xhr` is a legacy way to detect AJAX requests; modern clients rarely set the header.

---

### Purposes

- To implement conditional GET responses and reduce bandwidth.
- To return `304 Not Modified` when the client's cache is still valid.
- To detect legacy AJAX requests for differentiated responses.

---

### Syntax Rules and Structure

```javascript
app.get('/data', (req, res) => {
  if (req.fresh) {
    return res.status(304).end();
  }
  res.set('ETag', '"abc123"');
  res.json({ data: 'fresh content' });
});
```

**Component Breakdown**

| Property | Type | Meaning |
|----------|------|---------|
| `req.fresh` | Boolean | Cache is still valid  |
| `req.stale` | Boolean | Opposite of `req.fresh`  |
| `req.xhr` | Boolean | `X-Requested-With: XMLHttpRequest`  |

#### Syntax Rules

- Use `req.fresh` with ETag or Last-Modified headers; it only works when the response has cache validators.
- `req.fresh` returns `false` when the client sends `Cache-Control: no-cache` .
- `req.xhr` is not reliable for modern `fetch` requests unless the client explicitly sets the header.

#### Constraints and Limitations

- `req.fresh` depends on the response headers and the client's conditional headers; it is not a general-purpose caching mechanism.
- `req.xhr` is legacy and should not be used for new API design.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Conditional GET with `req.fresh`

```javascript
app.get('/article', (req, res) => {
  const article = { id: 1, title: 'Hello' };
  res.set('ETag', '"v1"');

  if (req.fresh) {
    return res.status(304).end();
  }

  res.json(article);
});
```

**Expected Output**

```
GET /article
→ 200 { "id": 1, "title": "Hello" }
ETag: "v1"

GET /article  (If-None-Match: "v1")
→ 304 Not Modified
```

**Why This Output Occurs**

- `req.fresh` becomes `true` when the client's `If-None-Match` matches the response's ETag, so the handler returns `304`.

---

### Real-World Cases

- **Static content APIs** — Returning `304` for unchanged resources.
- **Legacy AJAX detection** — Differentiating jQuery requests from full page loads.
- **Bandwidth optimization** — Reducing payload size for returning clients.

---

## Core Concept 7: Router Mounting and Application Metadata

### Definitions

**Core Definition**
`req.baseUrl`, `req.app`, and `req.res` expose the router's mount path, the Express application instance, and the response object associated with the request.

**Technical Definition**
`req.baseUrl` is the URL path on which a router instance was mounted; it returns the matched string even when the router was mounted with a path pattern . `req.app` holds a reference to the Express application instance, allowing middleware modules to access application-level settings and locals . `req.res` is a reference to the response object associated with the request .

**Beginner-Friendly Explanation**
`req.baseUrl` tells you the prefix under which the current router is mounted—like `/api` for a router mounted at `/api`. `req.app` lets middleware access the main Express app even if the middleware is defined in a separate file. `req.res` gives you the response object from inside the request object.

---

### Purposes

- To build full URLs by combining `req.baseUrl` with route paths.
- To access application-level settings from middleware defined in separate modules via `req.app`.
- To reference the response object from within request-centric code.

---

### Syntax Rules and Structure

```javascript
// router mounted at /api
app.use('/api', router);

router.get('/users', (req, res) => {
  console.log(req.baseUrl); // "/api"
  console.log(req.app.get('env')); // "development"
  console.log(req.res === res); // true
});
```

**Component Breakdown**

| Property | Meaning |
|----------|---------|
| `req.baseUrl` | Mount path of the current router  |
| `req.app` | Express application instance  |
| `req.res` | Response object reference  |

#### Syntax Rules

- `req.baseUrl` returns the matched mount string, not the original pattern .
- Use `req.app` to access `app.locals` and application settings from middleware modules.
- `req.res` is the same object as the `res` argument in the same handler.

#### Constraints and Limitations

- `req.baseUrl` is `""` when the router is mounted at the application root.
- `req.app` is the application instance, not the router; router-specific settings are not available through it.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Mount Path and Application Access

```javascript
const apiRouter = express.Router();

apiRouter.get('/users', (req, res) => {
  res.json({
    baseUrl: req.baseUrl,
    env: req.app.get('env'),
    hasRes: req.res === res
  });
});

app.use('/api', apiRouter);
```

**Expected Output**

```
GET /api/users
→ 200 { "baseUrl": "/api", "env": "development", "hasRes": true }
```

**Why This Output Occurs**

- The router is mounted at `/api`, so `req.baseUrl` is `"/api"`; `req.app` references the application; `req.res` is the same response object.

---

### Real-World Cases

- **Nested routers** — Building URLs relative to the mount path.
- **Shared middleware modules** — Accessing `app.locals` via `req.app`.
- **Debugging** — Verifying that `req.res` matches the handler's `res`.

---

## Core Concept 8: `req.next` — Not a Standard Express Property

### Definitions

**Core Definition**
`req.next` is **not** a standard Express request property. The `next` function is the third argument passed to middleware functions `(req, res, next)`, not a property attached to `req`.

**Technical Definition**
Express middleware receives `next` as a separate parameter. Some developers manually assign `req.next = next` to make it accessible deeper in the call chain, but this is **not part of the Express API** and is not recommended. The official Express request object documentation does not define `req.next`; the third argument to middleware is the only supported mechanism for continuing the middleware chain.

**Beginner-Friendly Explanation**
`next` is not something you read from `req`. It is a function passed to your middleware as the third argument: `(req, res, next)`. If you see `req.next` in code, it is a custom assignment, not a built-in Express feature.

---

### Purposes

- There is no official purpose for `req.next`; the supported pattern is to use the `next` parameter directly.

---

### Syntax Rules and Structure

```javascript
// CORRECT: use the next parameter
app.use((req, res, next) => {
  next();
});

// NOT STANDARD: manually attaching next to req
app.use((req, res, next) => {
  req.next = next; // avoid; not part of Express
  next();
});
```

#### Syntax Rules

- Always use the `next` parameter passed to middleware.
- Do not rely on `req.next` being present; it is not set by Express.

#### Constraints and Limitations

- `req.next` is `undefined` by default.
- Assigning `req.next` can create confusion and is not supported by Express documentation.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Correct Use of `next`

```javascript
app.use((req, res, next) => {
  console.log('Middleware 1');
  next(); // proceed to the next middleware
});

app.use((req, res) => {
  res.send('Done');
});
```

**Expected Output**

```
GET /
→ Console: Middleware 1
→ 200 "Done"
```

**Why This Output Occurs**

- The `next` parameter is called directly; no `req.next` property is involved.

---

### Real-World Cases

- **All middleware** — `next()` is the standard way to continue the chain.
- **Error handling** — `next(err)` forwards errors to error-handling middleware.

---

## Request Properties Comparison

| Property | Source | Middleware Required | Express 5 Notes |
|----------|--------|---------------------|-----------------|
| `req.params` | Router | Route with parameters | Optional params omitted; wildcards → arrays  |
| `req.query` | Query string | None | Getter-only; cannot reassign  |
| `req.body` | Body parser | `express.json()` / `express.urlencoded()` | `undefined` if no parser  |
| `req.headers` | Node.js | None | Lowercase keys  |
| `req.cookies` | `cookie-parser` | `cookie-parser` | Defaults to `{}`  |
| `req.signedCookies` | `cookie-parser` | `cookie-parser` with secret | Verified, not encrypted  |
| `req.method` | Node.js | None | `"GET"`, `"POST"`, etc.  |
| `req.url` | Node.js | None | Rewritten by `app.use()`  |
| `req.originalUrl` | Express | None | Full original URL  |
| `req.path` | Express | None | Path only; mount point excluded  |
| `req.hostname` | Express | `trust proxy` for forwarded host | Uses `X-Forwarded-Host` when trusted  |
| `req.ip` | Express | `trust proxy` for forwarded IP | Uses `X-Forwarded-For` when trusted  |
| `req.ips` | Express | `trust proxy` | Array from `X-Forwarded-For`  |
| `req.protocol` | Express | `trust proxy` for forwarded proto | `"http"` or `"https"`  |
| `req.secure` | Express | Derived from `req.protocol` | `protocol === "https"`  |
| `req.subdomains` | Express | `subdomain offset` setting | Array of subdomains  |
| `req.xhr` | Express | None | `X-Requested-With: XMLHttpRequest`  |
| `req.fresh` | Express | Cache validators | `true` when cache is fresh  |
| `req.stale` | Express | Cache validators | Opposite of `req.fresh`  |
| `req.route` | Router | Inside route handler | Matched route object  |
| `req.baseUrl` | Router | Inside router | Mount path  |
| `req.app` | Express | None | Application instance  |
| `req.res` | Express | None | Response object reference  |
| `req.next` | **Not standard** | — | Use the `next` parameter instead |

---

## Request Properties Best Practices

### Definitions

**Core Definition**
Request property best practices are the habits that keep access to `req` safe, predictable, and compatible across Express versions.

**Technical Definition**
Best practices include: validating all user-controlled properties (`req.body`, `req.params`, `req.query`, `req.cookies`) before use; configuring body parsers and `cookie-parser` explicitly; using `req.originalUrl` for logging and `req.path` for routing logic; configuring `trust proxy` correctly when behind a reverse proxy; using `req.fresh` with cache validators; and treating `req.next` as non-existent by using the `next` parameter directly.

### Purposes

- To prevent injection and malformed-input attacks through validation.
- To ensure properties are populated by the correct middleware.
- To avoid Express 5 compatibility issues.
- To maintain accurate logging and network metadata behind proxies.

### Syntax Rules and Structure

- **Validate everything** — `req.body`, `req.params`, `req.query`, and cookies are untrusted .
- **Register middleware in order** — Body parsers before routes; `cookie-parser` before routes that read cookies.
- **Use `req.originalUrl` for logs** — `req.url` is mutable by routers .
- **Configure `trust proxy`** — Otherwise `req.ip`, `req.protocol`, and `req.secure` may reflect the proxy, not the client .
- **Use the `next` parameter** — `req.next` is not part of Express.

### Constraints and Limitations

- Express 5 makes `req.query` getter-only; attach validated query data to a separate property .
- `req.body` is `undefined` without a body parser; always guard access .

---

## References

- Request Object · Express.js 5.x — https://expressjs.com/en/5x/api/request/
- Express 5.x API Reference — https://expressjs.com/en/5x/api.html
- Migrating to Express 5 (request property changes) — https://expressjs.com/en/guide/migrating-5.html
- `req.headers` inherited from Node.js `http.IncomingMessage` — https://nodejs.org/api/http.html#class-httpincomingmessage
- `cookie-parser` middleware — https://www.npmjs.com/package/cookie-parser