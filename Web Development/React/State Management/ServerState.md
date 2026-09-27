# React Server State: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Server State is the management of asynchronous data that originates from a remote server, characterised by its separate cache lifecycle, loading states, error states, and remote ownership—managed by libraries like TanStack Query (React Query).

**Technical Definition:** Server state refers to data that is owned by a remote server and fetched into the client for display. It differs fundamentally from client state: it is asynchronous (requires loading and error states), it can become stale (the server's copy may change independently), it has a cache lifecycle (staleness, garbage collection, refetching), and it is not owned by any single component. TanStack Query v5 is the dominant library for server-state management, providing a stale-while-revalidate cache keyed by deterministic arrays (query keys). It ships with production-ready defaults: `staleTime: 0` (data is instantly stale), `gcTime: 5 * 60_000` (unused cache kept for 5 minutes), `retry: 3` (3 retries with exponential backoff), `refetchOnWindowFocus: true`, and `refetchOnReconnect: true`. The philosophy is that TanStack Query is an **async state manager, not a data fetcher**: you provide the Promise, and Query manages caching, background updates, and synchronization. A critical rule is to never copy server data into a client state store; TanStack Query is the single source of truth for API data.

**Beginner-Friendly Explanation:** Imagine you order a book from an online store. The store (server) owns the book. You (the client) get a copy delivered to your house (the cache). The copy might be outdated if the store updates the book's contents. You do not "own" the book—you have a cached copy. TanStack Query is like a librarian who tracks which copies are fresh, automatically re-orders updated versions, and tells you when a copy is being fetched. You do not need to manually manage when to re-order or which copy is stale—the librarian handles it all.

### Key Characteristics

- **Asynchronous by Nature:** Server state requires loading, error, and success states that client state does not.
- **Cache Lifecycle:** Server state has a cache lifecycle controlled by `staleTime` (how long data is fresh) and `gcTime` (how long inactive data stays in memory).
- **Stale-While-Revalidate:** TanStack Query serves stale data immediately from cache while revalidating in the background.
- **Background Synchronization:** Queries automatically refetch when the window is refocused, the network reconnects, or new instances mount.
- **Mutation-Driven Updates:** State changes on the server are performed via mutations, which then invalidate and refetch related queries.
- **Optimistic Updates:** The UI can be updated immediately (before server confirmation) and rolled back if the mutation fails.
- **Single Source of Truth for API Data:** Server data lives in the Query cache, not in a client store.

### Prerequisites

- Solid understanding of JavaScript promises, `async`/`await`, and the event loop.
- Familiarity with React function components and the `useState`/`useEffect` Hooks.
- Working knowledge of the Fetch API and HTTP status codes.
- Basic understanding of caching concepts and cache invalidation.

### Related Programming Areas

- **State Architecture:** Distinguishing server state from client, form, and URL state.
- **Caching:** Managing cache lifecycles, staleness, and garbage collection.
- **Concurrency Control:** Handling race conditions and request cancellation.
- **Network Programming:** HTTP requests, retries, and offline behaviour.
- **User Experience Design:** Loading skeletons, error boundaries, and optimistic UI.

### Core Concepts / Features

1. Query State (Declarative Configuration and Status)
2. Cache State (Stale-While-Revalidate and Garbage Collection)
3. Synchronization with APIs (Background Refetching)
4. React Query / TanStack Query Concepts (Hooks and Orchestration)
5. Mutations (Server-Side Data Alterations)
6. Cache Invalidation (Targeted Staleness and Refetching)
7. Optimistic Updates (Instant UI Feedback with Rollback)
8. Background Refetching (Focus, Reconnect, and Polling)

---

## Core Concept 1: Query State (Declarative Configuration and Status)

### Definitions

**Core Definition:** Query state is the declarative configuration object passed to `useQuery` that wraps a network request configuration, handling standard loading, error, and ready conditions automatically.

**Technical Definition:** A query is a declarative dependency on an asynchronous source of data that is tied to a unique key. The `useQuery` Hook accepts an object with at least a `queryKey` (a unique array identifying the query) and a `queryFn` (a function that returns a promise resolving to the data). The Hook returns a result object containing the query's current state: `isPending` (no data yet), `isError` (error encountered), and `isSuccess` (data available). TanStack Query v5 renamed the loading state to `pending` to make it less confusing, since a disabled query that has not executed should not be in a "loading" state. Beyond the primary status, `isFetching` indicates whether the query is fetching at any time (including background refetching), and `error` and `data` provide the actual error object or data payload. In v5, Suspense is first-class via `useSuspenseQuery`, where data is never typed as `undefined` and loading/error states are handled by Suspense and Error Boundaries.

**Beginner-Friendly Explanation:** A query is like placing an order at a restaurant. You tell the waiter what you want (the `queryKey`) and how to get it (the `queryFn`). While you wait, the restaurant tells you the status: "preparing" (pending), "ready" (success), or "something went wrong" (error). You don't have to manually check the kitchen—the waiter keeps you updated automatically.

### Purposes

- To fetch remote data and expose its loading, error, and success states declaratively.
- To uniquely identify queries with query keys for caching, refetching, and sharing across components.
- To provide a consistent interface for handling asynchronous data regardless of the underlying HTTP method.
- To enable TypeScript type narrowing based on query status (checking `isLoading` and `isError` narrows `data`).
- To support Suspense and Error Boundaries for declarative loading and error handling.

### Syntax Rules and Structure

**General Syntax (`useQuery`):**
```tsx
import { useQuery } from '@tanstack/react-query';

function Todos() {
  const { isPending, isError, data, error, isFetching } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  });

  if (isPending) return <span>Loading...</span>;
  if (isError)   return <span>Error: {error.message}</span>;

  return (
    <ul>
      {data.map(todo => <li key={todo.id}>{todo.title}</li>)}
      {isFetching && <div>Updating...</div>}
    </ul>
  );
}
```

**Component Breakdown:**
- `queryKey: ['todos']`: A unique array identifying the query.
- `queryFn: fetchTodoList`: A function that returns a promise resolving to the data.
- `isPending`, `isError`, `data`, `error`: The query's status and payload.
- `isFetching`: `true` whenever a fetch is in progress, including background refetches.

**General Syntax (Suspense):**
```tsx
import { useSuspenseQuery } from '@tanstack/react-query';

function Todos() {
  const { data } = useSuspenseQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  });

  // data is guaranteed to be defined — no loading or error checks needed
  return <ul>{data.map(todo => <li key={todo.id}>{todo.title}</li>)}</ul>;
}
```

**Component Breakdown:**
- `useSuspenseQuery`: Suspense-first variant; data is never `undefined`.
- Loading and error states are handled by `<Suspense>` and Error Boundaries.

**Syntax Rules:**
- Always provide a `queryKey` as an array; it is used for caching, refetching, and sharing.
- The `queryFn` must return a promise that resolves with the data or throws an error.
- Check `isPending` first, then `isError`, then assume `isSuccess` and render the data.
- Use `isFetching` to show a subtle "updating" indicator during background refetches without blanking the screen.
- In v5, prefer `useSuspenseQuery` for declarative loading and error handling with Suspense.

**Constraints and Limitations:**
- Queries must be tied to a unique key; duplicate keys cause cache collisions.
- The `queryFn` must be a stable reference (defined outside the component or memoised) to avoid unnecessary refetches.
- `useSuspenseQuery` requires a `<Suspense>` boundary and an Error Boundary above it.
- In v5, `useQuery` callbacks (`onSuccess`, `onError`, `onSettled`) have been removed; use `useEffect` or the `queryFn` for side effects.

### Annotated Code Example: Todo List with Query States

```tsx
import { useQuery } from '@tanstack/react-query';

// The query function — defined outside the component for stability
async function fetchTodos() {
  const response = await fetch('https://jsonplaceholder.typicode.com/todos');
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

function TodoList() {
  const {
    isPending,    // No data yet — first load
    isError,      // Query encountered an error
    data,         // The resolved data (when isSuccess)
    error,        // The error object (when isError)
    isFetching,   // Fetching in background (including refetches)
  } = useQuery({
    queryKey: ['todos'],  // Unique key for caching and sharing
    queryFn: fetchTodos,   // Promise-returning function
  });

  // 1. First load: no data yet
  if (isPending) return <p>Loading todos...</p>;

  // 2. Error state: show error message
  if (isError) return <p>Error: {error.message}</p>;

  // 3. Success state: data is available
  return (
    <div>
      {/* Subtle indicator during background refetches — does not blank the screen */}
      {isFetching && <p>Updating...</p>}
      <ul>
        {data.map(todo => (
          <li key={todo.id}>
            {todo.completed ? '✅' : '⬜'} {todo.title}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default function App() {
  return <TodoList />;
}
```

**Expected Output:** Initially, "Loading todos..." is displayed. After the API responds, the todo list is rendered with checkboxes and titles. If the user refocuses the window and the data is stale, "Updating..." appears briefly while fresh data is fetched in the background, without blanking the list.

**Why This Output Occurs:** The `useQuery` Hook manages the entire fetch lifecycle. On mount, `isPending` is `true` because there is no cached data. When the promise resolves, `isSuccess` becomes `true` and `data` contains the todos. If the window is refocused and the query is stale (default `staleTime: 0`), `isFetching` becomes `true` while a background refetch occurs; the existing `data` remains displayed, so the UI does not flash. If the request fails, `isError` becomes `true` and `error` contains the thrown error.

### Real-World Cases

- **Product listings:** Fetching products with pagination, caching, and background refetching.
- **User dashboards:** Loading user statistics, activity feeds, and notifications with independent query states.
- **Search autocomplete:** Fetching search suggestions with debouncing and request cancellation.
- **Dependent data:** Fetching a user first, then fetching their orders with `enabled: !!userId`.

### References

- TanStack Query v5 – Queries: https://tanstack.com/query/latest/docs/framework/react/guides/queries
- TanStack Query v5 – useQuery Reference: https://tanstack.com/query/latest/docs/framework/react/reference/useQuery
- TanStack Query v5 – useSuspenseQuery: https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseQuery
- TanStack Query v5 – Suspense: https://tanstack.com/query/latest/docs/framework/react/guides/suspense

---

## Core Concept 2: Cache State (Stale-While-Revalidate and Garbage Collection)

### Definitions

**Core Definition:** Cache state is the localized repository where remote server payloads are saved to immediately render views on subsequent page transitions, governed by two timing parameters: `staleTime` (how long data is considered fresh) and `gcTime` (how long inactive data stays in memory).

**Technical Definition:** The QueryCache is responsible for storing and managing all query data. It is a `Map` where keys are query hashes (derived from query keys) and values are `Query` instances containing state and data. TanStack Query implements a **stale-while-revalidate** caching strategy: stale data is served immediately from the cache, and revalidation occurs in the background if the data is stale, with the UI updating when fresh data arrives. Two fundamental timing concepts control caching behaviour. **Stale time** (`staleTime`) determines how long data is considered fresh; the default is `0`, meaning data is immediately stale and will refetch on the next mount or window focus. Setting `staleTime: 'static'` means the data is never considered stale and will not automatically refetch. **GC time** (`gcTime`, formerly `cacheTime` in v4) determines how long inactive data stays in the cache; the default is 5 minutes (300,000 ms). When a query's cache becomes unused or inactive, that cache data is garbage collected after this duration. Setting `gcTime: Infinity` disables garbage collection. `gcTime` should always be greater than or equal to `staleTime`; otherwise, data might be removed from the cache while it is still fresh. TanStack Query also supports persisting the query cache to external storage (localStorage, IndexedDB) via `persistQueryClient`, allowing cached data to be restored across page reloads.

**Beginner-Friendly Explanation:** Imagine a library with a reading room. When you request a book (query), the librarian fetches it from the archive (server) and places it on a shelf (cache). The book stays on the shelf for a certain amount of time (staleTime) before it is considered outdated. If someone asks for it again while it is still fresh, the librarian hands it over immediately. If it is outdated, the librarian hands over the old copy but simultaneously sends a request to the archive for the latest edition. If nobody asks for the book for a long time (gcTime), the librarian removes it from the shelf to make room for other books.

### Purposes

- To serve cached data immediately on subsequent page transitions, providing a 0ms perceived load time.
- To control how long data is considered fresh before background refetching is triggered.
- To automatically garbage collect inactive query data to free memory.
- To persist the query cache to storage for offline access or cross-session caching.
- To implement a stale-while-revalidate strategy that balances freshness with instant rendering.

### Syntax Rules and Structure

**General Syntax (staleTime and gcTime):**
```tsx
useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 5 * 60 * 1000,  // 5 minutes — data considered fresh
  gcTime: 10 * 60 * 1000,    // 10 minutes — inactive data kept in cache
});
```

**Component Breakdown:**
- `staleTime`: Milliseconds until data becomes stale (default `0`).
- `gcTime`: Milliseconds until inactive data is garbage collected (default `300000`).
- `gcTime` should be >= `staleTime`.

**General Syntax (Cache Persistence):**
```tsx
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const persister = createSyncStoragePersister({
  storage: window.localStorage,
});

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      gcTime: 1000 * 60 * 60 * 24, // 24 hours
    },
  },
});

<PersistQueryClientProvider client={queryClient} persistOptions={{ persister }}>
  <App />
</PersistQueryClientProvider>
```

**Component Breakdown:**
- `createSyncStoragePersister`: Creates a persister that saves the cache to `localStorage`.
- `PersistQueryClientProvider`: Wraps the app and restores the cache on mount.
- `gcTime` must be overridden when persisting to prevent premature garbage collection.

**Syntax Rules:**
- Set `staleTime` deliberately (not the default `0`) based on how frequently the data changes.
- Set `gcTime` >= `staleTime` to prevent fresh data from being garbage collected.
- Use `staleTime: 'static'` for data that never needs automatic refetching.
- When persisting, override `gcTime` to a longer duration (e.g., 24 hours) to prevent the persisted cache from being discarded too quickly.
- The default `staleTime` of `0` means data is immediately stale and will refetch on mount, window focus, and reconnect.

**Constraints and Limitations:**
- A `staleTime` of `0` (default) causes aggressive refetching; raise it for data that does not change frequently.
- Persisted caches can grow large; limit the number of persisted queries or use `maxAge` in persist options.
- Garbage collection only removes inactive queries; active queries (rendered in components) are never garbage collected.
- `staleTime: 'static'` means the data will never automatically refetch; manual invalidation is still required to update it.

### Annotated Code Example: Posts with Custom Cache Timing

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchPosts() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

function Posts() {
  const { data, isFetching, isPending } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    // Data is considered fresh for 10 minutes
    staleTime: 10 * 60 * 1000,
    // Inactive cache data is kept for 30 minutes
    gcTime: 30 * 60 * 1000,
  });

  if (isPending) return <p>Loading posts...</p>;

  return (
    <div>
      {/* isFetching is true only when a background refetch occurs */}
      {isFetching && <p>Checking for updates...</p>}
      <ul>
        {data.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** On first mount, "Loading posts..." is displayed. After the API responds, the posts are rendered. If the user navigates away and returns within 10 minutes, the cached posts are rendered instantly with no "Loading" message (data is still fresh). After 10 minutes, the posts are rendered from cache while "Checking for updates..." appears briefly during the background refetch. If the component is unmounted for more than 30 minutes, the cache is garbage collected, and a fresh fetch occurs on the next mount.

**Why This Output Occurs:** `staleTime: 10 * 60 * 1000` means the data remains fresh for 10 minutes. During this window, remounting the component does not trigger a refetch—the cached data is served immediately. After 10 minutes, the data is stale, so the component renders the cached data while `isFetching` becomes `true` for a background refetch. `gcTime: 30 * 60 * 1000` means that if the query has no active observers for 30 minutes, the cache entry is removed, and the next mount starts from scratch.

### Real-World Cases

- **Dashboard analytics:** Setting `staleTime` to 5 minutes for metrics that update every few minutes.
- **User profiles:** Setting `staleTime` to 30 minutes for data that rarely changes.
- **Product catalogues:** Setting `staleTime` to 1 hour for product data that changes infrequently.
- **Offline-first apps:** Persisting the query cache to `localStorage` or `IndexedDB` for offline access.
- **Real-time dashboards:** Setting `staleTime: 0` and `refetchInterval: 30_000` for data that must stay current.

### References

- TanStack Query v5 – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query v5 – persistQueryClient: https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient
- TanStack Query v5 – createSyncStoragePersister: https://tanstack.com/query/latest/docs/framework/react/plugins/createSyncStoragePersister

---

## Core Concept 3: Synchronization with APIs (Background Refetching)

### Definitions

**Core Definition:** Synchronization with APIs is the background workflow that ensures local UI views reconcile regularly with underlying back-end data shifts, automatically refetching stale data when the window regains focus, the network reconnects, or new query instances mount.

**Technical Definition:** TanStack Query's synchronization mechanism is built on the principle that stale queries should be refetched automatically in the background. The default behaviour is aggressive but sane: query instances consider cached data as stale by default (`staleTime: 0`), and stale queries are refetched automatically when new instances of the query mount, the window is refocused, the network is reconnected, or the query is configured with a refetch interval. This is controlled by options like `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`, and `refetchInterval`. Window focus refetching is enabled by default: if a user leaves the application and returns, and the query data is stale, TanStack Query automatically requests fresh data in the background. Reconnect refetching is also enabled by default: if the network disconnects and reconnects, stale queries are refetched. Polling via `refetchInterval` makes a query refetch on a timer; by default, polling pauses when the browser tab loses focus, but `refetchIntervalInBackground` can be enabled to continue polling even when the tab is hidden. For infinite queries, `refetchOnMount`, `refetchOnWindowFocus`, and `refetchOnReconnect` default to `true`.

**Beginner-Friendly Explanation:** Synchronization is like having a news app that automatically refreshes when you open it, when your internet connection comes back, or every few minutes. You don't have to pull-to-refresh—the app quietly fetches the latest headlines while you're reading the old ones. When the fresh data arrives, the screen updates seamlessly. If you're on a plane with no internet, the app shows you the last downloaded articles until you reconnect.

### Purposes

- To ensure the UI displays the most current data without requiring manual refreshes.
- To automatically recover from network interruptions by refetching when connectivity is restored.
- To keep data fresh when the user returns to the tab after being away.
- To support polling for dashboards and real-time data that must stay current.
- To avoid unnecessary refetches by only refetching stale data.

### Syntax Rules and Structure

**General Syntax (Refetch Options):**
```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 30_000,              // Consider fresh for 30 seconds
  refetchOnMount: true,           // Refetch on mount if stale (default)
  refetchOnWindowFocus: true,     // Refetch on focus if stale (default)
  refetchOnReconnect: true,       // Refetch on reconnect if stale (default)
  refetchInterval: 60_000,        // Poll every 60 seconds
  refetchIntervalInBackground: false, // Pause polling when tab is hidden (default)
});
```

**Component Breakdown:**
- `refetchOnMount`: Refetch when a new instance of the query mounts and data is stale.
- `refetchOnWindowFocus`: Refetch when the browser window regains focus and data is stale.
- `refetchOnReconnect`: Refetch when the network reconnects and data is stale.
- `refetchInterval`: Poll on a timer (milliseconds).
- `refetchIntervalInBackground`: Continue polling when the tab is hidden (default `false`).

**General Syntax (Disabling Refetch on Focus Per-Query):**
```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  refetchOnWindowFocus: false, // Disable for this query only
});
```

**Syntax Rules:**
- Refetching only occurs if the data is stale. If `staleTime` is set high, refetching is less frequent.
- To disable window focus refetching globally, set `refetchOnWindowFocus: false` in the `QueryClient` defaults.
- To refetch on focus regardless of staleness, set `refetchOnWindowFocus: 'always'`.
- For polling, set `refetchInterval` and optionally `refetchIntervalInBackground`.
- In v5, `refetchOnWindowFocus` and `refetchOnReconnect` default to `true`.

**Constraints and Limitations:**
- `refetchOnWindowFocus` can trigger more refetches than expected during development when switching between the browser and DevTools.
- Polling with `refetchInterval` can consume significant bandwidth and server resources; use it judiciously.
- `refetchOnReconnect` fires only when the network is reconnected; it does not fire on initial mount.
- Refetching does not cancel in-flight requests from previous renders; use `AbortController` if needed.

### Annotated Code Example: Dashboard with Polling and Focus Refetching

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchMetrics() {
  const response = await fetch('/api/metrics');
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

function Dashboard() {
  const { data, isFetching, isPending } = useQuery({
    queryKey: ['metrics'],
    queryFn: fetchMetrics,
    // Data is considered stale after 30 seconds
    staleTime: 30_000,
    // Poll every 60 seconds (only when tab is focused)
    refetchInterval: 60_000,
    // Continue polling even when tab is hidden
    refetchIntervalInBackground: false,
    // Refetch when the window regains focus (default: true)
    refetchOnWindowFocus: true,
    // Refetch when the network reconnects (default: true)
    refetchOnReconnect: true,
  });

  if (isPending) return <p>Loading metrics...</p>;

  return (
    <div>
      {/* Show a subtle indicator during background refetches */}
      {isFetching && <span>⟳</span>}
      <h2>Active Users: {data.activeUsers}</h2>
      <h2>Revenue: ${data.revenue}</h2>
    </div>
  );
}
```

**Expected Output:** The dashboard displays metrics. A ⟳ indicator appears briefly every 60 seconds during polling, when the window is refocused, or when the network reconnects. The metrics update without the screen blanking.

**Why This Output Occurs:** `staleTime: 30_000` means data is fresh for 30 seconds. After 30 seconds, it becomes stale and eligible for refetching. `refetchInterval: 60_000` triggers a poll every 60 seconds; if the data is stale, it refetches. `refetchIntervalInBackground: false` pauses polling when the tab is hidden (saving bandwidth). `refetchOnWindowFocus: true` refetches when the user returns to the tab. `refetchOnReconnect: true` refetches when the network reconnects. The `isFetching` flag is `true` during all these background refetches, showing the ⟳ indicator.

### Real-World Cases

- **Real-time dashboards:** Polling every 30–60 seconds for metrics that need to stay current.
- **Notification badges:** Refetching unread counts when the window is focused.
- **Chat applications:** Refetching messages when the window regains focus after being away.
- **Stock tickers:** Polling for price updates at frequent intervals.
- **Offline-first apps:** Refetching when the network reconnects after being offline.

### References

- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query v5 – Window Focus Refetching: https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query v5 – Polling: https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query v5 – Network Mode: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode

---

## Core Concept 4: React Query / TanStack Query Concepts (Hooks and Orchestration)

### Definitions

**Core Definition:** TanStack Query (formerly React Query) is an async state management library that orchestrates data fetching, caching, synchronization, and updates using structural mechanisms like `useQuery`, `useMutation`, and `useQueryClient` via custom hooks.

**Technical Definition:** TanStack Query v5 is the current major version, with several breaking changes from v4. It unified every hook on a single object argument (no positional overloads) and made Suspense first-class via dedicated `useSuspenseQuery` and `useSuspenseInfiniteQuery` where data is never typed as `undefined`. Callbacks on `useQuery` (`onSuccess`, `onError`, `onSettled`) have been removed; side effects should be handled in the `queryFn` or via `useEffect`. The library requires React 18+ and uses `useSyncExternalStore`. Core hooks include: `useQuery` (fetch and cache data), `useMutation` (create, update, delete data), `useQueryClient` (access the `QueryClient` for imperative operations like `invalidateQueries`), `useInfiniteQuery` (pagination and infinite scroll), and `useQueries` (parallel queries). The `QueryClient` is the core class that manages query and mutation state. A `QueryClientProvider` must wrap the app to provide the client to all hooks. Query keys must be arrays; they uniquely identify queries and are used for caching, refetching, and sharing. Query key factories (e.g., `todoKeys.detail(id)`) are recommended for large applications to avoid key collisions. Dependent queries use the `enabled` option to gate execution until a prerequisite query resolves. Paginated queries include the page number in the query key and use `placeholderData: keepPreviousData` to avoid loading flashes. Infinite queries use `useInfiniteQuery` with a `pageParam` and `getNextPageParam`.

**Beginner-Friendly Explanation:** TanStack Query is a toolbox of hooks that handle all the tedious parts of fetching data from a server. You tell it what you want (`queryKey`) and how to get it (`queryFn`), and it handles the rest: caching, loading states, error handling, background updates, and retries. It's like having a personal assistant who remembers what you've already fetched, tells you when you need fresh data, and handles all the phone calls to the server.

### Purposes

- To provide a complete, hook-based API for fetching, caching, and synchronizing server state.
- To unify all hooks on a single object argument for consistency and TypeScript inference.
- To make Suspense a first-class feature for declarative loading and error handling.
- To provide tools for pagination (`useInfiniteQuery`), parallel queries (`useQueries`), and dependent queries (`enabled`).
- To offer a `QueryClient` for imperative cache operations and global configuration.

### Syntax Rules and Structure

**General Syntax (Setup):**
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,  // Global default: 1 minute
      gcTime: 5 * 60_000, // Global default: 5 minutes
      retry: 3,
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

**Component Breakdown:**
- `QueryClient`: The core class managing query and mutation state.
- `QueryClientProvider`: Provides the client to all hooks in the tree.
- `defaultOptions`: Global defaults for all queries.

**General Syntax (Query Key Factory):**
```tsx
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
};

// Usage
useQuery({ queryKey: todoKeys.detail(5), queryFn: () => fetchTodo(5) });
```

**Component Breakdown:**
- `todoKeys.all`: The base key for all todo queries.
- `todoKeys.detail(id)`: A specific key for a single todo.
- Invalidation by prefix (`invalidateQueries({ queryKey: todoKeys.all })`) invalidates all todo queries.

**Syntax Rules:**
- Wrap the app in `<QueryClientProvider>` with a `QueryClient` instance.
- Create one `QueryClient` per app (or per request on the server).
- Use query key factories for large applications to avoid key collisions.
- Use `useSuspenseQuery` with `<Suspense>` and Error Boundaries for declarative loading/error handling.
- Use `useQueries` for parallel queries; use `enabled` for dependent queries.
- Use `placeholderData: keepPreviousData` for paginated queries to avoid loading flashes.

**Constraints and Limitations:**
- v5 requires React 18+ and uses `useSyncExternalStore`.
- Callbacks on `useQuery` have been removed; use `useEffect` or the `queryFn` for side effects.
- The `QueryClient` should not be created inside a component; create it outside or with `useState`.
- Query key factories add boilerplate but prevent key collisions in large applications.

### Annotated Code Example: Complete TanStack Query Setup

```tsx
import { QueryClient, QueryClientProvider, useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useState } from 'react';

// Step 1: Create the QueryClient outside the component
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,
      gcTime: 5 * 60_000,
      retry: 2,
    },
  },
});

// Step 2: Define query key factory
const todoKeys = {
  all: ['todos'] as const,
  list: () => [...todoKeys.all, 'list'] as const,
};

// Step 3: Query function
async function fetchTodos() {
  const res = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

// Step 4: Component using useQuery
function TodoList() {
  const { data, isPending, isError, error } = useQuery({
    queryKey: todoKeys.list(),
    queryFn: fetchTodos,
  });

  if (isPending) return <p>Loading...</p>;
  if (isError) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.map(todo => <li key={todo.id}>{todo.title}</li>)}
    </ul>
  );
}

// Step 5: Component using useMutation
function AddTodo() {
  const queryClient = useQueryClient();
  const [text, setText] = useState('');

  const mutation = useMutation({
    mutationFn: (newTodo: { title: string; completed: boolean }) =>
      fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),
    onSettled: () => {
      // Invalidate and refetch after mutation
      queryClient.invalidateQueries({ queryKey: todoKeys.all });
    },
  });

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      if (text.trim()) {
        mutation.mutate({ title: text.trim(), completed: false });
        setText('');
      }
    }}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Adding...' : 'Add Todo'}
      </button>
    </form>
  );
}

// Step 6: App with provider
export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <h1>TanStack Query Demo</h1>
      <TodoList />
      <AddTodo />
    </QueryClientProvider>
  );
}
```

**Expected Output:** A list of five todos is fetched and displayed. A form allows adding new todos. When a new todo is added, the form disables during the mutation, and after it settles, the todo list is invalidated and refetched, showing the new todo.

**Why This Output Occurs:** The `QueryClient` is created outside the component to ensure a single instance. `todoKeys.list()` provides a stable, unique key for the todo list query. `useQuery` fetches the todos and manages loading/error states. `useMutation` handles the POST request, and `onSettled` invalidates the `todoKeys.all` prefix, triggering a background refetch of the todo list. The `useQueryClient` Hook provides access to the `QueryClient` for imperative invalidation.

### Real-World Cases

- **E-commerce:** Product listings with `useQuery`, add-to-cart with `useMutation`, and cache invalidation after mutations.
- **Social media:** Feed with `useInfiniteQuery`, likes with `useMutation`, and optimistic updates.
- **Admin panels:** Data tables with pagination (`keepPreviousData`), filters, and mutations.
- **Multi-step forms:** Dependent queries with `enabled` to fetch data based on previous steps.
- **Dashboards:** Parallel queries with `useQueries` for multiple independent widgets.

### References

- TanStack Query v5 – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query v5 – Migrating to v5: https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- TanStack Query v5 – Query Keys: https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query v5 – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query v5 – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- TanStack Query v5 – Infinite Queries: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries

---

## Core Concept 5: Mutations (Server-Side Data Alterations)

### Definitions

**Core Definition:** Mutations are asynchronous network fetch sequences explicitly executed to alter data values on a database or trigger server commands, created with the `useMutation` Hook.

**Technical Definition:** Unlike queries, mutations are typically used to create, update, or delete data on the server or perform server-side effects. A mutation is created using the `useMutation` Hook, which accepts a `mutationFn` (a function that performs the asynchronous task and returns a promise) and lifecycle callbacks (`onMutate`, `onSuccess`, `onError`, `onSettled`). The Hook returns a mutation result object with states: `idle` (mutation is idle or reset), `pending` (mutation is running), `error` (mutation encountered an error), and `success` (mutation succeeded). The result also includes `mutate` (fire-and-forget trigger) and `mutateAsync` (returns a promise). The `onMutate` callback fires before the mutation function and is passed the same variables; its return value is passed to `onSuccess`, `onError`, and `onSettled` for rollback and context passing. `onSuccess` fires when the mutation is successful; `onError` fires on error; `onSettled` fires when the mutation is either successful or encounters an error. Mutations do not retry by default (`retry: 0`).

**Beginner-Friendly Explanation:** A mutation is like sending a letter to a government office to change your address. You fill out the form (the `mutationFn`), drop it in the mailbox (call `mutate`), and wait for the response. If it succeeds, you get a confirmation; if it fails, you get an error. Unlike queries (which are like reading a notice board), mutations are actions that change something on the server.

### Purposes

- To create, update, or delete data on the server.
- To perform server-side side effects (sending emails, triggering webhooks).
- To provide lifecycle callbacks for optimistic updates, cache invalidation, and error handling.
- To track the mutation's progress (`isPending`) and result (`data`, `error`).
- To support both fire-and-forget (`mutate`) and promise-based (`mutateAsync`) execution.

### Syntax Rules and Structure

**General Syntax (`useMutation`):**
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodo() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newTodo: { title: string }) =>
      fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),
    onMutate: async (newTodo) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      // Snapshot previous value
      const previousTodos = queryClient.getQueryData(['todos']);
      // Optimistically update
      queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
      // Return context for rollback
      return { previousTodos };
    },
    onError: (err, newTodo, context) => {
      // Roll back on error
      queryClient.setQueryData(['todos'], context.previousTodos);
    },
    onSettled: () => {
      // Invalidate after success or error
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: 'New Todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding...' : 'Add Todo'}
    </button>
  );
}
```

**Component Breakdown:**
- `mutationFn`: The async function performing the mutation.
- `onMutate`: Fires before the mutation; used for optimistic updates and snapshotting.
- `onError`: Fires on error; used for rollback.
- `onSettled`: Fires after success or error; used for invalidation.
- `mutate`: Triggers the mutation (fire-and-forget).
- `mutateAsync`: Triggers the mutation and returns a promise.

**Syntax Rules:**
- Use `useMutation` for create, update, and delete operations; use `useQuery` for reads.
- Always invalidate related queries in `onSettled` to keep the cache in sync.
- Use `onMutate` for optimistic updates; return context for rollback.
- Use `mutate` for fire-and-forget; use `mutateAsync` when you need the promise.
- Mutations do not retry by default; set `retry` to a number to enable retries.

**Constraints and Limitations:**
- `mutate` does not return a promise; use `mutateAsync` if you need to await the result.
- Callbacks passed to `mutate` override the Hook-level callbacks but do not replace them for `onMutate`.
- Mutations are not automatically deduplicated; multiple rapid calls result in multiple requests.
- Optimistic updates add complexity; only use them when the UX benefit is significant.

### Annotated Code Example: Add Todo with Optimistic Update

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodoOptimistic() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newTodo: { title: string }) =>
      fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        body: JSON.stringify({ title: newTodo.title, completed: false, userId: 1 }),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),

    // 1. Optimistically update the cache before the server responds
    onMutate: async (newTodo) => {
      // Cancel any outgoing refetches so they don't overwrite our optimistic update
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot the previous value
      const previousTodos = queryClient.getQueryData(['todos']);

      // Optimistically add the new todo to the cache
      queryClient.setQueryData(['todos'], (old: any) => [
        ...(old || []),
        { id: Date.now(), title: newTodo.title, completed: false },
      ]);

      // Return a context object with the snapshotted value
      return { previousTodos };
    },

    // 2. If the mutation fails, roll back to the previous value
    onError: (err, newTodo, context) => {
      queryClient.setQueryData(['todos'], context?.previousTodos);
    },

    // 3. Always refetch after error or success to ensure cache consistency
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: 'Optimistic Todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding...' : 'Add Todo (Optimistic)'}
    </button>
  );
}
```

**Expected Output:** Clicking "Add Todo (Optimistic)" immediately adds a new todo to the list (with reduced opacity if using the UI variant). The mutation runs in the background. If it succeeds, the todo remains and is confirmed by the refetch. If it fails, the optimistic todo disappears and the cache rolls back to the previous state.

**Why This Output Occurs:** `onMutate` fires before the mutation function. It cancels outgoing refetches, snapshots the previous todos, and optimistically adds the new todo to the cache. The `context` object containing `previousTodos` is returned. If the mutation fails, `onError` fires and restores the previous todos from the context. `onSettled` always fires after success or error and invalidates the `['todos']` query, triggering a background refetch to ensure the cache matches the server.

### Real-World Cases

- **E-commerce:** Adding items to cart, placing orders, updating quantities.
- **Social media:** Creating posts, liking, commenting, following.
- **Admin panels:** Creating, updating, and deleting records.
- **Form submissions:** Submitting forms with optimistic updates for instant feedback.
- **Settings:** Updating user preferences with immediate UI feedback.

### References

- TanStack Query v5 – Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- TanStack Query v5 – useMutation Reference: https://tanstack.com/query/latest/docs/framework/react/reference/useMutation
- TanStack Query v5 – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query v5 – Invalidations from Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/invalidations-from-mutations

---

## Core Concept 6: Cache Invalidation (Targeted Staleness and Refetching)

### Definitions

**Core Definition:** Cache invalidation is the process of flagging cached data items as intentionally out of date to force an immediate background network trip to fetch fresh data, performed via `queryClient.invalidateQueries`.

**Technical Definition:** The `QueryClient` has an `invalidateQueries` method that intelligently marks queries as stale and potentially refetches them. When a query is invalidated, two things happen: it is marked as stale (overriding any `staleTime` configuration), and if the query is currently being rendered via `useQuery` or related hooks, it is refetched in the background. TanStack Query prescribes **targeted invalidation, background-refetching, and ultimately atomic updates**, avoiding the manual labour of maintaining normalized caches. Query matching supports prefix matching by default (`exact: false`), so invalidating `['todos']` also invalidates `['todos', { page: 1 }]`, `['todos', { type: 'done' }]`, and any other queries starting with `todos`. To invalidate only the exact base query, pass `exact: true`. A predicate function can be used for even more granular matching. Active queries (currently rendered) are refetched in the background; inactive queries are marked stale and will refetch on the next mount. `refetchQueries` can be used to force an immediate refetch regardless of staleness.

**Beginner-Friendly Explanation:** Cache invalidation is like telling the librarian, "That book on the shelf is outdated—please get the latest edition." The librarian marks the book as outdated and, if someone is currently reading it, quietly fetches the new edition in the background. When the new edition arrives, the reader's copy is updated seamlessly. If nobody is reading it, the outdated book stays on the shelf until someone asks for it again.

### Purposes

- To mark cached data as stale after a mutation, ensuring the UI reflects the latest server state.
- To trigger background refetches of active queries without blanking the screen.
- To support targeted invalidation by prefix, exact key, or predicate function.
- To avoid the manual labour of maintaining normalized caches.
- To ensure cache consistency between the client and the server.

### Syntax Rules and Structure

**General Syntax (`invalidateQueries`):**
```tsx
import { useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();

// Invalidate every query in the cache
queryClient.invalidateQueries();

// Invalidate every query with a key that starts with `todos`
queryClient.invalidateQueries({ queryKey: ['todos'] });

// Invalidate only the exact `todos` query (no subkeys)
queryClient.invalidateQueries({ queryKey: ['todos'], exact: true });

// Invalidate queries matching a predicate
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === 'todos' && query.queryKey[1]?.version >= 2,
});
```

**Component Breakdown:**
- `queryKey`: The key or prefix to invalidate.
- `exact: true`: Only invalidate the exact key, not subkeys.
- `predicate`: A function that returns `true` for queries to invalidate.

**General Syntax (Invalidation from Mutations):**
```tsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

**Component Breakdown:**
- `onSettled`: Fires after success or error; ideal for invalidation.
- `queryClient.invalidateQueries`: Marks the query as stale and refetches if active.

**Syntax Rules:**
- Always invalidate related queries after a mutation in `onSettled` (not just `onSuccess`) to handle both success and error cases.
- Use prefix matching to invalidate all related queries; use `exact: true` for precision.
- Use predicate functions for complex matching logic.
- `invalidateQueries` returns a promise; you can await it if needed.
- Inactive queries are marked stale and will refetch on the next mount; active queries are refetched immediately.

**Constraints and Limitations:**
- `invalidateQueries` triggers a refetch only if the query is active (rendered). Inactive queries are marked stale but not refetched.
- Prefix matching can invalidate more queries than intended; use `exact: true` or predicates for precision.
- Invalidation does not cancel in-flight requests; use `cancelQueries` before invalidation if needed.
- Over-invalidating (e.g., invalidating the entire cache after every mutation) can cause excessive network requests.

### Annotated Code Example: Invalidation After Add and Update Mutations

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function TodoActions() {
  const queryClient = useQueryClient();

  // Mutation 1: Add a todo
  const addMutation = useMutation({
    mutationFn: (title: string) =>
      fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify({ title }),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),
    onSettled: () => {
      // Invalidate all todo queries — the list and any detail queries
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  // Mutation 2: Update a single todo
  const updateMutation = useMutation({
    mutationFn: ({ id, title }: { id: number; title: string }) =>
      fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        body: JSON.stringify({ title }),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),
    onSettled: (data, error, variables) => {
      // Invalidate only the specific todo's detail query
      queryClient.invalidateQueries({ queryKey: ['todos', variables.id] });
      // Also invalidate the list to reflect the updated title
      queryClient.invalidateQueries({ queryKey: ['todos', 'list'] });
    },
  });

  return (
    <div>
      <button onClick={() => addMutation.mutate('New Todo')}>
        Add Todo
      </button>
      <button onClick={() => updateMutation.mutate({ id: 1, title: 'Updated' })}>
        Update Todo 1
      </button>
    </div>
  );
}
```

**Expected Output:** Clicking "Add Todo" adds a new todo and invalidates the entire `todos` namespace, refetching the list and any detail queries. Clicking "Update Todo 1" updates the specific todo and invalidates only `['todos', 1]` and `['todos', 'list']`, refetching those specific queries.

**Why This Output Occurs:** `invalidateQueries({ queryKey: ['todos'] })` uses prefix matching to invalidate `['todos']`, `['todos', 1]`, `['todos', 'list']`, and any other query starting with `todos`. `invalidateQueries({ queryKey: ['todos', variables.id] })` invalidates only the specific todo's detail query. `invalidateQueries({ queryKey: ['todos', 'list'] })` invalidates the list query to reflect the updated title. This targeted approach minimises unnecessary network requests.

### Real-World Cases

- **E-commerce:** Invalidating product listings after adding a product to the cart.
- **Social media:** Invalidating the feed after creating a post.
- **Admin panels:** Invalidating table data after creating, updating, or deleting a record.
- **Dashboard:** Invalidating metrics after a settings change.
- **Collaborative apps:** Invalidating shared documents after a save.

### References

- TanStack Query v5 – Query Invalidation: https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query v5 – Invalidations from Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/invalidations-from-mutations
- TanStack Query v5 – Query Filters: https://tanstack.com/query/latest/docs/framework/react/guides/filters

---

## Core Concept 7: Optimistic Updates (Instant UI Feedback with Rollback)

### Definitions

**Core Definition:** Optimistic updates are the practice of immediately modifying local application UI state to simulate a successful API mutation before the server has technically sent back a confirmation response, with automatic rollback if the mutation fails.

**Technical Definition:** React Query provides two ways to optimistically update the UI before a mutation has completed. The first is **via the UI**: leverage the returned `variables` from `useMutation` to render a temporary item while the mutation is pending. This is the simpler variant, as it does not interact with the cache directly; the temporary item disappears when the mutation completes, and the real item appears after the refetch. The second is **via the cache**: use the `onMutate` option to update the cache directly. When optimistically updating the cache, there is a chance the mutation will fail; in most failure cases, you can trigger a refetch to revert to the true server state, but in some cases refetching may not work, and you must roll back manually. The `onMutate` handler allows you to return a value (typically a rollback function or snapshotted data) that is passed to both `onError` and `onSettled` handlers. The standard pattern is: cancel outgoing refetches in `onMutate`, snapshot the previous value, optimistically update the cache, return the snapshot for rollback, restore the snapshot in `onError`, and always refetch in `onSettled`.

**Beginner-Friendly Explanation:** Imagine you are ordering a coffee. An optimistic update is like the barista immediately handing you a cup and saying, "Here you go!" before the coffee is actually made. If the espresso machine breaks, they take the cup back and apologise. In the meantime, you felt like you were served instantly. This is the power of optimistic updates: instant feedback with a safety net if something goes wrong.

### Purposes

- To provide instant UI feedback before the server confirms the mutation, improving perceived performance.
- To eliminate the perceived lag between a user action and the UI response.
- To handle mutation failures gracefully by rolling back to the previous state.
- To support concurrent optimistic updates with `useMutationState`.
- To create a "snappy" user experience for common actions like liking, adding to cart, or toggling.

### Syntax Rules and Structure

**Pattern 1: Optimistic Update via the UI (Simple)**
```tsx
const mutation = useMutation({
  mutationFn: (newTodo: string) => axios.post('/api/data', { text: newTodo }),
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
});

// In the component:
<ul>
  {todoQuery.items.map(todo => <li key={todo.id}>{todo.text}</li>)}
  {mutation.isPending && (
    <li style={{ opacity: 0.5 }}>{mutation.variables}</li>
  )}
</ul>
```

**Component Breakdown:**
- `mutation.variables`: The variables passed to `mutate`.
- `mutation.isPending`: `true` while the mutation is running.
- A temporary item is rendered with reduced opacity; it disappears when the mutation completes.

**Pattern 2: Optimistic Update via the Cache (Full Control)**
```tsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // 1. Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    // 2. Snapshot previous value
    const previousTodos = queryClient.getQueryData(['todos']);
    // 3. Optimistically update the cache
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
    // 4. Return context for rollback
    return { previousTodos };
  },
  onError: (err, newTodo, context) => {
    // 5. Roll back on error
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
  onSettled: () => {
    // 6. Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

**Component Breakdown:**
- `onMutate`: Cancels refetches, snapshots previous value, optimistically updates, returns context.
- `onError`: Restores the snapshot from context on failure.
- `onSettled`: Invalidates the query to refetch fresh data.

**Syntax Rules:**
- Always cancel outgoing refetches in `onMutate` to prevent them from overwriting the optimistic update.
- Always snapshot the previous value and return it as context for rollback.
- Always roll back in `onError` using the context.
- Always invalidate in `onSettled` to ensure cache consistency.
- Use `useMutationState` for cross-component optimistic updates with a `mutationKey`.
- Use `submittedAt` as a unique key for concurrent optimistic updates.

**Constraints and Limitations:**
- Optimistic updates add complexity; only use them when the UX benefit is significant.
- The mutation may fail for reasons that cannot be resolved by refetching; manual rollback is required.
- Concurrent optimistic updates require careful key management (`submittedAt`).
- The UI variant is simpler but less accurate for complex cache manipulations.

### Annotated Code Example: Optimistic Like Button

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function LikeButton({ postId, initialLikes, initialLiked }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (liked: boolean) =>
      fetch(`/api/posts/${postId}/like`, {
        method: liked ? 'DELETE' : 'POST',
      }).then(res => res.json()),

    onMutate: async (liked) => {
      // Cancel any outgoing refetches for this post
      await queryClient.cancelQueries({ queryKey: ['post', postId] });

      // Snapshot the previous post data
      const previousPost = queryClient.getQueryData(['post', postId]);

      // Optimistically update the like count and liked status
      queryClient.setQueryData(['post', postId], (old: any) => ({
        ...old,
        likes: liked ? old.likes - 1 : old.likes + 1,
        liked: !liked,
      }));

      return { previousPost };
    },

    onError: (err, liked, context) => {
      // Roll back to the previous post on error
      queryClient.setQueryData(['post', postId], context?.previousPost);
    },

    onSettled: () => {
      // Refetch to ensure consistency with the server
      queryClient.invalidateQueries({ queryKey: ['post', postId] });
    },
  });

  const post = queryClient.getQueryData(['post', postId]) as any;
  const liked = post?.liked ?? initialLiked;
  const likes = post?.likes ?? initialLikes;

  return (
    <button
      onClick={() => mutation.mutate(liked)}
      disabled={mutation.isPending}
    >
      {liked ? '❤️' : '🤍'} {likes}
    </button>
  );
}
```

**Expected Output:** Clicking the heart immediately toggles the like state and updates the count (optimistic update). If the server confirms, the state remains. If the request fails, the heart and count revert to their previous state. A refetch occurs after the mutation settles to ensure consistency.

**Why This Output Occurs:** `onMutate` cancels refetches, snapshots the previous post, and optimistically updates the like count and liked status. If the mutation fails, `onError` restores the snapshot. `onSettled` invalidates the post query, triggering a background refetch to confirm the final state with the server. The button reads from the cache, so it reflects the optimistic state immediately.

### Real-World Cases

- **Social media:** Liking, following, bookmarking.
- **E-commerce:** Adding to cart, updating quantities.
- **Task management:** Toggling todo completion, reordering items.
- **Chat:** Sending messages with optimistic rendering.
- **Settings:** Toggling preferences with instant feedback.

### References

- TanStack Query v5 – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query v5 – useMutation Reference: https://tanstack.com/query/latest/docs/framework/react/reference/useMutation
- TanStack Query v5 – useMutationState: https://tanstack.com/query/latest/docs/framework/react/reference/useMutationState

---

## Core Concept 8: Background Refetching (Focus, Reconnect, and Polling)

### Definitions

**Core Definition:** Background refetching is the quiet fetching of updated data from an API whenever a browser tab regains focus, network connections reconnect, or a polling interval elapses, ensuring data accuracy without disrupting the user experience.

**Technical Definition:** TanStack Query's background refetching is controlled by several options. **Window focus refetching** (`refetchOnWindowFocus`, default `true`): if a user leaves the application and returns, and the query data is stale, TanStack Query automatically requests fresh data in the background. **Reconnect refetching** (`refetchOnReconnect`, default `true`): if the network disconnects and reconnects, stale queries are refetched. **Mount refetching** (`refetchOnMount`, default `true`): if a new instance of the query mounts and data is stale, it refetches. **Polling** (`refetchInterval`): makes a query refetch on a timer; by default, polling pauses when the browser tab loses focus. `refetchIntervalInBackground` (default `false`) can be set to `true` to continue polling even when the tab is hidden. The `isFetching` flag is `true` during any background refetch, allowing the UI to show a subtle "updating" indicator without blanking the screen. The `refetchOnWindowFocus` option can be set to `'always'` to refetch on focus regardless of staleness. In network mode `'offlineFirst'`, `refetchOnReconnect` defaults to `false` because reconnecting is not a good indicator that stale queries should refetch.

**Beginner-Friendly Explanation:** Background refetching is like a news app that automatically refreshes when you open it, when your internet comes back, or every few minutes. You don't have to pull-to-refresh—the app quietly fetches the latest headlines while you're reading the old ones. When the fresh data arrives, the screen updates seamlessly. If you're on a plane with no internet, the app shows you the last downloaded articles until you reconnect.

### Purposes

- To ensure the UI displays the most current data without requiring manual refreshes.
- To automatically recover from network interruptions by refetching when connectivity is restored.
- To keep data fresh when the user returns to the tab after being away.
- To support polling for dashboards and real-time data that must stay current.
- To avoid unnecessary refetches by only refetching stale data.

### Syntax Rules and Structure

**General Syntax (Refetch Options):**
```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 30_000,              // Consider fresh for 30 seconds
  refetchOnMount: true,           // Refetch on mount if stale (default)
  refetchOnWindowFocus: true,     // Refetch on focus if stale (default)
  refetchOnReconnect: true,       // Refetch on reconnect if stale (default)
  refetchInterval: 60_000,        // Poll every 60 seconds
  refetchIntervalInBackground: false, // Pause polling when tab is hidden (default)
});
```

**Component Breakdown:**
- `refetchOnMount`: Refetch when a new instance mounts and data is stale.
- `refetchOnWindowFocus`: Refetch when the browser window regains focus and data is stale.
- `refetchOnReconnect`: Refetch when the network reconnects and data is stale.
- `refetchInterval`: Poll on a timer (milliseconds).
- `refetchIntervalInBackground`: Continue polling when the tab is hidden.

**General Syntax (Disabling Refetch on Focus Globally):**
```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false, // Disable globally
    },
  },
});
```

**Syntax Rules:**
- Refetching only occurs if the data is stale. Raise `staleTime` to reduce refetch frequency.
- To refetch on focus regardless of staleness, set `refetchOnWindowFocus: 'always'`.
- To disable window focus refetching globally, set it in `QueryClient` defaults.
- For polling, set `refetchInterval` and optionally `refetchIntervalInBackground`.
- In network mode `'offlineFirst'`, `refetchOnReconnect` defaults to `false`.

**Constraints and Limitations:**
- `refetchOnWindowFocus` can trigger more refetches than expected during development when switching between the browser and DevTools.
- Polling with `refetchInterval` can consume significant bandwidth and server resources.
- `refetchOnReconnect` fires only when the network is reconnected; it does not fire on initial mount.
- Refetching does not cancel in-flight requests from previous renders; use `AbortController` if needed.

### Annotated Code Example: Dashboard with Polling and Focus Refetching

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchMetrics() {
  const response = await fetch('/api/metrics');
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

function Dashboard() {
  const { data, isFetching, isPending } = useQuery({
    queryKey: ['metrics'],
    queryFn: fetchMetrics,
    staleTime: 30_000,             // Fresh for 30 seconds
    refetchInterval: 60_000,       // Poll every 60 seconds
    refetchIntervalInBackground: false, // Pause polling when tab is hidden
    refetchOnWindowFocus: true,    // Refetch on focus (default)
    refetchOnReconnect: true,      // Refetch on reconnect (default)
  });

  if (isPending) return <p>Loading metrics...</p>;

  return (
    <div>
      {isFetching && <span>⟳</span>}
      <h2>Active Users: {data.activeUsers}</h2>
      <h2>Revenue: ${data.revenue}</h2>
    </div>
  );
}
```

**Expected Output:** The dashboard displays metrics. A ⟳ indicator appears briefly every 60 seconds during polling, when the window is refocused, or when the network reconnects. The metrics update without the screen blanking.

**Why This Output Occurs:** `staleTime: 30_000` means data is fresh for 30 seconds. After 30 seconds, it becomes stale and eligible for refetching. `refetchInterval: 60_000` triggers a poll every 60 seconds; if the data is stale, it refetches. `refetchIntervalInBackground: false` pauses polling when the tab is hidden. `refetchOnWindowFocus: true` refetches when the user returns to the tab. `refetchOnReconnect: true` refetches when the network reconnects. The `isFetching` flag is `true` during all these background refetches, showing the ⟳ indicator.

### Real-World Cases

- **Real-time dashboards:** Polling every 30–60 seconds for metrics that need to stay current.
- **Notification badges:** Refetching unread counts when the window is focused.
- **Chat applications:** Refetching messages when the window regains focus after being away.
- **Stock tickers:** Polling for price updates at frequent intervals.
- **Offline-first apps:** Refetching when the network reconnects after being offline.

### References

- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query v5 – Window Focus Refetching: https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query v5 – Polling: https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query v5 – Network Mode: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode

---

## Comparison and Decision Guidance

| Concept | Primary Hook/API | Key Options | Default Behaviour |
|---|---|---|---|
| **Query State** | `useQuery` | `queryKey`, `queryFn` | `isPending` → `isError` / `isSuccess` |
| **Cache State** | `useQuery` | `staleTime`, `gcTime` | `staleTime: 0`, `gcTime: 5 min` |
| **Synchronization** | `useQuery` | `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect` | All `true` |
| **TanStack Query Setup** | `QueryClient`, `QueryClientProvider` | `defaultOptions` | Global defaults for all queries |
| **Mutations** | `useMutation` | `mutationFn`, `onMutate`, `onError`, `onSettled` | `retry: 0` |
| **Cache Invalidation** | `queryClient.invalidateQueries` | `queryKey`, `exact`, `predicate` | Prefix matching (`exact: false`) |
| **Optimistic Updates** | `useMutation` | `onMutate` (snapshot + return context) | Rollback on error in `onError` |
| **Background Refetching** | `useQuery` | `refetchInterval`, `refetchIntervalInBackground` | Polling pauses when tab hidden |

**Decision Guidance:**
- **Start with `useQuery`** for all data fetching; it handles loading, error, and success states automatically.
- **Set `staleTime` deliberately** (not the default `0`) based on how frequently the data changes.
- **Use `useMutation` for all writes** (create, update, delete) and always invalidate related queries in `onSettled`.
- **Use optimistic updates** for actions where instant feedback is critical (likes, toggles, cart adds).
- **Use `refetchInterval` for polling** dashboards and real-time data; disable `refetchIntervalInBackground` to save bandwidth.
- **Use query key factories** for large applications to avoid key collisions and enable targeted invalidation.
- **Never copy server data into a client store**; TanStack Query is the single source of truth for API data.

---

## References

- TanStack Query v5 – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query v5 – Queries: https://tanstack.com/query/latest/docs/framework/react/guides/queries
- TanStack Query v5 – Caching: https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query v5 – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query v5 – Mutations: https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- TanStack Query v5 – Query Invalidation: https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query v5 – Optimistic Updates: https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query v5 – Window Focus Refetching: https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query v5 – Polling: https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query v5 – Network Mode: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode
- TanStack Query v5 – Query Keys: https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query v5 – Dependent Queries: https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries
- TanStack Query v5 – Paginated Queries: https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries
- TanStack Query v5 – Infinite Queries: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
- TanStack Query v5 – useQuery Reference: https://tanstack.com/query/latest/docs/framework/react/reference/useQuery
- TanStack Query v5 – useMutation Reference: https://tanstack.com/query/latest/docs/framework/react/reference/useMutation
- TanStack Query v5 – useSuspenseQuery: https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseQuery
- TanStack Query v5 – Suspense: https://tanstack.com/query/latest/docs/framework/react/guides/suspense
- TanStack Query v5 – Migrating to v5: https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- TanStack Query v5 – persistQueryClient: https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient
- TanStack Query v5 – createSyncStoragePersister: https://tanstack.com/query/latest/docs/framework/react/plugins/createSyncStoragePersister
- TanStack Query v5 – Query Filters: https://tanstack.com/query/latest/docs/framework/react/guides/filters