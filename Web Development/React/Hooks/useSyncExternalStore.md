# `useSyncExternalStore` Hook — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** `useSyncExternalStore` is a React Hook that lets a component subscribe to an external data store and read its value in a way that is safe for concurrent rendering.

**Technical Definition:** `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` is a React Hook that subscribes a component to an external store. It returns the current snapshot of the store. The `subscribe` function registers a callback that React invokes when the store changes, and returns an unsubscribe function. The `getSnapshot` function returns the current value from the store; React compares successive snapshots with `Object.is` to decide whether a re-render is needed. The optional `getServerSnapshot` provides the initial snapshot during server-side rendering and hydration. The Hook guarantees that every component in a single render pass sees the same, consistent snapshot of the store, preventing "tearing" during concurrent rendering.

**Beginner-Friendly Explanation:** `useSyncExternalStore` is a tool for connecting React to data that lives outside of React — like a third-party state library, a browser API, or a custom event system. Normally, React manages its own state with `useState` or `useReducer`. But sometimes your data lives somewhere else, and React needs a safe way to read it. This Hook does exactly that: it subscribes to the external source, reads its current value, and tells React to re-render when the value changes. The word "Sync" is the key: it ensures every component in a single render sees the same version of the data, even while React is multitasking (concurrent rendering). Without it, different parts of the screen could show different, conflicting values — a problem called "tearing".

### Key Characteristics

- **External store subscription:** Connects React components to data sources that live outside React's state system.
- **Tearing-safe:** Guarantees a consistent snapshot across all components in a single render pass, even during concurrent updates.
- **Snapshot-based:** `getSnapshot` must return an immutable snapshot; React compares snapshots with `Object.is`.
- **Stable function requirements:** `subscribe` and `getSnapshot` should be stable (defined outside the component) to avoid unnecessary re-subscriptions.
- **SSR support:** The optional `getServerSnapshot` enables server rendering and hydration without mismatches.
- **Concurrent-aware:** If the store mutates during a non-blocking Transition, React falls back to a blocking update to preserve consistency.

### Prerequisites

- React function components and Hooks.
- Understanding of `useState`, `useEffect`, and the render cycle.
- Familiarity with external state sources (browser APIs, third-party stores).
- Basic knowledge of concurrent rendering and the concept of "tearing."

### Related Programming Areas

- **State Management Libraries** — Redux, Zustand, and Jotai use `useSyncExternalStore` internally for their React bindings.
- **Browser APIs** — `navigator.onLine`, `document.visibilityState`, `window.matchMedia`.
- **Concurrent Rendering** — the scheduling model that makes tearing possible without this Hook.
- **Server-Side Rendering** — hydration consistency with `getServerSnapshot`.

---

## 1. `useSyncExternalStore`

### Definitions

**Core Definition:** `useSyncExternalStore` is a React Hook that subscribes to an external store and returns its current snapshot.

**Technical Definition:** `const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` is called at the top level of a component. It takes three arguments: `subscribe`, a function that registers a callback and returns an unsubscribe function; `getSnapshot`, a pure function that returns the current immutable snapshot of the store; and an optional `getServerSnapshot` for SSR. The Hook returns the current snapshot. React re-renders the component when the store changes and the new snapshot differs (by `Object.is`) from the previous one.

**Beginner-Friendly Explanation:** You call this Hook at the top of your component. You give it a way to listen for changes (the `subscribe` function) and a way to read the current value (the `getSnapshot` function). It gives you back the current value. When the value changes, React updates your component.

### Purposes

- To subscribe a component to an external data store safely.
- To read the current value of an external store during render.
- To ensure all components in a render pass see the same snapshot (tearing prevention).
- To integrate third-party state libraries, browser APIs, or custom event systems with React.
- To support SSR and hydration without snapshot mismatches.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { useSyncExternalStore } from 'react';

const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?);
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `subscribe` | Function `(callback) => unsubscribe`. Registers the callback and returns a cleanup function. |
| `getSnapshot` | Function `() => snapshot`. Returns the current immutable snapshot of the store. |
| `getServerSnapshot?` | Optional function `() => snapshot` used during SSR and hydration. |
| `snapshot` | The current snapshot of the store. |

**Syntax Rules:**

1. Call `useSyncExternalStore` at the top level of a component or custom Hook; never inside loops or conditions.
2. The `subscribe` function must return an unsubscribe function.
3. `getSnapshot` must be pure and synchronous; it must return the same value if the store has not changed.
4. The snapshot must be immutable; if the store mutates, return a new snapshot object.
5. If `getServerSnapshot` is omitted, server rendering throws an error.

**Constraints and Limitations:**

- `getSnapshot` must return a cached, stable reference for object snapshots; returning a new object on every call causes infinite re-renders.
- `subscribe` and `getSnapshot` should be declared outside the component to avoid re-subscriptions on every render.
- It is not recommended to suspend a render based on a store value from `useSyncExternalStore`; store mutations cannot be marked as non-blocking transitions, so they trigger the nearest `Suspense` fallback.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Subscribing to a Custom Todo Store

```jsx
// todoStore.js — an external store (outside React)
let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }];
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  },
};

function emitChange() {
  for (let listener of listeners) listener();
}
```

```jsx
// TodosApp.jsx — React component using the store
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Expected Output:** A list of todos that updates automatically when `todosStore.addTodo()` is called.

**Why this output:** `useSyncExternalStore` subscribes to `todosStore`. When `addTodo` mutates the store and calls `emitChange`, React re-reads `getSnapshot` and re-renders with the new snapshot.

#### Example 2: Subscribing to a Browser API (Online Status)

```jsx
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // Default on the server
}

function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

**Expected Output:** The status bar shows "Online" or "Disconnected" and updates when the browser's connectivity changes.

**Why this output:** `subscribe` registers `online` and `offline` listeners. `getSnapshot` returns `navigator.onLine`. `useSyncExternalStore` re-renders when the browser fires the events.

### Real-World Cases

- **Third-party state libraries:** Redux, Zustand, and Jotai use `useSyncExternalStore` internally for their React bindings.
- **Browser APIs:** Online status, page visibility, media queries, geolocation.
- **Custom event systems:** Any mutable data source that React does not manage.

### References

- React — useSyncExternalStore — https://react.dev/reference/react/useSyncExternalStore
- React — useSyncExternalStore (React 18) — https://18.react.dev/reference/react/useSyncExternalStore
- Epic React — useSyncExternalStore Demystified — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- Telerik — The React useSyncExternalStore Hook — https://www.telerik.com/blogs/react-usesyncexternalstore-hook

---

## 2. External Stores

### Definitions

**Core Definition:** An external store is any source of mutable data that lives outside React's state management system and can be subscribed to for changes.

**Technical Definition:** An external store is a JavaScript object or API that holds state and provides a way to subscribe to changes. Unlike React state (managed by `useState` or `useReducer`), the external store owns its data and notifies subscribers when the data changes. Examples include third-party state libraries, browser APIs (`navigator.onLine`, `document.visibilityState`), global variables, and custom event emitters. `useSyncExternalStore` bridges the gap between React and these external sources.

**Beginner-Friendly Explanation:** An external store is data that lives outside React's world — like a global variable, a browser API, or a state library. React does not control it, but your components need to display it. `useSyncExternalStore` is the bridge that lets React safely read from that outside source.

### Purposes

- To connect React components to data sources that React does not manage.
- To integrate third-party state libraries with React's rendering model.
- To subscribe to browser APIs and global event systems.
- To ensure consistent rendering when the external data changes.

### Syntax Rules and Structure

**Anatomy of an External Store:**

```js
const store = {
  subscribe(callback) {
    // Register callback; return unsubscribe function
  },
  getSnapshot() {
    // Return current immutable snapshot
  },
};
```

**Syntax Rules:**

1. The store must provide a `subscribe` method that accepts a callback and returns an unsubscribe function.
2. The store must provide a `getSnapshot` method that returns an immutable snapshot.
3. The snapshot must be a stable reference when the store has not changed.
4. The store's data must be immutable from React's perspective; return a new snapshot when the data changes.

### Annotated Code Examples

```jsx
// A simple external store for counter state
let count = 0;
let listeners = [];

const counterStore = {
  increment() {
    count++;
    listeners.forEach(l => l());
  },
  subscribe(listener) {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
  getSnapshot() {
    return count; // Primitive snapshot — stable by value
  },
};
```

```jsx
function Counter() {
  const count = useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getSnapshot
  );
  return <button onClick={() => counterStore.increment()}>{count}</button>;
}
```

**Expected Output:** A button that displays the current count and increments when clicked.

**Why this output:** `useSyncExternalStore` subscribes to `counterStore`. When `increment` changes `count` and calls the listeners, React re-reads `getSnapshot` and re-renders with the new value.

### Real-World Cases

- **Redux store:** `useSyncExternalStore` is used internally by React-Redux.
- **Zustand store:** Zustand uses `useSyncExternalStore` for its `useStore` hook.
- **Browser API wrappers:** Custom stores for `navigator.onLine`, `matchMedia`, etc.

### References

- React — useSyncExternalStore: Subscribing to an external store — https://react.dev/reference/react/useSyncExternalStore
- Epic React — External stores and tearing — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- Mintlify — useSyncExternalStore: Third-party state libraries — https://mintlify.wiki/facebook/react/api/hooks/use-sync-external-store

---

## 3. Subscribe Function

### Definitions

**Core Definition:** The `subscribe` function is the first argument to `useSyncExternalStore`; it registers a callback that React calls when the external store changes.

**Technical Definition:** `subscribe(callback)` is a function that takes a single `callback` argument and subscribes it to the store. When the store changes, it must invoke the provided `callback`, which causes React to re-call `getSnapshot` and, if the snapshot differs, re-render the component. The `subscribe` function must return a function that cleans up the subscription (unsubscribes). React calls the cleanup function when the component unmounts or when the `subscribe` function itself changes between renders.

**Beginner-Friendly Explanation:** `subscribe` is how you tell React to listen for changes. You give it a callback, and whenever the store changes, you call that callback. You also return a way to stop listening (the unsubscribe function).

### Purposes

- To register a component's interest in store changes.
- To notify React when the store has changed so it can re-render.
- To provide a cleanup mechanism for unsubscription.
- To prevent memory leaks from lingering subscriptions.

### Syntax Rules and Structure

```jsx
function subscribe(callback) {
  // Register callback with the store
  store.on('change', callback);
  // Return unsubscribe function
  return () => {
    store.off('change', callback);
  };
}
```

**Syntax Rules:**

1. `subscribe` must accept a single `callback` argument.
2. It must call `callback` when the store changes.
3. It must return a function that unsubscribes.
4. `subscribe` should be stable (declared outside the component) to avoid re-subscribing on every render.
5. If `subscribe` changes between renders, React re-subscribes with the new function.

### Annotated Code Examples

```jsx
// ✅ Stable subscribe function outside the component
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  return useSyncExternalStore(subscribe, () => navigator.onLine);
}
```

**Expected Output:** The component subscribes to online/offline events once and unsubscribes on unmount.

**Why this output:** `subscribe` is defined outside the component, so its reference is stable. React subscribes once and does not re-subscribe unnecessarily.

### Real-World Cases

- **Event emitter stores:** Subscribing to `addEventListener` and returning `removeEventListener`.
- **WebSocket stores:** Subscribing to message events.
- **Custom stores:** Adding and removing listeners from an internal listener array.

### References

- React — useSyncExternalStore: subscribe parameter — https://react.dev/reference/react/useSyncExternalStore
- Mintlify — useSyncExternalStore: subscribe — https://mintlify.wiki/facebook/react/api/hooks/use-sync-external-store

---

## 4. Get Snapshot Function

### Definitions

**Core Definition:** The `getSnapshot` function is the second argument to `useSyncExternalStore`; it returns the current immutable snapshot of the store's data.

**Technical Definition:** `getSnapshot()` is a pure, synchronous function that returns the current value of the store. While the store has not changed, repeated calls to `getSnapshot` must return the same value. React compares successive snapshots with `Object.is` to determine whether a re-render is needed. If the store's data is mutable, `getSnapshot` must return a new immutable snapshot only when the data has changed; otherwise, it should return a cached last snapshot.

**Beginner-Friendly Explanation:** `getSnapshot` is how you read the current value from the store. It should return the same value every time you call it, as long as nothing has changed. React uses this to decide whether to re-render.

### Purposes

- To provide the current value of the store to the component.
- To enable React's `Object.is` comparison for re-render decisions.
- To ensure the snapshot is immutable and consistent.
- To support tearing-free rendering with a stable reference.

### Syntax Rules and Structure

```jsx
function getSnapshot() {
  return store.getState(); // Must return an immutable snapshot
}
```

**Syntax Rules:**

1. `getSnapshot` must be pure and synchronous.
2. It must return the same value if the store has not changed.
3. For object snapshots, it must return a cached reference, not a new object on every call.
4. If the store changes, return a new snapshot.
5. React compares snapshots with `Object.is`.

**Constraints and Limitations:**

- Returning a new object on every call causes infinite re-renders ("The result of getSnapshot should be cached").
- `getSnapshot` must not have side effects.

### Annotated Code Examples

```jsx
// ✅ Correct: cached snapshot for object data
let cachedSnapshot = null;

const store = {
  getSnapshot() {
    if (cachedSnapshot === null) {
      cachedSnapshot = { ...internalState }; // Cache the snapshot
    }
    return cachedSnapshot;
  },
  // Invalidate cache when state changes
  update(newState) {
    internalState = newState;
    cachedSnapshot = null;
    emitChange();
  },
};
```

**Expected Output:** The component receives the same object reference until the store actually changes.

**Why this output:** Caching the snapshot ensures `Object.is` returns `true` for unchanged data, preventing infinite re-renders.

### Real-World Cases

- **Redux selectors:** `useSyncExternalStore` with a selector that returns a stable reference.
- **Browser API wrappers:** Returning primitives (`navigator.onLine`) or cached objects (`matchMedia` results).
- **Custom stores:** Maintaining a `cachedSnapshot` variable.

### References

- React — useSyncExternalStore: getSnapshot parameter — https://react.dev/reference/react/useSyncExternalStore
- React — useSyncExternalStore: getSnapshot must be cached — https://18.react.dev/reference/react/useSyncExternalStore
- Epic React — getSnapshot stability — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0

---

## 5. Server Snapshot

### Definitions

**Core Definition:** The `getServerSnapshot` function is the optional third argument to `useSyncExternalStore`; it provides the initial snapshot used during server-side rendering and hydration.

**Technical Definition:** `getServerSnapshot()` returns the initial snapshot of the store. It is used only during server rendering and during hydration of server-rendered content on the client. The server snapshot must be the same on both the server and the client, and is usually serialized and passed from the server to the client. If `getServerSnapshot` is omitted, rendering the component on the server will throw an error.

**Beginner-Friendly Explanation:** `getServerSnapshot` is the value React uses when it renders your component on the server. It ensures the server and the browser start with the same data, so hydration works without errors.

### Purposes

- To provide a consistent initial snapshot for SSR.
- To prevent hydration mismatches between server and client.
- To support server-rendered components that read from external stores.
- To allow safe hydration when the store is client-only.

### Syntax Rules and Structure

```jsx
const snapshot = useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot
);
```

**Syntax Rules:**

1. `getServerSnapshot` is optional for client-only components but required for SSR.
2. It must return the same value on the server and the client.
3. The returned value is usually serialized and passed from server to client.
4. If omitted during SSR, React throws an error.
5. For client-only stores (e.g., `navigator.onLine`), return a sensible default (e.g., `true`).

### Annotated Code Examples

```jsx
function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true // Server snapshot: assume online
  );
}
```

**Expected Output:** During SSR, the component renders as "Online" by default. On the client, it switches to the actual `navigator.onLine` value after hydration.

**Why this output:** `getServerSnapshot` returns `true` on the server, preventing a hydration mismatch when the client's initial render also assumes `true` before the actual value is read.

### Real-World Cases

- **SSR frameworks (Next.js, Remix):** Components that read from external stores during server rendering.
- **Client-only browser APIs:** Providing a default server snapshot for `navigator.onLine`, `matchMedia`, etc.
- **Hydration consistency:** Ensuring server and client start with the same data.

### References

- React — useSyncExternalStore: getServerSnapshot — https://react.dev/reference/react/useSyncExternalStore
- Mintlify — useSyncExternalStore: Server snapshot — https://mintlify.wiki/facebook/react/api/hooks/use-sync-external-store
- GitHub — useSyncExternalStore getServerSnapshot example — https://github.com/saschb2b/okf-bundles

---

## 6. Concurrent Rendering

### Definitions

**Core Definition:** Concurrent rendering is React's interruptible rendering model that allows the framework to prepare multiple UI versions simultaneously and prioritise urgent updates.

**Technical Definition:** Concurrent rendering is a behind-the-scenes mechanism that enables React to prepare multiple versions of the UI at the same time. Rendering is interruptible: React may start rendering an update, pause in the middle, abandon it, or continue later. This model improves perceived performance but introduces the risk of "tearing" when components read from an external store that changes mid-render. `useSyncExternalStore` is specifically designed to be safe in this environment.

**Beginner-Friendly Explanation:** Concurrent rendering is React's ability to multitask. It makes apps feel faster, but it also means React might pause in the middle of rendering. If your data changes during that pause, different parts of the screen could show different versions of the data. `useSyncExternalStore` prevents this by ensuring every component in a single render sees the same snapshot.

### Purposes

- To understand why `useSyncExternalStore` exists.
- To recognise when concurrent rendering could cause tearing with external stores.
- To ensure components remain consistent during interruptible renders.

### Syntax Rules and Structure

**Concurrent Rendering and Store Mutations:**

| Scenario | React Behaviour |
|----------|-----------------|
| Store mutates during a Transition update | React falls back to a blocking update to preserve consistency. |
| Store value is read during render | `getSnapshot` is called synchronously during render. |
| Store value changes between render and commit | React detects the mismatch and re-renders with the latest snapshot. |

**Syntax Rules:**

1. `useSyncExternalStore` guarantees tearing-free rendering under concurrent rendering.
2. If the store mutates during a Transition, React forces a blocking update to prevent inconsistency.
3. The `subscribe` and `getSnapshot` pair should be stable for best results.

### Annotated Code Examples

```jsx
// A store that could be mutated during concurrent rendering
let state = { value: 0 };
let listeners = [];

const store = {
  increment() {
    state = { value: state.value + 1 };
    listeners.forEach(l => l());
  },
  subscribe(l) { listeners.push(l); return () => { listeners = listeners.filter(x => x !== l); }; },
  getSnapshot() { return state; },
};

function Display() {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <p>{snapshot.value}</p>;
}
```

**Expected Output:** All `Display` components show the same value, even if the store is mutated while React is rendering.

**Why this output:** `useSyncExternalStore` synchronises the snapshot across components and forces consistency when the store mutates during a concurrent update.

### Real-World Cases

- **Third-party state libraries:** Ensuring consistency with concurrent features.
- **Real-time data feeds:** WebSocket-driven stores that update frequently.
- **Browser APIs:** Online status, visibility state.

### References

- Epic React — useSyncExternalStore and tearing — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- React — useSyncExternalStore: Caveats (concurrent rendering) — https://react.dev/reference/react/useSyncExternalStore
- GitHub — Tearing-safe subscriptions — https://github.com/pproenca/dot-skills

---

## 7. Tearing Prevention

### Definitions

**Core Definition:** Tearing is a UI inconsistency where different components in the same render pass display different versions of the same external data.

**Technical Definition:** Tearing occurs during concurrent rendering when an external store changes while React is in the middle of rendering a component tree. Some components read the old value, others read the new value, and the UI shows conflicting information. `useSyncExternalStore` prevents tearing by providing a React-managed, synchronous way to subscribe to an external store. It guarantees that all components in a render pass see the same, consistent snapshot of the data, even during concurrent updates.

**Beginner-Friendly Explanation:** Tearing is when your UI literally "tears" — one part shows old data, another shows new data, and they do not match. `useSyncExternalStore` prevents this by making sure React freezes the data for an entire render, so every component sees the same version.

### Purposes

- To ensure UI consistency during concurrent rendering.
- To prevent conflicting data displays in different components.
- To provide a single source of truth for external store data.

### Syntax Rules and Structure

**How Tearing Prevention Works:**

| Mechanism | Description |
|-----------|-------------|
| Synchronous `getSnapshot` | Called during render to ensure a consistent value |
| `Object.is` comparison | React compares snapshots to decide re-renders |
| Blocking fallback | If store mutates during a Transition, React forces a blocking update |

**Syntax Rules:**

1. Use `useSyncExternalStore` for any external data read during render.
2. Do not use `useState` + `useEffect` for external store subscriptions; it is not tearing-safe.
3. Keep `getSnapshot` fast and synchronous.

### Annotated Code Examples

```jsx
// ❌ Not tearing-safe: useState + useEffect
function BadComponent() {
  const [value, setValue] = useState(store.getSnapshot());
  useEffect(() => {
    return store.subscribe(() => setValue(store.getSnapshot()));
  }, []);
  return <p>{value}</p>;
}

// ✅ Tearing-safe: useSyncExternalStore
function GoodComponent() {
  const value = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <p>{value}</p>;
}
```

**Expected Output:** The good component never shows conflicting values; the bad component may tear during concurrent rendering.

**Why this output:** `useSyncExternalStore` coordinates the snapshot read with React's render cycle, while `useState` + `useEffect` reads the store outside React's render coordination.

### Real-World Cases

- **Multi-component dashboards:** All widgets reading from the same store.
- **Real-time collaboration:** Multiple components displaying shared state.
- **Global state libraries:** Ensuring consistent selectors across the tree.

### References

- Epic React — Tearing prevention — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- React — useSyncExternalStore: Tearing-safe snapshots — https://react.dev/reference/react/useSyncExternalStore
- GitHub — useSyncExternalStore: tear-free snapshot contract — https://github.com/pproenca/dot-skills

---

## 8. `useSyncExternalStore` Use Cases

### Definitions

**Core Definition:** Use cases for `useSyncExternalStore` are scenarios where a React component needs to read data from a source outside React's state system in a concurrent-safe way.

**Technical Definition:** `useSyncExternalStore` is appropriate when: (1) integrating a third-party state management library that holds state outside React; (2) subscribing to browser APIs that expose mutable values and events; (3) reading from any mutable data source that React does not manage; or (4) building a custom store with a `subscribe`/`getSnapshot` contract. It is not a replacement for `useState` or `useReducer` for state that React owns.

**Beginner-Friendly Explanation:** Use this Hook when your data lives outside React — like in a state library, a browser API, or a global event system. Do not use it for normal React state; use `useState` or `useReducer` for that.

### Use Cases Table

| Use case | Why `useSyncExternalStore` |
|----------|---------------------------|
| **Third-party state libraries** | Redux, Zustand, and Jotai use it internally for React bindings. |
| **Browser APIs** | `navigator.onLine`, `document.visibilityState`, `matchMedia`. |
| **Custom stores** | Any mutable source with subscribe/getSnapshot semantics. |
| **Real-time data** | WebSocket or EventSource-driven stores. |
| **Global event systems** | Custom event emitters or observable patterns. |

### Annotated Code Examples

```jsx
// Browser API: online status
function useOnlineStatus() {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    () => navigator.onLine,
    () => true
  );
}
```

**Expected Output:** A boolean indicating whether the browser is online, with automatic updates on connectivity changes.

**Why this output:** `useSyncExternalStore` subscribes to the browser's online/offline events and reads `navigator.onLine` as the snapshot.

### Real-World Cases

- **Redux/Zustand bindings:** Internal use by state libraries.
- **Online/offline indicators:** Status bars and connectivity banners.
- **Theme/media queries:** Reacting to `matchMedia` changes.
- **Visibility tracking:** Pausing work when the tab is hidden.

### References

- React — useSyncExternalStore: Subscribing to a browser API — https://react.dev/reference/react/useSyncExternalStore
- Mintlify — useSyncExternalStore: Browser API usage — https://mintlify.wiki/facebook/react/api/hooks/use-sync-external-store
- Epic React — External store integration — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- Telerik — Third-party library integration — https://www.telerik.com/blogs/react-usesyncexternalstore-hook

---

## 9. `useSyncExternalStore` Best Practices

### Definitions

**Core Definition:** `useSyncExternalStore` best practices are the guidelines for using the Hook correctly — ensuring stable subscriptions, cached snapshots, SSR support, and tearing-safe rendering.

**Technical Definition:** Best practices centre on six principles: declare `subscribe` and `getSnapshot` outside the component (or use `useCallback`) to keep them stable; cache object snapshots and return the same reference when data has not changed; provide `getServerSnapshot` for SSR; avoid suspending on store values; use `useSyncExternalStore` only for external data, not React state; and be aware of the blocking fallback when the store mutates during a Transition.

**Beginner-Friendly Explanation:** Keep your subscribe and snapshot functions stable. Cache your snapshots so React does not re-render forever. Provide a server snapshot if you use SSR. Do not use this Hook for normal React state — use `useState` or `useReducer` instead.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Declare `subscribe` outside the component** | Prevents re-subscription on every render. |
| **Cache object snapshots** | Return the same reference when data has not changed. |
| **Provide `getServerSnapshot` for SSR** | Required to avoid server rendering errors. |
| **Do not suspend on store values** | Store mutations trigger Suspense fallbacks, causing poor UX. |
| **Use only for external data** | Use `useState` / `useReducer` for React-owned state. |
| **Keep `getSnapshot` pure and fast** | It may be called multiple times during render. |
| **Be aware of blocking fallback** | Store mutations during Transitions force blocking updates. |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// subscribe and getSnapshot declared outside
function subscribe(callback) {
  const id = setInterval(() => callback(store.getSnapshot()), 1000);
  return () => clearInterval(id);
}

function getSnapshot() {
  return store.getSnapshot(); // Cached, stable reference
}

function getServerSnapshot() {
  return store.getInitialSnapshot(); // Required for SSR
}

function Component() {
  const value = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return <p>{value}</p>;
}
```

**❌ Bad Practices:**

```jsx
// ❌ subscribe recreated every render
function BadComponent() {
  const value = useSyncExternalStore(
    (cb) => { /* ... */ }, // New function every render
    () => ({ ...store.getSnapshot() }) // New object every call
  );
}

// ❌ No getServerSnapshot for SSR
const value = useSyncExternalStore(subscribe, getSnapshot); // Throws on server
```

**Expected Output:** The good practices produce stable subscriptions and consistent rendering. The bad practices cause re-subscription loops, infinite re-renders, and SSR errors.

**Why this output:** Unstable functions force React to re-subscribe and re-render endlessly. Uncached object snapshots fail the `Object.is` check. Missing `getServerSnapshot` breaks server rendering.

### Real-World Cases

- **All applications using `useSyncExternalStore`:** These practices apply universally.
- **SSR frameworks:** `getServerSnapshot` is mandatory.
- **State library bindings:** Stable snapshots and subscriptions are essential.

### References

- React — useSyncExternalStore: Caveats — https://react.dev/reference/react/useSyncExternalStore
- React — useSyncExternalStore: Troubleshooting — https://18.react.dev/reference/react/useSyncExternalStore
- Epic React — Common mistakes — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0

---

## References

- React — useSyncExternalStore (Official) — https://react.dev/reference/react/useSyncExternalStore
- React — useSyncExternalStore (Markdown) — https://react.dev/reference/react/useSyncExternalStore.md
- React — useSyncExternalStore (React 18) — https://18.react.dev/reference/react/useSyncExternalStore
- Mintlify — useSyncExternalStore — https://mintlify.wiki/facebook/react/api/hooks/use-sync-external-store
- Epic React — useSyncExternalStore: Demystified for Practical React Development — https://www.epicreact.dev/use-sync-external-store-demystified-for-practical-react-development-w5ac0
- Telerik — The React useSyncExternalStore Hook — https://www.telerik.com/blogs/react-usesyncexternalstore-hook
- GitHub — useSyncExternalStore for reading a mutable singleton — https://github.com/pproenca/dot-skills
- GitHub — useSyncExternalStore: tear-free snapshot contract — https://github.com/saschb2b/okf-bundles
- GitHub — Tearing-safe subscriptions — https://github.com/pproenca/dot-skills