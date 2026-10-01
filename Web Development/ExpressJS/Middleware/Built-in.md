# Express.js Built-in Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Built-in middleware functions are utility functions shipped with Express.js that handle common request-processing tasks — such as parsing request bodies and serving static files — without requiring external dependencies.

**Technical Definition:** Express.js provides five built-in middleware functions: `express.json()`, `express.urlencoded()`, `express.static()`, `express.text()`, and `express.raw()`. All four body-parsing middleware are based on the `body-parser` module, which was previously a separate dependency and is now bundled with Express (available since Express 4.16.0). `express.static()` is based on the `serve-static` module. These middleware functions are attached to the application using `app.use()`.

**Beginner-Friendly Explanation:** Express comes with a set of pre-made tools for common jobs. Instead of writing code to read JSON data from a request, or to serve images and CSS files, you simply add one line like `app.use(express.json())` and Express handles it for you. These tools are called "built-in middleware" because they ship with Express and don't require installing anything extra.

### Key Characteristics

- **No external installation:** Built into Express 4.16.0 and above.
- **Based on battle-tested libraries:** `body-parser` for body parsing, `serve-static` for static files.
- **Content-Type aware:** Body parsers only activate when the request's `Content-Type` header matches.
- **Configurable:** Each middleware accepts an options object for limits, types, and behaviour.
- **Middleware chain compatible:** They follow the standard `(req, res, next)` signature.
- **Security-conscious:** Options like `limit` and `parameterLimit` prevent abuse.

### Prerequisites

- **Node.js runtime** (v18 or higher for Express 5.x; v4.16+ for built-in middleware).
- **Express.js installed:** `npm install express`.
- **Basic JavaScript knowledge:** Functions, objects, and middleware concepts.
- **Understanding of HTTP:** Content types, request methods, and the request-response cycle.

### Related Programming Areas

- **Body parsing:** Extracting structured data from request payloads.
- **Static file serving:** Delivering CSS, JavaScript, images, and other assets.
- **API development:** JSON and URL-encoded parsing are essential for REST APIs.
- **Security:** Size limits and parameter limits prevent DoS attacks.
- **Performance:** Caching headers for static assets improve load times.

### Core Concepts

1. **`express.json()`** — parsing JSON request bodies.
2. **`express.urlencoded()`** — parsing URL-encoded form data (`extended: true` vs `false`).
3. **`express.static()`** — serving static files with caching and absolute paths.
4. **`express.text()`** — parsing plain text request bodies.
5. **`express.raw()`** — parsing raw binary request bodies into Buffers.

---

## Core Concept 1: `express.json()`

### Definitions

**Core Definition:** `express.json()` is a built-in middleware function that parses incoming requests with JSON payloads and populates `req.body` with the parsed JavaScript object.

**Technical Definition:** `express.json([options])` returns middleware that only parses JSON and only looks at requests where the `Content-Type` header matches the `type` option (default: `application/json`). The parser accepts any Unicode encoding of the body and supports automatic inflation of `gzip` and `deflate` encodings. A new `body` object containing the parsed data is populated on the request object after the middleware (i.e., `req.body`), or an empty object (`{}`) if there was no body to parse, the `Content-Type` was not matched, or an error occurred. 

**Beginner-Friendly Explanation:** When a client sends JSON data to your server (like `{ "name": "Alice" }`), Express doesn't understand it by default — it just sees a stream of bytes. `express.json()` is like a translator that reads the JSON and turns it into a JavaScript object you can work with using `req.body.name`.

### Purposes

- To parse incoming requests with JSON payloads and make the result available on `req.body`.
- To enable REST API endpoints that accept JSON data (e.g., POST, PUT, PATCH).
- To automatically decompress gzipped or deflated JSON bodies.
- To enforce size limits on JSON payloads to prevent abuse.

### Syntax Rules and Structure

#### General Syntax

```js
app.use(express.json([options]));
```

| Component | Breakdown |
|-----------|-----------|
| `express.json()` | The middleware factory function. |
| `options` | Optional configuration object. |

#### Options Breakdown

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `inflate` | Boolean | `true` | Handle deflated (compressed) bodies. |
| `limit` | Mixed | `"100kb"` | Maximum request body size. |
| `reviver` | Function | `null` | Passed to `JSON.parse()` as the reviver. |
| `strict` | Boolean | `true` | Only accept arrays and objects. |
| `type` | Mixed | `"application/json"` | Media type to parse. |
| `verify` | Function | `undefined` | Called as `verify(req, res, buf, encoding)`. |

#### Syntax Rules

- Must be mounted with `app.use()` before route handlers that need `req.body`.
- Only parses requests where the `Content-Type` header matches the `type` option.
- `req.body` is populated with the parsed object, or `{}` if no body or type mismatch.
- The `strict` option defaults to `true`, meaning only arrays and objects are accepted.

#### Constraints and Limitations

- **Security warning:** `req.body` is user-controlled input and must be validated before trusting. `req.body.foo.toString()` may fail if `foo` is not a string. 
- Multipart bodies (file uploads) are not handled; use Multer.
- Default size limit is 100 KB; adjust with the `limit` option for larger payloads.

### Annotated Code Examples

#### Example 1: Basic JSON Parsing

```js
// json-basic.js
const express = require('express');
const app = express();

// Mount JSON body parser middleware
app.use(express.json());

app.post('/api/users', (req, res) => {
  // req.body contains the parsed JSON
  console.log('Received:', req.body);

  const { name, email } = req.body;

  if (!name) {
    return res.status(400).json({ error: 'Name is required' });
  }

  res.status(201).json({
    message: 'User created',
    user: { name, email }
  });
});

app.listen(3000, () => console.log('Server on port 3000'));
```

**Expected Output (for `POST /api/users` with `{ "name": "Alice", "email": "alice@example.com" }`):**
```
Received: { name: 'Alice', email: 'alice@example.com' }
{"message":"User created","user":{"name":"Alice","email":"alice@example.com"}}
```

**Expected Output (for `POST /api/users` with `Content-Type: text/plain`):**
```
Received: {}
{"error":"Name is required"}
```

**Why this output:** When the `Content-Type` is `application/json`, the middleware parses the body and populates `req.body`. When the content type doesn't match, `req.body` is an empty object `{}`, so the `name` validation fails.

#### Example 2: Configuring Limits and Strict Mode

```js
// json-options.js
const express = require('express');
const app = express();

// Configure JSON parser with a 1MB limit and non-strict mode
app.use(express.json({
  limit: '1mb',        // Allow up to 1MB
  strict: false        // Accept primitives (strings, numbers, booleans)
}));

app.post('/api/data', (req, res) => {
  res.json({ received: req.body, type: typeof req.body });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/data` with body `"hello"`):**
```
{"received":"hello","type":"string"}
```

**Expected Output (for `POST /api/data` with a 2MB JSON payload):**
```
HTTP/1.1 413 Payload Too Large
```

**Why this output:** The `limit: '1mb'` option rejects payloads larger than 1 MB with a 413 error. The `strict: false` option allows the parser to accept JSON primitives like strings, which would be rejected in strict mode (default).

### Real-World Cases

- **REST APIs:** `POST /api/users` accepting JSON user data.
- **Webhooks:** Receiving JSON event payloads from payment providers.
- **SPA backends:** JSON request bodies from React/Vue/Angular applications.
- **Configuration endpoints:** Accepting JSON configuration updates.

---

## Core Concept 2: `express.urlencoded()`

### Definitions

**Core Definition:** `express.urlencoded()` is a built-in middleware function that parses incoming requests with URL-encoded payloads (typically from HTML forms) and populates `req.body` with the parsed data.

**Technical Definition:** `express.urlencoded([options])` returns middleware that parses bodies with the `application/x-www-form-urlencoded` content type. The `extended` option controls the parsing library: when `false` (default), it uses Node's built-in `querystring` module (flat key-value pairs only); when `true`, it uses the `qs` library, which supports nested objects and arrays.  The `extended` syntax allows for rich objects and arrays to be encoded into the URL-encoded format, allowing for a JSON-like experience with URL-encoded data. 

**Beginner-Friendly Explanation:** HTML forms send data in a format called "URL-encoded" — it looks like `name=Alice&age=30`. `express.urlencoded()` translates this into a JavaScript object. The `extended` option lets you choose between simple parsing (just key-value pairs) and richer parsing that supports nested objects and arrays.

### Purposes

- To parse incoming requests with URL-encoded payloads.
- To handle HTML form submissions (contact forms, login forms, etc.).
- To enable nested data structures via the `extended: true` option.
- To enforce parameter limits and size limits for security.

### Sub-Feature 2.1: `extended: false` vs `extended: true`

#### Syntax Rules and Structure

```js
app.use(express.urlencoded({ extended: true }));  // or false
```

| Option | Library | Nested Objects | Arrays | Performance |
|--------|---------|---------------|--------|-------------|
| `extended: false` | `querystring` | ❌ | Limited | Faster |
| `extended: true` | `qs` | ✅ | ✅ | Slower |

#### Comparison Table

| Input | `extended: false` | `extended: true` |
|-------|-------------------|------------------|
| `name=Ashish&age=21` | `{ name: 'Ashish', age: '21' }` | `{ name: 'Ashish', age: '21' }` |
| `user[name]=Ashish&user[age]=21` | `{ 'user[name]': 'Ashish', 'user[age]': '21' }` | `{ user: { name: 'Ashish', age: '21' } }` |
| `colors[]=red&colors[]=blue` | `{ 'colors[]': ['red','blue'] }` | `{ colors: ['red', 'blue'] }` |

#### Syntax Rules

- `extended: false` uses `querystring.parse()` — flat key-value pairs only. 
- `extended: true` uses `qs.parse()` — supports nested objects and arrays. 
- The `parameterLimit` option (default 1000) controls the maximum number of parameters. 
- The `limit` option controls the maximum body size.

#### Constraints and Limitations

- **Security:** Deep nesting with `extended: true` can cause CPU/memory spikes (DoS). Mitigate with `limit` and `parameterLimit`. 
- **Prototype pollution:** Keep `qs` updated and validate all input. 
- **Everything is a string:** URL-encoded data does not preserve types (numbers, booleans, null). 

### Annotated Code Example

```js
// urlencoded-extended.js
const express = require('express');
const app = express();

// Use extended: true for nested form data
app.use(express.urlencoded({ extended: true, limit: '100kb', parameterLimit: 1000 }));

app.post('/submit', (req, res) => {
  console.log('Parsed body:', req.body);
  res.json({
    received: req.body,
    hasNested: !!(req.body.user && req.body.user.name)
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for a form POST with `user[name]=Alice&user[age]=30&colors[]=red&colors[]=blue`):**
```json
{
  "received": {
    "user": { "name": "Alice", "age": "30" },
    "colors": ["red", "blue"]
  },
  "hasNested": true
}
```

**Expected Output (with `extended: false` for the same input):**
```json
{
  "received": {
    "user[name]": "Alice",
    "user[age]": "30",
    "colors[]": ["red", "blue"]
  },
  "hasNested": false
}
```

**Why this output:** With `extended: true`, the `qs` library interprets the bracket notation and creates nested objects and arrays. With `extended: false`, the `querystring` library keeps the brackets as literal characters in the key names, resulting in a flat object.

### Real-World Cases

- **HTML forms:** Login, registration, and contact forms.
- **Legacy APIs:** Systems that use URL-encoded data instead of JSON.
- **Nested forms:** Forms with complex field structures (e.g., address forms).
- **OAuth callbacks:** Some OAuth providers use URL-encoded responses.

---

## Core Concept 3: `express.static()`

### Definitions

**Core Definition:** `express.static()` is a built-in middleware function that serves static files (images, CSS, JavaScript, HTML) from a specified directory.

**Technical Definition:** `express.static(root, [options])` serves static files and is based on the `serve-static` module. The `root` argument specifies the root directory from which to serve static assets. The function determines the file to serve by combining `req.url` with the provided `root` directory. When a file is not found, instead of sending a 404 response, it calls `next()` to move on to the next middleware, allowing for stacking and fallbacks. 

**Beginner-Friendly Explanation:** `express.static()` is like a file server built into your Express app. You tell it "any files in my `public` folder can be accessed by anyone," and Express automatically serves them. When someone visits `/images/logo.png`, Express finds that file in the `public` folder and sends it back.

### Purposes

- To serve static assets (images, CSS, JavaScript, fonts) without writing route handlers.
- To configure cache control headers for improved performance.
- To serve files from absolute paths using `path.join(__dirname, 'public')`.
- To support Single Page Applications (SPAs) with fallback routing.

### Syntax Rules and Structure

#### General Syntax

```js
app.use(express.static(root, [options]));
app.use('/prefix', express.static(root, [options]));
```

| Component | Breakdown |
|-----------|-----------|
| `root` | The root directory from which to serve static assets. |
| `options` | Optional configuration object. |
| `'/prefix'` | Optional URL prefix for the static files. |

#### Options Breakdown

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `dotfiles` | String | `undefined` | How to handle dotfiles (`allow`, `deny`, `ignore`). |
| `etag` | Boolean | `true` | Enable ETag generation (always weak). |
| `extensions` | Mixed | `false` | File extension fallbacks (e.g., `['html', 'htm']`). |
| `fallthrough` | Boolean | `true` | Let client errors fall through to next middleware. |
| `immutable` | Boolean | `false` | Enable `immutable` directive in Cache-Control. |
| `index` | Mixed | `"index.html"` | Directory index file; `false` to disable. |
| `lastModified` | Boolean | `true` | Set `Last-Modified` header. |
| `maxAge` | Number/String | `0` | Cache-Control `max-age` in ms or ms format string. |
| `redirect` | Boolean | `true` | Redirect to trailing "/" for directories. |
| `setHeaders` | Function | — | Custom header setter `(res, path, stat)`. |
| `acceptRanges` | Boolean | `true` | Enable ranged requests. |
| `cacheControl` | Boolean | `true` | Enable `Cache-Control` header. |

#### Syntax Rules

- The `root` path is relative to the directory from which the Node process is launched. For safety, use `path.join(__dirname, 'public')` to create an absolute path. 
- When a file is not found, `express.static()` calls `next()` instead of sending a 404. 
- Multiple static directories can be served by calling `express.static()` multiple times.
- `maxAge` is in milliseconds; it is converted to seconds for the `Cache-Control: max-age` header. 

#### Constraints and Limitations

- **Best practice:** Use a reverse proxy or CDN for static assets in production. 
- The `setHeaders` function cannot modify the `Content-Type` header after it has been set.
- Dotfiles are not served by default (they are ignored).

### Annotated Code Examples

#### Example 1: Basic Static File Serving

```js
// static-basic.js
const express = require('express');
const path = require('path');
const app = express();

// Serve files from the 'public' directory using an absolute path
app.use(express.static(path.join(__dirname, 'public')));

// Serve files from a custom URL prefix
app.use('/assets', express.static(path.join(__dirname, 'public')));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /styles/main.css`):**
```
(Serves the file at public/styles/main.css)
```

**Expected Output (for `GET /assets/styles/main.css`):**
```
(Serves the same file via the /assets prefix)
```

**Why this output:** `path.join(__dirname, 'public')` creates an absolute path to the `public` directory, ensuring it works regardless of where the Node process is launched. The `/assets` prefix creates an alternative URL for the same files.

#### Example 2: Configuring Cache Control and Custom Headers

```js
// static-cache.js
const express = require('express');
const path = require('path');
const app = express();

app.use('/static', express.static(path.join(__dirname, 'public'), {
  maxAge: '1d',                // Cache for 1 day (86,400,000 ms)
  etag: true,                  // Enable weak ETags
  lastModified: true,          // Set Last-Modified header
  immutable: true,             // Prevent conditional requests during maxAge
  setHeaders: (res, filePath) => {
    // Custom headers for specific file types
    if (path.extname(filePath) === '.html') {
      res.setHeader('Cache-Control', 'no-cache');  // HTML: always revalidate
    }
  }
}));

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `GET /static/logo.png`):**
```
HTTP/1.1 200 OK
Cache-Control: public, max-age=86400, immutable
ETag: W/"abc123"
Last-Modified: Wed, 15 Jan 2026 00:00:00 GMT
```

**Expected Output (for `GET /static/index.html`):**
```
HTTP/1.1 200 OK
Cache-Control: no-cache
```

**Why this output:** The `maxAge: '1d'` option sets `Cache-Control: public, max-age=86400` for all files. The `setHeaders` function overrides this for HTML files, setting `no-cache` to ensure users always get the latest HTML. The `immutable` directive tells browsers not to revalidate the file during the cache period.

### Real-World Cases

- **Web applications:** Serving CSS, JavaScript, and images from a `public` folder.
- **SPAs:** Serving the `dist` build folder with a fallback to `index.html`.
- **CDN origins:** Serving static assets that are cached by a CDN.
- **Documentation sites:** Serving generated HTML, CSS, and JS files.

---

## Core Concept 4: `express.text()`

### Definitions

**Core Definition:** `express.text()` is a built-in middleware function that parses incoming request payloads into a string and populates `req.body` with the decoded text.

**Technical Definition:** `express.text([options])` parses all bodies as a string and only looks at requests where the `Content-Type` header matches the `type` option (default: `text/plain`). The parser accepts any Unicode encoding of the body and supports automatic inflation of `gzip` and `deflate` encodings. A new `body` string containing the parsed data is populated on the request object (i.e., `req.body`), or an empty object (`{}`) if there was no body to parse or the content type was not matched. 

**Beginner-Friendly Explanation:** `express.text()` is like `express.json()` but for plain text. When a client sends raw text (like a plain string or some custom format), this middleware turns the raw bytes into a string you can use.

### Purposes

- To parse plain text request bodies into strings.
- To handle custom text-based protocols (e.g., CSV, plain text APIs).
- To receive text data from clients that don't use JSON or forms.

### Syntax Rules and Structure

```js
app.use(express.text([options]));
```

#### Options Breakdown

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `defaultCharset` | String | `"utf-8"` | Default character set if not in `Content-Type`. |
| `inflate` | Boolean | `true` | Handle compressed bodies. |
| `limit` | Mixed | `"100kb"` | Maximum body size. |
| `type` | Mixed | `"text/plain"` | Media type to parse. |
| `verify` | Function | `undefined` | Custom verification function. |

#### Syntax Rules

- Only parses requests where `Content-Type` matches `text/plain` (or the configured `type`).
- `req.body` is a string, or `{}` if no body or type mismatch.
- The `limit` option prevents excessively large text bodies.

#### Constraints and Limitations

- **Security:** `req.body` is user-controlled; validate before use. `req.body.trim()` may fail if `req.body` is not a string. 
- The parser does not validate the content; it only decodes it to a string.

### Annotated Code Example

```js
// text-middleware.js
const express = require('express');
const app = express();

// Parse text/plain bodies
app.use(express.text({ type: 'text/plain', limit: '50kb' }));

app.post('/api/log', (req, res) => {
  console.log('Received text:', req.body);
  console.log('Type:', typeof req.body);

  if (typeof req.body !== 'string') {
    return res.status(400).json({ error: 'Expected plain text' });
  }

  res.json({
    message: 'Log received',
    length: req.body.length,
    preview: req.body.substring(0, 50)
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/log` with `Content-Type: text/plain` and body `"Application started successfully"`):**
```
Received text: Application started successfully
Type: string
{"message":"Log received","length":32,"preview":"Application started successfully"}
```

**Expected Output (for `POST /api/log` with `Content-Type: application/json`):**
```
Received text: {}
Type: object
{"error":"Expected plain text"}
```

**Why this output:** The `type: 'text/plain'` option restricts parsing to plain text requests. When the content type doesn't match, `req.body` is `{}` (an object), triggering the type validation error.

### Real-World Cases

- **Log ingestion:** Receiving plain text log lines from applications.
- **Custom protocols:** Text-based APIs that don't use JSON.
- **CSV imports:** Uploading CSV data as plain text.
- **Webhook verification:** Reading raw text for signature verification.

---

## Core Concept 5: `express.raw()`

### Definitions

**Core Definition:** `express.raw()` is a built-in middleware function that parses incoming request payloads into a Buffer and populates `req.body` with the raw binary data.

**Technical Definition:** `express.raw([options])` parses incoming request payloads into a Buffer and is based on `body-parser`. It returns middleware that parses all bodies as a Buffer and only looks at requests where the `Content-Type` header matches the `type` option (default: `application/octet-stream`). A new `body` Buffer containing the parsed data is populated on the request object (i.e., `req.body`), or an empty object (`{}`) if there was no body to parse or the content type was not matched. 

**Beginner-Friendly Explanation:** `express.raw()` is for when a client sends binary data — not text, not JSON, but raw bytes (like an image or a protocol buffer). It gives you the raw bytes as a Buffer so you can process them yourself.

### Purposes

- To parse raw binary request bodies into Buffers.
- To handle binary protocols (e.g., Protocol Buffers, MessagePack).
- To receive file uploads when a full multipart parser is not needed.
- To access the raw body for signature verification (e.g., Stripe webhooks).

### Syntax Rules and Structure

```js
app.use(express.raw([options]));
```

#### Options Breakdown

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `inflate` | Boolean | `true` | Handle compressed bodies. |
| `limit` | Mixed | `"100kb"` | Maximum body size. |
| `type` | Mixed | `"application/octet-stream"` | Media type to parse. |
| `verify` | Function | `undefined` | Custom verification function. |

#### Syntax Rules

- Only parses requests where `Content-Type` matches `application/octet-stream` (or the configured `type`).
- `req.body` is a Buffer, or `{}` if no body or type mismatch.
- The `limit` option prevents excessively large binary bodies.
- Use `type: '*/*'` to parse any content type as raw bytes. 

#### Constraints and Limitations

- The middleware does not parse or validate the binary data; it only provides the raw Buffer.
- Larger limits may be needed for file uploads (default is 100 KB).
- Not suitable for multipart form data (use Multer instead).

### Annotated Code Example

```js
// raw-middleware.js
const express = require('express');
const app = express();

// Parse raw binary bodies with a 5MB limit
app.use(express.raw({
  type: 'application/octet-stream',
  limit: '5mb'
}));

app.post('/api/binary', (req, res) => {
  console.log('Received Buffer:', req.body);
  console.log('Is Buffer:', Buffer.isBuffer(req.body));
  console.log('Length:', req.body.length);

  // Process the binary data (e.g., save to file, parse protocol)
  const hexPreview = req.body.subarray(0, 8).toString('hex');

  res.json({
    received: true,
    bytes: req.body.length,
    hexPreview
  });
});

app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (for `POST /api/binary` with binary data):**
```
Received Buffer: <Buffer 89 50 4e 47 0d 0a 1a 0a ...>
Is Buffer: true
Length: 1024
{"received":true,"bytes":1024,"hexPreview":"89504e470d0a1a0a"}
```

**Why this output:** The `express.raw()` middleware provides the request body as a Buffer. The handler checks `Buffer.isBuffer()` to confirm, reads the length, and extracts the first 8 bytes as a hex preview (which for a PNG file starts with the PNG signature `89504e47`).

### Real-World Cases

- **Webhook signature verification:** Stripe requires the raw body for signature verification.
- **Binary protocols:** Receiving Protocol Buffers or MessagePack data.
- **File uploads:** Simple binary uploads without multipart encoding.
- **Image processing:** Receiving raw image data for processing.

---

## References

- Express.js 5.x — Express Object — https://expressjs.com/en/5x/api/express/
- Express.js 5.x — express.json() — https://expressjs.com/en/5x/api.html#express.json
- Express.js 5.x — express.urlencoded() — https://expressjs.com/en/5x/api.html#express.urlencoded
- Express.js 5.x — express.static() — https://expressjs.com/en/5x/api.html#express.static
- Express.js 4.x — express.text() — https://expressjs.com/en/4x/api.html#express.text
- Express.js 4.x — express.raw() — https://expressjs.com/en/4x/api.html#express.raw
- Express.js — Serving Static Files — https://expressjs.com/en/starter/static-files.html
- Express.js — Using Middleware — https://expressjs.com/en/guide/using-middleware.html
- body-parser — npm — https://www.npmjs.com/package/body-parser
- serve-static — npm — https://www.npmjs.com/package/serve-static
- qs Library — npm — https://www.npmjs.com/package/qs
- Express.js 5.x Migration Guide — https://expressjs.com/en/guide/migrating-5.html
- CoreUI — How to Serve Static Files in Express — https://coreui.io/answers/how-to-serve-static-files-in-express/
- Node.js — `querystring` Module — https://nodejs.org/api/querystring.html
- Stack Overflow — `extended: true` vs `extended: false` — https://stackoverflow.com/questions/29960799/what-does-extended-mean-in-express-4-0
- Stack Overflow — Express.urlencoded extended true vs false — https://stackoverflow.com/questions/78736609
- GeeksforGeeks — Express.raw() Function — https://www.geeksforgeeks.org/express-js-express-raw-function/
- GeeksforGeeks — Express.text() Function — https://www.geeksforgeeks.org/express-js-express-text-function/
- Microsoft Learn — Express.js Static Files — https://learn.microsoft.com/en-us/azure/app-service/tutorial-nodejs-express