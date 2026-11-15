# Recoil — Comprehensive Programming Cheat Sheet

> **⚠️ Critical Status Notice:** Recoil was archived by Meta in 2025 and is no longer maintained by its original creators. It **does not work with React 19** and will not receive bug fixes or security updates. Two paths exist for existing Recoil codebases: (1) the community-maintained fork `recoil-community` (React 18/19 compatible), or (2) migration to **Jotai**, the actively maintained atomic state library. This cheat sheet documents Recoil's concepts for maintenance and migration purposes, but **new projects should not adopt Recoil**.


## Topic Overview

### Definitions

**Core Definition:** Recoil is an atomic state management library for React that splits shared state into small, independent units called atoms and composes derived state through selectors.

**Technical Definition:** Recoil is a state management library for React created by Facebook/Meta that provides a simpler, more ergonomic alternative to Redux. It introduces two core concepts: **Atoms** — independent pieces of state that components can subscribe to — and **Selectors** — derived state computed from atoms or other selectors. Recoil's architecture is a graph of small state nodes rather than a single monolithic store. Components subscribe to exactly the atoms they need, and only dependent nodes recompute or re-render when inputs change. Recoil was archived in 2025; it is incompatible with React 19, and the community fork `recoil-community` provides React 18/19 compatibility, TypeScript 5.x support, and active maintenance. Jotai is the recommended migration target for new development.

**Beginner-Friendly Explanation:** Recoil breaks your app's shared state into tiny pieces called atoms — like individual cells in a spreadsheet. Selectors are like formulas that reference those cells: when a cell changes, only the formulas that depend on it recalculate. Components read and write atoms with hooks, and they re-render only when the atoms they use change. The important context for 2026 is that Recoil is no longer maintained by Meta and does not work with React 19. If you are maintaining an existing Recoil codebase, you can use the community fork, or you can migrate to Jotai, which is actively maintained and follows the same atomic philosophy.

### Key Characteristics

- **Atomic graph, not a single store:** State is a graph of small nodes (atoms and selectors) rather than one monolithic tree.
- **Fine-grained subscriptions:** Components subscribe to specific atoms and re-render only when those atoms change.
- **Derived state via selectors:** Selectors compute values from atoms or other selectors, with automatic dependency tracking.
- **Parameterised state via families:** `atomFamily` and `selectorFamily` create keyed instances for collections of similar state.
- **Async selectors:** Selectors can return Promises and integrate with React Suspense.
- **Archived status:** The original library is unmaintained and React 19-incompatible.
- **Community fork:** `recoil-community` provides React 18/19 compatibility and active maintenance.
- **Migration path:** Jotai is the recommended target for new atomic state projects.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`).
- JavaScript functions, closures, and Promises.
- Understanding of React Suspense (for async selectors).
- Basic knowledge of state management categories (client, server, URL, form).

### Related Programming Areas

- **State Management** — Jotai (migration target), Zustand, Redux, Context API.
- **Atomic Architecture** — graph of small state nodes.
- **Async State** — Promises, Suspense, `Loadable`.
- **Feature-Sliced Design** — Recoil's atomic model aligns naturally with slice ownership.

### Core Concepts / Features


## 1. Recoil

### Definitions

**Core Definition:** Recoil is an atomic state management library for React that composes shared state from small, independent atoms and derived selectors.

**Technical Definition:** Recoil provides `atom`, `selector`, `atomFamily`, and `selectorFamily` for defining state, and hooks (`useRecoilState`, `useRecoilValue`, `useSetRecoilState`, `useRecoilCallback`) for consuming it. Atoms are the smallest units of state; selectors are pure functions that derive values from atoms or other selectors. Components subscribe to exactly what they read. The `RecoilRoot` component provides the context in which atoms are stored. Recoil's atomic model addresses the scaling problem of a single global store by making shared state granular by default: instead of one big state tree, you get a graph of small nodes that can be owned by features and entities.

**Beginner-Friendly Explanation:** Recoil is a state manager built on atoms and selectors. You create atoms for your state pieces, selectors for computed values, and hooks to read and write them in components.

### Purposes

- To provide fine-grained, atomic state without a single monolithic store.
- To compose derived state declaratively through selectors.
- To minimise re-renders by subscribing components to exact state slices.
- To manage parameterised state for collections with families.
- To integrate async data with React Suspense.
- (Migration context) To understand the atomic model that Jotai continues.

### Syntax Rules and Structure

**General Syntax:**

```jsx
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil-community';

const countAtom = atom({ key: 'count', default: 0 });
const labelSelector = selector({
  key: 'label',
  get: ({ get }) => `${get(countAtom)} items`,
});

function Counter() {
  const [count, setCount] = useRecoilState(countAtom);
  const label = useRecoilValue(labelSelector);
  return <button onClick={() => setCount(c => c + 1)}>{label}</button>;
}
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `atom({ key, default })` | Primitive state unit |
| `selector({ key, get, set? })` | Derived state unit |
| `useRecoilState(atom)` | `[value, setValue]` |
| `useRecoilValue(selector)` | Read-only value |
| `useSetRecoilState(atom)` | Setter only |
| `RecoilRoot` | Provides the store context |

**Syntax Rules:**

1. Every atom and selector requires a globally unique `key` string.
2. Selectors are pure; `get` reads atoms or other selectors.
3. Async selectors return a Promise; consumers may suspend.
4. `RecoilRoot` must wrap the component tree that uses Recoil.
5. Keys must be unique across the entire application.

**Constraints and Limitations:**

- The original `recoil` package is archived and React 19-incompatible.
- Keys are globally scoped; collisions cause subtle bugs.
- Recoil does not have the ecosystem breadth of Redux or the active maintenance of Jotai.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Atom and Selector

```jsx
import { atom, selector, useRecoilState, useRecoilValue } from 'recoil-community';

const countAtom = atom({ key: 'count', default: 0 });
const doubledSelector = selector({
  key: 'doubled',
  get: ({ get }) => get(countAtom) * 2,
});

function Counter() {
  const [count, setCount] = useRecoilState(countAtom);
  const doubled = useRecoilValue(doubledSelector);
  return (
    <div>
      <p>Count: {count}, Doubled: {doubled}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

**Expected Output:** Count starts at 0, doubled at 0. Clicking +1 updates both.

**Why this output:** `doubledSelector` derives from `countAtom`. When `countAtom` changes, the selector recomputes and the component re-renders.

#### Example 2: Wrapping with `RecoilRoot`

```jsx
import { RecoilRoot } from 'recoil-community';

function App() {
  return (
    <RecoilRoot>
      <Counter />
    </RecoilRoot>
  );
}
```

**Expected Output:** The `Counter` component can read and write Recoil state.

**Why this output:** `RecoilRoot` provides the store context; Recoil hooks throw without it.

### Real-World Cases

- **Legacy codebases:** Maintaining existing Recoil applications while planning migration.
- **Understanding atomic state:** Learning the model that Jotai continues.
- **Feature-Sliced Design:** Aligning atom ownership with feature boundaries.

### References

- Recoil Community — npm — https://www.npmjs.com/package/recoil-community
- Recoil — Official Documentation (archived) — https://recoiljs.org
- Feature-Sliced Design — Recoil's Atomic Model — https://feature-sliced.design/blog/recoil-atomic-state-guide


## 2. Atoms

### Definitions

**Core Definition:** An atom is the smallest unit of Recoil state — an independent, subscribable piece of state with a default value and a globally unique key.

**Technical Definition:** `atom({ key, default })` creates a primitive state unit. The `key` is a globally unique string used internally to identify the atom; the `default` is its initial value. Components subscribe to atoms via hooks (`useRecoilState`, `useRecoilValue`, `useSetRecoilState`). Atoms can hold any serializable or non-serializable value, including objects, arrays, and Promises (for async default values). Atoms are writable by default; a `dangerouslyAllowMutability` flag is available but discouraged. Recoil caches atom values and shares them across all subscribers.

**Beginner-Friendly Explanation:** An atom is a piece of state with a name and a starting value. Components can read it, write to it, and subscribe to changes.

### Purposes

- To define primitive units of shared state.
- To provide a subscribable value with a unique identity.
- To enable fine-grained re-renders through exact subscriptions.
- To serve as the source nodes for selectors.
- To hold async default values (via Promises) when needed.

### Syntax Rules and Structure

**Primitive atom:**

```jsx
const countAtom = atom({ key: 'count', default: 0 });
const userAtom = atom({ key: 'user', default: null });
```

**Object atom:**

```jsx
const todosAtom = atom({
  key: 'todos',
  default: [{ id: 1, text: 'Learn Recoil', done: false }],
});
```

**Async default atom:**

```jsx
const userAtom = atom({
  key: 'user',
  default: fetch('/api/user').then(r => r.json()),
});
```

**Syntax Rules:**

1. Every atom requires a unique `key`.
2. `default` can be any value, including a Promise.
3. Atoms are writable with `useRecoilState` or `useSetRecoilState`.
4. Keys must be unique across the entire app.
5. Prefer immutable updates for object and array atoms.

**Constraints and Limitations:**

- Keys are globally scoped; collisions are silent and dangerous.
- Mutating atom values directly breaks change detection.
- Atoms cannot be created dynamically without `atomFamily`.

### Annotated Code Examples

```jsx
import { atom, useRecoilState } from 'recoil-community';

const countAtom = atom({ key: 'count', default: 0 });

function Counter() {
  const [count, setCount] = useRecoilState(countAtom);
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

**Expected Output:** The button displays and increments the count.

**Why this output:** `useRecoilState` subscribes the component to `countAtom` and returns the current value and a setter.

### Real-World Cases

- **UI toggles:** Modal open state, sidebar visibility.
- **User session:** Current user object.
- **Form drafts:** Multi-step form data.
- **Filters:** Active filter values.

### References

- Recoil — Atom API — https://recoiljs.org/docs/api-reference/core/atom
- Recoil Community — README — https://www.npmjs.com/package/recoil-community


## 3. Selectors

### Definitions

**Core Definition:** A selector is a pure function that derives a value from atoms or other selectors, recomputing only when its dependencies change.

**Technical Definition:** `selector({ key, get, set? })` creates a derived state unit. The `get` function receives `{ get }` and returns a value computed from other atoms or selectors. Recoil tracks which atoms/selectors are read and recomputes the selector when any dependency changes. The optional `set` function makes the selector writable, receiving `{ get, set }, newValue` and allowing writes to upstream atoms. Selectors are cached: the computation runs only when dependencies change, not on every render.

**Beginner-Friendly Explanation:** A selector is a formula. It reads atoms and computes a value. When the atoms it reads change, the formula recalculates automatically.

### Purposes

- To derive computed state without storing redundant data.
- To compose state declaratively.
- To enable write-through updates to upstream atoms with `set`.
- To minimise re-renders by caching derived values.
- To keep a single source of truth for each fact.

### Syntax Rules and Structure

**Read-only selector:**

```jsx
const doubledSelector = selector({
  key: 'doubled',
  get: ({ get }) => get(countAtom) * 2,
});
```

**Writable selector (write-through):**

```jsx
const celsiusSelector = selector({
  key: 'celsius',
  get: ({ get }) => get(fahrenheitAtom),
  set: ({ set }, newValue) => {
    set(fahrenheitAtom, newValue);
  },
});
```

**Syntax Rules:**

1. Selectors require a unique `key`.
2. `get` must be pure; do not perform side effects.
3. `get` can read other selectors, enabling composition.
4. `set` can write to atoms or other selectors.
5. Selectors are cached until dependencies change.

**Constraints and Limitations:**

- Selectors are read-only unless a `set` function is provided.
- Selectors cannot have async `set` functions (known limitation).
- Overly long selector chains are harder to debug.

### Annotated Code Examples

```jsx
const priceAtom = atom({ key: 'price', default: 10 });
const quantityAtom = atom({ key: 'quantity', default: 2 });

const totalSelector = selector({
  key: 'total',
  get: ({ get }) => get(priceAtom) * get(quantityAtom),
});
```

**Expected Output:** `totalSelector` returns 20 and recomputes when `priceAtom` or `quantityAtom` changes.

**Why this output:** The selector tracks its dependencies and caches the result until one changes.

### Real-World Cases

- **Cart totals:** Subtotal and tax.
- **Filtered lists:** Filtering a list by an active filter atom.
- **Formatted values:** Currency and dates.
- **Validation state:** Deriving `isValid` from fields.

### References

- Recoil — Selector API — https://recoiljs.org/docs/api-reference/core/selector
- Feature-Sliced Design — Recoil's Atomic Model (selectors) — https://feature-sliced.design/blog/recoil-atomic-state-guide


## 4. Atom Families

### Definitions

**Core Definition:** `atomFamily` creates parameterised atoms — a function that returns an atom for a given parameter, caching each instance by that parameter.

**Technical Definition:** `atomFamily({ key, default })` returns a function `(param) => RecoilState<T>`. Calling the family with a parameter returns the cached atom for that parameter or creates a new atom with the default value for that parameter. `default` can be a static value, a function of the parameter, or a `selectorFamily` for dynamic async defaults. Recoil caches atomFamily instances by parameter, so repeated calls with the same parameter return the same atom. This is used for managing collections of similar state (todos, users, posts) without manually creating separate atoms.

**Beginner-Friendly Explanation:** An atom family creates atoms dynamically based on an ID. `todoFamily(1)` returns the atom for todo 1; calling it again returns the same atom. This is useful when you have many similar items.

### Purposes

- To manage parameterised state for collections.
- To avoid manually creating separate atoms for each item.
- To cache per-item atoms so repeated access returns the same instance.
- To combine with `selectorFamily` for dynamic async defaults.
- To enable per-item re-render isolation.

### Syntax Rules and Structure

**Static default:**

```jsx
const todoFamily = atomFamily({
  key: 'todoFamily',
  default: { text: '', done: false },
});
```

**Dynamic default via `selectorFamily`:**

```jsx
const todoFamily = atomFamily({
  key: 'todoFamily',
  default: selectorFamily({
    key: 'todoDefault',
    get: (id) => async () => {
      const res = await fetch(`/api/todos/${id}`);
      return res.json();
    },
  }),
});
```

**Syntax Rules:**

1. `atomFamily` requires a unique `key`.
2. `default` can be a static value, a function, or a `selectorFamily`.
3. Each parameter gets its own cached atom.
4. `selectorFamily` allows access to the parameter in the default computation.
5. Families hold every atom they create until explicitly released.

**Constraints and Limitations:**

- Atom families can leak memory if instances are never released.
- Parameter values must be stable; new objects on every call create new atoms.
- Dynamic defaults via `selectorFamily` are the correct pattern for async per-item data.

### Annotated Code Examples

```jsx
const todosAtomFamily = atomFamily({
  key: 'todosAtomFamily',
  default: selectorFamily({
    key: 'todosSelectorFamily',
    get: (id) => async () => {
      const res = await fetch(`https://dummyjson.com/todos/${id}`);
      return res.json();
    },
  }),
});

function Todo({ id }) {
  const todo = useRecoilValue(todosAtomFamily(id));
  return <p>{todo.todo}</p>;
}
```

**Expected Output:** Each `<Todo id={1} />` and `<Todo id={2} />` fetches and caches its own todo. Multiple components with the same ID share the cached data.

**Why this output:** `atomFamily(id)` returns a per-ID atom whose default is an async selector that fetches that ID's data. Recoil caches the atom and its value.

### Real-World Cases

- **Todo lists:** Per-todo editing state.
- **User profiles:** Per-user data atoms.
- **Chat messages:** Per-message state.
- **Data grids:** Per-row state.

### References

- Recoil — atomFamily API — https://recoiljs.org/docs/api-reference/utils/atomFamily
- DEV Community — Harnessing atomFamily and selectorFamily — https://dev.to/safal_bhandari/harnessing-atomfamily-and-selectorfamily-in-recoil-for-dynamic-async-state-2cob
- Stack Overflow — atomFamily and selectorFamily for async data — https://stackoverflow.com


## 5. Async Selectors

### Definitions

**Core Definition:** An async selector is a selector whose `get` function returns a Promise, enabling asynchronous data fetching that integrates with React Suspense.

**Technical Definition:** A selector becomes async when its `get` function returns a Promise. When an async selector is read, Recoil suspends the component until the Promise resolves (unless the component uses `useRecoilValueLoadable` or `useRecoilStateLoadable`, which return a `Loadable` object with `state: 'loading' | 'hasValue' | 'hasError'`). Async selectors can depend on other atoms and selectors; when their dependencies change, they re-fetch. They are commonly used with `atomFamily` + `selectorFamily` for per-item async data. Async selector `set` functions are **not** supported; use `useRecoilCallback` for async writes across multiple atoms.

**Beginner-Friendly Explanation:** An async selector fetches data and suspends the component until the data is ready. You wrap the component in `<Suspense>` to show a fallback, or use `Loadable` hooks for manual state handling.

### Purposes

- To fetch data inside the Recoil graph, colocated with state.
- To integrate async data with React Suspense.
- To compose async and synchronous state in selectors.
- To provide loading and error states via `Loadable` hooks.
- To cache async results and re-fetch only when dependencies change.

### Syntax Rules and Structure

**Async selector (Suspense):**

```jsx
const userSelector = selector({
  key: 'user',
  get: async ({ get }) => {
    const id = get(userIdAtom);
    const res = await fetch(`/api/users/${id}`);
    return res.json();
  },
});
```

**Loadable consumption:**

```jsx
const userLoadable = useRecoilValueLoadable(userSelector);
// userLoadable.state: 'loading' | 'hasValue' | 'hasError'
```

**Async selector with parameter (`selectorFamily`):**

```jsx
const todoSelector = selectorFamily({
  key: 'todo',
  get: (id) => async () => {
    const res = await fetch(`/api/todos/${id}`);
    return res.json();
  },
});
```

**Syntax Rules:**

1. Returning a Promise from `get` makes the selector async.
2. Suspense consumers must be wrapped in `<Suspense>` and `<RecoilRoot>`.
3. `useRecoilValueLoadable` and `useRecoilStateLoadable` avoid Suspense.
4. Async `set` is not supported; use `useRecoilCallback` for async writes.
5. Async selectors cache their resolved value until dependencies change.

**Constraints and Limitations:**

- Async selectors without a Suspense boundary crash the app.
- Async `set` functions are a known limitation (open issue).
- Race conditions can occur if dependencies change during a fetch; use request identity or cancellation patterns.

### Annotated Code Examples

```jsx
const userIdAtom = atom({ key: 'userId', default: 1 });

const userSelector = selector({
  key: 'user',
  get: async ({ get }) => {
    const id = get(userIdAtom);
    const res = await fetch(`/api/users/${id}`);
    return res.json();
  },
});

function UserProfile() {
  const user = useRecoilValue(userSelector); // Suspends
  return <h1>{user.name}</h1>;
}

function App() {
  return (
    <RecoilRoot>
      <Suspense fallback={<p>Loading…</p>}>
        <UserProfile />
      </Suspense>
    </RecoilRoot>
  );
}
```

**Expected Output:** `UserProfile` shows a loading fallback until the user is fetched, then renders the name.

**Why this output:** The async selector returns a Promise; `useRecoilValue` suspends the component until it resolves.

### Real-World Cases

- **API data fetching:** Users, posts, products.
- **Async derived state:** Combining remote data with local state.
- **Parameterised async data:** Per-ID fetches with `selectorFamily`.

### References

- Recoil — Asynchronous Data Queries — https://recoiljs.org/docs/guides/asynchronous-data-queries
- DEV Community — Handling Async State with Loadable Hooks — https://dev.to
- GitHub Issue — Async selector set function — https://github.com/facebookexperimental/Recoil/issues/762


## 6. Recoil Best Practices

### Definitions

**Core Definition:** Recoil best practices are the guidelines for structuring atoms, selectors, families, and async data to build maintainable applications — with the overriding caveat that Recoil is archived and migration should be planned.

**Technical Definition:** Best practices for Recoil combine **unique, namespaced keys** (prevent collisions), **atomic granularity** (small, focused atoms rather than large objects), **derived state via selectors** (no stored redundancy), **families for collections** (`atomFamily` + `selectorFamily` instead of dynamic atom creation), **Suspense boundaries for async selectors** (or `Loadable` hooks), **`useRecoilCallback` for async writes across atoms**, and **Feature-Sliced Design alignment** (atoms owned by feature slices). The 2026 meta-practice is **migration planning**: assess whether `recoil-community` or Jotai is the right target, and avoid deepening investment in the archived original.

**Beginner-Friendly Explanation:** Good Recoil usage means using clear key names, keeping atoms small, using selectors for computed values, using families for lists, and wrapping async data in Suspense. But the most important practice is planning your migration away from the archived original library.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use unique, namespaced keys** | Prefix keys by feature (e.g., `'todos/add'`) |
| **Keep atoms small and focused** | One atom per state concern |
| **Derive with selectors** | Never store computed values |
| **Use `atomFamily` for collections** | Avoid dynamic atom creation |
| **Combine `atomFamily` + `selectorFamily`** | For parameterised async defaults |
| **Wrap async selectors in Suspense** | Or use `Loadable` hooks |
| **Use `useRecoilCallback` for async writes** | Async `set` is not supported |
| **Align atoms with feature slices** | Use Feature-Sliced Design boundaries |
| **Plan migration** | Evaluate `recoil-community` vs Jotai |
| **Do not start new projects on Recoil** | Use Jotai or Zustand instead |

### Annotated Code Examples

**✅ Good Practices:**

```jsx
// Namespaced keys
const todosAtom = atom({ key: 'todos/list', default: [] });
const filterAtom = atom({ key: 'todos/filter', default: 'all' });

// Derived selector
const filteredTodosSelector = selector({
  key: 'todos/filtered',
  get: ({ get }) => {
    const todos = get(todosAtom);
    const filter = get(filterAtom);
    if (filter === 'done') return todos.filter(t => t.done);
    return todos;
  },
});

// Parameterised async data
const todoFamily = atomFamily({
  key: 'todos/item',
  default: selectorFamily({
    key: 'todos/itemDefault',
    get: (id) => async () => (await fetch(`/api/todos/${id}`)).json(),
  }),
});
```

**❌ Bad Practices:**

```jsx
// ❌ Non-namespaced key (collision risk)
const atom1 = atom({ key: 'data', default: [] });

// ❌ Storing derived state
const totalAtom = atom({ key: 'total', default: 0 });

// ❌ Async set (not supported)
const badSelector = selector({
  key: 'bad',
  set: async ({ set }, value) => { /* not supported */ },
});

// ❌ Dynamic atom creation in component
function Bad() {
  const dynamicAtom = atom({ key: `dyn-${Math.random()}`, default: 0 });
}
```

**Expected Output:** The good practices produce collision-free, composable, maintainable state. The bad practices cause key collisions, redundant state, unsupported patterns, and memory leaks.

**Why this output:** Namespaced keys prevent silent collisions; derived selectors stay in sync; async `set` is a known limitation; dynamic atoms in components create unreleased instances.

### Real-World Cases

- **Feature-Sliced Design projects:** Align atoms with slice ownership.
- **Migration planning:** Audit atom keys and selector dependencies to plan a Jotai migration.
- **Legacy maintenance:** Apply best practices to reduce technical debt before migration.

### References

- Feature-Sliced Design — Recoil's Atomic Model — https://feature-sliced.design/blog/recoil-atomic-state-guide
- Recoil Community — npm — https://www.npmjs.com/package/recoil-community
- Jotai — Migration from Recoil — https://jotai.org/docs/guides/migrating-from-recoil


## References

- Recoil Community — npm (community fork, React 18/19 compatible) — https://www.npmjs.com/package/recoil-community
- Recoil — Official Documentation (archived) — https://recoiljs.org
- Recoil — Atom API — https://recoiljs.org/docs/api-reference/core/atom
- Recoil — Selector API — https://recoiljs.org/docs/api-reference/core/selector
- Recoil — atomFamily API — https://recoiljs.org/docs/api-reference/utils/atomFamily
- Recoil — Asynchronous Data Queries — https://recoiljs.org/docs/guides/asynchronous-data-queries
- Feature-Sliced Design — Recoil's Atomic Model — https://feature-sliced.design/blog/recoil-atomic-state-guide
- DEV Community — Harnessing atomFamily and selectorFamily — https://dev.to/safal_bhandari/harnessing-atomfamily-and-selectorfamily-in-recoil-for-dynamic-async-state-2cob
- DEV Community — Handling Async State with Loadable Hooks — https://dev.to
- GitHub Issue — Async selector set function — https://github.com/facebookexperimental/Recoil/issues/762
- Stack Overflow — atomFamily and selectorFamily for async data — https://stackoverflow.com
- Jotai — Migration from Recoil — https://jotai.org/docs/guides/migrating-from-recoil