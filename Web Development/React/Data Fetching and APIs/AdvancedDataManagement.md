# Advanced Data Management & Real-Time — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced data management and real-time patterns are the architectural techniques for keeping a React application's server-derived data fresh, consistent, performant, and synchronized with live events — going far beyond the basic `useEffect` + `fetch` model.

**Technical Definition:** This domain sits at the intersection of **server-state management** (caching, deduplication, pagination, prefetching, optimistic updates, background synchronization) and **real-time transport** (polling, long polling, Server-Sent Events, WebSockets, streaming). The central insight is that **server state is not client state**: it is remote, shared, cached, refetched, invalidated, and sometimes intentionally stale. Libraries like TanStack Query and SWR treat server state as a first-class citizen with a lifecycle (fresh → stale → inactive → garbage-collected), while real-time transports determine *how* updates arrive — pull (polling), push (SSE, WebSockets), or progressive delivery (streaming). TanStack Query’s defaults encode the industry consensus: `staleTime` of `0` (data is immediately stale) and `gcTime` of 5 minutes. Choosing the right transport is three questions: direction of data flow, frequency of change, and infrastructure constraints — one-way → SSE, two-way → WebSockets, rare changes → polling.

**Beginner-Friendly Explanation:** Once your app grows beyond a few screens, fetching data becomes complicated. You need to avoid fetching the same thing twice, keep data fresh without annoying the user, load more items as they scroll, make button clicks feel instant, recover when the network drops, and receive live updates (chat messages, notifications, prices). This cheat sheet covers the tools and patterns for each of those problems — from caching and pagination to WebSockets and streaming.

### Key Characteristics

- **Server state has a lifecycle:** Fresh → stale → refetching → inactive → garbage-collected. Libraries automate this lifecycle; manual code does not.
- **Deduplication is automatic in libraries, manual in raw fetch:** Multiple components requesting the same key share one in-flight request and one cache entry.
- **Pagination and infinite scrolling are query-key problems:** The page cursor must be part of the key so each page is cached independently.
- **Prefetching is intent-driven:** Hover, focus, and viewport visibility are signals to load data or code before the user commits to navigation.
- **Optimistic updates trade correctness for latency:** Apply the change immediately, snapshot the previous state, and roll back on failure.
- **Real-time transport is a trade-off:** Polling is simple but wasteful; SSE is one-way and HTTP-native; WebSockets are bidirectional and stateful.
- **Streaming is progressive delivery:** React’s server renderer emits HTML in chunks aligned with `<Suspense>` boundaries, so slow sections do not block fast ones.

### Prerequisites

- React function components, `useState`, `useEffect`, and event handlers.
- Promises, `async`/`await`, and `AbortController`.
- HTTP fundamentals: methods, status codes, headers, JSON.
- TanStack Query or SWR basics (for the library-based patterns).
- Browser APIs: `EventSource`, `WebSocket`, `IntersectionObserver`, `ReadableStream`.

### Related Programming Areas

- **Data Fetching Fundamentals** — the request lifecycle and caching basics.
- **API Integration** — REST, GraphQL, and request topology.
- **State Management** — server state vs. client state vs. URL state.
- **Performance** — waterfalls, prefetching, and streaming.
- **Real-Time Systems** — WebSockets, SSE, and pub/sub architectures.

---

## 1. Caching

### Definitions

**Core Definition:** Caching is the practice of storing fetched data so subsequent requests can reuse it instead of re-fetching, trading memory and potential staleness for speed and reduced network load.

**Technical Definition:** In TanStack Query (v5), caching has two orthogonal dimensions. **`staleTime`** (default `0`) controls how long data is considered *fresh*: within the window, the cache is served immediately with no network request; after it, cached data is still served immediately but a background refetch is triggered. **`gcTime`** (formerly `cacheTime`, default 5 minutes) controls how long *inactive* queries remain in memory before garbage collection. Active queries (those with mounted observers) are never garbage-collected. **Structural sharing** compares refetched data with cached data and preserves the previous reference when the values are deeply equal, preventing unnecessary re-renders.

**Beginner-Friendly Explanation:** Caching means remembering data you already fetched. Fresh data is served from memory without asking the server. Stale data is still shown immediately, but the app quietly checks for updates in the background.

### Purposes

- To eliminate redundant network requests for the same data.
- To make repeat visits and back-navigation instant.
- To enable stale-while-revalidate UX.
- To reduce server load and bandwidth.
- To control memory usage with garbage collection.
- To prevent unnecessary re-renders with structural sharing.

### Syntax Rules and Structure

**General syntax (TanStack Query):**

```tsx
useQuery({
  queryKey: ['user', userId],
  queryFn: ({ signal }) => fetchUser(userId, { signal }),
  staleTime: 5 * 60 * 1000, // Fresh for 5 minutes
  gcTime: 30 * 60 * 1000,   // Keep inactive for 30 minutes
});
```

**Cache configuration matrix:**

