# Express.js Built-in Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Built-in middleware are middleware functions that ship with Express.js itself, providing essential request-processing capabilities—body parsing, static file serving, and content-type handling—without requiring external packages.

**Technical Definition**
Express re-exports several middleware functions from the `body-parser` and `serve-static` packages as top-level properties of the `express` module: `express.json()`, `express.urlencoded()`, `express.static()`, `express.raw()`, and `express.text()` . These middleware are included “out of the box” since Express 4.16.0 and are maintained as part of the Express ecosystem . Each parser populates `req.body` with a value whose type depends on the middleware: an object for JSON and URL-encoded forms, a string for text, and a Buffer for raw binary data . `express.static()` is distinct in that it serves files rather than parsing request bodies.

**Beginner-Friendly Explanation**
Express comes with a small set of built-in helpers that handle common jobs so you don't have to write them yourself. If you need to read JSON sent by a frontend, you use `express.json()`. If you have an old-fashioned HTML form, you use `express.urlencoded()`. If you want to serve images, CSS, and HTML files from a folder, you use `express.static()`. For plain text or raw binary uploads, you use `express.text()` or `express.raw()`.

---

### Key Characteristics

- **No extra installation** — Built-in middleware are available directly on the `express` object; no separate `npm install` is required for the parsers or static serving .
- **Content-Type driven** — Body parsers activate only when the request's `Content-Type` header matches their configured `type` option; non-matching requests pass through untouched .
- **Environment-aware bodies** — Parsers only populate `req.body` when the request actually has a body; otherwise `req.body` may be `{}` or `undefined` depending on the parser .
- **Security-sensitive defaults** — The default body size limit is `100kb`, and parsed bodies are explicitly documented as untrusted user-controlled input that must be validated .
- **Express 5 changes** — In Express 5, `express.urlencoded()` defaults `extended` to `false`, and `req.body` is no longer always initialized to `{}` .

---

### Prerequisites

- Node.js 18 or higher (required by Express 5) .
- An Express application initialized with `const app = express()`.
- Basic understanding of middleware ordering and the `(req, res, next)` signature.
- Familiarity with HTTP headers, especially `Content-Type`.

---

### Related Programming Areas

- **Middleware** — Built-in middleware are registered like any other middleware and follow the same `next()` rules.
- **Request body parsing** — Built-in parsers are the standard way to populate `req.body`.
- **Static file serving** — `express.static()` is the primary mechanism for serving frontend assets.
- **Security** — Body size limits, `dotfiles` handling, and input validation are critical security controls.
- **Content negotiation** — Parsers activate based on `Content-Type`, which is part of HTTP content negotiation.

---

## Core Concept 1: `express.json()`

### Definitions

**Core Definition**
`express.json()` is built-in middleware that parses incoming requests with JSON payloads and populates `req.body` with the resulting JavaScript object.

**Technical Definition**
`express.json([options])` returns middleware that only parses JSON and only looks at requests where the `Content-Type` header matches the `type` option (default `"application/json"`) . It accepts any Unicode encoding and supports automatic inflation of gzip and deflate encodings . The parsed body is a new object on `req.body`, or `undefined` if there was no body, the content type did not match, or an error occurred . Because `req.body` is based on user-controlled input, all properties and values are untrusted and must be validated before use .

**Beginner-Friendly Explanation**
When a modern frontend sends JSON data to your server—like `{ "name": "Alice" }` with the header `Content-Type: application/json`—`express.json()` reads that JSON and turns it into a JavaScript object you can access as `req.body.name`. Without it, `req.body` would be empty.

---

### Purposes

- To parse JSON request bodies into usable JavaScript objects for route handlers.
- To enforce a maximum body size, protecting the server from oversized-payload denial-of-service attacks.
- To support compressed request bodies through automatic gzip and deflate inflation.
- To provide a consistent interface for handling JSON-based API requests.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.json([options]));
```

**Component Breakdown**

| Option | Type | Default | Role |
|--------|------|---------|------|
| `inflate` | Boolean | `true` | Enable/disable handling of deflated (compressed) bodies . |
| `limit` | Mixed | `"100kb"` | Maximum request body size; numbers are bytes, strings use the `bytes` library . |
| `reviver` | Function | `null` | Passed directly to `JSON.parse` as the second argument . |
| `strict` | Boolean | `true` | Only accept arrays and objects; when `false`, accepts anything `JSON.parse` accepts . |
| `type` | Mixed | `"application/json"` | Content-Type filter; can be a string, array, or function . |
| `verify` | Function | `undefined` | Called as `verify(req, res, buf, encoding)` for custom verification . |

#### Syntax Rules

- Register the middleware before any route that needs `req.body`.
- The middleware only parses requests whose `Content-Type` matches `type` .
- Malformed JSON produces a `400` error; oversized bodies produce `413` .
- Empty bodies result in `req.body = {}` for `express.json()` .

#### Constraints and Limitations

- The parsed body is **untrusted**; validate every property before use .
- In Express 5, `req.body` may be `undefined` if no body was parsed; in earlier versions it was initialized to `{}` .
- `strict: true` rejects primitive JSON values like `"hello"` or `42` .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Basic JSON Parsing with Size Limit

```javascript
const express = require('express');
const app = express();

// Parse JSON bodies with a 50kb limit
app.use(express.json({ limit: '50kb' }));

app.post('/api/users', (req, res) => {
  // req.body is populated only if Content-Type matched
  const { name, age } = req.body;

  if (typeof name !== 'string' || typeof age !== 'number') {
    return res.status(400).json({ error: 'Invalid input' });
  }

  res.status(201).json({ user: { name, age } });
});

app.listen(3000);
```

**Step-by-Step Explanation**

1. **Middleware registration** — `express.json()` is mounted globally, so every request is checked for a matching `Content-Type`.
2. **Request parsing** — A request with `Content-Type: application/json` and a valid JSON body is parsed into `req.body`.
3. **Validation** — The handler explicitly checks the types of `name` and `age` before using them, because `req.body` is untrusted .

**Expected Output**

```
POST /api/users
Content-Type: application/json
{ "name": "Alice", "age": 30 }
→ 201 { "user": { "name": "Alice", "age": 30 } }

POST /api/users
Content-Type: application/json
{ "name": 123, "age": "thirty" }
→ 400 { "error": "Invalid input" }
```

**Why This Output Occurs**

- The parser only populates `req.body` when the content type matches; the handler validates types because the body is user-controlled .

---

### Real-World Cases

- **REST APIs** — Every modern JSON API uses `express.json()` to read request payloads.
- **Webhook endpoints** — JSON payloads from Stripe, GitHub, and other services are parsed with `express.json()`.
- **GraphQL over HTTP** — JSON request bodies carry GraphQL queries and variables.

---

## Core Concept 2: `express.urlencoded()`

### Definitions

**Core Definition**
`express.urlencoded()` is built-in middleware that parses incoming requests with URL-encoded payloads—typically submitted by HTML forms—and populates `req.body` with the parsed fields.

**Technical Definition**
`express.urlencoded([options])` returns middleware that parses bodies with `Content-Type: application/x-www-form-urlencoded` . The `extended` option controls the parsing library: when `false`, it uses Node's `querystring` module (flat key-value pairs only); when `true`, it uses the `qs` library, which supports nested objects and arrays . In Express 5, `extended` defaults to `false` . The middleware only activates when the `Content-Type` matches, and the resulting `req.body` is an object whose shape depends on the parser used.

**Beginner-Friendly Explanation**
When a user submits a traditional HTML form, the browser sends the data as `key=value&key2=value2`. `express.urlencoded()` reads that format and turns it into an object you can access as `req.body.username`, `req.body.password`, and so on. The `extended` option determines whether nested structures like `user[name]=Alice` are supported.

---

### Purposes

- To parse HTML form submissions into usable JavaScript objects.
- To support both flat and nested form data through the `extended` option.
- To provide a consistent body-parsing interface alongside `express.json()` for applications that accept both JSON and forms.
- To enforce body size limits and content-type filtering for form requests.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.urlencoded({ extended: true | false, limit: '100kb' }));
```

**Component Breakdown**

| Option | Type | Default | Role |
|--------|------|---------|------|
| `extended` | Boolean | `false` (Express 5) | `true` uses `qs` for nested objects; `false` uses `querystring` for flat keys . |
| `limit` | Mixed | `"100kb"` | Maximum request body size. |
| `parameterLimit` | Number | `1000` | Maximum number of parameters (qs parser). |
| `type` | Mixed | `"application/x-www-form-urlencoded"` | Content-Type filter. |

#### Syntax Rules

- `extended` must be explicitly set; its default was deprecated in Express 4 and changed to `false` in Express 5 .
- With `extended: true`, nested keys like `user[name]=Alice` become `{ user: { name: 'Alice' } }`.
- With `extended: false`, nested keys remain flat or are not parsed as expected .
- `multipart/form-data` (file uploads) is **not** handled by this middleware; use `multer` or `busboy` .

#### Constraints and Limitations

- `extended: false` cannot parse nested objects or arrays .
- `extended: true` introduces the `qs` library's depth and parameter limits; configure `depth` and `parameterLimit` for security .
- The middleware does not handle file uploads.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Parsing a Login Form

```javascript
const express = require('express');
const app = express();

// Parse URL-encoded form bodies
app.use(express.urlencoded({ extended: true, limit: '10kb' }));

app.post('/login', (req, res) => {
  const { username, password } = req.body;

  if (!username || !password) {
    return res.status(400).json({ error: 'Missing fields' });
  }

  res.json({ message: `Logged in as ${username}` });
});

app.listen(3000);
```

**Expected Output**

```
POST /login
Content-Type: application/x-www-form-urlencoded

username=alice&password=secret
→ 200 { "message": "Logged in as alice" }

POST /login
(username=&password=)
→ 400 { "error": "Missing fields" }
```

**Why This Output Occurs**

- The middleware parses the URL-encoded body into `req.body`; the handler checks for required fields .

---

### Real-World Cases

- **Traditional server-rendered forms** — Login, registration, and search forms submit URL-encoded data.
- **Legacy integrations** — Older APIs and payment gateways often use URL-encoded payloads.
- **Hybrid applications** — Apps that accept both JSON APIs and HTML forms mount both `express.json()` and `express.urlencoded()` .

---

## Core Concept 3: `express.static()`

### Definitions

**Core Definition**
`express.static()` is built-in middleware that serves static files—such as images, CSS, JavaScript, and HTML—from a specified root directory based on the request URL.

**Technical Definition**
`express.static(root, [options])` serves static assets by combining `req.url` with the provided `root` directory to determine the file to serve . It is based on the `serve-static` package . When a file is not found, instead of sending a `404`, it calls `next()` to allow subsequent middleware and fallback routes to run . The middleware supports extensive options including `dotfiles`, `etag`, `extensions`, `fallthrough`, `immutable`, `index`, `lastModified`, `maxAge`, `redirect`, `setHeaders`, `acceptRanges`, and `cacheControl` .

**Beginner-Friendly Explanation**
`express.static()` lets you serve a folder of files directly to the browser. If you have a `public` folder with `style.css` and `logo.png`, you can make them available at `/style.css` and `/logo.png`. It handles caching headers, ETags, and content types automatically.

---

### Purposes

- To serve frontend assets (CSS, JavaScript, images, fonts) without writing individual routes.
- To enable caching through `Cache-Control`, `ETag`, and `Last-Modified` headers.
- To control access to dotfiles and directory listings.
- To provide fallback behavior when files are missing through `next()` .

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.static(root, [options]));
```

**Component Breakdown**

| Option | Type | Default | Role |
|--------|------|---------|------|
| `dotfiles` | String | `"ignore"` | `"allow"`, `"deny"` (403), or `"ignore"` (404) . |
| `etag` | Boolean | `true` | Enable weak ETag generation . |
| `extensions` | Mixed | `false` | File extension fallbacks, e.g., `['html', 'htm']` . |
| `fallthrough` | Boolean | `true` | Call `next()` on client errors instead of forwarding an error . |
| `immutable` | Boolean | `false` | Add `immutable` to `Cache-Control`; requires `maxAge` . |
| `index` | Mixed | `"index.html"` | Directory index file; `false` disables . |
| `lastModified` | Boolean | `true` | Set `Last-Modified` header . |
| `maxAge` | Number | `0` | `Cache-Control` max-age in milliseconds or ms string . |
| `redirect` | Boolean | `true` | Redirect to trailing `/` for directories . |
| `setHeaders` | Function | — | Custom header setter . |
| `acceptRanges` | Boolean | `true` | Enable ranged requests . |
| `cacheControl` | Boolean | `true` | Enable `Cache-Control`; disabling ignores `immutable` and `maxAge` . |

#### Syntax Rules

- The `root` argument is required and determines the directory from which files are served .
- When a file is not found, `next()` is called (with `fallthrough: true`) rather than sending a `404` .
- `dotfiles: 'ignore'` acts as if the dotfile does not exist, returning `404` and calling `next()` .
- `immutable` should only be used with `maxAge` for versioned assets .

#### Constraints and Limitations

- Serving static files directly from Express is less efficient than using a reverse proxy or CDN for production .
- `dotfiles: 'deny'` returns `403`; `'ignore'` returns `404` .
- The middleware does not provide directory listings by default; `index` serves a file if present.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Serving a Public Directory with Caching

```javascript
const express = require('express');
const path = require('path');
const app = express();

app.use(
  express.static(path.join(__dirname, 'public'), {
    maxAge: '1d',          // cache for 1 day
    etag: true,            // send weak ETags
    dotfiles: 'ignore',    // hide dotfiles
    index: 'index.html'    // serve index.html for directory requests
  })
);

app.listen(3000);
```

**Expected Output**

```
GET /style.css
→ 200 (file contents)
Cache-Control: public, max-age=86400
ETag: W/"..."

GET /.env
→ 404 (dotfile ignored; next() called)
```

**Why This Output Occurs**

- `maxAge` sets the `Cache-Control` header; `etag` enables conditional requests; `dotfiles: 'ignore'` hides dotfiles .

---

### Real-World Cases

- **Single-page applications** — Serving `index.html`, JavaScript bundles, and CSS from a `dist` or `public` folder.
- **Image and asset hosting** — Serving user-uploaded images with caching headers.
- **Documentation sites** — Serving static HTML documentation.

---

## Core Concept 4: `express.raw()`

### Definitions

**Core Definition**
`express.raw()` is built-in middleware that parses incoming request payloads into a `Buffer` and populates `req.body` with the raw binary data.

**Technical Definition**
`express.raw([options])` returns middleware that parses all bodies as a `Buffer` and only looks at requests where the `Content-Type` header matches the `type` option (default `"application/octet-stream"`) . It accepts any Unicode encoding and supports gzip and deflate inflation . The resulting `req.body` is a `Buffer`, or `undefined` if there was no body, the content type did not match, or an error occurred . Because the body may come from a different parser if multiple parsers are stacked, testing that `req.body` is a Buffer before calling Buffer methods is recommended .

**Beginner-Friendly Explanation**
Some requests send raw binary data—like file uploads or cryptographic signatures. `express.raw()` captures that data as a Buffer, which you can then process, save to disk, or verify.

---

### Purposes

- To capture raw binary request bodies for file uploads, image processing, or signature verification.
- To provide a Buffer representation of the body for use with libraries that expect binary data.
- To limit the size of binary uploads to prevent memory exhaustion.
- To support webhook signature verification that requires the exact raw body bytes.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.raw([options]));
```

**Component Breakdown**

| Option | Type | Default | Role |
|--------|------|---------|------|
| `inflate` | Boolean | `true` | Enable/disable compressed body handling . |
| `limit` | Mixed | `"100kb"` | Maximum body size . |
| `type` | Mixed | `"application/octet-stream"` | Content-Type filter . |
| `verify` | Function | `undefined` | Custom verification function . |

#### Syntax Rules

- The parsed body is a `Buffer`; use `Buffer.isBuffer(req.body)` before calling Buffer methods .
- The middleware only activates for matching `Content-Type` headers .
- Oversized bodies produce a `413` error.

#### Constraints and Limitations

- Stacking multiple parsers can cause `req.body` to be from a different parser; always verify the type .
- `express.raw()` is not a replacement for multipart form parsing; use `multer` for file uploads with fields .
- The default `100kb` limit is small for binary uploads; increase it deliberately .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Capturing a Raw Binary Upload

```javascript
const express = require('express');
const app = express();

// Parse raw binary bodies up to 5mb
app.post(
  '/upload',
  express.raw({ type: 'application/octet-stream', limit: '5mb' }),
  (req, res) => {
    if (!Buffer.isBuffer(req.body)) {
      return res.status(400).json({ error: 'Expected binary body' });
    }

    res.json({ bytesReceived: req.body.length });
  }
);

app.listen(3000);
```

**Expected Output**

```
POST /upload
Content-Type: application/octet-stream
<binary data>
→ 200 { "bytesReceived": 1024 }
```

**Why This Output Occurs**

- The middleware captures the raw bytes as a Buffer; the handler verifies the type and reports the size .

---

### Real-World Cases

- **Webhook signature verification** — Stripe and other providers require the raw body to verify signatures.
- **Binary file uploads** — Capturing raw image or PDF data before processing.
- **Protocol buffers** — Receiving protobuf-encoded payloads as Buffers.

---

## Core Concept 5: `express.text()`

### Definitions

**Core Definition**
`express.text()` is built-in middleware that parses incoming request payloads as plain text and populates `req.body` with a string.

**Technical Definition**
`express.text([options])` returns middleware that parses bodies with `Content-Type: text/plain` (or a custom `type`) into a string . It supports options including `defaultCharset`, `inflate`, `limit`, `type`, and `verify` . The resulting `req.body` is a string, or `undefined` if there was no body or the content type did not match. Like other body parsers, it is based on `body-parser` and is re-exported by Express .

**Beginner-Friendly Explanation**
When a client sends plain text—like a log line, a Markdown document, or a simple message—`express.text()` gives you that text as a string in `req.body`. It is useful when you don't need JSON or form parsing.

---

### Purposes

- To receive plain-text request bodies as strings for processing or storage.
- To parse custom text-based content types (e.g., `text/csv`, `application/toml`) by customizing `type` .
- To enforce size limits on text payloads.
- To support webhook or logging endpoints that accept raw text.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.text([options]));
```

**Component Breakdown**

| Option | Type | Default | Role |
|--------|------|---------|------|
| `defaultCharset` | String | `"utf-8"` | Charset when not specified in `Content-Type` . |
| `inflate` | Boolean | `true` | Enable compressed body handling . |
| `limit` | Mixed | `"100kb"` | Maximum body size . |
| `type` | Mixed | `"text/plain"` | Content-Type filter . |
| `verify` | Function | `undefined` | Custom verification function . |

#### Syntax Rules

- The parsed body is a string.
- Only requests with matching `Content-Type` are parsed; others pass through .
- Custom `type` values (e.g., `"application/toml"`) extend the middleware to other text-based formats .

#### Constraints and Limitations

- `express.text()` does not parse structured formats; use `express.json()` or a dedicated parser for those.
- The default limit is `100kb`; increase it for larger text payloads.
- The middleware does not handle binary data; use `express.raw()` for that.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Parsing Plain Text and Custom Text Types

```javascript
const express = require('express');
const app = express();

// Parse plain text bodies
app.use(express.text({ type: 'text/plain', limit: '1mb' }));

app.post('/log', (req, res) => {
  if (typeof req.body !== 'string') {
    return res.status(400).json({ error: 'Expected text' });
  }
  res.json({ length: req.body.length });
});

app.listen(3000);
```

**Expected Output**

```
POST /log
Content-Type: text/plain

Hello, world!
→ 200 { "length": 13 }
```

**Why This Output Occurs**

- The middleware parses the text body into a string; the handler measures its length .

---

### Real-World Cases

- **Log ingestion** — Receiving plain-text log lines from clients.
- **Markdown editors** — Accepting raw Markdown content.
- **Custom text formats** — Parsing CSV or TOML with a custom `type` .

---

## Core Concept 6: Built-in Middleware Best Practices

### Definitions

**Core Definition**
Built-in middleware best practices are the configuration and ordering patterns that keep body parsing and static file serving secure, performant, and predictable.

**Technical Definition**
Best practices include: registering body parsers only where needed or with explicit `limit` values; setting a deliberate `limit` on every parser to prevent denial-of-service through oversized payloads; validating every property of `req.body` because it is untrusted user input ; choosing `extended: false` for URL-encoded forms unless nested objects are required; configuring `dotfiles: 'ignore'` or `'deny'` for static serving; using a reverse proxy or CDN for production static assets ; and ordering parsers before routes and error handlers last .

**Beginner-Friendly Explanation**
Best practices are the habits that keep these built-in helpers from becoming security holes. Always set a size limit so nobody can crash your server with a giant request. Always validate what comes in, because attackers control it. And serve static files through a proper web server in production if you can.

---

### Purposes

- To prevent denial-of-service attacks through unbounded request bodies.
- To ensure parsed data is treated as untrusted and validated before use.
- To minimize attack surface by applying parsers only to routes that need them.
- To optimize static asset delivery through caching and reverse proxies.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Parser Configuration

```javascript
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ extended: false, limit: '100kb' }));

app.use(express.static('public', {
  dotfiles: 'ignore',
  maxAge: '1d',
  index: 'index.html'
}));
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Size limits | `limit: '100kb'` on every parser . |
| Untrusted input | Validate every property of `req.body` . |
| Minimal extended | `extended: false` unless nesting is needed . |
| Dotfile safety | `dotfiles: 'ignore'` or `'deny'` . |
| Production static | Use a reverse proxy or CDN . |

#### Syntax Rules

- Set an explicit `limit` on every body parser; the default `100kb` is a starting point .
- Validate all `req.body` properties before use; never trust their types or existence .
- Register parsers before routes that need `req.body`, and error handlers last .
- For static files, configure `dotfiles` to prevent accidental exposure of `.env` or `.git` files .

#### Constraints and Limitations

- Increasing `limit` for binary uploads also increases memory pressure; use streaming uploads for large files.
- `express.static()` is convenient but not the most performant option for high-traffic production deployments .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Hardened Built-in Middleware Setup

```javascript
const express = require('express');
const path = require('path');
const app = express();

// 1. Body parsers with explicit limits
app.use(express.json({ limit: '100kb' }));
app.use(express.urlencoded({ extended: false, limit: '100kb' }));

// 2. Static files with safe dotfile handling
app.use(
  express.static(path.join(__dirname, 'public'), {
    dotfiles: 'ignore',
    maxAge: '1d',
    index: 'index.html'
  })
);

// 3. Routes
app.post('/api/data', (req, res) => {
  if (!req.body || typeof req.body !== 'object') {
    return res.status(400).json({ error: 'Invalid body' });
  }
  res.json({ received: req.body });
});

// 4. Error handler last
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ error: err.message });
});

app.listen(3000);
```

**Expected Output**

```
POST /api/data
Content-Type: application/json
{ "name": "Alice" }
→ 200 { "received": { "name": "Alice" } }

GET /.env
→ 404 (dotfile ignored)

POST /api/data  (body > 100kb)
→ 413 Payload Too Large
```

**Why This Output Occurs**

- The parsers enforce size limits; the static middleware hides dotfiles; the error handler catches oversized-body errors .

---

### Real-World Cases

- **Production APIs** — Body size limits, input validation, and centralized error handling.
- **Static asset hosting** — Reverse proxy or CDN in front of `express.static()` for performance and caching.
- **Security audits** — `dotfiles: 'ignore'`, explicit `limit`, and untrusted-input validation are standard hardening checks.

---

## References

- Express 5.x API Reference (`express.json`, `express.raw`, `express.static`) — https://expressjs.com/en/5x/api.html
- Express 5.x API Reference (`express.urlencoded` and body parsing overview) — https://expressjs.com/en/5x/api.html
- Express `serve-static` middleware documentation — https://expressjs.com/en/resources/middleware/serve-static.html
- Express 如何获取 POST 表单字段？ (urlencoded and extended option) — https://www.guance.com/learn/articles/how-to-access-post-fields-in-express
- Express `urlencoded` extended option (Stack Overflow revision) — https://stackoverflow.com/revisions/8553cb97-3e0d-4057-85af-96ce34297f1e/view-source
- Body Parsing Middleware (DeepWiki) — https://deepwiki.com/expressjs/express/4.2-static-file-middleware
- Securing Express.js Applications (Express 5) — 2026 Playbook — https://safeguard.sh/resources/blog/securing-express-applications
- Express Middleware Order (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/2-express-middleware/7-express-middleware-order.mdx
- Express `express.text()` API (raw documentation) — https://raw.githubusercontent.com/expressjs/expressjs.com/gh-pages/en/5x/api/express.md
- Express `express.raw()` API (raw documentation) — https://raw.githubusercontent.com/expressjs/expressjs.com/gh-pages/en/5x/api/express.md
- `body-parser` documentation — https://expressjs.com/resources/middleware/body-parser.html