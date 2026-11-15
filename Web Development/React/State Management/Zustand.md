# Zustand — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Zustand is a small, fast, and scalable barebones state-management solution for React that uses simplified flux principles and a hooks-first API with no boilerplate and no provider requirement.

**Technical Definition:** Zustand (German for "state") is a minimal state-management library built on an immutable store model. A store is created with `create` and returns a hook that accepts a selector function. The store exposes `set` and `get` functions to actions, and a `subscribe` API for non-React usage. Because selectors subscribe to specific slices of state, components re-render only when the selected value changes, giving fine-grained reactivity without providers, reducers, or action types. Zustand ships a middleware ecosystem (`persist`, `devtools`, `immer`, `subscribeWithSelector`, `combine`, `redux`) and a vanilla `createStore` API for non-React environments and per-request SSR stores. Zustand v5 removed default exports, requires selectors to avoid infinite loops, and provides `useShallow` for selecting multiple values without re-render storms.

**Beginner-Friendly Explanation:** Zustand is the simplest way to share state across React components. You create a store with `create`, put your state and the functions that change it inside, and then any component can read just the piece it needs. There is no provider wrapping, no action types, and no reducers — just a hook. It is tiny (about 1 KB) and fast because each component only re-renders when the exact value it reads changes. If Redux feels like filling out paperwork to change a boolean, Zustand is the sticky note on your monitor.

### Key Characteristics

- **No providers:** Stores are module-level hooks; no `<Provider>` wrapper is required.
- **Selector-based subscriptions:** Components subscribe to specific slices and re-render only when those slices change.
- **Minimal API:** `create`, `set`, `get`, and `subscribe` are the core surface.
- **Tiny footprint:** ~1.2 KB gzipped, with no dependencies.
- **Middleware ecosystem:** `persist`, `devtools`, `immer`, `subscribeWithSelector`, `combine`, `redux`.
- **Vanilla store:** `createStore` works outside React and enables per-request SSR stores.
- **Transient updates:** `subscribe` with selectors enables updates without re-renders.
- **Slices pattern:** Large stores are composed from smaller, independently-typed slices.
- **v5 strictness:** Selectors are mandatory; `useShallow` is required for selecting multiple values safely.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`).
- JavaScript objects, functions, and closures.
- Understanding of reference equality and re-render triggers.
- TypeScript (recommended; Zustand's TypeScript inference is excellent).
- Basic knowledge of middleware concepts (for `persist`, `devtools`, `immer`).

### Related Programming Areas

- **State Management** — Redux, Jotai, Context API.
- **Server State** — TanStack Query, SWR (Zustand is for client state).
- **Persistence** — `localStorage`, `sessionStorage`, IndexedDB.
- **DevTools** — Redux DevTools Extension integration.

---

## 1. Zustand

### Definitions

**Core Definition:** Zustand is a minimal, hooks-first state-management library that creates a store from a single function and exposes it as a React hook.

**Technical Definition:** Zustand's core is a store factory (`create`) that accepts a state creator function `(set, get, store) => state`. The returned hook accepts a selector `(state) => slice` and subscribes the calling component to that slice. The store exposes `setState`, `getState`, `getInitialState`, and `subscribe`. Actions are plain functions defined inside the state creator, closing over `set` and `get`. Zustand does not use reducers, action types, or a provider by default; the store is a module-level singleton. For SSR, per-request stores are created with `createStore` and provided through React Context.

**Beginner-Friendly Explanation:** Zustand gives you a hook that holds your shared state. You define the state and the functions that change it in one place, and components call the hook with a selector to read what they need.

### Purposes

- To share client state across components without providers or boilerplate.
- To provide fine-grained subscriptions so only affected components re-render.
- To keep state logic colocated with the state itself.
- To offer a minimal API that scales from a single boolean to a large application.
- To support non-React usage through the vanilla store API.
- To enable persistence, DevTools, and immutable updates through middleware.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `create(stateCreator)` | Creates the store and returns a hook |
| `set(partial)` | Merges the partial into state and notifies subscribers |
| `get()` | Returns the current state |
| `useStore(selector)` | Subscribes the component to the selected slice |
| `useStore.getState()` | Reads state outside React |
| `useStore.setState()` | Updates state outside React |
| `useStore.subscribe()` | Listens for changes outside React |

**Syntax Rules:**

1. Define the store at module level; the hook is a singleton.
2. Always pass a selector to the hook in v5: `useStore((s) => s.count)`.
3. Define actions inside the state creator, closing over `set` and `get`.
4. Use the curried TypeScript syntax `create<State>()(...)` for inference.
5. Use `useShallow` when the selector returns a new object or array.

**Constraints and Limitations:**

- v5 removed default exports and requires selectors; calling the hook without a selector returns the whole state and re-renders on any change.
- Zustand has no built-in time-travel debugging; the `devtools` middleware provides action inspection.
- Zustand is for client state; server state belongs in TanStack Query or SWR.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Counter Store

```ts
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
  reset: () => void;
}

const useCounterStore = create<CounterState>()((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}));
```

```tsx
function Counter() {
  const count = useCounterStore((s) => s.count);
  const increment = useCounterStore((s) => s.increment);
  return <button onClick={increment}>{count}</button>;
}
```

**Expected Output:** Clicking the button increments the count; only the component reading `count` re-renders.

**Why this output:** The selector `(s) => s.count` subscribes the component to `count` only. `increment` is a stable function reference and does not cause re-renders.

#### Example 2: Store Outside React

```ts
const state = useCounterStore.getState();
state.increment();

useCounterStore.setState({ count: 10 });

const unsubscribe = useCounterStore.subscribe((state) => {
  console.log('count changed:', state.count);
});

unsubscribe();
```

**Expected Output:** `increment()` updates the count; `setState` replaces it; `subscribe` logs changes and can be unsubscribed.

**Why this output:** The store exposes imperative APIs alongside the React hook, enabling non-React usage and integration with tests.

### Real-World Cases

- **UI state:** Sidebar open/closed, modal stack, active workspace.
- **User preferences:** Theme, language, density.
- **Shopping cart:** Client-side cart state.
- **Multi-step forms:** Shared wizard state across steps.

### References

- Zustand — Official Documentation — https://zustand.docs.pmnd.rs/
- Zustand — GitHub — https://github.com/pmndrs/zustand
- Zustand — Getting Started — https://zustand.docs.pmnd.rs/getting-started/introduction

---

## 2. Store Creation

### Definitions

**Core Definition:** Store creation is the act of defining a Zustand store with `create`, producing a hook that reads and updates the state.

**Technical Definition:** `create(stateCreator)` accepts a function `(set, get, store) => state` and returns a React hook. The state creator defines the initial state and the actions. `set` accepts a partial object or an updater function `(state) => partial`; it shallowly merges the result into the existing state (unless the `replace` flag is used). `get` returns the current state. The store also exposes `getState`, `setState`, `getInitialState`, and `subscribe`. For TypeScript, the curried form `create<State>()(creator)` enables full inference. For SSR and non-React use, `createStore` from `zustand/vanilla` creates a store without a hook.

**Beginner-Friendly Explanation:** Store creation is how you define your Zustand store. You write one function that returns the initial state and the actions, and Zustand turns it into a hook your components can use.

### Purposes

- To define the initial state and actions in one place.
- To produce a hook that components can subscribe to with selectors.
- To expose imperative APIs (`getState`, `setState`, `subscribe`) for non-React code.
- To support per-request stores for SSR with `createStore`.
- To enable typed stores with TypeScript inference.
- To compose middleware for persistence, DevTools, and Immer.

### Syntax Rules and Structure

**General Syntax (React store):**

```ts
import { create } from 'zustand';

const useStore = create<State>()((set, get) => ({
  ...initialState,
  ...actions,
}));
```

**General Syntax (vanilla store):**

```ts
import { createStore } from 'zustand/vanilla';

const store = createStore<State>()((set, get) => ({ ... }));
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `create<State>()` | Curried form for TypeScript inference |
| `stateCreator(set, get, store)` | Function returning the initial state and actions |
| `set(partialOrUpdater, replace?)` | Merges state; `replace: true` replaces the entire state |
| `get()` | Returns the current state |
| `createStore` | Vanilla store without a React hook |

**Syntax Rules:**

1. Use the curried `create<State>()(...)` form in TypeScript; the non-curried form loses inference.
2. Define actions inside the state creator, not outside.
3. Use `set` with an updater function when the next state depends on the previous state.
4. Use `replace: true` only when the new state is complete.
5. Use `createStore` for per-request SSR stores or non-React environments.
6. Export the hook (and optionally the vanilla store) from a dedicated module.

**Constraints and Limitations:**

- A store created with `create` is a singleton; it is shared across all components and requests. Use `createStore` per request for SSR.
- Replacing the entire state with `set(newState, true)` can remove actions; use it carefully.
- The state creator runs once at module load; it should be pure and free of side effects.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: TypeScript Store with Actions

```ts
import { create } from 'zustand';

interface Todo {
  id: string;
  text: string;
  done: boolean;
}

interface TodoState {
  todos: Todo[];
  addTodo: (text: string) => void;
  toggleTodo: (id: string) => void;
}

export const useTodoStore = create<TodoState>()((set) => ({
  todos: [],
  addTodo: (text) =>
    set((state) => ({
      todos: [...state.todos, { id: crypto.randomUUID(), text, done: false }],
    })),
  toggleTodo: (id) =>
    set((state) => ({
      todos: state.todos.map((t) =>
        t.id === id ? { ...t, done: !t.done } : t
      ),
    })),
}));
```

**Expected Output:** `addTodo` appends a todo; `toggleTodo` flips the `done` flag immutably.

**Why this output:** `set` with an updater returns a new state object; the component re-renders only if the selected slice changes.

#### Example 2: Per-Request Store for SSR

```ts
// store/todo-store.ts
import { createStore } from 'zustand/vanilla';

export const createTodoStore = (initialTodos: Todo[] = []) =>
  createStore<TodoState>()((set) => ({
    todos: initialTodos,
    addTodo: (text) => set((s) => ({ todos: [...s.todos, { id: crypto.randomUUID(), text, done: false }] })),
    toggleTodo: (id) => set((s) => ({ todos: s.todos.map((t) => t.id === id ? { ...t, done: !t.done } : t) })),
  }));

export type TodoStore = ReturnType<typeof createTodoStore>;
```

```tsx
// app/providers.tsx
'use client';
import { createContext, useContext, useRef } from 'react';
import { useStore } from 'zustand';
import { createTodoStore, type TodoStore } from './store/todo-store';

const TodoStoreContext = createContext<TodoStore | null>(null);

export function TodoStoreProvider({ children, initialTodos }: { children: React.ReactNode; initialTodos?: Todo[] }) {
  const storeRef = useRef<TodoStore>();
  if (!storeRef.current) storeRef.current = createTodoStore(initialTodos);
  return <TodoStoreContext.Provider value={storeRef.current}>{children}</TodoStoreContext.Provider>;
}

export function useTodoStore<T>(selector: (state: TodoState) => T): T {
  const store = useContext(TodoStoreContext);
  if (!store) throw new Error('useTodoStore must be used within TodoStoreProvider');
  return useStore(store, selector);
}
```

**Expected Output:** A new store is created per request/component tree, preventing state leakage between users.

**Why this output:** `createStore` returns a vanilla store; the provider supplies it via context; `useStore(store, selector)` subscribes with the same selector semantics.

### Real-World Cases

- **Feature stores:** One store per feature (todos, cart, auth).
- **SSR apps:** Per-request stores via `createStore` and Context.
- **Non-React code:** Vanilla stores used in tests, workers, or legacy code.

### References

- Zustand — create — https://zustand.docs.pmnd.rs/apis/create
- Zustand — createStore (vanilla) — https://zustand.docs.pmnd.rs/apis/create-store
- Zustand — TypeScript Guide — https://zustand.docs.pmnd.rs/guides/typescript
- Zustand — Setup with Next.js — https://zustand.docs.pmnd.rs/guides/nextjs

---

## 3. State Selectors

### Definitions

**Core Definition:** A state selector is a function passed to the Zustand hook that extracts a specific slice of state and subscribes the component to that slice only.

**Technical Definition:** `useStore(selector)` calls the selector with the current state and subscribes the component to the result. Zustand compares the previous and next selector results with `Object.is`; the component re-renders only when the result changes. In v5, calling the hook without a selector returns the entire state and re-renders on any change, so selectors are mandatory for correctness. When a selector returns a new object or array, `useShallow` is required to prevent infinite re-render loops by performing a shallow comparison. `useShallow` is exported from `zustand/react/shallow`.

**Beginner-Friendly Explanation:** A selector is how a component says "I only care about this piece of the store." Zustand watches that piece and re-renders the component only when it changes.

### Purposes

- To subscribe components to only the state they need.
- To prevent unnecessary re-renders when unrelated state changes.
- To avoid infinite loops when selecting derived objects with `useShallow`.
- To keep components decoupled from the store's full shape.
- To enable stable function selectors without re-rendering.
- To derive computed values in a memoized, predictable way.

### Syntax Rules and Structure

**General Syntax:**

```ts
const count = useStore((state) => state.count);
const { a, b } = useStore(useShallow((state) => ({ a: state.a, b: state.b })));
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `(state) => slice` | Selector function |
| `useShallow(...)` | Wraps a selector that returns a new object/array |
| `Object.is` | Comparison used by Zustand |

**Syntax Rules:**

1. Always pass a selector; v5 requires it.
2. Use `useShallow` when the selector returns a new object or array.
3. Use atomic selectors (`state.count`) rather than selecting the whole state.
4. Define selectors outside the component when they are reused.
5. Use `useStore(state => state.action)` for actions; function references are stable.

**Constraints and Limitations:**

- A selector that returns a new object on every call causes infinite re-renders without `useShallow`.
- Selecting the whole state (`state => state`) re-renders on every change.
- Derived arrays/objects must be memoized with `useShallow` or `useMemo`.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Atomic Selectors

```tsx
const count = useCounterStore((s) => s.count);
const increment = useCounterStore((s) => s.increment);
```

**Expected Output:** The component re-renders only when `count` changes.

**Why this output:** The selector returns a primitive; `Object.is` compares values.

#### Example 2: Selecting Multiple Values with `useShallow`

```tsx
import { useShallow } from 'zustand/react/shallow';

const { count, step } = useCounterStore(
  useShallow((s) => ({ count: s.count, step: s.step }))
);
```

**Expected Output:** The component re-renders only when `count` or `step` changes, not on unrelated state changes.

**Why this output:** `useShallow` performs a shallow comparison of the returned object's properties, preventing infinite loops and unnecessary re-renders.

### Real-World Cases

- **Header components:** Selecting only the cart count.
- **Sidebar:** Selecting only the sidebar's open flag.
- **Lists:** Selecting only the relevant slice of a list.
- **Actions:** Selecting stable action references.

### References

- Zustand — Prevent rerenders with useShallow — https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow
- Zustand — Practice with no store actions — https://zustand.docs.pmnd.rs/guides/practice-with-no-store-actions
- Zustand — TypeScript Guide: Selecting multiple state slices — https://zustand.docs.pmnd.rs/guides/typescript#selecting-multiple-state-slices

---

## 4. Actions

### Definitions

**Core Definition:** An action is a plain function defined inside the Zustand state creator that reads or updates the store state.

**Technical Definition:** Actions are functions stored in the state object alongside the state values. They close over `set` and `get` from the state creator. `set` accepts a partial state or an updater function; `get` returns the current state. Actions are stable references (defined once at store creation), so selecting them in components does not cause re-renders. Zustand allows actions to call other actions and to read the latest state via `get()`. Unlike Redux, actions are not dispatched; they are called directly.

**Beginner-Friendly Explanation:** Actions are the functions that change your state. You write them inside the store definition, and components call them directly.

### Purposes

- To encapsulate state transitions in named functions.
- To provide stable function references for selectors.
- To allow actions to read and call other actions.
- To colocate state logic with state.
- To avoid action types, reducers, and dispatch.
- To support async actions that call `set` after awaiting.

### Syntax Rules and Structure

**General Syntax:**

```ts
const useStore = create<State>()((set, get) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
  incrementBy: (amount: number) => set((s) => ({ count: s.count + amount })),
  reset: () => set({ count: 0 }),
}));
```

**Syntax Rules:**

1. Define actions inside the state creator.
2. Use `set` with an updater when the next state depends on the previous.
3. Use `get()` to read the current state inside an action.
4. Actions can call other actions via `get().otherAction()`.
5. Async actions can call `await` and then `set`.
6. Actions should be pure with respect to the state they update; side effects belong in components or middleware.

**Constraints and Limitations:**

- Actions are not memoized beyond their initial definition; do not recreate them inside the state creator on every update.
- Async actions are not cancellable by default; use an `AbortController` for fetch cancellation.
- Calling `set` with `replace: true` inside an action can remove other actions.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Synchronous Actions

```ts
const useCounterStore = create<CounterState>()((set) => ({
  count: 0,
  step: 1,
  increment: () => set((s) => ({ count: s.count + s.step })),
  setStep: (step: number) => set({ step }),
  reset: () => set({ count: 0 }),
}));
```

**Expected Output:** `increment` reads the current `step` and updates `count`; `setStep` changes the step; `reset` resets the count.

**Why this output:** Each action uses `set` with an updater that reads the latest state.

#### Example 2: Async Action with `get()`

```ts
const useUserStore = create<UserState>()((set, get) => ({
  user: null,
  loading: false,
  error: null,
  fetchUser: async (id: string) => {
    set({ loading: true, error: null });
    try {
      const res = await fetch(`/api/users/${id}`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const user = await res.json();
      set({ user, loading: false });
    } catch (error) {
      set({ error: (error as Error).message, loading: false });
    }
  },
  clear: () => set({ user: null }),
}));
```

**Expected Output:** `fetchUser` sets loading, fetches, and updates `user` or `error`.

**Why this output:** The action uses `set` for each phase and `await` for the async work.

### Real-World Cases

- **Cart actions:** `addItem`, `removeItem`, `clearCart`.
- **Auth actions:** `login`, `logout`, `setUser`.
- **UI actions:** `openModal`, `closeModal`, `setTheme`.
- **Data actions:** `fetchUser`, `fetchTodos`.

### References

- Zustand — Practice with no store actions — https://zustand.docs.pmnd.rs/guides/practice-with-no-store-actions
- Zustand — TypeScript Guide — https://zustand.docs.pmnd.rs/guides/typescript

---

## 5. Middleware

### Definitions

**Core Definition:** Zustand middleware are functions that wrap the state creator to add capabilities — persistence, DevTools, Immer, subscription selectors, or Redux compatibility — without changing the core API.

**Technical Definition:** Middleware in Zustand is a higher-order function that takes a state creator and returns a new state creator with additional behaviour. The built-in middleware are `persist`, `devtools`, `immer`, `subscribeWithSelector`, `combine`, and `redux`. Middleware is composed by nesting: `create<State>()(devtools(persist(immer(creator), options)))`. The order matters: the innermost middleware runs first on updates, and the outermost wraps the resulting store. Custom middleware follows the `StateCreator<State, Mutators, [], State>` signature.

**Beginner-Friendly Explanation:** Middleware are add-ons for your store. You can add persistence, DevTools, or Immer by wrapping your store definition.

### Purposes

- To persist state to storage with `persist`.
- To integrate with Redux DevTools with `devtools`.
- To write "mutating" updates with Immer via `immer`.
- To subscribe to selectors with `subscribeWithSelector`.
- To support Redux-style reducers with `redux`.
- To compose multiple capabilities in a predictable order.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { create } from 'zustand';
import { devtools, persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

const useStore = create<State>()(
  devtools(
    persist(
      immer((set) => ({ ... })),
      { name: 'store', storage: createJSONStorage(() => localStorage) }
    ),
    { name: 'MyStore' }
  )
);
```

**Component Breakdown:**

| Middleware | Purpose |
|------------|---------|
| `persist` | Saves state to storage and rehydrates it |
| `devtools` | Connects the store to Redux DevTools |
| `immer` | Enables "mutating" updates via Immer |
| `subscribeWithSelector` | Enables selector-based `store.subscribe` |
| `combine` | Combines initial state and actions (legacy helper) |
| `redux` | Wraps a Redux-style reducer |

**Syntax Rules:**

1. Compose middleware by nesting; the innermost runs first.
2. Use `create<T>()(middleware(...))` for TypeScript inference.
3. The order of `persist` and `immer` affects how updates are serialized; `immer` should typically be innermost.
4. `devtools` should wrap the outer store to capture all actions.
5. Middleware options are passed as the second argument to each wrapper.

**Constraints and Limitations:**

- Middleware order matters; a wrong order can break persistence or DevTools.
- `persist` and `devtools` add small overhead; disable DevTools in production.
- `immer` changes the update semantics; do not mix mutable and immutable updates carelessly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Composing Persist, Immer, and DevTools

```ts
import { create } from 'zustand';
import { devtools, persist, createJSONStorage } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface State {
  nested: { count: number };
  increment: () => void;
}

export const useStore = create<State>()(
  devtools(
    persist(
      immer((set) => ({
        nested: { count: 0 },
        increment: () =>
          set((state) => {
            state.nested.count += 1;
          }),
      })),
      { name: 'nested-store', storage: createJSONStorage(() => localStorage) }
    ),
    { name: 'NestedStore' }
  )
);
```

**Expected Output:** The store persists to `localStorage`, updates use Immer's mutable syntax, and every action appears in Redux DevTools.

**Why this output:** `immer` enables mutable updates; `persist` serializes the result; `devtools` wraps the store and logs actions.

#### Example 2: `subscribeWithSelector` for Transient Updates

```ts
import { create } from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';

const useStore = create<State>()(
  subscribeWithSelector((set) => ({
    count: 0,
    increment: () => set((s) => ({ count: s.count + 1 })),
  }))
);

useStore.subscribe(
  (state) => state.count,
  (count, previousCount) => {
    console.log(`count changed: ${previousCount} → ${count}`);
  }
);
```

**Expected Output:** The subscriber is called only when `count` changes, with both the new and previous values.

**Why this output:** `subscribeWithSelector` enables `store.subscribe(selector, listener)`.

### Real-World Cases

- **Persistence:** Theme, sidebar state, draft forms.
- **DevTools:** Debugging state transitions in development.
- **Immer:** Deeply nested state updates.
- **Transient updates:** Animations, scroll tracking, canvas rendering.

### References

- Zustand — Middleware — https://zustand.docs.pmnd.rs/middlewares/persist
- Zustand — Devtools middleware — https://zustand.docs.pmnd.rs/middlewares/devtools
- Zustand — Immer middleware — https://zustand.docs.pmnd.rs/middlewares/immer
- Zustand — Subscribe with selector middleware — https://zustand.docs.pmnd.rs/middlewares/subscribe-with-selector

---

## 6. Persistence

### Definitions

**Core Definition:** Persistence is the `persist` middleware that saves store state to a storage backend and rehydrates it on load.

**Technical Definition:** `persist(stateCreator, options)` wraps the store and saves its state to a storage backend (default `localStorage`). Options include `name` (storage key), `storage` (a `StateStorage` adapter, typically `createJSONStorage(() => localStorage)`), `partialize` (select which state to persist), `version` (for migrations), `migrate` (upgrade persisted state), `onRehydrateStorage` (callback after rehydration), `skipHydration` (manual hydration), and `merge` (custom merge logic). The middleware stores the state as JSON and rehydrates asynchronously.

**Beginner-Friendly Explanation:** Persistence means your store remembers its state after a page reload. You add the `persist` middleware, and Zustand saves the state to `localStorage` and loads it back on startup.

### Purposes

- To preserve user preferences across sessions.
- To save draft forms and wizard progress.
- To persist authentication tokens (with security considerations).
- To migrate persisted state across versions.
- To control exactly which state is persisted.
- To support SSR with manual hydration.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { persist, createJSONStorage } from 'zustand/middleware';

const useStore = create<State>()(
  persist(
    (set) => ({ ... }),
    {
      name: 'app-store',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ theme: state.theme }),
      version: 1,
      migrate: (persisted, version) => { ... },
      onRehydrateStorage: () => (state, error) => { ... },
      skipHydration: false,
    }
  )
);
```

**Component Breakdown:**

| Option | Description |
|--------|-------------|
| `name` | Storage key |
| `storage` | Storage adapter (default `localStorage` via JSON) |
| `partialize` | Function selecting which state to persist |
| `version` | Persisted state version |
| `migrate` | Function upgrading old persisted state |
| `onRehydrateStorage` | Callback after rehydration |
| `skipHydration` | Defer hydration until `store.persist.rehydrate()` |

**Syntax Rules:**

1. Always specify `name` to avoid key collisions.
2. Use `createJSONStorage(() => localStorage)` for JSON-compatible state.
3. Use `partialize` to exclude sensitive or non-serializable state.
4. Increment `version` when the persisted shape changes and provide `migrate`.
5. Use `skipHydration: true` for SSR and call `useStore.persist.rehydrate()` on the client.
6. Provide `onRehydrateStorage` to handle errors and loading states.

**Constraints and Limitations:**

- Persisted state must be JSON-serializable unless a custom storage adapter is used.
- `localStorage` is synchronous and limited (~5 MB).
- Persisting sensitive data in `localStorage` is an XSS risk; prefer `HttpOnly` cookies for tokens.
- Hydration is asynchronous; components may render with initial state before rehydration completes.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Persisting a Theme with `partialize`

```ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface SettingsState {
  theme: 'light' | 'dark';
  fontSize: number;
  setTheme: (theme: 'light' | 'dark') => void;
  setFontSize: (size: number) => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'light',
      fontSize: 16,
      setTheme: (theme) => set({ theme }),
      setFontSize: (fontSize) => set({ fontSize }),
    }),
    {
      name: 'settings',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ theme: state.theme }),
    }
  )
);
```

**Expected Output:** Only `theme` is persisted; `fontSize` and actions are not. The theme is restored on reload.

**Why this output:** `partialize` returns the subset to persist. Actions and transient state are excluded.

#### Example 2: Versioned Persistence with Migration

```ts
const useStore = create<State>()(
  persist(
    (set) => ({ ... }),
    {
      name: 'app-store',
      version: 2,
      migrate: (persisted, version) => {
        if (version === 0) {
          return { ...persisted, newField: 'default' };
        }
        return persisted as State;
      },
    }
  )
);
```

**Expected Output:** Old persisted state (version 0) is migrated to the current shape by adding `newField`.

**Why this output:** `migrate` receives the persisted state and its version, and returns the upgraded state.

### Real-World Cases

- **Theme and preferences:** Persisting user settings.
- **Draft forms:** Saving form progress.
- **Wizard state:** Resuming multi-step flows.
- **Authentication:** Persisting non-sensitive session metadata.

### References

- Zustand — Persist middleware — https://zustand.docs.pmnd.rs/middlewares/persist
- Zustand — Persist: Hydration and async storage — https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Zustand — Persist: Migrations — https://zustand.docs.pmnd.rs/integrations/persisting-store-data#migrations

---

## 7. Immer Integration

### Definitions

**Core Definition:** Immer integration is the `immer` middleware that lets you write Zustand updates with mutable-looking syntax that is safely converted to immutable updates.

**Technical Definition:** `immer(stateCreator)` wraps the state creator so that `set` accepts a function that receives a draft state. Immer records mutations to the draft and produces a new immutable state. This simplifies deeply nested updates that would otherwise require manual spreading. The middleware is imported from `zustand/middleware/immer`. It does not change the store API; it only changes the semantics of the updater function passed to `set`.

**Beginner-Friendly Explanation:** Immer lets you write updates that look like direct mutations — `state.nested.count += 1` — while Zustand still produces a new immutable state behind the scenes.

### Purposes

- To simplify updates of deeply nested state.
- To avoid manual object spreading at every level.
- To make reducer-style logic readable.
- To preserve immutability guarantees without boilerplate.
- To reduce bugs from incomplete spreading.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { immer } from 'zustand/middleware/immer';

const useStore = create<State>()(
  immer((set) => ({
    nested: { count: 0 },
    increment: () => set((state) => { state.nested.count += 1; }),
  }))
);
```

**Syntax Rules:**

1. Wrap the state creator with `immer`.
2. The updater receives a draft; mutate it directly.
3. Do not return a value from the updater (Immer uses the mutations).
4. Do not mix Immer updaters with plain object returns in the same store without care.
5. Keep Immer as the innermost middleware when combined with `persist` and `devtools`.

**Constraints and Limitations:**

- Immer adds a small runtime overhead.
- Immer does not make async code immutable; only the draft mutations are captured.
- Large, frequently updated state can be slower with Immer than manual spreading.

### Annotated Code Examples

```ts
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface State {
  user: { profile: { name: string; age: number } };
  updateName: (name: string) => void;
  incrementAge: () => void;
}

export const useUserStore = create<State>()(
  immer((set) => ({
    user: { profile: { name: 'Ada', age: 30 } },
    updateName: (name) =>
      set((state) => {
        state.user.profile.name = name;
      }),
    incrementAge: () =>
      set((state) => {
        state.user.profile.age += 1;
      }),
  }))
);
```

**Expected Output:** `updateName('Grace')` and `incrementAge()` mutate the nested profile directly, and Zustand produces a new immutable state.

**Why this output:** The `immer` middleware wraps the updater in `produce`, so mutations are converted to immutable updates.

### Real-World Cases

- **Nested user profiles:** Updating nested fields.
- **Deeply nested settings:** Modifying a single leaf.
- **Normalized entities:** Updating an entity by ID.

### References

- Zustand — Immer middleware — https://zustand.docs.pmnd.rs/middlewares/immer
- Immer — Official Documentation — https://immerjs.github.io/immer/

---

## 8. Devtools Integration

### Definitions

**Core Definition:** DevTools integration is the `devtools` middleware that connects a Zustand store to the Redux DevTools Extension for action inspection and state tracking.

**Technical Definition:** `devtools(stateCreator, options)` wraps the store and sends every `set` call to the Redux DevTools Extension. Options include `name` (the store name shown in DevTools), `enabled` (whether to enable DevTools), `anonymousActionType` (default action name for anonymous updates), and `store` (a custom DevTools connection). Actions can be named by passing a third argument to `set`: `set(partial, false, 'actionName')`. In TypeScript, use `create<State>()(devtools(...))`.

**Beginner-Friendly Explanation:** The `devtools` middleware connects your Zustand store to the browser's Redux DevTools extension, so you can inspect every state change and debug like you would with Redux.

### Purposes

- To inspect every state transition in development.
- To name actions for readable DevTools history.
- To jump between states for debugging.
- To verify that updates are immutable.
- To keep debugging parity with Redux.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { devtools } from 'zustand/middleware';

const useStore = create<State>()(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((s) => ({ count: s.count + 1 }), false, 'counter/increment'),
    }),
    { name: 'CounterStore', enabled: process.env.NODE_ENV !== 'production' }
  )
);
```

**Component Breakdown:**

| Option / Argument | Description |
|--------------------|-------------|
| `name` | Store name in DevTools |
| `enabled` | Enable/disable DevTools |
| `anonymousActionType` | Default action name |
| `set(partial, replace, actionName)` | Named action |

**Syntax Rules:**

1. Wrap the state creator with `devtools`.
2. Pass a `name` for a readable store label.
3. Use `enabled` to disable DevTools in production.
4. Pass an action name as the third argument to `set` for readable history.
5. Compose `devtools` outside `persist` and `immer`.

**Constraints and Limitations:**

- DevTools requires the Redux DevTools Extension.
- Large state trees slow DevTools; disable it in production.
- Immer drafts and DevTools can conflict if action names are not provided.

### Annotated Code Examples

```ts
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

interface CounterState {
  count: number;
  increment: () => void;
}

export const useCounterStore = create<CounterState>()(
  devtools(
    (set) => ({
      count: 0,
      increment: () =>
        set((state) => ({ count: state.count + 1 }), false, 'counter/increment'),
    }),
    { name: 'CounterStore' }
  )
);
```

**Expected Output:** Every `increment` appears in DevTools as `counter/increment` with the previous and next state.

**Why this output:** `devtools` hooks into the store's `set` and forwards actions with the provided name.

### Real-World Cases

- **Debugging:** Inspecting state transitions in development.
- **Team consistency:** Shared action naming conventions.
- **Testing:** Replaying actions in DevTools.

### References

- Zustand — Devtools middleware — https://zustand.docs.pmnd.rs/middlewares/devtools
- Redux DevTools — GitHub — https://github.com/reduxjs/redux-devtools

---

## 9. Zustand vs Redux

### Definitions

**Core Definition:** Zustand and Redux are both client-state libraries, but Zustand is minimal and unopinionated while Redux is structured, explicit, and ecosystem-rich.

**Technical Definition:** Zustand is a small (~1.2 KB) hooks-first library with no providers, no action types, and no reducers. It provides selectors, middleware, and a vanilla store API. Redux (with Redux Toolkit) is a larger, opinionated ecosystem with explicit actions, reducers, middleware, time-travel DevTools, and a large ecosystem of extensions. Zustand is closer to a "global `useState`" with selectors, while Redux is a full application architecture. Both are production-grade; the choice depends on team size, debugging needs, and the amount of shared client state.

**Beginner-Friendly Explanation:** Zustand is the small, simple option — perfect for most apps. Redux is the big, structured option — better for large teams that want strict conventions and powerful debugging.

### Purposes

- To choose the library that matches the project's complexity and team.
- To understand the trade-offs in boilerplate, bundle size, and debugging.
- To plan migrations between the two.
- To avoid using a full architecture for a small amount of shared state.

### Comparison Table

| Dimension | Zustand | Redux (Toolkit) |
|-----------|---------|-----------------|
| **Bundle size** | ~1.2 KB | ~13 KB |
| **Providers** | None required | `<Provider store={store}>` |
| **Actions** | Plain functions | Action creators / slices |
| **Reducers** | Optional (`set` updaters) | Required |
| **Boilerplate** | Minimal | Moderate (even with RTK) |
| **DevTools** | Via middleware | Built-in |
| **Time travel** | Limited | Full |
| **Selectors** | Built-in hooks | `useSelector` + Reselect |
| **Async** | Async actions in store | `createAsyncThunk` / RTK Query |
| **Best for** | Small–medium client state | Large teams, complex state, strict conventions |

**Syntax Rules:**

1. Zustand is a good default for client state in small and medium apps.
2. Redux Toolkit is justified when the team benefits from strict conventions, DevTools time-travel, or a large existing ecosystem.
3. Neither replaces server-state libraries; both are for client state.
4. Zustand can coexist with Redux during a migration.
5. Zustand's minimalism is a feature, not a limitation.

### Annotated Code Examples

**Zustand (minimal):**

```ts
const useStore = create<State>()((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
}));
```

**Redux Toolkit (structured):**

```ts
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: { increment: (state) => { state.value += 1; } },
});
```

**Expected Output:** Both update a counter. Zustand requires less code; Redux provides a formal action/reducer structure.

**Why this output:** Zustand merges state and actions into one function. Redux separates them into slices with generated action types.

### Real-World Cases

- **Small apps:** Zustand for shared client state.
- **Large teams:** Redux Toolkit for strict conventions.
- **Migration:** Zustand can be adopted incrementally alongside Redux.
- **Server state:** Neither; use TanStack Query.

### References

- Zustand — Comparison — https://zustand.docs.pmnd.rs/getting-started/comparison
- Redux — Redux Toolkit Overview — https://redux.js.org/redux-toolkit/overview
- PkgPulse — Zustand vs Redux 2026 — https://www.pkgpulse.com

---

## 10. Zustand Best Practices

### Definitions

**Core Definition:** Zustand best practices are the guidelines for structuring stores, selectors, actions, middleware, and persistence to build maintainable and performant applications.

**Technical Definition:** Best practices combine **selector discipline** (always select the smallest slice; use `useShallow` for object/array selectors), **store organisation** (one store per feature, or one store with slices for large apps), **action colocation** (define actions in the store, not in components), **middleware composition** (persist, immer, devtools in a predictable order), **persistence hygiene** (partialize, version, migrate), **SSR awareness** (per-request stores via `createStore`), and **separation of concerns** (Zustand for client state, TanStack Query for server state).

**Beginner-Friendly Explanation:** Good Zustand usage means selecting only what you need, keeping actions in the store, using middleware deliberately, and not storing server data in Zustand.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Always use selectors** | v5 requires them; select the smallest slice |
| **Use `useShallow` for objects/arrays** | Prevents infinite re-renders |
| **Define actions in the store** | Colocate logic with state |
| **Use the slices pattern for large stores** | Compose typed slices |
| **Compose middleware in the right order** | `devtools(persist(immer(...)))` |
| **Use `partialize` for persistence** | Exclude sensitive and non-serializable state |
| **Version and migrate persisted state** | Handle schema changes |
| **Use `createStore` for SSR** | Per-request stores via Context |
| **Use `subscribeWithSelector` for transient updates** | Avoid re-renders for animations and scroll |
| **Keep server state out of Zustand** | Use TanStack Query or SWR |
| **Disable DevTools in production** | Avoid overhead |
| **Test stores with the vanilla API** | `getState`, `setState`, `subscribe` |

### Annotated Code Examples

**✅ Good Practices:**

```ts
// Feature store with slices pattern
interface BearSlice {
  bears: number;
  addBear: () => void;
}

interface FishSlice {
  fishes: number;
  addFish: () => void;
}

type Store = BearSlice & FishSlice;

const useStore = create<Store>()(
  devtools(
    immer((set) => ({
      bears: 0,
      fishes: 0,
      addBear: () => set((s) => { s.bears += 1; }),
      addFish: () => set((s) => { s.fishes += 1; }),
    })),
    { name: 'ZooStore' }
  )
);

// Selector with useShallow
const { bears, fishes } = useStore(useShallow((s) => ({ bears: s.bears, fishes: s.fishes })));
```

**❌ Bad Practices:**

```ts
// ❌ No selector: re-renders on every change
const state = useStore();

// ❌ New object selector without useShallow: infinite loop
const { bears, fishes } = useStore((s) => ({ bears: s.bears, fishes: s.fishes }));

// ❌ Server state in Zustand
const useUserStore = create((set) => ({ user: null, fetchUser: async () => { ... } }));

// ❌ Persisting sensitive data
persist((set) => ({ password: '' }), { name: 'auth' });
```

**Expected Output:** The good practices produce stable selectors, organised stores, and safe persistence. The bad practices cause re-render storms, stale server data, and security risks.

**Why this output:** No selector subscribes to the whole state; a new-object selector without `useShallow` re-renders infinitely; server state in Zustand duplicates TanStack Query's job; persisting passwords is an XSS risk.

### Real-World Cases

- **All Zustand applications:** These practices apply universally.
- **Large apps:** Slices pattern and typed stores.
- **SSR apps:** Per-request stores via Context.
- **Design systems:** Theme and UI state in Zustand.

### References

- Zustand — TypeScript Guide — https://zustand.docs.pmnd.rs/guides/typescript
- Zustand — Slices Pattern — https://zustand.docs.pmnd.rs/guides/slices-pattern
- Zustand — Prevent rerenders with useShallow — https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow
- Zustand — Practice with no store actions — https://zustand.docs.pmnd.rs/guides/practice-with-no-store-actions
- Zustand — Setup with Next.js — https://zustand.docs.pmnd.rs/guides/nextjs

---

## References

- Zustand — Official Documentation — https://zustand.docs.pmnd.rs/
- Zustand — GitHub — https://github.com/pmndrs/zustand
- Zustand — Getting Started — https://zustand.docs.pmnd.rs/getting-started/introduction
- Zustand — Comparison — https://zustand.docs.pmnd.rs/getting-started/comparison
- Zustand — TypeScript Guide — https://zustand.docs.pmnd.rs/guides/typescript
- Zustand — Practice with no store actions — https://zustand.docs.pmnd.rs/guides/practice-with-no-store-actions
- Zustand — Slices Pattern — https://zustand.docs.pmnd.rs/guides/slices-pattern
- Zustand — Prevent rerenders with useShallow — https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow
- Zustand — Setup with Next.js — https://zustand.docs.pmnd.rs/guides/nextjs
- Zustand — Testing — https://zustand.docs.pmnd.rs/guides/testing
- Zustand — create — https://zustand.docs.pmnd.rs/apis/create
- Zustand — createStore (vanilla) — https://zustand.docs.pmnd.rs/apis/create-store
- Zustand — Persist middleware — https://zustand.docs.pmnd.rs/middlewares/persist
- Zustand — Persisting store data — https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- Zustand — Devtools middleware — https://zustand.docs.pmnd.rs/middlewares/devtools
- Zustand — Immer middleware — https://zustand.docs.pmnd.rs/middlewares/immer
- Zustand — Subscribe with selector middleware — https://zustand.docs.pmnd.rs/middlewares/subscribe-with-selector
- Immer — Official Documentation — https://immerjs.github.io/immer/
- Redux — Redux Toolkit Overview — https://redux.js.org/redux-toolkit/overview
- Redux DevTools — GitHub — https://github.com/reduxjs/redux-devtools
- virastack academy — When you need a store — https://github.com/virastack/academy/blob/4e06ca19c061fe7ceb7fd1e93ae7e872f2a2098d/content/en/zustand/when-you-need-a-store.md
- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering/blob/61758f71713570475d12d0b36d24219bb84cf644/docs/03-application-architecture/state-management/categories-of-state.md
- PkgPulse — Zustand vs Redux 2026 — https://www.pkgpulse.com