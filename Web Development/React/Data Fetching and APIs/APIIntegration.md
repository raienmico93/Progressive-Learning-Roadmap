# API Integration & Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API integration is the practice of connecting a React application to backend services — REST endpoints, GraphQL servers, or third-party APIs — and managing the request, response, error, and caching lifecycle in a way that is predictable and maintainable.

**Technical Definition:** API integration in React spans the **transport layer** (HTTP clients such as `fetch` or Axios, and GraphQL clients such as Apollo Client, urql, or graphql-request), the **concern layer** (authentication headers, request interceptors, response transformation, and error normalization), and the **scheduling layer** (when and how requests are triggered — on mount, on event, on route change, on dependency change, in parallel, sequentially, or in a waterfall). The scheduling layer determines the application's request topology: **parallel fetching** runs independent requests concurrently, **sequential fetching** chains dependent requests deliberately, and **waterfall fetching** is the accidental serialization of independent requests that should have been parallel. The distinction between intentional sequencing and accidental waterfalls is the single most important performance decision in API integration.

**Beginner-Friendly Explanation:** API integration is everything involved in talking to a server. It is not just "call `fetch` and show the data." You have to decide *what kind* of API you are talking to (REST or GraphQL), *how* you prove who you are (auth headers), *what* happens to every request and response (interceptors and transformations), *how* errors are shaped (normalization), and *when* requests fire (on page load, on a click, on a navigation, or when a filter changes). The last part — timing — is where most performance problems live. If two pieces of data do not depend on each other, fetching them one after the other is a self-inflicted slowdown.

### Key Characteristics

- **Two API paradigms:** REST (resource-oriented, HTTP verbs) and GraphQL (query-language, single endpoint, client-specified shape).
- **Cross-cutting concerns belong in one layer:** Auth, logging, transformation, and error mapping should be centralised, not duplicated per component.
- **Request timing is architecture:** Mount, event, route change, and dependency change are distinct triggers with distinct lifecycle requirements.
- **Parallelism is a design decision:** Independent requests should run concurrently; dependent requests must run sequentially.
- **Waterfalls are the dominant performance bug:** A waterfall is not a slow request; it is a request that started later than it needed to.
- **Error normalization enables consistent UI:** A single error shape lets every component render errors the same way.
- **Caching changes everything:** Query libraries (TanStack Query, SWR) and GraphQL caches (Apollo, urql) remove most manual scheduling concerns.

### Prerequisites

- JavaScript Promises, `async`/`await`, and `try`/`catch`.
- HTTP fundamentals: methods, status codes, headers, JSON.
- React function components, `useState`, `useEffect`, and event handlers.
- Familiarity with `fetch` or Axios.
- Basic knowledge of React Router or TanStack Router (for route-change fetching).
- Understanding of GraphQL queries and mutations (for the GraphQL sections).

### Related Programming Areas

- **HTTP Fundamentals** — methods, status codes, headers, CORS, credentials.
- **Fetch API and Axios** — the transport clients.
- **Data Fetching Fundamentals** — the request lifecycle and caching.
- **State Management** — server state vs. client state.
- **Performance** — waterfalls, parallelisation, caching.

---

## 1. REST APIs

### Definitions

**Core Definition:** REST (Representational State Transfer) is an architectural style for APIs that models data as resources identified by URLs and manipulated with standard HTTP methods.

**Technical Definition:** A REST API exposes **resources** (nouns such as `/users`, `/orders/42`) that are acted upon by HTTP methods: `GET` (read), `POST` (create), `PUT` (replace), `PATCH` (partial update), and `DELETE` (remove). Responses carry status codes that describe the outcome, and representations are typically JSON. REST is **stateless**: each request carries all the information needed to process it. In React, REST integration means calling resource URLs and mapping status codes to UI states.

**Beginner-Friendly Explanation:** REST is the most common style of API. You have URLs that represent things — users, products, orders — and you use HTTP verbs to read, create, update, or delete them. `GET /users/42` means "give me user 42." `DELETE /users/42` means "delete user 42."

### Purposes

- To model application data as clear, predictable resource URLs.
- To use standard HTTP semantics for caching, idempotency, and status codes.
- To integrate with widely available tooling, proxies, and CDNs.
- To keep requests stateless so they scale horizontally.
- To expose a stable contract that multiple clients can consume.

### Syntax Rules and Structure

**Resource and method mapping:**

| Operation | Method | URL | Status |
|-----------|--------|-----|--------|
| List | `GET` | `/users` | `200` |
| Read | `GET` | `/users/42` | `200` / `404` |
| Create | `POST` | `/users` | `201` + `Location` |
| Replace | `PUT` | `/users/42` | `200` / `204` |
| Partial update | `PATCH` | `/users/42` | `200` / `204` |
| Delete | `DELETE` | `/users/42` | `204` / `404` |

**General syntax:**

```js
const response = await fetch('/api/users/42', {
  method: 'GET',
  headers: { Accept: 'application/json' },
});

if (!response.ok) throw new Error(`HTTP ${response.status}`);
const user = await response.json();
```

**Syntax Rules:**

1. Model resources as plural nouns (`/users`, not `/getUsers`).
2. Use query parameters for filtering, sorting, and pagination (`/users?page=2`).
3. Use status codes semantically (`201` for creation, `204` for no body, `404` for missing).
4. Use `PATCH` for partial updates and `PUT` for full replacement.
5. Never put secrets in URLs.

**Constraints and Limitations:**

- Over-fetching and under-fetching are common: a single endpoint returns a fixed shape.
- Multiple round trips are often needed to assemble a screen.
- API versioning and breaking changes require coordination.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: CRUD Client for a REST Resource

```js
const BASE = '/api';

async function request(path, options = {}) {
  const res = await fetch(`${BASE}${path}`, {
    headers: { Accept: 'application/json', ...options.headers },
    ...options,
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.status === 204 ? null : res.json();
}

export const usersApi = {
  list: (page = 1) => request(`/users?page=${page}`),
  get: (id) => request(`/users/${id}`),
  create: (data) =>
    request('/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    }),
  update: (id, data) =>
    request(`/users/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    }),
  remove: (id) => request(`/users/${id}`, { method: 'DELETE' }),
};
```

**Expected Output:** `usersApi.get(42)` returns the user object; `usersApi.remove(42)` returns `null` on `204`; invalid IDs throw an error.

**Why this output:** A single `request` helper centralises status checking and JSON parsing. Each method maps to the correct HTTP verb and URL, giving the app a typed, consistent REST client.

#### Example 2: Using the REST Client in a Component

```tsx
function UserList({ page }) {
  const [users, setUsers] = useState([]);
  const [isPending, setIsPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setIsPending(true);

    usersApi
      .list(page)
      .then(setUsers)
      .catch((err) => { if (!controller.signal.aborted) setError(err); })
      .finally(() => setIsPending(false));

    return () => controller.abort();
  }, [page]);

  if (isPending) return <Skeleton />;
  if (error) return <ErrorBanner error={error} />;
  if (users.length === 0) return <EmptyState />;
  return <UserTable users={users} />;
}
```

**Expected Output:** The list loads for the current page, shows a skeleton while pending, an error banner on failure, and an empty state when there are no users.

**Why this output:** The component delegates HTTP details to `usersApi` and focuses on rendering states. The effect re-runs when `page` changes.

### Real-World Cases

- **Admin dashboards:** CRUD interfaces over REST resources.
- **E-commerce:** Products, carts, orders, and payments.
- **Third-party integrations:** Stripe, GitHub, and Slack REST APIs.

### References

- MDN — HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- Roy Fielding — Architectural Styles and the Design of Network-based Software Architectures — https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

---

## 2. GraphQL

### Definitions

**Core Definition:** GraphQL is a query language and runtime for APIs that lets clients request exactly the data they need from a single endpoint.

**Technical Definition:** A GraphQL server exposes a **schema** defining types, queries, mutations, and subscriptions. Clients send a query document describing the exact fields they need; the server resolves it against resolvers and returns a JSON response matching the query's shape. Unlike REST, GraphQL has one endpoint (typically `POST /graphql`) and returns `200` even for field-level errors, which are reported in an `errors` array alongside partial `data`. React clients include Apollo Client, urql, and graphql-request; Apollo and urql provide normalized caches that automatically update the UI after mutations.

**Beginner-Friendly Explanation:** GraphQL lets you ask for exactly what you want. Instead of calling three REST endpoints to build a screen, you send one query that says "give me the user's name, their last five posts, and each post's comment count." The server returns exactly that shape.

### Purposes

- To eliminate over-fetching and under-fetching by letting clients specify fields.
- To fetch related data in a single round trip.
- To provide a typed, introspectable schema as a contract.
- To enable normalized client caching that updates automatically after mutations.
- To support real-time data through subscriptions.

### Syntax Rules and Structure

**General query syntax:**

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    posts(first: 5) {
      id
      title
      commentCount
    }
  }
}
```

**Client request with `fetch`:**

```js
async function gql(query, variables) {
  const res = await fetch('/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query, variables }),
  });

  const json = await res.json();
  if (json.errors) throw new Error(json.errors[0].message);
  return json.data;
}
```

**Syntax Rules:**

1. GraphQL requests are `POST` with a JSON body containing `query` and `variables`.
2. Use variables (`$id`) instead of string interpolation.
3. Handle the `errors` array even when `data` is present.
4. Prefer a client library (Apollo, urql) for caching, fragments, and mutations.
5. Use fragments to share field selections across queries.

**Constraints and Limitations:**

- HTTP status codes are less informative; errors arrive in the response body.
- Caching is more complex than REST (normalized caches, cache keys).
- Query complexity can cause performance problems without depth limits.
- File uploads require the multipart GraphQL spec or a separate endpoint.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Apollo Client Query

```tsx
import { gql, useQuery } from '@apollo/client';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

function UserProfile({ id }) {
  const { data, loading, error } = useQuery(GET_USER, { variables: { id } });

  if (loading) return <ProfileSkeleton />;
  if (error) return <ErrorBanner error={error} />;
  return <ProfileCard user={data.user} />;
}
```

**Expected Output:** The profile renders from the normalized Apollo cache, fetching only if the data is missing or stale.

**Why this output:** `useQuery` reads from the Apollo cache first and fetches only when needed. The component receives `loading`, `error`, and `data` states without manual lifecycle code.

#### Example 2: Mutation with Cache Update

```tsx
import { gql, useMutation } from '@apollo/client';

const CREATE_POST = gql`
  mutation CreatePost($title: String!) {
    createPost(title: $title) {
      id
      title
    }
  }
`;

function NewPostForm() {
  const [createPost, { loading, error }] = useMutation(CREATE_POST, {
    refetchQueries: ['GetPosts'],
    awaitRefetchQueries: true,
  });

  async function handleSubmit(e) {
    e.preventDefault();
    const title = new FormData(e.currentTarget).get('title');
    await createPost({ variables: { title } });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" />
      <button disabled={loading}>Create</button>
      {error && <p role="alert">{error.message}</p>}
    </form>
  );
}
```

**Expected Output:** After creating a post, the `GetPosts` query refetches and the list updates. The button disables while the mutation runs.

**Why this output:** `useMutation` provides a mutation function and status flags. `refetchQueries` updates the list query after the mutation completes.

### Real-World Cases

- **Social feeds:** Users, posts, comments, and likes in one query.
- **Dashboards:** Composing multiple data sources in a single request.
- **Mobile apps:** Minimising round trips on slow networks.

### References

- GraphQL — Official Documentation — https://graphql.org/learn/
- Apollo Client — Queries — https://www.apollographql.com/docs/react/data/queries/
- Apollo Client — Mutations — https://www.apollographql.com/docs/react/data/mutations/
- urql — Official Documentation — https://nearform.com/open-source/urql/docs/

---

## 3. Authentication Headers

### Definitions

**Core Definition:** Authentication headers are the HTTP headers that carry credentials proving the identity of the requester.

**Technical Definition:** The standard header is `Authorization: <scheme> <credentials>`. Common schemes are `Bearer` (JWT or opaque access tokens) and `Basic` (base64-encoded username:password). For cookie-based sessions, the browser sends `Cookie` automatically for same-origin requests, and cross-origin requests require `credentials: 'include'` plus CORS `Access-Control-Allow-Credentials: true`. API keys are often sent in a custom header (`X-API-Key`) or as a query parameter (discouraged). In React, auth headers are attached centrally — via a `fetch` wrapper, an Axios request interceptor, or a GraphQL client's `headers` config — never per component.

**Beginner-Friendly Explanation:** Authentication headers are how you prove to the server who you are. For token-based auth you send `Authorization: Bearer <token>`. For cookie-based auth the browser sends a cookie automatically, but cross-origin requests need explicit configuration.

### Purposes

- To authenticate every request to a protected API.
- To centralise token or session attachment so components do not handle credentials.
- To support token refresh and rotation.
- To keep credentials out of URLs and logs.
- To distinguish authenticated from anonymous request flows.

### Syntax Rules and Structure

**Bearer token:**

```js
const res = await fetch('/api/me', {
  headers: { Authorization: `Bearer ${token}` },
});
```

**Cookie session:**

```js
const res = await fetch('https://api.example.com/me', {
  credentials: 'include',
});
```

**Axios interceptor:**

```js
apiClient.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

**Syntax Rules:**

1. Use the `Authorization` header with a scheme (`Bearer`, `Basic`).
2. Attach tokens centrally, not in each component.
3. For cookies, use `credentials: 'include'` cross-origin and configure CORS.
4. Never put tokens in query strings or URLs.
5. Handle `401` centrally (refresh or redirect to login).

**Constraints and Limitations:**

- Tokens in `localStorage` are vulnerable to XSS; `HttpOnly` cookies mitigate this but require CSRF protection.
- CORS preflight must allow the `Authorization` header.
- Token expiry requires refresh logic; naive retries can loop.

### Annotated Code Examples

```js
async function apiFetch(path, options = {}) {
  const token = getAccessToken();

  const res = await fetch(path, {
    ...options,
    headers: {
      Accept: 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });

  if (res.status === 401) {
    await refreshToken();
    return apiFetch(path, options); // retry once
  }

  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

**Expected Output:** Every request carries the bearer token if one exists. A `401` triggers a token refresh and a single retry; other errors throw.

**Why this output:** The wrapper centralises auth attachment and refresh. `...options.headers` allows per-request overrides. The retry is bounded to one attempt, avoiding infinite loops.

### Real-World Cases

- **JWT SPAs:** Bearer tokens with refresh rotation.
- **Cookie sessions:** `HttpOnly` cookies with `SameSite` protection.
- **API keys:** Server-to-server calls with `X-API-Key`.

### References

- MDN — Authorization header — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization
- MDN — HTTP authentication — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication
- OWASP — JSON Web Token Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html

---

## 4. Request Interceptors

### Definitions

**Core Definition:** Request interceptors are functions that run before a request is sent, allowing global modification of headers, URLs, and payloads.

**Technical Definition:** In Axios, `instance.interceptors.request.use(onFulfilled, onRejected)` registers a function that receives the request config and must return it (or a promise resolving to it). In GraphQL clients, request-level middleware or `headers` functions serve the same purpose. In a custom `fetch` wrapper, the interceptor pattern is implemented as a function that transforms the options before calling `fetch`. Interceptors are the correct place for authentication, correlation IDs, locale headers, and request logging.

**Beginner-Friendly Explanation:** Request interceptors are hooks that run on every outgoing request. They are where you attach tokens, add tracing IDs, or set headers — once, instead of in every component.

### Purposes

- To attach authentication headers to every request.
- To add correlation and tracing IDs.
- To set locale, timezone, or feature-flag headers.
- To log outgoing requests for debugging.
- To transform request payloads consistently.

### Syntax Rules and Structure

**Axios request interceptor:**

```js
apiClient.interceptors.request.use(
  (config) => {
    const token = getAccessToken();
    if (token) config.headers.Authorization = `Bearer ${token}`;
    config.headers['X-Request-Id'] = crypto.randomUUID();
    return config;
  },
  (error) => Promise.reject(error)
);
```

**Fetch wrapper interceptor:**

```js
function withInterceptors(fetchImpl, interceptors) {
  return async (url, options = {}) => {
    let config = { url, options };
    for (const interceptor of interceptors) {
      config = await interceptor(config);
    }
    return fetchImpl(config.url, config.options);
  };
}
```

**Syntax Rules:**

1. Request interceptors must return the modified config.
2. Register interceptors at module level, not inside components.
3. Order matters: the first registered interceptor runs first.
4. Use the rejection handler for setup errors.
5. Eject interceptors when they are no longer needed.

**Constraints and Limitations:**

- Interceptors registered inside components accumulate on every render.
- Interceptor errors propagate to the caller.
- Too many interceptors make request behaviour hard to trace.

### Annotated Code Examples

```js
// api/client.js
import axios from 'axios';

export const apiClient = axios.create({ baseURL: '/api' });

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  config.headers['Accept-Language'] = navigator.language;
  config.headers['X-Request-Id'] = crypto.randomUUID();
  return config;
});
```

**Expected Output:** Every request through `apiClient` carries the bearer token (if present), the browser's language, and a unique request ID.

**Why this output:** The interceptor runs before each request and mutates the config headers. Because it is registered once at module load, it does not duplicate.

### Real-World Cases

- **Auth tokens:** Attaching JWTs to every request.
- **Tracing:** Propagating `X-Request-Id` across services.
- **Localisation:** Sending `Accept-Language` from the browser.

### References

- Axios — Interceptors — https://axios-http.com/docs/interceptors
- MDN — Headers — https://developer.mozilla.org/en-US/docs/Web/API/Headers
- Apollo Client — Advanced HTTP networking — https://www.apollographql.com/docs/react/networking/advanced-http-networking/

---

## 5. Response Transformation

### Definitions

**Core Definition:** Response transformation is the process of converting raw API responses into the shapes the application needs.

**Technical Definition:** Raw responses often require transformation: snake_case to camelCase, ISO strings to `Date` objects, nested envelopes (`{ data: { user } }`) to direct objects, and pagination metadata to a normalized structure. Axios provides `transformResponse`; GraphQL clients map query results to typed models; TanStack Query's `select` option transforms cached data without affecting the cache. Transformations should be applied at the API boundary (the client module) so components receive consistent, typed data.

**Beginner-Friendly Explanation:** Response transformation means cleaning up what the server sends before your components use it — renaming fields, converting dates, and unwrapping envelopes.

### Purposes

- To normalise naming conventions (snake_case → camelCase).
- To convert transport types (ISO strings → `Date`).
- To unwrap response envelopes.
- To reshape pagination metadata into a consistent structure.
- To isolate the rest of the app from API shape changes.

### Syntax Rules and Structure

**Boundary transformation:**

```js
function toUser(raw) {
  return {
    id: raw.id,
    name: raw.full_name,
    createdAt: new Date(raw.created_at),
  };
}

export async function fetchUser(id) {
  const raw = await apiFetch(`/users/${id}`);
  return toUser(raw);
}
```

**TanStack Query `select`:**

```js
useQuery({
  queryKey: ['users'],
  queryFn: fetchUsers,
  select: (users) => users.map(toUser),
});
```

**Syntax Rules:**

1. Transform at the API boundary, not inside components.
2. Keep transformations pure and testable.
3. Use `select` to derive UI shapes without duplicating cache.
4. Validate the transformed shape (Zod) when the API is untrusted.
5. Document the transformation when it is non-obvious.

**Constraints and Limitations:**

- Transforming in `select` runs on every render unless memoised by the library.
- Over-transforming can lose fields needed by other consumers.
- Transformations that discard information make debugging harder.

### Annotated Code Examples

```js
import { z } from 'zod';

const RawUser = z.object({
  id: z.number(),
  full_name: z.string(),
  created_at: z.string(),
});

const User = RawUser.transform((raw) => ({
  id: raw.id,
  name: raw.full_name,
  createdAt: new Date(raw.created_at),
}));

export async function fetchUser(id) {
  const raw = await apiFetch(`/users/${id}`);
  return User.parse(raw);
}
```

**Expected Output:** The function returns `{ id, name, createdAt: Date }` regardless of the server's field names.

**Why this output:** `RawUser` validates the transport shape, and `.transform` produces the application model. `User.parse` throws on invalid payloads, catching API contract drift early.

### Real-World Cases

- **Legacy APIs:** Converting snake_case responses to camelCase.
- **Date handling:** Parsing ISO strings into `Date` objects.
- **Envelope unwrapping:** Extracting `response.data.data` from wrapped APIs.

### References

- Axios — Response Schema — https://axios-http.com/docs/res_schema
- TanStack Query — Select — https://tanstack.com/query/latest/docs/framework/react/guides/render-optimizations
- Zod — Transformations — https://zod.dev

---

## 6. Error Normalization

### Definitions

**Core Definition:** Error normalization is the practice of converting all possible failure shapes into a single, predictable error type that the UI can render consistently.

**Technical Definition:** Errors arrive in many forms: network failures (`TypeError` in `fetch`), HTTP errors (status + body), GraphQL field errors (an `errors` array), validation errors (`422` with field maps), and client-side exceptions. Error normalization maps each of these into a shared shape — commonly `{ type, message, status, fields?, cause? }` — so components can render one error component and one error state. Axios provides `AxiosError`; `fetch` requires manual mapping; GraphQL clients surface `ApolloError`/`CombinedError`.

**Beginner-Friendly Explanation:** Error normalization means every failure, no matter where it came from, looks the same to your components. Instead of handling "network error" in one place, "401" in another, and "validation error" in a third, you convert them all into one error object with a `message` and a `type`.

### Purposes

- To give every component a single error shape to render.
- To distinguish error categories (network, auth, validation, server) programmatically.
- To map field-level validation errors to form fields.
- To log structured errors to monitoring services.
- To avoid leaking raw server error payloads to users.

### Syntax Rules and Structure

**Normalised error shape:**

```ts
type AppError = {
  type: 'network' | 'auth' | 'forbidden' | 'not_found' | 'validation' | 'server' | 'unknown';
  message: string;
  status?: number;
  fields?: Record<string, string>;
  cause?: unknown;
};
```

**Normalisation function:**

```js
function normalizeError(error) {
  if (error.name === 'AbortError') return null; // ignore cancellations
  if (!error.response) {
    return { type: 'network', message: 'Unable to reach the server', cause: error };
  }
  const { status, data } = error.response;
  if (status === 401) return { type: 'auth', message: 'Please log in again', status };
  if (status === 403) return { type: 'forbidden', message: 'You do not have permission', status };
  if (status === 404) return { type: 'not_found', message: 'Not found', status };
  if (status === 422) return { type: 'validation', message: 'Please fix the errors', status, fields: data?.errors };
  return { type: 'server', message: data?.message ?? 'Something went wrong', status };
}
```

**Syntax Rules:**

1. Normalise at the API boundary so components only see `AppError`.
2. Ignore cancellations (`AbortError`, `CanceledError`).
3. Map status codes to semantic types.
4. Extract field-level errors for `422`.
5. Never expose raw stack traces or server internals to users.

**Constraints and Limitations:**

- GraphQL errors may include partial data; decide whether to show data, error, or both.
- Over-normalising can lose detail needed for debugging; keep the `cause`.
- Different APIs may use different validation error shapes; document the mapping.

### Annotated Code Examples

```js
async function apiFetch(path, options) {
  try {
    const res = await fetch(path, options);
    if (!res.ok) {
      const body = await res.json().catch(() => null);
      throw { response: { status: res.status, data: body } };
    }
    return res.json();
  } catch (error) {
    const normalized = normalizeError(error);
    if (normalized) throw normalized;
    throw error;
  }
}
```

**Expected Output:** All failures surface as `AppError` objects with a `type` and `message`. Cancellations are re-thrown unchanged.

**Why this output:** The wrapper converts raw failures into the shared shape at the boundary, so components can render `AppError` uniformly.

### Real-World Cases

- **Forms:** Mapping `422` field errors to inputs.
- **Auth flows:** Distinguishing `401` from `403`.
- **Monitoring:** Logging structured error types.

### References

- Axios — Handling Errors — https://axios-http.com/docs/handling_errors
- Apollo Client — Error Handling — https://www.apollographql.com/docs/react/data/error-handling/
- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

---

## 7. Fetch on Mount

### Definitions

**Core Definition:** Fetch on mount is the pattern of initiating a data request when a component first renders, typically inside `useEffect` with an empty dependency array.

**Technical Definition:** `useEffect(() => { fetchData(); }, [])` runs after the component's first commit. The request fires once per mount. In React Strict Mode, the effect runs twice in development (mount → unmount → mount) to surface missing cleanup, so the fetch must be cancellable or idempotent. Modern alternatives include `useQuery` (which fetches on mount and caches) and Server Components (which fetch before render).

**Beginner-Friendly Explanation:** Fetch on mount means "get the data as soon as the page appears." It is the most common pattern for detail pages and dashboards.

### Purposes

- To load data needed to render a component on first appearance.
- To populate a page with server data after the shell renders.
- To trigger an initial request in a client-rendered SPA.
- To provide a baseline for caching and refetching.

### Syntax Rules and Structure

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetchData({ signal: controller.signal })
    .then(setData)
    .catch((err) => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, []);
```

**Syntax Rules:**

1. Use an empty dependency array `[]` to fetch once per mount.
2. Always return a cleanup that cancels the request.
3. Handle loading, error, and empty states.
4. Do not fetch in render; fetch in the effect.
5. Prefer a query library or framework loader for caching and deduplication.

**Constraints and Limitations:**

- Creates a waterfall: HTML → JS → effect → request → render.
- No data in the initial HTML (poor SEO and LCP for content pages).
- Strict Mode double-invokes the effect in development.

### Annotated Code Examples

```tsx
function Dashboard() {
  const [stats, setStats] = useState(null);
  const [isPending, setIsPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setIsPending(true);

    fetch('/api/stats', { signal: controller.signal })
      .then((res) => res.ok ? res.json() : Promise.reject(new Error(`HTTP ${res.status}`)))
      .then(setStats)
      .catch((err) => { if (err.name !== 'AbortError') setError(err); })
      .finally(() => setIsPending(false));

    return () => controller.abort();
  }, []);

  if (isPending) return <StatsSkeleton />;
  if (error) return <ErrorBanner />;
  return <StatsGrid stats={stats} />;
}
```

**Expected Output:** A skeleton appears while stats load, then the grid renders. Navigating away aborts the request.

**Why this output:** The empty dependency array limits the effect to mount. The cleanup aborts on unmount. State covers every visual state.

### Real-World Cases

- **Detail pages:** Loading a resource by ID on navigation.
- **Dashboards:** Loading initial stats after the shell renders.
- **SPAs:** Initial data for client-rendered views.

### References

- React — useEffect — https://react.dev/reference/react/useEffect
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- TanStack Query — Quick Start — https://tanstack.com/query/latest/docs/framework/react/quick-start

---

## 8. Fetch on Event

### Definitions

**Core Definition:** Fetch on event is the pattern of initiating a request in response to a user action, such as a button click, form submission, or file selection.

**Technical Definition:** Event-driven fetching calls the request inside an event handler (`onClick`, `onSubmit`, `onChange`) rather than in an effect. The handler manages the request's pending and error states locally or via a mutation library. Unlike effect-based fetching, event-driven fetching is user-initiated, so it does not run on mount and does not need to run on dependency changes. Cancellation is handled by the handler or by an `AbortController` stored in a ref.

**Beginner-Friendly Explanation:** Fetch on event means "get the data when the user does something" — clicking a button, submitting a form, or selecting a file.

### Purposes

- To fetch data in response to explicit user action.
- To submit mutations (create, update, delete).
- To search or filter when the user submits.
- To load more results on "Load more" clicks.
- To avoid unnecessary requests before user intent.

### Syntax Rules and Structure

**Click handler:**

```tsx
async function handleLoad() {
  setIsPending(true);
  try {
    const data = await fetchMore();
    setItems((prev) => [...prev, ...data]);
  } catch (err) {
    setError(err);
  } finally {
    setIsPending(false);
  }
}
```

**Mutation library:**

```tsx
const mutation = useMutation({ mutationFn: createPost });
<button onClick={() => mutation.mutate(input)}>Create</button>
```

**Syntax Rules:**

1. Call the request inside the event handler, not in render.
2. Track pending state to disable the trigger and prevent duplicates.
3. Handle errors with a visible message.
4. Use the functional state updater when appending results.
5. Prefer a mutation library for retries, invalidation, and optimistic updates.

**Constraints and Limitations:**

- No automatic caching or deduplication unless a library is used.
- Multiple rapid clicks can trigger duplicate requests unless guarded.
- Event handlers cannot be caught by error boundaries; handle errors locally.

### Annotated Code Examples

```tsx
function LoadMore({ nextCursor, onLoaded }) {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  async function handleClick() {
    if (isPending) return; // guard against double clicks
    setIsPending(true);
    setError(null);
    try {
      const data = await fetchPage(nextCursor);
      onLoaded(data);
    } catch (err) {
      setError(err);
    } finally {
      setIsPending(false);
    }
  }

  return (
    <div>
      <button onClick={handleClick} disabled={isPending}>
        {isPending ? 'Loading…' : 'Load more'}
      </button>
      {error && <p role="alert">{error.message}</p>}
    </div>
  );
}
```

**Expected Output:** Clicking "Load more" fetches the next page, disables the button while pending, and shows an error if the request fails.

**Why this output:** The handler owns the request lifecycle. The `isPending` guard prevents duplicate requests. The button reflects the pending state.

### Real-World Cases

- **Pagination:** "Load more" and "Next page" buttons.
- **Mutations:** Creating, updating, and deleting records.
- **Search:** Submitting a search form.
- **File uploads:** Uploading on file selection.

### References

- React — Responding to Events — https://react.dev/learn/responding-to-events
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- React — useState — https://react.dev/reference/react/useState

---

## 9. Fetch on Route Change

### Definitions

**Core Definition:** Fetch on route change is the pattern of loading data when the user navigates to a new route, using router data APIs or effects keyed to the route.

**Technical Definition:** Route-change fetching can be implemented in three ways: **router loaders** (React Router `loader`, TanStack Router `loader`, Next.js Server Components), which run before the route renders; **effect-based fetching** keyed to `location.pathname` or route params; or **query hooks** with route-derived query keys. Loaders and Server Components avoid the client waterfall and render data with the route; effect-based fetching runs after the route commits and shows a loading state.

**Beginner-Friendly Explanation:** Fetch on route change means "load the data for the page the user just opened." The best implementations fetch before the page renders, so the user never sees a loading flash.

### Purposes

- To load route-specific data when navigating.
- To avoid fetching data for routes the user has not visited.
- To integrate data loading with navigation and error boundaries.
- To enable prefetching of route data on hover or intent.
- To keep route data colocated with route definitions.

### Syntax Rules and Structure

**React Router loader:**

```jsx
export async function loader({ params }) {
  const user = await fetchUser(params.id);
  return { user };
}

function UserPage() {
  const { user } = useLoaderData();
  return <ProfileCard user={user} />;
}
```

**Effect keyed to route:**

```tsx
const { pathname } = useLocation();

useEffect(() => {
  const controller = new AbortController();
  fetchPageData(pathname, { signal: controller.signal }).then(setData);
  return () => controller.abort();
}, [pathname]);
```

**Syntax Rules:**

1. Prefer router loaders or Server Components to avoid client waterfalls.
2. If using effects, key them to route params or pathname.
3. Cancel the previous request on route change.
4. Use route-level error boundaries for loader failures.
5. Prefetch route data on hover or intent where supported.

**Constraints and Limitations:**

- Effect-based route fetching shows a loading state after navigation.
- Loaders delay navigation until data resolves unless streamed.
- Route params must be validated before fetching.

### Annotated Code Examples

```tsx
// React Router Data Mode
export async function loader({ params }) {
  const res = await fetch(`/api/users/${params.id}`);
  if (!res.ok) throw new Response('Not Found', { status: 404 });
  return { user: await res.json() };
}

export function UserPage() {
  const { user } = useLoaderData();
  return <ProfileCard user={user} />;
}
```

**Expected Output:** The user data is loaded before the route renders, so the profile appears immediately without a loading flash.

**Why this output:** The loader runs before rendering and provides data via `useLoaderData`. A `404` throws a `Response` that the nearest error boundary catches.

### Real-World Cases

- **Detail pages:** Loading a resource by route param.
- **Dashboards:** Loading per-route widgets.
- **E-commerce:** Product pages with route-specific data.

### References

- React Router — Data Loading — https://reactrouter.com/start/data/route-object
- TanStack Router — Data Loading — https://tanstack.com/router/latest/docs/framework/react/guide/data-loading
- Next.js — Fetching Data — https://nextjs.org/docs/app/building-your-application/data-fetching

---

## 10. Fetch on Dependency Change

### Definitions

**Core Definition:** Fetch on dependency change is the pattern of re-fetching data whenever a specific value — a filter, an ID, or a query — changes.

**Technical Definition:** In `useEffect`, the dependency array controls when the effect re-runs. Putting `[query]` in the array re-fetches whenever `query` changes. In a query library, the dependency becomes part of the **query key** (`['search', query]`), and the library refetches automatically when the key changes while preserving cached results for previous keys. Both approaches require cancellation or key-based isolation to prevent stale responses from overwriting newer data.

**Beginner-Friendly Explanation:** Fetch on dependency change means "when this value changes, get new data." If the user types a new search term or changes a filter, the app fetches fresh results.

### Purposes

- To re-fetch when a filter, search term, or ID changes.
- To keep the UI in sync with user-selected criteria.
- To leverage query-key caching for previous values.
- To avoid manual refetch orchestration.
- To debounce or defer rapid changes.

### Syntax Rules and Structure

**Effect dependency:**

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetchResults(query, { signal: controller.signal }).then(setResults);
  return () => controller.abort();
}, [query]);
```

**Query key:**

```tsx
useQuery({
  queryKey: ['search', query],
  queryFn: () => fetchResults(query),
});
```

**Syntax Rules:**

1. Include every value the request depends on in the dependency array or query key.
2. Cancel the previous request when the dependency changes.
3. Debounce rapid changes (e.g., keystrokes) to avoid request storms.
4. Use query keys to cache per-dependency results.
5. Reset state at the start of the effect when the dependency changes.

**Constraints and Limitations:**

- Missing dependencies cause stale data.
- Over-including dependencies causes unnecessary refetches.
- Rapid changes require debouncing or `useDeferredValue`.

### Annotated Code Examples

```tsx
function SearchResults({ query }) {
  const { data, isPending, isError } = useQuery({
    queryKey: ['search', query],
    queryFn: ({ signal }) =>
      fetch(`/api/search?q=${encodeURIComponent(query)}`, { signal }).then((r) => r.json()),
    enabled: query.length > 2,
    staleTime: 30_000,
  });

  if (query.length <= 2) return <p>Type at least 3 characters.</p>;
  if (isPending) return <ResultsSkeleton />;
  if (isError) return <ErrorBanner />;
  return <ResultList results={data} />;
}
```

**Expected Output:** Results re-fetch whenever `query` changes, cached per query, and only run when the query is at least three characters.

**Why this output:** The query key includes `query`, so changing it triggers a new fetch. `enabled` prevents pointless requests for short queries. The cache preserves results for previous queries.

### Real-World Cases

- **Search:** Re-fetch on query change.
- **Filters:** Re-fetch on category or sort change.
- **Detail views:** Re-fetch when the selected ID changes.

### References

- React — useEffect dependencies — https://react.dev/reference/react/useEffect
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Dependent Queries — https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries

---

## 11. Parallel Fetching

### Definitions

**Core Definition:** Parallel fetching is the practice of issuing independent requests concurrently so the total wait time is the slowest request rather than the sum of all requests.

**Technical Definition:** Parallel fetching uses `Promise.all` (fail-fast) or `Promise.allSettled` (collect all outcomes) to start multiple requests simultaneously. In React, parallel fetching can be implemented in a single effect, in a loader, or via multiple query hooks that run independently. The key criterion is **independence**: if request B does not need the result of request A, they should start together.

**Beginner-Friendly Explanation:** Parallel fetching means "start all the requests at once." If you need three independent pieces of data, fetching them one after another makes the user wait for all three in sequence. Fetching them together makes the user wait only for the slowest one.

### Purposes

- To reduce total loading time for independent data.
- To avoid waterfalls between unrelated requests.
- To load dashboard widgets concurrently.
- To fetch route data and user data in parallel.
- To improve perceived performance on multi-resource screens.

### Syntax Rules and Structure

**`Promise.all`:**

```js
const [user, posts, stats] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchStats(id),
]);
```

**Multiple query hooks:**

```tsx
const userQuery = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
const postsQuery = useQuery({ queryKey: ['posts', id], queryFn: () => fetchPosts(id) });
```

**Syntax Rules:**

1. Use `Promise.all` only for independent requests.
2. Use `Promise.allSettled` when partial failure is acceptable.
3. In React, multiple `useQuery` hooks run in parallel automatically.
4. In loaders, return `Promise.all` results in a single object.
5. Handle partial failures explicitly when using `allSettled`.

**Constraints and Limitations:**

- `Promise.all` rejects on the first failure, discarding successful results.
- Too many concurrent requests can saturate the connection pool or rate limits.
- Parallel fetching does not help when requests are genuinely dependent.

### Annotated Code Examples

```tsx
function Dashboard({ userId }) {
  const user = useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });
  const stats = useQuery({ queryKey: ['stats', userId], queryFn: () => fetchStats(userId) });
  const activity = useQuery({ queryKey: ['activity', userId], queryFn: () => fetchActivity(userId) });

  if (user.isPending || stats.isPending || activity.isPending) return <DashboardSkeleton />;
  if (user.isError || stats.isError || activity.isError) return <DashboardError />;

  return (
    <>
      <UserHeader user={user.data} />
      <StatsGrid stats={stats.data} />
      <ActivityFeed items={activity.data} />
    </>
  );
}
```

**Expected Output:** All three requests start immediately on mount. The dashboard renders once all three resolve, or shows an error if any fails.

**Why this output:** Each `useQuery` starts its request independently and runs in parallel. The combined pending check waits for all three before rendering.

### Real-World Cases

- **Dashboards:** Loading stats, activity, and notifications together.
- **Product pages:** Fetching product, reviews, and recommendations in parallel.
- **Profile pages:** User data, posts, and followers.

### References

- MDN — Promise.all — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN — Promise.allSettled — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- TanStack Query — Parallel Queries — https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries

---

## 12. Sequential Fetching

### Definitions

**Core Definition:** Sequential fetching is the deliberate chaining of requests where each request depends on the result of the previous one.

**Technical Definition:** Sequential fetching uses `await` in sequence or `.then()` chaining. It is correct when request B requires data from request A — for example, fetching a user's ID before fetching their orders, or fetching a parent resource before its children. The cost is additive latency, so sequential fetching should be used only when the dependency is real.

**Beginner-Friendly Explanation:** Sequential fetching means "wait for the first result before starting the second." It is correct when the second request genuinely needs something from the first.

### Purposes

- To fetch a resource and then fetch data that depends on it.
- To chain authentication and data retrieval.
- To fetch a parent record before its children.
- To perform multi-step workflows where each step informs the next.
- To avoid invalid requests with missing identifiers.

### Syntax Rules and Structure

```js
const user = await fetchUser(userId);
const orders = await fetchOrders(user.id); // depends on user
```

**Syntax Rules:**

1. Use sequential fetching only when there is a real dependency.
2. Handle errors at each step.
3. Consider whether the dependency can be eliminated by changing the API.
4. Use `Promise.all` for the independent parts within a step.
5. Avoid deep chains; each step adds latency.

**Constraints and Limitations:**

- Latency is additive, not concurrent.
- A failure in an early step prevents later steps.
- Deep chains are fragile and hard to debug.

### Annotated Code Examples

```tsx
async function loadOrderDetails(orderId) {
  const order = await fetchOrder(orderId);
  const [customer, items] = await Promise.all([
    fetchCustomer(order.customerId),
    fetchOrderItems(order.id),
  ]);
  return { order, customer, items };
}
```

**Expected Output:** The order is fetched first, then the customer and items are fetched in parallel using the order's IDs.

**Why this output:** `fetchOrder` is a genuine dependency for the subsequent requests. The customer and items requests are independent of each other, so they run in parallel, avoiding an unnecessary waterfall inside the sequential chain.

### Real-World Cases

- **Order details:** Fetch order, then customer and items.
- **Auth flows:** Login, then fetch profile.
- **Multi-step wizards:** Each step depends on the previous step's data.

### References

- MDN — await — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- TanStack Query — Dependent Queries — https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries

---

## 13. Waterfall Fetching

### Definitions

**Core Definition:** Waterfall fetching is the accidental serialization of requests that could have run in parallel, where each request waits for the previous one even though it does not depend on it.

**Technical Definition:** A waterfall occurs when an effect triggers a request, which triggers another effect, which triggers another request, and so on — or when independent data is awaited sequentially in a loader or component. Waterfalls compound latency: a waterfall of five 200ms requests takes 1000ms instead of 200ms. They are the most common performance defect in data-fetching code and are often invisible in development with fast local networks. Detection requires the Network panel (sequential request timing) or profiling; remediation requires parallelising independent requests, hoisting fetches to parents or loaders, and avoiding render-driven fetching chains.

**Beginner-Friendly Explanation:** A waterfall is when requests accidentally wait for each other even though they do not need to. It is like standing in line at five different counters that could all serve you at once. It is the most common way apps become slow without any single request being slow.

### Purposes

- (Waterfalls are a defect to eliminate, not a feature.)
- To recognise the difference between intentional sequencing and accidental waterfalls.
- To identify waterfall patterns in effects, loaders, and component trees.
- To apply the correct remediation: parallelise, hoist, or prefetch.

### Syntax Rules and Structure

**Waterfall pattern (anti-pattern):**

```tsx
// ❌ Child fetches only after parent renders → serialised
function Parent() {
  const [user, setUser] = useState(null);
  useEffect(() => { fetchUser().then(setUser); }, []);
  return user ? <Child userId={user.id} /> : <Skeleton />;
}

function Child({ userId }) {
  const [posts, setPosts] = useState([]);
  useEffect(() => { fetchPosts(userId).then(setPosts); }, [userId]);
  return <PostList posts={posts} />;
}
```

**Parallel pattern:**

```tsx
// ✅ Both requests start together
function Parent() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    Promise.all([fetchUser(), fetchPosts()]).then(([u, p]) => {
      setUser(u);
      setPosts(p);
    });
  }, []);

  return user ? <Profile user={user} posts={posts} /> : <Skeleton />;
}
```

**Syntax Rules:**

1. Identify requests that do not depend on each other.
2. Start independent requests together in a parent, loader, or `Promise.all`.
3. Use route loaders to fetch route data before rendering child components.
4. Use query libraries that prefetch dependent data based on known keys.
5. Avoid chaining effects that depend on rendered data unless the dependency is real.

**Constraints and Limitations:**

- Some dependencies are real and cannot be parallelised without API changes.
- Parallelising too aggressively can increase load on the server.
- Waterfalls are hard to spot on fast local networks; test on throttled connections.

### Annotated Code Examples

**Waterfall in a loader:**

```js
// ❌ Sequential: second request waits for the first
export async function loader({ params }) {
  const user = await fetchUser(params.id);
  const posts = await fetchPosts(params.id); // does not need user
  return { user, posts };
}
```

**Fixed loader:**

```js
// ✅ Parallel: both start together
export async function loader({ params }) {
  const [user, posts] = await Promise.all([
    fetchUser(params.id),
    fetchPosts(params.id),
  ]);
  return { user, posts };
}
```

**Expected Output:** The waterfall takes the sum of both requests (e.g., 400ms); the parallel version takes the maximum (e.g., 200ms).

**Why this output:** `await` suspends until each promise settles. `Promise.all` starts both immediately and waits for both. When the requests are independent, parallelising halves the load time.

### Real-World Cases

- **Nested components:** Each level fetching its own data.
- **Loaders:** Sequential `await` of independent resources.
- **Effect chains:** A fetch triggers a state change that triggers another fetch.

### References

- web.dev — Avoid waterfalls — https://web.dev/articles/avoiding-request-waterfalls
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- TanStack Query — Parallel Queries — https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries
- Next.js — Fetching Data (parallel vs sequential) — https://nextjs.org/docs/app/building-your-application/data-fetching/fetching

---

## References

- MDN — HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods
- MDN — HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
- MDN — Authorization header — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Authorization
- MDN — HTTP authentication — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN — Promise.all — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN — Promise.allSettled — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled
- MDN — await — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- GraphQL — Official Documentation — https://graphql.org/learn/
- Apollo Client — Queries — https://www.apollographql.com/docs/react/data/queries/
- Apollo Client — Mutations — https://www.apollographql.com/docs/react/data/mutations/
- Apollo Client — Error Handling — https://www.apollographql.com/docs/react/data/error-handling/
- Apollo Client — Advanced HTTP Networking — https://www.apollographql.com/docs/react/networking/advanced-http-networking/
- urql — Official Documentation — https://nearform.com/open-source/urql/docs/
- Axios — Documentation — https://axios-http.com/docs/intro
- Axios — Interceptors — https://axios-http.com/docs/interceptors
- Axios — Handling Errors — https://axios-http.com/docs/handling_errors
- Axios — Response Schema — https://axios-http.com/docs/res_schema
- React — useEffect — https://react.dev/reference/react/useEffect
- React — Responding to Events — https://react.dev/learn/responding-to-events
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React Router — Data Loading — https://reactrouter.com/start/data/route-object
- TanStack Router — Data Loading — https://tanstack.com/router/latest/docs/framework/react/guide/data-loading
- TanStack Query — Quick Start — https://tanstack.com/query/latest/docs/framework/react/quick-start
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Dependent Queries — https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query — Parallel Queries — https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- Next.js — Fetching Data — https://nextjs.org/docs/app/building-your-application/data-fetching
- Zod — Transformations — https://zod.dev
- OWASP — JSON Web Token Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- Roy Fielding — Architectural Styles and the Design of Network-based Software Architectures — https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
- web.dev — Avoid request waterfalls — https://web.dev/articles/avoiding-request-waterfalls