# React API Integration & Middleware: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React API integration and middleware is the practice of connecting a React application to remote data sources (REST or GraphQL APIs), securing those connections with authentication, and using middleware—interceptors, transformers, and normalisers—to handle cross-cutting concerns such as token refresh, response reshaping, and error standardisation.

**Technical Definition:** React API integration encompasses the architectural decisions and implementation patterns for consuming remote APIs from a client-side application. It spans the **consumption layer** (REST vs. GraphQL, queries vs. mutations), the **security layer** (authentication headers, bearer tokens, token lifecycle management), the **middleware layer** (request/response interceptors for automated token refresh and request modification), the **transformation layer** (response restructuring for UI consumption), and the **error layer** (normalising heterogeneous API error shapes into a unified client-side contract). In modern React, these concerns are typically addressed by combining an HTTP client (Axios or native `fetch`), a server-state library (TanStack Query, Apollo Client, urql), and a validation library (Zod) at the API boundary, with interceptors or middleware bridging the gaps between them.

**Beginner-Friendly Explanation:** Your React app needs to talk to a server to get data and save changes. That conversation has a lot of moving parts: choosing how to ask (REST or GraphQL), proving who you are (tokens), handling the case when your login expires (refresh tokens), reshaping the server's answer into something your UI can use, and translating the server's error messages into friendly ones. API integration and middleware are the patterns and tools that make all of this reliable, secure, and maintainable.

### Key Characteristics

- **REST vs. GraphQL Trade-Off:** REST dominates (70%+ of job listings) for its simplicity, caching, and tooling; GraphQL excels when multiple clients need different data shapes from a complex graph.
- **Token Lifecycle Security:** Short-lived access tokens (10–15 minutes) paired with long-lived refresh tokens (7–30 days) stored in `HttpOnly` cookies is the current security best practice.
- **Interceptor-Driven Middleware:** Axios interceptors (and `fetch` wrappers) centralise cross-cutting concerns: injecting auth headers, refreshing expired tokens, retrying failed requests, and transforming responses.
- **Error Normalisation as a Boundary:** Raw API errors (Axios `AxiosError`, `FetchBaseQueryError`, GraphQL `errors` array) are normalised into a single typed contract before reaching UI, forms, or telemetry.
- **Codegen Convergence:** REST and GraphQL code generators have converged on emitting framework-agnostic primitives (options factories, typed documents) rather than library-specific hooks.
- **Runtime Validation:** Zod schemas validate and transform API responses at the boundary, preventing invalid data from reaching the UI and generating TypeScript types automatically.
- **Request Deduplication and Queuing:** When multiple requests fail with 401 simultaneously, a token-refresh queue pauses them, refreshes once, then retries all with the new token.

### Prerequisites

- Solid understanding of JavaScript promises, `async`/`await`, and the event loop.
- Working knowledge of HTTP methods, status codes, headers, and CORS.
- Familiarity with React function components and server-state libraries (TanStack Query).
- Basic understanding of JWT (JSON Web Tokens) and authentication flows.
- Awareness of Axios interceptors or the `fetch` API's `Request`/`Response` objects.

### Related Programming Areas

- **Server-State Management:** Caching, invalidation, and synchronisation (TanStack Query, Apollo Client).
- **Authentication and Authorization:** JWT, OAuth 2.0, session management, and secure token storage.
- **API Design:** REST resource modelling, GraphQL schema design, OpenAPI specifications.
- **Error Handling:** Error Boundaries, retry logic, and user-facing error messaging.
- **Type Safety:** Zod runtime validation, TypeScript codegen, and typed API clients.

### Core Concepts / Features

1. Consumption of Architectural Styles: REST APIs vs. GraphQL
2. Security Integration: Authentication Headers, Bearer Tokens, and Token Lifecycle
3. Request and Response Interceptors for Automated Token Refreshing
4. Response Transformation and Data Restructuring for UI Consumption
5. Error Normalisation: Mapping API Error Arrays to Unified Client-Side Formats

---

## Core Concept 1: Consumption of Architectural Styles — REST APIs vs. GraphQL

### Definitions

**Core Definition:** REST and GraphQL are two architectural styles for consuming APIs: REST exposes multiple resource-oriented endpoints (each returning a fixed payload shape), while GraphQL exposes a single endpoint where the client specifies exactly which fields it needs.

**Technical Definition:** REST (Representational State Transfer) is a resource-oriented architectural style where each URL identifies a resource, HTTP methods define operations, and the server determines the response shape. GraphQL is a query language and runtime where the client sends a query document specifying the exact fields, nested relationships, and arguments it needs; the server resolves the query against a type system and returns a JSON object mirroring the query's shape. In React, REST is typically consumed with `fetch` or Axios, often wrapped by TanStack Query for caching and synchronisation. GraphQL is consumed with a client library—Apollo Client, urql, or TanStack Query with a custom fetcher—using `useQuery` for reads and `useMutation` for writes. The codegen ecosystem has converged (2024–2025) on emitting framework-agnostic primitives: REST codegen (Hey API) emits TanStack Query options factories; GraphQL codegen (client preset) emits typed documents (`TypedDocumentNode`) that plug into any GraphQL client.

**Beginner-Friendly Explanation:** REST is like ordering from a fixed menu—each endpoint is a dish, and you get what the chef decides to put on the plate. GraphQL is like a build-your-own-bowl restaurant—you specify exactly which ingredients you want, and you get exactly that (no more, no less). REST is simpler to start with and works with every browser's caching; GraphQL is more flexible when different parts of your app need different data from the same underlying graph.

### Purposes

- **REST:** To consume simple, resource-oriented APIs with standard HTTP semantics and browser caching.
- **REST:** To leverage the vast ecosystem of tooling (OpenAPI, Swagger, Postman) and the 70%+ of APIs that expose REST endpoints.
- **GraphQL:** To fetch only the fields the UI needs, eliminating over-fetching and under-fetching.
- **GraphQL:** To serve multiple clients (web, mobile, third-party) with different data requirements from a single endpoint.
- **GraphQL:** To use a self-documenting schema (introspection) and typed codegen for end-to-end type safety.
- **Both:** To separate the data-fetching mechanism from the UI, allowing the UI to declare what it needs rather than how to get it.

### Syntax Rules and Structure

**REST with TanStack Query:**
```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

async function fetchUsers() {
  const res = await fetch('/api/users');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

function UserList() {
  const { data, isPending, isError } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  if (isPending) return <p>Loading…</p>;
  if (isError) return <p>Error loading users.</p>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Component Breakdown:**
- `useQuery({ queryKey, queryFn })`: Fetches and caches the user list.
- `queryKey: ['users']`: A deterministic cache key.
- `isPending` / `isError`: Built-in state flags.

**GraphQL with Apollo Client:**
```jsx
import { gql, useQuery, useMutation } from '@apollo/client';

const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

const CREATE_USER = gql`
  mutation CreateUser($name: String!, $email: String!) {
    createUser(name: $name, email: $email) {
      id
      name
    }
  }
`;

function UserList() {
  const { data, loading, error } = useQuery(GET_USERS);
  const [createUser] = useMutation(CREATE_USER, {
    refetchQueries: ['GetUsers'],
  });

  if (loading) return <p>Loading…</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      <ul>{data.users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
      <button onClick={() => createUser({ variables: { name: 'New', email: 'new@example.com' } })}>
        Add User
      </button>
    </div>
  );
}
```

**Component Breakdown:**
- `gql`: Tagged template literal that parses GraphQL documents.
- `useQuery(GET_USERS)`: Executes the query and returns `data`, `loading`, `error`.
- `useMutation(CREATE_USER, { refetchQueries })`: Executes the mutation and refetches the query.

**Codegen Convergence (2026):**
```typescript
// Hey API (REST → TanStack Query) — emits options factories
import { getUsersOptions } from './client/@tanstack/react-query.gen';
useQuery(getUsersOptions());

// graphql-codegen client preset — emits TypedDocumentNode
import { GetUsersDocument } from './gql/graphql';
useQuery(GetUsersDocument);
```

**Syntax Rules:**
- **REST:** Use `queryKey` arrays that include all variables affecting the response; use `enabled: !!id` for dependent queries.
- **GraphQL:** Define operations with `gql` or import `TypedDocumentNode` from codegen; use `variables` for parameterisation.
- **Codegen (2026):** Generate *options factories* (REST) and *typed documents* (GraphQL), not generated hooks. Avoid `@graphql-codegen/typescript-react-apollo` and `openapi-typescript-codegen` hook generators—they are deprecated or stale.
- **REST:** Use OpenAPI + Hey API with the `@tanstack/react-query` plugin for end-to-end type safety.
- **GraphQL:** Use `graphql-codegen` client preset with `documentMode: "string"` for TanStack Query, or default AST mode for Apollo/urql.
- **Small GraphQL schemas:** Consider `gql.tada` for zero-codegen TypeScript inference.

**Constraints and Limitations:**
- REST over-fetches (returns fields the UI does not need) and under-fetches (requires multiple round trips for related data).
- GraphQL adds operational overhead: schema stitching, N+1 query problems, caching complexity, and a steeper learning curve.
- GraphQL's single endpoint bypasses HTTP caching (GET-based caching); use persisted queries or client-side caching.
- REST is simpler to cache with standard HTTP headers; GraphQL requires client-side normalised caches (Apollo `InMemoryCache`, urql `Graphcache`).
- Codegen adds a build step; `gql.tada` avoids it for small schemas but lacks the full feature set of the client preset.
- REST remains the dominant choice for public APIs and non-TypeScript consumers; GraphQL wins when clients have genuinely different data requirements.

### Annotated Code Example: GraphQL Query and Mutation with Typed Documents

```typescript
// gql/graphql.ts — generated by graphql-codegen client preset
import { TypedDocumentNode as DocumentNode } from '@graphql-typed-document-node/core';

export const GetUsersDocument = {
  kind: 'Document',
  definitions: [/* … */],
} as unknown as DocumentNode<GetUsersQuery, GetUsersQueryVariables>;

export type GetUsersQuery = {
  users: Array<{ id: string; name: string; email: string }>;
};
```

```tsx
// UserList.tsx
import { useQuery, useMutation } from '@apollo/client';
import { GetUsersDocument, CreateUserDocument, type GetUsersQuery } from './gql/graphql';

function UserList() {
  const { data, loading, error } = useQuery<GetUsersQuery>(GetUsersDocument);

  const [createUser, { loading: creating }] = useMutation(CreateUserDocument, {
    refetchQueries: [GetUsersDocument],
  });

  if (loading) return <p>Loading users…</p>;
  if (error) return <p role="alert">Failed to load users: {error.message}</p>;

  return (
    <div>
      <ul>
        {data?.users.map((user) => (
          <li key={user.id}>
            {user.name} — {user.email}
          </li>
        ))}
      </ul>

      <button
        disabled={creating}
        onClick={() =>
          createUser({
            variables: { name: 'New User', email: 'new@example.com' },
          })
        }
      >
        {creating ? 'Creating…' : 'Create User'}
      </button>
    </div>
  );
}
```

**Expected Output:** The component displays a loading message while the query runs, then renders the user list with names and emails. Clicking "Create User" runs the mutation, disables the button while creating, and refetches the user list to show the new user.

**Why This Output Occurs:** `GetUsersDocument` is a `TypedDocumentNode` generated by the client preset. `useQuery` executes it and returns typed `data`, `loading`, and `error`. `useMutation` executes `CreateUserDocument` with `variables`, and `refetchQueries` triggers a refetch of the user list after the mutation completes. The generated types ensure that `data.users` and `user.email` are type-safe.

### Real-World Cases

- **Public APIs:** REST for third-party APIs (Stripe, GitHub REST) where the schema is fixed and caching is important.
- **Mobile + web clients:** GraphQL to avoid multiple round trips and fetch only the fields each client needs.
- **Hybrid architecture:** REST for core backend services, GraphQL as a frontend aggregation layer (a common 2026 pattern).
- **Type-safe codegen:** OpenAPI + Hey API for REST; graphql-codegen client preset or `gql.tada` for GraphQL.
- **Complex data graphs:** GraphQL for social networks, e-commerce catalogues, and analytics dashboards with deeply nested relationships.

### References

- React Official Documentation – Server Components: https://react.dev/reference/rsc/server-components
- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- Apollo Client – React Documentation: https://www.apollographql.com/docs/react
- GraphQL Codegen – Client Preset: https://the-guild.dev/graphql/codegen/plugins/presets/preset-client
- Hey API – OpenAPI to TypeScript: https://heyapi.dev/
- gql.tada – Zero-codegen GraphQL: https://gql-tada.0no.co/
- tRPC vs REST vs GraphQL: Type-Safe APIs in Next.js 2026: https://www.pkgpulse.com/guides/trpc-vs-rest-vs-graphql-typesafe-apis-nextjs-2026

---

## Core Concept 2: Security Integration — Authentication Headers, Bearer Tokens, and Token Lifecycle

### Definitions

**Core Definition:** Security integration is the practice of attaching authentication credentials (typically bearer tokens) to API requests via headers, and managing the token lifecycle—issuance, storage, refresh, rotation, and revocation—to maintain a secure, persistent session.

**Technical Definition:** Bearer authentication (RFC 6750) uses the `Authorization: Bearer <token>` header, where the token is a signed credential (usually a JWT) that grants access to protected resources. The current security best practice is a **dual-token architecture**: a short-lived **access token** (10–15 minutes) used on every API request, and a long-lived **refresh token** (7–30 days) used only to obtain new access tokens. The refresh token is stored in an `HttpOnly`, `Secure`, `SameSite=Strict` cookie that JavaScript cannot read (mitigating XSS), while the access token is held in memory (React state, a ref, or a module-level variable) and never in `localStorage` or `sessionStorage`. Refresh tokens are rotated on every use, and the old token is invalidated, detecting replay attacks. Token lifecycle management includes sign-in (exchanging credentials for tokens), refresh (silently obtaining a new access token), and logout (clearing tokens and invalidating the refresh token server-side).

**Beginner-Friendly Explanation:** Think of an access token as a temporary key card that opens the building for 15 minutes. The refresh token is the long-term membership card you keep in your wallet (an `HttpOnly` cookie the JavaScript can't touch). When the key card expires, your app silently uses the membership card to get a new key card—you never see it happen. Storing the key card in `localStorage` is like taping it to the outside of your wallet—anyone who breaks in (XSS) can steal it. Keeping it in memory is safer because it disappears when the tab closes.

### Purposes

- To authenticate API requests by attaching a bearer token in the `Authorization` header.
- To minimise the attack surface by storing the access token in memory (not `localStorage`) and the refresh token in an `HttpOnly` cookie.
- To maintain a persistent session without forcing the user to log in again when the access token expires.
- To detect and mitigate refresh token theft through rotation and invalidation.
- To centralise token management in an auth context or API client, keeping tokens out of component code.
- To comply with security best practices (short access token TTL, secure cookie flags, server-side invalidation on logout).

### Syntax Rules and Structure

**Token Storage Strategy:**

| Token | Storage | Rationale |
|---|---|---|
| **Access token** | In-memory (React state/ref/module) | XSS cannot read memory; disappears on tab close |
| **Refresh token** | `HttpOnly` + `Secure` + `SameSite=Strict` cookie | JavaScript cannot read it; sent only to the refresh endpoint path |
| **Session metadata** | React Context | Exposes `user`, `isAuthenticated`, `loading` to components |

**Auth Context (React):**
```tsx
import { createContext, useContext, useState, useCallback, useMemo } from 'react';

interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | null>(null);

let accessToken: string | null = null; // Module-level memory store

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (credentials: Credentials) => {
    const res = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include', // Send/receive cookies
      body: JSON.stringify(credentials),
    });
    const { user, accessToken: token } = await res.json();
    accessToken = token; // Store in memory only
    setUser(user);
  }, []);

  const logout = useCallback(async () => {
    await fetch('/api/auth/logout', { method: 'POST', credentials: 'include' });
    accessToken = null;
    setUser(null);
  }, []);

  const value = useMemo(
    () => ({ user, isAuthenticated: !!user, login, logout }),
    [user, login, logout]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}

export function getAccessToken() {
  return accessToken;
}
```

**Attaching the Bearer Header (Request Interceptor):**
```typescript
import axios from 'axios';
import { getAccessToken } from './auth';

export const api = axios.create({
  baseURL: '/api',
  withCredentials: true, // Send cookies on every request
});

api.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

**Component Breakdown:**
- `accessToken`: A module-level variable—never in `localStorage`.
- `credentials: 'include'`: Ensures cookies (including the refresh token) are sent.
- `withCredentials: true`: Axios equivalent for cookie sending.
- Request interceptor: Injects `Authorization: Bearer <token>` on every request.

**Syntax Rules:**
- Never store the access token in `localStorage` or `sessionStorage`; use in-memory storage (module variable, React state/ref, or a state library).
- Store the refresh token in an `HttpOnly`, `Secure`, `SameSite=Strict` cookie set by the server.
- Use `credentials: 'include'` (fetch) or `withCredentials: true` (Axios) for cross-origin requests that need cookies.
- Set the refresh token cookie's `path` to the refresh endpoint only, so it is not sent on every request.
- Rotate refresh tokens on every use and invalidate the old token server-side.
- Clear the access token from memory on logout and invalidate the refresh token server-side.
- Use short access token TTLs (10–15 minutes) and longer refresh token TTLs (7–30 days).
- Hash refresh tokens before storing them in the database.
- Use `SameSite=Strict` or `SameSite=Lax` to mitigate CSRF; pair with CSRF tokens if needed.

**Constraints and Limitations:**
- In-memory access tokens are lost on page refresh; the app must silently refresh on mount (e.g., call the refresh endpoint in an auth bootstrap).
- `HttpOnly` cookies are vulnerable to CSRF if `SameSite` is not set correctly.
- Cross-origin cookies require `SameSite=None; Secure` and CORS `Access-Control-Allow-Credentials: true`.
- Multiple tabs share `HttpOnly` cookies but not in-memory tokens; each tab must refresh independently on load.
- Token rotation requires server-side storage of the current refresh token and detection of reused (stolen) tokens.
- Logout must invalidate the refresh token server-side; clearing the cookie alone is insufficient if the token was stolen.

### Annotated Code Example: Secure Login → Authenticated Request → Silent Refresh

```typescript
// api/auth.ts — API client with request interceptor
import axios from 'axios';

let accessToken: string | null = null;

export function setAccessToken(token: string | null) {
  accessToken = token;
}

export function getAccessToken() {
  return accessToken;
}

export const api = axios.create({
  baseURL: '/api',
  withCredentials: true,
});

api.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

```tsx
// auth/AuthProvider.tsx — login, logout, bootstrap
import { createContext, useContext, useState, useEffect, useCallback, useMemo } from 'react';
import { api, setAccessToken } from '../api/auth';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Bootstrap: silently refresh on mount
  useEffect(() => {
    api.post('/auth/refresh')
      .then(({ data }) => {
        setAccessToken(data.accessToken);
        setUser(data.user);
      })
      .catch(() => {
        setAccessToken(null);
        setUser(null);
      })
      .finally(() => setLoading(false));
  }, []);

  const login = useCallback(async (credentials) => {
    const { data } = await api.post('/auth/login', credentials);
    setAccessToken(data.accessToken);
    setUser(data.user);
  }, []);

  const logout = useCallback(async () => {
    await api.post('/auth/logout');
    setAccessToken(null);
    setUser(null);
  }, []);

  const value = useMemo(
    () => ({ user, loading, isAuthenticated: !!user, login, logout }),
    [user, loading, login, logout]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  return useContext(AuthContext);
}
```

**Expected Output:** On page load, the app silently calls `/auth/refresh`; if the refresh token cookie is valid, the user is authenticated and the access token is stored in memory. If not, the user is logged out. Logging in sends credentials, receives a new access token in the response body, and sets a refresh token cookie. Logging out clears the access token from memory and invalidates the refresh token server-side.

**Why This Output Occurs:** The `AuthProvider` bootstraps by calling the refresh endpoint, which reads the `HttpOnly` refresh token cookie and returns a new access token. The request interceptor injects the bearer token on every subsequent request. The access token lives only in memory, so XSS cannot steal it. The refresh token lives in an `HttpOnly` cookie, so JavaScript cannot read it.

### Real-World Cases

- **SaaS applications:** Short-lived access tokens + `HttpOnly` refresh tokens for secure, persistent sessions.
- **Multi-tab apps:** Each tab bootstraps independently; the shared `HttpOnly` cookie allows silent refresh without re-login.
- **Mobile web apps:** In-memory access tokens survive in-app navigation but are cleared when the browser is closed.
- **Banking and finance:** Token rotation and server-side invalidation are mandatory for compliance.
- **Third-party integrations:** OAuth 2.0 with bearer tokens for accessing external APIs.

### References

- RFC 6750 – The OAuth 2.0 Authorization Framework: Bearer Token Usage: https://www.rfc-editor.org/rfc/rfc6750.html
- RFC 9700 – Best Current Practice for OAuth 2.0 Security: https://www.rfc-editor.org/rfc/rfc9700.html
- OWASP – JSON Web Token for Java Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- MDN Web Docs – Set-Cookie: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN Web Docs – SameSite cookies: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- Auth0 – Refresh Token Rotation: https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation

---

## Core Concept 3: Request and Response Interceptors for Automated Token Refreshing

### Definitions

**Core Definition:** Interceptors are functions that Axios (or a `fetch` wrapper) calls before a request is sent or after a response is received, enabling automated token refresh, request modification, and response transformation in a single, centralised place.

**Technical Definition:** Axios interceptors are registered with `axios.interceptors.request.use(onFulfilled, onRejected)` and `axios.interceptors.response.use(onFulfilled, onRejected)`. Request interceptors receive the `config` object and can modify headers, URL, or body before the request is sent. Response interceptors receive the `response` object (for 2xx) or the `error` object (for non-2xx) and can transform data, handle errors, or retry requests. The automated token refresh pattern detects 401 responses, pauses all concurrent requests in a queue, calls the refresh endpoint **once**, updates the access token, then retries all queued requests with the new token. This prevents the "thundering herd" problem where five simultaneous 401s trigger five refresh requests. Libraries like `axios-auth-refresh-queue` implement this pattern with zero configuration, using a browser lock to coordinate across tabs.

**Beginner-Friendly Explanation:** Imagine five people try to enter a building at the same time, and all five key cards are expired. Without coordination, all five go back to the front desk to get new cards—chaos. With an interceptor queue, the first person goes to the front desk, the other four wait, the first person gets a new card, and then all five enter together. The interceptor is the security guard who coordinates this. Axios interceptors let you intercept requests and responses to add headers, refresh tokens, or retry failed calls—all in one place instead of in every component.

### Purposes

- To inject the `Authorization: Bearer <token>` header on every outgoing request automatically.
- To detect 401 Unauthorized responses and refresh the access token without user intervention.
- To queue concurrent requests during a refresh so only one refresh request is made.
- To retry the original requests automatically after the token is refreshed.
- To redirect to login if the refresh token is also invalid or expired.
- To modify request headers globally (e.g., `Accept`, `X-Request-Id`, locale).
- To transform response data globally (e.g., unwrapping `{ data: ... }` envelopes).
- To centralise error logging and telemetry without duplicating code in every component.

### Syntax Rules and Structure

**Axios Request Interceptor (Inject Token):**
```typescript
import axios from 'axios';

const api = axios.create({ baseURL: '/api', withCredentials: true });

api.interceptors.request.use(
  (config) => {
    const token = getAccessToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
```

**Axios Response Interceptor (Refresh on 401 with Queue):**
```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';

let isRefreshing = false;
let failedQueue: Array<{
  resolve: (token: string) => void;
  reject: (error: unknown) => void;
}> = [];

function processQueue(error: unknown, token: string | null = null) {
  failedQueue.forEach((prom) => {
    if (error) prom.reject(error);
    else prom.resolve(token!);
  });
  failedQueue = [];
}

api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    // Only handle 401, and only once per request
    if (error.response?.status !== 401 || originalRequest._retry) {
      return Promise.reject(error);
    }

    // If a refresh is already in progress, queue this request
    if (isRefreshing) {
      return new Promise<string>((resolve, reject) => {
        failedQueue.push({ resolve, reject });
      }).then((token) => {
        originalRequest.headers.Authorization = `Bearer ${token}`;
        return api(originalRequest);
      });
    }

    originalRequest._retry = true;
    isRefreshing = true;

    try {
      const { data } = await axios.post(
        '/api/auth/refresh',
        {},
        { withCredentials: true }
      );
      const newToken = data.accessToken;
      setAccessToken(newToken);
      processQueue(null, newToken);
      originalRequest.headers.Authorization = `Bearer ${newToken}`;
      return api(originalRequest);
    } catch (refreshError) {
      processQueue(refreshError, null);
      setAccessToken(null);
      window.location.href = '/login';
      return Promise.reject(refreshError);
    } finally {
      isRefreshing = false;
    }
  }
);
```

**Component Breakdown:**
- `isRefreshing`: A flag indicating whether a refresh is already in progress.
- `failedQueue`: An array of pending requests waiting for the new token.
- `originalRequest._retry`: A flag preventing infinite retry loops.
- `processQueue(error, token)`: Resolves or rejects all queued requests.
- `axios.post('/api/auth/refresh', {}, { withCredentials: true })`: Calls the refresh endpoint with the `HttpOnly` cookie.
- `originalRequest.headers.Authorization = ...; return api(originalRequest)`: Retries the original request with the new token.

**Using a Library (`axios-auth-refresh-queue`):**
```typescript
import axios from 'axios';
import { applyAuthTokenInterceptor } from 'axios-auth-refresh-queue';

const api = axios.create({ baseURL: '/api' });

applyAuthTokenInterceptor(api, {
  requestRefresh: async (failedRequest) => {
    const { data } = await axios.post('/api/auth/refresh', {}, { withCredentials: true });
    return data.accessToken;
  },
  onRefreshFailure: () => {
    window.location.href = '/login';
  },
});
```

**Component Breakdown:**
- `applyAuthTokenInterceptor`: Wraps the Axios instance with the queueing refresh logic.
- `requestRefresh`: Returns the new token (or throws).
- `onRefreshFailure`: Called if the refresh itself fails.

**Syntax Rules:**
- Always guard against infinite retry loops with a `_retry` flag on the original request config.
- Always queue concurrent 401s and refresh only once; never fire multiple refresh requests simultaneously.
- Always use `withCredentials: true` (Axios) or `credentials: 'include'` (fetch) on the refresh request so the `HttpOnly` cookie is sent.
- Redirect to login (or trigger a logout action) if the refresh itself fails.
- Do not retry requests that are themselves the refresh endpoint.
- Use a library (`axios-auth-refresh-queue`) for cross-tab coordination via browser locks if needed.
- Request interceptors run in reverse order of registration for the `onRejected` handler; be mindful of ordering.
- Response interceptors can transform data globally, but prefer transforming in the API client or query function for clarity.

**Constraints and Limitations:**
- The queue pattern must handle both resolve and reject paths; failing to reject queued requests leaves promises hanging.
- Cross-tab coordination requires a browser lock (Web Locks API) or a shared worker; a simple in-memory flag only works within one tab.
- The refresh endpoint must be excluded from the 401 interceptor (otherwise a 401 on refresh triggers another refresh).
- `originalRequest._retry` is a custom property; TypeScript requires extending the config type.
- Interceptors are global to the Axios instance; multiple instances require separate interceptors or a shared factory.
- If the access token is stored in memory and the page refreshes, the interceptor has no token until the bootstrap refresh completes; ensure the bootstrap runs before rendering protected content.

### Annotated Code Example: Complete Token Refresh Queue

```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';

// --- Token store (in-memory only) ---
let accessToken: string | null = null;
export const setAccessToken = (t: string | null) => { accessToken = t; };
export const getAccessToken = () => accessToken;

// --- Axios instance ---
export const api = axios.create({
  baseURL: '/api',
  withCredentials: true, // Send HttpOnly refresh cookie
});

// --- Request interceptor: inject Bearer token ---
api.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// --- Response interceptor: refresh on 401 with queue ---
let isRefreshing = false;
let queue: Array<{ resolve: (t: string) => void; reject: (e: unknown) => void }> = [];

function flushQueue(error: unknown, token: string | null = null) {
  queue.forEach(({ resolve, reject }) => (error ? reject(error) : resolve(token!)));
  queue = [];
}

api.interceptors.response.use(
  (res) => res,
  async (error: AxiosError) => {
    const original = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    // Pass through non-401 errors and already-retried requests
    if (error.response?.status !== 401 || original._retry) {
      return Promise.reject(error);
    }

    // Queue concurrent 401s while a refresh is in flight
    if (isRefreshing) {
      return new Promise<string>((resolve, reject) => {
        queue.push({ resolve, reject });
      }).then((token) => {
        original.headers.Authorization = `Bearer ${token}`;
        return api(original);
      });
    }

    original._retry = true;
    isRefreshing = true;

    try {
      // Use a bare axios call to avoid the interceptor on the refresh request
      const { data } = await axios.post(
        '/api/auth/refresh',
        {},
        { withCredentials: true }
      );
      setAccessToken(data.accessToken);
      flushQueue(null, data.accessToken);
      original.headers.Authorization = `Bearer ${data.accessToken}`;
      return api(original);
    } catch (refreshError) {
      flushQueue(refreshError, null);
      setAccessToken(null);
      window.location.href = '/login';
      return Promise.reject(refreshError);
    } finally {
      isRefreshing = false;
    }
  }
);
```

**Expected Output:** When the access token expires and the app fires five simultaneous requests, the first receives a 401, starts a refresh, and queues the other four. The refresh succeeds, all five requests retry with the new token, and the user never notices. If the refresh fails, all five are rejected, the access token is cleared, and the user is redirected to `/login`.

**Why This Output Occurs:** The `isRefreshing` flag and `queue` array implement the standard "refresh once, retry all" pattern. The `original._retry` flag prevents infinite loops. The refresh request uses a bare `axios.post` (not the `api` instance) to avoid re-triggering the interceptor. `withCredentials: true` ensures the `HttpOnly` refresh cookie is sent.

### Real-World Cases

- **Single-page applications:** Silent token refresh keeps users logged in for days without re-entering credentials.
- **Multi-tab applications:** `axios-auth-refresh-queue` uses Web Locks to coordinate refresh across tabs.
- **Mobile web:** Short access token TTLs (5–15 minutes) paired with silent refresh for a seamless experience.
- **Micro-frontends:** Each micro-frontend uses its own Axios instance with the same interceptor factory.
- **API gateways:** Interceptors add tracing headers (`X-Request-Id`) and locale headers globally.

### References

- Axios – Interceptors: https://axios-http.com/docs/interceptors
- axios-auth-refresh-queue – npm: https://www.npmjs.com/package/axios-auth-refresh-queue
- Axios – Handling Errors: https://axios-http.com/docs/handling_errors
- MDN Web Docs – Web Locks API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API
- GitHub – elaad24/react-auth-best-practice: https://github.com/elaad24/react-auth-best-practice-

---

## Core Concept 4: Response Transformation and Data Restructuring for UI Consumption

### Definitions

**Core Definition:** Response transformation is the practice of reshaping raw API responses—which may be nested, over-fetched, inconsistently named, or enveloped in `data`/`meta` wrappers—into a structure optimised for React component consumption.

**Technical Definition:** Response transformation operates at the boundary between the API client and the UI. Raw API responses often include envelope wrappers (`{ data: [...], meta: { total, page } }`), inconsistent field names (`snake_case` vs. `camelCase`), deeply nested relationships, redundant fields, or display-unfriendly formats (ISO date strings, numeric enums). Transformation functions normalise these into a shape that components can consume directly: flat arrays, camelCase fields, parsed dates, boolean flags, and computed display strings. In React, transformation can occur at three layers: (1) **in the API client** (Axios response interceptor or `fetch` wrapper), (2) **in the query function** (TanStack Query `queryFn`), or (3) **in a `select` function** (TanStack Query's `select` option, which memoises the transformed result). Zod's `.transform()` method allows validation and transformation in a single schema, making it the recommended approach for type-safe, runtime-validated transformation. Transformation in the query function or `select` is preferred over interceptors because it keeps the API client generic and the transformation co-located with the query that needs it.

**Beginner-Friendly Explanation:** The server sends data in its own format—maybe with nested objects, `snake_case` names, and a wrapper around everything. Your UI wants flat, simple data. Transformation is the step where you convert one to the other. Instead of making every component deal with `response.data.items[0].attributes.name`, you transform it once into `items[0].name`. Zod is the best tool for this because it validates *and* transforms in one step—if the data is wrong, you get an error; if it's right, you get the shape you want.

### Purposes

- To unwrap API envelopes (`{ data, meta }`) so components receive the payload directly.
- To rename fields from `snake_case` to `camelCase` (or vice versa) for consistency.
- To parse and format dates, numbers, and currency strings for display.
- To flatten nested relationships (e.g., `user.address.city` → `user.city`) for simpler UI code.
- To derive computed fields (e.g., `fullName = firstName + ' ' + lastName`, `isExpired = expiry < now`).
- To normalise inconsistent API shapes (e.g., some endpoints return `{ data: [...] }`, others return `[...]`).
- To validate runtime data against a Zod schema, preventing invalid data from reaching the UI.
- To memoise expensive transformations via TanStack Query's `select` option.

### Syntax Rules and Structure

**Transformation in the Query Function (TanStack Query):**
```typescript
import { useQuery } from '@tanstack/react-query';
import { z } from 'zod';

// 1. Validate and transform with Zod
const UserSchema = z.object({
  id: z.number(),
  first_name: z.string(),
  last_name: z.string(),
  email_address: z.string().email(),
  created_at: z.string().datetime(),
});

const UserListSchema = z.object({
  data: z.array(UserSchema),
  meta: z.object({ total: z.number() }),
});

type User = z.infer<typeof UserSchema>;

// 2. Transform in the queryFn
async function fetchUsers(): Promise<User[]> {
  const res = await fetch('/api/users');
  const json = await res.json();
  const parsed = UserListSchema.parse(json); // Throws on invalid shape

  return parsed.data.map((u) => ({
    id: u.id,
    firstName: u.first_name,
    lastName: u.last_name,
    email: u.email_address,
    fullName: `${u.first_name} ${u.last_name}`,
    createdAt: new Date(u.created_at),
  }));
}

function UserList() {
  const { data: users } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });
  // users is already transformed: { id, firstName, fullName, createdAt: Date }
}
```

**Component Breakdown:**
- `UserSchema`: Validates the raw API shape.
- `UserListSchema`: Validates the envelope (`{ data, meta }`).
- `parsed.data.map(...)`: Transforms `snake_case` to `camelCase`, adds `fullName`, parses `createdAt`.
- The component receives a clean, typed array.

**Transformation with Zod `.transform()` (Single Step):**
```typescript
const UserSchema = z.object({
  id: z.number(),
  first_name: z.string(),
  last_name: z.string(),
  created_at: z.string().datetime(),
}).transform((u) => ({
  id: u.id,
  firstName: u.first_name,
  lastName: u.last_name,
  fullName: `${u.first_name} ${u.last_name}`,
  createdAt: new Date(u.created_at),
}));

type User = z.infer<typeof UserSchema>;
// { id: number; firstName: string; lastName: string; fullName: string; createdAt: Date }
```

**Component Breakdown:**
- `.transform()`: Chains a transformation after validation.
- `z.infer<typeof UserSchema>`: Infers the *transformed* type, not the raw type.
- The schema is now the single source of truth for both validation and transformation.

**Transformation with TanStack Query's `select` (Memoised):**
```tsx
function UserNames() {
  const { data: names } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers, // returns User[]
    select: (users) => users.map((u) => u.fullName), // memoised
  });
  // names is string[] — only recomputed when users changes
  return <ul>{names?.map((n) => <li key={n}>{n}</li>)}</ul>;
}
```

**Component Breakdown:**
- `select`: Transforms the query result; the result is memoised and only recomputed when the underlying data changes.
- The component receives `string[]` instead of `User[]`.

**Generic Response Normaliser (Utility):**
```typescript
// @sgvolpe/api-response-normalizer style
function extractData(response: unknown): unknown[] {
  if (Array.isArray(response)) return response;
  if (response && typeof response === 'object') {
    for (const key of ['data', 'result', 'items', 'records']) {
      const value = (response as Record<string, unknown>)[key];
      if (Array.isArray(value)) return value;
      if (value && typeof value === 'object' && Array.isArray((value as any).data)) {
        return (value as any).data;
      }
    }
  }
  return [];
}
```

**Component Breakdown:**
- `extractData`: Handles common envelope shapes (`{ data }`, `{ result }`, `{ items }`, nested `{ data: { data: [...] } }`).
- Returns an empty array for unrecognised shapes, preventing crashes.

**Syntax Rules:**
- Prefer transformation in the query function or `select` over Axios interceptors; keep the API client generic.
- Use Zod's `.transform()` to validate and transform in one step; `z.infer` reflects the transformed type.
- Use TanStack Query's `select` for memoised, derived views of the same query data.
- Never mutate the original response object; always return a new object from the transformer.
- Handle missing or null fields explicitly in the transformer (defaults, optional chaining).
- Use a normaliser utility for APIs with inconsistent envelope shapes.
- Keep transformations pure and testable; they are just functions.
- When transforming in a `select`, ensure the transformer does not depend on external mutable state (it may be memoised and not re-run).

**Constraints and Limitations:**
- Transformations in interceptors apply globally, which may be wrong for endpoints with different shapes.
- Zod `.transform()` makes the schema input type and output type different; use `z.input` for the raw type and `z.output` for the transformed type.
- `select` results are memoised but not deeply compared; returning a new object on every call defeats memoisation.
- Flattening nested relationships can lose information needed by other views; consider keeping the raw data and transforming per-view.
- Heavy transformations in `select` run on every render if the selector identity changes; use `useCallback` or a stable selector.
- Date parsing with `new Date(string)` is implementation-dependent; use a library (date-fns, dayjs) for robust parsing.

### Annotated Code Example: Zod Validation + Transformation + TanStack Query `select`

```typescript
import { z } from 'zod';
import { useQuery } from '@tanstack/react-query';

// 1. Raw API schema (snake_case, envelope)
const ApiProductSchema = z.object({
  product_id: z.number(),
  product_name: z.string(),
  unit_price: z.number(),
  stock_quantity: z.number(),
  created_at: z.string().datetime(),
});

const ApiResponseSchema = z.object({
  data: z.array(ApiProductSchema),
  meta: z.object({ total: z.number(), page: z.number() }),
});

// 2. Transformed UI schema (camelCase, computed fields)
const ProductSchema = ApiProductSchema.transform((p) => ({
  id: p.product_id,
  name: p.product_name,
  price: p.unit_price,
  inStock: p.stock_quantity > 0,
  stockLabel: p.stock_quantity > 10 ? 'In stock' : p.stock_quantity > 0 ? 'Low stock' : 'Out of stock',
  createdAt: new Date(p.created_at),
}));

type Product = z.output<typeof ProductSchema>;

// 3. Query function: validate envelope, transform each item
async function fetchProducts(): Promise<Product[]> {
  const res = await fetch('/api/products');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const json = await res.json();
  const envelope = ApiResponseSchema.parse(json);
  return envelope.data.map((p) => ProductSchema.parse(p));
}

// 4. Component with memoised select for a derived view
function ProductList() {
  const { data: products, isPending, isError } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
  });

  const { data: lowStockNames } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
    select: (items) => items.filter((p) => p.stockLabel === 'Low stock').map((p) => p.name),
  });

  if (isPending) return <p>Loading products…</p>;
  if (isError) return <p role="alert">Failed to load products.</p>;

  return (
    <div>
      <h2>Products</h2>
      <ul>
        {products?.map((p) => (
          <li key={p.id}>
            {p.name} — ${p.price} — {p.stockLabel}
          </li>
        ))}
      </ul>
      <h3>Low stock: {lowStockNames?.join(', ') || 'None'}</h3>
    </div>
  );
}
```

**Expected Output:** The component displays a loading state, then a list of products with camelCase names, formatted prices, and stock labels ("In stock", "Low stock", "Out of stock"). A second view shows only the names of low-stock products, derived via `select` and memoised.

**Why This Output Occurs:** `ApiResponseSchema.parse(json)` validates the raw envelope and throws if the shape is unexpected. `ProductSchema.parse(p)` validates and transforms each product via `.transform()`. The `select` option derives `lowStockNames` from the same query data without an additional network request, and the result is memoised until the underlying data changes. The component receives clean, typed data.

### Real-World Cases

- **E-commerce:** Transforming product API responses (snake_case, nested variants) into flat, camelCase UI models with computed stock labels.
- **Dashboard analytics:** Unwrapping `{ data, meta }` envelopes and deriving display metrics.
- **User profiles:** Normalising inconsistent user shapes across multiple endpoints into a single `User` type.
- **Internationalisation:** Transforming date and currency strings into locale-aware display formats.
- **Legacy APIs:** Wrapping legacy endpoints with Zod schemas that validate and transform into the modern domain model.

### References

- Zod – Documentation: https://zod.dev/
- Zod – `.transform()`: https://zod.dev/?id=transform
- TanStack Query – `select`: https://tanstack.com/query/latest/docs/framework/react/guides/render-optimizations#select
- FreeCodeCamp – How to Use Zod for React API Validation: https://www.freecodecamp.org/news/how-to-use-zod-for-react-api-validation/
- Steve Kinney – Data Fetching and Runtime Validation: https://stevekinney.com/courses/react-typescript/data-fetching-and-runtime-validation
- @sgvolpe/api-response-normalizer – npm: https://www.npmjs.com/package/@sgvolpe/api-response-normalizer

---

## Core Concept 5: Error Normalisation — Mapping API Error Arrays to Unified Client-Side Formats

### Definitions

**Core Definition:** Error normalisation is the practice of converting heterogeneous API error shapes—Axios `AxiosError`, `FetchBaseQueryError`, GraphQL `errors` arrays, validation error objects—into a single, typed, client-side error contract before they reach state, UI, forms, or telemetry.

**Technical Definition:** A long-lived React application rarely has only one error shape. Axios returns `AxiosError`; RTK Query returns `FetchBaseQueryError`; GraphQL returns an `errors` array with `message`, `locations`, and `extensions`; validation endpoints return `{ errors: { field: [messages] } }`. If raw error objects cross layer boundaries, backend messages, stacks, and request details leak into Redux state, UI copy, and telemetry. Error normalisation introduces a **boundary**: unknown errors go in; a typed taxonomy (`AppError`), a message key, field-level issues, a retry policy, and exactly one presentation route come out. The normaliser inspects the error's provider dialect (Axios, RTK Query, GraphQL, standard `Error`), extracts status, code, message, and field errors, and maps them to a canonical `AppError` with properties: `status`, `code`, `message`, `fieldErrors`, `retryable`, and `traceId`. TanStack Query's global `QueryCache` callbacks can apply the normaliser to all query and mutation errors, and `setError` maps field errors to React Hook Form fields.

**Beginner-Friendly Explanation:** Every API returns errors in a different format. Axios has `error.response.data.message`; GraphQL has `errors[0].message`; a validation endpoint has `{ errors: { email: ['Invalid'] } }`. If you write UI code that tries to read all these shapes, it becomes a mess. Error normalisation is like a translator: no matter what language the server speaks, your UI hears a single, consistent message. You write the translator once, and every component can handle errors the same way.

### Purposes

- To provide a single, typed error contract (`AppError`) that all layers can depend on.
- To prevent raw error objects (with stacks, request configs, and backend messages) from leaking into Redux state, UI, or telemetry.
- To map HTTP status codes to user-facing messages (401 → "Please log in", 404 → "Not found", 5xx → "Something went wrong").
- To extract field-level validation errors (422) and map them to form fields via `setError`.
- To determine retryability (5xx, 429, 408 are retryable; 4xx are not).
- To attach a `traceId` for support and debugging without exposing internal details.
- To centralise error handling in a single normaliser function, applied globally to all queries and mutations.

### Syntax Rules and Structure

**Canonical Error Contract:**
```typescript
export type ErrorCode =
  | 'VALIDATION_ERROR'
  | 'UNAUTHORIZED'
  | 'FORBIDDEN'
  | 'NOT_FOUND'
  | 'CONFLICT'
  | 'RATE_LIMITED'
  | 'SERVER_ERROR'
  | 'NETWORK_ERROR'
  | 'UNKNOWN';

export interface AppError {
  code: ErrorCode;
  status: number | null;
  message: string;           // User-facing message (safe to display)
  fieldErrors?: Record<string, string[]>; // For 422 validation errors
  retryable: boolean;
  traceId?: string;
  cause?: unknown;           // Original error (for telemetry, not UI)
}
```

**Normaliser Function:**
```typescript
import axios, { AxiosError } from 'axios';

export function normalizeError(error: unknown): AppError {
  // 1. Axios errors
  if (axios.isAxiosError(error)) {
    const status = error.response?.status ?? null;
    const body = error.response?.data as any;
    const traceId = error.response?.headers?.['x-trace-id'];

    if (!error.response) {
      return {
        code: 'NETWORK_ERROR',
        status: null,
        message: 'Network error. Please check your connection.',
        retryable: true,
        cause: error,
      };
    }

    return {
      code: mapStatusToCode(status),
      status,
      message: body?.message ?? statusMessage(status),
      fieldErrors: body?.errors,
      retryable: status !== null && (status >= 500 || status === 429 || status === 408),
      traceId,
      cause: error,
    };
  }

  // 2. Standard Error
  if (error instanceof Error) {
    return {
      code: 'UNKNOWN',
      status: null,
      message: 'An unexpected error occurred.',
      retryable: false,
      cause: error,
    };
  }

  // 3. Unknown throw
  return {
    code: 'UNKNOWN',
    status: null,
    message: 'An unexpected error occurred.',
    retryable: false,
    cause: error,
  };
}

function mapStatusToCode(status: number | null): ErrorCode {
  switch (status) {
    case 400: return 'VALIDATION_ERROR';
    case 401: return 'UNAUTHORIZED';
    case 403: return 'FORBIDDEN';
    case 404: return 'NOT_FOUND';
    case 409: return 'CONFLICT';
    case 422: return 'VALIDATION_ERROR';
    case 429: return 'RATE_LIMITED';
    default:  return status && status >= 500 ? 'SERVER_ERROR' : 'UNKNOWN';
  }
}

function statusMessage(status: number | null): string {
  switch (status) {
    case 400: return 'The request was invalid.';
    case 401: return 'Please log in to continue.';
    case 403: return 'You do not have permission to do that.';
    case 404: return 'The requested resource was not found.';
    case 409: return 'This action conflicts with the current state.';
    case 422: return 'Please correct the highlighted fields.';
    case 429: return 'Too many requests. Please try again later.';
    default:  return status && status >= 500
      ? 'Something went wrong on our end. Please try again.'
      : 'An unexpected error occurred.';
  }
}
```

**Global Application with TanStack Query:**
```typescript
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      const appError = normalizeError(error);
      // Global toast for non-401 errors
      if (appError.code !== 'UNAUTHORIZED') {
        toast.error(appError.message);
      }
      // Telemetry receives only the taxonomy, never the raw error
      logToTelemetry({ code: appError.code, status: appError.status, traceId: appError.traceId });
    },
  }),
  mutationCache: new MutationCache({
    onError: (error) => {
      const appError = normalizeError(error);
      if (appError.fieldErrors) {
        // Handled by the form that owns the mutation
        return;
      }
      toast.error(appError.message);
    },
  }),
  defaultOptions: {
    queries: {
      retry: (count, error) => {
        const appError = normalizeError(error);
        return appError.retryable && count < 3;
      },
    },
  },
});
```

**Component Breakdown:**
- `normalizeError`: Converts any error shape into `AppError`.
- `QueryCache.onError`: Applies the normaliser globally to all query errors.
- `MutationCache.onError`: Applies it to mutations, respecting field errors for forms.
- `retry`: Uses `appError.retryable` to decide whether to retry.

**Mapping Field Errors to React Hook Form:**
```tsx
import { useForm } from 'react-hook-form';
import { useMutation } from '@tanstack/react-query';
import { normalizeError } from './errors';

function RegistrationForm() {
  const { register, handleSubmit, setError } = useForm();

  const mutation = useMutation({
    mutationFn: registerUser,
    onError: (error) => {
      const appError = normalizeError(error);
      if (appError.fieldErrors) {
        Object.entries(appError.fieldErrors).forEach(([field, messages]) => {
          setError(field, { type: 'server', message: messages[0] });
        });
      }
    },
  });

  return (
    <form onSubmit={handleSubmit((data) => mutation.mutate(data))}>
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      {/* … */}
    </form>
  );
}
```

**Component Breakdown:**
- `onError`: Normalises the error and, if `fieldErrors` exist, maps them to form fields via `setError`.
- `setError(field, { type: 'server', message })`: Sets the error on the matching field.
- The form displays field-level errors from the server, exactly like client-side validation errors.

**Syntax Rules:**
- Define a canonical `AppError` interface with `code`, `status`, `message`, `fieldErrors`, `retryable`, and `traceId`.
- Write one `normalizeError` function that handles all provider dialects (Axios, `fetch`, GraphQL, standard `Error`, unknown).
- Never expose `cause` or raw error objects to the UI or Redux state; use `cause` only for telemetry.
- Map status codes to user-facing messages in a single `statusMessage` function.
- Apply the normaliser globally via TanStack Query's `QueryCache` and `MutationCache` `onError` callbacks.
- Use `appError.retryable` in the `retry` function to retry only transient errors.
- Map `fieldErrors` to form fields via `setError` in mutation `onError` handlers.
- Attach a `traceId` from a response header (e.g., `X-Trace-Id`) for support; never generate one client-side if the server already provides it.
- Log only the taxonomy (code, status, traceId) to telemetry, not the raw error object.

**Constraints and Limitations:**
- Different APIs return different error shapes; the normaliser must handle each dialect it encounters.
- GraphQL returns 200 with an `errors` array; the normaliser must check the response body, not just the status code.
- Field error formats vary: some APIs use `{ errors: { email: ['msg'] } }`, others use `{ errors: [{ field: 'email', message: 'msg' }] }`.
- TanStack Query's global `onError` receives the error but cannot access the form's `setError`; field errors must be handled in the mutation's `onError`.
- `normalizeError` runs on every error; keep it fast and free of side effects.
- Retrying 401 errors is handled by the Axios interceptor (token refresh), not by TanStack Query's `retry`; ensure they do not conflict.
- `cause` may contain circular references; do not serialise it to JSON without a safe replacer.

### Annotated Code Example: End-to-End Error Normalisation

```typescript
// errors.ts — canonical contract and normaliser
export type ErrorCode =
  | 'VALIDATION_ERROR' | 'UNAUTHORIZED' | 'FORBIDDEN' | 'NOT_FOUND'
  | 'CONFLICT' | 'RATE_LIMITED' | 'SERVER_ERROR' | 'NETWORK_ERROR' | 'UNKNOWN';

export interface AppError {
  code: ErrorCode;
  status: number | null;
  message: string;
  fieldErrors?: Record<string, string[]>;
  retryable: boolean;
  traceId?: string;
  cause?: unknown;
}

export function normalizeError(error: unknown): AppError { /* … as above … */ }
```

```tsx
// queryClient.ts — global error handling
import { QueryClient, QueryCache, MutationCache } from '@tanstack/react-query';
import { normalizeError } from './errors';

export const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) => {
      const appError = normalizeError(error);
      if (appError.code === 'UNAUTHORIZED') return; // Interceptor handles redirect
      toast.error(appError.message);
      telemetry.log({ code: appError.code, status: appError.status, traceId: appError.traceId });
    },
  }),
  mutationCache: new MutationCache({
    onError: (error) => {
      const appError = normalizeError(error);
      if (appError.fieldErrors) return; // Form handles field errors
      toast.error(appError.message);
    },
  }),
});
```

```tsx
// ProfileForm.tsx — field error mapping
import { useForm } from 'react-hook-form';
import { useMutation } from '@tanstack/react-query';
import { normalizeError } from './errors';

function ProfileForm() {
  const { register, handleSubmit, setError, formState: { errors } } = useForm();

  const mutation = useMutation({
    mutationFn: (data: ProfileInput) =>
      fetch('/api/profile', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      }).then((res) => {
        if (!res.ok) throw res; // Will be normalised
        return res.json();
      }),
    onError: (error) => {
      const appError = normalizeError(error);
      if (appError.fieldErrors) {
        Object.entries(appError.fieldErrors).forEach(([field, messages]) => {
          setError(field, { type: 'server', message: messages[0] });
        });
      }
      // Non-field errors are handled by the global MutationCache onError
    },
  });

  return (
    <form onSubmit={handleSubmit((data) => mutation.mutate(data))}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" {...register('email')} aria-invalid={!!errors.email} />
        {errors.email && <span role="alert">{errors.email.message}</span>}
      </div>
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Saving…' : 'Save'}
      </button>
    </form>
  );
}
```

**Expected Output:** Submitting the form with an invalid email returns a 422 with `{ errors: { email: ['Email already in use'] } }`. The normaliser converts this to `AppError` with `fieldErrors.email = ['Email already in use']`. The mutation's `onError` maps it to `setError('email', ...)`, and the form displays "Email already in use" below the email field. Non-field errors (e.g., a 500) trigger a global toast via `MutationCache`.

**Why This Output Occurs:** The normaliser extracts `fieldErrors` from the API's validation error format and returns a typed `AppError`. The mutation's `onError` checks for `fieldErrors` and maps them to form fields. The global `MutationCache.onError` handles all other errors (toasts), but skips field errors so they are not double-reported. This cleanly separates field-level and form-level error presentation.

### Real-World Cases

- **Form-heavy applications:** 422 validation errors mapped to React Hook Form fields via `setError`.
- **Multi-provider monorepos:** Axios, RTK Query, and GraphQL errors normalised into a single `AppError` taxonomy.
- **Telemetry and monitoring:** Only the taxonomy (code, status, traceId) is sent to Sentry/Datadog; raw errors stay out of logs.
- **Global error toasts:** A single `QueryCache.onError` shows a toast for all non-field errors.
- **Retry logic:** `appError.retryable` drives TanStack Query's `retry` function, avoiding retries on 4xx errors.
- **Support workflows:** `traceId` is displayed in error messages or copied to clipboard for support tickets.

### References

- @shubhamgupta-oss/universal-error-handler – npm: https://www.npmjs.com/package/@shubhamgupta-oss/universal-error-handler
- Typed Errors from API to UI: https://wong-coupon.gitbook.io/flutter/my-reactjs/state-api/typed-errors-api-to-ui
- TanStack Query – Error Handling: https://tanstack.com/query/latest/docs/framework/react/guides/query-functions#handling-and-throwing-errors
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- RFC 9457 – Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457.html
- Axios – Handling Errors: https://axios-http.com/docs/handling_errors

---

## Comparison and Decision Guidance

| Concern | Tool/Pattern | When to Use | Key Risk |
|---|---|---|---|
| **REST consumption** | `fetch`/Axios + TanStack Query | Simple resources, public APIs, caching | Over-fetching, multiple round trips |
| **GraphQL consumption** | Apollo Client / urql / TanStack Query | Complex data graphs, multiple clients | N+1 queries, cache complexity |
| **Codegen** | Hey API (REST), graphql-codegen (GraphQL) | Type-safe API clients | Stale hook generators (avoid) |
| **Token storage** | Access token in memory; refresh in `HttpOnly` cookie | All authenticated apps | `localStorage` for access tokens |
| **Token refresh** | Axios response interceptor + queue | Silent session persistence | Thundering herd without queue |
| **Response transformation** | Zod `.transform()` + TanStack Query `select` | Reshaping API data for UI | Transformation in interceptors (global) |
| **Error normalisation** | `normalizeError` + `QueryCache`/`MutationCache` | Multi-provider apps, telemetry | Raw errors leaking to UI/state |

**Decision Guidance:**
- **Choose REST** for public APIs, simple resources, and teams that value simplicity; **choose GraphQL** when multiple clients need different data shapes from a complex graph.
- **Use Hey API + TanStack Query** for REST codegen; **use graphql-codegen client preset** or **gql.tada** for GraphQL.
- **Store access tokens in memory** (never `localStorage`); **store refresh tokens in `HttpOnly` cookies** with `Secure` and `SameSite=Strict`.
- **Implement a 401 refresh queue** (or use `axios-auth-refresh-queue`) to avoid multiple simultaneous refresh requests.
- **Transform responses in the query function or `select`**, not in global interceptors, to keep the API client generic.
- **Use Zod `.transform()`** for validation + transformation in one step; use `z.infer` for the transformed type.
- **Normalise all errors** into a single `AppError` contract before they reach UI, forms, or telemetry.
- **Apply error normalisation globally** via TanStack Query's `QueryCache` and `MutationCache`; handle field errors locally in mutation `onError`.
- **Never retry 4xx errors** (except 408 and 429); use `appError.retryable` to drive the `retry` function.
- **Attach a `traceId`** from the server and include it in error messages for support.

---

## References

- RFC 6750 – The OAuth 2.0 Authorization Framework: Bearer Token Usage: https://www.rfc-editor.org/rfc/rfc6750.html
- RFC 9700 – Best Current Practice for OAuth 2.0 Security: https://www.rfc-editor.org/rfc/rfc9700.html
- RFC 9457 – Problem Details for HTTP APIs: https://www.rfc-editor.org/rfc/rfc9457.html
- React Official Documentation – Server Components: https://react.dev/reference/rsc/server-components
- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- TanStack Query – Render Optimizations (`select`): https://tanstack.com/query/latest/docs/framework/react/guides/render-optimizations#select
- Apollo Client – React Documentation: https://www.apollographql.com/docs/react
- GraphQL Codegen – Client Preset: https://the-guild.dev/graphql/codegen/plugins/presets/preset-client
- Hey API – OpenAPI to TypeScript: https://heyapi.dev/
- gql.tada – Zero-codegen GraphQL: https://gql-tada.0no.co/
- Axios – Interceptors: https://axios-http.com/docs/interceptors
- Axios – Handling Errors: https://axios-http.com/docs/handling_errors
- axios-auth-refresh-queue – npm: https://www.npmjs.com/package/axios-auth-refresh-queue
- Zod – Documentation: https://zod.dev/
- Zod – `.transform()`: https://zod.dev/?id=transform
- FreeCodeCamp – How to Use Zod for React API Validation: https://www.freecodecamp.org/news/how-to-use-zod-for-react-api-validation/
- Steve Kinney – Data Fetching and Runtime Validation: https://stevekinney.com/courses/react-typescript/data-fetching-and-runtime-validation
- @shubhamgupta-oss/universal-error-handler – npm: https://www.npmjs.com/package/@shubhamgupta-oss/universal-error-handler
- Typed Errors from API to UI: https://wong-coupon.gitbook.io/flutter/my-reactjs/state-api/typed-errors-api-to-ui
- @sgvolpe/api-response-normalizer – npm: https://www.npmjs.com/package/@sgvolpe/api-response-normalizer
- GitHub – elaad24/react-auth-best-practice: https://github.com/elaad24/react-auth-best-practice-
- tRPC vs REST vs GraphQL: Type-Safe APIs in Next.js 2026: https://www.pkgpulse.com/guides/trpc-vs-rest-vs-graphql-typesafe-apis-nextjs-2026
- MDN Web Docs – Web Locks API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API
- MDN Web Docs – Set-Cookie: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
- MDN Web Docs – SameSite cookies: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite