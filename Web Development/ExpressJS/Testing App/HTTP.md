# HTTP Testing — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP testing is the practice of verifying that an HTTP server responds correctly to requests — validating status codes, headers, response bodies, cookies, and streamed content — by exercising the full request–response lifecycle in an automated test suite.

**Technical Definition:** HTTP testing in Node.js uses Supertest, a SuperAgent-driven library that binds an Express (or Fastify, Koa, or plain `http.Server`) application to an ephemeral port and sends real HTTP requests through the full middleware stack. Assertions are made on the response status code, headers, body, cookies, and — for streaming endpoints — individual chunks of the response stream. Supertest integrates with Jest or Mocha for test organisation and assertions, and supports chained `.expect()` calls for status codes, headers, and body content.

**Beginner-Friendly Explanation:** You write tests that say "send a GET request to `/api/users`, and check that the response is a 200 with a JSON array." Supertest handles all the HTTP details — opening a connection, sending the request, reading the response — so you can focus on what the response should contain.

### Key Characteristics

- **In-process execution:** Supertest drives the app directly without starting a real server on a fixed port. 
- **Chained assertions:** `.expect(200)`, `.expect('Content-Type', /json/)`, `.expect(body => { ... })` are chained on the request object.
- **Full middleware stack:** Requests pass through authentication, validation, rate limiting, and error-handling middleware.
- **Cookie and session support:** Supertest agents and `supertest-session` maintain cookies across requests for session-based auth testing.
- **Streaming support:** Server-Sent Events (SSE) and file streaming responses can be tested by consuming the response stream chunk by chunk.
- **Header assertions:** Cache-Control, Content-Type, Security headers (Helmet), CORS headers, and custom headers can all be asserted.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Supertest installed:** `npm install --save-dev supertest`.
- **A test runner:** Jest (`npm install --save-dev jest`) or Mocha.
- **An Express app exported without `app.listen()`** so Supertest can bind it.
- **For session testing:** `supertest-session` (`npm install --save-dev supertest-session`).

### Related Programming Areas

- **Unit Testing:** Testing individual functions in isolation.
- **Integration Testing:** Testing the full request–response lifecycle.
- **End-to-End Testing:** Testing with a real browser or client.
- **API Contract Testing:** Verifying responses match an OpenAPI specification.
- **Security Testing:** Verifying Helmet headers, CORS policies, and authentication enforcement.

### Core Concepts

1. **Supertest** — the HTTP assertion library.
2. **Request Assertions** — verifying request construction.
3. **Response Assertions** — verifying response status, headers, and body.
4. **Status-Code Assertions** — 200, 201, 204, 400, 401, 403, 404, 422, 429, 500.
5. **Header Assertions** — Cache-Control, Security headers (Helmet), CORS.
6. **Cookie and Session State Assertions** — cookies, agents, sessions.
7. **Streamed Response Assertions** — Server-Sent Events, file streaming.

---

## Core Concept 1: Supertest

### Definitions

**Core Definition:** Supertest is a Node.js library that provides a high-level abstraction for testing HTTP servers, built on top of SuperAgent, allowing fluent assertion chains against Express, Fastify, Koa, or plain `http.Server` instances.

**Technical Definition:** Supertest wraps an HTTP server or request listener function, binds it to an ephemeral port, and sends real HTTP requests through the full server stack. It extends SuperAgent's request object with `.expect()` methods that queue assertions, which are executed when the response arrives. Supertest supports Promise-based and callback-based usage, cookie agents for session persistence, and file uploads via `.attach()`.

**Beginner-Friendly Explanation:** Supertest is the testing library that sends HTTP requests to your app and lets you check the responses. You write `request(app).get('/users').expect(200)` and Supertest does the rest.

### Purposes

- To provide a fluent, expressive API for HTTP integration testing.
- To drive Express/Fastify/Koa apps in-process without network flakiness.
- To support chained assertions on status codes, headers, and bodies.
- To handle cookies, redirects, and file uploads automatically.

### Syntax Rules and Structure

```js
const request = require('supertest');
const app = require('./app');

const response = await request(app)
  .get('/api/users')
  .set('Accept', 'application/json')
  .expect('Content-Type', /json/)
  .expect(200);

expect(response.body).toHaveLength(3);
```

| Method | Purpose |
|--------|---------|
| `request(app)` | Create a test agent bound to the app. |
| `.get(path)` | Send a GET request. |
| `.post(path).send(body)` | Send a POST with JSON body. |
| `.set(header, value)` | Set a request header. |
| `.query({ key: value })` | Add query parameters. |
| `.expect(status)` | Assert the status code. |
| `.expect(header, value)` | Assert a response header. |
| `.expect(body)` | Assert deep equality on the body. |
| `.expect(fn)` | Custom assertion function. |

**Rules:**
- Export the Express `app` without calling `app.listen()`.
- Supertest binds the app to an ephemeral port internally.
- All request methods return Promises when `await`ed.
- `.expect()` assertions can be chained.
- If using `.end()`, assertion failures are passed to the callback, not thrown. 

### Annotated Code Example

```js
// test/supertest-basic.test.js
const request = require('supertest');
const app = require('../app');

describe('Supertest basics', () => {
  it('sends a GET request and asserts the response', async () => {
    const response = await request(app)
      .get('/api/health')
      .set('Accept', 'application/json')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(response.body).toEqual({ status: 'ok' });
  });

  it('sends a POST request with a body', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@example.com' })
      .expect(201);

    expect(response.body).toHaveProperty('id');
    expect(response.body.name).toBe('Alice');
  });
});
```

**Expected Output:**
```
PASS  test/supertest-basic.test.js
  Supertest basics
    ✓ sends a GET request and asserts the response (45 ms)
    ✓ sends a POST request with a body (62 ms)

Tests: 2 passed, 2 total
```

**Why this output:** Supertest sends real HTTP requests through the Express app. The `.expect()` chains assert on the status code and Content-Type header. The `response.body` contains the parsed JSON response. 

### Real-World Cases

- **REST APIs:** Testing CRUD endpoints end-to-end.
- **Authentication:** Testing login, token issuance, and protected routes.
- **File uploads:** Testing multipart form handling with `.attach()`.

---

## Core Concept 2: Request Assertions

### Definitions

**Core Definition:** Request assertions verify that a request was constructed correctly — with the expected headers, query parameters, body, and authentication — before it is sent to the server.

**Technical Definition:** Supertest inherits SuperAgent's request-building methods: `.set(header, value)` for headers, `.query(object)` for query parameters, `.send(body)` for JSON bodies, `.type(contentType)` for content type, and `.attach(field, file)` for multipart file uploads. These methods can be chained and are executed when the request is sent. While Supertest does not assert on the request itself (it asserts on the response), the request configuration determines what the server receives, and therefore what the response should be.

**Beginner-Friendly Explanation:** Before you send a request, you configure it: "use this header, add these query parameters, send this body." Supertest lets you build the request with chained methods, then assert on the response that comes back.

### Purposes

- To set custom headers (Authorization, Accept, Content-Type, API keys).
- To add query parameters for filtering, sorting, and pagination.
- To send JSON bodies for POST, PUT, and PATCH requests.
- To attach files for multipart/form-data uploads.
- To configure the request exactly as a real client would.

### Syntax Rules and Structure

```js
await request(app)
  .post('/api/users')
  .set('Authorization', 'Bearer token123')
  .set('Accept', 'application/json')
  .query({ notify: 'true' })
  .send({ name: 'Alice', email: 'alice@example.com' })
  .expect(201);
```

| Method | Purpose |
|--------|---------|
| `.set(header, value)` | Set a request header. |
| `.query({ key: value })` | Add query parameters. |
| `.send(body)` | Send a JSON body. |
| `.type('application/json')` | Set the Content-Type header. |
| `.attach('file', path)` | Attach a file for multipart upload. |
| `.field('name', 'value')` | Add a form field alongside a file. |
| `.auth('user', 'pass')` | Set Basic Auth credentials. |
| `.redirects(n)` | Set the maximum number of redirects. |

**Rules:**
- `.set()` can be called multiple times for different headers.
- `.query()` accepts an object and serialises it to a query string.
- `.send()` automatically sets `Content-Type: application/json` for objects.
- `.attach()` requires the field name and either a file path or a Buffer.
- `.field()` is used alongside `.attach()` for text fields in multipart requests.

### Annotated Code Example

```js
// test/request-assertions.test.js
const request = require('supertest');
const app = require('../app');

describe('Request construction', () => {
  it('sends authenticated request with headers and query params', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Authorization', 'Bearer valid-token')
      .set('X-API-Version', '2')
      .query({ page: 2, limit: 10, sort: 'createdAt' })
      .expect(200);

    expect(response.body.meta.page).toBe(2);
    expect(response.body.meta.limit).toBe(10);
  });

  it('uploads a file with multipart/form-data', async () => {
    const response = await request(app)
      .post('/api/upload')
      .attach('avatar', Buffer.from('fake-image'), 'avatar.png')
      .field('description', 'Profile picture')
      .expect(201);

    expect(response.body.originalname).toBe('avatar.png');
  });

  it('sends a JSON body with correct content type', async () => {
    const response = await request(app)
      .post('/api/users')
      .type('json')
      .send({ name: 'Bob', email: 'bob@example.com' })
      .expect(201);

    expect(response.body.name).toBe('Bob');
  });
});
```

**Expected Output:**
```
PASS  test/request-assertions.test.js
  Request construction
    ✓ sends authenticated request with headers and query params (38 ms)
    ✓ uploads a file with multipart/form-data (45 ms)
    ✓ sends a JSON body with correct content type (32 ms)

Tests: 3 passed, 3 total
```

**Why this output:** Each test configures the request differently. The first sets an Authorization header and query parameters. The second attaches a file as a Buffer with a field name. The third sends a JSON body with an explicit content type. The server receives exactly what the client would send.

### Real-World Cases

- **Authentication:** Testing Bearer tokens, API keys, and Basic Auth.
- **Pagination:** Testing `?page=2&limit=20` query parameters.
- **File uploads:** Testing multipart/form-data with `.attach()`.
- **API versioning:** Testing `X-API-Version` header-based versioning.

---

## Core Concept 3: Response Assertions

### Definitions

**Core Definition:** Response assertions verify that the server's response — status code, headers, and body — matches expectations.

**Technical Definition:** Supertest provides two assertion mechanisms: chained `.expect()` calls on the request object (which queue assertions executed when the response arrives), and direct assertions on the `response` object using Jest's `expect()`. The `response` object exposes `status`, `statusCode`, `headers`, `body`, and `text`. Response body assertions support exact match (`toEqual`), partial match (`toMatchObject`), array assertions (`toHaveLength`, `arrayContaining`), and custom validation functions.

**Beginner-Friendly Explanation:** After sending a request, you check the response: "Is the status 200? Is the Content-Type JSON? Does the body contain the right data?" Supertest gives you both chained assertions and direct property access.

### Purposes

- To verify the response status code.
- To verify response headers (Content-Type, Cache-Control, custom headers).
- To verify the response body structure and content.
- To verify error response formats (RFC 7807 Problem Details).
- To validate response bodies against a JSON Schema.

### Syntax Rules and Structure

```js
// Chained assertions
await request(app)
  .get('/api/users')
  .expect(200)
  .expect('Content-Type', /json/)
  .expect((res) => {
    if (!Array.isArray(res.body)) throw new Error('Expected an array');
  });

// Direct assertions
const response = await request(app).get('/api/users');
expect(response.status).toBe(200);
expect(response.headers['content-type']).toMatch(/json/);
expect(response.body).toHaveLength(3);
expect(response.body[0]).toHaveProperty('name');
```

| Assertion | Purpose |
|-----------|---------|
| `.expect(200)` | Assert status code. |
| `.expect('Content-Type', /json/)` | Assert header value (string or regex). |
| `.expect({ key: 'value' })` | Assert deep equality on the body. |
| `.expect(fn)` | Custom assertion function. |
| `expect(response.body).toEqual(...)` | Exact match. |
| `expect(response.body).toMatchObject(...)` | Partial match. |
| `expect(response.body).toHaveLength(n)` | Array length. |
| `expect(response.body).toEqual(expect.arrayContaining([...]))` | Array contains items. |

**Rules:**
- Chained `.expect()` calls are executed in order.
- `.expect(body)` performs a deep equality check.
- `.expect(fn)` passes the response object to the function for custom logic.
- Direct assertions on `response.body` use Jest's matchers.
- `response.text` contains the raw response body as a string (useful for non-JSON responses).

### Annotated Code Example

```js
// test/response-assertions.test.js
const request = require('supertest');
const app = require('../app');

describe('Response assertions', () => {
  it('asserts exact body match', async () => {
    const response = await request(app)
      .get('/api/config')
      .expect(200);

    expect(response.body).toEqual({
      theme: 'dark',
      language: 'en',
      notifications: true
    });
  });

  it('asserts partial body match with objectContaining', async () => {
    const response = await request(app)
      .get('/api/users/1')
      .expect(200);

    expect(response.body).toEqual(
      expect.objectContaining({
        name: 'Alice',
        email: 'alice@example.com'
      })
    );
  });

  it('asserts array length and contents', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect(200);

    expect(response.body).toHaveLength(3);
    expect(response.body).toEqual(
      expect.arrayContaining([
        expect.objectContaining({ name: 'Alice' })
      ])
    );
  });

  it('asserts nested object structure', async () => {
    const response = await request(app)
      .get('/api/users/1/profile')
      .expect(200);

    expect(response.body).toEqual({
      user: expect.objectContaining({
        id: expect.any(String),
        name: expect.any(String),
        address: expect.objectContaining({
          city: expect.any(String),
          country: expect.any(String)
        })
      }),
      metadata: expect.objectContaining({
        lastLogin: expect.any(String)
      })
    });
  });

  it('asserts error response format (RFC 7807)', async () => {
    const response = await request(app)
      .get('/api/users/999999')
      .expect(404);

    expect(response.body).toMatchObject({
      type: expect.stringContaining('not-found'),
      title: 'User Not Found',
      status: 404,
      detail: expect.any(String)
    });
  });
});
```

**Expected Output:**
```
PASS  test/response-assertions.test.js
  Response assertions
    ✓ asserts exact body match (22 ms)
    ✓ asserts partial body match with objectContaining (18 ms)
    ✓ asserts array length and contents (25 ms)
    ✓ asserts nested object structure (20 ms)
    ✓ asserts error response format (RFC 7807) (15 ms)

Tests: 5 passed, 5 total
```

**Why this output:** Each test uses a different assertion strategy. `toEqual` performs exact matching; `toMatchObject` allows partial matching. `arrayContaining` verifies that an array contains specific items without requiring exact order. The nested structure test uses `expect.any(String)` to verify types without hardcoding values. The RFC 7807 test verifies the error payload structure.

### Real-World Cases

- **API contract validation:** Verifying response bodies match the expected schema.
- **Error handling:** Verifying that errors follow a consistent format (RFC 7807).
- **Pagination:** Verifying that list responses include the correct number of items.
- **Data integrity:** Verifying that created resources are returned with all expected fields.

---

## Core Concept 4: Status-Code Assertions

### Definitions

**Core Definition:** Status-code assertions verify that the HTTP response status code matches the expected value for a given request.

**Technical Definition:** Supertest's `.expect(status)` method asserts that the response status code equals the provided integer. For programmatic assertions, `response.status` or `response.statusCode` can be checked with Jest's `expect().toBe()`. Status codes are the primary signal of request success or failure: 2xx for success, 3xx for redirects, 4xx for client errors, 5xx for server errors.

**Beginner-Friendly Explanation:** You check that `GET /users` returns 200, `POST /users` returns 201, `DELETE /users/1` returns 204, and `GET /users/999` returns 404. If the code is wrong, your test fails.

### Purposes

- To verify successful operations (200, 201, 204).
- To verify client errors (400, 401, 403, 404, 422, 429).
- To verify server errors (500).
- To catch regressions where a route starts returning the wrong status.

### Syntax Rules and Structure

```js
await request(app).get('/api/users').expect(200);
await request(app).post('/api/users').send(data).expect(201);
await request(app).delete('/api/users/1').expect(204);
await request(app).get('/api/missing').expect(404);
await request(app).post('/api/users').send({}).expect(422);
await request(app).post('/api/auth/login').send(badCreds).expect(401);
```

| Status | Meaning | Typical Use |
|--------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH. |
| 201 | Created | Successful POST. |
| 204 | No Content | Successful DELETE. |
| 400 | Bad Request | Malformed request. |
| 401 | Unauthorized | Missing/invalid auth. |
| 403 | Forbidden | Insufficient permissions. |
| 404 | Not Found | Resource doesn't exist. |
| 422 | Unprocessable Entity | Semantic validation error. |
| 429 | Too Many Requests | Rate limit exceeded. |
| 500 | Internal Server Error | Unhandled exception. |

**Rules:**
- Use `.expect(200)` for chained assertions.
- Use `expect(response.status).toBe(200)` for programmatic assertions.
- `response.statusCode` is an alias for `response.status`.
- Test both the happy path and error paths.

### Annotated Code Example

```js
// test/status-codes.test.js
const request = require('supertest');
const app = require('../app');

describe('Status code assertions', () => {
  it('returns 200 for successful GET', async () => {
    await request(app).get('/api/users').expect(200);
  });

  it('returns 201 for successful POST', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@example.com' })
      .expect(201);
  });

  it('returns 204 for successful DELETE', async () => {
    const create = await request(app)
      .post('/api/users')
      .send({ name: 'Temp', email: 'temp@example.com' });

    await request(app).delete(`/api/users/${create.body.id}`).expect(204);
  });

  it('returns 400 for malformed request', async () => {
    await request(app)
      .post('/api/users')
      .send({ name: 'Alice' })  // Missing email
      .expect(400);
  });

  it('returns 401 for missing authentication', async () => {
    await request(app).get('/api/protected').expect(401);
  });

  it('returns 403 for insufficient permissions', async () => {
    const userToken = jwt.sign({ id: 1, role: 'user' }, 'secret');
    await request(app)
      .get('/api/admin/users')
      .set('Authorization', `Bearer ${userToken}`)
      .expect(403);
  });

  it('returns 404 for non-existent resource', async () => {
    await request(app).get('/api/users/999999').expect(404);
  });

  it('returns 422 for validation error', async () => {
    await request(app)
      .post('/api/users')
      .send({ email: 'invalid-email' })
      .expect(422);
  });

  it('returns 500 for unhandled errors', async () => {
    const response = await request(app)
      .get('/api/error-prone')
      .expect(500);

    expect(response.body).toHaveProperty('error');
  });
});
```

**Expected Output:**
```
PASS  test/status-codes.test.js
  Status code assertions
    ✓ returns 200 for successful GET (18 ms)
    ✓ returns 201 for successful POST (25 ms)
    ✓ returns 204 for successful DELETE (32 ms)
    ✓ returns 400 for malformed request (15 ms)
    ✓ returns 401 for missing authentication (12 ms)
    ✓ returns 403 for insufficient permissions (14 ms)
    ✓ returns 404 for non-existent resource (10 ms)
    ✓ returns 422 for validation error (16 ms)
    ✓ returns 500 for unhandled errors (13 ms)

Tests: 9 passed, 9 total
```

**Why this output:** Each test asserts the exact status code the route should return for a specific scenario. The 204 test verifies no response body is returned. The 403 test uses a user-role token to verify RBAC enforcement. The 500 test verifies that unhandled errors are caught and formatted.

### Real-World Cases

- **REST API compliance:** Verifying that CRUD operations return the correct status codes.
- **Authentication:** Verifying 401 vs. 403 distinctions.
- **Rate limiting:** Verifying that the 6th request returns 429.

---

## Core Concept 5: Header Assertions

### Definitions

**Core Definition:** Header assertions verify that response headers — Content-Type, Cache-Control, security headers set by Helmet, CORS headers, and custom headers — have the expected values.

**Technical Definition:** Supertest's `.expect(headerName, valueOrRegex)` method asserts that a response header matches the expected string or regular expression. Headers can also be accessed via `response.headers` (lowercase keys) and asserted with Jest's matchers. Helmet sets 15 security headers including `Content-Security-Policy`, `X-Content-Type-Options`, `Strict-Transport-Security`, `Referrer-Policy`, and removes `X-Powered-By`. CORS middleware sets `Access-Control-Allow-Origin` and related headers.

**Beginner-Friendly Explanation:** You check that your API returns `Content-Type: application/json`, that `Cache-Control` is set correctly, and that Helmet's security headers are present. You also verify that CORS headers are set only for allowed origins.

### Purposes

- To verify Content-Type and Content-Length headers.
- To verify Cache-Control and caching directives.
- To verify Helmet security headers (CSP, HSTS, X-Content-Type-Options).
- To verify CORS headers for allowed origins.
- To verify custom headers (rate limit headers, request IDs).

### Syntax Rules and Structure

```js
// Chained assertions
await request(app)
  .get('/api/users')
  .expect('Content-Type', /application\/json/)
  .expect('Cache-Control', 'no-store');

// Direct assertions
const response = await request(app).get('/api/users');
expect(response.headers['content-type']).toMatch(/json/);
expect(response.headers['x-request-id']).toBeDefined();
```

| Header | Typical Assertion |
|--------|-------------------|
| `Content-Type` | `/application\/json/` |
| `Cache-Control` | `'no-store'` or `'public, max-age=3600'` |
| `X-Content-Type-Options` | `'nosniff'` |
| `Strict-Transport-Security` | `/max-age=\d+/` |
| `Content-Security-Policy` | `/default-src 'self'/` |
| `Access-Control-Allow-Origin` | `'https://example.com'` |
| `X-RateLimit-Limit` | `'100'` |

**Rules:**
- `.expect(header, string)` asserts exact match.
- `.expect(header, regex)` asserts pattern match.
- Header names in `response.headers` are lowercase.
- Helmet removes `X-Powered-By` — assert it is `undefined`.

### Annotated Code Example

```js
// test/header-assertions.test.js
const request = require('supertest');
const helmet = require('helmet');
const cors = require('cors');
const express = require('express');

const app = express();
app.use(helmet());
app.use(cors({ origin: ['https://myapp.com'] }));

app.get('/api/data', (req, res) => res.json({ data: 'value' }));
app.get('/api/cached', (req, res) => {
  res.set('Cache-Control', 'public, max-age=3600');
  res.json({ data: 'cached' });
});

describe('Header assertions', () => {
  describe('Standard headers', () => {
    it('asserts Content-Type', async () => {
      await request(app)
        .get('/api/data')
        .expect('Content-Type', /application\/json/);
    });

    it('asserts Cache-Control', async () => {
      await request(app)
        .get('/api/cached')
        .expect('Cache-Control', 'public, max-age=3600');
    });
  });

  describe('Helmet security headers', () => {
    it('sets Content-Security-Policy', async () => {
      const response = await request(app).get('/api/data');
      expect(response.headers['content-security-policy']).toContain("default-src 'self'");
    });

    it('sets X-Content-Type-Options: nosniff', async () => {
      const response = await request(app).get('/api/data');
      expect(response.headers['x-content-type-options']).toBe('nosniff');
    });

    it('sets Strict-Transport-Security', async () => {
      const response = await request(app).get('/api/data');
      expect(response.headers['strict-transport-security']).toMatch(/max-age=\d+/);
    });

    it('sets Referrer-Policy', async () => {
      const response = await request(app).get('/api/data');
      expect(response.headers['referrer-policy']).toBeDefined();
    });

    it('removes X-Powered-By', async () => {
      const response = await request(app).get('/api/data');
      expect(response.headers['x-powered-by']).toBeUndefined();
    });
  });

  describe('CORS headers', () => {
    it('sets CORS headers for allowed origin', async () => {
      const response = await request(app)
        .get('/api/data')
        .set('Origin', 'https://myapp.com');

      expect(response.headers['access-control-allow-origin']).toBe('https://myapp.com');
    });

    it('does not set CORS headers for disallowed origin', async () => {
      const response = await request(app)
        .get('/api/data')
        .set('Origin', 'https://evil.com');

      expect(response.headers['access-control-allow-origin']).toBeUndefined();
    });

    it('responds to preflight OPTIONS with correct headers', async () => {
      const response = await request(app)
        .options('/api/data')
        .set('Origin', 'https://myapp.com')
        .set('Access-Control-Request-Method', 'POST');

      expect(response.headers['access-control-allow-methods']).toContain('POST');
    });
  });

  describe('Custom headers', () => {
    it('returns custom rate limit headers', async () => {
      const response = await request(app)
        .get('/api/data')
        .set('X-RateLimit-Test', 'true');

      // In production, these come from express-rate-limit
      expect(response.headers).toBeDefined();
    });
  });
});
```

**Expected Output:**
```
PASS  test/header-assertions.test.js
  Header assertions
    Standard headers
      ✓ asserts Content-Type (12 ms)
      ✓ asserts Cache-Control (10 ms)
    Helmet security headers
      ✓ sets Content-Security-Policy (8 ms)
      ✓ sets X-Content-Type-Options: nosniff (7 ms)
      ✓ sets Strict-Transport-Security (7 ms)
      ✓ sets Referrer-Policy (6 ms)
      ✓ removes X-Powered-By (5 ms)
    CORS headers
      ✓ sets CORS headers for allowed origin (9 ms)
      ✓ does not set CORS headers for disallowed origin (8 ms)
      ✓ responds to preflight OPTIONS with correct headers (7 ms)
    Custom headers
      ✓ returns custom rate limit headers (6 ms)

Tests: 12 passed, 12 total
```

**Why this output:** The tests verify that Helmet sets the correct security headers and removes `X-Powered-By`. The CORS tests verify that the `Access-Control-Allow-Origin` header is set only for allowed origins. Preflight OPTIONS requests are verified to return the correct `Access-Control-Allow-Methods` header.

### Real-World Cases

- **Security hardening:** Verifying that Helmet is configured correctly.
- **CORS configuration:** Verifying that only allowed origins receive CORS headers.
- **Caching:** Verifying that Cache-Control headers are set for CDN optimisation.
- **Rate limiting:** Verifying that `RateLimit-*` headers are present.

---

## Core Concept 6: Cookie and Session State Assertions

### Definitions

**Core Definition:** Cookie and session assertions verify that the server sets the correct cookies (e.g., HttpOnly, Secure, SameSite) and that session state persists across requests when using a cookie-aware agent.

**Technical Definition:** Supertest provides `request.agent(app)` to create a persistent agent that stores and sends cookies across requests, simulating a browser session. The `supertest-session` library wraps Supertest with a session object that exposes cookies via `session.cookies`. Cookies can be asserted from the response via `response.headers['set-cookie']` or by inspecting the agent's cookie jar. Session persistence is verified by making a request that sets a cookie, then a subsequent request that requires that cookie.

**Beginner-Friendly Explanation:** You test that logging in sets a session cookie, then use that cookie to access a protected page. You also test that cookies have the correct flags (HttpOnly, Secure, SameSite).

### Purposes

- To verify that login sets a session cookie.
- To verify cookie attributes (HttpOnly, Secure, SameSite).
- To verify that cookies persist across requests using an agent.
- To test session-based authentication flows.
- To test cookie clearing on logout.

### Syntax Rules and Structure

**Supertest agent (persistent cookies):**
```js
const agent = request.agent(app);

// Login sets a cookie
await agent.post('/api/auth/login')
  .send({ email: 'user@example.com', password: 'password123' })
  .expect(200);

// Subsequent request uses the cookie automatically
await agent.get('/api/profile').expect(200);
```

**supertest-session:**
```js
const session = require('supertest-session');
const testSession = session(app);

await testSession.post('/api/auth/login')
  .send({ username: 'foo', password: 'password' })
  .expect(200);

// Access cookies from the session
const cookie = testSession.cookies.find(c => c.name === 'connect.sid');
```

**Asserting cookie attributes:**
```js
const response = await request(app)
  .post('/api/auth/login')
  .send(validCredentials);

const cookies = response.headers['set-cookie'];
expect(cookies[0]).toContain('HttpOnly');
expect(cookies[0]).toContain('Secure');
expect(cookies[0]).toContain('SameSite=Strict');
```

| Method | Purpose |
|--------|---------|
| `request.agent(app)` | Create a cookie-aware agent. |
| `session(app)` | Create a supertest-session instance. |
| `response.headers['set-cookie']` | Access raw Set-Cookie headers. |
| `session.cookies` | Access cookies from the session. |

**Rules:**
- `request.agent(app)` maintains cookies automatically.
- `supertest-session` is useful for custom session strategies.
- Cookie attributes can be asserted from the raw `Set-Cookie` header.
- Test both cookie setting (login) and cookie clearing (logout).

### Annotated Code Example

```js
// test/cookie-session.test.js
const request = require('supertest');
const app = require('../app');

describe('Cookie and session assertions', () => {
  describe('Cookie attributes', () => {
    it('sets HttpOnly, Secure, SameSite cookie on login', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'password123' })
        .expect(200);

      const cookies = response.headers['set-cookie'];
      expect(cookies).toBeDefined();
      expect(cookies[0]).toContain('HttpOnly');
      expect(cookies[0]).toContain('SameSite=Strict');
    });
  });

  describe('Session persistence with agent', () => {
    let agent;

    beforeEach(() => {
      agent = request.agent(app);
    });

    it('persists session across requests', async () => {
      // Login — cookie is stored in the agent
      await agent
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'password123' })
        .expect(200);

      // Subsequent request uses the cookie automatically
      const response = await agent
        .get('/api/profile')
        .expect(200);

      expect(response.body.email).toBe('alice@example.com');
    });

    it('rejects access without session cookie', async () => {
      await request(app)
        .get('/api/profile')
        .expect(401);
    });

    it('clears session cookie on logout', async () => {
      await agent
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'password123' })
        .expect(200);

      await agent.post('/api/auth/logout').expect(204);

      // Session cookie should be cleared
      await agent.get('/api/profile').expect(401);
    });
  });

  describe('Manual cookie handling', () => {
    it('sends a manually captured cookie', async () => {
      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({ email: 'alice@example.com', password: 'password123' })
        .expect(200);

      const cookies = loginResponse.headers['set-cookie'];

      const profileResponse = await request(app)
        .get('/api/profile')
        .set('Cookie', cookies)
        .expect(200);

      expect(profileResponse.body.email).toBe('alice@example.com');
    });
  });
});
```

**Expected Output:**
```
PASS  test/cookie-session.test.js
  Cookie and session assertions
    Cookie attributes
      ✓ sets HttpOnly, Secure, SameSite cookie on login (28 ms)
    Session persistence with agent
      ✓ persists session across requests (35 ms)
      ✓ rejects access without session cookie (12 ms)
      ✓ clears session cookie on logout (42 ms)
    Manual cookie handling
      ✓ sends a manually captured cookie (30 ms)

Tests: 5 passed, 5 total
```

**Why this output:** The cookie attribute test verifies that the login response sets a cookie with `HttpOnly` and `SameSite=Strict`. The agent test verifies that the session persists across requests. The logout test verifies that the session cookie is cleared. The manual test demonstrates extracting the cookie from the login response and sending it on a subsequent request.

### Real-World Cases

- **Session-based authentication:** Testing login, session persistence, and logout.
- **CSRF protection:** Verifying that CSRF cookies are set.
- **Cookie security:** Verifying HttpOnly, Secure, and SameSite flags.
- **Multi-step flows:** Testing forms that span multiple requests.

---

## Core Concept 7: Streamed Response Assertions

### Definitions

**Core Definition:** Streamed response assertions verify the content of responses that are delivered incrementally — such as Server-Sent Events (SSE) or file streaming — by consuming the response stream chunk by chunk.

**Technical Definition:** Supertest's `.expect()` assertions operate on the complete response, which is unsuitable for streams that never end (SSE) or deliver large files. For streaming responses, use the raw `http` module or Supertest's `.end()` callback to access the response stream, then listen for `data` events and accumulate chunks. For SSE, the response uses `Content-Type: text/event-stream` and each event is a `data:` line. For file streaming, the response body is a binary stream that can be buffered and asserted.

**Beginner-Friendly Explanation:** Normal Supertest assertions wait for the entire response. But some responses never end (like live updates) or are too large to buffer. For these, you read the response as it arrives — chunk by chunk — and check the contents.

### Purposes

- To test Server-Sent Events (SSE) routes.
- To test file streaming (downloads) with binary content.
- To verify that streaming responses have the correct Content-Type and headers.
- To assert on individual chunks or the accumulated stream content.

### Syntax Rules and Structure

**SSE test pattern:**
```js
const http = require('http');
const app = require('../app');

it('streams SSE events', (done) => {
  const server = app.listen(0, () => {
    const port = server.address().port;

    http.get(`http://localhost:${port}/api/events`, (res) => {
      expect(res.headers['content-type']).toBe('text/event-stream');
      expect(res.headers['cache-control']).toBe('no-cache');

      const chunks = [];
      res.on('data', (chunk) => {
        chunks.push(chunk.toString());
        if (chunks.length === 3) {
          res.destroy();
          server.close();
          expect(chunks[0]).toContain('data:');
          done();
        }
      });
    });
  });
});
```

**File streaming test pattern:**
```js
const response = await request(app)
  .get('/api/download/report.pdf')
  .responseType('blob');

expect(response.body).toBeInstanceOf(Buffer);
expect(response.body.length).toBeGreaterThan(0);
```

| Method | Purpose |
|--------|---------|
| `.responseType('blob')` | Buffer the response body as a `Buffer`. |
| `res.on('data', chunk => {})` | Listen for streaming chunks. |
| `res.headers['content-type']` | Check the Content-Type header. |

**Rules:**
- Supertest's `.expect()` cannot be used for infinite streams (SSE).
- For SSE, use the raw `http` module or Supertest's `.end()` callback to access the response stream.
- For file streaming, `.responseType('blob')` buffers the binary body into a `Buffer`.
- Close the server after the test to prevent Jest from hanging.

### Annotated Code Example

```js
// test/streamed-responses.test.js
const request = require('supertest');
const http = require('http');
const express = require('express');

const app = express();

// SSE route
app.get('/api/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  let counter = 0;
  const interval = setInterval(() => {
    counter++;
    res.write(`data: ${JSON.stringify({ event: 'tick', count: counter })}\n\n`);
    if (counter >= 3) {
      clearInterval(interval);
      res.end();
    }
  }, 50);

  req.on('close', () => clearInterval(interval));
});

// File streaming route
app.get('/api/download', (req, res) => {
  res.set('Content-Type', 'application/octet-stream');
  res.set('Content-Disposition', 'attachment; filename="test.txt"');
  res.write('chunk-1');
  res.write('chunk-2');
  res.end('chunk-3');
});

describe('Streamed response assertions', () => {
  describe('Server-Sent Events (SSE)', () => {
    it('streams events with correct headers', (done) => {
      const server = app.listen(0, () => {
        const port = server.address().port;
        const chunks = [];

        http.get(`http://localhost:${port}/api/events`, (res) => {
          expect(res.headers['content-type']).toBe('text/event-stream');
          expect(res.headers['cache-control']).toBe('no-cache');

          res.on('data', (chunk) => {
            chunks.push(chunk.toString());

            if (chunks.length === 3) {
              res.destroy();
              server.close();

              expect(chunks[0]).toContain('"event":"tick"');
              expect(chunks[0]).toContain('"count":1');
              expect(chunks[1]).toContain('"count":2');
              expect(chunks[2]).toContain('"count":3');
              done();
            }
          });
        });
      });
    });
  });

  describe('File streaming', () => {
    it('streams file content with correct headers', async () => {
      const response = await request(app)
        .get('/api/download')
        .responseType('blob')
        .expect('Content-Type', 'application/octet-stream')
        .expect('Content-Disposition', /attachment/);

      expect(response.body).toBeInstanceOf(Buffer);
      expect(response.body.toString()).toBe('chunk-1chunk-2chunk-3');
    });
  });
});
```

**Expected Output:**
```
PASS  test/streamed-responses.test.js
  Streamed response assertions
    Server-Sent Events (SSE)
      ✓ streams events with correct headers (85 ms)
    File streaming
      ✓ streams file content with correct headers (15 ms)

Tests: 2 passed, 2 total
```

**Why this output:** The SSE test uses the raw `http` module to access the streaming response. It accumulates chunks and asserts on the content of each event. The server is closed after the test to prevent Jest from hanging. The file streaming test uses `.responseType('blob')` to buffer the binary response and asserts on the accumulated content.

### Real-World Cases

- **Real-time dashboards:** Testing SSE routes that push live updates.
- **File downloads:** Testing that download endpoints return the correct binary content.
- **Video/audio streaming:** Testing that streaming endpoints return chunks with the correct headers.
- **Log streaming:** Testing that log tail endpoints stream the latest entries.

---

## References

- Supertest GitHub Repository — https://github.com/ladjs/supertest
- Supertest API Documentation — https://github.com/ladjs/supertest
- SuperTest Node.js API Testing Complete Guide 2026 — https://qaskills.sh/blog/supertest-node-api-testing-complete-guide
- claude-plugins: API Testing Reference — https://github.com/laurigates/claude-plugins/blob/main/api-plugin/skills/api-testing/REFERENCE.md
- agent-skills: Assertion Patterns — https://github.com/oakoss/agent-skills/blob/85e3a3919d9e0ec7f7302a5143ec4b3e66f5f6ad/skills/api-testing/references/assertion-patterns.md
- Supertest Header Assertions (Helmet) — https://github.com/alexvervloet/learn-javascript-backend-engineering/commit/9646b8ad6dbf4131d48fff3d9050a65cf40e0e48
- supertest-session on npm — https://www.npmjs.com/package/supertest-session
- How to test a Server Sent Events (SSE) route in NodeJS — https://stackoverflow.com/feeds/question/59936895
- How to test image upload (stream) with supertest and jest — https://stackoverflow.com/questions/49416514
- Supertest: Received stream is empty — https://stackoverflow.com/questions/64856865
- Express Security Best Practices (Helmet) — https://expressjs.com/en/advanced/best-practice-security.html
- Helmet.js Documentation — https://helmetjs.github.io/
- Jest Documentation — https://jestjs.io/docs/getting-started