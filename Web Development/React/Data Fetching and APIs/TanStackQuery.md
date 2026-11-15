# TanStack Query (React Query) — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** TanStack Query (formerly React Query) is a powerful asynchronous state management library for React that handles fetching, caching, synchronizing, and updating server state.

**Technical Definition:** TanStack Query is a server-state management library that provides hooks (`useQuery`, `useMutation`, `useInfiniteQuery`, `useQueries`) for declarative data fetching, caching, and synchronization. It maintains a `QueryClient` cache that stores query results keyed by serializable query keys. Queries go through a well-defined lifecycle (pending → success/error → stale → inactive → garbage-collected) and support background refetching, window focus refetching, network reconnect refetching, and polling. Mutations (`useMutation`) handle server-side effects with lifecycle callbacks (`onMutate`, `onSuccess`, `onError`, `onSettled`) and integrate with the cache for optimistic updates and invalidation. The library deduplicates concurrent requests by query key, supports parallel queries (`useQueries`) and infinite queries (`useInfiniteQuery`), and provides devtools for inspection.

**Beginner-Friendly Explanation:** TanStack Query is a tool that makes fetching data from a server much easier. Instead of writing `useEffect` with `fetch`, loading states, error states, and caching logic yourself, you call `useQuery` with a unique key and a function that fetches your data. TanStack Query handles the rest: it caches the result, shows you when it is loading or errored, refetches in the background when needed, and lets you update the cache after mutations. It treats "server state" — data that lives on a server and can be stale — as a first-class citizen, separate from your app's local UI state.

### Key Characteristics

- **Server-state focused:** Designed specifically for asynchronous server state, not local UI state.
- **Declarative hooks:** `useQuery`, `useMutation`, `useInfiniteQuery`, `useQueries` provide a clean API.
- **Query-key-based caching:** Every query is uniquely identified by a serializable query key array.
- **Automatic caching and deduplication:** Concurrent requests for the same key share one network request and one cache entry.
- **Background synchronization:** Refetches on window focus, network reconnect, and configurable intervals.
- **Mutation lifecycle:** `useMutation` handles create/update/delete with `onMutate`, `onSuccess`, `onError`, and `onSettled` callbacks.
- **Optimistic updates with rollback:** Apply changes before the server confirms and roll back on failure.
- **Infinite query support:** `useInfiniteQuery` handles pagination and infinite scrolling with cursor-based pagination.
- **Devtools:** Visual inspector for queries and mutations in development.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- JavaScript Promises and `async`/`await`.
- HTTP fundamentals: methods, status codes, JSON.
- Basic understanding of caching and stale-while-revalidate concepts.
- Familiarity with `fetch` or Axios (query functions typically use them).

### Related Programming Areas

- **Server State Management** — the core problem TanStack Query solves.
- **Data Fetching** — query functions and request lifecycle.
- **Caching and Invalidation** — `staleTime`, `gcTime`, `invalidateQueries`.
- **Mutations** — `useMutation` and cache updates.
- **Pagination and Infinite Scrolling** — `useInfiniteQuery`.
- **Performance** — deduplication, prefetching, and background refetching.

---

## 1. `useQuery`

### Definitions

**Core Definition:** `useQuery` is the primary hook for fetching and caching data from a server.

**Technical Definition:** `useQuery({ queryKey, queryFn, ...options })` returns an object containing `data`, `status`, `fetchStatus`, `isPending`, `isError`, `isFetching`, `error`, `refetch`, and other properties. It subscribes the component to a query identified by `queryKey`. If cached data exists and is fresh (within `staleTime`), it is returned immediately without a network request. If the data is stale or missing, `queryFn` is executed. The hook re-renders the component when the query state changes.

**Beginner-Friendly Explanation:** `useQuery` is how you get data from the server and keep it cached. You give it a unique name (the query key) and a function that fetches the data. It gives you back the data, tells you if it is loading or errored, and automatically refreshes when needed.

### Purposes

- To fetch and cache server data with minimal boilerplate.
- To share cached data across multiple components via a shared query key.
- To automatically refetch stale data on mount, window focus, and reconnect.
- To provide consistent loading, error, and success states.
- To enable background synchronization without blocking the UI.

### Syntax Rules and Structure

**General Syntax:**

```tsx
const result = useQuery({
  queryKey: ['resource', id],
  queryFn: ({ signal }) => fetchResource(id, { signal }),
  staleTime: 60_000,
  gcTime: 5 * 60_000,
  enabled: !!id,
});
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `queryKey` | Array that uniquely identifies the query; passed to `queryFn` via context |
| `queryFn` | Async function that returns the data or throws an error |
| `staleTime` | How long data is considered fresh (default `0`) |
| `gcTime` | How long inactive data stays in memory (default 5 minutes) |
| `enabled` | Set to `false` to disable automatic fetching |
| `refetchOnWindowFocus` | Refetch when the window regains focus (default `true`) |
| `refetchOnReconnect` | Refetch when the network reconnects (default `true`) |
| `select` | Transform or select part of the data |

**Syntax Rules:**

1. `queryKey` must be an array and must include every parameter that affects the result.
2. `queryFn` must return a Promise that resolves to data or throws an error.
3. `staleTime` defaults to `0` (data is immediately stale).
4. `gcTime` defaults to 5 minutes; set to `Infinity` to disable garbage collection.
5. Use `enabled` for dependent queries where the key is not ready yet.
6. The `signal` passed to `queryFn` can be used to cancel the request.

**Constraints and Limitations:**

- Data must not be `undefined`; return `null` for empty results.
- `queryFn` must throw on HTTP errors if using `fetch` (which does not throw by default).
- Query keys are hashed deterministically; object keys are sorted.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Fetching with Loading and Error States

```tsx
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isPending, isError, error, refetch } = useQuery({
    queryKey: ['user', userId],
    queryFn: ({ signal }) =>
      fetch(`/api/users/${userId}`, { signal }).then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      }),
    staleTime: 5 * 60 * 1000,
  });

  if (isPending) return <ProfileSkeleton />;
  if (isError) return <ErrorBanner message={error.message} onRetry={refetch} />;
  if (!data) return <EmptyState />;
  return <ProfileCard user={data} />;
}
```

**Expected Output:** A skeleton while loading, an error banner with retry on failure, an empty state if no data, and the profile card on success.

**Why this output:** `useQuery` manages the entire request lifecycle. `isPending` is true only when there is no cached data. `isError` and `error` provide failure information. `refetch` allows manual retry.

#### Example 2: Dependent Query with `enabled`

```tsx
function UserPosts({ userId }) {
  const { data: posts } = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId),
    enabled: Boolean(userId), // Only fetch when userId exists
  });

  if (!userId) return <p>Select a user to see posts.</p>;
  return <PostList posts={posts ?? []} />;
}
```

**Expected Output:** The query does not run until `userId` is truthy. No request is made for an empty user ID.

**Why this output:** `enabled: false` disables automatic fetching. The query remains in a `pending` state without executing `queryFn` until the dependency is ready.

### Real-World Cases

- **User profiles:** Fetching user data by ID.
- **Dashboards:** Loading stats and widgets.
- **Detail pages:** Loading a resource by route parameter.

### References

- TanStack Query — useQuery — https://tanstack.dev/query/v4/docs/framework/react/reference/useQuery
- TanStack Query — Query Functions — https://tanstack.com/query/v4/docs/framework/react/guides/query-functions
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults

---

## 2. `useMutation`

### Definitions

**Core Definition:** `useMutation` is the hook for creating, updating, or deleting server data and performing server-side effects.

**Technical Definition:** `useMutation({ mutationFn, onMutate, onSuccess, onError, onSettled, ... })` returns an object with `mutate`, `mutateAsync`, `data`, `error`, `isPending`, `isError`, `isSuccess`, `variables`, `reset`, and other properties. Unlike queries, mutations are imperative: they run when you call `mutate` or `mutateAsync`. The lifecycle callbacks receive the variables and context returned from `onMutate`, enabling optimistic updates and rollback.

**Beginner-Friendly Explanation:** `useMutation` is how you change data on the server — creating, updating, or deleting. You call `mutate` with the data, and the hook tracks whether it is pending, succeeded, or failed.

### Purposes

- To perform create, update, and delete operations.
- To track mutation status (pending, success, error).
- To run side effects on mutation lifecycle events.
- To implement optimistic updates with rollback.
- To invalidate and refetch queries after mutations.

### Syntax Rules and Structure

**General Syntax:**

```tsx
const mutation = useMutation({
  mutationFn: (variables) => apiCall(variables),
  onSuccess: (data, variables) => {
    queryClient.invalidateQueries({ queryKey: ['resource'] });
  },
});

mutation.mutate({ name: 'New item' });
```

**Syntax Rules:**

1. `mutationFn` is required and must return a Promise.
2. `mutate` is fire-and-forget; `mutateAsync` returns a Promise.
3. Callbacks can be passed at the hook level or per `mutate` call.
4. `onMutate` runs before `mutationFn`; its return value is passed to `onError` and `onSettled` as `context`.
5. `onSettled` runs after success or error.
6. `variables` contains the data passed to `mutate`.

**Constraints and Limitations:**

- Mutations are not cached; each call executes the mutation function.
- `onMutate` is the place to cancel outgoing refetches before optimistic updates.

### Annotated Code Examples

```tsx
function AddTodo() {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: (newTodo) => axios.post('/api/todos', newTodo),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ text: 'New todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding…' : 'Add todo'}
    </button>
  );
}
```

**Expected Output:** The button disables while the mutation is pending, and the todo list refetches on success.

**Why this output:** `useMutation` provides `isPending` and `mutate`. `onSuccess` invalidates the `todos` query, triggering a refetch that updates the list.

### Real-World Cases

- **Form submissions:** Creating and updating records.
- **Delete actions:** Removing items with confirmation.
- **Settings changes:** Saving preferences.

### References

- TanStack Query — useMutation — https://tanstack.dev/query/latest/docs/framework/react/reference/useMutation
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations

---

## 3. `useInfiniteQuery`

### Definitions

**Core Definition:** `useInfiniteQuery` is a hook for fetching paginated or infinite-scroll data, where each "page" is fetched and cached incrementally.

**Technical Definition:** `useInfiniteQuery({ queryKey, queryFn, initialPageParam, getNextPageParam, ... })` returns an object with `data.pages` (an array of page results), `data.pageParams`, `fetchNextPage`, `fetchPreviousPage`, `hasNextPage`, `hasPreviousPage`, `isFetchingNextPage`, and other properties. The `queryFn` receives `{ pageParam }` and returns a page of data. `getNextPageParam` receives the last page and all pages, and returns the next page parameter or `undefined`/`null` to indicate no more pages.

**Beginner-Friendly Explanation:** `useInfiniteQuery` is like `useQuery`, but for lists that load more data as you scroll. You provide a function that tells it how to get the next page, and it handles the rest.

### Purposes

- To implement infinite scrolling and "load more" pagination.
- To cache each page independently for instant back-navigation.
- To track whether more pages are available.
- To distinguish between initial loading and loading more.

### Syntax Rules and Structure

```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
  useInfiniteQuery({
    queryKey: ['projects'],
    queryFn: ({ pageParam }) => fetchProjects({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
    maxPages: 10,
  });
```

**Syntax Rules:**

1. `initialPageParam` is required in v5.
2. `getNextPageParam` must return the next cursor or `undefined`/`null` when exhausted.
3. `data.pages` is an array of page results; flatten it for rendering.
4. `maxPages` bounds memory for very long lists.
5. Use `hasNextPage` and `isFetchingNextPage` to guard `fetchNextPage` calls.

**Constraints and Limitations:**

- Cursor-based pagination is preferred over offset-based.
- Infinite scrolling can harm accessibility; provide a "Load more" button fallback.

### Annotated Code Examples

```tsx
function Projects() {
  const {
    data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending,
  } = useInfiniteQuery({
    queryKey: ['projects'],
    queryFn: ({ pageParam }) =>
      fetch(`/api/projects?cursor=${pageParam}`).then((r) => r.json()),
    initialPageParam: 0,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });

  if (isPending) return <Skeleton />;

  return (
    <div>
      {data.pages.flatMap((page) =>
        page.data.map((project) => (
          <ProjectCard key={project.id} project={project} />
        ))
      )}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage ? 'Loading more…' : hasNextPage ? 'Load more' : 'Nothing more to load'}
      </button>
    </div>
  );
}
```

**Expected Output:** The first page loads, and the "Load more" button fetches the next page until `hasNextPage` is false.

**Why this output:** `useInfiniteQuery` manages the pages array. `fetchNextPage` appends the next page. `hasNextPage` is derived from `getNextPageParam`.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Product listings:** Load more products.
- **Chat history:** Loading older messages.

### References

- TanStack Query — Infinite Queries — https://tanstack.dev/query/v5/docs/framework/react/guides/infinite-queries
- TanStack Query — useInfiniteQuery — https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery

---

## 4. `useQueries`

### Definitions

**Core Definition:** `useQueries` is a hook for fetching a variable number of queries in parallel.

**Technical Definition:** `useQueries({ queries: [queryOptions1, queryOptions2, ...] })` returns an array of query results in the same order as the input array. Each query option object is identical to `useQuery` options. This is useful when the number of queries is dynamic (e.g., fetching multiple IDs) and hooks cannot be called in a loop.

**Beginner-Friendly Explanation:** `useQueries` lets you fetch multiple independent pieces of data at once, even when you do not know in advance how many there will be.

### Purposes

- To fetch a dynamic number of queries in parallel.
- To avoid violating the Rules of Hooks when queries depend on an array.
- To combine multiple query results into one component.
- To simplify dashboards with multiple independent data sources.

### Syntax Rules and Structure

```tsx
const results = useQueries({
  queries: userIds.map((id) => ({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
    staleTime: 5 * 60 * 1000,
  })),
});
```

**Syntax Rules:**

1. `queries` is an array of `useQuery` option objects.
2. The return value is an array of query results in input order.
3. All queries run in parallel.
4. Use `combine` (v5) to merge results into a single value if needed.

### Annotated Code Examples

```tsx
function UserDetails({ userIds }) {
  const results = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),
    })),
  });

  const isLoading = results.some((r) => r.isPending);
  const hasError = results.some((r) => r.isError);

  if (isLoading) return <Skeleton />;
  if (hasError) return <ErrorBanner />;

  return (
    <ul>
      {results.map((r) => (
        <li key={r.data.id}>{r.data.name}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** All user queries run in parallel. The component waits for all to finish and renders the list.

**Why this output:** `useQueries` creates a query for each ID and returns their results as an array. The component aggregates the loading and error states.

### Real-World Cases

- **Dashboards:** Fetching stats for multiple IDs.
- **Compare views:** Loading multiple products side by side.
- **Batch lookups:** Fetching several resources at once.

### References

- TanStack Query — useQueries — https://tanstack.com/query/v4/docs/framework/react/reference/useQueries
- TanStack Query — Parallel Queries — https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries

---

## 5. Query Keys & Query Functions

### Definitions

**Core Definition:** Query keys uniquely identify a query's cache entry; query functions fetch the data for that key.

**Technical Definition:** A query key is a serializable array (e.g., `['todos', { status: 'done', page: 1 }]`) that TanStack Query hashes deterministically. The same key always maps to the same cache entry. The `queryFn` receives a `QueryFunctionContext` containing the `queryKey`, `signal` (for cancellation), and `meta`. Query functions must return a Promise and throw on error. For `fetch`, you must throw manually on non-2xx responses.

**Beginner-Friendly Explanation:** The query key is the name of the data in the cache. The query function is how you fetch it. If two components use the same key, they share the same data.

### Purposes

- To uniquely identify and cache data.
- To allow multiple components to share the same data.
- To invalidate or refetch specific data.
- To pass parameters to the query function.
- To enable deterministic cache matching.

### Syntax Rules and Structure

**Query key conventions:**

```tsx
// Simple list
['todos']

// List with filters
['todos', { status: 'done', page: 2 }]

// Detail by ID
['todo', todoId]

// Nested resource
['user', userId, 'posts']
```

**Query function with context:**

```tsx
useQuery({
  queryKey: ['todo', todoId],
  queryFn: ({ queryKey, signal }) => {
    const [, id] = queryKey;
    return fetch(`/api/todos/${id}`, { signal }).then((r) => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`);
      return r.json();
    });
  },
});
```

**Syntax Rules:**

1. Query keys must be arrays; they are hashed deterministically.
2. Include every parameter that affects the result.
3. Use hierarchical keys so prefix invalidation works (`['todos']` matches `['todos', 'done']`).
4. `queryFn` receives `{ queryKey, signal, meta, pageParam }`.
5. Throw errors in `queryFn` for TanStack Query to detect failures.

**Constraints and Limitations:**

- Query keys are serialized; non-serializable values (functions, class instances) break hashing.
- Key order matters for arrays; use objects for named parameters to avoid positional fragility.

### Annotated Code Examples

```tsx
function TodoDetail({ todoId }) {
  return useQuery({
    queryKey: ['todo', todoId],
    queryFn: ({ queryKey, signal }) => {
      const [, id] = queryKey;
      return fetch(`/api/todos/${id}`, { signal }).then((res) => {
        if (!res.ok) throw new Error('Failed to fetch todo');
        return res.json();
      });
    },
  });
}
```

**Expected Output:** The query fetches the todo by ID and caches it under `['todo', todoId]`.

**Why this output:** The query key includes the ID, so each todo has its own cache entry. The query function extracts the ID from the key.

### Real-World Cases

- **Lists and details:** `['users']` and `['user', id]`.
- **Filtered views:** `['todos', { status, page }]`.
- **Nested resources:** `['user', id, 'posts']`.

### References

- TanStack Query — Query Functions — https://tanstack.com/query/v4/docs/framework/react/guides/query-functions
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Query Options — https://tanstack.com/query/latest/docs/framework/react/guides/query-options

---

## 6. Query Caching, Invalidation & Refetching

### Definitions

**Core Definition:** Caching stores query results; invalidation marks them stale and triggers refetching; refetching updates them from the server.

**Technical Definition:** Cached data is considered fresh within `staleTime` and stale after. Stale data is served immediately and refetched in the background on mount, window focus, and reconnect. `queryClient.invalidateQueries({ queryKey })` marks matching queries as stale and refetches active ones, overriding `staleTime`. `refetchOnWindowFocus`, `refetchOnReconnect`, `refetchInterval`, and manual `refetch()` control when refetches occur.

**Beginner-Friendly Explanation:** Caching keeps data around. Invalidation tells TanStack Query "this data might be old now, go check." Refetching is the actual network request that updates the cache.

### Purposes

- To serve data instantly from cache.
- To keep data fresh in the background.
- To update the UI after mutations.
- To recover from offline periods.
- To control network usage with `staleTime` and refetch triggers.

### Syntax Rules and Structure

**Invalidation:**

```tsx
queryClient.invalidateQueries({ queryKey: ['todos'] });
queryClient.invalidateQueries({ queryKey: ['todos'], exact: true });
```

**Refetch triggers:**

```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 30_000,
  refetchOnWindowFocus: true,
  refetchOnReconnect: true,
  refetchInterval: 60_000,
});
```

**Syntax Rules:**

1. `staleTime` controls freshness; default `0`.
2. `gcTime` controls memory retention; default 5 minutes.
3. `invalidateQueries` matches by prefix unless `exact: true`.
4. Invalidation marks data stale and refetches active queries.
5. `refetchOnWindowFocus` and `refetchOnReconnect` default to `true`.

**Constraints and Limitations:**

- Over-invalidating causes refetch storms.
- `staleTime: Infinity` disables automatic refetches.
- Background refetches can overwrite optimistic updates if not cancelled first.

### Annotated Code Examples

```tsx
function TodoList() {
  const queryClient = useQueryClient();
  const { data, isFetching } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    staleTime: 60_000,
  });

  const mutation = useMutation({
    mutationFn: addTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <>
      {isFetching && <RefreshIndicator />}
      <TodoItems items={data} />
      <button onClick={() => mutation.mutate({ text: 'New' })}>Add</button>
    </>
  );
}
```

**Expected Output:** Adding a todo invalidates the `todos` query, which refetches and updates the list. A subtle indicator appears during background fetches.

**Why this output:** `invalidateQueries` marks the todos query stale and refetches it. The list updates with the new todo from the server.

### Real-World Cases

- **After mutations:** Invalidate lists and detail views.
- **Window focus:** Refresh data when the user returns.
- **Polling:** Periodic refetches for live data.

### References

- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query — Window Focus Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching

---

## 7. Query Prefetching

### Definitions

**Core Definition:** Prefetching is loading query data before it is needed, so the eventual render is instant.

**Technical Definition:** `queryClient.prefetchQuery({ queryKey, queryFn, staleTime })` populates the cache without creating a subscription. If the query already has cached state (including pending or error state), prefetching is skipped. Prefetching is commonly triggered on hover, focus, or viewport visibility. `usePrefetchQuery` and `usePrefetchInfiniteQuery` are hooks for prefetching during render.

**Beginner-Friendly Explanation:** Prefetching means guessing what the user will need next and loading it early, so it is already there when they click.

### Purposes

- To make navigations feel instant.
- To use idle time productively.
- To avoid loading states on detail pages.
- To prefetch the next page in pagination.
- To prefetch related data.

### Syntax Rules and Structure

```tsx
function PostLink({ post }) {
  const queryClient = useQueryClient();

  return (
    <Link
      to={`/posts/${post.id}`}
      onMouseEnter={() => {
        queryClient.prefetchQuery({
          queryKey: ['post', post.id],
          queryFn: () => fetchPost(post.id),
          staleTime: 10_000,
        });
      }}
    >
      {post.title}
    </Link>
  );
}
```

**Syntax Rules:**

1. Use `queryClient.prefetchQuery` for imperative prefetching.
2. Prefetch only if the query has no cached state.
3. Set `staleTime` to avoid unnecessary prefetches of fresh data.
4. Use `prefetchInfiniteQuery` for infinite queries.
5. Trigger on hover, focus, or viewport visibility.

**Constraints and Limitations:**

- Aggressive prefetching wastes bandwidth.
- Prefetched data is still subject to `staleTime` and refetch triggers.

### Annotated Code Examples

```tsx
function ProductGrid({ products }) {
  const queryClient = useQueryClient();

  return products.map((product) => (
    <div
      key={product.id}
      onMouseEnter={() => {
        queryClient.prefetchQuery({
          queryKey: ['product', product.id],
          queryFn: () => fetchProduct(product.id),
          staleTime: 30_000,
        });
      }}
    >
      <Link to={`/products/${product.id}`}>{product.name}</Link>
    </div>
  ));
}
```

**Expected Output:** Hovering a product prefetches its detail; clicking navigates without a loading state if the prefetch completed.

**Why this output:** `prefetchQuery` fills the cache before navigation. The detail page's `useQuery` finds the cached entry and renders instantly.

### Real-World Cases

- **E-commerce:** Prefetching product details from listing cards.
- **Dashboards:** Prefetching the next page of data.
- **Documentation:** Prefetching article content on hover.

### References

- TanStack Query — Prefetching — https://mintlify.wiki/TanStack/query/guides/prefetching
- TanStack Query — Prefetching & Router Integration — https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query — usePrefetchQuery — https://tanstack.com/query/latest/docs/framework/react/reference/usePrefetchQuery

---

## 8. Query Pagination & Infinite Scrolling

### Definitions

**Core Definition:** Pagination splits data into pages; infinite scrolling automatically loads the next page as the user scrolls.

**Technical Definition:** Offset pagination uses `useQuery` with the page number in the query key and `placeholderData: keepPreviousData` to avoid loading flashes. Cursor-based pagination uses `useInfiniteQuery` with `initialPageParam` and `getNextPageParam`. Infinite scrolling combines `useInfiniteQuery` with the `IntersectionObserver` API to call `fetchNextPage` when a sentinel enters the viewport.

**Beginner-Friendly Explanation:** Pagination shows data in chunks. Infinite scrolling loads the next chunk automatically when you scroll near the bottom.

### Purposes

- To display large datasets without rendering everything.
- To reduce initial load time.
- To provide seamless infinite scroll experiences.
- To cache each page independently.
- To prefetch the next page.

### Syntax Rules and Structure

**Offset pagination:**

```tsx
const { data, isPlaceholderData } = useQuery({
  queryKey: ['projects', page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData,
});
```

**Infinite scrolling:**

```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
  useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  });

const sentinelRef = useRef(null);

useEffect(() => {
  const observer = new IntersectionObserver(([entry]) => {
    if (entry.isIntersecting && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  });
  if (sentinelRef.current) observer.observe(sentinelRef.current);
  return () => observer.disconnect();
}, [hasNextPage, isFetchingNextPage, fetchNextPage]);
```

**Syntax Rules:**

1. Use `keepPreviousData` / `placeholderData` for offset pagination.
2. Use `useInfiniteQuery` for cursor-based pagination.
3. `initialPageParam` and `getNextPageParam` are required for infinite queries.
4. Guard `fetchNextPage` with `hasNextPage` and `isFetchingNextPage`.
5. Use `IntersectionObserver` for infinite scroll triggers.

**Constraints and Limitations:**

- Offset pagination is unstable when items are inserted or deleted.
- Infinite scrolling can harm accessibility; provide a "Load more" fallback.
- Very long lists can accumulate memory; use `maxPages` or virtualization.

### Annotated Code Examples

```tsx
function InfiniteFeed() {
  const {
    data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    maxPages: 20,
  });

  const sentinelRef = useRef(null);

  useEffect(() => {
    const el = sentinelRef.current;
    if (!el) return;
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting && hasNextPage && !isFetchingNextPage) {
        fetchNextPage();
      }
    });
    observer.observe(el);
    return () => observer.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isPending) return <FeedSkeleton />;

  return (
    <div>
      {data.pages.flatMap((page) =>
        page.items.map((item) => <FeedCard key={item.id} item={item} />)
      )}
      <div ref={sentinelRef} />
      {isFetchingNextPage && <Spinner />}
      {!hasNextPage && <p>You're all caught up.</p>}
    </div>
  );
}
```

**Expected Output:** The feed loads the first page, and scrolling near the bottom automatically fetches the next page until there are no more pages.

**Why this output:** The `IntersectionObserver` triggers `fetchNextPage` when the sentinel is visible. Guards prevent duplicate requests. `maxPages` bounds memory.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Chat history:** Loading older messages.
- **Product listings:** Load more products.

### References

- TanStack Query — Infinite Queries — https://tanstack.dev/query/v5/docs/framework/react/guides/infinite-queries
- TanStack Query — Pagination — https://tanstack.com/query/latest/docs/framework/react/guides/pagination
- TanStack Query — useInfiniteQuery — https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery

---

## 9. Query Optimistic Updates

### Definitions

**Core Definition:** Optimistic updates apply a mutation's effect to the UI before the server confirms it, with automatic rollback on failure.

**Technical Definition:** In `useMutation`, `onMutate` runs before the mutation function. It cancels outgoing refetches, snapshots the previous cache value, applies the optimistic change with `queryClient.setQueryData`, and returns the snapshot as context. `onError` receives the context and rolls back the cache. `onSettled` invalidates the query to reconcile with the server.

**Beginner-Friendly Explanation:** Optimistic updates make the UI feel instant. You change the screen immediately, assume the server will succeed, and undo the change if it fails.

### Purposes

- To provide instant feedback for mutations.
- To reduce perceived latency.
- To improve UX for likes, follows, and toggles.
- To maintain correctness with rollback on failure.

### Syntax Rules and Structure

```tsx
useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previous = queryClient.getQueryData(['todos']);
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
    return { previous };
  },
  onError: (_err, _newTodo, context) => {
    queryClient.setQueryData(['todos'], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

**Syntax Rules:**

1. Always `cancelQueries` before applying the optimistic update.
2. Always snapshot the previous value and return it as context.
3. Always roll back in `onError` using the context.
4. Always invalidate in `onSettled` to reconcile with the server.
5. Use immutable updates — never mutate the cached array.

**Constraints and Limitations:**

- Optimistic updates are a correctness trade-off.
- Not suitable for operations where failure is common (payments).
- Rollback must handle concurrent mutations carefully.

### Annotated Code Examples

```tsx
function TodoList() {
  const queryClient = useQueryClient();
  const { data: todos } = useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

  const mutation = useMutation({
    mutationFn: addTodo,
    onMutate: async (newTodo) => {
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      const previous = queryClient.getQueryData(['todos']);
      queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
      return { previous };
    },
    onError: (_err, _newTodo, context) => {
      queryClient.setQueryData(['todos'], context.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ id: crypto.randomUUID(), title: 'New' })}>
      Add todo
    </button>
  );
}
```

**Expected Output:** The new todo appears instantly. If the request fails, it disappears and the list reverts. After success or failure, the list refetches from the server.

**Why this output:** `onMutate` applies the optimistic change; `onError` restores the snapshot; `onSettled` reconciles with the server.

### Real-World Cases

- **Likes and reactions:** Toggling instantly.
- **Todo lists:** Adding items before the server responds.
- **Follow buttons:** Updating follower counts immediately.

### References

- TanStack Query — Optimistic Updates — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations

---

## 10. Query Devtools

### Definitions

**Core Definition:** TanStack Query Devtools is a visual interface for inspecting, debugging, and understanding queries and mutations.

**Technical Definition:** The devtools package `@tanstack/react-query-devtools` exports `ReactQueryDevtools`, which mounts a floating panel in development mode. It displays all queries and mutations, their states, cached data, and timing information. By default, it is only included when `process.env.NODE_ENV === 'development'`. It supports options like `initialIsOpen`, `buttonPosition`, and `position`.

**Beginner-Friendly Explanation:** The devtools are a visual dashboard that shows you what TanStack Query is doing — which queries are active, what data is cached, and when refetches happen.

### Purposes

- To inspect query and mutation states.
- To debug cache issues.
- To understand refetch triggers.
- To visualise query lifecycles.
- To test error states.

### Syntax Rules and Structure

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* app */}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

**Syntax Rules:**

1. Install `@tanstack/react-query-devtools` as a dev dependency.
2. Place `ReactQueryDevtools` inside `QueryClientProvider`.
3. It is automatically excluded from production builds.
4. Use `initialIsOpen` to control the default open state.
5. Use `buttonPosition` and `position` to customise placement.

### Annotated Code Examples

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Routes />
      <ReactQueryDevtools initialIsOpen={false} buttonPosition="bottom-right" />
    </QueryClientProvider>
  );
}
```

**Expected Output:** A TanStack Query logo appears in the bottom-right corner; clicking it opens a panel showing all queries and mutations.

**Why this output:** The devtools mount as a floating element and subscribe to the `QueryClient` to display its state.

### Real-World Cases

- **Debugging cache issues:** Seeing which keys are cached and their freshness.
- **Testing error states:** Predefining errors to trigger from the UI.
- **Performance:** Observing refetch frequency.

### References

- TanStack Query — Devtools — https://tanstack.com/query/latest/docs/framework/react/devtools
- TanStack Query — Devtools (Mintlify) — https://mintlify.wiki/TanStack/query/guides/devtools

---

## 11. Query Best Practices

### Definitions

**Core Definition:** TanStack Query best practices are the guidelines for structuring query keys, configuring caching, handling mutations, and maintaining a predictable server-state layer.

**Technical Definition:** Best practices combine query-key discipline (hierarchical, serializable, complete), cache configuration (`staleTime`, `gcTime` tuned to data volatility), mutation hygiene (invalidate after mutations, cancel before optimistic updates), prefetching on intent, and avoiding anti-patterns such as storing derived state, over-invalidating, or using `useEffect` for data fetching when `useQuery` would suffice. In v5, callbacks on `useQuery` are removed (use `useEffect` or the query cache), `cacheTime` is renamed to `gcTime`, and `keepPreviousData` is replaced by `placeholderData`.

**Beginner-Friendly Explanation:** Good TanStack Query usage means using clear query keys, setting sensible cache times, invalidating after changes, and letting the library handle the hard parts instead of fighting it.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use hierarchical query keys** | `['todos']`, `['todos', id]`, `['todos', { status }]` |
| **Include all parameters in the key** | Every value that affects the result |
| **Tune `staleTime`** | Higher for stable data, lower for volatile data |
| **Use `gcTime` for memory** | Default 5 minutes; `Infinity` to disable GC |
| **Invalidate after mutations** | `invalidateQueries` to reconcile with the server |
| **Cancel before optimistic updates** | `cancelQueries` to prevent race conditions |
| **Prefetch on intent** | Hover, focus, viewport |
| **Use `useQueries` for dynamic counts** | Never call hooks in loops |
| **Use `useInfiniteQuery` for cursor pagination** | Avoid offset-based infinite scroll |
| **Use v5 single-object signature** | `useQuery({ queryKey, queryFn })` |
| **Remove callbacks from `useQuery`** | v5 removed `onSuccess`/`onError` on queries |
| **Use `placeholderData` instead of `keepPreviousData`** | v5 migration |
| **Use `useSuspenseQuery` instead of `suspense: true`** | v5 migration |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Hierarchical keys
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
};

// Tuned cache
useQuery({
  queryKey: todoKeys.list('done'),
  queryFn: () => fetchTodos({ status: 'done' }),
  staleTime: 30_000,
});

// Invalidate after mutation
useMutation({
  mutationFn: addTodo,
  onSettled: () => queryClient.invalidateQueries({ queryKey: todoKeys.all }),
});
```

**❌ Bad Practices:**

```tsx
// ❌ Missing parameters in the key
useQuery({ queryKey: ['todos'], queryFn: () => fetchTodos({ page }) });

// ❌ Over-invalidating everything
queryClient.invalidateQueries();

// ❌ Storing derived state in the cache
queryClient.setQueryData(['todoCount'], data.length);

// ❌ Calling hooks in a loop
userIds.forEach((id) => useQuery({ queryKey: ['user', id], queryFn: ... }));
```

**Expected Output:** The good practices produce predictable caching, targeted invalidation, and correct updates. The bad practices cause cache collisions, refetch storms, stale derived data, and Hook violations.

**Why this output:** Complete keys prevent collisions. Targeted invalidation avoids over-fetching. Derived state should be computed, not cached. Hooks must be called at the top level.

### Real-World Cases

- **All TanStack Query applications:** These practices apply universally.
- **Large codebases:** Query key factories keep keys consistent.
- **Mutation-heavy apps:** Targeted invalidation prevents refetch storms.

### References

- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query v5 Migration Guide — https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- State Management Patterns (TanStack Query v5 + Zustand v5) — https://github.com/AsyrafHussin/agent-skills/tree/main/skills/state-management

---

## References

- TanStack Query — useQuery — https://tanstack.dev/query/v4/docs/framework/react/reference/useQuery
- TanStack Query — useMutation — https://tanstack.dev/query/latest/docs/framework/react/reference/useMutation
- TanStack Query — Infinite Queries — https://tanstack.dev/query/v5/docs/framework/react/guides/infinite-queries
- TanStack Query — useQueries — https://tanstack.com/query/v4/docs/framework/react/reference/useQueries
- TanStack Query — Query Functions — https://tanstack.com/query/v4/docs/framework/react/guides/query-functions
- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query — Prefetching — https://mintlify.wiki/TanStack/query/guides/prefetching
- TanStack Query — Prefetching & Router Integration — https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query — Optimistic Updates — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query — Devtools — https://tanstack.com/query/latest/docs/framework/react/devtools
- TanStack Query — Devtools (Mintlify) — https://mintlify.wiki/TanStack/query/guides/devtools
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Pagination — https://tanstack.com/query/latest/docs/framework/react/guides/pagination
- TanStack Query — Parallel Queries — https://tanstack.com/query/latest/docs/framework/react/guides/parallel-queries
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- TanStack Query — Window Focus Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query v5 Migration Guide — https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5
- State Management Patterns (TanStack Query v5 + Zustand v5) — https://github.com/AsyrafHussin/agent-skills/tree/main/skills/state-management