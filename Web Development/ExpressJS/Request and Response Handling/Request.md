# Express.js Request Object — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The `req` object represents the HTTP request and has properties for the request query string, parameters, body, HTTP headers, and so on. It is an enhanced version of Node's own request object and supports all built-in fields and methods.

**Technical Definition:** The `req` object is an instance of Node's `http.IncomingMessage` (or `http2.Http2ServerRequest`), augmented by Express with additional properties and methods for convenience. In this documentation and by convention, the object is always referred to as `req` (and the HTTP response is `res`), but its actual name is determined by the parameters to the callback function in which you're working. The `req` object contains a number of properties that provide information about the HTTP request, such as headers, query parameters, and more.

**Beginner-Friendly Explanation:** Think of `req` as a complete "information packet" that arrives at your server every time someone makes a request. It contains everything you need to know about that request: who sent it, what they want, what data they included, and more. Your job as the developer is to read the relevant pieces from this packet and respond accordingly.

### Key Characteristics

- **Built-in to every handler:** Every route handler and middleware function receives `req` as its first argument.
- **Enhanced Node.js object:** Extends Node's native `http.IncomingMessage` with Express-specific properties.
- **Read-only for most properties:** In Express 5, many properties like `req.query` are getters and cannot be directly reassigned.
- **Untrusted input:** All properties derived from user input (params, query, body, headers, cookies) are untrusted and must be validated.
- **Consistent interface:** Provides a uniform way to access request data regardless of the underlying HTTP implementation.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v0.10+ for Express 4.x).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Objects, arrays, and type coercion.
- **Understanding of HTTP:** Requests, responses, headers, and methods.

### Related Programming Areas

- **Middleware:** The `req` object flows through every middleware function in the stack.
- **Routing:** Route parameters and query parameters are accessed via `req`.
- **Authentication:** Tokens and credentials are read from `req.headers` or `req.cookies`.
- **Body parsing:** `req.body` is populated by body-parsing middleware.
- **Security:** `req.ip` and `req.protocol` are used for rate limiting and HTTPS enforcement.

### Core Concepts

1. **`req.params`** — reading parsed URL parameters and managing type safety.
2. **`req.query`** — accessing URL query strings and nested object parsing.
3. **`req.body`** — accessing the parsed payload (dependent on body-parsing middleware).
4. **`req.headers`** — reading HTTP request headers (case-insensitive key lookups).
5. **`req.cookies`** — reading signed vs. unsigned client cookies.
6. **`req.ip`** — extracting client IP addresses and configuring trust proxy.
7. **Request Metadata** — tracking `req.method`, `req.url`, `req.path`, `req.secure`, and protocol schemas.

---

## Core Concept 1: `req.params`

### Definitions

**Core Definition:** `req.params` is an object containing properties mapped to the named route "parameters" defined in the route path.

**Technical Definition:** This property is an object containing properties mapped to the named route "parameters". For example, if you have the route `/user/:name`, then the "name" property is available as `req.params.name`. This object defaults to `{}`. When you use a regular expression for the route definition, capture groups are provided in the array using `req.params[n]`, where `n` is the nth capture group. This rule is applied to unnamed wild card matches with string routes such as `/file/*`. Express automatically decodes the values in `req.params` using `decodeURIComponent`.

**Beginner-Friendly Explanation:** `req.params` is a collection of values extracted directly from the URL path itself. If your route is `/users/:id` and someone visits `/users/42`, then `req.params.id` will be `"42"`. These are different from query parameters — params are part of the URL structure, while queries come after the `?`.

### Purposes

- To read parsed URL parameters and manage type safety (e.g., string to integer casting).
- To identify specific resources in RESTful URLs without using query strings.
- To capture dynamic URL segment state for database lookups.
- To enable reusable route definitions that work for any resource identifier.

### Syntax Rules and Structure

```js
// Route definition
app.get('/user/:name', (req, res) => {
  const name = req.params.name;   // Access the captured value
  res.send(`User: ${name}`);
});
```

| Component | Breakdown |
|-----------|-----------|
| `:name` | Parameter name in the route path. |
| `req.params` | Object containing all captured parameters. |
| `req.params.name` | The value captured for the `:name` parameter (always a string). |

**Rules:**
- Parameter values are **always strings**; type conversion is the developer's responsibility.
- Express automatically decodes URI-encoded values using `decodeURIComponent`.
- If you need to make changes to a key in `req.params`, use the `app.param()` handler. Changes made directly to `req.params` in a middleware or route handler will be reset.
- Regular expression capture groups are stored by numeric index (`req.params[0]`, `req.params[1]`, etc.).

**Constraints and Limitations:**
- Parameter names must be valid JavaScript identifiers (letters, digits, underscores; cannot start with a digit).
- Unmatched optional parameters are omitted entirely from `req.params` in Express 5.

### Annotated Code Examples

#### Example 1: Basic Parameter Access with Type Casting

```js
// req-params-basic.js
const express = require('express');
const app = express();

app.get('/users/:id', (req, res) => {
  // req.params.id is always a string
  console.log('Raw:', req.params.id);            // "42"
  console.log('Type:', typeof req.params.id);    // "string"

  // Cast to integer for database lookup
  const userId = parseInt(req.params.id, 10);

  // Validate the cast
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }

  res.json({ userId, type: typeof userId });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /users/42`):**
```
Raw: 42
Type: string
{"userId":42,"type":"number"}
```

**Expected Output (for `GET /users/abc`):**
```
Raw: abc
Type: string
{"error":"Invalid user ID"}
```

**Why this output:** The route captures `42` as a string in `req.params.id`. The handler explicitly converts it to an integer using `parseInt` and validates the result. Invalid values like `abc` produce `NaN`, which is caught and rejected. This demonstrates the "type safety" pattern: never trust that a parameter is a number just because it looks like one.

#### Example 2: Regular Expression Capture Groups

```js
// req-params-regex.js
const express = require('express');
const app = express();

app.get(/^\/file\/(.*)$/, (req, res) => {
  // Unnamed capture groups are stored by numeric index
  console.log('Params:', req.params);           // { '0': 'path/to/file.txt' }
  console.log('First capture:', req.params[0]);  // "path/to/file.txt"
  res.send(`File path: ${req.params[0]}`);
});

app.listen(3000, () => console.log('Regex params on 3000'));
```

**Expected Output (for `GET /file/path/to/file.txt`):**
```
Params: { '0': 'path/to/file.txt' }
First capture: path/to/file.txt
File path: path/to/file.txt
```

**Why this output:** When a regular expression is used as the route path, Express stores captured groups by numeric index rather than by name. The first (and only) capture group contains the full path after `/file/`.

### Real-World Cases

- **REST APIs:** `/users/:id`, `/posts/:postId` for resource identification.
- **File browsers:** `/files/:folder/:filename` to navigate a virtual file system.
- **E-commerce:** `/products/:category/:productId` to identify a product within a category.
- **API versioning:** `/api/:version/users` to support multiple API versions.

---

## Core Concept 2: `req.query`

### Definitions

**Core Definition:** `req.query` is an object containing a property for each query string parameter in the request URL, automatically parsed by Express.

**Technical Definition:** This property is an object containing the parsed query-string, defaulting to `{}`. The parsing behaviour depends on the `query parser` setting. By default, Express uses the "simple" parser (Node's built-in `querystring` module), which produces flat key-value pairs. To parse nested query parameters like `user[name]=john` into `{user: {name: 'john'}}`, you must configure Express to use the extended query parser by setting `app.set('query parser', 'extended')`. Without this setting, bracket notation is kept as literal keys: `{'user[name]': 'john'}`.

**Beginner-Friendly Explanation:** `req.query` holds the options that come after the `?` in a URL. If someone visits `/search?q=books&page=2`, then `req.query.q` is `"books"` and `req.query.page` is `"2"`. For nested objects like `?filter[price]=50`, you need to enable the extended parser.

### Purposes

- To access URL query strings and understand nested object parsing (e.g., `?filters[price]=50`).
- To implement filtering, sorting, pagination, and search.
- To pass optional configuration to route handlers without changing the URL path.
- To enable user-driven data exploration.

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
| `q` | Example parameter name; value is always a string (or array). |
| `page`, `limit` | Additional parameters; values are strings until cast. |

**Rules:**
- Query parameter values are **always strings** (or arrays of strings for repeated keys).
- If a parameter appears multiple times (`?tag=js&tag=node`), the value becomes an array.
- Query strings are **not** considered when matching route paths.
- The `query parser` setting can be set to `'simple'` (default in Express 5), `'extended'` (qs library), or a custom function.

**Constraints and Limitations:**
- In Express 5, `req.query` is a getter and is no longer writable.
- Nested objects require the `'extended'` parser.
- The `qs` library limits array indices to a maximum of 20; higher indices become object keys.

### Annotated Code Examples

#### Example 1: Basic Query Parameters

```js
// req-query-basic.js
const express = require('express');
const app = express();

app.get('/search', (req, res) => {
  console.log('Full query:', req.query);        // { q: 'express', page: '2' }
  console.log('Search term:', req.query.q);     // "express"
  console.log('Page:', req.query.page);         // "2"

  res.json({
    q: req.query.q || '',
    page: parseInt(req.query.page, 10) || 1
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /search?q=express&page=2`):**
```
Full query: { q: 'express', page: '2' }
Search term: express
Page: 2
{"q":"express","page":2}
```

**Why this output:** Express parses the query string `?q=express&page=2` and creates the object `{ q: 'express', page: '2' }`. Both values are strings. The handler uses `parseInt` to convert `page` to a number for the JSON response.

#### Example 2: Nested Object Parsing with Extended Parser

```js
// req-query-nested.js
const express = require('express');
const app = express();

// Enable extended query parser for nested objects
app.set('query parser', 'extended');

app.get('/filter', (req, res) => {
  console.log('Nested query:', req.query);
  // ?price[gte]=10&price[lte]=50 becomes:
  // { price: { gte: '10', lte: '50' } }
  res.json(req.query);
});

app.listen(3000, () => console.log('Nested query on 3000'));
```

**Expected Output (for `GET /filter?price[gte]=10&price[lte]=50`):**
```
Nested query: { price: { gte: '10', lte: '50' } }
{"price":{"gte":"10","lte":"50"}}
```

**Expected Output (without extended parser):**
```
Nested query: { 'price[gte]': '10', 'price[lte]': '50' }
{"price[gte]":"10","price[lte]":"50"}
```

**Why this output:** With the `'extended'` parser, `price[gte]=10` is parsed as a nested object `{ price: { gte: '10' } }`. Without it, the key is the literal string `'price[gte]'`. The values are still strings; further casting (e.g., `parseInt`) is required for numeric operations.

### Real-World Cases

- **Search engines:** `/search?q=node.js&page=2&limit=10`.
- **E-commerce filtering:** `/products?category=electronics&minPrice=100&maxPrice=500`.
- **API pagination:** `/users?page=2&limit=20&sort=createdAt`.
- **Analytics:** `/events?startDate=2026-01-01&endDate=2026-01-31`.

---

## Core Concept 3: `req.body`

### Definitions

**Core Definition:** `req.body` contains key-value pairs of data submitted in the request body, populated by body-parsing middleware.

**Technical Definition:** By default, `req.body` is `undefined`, and is populated when you use body-parsing middleware such as `express.json()` or `express.urlencoded()`. As `req.body`'s shape is based on user-controlled input, all properties and values in this object are untrusted and should be validated before trusting. The `express.json()` middleware parses incoming requests with JSON payloads, while `express.urlencoded()` parses URL-encoded form data.

**Beginner-Friendly Explanation:** `req.body` holds the data that a client sends in the body of a POST, PUT, or PATCH request — like form submissions or JSON payloads. However, Express doesn't parse this data automatically. You must add middleware (like `express.json()`) to tell Express how to read the body and make it available on `req.body`.

### Purposes

- To access the parsed payload (dependent on matching body-parsing middleware).
- To receive and process JSON data from API clients.
- To handle HTML form submissions (URL-encoded).
- To enable file uploads (via multipart-handling middleware like Multer).

### Syntax Rules and Structure

```js
// Middleware setup
app.use(express.json());                    // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse form bodies

app.post('/api/data', (req, res) => {
  const data = req.body;                   // Access the parsed body
  res.json({ received: data });
});
```

| Middleware | Content-Type | Populates |
|-----------|-------------|-----------|
| `express.json()` | `application/json` | `req.body` as an object. |
| `express.urlencoded()` | `application/x-www-form-urlencoded` | `req.body` as an object. |
| `multer` | `multipart/form-data` | `req.file` / `req.files`. |

**Rules:**
- `req.body` is `undefined` without body-parsing middleware.
- The middleware must match the request's `Content-Type` header.
- `express.json()` and `express.urlencoded()` are built into Express 4.16.0+.
- Always validate `req.body` before use; it is user-controlled input.

**Constraints and Limitations:**
- Multipart bodies (file uploads) are not handled by `express.json()` or `express.urlencoded()`; use Multer or similar.
- Default body size limit is `100kb`; adjust with the `limit` option if needed.
- Malformed JSON produces a 400 error by default.

### Annotated Code Example

```js
// req-body.js
const express = require('express');
const app = express();

// Body-parsing middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// POST handler that reads req.body
app.post('/api/users', (req, res) => {
  console.log('Body:', req.body);

  // Validate required fields
  if (!req.body.name || typeof req.body.name !== 'string') {
    return res.status(400).json({ error: 'Name is required and must be a string' });
  }

  res.status(201).json({
    message: 'User created',
    user: { name: req.body.name, email: req.body.email }
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/users` with JSON `{ "name": "Alice", "email": "alice@example.com" }`):**
```
Body: { name: 'Alice', email: 'alice@example.com' }
{"message":"User created","user":{"name":"Alice","email":"alice@example.com"}}
```

**Expected Output (for `POST /api/users` with `{ "email": "alice@example.com" }`):**
```
Body: { email: 'alice@example.com' }
{"error":"Name is required and must be a string"}
```

**Why this output:** The `express.json()` middleware parses the JSON body and populates `req.body`. The handler validates that `name` exists and is a string before proceeding. Invalid requests receive a 400 error.

### Real-World Cases

- **REST APIs:** `POST /api/users` with JSON payload.
- **Form submissions:** `POST /contact` with URL-encoded form data.
- **File uploads:** `POST /upload` with multipart data (handled by Multer).
- **Webhooks:** `POST /webhooks/stripe` with JSON event payloads.

---

## Core Concept 4: `req.headers`

### Definitions

**Core Definition:** `req.headers` is an object containing all HTTP request headers, with header names normalized to lowercase for case-insensitive access.

**Technical Definition:** HTTP headers are simple key-value pairs sent at the beginning of HTTP requests and responses. In Express, header names are case-insensitive. Express normalizes all header names to lowercase: `req.headers['content-type']`, `req.headers['Content-Type']`, and `req.headers['CONTENT-TYPE']` all access the same header. The `req.get()` method provides a case-insensitive way to access headers and is cleaner than directly using `req.headers`.

**Beginner-Friendly Explanation:** `req.headers` is a dictionary of all the metadata the client sent along with the request — things like what type of data they're sending, what authentication they're using, and what browser they're on. Header names are case-insensitive, so you can look them up in any case.

### Purposes

- To read HTTP request headers (case-insensitive key lookups).
- To extract authentication tokens (e.g., `Authorization`).
- To determine the client's content type and accept headers.
- To read custom headers (e.g., `X-API-Key`, `X-Request-ID`).

### Syntax Rules and Structure

```js
app.get('/headers', (req, res) => {
  const userAgent = req.headers['user-agent'];  // Lowercase access
  const contentType = req.get('Content-Type');   // Case-insensitive method
  res.json({ userAgent, contentType });
});
```

| Access Method | Case Sensitivity | Notes |
|--------------|-----------------|-------|
| `req.headers['content-type']` | Lowercase only | Direct object access. |
| `req.headers['Content-Type']` | Works (Express normalizes) | Express lowercases all keys. |
| `req.get('Content-Type')` | Case-insensitive | Recommended convenience method. |
| `req.header('Content-Type')` | Case-insensitive | Alias for `req.get()`. |

**Rules:**
- Express normalizes all header names to lowercase.
- `req.get()` is case-insensitive and returns `undefined` for missing headers.
- The `Authorization` header contains credentials (e.g., `Bearer <token>`).
- Custom headers should use underscores instead of dashes for easier access.

**Constraints and Limitations:**
- Header values are always strings; no automatic type conversion.
- Sensitive headers (e.g., `Authorization`) should never be logged in production.
- Header size is limited by the HTTP server (default 16KB in Node.js).

### Annotated Code Example

```js
// req-headers.js
const express = require('express');
const app = express();

app.get('/headers', (req, res) => {
  // Case-insensitive access via req.get()
  const userAgent = req.get('User-Agent');
  const contentType = req.get('Content-Type');
  const authHeader = req.get('Authorization');

  // Direct access (lowercase keys)
  const accept = req.headers['accept'];

  console.log('User-Agent:', userAgent);
  console.log('Content-Type:', contentType);
  console.log('Authorization:', authHeader ? '***' : 'none');

  res.json({
    userAgent,
    contentType: contentType || 'none',
    authenticated: !!authHeader,
    accept
  });
});

app.listen(3000, () => console.log('Headers server on 3000'));
```

**Expected Output (for `GET /headers` with `User-Agent: Mozilla/5.0`, `Accept: application/json`, `Authorization: Bearer abc123`):**
```
User-Agent: Mozilla/5.0
Content-Type: undefined
Authorization: ***
{"userAgent":"Mozilla/5.0","contentType":"none","authenticated":true,"accept":"application/json"}
```

**Why this output:** `req.get()` retrieves headers case-insensitively. The `Authorization` header is present, so `authenticated` is `true`. The `Content-Type` header is not sent in a GET request, so it defaults to `'none'`. The `Accept` header is accessed directly via `req.headers['accept']` (lowercase).

### Real-World Cases

- **Authentication:** Reading `Authorization: Bearer <JWT>`.
- **Content negotiation:** Reading `Accept` to determine response format.
- **API versioning:** Reading `X-API-Version` to route requests.
- **Rate limiting:** Reading `X-Forwarded-For` to identify client IPs.

---

## Core Concept 5: `req.cookies`

### Definitions

**Core Definition:** `req.cookies` is an object containing cookies sent by the client, populated by the `cookie-parser` middleware. Signed cookies are accessible via `req.signedCookies`.

**Technical Definition:** When using `cookie-parser` middleware, `req.cookies` is an object that contains cookies sent by the request. If the request contains no cookies, it defaults to `{}`. If the cookie has been signed, you have to use `req.signedCookies`. The `cookie-parser` middleware parses the `Cookie` header and populates `req.cookies` with an object keyed by the cookie names. Optionally, you may enable signed cookie support by passing a `secret` string, which assigns `req.secret` so it may be used by other middleware.

**Beginner-Friendly Explanation:** Cookies are small pieces of data that the client sends back to the server with every request. `req.cookies` gives you a simple object of all the unsigned cookies. Signed cookies — cookies that have a cryptographic signature to prevent tampering — are placed in `req.signedCookies` instead. To use either, you must add the `cookie-parser` middleware.

### Purposes

- To read signed vs. unsigned client cookies (dependent on `cookie-parser`).
- To maintain session state across requests.
- To read authentication tokens stored in cookies.
- To implement "remember me" functionality.

### Syntax Rules and Structure

```js
const cookieParser = require('cookie-parser');
app.use(cookieParser('my-secret'));  // Secret enables signed cookies

app.get('/cookies', (req, res) => {
  console.log('Unsigned:', req.cookies);        // { session: 'abc' }
  console.log('Signed:', req.signedCookies);    // { userId: '42' }
  res.json({ cookies: req.cookies, signed: req.signedCookies });
});
```

| Property | Contents | Requirements |
|----------|----------|-------------|
| `req.cookies` | Unsigned cookies. | `cookie-parser` middleware. |
| `req.signedCookies` | Signed cookies (validated). | `cookie-parser` with `secret`. |

**Rules:**
- `cookie-parser` must be installed and mounted before routes that access cookies.
- Signed cookies are prefixed with `s:` in the raw `Cookie` header.
- Signed cookies that fail validation have the value `false` instead of the tampered value.
- JSON cookies are prefixed with `j:` and are automatically parsed via `JSON.parse`.

**Constraints and Limitations:**
- Without `cookie-parser`, `req.cookies` is `undefined`.
- Signed cookies require a secret; if the secret is compromised, all signed cookies can be forged.
- If your code reads the unsigned `req.cookies` copy of a cookie name as a fallback, you may have a signature bypass vulnerability.

### Annotated Code Example

```js
// req-cookies.js
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

// Mount cookie-parser with a secret for signed cookies
app.use(cookieParser('my-secret-key'));

// Route to set cookies
app.get('/set-cookies', (req, res) => {
  res.cookie('session', 'abc123', { httpOnly: true });
  res.cookie('userId', '42', { signed: true, httpOnly: true });
  res.send('Cookies set');
});

// Route to read cookies
app.get('/read-cookies', (req, res) => {
  console.log('Unsigned cookies:', req.cookies);
  console.log('Signed cookies:', req.signedCookies);

  res.json({
    session: req.cookies.session,          // "abc123"
    userId: req.signedCookies.userId       // "42"
  });
});

app.listen(3000, () => console.log('Cookie server on 3000'));
```

**Expected Output (after setting then reading cookies):**
```
Unsigned cookies: { session: 'abc123' }
Signed cookies: { userId: '42' }
{"session":"abc123","userId":"42"}
```

**Expected Output (if a signed cookie is tampered with):**
```
Unsigned cookies: { session: 'abc123' }
Signed cookies: { userId: false }
{"session":"abc123","userId":false}
```

**Why this output:** The unsigned `session` cookie is stored in `req.cookies`. The signed `userId` cookie is validated by `cookie-parser` and stored in `req.signedCookies`. If the signature is invalid (tampered), the value becomes `false`.

### Real-World Cases

- **Session management:** `sessionId` cookie to identify authenticated sessions.
- **Remember me:** A signed `userId` cookie that persists across browser sessions.
- **A/B testing:** A cookie storing the test variant.
- **Shopping carts:** A cookie storing cart items before login.

---

## Core Concept 6: `req.ip`

### Definitions

**Core Definition:** `req.ip` contains the remote IP address of the request. When the `trust proxy` setting is enabled, it returns the upstream address instead of the direct socket address.

**Technical Definition:** `req.ip` contains the remote IP address of the request. When the `trust proxy` setting does not evaluate to `false`, the value of this property is derived from the left-most entry in the `X-Forwarded-For` header. The `req.ips` property, when `trust proxy` is `true`, parses the `X-Forwarded-For` IP address list and returns an array; otherwise, an empty array is returned. For example, if the value were "client, proxy1, proxy2", you would receive the array `["client", "proxy1", "proxy2"]`.

**Beginner-Friendly Explanation:** `req.ip` tells you the IP address of the client making the request. When your app runs behind a reverse proxy (like Nginx or Cloudflare), the direct connection IP is the proxy's IP, not the client's. To get the real client IP, you must enable the `trust proxy` setting so Express trusts the `X-Forwarded-For` header that the proxy adds.

### Purposes

- To extract client IP addresses for logging, rate limiting, and geolocation.
- To configure trust proxy settings for reverse proxies (Nginx, Cloudflare).
- To implement IP-based access control (allowlists/blocklists).
- To detect and prevent abuse.

### Syntax Rules and Structure

```js
// Without trust proxy (direct connection)
app.get('/ip', (req, res) => {
  res.json({ ip: req.ip });  // Client's direct socket address
});

// With trust proxy (behind reverse proxy)
app.set('trust proxy', true);
app.get('/ip', (req, res) => {
  res.json({ ip: req.ip, ips: req.ips });  // Real client IP from X-Forwarded-For
});
```

| `trust proxy` value | Meaning |
|---------------------|---------|
| `false` (default) | Direct connection; `req.ip` is the socket address. |
| `true` | Trust the left-most entry in `X-Forwarded-For`. |
| IP address / subnet | Trust specific proxies. |
| Number (`1`, `2`, ...) | Trust `n` hops from the app. |

**Rules:**
- `trust proxy` must be set with `app.set('trust proxy', value)`.
- **Security warning:** Setting `trust proxy` to `true` without ensuring the proxy strips incoming `X-Forwarded-For` headers allows clients to spoof their IP.
- Use specific subnets (e.g., `'loopback'`, `'linklocal'`) rather than `true` for better security.

**Constraints and Limitations:**
- `req.ip` is read-only and cannot be set directly.
- Misconfigured `trust proxy` can lead to rate limiting bypass and IP spoofing.
- The `X-Forwarded-For` header can be spoofed if not properly sanitized by the proxy.

### Annotated Code Example

```js
// req-ip.js
const express = require('express');
const app = express();

// Configure trust proxy for a single reverse proxy
app.set('trust proxy', 1);  // Trust the first hop (the reverse proxy)

app.get('/ip', (req, res) => {
  res.json({
    ip: req.ip,           // Real client IP (from X-Forwarded-For)
    ips: req.ips,         // Array of all IPs in the chain
    protocol: req.protocol
  });
});

// Rate limiting example using req.ip
const requestCounts = new Map();

app.get('/limited', (req, res) => {
  const ip = req.ip;
  const count = (requestCounts.get(ip) || 0) + 1;
  requestCounts.set(ip, count);

  if (count > 5) {
    return res.status(429).json({ error: 'Too many requests' });
  }

  res.json({ ip, count, remaining: 5 - count });
});

app.listen(3000, () => console.log('IP server on 3000'));
```

**Expected Output (for `GET /ip` behind a proxy that sets `X-Forwarded-For: 203.0.113.5`):**
```
{"ip":"203.0.113.5","ips":["203.0.113.5"],"protocol":"https"}
```

**Expected Output (for `GET /limited` six times from the same IP):**
```
{"ip":"203.0.113.5","count":1,"remaining":4}
...
{"ip":"203.0.113.5","count":6,"remaining":-1}
{"error":"Too many requests"}
```

**Why this output:** With `trust proxy` set to `1`, Express trusts the first hop (the reverse proxy) and derives `req.ip` from the `X-Forwarded-For` header. The rate limiter uses `req.ip` as the key and rejects requests after 5 attempts.

### Real-World Cases

- **Rate limiting:** Limiting API requests per client IP.
- **Analytics:** Logging client IPs for geographic analysis.
- **Security:** Blocking malicious IPs from accessing the application.
- **Compliance:** Auditing access logs for regulatory requirements.

---

## Core Concept 7: Request Metadata

### Definitions

**Core Definition:** Request metadata includes properties such as `req.method`, `req.url`, `req.path`, `req.secure`, and `req.protocol`, which describe the HTTP request's method, URL structure, and connection security.

**Technical Definition:** `req.method` contains a string corresponding to the HTTP method of the request: `GET`, `POST`, `PUT`, `DELETE`, etc. `req.url` is not a native Express property; it is inherited from Node's `http` module. `req.path` contains the path part of the request URL. `req.protocol` contains the request protocol string: `http` or `https` when requested with TLS. `req.secure` is a Boolean that is `true` if a TLS connection is established, and is a shorthand for `'https' == req.protocol`.

**Beginner-Friendly Explanation:** These properties tell you the basic facts about the request: what HTTP method was used (`GET`, `POST`, etc.), what URL path was requested, and whether the connection is secure (HTTPS). They are useful for logging, debugging, and making routing decisions.

### Purposes

- To track `req.method`, `req.url`, `req.path`, `req.secure`, and protocol schemas (HTTP vs. HTTPS).
- To log request details for debugging and monitoring.
- To enforce HTTPS by redirecting insecure requests.
- To make routing decisions based on the request method or path.

### Syntax Rules and Structure

```js
app.use((req, res, next) => {
  console.log('Method:', req.method);      // "GET"
  console.log('URL:', req.url);            // "/api/users?page=2"
  console.log('Path:', req.path);          // "/api/users"
  console.log('Secure:', req.secure);      // false
  console.log('Protocol:', req.protocol);  // "http"
  next();
});
```

| Property | Type | Description |
|----------|------|-------------|
| `req.method` | String | HTTP method (`GET`, `POST`, etc.). |
| `req.url` | String | Full URL including query string. |
| `req.path` | String | Path part only (no query string). |
| `req.secure` | Boolean | `true` if TLS connection. |
| `req.protocol` | String | `"http"` or `"https"`. |
| `req.originalUrl` | String | Original URL before router rewriting. |

**Rules:**
- `req.protocol` respects the `trust proxy` setting; with `trust proxy` enabled, it uses the `X-Forwarded-Proto` header.
- `req.secure` is a shorthand for `'https' == req.protocol`.
- `req.path` is the pathname only, excluding the query string.

**Constraints and Limitations:**
- `req.url` is not a native Express property; it comes from Node's HTTP module.
- `req.protocol` may be `"http"` behind a proxy unless `trust proxy` is configured.
- `req.originalUrl` retains the original URL even after router mounting rewrites `req.url`.

### Annotated Code Example

```js
// req-metadata.js
const express = require('express');
const app = express();

// Enable trust proxy for correct protocol detection
app.set('trust proxy', 1);

// Logging middleware using request metadata
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.originalUrl}`);
  console.log(`  Path: ${req.path}`);
  console.log(`  Protocol: ${req.protocol}`);
  console.log(`  Secure: ${req.secure}`);
  next();
});

// HTTPS enforcement middleware
app.use((req, res, next) => {
  if (!req.secure && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.get('host')}${req.originalUrl}`);
  }
  next();
});

app.get('/api/data', (req, res) => {
  res.json({
    method: req.method,
    path: req.path,
    protocol: req.protocol,
    secure: req.secure
  });
});

app.listen(3000, () => console.log('Metadata server on 3000'));
```

**Expected Output (for `GET /api/data` over HTTP):**
```
[2026-01-15T12:00:00.000Z] GET /api/data
  Path: /api/data
  Protocol: http
  Secure: false
{"method":"GET","path":"/api/data","protocol":"http","secure":false}
```

**Expected Output (for `POST /api/users?page=2`):**
```
[2026-01-15T12:00:01.000Z] POST /api/users?page=2
  Path: /api/users
  Protocol: https
  Secure: true
```

**Why this output:** The logging middleware uses `req.method`, `req.originalUrl`, `req.path`, `req.protocol`, and `req.secure` to log request details. The HTTPS enforcement middleware redirects insecure requests in production. The route handler returns the metadata as JSON.

### Real-World Cases

- **Logging:** Recording request method, path, and protocol for debugging.
- **HTTPS enforcement:** Redirecting HTTP requests to HTTPS in production.
- **API analytics:** Tracking which endpoints are most frequently accessed.
- **Security auditing:** Monitoring for unusual request patterns.

---

## References

- Express.js 5.x Request Object — https://expressjs.com/en/5x/api.html#req
- Express.js 4.x Request Object — https://expressjs.com/en/4x/api.html#req
- Express.js 5.x — `req.params` — https://expressjs.com/en/5x/api.html#req.params
- Express.js 5.x — `req.query` — https://expressjs.com/en/5x/api.html#req.query
- Express.js 5.x — `req.body` — https://expressjs.com/en/5x/api.html#req.body
- Express.js 5.x — `req.cookies` — https://expressjs.com/en/5x/api.html#req.cookies
- Express.js 5.x — `req.signedCookies` — https://expressjs.com/en/5x/api.html#req.signedCookies
- Express.js 5.x — `req.ip` — https://expressjs.com/en/5x/api.html#req.ip
- Express.js 5.x — `req.ips` — https://expressjs.com/en/5x/api.html#req.ips
- Express.js 5.x — `req.method` — https://expressjs.com/en/5x/api.html#req.method
- Express.js 5.x — `req.path` — https://expressjs.com/en/5x/api.html#req.path
- Express.js 5.x — `req.protocol` — https://expressjs.com/en/5x/api.html#req.protocol
- Express.js 5.x — `req.secure` — https://expressjs.com/en/5x/api.html#req.secure
- Express.js — Behind Proxies Guide — https://expressjs.com/en/guide/behind-proxies.html
- Express.js — body-parser Middleware — https://expressjs.com/en/resources/middleware/body-parser.html
- Express.js — cookie-parser Middleware — https://expressjs.com/en/resources/middleware/cookie-parser.html
- Express.js Routing Guide — https://expressjs.com/en/guide/routing.html
- Node.js `http.IncomingMessage` — https://nodejs.org/api/http.html#class-httpincomingmessage
- `express-query-parser2` — Nested Query Support — https://www.npmjs.com/package/express-query-parser2
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- MDN — HTTP Headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers