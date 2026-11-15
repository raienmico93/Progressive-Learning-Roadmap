# Axios — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Axios is a Promise-based HTTP client for the browser and Node.js that provides a simpler, more feature-rich alternative to the native `fetch` API, with built-in interceptors, automatic JSON handling, and a consistent error model.

**Technical Definition:** Axios is an open-source JavaScript library that makes HTTP requests from browsers (via `XMLHttpRequest`) and Node.js (via `http`). It exposes a single `axios(config)` function, method aliases (`axios.get`, `axios.post`, etc.), and a `create()` factory for reusable instances. Its defining features are **request and response interceptors** (functions that run before a request is sent or after a response is received), **automatic JSON serialization and parsing**, **a structured error object** (`AxiosError`) that rejects on `4xx`/`5xx` status codes, and **cancellation** via `AbortController` (recommended) or the deprecated `CancelToken`. Axios automatically protects against XSRF and supports progress events, timeouts, and custom transforms. The library adds roughly **12–15 KB gzipped** to a bundle, but its feature set reduces boilerplate and centralises cross-cutting concerns in a way that `fetch` requires manual work to achieve.

**Beginner-Friendly Explanation:** Axios is a tool that makes talking to servers easier. Instead of writing the same status-checking, JSON-parsing, and error-handling code every time you call `fetch`, Axios does it for you. It also lets you set up **interceptors** — functions that automatically run on every request or response — so you can attach auth tokens, log requests, or handle `401` errors in one place instead of in every component. The trade-off is a small bundle-size cost and one more dependency. For simple projects, `fetch` is fine; for apps with authentication, interceptors, and consistent error handling, Axios often pays for itself.

### Key Characteristics

- **Promise-based:** Returns `Promise<AxiosResponse>`; works naturally with `async`/`await`.
- **Rejects on HTTP errors:** Unlike `fetch`, Axios rejects the Promise on `4xx` and `5xx` status codes, so `catch` handles both network and HTTP failures.
- **Interceptors:** Built-in request and response interceptors for auth, logging, and error transformation.
- **Automatic JSON:** Serialises request bodies and parses JSON responses by default.
- **Structured errors:** `AxiosError` exposes `response`, `request`, `config`, `code`, and `status`.
- **Cancellation:** Supports `AbortController` (recommended) and the deprecated `CancelToken`.
- **Instances:** `axios.create({ baseURL, headers, timeout })` creates reusable, pre-configured clients.
- **XSRF protection:** Automatically reads the `XSRF-TOKEN` cookie and sets the `X-XSRF-TOKEN` header.
- **Progress and timeouts:** Built-in `onUploadProgress`, `onDownloadProgress`, and `timeout` options.
- **Browser and Node.js:** Works in both environments with the same API.

### Prerequisites

- JavaScript Promises and `async`/`await`.
- HTTP fundamentals: methods, status codes, headers, and JSON.
- React function components, `useState`, and `useEffect`.
- Basic understanding of the native `fetch` API for comparison.
- Familiarity with `AbortController` for cancellation.

### Related Programming Areas

- **HTTP Fundamentals** — methods, status codes, headers, CORS, credentials.
- **Fetch API** — the native alternative and its trade-offs.
- **Data Fetching in React** — query libraries, loaders, Server Components.
- **Authentication** — token attachment and refresh flows via interceptors.
- **Error Handling** — centralised error mapping and UI feedback.

---

## 1. Axios

### Definitions

**Core Definition:** Axios is a Promise-based HTTP client that sends requests and returns structured responses with automatic JSON handling.

**Technical Definition:** `axios(config)` or `axios.get(url, config)` returns a Promise that resolves with an `AxiosResponse` object (`{ data, status, statusText, headers, config, request }`) or rejects with an `AxiosError`. The library uses `XMLHttpRequest` in browsers and `http`/`https` in Node.js. It applies default transforms: request data objects are serialised to JSON, and response data is parsed from JSON when the content type matches. Method aliases (`get`, `post`, `put`, `patch`, `delete`, `head`, `options`) map to the corresponding HTTP verbs.

**Beginner-Friendly Explanation:** Axios is the function you call to make a request. You give it a URL and options, and it gives you back a Promise that resolves with the response data.

### Purposes

- To make HTTP requests with less boilerplate than `fetch`.
- To automatically serialise and parse JSON.
- To reject on HTTP error statuses so `catch` handles all failures.
- To provide a consistent response shape across browsers and Node.js.
- To enable interceptors for cross-cutting concerns.
- To support cancellation, timeouts, and progress events.

### Syntax Rules and Structure

**General Syntax:**

```js
const response = await axios(config);
const response = await axios.get(url, config);
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `config` | Object with `url`, `method`, `headers`, `params`, `data`, `timeout`, `signal`, etc. |
| `response.data` | The parsed response body |
| `response.status` | HTTP status code |
| `response.headers` | Response headers |
| `response.config` | The request configuration used |

**Syntax Rules:**

1. `axios(config)` accepts a single configuration object.
2. Method aliases accept `(url, config)` or `(url, data, config)` for `post`/`put`/`patch`.
3. Request `data` objects are serialised to JSON automatically.
4. Response data is parsed from JSON when the content type is JSON.
5. Axios rejects on `4xx`/`5xx` by default; customise with `validateStatus`.

**Constraints and Limitations:**

- Adds ~12–15 KB gzipped to the bundle.
- Not native; requires a dependency.
- In Next.js, Axios does not integrate with the built-in `fetch` cache or ISR; wrap calls in `unstable_cache` if caching is needed.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic GET with Axios

```js
import axios from 'axios';

async function getUser(id) {
  try {
    const response = await axios.get(`/api/users/${id}`);
    return response.data;
  } catch (error) {
    if (error.response) {
      throw new Error(`HTTP ${error.response.status}: ${error.response.data?.message}`);
    }
    throw new Error('Network error');
  }
}
```

**Expected Output:** Returns the parsed user object on success. Throws a descriptive error for HTTP failures or network failures.

**Why this output:** Axios rejects on `4xx`/`5xx`, so the `catch` block handles both HTTP and network errors. `error.response` distinguishes HTTP errors from network errors where no response was received.

#### Example 2: POST with Automatic JSON

```js
async function createPost(post) {
  const response = await axios.post('/api/posts', post);
  return response.data;
}
```

**Expected Output:** Sends `post` as JSON and returns the created resource.

**Why this output:** Axios automatically sets `Content-Type: application/json` and serialises the object when `data` is a plain object.

### Real-World Cases

- **Authenticated APIs:** Attaching tokens via interceptors.
- **Form submissions:** Sending JSON or `FormData` with automatic handling.
- **File uploads:** Using `FormData` and progress events.
- **Node.js services:** Using the same client on the server.

### References

- Axios — Official README — https://github.com/axios/axios
- Axios — Documentation — https://axios-http.com/docs/intro
- Axios in React — Guvi — https://www.guvi.in/blog/axios-in-react-build-better-web-apps/

---

## 2. Axios Instance

### Definitions

**Core Definition:** An Axios instance is a pre-configured Axios client created with `axios.create()`, with its own `baseURL`, headers, timeout, and interceptors.

**Technical Definition:** `axios.create(config)` returns a new Axios instance with the provided config merged into the defaults. The instance exposes the same methods as the global `axios` object (`get`, `post`, `interceptors`, etc.) but with its own isolated configuration and interceptor chain. Instances are the recommended pattern for applications that call multiple APIs or need different auth strategies per API.

**Beginner-Friendly Explanation:** An Axios instance is a custom copy of Axios that already knows your base URL, your headers, and your auth setup. You create it once and use it everywhere.

### Purposes

- To avoid repeating `baseURL` and headers on every request.
- To isolate configuration per API (e.g., different auth for different services).
- To attach interceptors once instead of per request.
- To centralise timeout and credential settings.
- To make testing easier by swapping the instance.

### Syntax Rules and Structure

```js
import axios from 'axios';

const apiClient = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
});

export default apiClient;
```

**Syntax Rules:**

1. Use `axios.create(config)` to create an instance.
2. Export the instance from a dedicated module (e.g., `api/client.js`).
3. Import the instance in components and API modules.
4. Interceptors added to the instance apply only to that instance.
5. Use separate instances for different APIs or auth strategies.

**Constraints and Limitations:**

- Instance interceptors do not affect the global `axios` object.
- Over-splitting into many instances can complicate maintenance.

### Annotated Code Examples

```js
// api/client.js
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 15_000,
});

export const publicClient = axios.create({
  baseURL: import.meta.env.VITE_PUBLIC_API_URL,
  timeout: 10_000,
});
```

```js
// api/users.js
import { apiClient } from './client';

export async function fetchUser(id) {
  const { data } = await apiClient.get(`/users/${id}`);
  return data;
}
```

**Expected Output:** All calls through `apiClient` use the configured `baseURL` and timeout without repeating them.

**Why this output:** `axios.create` merges the config into the instance defaults. The instance methods use those defaults on every request.

### Real-World Cases

- **Multi-API apps:** Separate instances for internal and third-party APIs.
- **Auth separation:** Authenticated vs. public clients.
- **Testing:** Injecting a mock instance in tests.

### References

- Axios — Creating an instance — https://axios-http.com/docs/instance
- CoreUI — How to protect API requests in React — https://coreui.io/answers/how-to-protect-api-requests-in-react/
- BrowserStack — Axios Request Interceptors in Next.js — https://enterprise.browserstack.com

---

## 3. Axios Configuration

### Definitions

**Core Definition:** Axios configuration is the set of options that control how a request is sent, merged from defaults, instance defaults, and per-request overrides.

**Technical Definition:** Axios merges configuration from three sources in order of increasing precedence: **global defaults** (`axios.defaults`), **instance defaults** (passed to `axios.create`), and **request config** (passed to the method call). Config keys include `url`, `method`, `baseURL`, `headers`, `params`, `data`, `timeout`, `withCredentials`, `responseType`, `validateStatus`, `signal`, `onUploadProgress`, and `onDownloadProgress`.

**Beginner-Friendly Explanation:** Configuration is how you tell Axios what to do — where to send the request, what headers to use, how long to wait, and what to do with the response.

### Purposes

- To set a base URL and default headers once.
- To control timeouts and credentials per instance or request.
- To customise which status codes are treated as errors.
- To configure response types (JSON, blob, text, stream).
- To attach progress handlers for uploads and downloads.

### Syntax Rules and Structure

**Config precedence:**

```text
Global defaults < Instance defaults < Request config
```

**Common config keys:**

| Key | Purpose |
|-----|---------|
| `baseURL` | Prefix for relative URLs |
| `headers` | Request headers |
| `params` | Query string parameters |
| `data` | Request body |
| `timeout` | Milliseconds before aborting |
| `withCredentials` | Send cookies cross-origin |
| `responseType` | `'json'`, `'text'`, `'blob'`, `'stream'` |
| `validateStatus` | Function deciding which statuses reject |
| `signal` | `AbortSignal` for cancellation |

**Syntax Rules:**

1. Set global defaults with `axios.defaults.headers.common`.
2. Set instance defaults in `axios.create({...})`.
3. Override per request in the method config.
4. Use `params` for query strings instead of building URLs manually.
5. Use `validateStatus` to customise which statuses reject.

**Constraints and Limitations:**

- `withCredentials: true` requires server-side CORS configuration.
- `timeout` uses the browser's XHR timeout; it does not cover DNS or connection phases equally in all environments.
- Global defaults affect every request, including third-party libraries.

### Annotated Code Examples

```js
const apiClient = axios.create({
  baseURL: '/api',
  timeout: 10_000,
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
  withCredentials: true,
});

// Per-request override
const response = await apiClient.get('/reports', {
  params: { page: 2, sort: 'asc' },
  timeout: 30_000,
  validateStatus: (status) => status < 500,
});
```

**Expected Output:** The instance uses its defaults; the individual request overrides `timeout` and `validateStatus` while inheriting `baseURL` and headers.

**Why this output:** Axios merges request config over instance defaults. The `validateStatus` override means `4xx` responses resolve instead of rejecting.

### Real-World Cases

- **API clients:** Base URL and auth headers per instance.
- **File downloads:** `responseType: 'blob'`.
- **Progress UI:** `onUploadProgress` for uploads.

### References

- Axios — Request Config — https://axios-http.com/docs/req_config
- Axios — Config Defaults — https://axios-http.com/docs/config_defaults

---

## 4. Axios Interceptors

### Definitions

**Core Definition:** Axios interceptors are functions that run before a request is sent or after a response is received, allowing global modification of requests and responses.

**Technical Definition:** `instance.interceptors.request.use(onFulfilled, onRejected)` and `instance.interceptors.response.use(onFulfilled, onRejected)` register interceptor functions. Request interceptors receive the config and must return it (or a modified version). Response interceptors receive the response and must return it (or a transformed response). Interceptors run in the order they are added; multiple interceptors chain. They can be removed with `eject(id)`.

**Beginner-Friendly Explanation:** Interceptors are hooks that run automatically on every request or response. You can use them to attach auth tokens, log requests, refresh tokens on `401`, or transform responses.

### Purposes

- To attach authentication tokens to every request.
- To handle `401` responses and refresh tokens centrally.
- To log requests and responses for debugging.
- To transform request or response data globally.
- To handle loading indicators and global error toasts.
- To retry failed requests automatically.

### Syntax Rules and Structure

**Request interceptor:**

```js
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  (error) => Promise.reject(error)
);
```

**Response interceptor:**

```js
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      await refreshToken();
      return apiClient.request(error.config);
    }
    return Promise.reject(error);
  }
);
```

**Syntax Rules:**

1. Request interceptors must return `config` (or a promise resolving to it).
2. Response interceptors must return `response` (or a promise resolving to it).
3. Use the second argument (`onRejected`) for error handling in interceptors.
4. Interceptors run in registration order; the last registered runs first on responses.
5. Remove interceptors with `eject(id)` when they are no longer needed.

**Constraints and Limitations:**

- Interceptors added inside React components accumulate on every render unless ejected; set them up once at the module level.
- Interceptor errors propagate to the caller; do not swallow errors silently.
- Token refresh logic must prevent infinite loops (e.g., retry only once).

### Annotated Code Examples

```js
// api/client.js
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: '/api',
});

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('authToken');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('authToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

**Expected Output:** Every request made through `apiClient` includes the bearer token if present. A `401` response clears the token and redirects to login.

**Why this output:** The request interceptor attaches the token from storage. The response interceptor handles `401` centrally, so components do not need to duplicate this logic.

### Real-World Cases

- **JWT authentication:** Attach tokens and handle refresh.
- **Logging:** Capture request/response timing and payloads.
- **Global error toasts:** Show a toast on server errors.

### References

- Axios — Interceptors — https://axios-http.com/docs/interceptors
- CoreUI — How to protect API requests in React — https://coreui.io/answers/how-to-protect-api-requests-in-react/
- BrowserStack — Axios Request Interceptors in Next.js — https://enterprise.browserstack.com

---

## 5. Axios Error Handling

### Definitions

**Core Definition:** Axios error handling is the process of catching `AxiosError` objects and distinguishing between HTTP errors (server responded with a non-2xx status) and network errors (no response received).

**Technical Definition:** Axios rejects with an `AxiosError` that exposes `message`, `name` (`'AxiosError'`), `stack`, `config`, `code`, `status`, and — depending on the failure — `response` (present when the server responded with a non-2xx status) or `request` (present when the request was sent but no response was received). `validateStatus` can customise which statuses reject. `error.toJSON()` returns a serialisable error summary. Cancellation errors are `CanceledError` (or `Cancel` in legacy code).

**Beginner-Friendly Explanation:** Axios errors tell you what went wrong. If `error.response` exists, the server said no — you have a status code and a body. If `error.request` exists but no response, the network failed. If neither exists, something went wrong before the request was sent.

### Purposes

- To distinguish HTTP errors from network errors.
- To map status codes to user-facing messages.
- To handle validation errors (`422`) and auth errors (`401`/`403`).
- To log structured error information.
- To implement retry logic for transient failures.

### Syntax Rules and Structure

**Error structure:**

| Property | Present when | Meaning |
|----------|--------------|---------|
| `error.response` | Server responded with non-2xx | HTTP error with `data`, `status`, `headers` |
| `error.request` | Request sent, no response | Network error (timeout, offline, CORS) |
| `error.config` | Always | The request configuration |
| `error.code` | Often | Axios error code (e.g., `ECONNABORTED`) |
| `error.status` | Sometimes | HTTP status (convenience) |

**Syntax Rules:**

1. Check `error.response` first; if present, it is an HTTP error.
2. Otherwise, check `error.request`; if present, it is a network error.
3. Otherwise, the error occurred during request setup.
4. Use `error.response.status` to branch on specific status codes.
5. Use `validateStatus` to treat selected statuses as non-errors.

**Constraints and Limitations:**

- CORS failures and network failures both surface as request errors with little detail.
- `error.response` is absent for timeouts and aborts.
- Do not expose raw `error.response.data` to users without sanitising.

### Annotated Code Examples

```js
async function fetchUser(id) {
  try {
    const { data } = await axios.get(`/api/users/${id}`);
    return data;
  } catch (error) {
    if (error.response) {
      // Server responded with a non-2xx status
      switch (error.response.status) {
        case 401:
          throw new Error('Please log in again');
        case 403:
          throw new Error('You do not have permission');
        case 404:
          throw new Error('User not found');
        default:
          throw new Error(error.response.data?.message ?? 'Request failed');
      }
    } else if (error.request) {
      // Request sent, no response
      throw new Error('Network error: unable to reach the server');
    } else {
      // Setup error
      throw new Error('Request could not be sent');
    }
  }
}
```

**Expected Output:** A specific error message for `401`, `403`, `404`, other HTTP errors, network failures, and setup failures.

**Why this output:** The three-tier check (`response` → `request` → neither) covers every Axios failure mode and produces a user-appropriate message for each.

### Real-World Cases

- **Auth flows:** `401` triggers login; `403` shows forbidden.
- **Form validation:** `422` maps errors to fields.
- **Offline handling:** `error.request` signals connectivity issues.
- **Retry logic:** Retry on network errors and `5xx`, not `4xx`.

### References

- Axios — Handling Errors — https://axios-http.com/docs/handling_errors
- Axios Error Handling Reference — https://github.com/nick2bad4u/uptime-watcher
- Decodo — cURL in JavaScript: Fetch, Axios — https://decodo.com

---

## 6. Axios Cancellation

### Definitions

**Core Definition:** Axios cancellation is the ability to abort an in-flight request so it no longer consumes resources or updates state.

**Technical Definition:** Since v0.22.0, Axios supports the `AbortController` API: create a controller, pass its `signal` to the request config, and call `controller.abort()` to cancel. Aborted requests reject with a `CanceledError` (identifiable via `axios.isCancel(error)`). The legacy `CancelToken` API is deprecated and will be removed; `AbortController` is the recommended approach. One signal can cancel multiple requests.

**Beginner-Friendly Explanation:** Cancellation lets you say “stop this request, I no longer need it.” You create an `AbortController`, give its signal to Axios, and call `.abort()` when you want to cancel.

### Purposes

- To cancel requests when a component unmounts.
- To cancel stale requests when parameters change.
- To cancel multiple requests with one controller.
- To avoid memory leaks and race conditions.
- To support “cancel” buttons in the UI.

### Syntax Rules and Structure

```js
const controller = new AbortController();

axios.get('/api/data', { signal: controller.signal })
  .catch((error) => {
    if (axios.isCancel(error)) return; // expected cancellation
    throw error;
  });

controller.abort();
```

**Syntax Rules:**

1. Create an `AbortController` per logical request or group.
2. Pass `signal` in the request config.
3. Call `.abort()` in cleanup functions or event handlers.
4. Use `axios.isCancel(error)` to detect cancellations.
5. `CancelToken` is deprecated; use `AbortController`.

**Constraints and Limitations:**

- Cancellation stops the client from waiting; it does not guarantee the server stops processing.
- Cancelled requests reject; handle them explicitly to avoid spurious errors.
- `CancelToken` will be removed in a future major release.

### Annotated Code Examples

```jsx
useEffect(() => {
  const controller = new AbortController();

  (async () => {
    try {
      const { data } = await axios.get(`/api/search?q=${query}`, {
        signal: controller.signal,
      });
      setResults(data);
    } catch (error) {
      if (!axios.isCancel(error)) setError(error);
    }
  })();

  return () => controller.abort();
}, [query]);
```

**Expected Output:** When `query` changes, the previous request is aborted and only the latest result updates state. Unmounting aborts the final request.

**Why this output:** The cleanup calls `controller.abort()`. `axios.isCancel` prevents cancelled requests from being treated as errors.

### Real-World Cases

- **Search inputs:** Cancelling outdated queries on each keystroke.
- **Route changes:** Cancelling in-flight requests on navigation.
- **Modal close:** Cancelling data fetches when the modal closes.

### References

- Axios — Cancellation — https://axios-http.com/docs/cancellation
- GitHub — Axios Cancellation (v0.22.0+) — https://github.com/axios/axios
- Axios AbortController note — https://axios-http.com/docs/cancellation

---

## 7. Axios vs `fetch`

### Definitions

**Core Definition:** Axios and `fetch` are both HTTP clients, but they differ in error semantics, built-in features, and bundle cost.

**Technical Definition:** `fetch` is a native browser API defined by the WHATWG Fetch Standard. It resolves on HTTP error statuses and rejects only on network failures, requires manual status checking and JSON parsing, and has no built-in interceptors, timeout, or progress events. Axios is a third-party library that rejects on `4xx`/`5xx`, auto-parses JSON, provides interceptors, timeout, cancellation (via `AbortController`), progress events, and XSRF protection. Axios adds ~12–15 KB gzipped to the bundle; `fetch` adds nothing. In Next.js, `fetch` integrates with the built-in cache and ISR, while Axios does not (wrap it in `unstable_cache` if needed).

**Beginner-Friendly Explanation:** `fetch` is built into the browser — free, native, and simple. Axios is a library you install — it adds convenience features like interceptors and automatic error rejection, but it costs bundle size. Use `fetch` for simple projects and Axios for complex apps with auth, interceptors, and centralised error handling.

### Purposes

- To choose the right client for the project's complexity.
- To understand the trade-offs in bundle size and features.
- To migrate between the two with correct expectations.
- To integrate with framework caching (Next.js) correctly.

### Comparison Table

| Feature | `fetch` | Axios |
|---------|---------|-------|
| **Built-in** | Yes (native) | No (dependency) |
| **Bundle size** | 0 KB | ~12–15 KB gzipped |
| **Rejects on 4xx/5xx** | No (resolves) | Yes |
| **JSON parsing** | Manual (`.json()`) | Automatic |
| **Interceptors** | No | Yes |
| **Timeout** | Manual (`AbortSignal.timeout`) | Built-in `timeout` |
| **Cancellation** | `AbortController` | `AbortController` (recommended) |
| **Progress events** | Manual streams | `onUploadProgress` / `onDownloadProgress` |
| **XSRF protection** | Manual | Built-in |
| **Next.js cache/ISR** | Integrated | Not integrated (wrap in `unstable_cache`) |
| **Node.js support** | Native (18+) | Yes |
| **Best for** | Simple requests, bundle-sensitive apps | Complex apps with auth and interceptors |

**Syntax Rules:**

1. Use `fetch` when bundle size is critical and requests are simple.
2. Use Axios when you need interceptors, centralised auth, or automatic error rejection.
3. In Next.js, prefer `fetch` for cache/ISR integration; wrap Axios if needed.
4. Both support `AbortController`; cancellation code is nearly identical.
5. Performance difference for a single request is negligible; network latency dominates.

**Constraints and Limitations:**

- Axios adds bundle weight that matters for performance-critical landing pages.
- `fetch` requires more boilerplate for error handling and JSON parsing.
- Neither is universally “better”; the choice depends on the app's needs.

### Annotated Code Examples

**`fetch` (manual status and JSON):**

```js
const res = await fetch('/api/users/1');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const user = await res.json();
```

**Axios (automatic):**

```js
const { data: user } = await axios.get('/api/users/1');
```

**Expected Output:** Both return the user. Axios throws automatically on `404`; `fetch` requires the `res.ok` check.

**Why this output:** Axios rejects on non-2xx statuses and parses JSON by default. `fetch` resolves and requires manual handling.

### Real-World Cases

- **Bundle-sensitive marketing pages:** Prefer `fetch`.
- **Enterprise apps with JWT refresh:** Prefer Axios interceptors.
- **Next.js apps using ISR:** Prefer `fetch` or wrap Axios in `unstable_cache`.

### References

- Guvi — Axios vs Fetch — https://www.guvi.in/blog/axios-in-react-build-better-web-apps/
- Geonode — Axios vs Fetch 2026 — https://geonode.com
- Vercel — Axios vs Next.js fetch caching — https://github.com/vercel/next.js/discussions/89260
- Decodo — Fetch vs Axios error handling — https://decodo.com

---

## 8. Axios Best Practices

### Definitions

**Core Definition:** Axios best practices are the guidelines for structuring instances, interceptors, errors, and cancellation to keep HTTP logic centralised, testable, and maintainable.

**Technical Definition:** Best practices combine **instance-based architecture** (one configured client per API), **module-level interceptor setup** (never inside components), **centralised error taxonomy** (map `AxiosError` to typed application errors), **cancellation discipline** (`AbortController` in cleanup), **timeout configuration**, **credential and CORS correctness**, **avoiding interceptor duplication**, and **testing with mocked instances**. For Next.js, account for Axios's lack of native cache integration.

**Beginner-Friendly Explanation:** Good Axios usage means creating one configured client, setting up interceptors once, handling errors in one place, and always cancelling requests when they are no longer needed.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Create a dedicated instance** | `axios.create({ baseURL, timeout, headers })` per API. |
| **Set up interceptors at module level** | Never inside components; eject if dynamic. |
| **Centralise error handling** | Map `AxiosError` to typed errors in one module. |
| **Always cancel on unmount** | `AbortController` in `useEffect` cleanup. |
| **Set timeouts** | Per instance or per request; avoid indefinite hangs. |
| **Use `validateStatus` deliberately** | Do not silently treat `4xx` as success unless intended. |
| **Avoid duplicate interceptors** | Register once; eject on cleanup if needed. |
| **Handle token refresh carefully** | Retry once; avoid infinite loops. |
| **Do not attach tokens in components** | Use a request interceptor. |
| **Test with mocked instances** | Inject a mock client in tests. |
| **Account for Next.js caching** | Wrap Axios in `unstable_cache` for ISR. |
| **Prefer `AbortController` over `CancelToken`** | `CancelToken` is deprecated. |

### Annotated Code Examples

```js
// api/client.js — single configured instance
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 15_000,
  headers: { 'Content-Type': 'application/json' },
});

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

```jsx
// Component — cancellation in cleanup
useEffect(() => {
  const controller = new AbortController();

  apiClient.get('/dashboard', { signal: controller.signal })
    .then(({ data }) => setData(data))
    .catch((error) => {
      if (!axios.isCancel(error)) setError(error);
    });

  return () => controller.abort();
}, []);
```

**Expected Output:** All requests share the configured instance and auth interceptor. Components cancel on unmount and ignore cancellation errors.

**Why this output:** The instance centralises configuration and interceptors. The component only handles its own state and cancellation, not auth or global error logic.

### Real-World Cases

- **All Axios-based applications:** These practices apply universally.
- **JWT-auth SPAs:** Centralised token attachment and refresh.
- **Next.js apps:** Wrapping Axios for cache integration.
- **Testing:** Mocking the configured instance.

### References

- CoreUI — How to protect API requests in React — https://coreui.io/answers/how-to-protect-api-requests-in-react/
- BrowserStack — Axios Request Interceptors in Next.js — https://enterprise.browserstack.com
- CodeTreatise — API Integration Chapter — https://github.com/CodeTreatise/Build-Deploy-The-React-App
- Vercel — Axios vs Next.js fetch caching — https://github.com/vercel/next.js/discussions/89260
- Axios — Interceptors — https://axios-http.com/docs/interceptors

---

## References

- Axios — Official README — https://github.com/axios/axios
- Axios — Documentation — https://axios-http.com/docs/intro
- Axios — Creating an instance — https://axios-http.com/docs/instance
- Axios — Request Config — https://axios-http.com/docs/req_config
- Axios — Config Defaults — https://axios-http.com/docs/config_defaults
- Axios — Interceptors — https://axios-http.com/docs/interceptors
- Axios — Handling Errors — https://axios-http.com/docs/handling_errors
- Axios — Cancellation — https://axios-http.com/docs/cancellation
- Axios in React — Guvi — https://www.guvi.in/blog/axios-in-react-build-better-web-apps/
- CoreUI — How to protect API requests in React — https://coreui.io/answers/how-to-protect-api-requests-in-react/
- BrowserStack — Axios Request Interceptors in Next.js — https://enterprise.browserstack.com
- CodeTreatise — API Integration Chapter — https://github.com/CodeTreatise/Build-Deploy-The-React-App
- Vercel — Axios vs Next.js fetch caching — https://github.com/vercel/next.js/discussions/89260
- Geonode — Axios vs Fetch 2026 — https://geonode.com
- Decodo — cURL in JavaScript: Fetch, Axios — https://decodo.com
- Axios Error Handling Reference — https://github.com/nick2bad4u/uptime-watcher
- Axios Cancellation Reference — https://github.com/axios/axios
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController