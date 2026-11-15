# SWR — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SWR (stale-while-revalidate) is a React Hooks library for data fetching that first returns cached (stale) data immediately, then sends the revalidation request in the background, and finally delivers the up-to-date data.

**Technical Definition:** SWR is a lightweight (~4–5 KB gzipped) React Hooks library built around the HTTP `stale-while-revalidate` cache strategy (RFC 5861). `useSWR(key, fetcher, options)` subscribes a component to a cache entry identified by a serializable key, returns the cached data immediately, and revalidates in the background on mount, window focus, network reconnect, and configurable intervals. The library provides request deduplication, automatic retries, mutation APIs (`mutate`, `useSWRMutation`), pagination (`useSWRInfinite`), optimistic updates with automatic rollback, React Suspense integration, and global configuration via `SWRConfig`. SWR is Vercel's minimalist alternative to TanStack Query, prioritising convention over configuration and a single-hook API.

**Beginner-Friendly Explanation:** SWR is a tool that makes fetching data in React simple and fast. You give it a unique key (usually the API URL) and a function that fetches the data. It immediately shows whatever is cached, then quietly checks the server for updates in the background. If the data changed, the screen updates. If not, nothing flickers. It also handles loading and error states, avoids fetching the same thing twice, and lets you update the cache after mutations — all with one hook.

### Key Characteristics

- **Stale-while-revalidate by default:** Cached data renders instantly; revalidation happens in the background.
- **Single-hook API:** `useSWR` covers the majority of data-fetching needs with minimal surface area.
- **Automatic revalidation:** On mount, window focus, and network reconnect.
- **Request deduplication:** Multiple components using the same key share one in-flight request (`dedupingInterval`, default 2 seconds).
- **Built-in retry:** Automatic error retry with exponential backoff (`shouldRetryOnError`, default `true`).
- **Mutation APIs:** Bound `mutate`, global `mutate`, and the dedicated `useSWRMutation` hook.
- **Optimistic updates with rollback:** `optimisticData`, `populateCache`, and `rollbackOnError` options.
- **Infinite loading:** `useSWRInfinite` with `getKey` and `setSize` for pagination.
- **Suspense support:** `suspense: true` for React Suspense integration.
- **Minimal bundle:** ~4–5.5 KB gzipped, roughly one-third the size of TanStack Query.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- JavaScript Promises and `async`/`await`.
- HTTP fundamentals: methods, status codes, JSON.
- Basic understanding of caching and stale-while-revalidate concepts.
- Familiarity with `fetch` or Axios (fetcher functions typically use them).

### Related Programming Areas

- **Server State Management** — the core problem SWR solves.
- **Data Fetching** — fetcher functions and request lifecycle.
- **Caching and Revalidation** — cache keys, `dedupingInterval`, focus/reconnect revalidation.
- **Mutations** — `mutate` and `useSWRMutation`.
- **Pagination** — `useSWRInfinite`.
- **Performance** — bundle size, deduplication, and background revalidation.

---

## 1. SWR & `useSWR`

### Definitions

**Core Definition:** `useSWR` is the primary hook for fetching and caching data, accepting a unique key and a fetcher function.

**Technical Definition:** `const { data, error, isLoading, isValidating, mutate } = useSWR(key, fetcher, options)` subscribes the component to the cache entry identified by `key`. The `key` is a string, function, array, or `null` (to disable fetching). The `fetcher` is an async function that receives the key and returns the data. On mount, SWR returns cached data if available, then revalidates. The returned `data` is the resolved fetcher result (or `undefined` while loading), `error` is the thrown error (or `undefined`), `isLoading` is true only when there is no loaded data and a request is in flight, and `isValidating` is true during any request or revalidation.

**Beginner-Friendly Explanation:** `useSWR` is the hook you call to get data. You give it a name (the key, usually the URL) and a function that fetches the data. It gives you back the data, any error, loading flags, and a function to manually update the cache.

### Purposes

- To fetch and cache server data with one hook.
- To return cached data instantly while revalidating in the background.
- To share cached data across components via a shared key.
- To automatically revalidate on focus, reconnect, and intervals.
- To deduplicate concurrent requests for the same key.
- To provide loading, error, and validation states.

### Syntax Rules and Structure

**General Syntax:**

```tsx
const { data, error, isLoading, isValidating, mutate } = useSWR(
  key,
  fetcher,
  options
);
```

**Component Breakdown:**

| Parameter / Return | Description |
|---------------------|-------------|
| `key` | Unique identifier: string, function, array, or `null` to disable |
| `fetcher` | Async function that receives the key and returns data |
| `options` | Per-hook configuration (see Section 2) |
| `data` | Cached or fetched data; `undefined` while loading |
| `error` | Error thrown by the fetcher; `undefined` on success |
| `isLoading` | `true` when there is no loaded data and a request is in flight |
| `isValidating` | `true` during any request or revalidation |
| `mutate` | Bound function to update this key's cache |

**Syntax Rules:**

1. The `key` must be stable across renders; use a string or an array of primitives.
2. Pass `null` or a falsy value as the key to conditionally disable fetching.
3. The `fetcher` is optional if a default fetcher is provided via `SWRConfig`.
4. `isLoading` is `false` when `fallbackData` or `keepPreviousData` provides data.
5. `isValidating` is `true` during background revalidation even when `isLoading` is `false`.

**Constraints and Limitations:**

- SWR does not reject on HTTP errors by default; the fetcher must throw manually (e.g., check `res.ok`).
- Keys are hashed deterministically; non-serializable values break caching.
- Data must not be `undefined`; return `null` for empty results.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Fetching with Loading and Error States

```tsx
import useSWR from 'swr';

const fetcher = async (url) => {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
};

function UserProfile({ userId }) {
  const { data, error, isLoading, isValidating, mutate } = useSWR(
    `/api/users/${userId}`,
    fetcher
  );

  if (isLoading) return <ProfileSkeleton />;
  if (error) return <ErrorBanner message={error.message} onRetry={mutate} />;
  if (!data) return <EmptyState />;

  return (
    <>
      {isValidating && <RefreshIndicator />}
      <ProfileCard user={data} />
    </>
  );
}
```

**Expected Output:** A skeleton on first load, an error banner with retry on failure, an empty state if no data, and the profile card on success. A subtle indicator appears during background revalidation.

**Why this output:** `useSWR` manages the request lifecycle. `isLoading` is true only on the first load. `isValidating` indicates background activity. `mutate` refetches on retry.

#### Example 2: Conditional Fetching with a `null` Key

```tsx
function UserPosts({ userId }) {
  const { data: posts, error } = useSWR(
    userId ? `/api/users/${userId}/posts` : null,
    fetcher
  );

  if (!userId) return <p>Select a user to see posts.</p>;
  if (error) return <ErrorBanner />;
  return <PostList posts={posts ?? []} />;
}
```

**Expected Output:** No request is made until `userId` is truthy. The component renders a prompt when no user is selected.

**Why this output:** SWR skips fetching when the key is `null`. The hook remains mounted and will fetch once the key becomes valid.

### Real-World Cases

- **User profiles:** Fetching user data by ID.
- **Dashboards:** Loading stats and widgets.
- **Detail pages:** Loading a resource by route parameter.

### References

- SWR — API — https://swr.vercel.app/docs/api
- SWR — Getting Started — https://swr.vercel.app/docs/getting-started
- Refine — How to Fetch Data in React with useSWR — https://refine.dev/blog/useswr/

---

## 2. SWR Configuration

### Definitions

**Core Definition:** SWR configuration is the set of per-hook and global options that control caching, revalidation, retries, and rendering behaviour.

**Technical Definition:** Configuration can be applied per hook via the `options` object in `useSWR`, or globally via `<SWRConfig value={...}>` or `useSWRConfig()`. Global configuration is merged with per-hook options, with per-hook options taking precedence. Key options include `dedupingInterval` (default 2000ms), `focusThrottleInterval` (5000ms), `loadingTimeout` (3000ms), `revalidateOnFocus` (true), `revalidateOnReconnect` (true), `shouldRetryOnError` (true), `errorRetryInterval` (5000ms), `refreshInterval` (0, disabled), `suspense` (false), and `keepPreviousData`.

**Beginner-Friendly Explanation:** Configuration lets you tune how SWR behaves — how long it waits before refetching, how often it retries on errors, whether it refreshes when you switch tabs, and more.

### Purposes

- To set a default fetcher for the entire app.
- To tune deduplication and revalidation intervals.
- To configure retry policy and error handling.
- To enable or disable focus, reconnect, and polling revalidation.
- To enable Suspense mode.
- To configure global error reporting and caching providers.

### Syntax Rules and Structure

**Global configuration:**

```tsx
import { SWRConfig } from 'swr';

<SWRConfig
  value={{
    fetcher: (url) => fetch(url).then((r) => r.json()),
    revalidateOnFocus: true,
    revalidateOnReconnect: true,
    shouldRetryOnError: true,
    errorRetryCount: 3,
    dedupingInterval: 2000,
  }}
>
  <App />
</SWRConfig>
```

**Per-hook options:**

```tsx
useSWR('/api/data', fetcher, {
  refreshInterval: 5000,
  revalidateOnFocus: false,
  suspense: true,
});
```

**Key options table:**

| Option | Default | Purpose |
|--------|---------|---------|
| `dedupingInterval` | `2000` | Deduplication window in ms |
| `focusThrottleInterval` | `5000` | Minimum time between focus revalidations |
| `loadingTimeout` | `3000` | Timeout before `onLoadingSlow` fires |
| `revalidateOnFocus` | `true` | Revalidate when window regains focus |
| `revalidateOnReconnect` | `true` | Revalidate when network reconnects |
| `shouldRetryOnError` | `true` | Retry on fetcher error |
| `errorRetryInterval` | `5000` | Retry interval in ms |
| `refreshInterval` | `0` | Polling interval (disabled by default) |
| `suspense` | `false` | Enable React Suspense mode |
| `keepPreviousData` | `false` | Return previous data while fetching new key |

**Syntax Rules:**

1. `SWRConfig` merges configuration from parent contexts; nested providers inherit and can override.
2. Use `useSWRConfig()` to access global `mutate`, `cache`, and configuration.
3. Per-hook options override global options.
4. `refreshInterval` can be a number (ms) or a function receiving the latest data and returning the interval.
5. `refreshWhenHidden` and `refreshWhenOffline` default to `false`.

**Constraints and Limitations:**

- Global defaults can be overridden by per-hook options; do not rely on a single global setting for all cases.
- `suspense: true` changes the loading model; the component suspends instead of rendering a loading branch.

### Annotated Code Examples

```tsx
// providers/SWRProvider.tsx
import { SWRConfig } from 'swr';

const fetcher = async (url: string) => {
  const res = await fetch(url, {
    headers: { 'Content-Type': 'application/json' },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
};

export function SWRProvider({ children }: { children: React.ReactNode }) {
  return (
    <SWRConfig
      value={{
        fetcher,
        revalidateOnFocus: true,
        revalidateOnReconnect: true,
        shouldRetryOnError: true,
        errorRetryCount: 3,
        dedupingInterval: 2000,
      }}
    >
      {children}
    </SWRConfig>
  );
}
```

**Expected Output:** Every `useSWR` call without an explicit fetcher uses the global fetcher. All hooks share the retry and deduplication settings.

**Why this output:** `SWRConfig` provides a global fetcher and defaults. Individual hooks can still override any option.

### Real-World Cases

- **App-wide fetcher:** A single authenticated fetcher configured once.
- **Polling dashboards:** `refreshInterval` set per hook.
- **Suspense-enabled apps:** `suspense: true` for route-level loading.

### References

- SWR — Global Configuration — https://swr.vercel.app/docs/global-configuration
- SWR — API Options — https://swr.vercel.app/docs/api
- SWR — Best Practices (Configuration) — https://github.com/mindrally/skills/blob/7682ca77710e0971eab4e0ae5dddfa281aea0ba5/swr/SKILL.md

---

## 3. SWR Caching & Revalidation

### Definitions

**Core Definition:** SWR caching stores fetched data by key; revalidation refreshes it in the background using the stale-while-revalidate strategy.

**Technical Definition:** SWR maintains an in-memory cache keyed by the `useSWR` key. When a component mounts, SWR returns cached data immediately (if present), then triggers a background revalidation unless `revalidateIfStale` is `false`. Revalidation is triggered on mount, window focus (`revalidateOnFocus`), network reconnect (`revalidateOnReconnect`), and optional polling (`refreshInterval`). The `dedupingInterval` window prevents duplicate requests for the same key. Revalidation can be disabled per hook or globally. SWR's cache is provider-scoped; multiple caches can be configured for isolation.

**Beginner-Friendly Explanation:** SWR remembers data in a cache. When you visit a page again, it shows the cached data instantly and quietly checks the server for updates. If the data changed, it updates the screen. If not, nothing flickers.

### Purposes

- To serve cached data instantly on repeat visits.
- To keep data fresh without blocking the UI.
- To avoid duplicate requests within the deduplication window.
- To recover from network interruptions automatically.
- To support polling for near-real-time data.
- To allow manual revalidation via `mutate()`.

### Syntax Rules and Structure

**Revalidation triggers:**

```tsx
useSWR('/api/data', fetcher, {
  revalidateOnFocus: true,       // default
  revalidateOnReconnect: true,   // default
  refreshInterval: 30_000,       // optional polling
  dedupingInterval: 2_000,       // default
});
```

**Manual revalidation:**

```tsx
const { mutate } = useSWR('/api/data', fetcher);
mutate(); // trigger revalidation
```

**Syntax Rules:**

1. `revalidateIfStale` (default `true`) controls whether stale data triggers background revalidation.
2. `revalidateOnMount` can explicitly enable or disable mount revalidation.
3. `mutate(key)` without data triggers revalidation for that key.
4. `mutate` broadcasts to all hooks under the same cache provider; without a provider, it broadcasts to all hooks.
5. `focusThrottleInterval` prevents excessive revalidation on rapid focus changes.

**Constraints and Limitations:**

- Cached data can be stale; SWR intentionally serves it while revalidating.
- Revalidation on every focus can be excessive for some screens; disable per hook.
- `refreshInterval` polling wastes requests for data that does not change often.

### Annotated Code Examples

```tsx
function LiveDashboard() {
  const { data, isValidating, mutate } = useSWR('/api/stats', fetcher, {
    refreshInterval: 30_000,
    revalidateOnFocus: true,
    dedupingInterval: 5_000,
  });

  return (
    <div>
      {isValidating && <RefreshIndicator />}
      <StatsGrid stats={data} />
      <button onClick={() => mutate()}>Refresh now</button>
    </div>
  );
}
```

**Expected Output:** Stats refresh every 30 seconds and when the window regains focus. The refresh indicator appears during background revalidation. The manual button triggers an immediate revalidation.

**Why this output:** `refreshInterval` drives polling. `revalidateOnFocus` refreshes on tab return. `mutate()` triggers manual revalidation.

### Real-World Cases

- **Dashboards:** Polling stats every 30 seconds.
- **Notifications:** Focus-based revalidation.
- **Offline recovery:** Reconnect revalidation.

### References

- SWR — Revalidation — https://swr.vercel.app/docs/revalidation
- SWR — API Options — https://swr.vercel.app/docs/api
- ADR-067: Adopt SWR for client-side data caching — https://github.com

---

## 4. SWR Mutations

### Definitions

**Core Definition:** SWR mutations are APIs for changing remote data and updating the local cache.

**Technical Definition:** SWR provides three mutation mechanisms: **bound `mutate`** (returned by `useSWR`, pre-bound to the hook's key), **global `mutate`** (via `useSWRConfig()` or the `swr` import, accepts any key), and **`useSWRMutation`** (a dedicated hook that returns `trigger`, `isMutating`, `error`, `reset`, and `data`; it does not fetch on render and is designed for triggered writes). All mutation APIs support optimistic updates, cache population, and rollback on error.

**Beginner-Friendly Explanation:** Mutations are how you change data on the server and update the cache. You can use the `mutate` function returned by `useSWR`, the global `mutate`, or the dedicated `useSWRMutation` hook for triggered writes.

### Purposes

- To update remote data and refresh the cache.
- To perform optimistic updates with rollback.
- To avoid race conditions between reads and writes.
- To trigger writes imperatively with `useSWRMutation`.
- To populate the cache with mutation results.

### Syntax Rules and Structure

**Bound `mutate`:**

```tsx
const { data, mutate } = useSWR('/api/user', fetcher);
mutate({ ...data, name: 'New Name' }, { revalidate: true });
```

**Global `mutate`:**

```tsx
import { useSWRConfig } from 'swr';
const { mutate } = useSWRConfig();
mutate('/api/user'); // trigger revalidation
```

**`useSWRMutation`:**

```tsx
import useSWRMutation from 'swr/mutation';

async function updateUser(url, { arg }) {
  return fetch(url, { method: 'PATCH', body: JSON.stringify(arg) });
}

const { trigger, isMutating, error, reset } = useSWRMutation(
  '/api/user',
  updateUser
);

<button onClick={() => trigger({ name: 'New' })} disabled={isMutating}>
  {isMutating ? 'Saving…' : 'Save'}
</button>
```

**Syntax Rules:**

1. Bound `mutate` does not require the key.
2. Global `mutate` requires the key; without data, it triggers revalidation.
3. `useSWRMutation` does not fetch on render; it waits for `trigger` to be called.
4. `trigger` rejects on failure and also populates `error`.
5. `reset()` clears `data` and `error`.
6. Use `onSuccess` / `onError` callbacks for side effects.

**Constraints and Limitations:**

- Global `mutate` without a mounted hook for the key does not update the cache.
- `useSWRMutation` is designed for triggered writes, not for read-on-mount.
- Optimistic updates require careful handling of concurrent mutations.

### Annotated Code Examples

```tsx
import useSWRMutation from 'swr/mutation';

async function createPost(url, { arg }) {
  const res = await fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(arg),
  });
  if (!res.ok) throw new Error('Failed to create post');
  return res.json();
}

function CreatePostForm() {
  const { trigger, isMutating, error, reset } = useSWRMutation(
    '/api/posts',
    createPost
  );

  async function handleSubmit(e) {
    e.preventDefault();
    const form = new FormData(e.currentTarget);
    await trigger({ title: form.get('title') });
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && (
        <div role="alert">
          <span>{error.message}</span>
          <button type="button" onClick={reset}>Dismiss</button>
        </div>
      )}
      <input name="title" disabled={isMutating} />
      <button type="submit" disabled={isMutating}>
        {isMutating ? 'Creating…' : 'Create Post'}
      </button>
    </form>
  );
}
```

**Expected Output:** Submitting the form calls `trigger` with the form data. The button disables while mutating. Errors appear with a dismiss button that calls `reset()`.

**Why this output:** `useSWRMutation` provides `trigger` for imperative mutation, `isMutating` for pending state, `error` for failure, and `reset` to clear the error.

### Real-World Cases

- **Form submissions:** Creating and updating records.
- **Delete actions:** Removing items with confirmation.
- **Toggle actions:** Likes, follows, and switches.

### References

- SWR — Mutation & Revalidation — https://swr.vercel.app/docs/mutation
- SWR 2.0 Announcement — https://swr.vercel.app/blog/swr-v2
- SWR — Mutation Examples (agents-inc) — https://github.com/agents-inc/skills

---

## 5. SWR Pagination & Infinite Loading

### Definitions

**Core Definition:** SWR pagination uses `useSWR` for numbered pages and `useSWRInfinite` for accumulating lists (infinite loading).

**Technical Definition:** For numbered pagination, `useSWR` with the page index in the key is sufficient; each page is cached independently. For infinite loading, `useSWRInfinite` accepts a `getKey` function that receives the page index and previous page data, and returns the key for that page (or `null` to signal the end). It returns `data` (an array of page results), `size` (the number of pages), `setSize` (to control how many pages to load), and standard SWR states. `getKey` is the core mechanism that makes `useSWRInfinite` flexible for index-based, cursor-based, and offset-based pagination.

**Beginner-Friendly Explanation:** For pages with "Previous" and "Next" buttons, plain `useSWR` with the page number in the URL is enough. For infinite scrolling, where pages accumulate, use `useSWRInfinite` — it fetches the first page, then loads more as `setSize` increases.

### Purposes

- To implement numbered pagination with cached pages.
- To implement infinite scrolling with accumulating pages.
- To support cursor-based, offset-based, and index-based pagination.
- To signal the end of a list by returning `null` from `getKey`.
- To prefetch the next page using SWR's cache.

### Syntax Rules and Structure

**Numbered pagination with `useSWR`:**

```tsx
function Page({ index }) {
  const { data } = useSWR(`/api/data?page=${index}`, fetcher);
  return data?.map((item) => <div key={item.id}>{item.name}</div>);
}
```

**Infinite loading with `useSWRInfinite`:**

```tsx
import useSWRInfinite from 'swr/infinite';

const getKey = (pageIndex, previousPageData) => {
  if (previousPageData && !previousPageData.nextCursor) return null;
  if (pageIndex === 0) return '/api/posts';
  return `/api/posts?cursor=${previousPageData.nextCursor}`;
};

function InfinitePosts() {
  const { data, size, setSize, isLoading } = useSWRInfinite(getKey, fetcher);

  if (isLoading) return <Skeleton />;

  const posts = data ? data.flat() : [];

  return (
    <div>
      {posts.map((post) => <PostCard key={post.id} post={post} />)}
      <button onClick={() => setSize(size + 1)}>Load More</button>
    </div>
  );
}
```

**Syntax Rules:**

1. `getKey(pageIndex, previousPageData)` must return a key or `null` to stop.
2. `data` is an array of page results; flatten it for rendering.
3. `size` is the number of pages to load; `setSize(size + 1)` loads the next page.
4. `isLoading` is true only for the first page; use `isValidating` for subsequent pages.
5. Returning `null` from `getKey` signals that the list has ended.

**Constraints and Limitations:**

- `useSWRInfinite` does not automatically trigger `setSize` on scroll; you must combine it with an intersection observer or a "Load More" button.
- Offset-based infinite loading is unstable when items are inserted or deleted; prefer cursor-based.
- All pages are cached together; very long lists accumulate memory.

### Annotated Code Examples

```tsx
import useSWRInfinite from 'swr/infinite';

const getKey = (pageIndex, previousPageData) => {
  if (previousPageData && !previousPageData.nextCursor) return null;
  if (pageIndex === 0) return '/api/feed';
  return `/api/feed?cursor=${previousPageData.nextCursor}`;
};

function InfiniteFeed() {
  const {
    data, size, setSize, isLoading, isValidating,
  } = useSWRInfinite(getKey, fetcher);

  const items = data?.flatMap((page) => page.items) ?? [];

  return (
    <div>
      {items.map((item) => <FeedCard key={item.id} item={item} />)}
      <button
        onClick={() => setSize(size + 1)}
        disabled={isValidating || (data && !data[data.length - 1]?.nextCursor)}
      >
        {isValidating ? 'Loading…' : 'Load More'}
      </button>
    </div>
  );
}
```

**Expected Output:** The first page loads, and clicking "Load More" fetches the next page using the previous page's cursor. The button disables when the list is exhausted.

**Why this output:** `getKey` uses `previousPageData.nextCursor` to build the next key and returns `null` when there is no cursor. `setSize` increases the number of pages to load.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Search results:** Load more results.
- **Chat history:** Loading older messages.

### References

- SWR — Pagination — https://swr.vercel.app/docs/pagination
- SWR — Pagination Examples (agents-inc) — https://github.com/agents-inc/skills
- SWR — Infinite Loading — https://swr.vercel.app/docs/pagination#infinite-loading

---

## 6. SWR Optimistic Updates

### Definitions

**Core Definition:** SWR optimistic updates apply a mutation's effect to the cache immediately, before the server confirms, with automatic rollback on failure.

**Technical Definition:** SWR 2.0 added `optimisticData`, `populateCache`, and `rollbackOnError` options to `mutate` and `useSWRMutation`. `optimisticData` (a value or updater function) is written to the cache immediately. `populateCache: true` writes the remote mutation result to the cache after completion. `rollbackOnError: true` (default) restores the previous cache value if the mutation fails. `useSWRMutation` also avoids race conditions between `useSWR` and other mutations of the same resource.

**Beginner-Friendly Explanation:** Optimistic updates make the UI feel instant. You change the screen immediately, assume the server will succeed, and undo the change if it fails.

### Purposes

- To provide instant feedback for mutations.
- To reduce perceived latency.
- To improve UX for likes, follows, and toggles.
- To maintain correctness with automatic rollback on failure.
- To avoid race conditions between reads and writes.

### Syntax Rules and Structure

```tsx
mutate(updateFn, {
  optimisticData: (current) => ({ ...current, liked: true }),
  populateCache: true,
  rollbackOnError: true,
  revalidate: false,
});
```

**Syntax Rules:**

1. `optimisticData` is applied to the cache immediately.
2. `populateCache: true` writes the mutation result to the cache after completion.
3. `rollbackOnError: true` (default) restores the previous value on failure.
4. `revalidate: false` prevents a refetch after the mutation.
5. `useSWRMutation`'s `trigger` accepts the same options.

**Constraints and Limitations:**

- Optimistic updates are a correctness trade-off; not suitable for operations where failure is common.
- Rollback must handle concurrent mutations carefully.
- The optimistic value must match the shape of the cached data.

### Annotated Code Examples

```tsx
function LikeButton({ postId }) {
  const { data, mutate } = useSWR(`/api/posts/${postId}`, fetcher);

  async function handleLike() {
    await mutate(
      fetch(`/api/posts/${postId}/like`, { method: 'POST' }),
      {
        optimisticData: (current) => ({
          ...current,
          likedByMe: true,
          likes: current.likes + 1,
        }),
        populateCache: true,
        rollbackOnError: true,
        revalidate: false,
      }
    );
  }

  return (
    <button onClick={handleLike} aria-pressed={data?.likedByMe}>
      {data?.likedByMe ? '♥' : '♡'} {data?.likes}
    </button>
  );
}
```

**Expected Output:** Clicking the like button immediately updates the heart and count. If the request fails, the heart and count revert. If it succeeds, the server result is written to the cache.

**Why this output:** `optimisticData` applies the new state instantly. `rollbackOnError` restores the previous state on failure. `populateCache` stores the server result after success.

### Real-World Cases

- **Likes and reactions:** Toggling instantly.
- **Todo lists:** Adding items before the server responds.
- **Follow buttons:** Updating follower counts immediately.

### References

- SWR — Mutation & Revalidation (Optimistic UI) — https://swr.vercel.app/docs/mutation
- SWR 2.0 Announcement (Optimistic UI) — https://swr.vercel.app/blog/swr-v2
- SWR Optimistic Updates Reference — https://github.com/vercel/swr

---

## 7. SWR vs TanStack Query

### Definitions

**Core Definition:** SWR and TanStack Query are both server-state libraries for React, but SWR is the minimalist, Vercel-aligned choice, while TanStack Query is the feature-rich, framework-agnostic choice.

**Technical Definition:** SWR (~4–5.5 KB gzipped) provides a single-hook API with automatic revalidation, deduplication, mutations, and optimistic updates, prioritising convention over configuration. TanStack Query (~13–16 KB gzipped) provides a query-client architecture with richer invalidation, devtools, mutation lifecycle callbacks, query key factories, and framework-agnostic support. SWR is intentionally smaller and more opinionated; it does not ship a query client you instantiate and does not have a persistent mutation cache, leaning on refetching when in doubt. TanStack Query offers more machinery for complex mutation flows (optimistic update + rollback + invalidate-by-prefix).

**Beginner-Friendly Explanation:** SWR is the light, simple option — perfect for dashboards and read-heavy apps inside the Vercel/Next.js ecosystem. TanStack Query is the heavier, more powerful option — better for complex apps with many mutations, granular invalidation, and devtools.

### Purposes

- To choose the right library for the project's complexity and ecosystem.
- To understand the trade-offs in bundle size and features.
- To plan migrations between the two.
- To align the choice with the framework (Next.js vs. framework-agnostic).

### Comparison Table

| Dimension | SWR 2.x | TanStack Query v5 |
|-----------|---------|-------------------|
| **Bundle size (gzipped)** | ~4–5.5 KB | ~13–16 KB |
| **Philosophy** | Minimalist, convention-over-configuration | Feature-rich, explicit configuration |
| **Cache model** | Key-based, simpler | Key-based with query client and key factories |
| **Mutations** | `mutate` + `useSWRMutation` | `useMutation` with `onMutate`/`onError`/`onSettled` |
| **Optimistic updates** | `optimisticData`/`rollbackOnError` | Manual `onMutate`/rollback pattern |
| **Devtools** | SWR DevTools (2.0+) | TanStack Query Devtools (mature) |
| **Framework support** | React, Next.js-aligned | React, Vue, Solid, Svelte, Angular |
| **Best for** | Read-heavy dashboards, Vercel stack | Complex mutation flows, large apps |

**Syntax Rules:**

1. Choose SWR for minimal API surface and Vercel/Next.js alignment.
2. Choose TanStack Query for richer invalidation, devtools, and framework-agnostic use.
3. Both are production-grade; neither is universally "better."
4. SWR's smaller bundle is a decisive advantage for performance-sensitive apps.
5. TanStack Query's mutation lifecycle is cleaner for complex optimistic workflows.

### Annotated Code Examples

**SWR (minimal):**

```tsx
const { data, mutate } = useSWR('/api/todos', fetcher);
mutate(updateTodo, { optimisticData, rollbackOnError: true });
```

**TanStack Query (explicit lifecycle):**

```tsx
useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previous = queryClient.getQueryData(['todos']);
    queryClient.setQueryData(['todos'], newTodo);
    return { previous };
  },
  onError: (_err, _newTodo, context) => {
    queryClient.setQueryData(['todos'], context.previous);
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
});
```

**Expected Output:** Both achieve optimistic updates and rollback, but SWR does it with two options, while TanStack Query requires explicit lifecycle callbacks.

**Why this output:** SWR's `optimisticData`/`rollbackOnError` are declarative; TanStack Query's `onMutate`/`onError`/`onSettled` give full control.

### Real-World Cases

- **Vercel/Next.js dashboards:** SWR.
- **Large enterprise apps:** TanStack Query.
- **Bundle-sensitive marketing pages:** SWR's smaller footprint.

### References

- Refine — React Query vs TanStack Query vs SWR: A 2025 Comparison — https://refine.dev/blog/react-query-vs-tanstack-query-vs-swr-2025/
- Kanopy Labs — TanStack Query vs SWR vs Apollo Client: Data Fetching in 2026 — https://kanopylabs.com/blog/tanstack-query-vs-swr-vs-apollo
- TanStack Query — Comparison — https://tanstack.com/query/latest/docs/framework/react/comparison
- saschb2b/skills — SWR vs TanStack Query — https://github.com/saschb2b/skills
- PkgPulse — TanStack Query v5 vs SWR v3 vs RTK Query 2026 — https://www.pkgpulse.com

---

## 8. SWR Best Practices

### Definitions

**Core Definition:** SWR best practices are the guidelines for structuring keys, configuring revalidation, handling mutations, and keeping the server-state layer predictable.

**Technical Definition:** Best practices combine stable, serializable keys (strings or arrays of primitives), conditional fetching with `null` keys, global fetcher and retry configuration via `SWRConfig`, proper loading/error/validation state handling, optimistic updates with rollback, Suspense integration where appropriate, and avoiding anti-patterns such as unstable keys, over-polling, or mutating the cache without `mutate`. TypeScript is recommended for full type safety.

**Beginner-Friendly Explanation:** Good SWR usage means giving each request a stable name, configuring sensible retries and refresh intervals, handling all the states (loading, error, revalidating), and updating the cache correctly after changes.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use stable keys** | Strings or arrays of primitives; never objects or functions that change every render |
| **Use `null` keys for conditional fetching** | Skip fetching until dependencies are ready |
| **Configure a global fetcher** | Avoid repeating the fetcher in every hook |
| **Handle all states** | `isLoading`, `error`, `isValidating`, and empty data |
| **Use `isValidating` for background indicators** | `isLoading` is only for the first load |
| **Tune `dedupingInterval`** | Default 2 seconds; increase for slow-changing data |
| **Tune `refreshInterval` per hook** | Do not poll everything globally |
| **Use `useSWRMutation` for triggered writes** | Avoid race conditions between reads and writes |
| **Use optimistic updates with rollback** | `optimisticData` + `rollbackOnError` |
| **Use `suspense: true` where appropriate** | Integrates with React Suspense boundaries |
| **Use TypeScript** | Full type safety for data, errors, and fetcher arguments |
| **Prefer SWR for read-heavy apps** | Use TanStack Query for complex mutation flows |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Stable key + conditional fetching
const { data, error, isLoading, isValidating } = useSWR(
  userId ? `/api/users/${userId}` : null,
  fetcher,
  { revalidateOnFocus: true, dedupingInterval: 5000 }
);

// useSWRMutation for writes
const { trigger, isMutating } = useSWRMutation(
  '/api/posts',
  createPost,
  { onSuccess: () => mutate('/api/posts') }
);

// Optimistic update with rollback
await mutate(updateLike(postId), {
  optimisticData: (current) => ({ ...current, liked: true, likes: current.likes + 1 }),
  rollbackOnError: true,
  populateCache: true,
});
```

**❌ Bad Practices:**

```tsx
// ❌ Unstable key: new object every render
useSWR({ url: '/api/data', page }, fetcher);

// ❌ Missing error handling
const { data } = useSWR('/api/data', fetcher);
return <div>{data.name}</div>; // Crashes on undefined

// ❌ Polling everything globally
<SWRConfig value={{ refreshInterval: 1000 }}>

// ❌ Global mutate without a mounted hook
mutate('/api/data'); // Does nothing if no hook is mounted
```

**Expected Output:** The good practices produce stable caching, correct states, and safe mutations. The bad practices cause cache misses, crashes, excessive requests, and silent no-ops.

**Why this output:** Stable keys enable deduplication and cache hits. Error handling prevents crashes. Global polling wastes requests. Global mutate requires a mounted hook to update the cache.

### Real-World Cases

- **All SWR applications:** These practices apply universally.
- **Next.js dashboards:** Global fetcher and retry configuration.
- **Mutation-heavy apps:** `useSWRMutation` with optimistic updates.

### References

- SWR — Best Practices (mindrally) — https://github.com/mindrally/skills/blob/7682ca77710e0971eab4e0ae5dddfa281aea0ba5/swr/SKILL.md
- SWR — Patterns (agents-inc) — https://github.com/agents-inc/skills
- Next.js — How to fetch client-side data with SWR — https://nextjs.org/docs/app/guides/client-side-data-fetching
- CoreUI — How to use SWR in React — https://coreui.io

---

## References

- SWR — Official Documentation — https://swr.vercel.app/docs/getting-started
- SWR — API — https://swr.vercel.app/docs/api
- SWR — Global Configuration — https://swr.vercel.app/docs/global-configuration
- SWR — Revalidation — https://swr.vercel.app/docs/revalidation
- SWR — Mutation & Revalidation — https://swr.vercel.app/docs/mutation
- SWR — Pagination — https://swr.vercel.app/docs/pagination
- SWR — Infinite Loading — https://swr.vercel.app/docs/pagination#infinite-loading
- SWR — Suspense — https://swr.vercel.app/docs/suspense
- SWR — 2.0 Announcement — https://swr.vercel.app/blog/swr-v2
- SWR — Mutation Examples (agents-inc) — https://github.com/agents-inc/skills
- SWR — Pagination Examples (agents-inc) — https://github.com/agents-inc/skills
- SWR — Best Practices (mindrally) — https://github.com/mindrally/skills
- SWR — Patterns (agents-inc) — https://github.com/agents-inc/skills
- Refine — How to Fetch Data in React with useSWR — https://refine.dev/blog/useswr/
- Refine — React Query vs TanStack Query vs SWR: A 2025 Comparison — https://refine.dev/blog/react-query-vs-tanstack-query-vs-swr-2025/
- Kanopy Labs — TanStack Query vs SWR vs Apollo Client: Data Fetching in 2026 — https://kanopylabs.com/blog/tanstack-query-vs-swr-vs-apollo
- TanStack Query — Comparison — https://tanstack.com/query/latest/docs/framework/react/comparison
- saschb2b/skills — SWR vs TanStack Query — https://github.com/saschb2b/skills
- PkgPulse — TanStack Query v5 vs SWR v3 vs RTK Query 2026 — https://www.pkgpulse.com
- Next.js — How to fetch client-side data with SWR — https://nextjs.org/docs/app/guides/client-side-data-fetching
- CoreUI — How to use SWR in React — https://coreui.io
- ADR-067: Adopt SWR for client-side data caching — https://github.com
- SWR Optimistic Updates Reference — https://github.com/vercel/swr
- SWR Error Handling Examples — https://raw.githubusercontent.com/vercel/swr