| Scenario | `staleTime` | `gcTime` | Rationale |
|----------|-------------|----------|-----------|
| Lookup data (countries) | 1 hour | 24 hours | Rarely changes |
| User profile | 5 minutes | 30 minutes | Changes occasionally |
| Dashboard stats | 30 seconds | 5 minutes | Near-real-time |
| Live prices | 5 seconds | 1 minute | High volatility |

**Syntax Rules:**

1. `staleTime` defaults to `0`; data is immediately stale and refetches on mount/focus.
2. `gcTime` defaults to 5 minutes; set to `Infinity` to disable garbage collection.
3. Query keys define cache identity; every parameter affecting the result must be in the key.
4. Use `structuralSharing: false` for very large, frequently changing datasets to skip expensive deep diffs.
5. When using `select`, structural sharing runs twice; disable it for large datasets with frequent updates.

**Constraints and Limitations:**

- Cached data can be stale; invalidation is required after mutations.
- Over-caching sensitive data is a security risk.
- Structural sharing adds O(n) overhead per refetch.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Configuring Cache Freshness

```tsx
function CountryList() {
  const { data, isPending } = useQuery({
    queryKey: ['countries'],
    queryFn: fetchCountries,
    staleTime: 60 * 60 * 1000, // Fresh for 1 hour
    gcTime: 24 * 60 * 60 * 1000,
  });

  if (isPending) return <Skeleton />;
  return <ul>{data.map((c) => <li key={c.code}>{c.name}</li>)}</ul>;
}
```

**Expected Output:** The country list is fetched once, served from cache for an hour without network requests, and garbage-collected after a day of inactivity.

**Why this output:** `staleTime` prevents refetches within the hour; `gcTime` keeps the entry long enough for repeat visits.

#### Example 2: Disabling Structural Sharing for Large Data

```tsx
function RadarContacts() {
  const { data } = useQuery({
    queryKey: ['radar-contacts'],
    queryFn: fetchRadarContacts,
    staleTime: 2_000,
    structuralSharing: false, // Large, frequently changing dataset
  });
  return <Radar contacts={data} />;
}
```

**Expected Output:** The component re-renders with new data even if it is deeply equal, avoiding the O(n) diff cost on thousands of contacts.

**Why this output:** Structural sharing is beneficial for small datasets but adds overhead for large, fast-changing ones.

### Real-World Cases

- **Lookup tables:** Countries, currencies, and categories cached for hours.
- **User profiles:** Cached for minutes across navigations.
- **Dashboards:** Short staleness for near-real-time stats.

### References

- TanStack Query — Caching — https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- Core Concepts and Performance (TanStack Query Best Practices) — https://github.com/gohypergiant/standard-toolkit

---

## 2. Deduplication

### Definitions

**Core Definition:** Deduplication is the practice of ensuring that multiple simultaneous requests for the same data result in a single network request, with all callers sharing the result.

**Technical Definition:** TanStack Query deduplicates requests by **query key**: if multiple components mount and request the same key while a fetch is in flight, only one network request is made, and all observers receive the result. The cache key becomes the contract for reads, writes, invalidation, prefetching, and devtools. Deduplication also applies to `prefetchQuery` and `ensureQueryData`, which reuse existing cache entries and in-flight promises.

**Beginner-Friendly Explanation:** Deduplication means “if ten components ask for the same data at the same time, only one request goes out.” Everyone shares the answer.

### Purposes

- To prevent duplicate network requests from multiple components.
- To reduce server load and bandwidth.
- To ensure all components see the same data simultaneously.
- To simplify component code (no manual request coordination).
- To support prefetching without causing duplicate fetches.

### Syntax Rules and Structure

**Query key as the deduplication contract:**

```tsx
// Component A
useQuery({ queryKey: ['user', 42], queryFn: () => fetchUser(42) });

// Component B — same key, same in-flight request, same cache entry
useQuery({ queryKey: ['user', 42], queryFn: () => fetchUser(42) });
```

**Syntax Rules:**

1. Query keys must be serialisable and uniquely identify the request.
2. Include every parameter that affects the result in the key.
3. Use `queryClient.ensureQueryData` in prefetch/loader scenarios to reuse cache and in-flight promises.
4. Use `queryClient.cancelQueries` before optimistic updates to prevent refetches from overwriting optimistic state.

**Constraints and Limitations:**

- Deduplication only works when keys match exactly.
- Different keys for the “same” resource cause duplicate fetches.
- Without a query library, deduplication must be implemented manually with a promise cache.

### Annotated Code Examples

```tsx
function UserHeader({ userId }) {
  const { data } = useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });
  return <h1>{data?.name}</h1>;
}

function UserSidebar({ userId }) {
  const { data } = useQuery({ queryKey: ['user', userId], queryFn: () => fetchUser(userId) });
  return <p>{data?.email}</p>;
}
```

**Expected Output:** Mounting both components results in a single network request; both render from the shared cache entry.

**Why this output:** Both `useQuery` calls use the identical key `['user', userId]`. TanStack Query deduplicates the in-flight request and shares the result.

### Real-World Cases

- **Layouts:** Header, sidebar, and content requesting the same user.
- **Prefetching:** A loader and a component requesting the same data.
- **Nested routes:** Parent and child routes depending on the same resource.

### References

- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query Overview (dedupe) — https://tanstack.com/query/v3

---

## 3. Pagination

### Definitions

**Core Definition:** Pagination is the technique of splitting a large dataset into discrete pages that are fetched and displayed one at a time.

**Technical Definition:** In TanStack Query, **offset-based pagination** uses `useQuery` with the page number in the query key and `keepPreviousData` (v5: `placeholderData: keepPreviousData`) to avoid a loading flash when changing pages. **Cursor-based pagination** uses `useInfiniteQuery` with `initialPageParam` and `getNextPageParam`; each page is cached as a separate query and can be prefetched individually. Cursor-based pagination is generally preferred for large or changing datasets because it avoids the skip/limit instability of offset pagination.

**Beginner-Friendly Explanation:** Pagination means showing data in pages instead of all at once. Cursor-based pagination uses a “next page” token instead of a page number, which is more reliable when items are added or removed.

### Purposes

- To display large datasets without rendering everything.
- To reduce initial load time and payload size.
- To support “Previous” and “Next” navigation.
- To cache each page independently for instant back-navigation.
- To prefetch the next page for a smoother experience.

### Syntax Rules and Structure

**Offset pagination:**

```tsx
const [page, setPage] = useState(1);

const { data, isPlaceholderData } = useQuery({
  queryKey: ['projects', page],
  queryFn: () => fetchProjects(page),
  placeholderData: keepPreviousData, // Keep previous page while fetching
});
```

**Cursor pagination with `useInfiniteQuery`:**

```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['projects'],
  queryFn: ({ pageParam }) => fetchProjects({ cursor: pageParam }),
  initialPageParam: null,
  getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
  maxPages: 10, // Optional: limit stored pages
});
```

**Syntax Rules:**

1. Include the page cursor in the query key for offset pagination.
2. Use `keepPreviousData` / `placeholderData` to avoid loading flashes.
3. For infinite queries, `initialPageParam` and `getNextPageParam` are required in v5.
4. Use `maxPages` to bound memory for very long lists.
5. Each page is cached as a normal query, so back-navigation is instant.

**Constraints and Limitations:**

- Offset pagination is unstable when items are inserted or deleted.
- `useInfiniteQuery` requires a cursor or offset-based page parameter.
- `maxPages` removes pages from the cache when exceeded, which can cause refetching when navigating back.

### Annotated Code Examples

```tsx
function ProjectList() {
  const [page, setPage] = useState(1);
  const { data, isPending, isPlaceholderData } = useQuery({
    queryKey: ['projects', page],
    queryFn: () => fetchProjects(page),
    placeholderData: keepPreviousData,
  });

  if (isPending) return <Skeleton />;
  return (
    <>
      <ProjectGrid projects={data.projects} dimmed={isPlaceholderData} />
      <button onClick={() => setPage((p) => p + 1)} disabled={!data.hasNext}>
        Next
      </button>
    </>
  );
}
```

**Expected Output:** Clicking “Next” fetches the next page while keeping the previous page visible (dimmed) until the new data arrives.

**Why this output:** `placeholderData: keepPreviousData` prevents the UI from flashing a skeleton during page changes.

### Real-World Cases

- **Data tables:** Offset pagination with page controls.
- **Search results:** Cursor-based pagination for stable results.
- **Admin panels:** Paginated lists with prefetched next pages.

### References

- TanStack Query — Pagination — https://tanstack.com/query/latest/docs/framework/react/guides/pagination
- TanStack Query — useInfiniteQuery — https://mintlify.wiki/TanStack/query/api/react/use-infinite-query
- TanStack Query — Infinite Queries — https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries

---

## 4. Infinite Scrolling

### Definitions

**Core Definition:** Infinite scrolling is the pattern of automatically loading more data as the user approaches the end of a list, rather than requiring an explicit “Next” click.

**Technical Definition:** Infinite scrolling combines `useInfiniteQuery` with the **Intersection Observer API**: a sentinel element at the end of the list triggers `fetchNextPage` when it enters the viewport. The query aggregates all pages into `data.pages`, and `getNextPageParam` determines whether more data exists. TanStack Query’s `useInfiniteQuery` extends `useQuery` with `fetchNextPage`, `hasNextPage`, `isFetchingNextPage`, and `maxPages`.

**Beginner-Friendly Explanation:** Infinite scrolling means “load more when the user scrolls to the bottom.” An invisible sentinel element watches for the user reaching the end and triggers the next page fetch.

### Purposes

- To provide a seamless browsing experience for long feeds.
- To avoid explicit pagination controls.
- To load data incrementally as the user consumes it.
- To cache each page for instant back-navigation.
- To bound memory with `maxPages`.

### Syntax Rules and Structure

```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
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

const items = data?.pages.flatMap((page) => page.items) ?? [];
```

**Syntax Rules:**

1. Use `IntersectionObserver` rather than scroll event listeners for performance.
2. Guard with `hasNextPage` and `isFetchingNextPage` to prevent duplicate fetches.
3. Flatten `data.pages` into a single list for rendering.
4. Use stable keys (`item.id`) for list items.
5. Provide a “Load more” fallback button for accessibility and users who prefer explicit control.

**Constraints and Limitations:**

- Infinite scrolling can harm accessibility and “find on page” behaviour; provide a “Load more” alternative.
- Very long lists can accumulate memory; use `maxPages` or virtualization.
- Cursor-based pagination is preferred; offset-based infinite scroll is unstable.

### Annotated Code Examples

```tsx
function InfiniteFeed() {
  const {
    data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending,
  } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed({ cursor: pageParam }),
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
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

**Why this output:** The observer triggers `fetchNextPage` when the sentinel is visible. Guards prevent duplicate requests. `maxPages` bounds memory.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Chat history:** Loading older messages on scroll.
- **Product listings:** Infinite scroll with skeleton cards.

### References

- TanStack Query — Infinite Queries — https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
- TanStack Query — useInfiniteQuery — https://mintlify.wiki/TanStack/query/api/react/use-infinite-query
- MDN — Intersection Observer API — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API

---

## 5. Prefetching

### Definitions

**Core Definition:** Prefetching is loading data or code before the user explicitly requests it, based on predicted intent, so the eventual navigation or render feels instant.

**Technical Definition:** TanStack Query provides `queryClient.prefetchQuery` and `queryClient.ensureQueryData` to populate the cache. Prefetching is most effective when triggered by **user intent signals**: hovering a link, focusing a button, or a card entering the viewport. In frameworks, Server Components can prefetch queries and dehydrate the cache for client hydration. Prefetching differs from `staleTime` tuning: it proactively fills the cache rather than waiting for a component to mount.

**Beginner-Friendly Explanation:** Prefetching means “guess what the user will need next and load it early.” Hovering a link, for example, is a strong signal that the user is about to click.

### Purposes

- To make navigations feel instant.
- To use idle time and intent signals productively.
- To avoid loading flashes on detail pages.
- To improve perceived performance without blocking the current page.
- To hydrate the client cache from server-rendered data.

### Syntax Rules and Structure

**Hover prefetch:**

```tsx
function ProjectLink({ projectId }) {
  const queryClient = useQueryClient();

  function prefetch() {
    queryClient.prefetchQuery({
      queryKey: ['project', projectId],
      queryFn: () => fetchProject(projectId),
      staleTime: 60_000,
    });
  }

  return (
    <Link to={`/projects/${projectId}`} onMouseEnter={prefetch}>
      View project
    </Link>
  );
}
```

**Loader prefetch (React Router / TanStack Router):**

```tsx
export async function loader({ params }) {
  await queryClient.ensureQueryData(projectQueryOptions(params.id));
  return null;
}
```

**Syntax Rules:**

1. Prefetch on intent (hover, focus, viewport) rather than on mount for all links.
2. Use `prefetchQuery` for fire-and-forget; use `ensureQueryData` when you need the data or error to propagate.
3. Prefetched data respects `staleTime`; set `preloadStaleTime` in TanStack Router if applicable.
4. Do not prefetch routes the user is unlikely to visit.
5. In SSR, prefetch on the server and dehydrate the cache for client hydration.

**Constraints and Limitations:**

- Aggressive prefetching wastes bandwidth and server resources.
- Prefetched data can become stale; it is still subject to `staleTime`.
- Prefetching does not help if the user navigates before the prefetch completes.

### Annotated Code Examples

```tsx
function ProductCard({ product }) {
  const queryClient = useQueryClient();

  function handleMouseEnter() {
    queryClient.prefetchQuery({
      queryKey: ['product', product.id],
      queryFn: () => fetchProduct(product.id),
      staleTime: 30_000,
    });
  }

  return (
    <Link to={`/products/${product.id}`} onMouseEnter={handleMouseEnter}>
      {product.name}
    </Link>
  );
}
```

**Expected Output:** Hovering the product card prefetches the product detail; clicking navigates without a loading state if the prefetch completed.

**Why this output:** `prefetchQuery` fills the cache on hover. The detail page’s `useQuery` finds the cached entry and renders instantly.

### Real-World Cases

- **E-commerce:** Prefetching product details from listing cards.
- **Dashboards:** Prefetching the next page of data.
- **Documentation:** Prefetching article content on link hover.

### References

- TanStack Query — Prefetching & Router Integration — https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query — SSR — https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- TanStack Query — Pagination (prefetching next page) — https://tanstack.com/query/latest/docs/framework/react/guides/pagination

---

## 6. Optimistic Updates

### Definitions

**Core Definition:** An optimistic update is a UI change applied immediately, before the server confirms the mutation, with automatic rollback if the mutation fails.

**Technical Definition:** In TanStack Query, optimistic updates use `useMutation` with three callbacks: **`onMutate`** cancels outgoing refetches, snapshots the previous cache value, applies the optimistic change with `queryClient.setQueryData`, and returns the snapshot as context; **`onError`** rolls back to the snapshot; **`onSettled`** invalidates the query to reconcile with the server. The critical rule is to cancel in-flight refetches before applying the optimistic value, so a settling refetch cannot overwrite it.

**Beginner-Friendly Explanation:** Optimistic updates make the UI feel instant. You change the screen immediately, assume the server will succeed, and undo the change if it fails.

### Purposes

- To provide instant feedback for mutations.
- To reduce perceived latency.
- To improve UX for likes, follows, toggles, and simple edits.
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
5. Use immutable updates (`[...old, newItem]`) — never mutate the cached array.
6. Use optimistic updates only for operations that usually succeed; for critical mutations, wait for the server.

**Constraints and Limitations:**

- Optimistic updates are a correctness trade-off: the UI can show data the server has not accepted.
- Rollback must handle concurrent mutations carefully.
- Not suitable for operations where failure is common (payments, destructive actions).

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
- Optimistic Updates (v4) — https://tanstack.com/query/v4/docs/framework/react/guides/optimistic-updates

---

## 7. Background Synchronization / Fetching

### Definitions

**Core Definition:** Background synchronization is the practice of refreshing cached data without blocking the current UI, so users see stale data immediately while updates arrive quietly.

**Technical Definition:** TanStack Query performs background refetches on several triggers: **window focus** (default), **reconnect** (default), **mount** (when data is stale), and **polling intervals** (`refetchInterval`). The query remains in `success` state while `isFetching` is `true`, allowing the UI to keep rendering existing data. **Stale-while-revalidate** is the resulting pattern: serve stale data immediately, revalidate in the background, and swap in fresh data when it arrives. `invalidateQueries` marks data stale and triggers refetches for active queries.

**Beginner-Friendly Explanation:** Background synchronization means the app keeps showing the data it has while quietly checking for updates. When new data arrives, the screen updates without a loading spinner.

### Purposes

- To keep data fresh without blocking the user.
- To recover from offline periods when the connection returns.
- To update data when the user returns to the tab.
- To avoid loading spinners for routine refreshes.
- To reconcile optimistic updates with server truth.

### Syntax Rules and Structure

```tsx
useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  staleTime: 30_000,
  refetchOnWindowFocus: true, // default
  refetchOnReconnect: true,   // default
  refetchInterval: 60_000,    // optional polling
});
```

**Syntax Rules:**

1. Use `isFetching` (not `isPending`) to indicate background activity.
2. Keep existing data visible during background refetches.
3. Use `refetchOnWindowFocus` and `refetchOnReconnect` for freshness.
4. Use `invalidateQueries` after mutations to trigger background reconciliation.
5. Avoid full-page spinners for background fetches; use subtle indicators.

**Constraints and Limitations:**

- Background refetches consume bandwidth and server resources.
- Aggressive focus refetching can be annoying on some screens.
- Background updates can overwrite local edits if not coordinated.

### Annotated Code Examples

```tsx
function NotificationBell() {
  const { data, isFetching } = useQuery({
    queryKey: ['notifications'],
    queryFn: fetchNotifications,
    staleTime: 30_000,
    refetchOnWindowFocus: true,
  });

  return (
    <button disabled={isFetching}>
      {data?.length ?? 0} notifications {isFetching && '· refreshing…'}
    </button>
  );
}
```

**Expected Output:** The count is shown immediately. When the window regains focus and data is stale, a background refetch runs; the button shows a subtle “refreshing…” indicator without hiding the count.

**Why this output:** The query remains in `success` state during the refetch. `isFetching` drives the subtle indicator, not a full loading state.

### Real-World Cases

- **Notifications:** Focus-based refetch.
- **Dashboards:** Periodic background refresh.
- **Offline recovery:** Reconnect refetch.

### References

- TanStack Query — Window Focus Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query Overview (background refetching) — https://tanstack.com/query/v3

---

## 8. Polling & Long Polling

### Definitions

**Core Definition:** Polling is repeatedly asking the server for updates at fixed intervals; long polling is a variant where the server holds the request open until data is available or a timeout occurs.

**Technical Definition:** **Short polling** uses `setInterval` or TanStack Query’s `refetchInterval` to fetch on a schedule. Every cycle pays a full HTTP request/response, usually to hear “nothing yet,” and average delivery delay is half the interval plus a round trip. **Long polling** holds the request open on the server until an event occurs or a timeout fires, then responds and the client immediately re-requests. It simulates push over plain HTTP and is useful when WebSocket upgrades are blocked, but it ties up server resources per client. The 2026 consensus: use polling for rare changes, SSE for one-way push, and WebSockets for two-way communication.

**Beginner-Friendly Explanation:** Polling means asking “any updates?” every few seconds. Long polling means asking once and having the server wait to answer until something happens.

### Purposes

- To refresh data at a predictable interval (short polling).
- To simulate server push when SSE/WebSockets are unavailable (long polling).
- To keep dashboards and status indicators current.
- To provide a fallback transport behind an abstraction.

### Syntax Rules and Structure

**Short polling (TanStack Query):**

```tsx
useQuery({
  queryKey: ['status'],
  queryFn: fetchStatus,
  refetchInterval: 5_000, // Poll every 5 seconds
  refetchIntervalInBackground: false,
});
```

**Long polling (conceptual):**

```js
async function longPoll() {
  while (true) {
    const res = await fetch('/api/events?since=' + lastId);
    if (res.status === 200) {
      const event = await res.json();
      handleEvent(event);
      lastId = event.id;
    }
    // On timeout (204), immediately re-request
  }
}
```

**Syntax Rules:**

1. Use `refetchInterval` for short polling; stop it when the data is not needed.
2. Use `refetchIntervalInBackground: false` to pause polling when the tab is hidden.
3. For long polling, the server must hold the connection and respond on event or timeout.
4. Long polling must handle reconnection and backoff.
5. Prefer SSE or WebSockets when available; use long polling as a fallback behind an abstraction.

**Constraints and Limitations:**

- Short polling wastes requests and battery.
- Long polling ties up server connections per client.
- Both are inferior to SSE/WebSockets for real-time push in 2026.

### Annotated Code Examples

```tsx
function LiveStatus() {
  const { data, isFetching } = useQuery({
    queryKey: ['live-status'],
    queryFn: fetchStatus,
    refetchInterval: 10_000,
    refetchIntervalInBackground: false,
  });

  return <StatusBadge status={data?.status} refreshing={isFetching} />;
}
```

**Expected Output:** The status refreshes every 10 seconds while the tab is visible; polling pauses when the tab is hidden.

**Why this output:** `refetchInterval` drives short polling. `refetchIntervalInBackground: false` avoids wasted requests when the user is not looking.

### Real-World Cases

- **Dashboards:** Polling for stats every few seconds.
- **Job status:** Polling a build or export status.
- **Fallback real-time:** Long polling when WebSockets are blocked.

### References

- TanStack Query — Polling — https://tanstack.com/query/latest/docs/framework/react/guides/polling
- MDN — HTTP long polling (conceptual) — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- Back4App — SSE vs. WebSockets vs. Polling (2026) — https://www.back4app.com/glossary/sse-vs-websockets-vs-polling/

---

## 9. Server-Sent Events (SSE)

### Definitions

**Core Definition:** Server-Sent Events (SSE) is a standard for unidirectional, server-to-client event streaming over a single HTTP connection.

**Technical Definition:** SSE uses the `EventSource` interface. The client creates `new EventSource(url)`, and the server responds with `Content-Type: text/event-stream`, sending text events in a simple wire format (`data:`, `event:`, `id:`, `retry:`). The browser automatically reconnects on connection loss, and the `Last-Event-ID` header supports resumption. SSE is **one-way** (server → client) and works over standard HTTP, so it benefits from HTTP/2 multiplexing and avoids the connection limits that affect HTTP/1.1 (six connections per domain). It is ideal for notifications, live feeds, and status updates.

**Beginner-Friendly Explanation:** SSE is a one-way stream from the server to the browser. The browser opens a connection, and the server keeps pushing text events. If the connection drops, the browser reconnects automatically.

### Purposes

- To push one-way real-time updates from server to client.
- To avoid polling overhead for notifications and feeds.
- To leverage automatic reconnection built into `EventSource`.
- To work over standard HTTP without WebSocket upgrade.
- To stream text-based events (JSON, logs, progress).

### Syntax Rules and Structure

**Client:**

```js
const events = new EventSource('/api/stream');

events.onmessage = (event) => {
  const data = JSON.parse(event.data);
  handleEvent(data);
};

events.addEventListener('ping', (event) => {
  console.log('Ping:', JSON.parse(event.data));
});

// Close when done
events.close();
```

**Server response format:**

```text
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"type":"message","text":"Hello"}

id: 42
event: ping
data: {"time":"2026-01-01T00:00:00Z"}

```

**Syntax Rules:**

1. The server must send `Content-Type: text/event-stream`.
2. Events are separated by a blank line; `data:` lines form the payload.
3. Use named events (`event:`) and `addEventListener` for typed events.
4. Include `id:` to support `Last-Event-ID` resumption.
5. Close the `EventSource` when the component unmounts or the stream is no longer needed.
6. Pass credentials if required: `new EventSource(url, { withCredentials: true })`.

**Constraints and Limitations:**

- SSE is one-way; the client cannot send messages over the same connection.
- Over HTTP/1.1, browsers limit six SSE connections per domain; HTTP/2 raises this to negotiated stream limits (default ~100).
- SSE only carries text; binary data must be encoded.
- Some proxies buffer SSE; configure buffering off.

### Annotated Code Examples

```tsx
function LiveFeed() {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    const source = new EventSource('/api/feed');

    source.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setEvents((prev) => [...prev, data]);
    };

    source.onerror = () => {
      // EventSource reconnects automatically
      console.warn('SSE connection lost, reconnecting…');
    };

    return () => source.close();
  }, []);

  return <FeedList items={events} />;
}
```

**Expected Output:** New events are appended to the feed as the server pushes them. If the connection drops, the browser reconnects automatically; unmounting closes the stream.

**Why this output:** `EventSource` opens a persistent HTTP stream. `onmessage` handles default events. The cleanup closes the connection.

### Real-World Cases

- **Notifications:** Pushing new alerts to the client.
- **Live scores:** Streaming score updates.
- **Logs and progress:** Streaming build logs or upload progress.

### References

- MDN — Using server-sent events — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events
- MDN — EventSource — https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- WHATWG — HTML: Server-sent events — https://html.spec.whatwg.org/multipage/server-sent-events.html

---

## 10. WebSockets

### Definitions

**Core Definition:** WebSockets is a protocol for full-duplex, bidirectional communication between the browser and a server over a single persistent connection.

**Technical Definition:** The `WebSocket` API opens a connection via an HTTP upgrade handshake (`Sec-WebSocket-Key`), after which both client and server can send messages at any time. The API provides `send()`, `onmessage`, `onopen`, `onclose`, and `onerror`, with `MessageEvent` and `CloseEvent` for payloads and closure metadata. WebSockets are **bidirectional** and binary-capable, making them suitable for chat, collaboration, multiplayer games, and trading. Unlike SSE, WebSockets do **not** support backpressure in the classic `WebSocket` interface; when messages arrive faster than the application can process them, memory or CPU issues can occur. For backpressure-sensitive use cases, the `WebTransport` API is recommended. Open WebSocket connections may also prevent the page from entering the bfcache, so closing connections when done is good practice.

**Beginner-Friendly Explanation:** WebSockets are a two-way pipe between the browser and the server. Both sides can send messages whenever they want, without polling.

### Purposes

- To enable real-time bidirectional communication (chat, collaboration).
- To push server updates and receive client actions over one connection.
- To support binary data (files, media, game state).
- To reduce per-message overhead compared with HTTP polling.
- To power multiplayer and live-editing experiences.

### Syntax Rules and Structure

**Client:**

```js
const ws = new WebSocket('wss://api.example.com/live');

ws.onopen = () => {
  ws.send(JSON.stringify({ type: 'subscribe', room: 'general' }));
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  handleMessage(message);
};

ws.onclose = (event) => {
  if (event.code !== 1000) {
    // Unexpected close; reconnect with backoff
  }
};

ws.onerror = (error) => {
  console.error('WebSocket error', error);
};

// Close when done
ws.close(1000, 'Component unmounted');
```

**Syntax Rules:**

1. Use `wss://` in production (TLS).
2. Handle `onopen`, `onmessage`, `onerror`, and `onclose`.
3. Use `close(1000)` for normal closure; `1000` is the normal-closure code.
4. Implement reconnection with exponential backoff for unexpected closes.
5. Send a subscription or authentication message after `onopen`.
6. Close the connection when the component unmounts to avoid leaks and bfcache issues.

**Constraints and Limitations:**

- No built-in backpressure; use `WebTransport` or application-level flow control for high-throughput streams.
- Stateful connections require server-side connection management and scaling (sticky sessions, pub/sub).
- Proxies and firewalls may block WebSocket upgrades.
- Open connections can prevent bfcache; close when not needed.

### Annotated Code Examples

```tsx
function LiveChat({ room }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/chat?room=${room}`);

    ws.onopen = () => {
      ws.send(JSON.stringify({ type: 'join', room }));
    };

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      setMessages((prev) => [...prev, message]);
    };

    ws.onerror = () => console.error('Chat connection error');

    return () => ws.close(1000, 'Unmounted');
  }, [room]);

  function sendMessage(text) {
    // The WebSocket instance must be accessible; use a ref in practice
  }

  return <ChatUI messages={messages} onSend={sendMessage} />;
}
```

**Expected Output:** Messages appear in the chat as the server pushes them. Changing `room` closes the old connection and opens a new one.

**Why this output:** The effect owns the connection lifecycle. `onmessage` appends messages. Cleanup closes the connection when the room changes or the component unmounts.

### Real-World Cases

- **Chat applications:** Real-time messaging.
- **Collaborative editing:** Shared document state.
- **Multiplayer games:** Low-latency state sync.
- **Trading platforms:** Live price and order updates.

### References

- MDN — WebSocket API — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
- MDN — Writing WebSocket client applications — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications
- MDN — WebTransport API — https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API

---

## 11. Streaming

### Definitions

**Core Definition:** Streaming is the progressive delivery of data or HTML in chunks, allowing the client to render content as it arrives rather than waiting for the complete response.

**Technical Definition:** Streaming operates at two levels. **HTTP response streaming** uses `fetch` with `response.body` (a `ReadableStream`) or `EventSource`/SSE to process chunks incrementally. **React streaming SSR** uses `renderToPipeableStream` (Node) or `renderToReadableStream` (Web/Edge) to emit HTML in chunks aligned with `<Suspense>` boundaries: the static shell (layouts, navigation, fallbacks) is sent immediately, and each boundary’s HTML is streamed when its data resolves, along with inline scripts that swap the fallback DOM node. React 19 stabilises native streaming SSR; the browser can paint and hydrate progressively without waiting for the entire document.

**Beginner-Friendly Explanation:** Streaming means sending the page or data in pieces. The browser shows the header and layout immediately, and slower sections (comments, recommendations) arrive later without blocking the rest.

### Purposes

- To improve Time to First Byte (TTFB) and first paint.
- To let users see and interact with fast content while slow content loads.
- To avoid blocking the entire page on a single slow query.
- To enable progressive hydration of interactive components.
- To process large or infinite data streams incrementally.

### Syntax Rules and Structure

**React streaming SSR (Node):**

```js
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/client.js'],
  onShellReady() {
    res.setHeader('Content-Type', 'text/html');
    pipe(res);
  },
});
```

**React streaming SSR (Edge/Web):**

```js
import { renderToReadableStream } from 'react-dom/server';

const stream = await renderToReadableStream(<App />, {
  bootstrapScripts: ['/client.js'],
});

return new Response(stream, { headers: { 'Content-Type': 'text/html' } });
```

**Client-side `fetch` stream:**

```js
const response = await fetch('/api/stream');
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  processChunk(decoder.decode(value, { stream: true }));
}
```

**Syntax Rules:**

1. Wrap slow sections in `<Suspense>` boundaries; each boundary is a streamable chunk.
2. Start piping in `onShellReady` so the shell reaches the browser immediately.
3. Use `renderToReadableStream` for edge/Web runtimes and `renderToPipeableStream` for Node.
4. For client-side streaming, read `response.body` with a `ReadableStreamDefaultReader`.
5. Handle backpressure in client-side streams (pause reading when the consumer is slow).
6. Hydrate the shell with `hydrateRoot` for progressive interactivity.

**Constraints and Limitations:**

- Streaming SSR requires a server runtime that supports chunked transfer.
- Errors after the shell is sent cannot change the HTTP status code.
- Client-side `fetch` streaming does not work with CORS unless the server exposes the stream.
- SSE is the higher-level abstraction for text event streams; raw `ReadableStream` is lower-level.

### Annotated Code Examples

```tsx
export default function ProductPage({ product }) {
  return (
    <main>
      <ProductInfo product={product} />
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews productId={product.id} />
      </Suspense>
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations productId={product.id} />
      </Suspense>
    </main>
  );
}
```

**Expected Output:** The product info renders immediately. Reviews and recommendations stream in as their data resolves, each with its own skeleton.

**Why this output:** Each `<Suspense>` boundary is a streamable chunk. React sends the shell first, then streams each boundary’s HTML when ready.

### Real-World Cases

- **E-commerce:** Streaming recommendations and reviews after product info.
- **Content sites:** Streaming comments after the article.
- **AI chat:** Streaming token-by-token responses.
- **Dashboards:** Streaming widgets as data loads.

### References

- Next.js — Streaming — https://nextjs.org/docs/app/guides/streaming
- React — renderToPipeableStream — https://react.dev/reference/react-dom/server/renderToPipeableStream
- React — renderToReadableStream — https://react.dev/reference/react-dom/server/renderToReadableStream
- MDN — ReadableStream — https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream

---

## References

- TanStack Query — Caching — https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query — Query Keys — https://tanstack.com/query/latest/docs/framework/react/guides/query-keys
- TanStack Query — Pagination — https://tanstack.com/query/latest/docs/framework/react/guides/pagination
- TanStack Query — Infinite Queries — https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
- TanStack Query — useInfiniteQuery — https://mintlify.wiki/TanStack/query/api/react/use-infinite-query
- TanStack Query — Optimistic Updates — https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates
- TanStack Query — Mutations — https://tanstack.com/query/latest/docs/framework/react/guides/mutations
- TanStack Query — Prefetching — https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
- TanStack Query — SSR — https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- TanStack Query — Window Focus Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query — Polling — https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query Overview — https://tanstack.com/query/v3
- Core Concepts and Performance (TanStack Query Best Practices) — https://github.com/gohypergiant/standard-toolkit
- MDN — Using server-sent events — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events
- MDN — EventSource — https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- WHATWG — HTML: Server-sent events — https://html.spec.whatwg.org/multipage/server-sent-events.html
- MDN — WebSocket API — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
- MDN — Writing WebSocket client applications — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications
- MDN — WebTransport API — https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API
- Back4App — SSE vs. WebSockets vs. Polling (2026) — https://www.back4app.com/glossary/sse-vs-websockets-vs-polling/
- Next.js — Streaming — https://nextjs.org/docs/app/guides/streaming
- React — renderToPipeableStream — https://react.dev/reference/react-dom/server/renderToPipeableStream
- React — renderToReadableStream — https://react.dev/reference/react-dom/server/renderToReadableStream
- MDN — ReadableStream — https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream
- MDN — Intersection Observer API — https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController