# Valtio — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Valtio is a proxy-based state management library for React (and vanilla JS) where you mutate a plain JavaScript object directly, and React components read an immutable snapshot of that object through a hook.

**Technical Definition:** Valtio is a small (~3 KB gzipped) state-management library built on JavaScript `Proxy`. `proxy(initialObject)` returns a mutable proxy object; every property read and write on the proxy is intercepted and tracked. Components use `useSnapshot(proxyObject)` to receive a **frozen, immutable snapshot** of the current state and to subscribe to exactly the properties they read during render. Writes happen **outside** React by mutating the proxy directly (`state.count++`), with no actions, reducers, or setters. Changes propagate to subscribers synchronously (or batched per microtask). Valtio also exports `subscribe` for vanilla usage, `watch` for deep change callbacks, `ref` to opt out of tracking, `derive` to create computed proxies, and `useProxy` for component-local proxy state. It was created by Daishi Kato (also the author of Zustand) and is maintained under the pmndrs umbrella.

**Beginner-Friendly Explanation:** Valtio flips the usual React rule. Normally React wants immutable state and explicit setters. Valtio says: create a normal object, wrap it in `proxy()`, and then just mutate it whenever you want — `state.count++`, `state.todos.push(...)`, `state.user.name = 'Ada'`. React components read the state through `useSnapshot(state)`, which gives them a read-only frozen copy and automatically re-renders them when the parts they read change. The mental model is simple: **write directly to the proxy, read from the snapshot**. It feels like `useState` without setters, or like MobX without annotations.

### Key Characteristics

- **Mutable proxy, immutable snapshot:** You mutate the proxy; React reads a frozen snapshot.
- **No actions or reducers:** Direct assignment (`state.x = 1`) is the update mechanism.
- **Fine-grained subscriptions:** `useSnapshot` tracks which properties were read and re-renders only when those change.
- **Tiny and dependency-free:** ~3 KB gzipped, built on native `Proxy`.
- **Vanilla-first core:** `subscribe`, `watch`, `getVersion`, and `snapshot` work without React.
- **`ref` for non-reactive values:** Opt out of tracking for large, immutable, or external values.
- **`useProxy` for local state:** Component-scoped proxy state without a module-level store.
- **SSR-aware:** Per-request proxies can be created and passed through React Context.
- **Async-friendly:** No `runInAction` equivalent is needed; async mutations work directly.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`).
- JavaScript objects, arrays, `Proxy`, and `Reflect`.
- Understanding of reference equality and re-render triggers.
- TypeScript (recommended; Valtio has first-class types).
- Basic knowledge of state management categories (client, server, URL).

### Related Programming Areas

- **State Management** — Zustand (same author), MobX (reactive), Redux, Jotai.
- **Proxy API** — `Proxy`, `Reflect`, traps.
- **Server State** — TanStack Query, SWR (Valtio is for client state).
- **React Integration** — `useSnapshot`, `useProxy`, `useRef`-like patterns.

---

## 1. Valtio

### Definitions

**Core Definition:** Valtio is a proxy-based state-management library that lets you mutate state directly while React reads immutable snapshots.

**Technical Definition:** Valtio's core API exports `proxy`, `useSnapshot`, `useProxy`, `subscribe`, `watch`, `ref`, `derive`, `getVersion`, and `snapshot`. `proxy(obj)` wraps an object in a `Proxy` that records reads and writes. `useSnapshot(proxyObject)` returns a read-only, frozen snapshot and subscribes the calling component to the properties it reads. `subscribe(proxyObject, callback)` and `watch(proxyObject, callback)` provide non-React subscription APIs. `ref(obj)` marks a value as non-tracked. `derive(fn)` creates a proxy whose value is computed from other proxies. Valtio batches updates per microtask by default and offers `sync` options where needed.

**Beginner-Friendly Explanation:** Valtio is the "just mutate it" state manager. You create a proxy, change it whenever you like, and components read it through `useSnapshot` to stay in sync.

### Purposes

- To provide the simplest possible mental model: mutate state directly, read snapshots in React.
- To eliminate actions, reducers, setters, and boilerplate.
- To keep fine-grained reactivity without manual selectors.
- To scale from a single object to a full application with multiple proxies.
- To work in React and vanilla JavaScript with the same core.
- To support local component state (`useProxy`) and SSR (per-request proxies).

### Syntax Rules and Structure

**General Syntax:**

```ts
import { proxy, useSnapshot } from 'valtio';

