# Express.js Headers — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
HTTP headers are key–value pairs exchanged between client and server that carry metadata about the request or response—describing content type, caching policy, security requirements, authentication credentials, and cross-origin permissions.

**Technical Definition**
HTTP headers are defined by RFC 9110 and categorized as **request headers** (sent by the client), **response headers** (sent by the server), and **representation headers** (describing the body, such as `Content-Type` and `Content-Encoding`). In Express.js, response headers are managed through `res.set()` (alias `res.header()`), `res.append()`, `res.get()`, and `res.removeHeader()` . `res.set()` accepts a single field–value pair or an object for multiple headers, and supports arrays for repeatable headers such as `Set-Cookie` . Express also automatically generates `ETag` headers for responses when the `etag` application setting is enabled, and sets `Content-Type` based on the response body . Request headers are accessed via `req.get()` (alias `req.header()`), which performs case-insensitive lookup.

**Beginner-Friendly Explanation**
Headers are the “envelope information” of an HTTP message. The body is the letter; the headers are the labels, postmarks, and handling instructions on the envelope. They tell the client what kind of content is inside, how long it can be cached, whether it must be encrypted, and which other websites are allowed to read it. In Express, you set response headers with `res.set()` before sending the body, and you read request headers with `req.get()`.

---

### Key Characteristics

- **Case-insensitive names** — HTTP header names are case-insensitive; `Content-Type` and `content-type` are the same header.
- **Response headers are set before the body** — Once the response is sent, headers cannot be modified.
- **Express convenience methods** — `res.set()` sets or replaces a header; `res.append()` adds a value to an existing header; `res.get()` reads a header; `res.removeHeader()` removes one .
- **Security headers are not enabled by default** — A bare Express server sends minimal headers and exposes `X-Powered-By: Express`, which reveals the technology stack .
- **Caching is layered** — `Cache-Control` controls freshness, `ETag`/`Last-Modified` enable validation, and `Vary` tells caches which request headers affect the response.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- `helmet` and `cors` packages installed for security and CORS headers.
- Understanding of HTTP request/response cycles and status codes.
- Familiarity with the `(req, res, next)` middleware signature.

---

### Related Programming Areas

- **Security** — Helmet, CSP, HSTS, and permissions policies protect against XSS, clickjacking, and protocol downgrade attacks .
- **Caching** — `Cache-Control`, `ETag`, and `Last-Modified` reduce bandwidth and improve latency .
- **CORS** — `Access-Control-Allow-*` headers control which origins may read responses .
- **Content negotiation** — `Accept`, `Content-Type`, and `Accept-Encoding` determine the response format.
- **Observability** — `X-Request-Id` and custom headers support tracing and debugging.

---

## Core Concept 1: Request Headers

### Definitions

**Core Definition**
Request headers are metadata sent by the client that describe the request, the client's capabilities, and any credentials or preferences.

**Technical Definition**
Request headers are accessed through `req.headers` (a plain object with lowercase keys) or `req.get(field)` (case-insensitive lookup). Common request headers include `Accept` (acceptable response media types), `Accept-Encoding` (acceptable content codings), `Accept-Language` (preferred natural languages), `Authorization` (credentials), `Content-Type` (body media type), `Origin` (CORS origin), `Range` (partial content request), and `If-None-Match` / `If-Modified-Since` (conditional request validators).

**Beginner-Friendly Explanation**
Request headers are what the client tells the server about itself and what it wants. “I can accept JSON,” “I prefer French,” “Here is my Bearer token,” “I only want the first 500 bytes of this file.” Express gives you `req.get()` to read them safely and case-insensitively.

---

### Purposes

- To read authentication tokens and API keys from the `Authorization` header.
- To negotiate response formats through `Accept`, `Accept-Encoding`, and `Accept-Language`.
- To detect the request body media type through `Content-Type`.
- To support conditional requests and partial content through `If-None-Match` and `Range`.

---

### Syntax Rules and Structure

```javascript
req.headers['content-type'];     // lowercase key access
req.get('Content-Type');         // case-insensitive lookup
req.get('Authorization');        // credentials
req.get('Accept');               // content negotiation
```

**Component Breakdown**

| Header | Purpose | Express Access |
|--------|---------|----------------|
| `Accept` | Response media types | `req.get('Accept')` |
| `Authorization` | Credentials | `req.get('Authorization')` |
| `Content-Type` | Body media type | `req.is()` or `req.get('Content-Type')` |
| `Origin` | CORS origin | `req.get('Origin')` |
| `Range` | Partial content | `req.range(size)` |
| `If-None-Match` | Conditional request | `req.fresh` |

#### Syntax Rules

- Header names are case-insensitive in `req.get()`; `req.headers` keys are lowercase.
- `req.get()` returns `undefined` for missing headers unless a default is supplied.
- Treat all request headers as untrusted input; validate before use.

#### Constraints and Limitations

- Repeated headers may appear as arrays or comma-joined strings in `req.headers`; `req.get()` returns the first value.
- The `Referer` header is notoriously misspelled in the HTTP specification; `req.get('Referrer')` and `req.get('Referer')` are interchangeable.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Reading Authorization and Content-Type

```javascript
app.get('/protected', (req, res) => {
  const auth = req.get('Authorization');
  const contentType = req.get('Content-Type', 'none');

  if (!auth || !auth.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Bearer token required' });
  }

  res.json({ token: auth.slice(7), contentType });
});
```

**Expected Output**

```
GET /protected  (Authorization: Bearer abc123, Content-Type: application/json)
→ 200 { "token": "abc123", "contentType": "application/json" }

GET /protected  (no Authorization)
→ 401 { "error": "Bearer token required" }
```

**Why This Output Occurs**

- `req.get('Authorization')` reads the header case-insensitively; the default `'none'` is used when `Content-Type` is absent.

---

### Real-World Cases

- **API authentication** — Reading Bearer tokens from `Authorization`.
- **Content negotiation** — Serving JSON or HTML based on `Accept`.
- **Webhook validation** — Checking `Content-Type` and signature headers.

---

## Core Concept 2: Response Headers

### Definitions

**Core Definition**
Response headers are metadata sent by the server that describe the response body, its caching policy, its security requirements, and other handling instructions.

**Technical Definition**
Express provides `res.set(field, value)` (alias `res.header()`) to set a header, `res.set(object)` to set multiple headers at once, `res.append(field, value)` to add a value to a repeatable header without overwriting existing values, `res.get(field)` to read a previously set header, and `res.removeHeader(field)` to remove one . The `Content-Type` header is set automatically when a response body is sent, but it can be overridden with `res.type()` or `res.set('Content-Type', ...)` . Express automatically generates an `ETag` header when `res.send()` is called, provided the `etag` setting is enabled and no `ETag` has been manually set .

**Beginner-Friendly Explanation**
Response headers are what the server tells the client about the response. “This is JSON,” “You can cache this for 60 seconds,” “This page must never be framed,” “Here is a new cookie.” Express gives you `res.set()` to set them, `res.append()` to add to multi-value headers like `Set-Cookie`, and `res.get()` to inspect what you have set.

---

### Purposes

- To set content type, cache policy, and security headers on every response.
- To append multiple values to repeatable headers such as `Set-Cookie` and `Link`.
- To inspect and modify headers before the response is sent.
- To remove sensitive or unnecessary headers such as `X-Powered-By`.

---

### Syntax Rules and Structure

```javascript
// Single header
res.set('Content-Type', 'application/json');

// Multiple headers
res.set({
  'Cache-Control': 'public, max-age=3600',
  'X-Content-Type-Options': 'nosniff'
});

// Append to a repeatable header
res.append('Set-Cookie', 'session=abc; HttpOnly');

// Read a previously set header
const cacheControl = res.get('Cache-Control');

// Remove a header
res.removeHeader('X-Powered-By');
```

**Component Breakdown**

| Method | Role |
|--------|------|
| `res.set(field, value)` | Sets or replaces a header |
| `res.set(object)` | Sets multiple headers at once |
| `res.append(field, value)` | Adds a value to a repeatable header |
| `res.get(field)` | Reads a header set on the response |
| `res.removeHeader(field)` | Removes a header |
| `res.type(type)` | Sets `Content-Type` from a MIME type or extension |

#### Syntax Rules

- `res.set()` **replaces** an existing header; `res.append()` **adds** to it .
- Calling `res.set()` after `res.append()` resets the previously appended values .
- Array values in `res.set()` set multiple headers, useful for `Set-Cookie` .
- Response headers must be set **before** the body is sent; after `res.send()`, headers are immutable.

#### Constraints and Limitations

- `res.append()` is available since Express 4.11.0.
- Setting a header after the response has been sent throws `ERR_HTTP_HEADERS_SENT`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Setting Multiple Response Headers

```javascript
app.get('/data', (req, res) => {
  res.set({
    'Content-Type': 'application/json',
    'Cache-Control': 'public, max-age=300',
    'X-Content-Type-Options': 'nosniff'
  });

  res.json({ data: 'value' });
});
```

**Expected Output**

```
GET /data
→ 200 { "data": "value" }
Content-Type: application/json
Cache-Control: public, max-age=300
X-Content-Type-Options: nosniff
```

**Why This Output Occurs**

- `res.set(object)` sets all three headers in a single call before `res.json()` sends the body.

---

#### Example 2: Appending Multiple `Set-Cookie` Headers

```javascript
app.get('/login', (req, res) => {
  res.append('Set-Cookie', 'session=abc123; HttpOnly; Secure; SameSite=Strict');
  res.append('Set-Cookie', 'theme=dark; Path=/; Max-Age=31536000');
  res.json({ loggedIn: true });
});
```

**Expected Output**

```
GET /login
→ 200 { "loggedIn": true }
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
Set-Cookie: theme=dark; Path=/; Max-Age=31536000
```

**Why This Output Occurs**

- `res.append()` adds each cookie as a separate header value; `res.set()` would have replaced the first cookie with the second.

---

### Real-World Cases

- **Security headers** — Setting `X-Content-Type-Options`, `X-Frame-Options`, and `Content-Security-Policy` on every response.
- **Cache headers** — Setting `Cache-Control` and `ETag` for static and API responses.
- **Cookie management** — Appending multiple `Set-Cookie` headers for session and preference cookies.

---

## Core Concept 3: Custom Headers

### Definitions

**Core Definition**
Custom headers are application-defined headers, prefixed with `X-` by convention (or unprefixed for registered headers), that carry application-specific metadata such as request IDs, API versions, or feature flags.

**Technical Definition**
Custom headers are set and read through the same `res.set()` and `req.get()` methods used for standard headers. The `X-` prefix is a historical convention for non-standard headers; since RFC 6648, the prefix is officially discouraged for new headers, but it remains widely used. Common custom headers in Express applications include `X-Request-Id` for tracing, `X-API-Version` for versioning, and `X-RateLimit-*` for rate-limit metadata.

**Beginner-Friendly Explanation**
Custom headers are headers you invent for your own application. If you want every response to carry a unique request ID for debugging, you add an `X-Request-Id` header. If you want clients to know which API version they are using, you add `X-API-Version`. They work exactly like standard headers—you just define the name and meaning yourself.

---

### Purposes

- To propagate correlation IDs for distributed tracing.
- To expose API version and deprecation metadata.
- To carry application-specific feature flags or tenant identifiers.
- To provide rate-limit information (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`).

---

### Syntax Rules and Structure

```javascript
// Set a custom response header
res.set('X-Request-Id', req.id);
res.set('X-API-Version', '2.0');

// Read a custom request header
const requestId = req.get('X-Request-Id');
```

**Component Breakdown**

| Header | Purpose |
|--------|---------|
| `X-Request-Id` | Correlation ID for tracing |
| `X-API-Version` | API version metadata |
| `X-RateLimit-*` | Rate-limit information |
| `X-Custom-Header` | Any application-specific value |

#### Syntax Rules

- Custom header names must be valid HTTP token characters; avoid spaces and non-ASCII characters.
- Use the `X-` prefix only for historical compatibility; RFC 6648 discourages it for new headers.
- Document custom headers in the API contract so clients know what to expect.

#### Constraints and Limitations

- Custom headers are visible to any intermediary; do not put secrets in them.
- Some proxies strip unknown headers; verify behavior in your deployment environment.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Request ID and API Version Headers

```javascript
const crypto = require('crypto');

app.use((req, res, next) => {
  const requestId = req.get('X-Request-Id') || crypto.randomUUID();
  req.id = requestId;
  res.set('X-Request-Id', requestId);
  res.set('X-API-Version', '2.0');
  next();
});

app.get('/data', (req, res) => {
  res.json({ requestId: req.id });
});
```

**Expected Output**

```
GET /data  (no X-Request-Id)
→ 200 { "requestId": "a1b2c3..." }
X-Request-Id: a1b2c3...
X-API-Version: 2.0

GET /data  (X-Request-Id: upstream-123)
→ 200 { "requestId": "upstream-123" }
X-Request-Id: upstream-123
X-API-Version: 2.0
```

**Why This Output Occurs**

- The middleware propagates an incoming request ID or generates a new one; it sets the response header and attaches the ID to `req` for downstream handlers.

---

### Real-World Cases

- **Distributed tracing** — Propagating `X-Request-Id` across microservices.
- **API versioning** — Exposing `X-API-Version` for client compatibility checks.
- **Rate limiting** — Returning `X-RateLimit-Remaining` and `X-RateLimit-Reset` to clients.

---

## Core Concept 4: Security Headers

### Definitions

**Core Definition**
Security headers are response headers that instruct the browser to enable specific security protections—forcing HTTPS, blocking MIME sniffing, preventing clickjacking, restricting content sources, controlling referrer information, and limiting browser feature access.

**Technical Definition**
Express does not enable security headers by default; a bare Express server sends minimal headers and exposes `X-Powered-By: Express` . Helmet.js is the standard middleware for setting security headers; it configures twelve headers by default, including `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, and `Permissions-Policy` . The OWASP Top 10 classifies security misconfiguration (A05) as a top-five risk, and missing security headers are one of the most common and easily fixed vulnerabilities .

**Beginner-Friendly Explanation**
Security headers are the browser's rulebook for your page. “Only load scripts from these sources.” “Never render this page in a frame.” “Always use HTTPS.” “Don't send the full URL to other sites.” “Don't let anyone use the camera without permission.” Helmet.js sets sensible defaults for all of these in one line of code.

---

### Purposes

- To mitigate XSS through Content-Security-Policy.
- To prevent clickjacking through `X-Frame-Options` or CSP `frame-ancestors`.
- To block MIME sniffing through `X-Content-Type-Options: nosniff`.
- To enforce HTTPS through `Strict-Transport-Security`.
- To control referrer leakage through `Referrer-Policy`.
- To restrict browser features through `Permissions-Policy`.

---

### Sub-Concept: `Strict-Transport-Security` (HSTS)

#### Definitions

**Core Definition**
`Strict-Transport-Security` tells browsers to only access the site over HTTPS for a specified duration, and optionally includes subdomains and preload lists.

**Technical Definition**
The header value is `max-age=<seconds>; includeSubDomains; preload`. `max-age` specifies how long the browser should remember to use HTTPS only. `includeSubDomains` applies the policy to all subdomains. `preload` signals eligibility for browser preload lists. Helmet sets `max-age=15552000; includeSubDomains` by default; a common production configuration is `max-age=31536000; includeSubDomains; preload` (one year) .

**Beginner-Friendly Explanation**
HSTS tells the browser: “For the next year, never connect to this site over HTTP—always use HTTPS, even if the user types `http://`.” This prevents downgrade attacks and cookie theft on insecure connections.

---

#### Purposes

- To prevent protocol downgrade attacks and cookie hijacking.
- To enforce HTTPS for all future requests to the domain and its subdomains.
- To signal eligibility for browser HSTS preload lists.

---

#### Syntax Rules and Structure

```javascript
// Helmet configuration
app.use(helmet.hsts({
  maxAge: 31536000,        // 1 year
  includeSubDomains: true,
  preload: true
}));
```

**Component Breakdown**

| Directive | Effect |
|-----------|--------|
| `max-age` | Duration in seconds the browser remembers HTTPS-only |
| `includeSubDomains` | Applies to all subdomains |
| `preload` | Signals eligibility for browser preload lists |

#### Constraints and Limitations

- HSTS is ignored over HTTP; it only takes effect after the first HTTPS response.
- `preload` requires submission to the HSTS preload list and is difficult to reverse.
- A misconfigured `max-age` can lock users out if HTTPS breaks on subdomains.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: HSTS with Helmet

```javascript
const helmet = require('helmet');

app.use(helmet.hsts({
  maxAge: 31536000,
  includeSubDomains: true,
  preload: true
}));
```

**Expected Output**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

**Why This Output Occurs**

- Helmet serializes the configuration into the standard HSTS header value.

---

#### Real-World Cases

- **Financial and healthcare sites** — Mandatory HTTPS enforcement.
- **SaaS platforms** — Protecting all subdomains from downgrade attacks.
- **Public-facing applications** — Preload submission for maximum protection.

---

### Sub-Concept: `X-Content-Type-Options`

#### Definitions

**Core Definition**
`X-Content-Type-Options: nosniff` prevents browsers from MIME-sniffing a response away from the declared `Content-Type`, blocking attacks that exploit content-type confusion.

**Technical Definition**
When `nosniff` is present, browsers strictly honor the `Content-Type` header. Scripts and stylesheets with mismatched MIME types are blocked. Helmet enables `noSniff: true` by default .

**Beginner-Friendly Explanation**
Some browsers try to guess what a file is, even if the server says it is something else. An attacker can upload a malicious script disguised as an image, and the browser might “sniff” it as JavaScript and execute it. `nosniff` tells the browser: “Trust the declared type—don't guess.”

---

#### Purposes

- To prevent MIME-sniffing attacks that execute disguised scripts.
- To enforce correct `Content-Type` handling for scripts, styles, and documents.
- To block drive-by downloads of malicious content.

---

#### Syntax Rules and Structure

```javascript
app.use(helmet.noSniff());
// or manually:
app.use((req, res, next) => {
  res.set('X-Content-Type-Options', 'nosniff');
  next();
});
```

**Component Breakdown**

| Header | Value | Effect |
|--------|-------|--------|
| `X-Content-Type-Options` | `nosniff` | Browsers honor declared `Content-Type` |

#### Constraints and Limitations

- `nosniff` can break legacy content that relies on MIME sniffing; test before deploying.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Enabling `nosniff`

```javascript
app.use(helmet.noSniff());
```

**Expected Output**

```
X-Content-Type-Options: nosniff
```

**Why This Output Occurs**

- Helmet sets the header to `nosniff` on every response.

---

#### Real-World Cases

- **File upload platforms** — Preventing uploaded files from being sniffed and executed.
- **APIs serving user content** — Ensuring JSON is never interpreted as HTML.

---

### Sub-Concept: `X-Frame-Options`

#### Definitions

**Core Definition**
`X-Frame-Options` controls whether a page can be embedded in a `<frame>`, `<iframe>`, or `<object>`, protecting against clickjacking attacks.

**Technical Definition**
Valid values are `DENY` (never frame), `SAMEORIGIN` (frame only from the same origin), and `ALLOW-FROM <uri>` (deprecated and inconsistently supported). Helmet’s `frameguard` sets `SAMEORIGIN` by default; `action: "deny"` sets `DENY` . CSP `frame-ancestors` is the modern replacement and takes precedence in browsers that support it .

**Beginner-Friendly Explanation**
Clickjacking is when an attacker puts your site inside an invisible iframe on their page and tricks users into clicking buttons they didn't intend to click—like “Delete account” or “Transfer money.” `X-Frame-Options: DENY` tells the browser: “Never let this page be framed.”

---

#### Purposes

- To prevent clickjacking attacks.
- To control which origins, if any, may embed the page.
- To complement CSP `frame-ancestors` for defense in depth.

---

#### Syntax Rules and Structure

```javascript
app.use(helmet.frameguard({ action: 'deny' }));
// or:
app.use((req, res, next) => {
  res.set('X-Frame-Options', 'DENY');
  next();
});
```

**Component Breakdown**

| Value | Effect |
|-------|--------|
| `DENY` | Never allow framing |
| `SAMEORIGIN` | Allow framing from the same origin only |

#### Constraints and Limitations

- `ALLOW-FROM` is deprecated and not supported in modern browsers; use CSP `frame-ancestors` instead.
- `X-Frame-Options` is ignored by browsers that support CSP `frame-ancestors`; set both for compatibility.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Denying Framing

```javascript
app.use(helmet.frameguard({ action: 'deny' }));
```

**Expected Output**

```
X-Frame-Options: DENY
```

**Why This Output Occurs**

- Helmet sets `DENY` when `action: 'deny'` is configured.

---

#### Real-World Cases

- **Banking applications** — Preventing clickjacking of transfer forms.
- **Admin panels** — Blocking embedding in malicious sites.
- **Login pages** — Preventing credential-harvesting overlays.

---

### Sub-Concept: `Content-Security-Policy` (CSP)

#### Definitions

**Core Definition**
`Content-Security-Policy` is a powerful allow-list header that restricts which sources of scripts, styles, images, fonts, and other resources a page may load, mitigating XSS and data-injection attacks.

**Technical Definition**
CSP is expressed as a semicolon-separated list of directives, each with a space-separated list of source expressions. Common directives include `default-src` (fallback), `script-src`, `style-src`, `img-src`, `font-src`, `connect-src` (fetch/XHR/WebSocket), `frame-src`, `frame-ancestors`, `object-src`, `base-uri`, and `form-action`. Helmet sets a default CSP that can be customized with the `directives` option . CSP can be deployed in **report-only** mode (`Content-Security-Policy-Report-Only`) to collect violations before enforcement.

**Beginner-Friendly Explanation**
CSP is the strictest and most valuable security header. It tells the browser: “Only run scripts from these domains; only load styles from these sources; never load plugins; never frame this page.” It is the primary defense against XSS because even if an attacker injects a script tag, the browser refuses to execute it unless the source is allowed.

---

#### Purposes

- To mitigate XSS by restricting script and style sources.
- To prevent data exfiltration to unauthorized domains.
- To block mixed content and insecure resource loading.
- To control framing through `frame-ancestors`.

---

#### Syntax Rules and Structure

```javascript
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'strict-dynamic'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "data:", "https:"],
    fontSrc: ["'self'", "https://fonts.gstatic.com"],
    connectSrc: ["'self'", "https://api.example.com"],
    frameSrc: ["'none'"],
    objectSrc: ["'none'"],
    baseUri: ["'self'"],
    formAction: ["'self'"],
    frameAncestors: ["'none'"],
    upgradeInsecureRequests: []
  }
}));
```

**Component Breakdown**

| Directive | Controls |
|-----------|----------|
| `default-src` | Fallback for all resource types |
| `script-src` | JavaScript sources |
| `style-src` | CSS sources |
| `img-src` | Image sources |
| `font-src` | Font sources |
| `connect-src` | fetch/XHR/WebSocket destinations |
| `frame-ancestors` | Who may frame this page |
| `object-src` | Plugins (`<object>`, `<embed>`) |
| `base-uri` | `<base>` tag URLs |
| `form-action` | Form submission targets |

#### Syntax Rules

- `'self'` allows the page's own origin; `'none'` allows nothing; `'unsafe-inline'` allows inline scripts/styles (avoid for scripts).
- `'strict-dynamic'` trusts scripts loaded by already-trusted scripts, enabling modern bundler workflows.
- Use a per-request **nonce** for inline scripts: `script-src 'nonce-<random>' 'strict-dynamic'`.
- Deploy in **report-only** mode first to discover violations without breaking functionality.

#### Constraints and Limitations

- A strict CSP can break legitimate functionality (analytics, inline styles, third-party widgets); tune directives carefully.
- CSP is not a substitute for input validation and output encoding.
- `frame-ancestors` supersedes `X-Frame-Options` in modern browsers.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Strict CSP with Helmet

```javascript
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'"],
    imgSrc: ["'self'"],
    objectSrc: ["'none'"],
    frameAncestors: ["'none'"]
  }
}));
```

**Expected Output**

```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self'; object-src 'none'; frame-ancestors 'none'
```

**Why This Output Occurs**

- Helmet serializes the directive object into the CSP header value.

---

#### Real-World Cases

- **All production web applications** — CSP is the primary XSS defense.
- **Financial applications** — Strict `script-src` and `frame-ancestors` policies.
- **Content platforms** — CSP with nonces for user-generated inline content.

---

### Sub-Concept: `Referrer-Policy`

#### Definitions

**Core Definition**
`Referrer-Policy` controls how much URL information is sent in the `Referer` header when navigating from your page to another origin.

**Technical Definition**
Valid values include `no-referrer` (never send), `same-origin` (send only for same-origin requests), `strict-origin-when-cross-origin` (send full URL for same-origin, only origin for cross-origin, nothing on HTTPS→HTTP downgrade), and `unsafe-url` (always send full URL). Helmet sets `no-referrer` by default; `strict-origin-when-cross-origin` is a common production choice that balances analytics with privacy .

**Beginner-Friendly Explanation**
When a user clicks a link from your site to another site, the browser sends a `Referer` header that tells the other site where the user came from. This can leak sensitive URL paths—like password reset tokens or internal IDs. `Referrer-Policy` controls how much of that URL is shared.

---

#### Purposes

- To prevent leakage of sensitive URL paths to third parties.
- To balance analytics needs against privacy.
- To comply with privacy regulations.

---

#### Syntax Rules and Structure

```javascript
app.use(helmet.referrerPolicy({ policy: 'strict-origin-when-cross-origin' }));
```

**Component Breakdown**

| Value | Effect |
|-------|--------|
| `no-referrer` | Never send the `Referer` header |
| `same-origin` | Send full URL for same-origin only |
| `strict-origin-when-cross-origin` | Full URL same-origin; origin only cross-origin; nothing on downgrade |
| `unsafe-url` | Always send full URL (avoid) |

#### Constraints and Limitations

- `strict-origin-when-cross-origin` is the modern browser default, but explicit headers ensure consistency.
- `no-referrer` breaks some analytics and affiliate tracking; choose deliberately.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Referrer Policy Configuration

```javascript
app.use(helmet.referrerPolicy({ policy: 'strict-origin-when-cross-origin' }));
```

**Expected Output**

```
Referrer-Policy: strict-origin-when-cross-origin
```

**Why This Output Occurs**

- Helmet sets the header to the configured policy value.

---

#### Real-World Cases

- **Authentication flows** — Preventing reset tokens from leaking in `Referer`.
- **SaaS dashboards** — Avoiding leakage of internal IDs to third-party analytics.
- **Public content sites** — Balancing analytics with privacy.

---

### Sub-Concept: `Permissions-Policy`

#### Definitions

**Core Definition**
`Permissions-Policy` (formerly Feature-Policy) controls which browser features and APIs—such as camera, microphone, geolocation, and payment—are allowed on the page and in embedded iframes.

**Technical Definition**
The header is a comma-separated list of feature–allowlist pairs: `feature=(allowlist)`. An empty allowlist `()` disables the feature entirely. Common features include `accelerometer`, `camera`, `geolocation`, `microphone`, `payment`, `usb`, `fullscreen`, and `autoplay`. Helmet’s `permittedCrossDomainPolicies` is a different header (`X-Permitted-Cross-Domain-Policies`); `Permissions-Policy` is typically set manually or through a dedicated middleware .

**Beginner-Friendly Explanation**
`Permissions-Policy` lets you say: “This page may not use the camera, the microphone, or geolocation.” If an attacker injects a script, it still cannot access those features because the policy disables them. It follows the principle of least privilege for browser APIs.

---

#### Purposes

- To disable unused browser features and reduce the attack surface.
- To prevent unauthorized access to camera, microphone, and location.
- To control which iframes may use specific features.
- To enforce least-privilege for browser APIs.

---

#### Syntax Rules and Structure

```javascript
app.use((req, res, next) => {
  res.set('Permissions-Policy', 'geolocation=(), microphone=(), camera=(), payment=(), usb=()');
  next();
});
```

**Component Breakdown**

| Directive | Effect |
|-----------|--------|
| `geolocation=()` | Disable geolocation entirely |
| `camera=()` | Disable camera access |
| `microphone=()` | Disable microphone access |
| `payment=()` | Disable Payment Request API |
| `usb=()` | Disable WebUSB |

#### Syntax Rules

- An empty allowlist `()` disables the feature for all origins.
- `self` allows the current origin; `*` allows all origins (use sparingly).
- Feature names are case-sensitive and follow the Permissions Policy specification.

#### Constraints and Limitations

- Browser support varies; some features may be ignored in older browsers.
- Disabling features may break legitimate functionality; audit feature usage before enforcing.

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Disabling Sensitive Features

```javascript
app.use((req, res, next) => {
  res.set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  next();
});
```

**Expected Output**

```
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Why This Output Occurs**

- The header explicitly disables the three sensitive features for all origins.

---

#### Real-World Cases

- **Content sites** — Disabling camera and microphone where they are not needed.
- **Financial applications** — Disabling geolocation and payment APIs unless required.
- **Embedded iframes** — Restricting third-party feature access.

---

## Core Concept 5: CORS Headers

### Definitions

**Core Definition**
Cross-Origin Resource Sharing (CORS) headers are response headers that tell the browser which other origins are allowed to read the response to a cross-origin request.

**Technical Definition**
CORS is enforced by the browser, not the server. The `cors` npm package is the standard Express middleware; it sets `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Allow-Credentials`, `Access-Control-Max-Age`, and `Access-Control-Expose-Headers`. For requests that could have side effects—non-simple methods, JSON `Content-Type`, or custom headers—the browser first sends an `OPTIONS` **preflight** request. The server must respond to the preflight with the allowed methods and headers, or the real request never happens . The wildcard `*` for `Access-Control-Allow-Origin` is **forbidden** when `credentials: true`; use an explicit allowlist instead .

**Beginner-Friendly Explanation**
CORS is the browser's security rule: JavaScript on `app.example.com` cannot read responses from `api.example.com` unless the API explicitly says “this origin is allowed.” The `cors` middleware adds the headers that grant that permission. The most dangerous mistake is reflecting any origin while allowing credentials—that lets any malicious site make authenticated requests to your API and read the responses .

---

### Purposes

- To allow legitimate cross-origin frontends to read API responses.
- To restrict access to an explicit allowlist of trusted origins.
- To handle preflight requests correctly for non-simple methods and headers.
- To support credentialed requests (cookies, `Authorization`) safely.

---

### Syntax Rules and Structure

```javascript
const cors = require('cors');

const allowedOrigins = new Set([
  'https://app.example.com',
  'https://admin.example.com'
]);

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.has(origin)) {
      return callback(null, true);
    }
    callback(new Error('Origin not allowed by CORS'));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600
}));
```

**Component Breakdown**

| Header | Purpose |
|--------|---------|
| `Access-Control-Allow-Origin` | Which origin(s) may read the response |
| `Access-Control-Allow-Methods` | Methods allowed in preflight |
| `Access-Control-Allow-Headers` | Headers allowed in preflight |
| `Access-Control-Allow-Credentials` | Whether cookies/credentials are allowed |
| `Access-Control-Max-Age` | How long preflight results are cached |
| `Access-Control-Expose-Headers` | Headers readable by the browser |

#### Syntax Rules

- Use an **exact-match allowlist** (`Set`), not a regex that can match malicious domains.
- `credentials: true` **cannot** be combined with `origin: '*'`; browsers reject it.
- Scope `methods` and `allowedHeaders` to what the API actually uses.
- Use a modest `maxAge` (e.g., 600 seconds) to avoid pinning stale policy in browsers .
- For public, credential-free APIs, `origin: '*'` with `credentials: false` is acceptable.

#### Constraints and Limitations

- CORS does **not** protect server-to-server calls; `curl` and other HTTP clients ignore it.
- A blocked CORS response may still have executed server-side; CORS is not an authorization mechanism .
- Reflecting the request's `Origin` header without validation is a common and dangerous anti-pattern .

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Safe CORS Allowlist

```javascript
const cors = require('cors');

const allowlist = new Set([
  'https://app.example.com',
  'https://admin.example.com'
]);

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowlist.has(origin)) {
      return callback(null, true);
    }
    callback(new Error('Origin not allowed'));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600
}));

app.get('/api/data', (req, res) => {
  res.json({ data: 'value' });
});
```

**Expected Output**

```
GET /api/data  (Origin: https://app.example.com)
→ 200 { "data": "value" }
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true

GET /api/data  (Origin: https://evil.example.com)
→ CORS error (callback receives error)
```

**Why This Output Occurs**

- The origin function checks the request's `Origin` against the exact-match allowlist; allowed origins receive the CORS headers, disallowed origins are rejected.

---

### Real-World Cases

- **SPA + API separation** — Frontend and backend on different domains.
- **Multi-tenant SaaS** — Dynamic allowlists loaded from tenant configuration.
- **Public APIs** — `origin: '*'` with `credentials: false` for credential-free access.

---

## Core Concept 6: Cache Headers

### Definitions

**Core Definition**
Cache headers control whether, how long, and under what conditions a response may be stored and reused by browsers, proxies, and CDNs.

**Technical Definition**
HTTP caching has three layers: **freshness** (`Cache-Control: max-age`), **validation** (`ETag` with `If-None-Match` or `Last-Modified` with `If-Modified-Since`), and **invalidation** (explicit purges, short TTLs, or `stale-while-revalidate`). `Cache-Control` is the primary directive header; common directives include `public`, `private`, `no-cache`, `no-store`, `max-age`, `s-maxage`, `must-revalidate`, `stale-while-revalidate`, and `immutable`. Express automatically generates `ETag` headers when `res.send()` is called, unless an `ETag` is set manually or the `etag` setting is disabled . Prefer `ETag` over `Last-Modified` for APIs because `Last-Modified` has only one-second granularity and is susceptible to clock skew .

**Beginner-Friendly Explanation**
Cache headers tell the client and any intermediary: “You may keep this response for 60 seconds,” “You must check with me before using it again,” or “Never store this at all.” `ETag` is a fingerprint; the client sends it back, and if nothing has changed, the server responds `304 Not Modified` with no body. This saves bandwidth and speeds up repeat requests.

---

### Purposes

- To reduce bandwidth and latency by allowing clients to reuse cached responses.
- To enable conditional requests with `304 Not Modified` responses.
- To prevent caching of sensitive data with `no-store`.
- To give CDNs longer TTLs than browsers with `s-maxage`.

---

### Syntax Rules and Structure

```javascript
// Public, cacheable for 5 minutes in browsers and 1 hour in CDNs
res.set('Cache-Control', 'public, max-age=300, s-maxage=3600, stale-while-revalidate=86400');

// Private, browser-only cache for 60 seconds
res.set('Cache-Control', 'private, max-age=60');

// Never cache (sensitive data)
res.set('Cache-Control', 'no-store');

// ETag for conditional requests
res.set('ETag', '"v1"');
if (req.fresh) return res.status(304).end();
```

**Component Breakdown**

| Directive | Scope | Meaning |
|-----------|-------|---------|
| `public` | Shared + private caches | Response is identical for all users |
| `private` | Browser only | User-specific response; CDNs must not cache |
| `no-cache` | All caches | May store but must revalidate before use |
| `no-store` | All caches | Never store |
| `max-age=N` | All caches | Fresh for N seconds |
| `s-maxage=N` | Shared caches only | Overrides `max-age` for CDNs |
| `must-revalidate` | All caches | Do not serve stale after expiry |
| `stale-while-revalidate=N` | All caches | Serve stale for N seconds while revalidating |
| `immutable` | All caches | Body will never change for this URL |

#### Syntax Rules

- **Always set `Cache-Control` explicitly**; omitting it forces heuristic caching, which is unpredictable .
- Use `no-store` for financial data, PII, and authenticated responses.
- `no-cache` and `no-store` are different: `no-cache` stores but validates; `no-store` prohibits storage .
- Use `private` for user-specific data to prevent CDN caching.
- Use `immutable` only for versioned URLs with content hashes.
- Send both `ETag` and `Last-Modified`; prefer `ETag` for APIs .

#### Constraints and Limitations

- Contradictory directives (e.g., `public, no-store`) are nonsensical and should be avoided.
- `max-age` does not invalidate; stale content can persist until the TTL expires or the cache is purged.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Conditional GET with ETag and Cache-Control

```javascript
app.get('/article', (req, res) => {
  const article = { id: 1, title: 'Hello' };
  const etag = '"v1"';

  res.set('ETag', etag);
  res.set('Cache-Control', 'private, max-age=60');

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
Cache-Control: private, max-age=60

GET /article  (If-None-Match: "v1")
→ 304 Not Modified
```

**Why This Output Occurs**

- The client caches the response and its ETag; after 60 seconds it revalidates with `If-None-Match`; the server returns `304` because the ETag still matches.

---

#### Example 2: Static Assets with Immutable Caching

```javascript
app.use('/static', express.static('dist', {
  maxAge: '1y',
  immutable: true,
  setHeaders: (res, filePath) => {
    if (filePath.endsWith('.html')) {
      res.setHeader('Cache-Control', 'no-cache');
    }
  }
}));
```

**Expected Output**

```
GET /static/app.abc123.js
→ Cache-Control: public, max-age=31536000, immutable

GET /static/index.html
→ Cache-Control: no-cache
```

**Why This Output Occurs**

- Content-hashed assets receive long-term immutable caching; HTML files are revalidated because their names do not change.

---

### Real-World Cases

- **Static assets** — Hashed bundles cached for a year with `immutable`.
- **API responses** — `private, max-age=60` with ETag validation.
- **Sensitive data** — `no-store` for balances, transactions, and PII.

---

## Core Concept 7: Header Best Practices

### Definitions

**Core Definition**
Header best practices are the design and configuration patterns that keep headers secure, correct, and performant across an Express application.

**Technical Definition**
Best practices include: setting security headers on every response via Helmet; using an explicit CORS allowlist with credentials; always setting `Cache-Control` explicitly; removing `X-Powered-By`; using `Vary` on headers that affect the response (e.g., `Accept-Encoding`, `Origin`); avoiding secrets in custom headers; and testing headers with tools like `curl -I` or security header scanners.

**Beginner-Friendly Explanation**
Best practices are the habits that keep headers from becoming a security hole or a debugging nightmare. Use Helmet for security headers. Use an exact allowlist for CORS. Always set caching explicitly. Remove the `X-Powered-By` header so attackers don't know you use Express. And never put secrets in headers.

---

### Purposes

- To prevent security misconfiguration and information leakage.
- To ensure caching behaves predictably.
- To avoid CORS misconfigurations that enable cross-site data theft.
- To maintain a consistent, auditable header policy.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Header Pipeline

```javascript
const helmet = require('helmet');
const cors = require('cors');

// 1. Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      frameAncestors: ["'none'"]
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

// 2. Permissions-Policy
app.use((req, res, next) => {
  res.set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  next();
});

// 3. CORS allowlist
app.use(cors({ origin: allowlist, credentials: true }));

// 4. Remove X-Powered-By
app.disable('x-powered-by');

// 5. Explicit cache headers per route
app.get('/api/data', (req, res) => {
  res.set('Cache-Control', 'private, max-age=60');
  res.json({ data: 'value' });
});
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| Security headers | Helmet with CSP, HSTS, Referrer-Policy |
| Permissions-Policy | Disable unused features |
| CORS allowlist | Exact-match `Set`, credentials scoped |
| Remove fingerprinting | `app.disable('x-powered-by')` |
| Explicit caching | `Cache-Control` on every route |
| `Vary` header | Include when response depends on headers |

#### Syntax Rules

- **Use Helmet** for security headers; do not set them manually unless you need full control .
- **Use an exact-match CORS allowlist** with `credentials: true`; never reflect arbitrary origins .
- **Always set `Cache-Control`**; do not rely on heuristic caching .
- **Remove `X-Powered-By`** to avoid revealing the framework .
- **Set `Vary`** on headers that affect the response, such as `Accept-Encoding` and `Origin`.
- **Never put secrets in headers**; they are visible to intermediaries.

#### Constraints and Limitations

- Overly strict CSP can break legitimate functionality; deploy in report-only mode first.
- CORS with credentials requires an explicit origin; wildcard is forbidden.
- `immutable` caching requires versioned URLs; otherwise stale content persists.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Hardened Header Configuration

```javascript
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const app = express();

app.disable('x-powered-by');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", "data:"],
      objectSrc: ["'none'"],
      frameAncestors: ["'none'"]
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

app.use((req, res, next) => {
  res.set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');
  next();
});

const allowlist = new Set(['https://app.example.com']);
app.use(cors({
  origin: (origin, cb) => cb(null, !origin || allowlist.has(origin)),
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600
}));

app.get('/api/data', (req, res) => {
  res.set('Cache-Control', 'private, max-age=60');
  res.json({ data: 'value' });
});

app.listen(3000);
```

**Expected Output**

```
GET /api/data  (Origin: https://app.example.com)
→ 200 { "data": "value" }

Response headers:
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none'
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Cache-Control: private, max-age=60
(No X-Powered-By header)
```

**Why This Output Occurs**

- Helmet sets the security headers; the custom middleware sets `Permissions-Policy`; the CORS middleware sets the access-control headers for the allowed origin; the route sets an explicit cache policy; and `app.disable('x-powered-by')` removes the framework fingerprint.

---

### Real-World Cases

- **Production Express applications** — Helmet, CORS allowlist, explicit caching, and `X-Powered-By` removal.
- **Security audits** — Header scanners verify CSP, HSTS, and `nosniff` are present.
- **High-traffic APIs** — Cache-Control and ETag reduce bandwidth and latency.

---

## Header Comparison Table

| Header | Type | Purpose | Express / Middleware |
|--------|------|---------|---------------------|
| `Authorization` | Request | Credentials | `req.get('Authorization')` |
| `Content-Type` | Request/Response | Body media type | `req.is()`, `res.type()` |
| `Accept` | Request | Response formats | `req.accepts()` |
| `Cache-Control` | Response | Freshness and storage policy | `res.set()` or `express.static()` |
| `ETag` | Response | Validation fingerprint | Auto-generated by `res.send()` |
| `Last-Modified` | Response | Timestamp validation | `express.static()` default |
| `Strict-Transport-Security` | Response | Force HTTPS | Helmet `hsts` |
| `X-Content-Type-Options` | Response | Block MIME sniffing | Helmet `noSniff` |
| `X-Frame-Options` | Response | Prevent clickjacking | Helmet `frameguard` |
| `Content-Security-Policy` | Response | Restrict resource sources | Helmet `contentSecurityPolicy` |
| `Referrer-Policy` | Response | Control referrer leakage | Helmet `referrerPolicy` |
| `Permissions-Policy` | Response | Restrict browser features | Manual or dedicated middleware |
| `Access-Control-Allow-Origin` | Response | CORS origin allowlist | `cors` middleware |
| `Access-Control-Allow-Credentials` | Response | Allow cookies in CORS | `cors({ credentials: true })` |
| `Vary` | Response | Headers that affect response | `res.set('Vary', ...)` |

---

## References

- Express Response Headers · Compile-N-Run — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/4-express-response-handling/3-express-response-headers.mdx
- Express Response Object · Express.js 5.x — https://expressjs.com/en/5x/api/response/
- Headers and Caching · Express.js DeepWiki — https://deepwiki.com/expressjs/express/5.5-headers-and-caching
- Helmet.js README — https://github.com/helmetjs/helmet
- Helmet default headers (npm) — https://www.npmjs.com/package/helmet
- HTTP Security Headers reference (F5 Framework) — https://github.com/Fujigo-Software/f5-framework-claude/blob/main/plugins/f5-core/skills/security/infrastructure/headers-security.md
- Node.js/Express Security Headers configuration — https://github.com/aj-geddes/useful-ai-prompts/blob/main/skills/security-headers-configuration/references/nodejsexpress-security-headers.md
- Security Headers in Node.js (Shattered, 2026) — https://shattered.io/fr/security-headers-nodejs/
- CORS npm Package Security Review — https://safeguard.sh/resources/blog/cors-npm
- Why We Use CORS in Node.js — https://safeguard.sh/resources/blog/cors-in-nodejs-explained
- CORS 跨域不通？最小配置 — https://cloud.tencent.com.cn/developer/article/2590630
- API Caching with ETag and Cache-Control — https://apidog.com/blog/api-caching-etag-cache-control/
- Caching and Performance · fractional-cto — https://github.com/oborchers/fractional-cto/blob/main/api-design-principles/skills/caching-and-performance/SKILL.md
- MDN Cache-Control — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
- MDN Referrer-Policy — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
- Permissions-Policy feature list (Chromium) — https://chromium.googlesource.com/chromium/src/third_party/blink/renderer/core/permissions_policy/feature_policy.dict
- OWASP HTTP Headers Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
- RFC 9110: HTTP Semantics — https://httpwg.org/specs/rfc9110.html