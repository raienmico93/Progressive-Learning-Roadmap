# HTTP Fundamentals for React: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** HTTP (Hypertext Transfer Protocol) is the application-layer protocol that governs how React applications communicate with servers, defining the methods, status codes, headers, and payload formats used in every network request.

**Technical Definition:** HTTP is a stateless, request-response protocol operating over TCP (or QUIC in HTTP/3). A client (the React application running in the browser) sends an HTTP request consisting of a method, a URL, headers, and an optional body. The server responds with a status code, headers, and an optional body. HTTP/1.1 (RFC 7230–7235), HTTP/2 (RFC 7540), and HTTP/3 (RFC 9114) define the protocol's evolution, with HTTP/2 introducing multiplexing and header compression, and HTTP/3 using QUIC for reduced latency. In React applications, HTTP requests are typically made via the `fetch` API or Axios, with responses managed by server-state libraries such as TanStack Query. Understanding HTTP semantics—method safety and idempotency, status code families, content negotiation, and cross-origin constraints—is essential for building correct, performant, and secure data-fetching layers.

**Beginner-Friendly Explanation:** Every time your React app talks to a server—loading a list of products, saving a form, deleting an item—it sends an HTTP message. That message has a verb (what you want to do), a status code (what happened), headers (extra information), and a body (the data). HTTP is the language both sides agree on. This cheat sheet explains the vocabulary of that language so you can debug network issues, design APIs, and write data-fetching code that behaves predictably.

### Key Characteristics

- **Stateless:** Each request is independent; the server does not remember previous requests unless state is carried in cookies, tokens, or headers.
- **Request-Response:** The client always initiates; the server always responds. No server push exists in HTTP/1.1 (HTTP/2 server push was deprecated in Chrome).
- **Method Semantics:** Methods have defined properties—safety (no side effects) and idempotency (repeating produces the same result)—that clients, caches, and proxies rely on.
- **Status Code Families:** The first digit of a status code categorises the outcome (1xx informational, 2xx success, 3xx redirect, 4xx client error, 5xx server error).
- **Content Negotiation:** Clients declare what they accept (`Accept`) and servers declare what they send (`Content-Type`).
- **Caching:** `Cache-Control`, `ETag`, `Last-Modified`, and `Vary` headers govern how responses are cached and revalidated.
- **Cross-Origin Security:** The Same-Origin Policy restricts cross-origin reads; CORS relaxes it via explicit server headers and preflight requests.
- **Serialisation:** JSON (RFC 8259) is the dominant payload format for REST APIs, serialised with `JSON.stringify` and deserialised with `JSON.parse`.

### Prerequisites

- Solid understanding of JavaScript, including `fetch`, promises, and `async`/`await`.
- Familiarity with React function components and the `useEffect`/`useState` Hooks.
- Working knowledge of TanStack Query or similar server-state libraries.
- Basic understanding of URLs, DNS, and the browser's security model.

### Related Programming Areas

- **REST API Design:** Resource-oriented architecture using HTTP methods and status codes.
- **Server-State Management:** Caching, invalidation, and synchronisation (TanStack Query, SWR).
- **Authentication and Authorization:** Bearer tokens, cookies, and session management.
- **Security:** CORS, CSRF, XSS, and content security policy.
- **Performance:** Caching, compression, HTTP/2 multiplexing, and connection reuse.

### Core Concepts / Features

1. HTTP Methods in a CRUD Paradigm (GET, POST, PUT, PATCH, DELETE)
2. HTTP Status Codes and Semantic Error Handling (2xx, 3xx, 4xx, 5xx)
3. Headers Management (Content-Type, Cache-Control, Accept)
4. JSON Serialisation, Deserialisation, and Payload Structure
5. Browser Network Constraints, CORS, and Preflight Requests

---

## Core Concept 1: HTTP Methods in a CRUD Paradigm (GET, POST, PUT, PATCH, DELETE)

### Definitions

**Core Definition:** HTTP methods are the verbs of the protocol that indicate the desired action on a resource, mapping naturally to the CRUD operations of Create (POST), Read (GET), Update (PUT/PATCH), and Delete (DELETE).

**Technical Definition:** RFC 9110 (HTTP Semantics) defines method properties. A method is **safe** if it does not alter server state (GET, HEAD, OPTIONS, TRACE). A method is **idempotent** if multiple identical requests have the same effect as a single request (GET, HEAD, PUT, DELETE, OPTIONS, TRACE). POST is neither safe nor idempotent. GET retrieves a representation of a resource; POST creates a subordinate resource or triggers processing; PUT replaces the target resource entirely; PATCH applies partial modifications; DELETE removes the target resource. These semantics are not merely conventions—caches, proxies, and browsers rely on them to make correct decisions about retrying, caching, and prefetching.

**Beginner-Friendly Explanation:** HTTP methods are like verbs in a sentence. GET means "give me something." POST means "create something new." PUT means "replace this with that." PATCH means "change part of this." DELETE means "remove this." The CRUD acronym (Create, Read, Update, Delete) maps directly: POST creates, GET reads, PUT/PATCH update, DELETE deletes.

### Purposes

- **GET:** To retrieve a resource representation without side effects.
- **POST:** To create a new resource or submit data for processing.
- **PUT:** To replace an existing resource entirely with the provided representation.
- **PATCH:** To apply a partial modification to an existing resource.
- **DELETE:** To remove a resource from the server.
- **HEAD:** To retrieve response headers without a body (useful for checking existence or freshness).
- **OPTIONS:** To discover communication options for a resource (used in CORS preflight).

### Syntax Rules and Structure

**Method Semantics Table:**

| Method | Safe | Idempotent | Has Body | Typical CRUD | Success Codes |
|---|---|---|---|---|---|
| **GET** | ✅ | ✅ | No | Read | 200, 304 |
| **POST** | ❌ | ❌ | Yes | Create | 201, 200, 204 |
| **PUT** | ❌ | ✅ | Yes | Replace | 200, 204 |
| **PATCH** | ❌ | ❌* | Yes | Update | 200, 204 |
| **DELETE** | ❌ | ✅ | Optional | Delete | 200, 204 |
| **HEAD** | ✅ | ✅ | No | Read metadata | 200, 304 |
| **OPTIONS** | ✅ | ✅ | No | Discover | 200, 204 |

*PATCH is not required to be idempotent by the specification, but can be implemented as idempotent.

**General Syntax (`fetch`):**
```javascript
// GET: retrieve a list of todos
const todos = await fetch('/api/todos').then(r => r.json());

// POST: create a new todo
const created = await fetch('/api/todos', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ title: 'New Todo', completed: false }),
}).then(r => r.json());

// PUT: replace a todo entirely
await fetch('/api/todos/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ title: 'Updated', completed: true }),
});

// PATCH: partially update a todo
await fetch('/api/todos/1', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ completed: true }),
});

// DELETE: remove a todo
await fetch('/api/todos/1', { method: 'DELETE' });
```

**Component Breakdown:**
- `method`: The HTTP verb; defaults to `GET` if omitted.
- `headers`: Metadata about the request (e.g., `Content-Type`).
- `body`: The payload; must be a string (or `FormData`, `Blob`, etc.).
- `JSON.stringify`: Serialises the JavaScript object to a JSON string.

**Syntax Rules:**
- Use `GET` for reads; never include a request body.
- Use `POST` for creates; the server assigns the resource ID and returns it (often in the `Location` header).
- Use `PUT` when the client sends the complete resource representation.
- Use `PATCH` when the client sends only the changed fields.
- Use `DELETE` for removals; the response body is often empty (204).
- Always set `Content-Type: application/json` when sending JSON.
- Do not use `GET` for operations with side effects; browsers, caches, and proxies may prefetch or retry GET requests.

**Constraints and Limitations:**
- `GET` requests have length limits imposed by browsers and servers (typically 2,000–8,000 characters for the URL).
- `PATCH` is not universally supported by all proxies and older servers; verify compatibility.
- `PUT` requires the client to know the complete resource representation; if a field is omitted, it may be deleted.
- `POST` is not idempotent; retrying a POST can create duplicate resources. Use idempotency keys for safe retries.
- `DELETE` is idempotent—deleting an already-deleted resource returns 404 or 204, but the state is the same.

### Annotated Code Example: Full CRUD with React and TanStack Query

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const API = '/api/todos';

// GET — read
async function fetchTodos() {
  const res = await fetch(API);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// POST — create
async function createTodo(newTodo) {
  const res = await fetch(API, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newTodo),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// PUT — replace
async function replaceTodo({ id, ...todo }) {
  const res = await fetch(`${API}/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(todo),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// PATCH — partial update
async function toggleTodo({ id, completed }) {
  const res = await fetch(`${API}/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ completed }),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// DELETE — remove
async function deleteTodo(id) {
  const res = await fetch(`${API}/${id}`, { method: 'DELETE' });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
}

function TodoApp() {
  const queryClient = useQueryClient();

  const { data: todos, isPending, isError, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });

  const invalidate = () => queryClient.invalidateQueries({ queryKey: ['todos'] });

  const createMutation  = useMutation({ mutationFn: createTodo,  onSettled: invalidate });
  const replaceMutation = useMutation({ mutationFn: replaceTodo, onSettled: invalidate });
  const toggleMutation  = useMutation({ mutationFn: toggleTodo,  onSettled: invalidate });
  const deleteMutation  = useMutation({ mutationFn: deleteTodo,  onSettled: invalidate });

  if (isPending) return <p>Loading…</p>;
  if (isError) return <p>Error: {error.message}</p>;

  return (
    <div>
      <button onClick={() => createMutation.mutate({ title: 'New', completed: false })}>
        Add
      </button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleMutation.mutate({ id: todo.id, completed: !todo.completed })}
            />
            {todo.title}
            <button onClick={() => deleteMutation.mutate(todo.id)}>✕</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** A todo list with an "Add" button and per-item toggle/delete controls. Clicking "Add" sends a POST and refetches the list. Toggling a checkbox sends a PATCH with only `completed`. Deleting sends a DELETE and refetches.

**Why This Output Occurs:** Each mutation uses the semantically correct HTTP method for its CRUD operation. `onSettled` invalidates the `['todos']` query after every mutation, so the list stays in sync with the server. GET is used for reads only; POST for creates; PATCH for partial updates (only the changed field); DELETE for removals.

### Real-World Cases

- **REST APIs:** Most REST APIs map CRUD to the five methods, with resource-oriented URLs (`/users/123`).
- **GraphQL:** Typically uses POST for all operations, with the query/mutation distinction in the body.
- **Form submissions:** HTML forms only support GET and POST; JavaScript `fetch` unlocks the full method set.
- **Bulk operations:** `PATCH /users` with an array body for batch updates; `DELETE /users?ids=1,2,3` for batch deletes.
- **Idempotent retries:** `PUT` and `DELETE` can be safely retried; `POST` requires idempotency keys.

### References

- RFC 9110 – HTTP Semantics (Methods): https://www.rfc-editor.org/rfc/rfc9110.html#name-methods
- MDN Web Docs – HTTP request methods: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN Web Docs – Idempotent: https://developer.mozilla.org/en-US/docs/Glossary/Idempotent
- RFC 5789 – PATCH Method for HTTP: https://www.rfc-editor.org/rfc/rfc5789.html

---

## Core Concept 2: HTTP Status Codes and Semantic Error Handling (2xx, 3xx, 4xx, 5xx)

### Definitions

**Core Definition:** HTTP status codes are three-digit integers in the response that indicate the outcome of a request, grouped into five families by their first digit—each family communicating a distinct category of result.

**Technical Definition:** RFC 9110 defines status code classes. **1xx (Informational):** The request was received and processing continues (e.g., 100 Continue, 101 Switching Protocols). **2xx (Success):** The request was successfully received, understood, and accepted (200 OK, 201 Created, 202 Accepted, 204 No Content, 206 Partial Content). **3xx (Redirection):** Further action is needed to complete the request (301 Moved Permanently, 302 Found, 304 Not Modified, 307 Temporary Redirect, 308 Permanent Redirect). **4xx (Client Error):** The request contains bad syntax or cannot be fulfilled (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 405 Method Not Allowed, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests). **5xx (Server Error):** The server failed to fulfil a valid request (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout).

**Beginner-Friendly Explanation:** Status codes are the server's way of saying what happened. **2xx** means "all good"—here's your data. **3xx** means "go somewhere else"—the resource moved. **4xx** means "you did something wrong"—the request was bad. **5xx** means "I did something wrong"—the server has a problem. The first digit tells you the category; the other two tell you the specifics.

### Purposes

- **2xx:** To confirm that the request succeeded and, for creates, to return the new resource's URL.
- **3xx:** To redirect clients to a new location or to indicate that a cached response is still fresh.
- **4xx:** To inform the client that the request was invalid, unauthorised, forbidden, or not found.
- **5xx:** To inform the client that the server encountered an error and the request may be retried.
- **Semantic Error Handling:** To enable clients to distinguish retryable errors (5xx, 429, 408) from non-retryable ones (4xx) and to map errors to user-facing messages.

### Syntax Rules and Structure

**Common Status Codes Table:**

| Code | Name | Meaning | React Handling |
|---|---|---|---|
| **200** | OK | Success with body | Parse JSON, render data |
| **201** | Created | Resource created | Read `Location` header; invalidate queries |
| **204** | No Content | Success, no body | Do not call `.json()`; invalidate queries |
| **301** | Moved Permanently | Resource moved permanently | Update bookmarks; follow redirect |
| **304** | Not Modified | Cached copy is fresh | Use cache; no body |
| **400** | Bad Request | Malformed request | Show validation errors |
| **401** | Unauthorized | Authentication required | Redirect to login |
| **403** | Forbidden | Authenticated but not allowed | Show "access denied" |
| **404** | Not Found | Resource does not exist | Show "not found" |
| **409** | Conflict | State conflict (e.g., duplicate) | Show conflict message |
| **422** | Unprocessable Entity | Validation failed | Map errors to fields |
| **429** | Too Many Requests | Rate limited | Back off; respect `Retry-After` |
| **500** | Internal Server Error | Server bug | Show generic error; report |
| **502** | Bad Gateway | Upstream failure | Retry with backoff |
| **503** | Service Unavailable | Server overloaded/maintenance | Retry with backoff; respect `Retry-After` |
| **504** | Gateway Timeout | Upstream timeout | Retry with backoff |

**Semantic Error Handling Pattern (`fetch`):**
```javascript
async function fetchJSON(url, options = {}) {
  const res = await fetch(url, options);

  if (res.status === 204) return null; // No content — do not parse

  const body = await res.json().catch(() => null);

  if (!res.ok) {
    const error = new Error(body?.message ?? `HTTP ${res.status}`);
    error.status = res.status;
    error.body = body;
    error.retryable =
      res.status >= 500 || res.status === 429 || res.status === 408;
    throw error;
  }

  return body;
}
```

**Component Breakdown:**
- `res.status === 204`: Returns `null` for no-content responses (do not call `.json()`).
- `res.ok`: `true` for 2xx statuses; `false` otherwise.
- `error.status`: Preserves the status code for downstream logic.
- `error.retryable`: Marks 5xx, 429, and 408 as retryable.

**TanStack Query Retry Configuration:**
```javascript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Do not retry 4xx errors (except 408 and 429)
        if (error.status >= 400 && error.status < 500) {
          return error.status === 408 || error.status === 429
            ? failureCount < 3
            : false;
        }
        return failureCount < 3; // Retry 5xx up to 3 times
      },
    },
  },
});
```

**Component Breakdown:**
- `retry`: A function receiving `failureCount` and `error`.
- 4xx errors are not retried (except 408 Request Timeout and 429 Too Many Requests).
- 5xx errors are retried up to 3 times with exponential backoff.

**Syntax Rules:**
- Always check `res.ok` before parsing JSON.
- Handle 204 explicitly—it has no body.
- Distinguish retryable (5xx, 429, 408) from non-retryable (4xx) errors.
- Respect the `Retry-After` header for 429 and 503 responses.
- Map status codes to user-facing messages: 401 → "Please log in", 403 → "Access denied", 404 → "Not found", 5xx → "Something went wrong".
- Throw errors with the status code attached so callers can branch on it.

**Constraints and Limitations:**
- `fetch` does not reject on HTTP error statuses; it only rejects on network failure. Always check `res.ok`.
- Some APIs return 200 with an error body; check the body structure, not just the status.
- 3xx redirects are followed automatically by `fetch` in `follow` mode (the default); use `redirect: 'manual'` to handle them yourself.
- 401 vs. 403 is often confused: 401 means "not authenticated"; 403 means "authenticated but not authorised".
- 422 is not universally used; some APIs use 400 for all validation errors.

### Annotated Code Example: Semantic Error Handling in TanStack Query

```jsx
import { useQuery } from '@tanstack/react-query';

class ApiError extends Error {
  constructor(status, body) {
    super(body?.message ?? `HTTP ${status}`);
    this.status = status;
    this.body = body;
    this.retryable = status >= 500 || status === 429 || status === 408;
  }
}

async function fetchJSON(url) {
  const res = await fetch(url);

  if (res.status === 204) return null;

  const body = await res.json().catch(() => null);

  if (!res.ok) throw new ApiError(res.status, body);

  return body;
}

function UserProfile({ userId }) {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchJSON(`/api/users/${userId}`),
    retry: (count, err) => err.retryable && count < 3,
  });

  if (isPending) return <p>Loading…</p>;

  if (isError) {
    if (error.status === 404) return <p>User not found.</p>;
    if (error.status === 401) return <p>Please log in.</p>;
    if (error.status === 403) return <p>Access denied.</p>;
    if (error.status >= 500) return <p>Server error. Please try again.</p>;
    return <p>Error: {error.message}</p>;
  }

  return <h1>{data.name}</h1>;
}
```

**Expected Output:** Loading a valid user shows their name. A 404 shows "User not found." A 401 shows "Please log in." A 5xx shows "Server error. Please try again." and retries up to 3 times.

**Why This Output Occurs:** The `ApiError` class preserves the status code and marks retryable errors. The `retry` function only retries when `error.retryable` is true. The component branches on `error.status` to render the appropriate message. This separates the concerns of error detection (in the fetch layer) and error presentation (in the component).

### Real-World Cases

- **Authentication:** 401 triggers a redirect to login; 403 shows an access-denied page.
- **Resource not found:** 404 renders a "not found" page with a link back to the list.
- **Rate limiting:** 429 triggers exponential backoff with `Retry-After` respect.
- **Server outages:** 503 shows a maintenance banner and retries in the background.
- **Validation:** 422 responses map `{ errors: { field: message } }` to form fields via `setError`.
- **Optimistic updates:** 409 conflicts trigger rollback of optimistic UI.

### References

- RFC 9110 – HTTP Semantics (Status Codes): https://www.rfc-editor.org/rfc/rfc9110.html#name-status-codes
- MDN Web Docs – HTTP response status codes: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- RFC 6585 – Additional HTTP Status Codes (429): https://www.rfc-editor.org/rfc/rfc6585.html
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries

---

## Core Concept 3: Headers Management (Content-Type, Cache-Control, Accept)

### Definitions

**Core Definition:** HTTP headers are key-value pairs sent with requests and responses that carry metadata—such as the format of the body, caching directives, and content negotiation preferences—separate from the payload itself.

**Technical Definition:** RFC 9110 defines header fields as case-insensitive names followed by values. **Representation headers** describe the body: `Content-Type` (media type, e.g., `application/json`), `Content-Length` (byte length), `Content-Encoding` (e.g., `gzip`, `br`). **Content negotiation headers** let the client declare preferences: `Accept` (media types), `Accept-Language`, `Accept-Encoding`. **Caching headers** control cache behaviour: `Cache-Control` (directives like `max-age`, `no-store`, `no-cache`, `stale-while-revalidate`), `ETag` (validator), `Last-Modified`, `Vary` (which request headers affect the response), `Expires`. **Authentication headers:** `Authorization` (e.g., `Bearer <token>`), `WWW-Authenticate`. **CORS headers:** `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Max-Age`. In `fetch`, headers are set via the `headers` option and read via `res.headers.get(name)`.

**Beginner-Friendly Explanation:** Headers are like the labels on a package. `Content-Type` says "this box contains JSON." `Accept` says "I can only accept JSON." `Cache-Control` says "keep this in the warehouse for 5 minutes." `Authorization` says "here's my ID badge." Without headers, the server and client would have to guess what's inside the package and how to handle it.

### Purposes

- **Content-Type:** To declare the media type of the request or response body.
- **Accept:** To declare which media types the client can handle.
- **Cache-Control:** To control how responses are cached, revalidated, and served.
- **Authorization:** To carry authentication credentials (e.g., bearer tokens).
- **ETag / If-None-Match:** To enable conditional requests and 304 responses.
- **Vary:** To tell caches which request headers affect the response.
- **CORS headers:** To permit or deny cross-origin requests.

### Syntax Rules and Structure

**Setting Request Headers (`fetch`):**
```javascript
const res = await fetch('/api/todos', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'Authorization': `Bearer ${token}`,
    'X-Request-Id': crypto.randomUUID(),
  },
  body: JSON.stringify({ title: 'New' }),
});
```

**Component Breakdown:**
- `Content-Type`: The format of the request body.
- `Accept`: The formats the client can handle.
- `Authorization`: The bearer token for authentication.
- `X-Request-Id`: A custom header for tracing.

**Reading Response Headers:**
```javascript
const res = await fetch('/api/todos');

const contentType = res.headers.get('Content-Type'); // "application/json; charset=utf-8"
const cacheControl = res.headers.get('Cache-Control'); // "max-age=300, stale-while-revalidate=60"
const etag = res.headers.get('ETag'); // "\"abc123\""
const location = res.headers.get('Location'); // "/api/todos/42" (on 201)
```

**Caching with `Cache-Control`:**
```
Cache-Control: max-age=300, stale-while-revalidate=60
```
- `max-age=300`: The response is fresh for 300 seconds.
- `stale-while-revalidate=60`: After 300 seconds, the stale response can be served for up to 60 seconds while a revalidation happens in the background.
- `no-store`: Never cache.
- `no-cache`: Cache but revalidate before every use.
- `private`: Only the browser may cache, not shared caches.
- `public`: Shared caches may cache.

**Conditional Requests with `ETag`:**
```javascript
// First request: server returns ETag
const first = await fetch('/api/user');
const etag = first.headers.get('ETag');

// Subsequent request: client sends If-None-Match
const second = await fetch('/api/user', {
  headers: { 'If-None-Match': etag },
});

if (second.status === 304) {
  // Cached copy is still fresh; use it
}
```

**CORS Headers (Server Response):**
```
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

**Syntax Rules:**
- Always set `Content-Type: application/json` when sending JSON.
- Always set `Accept: application/json` to signal the expected response format.
- Use `Authorization: Bearer <token>` for token-based authentication.
- Use `Cache-Control: no-store` for sensitive data (tokens, personal info).
- Use `ETag` + `If-None-Match` for efficient revalidation of large resources.
- Use `Vary: Accept-Encoding` when the response varies by encoding.
- Header names are case-insensitive, but convention is kebab-case (`Content-Type`).
- Do not set `Content-Type` manually when sending `FormData`; the browser sets it with the correct multipart boundary.

**Constraints and Limitations:**
- Some headers are **forbidden** in `fetch` and cannot be set by JavaScript (`Host`, `Connection`, `Content-Length`, `Cookie`, `Set-Cookie`).
- Custom headers (e.g., `X-Request-Id`) trigger a CORS preflight request.
- `Cache-Control` is a hint; caches may ignore it under certain conditions.
- `ETag` values must be quoted strings.
- `Authorization` headers are not sent on cross-origin redirects by default for security reasons.
- Browsers cap the size of headers (typically 8–16 KB total).

### Annotated Code Example: Authenticated Fetch Wrapper with Caching

```javascript
class HttpClient {
  constructor(baseURL, getToken) {
    this.baseURL = baseURL;
    this.getToken = getToken;
  }

  async request(path, { method = 'GET', body, headers = {}, ...rest } = {}) {
    const token = this.getToken();

    const res = await fetch(`${this.baseURL}${path}`, {
      method,
      headers: {
        'Accept': 'application/json',
        ...(body ? { 'Content-Type': 'application/json' } : {}),
        ...(token ? { 'Authorization': `Bearer ${token}` } : {}),
        ...headers,
      },
      ...(body ? { body: JSON.stringify(body) } : {}),
      ...rest,
    });

    if (res.status === 204) return null;

    const data = await res.json().catch(() => null);

    if (!res.ok) {
      const error = new Error(data?.message ?? `HTTP ${res.status}`);
      error.status = res.status;
      error.data = data;
      throw error;
    }

    return data;
  }

  get(path, options) { return this.request(path, { ...options, method: 'GET' }); }
  post(path, body, options) { return this.request(path, { ...options, method: 'POST', body }); }
  put(path, body, options) { return this.request(path, { ...options, method: 'PUT', body }); }
  patch(path, body, options) { return this.request(path, { ...options, method: 'PATCH', body }); }
  delete(path, options) { return this.request(path, { ...options, method: 'DELETE' }); }
}

// Usage
const api = new HttpClient('/api', () => localStorage.getItem('token'));

const todos = await api.get('/todos');
const created = await api.post('/todos', { title: 'New', completed: false });
```

**Expected Output:** A reusable HTTP client that automatically sets `Accept`, `Content-Type` (when a body is present), and `Authorization` headers. It handles 204 responses, parses JSON, and throws structured errors with status codes.

**Why This Output Occurs:** The `HttpClient` class centralises header management. The `Accept` header is always set; `Content-Type` is only set when a body exists (avoiding the error of setting it on GET requests); `Authorization` is set when a token exists. The error object carries the status code and parsed body for downstream handling.

### Real-World Cases

- **Authentication:** Sending `Authorization: Bearer <token>` on every authenticated request.
- **API versioning:** Using `Accept: application/vnd.api+json; version=2` for versioned APIs.
- **Caching:** Setting `Cache-Control: max-age=3600` on static assets; `no-store` on sensitive endpoints.
- **Tracing:** Sending `X-Request-Id` for distributed tracing; servers log it and return it.
- **Content negotiation:** Using `Accept-Language` to return localised content.
- **Idempotency:** Sending `Idempotency-Key` on POST requests to enable safe retries.

### References

- RFC 9110 – HTTP Semantics (Fields): https://www.rfc-editor.org/rfc/rfc9110.html#name-header-fields
- MDN Web Docs – HTTP headers: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers
- MDN Web Docs – Cache-Control: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
- MDN Web Docs – Content-Type: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type
- MDN Web Docs – Accept: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
- RFC 9111 – HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111.html

---

## Core Concept 4: JSON Serialisation, Deserialisation, and Payload Structure

### Definitions

**Core Definition:** JSON (JavaScript Object Notation) is the text-based data-interchange format used by most REST APIs, serialised in React with `JSON.stringify` and deserialised with `JSON.parse`, with payload structure determining how clients and servers interpret the data.

**Technical Definition:** JSON is defined by RFC 8259 (STD 90) and ECMA-404. It supports six value types: object, array, string, number, boolean, and null. Strings must be double-quoted; numbers cannot be `NaN` or `Infinity`; objects cannot have trailing commas or comments. `JSON.stringify(value, replacer, space)` converts a JavaScript value to a JSON string; `JSON.parse(text, reviver)` converts a JSON string to a JavaScript value. In React, `fetch` does not automatically serialise or parse JSON—the developer must call `JSON.stringify` before sending and `res.json()` after receiving. Common payload structures for REST APIs include single resources (`{ id, ...fields }`), collections (`[ { ... }, { ... } ]` or `{ data: [...], meta: { ... } }`), and error envelopes (`{ error: { code, message, details } }`).

**Beginner-Friendly Explanation:** JSON is a way to write data as text so both the browser and the server can read it. JavaScript objects are converted to JSON strings with `JSON.stringify` before sending, and JSON strings are converted back to objects with `JSON.parse` after receiving. The shape of the JSON—whether it's a single object, an array, or a wrapper with `data` and `meta`—is called the payload structure. A consistent payload structure makes the client code predictable.

### Purposes

- **Serialisation:** To convert a JavaScript object into a JSON string for transmission.
- **Deserialisation:** To convert a JSON string from the server into a JavaScript object.
- **Payload Structure:** To define a consistent shape for resources, collections, and errors.
- **Error Envelopes:** To standardise how errors are returned so clients can handle them uniformly.
- **Pagination Metadata:** To include `total`, `page`, `pageSize`, and `nextCursor` alongside collection data.
- **Versioning:** To include API version information in the payload or headers.

### Syntax Rules and Structure

**Serialisation:**
```javascript
const todo = { id: 1, title: 'Buy milk', completed: false, dueDate: new Date('2024-12-31') };

// Basic
const json = JSON.stringify(todo);
// '{"id":1,"title":"Buy milk","completed":false,"dueDate":"2024-12-31T00:00:00.000Z"}'

// With replacer (omit sensitive fields)
const safe = JSON.stringify(todo, (key, value) =>
  key === 'password' ? undefined : value
);

// With indentation (for debugging)
const pretty = JSON.stringify(todo, null, 2);
```

**Deserialisation:**
```javascript
const json = '{"id":1,"title":"Buy milk","completed":false}';

const todo = JSON.parse(json);
// { id: 1, title: 'Buy milk', completed: false }

// With reviver (convert date strings to Date objects)
const withDates = JSON.parse(json, (key, value) =>
  key === 'dueDate' ? new Date(value) : value
);
```

**Standard Payload Structures:**

**Single Resource:**
```json
{
  "id": 1,
  "title": "Buy milk",
  "completed": false,
  "createdAt": "2024-12-01T10:00:00Z",
  "updatedAt": "2024-12-02T14:30:00Z"
}
```

**Collection (Array):**
```json
[
  { "id": 1, "title": "Buy milk", "completed": false },
  { "id": 2, "title": "Walk dog", "completed": true }
]
```

**Collection with Metadata (JSON:API style):**
```json
{
  "data": [
    { "id": 1, "title": "Buy milk", "completed": false },
    { "id": 2, "title": "Walk dog", "completed": true }
  ],
  "meta": {
    "total": 42,
    "page": 1,
    "pageSize": 10,
    "nextCursor": "eyJpZCI6MTB9"
  }
}
```

**Error Envelope (RFC 9457 Problem Details):**
```json
{
  "type": "https://example.com/errors/validation",
  "title": "Validation failed",
  "status": 422,
  "detail": "One or more fields are invalid.",
  "errors": {
    "email": ["Email is required"],
    "password": ["Password must be at least 8 characters"]
  }
}
```

**Syntax Rules:**
- Always `JSON.stringify` the body before sending; `fetch` does not serialise automatically.
- Always set `Content-Type: application/json` when sending JSON.
- Always `await res.json()` to parse the response; `fetch` does not deserialise automatically.
- Handle `204 No Content` before calling `res.json()`.
- Use a consistent payload structure across all endpoints.
- Include `createdAt` and `updatedAt` timestamps on resources.
- Use a consistent error envelope with `status`, `message`, and `errors` fields.
- Never include sensitive data (passwords, tokens) in serialised payloads.

**Constraints and Limitations:**
- JSON does not support `undefined`, `NaN`, `Infinity`, `Date`, `Map`, `Set`, or `BigInt` natively; convert them to supported types first.
- `JSON.stringify` throws on circular references.
- `JSON.parse` throws on malformed JSON; always wrap in `try/catch` or use `.catch(() => null)`.
- `fetch`'s `res.json()` consumes the response body; calling it twice throws.
- Large payloads should be paginated; avoid returning unbounded arrays.
- JSON is more verbose than binary formats (Protobuf, MessagePack); use compression (`gzip`, `br`) for large payloads.

### Annotated Code Example: Typed API Client with Payload Validation

```typescript
import { z } from 'zod';

// Define the payload schema
const TodoSchema = z.object({
  id: z.number(),
  title: z.string(),
  completed: z.boolean(),
  createdAt: z.string().datetime(),
});

const TodoListSchema = z.object({
  data: z.array(TodoSchema),
  meta: z.object({
    total: z.number(),
    page: z.number(),
    pageSize: z.number(),
  }),
});

type Todo = z.infer<typeof TodoSchema>;
type TodoList = z.infer<typeof TodoListSchema>;

// API client that validates payloads
async function fetchTodos(page = 1): Promise<TodoList> {
  const res = await fetch(`/api/todos?page=${page}`, {
    headers: { 'Accept': 'application/json' },
  });

  if (!res.ok) {
    const body = await res.json().catch(() => null);
    throw new Error(body?.message ?? `HTTP ${res.status}`);
  }

  const json = await res.json();
  const parsed = TodoListSchema.safeParse(json);

  if (!parsed.success) {
    console.error('Invalid payload:', parsed.error.flatten());
    throw new Error('Server returned an invalid payload');
  }

  return parsed.data;
}

async function createTodo(input: { title: string }): Promise<Todo> {
  const res = await fetch('/api/todos', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    },
    body: JSON.stringify(input),
  });

  if (!res.ok) {
    const body = await res.json().catch(() => null);
    throw new Error(body?.message ?? `HTTP ${res.status}`);
  }

  const json = await res.json();
  return TodoSchema.parse(json); // Throws if invalid
}
```

**Expected Output:** `fetchTodos` returns a validated `TodoList` object with typed `data` and `meta`. If the server returns an unexpected shape, the function throws "Server returned an invalid payload" and logs the Zod error. `createTodo` returns a validated `Todo`.

**Why This Output Occurs:** The Zod schema defines the expected payload structure. `safeParse` returns a result object rather than throwing, allowing graceful error handling. `parse` throws on invalid data, which is appropriate when the caller expects a valid response and wants to fail fast. The `Accept` and `Content-Type` headers ensure the server returns and accepts JSON.

### Real-World Cases

- **REST APIs:** Single resources, collections, and error envelopes following JSON:API or custom conventions.
- **GraphQL:** Always POST with `{ query, variables, operationName }` and receive `{ data, errors }`.
- **Pagination:** `{ data, meta: { total, page, pageSize, nextCursor } }` for cursor-based or offset-based pagination.
- **Validation errors:** `{ status: 422, errors: { field: [messages] } }` mapped to form fields.
- **Bulk operations:** `{ data: [ ... ], meta: { succeeded, failed, errors } }` for batch results.
- **File metadata:** `{ id, filename, size, mimeType, url }` returned after upload.

### References

- RFC 8259 – The JavaScript Object Notation (JSON) Data Interchange Format: https://www.rfc-editor.org/rfc/rfc8259.html
- ECMA-404 – The JSON Data Interchange Syntax: https://www.ecma-international.org/publications-and-standards/standards/ecma-404/
- MDN Web Docs – JSON.stringify(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
- MDN Web Docs – JSON.parse(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- RFC 9457 – Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457.html
- Zod – Documentation: https://zod.dev/

---

## Core Concept 5: Browser Network Constraints, CORS, and Preflight Requests

### Definitions

**Core Definition:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism that uses HTTP headers to permit or deny cross-origin requests, with preflight requests (OPTIONS) checking permissions before the actual request is sent.

**Technical Definition:** The **Same-Origin Policy (SOP)** restricts how a document or script loaded from one origin can interact with resources from another origin. An origin is defined by the triple `(scheme, host, port)`. CORS is the W3C (now WHATWG) mechanism that relaxes SOP for controlled cross-origin access. A **simple request** (GET, HEAD, POST with `Content-Type` of `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`, and no custom headers) is sent directly. A **preflighted request** (any other method or content type, or with custom headers) is preceded by an OPTIONS request that asks the server for permission. The server responds with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, and optionally `Access-Control-Allow-Credentials` and `Access-Control-Max-Age`. The browser then decides whether to send the actual request. If the response lacks the required headers, the browser blocks the response from reaching the JavaScript—even though the server may have processed the request.

**Beginner-Friendly Explanation:** Browsers have a security rule: a page from `app.example.com` cannot read responses from `api.other.com` unless `api.other.com` explicitly says "I allow `app.example.com`." That's CORS. Before sending certain requests (like PUT or POST with JSON), the browser sends a small "preflight" request (OPTIONS) to ask the server: "Will you allow this?" If the server says yes, the browser sends the real request. If the server doesn't answer correctly, the browser blocks the response.

### Purposes

- To enable controlled cross-origin API access from browsers.
- To prevent malicious sites from reading sensitive data from other origins.
- To allow servers to whitelist specific origins, methods, and headers.
- To reduce preflight overhead with `Access-Control-Max-Age`.
- To support credentialed requests (cookies, `Authorization` headers) with `Access-Control-Allow-Credentials`.

### Syntax Rules and Structure

**Simple Request (No Preflight):**
```javascript
// GET with no custom headers — simple, no preflight
fetch('https://api.other.com/data');
```

**Preflighted Request:**
```javascript
// PUT with JSON and custom headers — triggers preflight
fetch('https://api.other.com/data/1', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'X-Request-Id': 'abc-123',
  },
  body: JSON.stringify({ name: 'Updated' }),
});
```

The browser sends:
```
OPTIONS /data/1 HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: content-type, x-request-id
```

The server must respond:
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, X-Request-Id
Access-Control-Max-Age: 86400
```

Then the browser sends the actual PUT request.

**Credentialed Request:**
```javascript
fetch('https://api.other.com/me', {
  credentials: 'include', // Send cookies
  headers: { 'Authorization': `Bearer ${token}` },
});
```

Server must respond with:
```
Access-Control-Allow-Origin: https://app.example.com  // NOT *
Access-Control-Allow-Credentials: true
```

**Syntax Rules:**
- `Access-Control-Allow-Origin` must be a specific origin (not `*`) when `Access-Control-Allow-Credentials: true`.
- Preflight responses should return 204 No Content (or 200 with a body).
- `Access-Control-Max-Age` caches the preflight result (in seconds) to reduce OPTIONS requests.
- `Access-Control-Allow-Methods` must include the requested method.
- `Access-Control-Allow-Headers` must include all custom headers.
- `credentials: 'include'` is required on the client for cookies to be sent cross-origin.
- `Vary: Origin` should be set when the response depends on the `Origin` header (for shared caches).

**Constraints and Limitations:**
- CORS is enforced by the browser, not the server. The server may process the request even if the browser blocks the response.
- Preflight requests add latency (an extra round trip); use `Access-Control-Max-Age` to cache them.
- `Access-Control-Allow-Origin: *` cannot be used with credentials.
- Custom headers (e.g., `Authorization`, `X-Request-Id`) always trigger a preflight.
- Some simple requests become preflighted if they use `Content-Type: application/json` (which is not a "simple" content type).
- CORS does not protect against CSRF; use CSRF tokens or `SameSite` cookies.
- Browser DevTools may show "CORS error" even when the server responded correctly; the error means the browser blocked the response, not that the server failed.

### Annotated Code Example: Diagnosing and Fixing a CORS Error

```javascript
// ❌ Frontend: request that triggers a CORS error
fetch('https://api.other.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json', // Not a "simple" content type
    'Authorization': 'Bearer token123', // Custom header
  },
  body: JSON.stringify({ name: 'Test' }),
});

// Error in console:
// Access to fetch at 'https://api.other.com/data' from origin
// 'https://app.example.com' has been blocked by CORS policy:
// Response to preflight request doesn't pass access control check:
// No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

**The Fix (Server-Side — Express):**
```javascript
import cors from 'cors';
import express from 'express';

const app = express();

app.use(cors({
  origin: 'https://app.example.com', // Specific origin, not '*'
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-Id'],
  credentials: true, // Allow cookies and Authorization headers
  maxAge: 86400,      // Cache preflight for 24 hours
}));

app.use(express.json());

app.post('/data', (req, res) => {
  res.json({ received: req.body });
});

app.listen(3000);
```

**The Fix (Server-Side — Manual Headers):**
```javascript
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const allowed = ['https://app.example.com', 'https://staging.example.com'];

  if (allowed.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Vary', 'Origin');
  }

  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Request-Id');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Max-Age', '86400');

  if (req.method === 'OPTIONS') {
    return res.sendStatus(204); // Preflight response
  }

  next();
});
```

**Expected Output:** The preflight OPTIONS request receives a 204 response with the correct CORS headers. The browser then sends the actual POST request. The server responds with `Access-Control-Allow-Origin: https://app.example.com`, and the browser allows the JavaScript to read the response.

**Why This Output Occurs:** The `cors` middleware (or manual headers) adds the required `Access-Control-Allow-*` headers to the response. The browser checks these headers against the request's `Origin`, `Access-Control-Request-Method`, and `Access-Control-Request-Headers`. If all match, the browser sends the actual request. The `Vary: Origin` header ensures that shared caches do not serve the wrong `Access-Control-Allow-Origin` to different origins.

### Real-World Cases

- **Frontend/backend separation:** A React app on `app.example.com` calling an API on `api.example.com`.
- **Third-party APIs:** Calling Stripe, Auth0, or GitHub APIs from the browser.
- **CDN assets:** Loading fonts or images from a CDN with `Access-Control-Allow-Origin`.
- **Local development:** A React dev server on `localhost:3000` calling an API on `localhost:4000`; configure CORS on the API or use a proxy.
- **Microservices:** Multiple services on different subdomains calling each other from the browser.
- **GraphQL:** Usually POST with `Content-Type: application/json`, triggering preflight.

### References

- MDN Web Docs – Cross-Origin Resource Sharing (CORS): https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- MDN Web Docs – Same-origin policy: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
- Fetch Living Standard – CORS protocol: https://fetch.spec.whatwg.org/#http-cors-protocol
- MDN Web Docs – Access-Control-Allow-Origin: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin
- MDN Web Docs – Preflight request: https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request
- Express cors middleware: https://expressjs.com/en/resources/middleware/cors.html

---

## Comparison and Decision Guidance

| Concern | Tool/Header | When to Use | Common Pitfall |
|---|---|---|---|
| **Read data** | `GET` | Retrieving resources | Including a body; using for side effects |
| **Create resource** | `POST` | Creating new resources | Retrying without idempotency keys |
| **Replace resource** | `PUT` | Full replacement | Omitting fields (they get deleted) |
| **Partial update** | `PATCH` | Updating specific fields | Assuming idempotency |
| **Delete resource** | `DELETE` | Removing resources | Not handling 404 gracefully |
| **Success** | 2xx | Confirming outcomes | Not checking `res.ok` |
| **Redirect** | 3xx | Moving resources | Not handling `Location` |
| **Client error** | 4xx | Invalid requests | Retrying non-retryable errors |
| **Server error** | 5xx | Server failures | Not retrying or backing off |
| **Body format** | `Content-Type` | Declaring media type | Setting manually for FormData |
| **Response format** | `Accept` | Content negotiation | Forgetting it; server returns wrong type |
| **Caching** | `Cache-Control` | Controlling caches | Caching sensitive data |
| **Revalidation** | `ETag` + `If-None-Match` | Efficient freshness checks | Not quoting ETag values |
| **JSON send** | `JSON.stringify` | Serialising payloads | Forgetting `Content-Type` |
| **JSON receive** | `res.json()` | Deserialising payloads | Calling `.json()` on 204 |
| **Cross-origin** | CORS headers | Browser API calls | `*` with credentials |
| **Preflight** | OPTIONS | Non-simple requests | No `Access-Control-Max-Age` |

**Decision Guidance:**
- **Map CRUD to methods:** GET reads, POST creates, PUT replaces, PATCH updates, DELETE removes.
- **Always check `res.ok`:** `fetch` only rejects on network failure, not HTTP errors.
- **Handle 204 explicitly:** Do not call `.json()` on a 204 response.
- **Distinguish retryable errors:** Retry 5xx, 429, and 408; do not retry 4xx.
- **Set `Content-Type` and `Accept`:** Both are required for JSON APIs.
- **Use `Cache-Control` deliberately:** `no-store` for sensitive data; `max-age` for static assets.
- **Validate payloads with Zod:** Catch shape mismatches at the boundary.
- **Configure CORS on the server:** Use specific origins, not `*`, when credentials are involved.
- **Cache preflights with `Access-Control-Max-Age`:** Reduces OPTIONS overhead.
- **Never trust client-side validation alone:** Always validate on the server.

---

## References

- RFC 9110 – HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110.html
- RFC 9111 – HTTP Caching: https://www.rfc-editor.org/rfc/rfc9111.html
- RFC 9114 – HTTP/3: https://www.rfc-editor.org/rfc/rfc9114.html
- RFC 8259 – JSON Data Interchange Format: https://www.rfc-editor.org/rfc/rfc8259.html
- RFC 9457 – Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457.html
- RFC 5789 – PATCH Method for HTTP: https://www.rfc-editor.org/rfc/rfc5789.html
- RFC 6585 – Additional HTTP Status Codes: https://www.rfc-editor.org/rfc/rfc6585.html
- ECMA-404 – JSON Data Interchange Syntax: https://www.ecma-international.org/publications-and-standards/standards/ecma-404/
- MDN Web Docs – HTTP request methods: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN Web Docs – HTTP response status codes: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- MDN Web Docs – HTTP headers: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers
- MDN Web Docs – Cache-Control: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
- MDN Web Docs – Content-Type: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type
- MDN Web Docs – Accept: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
- MDN Web Docs – Cross-Origin Resource Sharing (CORS): https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- MDN Web Docs – Same-origin policy: https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
- MDN Web Docs – JSON.stringify(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
- MDN Web Docs – JSON.parse(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
- Fetch Living Standard – CORS protocol: https://fetch.spec.whatwg.org/#http-cors-protocol
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- Express cors middleware: https://expressjs.com/en/resources/middleware/cors.html
- Zod – Documentation: https://zod.dev/