export const state = proxy({
  count: 0,
  todos: [] as { id: string; text: string; done: boolean }[],
});

function Counter() {
  const snap = useSnapshot(state); // Read-only snapshot + subscription
  return (
    <button onClick={() => state.count++}>{snap.count}</button>
  );
}
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `proxy(initialObject)` | Creates a mutable proxy; returns the proxy object |
| `state.count++` | Direct mutation; triggers subscribers |
| `useSnapshot(state)` | Returns a frozen snapshot; subscribes the component |
| `snapshot(state)` | Returns a snapshot without subscribing (vanilla) |
| `subscribe(state, cb)` | Subscribes to any change (vanilla) |
| `watch(state, cb)` | Deep watch with operation metadata |
| `useProxy(init)` | Component-local proxy state |
| `ref(value)` | Marks a value as non-tracked |
| `derive(getter)` | Creates a computed proxy |

**Syntax Rules:**

1. Create the proxy at module level (or via `useProxy`/Context for local/SSR state).
2. Mutate the proxy directly; do not call setters.
3. Read state in React **only** through `useSnapshot` (or `useProxy`).
4. Snapshot values are frozen and must be treated as read-only.
5. Subscriptions are per-property-read: only the properties accessed during render trigger re-renders.
6. Use `ref` for values that should not be tracked (large, immutable, or external).
7. Use `subscribe`/`watch` for non-React side effects.

**Constraints and Limitations:**

- Mutating a snapshot throws in development (snapshots are frozen).
- The proxy is a singleton if created at module level; per-request state requires Context.
- Nested objects in snapshots are also frozen, but accessing them repeatedly can create many snapshot objects.
- Valtio does not provide built-in server-state caching; use TanStack Query.
- React 18 Suspense integration has known caveats; keep async logic outside render.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Proxy and Snapshot

```tsx
import { proxy, useSnapshot } from 'valtio';

const counterState = proxy({ count: 0, step: 1 });

function Counter() {
  const snap = useSnapshot(counterState);

  return (
    <div>
      <p>Count: {snap.count}</p>
      <button onClick={() => counterState.count++}>+1</button>
      <button onClick={() => (counterState.step += 1)}>
        Step: {snap.step}
      </button>
    </div>
  );
}
```

**Expected Output:** Clicking `+1` increments `count`; clicking `Step` increments `step`. Only the properties read (`count`, `step`) trigger re-renders.

**Why this output:** `useSnapshot` tracks the property reads (`snap.count`, `snap.step`). Mutating `count` re-renders; mutating an unread property would not.

#### Example 2: Store Outside React with `subscribe`

```ts
import { proxy, subscribe } from 'valtio';

const state = proxy({ count: 0 });

const unsubscribe = subscribe(state, () => {
  console.log('count changed:', state.count);
});

state.count++;         // logs "count changed: 1"
unsubscribe();
state.count++;         // no log
```

**Expected Output:** The subscriber logs once for the first increment; after `unsubscribe()`, no more logs.

**Why this output:** `subscribe` registers a callback on the proxy; the returned function removes it.

### Real-World Cases

- **UI state:** Sidebar open/closed, modal stack, theme.
- **Shopping carts:** Mutable cart array with direct `push`/`splice`.
- **Form drafts:** Mutable draft object shared across steps.
- **Multi-store apps:** Separate proxies per feature.

### References

- Valtio — GitHub Repository — https://github.com/pmndrs/valtio
- Valtio — Official Documentation — https://valtio.dev/
- Valtio — npm — https://www.npmjs.com/package/valtio

---

## 2. Proxy State

### Definitions

**Core Definition:** Proxy state is state wrapped in a JavaScript `Proxy` that tracks reads and writes, allowing direct mutation while notifying subscribers.

**Technical Definition:** `proxy(initialObject)` returns a deep `Proxy` around the object. Property gets are recorded as dependencies for any active snapshot/subscription; property sets trigger notifications. The proxy is **mutable**: `state.x = 1`, `state.arr.push(y)`, and `state.nested.deep = z` all work directly. The proxy is not the same object as the snapshot: the proxy is the writable source of truth, and `snapshot(proxy)` produces an immutable frozen clone whose identity changes only when the underlying data changes. Mutations are batched per microtask by default; `subscribe` callbacks fire after the mutation batch. Use `ref(value)` to store non-tracked values inside the proxy, and `derive` to create proxies whose value is computed from other proxies.

**Beginner-Friendly Explanation:** Proxy state is the mutable object you create with `proxy()`. You change it like a normal object. Valtio watches it and keeps React in sync.

### Purposes

- To provide a writable source of truth with automatic change tracking.
- To allow direct mutation of objects, arrays, and nested structures.
- To batch multiple mutations into a single notification cycle.
- To enable fine-grained subscriptions through property-read tracking.
- To support non-reactive values via `ref` without exposing them to tracking.
- To compose derived state via `derive` without storing duplicates.

### Syntax Rules and Structure

**Object proxy:**

```ts
import { proxy } from 'valtio';

const state = proxy({
  count: 0,
  user: { name: 'Ada', age: 30 },
  todos: [] as Todo[],
});

state.count++;
state.user.name = 'Grace';
state.todos.push({ id: '1', text: 'Learn Valtio', done: false });
```

**Array proxy:**

```ts
const list = proxy<string[]>([]);
list.push('a');
list.splice(0, 1);
```

**`ref` for non-tracked values:**

```ts
import { proxy, ref } from 'valtio';

const state = proxy({
  user: { name: 'Ada' },
  largeConfig: ref({ /* not tracked, must be replaced not mutated */ }),
});
```

**`derive` for computed proxies:**

```ts
import { proxy, derive } from 'valtio/utils';

const state = proxy({ count: 0 });
const doubled = derive({
  doubled: (get) => get(state).count * 2,
});
```

**Syntax Rules:**

1. Mutate the proxy directly; never replace it wholesale (`state = {...}`) — mutate properties instead.
2. Arrays are fully mutable: `push`, `pop`, `splice`, `sort`, and index assignment all work.
3. Nested objects are tracked recursively by default.
4. Use `ref(value)` for values that should not be tracked; replace `ref` values rather than mutating them.
5. Use `derive` for computed values to avoid storing redundant state.
6. Batching is per microtask; use `subscribe` with `{ sync: true }` only when synchronous notifications are required.

**Constraints and Limitations:**

- Proxy state must not be frozen; freezing a proxy breaks mutation tracking.
- `ref` values are not tracked; mutating them does not trigger re-renders.
- Very large or deeply nested proxies add tracking overhead; use `ref` for large immutable sub-trees.
- Reassigning the proxy variable does not work: `let state = proxy(...); state = {...}` breaks subscriptions.
- `Proxy` is not supported in some older environments; Valtio assumes modern JavaScript.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Mutable Array Operations

```ts
import { proxy } from 'valtio';

const todos = proxy<{ id: string; text: string; done: boolean }[]>([]);

function addTodo(text: string) {
  todos.push({ id: crypto.randomUUID(), text, done: false });
}

function toggleTodo(id: string) {
  const todo = todos.find((t) => t.id === id);
  if (todo) todo.done = !todo.done;
}

function removeTodo(id: string) {
  const index = todos.findIndex((t) => t.id === id);
  if (index !== -1) todos.splice(index, 1);
}
```

**Expected Output:** `addTodo` appends, `toggleTodo` flips `done`, and `removeTodo` removes by index. All operations are direct mutations.

**Why this output:** Valtio tracks array mutations (`push`, `splice`) and nested property writes (`todo.done = ...`) transparently.

#### Example 2: `ref` for Non-Tracked Values

```ts
import { proxy, ref } from 'valtio';

const state = proxy({
  user: { name: 'Ada' },
  config: ref({ theme: 'dark', locale: 'en' }),
});

// Replacing ref values is allowed
state.config = ref({ theme: 'light', locale: 'en' });

// Mutating the ref object does NOT trigger re-renders
state.config.theme = 'blue'; // no notification
```

**Expected Output:** Replacing `config` notifies subscribers; mutating `config.theme` does not.

**Why this output:** `ref` marks the value as non-reactive. Valtio tracks the reference, not the internals.

### Real-World Cases

- **Carts and lists:** Arrays mutated with `push`/`splice`.
- **Nested forms:** Mutable nested form state.
- **Config objects:** Large immutable configs wrapped in `ref`.
- **Derived totals:** `derive` for computed values.

### References

- Valtio — Proxy state — https://valtio.dev/docs/api/basic/proxy
- Valtio — Ref — https://valtio.dev/docs/api/basic/ref
- Valtio — Derive — https://valtio.dev/docs/api/utils/derive
- Valtio — GitHub Repository — https://github.com/pmndrs/valtio

---

## 3. `useSnapshot`

### Definitions

**Core Definition:** `useSnapshot` is the React hook that returns a read-only snapshot of a proxy and subscribes the component to the properties it reads.

**Technical Definition:** `useSnapshot(proxyObject, options?)` returns a deeply frozen snapshot of the proxy's current state and subscribes the calling component to the specific properties accessed during render. It triggers a re-render only when a property that was read during the previous render changes (or, for newly added properties, when the relevant branch changes). The snapshot is immutable and its nested values are frozen in development. `useSnapshot` is the only sanctioned way to read proxy state in React; reading the proxy directly in render bypasses subscription and produces stale UI. The hook optionally accepts options such as `sync` to control batching behaviour.

**Beginner-Friendly Explanation:** `useSnapshot` gives your component a read-only copy of the current state and automatically re-renders the component when the parts it read change. Read from the snapshot, write to the proxy.

### Purposes

- To read proxy state in React with automatic, fine-grained subscription.
- To provide an immutable, frozen view of mutable state.
- To re-render components only when the properties they read change.
- To prevent accidental mutation of state inside render.
- To keep React's rendering model safe while allowing direct mutations elsewhere.

### Syntax Rules and Structure

**General Syntax:**

```tsx
import { proxy, useSnapshot } from 'valtio';

const state = proxy({ count: 0, user: { name: 'Ada' } });

function Counter() {
  const snap = useSnapshot(state);
  return <button onClick={() => state.count++}>{snap.count}</button>;
}
```

**Reading nested values:**

```tsx
const snap = useSnapshot(state);
return <span>{snap.user.name}</span>;
```

**Reading with stable callbacks:**

```tsx
function TodoList() {
  const snap = useSnapshot(state);
  return snap.todos.map((todo) => (
    <li key={todo.id} onClick={() => { state.todos.find(t => t.id === todo.id)!.done = true; }}>
      {todo.text}
    </li>
  ));
}
```

**Syntax Rules:**

1. Always read state through the snapshot inside render.
2. Never mutate the snapshot; it is frozen.
3. Mutate the **proxy** (the original `state` object) for updates.
4. Selectors are not needed; access only the properties you need, and Valtio tracks them.
5. Accessing a property during render creates a dependency on that property's value.
6. Passing `state` itself (not `snap`) to children is an anti-pattern; pass `snap` values or the proxy for mutation.

**Constraints and Limitations:**

- Mutating a snapshot throws in development (`Object.freeze`).
- Reading the proxy directly in render produces no subscription and stale UI.
- Accessing a nested object (e.g., `snap.user`) subscribes to that branch; granular access (`snap.user.name`) is more precise.
- Passing the snapshot to a child component re-renders that child when the snapshot identity changes, which can be broader than necessary; pass only what the child needs.
- Suspense/async rendering interactions have known caveats; keep async reads outside render.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Fine-Grained Re-renders

```tsx
import { proxy, useSnapshot } from 'valtio';

const state = proxy({ count: 0, name: 'Ada' });

function CountDisplay() {
  const snap = useSnapshot(state);
  console.log('CountDisplay rendered');
  return <span>{snap.count}</span>; // Reads only count
}

function NameDisplay() {
  const snap = useSnapshot(state);
  console.log('NameDisplay rendered');
  return <span>{snap.name}</span>; // Reads only name
}

function App() {
  return (
    <>
      <CountDisplay />
      <NameDisplay />
      <button onClick={() => state.count++}>Increment</button>
      <button onClick={() => (state.name = 'Grace')}>Rename</button>
    </>
  );
}
```

**Expected Output:** Clicking “Increment” logs only `CountDisplay rendered`; clicking “Rename” logs only `NameDisplay rendered`.

**Why this output:** Each component reads a distinct property through `useSnapshot`; Valtio subscribes each to the property it read.

#### Example 2: Snapshot Is Read-Only

```tsx
function Bad() {
  const snap = useSnapshot(state);
  snap.count++; // Throws in development: snapshot is frozen
  return <span>{snap.count}</span>;
}

function Good() {
  const snap = useSnapshot(state);
  return <button onClick={() => state.count++}>{snap.count}</button>;
}
```

**Expected Output:** `Bad` throws in development; `Good` works correctly.

**Why this output:** Snapshots are frozen to enforce read-only access; mutations must target the proxy.

### Real-World Cases

- **Any component reading shared state:** The standard Valtio pattern.
- **Lists and nested data:** Reading arrays and nested objects through snapshots.
- **Performance-sensitive components:** Fine-grained subscriptions reduce re-renders.

### References

- Valtio — useSnapshot — https://valtio.dev/docs/api/basic/useSnapshot
- Valtio — GitHub Repository — https://github.com/pmndrs/valtio
- Valtio — React Integration — https://valtio.dev/docs/api/basic/useSnapshot

---

## 4. Valtio Best Practices

### Definitions

**Core Definition:** Valtio best practices are the guidelines for structuring proxies, reading snapshots, handling async updates, and organising stores to build maintainable and performant applications.

**Technical Definition:** Best practices centre on **separating reads from writes** (mutate the proxy, read the snapshot), **avoiding snapshot mutation**, **using `ref` for non-reactive values**, **using `useProxy` for component-local state**, **using `derive` for computed values instead of storing them**, **splitting proxies by domain**, **handling SSR with per-request proxies via Context**, **avoiding passing the whole proxy or whole snapshot to children**, **using `subscribe`/`watch` for side effects**, and **understanding batching and `sync` options**.

**Beginner-Friendly Explanation:** Good Valtio usage means mutating the proxy and reading the snapshot, never mutating the snapshot, using `ref` for big non-reactive values, using `useProxy` for local state, splitting stores by feature, and wrapping per-request proxies for SSR.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Mutate proxy, read snapshot** | Writes go to the proxy; reads go through `useSnapshot` |
| **Never mutate a snapshot** | Snapshots are frozen and read-only |
| **Read only what you need** | Fine-grained property access reduces re-renders |
| **Pass values, not the whole snapshot** | Avoid broad re-renders in children |
| **Use `ref` for non-reactive values** | Large immutable data, external objects |
| **Use `useProxy` for local state** | Component-scoped proxies without module singletons |
| **Use `derive` for computed values** | Avoid stored redundant state |
| **Split proxies by domain** | One proxy per feature/store |
| **Use `subscribe`/`watch` for side effects** | Persistence, analytics, external sync |
| **Handle SSR with Context** | Create per-request proxies and pass via Context |
| **Understand batching** | Updates are batched per microtask; use `sync` only when needed |
| **Avoid reading the proxy in render** | It bypasses subscription and produces stale UI |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
import { proxy, useSnapshot, useProxy } from 'valtio';

// Domain-scoped proxy
export const cartState = proxy({
  items: [] as { id: string; price: number; qty: number }[],
  discountCode: null as string | null,
});

// Read snapshot, mutate proxy
function CartTotal() {
  const snap = useSnapshot(cartState);
  const total = snap.items.reduce((sum, i) => sum + i.price * i.qty, 0);
  return <span>${total.toFixed(2)}</span>;
}

// Component-local proxy
function Counter() {
  const state = useProxy({ count: 0 });
  const snap = useSnapshot(state);
  return <button onClick={() => state.count++}>{snap.count}</button>;
}

// Pass only the needed value to children
function ItemRow({ item }: { item: { id: string; price: number } }) {
  return <span>{item.price}</span>;
}
```

**❌ Bad Practices:**

```tsx
// ❌ Reading the proxy directly in render: no subscription
function Bad() {
  return <span>{cartState.items.length}</span>; // Stale
}

// ❌ Mutating the snapshot
const snap = useSnapshot(cartState);
snap.items.push(...); // Throws: snapshot is frozen

// ❌ Passing the whole proxy to a child
<Child state={cartState} /> // Child reads without useSnapshot

// ❌ Storing derived state in the proxy
const state = proxy({ items: [], total: 0 }); // Derive instead

// ❌ Freezing the proxy
Object.freeze(state); // Breaks mutation tracking

// ❌ Global singleton for per-request data
export const state = proxy({ user: null }); // Use Context for SSR
```

**Expected Output:** The good practices produce reactive, fine-grained, SSR-safe state. The bad practices cause stale UI, runtime errors, broad re-renders, redundant state, and cross-request leaks.

**Why this output:** Reading the proxy bypasses tracking; snapshots are frozen; passing the proxy to children bypasses `useSnapshot`; stored derived state drifts; freezing breaks Proxy tracking; global singletons leak across SSR requests.

### Real-World Cases

- **All Valtio applications:** These practices apply universally.
- **SSR apps:** Per-request proxies via Context.
- **Large apps:** Domain-scoped proxies and `derive` for computed values.
- **Performance-sensitive UIs:** Fine-grained reads and avoiding broad snapshot passing.

### References

- Valtio — Official Documentation — https://valtio.dev/
- Valtio — GitHub Repository — https://github.com/pmndrs/valtio
- Valtio — useProxy — https://valtio.dev/docs/api/basic/useProxy
- Valtio — Subscribe — https://valtio.dev/docs/api/basic/subscribe
- Valtio — Watch — https://valtio.dev/docs/api/basic/watch
- Valtio — Derive — https://valtio.dev/docs/api/utils/derive

---

## References

- Valtio — Official Documentation — https://valtio.dev/
- Valtio — GitHub Repository — https://github.com/pmndrs/valtio
- Valtio — npm — https://www.npmjs.com/package/valtio
- Valtio — Proxy state — https://valtio.dev/docs/api/basic/proxy
- Valtio — useSnapshot — https://valtio.dev/docs/api/basic/useSnapshot
- Valtio — useProxy — https://valtio.dev/docs/api/basic/useProxy
- Valtio — subscribe — https://valtio.dev/docs/api/basic/subscribe
- Valtio — watch — https://valtio.dev/docs/api/basic/watch
- Valtio — ref — https://valtio.dev/docs/api/basic/ref
- Valtio — derive — https://valtio.dev/docs/api/utils/derive
- Valtio — Snapshot — https://valtio.dev/docs/api/advanced/snapshot
- Valtio — Comparison with other libraries — https://github.com/pmndrs/valtio#how-does-valtio-compare-with-redux
- MDN — Proxy — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
- MDN — Reflect — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect