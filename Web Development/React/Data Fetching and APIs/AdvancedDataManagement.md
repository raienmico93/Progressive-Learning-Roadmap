# React Advanced Data Management & State Synchronizers: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React advanced data management and state synchronizers are the libraries, patterns, and lifecycle configurations that keep a React application's view of server data synchronised with the server itself—handling caching, deduplication, pagination, prefetching, mutations, and real-time updates without manual state management.

**Technical Definition:** Advanced server-state management treats remote data as a first-class, cached resource with its own lifecycle: staleness windows, garbage collection, background revalidation, and mutation-driven invalidation. TanStack Query (formerly React Query) and SWR are the dominant declarative caching engines. They provide a `QueryClient`/cache store keyed by deterministic query keys, deduplicate concurrent identical requests, persist cache entries according to `staleTime` and `gcTime` (TanStack Query) or `dedupingInterval` and `refreshInterval` (SWR), and expose hooks (`useQuery`, `useMutation`, `useInfiniteQuery`, `usePrefetchQuery`) that integrate with React's render cycle. Advanced patterns include offset/limit and cursor-based pagination, infinite scrolling via `useInfiniteQuery`, prefetching and hydration for SSR/SSG, optimistic updates with rollback, and real-time synchronisation via polling, WebSockets, or Server-Sent Events with cache write-through.

**Beginner-Friendly Explanation:** Your React app needs data from a server, but fetching it on every render is wasteful, slow, and buggy. Advanced data management libraries act like a smart assistant: they remember what they've already fetched (caching), don't fetch the same thing twice at the same time (deduplication), quietly check for updates in the background (revalidation), and know when to throw away old data (garbage collection). They also handle tricky cases: loading pages of data, showing old data while new data loads, updating the UI before the server confirms (optimistic updates), and keeping everything in sync when the server pushes changes (WebSockets, polling).

### Key Characteristics

- **Server State ≠ Client State:** Server state is asynchronous, remotely owned, potentially stale, and shared; it must not be stored in Redux or Context as if it were client state.
- **Query Keys as Cache Identity:** Every cache entry is identified by a deterministic, serialisable query key (e.g., `['todos', { status: 'done', page: 1 }]`).
- **Stale-While-Revalidate:** Cached data is served immediately, and a background revalidation updates the cache when stale.
- **Deduplication by Default:** Concurrent identical requests share a single in-flight promise.
- **Lifecycle Separation:** `staleTime` controls freshness; `gcTime` controls memory retention.
- **Mutation-Driven Invalidation:** Mutations trigger targeted invalidation of related queries via prefix matching or predicates.
- **Pagination as a First-Class API:** `useInfiniteQuery`, `placeholderData: keepPreviousData`, and cursor-based keys are built-in.
- **Optimistic Updates with Rollback:** `onMutate` snapshots the cache, updates it optimistically, and `onError` rolls back.
- **Real-Time Integration:** Polling, WebSockets, and SSE write through to the same cache used by queries.

### Prerequisites

- Solid understanding of React function components, Hooks, and the render cycle.
- Working knowledge of HTTP methods, status codes, headers, and CORS.
- Familiarity with `fetch`/Axios, promises, and `async`/`await`.
- Basic understanding of caching concepts (staleness, invalidation, garbage collection).
- Awareness of React Suspense and Error Boundaries (for advanced patterns).

### Related Programming Areas

- **HTTP Fundamentals:** Methods, status codes, headers, caching directives.
- **Authentication:** Token lifecycle, interceptors, 401 handling.
- **Performance:** Prefetching, hydration, deduplication, code splitting.
- **Real-Time Systems:** WebSockets, SSE, polling, background sync.
- **Observability:** Query devtools, error tracking, and performance monitoring.

### Core Concepts / Features

1. Declarative Server-State Caching Engines (TanStack Query vs. SWR)
2. Caching Lifecycle Configuration (staleTime, gcTime, Automatic Invalidation)
3. Request Deduplication and Network Bottleneck Elimination
4. Data Stream Pagination Models (Offset/Limit, Cursor, Infinite Scrolling)
5. Performance Tuning: Prefetching, Hydration, and Placeholder Data
6. Mutating Server State: Optimistic Updates, Rollback, and Cache Triggers
7. Real-Time Sync Paradigms: Smart Polling, WebSockets, Background Sync

---

## Core Concept 1: Declarative Server-State Caching Engines (TanStack Query vs. SWR)

### Definitions

**Core Definition:** Declarative server-state caching engines are libraries that manage the fetch-cache-revalidate lifecycle of remote data through hooks, eliminating manual `useEffect` + `useState` fetching.

**Technical Definition:** TanStack Query (formerly React Query) and SWR are the two dominant server-state caching engines. TanStack Query provides a `QueryClient` that owns a `QueryCache` and `MutationCache`, keyed by deterministic query keys. It exposes `useQuery`, `useMutation`, `useInfiniteQuery`, `useQueries`, `useSuspenseQuery`, and a rich `QueryClient` API for prefetching, hydration, invalidation, and cache manipulation. SWR ("stale-while-revalidate") is a lighter library focused on the stale-while-revalidate pattern with a simpler API (`useSWR(key, fetcher)`) and built-in features like `mutate`, `useSWRInfinite`, and optimistic updates. Both libraries deduplicate requests, cache responses, revalidate on focus/reconnect, and integrate with React's render cycle.

**Beginner-Friendly Explanation:** These libraries are the "memory" of your data-fetching layer. Instead of writing the same fetch-load-cache-revalidate code in every component, you declare what data you need (the query key) and how to fetch it (the query function), and the library handles the rest. TanStack Query is the full-featured option; SWR is the minimalist option.

### Purposes

- To declare data requirements declaratively instead of imperatively.
- To cache server responses and serve them instantly on remount or navigation.
- To revalidate stale data in the background without blocking the UI.
- To deduplicate concurrent identical requests.
- To integrate mutations with cache invalidation automatically.
- To provide devtools for inspecting cache state.

### Syntax Rules and Structure

**TanStack Query (Setup):**
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,   // 1 minute
      gcTime: 5 * 60_000,  // 5 minutes
      retry: 2,
      refetchOnWindowFocus: true,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyApp />
    </QueryClientProvider>
  );
}
```

**TanStack Query (`useQuery`):**
```tsx
import { useQuery } from '@tanstack/react-query';

function Todos() {
  const { data, isPending, isError, error, isFetching } = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(r => r.json()),
  });

  if (isPending) return <p>Loading…</p>;
  if (isError) return <p>Error: {error.message}</p>;

  return (
    <>
      {isFetching && <span>Updating…</span>}
      <ul>{data.map(t => <li key={t.id}>{t.title}</li>)}</ul>
    </>
  );
}
```

**SWR (`useSWR`):**
```tsx
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

function Todos() {
  const { data, error, isLoading, isValidating, mutate } = useSWR('/api/todos', fetcher, {
    revalidateOnFocus: true,
    revalidateOnReconnect: true,
    dedupingInterval: 2000,
  });

  if (isLoading) return <p>Loading…</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <>
      {isValidating && <span>Updating…</span>}
      <ul>{data.map(t => <li key={t.id}>{t.title}</li>)}</ul>
    </>
  );
}
```

**Comparison Table:**

| Feature | TanStack Query | SWR |
|---|---|---|
| **API surface** | Rich (`useQuery`, `useMutation`, `useInfiniteQuery`, `useQueries`) | Minimal (`useSWR`, `useSWRInfinite`, `mutate`) |
| **Cache control** | `QueryClient` with `staleTime`, `gcTime`, `invalidateQueries` | `dedupingInterval`, `refreshInterval`, `mutate` |
| **Mutations** | First-class `useMutation` with `onMutate`, `onError`, `onSettled` | `mutate` function (optimistic via `mutate(data, { optimisticData })`) |
| **Devtools** | Official, feature-rich | Community devtools |
| **Suspense** | `useSuspenseQuery`, `useSuspenseInfiniteQuery` | Via `suspense: true` option |
| **Infinite queries** | `useInfiniteQuery` with `getNextPageParam` | `useSWRInfinite` with `getKey` |
| **Bundle size** | ~12–15 KB gzipped | ~5 KB gzipped |
| **Best for** | Complex apps with mutations, pagination, and devtools needs | Simple apps with read-heavy data and minimal mutation logic |

**Syntax Rules:**
- Wrap the app in `<QueryClientProvider>` (TanStack Query); SWR requires no provider.
- Always use deterministic query keys that include all variables affecting the response.
- Define `queryFn` as a stable function (outside the component or memoised).
- Set `staleTime` deliberately; the default is `0` (data is instantly stale).
- Use `gcTime` >= `staleTime` to avoid garbage-collecting fresh data.
- Use `isPending` (no data yet) vs. `isFetching` (background refetch) to differentiate UI states.
- Use the `mutate` function (SWR) or `useMutation` + `invalidateQueries` (TanStack Query) for mutations.

**Constraints and Limitations:**
- Neither library replaces client-state management (Redux, Zustand); they manage *server* state only.
- TanStack Query's bundle size is larger than SWR's.
- SWR's mutation story is simpler but less powerful than TanStack Query's `useMutation` lifecycle.
- Both require careful `staleTime`/`dedupingInterval` tuning to avoid over-fetching.
- Query keys must be JSON-serialisable; functions, `Symbol`s, and circular references break hashing.

### Annotated Code Example: Same Feature in Both Libraries

```tsx
// --- TanStack Query ---
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function TodoAppTQ() {
  const queryClient = useQueryClient();

  const { data: todos, isPending } = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(r => r.json()),
  });

  const addMutation = useMutation({
    mutationFn: (title: string) =>
      fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title }),
      }).then(r => r.json()),
    onSettled: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
  });

  if (isPending) return <p>Loading…</p>;

  return (
    <div>
      <ul>{todos.map(t => <li key={t.id}>{t.title}</li>)}</ul>
      <button onClick={() => addMutation.mutate('New Todo')}>Add</button>
    </div>
  );
}
```

```tsx
// --- SWR ---
import useSWR, { useSWRConfig } from 'swr';

function TodoAppSWR() {
  const { data: todos, isLoading } = useSWR('/api/todos', url => fetch(url).then(r => r.json()));
  const { mutate } = useSWRConfig();

  async function addTodo(title: string) {
    await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title }),
    });
    mutate('/api/todos');
  }

  if (isLoading) return <p>Loading…</p>;

  return (
    <div>
      <ul>{todos.map(t => <li key={t.id}>{t.title}</li>)}</ul>
      <button onClick={() => addTodo('New Todo')}>Add</button>
    </div>
  );
}
```

**Expected Output:** Both components render a list of todos and an "Add" button. Clicking "Add" creates a todo and refetches the list.

**Why This Output Occurs:** Both libraries cache the `todos` query, serve it from cache on remount, and revalidate after the mutation. TanStack Query uses `useMutation` with `onSettled` + `invalidateQueries`; SWR uses the global `mutate` function to revalidate the key.

### Real-World Cases

- **Dashboards:** Multiple independent widgets fetched with `useQueries` (TanStack Query) or `useSWR` with separate keys.
- **E-commerce:** Product listings, cart state, and order history cached with different `staleTime` values.
- **Social apps:** Feed, notifications, and profile data cached and revalidated on focus.
- **Admin panels:** Data tables with pagination and mutations, requiring the full TanStack Query API.

### References

- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query – Quick Start: https://tanstack.com/query/latest/docs/framework/react/quick-start
- SWR – Documentation: https://swr.vercel.app/
- SWR – Getting Started: https://swr.vercel.app/docs/getting-started

---

## Core Concept 2: Caching Lifecycle Configuration (staleTime, gcTime, Automatic Invalidation)

### Definitions

**Core Definition:** Caching lifecycle configuration is the set of timing parameters—`staleTime`, `gcTime` (TanStack Query), `dedupingInterval`, `refreshInterval` (SWR)—that control when cached data is considered fresh, when it is revalidated, and when it is removed from memory.

**Technical Definition:** In TanStack Query, two fundamental timing parameters govern the cache. **`staleTime`** (default `0`) determines how long data is considered fresh; while fresh, queries are served from cache without refetching on mount, focus, or reconnect. Setting `staleTime: Infinity` means data is never automatically considered stale. **`gcTime`** (formerly `cacheTime`, default `5 * 60_000` ms) determines how long inactive query data remains in memory after all observers unmount; when this time elapses, the data is garbage collected. `gcTime` should always be greater than or equal to `staleTime`. **Automatic invalidation** occurs via `queryClient.invalidateQueries({ queryKey })`, which marks queries as stale and refetches active ones in the background. Prefix matching is the default; `exact: true` limits to a specific key, and a `predicate` function allows arbitrary matching. In SWR, `dedupingInterval` (default 2000ms) controls how long identical requests are deduplicated, `refreshInterval` controls polling, and `mutate(key)` triggers revalidation.

**Beginner-Friendly Explanation:** Think of cached data as milk in a fridge. `staleTime` is the "best before" date—while the milk is fresh, you use it without checking the store. `gcTime` is how long you keep the milk in the fridge after you stop using it, before throwing it away. "Invalidation" is when you deliberately decide the milk is expired and go buy fresh. Setting `staleTime: 0` means the milk is always considered old, so you check the store every time—aggressive but fresh. Setting `staleTime: Infinity` means you never check the store—fast but potentially stale.

### Purposes

- **`staleTime`:** To control how long data is considered fresh before background revalidation is triggered.
- **`gcTime`:** To control how long inactive query data is retained in memory before garbage collection.
- **Invalidation:** To mark cached data as stale after mutations and trigger refetches.
- **Prefix invalidation:** To invalidate all queries under a namespace (e.g., `['todos']` matches `['todos', 1]`).
- **Predicate invalidation:** To invalidate queries matching a custom condition.
- **`dedupingInterval` (SWR):** To prevent duplicate requests within a time window.

### Syntax Rules and Structure

**TanStack Query — `staleTime` and `gcTime`:**
```tsx
useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 5 * 60 * 1000,   // 5 minutes fresh
  gcTime: 10 * 60 * 1000,     // 10 minutes in memory after unmount
});
```

**Global Defaults:**
```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,        // 1 minute
      gcTime: 5 * 60_000,       // 5 minutes
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
      refetchOnMount: true,
      retry: 3,
    },
  },
});
```

**Invalidation by Prefix:**
```tsx
// Invalidate all queries whose key starts with ['todos']
queryClient.invalidateQueries({ queryKey: ['todos'] });
// Matches: ['todos'], ['todos', 1], ['todos', { page: 2 }]
```

**Invalidation by Exact Key:**
```tsx
queryClient.invalidateQueries({ queryKey: ['todos'], exact: true });
// Matches only ['todos']
```

**Invalidation by Predicate:**
```tsx
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === 'todos' &&
    (query.queryKey[1] as any)?.version >= 2,
});
```

**SWR — `dedupingInterval` and `refreshInterval`:**
```tsx
useSWR('/api/todos', fetcher, {
  dedupingInterval: 2000,    // 2s — dedupe identical requests within this window
  refreshInterval: 30_000,   // 30s — poll every 30s
  revalidateOnFocus: true,
  revalidateOnReconnect: true,
  keepPreviousData: true,
});
```

**Syntax Rules:**
- Set `staleTime` based on how frequently the data changes. Static data: `Infinity`; real-time data: `0`.
- Set `gcTime` >= `staleTime` to prevent fresh data from being garbage collected.
- Use `staleTime: 'static'` (TanStack Query v5) for data that never needs automatic refetching.
- Invalidate by prefix for related queries; use `exact: true` for precision.
- Use `predicate` for complex invalidation logic.
- Always invalidate in `onSettled` (not just `onSuccess`) to handle error cases.
- SWR's `dedupingInterval` defaults to 2000ms; lower it for more real-time behaviour, raise it to reduce requests.
- SWR's `keepPreviousData: true` prevents loading flashes during pagination.

**Constraints and Limitations:**
- `staleTime: 0` (default) causes aggressive refetching; raise it for data that does not change frequently.
- Garbage collection only removes inactive queries; active queries (rendered in components) are never collected.
- Invalidation triggers a refetch only if the query is active; inactive queries are marked stale and refetch on next mount.
- Prefix invalidation can invalidate more queries than intended; use `exact: true` or predicates for precision.
- Persisted caches require `gcTime` to be overridden to a longer duration (e.g., 24 hours).

### Annotated Code Example: Tunable Cache Lifecycle

```tsx
import { useQuery, useQueryClient, useMutation } from '@tanstack/react-query';

// Post list: moderate freshness
function PostList() {
  const { data } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    staleTime: 2 * 60 * 1000,   // Fresh for 2 minutes
    gcTime: 10 * 60 * 1000,      // Kept in memory for 10 minutes
  });

  return <ul>{data?.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}

// User profile: rarely changes
function UserProfile({ userId }) {
  const { data } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 30 * 60 * 1000,   // Fresh for 30 minutes
    gcTime: 60 * 60 * 1000,      // Kept for 1 hour
  });

  return <h1>{data?.name}</h1>;
}

// Create post: invalidates the post list
function CreatePost() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createPost,
    onSettled: () => {
      // Invalidate all post queries (list + details)
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });

  return <button onClick={() => mutation.mutate({ title: 'New' })}>Create</button>;
}
```

**Expected Output:** The post list serves cached data instantly on remount within 2 minutes; after 2 minutes, it refetches in the background. The user profile is cached for 30 minutes. Creating a post invalidates all `posts` queries (prefix match), causing the list to refetch and display the new post.

**Why This Output Occurs:** `staleTime` controls freshness; `gcTime` controls retention. Different queries have different staleness profiles—posts change more frequently than user profiles. `invalidateQueries({ queryKey: ['posts'] })` uses prefix matching to invalidate `['posts']` and any sub-key.

### Real-World Cases

- **E-commerce product pages:** `staleTime: 5 * 60 * 1000` for product listings; `staleTime: 30 * 60 * 1000` for categories.
- **User dashboards:** `staleTime: 0` for real-time metrics; `staleTime: 10 * 60 * 1000` for user settings.
- **News feeds:** `staleTime: 60_000` with `refetchOnWindowFocus: true` to keep content fresh.
- **Static content:** `staleTime: Infinity` for documentation and legal pages.

### References

- TanStack Query – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query – Query Invalidation: https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query – Query Filters: https://tanstack.com/query/latest/docs/framework/react/guides/filters
- SWR – Options: https://swr.vercel.app/docs/options

---

## Core Concept 3: Request Deduplication and Network Bottleneck Elimination

### Definitions

**Core Definition:** Request deduplication is the practice of ensuring that identical concurrent requests share a single in-flight promise, so multiple components requesting the same data trigger only one network call.

**Technical Definition:** TanStack Query's `QueryCache` maintains a record of every query key. When multiple components call `useQuery` with the same key while a request is in flight, they all subscribe to the same underlying promise; only one network request is made. This is automatic and requires no configuration. TanStack Query v5 further improves this with optimised sharing between components and queries, eliminating redundant network calls and improving performance. SWR deduplicates identical requests within the `dedupingInterval` window (default 2000ms). Beyond deduplication, network bottleneck elimination includes: avoiding request waterfalls (using `Promise.all` or parallel queries), prefetching critical data, using `useQueries` for parallel independent queries, and leveraging HTTP/2 multiplexing (which allows many requests over one connection). The `fetch` API does not deduplicate by default; Axios also does not deduplicate. Libraries like `dedupe-promise` and TanStack Query's cache handle this.

**Beginner-Friendly Explanation:** Imagine five components on the same page all need the current user's profile. Without deduplication, five identical requests hit the server—wasteful and slow. With deduplication, the first request goes out, and the other four components "join" it—they all receive the same response when it arrives. One request, five subscribers. This is automatic in TanStack Query and SWR.

### Purposes

- To eliminate duplicate network requests when multiple components request the same data.
- To reduce server load and bandwidth consumption.
- To improve perceived performance by avoiding redundant loading states.
- To prevent request waterfalls by parallelising independent queries.
- To consolidate loading states across components that share data.

### Syntax Rules and Structure

**Automatic Deduplication (TanStack Query):**
```tsx
// Three components, same query key — only ONE request is made
function ComponentA() {
  const { data } = useQuery({ queryKey: ['user', 1], queryFn: () => fetchUser(1) });
  return <p>{data?.name}</p>;
}

function ComponentB() {
  const { data } = useQuery({ queryKey: ['user', 1], queryFn: () => fetchUser(1) });
  return <p>{data?.email}</p>;
}

function ComponentC() {
  const { data } = useQuery({ queryKey: ['user', 1], queryFn: () => fetchUser(1) });
  return <p>{data?.phone}</p>;
}

// All three share the same cache entry and in-flight promise.
```

**Parallel Queries with `useQueries`:**
```tsx
import { useQueries } from '@tanstack/react-query';

function Dashboard({ userIds }: { userIds: number[] }) {
  const results = useQueries({
    queries: userIds.map(id => ({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),
    })),
  });

  return (
    <ul>
      {results.map((r, i) => (
        <li key={i}>{r.isPending ? 'Loading…' : r.data?.name}</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `useQueries`: Runs multiple queries in parallel (not sequentially).
- Each query has its own cache entry and state.
- No waterfall: all requests start simultaneously.

**SWR Deduplication:**
```tsx
// Within the dedupingInterval (default 2s), identical requests are deduped
useSWR('/api/user/1', fetcher, { dedupingInterval: 2000 });
```

**Manual Deduplication with `dedupe-promise`:**
```tsx
import dedupe from 'dedupe-promise';

const fetchUserDeduped = dedupe(fetchUser);

// Multiple calls in the same tick share one promise
const [a, b, c] = await Promise.all([
  fetchUserDeduped(1),
  fetchUserDeduped(1),
  fetchUserDeduped(1),
]);
```

**Syntax Rules:**
- Use a single query key for shared data; all components with that key subscribe to the same cache entry.
- Use `useQueries` for parallel independent queries, not a loop with `await`.
- Avoid `queryKey` values that change on every render (e.g., objects created inline); use stable references.
- For dependent queries, use `enabled: !!dependency` to gate execution.
- Prefetch critical data on route transition to avoid waterfalls.
- Deduplication applies to *concurrent* requests; sequential requests with different keys are separate.

**Constraints and Limitations:**
- Deduplication does not apply to different query keys, even if they resolve to the same data.
- `fetch` and Axios do not deduplicate by default; only caching engines do.
- Deduplication is per-cache-instance; multiple `QueryClient` instances do not share.
- Prefetching can cause over-fetching if not scoped carefully.
- The `dedupingInterval` (SWR) can delay fresh data if set too high.

### Annotated Code Example: Eliminating a Waterfall

```tsx
// ❌ Waterfall: sequential fetches
function SlowDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetchUser(userId).then(u => {
      setUser(u);
      return fetchPosts(u.id); // Waits for user before fetching posts
    }).then(setPosts);
  }, [userId]);
  // Total time: t(user) + t(posts)
}

// ✅ Parallel: independent fetches with useQueries
import { useQueries } from '@tanstack/react-query';

function FastDashboard({ userId }) {
  const results = useQueries({
    queries: [
      { queryKey: ['user', userId], queryFn: () => fetchUser(userId) },
      { queryKey: ['posts', userId], queryFn: () => fetchPosts(userId) },
      { queryKey: ['notifications', userId], queryFn: () => fetchNotifications(userId) },
    ],
  });

  const [user, posts, notifications] = results;
  // Total time: max(t(user), t(posts), t(notifications))
}

// ✅ Mixed: sequential dependency, parallel after
function MixedDashboard({ email }) {
  const { data: user } = useQuery({
    queryKey: ['user', email],
    queryFn: () => fetchUserByEmail(email),
  });

  const userId = user?.id;

  const results = useQueries({
    queries: [
      { queryKey: ['posts', userId], queryFn: () => fetchPosts(userId), enabled: !!userId },
      { queryKey: ['notifications', userId], queryFn: () => fetchNotifications(userId), enabled: !!userId },
    ],
  });
  // Total time: t(user) + max(t(posts), t(notifications))
}
```

**Expected Output:** `SlowDashboard` takes the sum of all request times. `FastDashboard` takes the maximum. `MixedDashboard` takes one sequential step plus the maximum of the parallel step.

**Why This Output Occurs:** `useQueries` starts all queries in parallel. The `enabled: !!userId` option gates the dependent queries until `userId` is available, then they run in parallel. This eliminates the sequential waterfall.

### Real-World Cases

- **Dashboards:** Multiple widgets fetching independent data with `useQueries`.
- **E-commerce product pages:** Prefetching related products on hover.
- **Social feeds:** Deduplicating profile data requested by multiple feed items.
- **Admin tables:** Parallel loading of table data and filter options.

### References

- TanStack Query – Parallel Queries: https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries
- TanStack Query – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query – v5 Release Notes (Deduplication Improvements): https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- SWR – Deduplication: https://swr.vercel.app/docs/options#dedupinginterval

---

## Core Concept 4: Data Stream Pagination Models (Offset/Limit, Cursor, Infinite Scrolling)

### Definitions

**Core Definition:** Pagination models are strategies for loading large datasets in chunks—offset/limit uses numeric offsets, cursor-based uses opaque pointers, and infinite scrolling appends pages as the user scrolls.

**Technical Definition:** **Offset/limit pagination** requests pages by numeric offset: `GET /items?offset=20&limit=10`. It is simple but suffers from drift when items are inserted or deleted between requests, and degrades in performance for deep pages (large offsets). **Cursor-based pagination** uses an opaque pointer to the last item: `GET /items?cursor=eyJpZCI6MjB9&limit=10`. It is stable under concurrent writes and efficient for deep pagination, but cannot jump to arbitrary pages. **Infinite scrolling** is a UI pattern that appends pages as the user scrolls, typically using an Intersection Observer to detect when the sentinel enters the viewport. TanStack Query's `useInfiniteQuery` models infinite scrolling with `getNextPageParam`, `getPreviousPageParam`, `fetchNextPage`, and `hasNextPage`. The `data.pages` array contains each page's response, and `data.pageParams` contains the parameters used for each page. `placeholderData: keepPreviousData` avoids loading flashes during pagination.

**Beginner-Friendly Explanation:** Pagination is like reading a book in chapters. **Offset/limit** is "give me pages 20–29"—simple, but if someone inserts a page in the middle, your numbers shift. **Cursor-based** is "give me the next 10 pages after this bookmark"—stable even if the book changes, but you can't jump to page 50 directly. **Infinite scrolling** is the "load more as you scroll" pattern, like social media feeds.

### Purposes

- **Offset/limit:** To support page-number navigation and "jump to page N" UI.
- **Cursor-based:** To support stable, efficient deep pagination for feeds and timelines.
- **Infinite scrolling:** To provide a continuous browsing experience without explicit pagination controls.
- **`useInfiniteQuery`:** To manage the pages array, fetch the next page, and track `hasNextPage`.
- **`keepPreviousData`:** To show the previous page's data while the next page loads, avoiding flashes.

### Syntax Rules and Structure

**Offset/Limit with `useQuery` and `placeholderData`:**
```tsx
import { useQuery, keepPreviousData } from '@tanstack/react-query';

function PaginatedList() {
  const [page, setPage] = useState(1);

  const { data, isPlaceholderData } = useQuery({
    queryKey: ['items', page],
    queryFn: () => fetchItems({ offset: (page - 1) * 10, limit: 10 }),
    placeholderData: keepPreviousData, // Show previous page while loading next
  });

  return (
    <div>
      <ul>{data?.items.map(i => <li key={i.id}>{i.name}</li>)}</ul>
      <button onClick={() => setPage(p => Math.max(1, p - 1))} disabled={page === 1}>Prev</button>
      <span>Page {page}</span>
      <button
        onClick={() => setPage(p => p + 1)}
        disabled={isPlaceholderData || !data?.hasMore}
      >
        Next
      </button>
    </div>
  );
}
```

**Cursor-Based with `useInfiniteQuery`:**
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

function InfiniteFeed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isPending,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed({ cursor: pageParam, limit: 10 }),
    initialPageParam: null, // Start with no cursor
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });

  if (isPending) return <p>Loading…</p>;

  return (
    <div>
      {data.pages.map((page, i) => (
        <ul key={i}>
          {page.items.map(item => <li key={item.id}>{item.title}</li>)}
        </ul>
      ))}
      <button onClick={() => fetchNextPage()} disabled={!hasNextPage || isFetchingNextPage}>
        {isFetchingNextPage ? 'Loading more…' : hasNextPage ? 'Load More' : 'No more'}
      </button>
    </div>
  );
}
```

**Component Breakdown:**
- `initialPageParam: null`: The starting cursor.
- `getNextPageParam(lastPage)`: Returns the cursor for the next page, or `undefined` if there is none.
- `fetchNextPage()`: Loads the next page and appends it to `data.pages`.
- `hasNextPage`: `true` if `getNextPageParam` returned a non-undefined value.

**Infinite Scrolling with Intersection Observer:**
```tsx
import { useEffect, useRef } from 'react';

function InfiniteScrollList() {
  const sentinelRef = useRef(null);
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed(pageParam),
    initialPageParam: null,
    getNextPageParam: (last) => last.nextCursor,
  });

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasNextPage && !isFetchingNextPage) {
          fetchNextPage();
        }
      },
      { rootMargin: '200px' } // Load before reaching the bottom
    );

    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div>
      {data?.pages.map((page, i) => (
        <ul key={i}>{page.items.map(item => <li key={item.id}>{item.title}</li>)}</ul>
      ))}
      <div ref={sentinelRef} style={{ height: 1 }} />
      {isFetchingNextPage && <p>Loading more…</p>}
    </div>
  );
}
```

**Component Breakdown:**
- `sentinelRef`: Attached to a div at the bottom of the list.
- `IntersectionObserver`: Fires when the sentinel enters the viewport.
- `rootMargin: '200px'`: Triggers the fetch 200px before the sentinel is visible.
- `fetchNextPage()`: Appends the next page to the cache.

**Syntax Rules:**
- Use `useInfiniteQuery` for cursor-based and infinite scrolling; use `useQuery` + `keepPreviousData` for offset/limit with page numbers.
- Always provide `initialPageParam` (v5 requirement).
- `getNextPageParam` must return `undefined` (or `null`) when there are no more pages.
- Use `placeholderData: keepPreviousData` to avoid loading flashes during page transitions.
- Use Intersection Observer with `rootMargin` for smooth infinite scrolling.
- Cancel the observer on unmount (`observer.disconnect()`).
- Use `maxPages` to limit the number of pages kept in memory for infinite queries.

**Constraints and Limitations:**
- Offset/limit pagination drifts when items are inserted or deleted between requests.
- Cursor-based pagination cannot jump to arbitrary pages.
- Infinite scrolling can cause memory growth; use `maxPages` to cap retained pages.
- Intersection Observer requires a sentinel element and cleanup.
- `keepPreviousData` shows stale data during transitions; communicate this to the user (e.g., via a subtle "updating" indicator).
- Bidirectional infinite queries (`getPreviousPageParam`) require careful handling of scroll position.

### Annotated Code Example: Cursor-Based Infinite Feed

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useRef, useEffect } from 'react';

async function fetchFeed({ cursor, limit = 10 }: { cursor: string | null; limit?: number }) {
  const params = new URLSearchParams({ limit: String(limit) });
  if (cursor) params.set('cursor', cursor);

  const res = await fetch(`/api/feed?${params}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json(); // { items: [...], nextCursor: string | null }
}

export default function InfiniteFeed() {
  const sentinelRef = useRef<HTMLDivElement>(null);

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isPending,
    isError,
    error,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed({ cursor: pageParam }),
    initialPageParam: null as string | null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
    maxPages: 5, // Keep at most 5 pages in memory
  });

  useEffect(() => {
    const el = sentinelRef.current;
    if (!el) return;

    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasNextPage && !isFetchingNextPage) {
          fetchNextPage();
        }
      },
      { rootMargin: '200px' }
    );

    observer.observe(el);
    return () => observer.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isPending) return <p>Loading feed…</p>;
  if (isError) return <p role="alert">Error: {error.message}</p>;

  return (
    <div>
      {data.pages.map((page, pageIndex) => (
        <ul key={pageIndex}>
          {page.items.map((item: any) => (
            <li key={item.id}>{item.title}</li>
          ))}
        </ul>
      ))}

      <div ref={sentinelRef} style={{ height: 1 }} aria-hidden="true" />

      {isFetchingNextPage && <p>Loading more…</p>}
      {!hasNextPage && <p>You've reached the end.</p>}
    </div>
  );
}
```

**Expected Output:** The feed loads the first 10 items, then automatically fetches more as the user scrolls near the bottom. Each page is appended to the list. When `nextCursor` is `null`, "You've reached the end." is displayed.

**Why This Output Occurs:** `useInfiniteQuery` calls `fetchFeed` with the current `pageParam` (initially `null`). `getNextPageParam` extracts `nextCursor` from the response. The Intersection Observer triggers `fetchNextPage` when the sentinel is within 200px of the viewport. `maxPages: 5` limits memory usage by discarding pages beyond the 5 most recent.

### Real-World Cases

- **Social media feeds:** Infinite scrolling with cursor-based pagination.
- **E-commerce product grids:** Offset/limit pagination with page-number navigation.
- **Search results:** Cursor-based pagination for stable results under concurrent indexing.
- **Admin tables:** Offset/limit with `keepPreviousData` for smooth page transitions.
- **Chat history:** Bidirectional infinite queries (load older/newer messages).

### References

- TanStack Query – Infinite Queries: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- TanStack Query – `useInfiniteQuery`: https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery
- SWR – Pagination: https://swr.vercel.app/docs/pagination
- MDN Web Docs – Intersection Observer API: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

## Core Concept 5: Performance Tuning — Prefetching, Hydration, and Placeholder Data

### Definitions

**Core Definition:** Performance tuning in server-state management is the practice of prefetching data before it is needed, hydrating the cache from server-rendered data, and using placeholder data to avoid loading flashes.

**Technical Definition:** **Prefetching** (`queryClient.prefetchQuery`) starts a fetch before the component that needs the data renders, populating the cache so the component renders instantly from cache. Prefetching is commonly triggered on hover, route transition, or after a mutation. **Hydration** (`HydrationBoundary` in Next.js App Router, `dehydrate`/`hydrate` in TanStack Query) serialises the server-side cache and transfers it to the client, so the client does not refetch data already fetched on the server. **Placeholder data** (`placeholderData` in TanStack Query v5, `keepPreviousData` for pagination) provides temporary data to display while the real data loads, avoiding empty or loading states. `initialData` is different: it is treated as real data and persisted in the cache, while `placeholderData` is ephemeral. TanStack Query v5 renamed `keepPreviousData` (a boolean) to `placeholderData: keepPreviousData` (a function) and added `isPlaceholderData` to distinguish placeholder from real data.

**Beginner-Friendly Explanation:** Prefetching is like ordering your coffee before you reach the counter—when you arrive, it's ready. Hydration is like carrying your notes from home to school—you don't have to re-derive everything on the client. Placeholder data is like a "loading" placeholder that shows the shape of the content before it arrives, so the page doesn't jump around.

### Purposes

- **Prefetching:** To start fetches before components render, eliminating loading states on navigation.
- **Hydration:** To transfer server-fetched data to the client cache, avoiding duplicate fetches.
- **Placeholder data:** To show temporary data while real data loads, reducing perceived latency.
- **`initialData`:** To seed the cache with real data (e.g., from a server component or a parent query).
- **`keepPreviousData`:** To keep the previous page's data visible during pagination.

### Syntax Rules and Structure

**Prefetching on Hover:**
```tsx
import { useQueryClient } from '@tanstack/react-query';

function ProductLink({ productId }) {
  const queryClient = useQueryClient();

  function handleMouseEnter() {
    queryClient.prefetchQuery({
      queryKey: ['product', productId],
      queryFn: () => fetchProduct(productId),
      staleTime: 60_000,
    });
  }

  return <a href={`/products/${productId}`} onMouseEnter={handleMouseEnter}>View</a>;
}
```

**Prefetching on Route Transition:**
```tsx
function usePrefetchOnRoute() {
  const queryClient = useQueryClient();

  return useCallback((nextRoute: string) => {
    if (nextRoute === '/dashboard') {
      queryClient.prefetchQuery({
        queryKey: ['dashboard'],
        queryFn: fetchDashboard,
      });
    }
  }, [queryClient]);
}
```

**Hydration (Next.js App Router):**
```tsx
// app/providers.tsx
'use client';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';

export function Providers({ children }) {
  const [client] = useState(() => new QueryClient());
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>;
}
```

```tsx
// app/page.tsx (Server Component)
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query';

export default async function Page() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <TodoList />
    </HydrationBoundary>
  );
}
```

**Placeholder Data:**
```tsx
import { useQuery, keepPreviousData } from '@tanstack/react-query';

// keepPreviousData for pagination
const { data, isPlaceholderData } = useQuery({
  queryKey: ['items', page],
  queryFn: () => fetchItems(page),
  placeholderData: keepPreviousData,
});

// Static placeholder
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  placeholderData: { name: 'Loading…', email: '' },
});
```

**`initialData` (Real Data):**
```tsx
function UserProfile({ initialUser }) {
  const { data } = useQuery({
    queryKey: ['user', initialUser.id],
    queryFn: () => fetchUser(initialUser.id),
    initialData: initialUser, // Treated as real data
    staleTime: 60_000,
  });
  return <h1>{data.name}</h1>;
}
```

**Syntax Rules:**
- Use `prefetchQuery` for data that will be needed soon (hover, route transition, after mutation).
- Use `HydrationBoundary` + `dehydrate` for SSR/SSG to transfer the server cache to the client.
- Use `placeholderData: keepPreviousData` for pagination to avoid loading flashes.
- Use `initialData` when you have real data to seed the cache (e.g., from a parent query or server component).
- Use `isPlaceholderData` to show a subtle "updating" indicator when displaying placeholder data.
- Set `staleTime` on prefetched queries to prevent immediate refetch on mount.
- Prefetch only critical data; over-prefetching wastes bandwidth.

**Constraints and Limitations:**
- Prefetching on hover can cause unnecessary fetches if the user does not navigate.
- `initialData` is persisted in the cache; it must be real, valid data.
- `placeholderData` is ephemeral and not persisted; it does not trigger a refetch on mount.
- Hydration requires the same query keys on server and client; mismatches cause refetches.
- `keepPreviousData` shows stale data; `isPlaceholderData` should be used to indicate this.
- Over-prefetching can compete with critical requests for bandwidth.

### Annotated Code Example: Prefetch on Hover + Placeholder

```tsx
import { useQuery, useQueryClient, keepPreviousData } from '@tanstack/react-query';

function ProductList() {
  const queryClient = useQueryClient();

  const { data: products } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProducts,
    staleTime: 5 * 60_000,
  });

  function prefetchProduct(id: number) {
    queryClient.prefetchQuery({
      queryKey: ['product', id],
      queryFn: () => fetchProduct(id),
      staleTime: 5 * 60_000,
    });
  }

  return (
    <ul>
      {products?.map(p => (
        <li key={p.id} onMouseEnter={() => prefetchProduct(p.id)}>
          <a href={`/products/${p.id}`}>{p.name}</a>
        </li>
      ))}
    </ul>
  );
}

function ProductDetail({ productId }) {
  const { data, isPlaceholderData } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
    placeholderData: keepPreviousData,
    staleTime: 5 * 60_000,
  });

  return (
    <div>
      {isPlaceholderData && <span>Updating…</span>}
      <h1>{data?.name}</h1>
      <p>{data?.description}</p>
    </div>
  );
}
```

**Expected Output:** Hovering over a product link prefetches its detail data. Clicking the link navigates to the detail page, which renders instantly from the prefetched cache. If the data is stale and refetching, "Updating…" appears without blanking the content.

**Why This Output Occurs:** `prefetchQuery` populates the cache with the product data before navigation. When `ProductDetail` mounts, `useQuery` finds the cached data and renders it immediately. `isPlaceholderData` is `false` because the data is real (prefetched), not a placeholder. If the user navigates to a product that was not prefetched, `keepPreviousData` shows the previous product's data while the new data loads.

### Real-World Cases

- **E-commerce:** Prefetching product details on hover; hydrating product listings from SSR.
- **Dashboards:** Prefetching dashboard data on login; hydrating from server-rendered HTML.
- **Social apps:** Prefetching profile data on avatar hover.
- **Documentation sites:** Hydrating static content from SSG.
- **Admin panels:** Placeholder data for tables during pagination.

### References

- TanStack Query – Prefetching: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query – SSR & Hydration: https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- TanStack Query – Placeholder Query Data: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
- TanStack Query – Initial Query Data: https://tanstack.com/query/latest/docs/framework/react/guides/initial-query-data
- TanStack Query – Paginated Queries (`keepPreviousData`): https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- Next.js – HydrationBoundary: https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr

---

## Core Concept 6: Mutating Server State — Optimistic Updates, Rollback, and Cache Triggers

### Definitions

**Core Definition:** Mutating server state is the practice of performing create, update, or delete operations via `useMutation`, updating the cache optimistically before the server confirms, and rolling back on failure.

**Technical Definition:** `useMutation` manages the lifecycle of a write operation: `mutate`/`mutateAsync` triggers it, `onMutate` runs before the request (for optimistic updates), `onError` runs on failure (for rollback), `onSuccess` runs on success, and `onSettled` runs after either. The standard optimistic update pattern is: cancel outgoing refetches, snapshot the previous cache value, write the optimistic value, return the snapshot as context, roll back in `onError` using the snapshot, and always invalidate in `onSettled` to reconcile with the server. TanStack Query provides `useMutationState` for cross-component optimistic updates, using `mutationKey` and `submittedAt` for identity. React 19's `useOptimistic` provides a similar pattern for form Actions. The key rules are: optimistic updates are optional and add complexity; always roll back on error; always invalidate in `onSettled`; and never mutate the cache directly outside `onMutate`/`setQueryData`.

**Beginner-Friendly Explanation:** When you click "like" on a post, you expect the heart to fill instantly—not after a server round trip. An optimistic update shows the change immediately, assuming the server will succeed. If the server fails, the app silently rolls back to the previous state and shows an error. This makes the app feel instant while keeping data consistent.

### Purposes

- To update the UI immediately after a mutation, without waiting for the server.
- To roll back to the previous state if the mutation fails.
- To reconcile the cache with the server after the mutation settles.
- To prevent race conditions by cancelling outgoing refetches before optimistic updates.
- To support concurrent optimistic updates with `useMutationState` and `submittedAt`.

### Syntax Rules and Structure

**Optimistic Update Pattern (TanStack Query):**
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useToggleTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, completed }) =>
      fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed }),
      }).then(r => r.json()),

    onMutate: async ({ id, completed }) => {
      // 1. Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // 2. Snapshot the previous value
      const previous = queryClient.getQueryData(['todos']);

      // 3. Optimistically update the cache
      queryClient.setQueryData(['todos'], (old: any[]) =>
        old?.map(t => (t.id === id ? { ...t, completed } : t))
      );

      // 4. Return context for rollback
      return { previous };
    },

    onError: (_err, _vars, context) => {
      // 5. Roll back on error
      if (context?.previous) {
        queryClient.setQueryData(['todos'], context.previous);
      }
    },

    onSettled: () => {
      // 6. Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

**Component Breakdown:**
- `onMutate`: Runs before the mutation; cancels refetches, snapshots, and optimistically updates.
- `return { previous }`: Context passed to `onError` and `onSettled`.
- `onError`: Rolls back to the snapshot.
- `onSettled`: Invalidates the query to reconcile with the server.

**Concurrent Optimistic Updates (`useMutationState`):**
```tsx
import { useMutationState } from '@tanstack/react-query';

function TodoList() {
  const { data: todos } = useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

  const pendingTodos = useMutationState({
    filters: { mutationKey: ['addTodo'], status: 'pending' },
    select: (mutation) => mutation.state.variables,
  });

  return (
    <ul>
      {todos?.map(t => <li key={t.id}>{t.title}</li>)}
      {pendingTodos.map((t: any, i) => (
        <li key={`pending-${i}`} style={{ opacity: 0.5 }}>{t.title} (saving…)</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `useMutationState`: Reads the state of all mutations matching the filter.
- `filters: { mutationKey, status: 'pending' }`: Selects pending `addTodo` mutations.
- `select: (m) => m.state.variables`: Extracts the variables (the new todo).

**Optimistic Update via UI (Simpler Variant):**
```tsx
const mutation = useMutation({
  mutationFn: addTodo,
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
});

return (
  <ul>
    {todos?.map(t => <li key={t.id}>{t.title}</li>)}
    {mutation.isPending && (
      <li style={{ opacity: 0.5 }}>{mutation.variables.title}</li>
    )}
  </ul>
);
```

**Syntax Rules:**
- Always cancel outgoing refetches in `onMutate` before writing to the cache.
- Always snapshot the previous value and return it as context.
- Always roll back in `onError` using the context.
- Always invalidate in `onSettled` (not just `onSuccess`) to handle both outcomes.
- Use `submittedAt` as a unique key for concurrent optimistic updates.
- Use the UI variant for simple cases; use the cache variant for complex cache manipulations.
- Never mutate the cache outside `onMutate` or `setQueryData`.
- Handle the case where `context` is `undefined` (e.g., if `onMutate` failed).

**Constraints and Limitations:**
- Optimistic updates add complexity; only use them when the UX benefit is significant.
- Rolling back can cause visual flicker if the mutation fails quickly.
- Concurrent optimistic updates require careful key management.
- Invalidation after optimistic updates can cause a brief "double render" (optimistic + server data).
- React 19's `useOptimistic` is an alternative for form Actions; it does not replace `useMutation`.

### Annotated Code Example: Optimistic Like Button

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function LikeButton({ postId }) {
  const queryClient = useQueryClient();

  const { data: post } = useQuery({
    queryKey: ['post', postId],
    queryFn: () => fetchPost(postId),
  });

  const mutation = useMutation({
    mutationFn: (liked: boolean) =>
      fetch(`/api/posts/${postId}/like`, {
        method: liked ? 'DELETE' : 'POST',
      }).then(r => r.json()),

    onMutate: async (liked) => {
      await queryClient.cancelQueries({ queryKey: ['post', postId] });
      const previous = queryClient.getQueryData(['post', postId]);

      queryClient.setQueryData(['post', postId], (old: any) => ({
        ...old,
        likes: liked ? old.likes - 1 : old.likes + 1,
        liked: !liked,
      }));

      return { previous };
    },

    onError: (_err, _liked, context) => {
      if (context?.previous) {
        queryClient.setQueryData(['post', postId], context.previous);
      }
    },

    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['post', postId] });
    },
  });

  return (
    <button onClick={() => mutation.mutate(post?.liked ?? false)}>
      {post?.liked ? '❤️' : '🤍'} {post?.likes}
    </button>
  );
}
```

**Expected Output:** Clicking the heart instantly toggles the like state and updates the count. If the server succeeds, the state remains. If it fails, the heart and count revert, and the cache is invalidated to refetch the true state.

**Why This Output Occurs:** `onMutate` cancels refetches, snapshots the post, and optimistically updates the like state. `onError` rolls back using the snapshot. `onSettled` invalidates the post query, triggering a refetch to confirm the final state.

### Real-World Cases

- **Social media:** Liking, following, bookmarking with optimistic updates.
- **E-commerce:** Adding to cart, updating quantities.
- **Task management:** Toggling todo completion, reordering items.
- **Chat:** Sending messages with optimistic rendering.
- **Settings:** Toggling preferences with instant feedback.

### References

- TanStack Query – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query – `useMutation`: https://tanstack.com/query/latest/docs/framework/react/reference/useMutation
- TanStack Query – `useMutationState`: https://tanstack.com/query/latest/docs/framework/react/reference/useMutationState
- TanStack Query – Invalidations from Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/invalidations-from-mutations
- React – `useOptimistic`: https://react.dev/reference/react/useOptimistic

---

## Core Concept 7: Real-Time Sync Paradigms — Smart Polling, WebSockets, Background Sync

### Definitions

**Core Definition:** Real-time sync paradigms are strategies for keeping the client cache synchronised with server changes as they happen—through polling, WebSocket push, or background synchronisation.

**Technical Definition:** **Smart polling** uses `refetchInterval` (TanStack Query) or `refreshInterval` (SWR) to periodically refetch data. By default, polling pauses when the browser tab is hidden; `refetchIntervalInBackground: true` continues polling. Polling is simple but wasteful and inherently laggy. **WebSocket integration** opens a persistent, full-duplex connection; on each server message, the client writes through to the cache using `queryClient.setQueryData` (for known updates) or invalidates queries (for unknown updates). WebSockets provide instant updates with minimal overhead but require connection management, reconnection logic, and authentication. **Background synchronisation** uses the Background Sync API (service workers) or a queue of failed mutations that are retried when connectivity is restored. TanStack Query supports offline-first patterns with `networkMode: 'offlineFirst'` and mutation persistence via `persistQueryClient`. The `useSyncExternalStore` API allows integrating custom real-time sources (WebSocket, SSE) with React's rendering.

**Beginner-Friendly Explanation:** Real-time sync is about keeping your app's data fresh as the server changes. **Polling** is like refreshing your email every 30 seconds—simple but wasteful. **WebSockets** are like a phone call—the server can push updates instantly. **Background sync** is like a mail carrier who keeps trying to deliver a package until it succeeds, even if you go offline temporarily.

### Purposes

- **Polling:** To keep data fresh for dashboards and feeds without WebSocket infrastructure.
- **WebSockets:** To push real-time updates (chat, notifications, collaborative editing) with minimal latency.
- **SSE:** To stream one-way updates (stock tickers, live scores) over HTTP.
- **Cache write-through:** To update the cache directly when a known change arrives.
- **Cache invalidation:** To refetch when an unknown change arrives.
- **Background sync:** To queue mutations when offline and retry when connectivity returns.
- **Offline-first:** To serve cached data when offline and reconcile when online.

### Syntax Rules and Structure

**Smart Polling (TanStack Query):**
```tsx
useQuery({
  queryKey: ['metrics'],
  queryFn: fetchMetrics,
  refetchInterval: 30_000,               // Poll every 30s
  refetchIntervalInBackground: false,    // Pause when tab is hidden
  staleTime: 10_000,                     // Consider fresh for 10s
});
```

**Smart Polling (SWR):**
```tsx
useSWR('/api/metrics', fetcher, {
  refreshInterval: 30_000,
  refreshWhenHidden: false,
  refreshWhenOffline: false,
});
```

**WebSocket Integration — Cache Write-Through:**
```tsx
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';

function useTodoSocket() {
  const queryClient = useQueryClient();

  useEffect(() => {
    const socket = new WebSocket('wss://api.example.com/todos');

    socket.onmessage = (event) => {
      const message = JSON.parse(event.data);

      switch (message.type) {
        case 'todo:created':
          // Write-through: append to the cached list
          queryClient.setQueryData(['todos'], (old: any[] = []) => [
            ...old,
            message.todo,
          ]);
          break;

        case 'todo:updated':
          queryClient.setQueryData(['todos'], (old: any[] = []) =>
            old.map(t => (t.id === message.todo.id ? message.todo : t))
          );
          break;

        case 'todo:deleted':
          queryClient.setQueryData(['todos'], (old: any[] = []) =>
            old.filter(t => t.id !== message.todoId)
          );
          break;

        default:
          // Unknown change: invalidate to refetch
          queryClient.invalidateQueries({ queryKey: ['todos'] });
      }
    };

    return () => socket.close();
  }, [queryClient]);
}
```

**Component Breakdown:**
- `new WebSocket('wss://…')`: Opens the connection.
- `socket.onmessage`: Handles incoming messages.
- `setQueryData`: Writes through to the cache for known changes.
- `invalidateQueries`: Refetches for unknown changes.
- `socket.close()`: Cleanup on unmount.

**WebSocket with Reconnection and Backoff:**
```tsx
function useReconnectingSocket(url: string, onMessage: (data: any) => void) {
  useEffect(() => {
    let socket: WebSocket | null = null;
    let retries = 0;
    let reconnectTimer: ReturnType<typeof setTimeout>;
    let closed = false;

    function connect() {
      socket = new WebSocket(url);

      socket.onopen = () => { retries = 0; };
      socket.onmessage = (event) => onMessage(JSON.parse(event.data));
      socket.onclose = () => {
        if (closed) return;
        const delay = Math.min(1000 * 2 ** retries, 30_000);
        const jitter = Math.random() * 1000;
        reconnectTimer = setTimeout(connect, delay + jitter);
        retries++;
      };
      socket.onerror = () => socket?.close();
    }

    connect();

    return () => {
      closed = true;
      clearTimeout(reconnectTimer);
      socket?.close();
    };
  }, [url, onMessage]);
}
```

**Component Breakdown:**
- `retries`: Counts reconnection attempts.
- `Math.min(1000 * 2 ** retries, 30_000)`: Exponential backoff capped at 30s.
- `Math.random() * 1000`: Jitter to prevent thundering herd.
- `closed`: Prevents reconnection after unmount.

**Background Sync with Mutations (TanStack Query Persist):**
```tsx
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const persister = createSyncStoragePersister({
  storage: window.localStorage,
});

<PersistQueryClientProvider
  client={queryClient}
  persistOptions={{ persister, maxAge: 24 * 60 * 60 * 1000 }}
>
  <App />
</PersistQueryClientProvider>
```

**Component Breakdown:**
- `persister`: Persists the query cache to `localStorage`.
- `maxAge`: Discards persisted data older than 24 hours.
- Mutations can also be persisted with `persistQueryClient` + `defaultOptions.mutations`.

**Syntax Rules:**
- Use `refetchInterval` for polling; set `refetchIntervalInBackground: false` (default) to save bandwidth.
- Use WebSockets for real-time push; write through to the cache for known changes and invalidate for unknown ones.
- Always handle WebSocket reconnection with exponential backoff and jitter.
- Always clean up the WebSocket on unmount (`socket.close()`).
- Use `networkMode: 'offlineFirst'` to serve cached data when offline.
- Persist the query cache with `persistQueryClient` for offline-first apps.
- Use `useSyncExternalStore` to integrate custom real-time sources with React.
- Prefer SSE over WebSockets for one-way server-to-client streaming.

**Constraints and Limitations:**
- Polling is wasteful and inherently laggy; use it only when WebSockets are not feasible.
- WebSockets require server infrastructure, authentication, and reconnection logic.
- WebSocket messages can arrive out of order; use timestamps or sequence numbers to reconcile.
- Background sync requires service workers; browser support varies.
- Persisted caches can grow large; limit `maxAge` and the number of persisted queries.
- `networkMode: 'offlineFirst'` disables some retry behaviours; understand the trade-offs.

### Annotated Code Example: WebSocket + TanStack Query Cache Write-Through

```tsx
import { useEffect } from 'react';
import { useQuery, useQueryClient } from '@tanstack/react-query';

async function fetchMessages(roomId: string) {
  const res = await fetch(`/api/rooms/${roomId}/messages`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

function useRoomSocket(roomId: string) {
  const queryClient = useQueryClient();

  useEffect(() => {
    const socket = new WebSocket(`wss://api.example.com/rooms/${roomId}`);

    socket.onmessage = (event) => {
      const message = JSON.parse(event.data);

      // Write-through: append the new message to the cached list
      queryClient.setQueryData(['messages', roomId], (old: any[] = []) => [
        ...old,
        message,
      ]);
    };

    socket.onerror = () => {
      // On error, invalidate to refetch the full list
      queryClient.invalidateQueries({ queryKey: ['messages', roomId] });
    };

    return () => socket.close();
  }, [roomId, queryClient]);
}

export default function ChatRoom({ roomId }: { roomId: string }) {
  const { data: messages, isPending } = useQuery({
    queryKey: ['messages', roomId],
    queryFn: () => fetchMessages(roomId),
    staleTime: Infinity, // WebSocket keeps the cache fresh
  });

  useRoomSocket(roomId);

  if (isPending) return <p>Loading messages…</p>;

  return (
    <ul>
      {messages.map((m: any) => (
        <li key={m.id}>
          <strong>{m.author}:</strong> {m.text}
        </li>
      ))}
    </ul>
  );
}
```

**Expected Output:** The chat room loads the initial message history via `useQuery`. New messages arrive via WebSocket and are appended to the cache via `setQueryData`, causing the component to re-render with the new message. The `staleTime: Infinity` prevents TanStack Query from refetching, since the WebSocket keeps the cache fresh.

**Why This Output Occurs:** `useQuery` fetches the initial messages and caches them under `['messages', roomId]`. `useRoomSocket` opens a WebSocket and, on each incoming message, calls `setQueryData` to append it to the cached list. React re-renders with the updated list. The `staleTime: Infinity` tells TanStack Query not to refetch, because the WebSocket is the source of truth for updates.

### Real-World Cases

- **Chat applications:** WebSocket push with cache write-through for instant message rendering.
- **Collaborative editing:** WebSocket or WebRTC for real-time document updates.
- **Stock tickers:** SSE or WebSocket for streaming price updates.
- **Notifications:** WebSocket push for real-time notification badges.
- **Offline-first apps:** Persisted query cache + background sync for offline access and reconciliation.
- **Live dashboards:** Smart polling with `refetchInterval` for metrics that must stay current.

### References

- TanStack Query – Polling: https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query – Network Mode: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode
- TanStack Query – Persist Query Client: https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient
- TanStack Query – `useSyncExternalStore` Integration: https://react.dev/reference/react/useSyncExternalStore
- MDN Web Docs – WebSocket API: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – Server-Sent Events: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- MDN Web Docs – Background Synchronization API: https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API
- SWR – Real-Time: https://swr.vercel.app/docs/revalidation

---

## Comparison and Decision Guidance

| Concern | Tool/Pattern | When to Use | Key Risk |
|---|---|---|---|
| **Caching engine** | TanStack Query | Complex apps with mutations, pagination, devtools | Bundle size (~12–15 KB) |
| **Caching engine** | SWR | Simple read-heavy apps | Less powerful mutations |
| **Freshness** | `staleTime` (TanStack) / `dedupingInterval` (SWR) | All queries | `staleTime: 0` causes over-fetching |
| **Retention** | `gcTime` | All queries | `gcTime < staleTime` collects fresh data |
| **Deduplication** | Automatic in both libraries | All concurrent identical requests | Does not apply across different keys |
| **Pagination** | `useInfiniteQuery` (TanStack) / `useSWRInfinite` (SWR) | Feeds, lists, tables | Memory growth without `maxPages` |
| **Prefetching** | `prefetchQuery` | Hover, route transition | Over-prefetching wastes bandwidth |
| **Hydration** | `dehydrate` + `HydrationBoundary` | SSR/SSG | Key mismatches cause refetches |
| **Placeholder data** | `placeholderData: keepPreviousData` | Pagination | Shows stale data (use `isPlaceholderData`) |
| **Optimistic updates** | `onMutate` + `onError` + `onSettled` | Likes, toggles, cart | Complexity; rollback flicker |
| **Real-time** | WebSocket + `setQueryData` | Chat, notifications | Reconnection, ordering |
| **Polling** | `refetchInterval` | Dashboards without WebSockets | Wasteful, laggy |
| **Offline-first** | `persistQueryClient` + `networkMode: 'offlineFirst'` | PWAs, mobile | Storage limits, stale data |

**Decision Guidance:**
- **Start with TanStack Query** for any app with mutations, pagination, or complex cache needs.
- **Use SWR** for read-heavy apps with minimal mutation logic and a smaller bundle target.
- **Set `staleTime` deliberately:** `Infinity` for static data, `0` for real-time data, and 1–30 minutes for most data.
- **Set `gcTime` >= `staleTime`** to prevent fresh data from being garbage collected.
- **Use `useInfiniteQuery`** for cursor-based pagination and infinite scrolling; use `useQuery` + `keepPreviousData` for offset/limit with page numbers.
- **Prefetch on hover and route transition** for a snappy navigation experience.
- **Hydrate from the server** in Next.js App Router or Remix to avoid duplicate fetches.
- **Use optimistic updates** for user actions where instant feedback matters (likes, toggles, cart).
- **Integrate WebSockets** for real-time chat, notifications, and collaborative editing; write through to the cache.
- **Persist the query cache** for offline-first PWAs; cap `maxAge` to avoid unbounded growth.

---

## References

- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query – Query Invalidation: https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query – Query Filters: https://tanstack.com/query/latest/docs/framework/react/guides/filters
- TanStack Query – Parallel Queries: https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries
- TanStack Query – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query – Infinite Queries: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
- TanStack Query – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- TanStack Query – Prefetching: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query – SSR & Hydration: https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- TanStack Query – Placeholder Query Data: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
- TanStack Query – Initial Query Data: https://tanstack.com/query/latest/docs/framework/react/guides/initial-query-data
- TanStack Query – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query – Invalidations from Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/invalidations-from-mutations
- TanStack Query – Polling: https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query – Network Mode: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode
- TanStack Query – Persist Query Client: https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient
- TanStack Query – `useMutation`: https://tanstack.com/query/latest/docs/framework/react/reference/useMutation
- TanStack Query – `useMutationState`: https://tanstack.com/query/latest/docs/framework/react/reference/useMutationState
- TanStack Query – `useInfiniteQuery`: https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery
- SWR – Documentation: https://swr.vercel.app/
- SWR – Options: https://swr.vercel.app/docs/options
- SWR – Pagination: https://swr.vercel.app/docs/pagination
- SWR – Revalidation: https://swr.vercel.app/docs/revalidation
- React – `useOptimistic`: https://react.dev/reference/react/useOptimistic
- React – `useSyncExternalStore`: https://react.dev/reference/react/useSyncExternalStore
- MDN Web Docs – WebSocket API: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – Server-Sent Events: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- MDN Web Docs – Background Synchronization API: https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API
- MDN Web Docs – Intersection Observer API: https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API