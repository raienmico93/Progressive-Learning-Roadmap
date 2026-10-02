# Advanced Custom Hooks: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced custom Hooks are `use`-prefixed functions that go beyond simple state encapsulation—using TypeScript generics, composition, resource lifecycle management, async state machines, external-store subscriptions, and SSR-safe execution to solve complex, reusable problems across a codebase.

**Technical Definition:** Advanced custom Hooks extend the fundamentals of custom Hooks (extraction, isolation, composition) with type-level programming (generics, conditional types, `as const`), lifecycle discipline (`AbortController`, `ResizeObserver`, listener teardown), async state machines (discriminated unions of `Idle | Pending | Resolved | Rejected` with race-condition guards), external-store integration (`useSyncExternalStore` for browser APIs and third-party stores to prevent tearing under concurrent rendering), and SSR/hydration safety (guarding browser-only APIs with `typeof window` checks, `useEffect` execution, and `getServerSnapshot` defaults). These Hooks are the foundation of shared libraries, design systems, and production-grade applications, and they must respect the Rules of Hooks, expose stable references, and clean up every resource they create.

**Beginner-Friendly Explanation:** Basic custom Hooks are reusable functions for state. Advanced custom Hooks are reusable functions for *complex* logic—the kind that involves TypeScript generics, nested Hooks, network cancellation, async state transitions, browser APIs, and server rendering. If a basic Hook is a reusable recipe, an advanced Hook is a reusable recipe that scales to any ingredient, cleans up after itself, handles failure, and works whether it runs in the browser or on the server.

### Key Characteristics

- **Type-Level Generics:** Hooks like `useHistory<T>`, `useSelection<T>`, and `useFetch<T>` preserve the caller's type through the Hook boundary.
- **Composition Over Monoliths:** Advanced Hooks compose smaller Hooks rather than accumulating responsibilities.
- **Resource Lifecycle Discipline:** Every subscription, timer, observer, and request has an explicit teardown.
- **Async State Machines:** Complex async logic is modelled as a discriminated union of states with exhaustive handling and race-condition guards.
- **External-Store Safety:** `useSyncExternalStore` replaces ad-hoc `useEffect` + `useState` subscriptions for browser APIs and external stores.
- **SSR and Hydration Awareness:** Browser-only APIs are guarded, deferred to effects, or given server snapshots.
- **Stable Public API:** Return values are stable where they need to be (functions, objects passed to other Hooks) and reactive where they need to be (data, status).
- **Library-Grade Quality:** These patterns are the ones used in TanStack Query, SWR, React Aria, and design systems.

### Prerequisites

- Solid understanding of React function components, JSX, and all primary Hooks.
- Working knowledge of custom Hook fundamentals (extraction, naming, Rules of Hooks, composition, dependency management).
- Intermediate TypeScript skills (generics, unions, discriminated unions, conditional types, `as const`).
- Familiarity with browser APIs (`fetch`, `AbortController`, `matchMedia`, `ResizeObserver`, `localStorage`, `navigator.onLine`).
- Basic understanding of SSR and hydration in Next.js or Remix.
- Awareness of React 18's concurrent rendering and tearing.

### Related Programming Areas

- **TypeScript Advanced Types:** Generics, conditional types, mapped types, `infer`.
- **Server-State Management:** TanStack Query, SWR (which are built from these patterns).
- **Concurrency:** `useSyncExternalStore`, `useTransition`, `useDeferredValue`.
- **Browser APIs:** Observers, Media Queries, Storage, Network.
- **SSR Frameworks:** Next.js App Router, Remix, React Server Components.

### Core Concepts / Features

1. Generic Hooks
2. Composable Hooks
3. Resource Management & Cleanup
4. Async State Machines
5. External-Store Integration (`useSyncExternalStore`)
6. SSR & Hydration Safety

---

## Core Concept 1: Generic Hooks

### Definitions

**Core Definition:** A generic Hook is a custom Hook that accepts a type parameter (`<T>`), allowing it to work with any data type while preserving full type safety for the consumer.

**Technical Definition:** A generic Hook is declared as `function useThing<T>(...): ReturnType<T>` and infers `T` from its arguments (or accepts it explicitly). TypeScript propagates `T` through the Hook's internal state, callbacks, and return value, so the consumer gets the same type they passed in—not `any` or `unknown`. Generic Hooks often use `as const` for tuple returns, `extends` constraints to require certain shapes (e.g., `<T extends { id: string | number }>`), and utility types (`Pick`, `Omit`, `Partial`) to derive sub-types. They are the foundation of reusable data utilities (`useHistory<T>`, `useSelection<T>`, `useLocalStorage<T>`, `useFetch<T>`, `useForm<T>`) and are essential for library-grade Hooks.

**Beginner-Friendly Explanation:** A generic Hook is like a container that adapts to whatever you put in it. `useHistory<number>` tracks a history of numbers; `useHistory<string>` tracks a history of strings. The Hook's logic is the same; only the type changes. This is how you write one Hook that works for every data type without losing TypeScript's safety.

### Purposes

- To write one Hook that works with any data type.
- To preserve the caller's type through the Hook boundary.
- To constrain the type when the Hook requires certain properties.
- To derive sub-types from the generic (`Pick<T, K>`, `Omit<T, K>`).
- To provide autocomplete and type checking for consumers.
- To build library-grade Hooks that scale across projects.

### Syntax Rules and Structure

**Basic Generic Hook (`useHistory<T>`):**
```tsx
function useHistory<T>(initialValue: T) {
  const [history, setHistory] = React.useState<T[]>([initialValue]);

  const push = React.useCallback((value: T) => {
    setHistory((h) => [...h, value]);
  }, []);

  const undo = React.useCallback(() => {
    setHistory((h) => (h.length > 1 ? h.slice(0, -1) : h));
  }, []);

  const reset = React.useCallback(() => {
    setHistory([initialValue]);
  }, [initialValue]);

  const current = history[history.length - 1];

  return { current, history, push, undo, reset };
}

// Usage — T is inferred as number
const { current, push, undo } = useHistory(0);
push(1); // OK
// push('a'); // Error: string is not assignable to number
```

**Component Breakdown:**
- `<T>`: Declares the generic type parameter.
- `initialValue: T`: Infers `T` from the argument.
- `history: T[]`: The state is an array of `T`.
- `push: (value: T) => void`: Only accepts the correct type.
- `current: T`: The latest value, correctly typed.

**Generic Hook with Constraint (`useSelection<T extends { id: string | number }>`):**
```tsx
function useSelection<T extends { id: string | number }>(items: T[]) {
  const [selectedIds, setSelectedIds] = React.useState<Set<T['id']>>(new Set());

  const toggle = React.useCallback((id: T['id']) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id);
      else next.add(id);
      return next;
    });
  }, []);

  const clear = React.useCallback(() => setSelectedIds(new Set()), []);

  const isSelected = React.useCallback(
    (id: T['id']) => selectedIds.has(id),
    [selectedIds]
  );

  const selectedItems = React.useMemo(
    () => items.filter((item) => selectedIds.has(item.id)),
    [items, selectedIds]
  );

  return { selectedIds, selectedItems, toggle, clear, isSelected };
}

// Usage — T is inferred as User
type User = { id: number; name: string };
const { selectedItems, toggle } = useSelection<User>(users);
// selectedItems: User[]
// toggle: (id: number) => void
```

**Component Breakdown:**
- `<T extends { id: string | number }>`: Constrains `T` to have an `id` property.
- `T['id']`: Indexed access type for the ID.
- `selectedItems: T[]`: The selected objects, correctly typed.

**Generic Hook with `as const` for Tuple Return:**
```tsx
function useToggle<T = boolean>(initial: T): [T, () => void, (value: T) => void] {
  const [value, setValue] = React.useState(initial);
  const toggle = React.useCallback(
    () => setValue((v) => (v === initial ? (initial as unknown as T) : initial)),
    [initial]
  );
  return [value, toggle, setValue] as const;
}

// Usage
const [isOpen, toggleOpen, setOpen] = useToggle(false);
// isOpen: boolean, toggleOpen: () => void, setOpen: (v: boolean) => void
```

**Component Breakdown:**
- `as const`: Preserves the tuple structure.
- `[T, () => void, (value: T) => void]`: The explicit tuple return type.
- Destructuring gives each element its correct type.

**Generic Hook with Derived Types:**
```tsx
function useTable<T extends { id: string | number }>(rows: T[]) {
  type Column = {
    key: keyof T;
    header: string;
    render?: (row: T) => React.ReactNode;
  };

  const [columns, setColumns] = React.useState<Column[]>([]);

  const addColumn = React.useCallback((column: Column) => {
    setColumns((c) => [...c, column]);
  }, []);

  return { rows, columns, addColumn };
}
```

**Component Breakdown:**
- `Column.key: keyof T`: Only valid keys of `T` are accepted.
- `render: (row: T) => ReactNode`: The row is correctly typed.

**Syntax Rules:**
- Declare the type parameter immediately after the Hook name: `function useThing<T>(...)`.
- Infer `T` from arguments whenever possible; only require explicit typing when inference fails.
- Use `extends` to constrain `T` when the Hook requires certain properties.
- Use `T['key']` (indexed access) to extract property types.
- Use `keyof T` to type keys of `T`.
- Use `as const` or an explicit tuple type for tuple returns.
- Use utility types (`Pick`, `Omit`, `Partial`) to derive sub-types from `T`.
- Document whether `T` is inferred or must be specified.

**Constraints and Limitations:**
- TypeScript cannot infer `T` if it appears only in the return type without a corresponding argument.
- Overly constrained generics reduce flexibility; use the loosest constraint that works.
- Generic Hooks with `useCallback` and `useMemo` require careful dependency typing.
- Generic Hooks cannot be memoised with `React.memo` directly; wrap the calling component instead.
- The generic is erased at runtime; there is no way to introspect `T`.
- Complex conditional types inside Hooks can hurt editor performance.

### Annotated Code Example: `useSelection<T>` with Select-All

```tsx
function useSelection<T extends { id: string | number }>(items: T[]) {
  const [selectedIds, setSelectedIds] = React.useState<Set<T['id']>>(() => new Set());

  const toggle = React.useCallback((id: T['id']) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }, []);

  const selectAll = React.useCallback(() => {
    setSelectedIds(new Set(items.map((item) => item.id)));
  }, [items]);

  const clear = React.useCallback(() => setSelectedIds(new Set()), []);

  const isSelected = React.useCallback((id: T['id']) => selectedIds.has(id), [selectedIds]);

  const selectedItems = React.useMemo(
    () => items.filter((item) => selectedIds.has(item.id)),
    [items, selectedIds]
  );

  const isAllSelected = items.length > 0 && selectedIds.size === items.length;

  return { selectedIds, selectedItems, isAllSelected, toggle, selectAll, clear, isSelected };
}

// Usage with a User type
type User = { id: number; name: string; email: string };

function UserTable({ users }: { users: User[] }) {
  const { selectedItems, isAllSelected, toggle, selectAll, clear } = useSelection(users);

  return (
    <div>
      <header>
        <button onClick={selectAll} disabled={isAllSelected}>Select All</button>
        <button onClick={clear}>Clear</button>
        <span>{selectedItems.length} selected</span>
      </header>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            <input type="checkbox" checked={selectedItems.some((u) => u.id === user.id)} onChange={() => toggle(user.id)} />
            {user.name}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** A table of users with a checkbox per row, a "Select All" button, a "Clear" button, and a count of selected users. TypeScript infers `T = User`, so `selectedItems` is `User[]`, `toggle` accepts a `number`, and `selectAll`/`clear` are stable functions.

**Why This Output Occurs:** The generic `<T extends { id: string | number }>` preserves the `User` type through the Hook. `T['id']` types the ID as `number`. The `Set<T['id']>` state and the `items.filter` operation are all correctly typed.

### Real-World Cases

- **`useHistory<T>`:** Undo/redo for any value type.
- **`useSelection<T>`:** Multi-select for tables, lists, and galleries.
- **`useLocalStorage<T>`:** Persisting any type.
- **`useFetch<T>`:** Fetching any data type.
- **`useForm<T>`:** Typing form values.
- **`useTable<T>`:** Data tables with typed columns and rows.
- **`usePagination<T>`:** Paginated lists with typed items.

### References

- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Generic Constraints - https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints
- TypeScript Handbook – Indexed Access Types - https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Handbook – `keyof` Type Operator - https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook – `as const` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions
- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- Matt Pocock – Generic Hooks - https://www.totaltypescript.com/

---

## Core Concept 2: Composable Hooks

### Definitions

**Core Definition:** Composable Hooks are custom Hooks that call other custom Hooks, layering small, focused Hooks into larger, cohesive APIs where one Hook's output directly drives another Hook's input.

**Technical Definition:** Composition in Hooks mirrors composition in components: a Hook can call any built-in Hook and any other custom Hook, as long as the Rules of Hooks are respected. Composition enables layered abstractions—for example, `useAuth` composes `useUser`, `useToken`, and `usePermissions`; `useSearch` composes `useDebounce` and `useFetch`; `useForm` composes `useField` and `useValidation`. Each layer adds behaviour without duplicating logic. Composition preserves state isolation (each call gets its own instance) and referential stability (each Hook's return values must be stable where required). The output of one Hook is often the input of another: `useDebouncedValue(query)` feeds `useFetch(url)`, and `useFetch`'s `refetch` feeds `useInterval`. Composition is React's answer to the "mixin" and "higher-order component" problems.

**Beginner-Friendly Explanation:** Composition is building bigger Hooks out of smaller ones. Instead of writing one giant Hook that does everything, you write small Hooks that each do one thing—`useDebounce`, `useFetch`, `useLocalStorage`—and then combine them: `useSearch` uses `useDebounce` and `useFetch`; `useAuth` uses `useUser` and `useToken`. Each Hook stays simple, and the composition is where the complexity lives.

### Purposes

- To build complex Hooks from small, focused ones.
- To avoid duplicating logic across Hooks.
- To layer abstractions (e.g., a facade Hook over several primitive Hooks).
- To chain one Hook's output into another Hook's input.
- To keep each Hook testable in isolation.
- To replace HOC and mixin patterns with composable functions.

### Syntax Rules and Structure

**Layered Composition (`useAuth` over `useUser`, `useToken`):**
```tsx
function useUser() {
  const [user, setUser] = React.useState<User | null>(null);
  React.useEffect(() => {
    fetch('/api/me').then((r) => r.json()).then(setUser);
  }, []);
  return { user, setUser };
}

function useToken() {
  const [token, setToken] = React.useState<string | null>(() => localStorage.getItem('token'));
  const setAndPersist = React.useCallback((value: string | null) => {
    setToken(value);
    if (value) localStorage.setItem('token', value);
    else localStorage.removeItem('token');
  }, []);
  return { token, setToken: setAndPersist };
}

function usePermissions(user: User | null) {
  return React.useMemo(() => {
    if (!user) return [];
    return user.roles.flatMap((role) => role.permissions);
  }, [user]);
}

function useAuth() {
  const { user, setUser } = useUser();
  const { token, setToken } = useToken();
  const permissions = usePermissions(user);

  const login = React.useCallback(async (credentials: Credentials) => {
    const res = await fetch('/api/login', { method: 'POST', body: JSON.stringify(credentials) });
    const { user, token } = await res.json();
    setUser(user);
    setToken(token);
  }, [setUser, setToken]);

  const logout = React.useCallback(() => {
    setUser(null);
    setToken(null);
  }, [setUser, setToken]);

  const can = React.useCallback(
    (permission: string) => permissions.includes(permission),
    [permissions]
  );

  return { user, token, permissions, can, login, logout };
}
```

**Component Breakdown:**
- `useUser`, `useToken`, `usePermissions`: Small, focused Hooks.
- `useAuth`: A facade Hook that composes them into one API.
- `can`: A derived helper that uses `permissions`.

**Chaining Hook Output → Hook Input (`useSearch`):**
```tsx
function useSearch(query: string) {
  const debouncedQuery = useDebounce(query, 300);
  const url = debouncedQuery
    ? `/api/search?q=${encodeURIComponent(debouncedQuery)}`
    : '';

  const { data, status, error, refetch } = useFetch<SearchResult[]>(url);
  const isStale = query !== debouncedQuery;

  return { results: data ?? [], status, error, refetch, isStale };
}
```

**Component Breakdown:**
- `useDebounce(query)` produces `debouncedQuery`.
- `useFetch(url)` uses the debounced query to build the URL.
- `isStale` compares the current query with the debounced one.

**Chaining Refetch → Interval (`usePolling`):**
```tsx
function usePolling<T>(url: string, interval: number) {
  const { data, status, error, refetch } = useFetch<T>(url);

  React.useEffect(() => {
    if (!interval) return;
    const id = setInterval(refetch, interval);
    return () => clearInterval(id);
  }, [interval, refetch]);

  return { data, status, error, refetch };
}
```

**Component Breakdown:**
- `useFetch` provides `refetch`.
- The interval effect calls `refetch` on a schedule.
- `refetch` must be stable (from `useCallback`) for the effect to run once.

**Composition with Conditional Logic:**
```tsx
function useOptimisticToggle(initialValue: boolean, sync: (value: boolean) => Promise<void>) {
  const [value, setValue] = React.useState(initialValue);
  const [isPending, startTransition] = React.useTransition();

  const toggle = React.useCallback(() => {
    const next = !value;
    setValue(next); // Optimistic
    startTransition(async () => {
      try {
        await sync(next);
      } catch {
        setValue(value); // Rollback on error
      }
    });
  }, [value, sync]);

  return { value, isPending, toggle };
}
```

**Component Breakdown:**
- Composes `useState`, `useTransition`, and `useCallback`.
- Updates state optimistically, syncs in a transition, rolls back on error.

**Syntax Rules:**
- Call custom Hooks at the top level of a custom Hook or component.
- Keep each Hook focused on one concern; compose rather than accumulate.
- Chain Hook outputs into Hook inputs by calling them in sequence.
- Ensure the output of one Hook is stable if it is a dependency of another.
- Return only what the caller needs; hide internal Hooks.
- Document the facade Hook's API; composition is an implementation detail.
- Test each Hook in isolation before testing the composition.

**Constraints and Limitations:**
- Composition can obscure which Hook owns which state; document the facade.
- Deeply composed Hooks are harder to debug; use React DevTools to inspect Hook state.
- A facade Hook inherits the constraints of its composed Hooks.
- Over-composition fragments logic into too many layers; keep the tree shallow.
- Composed Hooks re-render the component when any of their state changes.
- Cyclic composition (A calls B, B calls A) is impossible and would be a design error.

### Annotated Code Example: `useAuth` Composing Four Hooks

```tsx
function useAuth() {
  const { user, setUser } = useUser();
  const { token, setToken } = useToken();
  const permissions = usePermissions(user);
  const { mutate: refreshToken } = useRefreshToken(setToken);

  const login = React.useCallback(async (credentials: Credentials) => {
    const res = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const { user, token } = await res.json();
    setUser(user);
    setToken(token);
  }, [setUser, setToken]);

  const logout = React.useCallback(() => {
    setUser(null);
    setToken(null);
  }, [setUser, setToken]);

  const can = React.useCallback(
    (permission: string) => permissions.includes(permission),
    [permissions]
  );

  const isAuthenticated = user !== null && token !== null;

  return { user, token, permissions, can, isAuthenticated, login, logout, refreshToken };
}
```

**Expected Output:** A single `useAuth` Hook that exposes user, token, permissions, a `can` helper, login, logout, and refresh-token functionality. Components call `useAuth()` and get everything they need.

**Why This Output Occurs:** `useAuth` composes four smaller Hooks. Each smaller Hook manages its own state; `useAuth` combines them into one API. The functions are stable (via `useCallback`), so consumers can safely use them in dependency arrays.

### Real-World Cases

- **`useAuth`:** Composing user, token, permissions, refresh.
- **`useSearch`:** Composing `useDebounce` and `useFetch`.
- **`usePolling`:** Composing `useFetch` and `useInterval`.
- **`useForm`:** Composing `useField`, `useValidation`, `useSubmission`.
- **`useDisclosure`:** Composing `useBoolean` and `useCallback`.
- **`useSyncedState`:** Composing `useLocalStorage` and `useBroadcastChannel`.

### References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Building Your Own Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks#building-your-own-hooks
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – `useTransition` - https://react.dev/reference/react/useTransition
- React Official Documentation – `useCallback` - https://react.dev/reference/react/useCallback

---

## Core Concept 3: Resource Management & Cleanup

### Definitions

**Core Definition:** Resource management and cleanup is the discipline of ensuring that every resource a Hook creates—timers, event listeners, observers, subscriptions, WebSockets, and network requests—is released when the Hook's Effect is cleaned up or the component unmounts.

**Technical Definition:** Every Hook that creates a side effect must return a cleanup function from `useEffect` (or dispose of the resource in a `ref`-based teardown). The cleanup must: (1) cancel in-flight requests with `AbortController.abort()`; (2) remove event listeners with the exact same function reference used to add them; (3) clear timers with `clearTimeout`/`clearInterval`; (4) disconnect observers with `observer.disconnect()`; (5) unsubscribe from stores with the returned unsubscribe function; (6) close WebSockets with `socket.close()`; and (7) release any third-party handles (charts, maps, editors) with their `destroy()` method. Failing to clean up causes memory leaks, state updates on unmounted components, orphaned network requests, and duplicated listeners. In React 18+ StrictMode, Effects run twice in development (setup, cleanup, setup), so cleanup functions must be idempotent. The `useSyncExternalStore` Hook encapsulates subscribe/unsubscribe for external stores, but `useEffect` is still needed for side effects that are not simple subscriptions.

**Beginner-Friendly Explanation:** Every time a Hook starts something—a timer, a listener, a request—it must also stop it when the component goes away. If it does not, the resource stays in memory, and the app gets slower and buggier over time. Cleanup is the "undo" of the effect. React calls the cleanup function automatically when the component unmounts or when the effect re-runs. Think of it as the difference between leaving the lights on when you leave a room (a leak) and turning them off (cleanup).

### Purposes

- To prevent memory leaks from timers, listeners, and observers.
- To cancel in-flight network requests that are no longer needed.
- To avoid state updates on unmounted components.
- To prevent duplicated listeners and observers when effects re-run.
- To release third-party library instances (charts, maps, editors).
- To ensure Effects are idempotent in StrictMode.
- To keep the app's memory footprint stable over time.

### Syntax Rules and Structure

**Cancelling Fetch Requests:**
```tsx
function useFetch<T>(url: string) {
  const [data, setData] = React.useState<T | null>(null);

  React.useEffect(() => {
    const controller = new AbortController();

    fetch(url, { signal: controller.signal })
      .then((res) => res.json())
      .then(setData)
      .catch((err) => {
        if (err.name === 'AbortError') return;
        console.error(err);
      });

    return () => controller.abort();
  }, [url]);

  return data;
}
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller.
- `{ signal: controller.signal }`: Passes the signal to `fetch`.
- `return () => controller.abort()`: Cancels the request on cleanup.

**Removing Event Listeners:**
```tsx
function useEventListener<K extends keyof WindowEventMap>(
  eventName: K,
  handler: (event: WindowEventMap[K]) => void
) {
  const handlerRef = React.useRef(handler);

  React.useEffect(() => {
    handlerRef.current = handler;
  }, [handler]);

  React.useEffect(() => {
    const listener = (event: WindowEventMap[K]) => handlerRef.current(event);
    window.addEventListener(eventName, listener);
    return () => window.removeEventListener(eventName, listener);
  }, [eventName]);
}
```

**Component Breakdown:**
- `handlerRef`: Stores the latest handler without re-subscribing.
- The effect adds the listener with `listener` and removes it with the same reference.
- The `[eventName]` dependency re-subscribes only when the event name changes.

**Clearing Timers:**
```tsx
function useInterval(callback: () => void, delay: number | null) {
  const callbackRef = React.useRef(callback);

  React.useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  React.useEffect(() => {
    if (delay === null) return;
    const id = setInterval(() => callbackRef.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

**Component Breakdown:**
- `setInterval` returns an ID; `clearInterval` clears it in cleanup.
- `delay === null` pauses the interval by not setting one.

**Disconnecting Observers:**
```tsx
function useElementSize<T extends HTMLElement>() {
  const ref = React.useRef<T>(null);
  const [size, setSize] = React.useState({ width: 0, height: 0 });

  React.useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new ResizeObserver((entries) => {
      const { width, height } = entries[0].contentRect;
      setSize({ width, height });
    });

    observer.observe(element);
    return () => observer.disconnect();
  }, []);

  return [ref, size] as const;
}
```

**Component Breakdown:**
- `new ResizeObserver(callback)`: Observes size changes.
- `observer.observe(element)`: Starts observing.
- `return () => observer.disconnect()`: Stops observing.

**Closing WebSockets:**
```tsx
function useWebSocket(url: string, onMessage: (data: unknown) => void) {
  const onMessageRef = React.useRef(onMessage);

  React.useEffect(() => {
    onMessageRef.current = onMessage;
  }, [onMessage]);

  React.useEffect(() => {
    const socket = new WebSocket(url);
    socket.onmessage = (event) => onMessageRef.current(JSON.parse(event.data));
    return () => socket.close();
  }, [url]);
}
```

**Component Breakdown:**
- `new WebSocket(url)`: Opens the connection.
- `socket.close()`: Closes it on cleanup.

**Comprehensive Cleanup Example:**
```tsx
function useDashboardResources(dashboardId: string) {
  React.useEffect(() => {
    const controller = new AbortController();
    const socket = new WebSocket(`wss://api.example.com/dashboards/${dashboardId}`);
    const handleOnline = () => console.log('Online');
    const handleOffline = () => console.log('Offline');
    const intervalId = setInterval(() => { /* poll */ }, 30000);
    const observer = new ResizeObserver(() => { /* handle resize */ });

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    if (document.body) observer.observe(document.body);

    fetch(`/api/dashboards/${dashboardId}`, { signal: controller.signal });

    return () => {
      controller.abort();
      socket.close();
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
      clearInterval(intervalId);
      observer.disconnect();
    };
  }, [dashboardId]);
}
```

**Component Breakdown:**
- Every resource created is released in the cleanup.
- The order of cleanup is the reverse of setup, though React does not require this.

**Syntax Rules:**
- Always return a cleanup function from `useEffect` if the effect creates a resource.
- Use `AbortController` for every `fetch` call.
- Store the same function reference for `addEventListener` and `removeEventListener`.
- Clear every `setTimeout` and `setInterval` in cleanup.
- Disconnect every observer (`ResizeObserver`, `IntersectionObserver`, `MutationObserver`).
- Close every WebSocket, EventSource, and BroadcastChannel.
- Call `destroy()` on third-party instances (charts, maps, editors).
- Make cleanup idempotent for StrictMode (safe to call twice).
- Never leave a subscription without an unsubscribe.

**Constraints and Limitations:**
- Some resources (e.g., `AbortController`) are not available in older browsers.
- Some libraries do not provide a `destroy()` method; you may need to remove their DOM.
- Cleanup functions cannot be async; if you need to `await`, handle it separately.
- Cleanup does not run when the tab is closed; use `beforeunload` if needed.
- In StrictMode, cleanup runs twice in development; ensure it is safe.
- `useEffect` cleanup runs after the component unmounts but before the next Effect.

### Annotated Code Example: `useSubscription` with Full Cleanup

```tsx
function useSubscription<T>(topic: string, onMessage: (data: T) => void) {
  const onMessageRef = React.useRef(onMessage);

  React.useEffect(() => {
    onMessageRef.current = onMessage;
  }, [onMessage]);

  React.useEffect(() => {
    const controller = new AbortController();
    let socket: WebSocket | null = null;

    // Fetch initial snapshot
    fetch(`/api/topics/${topic}`, { signal: controller.signal })
      .then((res) => res.json())
      .then((data: T) => onMessageRef.current(data))
      .catch((err) => {
        if (err.name !== 'AbortError') console.error(err);
      });

    // Open WebSocket for live updates
    socket = new WebSocket(`wss://api.example.com/topics/${topic}`);
    socket.onmessage = (event) => onMessageRef.current(JSON.parse(event.data));

    return () => {
      controller.abort();
      socket?.close();
    };
  }, [topic]);
}
```

**Expected Output:** The Hook fetches an initial snapshot and opens a WebSocket for updates. When `topic` changes or the component unmounts, the fetch is aborted and the WebSocket is closed.

**Why This Output Occurs:** The effect creates an `AbortController` and a WebSocket. The cleanup aborts the fetch and closes the socket. The `onMessageRef` ensures the latest `onMessage` is used without re-running the effect.

### Real-World Cases

- **Real-time dashboards:** WebSockets, polling intervals, resize observers.
- **Maps and charts:** Third-party instances that must be destroyed.
- **Event listeners:** Window, document, and element listeners.
- **Data fetching:** Aborting stale requests.
- **Subscriptions:** Store subscriptions, RxJS observables.
- **Animations:** Cancelling `requestAnimationFrame` loops.

### References

- React Official Documentation – Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – You Might Not Need an Effect - https://react.dev/learn/you-might-not-need-an-effect
- MDN Web Docs – `AbortController` - https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – `ResizeObserver` - https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver
- MDN Web Docs – `IntersectionObserver` - https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver
- MDN Web Docs – `WebSocket` - https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – `EventTarget.addEventListener()` - https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore

---

## Core Concept 4: Async State Machines

### Definitions

**Core Definition:** An async state machine is a Hook that models an asynchronous operation as a discriminated union of discrete states—`Idle`, `Pending`, `Resolved`, `Rejected`—with explicit transitions, preventing race conditions and impossible states.

**Technical Definition:** An async state machine replaces multiple boolean flags (`isLoading`, `isError`, `isSuccess`) with a single state value that can be exactly one of the union's variants. Each variant carries the data it owns: `{ status: 'idle' }`, `{ status: 'pending' }`, `{ status: 'resolved'; data: T }`, `{ status: 'rejected'; error: Error }`. Transitions are triggered by events (`FETCH`, `RESOLVE`, `REJECT`, `RESET`), often handled by `useReducer`. Race conditions are prevented by a request ID or `AbortController` guard: only the latest request's resolution can transition the machine to `resolved` or `rejected`. The consumer switches on `state.status` and TypeScript narrows the type, ensuring `state.data` is only accessible in the `resolved` branch. This pattern is the foundation of TanStack Query's internal state machine and React 19's `useActionState`.

**Beginner-Friendly Explanation:** When you fetch data, the request has four possible states: not started, in progress, succeeded, or failed. If you track these with separate booleans, you can accidentally set both `isLoading` and `isError` to `true`—an impossible state. An async state machine uses a single status field, so only one state is active at a time. It also attaches the data to the correct state, so you cannot read `data` while loading. And it guards against race conditions: if you start a request and then start another, only the latest one can update the state.

### Purposes

- To replace multiple boolean flags with a single, exhaustive status.
- To attach data to the state that owns it (data in `resolved`, error in `rejected`).
- To prevent impossible states (`isLoading && isError`).
- To guard against race conditions when dependencies change rapidly.
- To provide exhaustive handling via TypeScript's discriminated unions.
- To make async logic explicit and testable.

### Syntax Rules and Structure

**Discriminated Union State:**
```tsx
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'pending' }
  | { status: 'resolved'; data: T }
  | { status: 'rejected'; error: Error };
```

**Reducer:**
```tsx
type AsyncAction<T> =
  | { type: 'FETCH' }
  | { type: 'RESOLVE'; data: T }
  | { type: 'REJECT'; error: Error }
  | { type: 'RESET' };

function asyncReducer<T>(state: AsyncState<T>, action: AsyncAction<T>): AsyncState<T> {
  switch (action.type) {
    case 'FETCH': return { status: 'pending' };
    case 'RESOLVE': return { status: 'resolved', data: action.data };
    case 'REJECT': return { status: 'rejected', error: action.error };
    case 'RESET': return { status: 'idle' };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}
```

**`useAsync` Hook with Race-Condition Guard:**
```tsx
function useAsync<T>(asyncFn: () => Promise<T>, deps: React.DependencyList) {
  const [state, dispatch] = React.useReducer(asyncReducer<T>, { status: 'idle' });
  const requestIdRef = React.useRef(0);

  React.useEffect(() => {
    const requestId = ++requestIdRef.current;
    dispatch({ type: 'FETCH' });

    asyncFn()
      .then((data) => {
        if (requestId === requestIdRef.current) {
          dispatch({ type: 'RESOLVE', data });
        }
      })
      .catch((error) => {
        if (requestId === requestIdRef.current) {
          dispatch({ type: 'REJECT', error });
        }
      });
  }, deps);

  const reset = React.useCallback(() => dispatch({ type: 'RESET' }), []);

  return { state, reset };
}
```

**Component Breakdown:**
- `asyncReducer`: Pure reducer with exhaustive handling.
- `requestIdRef`: Guards against stale resolutions.
- Only the latest request's `.then`/`.catch` dispatches actions.
- `reset`: Returns the machine to `idle`.

**Usage:**
```tsx
function UserProfile({ userId }: { userId: number }) {
  const { state, reset } = useAsync<User>(
    () => fetch(`/api/users/${userId}`).then((r) => r.json()),
    [userId]
  );

  switch (state.status) {
    case 'idle':
    case 'pending':
      return <p>Loading...</p>;
    case 'rejected':
      return (
        <div role="alert">
          <p>Error: {state.error.message}</p>
          <button onClick={reset}>Retry</button>
        </div>
      );
    case 'resolved':
      return <h1>{state.data.name}</h1>;
  }
}
```

**Component Breakdown:**
- The `switch` is exhaustive; TypeScript narrows `state` in each branch.
- `state.data` is only accessible in the `resolved` branch.
- `state.error` is only accessible in the `rejected` branch.

**Async State Machine with `AbortController`:**
```tsx
function useAsyncFetch<T>(url: string) {
  const [state, dispatch] = React.useReducer(asyncReducer<T>, { status: 'idle' });

  React.useEffect(() => {
    const controller = new AbortController();
    dispatch({ type: 'FETCH' });

    fetch(url, { signal: controller.signal })
      .then(async (res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return (await res.json()) as T;
      })
      .then((data) => dispatch({ type: 'RESOLVE', data }))
      .catch((error: Error) => {
        if (error.name !== 'AbortError') dispatch({ type: 'REJECT', error });
      });

    return () => controller.abort();
  }, [url]);

  return state;
}
```

**Component Breakdown:**
- `AbortController` cancels the request when `url` changes.
- The reducer transitions through `FETCH → RESOLVE | REJECT`.
- `AbortError` is ignored (the cleanup's expected rejection).

**React 19 `useActionState` (Built-in State Machine):**
```tsx
import { useActionState } from 'react';

async function submitAction(prevState: AsyncState<User>, formData: FormData): Promise<AsyncState<User>> {
  try {
    const user = await createUser(Object.fromEntries(formData));
    return { status: 'resolved', data: user };
  } catch (error) {
    return { status: 'rejected', error: error as Error };
  }
}

function Form() {
  const [state, formAction, isPending] = useActionState(submitAction, { status: 'idle' });
  // state: AsyncState<User>, isPending: boolean
}
```

**Component Breakdown:**
- `useActionState` manages the state machine automatically.
- `isPending` mirrors the `pending` status.
- The action returns the next state.

**Syntax Rules:**
- Model the state as a discriminated union, not multiple booleans.
- Use `useReducer` for complex transitions; `useState` for simple ones.
- Guard against race conditions with a request ID or `AbortController`.
- Use `switch (state.status)` for exhaustive handling.
- Include a `default: never` case to catch unhandled states.
- Attach data to the `resolved` variant and error to the `rejected` variant.
- Provide a `reset` action to return to `idle`.
- Ignore `AbortError` in the catch block.

**Constraints and Limitations:**
- The reducer must be pure; no side effects inside it.
- Race conditions can also occur in the `useEffect` dependencies; keep them minimal.
- `useReducer` adds boilerplate; for simple cases, a single state variable may suffice.
- The `never` exhaustiveness check requires `strict: true`.
- Async state machines do not cache or deduplicate requests; use TanStack Query for that.

### Annotated Code Example: Multi-Step Async Machine

```tsx
type CheckoutState =
  | { step: 'cart'; status: 'idle' }
  | { step: 'payment'; status: 'pending' }
  | { step: 'payment'; status: 'rejected'; error: Error }
  | { step: 'success'; status: 'resolved'; orderId: string };

type CheckoutAction =
  | { type: 'SUBMIT_PAYMENT' }
  | { type: 'PAYMENT_RESOLVED'; orderId: string }
  | { type: 'PAYMENT_REJECTED'; error: Error }
  | { type: 'RESET' };

function checkoutReducer(state: CheckoutState, action: CheckoutAction): CheckoutState {
  switch (action.type) {
    case 'SUBMIT_PAYMENT':
      return { step: 'payment', status: 'pending' };
    case 'PAYMENT_RESOLVED':
      return { step: 'success', status: 'resolved', orderId: action.orderId };
    case 'PAYMENT_REJECTED':
      return { step: 'payment', status: 'rejected', error: action.error };
    case 'RESET':
      return { step: 'cart', status: 'idle' };
    default: {
      const _exhaustive: never = action;
      return state;
    }
  }
}

function Checkout() {
  const [state, dispatch] = React.useReducer(checkoutReducer, { step: 'cart', status: 'idle' });

  async function handlePayment() {
    dispatch({ type: 'SUBMIT_PAYMENT' });
    try {
      const { orderId } = await processPayment();
      dispatch({ type: 'PAYMENT_RESOLVED', orderId });
    } catch (error) {
      dispatch({ type: 'PAYMENT_REJECTED', error: error as Error });
    }
  }

  switch (state.step) {
    case 'cart':
      return <button onClick={handlePayment}>Proceed to Payment</button>;
    case 'payment':
      if (state.status === 'pending') return <p>Processing payment...</p>;
      return (
        <div role="alert">
          <p>Payment failed: {state.error.message}</p>
          <button onClick={handlePayment}>Retry</button>
        </div>
      );
    case 'success':
      return <p>Order {state.orderId} confirmed!</p>;
  }
}
```

**Expected Output:** A checkout flow that moves from cart to payment (pending) to either success or rejection with retry. TypeScript narrows the state at each branch, ensuring `orderId` is only accessible in the `success` step.

**Why This Output Occurs:** The reducer handles all transitions. The `switch (state.step)` narrows the union, and the `never` check ensures exhaustiveness. The state machine prevents invalid states (e.g., `step: 'success'` with `status: 'pending'`).

### Real-World Cases

- **Data fetching:** `Idle | Pending | Resolved | Rejected` for any request.
- **Multi-step forms:** `Step1 | Step2 | Step3 | Success | Error`.
- **Authentication:** `Idle | LoggingIn | Authenticated | Error`.
- **Payment processing:** `Cart | PaymentPending | Success | Failed`.
- **File uploads:** `Idle | Uploading | Processing | Complete | Error`.
- **Wizards:** Explicit step transitions with validation gates.

### References

- React Official Documentation – `useReducer` - https://react.dev/reference/react/useReducer
- React Official Documentation – `useActionState` - https://react.dev/reference/react/useActionState
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- David Khourshid – State Machines in React - https://www.youtube.com/watch?v=RqTxtOXcv8Y
- XState – Documentation - https://stately.ai/docs/xstate

---

## Core Concept 5: External-Store Integration (`useSyncExternalStore`)

### Definitions

**Core Definition:** `useSyncExternalStore` is a React Hook that subscribes to an external store and returns its current value, ensuring a consistent snapshot across all components and preventing tearing during concurrent rendering.

**Technical Definition:** `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` is the recommended way to read from external stores (Redux, Zustand, browser APIs, custom stores) in React 18+. It accepts a `subscribe` function (which registers a callback and returns an unsubscribe function), a `getSnapshot` function (which returns the current store value), and an optional `getServerSnapshot` function (for SSR). React calls `getSnapshot` during render and compares the result with `Object.is` to decide whether to re-render. When the store changes, React schedules a re-render. The Hook ensures that all components see a consistent snapshot of the store, even during concurrent rendering—this is called "tearing prevention." Custom Hooks that wrap browser APIs (`matchMedia`, `navigator.onLine`, `ResizeObserver`) should use `useSyncExternalStore` for correctness under concurrent rendering.

**Beginner-Friendly Explanation:** Imagine a big scoreboard at a sports game. Multiple cameras (components) show the score at slightly different times. During concurrent rendering, React might pause one camera to update another, and if the score changes in the meantime, one camera might show an old score while another shows a new one. That is "tearing." `useSyncExternalStore` is the official scoreboard feed that ensures every camera sees the same score at the same time. It is the recommended way to connect React to external state.

### Purposes

- To subscribe to external stores (Redux, Zustand, custom stores) safely in React 18+.
- To prevent tearing during concurrent rendering.
- To read browser APIs (`matchMedia`, `navigator.onLine`, `ResizeObserver`) reactively.
- To provide SSR-safe snapshots via `getServerSnapshot`.
- To encapsulate subscribe/getSnapshot logic in reusable Hooks.
- To integrate third-party state managers with React's rendering model.

### Syntax Rules and Structure

**General Syntax:**
```tsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
```

**Component Breakdown:**
- `subscribe(onStoreChange)`: Registers a callback that React calls when the store changes; returns an unsubscribe function.
- `getSnapshot()`: Returns the current store value; must be stable (same value for the same store state).
- `getServerSnapshot()`: Returns the initial value on the server; must match the client's initial value.

**Custom Store Example:**
```tsx
function createStore<T>(initialState: T) {
  let state = initialState;
  const listeners = new Set<() => void>();

  return {
    getState: () => state,
    setState: (newState: T) => {
      state = newState;
      listeners.forEach((listener) => listener());
    },
    subscribe: (listener: () => void) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const counterStore = createStore({ count: 0 });

function useCounter() {
  return React.useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getState,
    counterStore.getState
  );
}

function Counter() {
  const { count } = useCounter();
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => counterStore.setState({ count: count + 1 })}>+</button>
    </div>
  );
}
```

**Component Breakdown:**
- `createStore`: A minimal external store with `getState`, `setState`, and `subscribe`.
- `useCounter`: Wraps the store with `useSyncExternalStore`.
- The component re-renders when the store's state changes.

**Browser API Example (Online Status):**
```tsx
function subscribe(callback: () => void) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus(): boolean {
  return React.useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
}
```

**Component Breakdown:**
- `subscribe`: Registers the online/offline listeners.
- `getSnapshot`: Returns `navigator.onLine`.
- `getServerSnapshot`: Returns `true` during SSR.

**Media Query Example:**
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
- `subscribe` and `getSnapshot` are stable (via `useCallback`) so the Hook does not resubscribe on every render.
- `getServerSnapshot` returns `false` during SSR.

**Window Size Example (Cached Snapshot):**
```tsx
let cachedSize = { width: 0, height: 0 };

function useWindowSize() {
  const subscribe = React.useCallback((callback: () => void) => {
    window.addEventListener('resize', callback);
    return () => window.removeEventListener('resize', callback);
  }, []);

  const getSnapshot = () => {
    const width = window.innerWidth;
    const height = window.innerHeight;
    if (cachedSize.width !== width || cachedSize.height !== height) {
      cachedSize = { width, height };
    }
    return cachedSize;
  };

  const getServerSnapshot = () => ({ width: 0, height: 0 });

  return React.useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}
```

**Component Breakdown:**
- `cachedSize`: A module-level cache ensures `getSnapshot` returns the same object reference when the size has not changed.
- Returning a new object on every call would cause infinite re-renders.

**Syntax Rules:**
- Use `useSyncExternalStore` for any external store or browser API subscription.
- `subscribe` must return an unsubscribe function.
- `getSnapshot` must return the same value for the same store state (cache objects/arrays).
- Provide `getServerSnapshot` for SSR; it must match the client's initial snapshot.
- `subscribe` and `getSnapshot` should be stable (via `useCallback` or module-level functions).
- Never mutate the store during render; updates happen in event handlers or effects.
- Use `useShallow` (Zustand) or custom selectors for fine-grained subscriptions.

**Constraints and Limitations:**
- `getSnapshot` must return a cached value; returning a new object/array on every call causes an infinite loop.
- The store must notify subscribers synchronously; async notifications may cause tearing.
- `useSyncExternalStore` does not provide selectors; you must implement selective subscription yourself.
- Most state libraries (Redux, Zustand) already use `useSyncExternalStore` internally; you rarely call it directly.
- The `subscribe` function must handle multiple subscribers correctly.
- Browser APIs may not fire events consistently; test with real browsers.

### Annotated Code Example: `useMediaQuery` with `useSyncExternalStore`

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

  const getSnapshot = React.useCallback(
    () => window.matchMedia(query).matches,
    [query]
  );

  const getServerSnapshot = () => false;

  return React.useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

function ResponsiveLayout({ children }: { children: React.ReactNode }) {
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  const prefersDark = useMediaQuery('(prefers-color-scheme: dark)');

  return (
    <div data-theme={prefersDark ? 'dark' : 'light'}>
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

**Expected Output:** The layout renders with a sidebar on desktop and without one on mobile. The theme attribute reflects the user's colour-scheme preference. Both update automatically when the conditions change.

**Why This Output Occurs:** `useSyncExternalStore` subscribes to `matchMedia` and re-renders when the media query's match status changes. The `getServerSnapshot` returns `false` during SSR, avoiding hydration mismatches. The Hook prevents tearing under concurrent rendering.

### Real-World Cases

- **State libraries:** Redux, Zustand, Jotai, and Valtio use `useSyncExternalStore` internally.
- **Browser APIs:** Online status, media queries, window dimensions, `prefers-color-scheme`.
- **Custom stores:** Application-specific stores that need to be shared across components.
- **WebSocket state:** Connection status and last message timestamp.
- **Third-party libraries:** Any external library with a subscription API.
- **URL state:** `history` and `location` subscriptions.

### References

- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – Adding a Store - https://react.dev/reference/react/useSyncExternalStore#adding-support-for-a-store
- React Official Documentation – Tearing - https://react.dev/reference/react/useSyncExternalStore#my-subscribe-function-gets-called-after-every-re-render
- Redux – `useSyncExternalStore` Integration - https://redux.js.org/usage/implementing-undo-history
- Zustand – `useSyncExternalStore` - https://zustand.docs.pmnd.rs/
- MDN Web Docs – `window.matchMedia()` - https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia
- MDN Web Docs – `navigator.onLine` - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine

---

## Core Concept 6: SSR & Hydration Safety

### Definitions

**Core Definition:** SSR and hydration safety is the practice of ensuring that custom Hooks do not access browser-only APIs during server rendering, and that the server-rendered HTML matches the client's initial render to avoid hydration mismatches.

**Technical Definition:** During server rendering, `window`, `document`, `localStorage`, `matchMedia`, and other browser APIs are unavailable. A Hook that reads these APIs during render or in a `useState` initialiser will throw on the server (or return a different value than on the client), causing a hydration mismatch. The solutions are: (1) **Guard** with `typeof window !== 'undefined'` before accessing browser APIs; (2) **Defer** to `useEffect`, which runs only on the client; (3) **Provide a server snapshot** via `getServerSnapshot` for `useSyncExternalStore`; (4) **Lazy initialise** state with a function that checks the environment; (5) **Two-pass rendering** for values that must differ between server and client (e.g., a theme toggler that reads `localStorage`). Frameworks like Next.js and Remix provide `useIsomorphicLayoutEffect` (which uses `useEffect` on the server, `useLayoutEffect` on the client) and `next/dynamic` with `{ ssr: false }` for client-only components. The goal is a stable first render that matches the server-rendered HTML, followed by client-side enhancement.

**Beginner-Friendly Explanation:** When your app is rendered on the server, there is no browser—no `window`, no `localStorage`, no `matchMedia`. If a Hook tries to read those during the server render, the app crashes. Even if it does not crash, the server might render one thing (e.g., "light theme") and the client another (e.g., "dark theme"), causing a hydration mismatch where React throws away the server HTML and re-renders. SSR-safe Hooks guard against these problems by checking for the browser, deferring to `useEffect`, or providing a server snapshot that matches the client's initial value.

### Purposes

- To prevent server-render crashes from browser-only APIs.
- To prevent hydration mismatches between server HTML and client render.
- To provide consistent initial values across server and client.
- To defer browser-only logic to the client.
- To use `useSyncExternalStore`'s `getServerSnapshot` for external stores.
- To guard `useLayoutEffect` (which warns on the server).

### Syntax Rules and Structure

**Guarding Browser APIs:**
```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [stored, setStored] = React.useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = window.localStorage.getItem(key);
      return item ? (JSON.parse(item) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = React.useCallback((value: T) => {
    setStored(value);
    if (typeof window !== 'undefined') {
      window.localStorage.setItem(key, JSON.stringify(value));
    }
  }, [key]);

  return [stored, setValue] as const;
}
```

**Component Breakdown:**
- `typeof window === 'undefined'`: Guards the server.
- The lazy initialiser returns `initialValue` on the server.
- `setValue` guards the write.

**Deferring to `useEffect`:**
```tsx
function useWindowWidth() {
  const [width, setWidth] = React.useState(0); // Server-safe default

  React.useEffect(() => {
    setWidth(window.innerWidth);
    function handleResize() { setWidth(window.innerWidth); }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return width;
}
```

**Component Breakdown:**
- Initial state is `0`, which is safe on the server.
- The effect runs only on the client, setting the real width.
- The server renders `0`; the client hydrates with `0` and then updates.

**`useSyncExternalStore` with `getServerSnapshot`:**
```tsx
function useOnlineStatus(): boolean {
  return React.useSyncExternalStore(
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    () => navigator.onLine,
    () => true // Server snapshot
  );
}
```

**Component Breakdown:**
- `getServerSnapshot` returns `true` on the server.
- The server renders as if online; the client updates if offline.
- This avoids hydration mismatches because both server and client start with `true`.

**`useIsomorphicLayoutEffect` (Framework Utility):**
```tsx
const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? React.useLayoutEffect : React.useEffect;
```

**Component Breakdown:**
- On the server, uses `useEffect` (which does nothing).
- On the client, uses `useLayoutEffect` (which runs synchronously before paint).
- Prevents the "useLayoutEffect does nothing on the server" warning.

**Two-Pass Rendering (Theme Toggle):**
```tsx
function useTheme() {
  const [theme, setTheme] = React.useState<'light' | 'dark'>('light');
  const [mounted, setMounted] = React.useState(false);

  React.useEffect(() => {
    setMounted(true);
    const stored = localStorage.getItem('theme');
    if (stored === 'dark' || stored === 'light') setTheme(stored);
  }, []);

  return { theme, setTheme, mounted };
}

function ThemeToggle() {
  const { theme, setTheme, mounted } = useTheme();

  // On the server and first client render, render a neutral placeholder
  if (!mounted) return <div style={{ width: 40, height: 40 }} aria-hidden="true" />;

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}
```

**Component Breakdown:**
- `mounted` is `false` on the server and the first client render.
- The placeholder matches on both sides, avoiding hydration mismatch.
- After mount, the real theme is loaded from `localStorage`.

**Next.js Dynamic Import with `ssr: false`:**
```tsx
import dynamic from 'next/dynamic';

const ClientOnlyChart = dynamic(() => import('./Chart'), {
  ssr: false,
  loading: () => <p>Loading chart...</p>,
});

function Page() {
  return <ClientOnlyChart data={data} />;
}
```

**Component Breakdown:**
- `ssr: false`: The component is not rendered on the server.
- `loading`: The fallback shown while the component loads.

**Syntax Rules:**
- Guard browser APIs with `typeof window !== 'undefined'`.
- Defer browser-only logic to `useEffect`.
- Use `getServerSnapshot` for `useSyncExternalStore`.
- Use `useIsomorphicLayoutEffect` instead of `useLayoutEffect` in SSR.
- For values that differ between server and client (theme, time), use two-pass rendering with a `mounted` flag.
- In Next.js, use `next/dynamic` with `{ ssr: false }` for client-only components.
- Avoid `Date.now()`, `Math.random()`, and locale-dependent formatting in render; defer to effects.
- Ensure the server-rendered HTML matches the client's initial render.

**Constraints and Limitations:**
- Two-pass rendering causes a brief flash for the client-only content; use a placeholder that matches the final layout.
- `typeof window` checks are not enough for values that must be read on the client; defer to effects.
- `useIsomorphicLayoutEffect` runs `useEffect` on the server, which does nothing; some layout-dependent logic may not work.
- Next.js's `ssr: false` removes server rendering for the component, hurting SEO and initial load.
- Hydration mismatches can be caused by any difference between server and client render, not just browser APIs (e.g., time, random values, locale).
- React 18's `useId` generates SSR-safe IDs; use it instead of random IDs.

### Annotated Code Example: SSR-Safe `useLocalStorage` with Two-Pass Rendering

```tsx
function useSSRLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void, boolean] {
  const [stored, setStored] = React.useState<T>(initialValue);
  const [isHydrated, setIsHydrated] = React.useState(false);

  React.useEffect(() => {
    try {
      const item = window.localStorage.getItem(key);
      if (item) setStored(JSON.parse(item) as T);
    } catch (error) {
      console.warn(`Error reading localStorage key "${key}":`, error);
    } finally {
      setIsHydrated(true);
    }
  }, [key]);

  const setValue = React.useCallback(
    (value: T) => {
      setStored(value);
      try {
        window.localStorage.setItem(key, JSON.stringify(value));
      } catch (error) {
        console.warn(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key]
  );

  return [stored, setValue, isHydrated];
}

function ThemeToggle() {
  const [theme, setTheme, isHydrated] = useSSRLocalStorage<'light' | 'dark'>('theme', 'light');

  if (!isHydrated) {
    // Server and first client render match
    return <div style={{ width: 120, height: 32 }} aria-hidden="true" />;
  }

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Switch to {theme === 'light' ? 'dark' : 'light'}
    </button>
  );
}
```

**Expected Output:** On the server and first client render, a placeholder is shown. After hydration, the stored theme is loaded, and the button appears with the correct label. No hydration mismatch occurs.

**Why This Output Occurs:** The initial state is `initialValue`, which matches on server and client. The `isHydrated` flag defers the real value to the client. The placeholder occupies the same space as the final button, preventing layout shift.

### Real-World Cases

- **Theme toggles:** Reading `localStorage` after mount.
- **Time displays:** Deferring `Date.now()` to an effect.
- **Random values:** Deferring `Math.random()` to an effect or using `useId`.
- **Client-only widgets:** Charts, maps, and editors with `next/dynamic` + `ssr: false`.
- **Media queries:** Using `useSyncExternalStore` with a server snapshot.
- **Online status:** Using `useSyncExternalStore` with a server snapshot.

### References

- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – Hydration - https://react.dev/reference/react-dom/client/hydrateRoot
- React Official Documentation – `useLayoutEffect` - https://react.dev/reference/react/useLayoutEffect
- Next.js – Dynamic Imports - https://nextjs.org/docs/app/guides/lazy-loading
- Next.js – SSR and Hydration - https://nextjs.org/docs/app/building-your-application/rendering
- Remix – Hydration - https://remix.run/docs/en/main/discussion/hydration
- Josh Comeau – The Perils of Rehydration - https://www.joshwcomeau.com/react/the-perils-of-rehydration/

---

## Comparison and Decision Guidance

| Concept | When to Use | When to Avoid | Key Risk |
|---|---|---|---|
| **Generic Hooks** | Reusable utilities for any type | Single-type Hooks | Over-constrained generics |
| **Composable Hooks** | Building complex logic from small Hooks | Simple, single-purpose Hooks | Over-composition |
| **Resource Cleanup** | Every effect that creates a resource | Pure computations | Memory leaks |
| **Async State Machines** | Complex async flows with multiple states | Simple one-off fetches | Boilerplate |
| **`useSyncExternalStore`** | External stores, browser APIs | Local component state | Infinite loops (uncached snapshots) |
| **SSR Safety** | Any Hook using browser APIs | Pure logic Hooks | Hydration mismatches |

**Decision Guidance:**
- **Use generics** for any Hook that works with multiple data types.
- **Compose** small Hooks into facades; keep each Hook focused on one concern.
- **Clean up every resource**; use `AbortController` for fetch, `clearInterval` for timers, `disconnect()` for observers, `close()` for sockets.
- **Model async logic as a state machine** when there are more than two states or when race conditions are possible.
- **Use `useSyncExternalStore`** for any external store or browser API subscription.
- **Guard browser APIs** with `typeof window` and defer to `useEffect`.
- **Provide `getServerSnapshot`** for `useSyncExternalStore`.
- **Use two-pass rendering** for values that must differ between server and client.
- **Test in SSR** (Next.js, Remix) to catch hydration mismatches.
- **Adopt a library** (TanStack Query, SWR, React Hook Form, `usehooks-ts`) rather than reimplementing these patterns for production.

---

## References

- React Official Documentation – Reusing Logic with Custom Hooks - https://react.dev/learn/reusing-logic-with-custom-hooks
- React Official Documentation – Rules of Hooks - https://react.dev/reference/rules/rules-of-hooks
- React Official Documentation – Synchronizing with Effects - https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – `useSyncExternalStore` - https://react.dev/reference/react/useSyncExternalStore
- React Official Documentation – `useReducer` - https://react.dev/reference/react/useReducer
- React Official Documentation – `useActionState` - https://react.dev/reference/react/useActionState
- React Official Documentation – Hydration - https://react.dev/reference/react-dom/client/hydrateRoot
- React Official Documentation – `useLayoutEffect` - https://react.dev/reference/react/useLayoutEffect
- TypeScript Handbook – Generics - https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook – Discriminated Unions - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook – Exhaustiveness Checking - https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- MDN Web Docs – `AbortController` - https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – `ResizeObserver` - https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver
- MDN Web Docs – `IntersectionObserver` - https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver
- MDN Web Docs – `WebSocket` - https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN Web Docs – `window.matchMedia()` - https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia
- MDN Web Docs – `navigator.onLine` - https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine
- Next.js – Dynamic Imports - https://nextjs.org/docs/app/guides/lazy-loading
- Next.js – SSR and Hydration - https://nextjs.org/docs/app/building-your-application/rendering
- Remix – Hydration - https://remix.run/docs/en/main/discussion/hydration
- TanStack Query – Overview - https://tanstack.com/query/latest/docs/framework/react/overview
- SWR – Documentation - https://swr.vercel.app/
- React Hook Form – Documentation - https://react-hook-form.com/
- usehooks-ts - https://usehooks-ts.com/
- XState – Documentation - https://stately.ai/docs/xstate
- Josh Comeau – The Perils of Rehydration - https://www.joshwcomeau.com/react/the-perils-of-rehydration/