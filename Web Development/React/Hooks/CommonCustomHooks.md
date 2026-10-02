# Common Custom Hooks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Common custom Hooks are battle-tested, reusable `use`-prefixed functions that encapsulate frequently-needed stateful behaviour—data fetching, debouncing, storage, media queries, dimension tracking, online status, form logic, and interaction patterns—so that React components can consume them without re-implementing the same logic.

**Technical Definition:** Common custom Hooks are idiomatic React patterns that combine built-in Hooks (`useState`, `useEffect`, `useRef`, `useCallback`, `useMemo`, `useSyncExternalStore`) into reusable units for recurring problems. They leverage browser APIs (`fetch`, `AbortController`, `setTimeout`, `setInterval`, `localStorage`, `matchMedia`, `ResizeObserver`, `IntersectionObserver`, `navigator.onLine`, WebSocket, Clipboard, Keyboard) and follow the Rules of Hooks. Each Hook must manage its own cleanup, dependency arrays, and referential stability; expose a minimal, typed API; and preserve the isolation guarantee (each call gets its own state). Modern implementations increasingly use `useSyncExternalStore` for browser API subscriptions to prevent tearing under concurrent rendering. Each Hook documented here is a template: the implementation is a starting point, not a prescription, and libraries like `react-use`, `usehooks-ts`, `ahooks`, and TanStack Query provide production-ready versions of most of them.

**Beginner-Friendly Explanation:** Some problems come up in almost every React app: fetching data, debouncing a search input, saving to `localStorage`, checking the window size, detecting dark mode, watching online status, handling a form, or detecting a click outside. Instead of writing the same code over and over, you write a custom Hook once—`useFetch`, `useDebounce`, `useLocalStorage`, `useWindowSize`, `useMediaQuery`, `useOnlineStatus`, `useForm`, `useClickOutside`—and reuse it everywhere. This cheat sheet provides battle-tested implementations of the eight most common Hooks, each with proper cleanup, typing, and edge-case handling.

### Key Characteristics

- **Reusable Across Projects:** Common Hooks solve problems that appear in nearly every app.
- **Cleanup-First Design:** Every Hook that subscribes (timers, listeners, observers) cleans up in `useEffect`'s return function.
- **Isolated State:** Each call to a common Hook gets its own state instance; sharing requires lifting, Context, or a store.
- **Stable APIs:** Common Hooks return stable functions (via `useCallback`) so consumers can use them in dependency arrays.
- **Typed Contracts:** Generic parameters and typed return values preserve inference across the Hook boundary.
- **SSR-Safe:** Browser API access is guarded (`typeof window !== 'undefined'`) or deferred to effects.
- **`useSyncExternalStore` Where Appropriate:** Browser APIs (online status, media queries, dimensions) benefit from `useSyncExternalStore` to prevent tearing.
- **Library Alternatives Exist:** `react-use`, `usehooks-ts`, `ahooks`, `@uidotdev/usehooks`, TanStack Query, and React Hook Form provide production-ready versions.

### Prerequisites

- Solid understanding of React function components, JSX, and the primary Hooks.
- Working knowledge of the Rules of Hooks and dependency arrays.
- Familiarity with browser APIs (`fetch`, `setTimeout`, `matchMedia`, `ResizeObserver`, `navigator.onLine`, `localStorage`).
- Basic understanding of `useSyncExternalStore` and when it is appropriate.
- Awareness of TypeScript generics for typed Hook APIs.

### Related Programming Areas

- **Server-State Management:** TanStack Query, SWR.
- **Form Architecture:** React Hook Form, Formik, TanStack Form.
- **Browser APIs:** Storage, media queries, observers, network status.
- **Interaction Design:** Hover, focus, click-outside, keyboard shortcuts.
- **Concurrency:** Debouncing, throttling, race-condition guards.

### Core Concepts / Features

1. Data Fetching (`useFetch`)
2. Debouncing & Throttling (`useDebounce`, `useThrottle`)
3. Local / Session Storage (`useLocalStorage`, `useSessionStorage`)
4. Media Queries & Theme Detection (`useMediaQuery`, `usePrefersColorScheme`)
5. Window Dimensions & Element Bounds (`useWindowSize`, `useElementSize`)
6. Online/Offline Status (`useOnlineStatus`)
7. Form Logic (`useForm`)
8. Interaction Hooks (`useHover`, `useClickOutside`, `useFocusTrap`, `useKeyPress`)

---

## Core Concept 1: Data Fetching (`useFetch`)

### Definitions

**Core Definition:** `useFetch` is a custom Hook that fetches data from a URL and returns the request's loading, error, and data states, with cancellation, deduplication, and stale-while-revalidate semantics where appropriate.

**Technical Definition:** A production-grade `useFetch` Hook wraps the `fetch` API with `AbortController` for cancellation, tracks a discriminated-union status (`idle | loading | success | error`), and re-runs the fetch when the URL or options change. The Hook uses `useEffect` (or `useSyncExternalStore` for a shared cache) to synchronise the fetch lifecycle with the component's render cycle. A minimal `useFetch` is straightforward; a production-grade `useFetch` (with cache, deduplication, retries, and stale-while-revalidate) is a significant undertaking—for which TanStack Query or SWR is the recommended choice. This cheat sheet presents a minimal, correct `useFetch` with cancellation and a discriminated union, plus guidance on when to reach for a library.

**Beginner-Friendly Explanation:** `useFetch` lets a component say "fetch this URL and give me the data, loading state, and error." It handles the request lifecycle, cancels the request if the component unmounts or the URL changes, and lets you render loading, error, and success states. For anything beyond simple cases—caching, refetching, mutations, pagination—use TanStack Query or SWR.

### Purposes

- To fetch data from a URL with loading, error, and success states.
- To cancel in-flight requests when the component unmounts or the URL changes.
- To prevent race conditions where a slow earlier request overwrites a newer result.
- To provide a typed API for the fetched data.
- To serve as a starting point when a full server-state library is overkill.

### Syntax Rules and Structure

**Minimal `useFetch` with AbortController:**
```tsx
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function useFetch<T>(url: string, options?: RequestInit): FetchState<T> & { refetch: () => void } {
  const [state, setState] = React.useState<FetchState<T>>({ status: 'idle' });
  const [reloadKey, setReloadKey] = React.useState(0);

  React.useEffect(() => {
    const controller = new AbortController();
    setState({ status: 'loading' });

    fetch(url, { ...options, signal: controller.signal })
      .then(async (res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data = (await res.json()) as T;
        setState({ status: 'success', data });
      })
      .catch((error: Error) => {
        if (error.name === 'AbortError') return;
        setState({ status: 'error', error });
      });

    return () => controller.abort();
  }, [url, reloadKey]);

  const refetch = React.useCallback(() => setReloadKey((k) => k + 1), []);
  return { ...state, refetch };
}
```

**Component Breakdown:**
- `FetchState<T>`: Discriminated union of the four request states.
- `AbortController`: Cancels the request on cleanup.
- `reloadKey`: Manual refetch trigger.
- `refetch`: Stable function to trigger a refetch.

**Stale-While-Revalidate with `useSyncExternalStore`:**
```tsx
// A minimal SWR-like cache shared across component instances
const cache = new Map<string, { data: unknown; timestamp: number }>();
const listeners = new Set<() => void>();

function subscribe(listener: () => void) {
  listeners.add(listener);
  return () => listeners.delete(listener);
}

function getSnapshot(url: string) {
  return cache.get(url) ?? null;
}

function revalidate(url: string) {
  fetch(url)
    .then((res) => res.json())
    .then((data) => {
      cache.set(url, { data, timestamp: Date.now() });
      listeners.forEach((l) => l());
    });
}

function useSWR<T>(url: string, staleTime = 30_000): { data: T | null; isStale: boolean } {
  const snapshot = React.useSyncExternalStore(subscribe, () => getSnapshot(url));

  React.useEffect(() => {
    const entry = cache.get(url);
    const isStale = !entry || Date.now() - entry.timestamp > staleTime;
    if (isStale) revalidate(url);
  }, [url, staleTime]);

  const isStale = !snapshot || Date.now() - snapshot.timestamp > staleTime;
  return { data: snapshot?.data as T | null, isStale };
}
```

**Component Breakdown:**
- `cache`: A shared `Map` keyed by URL.
- `subscribe` / `getSnapshot`: `useSyncExternalStore` contract.
- The effect revalidates when the cached entry is stale.

**Syntax Rules:**
- Always use `AbortController` to cancel in-flight requests.
- Ignore `AbortError` in the catch block; it is expected on cleanup.
- Check `res.ok` before parsing JSON.
- Model the state as a discriminated union (`idle | loading | success | error`).
- Include the URL (and any serialised options) in the effect's dependency array.
- Return a stable `refetch` function via `useCallback`.
- For anything beyond simple cases, use TanStack Query or SWR.
- Do not store the fetched data in Redux or Zustand; it is server state.

**Constraints and Limitations:**
- A minimal `useFetch` does not cache across component instances; each mount fetches again.
- It does not deduplicate concurrent requests for the same URL.
- It does not retry on failure.
- It does not support mutations, pagination, or optimistic updates.
- It does not integrate with Suspense (see React 19's `use` for that).
- For production, TanStack Query's `useQuery` is preferred.

### Annotated Code Example: User Profile with `useFetch`

```tsx
type User = { id: number; name: string; email: string };

function UserProfile({ userId }: { userId: number }) {
    const state = useFetch<User>(`/api/users/${userId}`);
  
    switch (state.status) {
        case 'idle':
        case 'loading':
          return <p>Loading user...</p>;
        case 'error':
            return (
                <div role="alert">
                    <p>Error: {state.error.message}</p>
                    <button onClick={state.refetch}>Retry</button>
                </div>
            );
        case 'success':
            return (
                <div>
                    <h1>{state.data.name}</h1>
                    <p>{state.data.email}</p>
                </div>
            );
    }
}
```

**Expected Output:** The component shows "Loading user..." initially, then either the user's name and email, or an error with a "Retry" button. If the component unmounts or `userId` changes before the request resolves, the request is aborted.

**Why This Output Occurs:** The discriminated union allows exhaustive switching over the four states. The `AbortController` cancels the request on cleanup. The `refetch` function is stable and triggers a re-fetch.

### Real-World Cases

- **Simple dashboards:** Fetching a single resource on mount.
- **Detail pages:** Fetching a resource by ID from route parameters.
- **Prototypes:** A minimal `useFetch` for demos.
- **Production apps:** TanStack Query or SWR instead of `useFetch`.

### References

- React Official Documentation – Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – You Might Not Need an Effect - https://react.dev/learn/you-might-not-need-an-effect
- MDN Web Docs – AbortController - https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- TanStack Query – Overview - https://tanstack.com/query/latest/docs/framework/react/overview
- SWR – Documentation - https://swr.vercel.app/
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore

---

## Core Concept 2: Debouncing & Throttling (`useDebounce`, `useThrottle`)

### Definitions

**Core Definition:** Debouncing delays a value update until the input has been stable for a specified time; throttling limits updates to at most once per interval. Both reduce the frequency of expensive operations triggered by rapid input.

**Technical Definition:** `useDebounce(value, delay)` returns a debounced copy of `value` that lags behind the original until `delay` ms have passed without a change. `useThrottle(value, interval)` returns a throttled copy that updates at most once per `interval`. Both use `useState` and `useEffect` with `setTimeout` and cleanup to cancel the previous timer. Debouncing is for input-driven events (search, resize) where the final value matters. Throttling is for continuous events (scroll, mousemove) where periodic updates are acceptable. A common variant is `useDebouncedCallback`, which returns a debounced function rather than a value—useful for imperative calls. `useDebounce` is not a replacement for `useTransition` or `useDeferredValue`, which defer rendering rather than delaying state updates.

**Beginner-Friendly Explanation:** If you fire a request on every keystroke, you overwhelm the server. Debouncing waits until the user pauses typing (say, 300ms) before firing. Throttling fires at most once per interval (say, once per 100ms) while the user keeps typing. Debouncing is for "what did they finally type?" and throttling is for "how often should I check?" Both avoid the same problem: too many requests.

### Purposes

- To delay expensive operations (search, validation) until the input stabilises.
- To limit the frequency of scroll, resize, and mousemove handlers.
- To reduce server load from rapid input.
- To provide a stable debounced value that can be used as a dependency.
- To provide a debounced callback for imperative calls.

### Syntax Rules and Structure

**`useDebounce` (Value):**
```tsx
function useDebounce<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = React.useState(value);

  React.useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}
```

**Component Breakdown:**
- `value`: The value to debounce.
- `delay`: The debounce delay in milliseconds.
- The effect resets the timer on every change to `value`.

**`useThrottle` (Value):**
```tsx
function useThrottle<T>(value: T, interval = 100): T {
  const [throttled, setThrottled] = React.useState(value);
  const lastRun = React.useRef(Date.now());

  React.useEffect(() => {
    const now = Date.now();
    if (now - lastRun.current >= interval) {
      lastRun.current = now;
      setThrottled(value);
    } else {
      const timer = setTimeout(() => {
        lastRun.current = Date.now();
        setThrottled(value);
      }, interval - (now - lastRun.current));
      return () => clearTimeout(timer);
    }
  }, [value, interval]);

  return throttled;
}
```

**Component Breakdown:**
- `lastRun`: Tracks the last time the throttled value was updated.
- If enough time has passed, update immediately; otherwise, schedule a trailing update.

**`useDebouncedCallback` (Function):**
```tsx
function useDebouncedCallback<Args extends unknown[]>(
  callback: (...args: Args) => void,
  delay = 300
): (...args: Args) => void {
  const timeoutRef = React.useRef<ReturnType<typeof setTimeout> | null>(null);
  const callbackRef = React.useRef(callback);

  React.useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  return React.useCallback((...args: Args) => {
    if (timeoutRef.current) clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => callbackRef.current(...args), delay);
  }, [delay]);
}
```

**Component Breakdown:**
- `timeoutRef`: Holds the debounce timer.
- `callbackRef`: Keeps the latest callback without re-creating the debounced function.
- The returned function is stable; its identity does not change when `callback` changes.

**Syntax Rules:**
- Use `useDebounce` for input values (search, resize).
- Use `useThrottle` for continuous events (scroll, mousemove).
- Use `useDebouncedCallback` for imperative calls where the function itself is debounced.
- Always clear the previous timer in cleanup.
- Keep the `delay` or `interval` in the dependency array.
- Use the debounced value as a dependency for effects that fetch or compute.
- Debouncing delays the update; it does not defer rendering (`useDeferredValue` does that).
- Consider 300ms for search inputs and 100ms for scroll throttling.

**Constraints and Limitations:**
- Debouncing adds latency; 300ms is imperceptible, 1000ms is not.
- Throttling can miss the final value; the trailing update handles this.
- Debouncing a controlled input's value causes the input to lag behind the user's typing; debounce the derived value (e.g., the query sent to the server), not the input itself.
- Neither Hook cancels the underlying work; it only delays it.
- In React 18+, `useDeferredValue` may be a better fit for expensive renders.

### Annotated Code Example: Debounced Search

```tsx
function SearchBox() {
  const [query, setQuery] = React.useState('');
  const debouncedQuery = useDebounce(query, 300);

  const state = useFetch<SearchResult[]>(
    debouncedQuery ? `/api/search?q=${encodeURIComponent(debouncedQuery)}` : ''
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {state.status === 'loading' && <p>Searching...</p>}
      {state.status === 'error' && <p role="alert">Error: {state.error.message}</p>}
      {state.status === 'success' && (
        <ul>
          {state.data.map((r) => <li key={r.id}>{r.title}</li>)}
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** Typing in the input updates the value immediately. After 300ms of inactivity, the search request fires. Results are displayed when they arrive. Rapid typing fires only one request for the final query.

**Why This Output Occurs:** `useDebounce(query, 300)` returns a debounced query. `useFetch` depends on the debounced query, so it re-runs only when the debounced value changes. The input remains responsive because its value is not debounced.

### Real-World Cases

- **Search autocomplete:** Debounce the query by 300ms.
- **Window resize:** Throttle the resize handler by 100ms.
- **Scroll tracking:** Throttle the scroll handler by 100ms.
- **Form validation:** Debounce server-side validation by 500ms.
- **API rate limiting:** Throttle outgoing requests by 1000ms.

### References

- React Official Documentation – `useDeferredValue` - https://react.dev/reference/react/useDeferredValue
- MDN Web Docs – `setTimeout()` - https://developer.mozilla.org/en-US/docs/Web/API/setTimeout
- usehooks-ts – `useDebounce` - https://usehooks-ts.com/react-hook/use-debounce
- usehooks-ts – `useDebounceCallback` - https://usehooks-ts.com/react-hook/use-debounce-callback
- Lodash – `debounce` and `throttle` - https://lodash.com/docs/#debounce

---

## Core Concept 3: Local / Session Storage (`useLocalStorage`, `useSessionStorage`)

### Definitions

**Core Definition:** `useLocalStorage` and `useSessionStorage` are custom Hooks that synchronise a piece of React state with the Web Storage API, persisting values across page loads and, optionally, across browser tabs.

**Technical Definition:** The Web Storage API provides two mechanisms: `localStorage` (persists until cleared) and `sessionStorage` (persists until the tab closes). Both store strings, so the Hook must serialise values with `JSON.stringify` and deserialise with `JSON.parse`. The Hook uses a lazy `useState` initialiser to read the stored value on mount, an effect to write updates back to storage, and a `storage` event listener to sync changes from other tabs. The `storage` event fires only in other tabs, not the tab that made the change. Cross-tab sync is optional but valuable for multi-tab apps. The Hook must handle JSON parse errors, storage quota errors, and SSR (where `window` is undefined). A generic `<T>` preserves the value's type; the tuple return `[T, (value: T) => void]` preserves the setter's type. `useSyncExternalStore` can be used for a more robust implementation that prevents tearing.

**Beginner-Friendly Explanation:** `useLocalStorage` is like `useState`, but the value survives a page reload. When the user closes and reopens the tab, the value is still there. `useSessionStorage` is the same, except the value is cleared when the tab closes. The Hook serialises the value to JSON, writes it on every change, and reads it on mount. If you want two tabs to stay in sync, the Hook can listen to the `storage` event and update the state when another tab changes the value.

### Purposes

- To persist state across page reloads (theme, locale, draft forms).
- To store session-scoped data (tab-specific filters, temporary state).
- To sync state across tabs (for `localStorage`).
- To provide a typed API for persisted values.
- To handle serialisation and deserialisation safely.

### Syntax Rules and Structure

**`useLocalStorage` with Cross-Tab Sync:**
```tsx
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [stored, setStored] = React.useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = React.useCallback(
    (value: T) => {
      try {
        setStored(value);
        window.localStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key]
  );

  React.useEffect(() => {
    function handleStorage(event: StorageEvent) {
      if (event.key === key && event.newValue !== null) {
        try {
          setStored(JSON.parse(event.newValue) as T);
        } catch (error) {
          console.warn(`Error parsing storage event for "${key}":`, error);
        }
      }
    }
    window.addEventListener('storage', handleStorage);
    return () => window.removeEventListener('storage', handleStorage);
  }, [key]);

  return [stored, setValue];
}
```

**Component Breakdown:**
- Lazy initialiser: Reads the value on mount, guarded for SSR.
- `setValue`: Writes to both state and `localStorage`; stable via `useCallback`.
- `storage` event: Syncs changes from other tabs.

**`useSessionStorage` (Same Pattern, Different API):**
```tsx
function useSessionStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  // Same as useLocalStorage but uses window.sessionStorage
}
```

**Component Breakdown:**
- `sessionStorage` clears when the tab closes; no cross-tab sync.

**Typed Usage:**
```tsx
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Theme: {theme}
    </button>
  );
}
```

**Component Breakdown:**
- `<T>` preserves the type; `setTheme` accepts only `'light' | 'dark'`.

**Syntax Rules:**
- Use a lazy initialiser to read the stored value synchronously on mount.
- Guard against `typeof window === 'undefined'` for SSR.
- Wrap `JSON.parse` and `JSON.stringify` in `try/catch`.
- Use `useCallback` for the setter with `[key]` as its dependency.
- Listen to the `storage` event for cross-tab sync (only `localStorage`).
- Type the Hook generically: `useLocalStorage<T>(key, initial): [T, (v: T) => void]`.
- Never store sensitive data (tokens, passwords) in `localStorage`.
- Keep stored values small; `localStorage` is synchronous and has a ~5MB limit.

**Constraints and Limitations:**
- `localStorage` is synchronous and blocks the main thread for large values.
- `localStorage` only stores strings; complex objects require JSON serialisation.
- The `storage` event fires only in other tabs, not the current one.
- Quota errors occur when storage is full; handle them gracefully.
- SSR requires guarding or a fallback to `initialValue`.
- Cross-tab sync is eventually consistent; there may be a brief delay.

### Annotated Code Example: Persisted Theme with Cross-Tab Sync

```tsx
function ThemeApp() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

  React.useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
  }, [theme]);

  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Switch to {theme === 'light' ? 'dark' : 'light'}
      </button>
      <p>Current theme: {theme}</p>
    </div>
  );
}
```

**Expected Output:** The button toggles the theme, updates the `data-theme` attribute, and persists the choice. Opening the app in a second tab and toggling the theme updates the first tab as well, because of the `storage` event listener.

**Why This Output Occurs:** `useLocalStorage` reads the stored theme on mount, writes on change, and syncs across tabs via the `storage` event. The `useEffect` applies the theme to the document.

### Real-World Cases

- **Theme preferences:** Persist light/dark mode.
- **User settings:** Language, sidebar state, table column widths.
- **Form drafts:** Persist incomplete forms across sessions.
- **Cart:** Persist a cart across page loads.
- **Session filters:** Tab-scoped filters with `sessionStorage`.

### References

- MDN Web Docs – `Window.localStorage` - https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – `Window.sessionStorage` - https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage
- MDN Web Docs – `StorageEvent` - https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent
- usehooks-ts – `useLocalStorage` - https://usehooks-ts.com/react-hook/use-local-storage
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore

---

## Core Concept 4: Media Queries & Theme Detection (`useMediaQuery`, `usePrefersColorScheme`)

### Definitions

**Core Definition:** `useMediaQuery` subscribes to a CSS media query and returns whether it currently matches; `usePrefersColorScheme` returns the user's preferred colour scheme (`'light' | 'dark'`) and updates when the system preference changes.

**Technical Definition:** Both Hooks use `window.matchMedia(query)` to obtain a `MediaQueryList`, then subscribe to its `change` event to update state when the match status changes. `useMediaQuery` accepts any valid CSS media query string (`'(min-width: 768px)'`, `'(prefers-reduced-motion: reduce)'`) and returns a boolean. `usePrefersColorScheme` is a specialised wrapper around `'(prefers-color-scheme: dark)'` that returns `'light' | 'dark'`. The recommended implementation uses `useSyncExternalStore` with `matchMedia` to prevent tearing during concurrent rendering. For SSR, `getServerSnapshot` should return a sensible default (e.g., `false` or `'light'`) to avoid hydration mismatches. These Hooks are the foundation of responsive components that adapt to viewport size and user preferences.

**Beginner-Friendly Explanation:** `useMediaQuery` asks the browser "does this media query match right now?" and re-asks whenever the answer changes. For example, `useMediaQuery('(min-width: 768px)')` tells your component whether the viewport is at least 768px wide. `usePrefersColorScheme` tells you whether the user has set their system to dark or light mode, and it updates automatically when they change it. These Hooks let your React components respond to CSS-level conditions without duplicating the logic in CSS.

### Purposes

- To detect whether a media query matches (viewport size, orientation, user preferences).
- To detect the user's preferred colour scheme (`light` or `dark`).
- To detect user preferences for reduced motion, contrast, and transparency.
- To drive conditional rendering based on device characteristics.
- To synchronise React state with CSS-level conditions.

### Syntax Rules and Structure

**`useMediaQuery` with `useSyncExternalStore`:**
```tsx
function useMediaQuery(query: string): boolean {
  const subscribe = React.useCallback(
    (callback: () => void) => {
      const mql = window.matchMedia(query);
      mql.addEventListener('change', callback);
      return () => mql.removeEventListener('change', callback);
    },
    [query]
  );

  const getSnapshot = React.useCallback(() => window.matchMedia(query).matches, [query]);
  const getServerSnapshot = () => false;

  return React.useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}
```

**Component Breakdown:**
- `subscribe`: Registers a `change` listener on the `MediaQueryList`.
- `getSnapshot`: Returns the current match status.
- `getServerSnapshot`: Returns `false` during SSR to avoid hydration mismatches.

**`usePrefersColorScheme`:**
```tsx
function usePrefersColorScheme(): 'light' | 'dark' {
  const isDark = useMediaQuery('(prefers-color-scheme: dark)');
  return isDark ? 'dark' : 'light';
}
```

**Component Breakdown:**
- `useMediaQuery` reads the `prefers-color-scheme: dark` query.
- Returns `'dark'` when the user prefers dark mode; `'light'` otherwise.
- Updates automatically when the system preference changes.

**Additional Preference Hooks:**
```tsx
const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');
const prefersMoreContrast = useMediaQuery('(prefers-contrast: more)');
const isPortrait = useMediaQuery('(orientation: portrait)');
const isDesktop = useMediaQuery('(min-width: 1024px)');
```

**Syntax Rules:**
- Use `useSyncExternalStore` for the most robust implementation.
- Provide a `getServerSnapshot` that returns a sensible default for SSR.
- Use valid CSS media query strings, including the parentheses.
- Keep the `query` string stable; if dynamic, include it in the dependencies.
- Prefer `usePrefersColorScheme` over `window.matchMedia` directly.
- Combine with CSS variables for theme switching (see the CSS cheat sheet).
- Respect `prefers-reduced-motion` by disabling animations.
- Test on real devices; DevTools emulation is approximate.

**Constraints and Limitations:**
- `matchMedia` is not available during SSR; the `getServerSnapshot` must return a default.
- Hydration mismatches occur if the server and client render different content; use CSS-based solutions where possible.
- Media queries do not support custom properties (e.g., `var(--breakpoint)`).
- Older browsers may not support `addEventListener` on `MediaQueryList`; use `addListener` as a fallback.
- Some queries (e.g., `prefers-color-scheme`) are not supported in older browsers.

### Annotated Code Example: Responsive Layout with Theme Detection

```tsx
function ResponsiveLayout({ children }: { children: React.ReactNode }) {
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  const prefersDark = usePrefersColorScheme() === 'dark';
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');

  return (
    <div
      data-theme={prefersDark ? 'dark' : 'light'}
      data-reduced-motion={prefersReducedMotion}
    >
      {isDesktop ? (
        <div className="sidebar-layout">
          <aside>Sidebar</aside>
          <main>{children}</main>
        </div>
      ) : (
        <main>{children}</main>
      )}
    </div>
  );
}
```

**Expected Output:** The layout renders with a sidebar on desktop and without one on mobile. The `data-theme` attribute reflects the user's colour-scheme preference, and `data-reduced-motion` reflects their motion preference.

**Why This Output Occurs:** `useMediaQuery` subscribes to the viewport size and updates when the user resizes. `usePrefersColorScheme` subscribes to the colour-scheme preference. Both update automatically when the conditions change.

### Real-World Cases

- **Responsive layouts:** Show/hide a sidebar based on viewport width.
- **Theme detection:** Apply dark mode based on system preference.
- **Accessibility:** Respect `prefers-reduced-motion` and `prefers-contrast`.
- **Orientation:** Adjust layout for portrait vs. landscape.
- **Print styles:** Detect `print` media.

### References

- MDN Web Docs – `window.matchMedia()` - https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia
- MDN Web Docs – `MediaQueryList` - https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList
- MDN Web Docs – `prefers-color-scheme` - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme
- MDN Web Docs – `prefers-reduced-motion` - https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- usehooks-ts – `useMediaQuery` - https://usehooks-ts.com/react-hook/use-media-query
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore

---

## Core Concept 5: Window Dimensions & Element Bounds (`useWindowSize`, `useElementSize`)

### Definitions

**Core Definition:** `useWindowSize` tracks the viewport's width and height; `useElementSize` tracks the dimensions of a specific DOM element using `ResizeObserver`.

**Technical Definition:** `useWindowSize` subscribes to the `resize` event on `window` and reads `window.innerWidth` and `window.innerHeight`. The recommended implementation uses `useSyncExternalStore` to prevent tearing. `useElementSize` uses `ResizeObserver` to observe a specific element and returns its `contentRect` (or `getBoundingClientRect()`). `ResizeObserver` fires when the element's size changes, including changes caused by layout (e.g., a flexbox sibling growing). The Hook returns a `ref` to attach to the element, plus the observed `width` and `height`. Both Hooks clean up their listeners/observers on unmount. For high-frequency updates, consider throttling or using `requestAnimationFrame`.

**Beginner-Friendly Explanation:** `useWindowSize` tells you how wide and tall the browser window is, and updates when the user resizes it. `useElementSize` tells you how wide and tall a specific element is, and updates when that element changes size—even if the browser window does not. `useElementSize` uses `ResizeObserver`, a browser API that watches a specific element. These Hooks power responsive layouts, charts that resize to fit, and virtualised lists.

### Purposes

- To track the viewport's dimensions for responsive layouts.
- To track a specific element's dimensions for charts, canvas, or virtual lists.
- To update a component when the viewport or element resizes.
- To avoid media-query breakpoints when component-level sizing is needed.
- To power container-query-like behaviour in JavaScript.

### Syntax Rules and Structure

**`useWindowSize` with `useSyncExternalStore`:**
```tsx
function useWindowSize(): { width: number; height: number } {
  const subscribe = React.useCallback((callback: () => void) => {
    window.addEventListener('resize', callback);
    return () => window.removeEventListener('resize', callback);
  }, []);

  const getSnapshot = () => ({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  const getServerSnapshot = () => ({ width: 0, height: 0 });

  return React.useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}
```

**Component Breakdown:**
- `subscribe`: Adds and removes the `resize` listener.
- `getSnapshot`: Returns the current dimensions.
- `getServerSnapshot`: Returns a default during SSR.
- Note: `getSnapshot` must return a stable reference for the same values; in practice, React re-renders when the object identity changes. For a more robust implementation, cache the object.

**`useElementSize` with `ResizeObserver`:**
```tsx
function useElementSize<T extends HTMLElement>(): [
  React.RefObject<T>,
  { width: number; height: number }
] {
  const ref = React.useRef<T>(null);
  const [size, setSize] = React.useState({ width: 0, height: 0 });

  React.useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        const { width, height } = entry.contentRect;
        setSize({ width, height });
      }
    });

    observer.observe(element);
    return () => observer.disconnect();
  }, []);

  return [ref, size];
}
```

**Component Breakdown:**
- `ref`: Attached to the element to observe.
- `ResizeObserver`: Fires when the element's size changes.
- `entry.contentRect`: The element's content box dimensions.
- Cleanup disconnects the observer.

**Typed Usage:**
```tsx
function ResponsiveChart() {
  const [ref, { width, height }] = useElementSize<HTMLDivElement>();

  return (
    <div ref={ref} style={{ width: '100%', height: 300 }}>
      <svg width={width} height={height}>
        {/* chart rendered at the measured size */}
      </svg>
    </div>
  );
}
```

**Component Breakdown:**
- The chart's `<svg>` dimensions match the measured container size.
- The chart re-renders when the container resizes.

**Syntax Rules:**
- Use `useSyncExternalStore` for `useWindowSize` to prevent tearing.
- Use `ResizeObserver` for `useElementSize`; `window`'s `resize` event does not fire for element size changes.
- Always disconnect the observer in cleanup.
- Provide a sensible default for SSR (`{ width: 0, height: 0 }`).
- For high-frequency updates, throttle or use `requestAnimationFrame`.
- Debounce the values if they drive expensive renders.
- Prefer CSS container queries over `useElementSize` when possible.

**Constraints and Limitations:**
- `window.innerWidth` includes the scrollbar; use `document.documentElement.clientWidth` for content width.
- `ResizeObserver` fires on every size change; frequency depends on layout.
- SSR cannot access `window` or `ResizeObserver`; the Hook must guard.
- `ResizeObserver` is not supported in very old browsers.
- Returning a new object from `getSnapshot` on every call can cause infinite re-renders; cache the object.

### Annotated Code Example: Responsive Chart with `useElementSize`

```tsx
function MetricChart({ data }: { data: number[] }) {
  const [ref, { width, height }] = useElementSize<HTMLDivElement>();

  const points = React.useMemo(() => {
    if (width === 0 || data.length === 0) return '';
    const max = Math.max(...data);
    return data
      .map((value, i) => {
        const x = (i / (data.length - 1)) * width;
        const y = height - (value / max) * height;
        return `${x},${y}`;
      })
      .join(' ');
  }, [data, width, height]);

  return (
    <div ref={ref} style={{ width: '100%', height: 200 }}>
      <svg width={width} height={height}>
        <polyline points={points} fill="none" stroke="steelblue" strokeWidth={2} />
      </svg>
    </div>
  );
}
```

**Expected Output:** A line chart that fills its container and re-renders with the correct dimensions when the container resizes (e.g., when the sidebar collapses).

**Why This Output Occurs:** `useElementSize` measures the container. When the container's size changes, `ResizeObserver` fires, `setSize` updates the state, and the component re-renders with the new dimensions and recomputed SVG points.

### Real-World Cases

- **Charts:** Sizing charts to their container.
- **Virtualised lists:** Measuring the scroll container.
- **Responsive layouts:** Adjusting layout based on viewport size.
- **Canvas:** Setting canvas dimensions to match the element.
- **Tooltips/popovers:** Positioning based on trigger dimensions.

### References

- MDN Web Docs – `Window.innerWidth` - https://developer.mozilla.org/en-US/docs/Web/API/Window/innerWidth
- MDN Web Docs – `ResizeObserver` - https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver
- MDN Web Docs – `Element.getBoundingClientRect()` - https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
- usehooks-ts – `useWindowSize` - https://usehooks-ts.com/react-hook/use-window-size
- usehooks-ts – `useResizeObserver` - https://usehooks-ts.com/react-hook/use-resize-observer
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore

---

## Core Concept 6: Online/Offline Status (`useOnlineStatus`)

### Definitions

**Core Definition:** `useOnlineStatus` is a custom Hook that returns whether the browser is currently online, updating automatically when the network status changes.

**Technical Definition:** The Hook subscribes to the `online` and `offline` events on `window` and reads `navigator.onLine` for the initial value. `navigator.onLine` is a boolean that reflects whether the browser has a network connection (though it does not guarantee internet access—captive portals report `true`). The recommended implementation uses `useSyncExternalStore` with `navigator.onLine` as the snapshot, `window`'s `online`/`offline` events as the subscription, and `true` as the server snapshot. The Hook is often combined with a banner or toast that notifies the user when the connection is lost, and with retry logic that re-runs failed requests when the connection is restored. TanStack Query's `refetchOnReconnect` uses this pattern internally.

**Beginner-Friendly Explanation:** `useOnlineStatus` tells your component whether the user's browser thinks it is online. When the user loses Wi-Fi or mobile data, the Hook updates and your component can show a banner saying "You are offline." When the connection returns, the Hook updates again, and your component can retry failed requests or hide the banner.

### Purposes

- To display a banner or toast when the user goes offline.
- To disable actions that require network access.
- To trigger retries of failed requests when the connection returns.
- To provide a graceful offline experience.
- To detect captive portals (indirectly, via failed requests).

### Syntax Rules and Structure

**`useOnlineStatus` with `useSyncExternalStore`:**
```tsx
function useOnlineStatus(): boolean {
  const subscribe = React.useCallback((callback: () => void) => {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
      window.removeEventListener('online', callback);
      window.removeEventListener('offline', callback);
    };
  }, []);

  const getSnapshot = () => navigator.onLine;
  const getServerSnapshot = () => true;

  return React.useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}
```

**Component Breakdown:**
- `subscribe`: Registers the `online` and `offline` listeners.
- `getSnapshot`: Returns `navigator.onLine`.
- `getServerSnapshot`: Returns `true` during SSR (assume online).

**Usage with a Banner:**
```tsx
function OfflineBanner() {
  const isOnline = useOnlineStatus();

  if (isOnline) return null;

  return (
    <div role="status" aria-live="polite" className="offline-banner">
      You are offline. Changes will be saved when the connection returns.
    </div>
  );
}
```

**Component Breakdown:**
- The banner renders only when offline.
- `role="status"` and `aria-live="polite"` announce the change to screen readers.

**Retry on Reconnect:**
```tsx
function useRetryOnReconnect(refetch: () => void) {
  const isOnline = useOnlineStatus();
  const wasOnline = React.useRef(isOnline);

  React.useEffect(() => {
    if (!wasOnline.current && isOnline) {
      refetch();
    }
    wasOnline.current = isOnline;
  }, [isOnline, refetch]);
}
```

**Component Breakdown:**
- Tracks the previous online status.
- When the status transitions from offline to online, calls `refetch`.

**Syntax Rules:**
- Use `useSyncExternalStore` for the most robust implementation.
- Listen to both `online` and `offline` events.
- Use `navigator.onLine` for the initial and current snapshot.
- Provide `true` as the server snapshot.
- Combine with TanStack Query's `refetchOnReconnect` for automatic retries.
- Show a non-blocking banner or toast; do not block the UI.
- Test with Chrome DevTools' Network panel set to "Offline".

**Constraints and Limitations:**
- `navigator.onLine` reports whether the browser has a network interface, not whether the internet is reachable (captive portals report `true`).
- The events may not fire in all environments (e.g., some mobile browsers).
- The Hook does not detect slow connections; use `navigator.connection` for that (experimental).
- Server-side rendering cannot know the client's status; the server snapshot is a guess.
- The Hook does not queue or retry requests; combine with a data-fetching library for that.

### Annotated Code Example: Offline Banner with Retry

```tsx
function OfflineAwareApp() {
  const isOnline = useOnlineStatus();
  const queryClient = useQueryClient();

  React.useEffect(() => {
    if (isOnline) {
      queryClient.refetchQueries({ type: 'active' });
    }
  }, [isOnline, queryClient]);

  return (
    <div>
      {!isOnline && (
        <div role="status" aria-live="polite" style={{ background: '#fef3c7', padding: 8 }}>
          You are offline. Reconnecting...
        </div>
      )}
      <main>{/* app content */}</main>
    </div>
  );
}
```

**Expected Output:** When the user goes offline, a yellow banner appears. When the connection returns, the banner disappears and active queries are refetched.

**Why This Output Occurs:** `useOnlineStatus` updates when the network changes. The effect refetches active queries when the status becomes `true`. The banner is rendered conditionally.

### Real-World Cases

- **PWAs:** Show an offline banner and serve cached content.
- **E-commerce:** Queue orders and retry on reconnect.
- **Chat apps:** Show "Connecting..." when the connection drops.
- **Dashboards:** Pause polling when offline.
- **Form apps:** Save drafts locally and sync on reconnect.

### References

- MDN Web Docs – `navigator.onLine` - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine
- MDN Web Docs – `online` event - https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event
- MDN Web Docs – `offline` event - https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event
- TanStack Query – `refetchOnReconnect` - https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- usehooks-ts – `useNetworkState` - https://usehooks-ts.com/react-hook/use-network-state

---

## Core Concept 7: Form Logic (`useForm`)

### Definitions

**Core Definition:** A minimal `useForm` custom Hook tracks form values, validation errors, touched fields, and submission state, providing change and submit handlers—though production forms typically use React Hook Form instead.

**Technical Definition:** A minimal `useForm<T>` Hook manages a state object of values, a state object of errors, a state object of touched fields, and a submission status. It returns `values`, `errors`, `touched`, `handleChange`, `handleBlur`, `handleSubmit`, and `reset`. Validation can be synchronous (schema-based or custom) or asynchronous; the Hook runs validation on change, blur, or submit depending on configuration. For anything beyond trivial forms, React Hook Form is the recommended choice because it minimises re-renders (uncontrolled inputs), integrates with schema resolvers (Zod), and provides `useFieldArray`, `Controller`, and `FormProvider`. This cheat sheet presents a minimal `useForm` to illustrate the pattern and recommends React Hook Form for production.

**Beginner-Friendly Explanation:** A form Hook tracks what the user typed, whether each field has been touched, whether the values are valid, and whether the form is submitting. It gives you the values and handlers you need to bind to inputs. For simple forms, you can write this yourself. For real forms—with validation, dynamic fields, and submission—use React Hook Form, which handles the hard parts.

### Purposes

- To track form values, errors, and touched fields.
- To validate values on change, blur, or submit.
- To handle submission with loading and error states.
- To provide a reset function.
- To illustrate the pattern before adopting a library.

### Syntax Rules and Structure

**Minimal `useForm<T>`:**
```tsx
type Validator<T> = (values: T) => Partial<Record<keyof T, string>>;

function useForm<T extends Record<string, unknown>>(
  initialValues: T,
  validate?: Validator<T>
) {
  const [values, setValues] = React.useState<T>(initialValues);
  const [errors, setErrors] = React.useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = React.useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = React.useState(false);

  const handleChange = React.useCallback(
    (name: keyof T) => (e: React.ChangeEvent<HTMLInputElement>) => {
      const value = e.target.type === 'checkbox' ? e.target.checked : e.target.value;
      setValues((v) => ({ ...v, [name]: value }));
    },
    []
  );

  const handleBlur = React.useCallback(
    (name: keyof T) => () => {
      setTouched((t) => ({ ...t, [name]: true }));
      if (validate) {
        const validationErrors = validate(values);
        setErrors(validationErrors);
      }
    },
    [validate, values]
  );

  const handleSubmit = React.useCallback(
    (onSubmit: (values: T) => Promise<void> | void) =>
      async (e: React.FormEvent) => {
        e.preventDefault();
        const validationErrors = validate ? validate(values) : {};
        setErrors(validationErrors);
        setTouched(Object.keys(values).reduce((acc, k) => ({ ...acc, [k]: true }), {}));
        if (Object.keys(validationErrors).length > 0) return;

        setIsSubmitting(true);
        try {
          await onSubmit(values);
        } finally {
          setIsSubmitting(false);
        }
      },
    [values, validate]
  );

  const reset = React.useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  return { values, errors, touched, isSubmitting, handleChange, handleBlur, handleSubmit, reset };
}
```

**Component Breakdown:**
- `values`, `errors`, `touched`, `isSubmitting`: Form state.
- `handleChange(name)`: Returns a change handler for a field.
- `handleBlur(name)`: Returns a blur handler that validates.
- `handleSubmit(onSubmit)`: Returns a submit handler.
- `reset`: Resets the form.

**Usage:**
```tsx
type LoginValues = { email: string; password: string };

function LoginForm({ onLogin }: { onLogin: (email: string, password: string) => Promise<void> }) {
  const form = useForm<LoginValues>(
    { email: '', password: '' },
    (values) => {
      const errors: Partial<Record<keyof LoginValues, string>> = {};
      if (!values.email.includes('@')) errors.email = 'Invalid email';
      if (values.password.length < 8) errors.password = 'At least 8 characters';
      return errors;
    }
  );

  return (
    <form onSubmit={form.handleSubmit((v) => onLogin(v.email, v.password))}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" value={form.values.email} onChange={form.handleChange('email')} onBlur={form.handleBlur('email')} />
        {form.touched.email && form.errors.email && <span role="alert">{form.errors.email}</span>}
      </div>
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" type="password" value={form.values.password} onChange={form.handleChange('password')} onBlur={form.handleBlur('password')} />
        {form.touched.password && form.errors.password && <span role="alert">{form.errors.password}</span>}
      </div>
      <button type="submit" disabled={form.isSubmitting}>{form.isSubmitting ? 'Logging in...' : 'Log In'}</button>
    </form>
  );
}
```

**Component Breakdown:**
- The form binds `values`, `handleChange`, `handleBlur`, `errors`, and `touched`.
- Errors are shown only after the field has been touched.
- The submit button is disabled during submission.

**Recommended: React Hook Form for Production:**
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

type FormValues = z.infer<typeof schema>;

function LoginForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: { email: '', password: '' },
  });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <input type="password" {...register('password')} />
      {errors.password && <span role="alert">{errors.password.message}</span>}
      <button type="submit" disabled={isSubmitting}>Log In</button>
    </form>
  );
}
```

**Component Breakdown:**
- `register`: Connects inputs to RHF without controlled re-renders.
- `formState.errors`: Typed errors.
- `isSubmitting`: True during submission.

**Syntax Rules:**
- For trivial forms (1–3 fields), a custom `useForm` is acceptable.
- For anything beyond trivial, use React Hook Form.
- Type the form values with a generic or a schema.
- Validate on blur for the first pass, then on change to clear errors.
- Show errors only after the field is touched.
- Disable the submit button during submission.
- Provide a reset function.
- Use `aria-invalid` and `aria-describedby` for accessible errors.

**Constraints and Limitations:**
- A minimal `useForm` re-renders on every keystroke; RHF does not.
- It does not support dynamic field arrays; use `useFieldArray`.
- It does not support schema validation; use Zod/Yup with RHF.
- It does not handle async validation, race conditions, or stale results.
- It is a teaching example; production code should use RHF.

### Annotated Code Example: Minimal Form with Validation

```tsx
function RegistrationForm() {
  const form = useForm<{ name: string; email: string }>(
    { name: '', email: '' },
    (values) => {
      const errors: Partial<Record<'name' | 'email', string>> = {};
      if (values.name.length < 2) errors.name = 'Name is too short';
      if (!values.email.includes('@')) errors.email = 'Invalid email';
      return errors;
    }
  );

  return (
    <form onSubmit={form.handleSubmit((values) => alert(JSON.stringify(values)))}>
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          value={form.values.name}
          onChange={form.handleChange('name')}
          onBlur={form.handleBlur('name')}
          aria-invalid={!!form.errors.name}
        />
        {form.touched.name && form.errors.name && <span role="alert">{form.errors.name}</span>}
      </div>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          value={form.values.email}
          onChange={form.handleChange('email')}
          onBlur={form.handleBlur('email')}
          aria-invalid={!!form.errors.email}
        />
        {form.touched.email && form.errors.email && <span role="alert">{form.errors.email}</span>}
      </div>
      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? 'Submitting...' : 'Register'}
      </button>
    </form>
  );
}
```

**Expected Output:** A registration form with name and email fields. Errors appear on blur if the values are invalid. Submitting with valid values shows an alert.

**Why This Output Occurs:** `useForm` tracks values, errors, and touched. `handleBlur` runs validation when the user leaves a field. `handleSubmit` validates and calls the callback.

### Real-World Cases

- **Login and signup forms:** Email, password, name.
- **Profile editing:** Name, bio, avatar.
- **Contact forms:** Name, email, message.
- **Checkout forms:** Address, payment.
- **Multi-step wizards:** Each step has its own fields.

### References

- React Hook Form – Documentation - https://react-hook-form.com/
- React Hook Form – TypeScript - https://react-hook-form.com/ts
- React Hook Form – `useForm` - https://react-hook-form.com/docs/useform
- React Hook Form – `useFieldArray` - https://react-hook-form.com/docs/usefieldarray
- @hookform/resolvers – GitHub - https://github.com/react-hook-form/resolvers
- Zod – Documentation - https://zod.dev/

---

## Core Concept 8: Interaction Hooks (`useHover`, `useClickOutside`, `useFocusTrap`, `useKeyPress`)

### Definitions

**Core Definition:** Interaction Hooks detect user interactions—hovering, clicking outside, trapping focus, and pressing keys—so components can respond to input without duplicating listener logic.

**Technical Definition:** Interaction Hooks attach DOM event listeners (`mouseenter`/`mouseleave`, `mousedown`, `keydown`, `focusin`/`focusout`), update state in response, and clean up on unmount. `useHover` returns a ref and a boolean indicating whether the element is hovered. `useClickOutside` returns a ref and calls a callback when a click occurs outside the referenced element. `useFocusTrap` traps focus within a container (for modals and dialogs) by intercepting Tab and Shift+Tab. `useKeyPress` listens for a specific key (or key combination) and invokes a callback. All Hooks must use the correct event phase (`capture` vs. bubble), handle cleanup, and avoid stale closures via refs or stable callbacks. These Hooks are the building blocks of accessible, interactive components.

**Beginner-Friendly Explanation:** These Hooks detect what the user is doing. `useHover` tells you when the mouse is over an element. `useClickOutside` tells you when the user clicks elsewhere (to close a dropdown). `useFocusTrap` keeps the Tab key inside a modal so users cannot tab behind it. `useKeyPress` listens for shortcuts like Escape or Ctrl+K. Each Hook wires up a listener, updates state, and cleans up when the component unmounts.

### Purposes

- **`useHover`:** To show tooltips, highlight items, or reveal actions on hover.
- **`useClickOutside`:** To close dropdowns, modals, and popovers when the user clicks elsewhere.
- **`useFocusTrap`:** To keep keyboard focus within a modal or dialog.
- **`useKeyPress`:** To implement keyboard shortcuts (Escape to close, Ctrl+K to search).

### Syntax Rules and Structure

**`useHover`:**
```tsx
function useHover<T extends HTMLElement>(): [React.RefObject<T>, boolean] {
  const ref = React.useRef<T>(null);
  const [isHovered, setIsHovered] = React.useState(false);

  React.useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const onEnter = () => setIsHovered(true);
    const onLeave = () => setIsHovered(false);

    element.addEventListener('mouseenter', onEnter);
    element.addEventListener('mouseleave', onLeave);
    return () => {
      element.removeEventListener('mouseenter', onEnter);
      element.removeEventListener('mouseleave', onLeave);
    };
  }, []);

  return [ref, isHovered];
}

// Usage
function HoverCard() {
  const [ref, isHovered] = useHover<HTMLDivElement>();
  return (
    <div ref={ref}>
      {isHovered ? <p>Hovered!</p> : <p>Hover me</p>}
    </div>
  );
}
```

**`useClickOutside`:**
```tsx
function useClickOutside<T extends HTMLElement>(
  onClickOutside: () => void
): React.RefObject<T> {
  const ref = React.useRef<T>(null);
  const callbackRef = React.useRef(onClickOutside);

  React.useEffect(() => {
    callbackRef.current = onClickOutside;
  }, [onClickOutside]);

  React.useEffect(() => {
    function handleClick(event: MouseEvent) {
      if (ref.current && !ref.current.contains(event.target as Node)) {
        callbackRef.current();
      }
    }
    document.addEventListener('mousedown', handleClick);
    return () => document.removeEventListener('mousedown', handleClick);
  }, []);

  return ref;
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = React.useState(false);
  const ref = useClickOutside<HTMLDivElement>(() => setIsOpen(false));

  return (
    <div ref={ref}>
      <button onClick={() => setIsOpen((o) => !o)}>Toggle</button>
      {isOpen && <ul><li>Item 1</li><li>Item 2</li></ul>}
    </div>
  );
}
```

**`useFocusTrap`:**
```tsx
const FOCUSABLE = [
  'a[href]',
  'button:not([disabled])',
  'textarea:not([disabled])',
  'input:not([disabled])',
  'select:not([disabled])',
  '[tabindex]:not([tabindex="-1"])',
].join(',');

function useFocusTrap<T extends HTMLElement>(isActive: boolean): React.RefObject<T> {
  const ref = React.useRef<T>(null);

  React.useEffect(() => {
    if (!isActive) return;
    const element = ref.current;
    if (!element) return;

    const previousFocus = document.activeElement as HTMLElement | null;
    const focusables = element.querySelectorAll<HTMLElement>(FOCUSABLE);
    const first = focusables[0];
    const last = focusables[focusables.length - 1];
    first?.focus();

    function handleKeyDown(event: KeyboardEvent) {
      if (event.key !== 'Tab') return;
      const currentFocusables = element!.querySelectorAll<HTMLElement>(FOCUSABLE);
      const firstEl = currentFocusables[0];
      const lastEl = currentFocusables[currentFocusables.length - 1];

      if (event.shiftKey && document.activeElement === firstEl) {
        event.preventDefault();
        lastEl?.focus();
      } else if (!event.shiftKey && document.activeElement === lastEl) {
        event.preventDefault();
        firstEl?.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      previousFocus?.focus();
    };
  }, [isActive]);

  return ref;
}

// Usage
function Modal({ isOpen, onClose, children }: { isOpen: boolean; onClose: () => void; children: React.ReactNode }) {
  const ref = useFocusTrap<HTMLDivElement>(isOpen);

  if (!isOpen) return null;
  return (
    <div ref={ref} role="dialog" aria-modal="true">
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

**`useKeyPress`:**
```tsx
function useKeyPress(targetKey: string, callback: (event: KeyboardEvent) => void) {
  const callbackRef = React.useRef(callback);

  React.useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  React.useEffect(() => {
    function handleKeyDown(event: KeyboardEvent) {
      if (event.key === targetKey) callbackRef.current(event);
    }
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [targetKey]);
}

// Usage
function SearchShortcut() {
  useKeyPress('k', (e) => {
    if (e.ctrlKey || e.metaKey) {
      e.preventDefault();
      document.getElementById('search')?.focus();
    }
  });
  return <input id="search" placeholder="Search (Ctrl+K)" />;
}
```

**Syntax Rules:**
- Use `mouseenter`/`mouseleave` (not `mouseover`/`mouseout`) for hover; they do not bubble.
- Use `mousedown` (not `click`) for click-outside to fire before the click handler.
- Use a ref (`callbackRef`) to avoid stale closures in event listeners.
- Trap focus only when the container is active (e.g., a modal is open).
- Restore focus to the previously focused element on cleanup.
- Use `event.key` (not `event.keyCode`) for keyboard events.
- Check modifier keys (`ctrlKey`, `metaKey`, `shiftKey`, `altKey`) for shortcuts.
- Clean up all listeners on unmount.
- Use `aria-modal="true"` and `role="dialog"` with `useFocusTrap`.

**Constraints and Limitations:**
- `useHover` does not work on touch devices; provide alternative interactions.
- `useClickOutside` fires on `mousedown`; if the target is a portal, `contains` may fail—check the portal's root.
- `useFocusTrap` must handle dynamically added focusable elements; re-query on each Tab press.
- `useKeyPress` with a single key may conflict with browser shortcuts; check modifiers.
- Custom focus traps are error-prone; prefer a library (`focus-trap-react`) or the native `<dialog>`.
- Native `<dialog>` element with `showModal()` handles focus trapping, Escape, and backdrop automatically.

### Annotated Code Example: Accessible Dropdown with Click-Outside and Key Press

```tsx
function AccessibleDropdown({ items }: { items: string[] }) {
  const [isOpen, setIsOpen] = React.useState(false);
  const [activeIndex, setActiveIndex] = React.useState(0);
  const ref = useClickOutside<HTMLDivElement>(() => setIsOpen(false));
  const buttonRef = React.useRef<HTMLButtonElement>(null);

  useKeyPress('Escape', () => {
    if (isOpen) {
      setIsOpen(false);
      buttonRef.current?.focus();
    }
  });

  function handleKeyDown(e: React.KeyboardEvent) {
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      setActiveIndex((i) => (i + 1) % items.length);
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      setActiveIndex((i) => (i - 1 + items.length) % items.length);
    }
  }

  return (
    <div ref={ref} onKeyDown={handleKeyDown}>
      <button
        ref={buttonRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        onClick={() => setIsOpen((o) => !o)}
      >
        Actions
      </button>
      {isOpen && (
        <ul role="menu">
          {items.map((item, i) => (
            <li
              key={item}
              role="menuitem"
              tabIndex={-1}
              aria-selected={i === activeIndex}
              onClick={() => setIsOpen(false)}
            >
              {item}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Expected Output:** A dropdown that opens on click, closes on Escape (returning focus to the button), and closes on click outside. Arrow keys move the active item.

**Why This Output Occurs:** `useClickOutside` attaches a `mousedown` listener to the document and closes the dropdown if the click is outside. `useKeyPress('Escape', ...)` closes the dropdown and returns focus. The inline `onKeyDown` handles arrow navigation.

### Real-World Cases

- **Tooltips and popovers:** `useHover` for showing/hiding.
- **Dropdowns and menus:** `useClickOutside` for closing.
- **Modals and dialogs:** `useFocusTrap` for keyboard trapping.
- **Keyboard shortcuts:** `useKeyPress` for Ctrl+K, Escape, etc.
- **Command palettes:** Combination of `useKeyPress` and `useFocusTrap`.

### References

- React Official Documentation – Responding to Events - https://react.dev/learn/responding-to-events
- React Official Documentation – Manipulating the DOM with Refs - https://react.dev/learn/manipulating-the-dom-with-refs
- MDN Web Docs – `mouseenter` - https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseenter_event
- MDN Web Docs – `mousedown` - https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event
- MDN Web Docs – `keydown` - https://developer.mozilla.org/en-US/docs/Web/API/Element/keydown_event
- MDN Web Docs – `<dialog>` - https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog
- focus-trap-react – npm - https://www.npmjs.com/package/focus-trap-react
- usehooks-ts – `useHover` - https://usehooks-ts.com/react-hook/use-hover
- usehooks-ts – `useClickAnyWhere` - https://usehooks-ts.com/react-hook/use-click-any-where
- usehooks-ts – `useKeyPress` - https://usehooks-ts.com/react-hook/use-key-press

---

## Comparison and Decision Guidance

| Hook | Primary Browser API | Cleanup | When to Use | Library Alternative |
|---|---|---|---|---|
| **`useFetch`** | `fetch`, `AbortController` | Abort request | Simple one-off fetches | TanStack Query, SWR |
| **`useDebounce`** | `setTimeout` | Clear timer | Input, resize | `use-debounce` |
| **`useThrottle`** | `setTimeout` | Clear timer | Scroll, mousemove | `use-debounce` (throttle) |
| **`useLocalStorage`** | `localStorage` | Remove `storage` listener | Persisted state | `usehooks-ts` |
| **`useSessionStorage`** | `sessionStorage` | — | Tab-scoped state | `usehooks-ts` |
| **`useMediaQuery`** | `matchMedia` | Remove listener | Responsive breakpoints | `usehooks-ts` |
| **`usePrefersColorScheme`** | `matchMedia` | Remove listener | Dark mode detection | `usehooks-ts` |
| **`useWindowSize`** | `resize` event | Remove listener | Viewport tracking | `usehooks-ts` |
| **`useElementSize`** | `ResizeObserver` | Disconnect observer | Container sizing | `usehooks-ts` |
| **`useOnlineStatus`** | `navigator.onLine`, `online`/`offline` | Remove listeners | Offline UX | `usehooks-ts` |
| **`useForm`** | — | — | Trivial forms | React Hook Form |
| **`useHover`** | `mouseenter`/`mouseleave` | Remove listeners | Tooltips, highlights | `usehooks-ts` |
| **`useClickOutside`** | `mousedown` | Remove listener | Dropdowns, modals | `usehooks-ts` |
| **`useFocusTrap`** | `keydown` (Tab) | Remove listener, restore focus | Modals, dialogs | `focus-trap-react`, `<dialog>` |
| **`useKeyPress`** | `keydown` | Remove listener | Keyboard shortcuts | `react-hotkeys-hook` |

**Decision Guidance:**
- **Start with a library:** `usehooks-ts`, `react-use`, or `ahooks` provides production-ready versions of most of these Hooks.
- **Use TanStack Query or SWR** for data fetching; do not write `useFetch` for production.
- **Use React Hook Form** for forms; do not write `useForm` for production.
- **Use `useSyncExternalStore`** for browser API subscriptions (online status, media queries, dimensions) to prevent tearing.
- **Use `ResizeObserver`** for element size; `window`'s `resize` event does not fire for element changes.
- **Use `useDeferredValue`** instead of debouncing when the goal is to defer rendering.
- **Use `focus-trap-react`** or the native `<dialog>` element instead of writing a focus trap.
- **Use `react-hotkeys-hook`** for complex keyboard shortcuts.
- **Extract a custom Hook** only when the same logic appears twice, or when a browser API subscription needs encapsulation.

---

## References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – `useDeferredValue` - https://react.dev/reference/react/useDeferredValue
- React Official Documentation – `useTransition` - https://react.dev/reference/react/useTransition
- MDN Web Docs – `AbortController` - https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – `setTimeout()` - https://developer.mozilla.org/en-US/docs/Web/API/setTimeout
- MDN Web Docs – `Window.localStorage` - https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN Web Docs – `Window.sessionStorage` - https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage
- MDN Web Docs – `StorageEvent` - https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent
- MDN Web Docs – `window.matchMedia()` - https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia
- MDN Web Docs – `MediaQueryList` - https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList
- MDN Web Docs – `ResizeObserver` - https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver
- MDN Web Docs – `navigator.onLine` - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine
- MDN Web Docs – `online` event - https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event
- MDN Web Docs – `offline` event - https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event
- MDN Web Docs – `mouseenter` - https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseenter_event
- MDN Web Docs – `mousedown` - https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event
- MDN Web Docs – `keydown` - https://developer.mozilla.org/en-US/docs/Web/API/Element/keydown_event
- MDN Web Docs – `<dialog>` - https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog
- usehooks-ts - https://usehooks-ts.com/
- react-use - https://github.com/streamich/react-use
- ahooks - https://ahooks.js.org/
- @uidotdev/usehooks - https://usehooks.com/
- TanStack Query – Overview - https://tanstack.com/query/latest/docs/framework/react/overview
- SWR – Documentation - https://swr.vercel.app/
- React Hook Form – Documentation - https://react-hook-form.com/
- focus-trap-react – npm - https://www.npmjs.com/package/focus-trap-react
- react-hotkeys-hook – npm - https://www.npmjs.com/package/react-hotkeys-hook
- Testing Library – `renderHook` - https://testing-library.com/docs/react-testing-library/api#renderhook