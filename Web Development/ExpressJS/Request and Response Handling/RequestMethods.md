# Express.js Request Methods — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Request methods are functions attached to the Express.js `req` object that inspect, parse, and negotiate data from the incoming HTTP request—headers, content types, accepted formats, and byte ranges—giving route handlers structured access to client intent.

**Technical Definition**
Express extends Node.js's `http.IncomingMessage.prototype` with a set of request helper methods. These methods are **getter-like helpers** that read from request headers and URL components and return parsed, negotiated, or boolean results. The `accepts` family (`req.accepts`, `req.acceptsCharsets`, `req.acceptsEncodings`, `req.acceptsLanguages`) implements HTTP content negotiation, returning the best match among the types the client advertises it can handle. `req.get()` (aliased as `req.header()`) performs case-insensitive header lookup with `Referer`/`Referrer` aliasing. `req.is()` checks the request's `Content-Type` against one or more MIME types and returns the matched type string, `false`, or `null`. `req.range()` parses the `Range` header into byte ranges for partial-content responses. Express 5 removed `req.param(name)` and pluralized the `acceptsCharset`, `acceptsEncoding`, and `acceptsLanguage` method names.

**Beginner-Friendly Explanation**
Request methods are little helpers on the `req` object that answer specific questions about what the client sent. `req.get('Authorization')` reads a header. `req.accepts('html')` asks “can this client understand HTML?” `req.is('json')` asks “is this request's body JSON?” `req.range(1000)` asks “does the client want only part of this file?” They save you from manually parsing raw header strings and make content negotiation and conditional responses straightforward.

---

### Key Characteristics

- **Header-focused** — Most request methods read from request headers (`Accept`, `Content-Type`, `Range`) and return parsed or negotiated results.
- **Case-insensitive** — `req.get()` and the `accepts` family treat header names and media types case-insensitively.
- **Express 5 breaking changes** — `req.param()` is **removed**; use `req.params`, `req.query`, or `req.body` directly. The singular `acceptsCharset`, `acceptsEncoding`, and `acceptsLanguage` methods are **removed** and replaced by their pluralized forms.
- **Content negotiation** — The `accepts` family returns the best match or a falsy value, enabling `406 Not Acceptable` responses.
- **Range support** — `req.range()` enables `206 Partial Content` responses for large resources and resumable downloads.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- For `req.body`: `express.json()` or `express.urlencoded()` middleware registered before routes.
- Basic understanding of HTTP headers (`Accept`, `Content-Type`, `Range`) and MIME types.
- Familiarity with the `(req, res, next)` middleware signature.

---

### Related Programming Areas

- **Content negotiation** — The `accepts` family implements HTTP content negotiation.
- **Body parsing** — `req.body` depends on body-parsing middleware.
- **Conditional and partial responses** — `req.range()` and `req.fresh` support `206` and `304` responses.
- **Security** — Header inspection for `Authorization`, `Content-Type`, and API keys.
- **Static file serving** — `req.range()` and `req.accepts` are used by `res.sendFile()` internally.

---

## Core Concept 1: `req.get()` and `req.header()`

### Definitions

**Core Definition**
`req.get(field)` returns the value of the specified request header, performing a case-insensitive match, and is aliased as `req.header(field)`.

**Technical Definition**
`req.get()` reads from `req.headers` and returns the header value as a string. Header names are matched case-insensitively. The `Referrer` header field is special-cased: both `Referrer` and `Referer` are interchangeable. If the header is not present, `req.get()` returns `undefined`; an optional `defaultValue` can be supplied. `req.header()` is a direct alias with identical behavior.

**Beginner-Friendly Explanation**
`req.get('Authorization')` gives you the value of the `Authorization` header, regardless of whether it was sent as `authorization` or `Authorization`. It is the safe, idiomatic way to read a single header.

---

### Purposes

- To read authentication tokens and API keys from the `Authorization` header.
- To inspect `Content-Type`, `Accept`, and other protocol headers.
- To retrieve the `Referrer`/`Referer` header for redirect-back flows.
- To supply a default value when a header is absent.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
req.get(field);                  // returns header value or undefined
req.get(field, defaultValue);    // returns defaultValue if absent
req.header(field);               // alias of req.get()
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `field` | Header name (case-insensitive) |
| `defaultValue` | Optional value returned when the header is missing |
| `req.header()` | Alias, identical behavior |

#### Syntax Rules

- Header names are case-insensitive; `req.get('Content-Type')` and `req.get('content-type')` return the same value.
- `Referrer` and `Referer` are interchangeable.
- Returns `undefined` when the header is absent and no default is provided.
- `req.headers` remains available for whole-object access.

#### Constraints and Limitations

- `req.get()` returns the first value for repeated headers; use `req.headers` for raw access if multiple values matter.
- Header values are always strings; numeric or boolean interpretation requires explicit conversion.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Reading the Authorization Header

```javascript
app.get('/protected', (req, res) => {
  const auth = req.get('Authorization');
  if (!auth || !auth.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing or invalid Authorization header' });
  }
  res.json({ token: auth.slice(7) });
});
```

**Expected Output**

```
GET /protected  (Authorization: Bearer abc123)
→ 200 { "token": "abc123" }

GET /protected  (no header)
→ 401 { "error": "Missing or invalid Authorization header" }
```

**Why This Output Occurs**

- `req.get('Authorization')` returns the header string; the handler validates its format before extracting the token.

---

#### Example 2: Referrer Fallback with Default Value

```javascript
app.post('/logout', (req, res) => {
  const referrer = req.get('Referrer', '/');
  res.redirect(referrer);
});
```

**Expected Output**

```
POST /logout  (Referrer: /dashboard)
→ 302 Location: /dashboard

POST /logout  (no Referrer)
→ 302 Location: /
```

**Why This Output Occurs**

- The `Referrer` header is read case-insensitively; the default `/` is used when the header is absent.

---

### Real-World Cases

- **Authentication** — Reading Bearer tokens from `Authorization`.
- **Redirect-back flows** — Using the `Referrer` header to return users to their previous page.
- **API versioning** — Reading custom version headers such as `API-Version`.

---

## Core Concept 2: `req.accepts()`

### Definitions

**Core Definition**
`req.accepts(types)` checks the request's `Accept` header against one or more media types and returns the **best match** if the client accepts any of them, or `false` if none are acceptable.

**Technical Definition**
`req.accepts()` uses the `accepts` package to perform content negotiation. It accepts a variable number of type arguments (or an array) and returns the best matching type string. When the `Accept` header is absent, it returns the first supplied type. The method supports MIME types (`'text/html'`), subtypes (`'html'`), and wildcards (`'image/*'`). If no supplied type is acceptable, it returns `false`, and the application should respond with `406 Not Acceptable`.

**Beginner-Friendly Explanation**
`req.accepts('html', 'json')` asks: “Can the client handle HTML or JSON?” If it can handle HTML, you get `'html'`; if only JSON, you get `'json'`; if neither, you get `false`. It is how you decide what format to send back.

---

### Purposes

- To select the response format (HTML, JSON, XML) based on the client's `Accept` header.
- To implement `406 Not Acceptable` responses when no supported format is acceptable.
- To support API clients that request different representations of the same resource.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
req.accepts(types);              // single type or array of types
req.accepts('html');
req.accepts('html', 'json');
req.accepts(['html', 'json']);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `types` | One or more media types, subtypes, or wildcards |
| Return value | Best match string, or `false` |

#### Syntax Rules

- Returns the **best** match when multiple types are supplied.
- Returns `false` when none of the supplied types are acceptable.
- When the `Accept` header is absent, returns the first supplied type.
- Supports shorthand subtypes (`'html'`) and wildcards (`'image/*'`).

#### Constraints and Limitations

- `req.accepts()` does not set the response `Content-Type`; you must do that explicitly.
- `false` should trigger a `406` response, not a `200`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Format Selection with `req.accepts()`

```javascript
app.get('/data', (req, res) => {
  if (req.accepts('json')) {
    return res.json({ data: 'value' });
  }
  if (req.accepts('html')) {
    return res.send('<p>value</p>');
  }
  res.status(406).json({ error: 'Not Acceptable' });
});
```

**Expected Output**

```
GET /data  (Accept: application/json)  → 200 { "data": "value" }
GET /data  (Accept: text/html)         → 200 "<p>value</p>"
GET /data  (Accept: application/xml)   → 406 { "error": "Not Acceptable" }
```

**Why This Output Occurs**

- `req.accepts('json')` returns truthy when JSON is acceptable; `req.accepts('html')` handles HTML; otherwise `406` is returned.

---

### Real-World Cases

- **REST APIs with multiple representations** — JSON, XML, and HTML for the same resource.
- **Content negotiation** — Serving different formats based on client capabilities.
- **API versioning** — Using custom media types such as `application/vnd.api.v1+json`.

---

## Core Concept 3: `req.acceptsCharsets()`, `req.acceptsEncodings()`, `req.acceptsLanguages()`

### Definitions

**Core Definition**
These methods check the request's `Accept-Charset`, `Accept-Encoding`, and `Accept-Language` headers against supplied values and return the best match.

**Technical Definition**
Each method accepts one or more values (or an array) and returns the best match based on the client's quality values. `req.acceptsCharsets()` negotiates character encodings (e.g., `utf-8`), `req.acceptsEncodings()` negotiates content codings (e.g., `gzip`, `br`), and `req.acceptsLanguages()` negotiates natural languages (e.g., `en`, `fr`). In Express 5, the singular forms `acceptsCharset`, `acceptsEncoding`, and `acceptsLanguage` are **removed** and replaced by these pluralized forms.

**Beginner-Friendly Explanation**
`req.acceptsLanguages('en', 'fr')` asks: “Does the client prefer English or French?” and returns the best match. `req.acceptsEncodings('gzip', 'br')` asks: “Can the client handle gzip or Brotli compression?” These methods are how you negotiate character sets, compression, and language.

---

### Purposes

- To negotiate content compression (`gzip`, `br`, `deflate`).
- To select the character encoding for text responses.
- To choose the response language for internationalized applications.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
req.acceptsCharsets(charsets);
req.acceptsEncodings(encodings);
req.acceptsLanguages(languages);

req.acceptsCharsets('utf-8');
req.acceptsEncodings('gzip', 'br');
req.acceptsLanguages('en', 'fr');
```

**Component Breakdown**

| Method | Header | Typical values |
|--------|--------|----------------|
| `req.acceptsCharsets()` | `Accept-Charset` | `utf-8`, `iso-8859-1` |
| `req.acceptsEncodings()` | `Accept-Encoding` | `gzip`, `br`, `deflate` |
| `req.acceptsLanguages()` | `Accept-Language` | `en`, `fr`, `de` |

#### Syntax Rules

- Return the best match based on quality values, or `false` if none match.
- The singular forms are removed in Express 5; use the pluralized methods.
- Accept arrays or variable arguments.

#### Constraints and Limitations

- These methods negotiate, but do not apply the encoding or language; you must set the appropriate response headers and content.
- `Accept-Encoding` is often handled automatically by the `compression` middleware.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Compression and Language Negotiation

```javascript
app.get('/greeting', (req, res) => {
  const lang = req.acceptsLanguages('en', 'fr', 'de') || 'en';
  const encoding = req.acceptsEncodings('gzip', 'br', 'identity');

  const messages = {
    en: 'Hello',
    fr: 'Bonjour',
    de: 'Hallo'
  };

  res.set('Content-Encoding', encoding || 'identity');
  res.json({ language: lang, encoding, message: messages[lang] });
});
```

**Expected Output**

```
GET /greeting  (Accept-Language: fr, Accept-Encoding: br)
→ 200 { "language": "fr", "encoding": "br", "message": "Bonjour" }

GET /greeting  (Accept-Language: es, Accept-Encoding: gzip)
→ 200 { "language": "en", "encoding": "gzip", "message": "Hello" }
```

**Why This Output Occurs**

- `req.acceptsLanguages('en', 'fr', 'de')` returns `'fr'` when French is preferred and falls back to `'en'` when Spanish is requested. `req.acceptsEncodings` selects the best supported encoding.

---

### Real-World Cases

- **Internationalized applications** — Selecting the response language from `Accept-Language`.
- **Compression negotiation** — Choosing `gzip` or `br` based on `Accept-Encoding`.
- **Legacy encoding support** — Serving `iso-8859-1` when requested.

---

## Core Concept 4: `req.is()`

### Definitions

**Core Definition**
`req.is(type)` checks whether the incoming request's `Content-Type` header matches the given MIME type and returns the matched type string, `false`, or `null`.

**Technical Definition**
`req.is()` uses the `type-is` package to compare the request's `Content-Type` against one or more types. It returns the matched type string (e.g., `'json'` or `'text/html'`), `false` if the content type does not match any supplied type, or `null` if the request has no body or no `Content-Type` header. Shorthand types such as `'json'` and wildcards such as `'image/*'` are supported. `req.is()` is the inverse of `req.accepts()`: it checks what the client sent, not what it can receive.

**Beginner-Friendly Explanation**
`req.is('json')` asks: “Is the body of this request JSON?” If yes, you get `'json'` back; if not, `false`; if there is no body, `null`. It is how you check what format the client actually sent.

---

### Purposes

- To reject requests with unsupported `Content-Type` values with `415 Unsupported Media Type`.
- To branch on the request's body format (JSON, form, text, XML).
- To validate webhook payloads before parsing.
- To enforce API contracts that only accept specific media types.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
req.is(type);          // single type
req.is(type1, type2);  // multiple types
req.is(['json', 'urlencoded']);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `type` | MIME type, shorthand subtype, or wildcard |
| Return value | Matched type string, `false`, or `null` |

#### Syntax Rules

- Returns the matched type string when the `Content-Type` matches.
- Returns `false` when the `Content-Type` does not match any supplied type.
- Returns `null` when the request has no body or no `Content-Type` header.
- Shorthand (`'json'`, `'html'`) and wildcards (`'image/*'`) are supported.

#### Constraints and Limitations

- `req.is()` checks the **request** `Content-Type`, not the `Accept` header.
- A `null` result means the request has no body; do not treat it as an error unless a body is required.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Content-Type Validation

```javascript
app.post('/webhook', (req, res) => {
  const type = req.is('application/json');

  if (!type) {
    return res.status(415).json({ error: 'Content-Type must be application/json' });
  }

  res.json({ received: req.body });
});
```

**Expected Output**

```
POST /webhook  (Content-Type: application/json)
→ 200 { "received": { ... } }

POST /webhook  (Content-Type: text/plain)
→ 415 { "error": "Content-Type must be application/json" }

POST /webhook  (no body)
→ 415 { "error": "Content-Type must be application/json" }
```

**Why This Output Occurs**

- `req.is('application/json')` returns the matched type when JSON is sent and `false` or `null` otherwise, triggering a `415` response.

---

### Real-World Cases

- **Webhook endpoints** — Validating that payloads are JSON before parsing.
- **File upload routes** — Checking `image/*` or `application/pdf` content types.
- **API contracts** — Rejecting unsupported media types with `415`.

---

## Core Concept 5: `req.param()` — Removed in Express 5

### Definitions

**Core Definition**
`req.param(name)` was a legacy Express method that searched `req.params`, `req.body`, and `req.query` in that order for a named parameter. It was **removed in Express 5** because it was confusing and potentially dangerous.

**Technical Definition**
In Express 4, `req.param(name)` returned the value of `name` from route parameters first, then the request body, then the query string. This behavior could silently retrieve data from unintended sources, creating security risks. Express 5 **removes** `req.param()` entirely; developers must explicitly read from `req.params`, `req.body`, or `req.query`. The Express codemod can automate the migration.

**Beginner-Friendly Explanation**
`req.param()` was a shortcut that looked in three places for a value. It was removed because it could accidentally pick up a value from the wrong place—like a query parameter when you expected a route parameter. In Express 5, you must say exactly where to look: `req.params.id`, `req.body.name`, or `req.query.search`.

---

### Purposes

- **Express 4 only**: To retrieve a parameter without specifying its source.
- **Express 5**: No purpose—use `req.params`, `req.body`, or `req.query` directly.

---

### Syntax Rules and Structure

#### Complete General Syntax: Express 4 (Removed in Express 5)

```javascript
// Express 4 only — removed in Express 5
req.param('id');   // checks params, then body, then query

// Express 5 equivalents
req.params.id;     // route parameters
req.body.id;       // request body
req.query.id;      // query string
```

**Component Breakdown**

| Express 4 | Express 5 Equivalent |
|-----------|---------------------|
| `req.param('id')` | `req.params.id` |
| `req.param('name')` | `req.body.name` |
| `req.param('search')` | `req.query.search` |

#### Syntax Rules

- **Do not use `req.param()` in Express 5** — it is removed and will throw or be `undefined`.
- Read from the specific source object explicitly.
- Use the codemod `npx @expressjs/codemod req-param` to automate migration.

#### Constraints and Limitations

- `req.param()` is a breaking change in Express 5; applications upgrading from Express 4 must update every call site.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Migrating from `req.param()` to Explicit Sources

```javascript
// Express 4 (removed in Express 5)
app.get('/users/:id', (req, res) => {
  const id = req.param('id');
  res.json({ id });
});

// Express 5
app.get('/users/:id', (req, res) => {
  const id = req.params.id;
  res.json({ id });
});
```

**Expected Output**

```
GET /users/42
→ 200 { "id": "42" }
```

**Why This Output Occurs**

- The Express 5 version reads directly from `req.params`, avoiding the ambiguous multi-source lookup.

---

### Real-World Cases

- **Express 4 to 5 migrations** — Updating legacy code that relied on `req.param()`.
- **Security audits** — Ensuring parameters are read from the intended source, not accidentally from the body or query string.

---

## Core Concept 6: `req.range()`

### Definitions

**Core Definition**
`req.range(size, options)` parses the request's `Range` header and returns an array of `{ start, end }` ranges for the specified resource size, or a status code indicating unsatisfiable or invalid ranges.

**Technical Definition**
`req.range()` uses the `range-parser` package to parse the `Range` header. It returns an **array of ranges** when the header is valid, `-1` when the range is **unsatisfiable** (e.g., the requested range exceeds the resource size), `-2` when the header is **invalid**, or `undefined` when no `Range` header is present. The `options.combine` flag merges overlapping or adjacent ranges into a single range. This method enables `206 Partial Content` responses for large files, resumable downloads, and media streaming.

**Beginner-Friendly Explanation**
`req.range(fileSize)` asks: “Does the client want only part of this file?” If the client sends `Range: bytes=0-499`, you get an array with `{ start: 0, end: 499 }`. You can then send only those bytes with a `206 Partial Content` status. If the range is invalid or unsatisfiable, you get a negative number you can use to send the correct error response.

---

### Purposes

- To implement `206 Partial Content` responses for large files.
- To support resumable downloads and media streaming.
- To validate range requests and return `416 Range Not Satisfiable` when appropriate.
- To combine overlapping ranges for efficient serving.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
req.range(size);                    // returns array, -1, -2, or undefined
req.range(size, { combine: true }); // combines overlapping ranges
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `size` | Total resource size in bytes |
| `options.combine` | Merges overlapping/adjacent ranges (default `false`) |
| Return value | Array of `{ start, end }`, `-1`, `-2`, or `undefined` |

#### Return Values

| Value | Meaning |
|-------|---------|
| `Array<{ start, end }>` | Valid ranges |
| `-1` | Unsatisfiable range (respond `416`) |
| `-2` | Invalid/malformed range header |
| `undefined` | No `Range` header (respond `200` with full body) |

#### Syntax Rules

- Pass the **total resource size** so `req.range()` can validate the requested ranges.
- Use `options.combine: true` to merge overlapping ranges.
- Check for `-1` and `-2` before treating the result as an array.
- Set `Accept-Ranges: bytes` on responses that support ranges.

#### Constraints and Limitations

- `req.range()` only parses the header; it does not read or send file bytes.
- Multi-range responses require `multipart/byteranges`; most applications serve a single range for simplicity.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Partial Content with `req.range()`

```javascript
app.get('/video', (req, res) => {
  const size = 10_000;
  const ranges = req.range(size);

  if (ranges === -1) {
    return res.status(416).set('Content-Range', `bytes */${size}`).end();
  }
  if (ranges === -2 || !ranges) {
    return res.status(200).send('full content');
  }

  const { start, end } = ranges[0];
  res.status(206).set({
    'Content-Range': `bytes ${start}-${end}/${size}`,
    'Content-Length': end - start + 1
  }).send(`bytes ${start}-${end}`);
});
```

**Expected Output**

```
GET /video  (Range: bytes=0-499)
→ 206 Partial Content
Content-Range: bytes 0-499/10000

GET /video  (Range: bytes=20000-30000)
→ 416 Range Not Satisfiable
Content-Range: bytes */10000
```

**Why This Output Occurs**

- `req.range(10000)` returns an array for a valid range, `-1` for an unsatisfiable range, and `undefined` when no `Range` header is sent.

---

### Real-World Cases

- **Video and audio streaming** — Serving media in byte ranges.
- **Large file downloads** — Resumable downloads for unstable networks.
- **PDF viewers** — Fetching only the requested pages.

---

## Core Concept 7: `req.body` — Body-Parser-Dependent

### Definitions

**Core Definition**
`req.body` is an object containing the parsed request body, populated only when body-parsing middleware such as `express.json()` or `express.urlencoded()` is registered.

**Technical Definition**
By default, `req.body` is `undefined`. When `express.json()` is mounted, it parses JSON payloads and populates `req.body` as a JavaScript object. When `express.urlencoded()` is mounted, it parses URL-encoded form payloads and populates `req.body` with the form fields. The shape of `req.body` is entirely determined by the parser and the client's `Content-Type`; every property is **untrusted user input** and must be validated.

**Beginner-Friendly Explanation**
`req.body` is the data the client sent in the request. Express does not parse it automatically. You must add `app.use(express.json())` for JSON bodies or `app.use(express.urlencoded())` for form bodies. Once parsed, you can read `req.body.email` or `req.body.password`, but you must validate everything because the client controls it.

---

### Purposes

- To read JSON payloads from API clients.
- To read HTML form submissions encoded as `application/x-www-form-urlencoded`.
- To validate and process data submitted by POST, PUT, and PATCH requests.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.post('/users', (req, res) => {
  const { email, password } = req.body; // available after parsing
  // validate before use
});
```

**Component Breakdown**

| Parser | Content-Type | Result |
|--------|-------------|--------|
| `express.json()` | `application/json` | Object or array |
| `express.urlencoded()` | `application/x-www-form-urlencoded` | Object of form fields |

#### Syntax Rules

- Register body-parsing middleware **before** routes that read `req.body`.
- Validate every property of `req.body`; never trust its shape.
- Use `express.json()` and `express.urlencoded()` instead of the deprecated `body-parser` package.

#### Constraints and Limitations

- `req.body` is `undefined` when no parser is registered or the `Content-Type` does not match.
- Body size limits should be configured to prevent denial-of-service.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: JSON Body Parsing

```javascript
app.use(express.json());

app.post('/users', (req, res) => {
  const { email, password } = req.body;
  if (typeof email !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  res.status(201).json({ email });
});
```

**Expected Output**

```
POST /users  { "email": "alice@example.com", "password": "secret" }
→ 201 { "email": "alice@example.com" }

POST /users  { "email": 123 }
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

## Request Methods Comparison

| Method | Purpose | Return Value | Express 5 Status |
|--------|---------|--------------|------------------|
| `req.get(field)` | Read a header case-insensitively | String or `undefined` | Supported |
| `req.header(field)` | Alias of `req.get()` | String or `undefined` | Supported |
| `req.accepts(types)` | Best `Accept` match | Best match string or `false` | Supported |
| `req.acceptsCharsets()` | Best `Accept-Charset` match | Best match or `false` | Supported (pluralized) |
| `req.acceptsEncodings()` | Best `Accept-Encoding` match | Best match or `false` | Supported (pluralized) |
| `req.acceptsLanguages()` | Best `Accept-Language` match | Best match or `false` | Supported (pluralized) |
| `req.is(type)` | Check `Content-Type` match | Matched string, `false`, or `null` | Supported |
| `req.param(name)` | Multi-source parameter lookup | Parameter value | **Removed** |
| `req.range(size)` | Parse `Range` header | Array of ranges, `-1`, `-2`, or `undefined` | Supported |
| `req.body` | Parsed request body | Object (parser-dependent) | Supported (parser required) |

---

## Request Methods Best Practices

### Definitions

**Core Definition**
Request methods best practices are the habits that keep header access, content negotiation, range parsing, and body handling safe, predictable, and Express 5-compatible.

**Technical Definition**
Best practices include: using `req.get()` instead of raw `req.headers` for case-insensitive header access; using the pluralized `acceptsCharsets`, `acceptsEncodings`, and `acceptsLanguages` methods in Express 5; returning `406 Not Acceptable` when `req.accepts()` returns `false`; using `req.is()` to return `415 Unsupported Media Type` for wrong content types; replacing `req.param()` with explicit `req.params`, `req.body`, or `req.query` access; validating `req.range()` results before serving partial content; and always registering body parsers before reading `req.body`.

### Purposes

- To avoid Express 5 breaking changes and deprecated APIs.
- To implement correct HTTP semantics (`406`, `415`, `416`, `206`).
- To prevent security issues from ambiguous parameter sources.
- To ensure body data is parsed and validated before use.

### Syntax Rules and Structure

- **Use `req.get()`** for headers; it is case-insensitive and handles `Referer`/`Referrer`.
- **Use pluralized accepts methods** in Express 5; the singular forms are removed.
- **Return `406`** when `req.accepts()` returns `false`.
- **Return `415`** when `req.is()` returns `false` or `null` for required bodies.
- **Never use `req.param()`**; read from the specific source object.
- **Validate `req.range()`** return values before slicing content.
- **Register body parsers** before routes that read `req.body`.

### Constraints and Limitations

- `req.accepts()` does not set `Content-Type`; you must set it explicitly.
- `req.range()` only parses the header; serving bytes is your responsibility.

---

## References

- Express 5.x API Reference — https://expressjs.com/en/5x/api/
- Migrating to Express 5 (pluralized `accepts` methods, `req.param()` removal) — https://expressjs.com/en/guide/migrating-5.html
- Express Request prototype (`req.get`, `req.accepts`, `req.range`, `req.is`) — https://factory.com/open-source-wikis/express?page=systems%2Frequest.md
- `req.accepts()` return value and `406` guidance — https://github.com/expressjs/expressjs.com/blob/dcecd7a0e05963e0622bbe0801d9e265fda5924b/_includes/api/en/4x/req-accepts.jade
- `range-parser` options (`combine`) — https://github.com/jshttp/range-parser
- `req.is()` and `type-is` — https://github.com/jshttp/type-is
- `express.json()` and `express.urlencoded()` — https://expressjs.com/en/5x/api/#express.json
- Express Body Parser (Compile-N-Run) — https://github.com/Compile-N-Run/Compile-N-Run/blob/main/docs/framework/express/2-express-middleware/10-express-body-parser.mdx