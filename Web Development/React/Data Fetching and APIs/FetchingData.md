# React Data Fetching Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Data fetching is the process by which a React application requests, receives, caches, and renders data from external sources such as APIs, databases, or files.

**Technical Definition:** Data fetching in React spans three architectural layers. The **transport layer** handles the request itself — HTTP via `fetch`, GraphQL, WebSockets, or server functions. The **state layer** tracks the request's lifecycle: idle, loading, success, error, and the cached data associated with each key. The **rendering layer** determines how data reaches components — through props, hooks (`useEffect` + `fetch`), query libraries (TanStack Query, SWR), router data APIs (React Router loaders, TanStack Router loaders), or Server Components. React itself provides no built-in data-fetching primitive. React 19 adds the `use()` Hook for reading Promises during render and makes Server Components stable. The chosen approach determines caching behaviour, loading UX, error handling, and whether data is fetched on the client, at build time, or on the server per request.

**Beginner-Friendly Explanation:** Data fetching is how your app gets information from somewhere else — an API, a database, a file — and shows it on screen. It sounds simple, but there are many decisions: *where* does the fetch happen (browser or server)? *When* does it happen (on every request, at build time, or when the user clicks)? *What does the user see while waiting* (spinner, skeleton, nothing)? *What happens if it fails* (error message, retry)? And *how do you avoid fetching the same thing twice*? This cheat sheet walks through every stage of that journey.

### Key Characteristics

- **No built-in primitive:** React provides no official data-fetching API; the ecosystem supplies patterns, libraries, and framework primitives.
- **Multiple architectural layers:** Transport, state, and rendering are distinct concerns with distinct tools.
- **Caching is central:** Every modern approach caches data and needs a policy for freshness, invalidation, and garbage collection.
- **A request is a state machine:** Requests move through idle → loading → success/error, and UI must handle every state.
- **Server vs. client trade-offs:** Client fetching is simple but waterfall-prone; server fetching improves first paint but adds infrastructure.
- **Streaming and Suspense:** Modern React streams server-rendered content progressively instead of blocking the whole page.
- **Version-sensitive:** Next.js 15 changed `fetch` caching defaults; TanStack Query v5 renamed `cacheTime` to `gcTime`; Pages Router data functions are legacy.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- JavaScript Promises, `async`/`await`, and `try`/`catch`.
- HTTP basics: `fetch`, status codes, JSON, and request cancellation.
- Basic understanding of server-side rendering and static generation concepts.
- Familiarity with a data-fetching library (helpful but not required).

### Related Programming Areas

- **State Management** — server state vs. client state vs. URL state.
- **Caching** — HTTP cache, application cache, and stale-while-revalidate.
- **Performance** — request waterfalls, bundle size, and streaming.
- **Error Handling** — error boundaries, retries, and fallbacks.
- **React Server Components** — server-first data fetching.

---

## 1. Data Fetching Lifecycle

### Definitions

**Core Definition:** The data fetching lifecycle is the sequence of stages a request passes through from initiation to rendering: trigger, request, pending, resolution (success or error), cache update, and render.

**Technical Definition:** Every data request in React follows a lifecycle: (1) a **trigger** initiates the request (component mount, route navigation, user action, or server render); (2) the **request** is dispatched with parameters, headers, and cancellation signals; (3) the request enters a **pending** state while the network or server processes it; (4) it **resolves** with data or rejects with an error; (5) the result is written to a **cache** keyed by request identity; and (6) components **render** with the new data, loading state, or error state. Libraries and frameworks automate parts of this lifecycle, but the stages themselves are universal.

**Beginner-Friendly Explanation:** Every time your app needs data, it goes through a journey: something triggers the request, the request is sent, you wait, the answer comes back (or an error), the answer is stored, and the screen updates. Understanding this journey helps you decide where to put loading spinners, error messages, and caching.

### Purposes

- To understand the full journey of a request so every stage is handled.
- To identify where loading, error, and empty states belong.
- To choose the right tool for each stage (transport, cache, render).
- To avoid waterfalls by understanding trigger and resolution ordering.
- To design cancellation and cleanup for requests that are no longer needed.

### Syntax Rules and Structure

**Lifecycle Stages:**

| Stage | Description | Typical Handling |
|-------|-------------|------------------|
| Trigger | Component mount, navigation, user action, server render | `useEffect`, loader, event handler |
| Request | Dispatch with params and signal | `fetch(url, { signal })`, query function |
| Pending | Waiting for response | `isPending`, `isLoading`, `Suspense` fallback |
| Resolution | Data or error | `.then`, `await`, `onError` |
| Cache | Store result by key | Query cache, HTTP cache, framework cache |
| Render | UI reflects result | Conditional rendering, error boundary |

**Syntax Rules:**

1. Every request needs a unique, serialisable identity (URL + params, or a query key) so the cache can store and deduplicate it.
2. Pending state must be represented in the UI, not left as a blank screen.
3. Errors must be caught and surfaced, not swallowed.
4. Requests that are no longer needed must be cancelled or ignored.
5. The cache must be invalidated when the underlying data changes.

**Constraints and Limitations:**

- Client-side fetching in `useEffect` creates a waterfall: render → effect → fetch → render.
- Without cancellation, late responses can overwrite newer data (race conditions).
- Without caching, navigating back re-fetches everything.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Manual Lifecycle with `useEffect`

```jsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
  // Stage 3: state represents pending, success, and error
  const [data, setData] = useState(null);
  const [isPending, setIsPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Stage 1: trigger on mount or userId change
    const controller = new AbortController();
    setIsPending(true);
    setError(null);

    // Stage 2: dispatch request
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => {
        // Stage 4: resolution success
        setData(json);
      })
      .catch((err) => {
        // Stage 4: resolution error (ignore aborts)
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => {
        setIsPending(false);
      });

    // Stage 5: cleanup cancels stale requests
    return () => controller.abort();
  }, [userId]);

  // Stage 6: render reflects the state
  if (isPending) return <p>Loading…</p>;
  if (error) return <p role="alert">Failed to load profile.</p>;
  if (!data) return <p>No profile found.</p>;
  return <h1>{data.name}</h1>;
}
```

**Expected Output:** A loading message appears while the request is pending. The user's name renders on success. An error message renders on failure. Navigating away cancels the request.

**Why this output:** `useEffect` triggers the fetch and owns the lifecycle. State variables represent pending, error, and success. The cleanup aborts the request when `userId` changes or the component unmounts, preventing stale updates.

#### Example 2: Automated Lifecycle with TanStack Query

```tsx
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isPending, isError, error, refetch } = useQuery({
    queryKey: ['user', userId],       // Cache identity
    queryFn: ({ signal }) =>
      fetch(`/api/users/${userId}`, { signal }).then((res) => res.json()),
    staleTime: 5 * 60 * 1000,          // Fresh for 5 minutes
  });

  if (isPending) return <p>Loading…</p>;
  if (isError) return <button onClick={refetch}>Retry: {error.message}</button>;
  return <h1>{data.name}</h1>;
}
```

**Expected Output:** The same three states render, but the library handles caching, deduplication, cancellation, and background refetching.

**Why this output:** `useQuery` automates the request, pending, resolution, and cache stages. `queryKey` is the cache identity. `staleTime` controls freshness. `refetch` retries on error.

### Real-World Cases

- **User profiles:** Fetching user data on mount with loading and error states.
- **Search results:** Triggering a request on query change with cancellation.
- **Dashboards:** Fetching multiple independent resources in parallel.

### References

- React — Fetching Data (Legacy Docs) — https://legacy.reactjs.org/docs/faq-ajax.html
- React — You Might Not Need an Effect: Fetching data — https://react.dev/learn/you-might-not-need-an-effect
- TanStack Query — Overview — https://tanstack.com/query/latest/docs/framework/react/overview
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API

---

## 2. Async Functions

### Definitions

**Core Definition:** An async function is a JavaScript function declared with `async` that always returns a Promise and allows `await` to pause execution until another Promise settles.

**Technical Definition:** `async function` is syntactic sugar over Promises. Calling an async function returns a Promise that resolves with the function's return value or rejects with a thrown error. `await` suspends the function's execution until the awaited Promise settles, without blocking the main thread. Async functions are the foundation of modern data fetching: `fetch` returns a Promise, and `async`/`await` makes sequential and parallel request code readable. `try`/`catch` handles rejected Promises. `Promise.all` runs independent requests in parallel.

**Beginner-Friendly Explanation:** An async function is a function that can wait for something slow (like a network request) without freezing the whole page. You mark it `async`, and you use `await` to say "pause here until this finishes."

### Purposes

- To write asynchronous code that reads like synchronous code.
- To handle request success and failure with `try`/`catch`.
- To run independent requests in parallel with `Promise.all`.
- To sequence dependent requests with `await`.
- To propagate errors to callers, error boundaries, or route error handlers.

### Syntax Rules and Structure

**General Syntax:**

```js
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    // Handle or rethrow
    throw error;
  }
}
```

**Parallel Requests:**

```js
const [user, posts] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
]);
```

**Syntax Rules:**

1. An `async` function always returns a Promise.
2. `await` must be used inside an `async` function (or a module with top-level await).
3. Wrap `await` calls in `try`/`catch` when the caller needs to handle errors.
4. Use `Promise.all` for independent requests; use sequential `await` only when a request depends on a previous result.
5. Check `res.ok` before parsing; `fetch` does not reject on HTTP errors.

**Constraints and Limitations:**

- `await` inside a loop serialises requests; use `Promise.all` when independent.
- `async` functions cannot be passed directly to `useEffect`; define and call them inside the effect.
- Top-level `await` is available in ES modules but not in CommonJS.

### Annotated Code Examples

**Sequential vs. parallel:**

```js
// ❌ Waterfall: posts waits for user
async function loadProfile(id) {
  const user = await fetchUser(id);
  const posts = await fetchPosts(user.id); // Depends on user
  return { user, posts };
}

// ✅ Parallel: independent requests
async function loadDashboard(id) {
  const [stats, activity, notifications] = await Promise.all([
    fetchStats(id),
    fetchActivity(id),
    fetchNotifications(id),
  ]);
  return { stats, activity, notifications };
}
```

**Expected Output:** The sequential version takes the sum of both request times; the parallel version takes the maximum.

**Why this output:** `await` suspends until each Promise settles. `Promise.all` starts all requests immediately and waits for all to settle.

### Real-World Cases

- **Parallel dashboards:** Fetching independent widgets simultaneously.
- **Dependent requests:** Fetching a user, then their posts.
- **Error propagation:** Rethrowing in an async function so the caller can handle it.

### References

- MDN — async function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN — await — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- MDN — Promise.all — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all

---

## 3. Request State

### Definitions

**Core Definition:** Request state is the set of values that describe a request's current status and result, including whether it is pending, succeeded, or failed, and the associated data or error.

**Technical Definition:** Request state is a state machine with at least four states: **idle** (no request issued), **pending** (request in flight), **success** (data received), and **error** (request failed). Additional dimensions include **fetching** (a background refetch while data is already present) and **stale** (cached data that may be outdated). Modern libraries expose these as separate booleans (`isPending`, `isFetching`, `isError`, `isSuccess`) rather than a single status string, because a request can be simultaneously "successful with data" and "fetching in the background."

**Beginner-Friendly Explanation:** Request state is “what is happening with my request right now?” It could be waiting, done, or failed — and sometimes it is “done, but checking for updates in the background.”

### Purposes

- To render the correct UI for each request status.
- To distinguish initial loading from background refetching.
- To show stale data while refreshing it.
- To provide retry and error affordances.
- To avoid rendering empty UI during pending states.

### Syntax Rules and Structure

**State Dimensions:**

| Dimension | Meaning | Typical UI |
|-----------|---------|------------|
| `isPending` | No data yet, request in flight | Skeleton or spinner |
| `isFetching` | Request in flight (may have data) | Subtle indicator |
| `isSuccess` | Data available | Render data |
| `isError` | Request failed | Error message + retry |
| `isStale` | Cached data may be outdated | Dimmed content, refetch |

**Syntax Rules:**

1. Distinguish initial loading (`isPending`) from background fetching (`isFetching`).
2. Do not show a full-page spinner when stale data is already available; show a subtle indicator instead.
3. Derive UI from the smallest set of booleans that captures the state.
4. Reset error state when retrying.

### Annotated Code Examples

```tsx
import { useQuery } from '@tanstack/react-query';

function UserCard({ userId }) {
  const { data, isPending, isFetching, isError, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 60_000,
  });

  if (isPending) return <UserSkeleton />;
  if (isError) return <ErrorBanner message={error.message} />;

  return (
    <div style={{ opacity: isFetching ? 0.6 : 1 }}>
      <h2>{data.name}</h2>
      {isFetching && <span>Refreshing…</span>}
    </div>
  );
}
```

**Expected Output:** A skeleton on first load. The user's name renders with data. During a background refetch, the content stays visible but dims slightly and shows “Refreshing…”.

**Why this output:** `isPending` is true only when there is no data. `isFetching` is true during any request, including background refetches. The UI distinguishes the two so users never lose content they can already see.

### Real-World Cases

- **Dashboards:** Showing cached stats while refreshing.
- **Search:** Showing previous results while fetching new ones.
- **Profiles:** Rendering cached profile data with a refresh indicator.

### References

- TanStack Query — Query States — https://tanstack.com/query/latest/docs/framework/react/guides/query-states
- SWR — Conditional Fetching and States — https://swr.vercel.app/docs/conditional-fetching
- React — useTransition — https://react.dev/reference/react/useTransition

---

## 4. Loading States

### Definitions

**Core Definition:** A loading state is the UI shown while a request is pending and data is not yet available.

**Technical Definition:** Loading states communicate that work is in progress. They take several forms: **spinners** (indeterminate activity), **skeletons** (placeholders matching the final layout), **progress bars** (determinate progress), and **Suspense fallbacks** (declarative loading boundaries). The choice depends on expected duration and layout stability: skeletons reduce layout shift and perceived wait time; spinners are simpler but less informative. Loading states should appear immediately for fast connections and be delayed slightly (e.g., 200–500ms) for very fast requests to avoid flashing.

**Beginner-Friendly Explanation:** A loading state is what the user sees while they wait — a spinner, a grey placeholder box, or a progress bar. Good loading states make the wait feel shorter and prevent the page from jumping around when content arrives.

### Purposes

- To communicate that a request is in progress.
- To prevent blank screens and perceived freezes.
- To reduce layout shift with skeletons that match final content.
- To provide accessible status announcements (`aria-busy`, `role="status"`).
- To avoid flashing for very fast requests with delayed indicators.

### Syntax Rules and Structure

**Loading Patterns:**

| Pattern | Best for | Implementation |
|---------|----------|----------------|
| Spinner | Short, indeterminate waits | CSS animation, `role="status"` |
| Skeleton | Layout-stable content | Placeholder blocks matching dimensions |
| Progress bar | Determinate progress | `role="progressbar"`, `aria-valuenow` |
| Suspense fallback | Route or component boundaries | `<Suspense fallback={...}>` |
| Delayed indicator | Fast requests | Timer before showing spinner |

**Syntax Rules:**

1. Announce loading state with `role="status"` or `aria-busy="true"`.
2. Skeletons should match the dimensions of the final content to prevent layout shift.
3. Delay spinners by 200–500ms to avoid flashing on fast requests.
4. Do not block the entire page for a small section's loading state; scope the indicator.
5. Ensure the loading state is distinguishable from an empty state.

### Annotated Code Examples

**Skeleton loading state:**

```tsx
function UserCard({ userId }) {
  const { data, isPending } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  if (isPending) {
    return (
      <div role="status" aria-busy="true" aria-label="Loading profile">
        <div className="skeleton skeleton-avatar" />
        <div className="skeleton skeleton-name" />
        <div className="skeleton skeleton-bio" />
      </div>
    );
  }

  return <ProfileCard user={data} />;
}
```

**Expected Output:** A grey placeholder avatar, name, and bio appear while loading. When data arrives, the real profile replaces them without layout shift.

**Why this output:** The skeleton mirrors the final layout. `role="status"` and `aria-busy` announce the loading state to assistive technology. The `aria-label` describes what is loading.

### Real-World Cases

- **Social feeds:** Skeleton cards while posts load.
- **Dashboards:** Skeleton widgets for each panel.
- **Product pages:** Skeleton images and text while data loads.

### References

- React — Suspense — https://react.dev/reference/react/Suspense
- W3C WAI — aria-busy — https://www.w3.org/WAI/ARIA/apg/patterns/
- MDN — ARIA: status role — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/status_role
- TanStack Query — Suspense — https://tanstack.com/query/latest/docs/framework/react/guides/suspense

---

## 5. Error States

### Definitions

**Core Definition:** An error state is the UI shown when a request fails, explaining what went wrong and offering a way to recover.

**Technical Definition:** Error states occur when a request rejects: network failure, HTTP error status, timeout, parse error, or server exception. React handles these through error boundaries (for rendering and thrown errors), library-provided `error` values (TanStack Query, SWR), and router error elements (React Router `errorElement`, Next.js `error.tsx`). A good error state is specific (it names the failure), actionable (it offers retry or an alternative), and accessible (`role="alert"`). Distinguish transient errors (retryable) from permanent ones (not found, forbidden).

**Beginner-Friendly Explanation:** An error state is what the user sees when something goes wrong — “Could not load your data. Try again.” It should explain the problem and give the user a way forward, not just say “Error.”

### Purposes

- To inform the user that the request failed.
- To explain the failure in actionable terms.
- To offer a retry mechanism for transient errors.
- To distinguish transient from permanent failures.
- To announce errors to assistive technology with `role="alert"`.
- To log errors to monitoring services.

### Syntax Rules and Structure

**Error Handling Layers:**

| Layer | Mechanism | Catches |
|-------|-----------|---------|
| Fetch | `try`/`catch`, `res.ok` check | Network, HTTP, parse errors |
| Query library | `isError`, `error`, `refetch` | Query function rejections |
| Error boundary | `errorElement`, `error.tsx`, `ErrorBoundary` | Rendering and loader errors |
| Global | `window.onerror`, `unhandledrejection` | Uncaught errors |

**Syntax Rules:**

1. Check `res.ok` before parsing; `fetch` does not reject on HTTP errors.
2. Ignore `AbortError`; cancellation is not a user-facing error.
3. Distinguish 404 (not found) and 403 (forbidden) from 500 (server error).
4. Provide a retry action for transient errors.
5. Announce errors with `role="alert"` or an error boundary fallback.
6. Log errors to a monitoring service (Sentry, Datadog) in the error handler.

**Constraints and Limitations:**

- Error boundaries do not catch event-handler or async errors; use `try`/`catch` or library error states.
- Retrying a permanent error (404) wastes time; distinguish error types.

### Annotated Code Examples

```tsx
function UserCard({ userId }) {
  const { data, isError, error, refetch, isFetching } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    retry: (failureCount, error) => {
      // Do not retry 404s
      if (error.status === 404) return false;
      return failureCount < 2;
    },
  });

  if (isError) {
    if (error.status === 404) {
      return <p>User not found.</p>;
    }
    return (
      <div role="alert">
        <p>Could not load user.</p>
        <button onClick={() => refetch()} disabled={isFetching}>
          {isFetching ? 'Retrying…' : 'Try again'}
        </button>
      </div>
    );
  }

  return <h1>{data.name}</h1>;
}
```

**Expected Output:** A 404 renders “User not found.” without a retry button. Other errors render an alert with a retry button that disables while retrying.

**Why this output:** The `retry` function stops retries for 404s. The error UI distinguishes permanent from transient failures. `role="alert"` announces the error. The retry button reflects the fetching state.

### Real-World Cases

- **Network failures:** Offline or flaky connections with retry.
- **Not found:** Missing resources with a clear message.
- **Server errors:** 500s with retry and error reporting.
- **Rate limits:** 429s with a “try again later” message.

### References

- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- TanStack Query — Query Retries — https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- SWR — Error Handling — https://swr.vercel.app/docs/error-handling
- MDN — ARIA: alert role — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/alert_role

---

## 6. Empty States

### Definitions

**Core Definition:** An empty state is the UI shown when a request succeeds but returns no data, or when there is nothing to display yet.

**Technical Definition:** Empty states are distinct from loading and error states: the request succeeded, but the result is an empty array, `null`, or a zero count. They are also used for “no results” after filtering, “no items yet” for new accounts, and “nothing selected” in master-detail views. A good empty state explains *why* it is empty and offers a next action (create, clear filters, invite, etc.). Empty states are frequently conflated with loading states, leading to a confusing flash of “no results” before data arrives.

**Beginner-Friendly Explanation:** An empty state is what you show when there is nothing to show — “No messages yet,” “No results found,” or “Your cart is empty.” It should explain why and suggest what to do next.

### Purposes

- To distinguish “no data” from “still loading” and “something failed.”
- To explain why the list or view is empty.
- To guide the user to the next action (create, clear filters, invite).
- To avoid blank, confusing screens.
- To provide a better first-run experience for new users.

### Syntax Rules and Structure

**Empty State Variants:**

| Variant | When | Example |
|---------|------|---------|
| No results | Search/filter returned nothing | “No products match your filters.” |
| First-run | User has no data yet | “Create your first project.” |
| Nothing selected | Master-detail with no selection | “Select a conversation to start.” |
| Cleared | User completed/archived everything | “Inbox zero!” |

**Syntax Rules:**

1. Render the empty state only after the request resolves successfully with empty data.
2. Distinguish empty from loading (`isPending`) and error (`isError`).
3. Explain why the view is empty.
4. Offer a next action (button, link, or suggestion).
5. Use accessible semantics (`role="status"` for dynamic empty states).

**Constraints and Limitations:**

- Rendering an empty state during loading causes a confusing flash.
- Generic empty states (“No data”) waste an opportunity to guide the user.

### Annotated Code Examples

```tsx
function MessageList({ conversationId }) {
  const { data, isPending, isError } = useQuery({
    queryKey: ['messages', conversationId],
    queryFn: () => fetchMessages(conversationId),
  });

  if (isPending) return <MessageSkeleton />;
  if (isError) return <ErrorBanner />;

  if (data.length === 0) {
    return (
      <div role="status">
        <p>No messages yet.</p>
        <p>Start the conversation by sending the first message.</p>
      </div>
    );
  }

  return <Messages messages={data} />;
}
```

**Expected Output:** A skeleton while loading. An error banner on failure. An empty state with a prompt when the conversation has no messages. The message list when data exists.

**Why this output:** The three states are checked in order: pending, error, empty. The empty state explains the situation and suggests an action. `role="status"` announces it.

### Real-World Cases

- **Search results:** “No results for ‘xyz’” with a clear-filters action.
- **Inboxes:** “You're all caught up” when there are no unread messages.
- **Dashboards:** “No data for this period” with a link to change the range.
- **Onboarding:** “Create your first project” for new accounts.

### References

- Nielsen Norman Group — Empty States — https://www.nngroup.com/articles/empty-state-interface-design/
- TanStack Query — Query States — https://tanstack.com/query/latest/docs/framework/react/guides/query-states
- W3C WAI — status role — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/status_role

---

## 7. Success States

### Definitions

**Core Definition:** A success state is the UI rendered when a request completes successfully and returns data to display.

**Technical Definition:** The success state is the “happy path”: data is available, validated, and rendered. It includes rendering lists, details, charts, and forms populated with data. Success states must also handle edge cases within the data (missing optional fields, pagination boundaries, partial data). Libraries expose success as `isSuccess` and `data`. When combining success with background refetching, the UI should render data immediately and indicate freshness subtly rather than replacing content with a spinner.

**Beginner-Friendly Explanation:** The success state is what the user actually wanted to see — the data, rendered correctly.

### Purposes

- To render the requested data accurately.
- To handle optional and edge-case data gracefully.
- To combine data with background refresh indicators.
- To provide the foundation for interaction (pagination, sorting, editing).
- To ensure the UI is stable and accessible when data is present.

### Syntax Rules and Structure

**Success Handling:**

```tsx
if (isPending) return <Skeleton />;
if (isError) return <Error />;
if (!data || data.length === 0) return <Empty />;
return <Content data={data} />;
```

**Syntax Rules:**

1. Check pending, error, and empty before success.
2. Use optional chaining and defaults for optional fields (`data?.name ?? 'Unknown'`).
3. Use stable keys for lists (`item.id`, not index).
4. Render background refresh indicators alongside success content.
5. Validate data shape if the API is untrusted.

**Constraints and Limitations:**

- Assuming data is always present causes runtime errors.
- Rendering success content before checking empty/error states can flash incorrect UI.

### Annotated Code Examples

```tsx
function ProductList({ category }) {
  const { data, isPending, isError, isFetching } = useQuery({
    queryKey: ['products', category],
    queryFn: () => fetchProducts(category),
  });

  if (isPending) return <ProductGridSkeleton />;
  if (isError) return <ErrorBanner />;
  if (data.length === 0) return <EmptyState category={category} />;

  return (
    <>
      {isFetching && <RefreshIndicator />}
      <ul>
        {data.map((product) => (
          <li key={product.id}>
            <ProductCard product={product} />
          </li>
        ))}
      </ul>
    </>
  );
}
```

**Expected Output:** Skeleton while loading. Error on failure. Empty state for no products. The product grid on success, with a refresh indicator during background refetches.

**Why this output:** States are handled in order. `key={product.id}` ensures stable reconciliation. `isFetching` shows a subtle indicator without hiding the existing data.

### Real-World Cases

- **E-commerce:** Product grids with stable keys and refresh indicators.
- **Dashboards:** Charts and stats rendered from fetched data.
- **Social feeds:** Posts rendered with stable IDs and pagination.

### References

- TanStack Query — Query States — https://tanstack.com/query/latest/docs/framework/react/guides/query-states
- React — Rendering Lists — https://react.dev/learn/rendering-lists
- React — Conditional Rendering — https://react.dev/learn/conditional-rendering

---

## 8. Client-Side Fetching

### Definitions

**Core Definition:** Client-side fetching is the practice of requesting data from the browser after the component mounts or in response to user interaction.

**Technical Definition:** Client-side fetching runs in the browser, typically inside `useEffect` or through a query library. The initial HTML contains no data; the browser fetches it after hydration. This creates a waterfall (render → fetch → render) and delays content until JavaScript executes and the network responds. Client-side fetching is appropriate for user-specific, interactive, or post-load data, but it is a poor choice for SEO-critical or above-the-fold content. Modern approaches prefer TanStack Query or SWR over raw `useEffect` + `fetch` because they provide caching, deduplication, cancellation, and retry.

**Beginner-Friendly Explanation:** Client-side fetching means the browser asks for the data after the page has loaded. It is simple and works everywhere, but the user waits longer because the request only starts after the JavaScript runs.

### Purposes

- To fetch user-specific or interactive data after login.
- To fetch data that changes frequently and should not be cached at build time.
- To avoid server infrastructure for simple applications.
- To support client-only environments (SPAs, Electron, React Native).
- To fetch data in response to user actions (search, filter, pagination).

### Syntax Rules and Structure

**Raw `useEffect` (basic but limited):**

```tsx
useEffect(() => {
  const controller = new AbortController();
  fetch(url, { signal: controller.signal })
    .then((res) => res.json())
    .then(setData)
    .catch((err) => { if (err.name !== 'AbortError') setError(err); });
  return () => controller.abort();
}, [url]);
```

**TanStack Query (recommended):**

```tsx
const { data, isPending, isError } = useQuery({
  queryKey: ['resource', id],
  queryFn: ({ signal }) => fetch(`/api/${id}`, { signal }).then((r) => r.json()),
});
```

**Syntax Rules:**

1. Always cancel or ignore stale requests; use `AbortController` or the library's signal.
2. Always handle pending and error states.
3. Use a cache key (query key) for deduplication and caching.
4. Do not fetch in render; fetch in effects, event handlers, or query functions.
5. In React Strict Mode, effects run twice in development; ensure fetch logic is idempotent or deduplicated.

**Constraints and Limitations:**

- Waterfall: content appears only after JS loads and the request completes.
- No data in initial HTML → poor SEO and slow LCP for content pages.
- Raw `useEffect` fetching lacks caching, deduplication, and retry.
- Strict Mode double-invokes effects, causing duplicate requests unless deduplicated.

### Annotated Code Examples

```tsx
import { useQuery } from '@tanstack/react-query';

function NotificationsPanel({ userId }) {
  const { data, isPending, isError, refetch } = useQuery({
    queryKey: ['notifications', userId],
    queryFn: ({ signal }) =>
      fetch(`/api/users/${userId}/notifications`, { signal }).then((r) => r.json()),
    refetchOnWindowFocus: true,
    staleTime: 30_000,
  });

  if (isPending) return <NotificationsSkeleton />;
  if (isError) return <button onClick={() => refetch()}>Retry</button>;

  return (
    <ul>
      {data.map((n) => (
        <li key={n.id}>{n.message}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** Notifications load after mount. The list refetches when the window regains focus if data is older than 30 seconds. Errors show a retry button.

**Why this output:** `useQuery` handles caching, deduplication, cancellation, and refetching. `refetchOnWindowFocus` keeps notifications fresh. `staleTime` prevents unnecessary refetches within 30 seconds.

### Real-World Cases

- **Notifications:** Fetching user-specific notifications after login.
- **Search:** Fetching results as the user types.
- **Dashboards:** Loading widgets after the shell renders.
- **SPAs:** Applications without a server-rendering layer.

### References

- React — You Might Not Need an Effect: Fetching data — https://react.dev/learn/you-might-not-need-an-effect
- TanStack Query — Quick Start — https://tanstack.com/query/latest/docs/framework/react/quick-start
- SWR — Getting Started — https://swr.vercel.app/docs/getting-started
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## 9. Server-Side Fetching

### Definitions

**Core Definition:** Server-side fetching is requesting data on the server — at build time, per request, or during server rendering — and sending the resulting HTML or serialized data to the browser.

**Technical Definition:** Server-side fetching moves the request from the browser to the server. In the Next.js App Router, async Server Components fetch data directly with `await fetch(...)` or database queries, and the result is rendered into HTML on the server. In React Router Data Mode, `loader` functions fetch data before rendering. In TanStack Start, loaders run on the server. Server-side fetching eliminates client waterfalls, keeps secrets on the server, and improves first paint, but it adds server load and requires handling per-request caching and authentication.

**Beginner-Friendly Explanation:** Server-side fetching means the server gets the data before sending the page. The browser receives HTML that already contains the content, so the user sees it faster.

### Purposes

- To render data in the initial HTML for faster first paint and SEO.
- To keep API keys and database credentials on the server.
- To avoid client-side waterfalls for critical content.
- To centralise data access and caching.
- To support Server Components that ship zero JavaScript.

### Syntax Rules and Structure

**Next.js App Router (Server Component):**

```tsx
export default async function Page() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }, // Cache for 60 seconds (Next.js 15+ opt-in)
  });
  const posts = await res.json();
  return <PostList posts={posts} />;
}
```

**React Router Data Mode (loader):**

```jsx
export async function loader() {
  const posts = await fetchPosts();
  return { posts };
}
```

**Syntax Rules:**

1. Server Components can be `async` and `await` data directly.
2. In Next.js 15+, `fetch` is **not cached by default**; opt in with `cache: 'force-cache'` or `next: { revalidate: n }`.
3. React Router loaders run before rendering and provide data via `useLoaderData()`.
4. Keep secrets and database clients in server-only modules.
5. Handle errors with error boundaries (`error.tsx`, `errorElement`).

**Constraints and Limitations:**

- Server fetching adds latency to every request unless cached or statically generated.
- Server Components cannot use client-side Hooks or browser APIs.
- Per-request fetching requires careful caching to avoid overloading the backend.

### Annotated Code Examples

```tsx
// app/posts/page.tsx (Next.js App Router, Next.js 15+)
export default async function PostsPage() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }, // ISR-style caching for 60 seconds
  });
  if (!res.ok) throw new Error('Failed to fetch posts');
  const posts = await res.json();

  return (
    <main>
      <h1>Posts</h1>
      <PostList posts={posts} />
    </main>
  );
}
```

**Expected Output:** The server fetches posts, renders the list into HTML, and sends it to the browser. The response is cached for 60 seconds.

**Why this output:** The async Server Component awaits the fetch on the server. `next: { revalidate: 60 }` opts into caching with revalidation. The rendered HTML contains the posts, so the browser does not need a client-side fetch.

### Real-World Cases

- **Content sites:** Rendering articles and product pages with SEO.
- **Dashboards:** Fetching user-specific data on the server.
- **APIs:** Keeping API keys and database credentials server-side.

### References

- Next.js — Fetching Data (App Router) — https://nextjs.org/docs/app/building-your-application/data-fetching
- Next.js — Server Components — https://nextjs.org/docs/app/building-your-application/rendering/server-components
- React Router — Data Loading — https://reactrouter.com/start/data/route-object
- React — Server Components — https://react.dev/reference/rsc/server-components

---

## 10. Static Generation

### Definitions

**Core Definition:** Static generation is the practice of fetching data and rendering pages at build time, producing HTML files that are served identically to every visitor.

**Technical Definition:** Static generation (SSG) runs data fetching and rendering during the build, not per request. The output is static HTML, JSON, and assets that can be served from a CDN. In Next.js App Router, pages without dynamic APIs (`cookies()`, `headers()`, `searchParams`) are statically rendered by default, and `generateStaticParams` pre-renders dynamic routes. In Pages Router, `getStaticProps` and `getStaticPaths` perform static generation. SSG is ideal for content that does not change per user and can tolerate build-time staleness.

**Beginner-Friendly Explanation:** Static generation means the page is built once, ahead of time, and every visitor gets the same fast file. It is perfect for blogs, marketing pages, and documentation.

### Purposes

- To serve pages from a CDN with minimal latency.
- To eliminate per-request server work for stable content.
- To improve first paint and SEO.
- To reduce infrastructure cost.
- To pre-render dynamic routes with `generateStaticParams`.

### Syntax Rules and Structure

**Next.js App Router (`generateStaticParams`):**

```tsx
export async function generateStaticParams() {
  const posts = await fetchPosts();
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function PostPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetchPost(slug);
  return <Article post={post} />;
}
```

**Pages Router (legacy):**

```tsx
export async function getStaticProps() {
  const posts = await fetchPosts();
  return { props: { posts }, revalidate: 60 };
}
```

**Syntax Rules:**

1. In App Router, routes are static by default unless they use dynamic APIs.
2. Use `generateStaticParams` to pre-render dynamic segments.
3. Do not use `cookies()`, `headers()`, or `searchParams` in static routes.
4. In Pages Router, `getStaticProps` and `getStaticPaths` are legacy but still supported.
5. Combine with ISR (`revalidate`) for periodic updates.

**Constraints and Limitations:**

- Data is frozen at build time unless ISR is used.
- Build times grow with the number of pages.
- Personalised content cannot be statically generated.

### Annotated Code Examples

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetchAllPosts();
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetchPost(slug);
  if (!post) notFound();
  return <article><h1>{post.title}</h1><p>{post.body}</p></article>;
}
```

**Expected Output:** At build time, Next.js generates an HTML file for each post slug. Requests are served from the CDN without a server round trip.

**Why this output:** `generateStaticParams` returns the slugs to pre-render. The page component is static because it uses no dynamic APIs. `notFound()` handles missing slugs.

### Real-World Cases

- **Blogs:** Pre-rendering articles at build time.
- **Marketing sites:** Static landing pages on a CDN.
- **Documentation:** Pre-rendered docs with fast navigation.
- **E-commerce:** Pre-rendering popular product pages.

### References

- Next.js — Static Rendering — https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default
- Next.js — generateStaticParams — https://nextjs.org/docs/app/api-reference/functions/generate-static-params
- Next.js — getStaticProps (Pages Router) — https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props

---

## 11. Incremental Static Regeneration (ISR)

### Definitions

**Core Definition:** Incremental Static Regeneration (ISR) is a hybrid model that serves statically generated pages while revalidating them in the background after a configured interval, without rebuilding the entire site.

**Technical Definition:** ISR lets statically generated pages be updated after deployment. In Next.js, `revalidate` controls the interval: `export const revalidate = 60` on a route segment, `next: { revalidate: 60 }` on a fetch, or `revalidatePath()`/`revalidateTag()` for on-demand revalidation. When a request arrives after the revalidation window, the stale page is served immediately and a background regeneration is triggered; the next request receives the fresh page. ISR combines the speed of static generation with the freshness of server rendering.

**Beginner-Friendly Explanation:** ISR means your static pages update themselves in the background after a set time. Visitors always get a fast page, and the page refreshes periodically without a full rebuild.

### Purposes

- To serve static pages with periodic freshness.
- To avoid full-site rebuilds for content updates.
- To handle large sites with thousands of pages.
- To combine CDN speed with data freshness.
- To support on-demand revalidation for editorial workflows.

### Syntax Rules and Structure

**Route segment config:**

```tsx
export const revalidate = 60; // Revalidate every 60 seconds
```

**Fetch-level:**

```tsx
const res = await fetch(url, { next: { revalidate: 60 } });
```

**On-demand revalidation:**

```tsx
import { revalidatePath, revalidateTag } from 'next/cache';

revalidatePath('/blog');
revalidateTag('posts');
```

**Syntax Rules:**

1. Set `revalidate` on the route segment or on individual fetches.
2. Use `revalidatePath` or `revalidateTag` after mutations for immediate updates.
3. ISR requires a server runtime that supports it (Next.js, or a platform with ISR support).
4. Combine with `generateStaticParams` for dynamic routes.
5. Stale content is served during background regeneration.

**Constraints and Limitations:**

- The first request after the window may still receive stale content.
- Not all hosting platforms support ISR.
- Very short revalidation intervals reduce the benefit of static generation.

### Annotated Code Examples

```tsx
// app/products/[slug]/page.tsx
export const revalidate = 300; // Revalidate every 5 minutes

export async function generateStaticParams() {
  const products = await fetchAllProducts();
  return products.map((p) => ({ slug: p.slug }));
}

export default async function ProductPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const product = await fetchProduct(slug);
  return <ProductDetail product={product} />;
}
```

**Expected Output:** Product pages are generated at build time and revalidated every 5 minutes. Visitors receive the cached page immediately; the background regenerates it after the window.

**Why this output:** `revalidate = 300` sets the ISR interval. `generateStaticParams` pre-renders known products. The page serves stale content while regenerating in the background.

### Real-World Cases

- **E-commerce:** Product pages updated every few minutes.
- **News sites:** Articles revalidated frequently.
- **Documentation:** Content updated on a schedule.
- **CMS-driven sites:** On-demand revalidation after publishing.

### References

- Next.js — Incremental Static Regeneration — https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration
- Next.js — revalidatePath — https://nextjs.org/docs/app/api-reference/functions/revalidatePath
- Next.js — revalidateTag — https://nextjs.org/docs/app/api-reference/functions/revalidateTag

---

## 12. Server-Side Rendering (SSR)

### Definitions

**Core Definition:** Server-side rendering (SSR) is the practice of rendering React components to HTML on the server for each request, then hydrating the HTML on the client.

**Technical Definition:** SSR runs the component tree on the server per request, producing HTML that the browser displays immediately. The client then hydrates the HTML, attaching event handlers and making it interactive. In Next.js App Router, routes become dynamic when they use `cookies()`, `headers()`, or `searchParams`, and are rendered per request. In Pages Router, `getServerSideProps` fetches data per request. SSR is appropriate for personalised, frequently changing, or authenticated content that cannot be statically generated.

**Beginner-Friendly Explanation:** SSR means the server builds the HTML for each request. The user sees content quickly, and the page becomes interactive after the JavaScript loads.

### Purposes

- To render personalised or per-request content.
- To provide fast first paint for dynamic pages.
- To support SEO for content that changes per request.
- To access request-specific data (cookies, headers, user session).
- To keep sensitive data fetching on the server.

### Syntax Rules and Structure

**Next.js App Router (dynamic rendering):**

```tsx
import { cookies } from 'next/headers';

export default async function Dashboard() {
  const session = (await cookies()).get('session');
  const data = await fetchDashboard(session?.value);
  return <DashboardView data={data} />;
}
```

**Pages Router (legacy):**

```tsx
export async function getServerSideProps({ req }) {
  const data = await fetchData(req.cookies.session);
  return { props: { data } };
}
```

**Syntax Rules:**

1. In App Router, using `cookies()`, `headers()`, or `searchParams` opts the route into dynamic rendering.
2. In Pages Router, `getServerSideProps` runs on every request (legacy).
3. Hydration requires matching server and client output; avoid non-deterministic rendering.
4. Use Suspense and streaming to avoid blocking the whole page on slow data.
5. Cache aggressively where possible; SSR is expensive per request.

**Constraints and Limitations:**

- Higher server cost and latency than static generation.
- Hydration mismatches if server and client render different output.
- Requires a server runtime; not deployable as a pure static site.

### Annotated Code Examples

```tsx
// app/account/page.tsx
import { cookies } from 'next/headers';
import { redirect } from 'next/navigation';

export default async function AccountPage() {
  const session = (await cookies()).get('session');
  if (!session) redirect('/login');

  const account = await fetchAccount(session.value);
  return <AccountDetails account={account} />;
}
```

**Expected Output:** Each request renders the account page with the current session's data. Unauthenticated users are redirected to login.

**Why this output:** `cookies()` makes the route dynamic. The server fetches account data per request and renders HTML. `redirect()` sends unauthenticated users away before rendering.

### Real-World Cases

- **Account pages:** Personalised dashboards.
- **E-commerce cart:** Per-session cart contents.
- **Admin panels:** Authenticated, per-request data.
- **Search results:** Query-dependent pages.

### References

- Next.js — Dynamic Rendering — https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering
- Next.js — cookies — https://nextjs.org/docs/app/api-reference/functions/cookies
- Next.js — getServerSideProps (Pages Router) — https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot

---

## 13. Streaming

### Definitions

**Core Definition:** Streaming is the technique of sending server-rendered HTML to the browser in chunks as they become ready, rather than waiting for the entire page to render.

**Technical Definition:** Streaming SSR uses `renderToPipeableStream` (Node) or `renderToReadableStream` (Web Streams) to send the document shell first, then stream in Suspense boundary content as it resolves. In Next.js App Router, `loading.tsx` and `<Suspense>` boundaries define streamable chunks. The browser parses and renders each chunk as it arrives, and React hydrates progressively. Streaming improves Time to First Byte (TTFB) and allows users to see and interact with fast content while slow content loads.

**Beginner-Friendly Explanation:** Streaming means the server sends the page in pieces. The header and layout arrive immediately, and slower sections (like comments or recommendations) arrive later without blocking the rest of the page.

### Purposes

- To improve TTFB by sending the shell immediately.
- To let users see fast content while slow content loads.
- To avoid blocking the entire page on a slow data dependency.
- To enable selective hydration of interactive components.
- To improve perceived performance on slow networks.

### Syntax Rules and Structure

**Next.js App Router (`loading.tsx` + Suspense):**

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}

// app/dashboard/page.tsx
export default function Dashboard() {
  return (
    <>
      <Header />
      <Suspense fallback={<StatsSkeleton />}>
        <Stats /> {/* Streams when ready */}
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart /> {/* Streams when ready */}
      </Suspense>
    </>
  );
}
```

**Node streaming API:**

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

**Syntax Rules:**

1. Wrap slow sections in `<Suspense>` boundaries.
2. Use `loading.tsx` for route-level streaming in Next.js.
3. Start piping in `onShellReady` so the shell reaches the browser immediately.
4. Use `hydrateRoot` on the client for progressive hydration.
5. Handle errors during streaming with `onError`.

**Constraints and Limitations:**

- Requires a server runtime that supports streaming.
- Some bots and older clients may not handle streamed HTML well.
- Errors after the shell has been sent cannot change the HTTP status code.

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

**Why this output:** Each `<Suspense>` boundary is a streamable chunk. React sends the shell first, then streams each boundary's HTML when ready.

### Real-World Cases

- **E-commerce:** Streaming recommendations after product info.
- **Content sites:** Streaming comments after the article.
- **Dashboards:** Streaming widgets as their data loads.

### References

- React — renderToPipeableStream — https://react.dev/reference/react-dom/server/renderToPipeableStream
- Next.js — Loading UI and Streaming — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- React — Suspense — https://react.dev/reference/react/Suspense

---

## 14. Data Caching

### Definitions

**Core Definition:** Data caching is the practice of storing fetched data so subsequent requests can reuse it instead of re-fetching.

**Technical Definition:** Data caching operates at multiple layers. The **HTTP cache** follows server headers (`Cache-Control`, `ETag`). The **framework cache** (Next.js `fetch` cache, `React.cache`, `unstable_cache`) stores fetch results on the server. The **application cache** (TanStack Query, SWR) stores results in memory on the client, keyed by query keys, with configurable freshness (`staleTime`) and garbage collection (`gcTime`). Caching reduces network requests, improves performance, and enables stale-while-revalidate UX, but requires invalidation to avoid stale data.

**Beginner-Friendly Explanation:** Caching means remembering data you already fetched. If the user goes back to a page, the app shows the cached data instantly instead of asking the server again.

### Purposes

- To eliminate redundant network requests.
- To make repeat visits instant.
- To reduce server load and bandwidth.
- To enable stale-while-revalidate UX.
- To share data across components without prop drilling.

### Syntax Rules and Structure

**TanStack Query (client cache):**

```tsx
useQuery({
  queryKey: ['user', id],
  queryFn: fetchUser,
  staleTime: 5 * 60 * 1000, // Fresh for 5 minutes
  gcTime: 30 * 60 * 1000,   // Keep unused for 30 minutes
});
```

**Next.js server cache:**

```tsx
const res = await fetch(url, {
  next: { revalidate: 60 }, // Cache for 60 seconds
});
```

**React cache (per-request deduplication):**

```tsx
import { cache } from 'react';

const getUser = cache(async (id) => fetchUser(id));
```

**Syntax Rules:**

1. Choose a stable cache key (query key, URL + params).
2. Set `staleTime` to control freshness; `gcTime` to control memory retention.
3. Use `React.cache` for per-request deduplication in Server Components.
4. In Next.js 15+, `fetch` is **not cached by default**; opt in explicitly.
5. Cache only data that is safe to share; never cache user-specific data in a shared cache.

**Constraints and Limitations:**

- Stale caches show outdated data; invalidation is mandatory after mutations.
- Over-caching sensitive data is a security risk.
- Cache keys must include all parameters that affect the result.

### Annotated Code Examples

```tsx
function UserProfile({ userId }) {
  const { data } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,
  });

  return <Profile user={data} />;
}
```

**Expected Output:** The profile is fetched once and cached for 5 minutes. Revisiting the same user within 5 minutes shows cached data without a request.

**Why this output:** `queryKey` identifies the cache entry. `staleTime` marks the data fresh for 5 minutes, so no refetch occurs within that window.

### Real-World Cases

- **User profiles:** Caching per-user data across navigations.
- **Product catalogs:** Caching category and product lists.
- **Dashboards:** Caching stats with short staleness.
- **Server Components:** Deduplicating database queries per request.

### References

- TanStack Query — Caching — https://tanstack.com/query/latest/docs/framework/react/guides/caching
- Next.js — Caching — https://nextjs.org/docs/app/building-your-application/caching
- React — cache — https://react.dev/reference/react/cache
- SWR — Caching — https://swr.vercel.app/docs/advanced/cache

---

## 15. Data Invalidation

### Definitions

**Core Definition:** Data invalidation is the process of marking cached data as stale or removing it so the next read fetches fresh data.

**Technical Definition:** Invalidation ensures that caches do not serve outdated data after mutations. TanStack Query invalidates with `queryClient.invalidateQueries({ queryKey })`, which marks matching queries as stale and refetches active ones. Next.js invalidates server caches with `revalidatePath(path)` and `revalidateTag(tag)`. React Router invalidates loader data with `router.invalidate()`. SWR uses `mutate(key)` to revalidate. Invalidation can be **targeted** (specific keys/tags) or **broad** (all queries), and should be triggered after successful mutations, not before.

**Beginner-Friendly Explanation:** Invalidation means telling the cache “this data might be old now, go check.” You do it after the user changes something, so the screen updates with the new data.

### Purposes

- To keep cached data in sync with server state after mutations.
- To avoid stale UI after create, update, or delete operations.
- To target invalidation to the affected data.
- To support on-demand revalidation for editorial workflows.
- To prevent users from acting on outdated information.

### Syntax Rules and Structure

**TanStack Query:**

```tsx
const queryClient = useQueryClient();

useMutation({
  mutationFn: updateUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['user', userId] });
  },
});
```

**Next.js:**

```tsx
import { revalidatePath, revalidateTag } from 'next/cache';

revalidatePath('/blog');
revalidateTag('posts');
```

**React Router:**

```jsx
await router.invalidate({ sync: true });
```

**Syntax Rules:**

1. Invalidate **after** the mutation succeeds, not before.
2. Target the smallest set of keys/tags affected by the mutation.
3. Use tags for cross-route server caches; use query keys for client caches.
4. Combine targeted and broad invalidation carefully; broad invalidation causes refetch storms.
5. Verify invalidation by checking that the UI reflects the new data.

**Constraints and Limitations:**

- Invalidating everything causes unnecessary refetches and poor performance.
- Invalidating before the mutation completes can re-fetch stale data.
- Server and client caches are separate; invalidate both when both are used.

### Annotated Code Examples

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function EditUserForm({ userId }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (data) => updateUser(userId, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['user', userId] });
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  return <form onSubmit={(e) => { e.preventDefault(); mutation.mutate(readForm(e)); }} />;
}
```

**Expected Output:** After saving, the individual user query and the user list query are invalidated and refetched, so both views show the updated data.

**Why this output:** The mutation persists the change; `onSuccess` invalidates the affected query keys, causing active queries to refetch.

### Real-World Cases

- **CRUD apps:** Invalidating lists and detail views after edits.
- **CMS:** Revalidating published content by tag.
- **Dashboards:** Refreshing stats after mutations.
- **E-commerce:** Invalidating cart and product caches after adding items.

### References

- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- Next.js — revalidatePath — https://nextjs.org/docs/app/api-reference/functions/revalidatePath
- Next.js — revalidateTag — https://nextjs.org/docs/app/api-reference/functions/revalidateTag
- React Router — router.invalidate — https://reactrouter.com/api/data-routers/Router
- SWR — Mutation — https://swr.vercel.app/docs/mutation

---

## 16. Data Refetching

### Definitions

**Core Definition:** Data refetching is the practice of re-requesting data that is already cached, either on a schedule, on focus, on reconnect, or on demand.

**Technical Definition:** Refetching updates cached data without discarding it. Libraries support **refetch on window focus**, **refetch on reconnect**, **refetch on mount**, **polling** (`refetchInterval`), and **manual refetch** (`refetch()`). Refetching differs from invalidation: invalidation marks data stale and triggers refetch for active queries, while refetching can occur on a schedule or event regardless of staleness. Refetching should be tuned to the data's volatility: real-time data may poll, while stable data may refetch only on focus.

**Beginner-Friendly Explanation:** Refetching means asking for the data again to see if anything changed. It can happen when you switch back to the tab, when the internet reconnects, or on a timer.

### Purposes

- To keep data fresh during long sessions.
- To update data when the user returns to the tab.
- To recover from network interruptions.
- To poll for real-time updates.
- To provide manual refresh controls.

### Syntax Rules and Structure

**TanStack Query:**

```tsx
useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  refetchOnWindowFocus: true,
  refetchOnReconnect: true,
  refetchInterval: 30_000, // Poll every 30 seconds
});
```

**Manual refetch:**

```tsx
const { refetch } = useQuery({ ... });
<button onClick={() => refetch()}>Refresh</button>
```

**SWR:**

```tsx
useSWR('/api/notifications', fetcher, {
  refreshInterval: 30_000,
  revalidateOnFocus: true,
});
```

**Syntax Rules:**

1. Tune refetch frequency to data volatility.
2. Use `refetchOnWindowFocus` for data that changes while the user is away.
3. Use `refetchInterval` sparingly; polling is expensive.
4. Provide manual refetch for user control.
5. Cancel refetches on unmount to avoid memory leaks.

**Constraints and Limitations:**

- Aggressive polling increases server load and battery usage.
- Refetching during active user interaction can cause UI jumps.
- Refetching does not replace invalidation; both are needed for correctness after mutations.

### Annotated Code Examples

```tsx
function NotificationBell() {
  const { data, refetch, isFetching } = useQuery({
    queryKey: ['notifications'],
    queryFn: fetchNotifications,
    refetchInterval: 30_000,
    refetchOnWindowFocus: true,
  });

  return (
    <button onClick={() => refetch()} disabled={isFetching}>
      {data?.length ?? 0} notifications
    </button>
  );
}
```

**Expected Output:** The notification count refreshes every 30 seconds and when the window regains focus. Clicking the bell triggers a manual refetch.

**Why this output:** `refetchInterval` polls every 30 seconds. `refetchOnWindowFocus` refreshes when the user returns. `refetch` provides manual control.

### Real-World Cases

- **Notifications:** Polling or focus-based refetch.
- **Chat:** Real-time message refresh.
- **Dashboards:** Periodic stats refresh.
- **Stock prices:** High-frequency polling.

### References

- TanStack Query — Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query — Polling — https://tanstack.com/query/latest/docs/framework/react/guides/polling
- SWR — Revalidation — https://swr.vercel.app/docs/revalidation

---

## 17. Data Fetching Best Practices

### Definitions

**Core Definition:** Data fetching best practices are the guidelines for requesting, caching, rendering, and invalidating data in a way that is correct, performant, and maintainable.

**Technical Definition:** Best practices combine **architecture** (server-first for SEO and first paint, client-side for interactive data), **library choice** (TanStack Query or SWR for client state, framework primitives for server data), **state handling** (pending, error, empty, success, fetching), **caching strategy** (staleTime, gcTime, revalidation), **performance** (avoiding waterfalls, parallelising requests, streaming), **correctness** (cancellation, deduplication, invalidation after mutations), and **security** (server-side authorisation, no secrets in client code).

**Beginner-Friendly Explanation:** Good data fetching means the right data loads at the right time, the user always sees a sensible state (loading, error, empty, or data), nothing is fetched twice unnecessarily, and changes show up when they should.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Prefer server fetching for critical content** | Server Components, loaders, or SSR for SEO and first paint. |
| **Use a query library for client data** | TanStack Query or SWR over raw `useEffect` + `fetch`. |
| **Handle all five states** | Pending, error, empty, success, background fetching. |
| **Cancel stale requests** | `AbortController` or the library's signal. |
| **Deduplicate requests** | Stable query keys, `React.cache` for per-request dedupe. |
| **Avoid waterfalls** | `Promise.all` for independent requests; prefetch or parallelise. |
| **Cache deliberately** | Set `staleTime` and `gcTime` based on data volatility. |
| **Invalidate after mutations** | `invalidateQueries`, `revalidatePath`, `revalidateTag`. |
| **Distinguish initial loading from refetching** | `isPending` vs `isFetching`. |
| **Provide retry for transient errors** | Do not retry permanent errors (404, 403). |
| **Never fetch in render** | Fetch in effects, event handlers, loaders, or Server Components. |
| **Keep secrets on the server** | Never expose API keys in client code. |
| **Enforce authorisation on the server** | Client checks are UX; server checks are security. |
| **Use Suspense and streaming for slow sections** | Stream in slow content instead of blocking the page. |
| **Respect Strict Mode** | Effects run twice in development; ensure fetches are idempotent or deduplicated. |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
import { useQuery } from '@tanstack/react-query';

function Dashboard({ userId }) {
  const userQuery = useQuery({
    queryKey: ['user', userId],
    queryFn: ({ signal }) => fetchUser(userId, { signal }),
    staleTime: 60_000,
  });

  const statsQuery = useQuery({
    queryKey: ['stats', userId],
    queryFn: ({ signal }) => fetchStats(userId, { signal }),
    staleTime: 30_000,
  });

  if (userQuery.isPending || statsQuery.isPending) return <DashboardSkeleton />;
  if (userQuery.isError || statsQuery.isError) return <DashboardError />;
  if (!userQuery.data || !statsQuery.data) return <DashboardEmpty />;

  return (
    <>
      <UserHeader user={userQuery.data} />
      <StatsGrid stats={statsQuery.data} />
    </>
  );
}
```

**❌ Bad Practices:**

```tsx
// ❌ Waterfall: sequential requests
const user = await fetchUser(id);
const stats = await fetchStats(user.id);

// ❌ No error or empty state
const { data } = useQuery({ queryKey: ['user'], queryFn: fetchUser });
return <h1>{data.name}</h1>; // Crashes when data is undefined

// ❌ Fetching in render
function Bad() {
  fetch('/api/data'); // Runs on every render
  return <div />;
}

// ❌ No invalidation after mutation
await updateUser(id, data); // Cache still shows old data
```

**Expected Output:** The good practices produce parallel fetching, explicit states, and correct caching. The bad practices cause waterfalls, crashes on missing data, infinite fetch loops, and stale UI.

**Why this output:** Independent queries run in parallel. State checks prevent rendering undefined data. Fetching in render causes loops. Missing invalidation leaves stale caches.

### Real-World Cases

- **All data-driven React applications:** These practices apply universally.
- **Dashboards:** Parallel queries with per-widget states.
- **E-commerce:** Server-rendered product pages with client-side cart updates.
- **Content sites:** Static generation with ISR and client-side comments.

### References

- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- Next.js — Data Fetching Patterns — https://nextjs.org/docs/app/building-your-application/data-fetching/fetching
- web.dev — Avoid large network payloads — https://web.dev/articles/reduce-network-payloads-using-text-compression
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController

---

## References

- React — Fetching Data (Legacy FAQ) — https://legacy.reactjs.org/docs/faq-ajax.html
- React — You Might Not Need an Effect — https://react.dev/learn/you-might-not-need-an-effect
- React — Suspense — https://react.dev/reference/react/Suspense
- React — cache — https://react.dev/reference/react/cache
- React — Server Components — https://react.dev/reference/rsc/server-components
- React — renderToPipeableStream — https://react.dev/reference/react-dom/server/renderToPipeableStream
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot
- React — Error Boundaries — https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- TanStack Query — Overview — https://tanstack.com/query/latest/docs/framework/react/overview
- TanStack Query — Query States — https://tanstack.com/query/latest/docs/framework/react/guides/query-states
- TanStack Query — Caching — https://tanstack.com/query/latest/docs/framework/react/guides/caching
- TanStack Query — Query Invalidation — https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation
- TanStack Query — Query Retries — https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- TanStack Query — Window Focus Refetching — https://tanstack.com/query/latest/docs/framework/react/guides/window-focus-refetching
- TanStack Query — Polling — https://tanstack.com/query/latest/docs/framework/react/guides/polling
- TanStack Query — Suspense — https://tanstack.com/query/latest/docs/framework/react/guides/suspense
- TanStack Query — Important Defaults — https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- SWR — Getting Started — https://swr.vercel.app/docs/getting-started
- SWR — Caching — https://swr.vercel.app/docs/advanced/cache
- SWR — Error Handling — https://swr.vercel.app/docs/error-handling
- SWR — Revalidation — https://swr.vercel.app/docs/revalidation
- SWR — Mutation — https://swr.vercel.app/docs/mutation
- Next.js — Fetching Data (App Router) — https://nextjs.org/docs/app/building-your-application/data-fetching
- Next.js — Caching — https://nextjs.org/docs/app/building-your-application/caching
- Next.js — Static Rendering — https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default
- Next.js — Dynamic Rendering — https://nextjs.org/docs/app/building-your-application/rendering/server-components#dynamic-rendering
- Next.js — generateStaticParams — https://nextjs.org/docs/app/api-reference/functions/generate-static-params
- Next.js — Incremental Static Regeneration — https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration
- Next.js — revalidatePath — https://nextjs.org/docs/app/api-reference/functions/revalidatePath
- Next.js — revalidateTag — https://nextjs.org/docs/app/api-reference/functions/revalidateTag
- Next.js — cookies — https://nextjs.org/docs/app/api-reference/functions/cookies
- Next.js — Loading UI and Streaming — https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming
- Next.js — getStaticProps (Pages Router) — https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-props
- Next.js — getServerSideProps (Pages Router) — https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props
- React Router — Data Loading — https://reactrouter.com/start/data/route-object
- React Router — router.invalidate — https://reactrouter.com/api/data-routers/Router
- MDN — Fetch API — https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
- MDN — AbortController — https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN — async function — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- MDN — await — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
- MDN — Promise.all — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
- MDN — ARIA: status role — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/status_role
- MDN — ARIA: alert role — https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/alert_role
- W3C WAI — ARIA Authoring Practices — https://www.w3.org/WAI/ARIA/apg/
- Nielsen Norman Group — Empty States — https://www.nngroup.com/articles/empty-state-interface-design/
- web.dev — Code splitting — https://web.dev/articles/reduce-javascript-payloads-with-code-splitting