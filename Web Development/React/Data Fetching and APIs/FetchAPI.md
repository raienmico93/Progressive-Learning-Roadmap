# Fetch API — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The Fetch API is the browser's modern interface for making HTTP requests, providing a Promise-based replacement for `XMLHttpRequest` and the foundation for all client-side data fetching in React.

**Technical Definition:** `fetch(input, init)` is a global function defined by the WHATWG Fetch Standard. It takes a URL (or `Request` object) and an optional `init` object containing request options, and returns a `Promise<Response>`. The Promise resolves as soon as the response headers are received — **not** when the body has finished downloading — and it rejects only on network-level failures (DNS, connection refused, CORS), **never** on HTTP error status codes. The `Response` object exposes the status, headers, and a body that can be consumed exactly once via `.json()`, `.text()`, `.blob()`, `.arrayBuffer()`, `.formData()`, or `.body` (a `ReadableStream`). Request cancellation is handled through `AbortSignal`, and the `Request`, `Response`, `Headers`, and `AbortController` interfaces are all part of the standard.

**Beginner-Friendly Explanation:** `fetch` is how your JavaScript code talks to a server. You give it a URL, it sends a request, and it gives you back a Promise that eventually resolves to a response. The most important thing to understand — and the source of most beginner bugs — is that `fetch` does **not** throw when the server returns a 404 or a 500. It only throws when the network itself fails. A 404 is still a "successful" fetch from the browser's point of view; your code has to check `response.ok` yourself. Once you know that, `fetch` becomes a reliable, composable primitive that works with `async`/`await`, cancellation signals, and streaming.

### Key Characteristics

- **Promise-based:** Returns `Promise<Response>`; works naturally with `async`/`await` and `Promise.all`.
- **Resolves early:** The Promise settles when headers arrive, before the body is fully downloaded.
- **Does not reject on HTTP errors:** A `404` or `500` resolves normally; you must check `response.ok` or `response.status`.
- **Single-use body:** The body stream can be consumed only once; use `response.clone()` to read it twice.
- **Cancellable:** Accepts an `AbortSignal` via the `signal` option; aborted requests reject with `AbortError`.
- **No built-in timeout:** Without a signal or `AbortSignal.timeout()`, a stalled request hangs indefinitely.
- **CORS-aware:** Enforces the same-origin policy; cross-origin requests need server-side CORS headers.
- **Credentials opt-in:** Cookies are sent same-origin by default; cross-origin requires `credentials: 'include'`.
- **Streaming-capable:** `response.body` is a `ReadableStream` that can be read incrementally.

### Prerequisites

- JavaScript Promises and `async`/`await`.
- Basic understanding of HTTP methods, status codes, and headers.
- Familiarity with JSON serialization (`JSON.stringify` / `JSON.parse`).
- React function components, `useState`, and `useEffect`.
- Knowledge of the browser's same-origin policy and CORS.

### Related Programming Areas

- **HTTP Fundamentals** — methods, status codes, headers, CORS, credentials.
- **Data Fetching in React** — query libraries, loaders, Server Components.
- **Error Handling** — typed errors, retries, error boundaries.
- **Performance** — caching, request deduplication, waterfalls.
- **Streaming** — `ReadableStream`, server-sent events, progressive rendering.

---

## 1. `fetch`

### Definitions

**Core Definition:** `fetch()` is the global function that sends an HTTP request and returns a Promise resolving to a `Response`.

**Technical Definition:** `fetch(input, init?)` accepts a URL string, `URL` object, or `Request` instance as `input`, and an optional `init` object containing method, headers, body, mode, credentials, cache, redirect, referrer, integrity, keepalive, signal, and priority. It returns `Promise<Response>`. The Promise resolves when the response headers are available; the body is read separately. Rejection occurs only on network-level failures (including CORS failures), not on HTTP error statuses.

**Beginner-Friendly Explanation:** `fetch` is the function you call to make a request. You give it a URL, and it gives you back a Promise that resolves to a response object.

### Purposes

- To send HTTP requests from the browser without a library.
- To retrieve JSON, text, binary data, or streams from a server.
- To submit forms and JSON payloads.
- To cancel in-flight requests when they are no longer needed.
- To integrate with `async`/`await` control flow for readable request logic.

### Syntax Rules and Structure

**General Syntax:**

```js
const response = await fetch(input, init);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `input` | URL string, `URL` object, or `Request` instance |
| `init` | Optional object with method, headers, body, signal, etc. |
| `response` | `Response` object with `ok`, `status`, `headers`, and body |

**Syntax Rules:**

1. `fetch` returns a Promise that resolves when the response headers are received.
2. The Promise rejects only on network failure, not on HTTP error statuses.
3. The response body must be consumed via `.json()`, `.text()`, `.blob()`, etc., and can be read only once.
4. Use `await` inside an `async` function or chain `.then()`.
5. Pass an `AbortSignal` via `signal` to make the request cancellable.

**Constraints and Limitations:**

- No built-in timeout; stalled requests hang indefinitely.
- Body can only be read once; use `.clone()` to read twice.
- No automatic retry, caching, or deduplication.
- Opaque responses (`mode: 'no-cors'`) expose no status or body.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic GET with `async`/`await`

```js
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return response.json();
}
```

**Expected Output:** Resolves to the parsed user object on success; throws an `Error` on HTTP failure; rejects with a `TypeError` on network failure.

**Why this output:** `fetch` resolves even for `404`/`500`, so the `response.ok` check converts HTTP errors into exceptions. `.json()` parses the body into a JavaScript object.

#### Example 2: POST with JSON Body

```js
async function createPost(post) {
  const response = await fetch('/api/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(post),
  });

  if (!response.ok) {
    const problem = await response.json().catch(() => ({}));
    throw new Error(problem.message ?? `HTTP ${response.status}`);
  }

  return response.json();
}
```

**Expected Output:** Creates the post and returns the created resource JSON. On validation failure, throws with the server's error message.

**Why this output:** `Content-Type: application/json` tells the server the body format. `response.json().catch(() => ({}))` safely parses an error body that may not be JSON.

### Real-World Cases

- **Data dashboards:** Fetching stats from an API on mount.
- **Form submissions:** POSTing user input to a backend.
- **File uploads:** Sending `FormData` with `fetch`.
- **Streaming:** Reading `response.body` for server-sent data.

### References

- MDN — Window: fetch() method — https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch
- WHATWG — Fetch Standard — https://fetch.spec.whatwg.org/
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API

---

## 2. Request Options

### Definitions

**Core Definition:** Request options are the properties of the `init` object that configure how `fetch` sends a request.

**Technical Definition:** The `init` object accepts `method`, `headers`, `body`, `mode`, `credentials`, `cache`, `redirect`, `referrer`, `referrerPolicy`, `integrity`, `keepalive`, `signal`, and `priority`. These options map to the underlying `Request` interface and control the method, payload, security context, caching behaviour, redirect handling, and cancellation of the request.

**Beginner-Friendly Explanation:** Request options are the settings you pass to `fetch` to control how the request is sent — what method, what headers, what body, whether to include cookies, and whether it can be cancelled.

### Purposes

- To specify the HTTP method (GET, POST, PUT, PATCH, DELETE).
- To send headers and a request body.
- To control credential inclusion for cookies.
- To control caching and redirect behaviour.
- To enable cancellation through an `AbortSignal`.
- To keep requests alive during page unload (`keepalive`).

### Syntax Rules and Structure

**Common Options:**

| Option | Values | Default | Purpose |
|--------|--------|---------|---------|
| `method` | `'GET'`, `'POST'`, `'PUT'`, `'PATCH'`, `'DELETE'`, `'HEAD'`, `'OPTIONS'` | `'GET'` | HTTP verb |
| `headers` | `Headers` or object | `{}` | Request metadata |
| `body` | string, `FormData`, `Blob`, `ArrayBuffer`, `URLSearchParams`, `ReadableStream` | `null` | Payload |
| `mode` | `'cors'`, `'no-cors'`, `'same-origin'`, `'navigate'` | `'cors'` | CORS behaviour |
| `credentials` | `'omit'`, `'same-origin'`, `'include'` | `'same-origin'` | Cookie inclusion |
| `cache` | `'default'`, `'no-store'`, `'reload'`, `'no-cache'`, `'force-cache'`, `'only-if-cached'` | `'default'` | HTTP cache behaviour |
| `redirect` | `'follow'`, `'error'`, `'manual'` | `'follow'` | Redirect handling |
| `signal` | `AbortSignal` | `undefined` | Cancellation |
| `keepalive` | boolean | `false` | Outlive the page |
| `integrity` | SRI hash string | `''` | Subresource integrity |
| `priority` | `'high'`, `'low'`, `'auto'` | `'auto'` | Fetch priority hint |

**Syntax Rules:**

1. `body` must be omitted for `GET` and `HEAD`.
2. When `body` is a `FormData` instance, do not set `Content-Type` manually; the browser adds the multipart boundary.
3. `credentials: 'include'` requires server-side `Access-Control-Allow-Credentials: true` and a specific origin.
4. `keepalive: true` is limited to ~64 KB of body and is intended for analytics/beacon-style requests.
5. `mode: 'no-cors'` returns opaque responses with `status === 0` and no readable body.

**Constraints and Limitations:**

- `no-cors` mode hides the status and body, making it unsuitable for most API calls.
- Setting forbidden headers (`Host`, `Content-Length`) is silently ignored or rejected.
- `keepalive` bodies over the size limit cause the request to fail.

### Annotated Code Examples

```js
async function request(url, { method = 'GET', body, token, signal } = {}) {
  const response = await fetch(url, {
    method,
    headers: {
      Accept: 'application/json',
      ...(body ? { 'Content-Type': 'application/json' } : {}),
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
    body: body ? JSON.stringify(body) : undefined,
    credentials: 'same-origin',
    signal,
  });

  return response;
}
```

**Expected Output:** A request configured with the chosen method, JSON headers, bearer token, same-origin credentials, and optional cancellation.

**Why this output:** Conditional spreading avoids sending `Content-Type` without a body and omits `Authorization` when no token is provided. `signal` enables cancellation.

### Real-World Cases

- **Authenticated APIs:** Bearer tokens and credentials.
- **FormData uploads:** File uploads without manual `Content-Type`.
- **Analytics beacons:** `keepalive: true` for unload events.
- **Cached reads:** `cache: 'force-cache'` for immutable assets.

### References

- MDN — Request — https://developer.mozilla.org/en-US/docs/Web/API/Request
- MDN — RequestInit — https://developer.mozilla.org/en-US/docs/Web/API/RequestInit
- WHATWG — Fetch Standard: Request — https://fetch.spec.whatwg.org/#request-class

---

## 3. Response Handling

### Definitions

**Core Definition:** Response handling is the process of inspecting the `Response` object — its status, headers, and body — to decide what the application should do.

**Technical Definition:** The `Response` object exposes `ok` (true for 200–299), `status`, `statusText`, `headers` (a `Headers` instance), `type` (`'basic'`, `'cors'`, `'opaque'`, `'opaqueredirect'`, `'error'`), `url`, `redirected`, and body accessors (`json()`, `text()`, `blob()`, `arrayBuffer()`, `formData()`, `body`). Each accessor returns a Promise and consumes the body stream, so the body can only be read once unless cloned.

**Beginner-Friendly Explanation:** Once `fetch` gives you a response, you have to look at it: did it succeed (`ok`), what was the status code, what headers came back, and what does the body contain? Response handling is the code that answers those questions.

### Purposes

- To distinguish successful responses from HTTP errors.
- To read the response body in the appropriate format.
- To inspect status codes and route to the correct UI.
- To read headers for pagination, caching, and tracing.
- To handle opaque and redirected responses safely.

### Syntax Rules and Structure

**Response properties:**

| Property | Type | Meaning |
|----------|------|---------|
| `ok` | boolean | `true` for 200–299 |
| `status` | number | HTTP status code |
| `statusText` | string | Status message |
| `headers` | `Headers` | Response headers |
| `type` | string | `'basic'`, `'cors'`, `'opaque'`, `'opaqueredirect'`, `'error'` |
| `url` | string | Final URL after redirects |
| `redirected` | boolean | Whether redirects occurred |
| `body` | `ReadableStream` | Raw body stream |

**Body readers:**

| Method | Returns |
|--------|---------|
| `.json()` | Parsed JSON |
| `.text()` | String |
| `.blob()` | `Blob` |
| `.arrayBuffer()` | `ArrayBuffer` |
| `.formData()` | `FormData` |
| `.body` | `ReadableStream` |

**Syntax Rules:**

1. Check `response.ok` or `response.status` before parsing.
2. Do not parse a body for `204 No Content` or `304 Not Modified`.
3. The body can be read only once; call `response.clone()` before reading if you need it twice.
4. Use `response.headers.get('X-Custom')` to read custom headers.
5. `response.type === 'opaque'` means the body and status are inaccessible.

**Constraints and Limitations:**

- Parsing a non-JSON body with `.json()` throws a `SyntaxError`.
- Cloning large bodies duplicates memory.
- Opaque responses expose `status === 0` and no body.

### Annotated Code Examples

```jsx
async function fetchUsers(page) {
  const response = await fetch(`/api/users?page=${page}`);

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  const total = Number(response.headers.get('X-Total-Count') ?? '0');
  const users = await response.json();

  return { users, total, hasNext: users.length > 0 };
}
```

**Expected Output:** An object with the parsed users, the total count from a response header, and a `hasNext` flag derived from the page contents.

**Why this output:** `response.ok` guards against HTTP errors. `response.headers.get` reads metadata. `.json()` parses the body once.

### Real-World Cases

- **Paginated lists:** Reading `X-Total-Count` and `Link` headers.
- **Binary downloads:** Using `.blob()` for images and files.
- **FormData responses:** Parsing multipart responses with `.formData()`.

### References

- MDN — Response — https://developer.mozilla.org/en-US/docs/Web/API/Response
- MDN — Response.ok — https://developer.mozilla.org/en-US/docs/Web/API/Response/ok
- MDN — Response.clone() — https://developer.mozilla.org/en-US/docs/Web/API/Response/clone

---

## 4. JSON Parsing

### Definitions

**Core Definition:** JSON parsing is the conversion of a response body from a JSON string into a JavaScript value using `response.json()`.

**Technical Definition:** `response.json()` reads the entire response body as text and parses it with `JSON.parse`, returning a Promise that resolves to the resulting object, array, or primitive. It rejects with a `SyntaxError` if the body is not valid JSON, and it consumes the body stream so it cannot be read again. For streaming JSON or large payloads, `response.body` can be read as a `ReadableStream` with a parser.

**Beginner-Friendly Explanation:** When the server sends JSON, `response.json()` turns that text into a real JavaScript object you can use.

### Purposes

- To convert JSON responses into usable JavaScript objects.
- To validate that a response is actually JSON before using it.
- To handle JSON parse failures gracefully.
- To read streaming JSON incrementally for large payloads.
- To combine parsing with schema validation (Zod, Valibot).

### Syntax Rules and Structure

```js
const data = await response.json();
```

**Syntax Rules:**

1. `.json()` returns a Promise; always `await` it.
2. It consumes the body; call `.clone()` first if you need the raw text as well.
3. It throws `SyntaxError` on invalid JSON.
4. Wrap in `try`/`catch` when the response might not be JSON (e.g., HTML error pages).
5. For large payloads, consider streaming via `response.body`.

**Constraints and Limitations:**

- A response with an empty body throws on `.json()`.
- An HTML error page (common with proxies) throws a `SyntaxError`.
- Large JSON payloads block the main thread during parse.

### Annotated Code Examples

```js
async function safeJson(response) {
  const text = await response.text();

  if (!text) return null;

  try {
    return JSON.parse(text);
  } catch {
    throw new Error('Expected JSON but received a non-JSON response');
  }
}
```

**Expected Output:** Returns the parsed JSON, `null` for empty bodies, or throws a descriptive error for non-JSON bodies.

**Why this output:** Reading `.text()` first allows detecting empty bodies and producing a clear error instead of a cryptic `SyntaxError` from `.json()`.

### Real-World Cases

- **API clients:** Parsing JSON responses uniformly.
- **Error handling:** Detecting HTML error pages from proxies.
- **Validation:** Parsing then validating with Zod.

### References

- MDN — Response.json() — https://developer.mozilla.org/en-US/docs/Web/API/Response/json
- MDN — JSON.parse() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- MDN — Response.text() — https://developer.mozilla.org/en-US/docs/Web/API/Response/text

---

## 5. Error Handling

### Definitions

**Core Definition:** Fetch error handling is the set of techniques for detecting, categorising, and responding to failures — network errors, HTTP error statuses, parse errors, and aborts.

**Technical Definition:** `fetch` rejects with a `TypeError` only on network-level failures (DNS, connection, CORS). HTTP error statuses resolve normally and must be detected via `response.ok`. Body parsing can reject with `SyntaxError`. Aborts reject with a `DOMException` named `AbortError`. Timeouts via `AbortSignal.timeout()` reject with a `DOMException` named `TimeoutError`. Robust error handling distinguishes these cases and maps them to application-level error types and UI.

**Beginner-Friendly Explanation:** `fetch` only throws when the network breaks. If the server says “404” or “500,” `fetch` considers that a successful conversation — you have to check `response.ok` yourself. And if you abort a request, that is also a rejection you need to handle.

### Purposes

- To detect HTTP errors that `fetch` does not reject on.
- To distinguish network failures from HTTP failures.
- To handle aborts without showing errors to the user.
- To handle timeouts with retry logic.
- To map failures to typed application errors and UI states.
- To log errors to monitoring services.

### Syntax Rules and Structure

**Error categories:**

| Cause | Rejection type | Handling |
|-------|----------------|----------|
| Network failure | `TypeError` | Show offline/connection error |
| CORS failure | `TypeError` | Show generic failure; log |
| HTTP 4xx/5xx | Resolves | Check `response.ok` |
| Invalid JSON | `SyntaxError` | Show parse error |
| Abort | `DOMException: AbortError` | Ignore |
| Timeout | `DOMException: TimeoutError` | Retry or show timeout |
| Opaque response | Resolves, `status === 0` | Avoid `no-cors` mode |

**Syntax Rules:**

1. Always check `response.ok` after `await fetch`.
2. Ignore `AbortError`; it is intentional cancellation, not a failure.
3. Distinguish `TimeoutError` from other failures for retry logic.
4. Wrap `.json()` in `try`/`catch` when the response may not be JSON.
5. Convert low-level errors into typed application errors at the boundary.

**Constraints and Limitations:**

- CORS failures and network failures both surface as `TypeError` with little detail (browser security).
- Opaque responses cannot be inspected, so errors are invisible.
- `fetch` does not retry automatically.

### Annotated Code Examples

```js
class HttpError extends Error {
  constructor(status, body) {
    super(`HTTP ${status}`);
    this.name = 'HttpError';
    this.status = status;
    this.body = body;
  }
}

async function fetchJson(url, options) {
  let response;

  try {
    response = await fetch(url, options);
  } catch (error) {
    if (error.name === 'AbortError') throw error;
    throw new Error('Network error: unable to reach the server');
  }

  if (!response.ok) {
    const body = await response.json().catch(() => null);
    throw new HttpError(response.status, body);
  }

  return response.json();
}
```

**Expected Output:** Returns parsed JSON on success. Throws `HttpError` with status and body for HTTP failures, a `TypeError`-derived message for network failures, and rethrows `AbortError` for cancellations.

**Why this output:** The `try`/`catch` around `fetch` handles network rejection. The `response.ok` check handles HTTP errors. The `AbortError` branch preserves cancellation semantics.

### Real-World Cases

- **Offline handling:** Distinguishing network failure from server errors.
- **Auth failures:** Handling `401` with a login redirect.
- **Validation errors:** Mapping `422` bodies to form fields.
- **Retry logic:** Retrying `5xx` and `TimeoutError` but not `4xx`.

### References

- MDN — Response.ok — https://developer.mozilla.org/en-US/docs/Web/API/Response/ok
- MDN — DOMException — https://developer.mozilla.org/en-US/docs/Web/API/DOMException
- MDN — AbortSignal — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- web.dev — Fetch errors — https://web.dev/articles/fetch-api-guide

---

## 6. AbortController

### Definitions

**Core Definition:** `AbortController` is the interface that creates an `AbortSignal` used to cancel one or more `fetch` requests.

**Technical Definition:** `new AbortController()` creates a controller with a `.signal` property (`AbortSignal`) and an `.abort(reason)` method. Passing `signal` to `fetch` links the request to the controller; calling `.abort()` causes the pending `fetch` Promise to reject with a `DOMException` named `AbortError`. The signal can be passed to multiple requests to cancel them together. `AbortSignal` also supports `abort` event listeners and the static combinators `AbortSignal.any()` and `AbortSignal.timeout()`.

**Beginner-Friendly Explanation:** `AbortController` is the cancel button for a request. You create a controller, hand its signal to `fetch`, and when you no longer need the request, you call `.abort()`.

### Purposes

- To cancel requests when a component unmounts.
- To cancel stale requests when parameters change.
- To cancel multiple requests with a single controller.
- To implement timeouts by combining signals.
- To avoid memory leaks and wasted network usage.
- To prevent race conditions from late responses.

### Syntax Rules and Structure

```js
const controller = new AbortController();

fetch(url, { signal: controller.signal })
  .catch((err) => {
    if (err.name === 'AbortError') return; // expected
    throw err;
  });

controller.abort(); // cancels the request
```

**Syntax Rules:**

1. Create a controller per logical request or group of requests.
2. Pass `signal` in the `init` object.
3. Call `.abort()` in cleanup functions (`useEffect` return, event handlers).
4. Handle `AbortError` explicitly; do not treat it as a user-facing failure.
5. Use `AbortSignal.any([...])` to combine multiple signals.

**Constraints and Limitations:**

- Aborting a completed request has no effect.
- The `AbortError` message is browser-specific; check `error.name`.
- Aborting does not guarantee the server stops processing; it only stops the client from waiting.

### Annotated Code Examples

```jsx
useEffect(() => {
  const controller = new AbortController();

  (async () => {
    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
        signal: controller.signal,
      });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      setResults(await response.json());
    } catch (error) {
      if (error.name !== 'AbortError') setError(error);
    }
  })();

  return () => controller.abort();
}, [query]);
```

**Expected Output:** When `query` changes, the previous request is aborted and a new one starts. Only the latest request updates state. Unmounting aborts the final request.

**Why this output:** The cleanup calls `controller.abort()` on dependency change or unmount. The `AbortError` branch prevents stale errors from being shown.

### Real-World Cases

- **Search inputs:** Cancelling outdated queries on every keystroke.
- **Route changes:** Cancelling in-flight requests when navigating away.
- **Modal close:** Cancelling requests for data no longer needed.
- **Batch cancellation:** Cancelling multiple requests on logout.

### References

- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN — AbortSignal — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- MDN — AbortSignal.any() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static

---

## 7. Timeouts

### Definitions

**Core Definition:** A fetch timeout is a mechanism that aborts a request if it does not complete within a specified duration.

**Technical Definition:** `fetch` has no built-in timeout. Timeouts are implemented by combining an `AbortController` with a timer, or by using `AbortSignal.timeout(ms)`, which returns a signal that aborts with a `TimeoutError` after the specified duration. `AbortSignal.any([signalA, signalB])` combines multiple signals — for example, a user-cancellation signal and a timeout signal — so the request aborts if either fires.

**Beginner-Friendly Explanation:** A timeout says “if this request takes too long, give up.” `fetch` does not do this automatically, so you add a signal that fires after a set number of milliseconds.

### Purposes

- To prevent requests from hanging indefinitely.
- To provide feedback when the network is slow or unresponsive.
- To retry after a timeout.
- To distinguish timeouts from user-initiated cancellations.
- To enforce SLAs on critical requests.

### Syntax Rules and Structure

**Native timeout:**

```js
const response = await fetch(url, {
  signal: AbortSignal.timeout(5000),
});
```

**Combined timeout and user cancellation:**

```js
const controller = new AbortController();
const signal = AbortSignal.any([
  controller.signal,
  AbortSignal.timeout(5000),
]);
```

**Manual timeout (for older environments):**

```js
const controller = new AbortController();
const timer = setTimeout(() => controller.abort(new DOMException('Timeout', 'TimeoutError')), 5000);
```

**Syntax Rules:**

1. Use `AbortSignal.timeout(ms)` for simple timeouts.
2. Use `AbortSignal.any([...])` to combine timeout and user cancellation.
3. Check `error.name === 'TimeoutError'` to distinguish timeouts from aborts.
4. Always clear manual timers when the request settles.
5. Choose timeouts based on the endpoint's expected latency, not a single global value.

**Constraints and Limitations:**

- `AbortSignal.timeout()` and `AbortSignal.any()` require modern browsers (widely available since 2023–2024).
- A timeout does not tell you whether the server processed the request.
- Timeouts on non-idempotent requests (POST) can cause duplicate operations if retried.

### Annotated Code Examples

```js
async function fetchWithTimeout(url, { timeout = 5000, signal, ...options } = {}) {
  const timeoutSignal = AbortSignal.timeout(timeout);
  const combined = signal ? AbortSignal.any([signal, timeoutSignal]) : timeoutSignal;

  try {
    const response = await fetch(url, { ...options, signal: combined });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (error) {
    if (error.name === 'TimeoutError') {
      throw new Error(`Request timed out after ${timeout}ms`);
    }
    throw error;
  }
}
```

**Expected Output:** Resolves with parsed JSON, throws a descriptive timeout error on timeout, and rethrows other errors (including `AbortError`).

**Why this output:** `AbortSignal.any` combines the caller's cancellation signal with the timeout. The `TimeoutError` branch converts the DOM exception into a readable message.

### Real-World Cases

- **Search APIs:** Short timeouts (2–5s) to keep the UI responsive.
- **Critical mutations:** Longer timeouts (15–30s) with retry on timeout.
- **Health checks:** Very short timeouts with fallback UI.

### References

- MDN — AbortSignal.timeout() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- MDN — AbortSignal.any() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static
- web.dev — Fetch API guide — https://web.dev/articles/fetch-api-guide

---

## 8. Fetch in Effects

### Definitions

**Core Definition:** Fetching in effects is the practice of initiating a `fetch` request inside `useEffect` in response to a component mount or dependency change.

**Technical Definition:** `useEffect(() => { ... }, deps)` runs after the component commits. Fetching inside it is the classic client-side data-fetching pattern. The effect must handle cleanup — aborting the request — because effects re-run on dependency changes and, in React Strict Mode, are mounted, unmounted, and re-mounted in development to surface missing cleanup. React 18+ recommends libraries (TanStack Query, SWR) or framework loaders over raw `useEffect` fetching for anything beyond trivial cases.

**Beginner-Friendly Explanation:** Fetching in effects means starting the request after the component appears. You must clean up the request when the component goes away or when the inputs change — otherwise you get duplicate requests and state updates on unmounted components.

### Purposes

- To fetch data when a component mounts.
- To refetch when a dependency (ID, query) changes.
- To handle the request lifecycle with loading, error, and success states.
- To clean up in-flight requests on unmount.
- To integrate with `AbortController` for cancellation.

### Syntax Rules and Structure

```jsx
useEffect(() => {
  const controller = new AbortController();

  (async () => {
    try {
      const res = await fetch(url, { signal: controller.signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      setData(await res.json());
    } catch (err) {
      if (err.name !== 'AbortError') setError(err);
    } finally {
      setIsPending(false);
    }
  })();

  return () => controller.abort();
}, [url]);
```

**Syntax Rules:**

1. Define and call an async function inside the effect; never make the effect callback itself `async`.
2. Always return a cleanup that aborts the request.
3. Ignore `AbortError` in the catch block.
4. Include every dependency the URL depends on.
5. Reset state at the start of the effect when the dependency changes.

**Constraints and Limitations:**

- Strict Mode runs the effect twice in development, causing duplicate requests unless deduplicated or aborted.
- Raw effect fetching creates waterfalls and lacks caching.
- Race conditions occur if stale responses are not ignored.

### Annotated Code Examples

```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [isPending, setIsPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setIsPending(true);
    setError(null);

    (async () => {
      try {
        const res = await fetch(`/api/users/${userId}`, { signal: controller.signal });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        setUser(await res.json());
      } catch (err) {
        if (err.name !== 'AbortError') setError(err);
      } finally {
        setIsPending(false);
      }
    })();

    return () => controller.abort();
  }, [userId]);

  if (isPending) return <ProfileSkeleton />;
  if (error) return <ErrorBanner error={error} />;
  if (!user) return <EmptyState />;
  return <ProfileCard user={user} />;
}
```

**Expected Output:** A skeleton while loading, an error banner on failure, an empty state if no user exists, and the profile on success. Changing `userId` aborts the previous request and starts a new one.

**Why this output:** The effect re-runs on `userId` change. The cleanup aborts the previous request, preventing stale updates. The state resets at the start of each effect run.

### Real-World Cases

- **Detail pages:** Fetching a resource by ID on navigation.
- **Search:** Fetching results when the query changes.
- **Polling:** Fetching on an interval with cleanup.

### References

- React — useEffect — https://react.dev/reference/react/useEffect
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects

---

## 9. Fetch Pitfalls

### Definitions

**Core Definition:** Fetch pitfalls are the common mistakes and surprising behaviours that cause bugs in client-side fetching.

**Technical Definition:** The most common pitfalls are: assuming `fetch` rejects on HTTP errors; forgetting cancellation, which causes memory leaks and race conditions; reading the body twice without cloning; ignoring timeouts; misconfiguring CORS and credentials; treating `AbortError` as a user-facing error; relying on Strict Mode–doubled effects without deduplication; and parsing non-JSON responses with `.json()`. Each pitfall has a specific, well-understood remedy.

**Beginner-Friendly Explanation:** These are the traps that catch almost everyone the first time they use `fetch`. Knowing them saves hours of debugging.

### Purposes

- To recognise and avoid the most common `fetch` mistakes.
- To prevent race conditions and memory leaks.
- To distinguish real failures from expected cancellations.
- To configure CORS and credentials correctly.
- To build reusable, safe fetch helpers.

### Common Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| **No `response.ok` check** | 404/500 treated as success | Check `response.ok` before parsing |
| **No cancellation** | Stale responses overwrite newer data | `AbortController` in cleanup |
| **Reading body twice** | `TypeError: body used already` | `response.clone()` before reading twice |
| **No timeout** | Request hangs forever | `AbortSignal.timeout()` |
| **Treating `AbortError` as an error** | Spurious error UI on navigation | Ignore `error.name === 'AbortError'` |
| **`.json()` on empty/HTML body** | `SyntaxError` | Check content or use `.text()` first |
| **Missing `credentials`** | Cookies not sent cross-origin | `credentials: 'include'` + server CORS |
| **`no-cors` expectations** | Opaque response, no status/body | Use `cors` mode with server support |
| **Unencoded query params** | Broken URLs | `encodeURIComponent` or `URLSearchParams` |
| **Strict Mode double fetch** | Duplicate requests in dev | Abort in cleanup or deduplicate |

### Annotated Code Examples

**Pitfall 1: No `ok` check**

```js
// ❌ Resolves on 404 and crashes on .json()
const data = await fetch('/api/user/1').then((r) => r.json());

// ✅ Check status first
const res = await fetch('/api/user/1');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();
```

**Pitfall 2: Race condition**

```jsx
// ❌ Older response can overwrite newer state
useEffect(() => {
  fetch(`/api/search?q=${query}`).then((r) => r.json()).then(setResults);
}, [query]);

// ✅ Abort the previous request
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/search?q=${encodeURIComponent(query)}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setResults)
    .catch((err) => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [query]);
```

**Pitfall 3: Body used twice**

```js
// ❌ Throws: body already used
const res = await fetch('/api/data');
await res.json();
await res.text();

// ✅ Clone before the first read
const res = await fetch('/api/data');
const copy = res.clone();
await res.json();
await copy.text();
```

**Expected Output:** The correct versions avoid crashes, stale state, and body-consumption errors.

**Why this output:** `response.ok` converts HTTP errors into exceptions. Aborting prevents stale writes. Cloning gives each reader its own stream.

### Real-World Cases

- **Search UIs:** Race conditions from un-aborted requests.
- **Detail pages:** Body-used errors from double parsing.
- **Cross-origin APIs:** Credential and CORS misconfiguration.

### References

- MDN — Response.clone() — https://developer.mozilla.org/en-US/docs/Web/API/Response/clone
- MDN — Response.ok — https://developer.mozilla.org/en-US/docs/Web/API/Response/ok
- React — Strict Mode — https://react.dev/reference/react/StrictMode
- web.dev — Fetch API guide — https://web.dev/articles/fetch-api-guide

---

## 10. Fetch Best Practices

### Definitions

**Core Definition:** Fetch best practices are the guidelines for writing request code that is correct, safe, cancellable, and maintainable.

**Technical Definition:** Best practices combine **status checking** (`response.ok`), **cancellation** (`AbortController` + cleanup), **timeouts** (`AbortSignal.timeout`/`any`), **error taxonomy** (network vs. HTTP vs. parse vs. abort), **safe parsing** (`.text()` then `JSON.parse` when uncertain), **credential and CORS correctness**, **query encoding**, **deduplication and caching** (via TanStack Query/SWR or a small helper), and **layer separation** (a `fetchJson` boundary that maps low-level errors to typed application errors). For anything beyond trivial, prefer a query library over raw `useEffect` + `fetch`.

**Beginner-Friendly Explanation:** Good fetch code checks the status, cancels when the component goes away, times out eventually, and turns low-level errors into clear ones. And for real apps, use a library that handles caching and deduplication instead of writing the same effect over and over.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Always check `response.ok`** | Convert HTTP errors into exceptions at the boundary. |
| **Always pass an `AbortSignal`** | Cancel on unmount and on dependency changes. |
| **Set timeouts** | Use `AbortSignal.timeout()` or `AbortSignal.any()`. |
| **Distinguish error types** | Network, HTTP, parse, abort, timeout. |
| **Ignore `AbortError`** | It is cancellation, not failure. |
| **Parse safely** | Read `.text()` first when the body may not be JSON. |
| **Encode query parameters** | `encodeURIComponent` or `URLSearchParams`. |
| **Configure credentials explicitly** | `credentials: 'include'` only when needed. |
| **Use a single `fetchJson` helper** | Centralise status checks, parsing, and error mapping. |
| **Prefer a query library** | TanStack Query/SWR for caching, dedup, retries. |
| **Avoid waterfalls** | `Promise.all` for independent requests. |
| **Do not fetch in render** | Fetch in effects, handlers, loaders, or Server Components. |

### Annotated Code Examples

```js
// A small, reusable fetch helper that encodes the best practices
export async function fetchJson(url, { timeout = 8000, signal, ...init } = {}) {
  const timeoutSignal = AbortSignal.timeout(timeout);
  const combined = signal ? AbortSignal.any([signal, timeoutSignal]) : timeoutSignal;

  let response;
  try {
    response = await fetch(url, { ...init, signal: combined });
  } catch (error) {
    if (error.name === 'AbortError') throw error;
    if (error.name === 'TimeoutError') throw new Error('Request timed out');
    throw new Error('Network error: unable to reach the server');
  }

  if (!response.ok) {
    const body = await response.json().catch(() => null);
    throw new Error(body?.message ?? `HTTP ${response.status}`);
  }

  if (response.status === 204) return null;
  return response.json();
}
```

**Expected Output:** A consistent API for all requests: parsed JSON on success, `null` for `204`, typed errors for timeouts, network failures, HTTP errors, and preserved `AbortError` for cancellations.

**Why this output:** The helper centralises status checking, timeout handling, safe parsing, and error mapping, so every call site gets the same correct behaviour.

### Real-World Cases

- **All client-side fetching:** These practices apply universally.
- **Design systems:** Shared API clients built on a `fetchJson` helper.
- **Data-heavy apps:** Query libraries built on top of a safe fetch layer.

### References

- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — Response.ok — https://developer.mozilla.org/en-US/docs/Web/API/Response/ok
- MDN — AbortSignal.timeout() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- MDN — AbortSignal.any() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static
- web.dev — Fetch API guide — https://web.dev/articles/fetch-api-guide
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- TanStack Query — Overview — https://tanstack.com/query/latest/docs/framework/react/overview

---

## References

- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — Window: fetch() method — https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch
- MDN — Request — https://developer.mozilla.org/en-US/docs/Web/API/Request
- MDN — RequestInit — https://developer.mozilla.org/en-US/docs/Web/API/RequestInit
- MDN — Response — https://developer.mozilla.org/en-US/docs/Web/API/Response
- MDN — Response.ok — https://developer.mozilla.org/en-US/docs/Web/API/Response/ok
- MDN — Response.clone() — https://developer.mozilla.org/en-US/docs/Web/API/Response/clone
- MDN — Response.json() — https://developer.mozilla.org/en-US/docs/Web/API/Response/json
- MDN — Response.text() — https://developer.mozilla.org/en-US/docs/Web/API/Response/text
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN — AbortSignal — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal
- MDN — AbortSignal.timeout() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- MDN — AbortSignal.any() — https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static
- MDN — DOMException — https://developer.mozilla.org/en-US/docs/Web/API/DOMException
- MDN — CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN — JSON.parse() — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- WHATWG — Fetch Standard — https://fetch.spec.whatwg.org/
- RFC 9110 — HTTP Semantics — https://www.rfc-editor.org/rfc/rfc9110
- React — useEffect — https://react.dev/reference/react/useEffect
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Synchronizing with Effects — https://react.dev/learn/synchronizing-with-effects
- React — Strict Mode — https://react.dev/reference/react/StrictMode
- TanStack Query — Overview — https://tanstack.com/query/latest/docs/framework/react/overview
- web.dev — Fetch API guide — https://web.dev/articles/fetch-api-guide