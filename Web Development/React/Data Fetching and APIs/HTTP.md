# HTTP Fundamentals for React — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP (HyperText Transfer Protocol) is the request–response protocol that React applications use to communicate with servers — fetching data, submitting forms, and mutating resources.

**Technical Definition:** HTTP is a stateless, application-layer protocol defined by the IETF (RFC 9110–9114). A client (the browser running React) sends a **request** consisting of a method, a target URI, headers, and an optional body; the server returns a **response** consisting of a status code, headers, and an optional body. React itself has no HTTP client — it relies on the browser's `fetch` API (or libraries such as Axios) inside effects, event handlers, route loaders, or Server Components. Correct use of HTTP methods, status codes, headers, CORS, and credentials determines whether data flows securely and whether caches, proxies, and browsers behave predictably.

**Beginner-Friendly Explanation:** HTTP is the language your React app speaks to the server. Every time you load data, submit a form, or delete an item, your app sends a message (a request) and gets a reply (a response). The message has a verb (GET, POST, etc.), a destination (URL), some metadata (headers), and sometimes a package of data (body). The reply has a number (status code) that says what happened — 200 means “here you go,” 404 means “not found,” 500 means “the server broke.” Getting these details right is what separates an app that works reliably and securely from one that mysteriously fails in production.

### Key Characteristics

- **Stateless:** Each request is independent; state must be carried by tokens, cookies, or identifiers.
- **Method semantics matter:** GET is safe and idempotent; POST is neither; PUT and DELETE are idempotent; PATCH is usually not.
- **Status codes are a contract:** Clients (and React error handling) branch on status codes, not on response text.
- **Headers carry metadata:** Content negotiation, authentication, caching, and CORS all live in headers.
- **CORS is a browser security feature:** The server must explicitly opt in to cross-origin requests.
- **Credentials are opt-in:** Cookies and auth headers are not sent cross-origin unless explicitly configured.
- **JSON is the default payload format:** It is human-readable, universally supported, and maps naturally to JavaScript objects.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- JavaScript Promises and `async`/`await`.
- Basic understanding of URLs, domains, and origins.
- Familiarity with `fetch` or an HTTP client library.
- Basic knowledge of browser developer tools (Network tab).

### Related Programming Areas

- **Data Fetching** — the React-side lifecycle that consumes HTTP.
- **Authentication** — cookies, tokens, and credential handling.
- **Caching** — HTTP cache headers and conditional requests.
- **Security** — CORS, CSRF, and content security.
- **API Design** — REST semantics and resource modelling.

---

## 1. GET

### Definitions

**Core Definition:** GET is the HTTP method used to retrieve a representation of a resource without modifying it.

**Technical Definition:** GET requests a representation of the target resource. It is defined as **safe** (no side effects on the server) and **idempotent** (multiple identical requests have the same effect as one). GET requests should not carry a request body; parameters are sent in the query string. Responses to GET are cacheable by default and may be stored by browsers, proxies, and CDNs according to `Cache-Control`, `ETag`, and related headers.

**Beginner-Friendly Explanation:** GET is how you ask the server for something — “give me the list of products,” “give me user 42.” It never changes anything on the server, which is why browsers and caches are allowed to store the answer.

### Purposes

- To retrieve a resource or collection from the server.
- To read data without causing side effects.
- To leverage browser and CDN caching for repeated reads.
- To support shareable, bookmarkable URLs via query parameters.
- To perform conditional requests using `ETag`/`If-None-Match` to save bandwidth.

### Syntax Rules and Structure

**General Syntax:**

```js
fetch(url, {
  method: 'GET', // default; can be omitted
  headers: { Accept: 'application/json' },
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `url` | Target resource including query string parameters |
| `method: 'GET'` | The default method; explicit for clarity |
| `headers` | Metadata such as `Accept` and `Authorization` |
| `body` | Must not be present for GET |

**Syntax Rules:**

1. Parameters go in the query string (`?page=2&sort=asc`), not the body.
2. Use `Accept` to request a specific representation (`application/json`).
3. Cacheable by default; control with `Cache-Control` and `ETag`.
4. Conditional GETs use `If-None-Match` or `If-Modified-Since` and may return `304 Not Modified`.

**Constraints and Limitations:**

- URLs are logged, stored in history, and visible in the address bar — never put secrets in query strings.
- URL length limits vary by browser and server (practically a few thousand characters).
- GET requests cannot carry a request body reliably (semantics forbid it).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Fetching a List in React

```jsx
import { useEffect, useState } from 'react';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [isPending, setIsPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    fetch('/api/products?page=1&sort=price', {
      method: 'GET',
      headers: { Accept: 'application/json' },
      signal: controller.signal,
    })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(setProducts)
      .catch((err) => {
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => setIsPending(false));

    return () => controller.abort();
  }, []);

  if (isPending) return <p>Loading…</p>;
  if (error) return <p role="alert">Failed to load products.</p>;
  return <ul>{products.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**Expected Output:** The list renders after the GET completes. Query parameters `page=1&sort=price` are sent to the server. Navigating away aborts the request.

**Why this output:** GET is the default method; parameters are in the query string. `res.ok` distinguishes HTTP success from errors. `AbortController` cancels the request on unmount.

#### Example 2: Conditional GET with ETag

```jsx
async function fetchWithEtag(url, etag) {
  const res = await fetch(url, {
    headers: etag ? { 'If-None-Match': etag } : {},
  });

  if (res.status === 304) {
    return { notModified: true };
  }

  return {
    data: await res.json(),
    etag: res.headers.get('ETag'),
  };
}
```

**Expected Output:** If the resource has not changed, the server returns `304 Not Modified` with no body, saving bandwidth. Otherwise, the new data and ETag are returned.

**Why this output:** `If-None-Match` sends the cached ETag. A `304` tells the client to reuse its cached copy. `res.headers.get('ETag')` captures the new validator for the next request.

### Real-World Cases

- **Product listings:** `/api/products?category=shoes&page=2`.
- **User profiles:** `/api/users/42`.
- **Search:** `/api/search?q=react&limit=20`.
- **Conditional polling:** ETag-based refresh to avoid re-downloading unchanged data.

### References

- MDN — HTTP request methods: GET — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/GET
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- MDN — Conditional requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests

---

## 2. POST

### Definitions

**Core Definition:** POST is the HTTP method used to submit data to the server, typically to create a new resource or trigger processing.

**Technical Definition:** POST sends a representation to the target resource, asking the server to process it. POST is neither **safe** nor **idempotent**: repeating a POST can create duplicate resources or trigger duplicate side effects. The request body carries the payload (commonly JSON or `multipart/form-data`). A successful POST commonly returns `201 Created` with a `Location` header pointing to the new resource.

**Beginner-Friendly Explanation:** POST is how you send new data to the server — “here is a new order, please create it.” Unlike GET, POST changes things, and repeating it can create duplicates.

### Purposes

- To create new resources (users, orders, posts).
- To submit form data and trigger server-side processing.
- To send complex payloads (JSON, files) that do not fit in a URL.
- To trigger non-idempotent operations where duplication must be prevented.
- To authenticate (login endpoints typically use POST).

### Syntax Rules and Structure

**General Syntax:**

```js
fetch('/api/products', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Shoes', price: 99 }),
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `method: 'POST'` | The request method |
| `Content-Type` | Declares the body format (`application/json`) |
| `body` | The serialized payload |
| `Location` (response) | Points to the created resource |

**Syntax Rules:**

1. Always set `Content-Type` to match the body format.
2. Serialize objects with `JSON.stringify` when sending JSON.
3. Expect `201 Created` and a `Location` header for resource creation.
4. Use `Idempotency-Key` headers for payment or order endpoints where duplicates are dangerous.

**Constraints and Limitations:**

- Not idempotent: retries can create duplicates.
- Not cacheable by default.
- CSRF protection is required when using cookie-based auth.

### Annotated Code Examples

```tsx
async function createProduct(input) {
  const res = await fetch('/api/products', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(input),
  });

  if (!res.ok) {
    const problem = await res.json().catch(() => ({}));
    throw new Error(problem.message ?? `HTTP ${res.status}`);
  }

  return res.json();
}
```

**Expected Output:** A successful creation returns `201 Created` with the new resource JSON. Validation failures return `400` or `422` with an error body.

**Why this output:** POST with `Content-Type: application/json` and a JSON body creates the resource. `res.ok` checks for success; the error path parses the server's error message.

### Real-World Cases

- **Checkout:** Submitting an order with line items and payment details.
- **Signup/login:** Sending credentials for authentication.
- **File uploads:** Sending `multipart/form-data` to an upload endpoint.
- **Webhooks:** Third-party services POSTing events to your server.

### References

- MDN — HTTP request methods: POST — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- MDN — POST — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST

---

## 3. PUT

### Definitions

**Core Definition:** PUT is the HTTP method used to replace an existing resource entirely with the provided representation.

**Technical Definition:** PUT sends a complete representation of the target resource. If the resource exists, it is replaced; if it does not exist, the server may create it at the given URI. PUT is **idempotent**: repeating the same request produces the same result. Unlike PATCH, PUT replaces the entire resource — omitted fields are typically removed or reset, not preserved.

**Beginner-Friendly Explanation:** PUT means “replace this whole thing with what I am sending.” If you leave a field out, it may be erased. Repeating the same PUT has the same effect as sending it once.

### Purposes

- To replace an entire resource with a known complete representation.
- To perform idempotent updates that are safe to retry.
- To create a resource at a client-specified URI when supported.
- To support offline-first sync where the full object is known.
- To simplify conflict handling with `ETag`/`If-Match` preconditions.

### Syntax Rules and Structure

```js
fetch('/api/users/42', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'If-Match': etag,
  },
  body: JSON.stringify({ id: 42, name: 'Ada', email: 'ada@example.com' }),
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `method: 'PUT'` | Replace semantics |
| `If-Match` | Optional precondition to prevent lost updates |
| `body` | The complete replacement representation |

**Syntax Rules:**

1. Send the complete resource, not a partial patch.
2. Use `If-Match` with an ETag for optimistic concurrency.
3. PUT is idempotent; retries are safe.
4. Return `200` or `204` on success; `201` if a resource was created.

**Constraints and Limitations:**

- Partial fields may be erased if omitted; use PATCH for partial updates.
- Not all APIs support client-specified URIs for creation.

### Annotated Code Examples

```tsx
async function replaceUser(user) {
  const res = await fetch(`/api/users/${user.id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(user),
  });

  if (res.status === 409) throw new Error('Conflict: resource changed');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.status === 204 ? null : res.json();
}
```

**Expected Output:** The full user object is replaced. A `409 Conflict` signals that the resource changed on the server since the client read it.

**Why this output:** PUT replaces the whole resource. The `409` branch handles optimistic concurrency conflicts.

### Real-World Cases

- **Profile replacement:** Saving a complete profile form.
- **Settings sync:** Replacing a configuration object.
- **Offline sync:** Pushing a fully known object after reconnect.

### References

- MDN — HTTP request methods: PUT — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PUT
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- MDN — Conditional requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests

---

## 4. PATCH

### Definitions

**Core Definition:** PATCH is the HTTP method used to apply partial modifications to a resource.

**Technical Definition:** PATCH sends a set of changes rather than a complete replacement. The request body format is defined by the server and indicated by the `Content-Type`. Common formats are **JSON Merge Patch** (`application/merge-patch+json`, RFC 7396) and **JSON Patch** (`application/json-patch+json`, RFC 6902). PATCH is not guaranteed to be idempotent: applying the same patch twice may produce a different result depending on the operation (e.g., incrementing a counter).

**Beginner-Friendly Explanation:** PATCH means “change just these fields.” You send only the parts that changed, not the whole object. Whether repeating it is safe depends on what the patch does.

### Purposes

- To update individual fields without sending the whole resource.
- To reduce payload size for large resources.
- To support fine-grained, atomic modifications (JSON Patch).
- To express explicit operations (add, remove, replace) in JSON Patch.
- To support merge semantics where `null` deletes a field (JSON Merge Patch).

### Syntax Rules and Structure

**JSON Merge Patch (RFC 7396):**

```js
fetch('/api/users/42', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/merge-patch+json' },
  body: JSON.stringify({ name: 'Ada Lovelace' }), // only changed fields
});
```

**JSON Patch (RFC 6902):**

```js
fetch('/api/users/42', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json-patch+json' },
  body: JSON.stringify([
    { op: 'replace', path: '/name', value: 'Ada Lovelace' },
  ]),
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `method: 'PATCH'` | Partial update semantics |
| `Content-Type` | `merge-patch+json` or `json-patch+json` |
| `body` | Changed fields or operation list |

**Syntax Rules:**

1. Use `application/merge-patch+json` for simple field merges.
2. Use `application/json-patch+json` for explicit operations.
3. Do not assume idempotency; design patches carefully for retries.
4. Use `If-Match` for optimistic concurrency when needed.

**Constraints and Limitations:**

- Not guaranteed idempotent.
- Semantics depend on the server's chosen patch format.
- Some proxies and clients historically mishandled PATCH; verify support.

### Annotated Code Examples

```tsx
async function renameUser(id, name) {
  const res = await fetch(`/api/users/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/merge-patch+json' },
    body: JSON.stringify({ name }),
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

**Expected Output:** Only the `name` field is updated; other fields remain unchanged.

**Why this output:** JSON Merge Patch applies the provided fields to the existing resource. Omitted fields are left untouched.

### Real-World Cases

- **Inline editing:** Updating a single field in a table row.
- **Preference toggles:** Changing one setting without sending the whole settings object.
- **Order status updates:** Patching `status` without resending the full order.

### References

- MDN — HTTP request methods: PATCH — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PATCH
- RFC 7396 — JSON Merge Patch — https://www.rfc-editor.org/rfc/rfc7396
- RFC 6902 — JSON Patch — https://www.rfc-editor.org/rfc/rfc6902

---

## 5. DELETE

### Definitions

**Core Definition:** DELETE is the HTTP method used to remove a resource from the server.

**Technical Definition:** DELETE requests the removal of the target resource. It is **idempotent**: deleting a resource twice leaves the server in the same state (the resource is gone). A successful DELETE commonly returns `204 No Content` or `200 OK` with a body describing the result. After deletion, the resource's URI should no longer resolve.

**Beginner-Friendly Explanation:** DELETE is how you remove something — “delete this comment.” Deleting it twice has the same end result: it is gone.

### Purposes

- To remove a resource permanently.
- To perform idempotent deletions that are safe to retry.
- To support soft-delete APIs that mark resources as removed.
- To clean up dependent data through cascading server logic.
- To return a confirmation body when the client needs details.

### Syntax Rules and Structure

```js
fetch('/api/posts/42', {
  method: 'DELETE',
  headers: { Authorization: 'Bearer ' + token },
});
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `method: 'DELETE'` | Removal semantics |
| `Authorization` | Auth header if required |
| Body | Usually absent; some APIs allow a body for batch deletes |

**Syntax Rules:**

1. DELETE should be idempotent; a second DELETE of the same URI typically returns `404` or `204`.
2. Return `204 No Content` when there is nothing to return.
3. Use `Authorization` to protect delete endpoints.
4. Confirm destructive actions in the UI before sending the request.

**Constraints and Limitations:**

- Destructive and irreversible unless the server implements soft delete.
- Some servers reject DELETE with a body; avoid it unless documented.
- Caches must invalidate the deleted resource.

### Annotated Code Examples

```tsx
async function deletePost(id) {
  const res = await fetch(`/api/posts/${id}`, { method: 'DELETE' });

  if (res.status === 204 || res.status === 200) return;
  if (res.status === 404) return; // Already gone
  throw new Error(`HTTP ${res.status}`);
}
```

**Expected Output:** The post is removed; the server responds with `204`. A repeat delete returns `404`, which the client treats as success (already gone).

**Why this output:** DELETE is idempotent. Treating `404` as success reflects the desired end state: the resource no longer exists.

### Real-World Cases

- **Comment deletion:** Removing a user's comment.
- **Account deletion:** Removing a user account and related data.
- **Cart item removal:** Deleting a line item from a cart.

### References

- MDN — HTTP request methods: DELETE — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/DELETE
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110

---

## 6. HTTP Status Codes

### Definitions

**Core Definition:** HTTP status codes are three-digit numbers in the response that indicate the outcome of the request.

**Technical Definition:** Status codes are grouped into five classes: **1xx Informational**, **2xx Success**, **3xx Redirection**, **4xx Client Error**, and **5xx Server Error**. The first digit defines the class; the remaining digits identify the specific outcome. React applications branch on status codes to decide whether to render data, show an error, redirect, or retry. Status codes are semantic contracts, not decoration — using the right code lets browsers, caches, and clients behave correctly.

**Beginner-Friendly Explanation:** Status codes are the server's way of saying what happened. `200` means “OK,” `404` means “not found,” `500` means “the server broke.” Your React code checks the number and decides what to show.

### Purposes

- To communicate the outcome of a request unambiguously.
- To let clients branch between success, redirect, and failure paths.
- To enable correct caching behaviour (`304`, `200`).
- To drive retry logic (e.g., retry `5xx`, not `4xx`).
- To support accessibility by rendering appropriate error messages.

### Syntax Rules and Structure

**Common Status Codes:**

| Code | Meaning | React Handling |
|------|---------|----------------|
| `200 OK` | Success with body | Parse and render |
| `201 Created` | Resource created | Use `Location` header |
| `204 No Content` | Success without body | Do not parse |
| `301/308` | Permanent redirect | Usually handled by the browser |
| `304 Not Modified` | Cached copy valid | Use cached data |
| `400 Bad Request` | Malformed request | Show validation error |
| `401 Unauthorized` | Not authenticated | Redirect to login |
| `403 Forbidden` | Authenticated but not allowed | Show forbidden message |
| `404 Not Found` | Resource missing | Show not-found UI |
| `409 Conflict` | State conflict | Show conflict resolution UI |
| `422 Unprocessable` | Validation failure | Map errors to fields |
| `429 Too Many Requests` | Rate limited | Back off and retry |
| `500 Internal Server Error` | Server failure | Show generic error + log |
| `503 Service Unavailable` | Temporarily unavailable | Retry with backoff |

**Syntax Rules:**

1. Check `res.status` or `res.ok` before parsing.
2. Handle `204` and `304` without parsing a body.
3. Distinguish `401` (authenticate) from `403` (authorise).
4. Retry `5xx` and `429`; do not retry `4xx` validation errors.
5. Surface server-provided error messages for `400`, `409`, and `422`.

**Constraints and Limitations:**

- Status codes alone do not describe the error; read the body when available.
- Some APIs return `200` with an error payload; validate the body shape.
- Proxies and CDNs may rewrite or cache status codes unexpectedly.

### Annotated Code Examples

```tsx
async function request(url, options) {
  const res = await fetch(url, options);

  if (res.status === 204 || res.status === 304) return { data: null, res };

  const body = await res.json().catch(() => null);

  if (res.ok) return { data: body, res };

  switch (res.status) {
    case 401:
      throw new AuthError('Please log in again');
    case 403:
      throw new ForbiddenError('You do not have permission');
    case 404:
      throw new NotFoundError('Resource not found');
    case 422:
      throw new ValidationError(body?.errors ?? {});
    default:
      throw new HttpError(res.status, body?.message);
  }
}
```

**Expected Output:** The function returns data on `2xx`, returns `null` for `204`/`304`, and throws typed errors for `401`, `403`, `404`, `422`, and other failures.

**Why this output:** Checking status before parsing prevents crashes on empty bodies. Typed errors let React render specific UI for each failure class.

### Real-World Cases

- **Auth flows:** `401` triggers a login redirect.
- **Form validation:** `422` maps errors to fields.
- **Rate limiting:** `429` triggers backoff.
- **Caching:** `304` reuses cached data.

### References

- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- IANA — HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/

---

## 7. Headers

### Definitions

**Core Definition:** HTTP headers are key-value metadata sent with requests and responses that describe the message, control caching, negotiate content, and carry authentication.

**Technical Definition:** Headers are case-insensitive field names paired with values. They are grouped into **request headers** (sent by the client), **response headers** (sent by the server), **representation headers** (describe the body), and **general headers**. Key categories include content negotiation (`Accept`, `Content-Type`), authentication (`Authorization`, `WWW-Authenticate`), caching (`Cache-Control`, `ETag`, `If-None-Match`), CORS (`Access-Control-Allow-Origin`), and custom headers. Headers are the primary mechanism for controlling how a request and response are processed by intermediaries.

**Beginner-Friendly Explanation:** Headers are the labels on the envelope. They say what is inside (`Content-Type`), who is allowed to open it (`Authorization`), and whether it can be reused (`Cache-Control`).

### Purposes

- To declare the format of request and response bodies.
- To authenticate and authorise requests.
- To control caching and conditional requests.
- To enable CORS for cross-origin access.
- To pass custom application metadata (e.g., request IDs, feature flags).
- To negotiate content types and languages.

### Syntax Rules and Structure

**Common Headers:**

| Header | Direction | Purpose |
|--------|-----------|---------|
| `Accept` | Request | Preferred response format |
| `Content-Type` | Both | Body format |
| `Authorization` | Request | Credentials (Bearer token, Basic) |
| `Cache-Control` | Both | Caching directives |
| `ETag` | Response | Cache validator |
| `If-None-Match` | Request | Conditional GET |
| `If-Match` | Request | Optimistic concurrency |
| `Access-Control-Allow-Origin` | Response | CORS allowlist |
| `X-Request-Id` | Both | Tracing |

**Syntax Rules:**

1. Header names are case-insensitive; values are case-sensitive.
2. Multiple values for the same header can be comma-separated or repeated.
3. `Content-Type` must match the body format (`application/json`, `multipart/form-data`).
4. `Authorization` is not sent cross-origin unless the request is configured to include credentials.
5. Custom headers often require CORS preflight approval.

**Constraints and Limitations:**

- Browsers restrict which headers JavaScript can read (`Set-Cookie`, etc.).
- Some headers are forbidden in `fetch` (`Host`, `Content-Length`).
- Header size limits vary by server and proxy.

### Annotated Code Examples

```tsx
async function fetchWithHeaders(url) {
  const res = await fetch(url, {
    headers: {
      Accept: 'application/json',
      'X-Request-Id': crypto.randomUUID(),
    },
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);

  const requestId = res.headers.get('X-Request-Id');
  console.log('Request ID:', requestId);
  return res.json();
}
```

**Expected Output:** The request includes `Accept` and a tracing ID. The response's `X-Request-Id` is read and logged.

**Why this output:** Request headers declare expectations; response headers expose server metadata that JavaScript is allowed to read.

### Real-World Cases

- **API clients:** Sending `Accept` and `Authorization` on every request.
- **Tracing:** Correlating requests with `X-Request-Id`.
- **Caching:** Using `ETag` and `If-None-Match` to avoid re-downloading.

### References

- MDN — HTTP headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110

---

## 8. JSON

### Definitions

**Core Definition:** JSON (JavaScript Object Notation) is a text-based data-interchange format that represents objects, arrays, strings, numbers, booleans, and null.

**Technical Definition:** JSON is defined by RFC 8259. It supports six value types: object, array, string, number, boolean, and `null`. It does not support `undefined`, functions, `Date` objects, `Map`, `Set`, or comments. In HTTP, JSON is identified by `Content-Type: application/json` and is serialized with `JSON.stringify` and parsed with `JSON.parse`. Dates become ISO strings; `undefined` properties are omitted; circular references throw.

**Beginner-Friendly Explanation:** JSON is a text format for data that looks like JavaScript objects. When you send data to a server, you convert it to a JSON string; when you receive data, you parse it back into objects.

### Purposes

- To serialize structured data for transport over HTTP.
- To provide a human-readable, language-agnostic payload format.
- To map naturally to JavaScript objects and arrays.
- To integrate with APIs, databases, and configuration files.
- To validate payload shape with schemas (Zod, JSON Schema).

### Syntax Rules and Structure

```js
const payload = { name: 'Ada', age: 36, active: true, tags: ['math'] };

const json = JSON.stringify(payload);
// '{"name":"Ada","age":36,"active":true,"tags":["math"]}'

const parsed = JSON.parse(json);
```

**Syntax Rules:**

1. Keys must be double-quoted strings.
2. Values must be object, array, string, number, boolean, or `null`.
3. No trailing commas, comments, or `undefined`.
4. Dates must be serialized explicitly (ISO strings) and parsed back into `Date` objects.
5. Use `JSON.stringify(value, replacer, space)` for custom serialization and formatting.

**Constraints and Limitations:**

- No native support for `Date`, `Map`, `Set`, `BigInt` (throws), or `undefined`.
- `NaN` and `Infinity` become `null`.
- Deeply nested or very large JSON can be slow to parse.
- JSON is not safe to inject into HTML without escaping (XSS risk).

### Annotated Code Examples

```tsx
async function saveSettings(settings) {
  const res = await fetch('/api/settings', {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(settings),
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

**Expected Output:** The settings object is serialized to JSON and sent. The response is parsed back into an object.

**Why this output:** `JSON.stringify` converts the object to a JSON string. `Content-Type: application/json` tells the server how to parse it. `res.json()` parses the response.

### Real-World Cases

- **API requests:** Sending and receiving JSON payloads.
- **Configuration:** Storing app config as JSON.
- **Local storage:** Persisting state with `JSON.stringify`/`JSON.parse`.

### References

- MDN — JSON — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format — https://www.rfc-editor.org/rfc/rfc8259
- MDN — JSON.stringify — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify

---

## 9. CORS

### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism that controls whether a web page at one origin can access resources from a different origin.

**Technical Definition:** An **origin** is the combination of scheme, host, and port. By default, browsers block cross-origin reads of responses (the **same-origin policy**). CORS allows the server to relax this by sending `Access-Control-Allow-Origin` and related headers. For requests that are not "simple" (e.g., using `PUT`, `DELETE`, custom headers, or `Content-Type: application/json`), the browser first sends an **`OPTIONS` preflight** request to ask the server whether the actual request is permitted. The server responds with `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and optionally `Access-Control-Max-Age`.

**Beginner-Friendly Explanation:** CORS is the browser's rule that a page from `app.example.com` cannot silently read data from `api.other.com` unless the API explicitly says it is allowed. If the API does not say so, the browser blocks the response — even though the request may have reached the server.

### Purposes

- To allow controlled cross-origin access to APIs.
- To protect users from malicious cross-origin reads.
- To enable preflight negotiation for non-simple requests.
- To support credentials safely across origins.
- To allow CDNs and third-party services to serve assets to your app.

### Syntax Rules and Structure

**Preflight flow:**

```text
OPTIONS /api/data HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: content-type

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

**Simple vs. preflighted requests:**

| Condition | Simple request | Preflight required |
|-----------|----------------|--------------------|
| Method GET/HEAD/POST | Yes | No |
| `Content-Type` text/plain, form-urlencoded, multipart | Yes | No |
| `Content-Type: application/json` | No | Yes |
| Custom headers | No | Yes |
| PUT/DELETE/PATCH | No | Yes |

**Syntax Rules:**

1. The server must return `Access-Control-Allow-Origin` matching the request origin or `*`.
2. Preflight responses must include `Access-Control-Allow-Methods` and `Access-Control-Allow-Headers`.
3. `Access-Control-Allow-Credentials: true` is required for credentialed requests and cannot be combined with `*` origin.
4. `Access-Control-Max-Age` caches the preflight result to reduce requests.
5. CORS is enforced by the browser, not the server — non-browser clients ignore it.

**Constraints and Limitations:**

- `*` origin cannot be used with credentials.
- Preflight requests add latency; cache them with `Access-Control-Max-Age`.
- CORS does not protect the server; it protects users' browsers.
- Misconfigured `Access-Control-Allow-Origin: *` on credentialed endpoints is a security flaw.

### Annotated Code Examples

```tsx
async function callApi() {
  const res = await fetch('https://api.example.com/data', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ hello: 'world' }),
    credentials: 'include',
  });

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

**Expected Output:** Because the request uses `application/json` and credentials, the browser sends an `OPTIONS` preflight first. The server must allow the origin, the `content-type` header, and credentials. If not, the browser blocks the request with a CORS error.

**Why this output:** `application/json` and `credentials: 'include'` make this a non-simple, credentialed request. The server must respond to preflight with matching CORS headers and `Access-Control-Allow-Credentials: true`.

### Real-World Cases

- **SPA → API:** A React app on `app.example.com` calling `api.example.com`.
- **Third-party services:** Calling Stripe, Auth0, or analytics APIs.
- **CDN assets:** Loading fonts and images cross-origin.

### References

- MDN — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- Fetch Standard — CORS protocol — https://fetch.spec.whatwg.org/#http-cors-protocol
- MDN — Access-Control-Allow-Origin — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Access-Control-Allow-Origin

---

## 10. Credentials

### Definitions

**Core Definition:** Credentials are the authentication data — cookies, HTTP authentication, or client certificates — that identify the user making a request.

**Technical Definition:** In `fetch`, the `credentials` option controls whether credentials are sent and whether `Set-Cookie` response headers are respected: `'omit'` (never), `'same-origin'` (default; only same-origin), and `'include'` (always, including cross-origin). Cookie-based credentials are subject to `SameSite`, `Secure`, and `HttpOnly` attributes. For cross-origin credentialed requests, the server must return `Access-Control-Allow-Credentials: true` and an explicit origin (not `*`). Bearer tokens in the `Authorization` header are not "credentials" in the CORS sense — they are just headers, and they require `Access-Control-Allow-Headers: Authorization` in preflight.

**Beginner-Friendly Explanation:** Credentials are how the server knows who you are. Cookies are sent automatically to the same origin, but for cross-origin requests you must explicitly ask to include them, and the server must explicitly allow it.

### Purposes

- To authenticate requests using cookies or tokens.
- To control whether cookies are sent cross-origin.
- To protect sessions from CSRF and XSS.
- To configure secure cookie attributes (`HttpOnly`, `Secure`, `SameSite`).
- To enable single sign-on across subdomains.

### Syntax Rules and Structure

```js
// Same-origin cookies (default)
fetch('/api/me', { credentials: 'same-origin' });

// Cross-origin cookies (requires server CORS support)
fetch('https://api.example.com/me', { credentials: 'include' });

// No credentials
fetch('/api/public', { credentials: 'omit' });
```

**Cookie attributes:**

| Attribute | Purpose |
|-----------|---------|
| `HttpOnly` | Not accessible to JavaScript (XSS mitigation) |
| `Secure` | Sent only over HTTPS |
| `SameSite=Strict/Lax/None` | Controls cross-site sending |
| `Domain` | Which hosts receive the cookie |
| `Path` | Which paths receive the cookie |

**Syntax Rules:**

1. `credentials: 'include'` is required for cross-origin cookies.
2. The server must respond with `Access-Control-Allow-Credentials: true` and a specific origin.
3. `SameSite=None` requires `Secure`.
4. Never store session tokens in `localStorage` if an `HttpOnly` cookie is available.
5. Use `Authorization: Bearer <token>` for token-based auth; include `Authorization` in `Access-Control-Allow-Headers`.

**Constraints and Limitations:**

- `Access-Control-Allow-Origin: *` cannot be used with credentials.
- Cookies are sent automatically on same-origin requests, which creates CSRF risk; use `SameSite` and CSRF tokens.
- Third-party cookie blocking in browsers affects cross-site credential flows.

### Annotated Code Examples

```tsx
async function fetchProfile() {
  const res = await fetch('https://api.example.com/me', {
    credentials: 'include',
    headers: { Accept: 'application/json' },
  });

  if (res.status === 401) throw new Error('Not authenticated');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

**Expected Output:** The browser sends the session cookie to the API. If the session is valid, the profile is returned. If not, `401` triggers a login flow.

**Why this output:** `credentials: 'include'` sends cookies cross-origin. The server must allow the origin and credentials via CORS. `401` distinguishes authentication failure from other errors.

### Real-World Cases

- **Session-based auth:** Cookies with `HttpOnly`, `Secure`, `SameSite=Lax`.
- **SSO across subdomains:** Cookies scoped to `.example.com`.
- **Token-based APIs:** `Authorization: Bearer` headers.
- **Third-party APIs:** OAuth tokens in headers.

### References

- MDN — Request.credentials — https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials
- MDN — Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- MDN — SameSite cookies — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie/SameSite
- Fetch Standard — credentials — https://fetch.spec.whatwg.org/#concept-request-credentials-mode

---

## References

- MDN — HTTP — https://developer.mozilla.org/en-US/docs/Web/HTTP
- MDN — HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- MDN — HTTP headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers
- MDN — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN — Conditional requests — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Conditional_requests
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — Request.credentials — https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials
- MDN — Set-Cookie — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- MDN — JSON — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- RFC 9111 — HTTP Caching — https://www.rfc-editor.org/rfc/rfc9111
- RFC 8259 — JSON Data Interchange Format — https://www.rfc-editor.org/rfc/rfc8259
- RFC 7396 — JSON Merge Patch — https://www.rfc-editor.org/rfc/rfc7396
- RFC 6902 — JSON Patch — https://www.rfc-editor.org/rfc/rfc6902
- Fetch Standard — https://fetch.spec.whatwg.org/
- IANA — HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/