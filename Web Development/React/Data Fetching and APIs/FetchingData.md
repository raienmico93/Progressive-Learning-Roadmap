# React Fetching Data & Component Lifecycle: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React data fetching is the practice of retrieving remote data, managing the network lifecycle (loading, error, empty, success), and synchronising that data with the component render cycle—using either legacy `useEffect` patterns or modern React 19 primitives like the `use` hook and Suspense.

**Technical Definition:** React data fetching encompasses the transport layer (native `fetch` vs. Axios), the orchestration layer (`async`/`await` and Promise chaining), the state layer (request, loading, error, empty, and success states), the lifecycle layer (`useEffect` with cleanup and race-condition mitigation), and the modern streaming layer (React 19's `use` hook and Suspense integration). The `fetch` API is a native browser interface that returns a `Promise<Response>` and rejects only on network failure, not HTTP error statuses. Axios is a library built on XMLHttpRequest (browser) and the Node `http` module (server) that provides automatic JSON transformation, request/response interceptors, timeout configuration, upload/download progress, and consistent error objects. React 19 introduced the `use` hook, which reads a promise (or context) and suspends the component until the promise settles, enabling declarative loading and error handling via `<Suspense>` and Error Boundaries. The `use` hook can be called conditionally—unlike all other hooks—because it does not register state.

**Beginner-Friendly Explanation:** When your React app needs data from a server, you have to do three things: send the request, wait for it to come back, and figure out what to show while you're waiting. The browser gives you `fetch` for free; Axios is a fancier tool with more features. Promises and `async`/`await` are the JavaScript way of saying "wait for this." Components have four possible states during a fetch: loading, error, empty (the request succeeded but returned nothing), and success. Older React code uses `useEffect` to start fetches and needs careful cleanup to avoid bugs. React 19 offers a cleaner way: the `use` hook, which lets components "pause" until data is ready, with Suspense showing a fallback.

### Key Characteristics

- **fetch is Native, Axios is a Library:** `fetch` requires no dependency but lacks interceptors, progress, and automatic JSON; Axios provides these at the cost of ~13 KB.
- **fetch Rejects Only on Network Failure:** HTTP errors (404, 500) resolve the promise with `res.ok === false`. Axios rejects on HTTP errors by default.
- **Promise Orchestration Matters:** Sequential `await` chains are slower than `Promise.all`; mixing `await` with `.then()` is an anti-pattern.
- **Four Network States:** Loading, Error, Empty, and Success—each requires distinct UI handling. "Empty" is often forgotten.
- **Legacy Patterns Require Cleanup:** `useEffect` fetches need `AbortController` or an `ignore` flag to prevent stale results and state updates on unmounted components.
- **React 19 `use` is Conditional-Friendly:** Unlike other hooks, `use` can be called inside conditionals and loops because it does not allocate state.
- **Suspense is the Fallback Mechanism:** With `use`, loading and error states are handled declaratively by `<Suspense>` and Error Boundaries rather than by `isLoading`/`isError` flags.

### Prerequisites

- Solid understanding of JavaScript promises, `async`/`await`, and the event loop.
- Familiarity with React function components and the `useState`/`useEffect` Hooks.
- Working knowledge of the Fetch API and HTTP status codes.
- Basic understanding of React Suspense and Error Boundaries.
- Awareness of server-state libraries (TanStack Query) as an alternative to manual fetching.

### Related Programming Areas

- **Server-State Management:** Caching, invalidation, and synchronisation (TanStack Query, SWR).
- **HTTP Fundamentals:** Methods, status codes, headers, CORS.
- **Concurrency Control:** Race conditions, cancellation, and request sequencing.
- **Error Handling:** Error Boundaries, fallback UIs, retry logic.
- **Performance:** Streaming, code splitting, and progressive rendering.

### Core Concepts / Features

1. Native `fetch` API vs. Axios
2. Orchestrating Asynchronous Operations (`async`/`await` and Promise Chaining)
3. Component Network States (Request, Error, Loading, Empty)
4. Legacy Data Fetching Patterns (`useEffect`, Cleanup, Race Conditions)
5. Modern React 19 Data Streaming (the `use` Hook and Suspense)

---

## Core Concept 1: Native `fetch` API vs. Axios

### Definitions

**Core Definition:** `fetch` is the browser's built-in HTTP client that returns promises, while Axios is a third-party library that wraps XMLHttpRequest (browser) or the Node `http` module (server) with additional features like interceptors, automatic JSON, and progress tracking.

**Technical Definition:** The `fetch()` function is part of the Fetch Living Standard and is available globally in browsers and Node 18+. It returns a `Promise<Response>`, where `Response` is a stream-based object with methods `json()`, `text()`, `blob()`, `arrayBuffer()`, and `formData()`. Crucially, `fetch` resolves on any HTTP response (including 4xx and 5xx) and rejects only on network failure, timeout (if configured via `AbortSignal.timeout()`), or CORS blockage. Axios is a promise-based HTTP client that uses `XMLHttpRequest` in the browser and the Node `http` module on the server. It automatically transforms request and response data to/from JSON, rejects on HTTP error statuses by default, supports request and response interceptors, provides `onUploadProgress`/`onDownloadProgress`, supports timeouts natively, and offers `axios.all()` for parallel requests. Both support `AbortController` for cancellation, but Axios uses the same `signal` option and also provides `CancelToken` (deprecated).

**Beginner-Friendly Explanation:** `fetch` is like a basic tool that comes free with your browser—it works, but you have to do a lot of the setup yourself. Axios is like a Swiss Army knife—it costs a small bundle size but gives you interceptors (hooks into every request), automatic JSON parsing, progress bars, and timeouts out of the box. For simple projects, `fetch` is enough. For complex ones, Axios saves you a lot of boilerplate.

### Purposes

- **`fetch`:** To make HTTP requests with zero dependencies and native browser support.
- **`fetch`:** To use stream-based response processing (`res.body.getReader()`) for large payloads.
- **`fetch`:** To leverage `Request`, `Response`, and `Headers` objects that align with service workers and cache APIs.
- **Axios:** To automatically serialise and deserialise JSON without manual `JSON.stringify`/`res.json()`.
- **Axios:** To use interceptors for adding auth tokens, logging, or refresh logic globally.
- **Axios:** To track upload and download progress with `onUploadProgress`/`onDownloadProgress`.
- **Axios:** To set a default timeout and base URL across all requests.
- **Axios:** To get consistent error objects (`error.response`, `error.request`, `error.message`).

### Syntax Rules and Structure

**Native `fetch` — GET and POST:**
```javascript
// GET
const res = await fetch('/api/users');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const users = await res.json();

// POST with JSON
const res = await fetch('/api/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Alice' }),
});
const created = await res.json();
```

**Component Breakdown:**
- `fetch(url)`: Returns a `Promise<Response>`.
- `res.ok`: `true` for 2xx; must be checked manually.
- `res.json()`: Parses the body as JSON; returns a `Promise`.
- `Content-Type`: Must be set manually for JSON bodies.

**Axios — GET and POST:**
```javascript
import axios from 'axios';

// GET — automatic JSON parsing
const { data: users } = await axios.get('/api/users');

// POST — automatic JSON serialisation
const { data: created } = await axios.post('/api/users', { name: 'Alice' });
```

**Component Breakdown:**
- `axios.get(url)`: Returns a `Promise<AxiosResponse>`.
- `response.data`: The parsed body (automatically JSON if the content type matches).
- `axios.post(url, body)`: Serialises the body and sets `Content-Type: application/json` automatically.

**Axios Configuration Defaults:**
```javascript
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10_000,
  headers: { 'Accept': 'application/json' },
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

**Component Breakdown:**
- `axios.create({ baseURL, timeout, headers })`: Creates a configured instance.
- `interceptors.request.use`: Runs before every request; adds auth tokens.
- `interceptors.response.use`: Runs after every response; handles 401 globally.

**Comparison Table:**

| Feature | `fetch` | Axios |
|---|---|---|
| **Native support** | ✅ Browser, Node 18+ | ❌ Requires install (~13 KB) |
| **HTTP errors reject** | ❌ (resolves with `ok: false`) | ✅ (rejects by default) |
| **JSON serialisation** | Manual (`JSON.stringify`) | Automatic |
| **JSON deserialisation** | Manual (`res.json()`) | Automatic (`res.data`) |
| **Interceptors** | ❌ | ✅ Request + response |
| **Timeout** | Via `AbortSignal.timeout()` | Native `timeout` option |
| **Upload progress** | ❌ | ✅ `onUploadProgress` |
| **Download progress** | ❌ (use streams) | ✅ `onDownloadProgress` |
| **Request cancellation** | `AbortController` | `AbortController` |
| **Base URL / defaults** | Manual | `axios.create` |
| **Node.js support** | ✅ (Node 18+) | ✅ (all versions) |
| **Bundle size** | 0 KB | ~13 KB |

**Syntax Rules:**
- Always check `res.ok` after `fetch`; it does not reject on HTTP errors.
- Always set `Content-Type: application/json` when sending JSON with `fetch`.
- Use `axios.create` for configured instances with `baseURL`, `timeout`, and headers.
- Use interceptors for cross-cutting concerns (auth, logging, error handling).
- Use `AbortController` for cancellation with both `fetch` and Axios.
- In Node 18+, `fetch` is global; in older versions, use `node-fetch` or Axios.
- Choose `fetch` for simple projects and zero dependencies; choose Axios for complex projects with interceptors and progress tracking.

**Constraints and Limitations:**
- `fetch` does not support upload progress; use `XMLHttpRequest` or Axios if you need it.
- `fetch` in older browsers (IE11) requires a polyfill; Axios works via XHR.
- Axios adds a bundle size cost and a dependency to maintain.
- Axios's `CancelToken` is deprecated; use `AbortController` instead.
- Neither library handles retries, caching, or deduplication automatically; use TanStack Query for those.

### Annotated Code Example: Same Request with Both Libraries

```javascript
// --- fetch version ---
async function fetchUsersWithFetch() {
  const res = await fetch('/api/users', {
    headers: { 'Accept': 'application/json' },
  });

  if (!res.ok) {
    throw new Error(`HTTP ${res.status}: ${res.statusText}`);
  }

  return res.json();
}

// --- Axios version ---
import axios from 'axios';

async function fetchUsersWithAxios() {
  try {
    const response = await axios.get('/api/users', {
      headers: { 'Accept': 'application/json' },
    });
    return response.data;
  } catch (error) {
    if (error.response) {
      // Server responded with 4xx/5xx
      throw new Error(`HTTP ${error.response.status}: ${error.response.statusText}`);
    } else if (error.request) {
      // No response received
      throw new Error('No response from server');
    } else {
      throw new Error(error.message);
    }
  }
}
```

**Expected Output:** Both functions return the parsed user list on success. On HTTP error, `fetchUsersWithFetch` throws because we explicitly checked `res.ok`; `fetchUsersWithAxios` catches the Axios rejection and rethrows a normalised error.

**Why This Output Occurs:** With `fetch`, HTTP errors resolve the promise, so the explicit `res.ok` check is mandatory. With Axios, HTTP errors reject the promise, so a `try/catch` is required. The error handling differs: `fetch` errors only carry the status from the `Response`; Axios errors carry `error.response` (with status and data), `error.request`, and `error.message`.

### Real-World Cases

- **Simple SPA:** `fetch` is sufficient for a small app with a few endpoints.
- **Enterprise app:** Axios with interceptors for auth, logging, and global error handling.
- **File uploads with progress:** Axios's `onUploadProgress` is the standard choice.
- **Streaming responses:** `fetch` with `res.body.getReader()` for SSE-like streaming.
- **React Native:** Both work; `fetch` is built in; Axios adds interceptors.
- **Node scripts:** Both work; Axios handles older Node versions.

### References

- MDN Web Docs – Fetch API: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN Web Docs – Using Fetch: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- Axios – Documentation: https://axios-http.com/docs/intro
- Axios – Interceptors: https://axios-http.com/docs/interceptors
- Axios – Request Config: https://axios-http.com/docs/req_config

---

## Core Concept 2: Orchestrating Asynchronous Operations (`async`/`await` and Promise Chaining)

### Definitions

**Core Definition:** `async`/`await` is syntactic sugar over Promises that lets asynchronous code read like synchronous code, while Promise chaining uses `.then()`/`.catch()`/`.finally()` to compose asynchronous operations without `await`.

**Technical Definition:** An `async` function always returns a `Promise`. Inside it, `await` pauses execution until the awaited promise settles, then resumes with the resolved value. Rejections are thrown as exceptions and can be caught with `try/catch`. Promise chaining uses `.then(onFulfilled, onRejected)` to register callbacks, `.catch(onRejected)` for error handling, and `.finally(onFinally)` for cleanup. The key difference: `await` sequences operations (each waits for the previous), while `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()` run operations in parallel. `Promise.all` rejects if any promise rejects; `Promise.allSettled` waits for all and returns their outcomes; `Promise.race` settles with the first settled promise; `Promise.any` settles with the first fulfilled promise. Mixing `await` with `.then()` is an anti-pattern that makes code harder to read and error-prone.

**Beginner-Friendly Explanation:** A Promise is like an IOU—it's a placeholder for a value that will arrive later. `async`/`await` lets you write code that "pauses" until the IOU is paid, without blocking the whole program. Promise chaining is the older way of saying "when this is done, do that, then do that." For operations that don't depend on each other, run them in parallel with `Promise.all`—don't make them wait in line.

### Purposes

- **Sequential `await`:** To run dependent async operations in order (each needs the previous result).
- **`Promise.all`:** To run independent async operations in parallel and wait for all to complete.
- **`Promise.allSettled`:** To run parallel operations and collect all results, regardless of individual failures.
- **`Promise.race`:** To settle with the first completed operation (e.g., timeout patterns).
- **`Promise.any`:** To settle with the first successful operation (e.g., redundant endpoints).
- **`try/catch`:** To handle errors from `await`ed operations.
- **`.finally()`:** To run cleanup regardless of success or failure.

### Syntax Rules and Structure

**Sequential `await` (Dependent Operations):**
```javascript
async function loadUserAndPosts(userId) {
  const user = await fetchUser(userId);       // Wait for user
  const posts = await fetchPosts(user.id);    // Then fetch posts using user.id
  return { user, posts };
}
```

**Component Breakdown:**
- `await fetchUser(userId)`: Pauses until the user is fetched.
- `await fetchPosts(user.id)`: Uses the user's ID to fetch posts.
- Total time = `t(user) + t(posts)` (sequential).

**Parallel with `Promise.all` (Independent Operations):**
```javascript
async function loadDashboard(userId) {
  const [user, posts, notifications] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchNotifications(userId),
  ]);
  return { user, posts, notifications };
}
```

**Component Breakdown:**
- `Promise.all([...])`: Starts all three requests simultaneously.
- Total time = `max(t(user), t(posts), t(notifications))` (parallel).

**`Promise.allSettled` (Collect All Outcomes):**
```javascript
const results = await Promise.allSettled([
  fetchUser(id),
  fetchPosts(id),
  fetchNotifications(id),
]);

results.forEach((result) => {
  if (result.status === 'fulfilled') {
    console.log('Success:', result.value);
  } else {
    console.error('Failed:', result.reason);
  }
});
```

**Component Breakdown:**
- `Promise.allSettled`: Never rejects; always resolves with an array of `{ status, value | reason }`.
- Useful when partial success is acceptable.

**`Promise.race` (Timeout Pattern):**
```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Request timed out')), ms)
  );
  return Promise.race([promise, timeout]);
}

const data = await withTimeout(fetch('/api/data'), 5000);
```

**Component Breakdown:**
- `Promise.race`: Settles with the first settled promise (fulfilled or rejected).
- The timeout promise rejects after `ms`, racing the fetch.

**Mixing `await` and `.then()` (Anti-Pattern):**
```javascript
// ❌ Anti-pattern: mixing await and .then
async function bad() {
  const user = await fetchUser(1);
  return fetchPosts(user.id).then(posts => posts.filter(p => p.published));
}

// ✅ Correct: consistent await
async function good() {
  const user = await fetchUser(1);
  const posts = await fetchPosts(user.id);
  return posts.filter(p => p.published);
}
```

**Syntax Rules:**
- Use `await` for dependent operations; use `Promise.all` for independent ones.
- Always wrap `await` in `try/catch` when errors need to be handled.
- Use `Promise.allSettled` when partial success is acceptable.
- Use `Promise.race` for timeout patterns or "first response wins."
- Do not mix `await` and `.then()` in the same operation chain.
- Do not `await` inside a loop unless the iterations are genuinely dependent; use `Promise.all` with `.map()` instead.
- Use `.finally()` for cleanup that must run regardless of outcome.

**Constraints and Limitations:**
- `await` inside a loop serialises operations; use `Promise.all(array.map(...))` for parallel execution.
- `Promise.all` rejects on the first rejection; if you need all results, use `Promise.allSettled`.
- `async` functions always return promises; you cannot synchronously read their result.
- Unhandled promise rejections cause warnings or crashes; always attach `.catch()` or use `try/catch`.
- `await` at the top level is supported in ES modules and Node 14.8+ but not in CommonJS.

### Annotated Code Example: Sequential vs. Parallel Fetching

```javascript
// ❌ Sequential — slower (total = 3 × request time)
async function loadSequential(userId) {
  const user = await fetchUser(userId);           // ~300ms
  const posts = await fetchPosts(userId);         // ~300ms
  const comments = await fetchComments(userId);   // ~300ms
  return { user, posts, comments };
  // Total: ~900ms
}

// ✅ Parallel — faster (total = max request time)
async function loadParallel(userId) {
  const [user, posts, comments] = await Promise.all([
    fetchUser(userId),      // ~300ms
    fetchPosts(userId),     // ~300ms
    fetchComments(userId),  // ~300ms
  ]);
  return { user, posts, comments };
  // Total: ~300ms
}

// ✅ Mixed — parallel where possible, sequential where necessary
async function loadMixed(userId) {
  // Step 1: fetch user first (needed for dependent calls)
  const user = await fetchUser(userId);

  // Step 2: fetch posts and comments in parallel (both depend on user.id)
  const [posts, comments] = await Promise.all([
    fetchPosts(user.id),
    fetchComments(user.id),
  ]);

  return { user, posts, comments };
  // Total: ~t(user) + max(t(posts), t(comments))
}
```

**Expected Output:** `loadSequential` takes approximately 900ms; `loadParallel` takes approximately 300ms; `loadMixed` takes approximately 600ms (one sequential step + one parallel step).

**Why This Output Occurs:** `await` pauses the function until each promise resolves, so sequential awaits add their latencies. `Promise.all` starts all promises simultaneously, so total latency is the maximum of the individual latencies. `loadMixed` demonstrates that dependent calls must be sequential, but independent calls can be parallelised after the dependency resolves.

### Real-World Cases

- **Dashboard loading:** Parallel fetch of user, metrics, notifications with `Promise.all`.
- **Checkout flow:** Sequential fetch of cart → shipping options → payment methods.
- **Search with fallback APIs:** `Promise.any` to fetch from a primary and backup API.
- **Partial failures:** `Promise.allSettled` for a dashboard where one failing widget doesn't break the rest.
- **Timeout enforcement:** `Promise.race` with `AbortSignal.timeout()` for strict time limits.
- **Bulk operations:** `Promise.all` for creating multiple records in parallel.

### References

- MDN Web Docs – async function: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN Web Docs – await: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- MDN Web Docs – Promise.all(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN Web Docs – Promise.allSettled(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN Web Docs – Promise.race(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
- MDN Web Docs – Promise.any(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any
- MDN Web Docs – Using promises: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises

---

## Core Concept 3: Component Network States (Request, Error, Loading, Empty)

### Definitions

**Core Definition:** Component network states are the distinct UI conditions a component can be in during and after a data fetch: request (fetch initiated), loading (fetch in flight), error (fetch failed), empty (fetch succeeded but returned no data), and success (fetch succeeded with data).

**Technical Definition:** A component that fetches data transitions through a state machine: `idle` → `loading` → (`success` | `error` | `empty`). The `idle` state precedes the request (before the Effect or query fires). `loading` covers the period from request initiation to response. `error` covers network failures, HTTP errors, and parsing failures. `empty` is a sub-state of `success` where the response is valid but contains no records (e.g., `[]`, `null`, or `{ data: [] }`). Libraries like TanStack Query model these states with `status` (`'pending' | 'error' | 'success'`) and derived booleans (`isPending`, `isError`, `isSuccess`), plus `fetchStatus` (`'fetching' | 'paused' | 'idle'`) and `isRefetching`. Manual implementations with `useEffect` and `useState` require explicit state management for each condition.

**Beginner-Friendly Explanation:** Imagine you order a pizza. **Loading** is the time between placing the order and the delivery arriving. **Success** is the pizza arriving. **Error** is the delivery driver calling to say they got lost. **Empty** is the pizza arriving but with no toppings—technically successful, but not what you hoped for. Each state needs a different response from your app: a spinner for loading, a message for error, a "no items found" for empty, and the actual content for success.

### Purposes

- **Request state:** To show a "fetching..." indicator before the response arrives.
- **Loading state:** To display spinners, skeletons, or placeholders during the fetch.
- **Error state:** To display error messages, retry buttons, or fallback UIs when the fetch fails.
- **Empty state:** To display a helpful message when the fetch succeeds but returns no data.
- **Success state:** To render the fetched data.
- **Refetching state:** To show a subtle "updating..." indicator during background refetches without blanking the screen.

### Syntax Rules and Structure

**Manual State Management (useEffect + useState):**
```jsx
function UserList() {
  const [status, setStatus] = useState('idle'); // idle | loading | success | error
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    let ignore = false;
    setStatus('loading');
    setError(null);

    async function load() {
      try {
        const res = await fetch('/api/users');
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const json = await res.json();
        if (!ignore) {
          setData(json);
          setStatus('success');
        }
      } catch (err) {
        if (!ignore) {
          setError(err.message);
          setStatus('error');
        }
      }
    }

    load();
    return () => { ignore = true; };
  }, []);

  if (status === 'loading') return <Skeleton />;
  if (status === 'error') return <ErrorState message={error} onRetry={() => setStatus('idle')} />;
  if (status === 'success' && data.length === 0) return <EmptyState />;
  if (status === 'success') return <List items={data} />;

  return null;
}
```

**Component Breakdown:**
- `status`: A single string tracking the current state machine position.
- `data` and `error`: Separate state for the payload and error message.
- `if (status === 'loading')`: Renders the loading skeleton.
- `if (status === 'error')`: Renders the error state with a retry.
- `if (data.length === 0)`: Renders the empty state (must be checked *after* loading and error).
- `if (status === 'success')`: Renders the data.

**TanStack Query States:**
```jsx
import { useQuery } from '@tanstack/react-query';

function UserList() {
  const {
    data,           // The resolved data (undefined until success)
    isPending,      // No data yet — first load
    isError,        // Error encountered
    error,          // The error object
    isFetching,     // Fetching in background (including refetches)
    isSuccess,      // Data is available
  } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  if (isPending) return <Skeleton />;
  if (isError) return <ErrorState message={error.message} />;
  if (data.length === 0) return <EmptyState />;
  return (
    <>
      {isFetching && <span>Updating…</span>}
      <List items={data} />
    </>
  );
}
```

**Component Breakdown:**
- `isPending`: No data yet — first load.
- `isError`: Error encountered; `error` holds the error object.
- `isFetching`: Fetching in background — shows "Updating…" without blanking.
- `data.length === 0`: Empty state.

**State Priority Table:**

| State | Condition | UI |
|---|---|---|
| **Idle** | Before request | Nothing / initial placeholder |
| **Loading** | Request in flight, no data | Spinner / skeleton |
| **Error** | Request failed | Error message / retry |
| **Empty** | Request succeeded, no data | "No items found" message |
| **Success** | Request succeeded, data exists | Data list |
| **Refetching** | Background fetch with existing data | Subtle "updating" indicator |

**Syntax Rules:**
- Check loading **first**, then error, then empty, then success.
- Use a single `status` string (state machine) rather than multiple booleans to avoid impossible states (e.g., `isLoading && isError`).
- Always render a **Retry** button in the error state.
- The empty state must be checked **after** loading and error — otherwise a loading state with `data.length === 0` would render the empty message.
- Use `isFetching` (or equivalent) to show background refetches without blanking the screen.
- Distinguish `isPending` (no data yet) from `isFetching` (any fetch, including refetches).
- Test each state explicitly by mocking network responses.

**Constraints and Limitations:**
- Multiple booleans (`isLoading`, `isError`, `isEmpty`) can lead to impossible states; prefer a single status enum.
- `data.length === 0` on `null` data throws; check `data && data.length === 0` or use `Array.isArray(data) && data.length === 0`.
- Empty states are often overlooked; they are one of the most common UX gaps.
- Loading states that replace content (rather than overlaying it) cause layout shift (CLS).
- A component can be in `success` and `isFetching` simultaneously during background refetches.

### Annotated Code Example: Complete State Machine

```jsx
import { useReducer, useEffect } from 'react';

const initialState = { status: 'idle', data: null, error: null };

function reducer(state, action) {
  switch (action.type) {
    case 'request': return { status: 'loading', data: null, error: null };
    case 'success': return { status: 'success', data: action.data, error: null };
    case 'error':   return { status: 'error', data: null, error: action.error };
    case 'reset':   return initialState;
    default:        throw new Error(`Unknown action: ${action.type}`);
  }
}

function UserList() {
  const [state, dispatch] = useReducer(reducer, initialState);

  useEffect(() => {
    let ignore = false;
    dispatch({ type: 'request' });

    fetch('/api/users')
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => { if (!ignore) dispatch({ type: 'success', data }); })
      .catch(err => { if (!ignore) dispatch({ type: 'error', error: err.message }); });

    return () => { ignore = true; };
  }, []);

  if (state.status === 'idle' || state.status === 'loading') {
    return <Skeleton />;
  }

  if (state.status === 'error') {
    return (
      <div role="alert">
        <p>Error: {state.error}</p>
        <button onClick={() => dispatch({ type: 'reset' })}>Retry</button>
      </div>
    );
  }

  if (state.status === 'success' && state.data.length === 0) {
    return <p>No users found. <a href="/users/new">Create one</a>.</p>;
  }

  return (
    <ul>
      {state.data.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**Expected Output:** The component starts in `idle`, transitions to `loading` (shows skeleton), then either `error` (shows error + retry), `success` with empty data (shows "No users found"), or `success` with data (shows the list).

**Why This Output Occurs:** The reducer enforces the state machine: only one status is active at a time, preventing impossible combinations. The `ignore` flag prevents stale updates. The render logic checks statuses in priority order: loading → error → empty → success.

### Real-World Cases

- **E-commerce product list:** Loading skeleton, error with retry, empty "no products match your filters," success grid.
- **Dashboard widgets:** Each widget has its own loading, error, and empty state.
- **Search results:** Loading spinner, "no results for 'query'," error on network failure.
- **User profile:** Loading skeleton, "user not found" (404), error with retry, success profile.
- **Real-time feeds:** Loading, error, empty ("No notifications"), success with a subtle refetch indicator.

### References

- TanStack Query – Queries: https://tanstack.com/query/latest/docs/framework/react/guides/queries
- TanStack Query – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- Smashing Magazine – Better UX for Loading States: https://www.smashingmagazine.com/2024/10/better-ux-for-loading-states/
- React Official Documentation – Conditional Rendering: https://react.dev/learn/conditional-rendering

---

## Core Concept 4: Legacy Data Fetching Patterns (`useEffect`, Cleanup, Race Conditions)

### Definitions

**Core Definition:** Legacy data fetching in React uses the `useEffect` Hook to initiate a fetch after render, `useState` to store the result, and a cleanup function to prevent race conditions, stale results, and state updates on unmounted components.

**Technical Definition:** In React 16.8–18, the canonical way to fetch data in a function component was `useEffect(() => { fetch(...).then(setData) }, [deps])`. This pattern has three well-documented pitfalls. First, **race conditions**: if `deps` changes rapidly (e.g., the user types a search query), multiple requests fire, and a slow earlier response can overwrite a faster later one. The fix is an `ignore` flag set in the cleanup function, checked before every `setState`. Second, **state updates on unmounted components**: if the component unmounts before the fetch resolves, calling `setState` triggers a warning (React 17) or is a no-op (React 18+). The `ignore` flag also handles this. Third, **stale closures**: if the Effect captures an old value of a prop or state, it operates on outdated data. The fix is to include all reactive values in the dependency array. React's official documentation now recommends using a framework's data-fetching mechanism or a library like TanStack Query instead of `useEffect` for data fetching.

**Beginner-Friendly Explanation:** `useEffect` is the old way of saying "after the component renders, go fetch some data." But if the user types quickly, multiple fetches fire, and the responses can arrive out of order—like a race where the slowest runner wins. You need a "guard" that says "if this response is from an old request, ignore it." The cleanup function in `useEffect` is where you set that guard.

### Purposes

- **`useEffect`:** To initiate a data fetch after the component renders.
- **Cleanup function:** To prevent state updates after unmount and to ignore stale results.
- **`ignore` flag:** To discard results from superseded requests.
- **`AbortController`:** To cancel in-flight requests when dependencies change.
- **Dependency array:** To re-run the Effect when relevant values change.
- **`try/catch`:** To handle errors and surface them to state.

### Syntax Rules and Structure

**Basic `useEffect` Fetch:**
```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    try {
      const res = await fetch(`/api/users/${userId}`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      if (!ignore) setData(data);
    } catch (err) {
      if (!ignore) setError(err.message);
    }
  }

  load();

  return () => { ignore = true; };
}, [userId]);
```

**Component Breakdown:**
- `let ignore = false`: A per-Effect-run flag.
- `if (!ignore) setData(data)`: Guards the state update.
- `return () => { ignore = true }`: The cleanup function invalidates the Effect's results.

**With `AbortController` (Cancels the Request):**
```jsx
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const res = await fetch(`/api/users/${userId}`, {
        signal: controller.signal,
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      setData(data);
    } catch (err) {
      if (err.name !== 'AbortError') setError(err.message);
    }
  }

  load();

  return () => controller.abort();
}, [userId]);
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller with a `signal`.
- `{ signal: controller.signal }`: Passes the signal to `fetch`.
- `controller.abort()`: Cancels the request on cleanup.
- `if (err.name !== 'AbortError')`: Ignores the expected cancellation error.

**Syntax Rules:**
- Always define an async function inside the Effect and call it; `useEffect` cannot take an async callback directly.
- Always return a cleanup function that sets the `ignore` flag or aborts the request.
- Always check `res.ok` before parsing JSON.
- Always include all reactive values used inside the Effect in the dependency array.
- Prefer `AbortController` when the request API supports it; it cancels the actual network request.
- Use the `ignore` flag when the request API does not support cancellation, or as a simpler alternative.
- Never call `setState` unconditionally after an `await`; always guard it.

**Constraints and Limitations:**
- `useEffect` fetches run after render, causing a "waterfall" (render → Effect → fetch → state update → re-render).
- The `ignore` flag prevents state updates but does not cancel the network request.
- In React StrictMode (development), the Effect runs twice, causing two requests; the cleanup must handle this.
- The dependency array is error-prone; missing dependencies cause stale closures, and unstable dependencies cause infinite loops.
- React's official documentation recommends against `useEffect` for data fetching in new code; use a framework or library instead.
- Race conditions are intermittent and hard to reproduce; the absence of a guard is a bug regardless of how rare the race seems.

### Annotated Code Example: Search with Race Condition Guard

```jsx
import { useState, useEffect } from 'react';

function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [status, setStatus] = useState('idle'); // idle | loading | success | error
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!query) {
      setResults([]);
      setStatus('idle');
      return;
    }

    let ignore = false;
    const controller = new AbortController();

    setStatus('loading');
    setError(null);

    async function search() {
      try {
        const res = await fetch(
          `/api/search?q=${encodeURIComponent(query)}`,
          { signal: controller.signal }
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data = await res.json();
        if (!ignore) {
          setResults(data);
          setStatus('success');
        }
      } catch (err) {
        if (err.name === 'AbortError') return; // Expected on rapid typing
        if (!ignore) {
          setError(err.message);
          setStatus('error');
        }
      }
    }

    search();

    return () => {
      ignore = true;
      controller.abort();
    };
  }, [query]);

  if (status === 'loading') return <p>Searching…</p>;
  if (status === 'error') return <p role="alert">Error: {error}</p>;
  if (status === 'success' && results.length === 0) {
    return <p>No results for "{query}".</p>;
  }
  if (status === 'success') {
    return (
      <ul>
        {results.map(r => <li key={r.id}>{r.title}</li>)}
      </ul>
    );
  }
  return null;
}
```

**Expected Output:** As the user types a query, "Searching…" appears, then either the results, "No results for 'query'", or an error message. If the user types quickly, only the latest query's results are shown; earlier requests are aborted and ignored.

**Why This Output Occurs:** The `AbortController` cancels the previous request when `query` changes. The `ignore` flag prevents stale results from updating state. The `status` state machine ensures the correct UI for each condition. The `err.name === 'AbortError'` check ignores the expected cancellation error.

### Real-World Cases

- **Search autocomplete:** Rapid typing triggers multiple searches; the latest should win.
- **Tab switching:** Switching tabs quickly triggers multiple fetches; only the latest tab's data should display.
- **User profile switching:** Selecting different users rapidly; the latest user's profile should display.
- **Dashboard filters:** Changing filters quickly; only the latest filter combination's data should display.
- **Legacy codebases:** Many production apps still use `useEffect` fetching; understanding cleanup is essential for maintenance.

### References

- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – Fetching Data (Legacy): https://react.dev/learn/synchronizing-with-effects#fetching-data
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## Core Concept 5: Modern React 19 Data Streaming (the `use` Hook and Suspense)

### Definitions

**Core Definition:** React 19's `use` hook reads a promise (or context) and suspends the component until the promise settles, with `<Suspense>` rendering a fallback during loading and an Error Boundary catching rejections—enabling declarative, streaming-friendly data fetching.

**Technical Definition:** `use` is a new React API introduced in React 19 (stable). Unlike all other hooks, `use` can be called conditionally and inside loops because it does not allocate state or register a subscription—it simply reads the value of a resource. When called with a promise, `use` suspends the component until the promise resolves, throwing the promise to the nearest `<Suspense>` boundary. When the promise resolves, React retries rendering the component, and `use` returns the resolved value. If the promise rejects, React throws the rejection to the nearest Error Boundary. Promises passed to `use` must be stable across renders (created outside the component, memoised, or provided by a Server Component); creating a new promise on every render causes an infinite suspend-retry loop. React 19 also supports streaming promises from Server Components to Client Components, so the server can start rendering the shell while data loads in parallel.

**Beginner-Friendly Explanation:** The `use` hook is like a pause button. When a component needs data that isn't ready yet, `use` says "wait here" and React shows the nearest `<Suspense>` fallback (like a skeleton). When the data arrives, React resumes the component and renders it with the data. Unlike `useEffect`, you don't write loading or error states manually—React handles them for you. The catch is that the promise must be stable; creating a new one on every render causes an infinite loop.

### Purposes

- **Declarative loading:** To let `<Suspense>` handle loading states instead of manual `isLoading` flags.
- **Declarative errors:** To let Error Boundaries handle rejections instead of manual `isError` flags.
- **Conditional reading:** To call `use` inside conditionals and loops (unlike other hooks).
- **Context reading:** To read context with `use(Context)` (equivalent to `useContext` but conditional-friendly).
- **Server-to-client streaming:** To pass promises from Server Components to Client Components and unwrap them with `use`.
- **Simplified code:** To eliminate `useEffect`, `useState`, and cleanup boilerplate for data fetching.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { use, Suspense } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise); // Suspends until resolved
  return <h1>{user.name}</h1>;
}

function App({ userPromise }) {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `use(userPromise)`: Reads the promise; suspends if pending; throws if rejected.
- `<Suspense fallback={<ProfileSkeleton />}>`: Renders the fallback while suspended.
- `userPromise`: Must be stable across renders (created outside the component or memoised).

**Caching the Promise (Required for Client-Side Fetching):**
```jsx
import { use, Suspense } from 'react';

// Cache promises by key so they're stable across renders
const promiseCache = new Map();

function getCachedPromise(key, fn) {
  if (!promiseCache.has(key)) {
    promiseCache.set(key, fn());
  }
  return promiseCache.get(key);
}

function UserProfile({ userId }) {
  const userPromise = getCachedPromise(
    `user-${userId}`,
    () => fetch(`/api/users/${userId}`).then(r => r.json())
  );
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}

function App({ userId }) {
  return (
    <Suspense fallback={<p>Loading user…</p>}>
      <UserProfile userId={userId} />
    </Suspense>
  );
}
```

**Component Breakdown:**
- `promiseCache`: A `Map` that stores promises by key.
- `getCachedPromise`: Returns the cached promise or creates a new one.
- `use(userPromise)`: Suspends until the cached promise resolves.
- Without caching, a new promise is created on every render, causing infinite suspension.

**Error Handling with Error Boundary:**
```jsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function UserProfile({ userPromise }) {
  const user = use(userPromise); // Throws to ErrorBoundary on rejection
  return <h1>{user.name}</h1>;
}

function App({ userPromise }) {
  return (
    <ErrorBoundary fallback={<p>Failed to load user.</p>}>
      <Suspense fallback={<p>Loading…</p>}>
        <UserProfile userPromise={userPromise} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `use(userPromise)`: Throws the rejection to the nearest Error Boundary.
- `<ErrorBoundary>`: Catches the rejection and renders the fallback.
- `<Suspense>`: Handles the loading state.

**Server-to-Client Streaming (Next.js / RSC):**
```jsx
// Server Component
async function UserProfileServer({ userId }) {
  const userPromise = fetchUser(userId); // Do NOT await
  return <UserProfileClient userPromise={userPromise} />;
}

// Client Component
'use client';
import { use } from 'react';

function UserProfileClient({ userPromise }) {
  const user = use(userPromise); // Unwraps the streamed promise
  return <h1>{user.name}</h1>;
}
```

**Component Breakdown:**
- Server Component creates the promise but does **not** await it.
- The promise is passed to the Client Component as a prop.
- The Client Component uses `use` to unwrap it, suspending until resolution.
- React streams the resolved HTML as the promise settles.

**Syntax Rules:**
- Call `use` at the top level of a component or Hook (like other Hooks).
- Unlike other Hooks, `use` **can** be called conditionally and inside loops.
- Always wrap components that call `use` with a promise in `<Suspense>`.
- Always wrap them in an Error Boundary for rejection handling.
- The promise must be stable across renders; cache it or create it outside the component.
- Do not create the promise inside the component's render without caching.
- On the server, do not `await` the promise before passing it to the client; pass the promise itself.
- Do not use `use` with promises created in `useEffect`; use Suspense-enabled data sources (RSC, TanStack Query's `useSuspenseQuery`, Relay).

**Constraints and Limitations:**
- `use` requires React 19 or later.
- Creating a new promise on every render causes an infinite suspend-retry loop.
- `use` cannot be used with promises created inside `useEffect`.
- Error Boundaries are still class components (or use `react-error-boundary`).
- Client-side caching of promises is the developer's responsibility; React does not deduplicate promises.
- Streaming requires a framework that supports Server Components (Next.js App Router, Remix, Waku).
- `useSuspenseQuery` from TanStack Query provides a similar API with built-in caching and deduplication.

### Annotated Code Example: React 19 `use` with Suspense and Error Boundary

```jsx
import { use, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Cache promises by key so they're stable across renders
const cache = new Map();

function fetchUser(userId) {
  const key = `user-${userId}`;
  if (!cache.has(key)) {
    cache.set(
      key,
      fetch(`/api/users/${userId}`).then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
    );
  }
  return cache.get(key);
}

function UserProfile({ userId }) {
  const user = use(fetchUser(userId)); // Suspends until resolved
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

function ProfileSkeleton() {
  return (
    <div className="skeleton">
      <div className="skeleton-title" />
      <div className="skeleton-text" />
    </div>
  );
}

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Failed to load user: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

export default function App({ userId }) {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Suspense fallback={<ProfileSkeleton />}>
        <UserProfile userId={userId} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Expected Output:** While the user is loading, the `ProfileSkeleton` is displayed. When the data arrives, the real user profile is rendered. If the request fails, the `ErrorFallback` is displayed with a "Try again" button.

**Why This Output Occurs:** `use(fetchUser(userId))` suspends the `UserProfile` component until the cached promise resolves. React catches the suspension at the nearest `<Suspense>` boundary and renders the skeleton. When the promise resolves, React retries rendering `UserProfile` with the resolved data. If the promise rejects, React throws the error to the nearest Error Boundary, which renders the `ErrorFallback`.

### Real-World Cases

- **Server-rendered apps:** Next.js App Router streams promises from Server Components to Client Components, unwrapped with `use`.
- **Suspense-enabled data libraries:** TanStack Query's `useSuspenseQuery` and Relay's `usePreloadedQuery` integrate with Suspense.
- **Route-level data:** React Router's `loader` + `useLoaderData` provides similar ergonomics without `use`.
- **Progressive dashboards:** Each widget wrapped in its own Suspense boundary loads and renders independently.
- **Streaming SSR:** The server sends the HTML shell immediately, then streams in the data as it resolves.

### References

- React Official Documentation – `use`: https://react.dev/reference/react/use
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – Server Components: https://react.dev/reference/rsc/server-components
- React Official Documentation – Streaming: https://react.dev/reference/rsc/server-components#streaming
- react-error-boundary – Documentation: https://github.com/bvaughn/react-error-boundary
- TanStack Query – Suspense: https://tanstack.com/query/latest/docs/framework/react/guides/suspense

---

## Comparison and Decision Guidance

| Approach | Best For | Loading State | Error State | Race Conditions | Cleanup |
|---|---|---|---|---|---|
| **`fetch` + `useEffect`** | Simple legacy code | Manual `isLoading` | Manual `isError` | Manual `ignore`/`AbortController` | Manual cleanup |
| **Axios + `useEffect`** | Apps needing interceptors | Manual `isLoading` | Manual `isError` | Manual `ignore`/`AbortController` | Manual cleanup |
| **TanStack Query** | Most production apps | `isPending` | `isError` | Handled automatically | Handled automatically |
| **React 19 `use` + Suspense** | RSC, streaming, Suspense-first | `<Suspense>` fallback | Error Boundary | Handled by promise identity | Handled by React |
| **`useSuspenseQuery`** | Suspense-enabled data | `<Suspense>` fallback | Error Boundary | Handled automatically | Handled automatically |

**Decision Guidance:**
- **Start with TanStack Query** for most client-side data fetching; it handles caching, refetching, race conditions, and retries out of the box.
- **Use React 19 `use` + Suspense** when working with Server Components or streaming SSR.
- **Use `fetch` directly** only for simple, one-off requests where a library is overkill.
- **Use Axios** when you need interceptors, upload progress, or automatic JSON handling.
- **Avoid raw `useEffect` fetching** in new code; React's official documentation recommends frameworks or libraries instead.
- **Always handle all four states:** loading, error, empty, and success.
- **Cache promises** when using `use` on the client; otherwise, you create a new request on every render.
- **Wrap `use` in `<Suspense>` and an Error Boundary** — both are required for a complete UX.
- **Use `Promise.all`** for independent requests; never `await` them sequentially.

---

## References

- React Official Documentation – `use`: https://react.dev/reference/react/use
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – You Might Not Need an Effect: https://react.dev/learn/you-might-not-need-an-effect
- React Official Documentation – Server Components: https://react.dev/reference/rsc/server-components
- React Official Documentation – Streaming: https://react.dev/reference/rsc/server-components#streaming
- MDN Web Docs – Fetch API: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN Web Docs – Using Fetch: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- MDN Web Docs – AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – async function: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN Web Docs – Promise.all(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN Web Docs – Promise.allSettled(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN Web Docs – Promise.race(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
- MDN Web Docs – Promise.any(): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any
- Axios – Documentation: https://axios-http.com/docs/intro
- Axios – Interceptors: https://axios-http.com/docs/interceptors
- Axios – Request Config: https://axios-http.com/docs/req_config
- TanStack Query – Queries: https://tanstack.com/query/latest/docs/framework/react/guides/queries
- TanStack Query – Suspense: https://tanstack.com/query/latest/docs/framework/react/guides/suspense
- TanStack Query – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- react-error-boundary – Documentation: https://github.com/bvaughn/react-error-boundary
- Smashing Magazine – Better UX for Loading States: https://www.smashingmagazine.com/2024/10/better-ux-for-loading-states/