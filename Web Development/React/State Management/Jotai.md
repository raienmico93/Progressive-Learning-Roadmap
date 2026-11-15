# Jotai — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Jotai is a primitive and flexible atomic state management library for React where state is composed bottom-up from small, independent atoms rather than a single monolithic store.

**Technical Definition:** Jotai is an atomic state management library for React. Its core API exposes `atom`, `useAtom`, `Store`, and `Provider`. An **atom** is a small, isolated piece of state; a **derived atom** computes a value from other atoms; a **write-only atom** encapsulates actions; and an **async atom** integrates with React Suspense. Jotai provides automatic dependency tracking: when an atom's dependencies change, only the atoms and components that depend on it recompute and re-render. It offers a `jotai/utils` bundle with utilities such as `atomWithStorage`, `loadable`, `unwrap`, `splitAtom`, and `selectAtom`, plus extensions for React Query, XState, Immer, tRPC, and more. Jotai v3 continues the atomic model with a minimal core API (2 KB) and first-class TypeScript support.

**Beginner-Friendly Explanation:** Jotai is a state management library built on the idea of **atoms** — tiny pieces of state, like cells in a spreadsheet. Each atom holds one value. Derived atoms are like formulas that reference other cells: when a cell changes, only the formulas that depend on it recalculate. Components read and write atoms with hooks, and they re-render only when the atoms they use change. There is no giant store, no reducers, and no providers required. You build your state bottom-up from small pieces that compose naturally.

### Key Characteristics

- **Atomic, bottom-up state:** State is composed from small, independent atoms rather than a single store.
- **Automatic dependency tracking:** Derived atoms and components re-render only when their specific dependencies change.
- **Minimal core API:** `atom`, `useAtom`, `Store`, and `Provider` are the core exports; the core is ~2 KB.
- **First-class async support:** Async atoms trigger Suspense by default; `loadable` and `unwrap` provide non-Suspense alternatives.
- **TypeScript-oriented:** Type inference works through primitive and derived atoms; explicit typing is optional.
- **Utilities and extensions:** `atomWithStorage`, `splitAtom`, `selectAtom`, `atomFamily` (via `jotai-family`), React Query, XState, Immer, tRPC, and more.
- **Store and Provider:** Custom stores enable non-React access, test isolation, and multi-store architectures.
- **No provider required:** Jotai works without a Provider using a default global store; Providers are optional for isolation.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`).
- JavaScript functions, closures, and Promises.
- Understanding of React Suspense (for async atoms).
- TypeScript (recommended; Jotai's inference is a major benefit).
- Basic knowledge of state management categories (client, server, URL, form).

### Related Programming Areas

- **State Management** — Zustand, Redux, Recoil, Context API.
- **Server State** — TanStack Query, SWR (Jotai is for client state).
- **Persistence** — `atomWithStorage`, `localStorage`, `sessionStorage`.
- **Async State** — Suspense, `loadable`, `unwrap`.
- **Atom Collections** — `atomFamily`, `jotai-family`.

---

## 1. Jotai

### Definitions

**Core Definition:** Jotai is an atomic state management library for React that composes state from small, independent atoms with automatic dependency tracking.

**Technical Definition:** Jotai's core API exports `atom`, `useAtom`, `Store`, and `Provider`. `atom(initialValue)` creates a primitive atom. `atom((get) => expression)` creates a read-only derived atom. `atom(null, (get, set, arg) => void)` creates a write-only atom. `atom((get) => expression, (get, set, arg) => void)` creates a read-write atom. `useAtom(atom)` returns `[value, setValue]`. `useAtomValue(atom)` reads a value; `useSetAtom(atom)` returns a setter. `createStore()` creates a vanilla store for non-React usage, and `<Provider store={store}>` scopes atoms to a subtree. Jotai tracks dependencies at the atom level: when a primitive atom changes, only derived atoms that read it recompute, and only components that read those atoms re-render.

**Beginner-Friendly Explanation:** Jotai gives you a `atom` function to create state pieces and a `useAtom` hook to read and write them in components. You build your state from small atoms that reference each other, and Jotai figures out what needs to update.

### Purposes

- To provide fine-grained reactivity without manual memoization.
- To compose state bottom-up from small, reusable atoms.
- To track dependencies automatically so only affected components re-render.
- To integrate async state with React Suspense.
- To offer a minimal API that scales from a `useState` replacement to a large application.
- To enable code splitting and natural modularity through atom composition.

### Syntax Rules and Structure

**General Syntax:**

```ts
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);
const incrementAtom = atom(null, (get, set) => {
  set(countAtom, get(countAtom) + 1);
});
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `atom(initialValue)` | Primitive atom (readable and writable) |
| `atom((get) => expr)` | Read-only derived atom |
| `atom(null, (get, set, arg) => ...)` | Write-only atom |
| `atom((get) => expr, (get, set, arg) => ...)` | Read-write atom |
| `useAtom(atom)` | `[value, setValue]` |
| `useAtomValue(atom)` | Read-only value |
| `useSetAtom(atom)` | Setter function |
| `createStore()` | Vanilla store for non-React usage |
| `<Provider store={store}>` | Scopes atoms to a subtree |

**Syntax Rules:**

1. Define atoms at module level; never create atoms inside components.
2. Use primitive atoms for values, derived atoms for computed state, write-only atoms for actions.
3. Use `useAtomValue` for reads and `useSetAtom` for writes to avoid accidental subscriptions.
4. Wrap async atom consumers in a Suspense boundary, or use `loadable`/`unwrap`.
5. Use `jotai-family` instead of the deprecated `atomFamily` from `jotai/utils` for new code.

**Constraints and Limitations:**

- Creating atoms inside components breaks state persistence (a new atom every render).
- Async atoms suspend by default and require a Suspense boundary.
- `atomFamily` from `jotai/utils` is deprecated; use `jotai-family`.
- Jotai is for client state; server state belongs in TanStack Query or SWR.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Primitive and Derived Atoms

```ts
import { atom, useAtomValue, useSetAtom } from 'jotai';

const priceAtom = atom(10);
const quantityAtom = atom(2);
const subtotalAtom = atom((get) => get(priceAtom) * get(quantityAtom));
const taxAtom = atom((get) => get(subtotalAtom) * 0.1);
const totalAtom = atom((get) => get(subtotalAtom) + get(taxAtom));

function CartTotal() {
  const total = useAtomValue(totalAtom);
  const setQuantity = useSetAtom(quantityAtom);
  return (
    <div>
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={() => setQuantity((q) => q + 1)}>Add one</button>
    </div>
  );
}
```

**Expected Output:** The total is `$22.00` initially (`subtotal` 20 + `tax` 2). Clicking “Add one” increments `quantityAtom`, and `subtotal`, `tax`, and `total` recompute automatically.

**Why this output:** `subtotalAtom`, `taxAtom`, and `totalAtom` are derived atoms that track their dependencies. Changing `quantityAtom` updates only the derived atoms that read it and the component that reads `totalAtom`.

#### Example 2: Write-Only Atom for Actions

```ts
const countAtom = atom(0);
const itemsAtom = atom<string[]>([]);
const resetAllAtom = atom(null, (get, set) => {
  set(countAtom, 0);
  set(itemsAtom, []);
});

function ResetButton() {
  const resetAll = useSetAtom(resetAllAtom);
  return <button onClick={resetAll}>Reset all</button>;
}
```

**Expected Output:** Clicking “Reset all” resets both `countAtom` and `itemsAtom` atomically.

**Why this output:** The write-only atom encapsulates a multi-atom action. Components only need the action setter, not the underlying atoms.

### Real-World Cases

- **Shopping cart totals:** Derived atoms for subtotal, tax, and total.
- **UI state:** Sidebar open state, theme, active tab.
- **Multi-step forms:** Shared form state across steps.
- **Feature flags:** Individual atoms for each flag.

### References

- Jotai — Documentation — https://jotai.org/docs
- Jotai — Tutorial: Creating your first atom — https://tutorial.jotai.org
- agents-inc/skills — Jotai Quick Guide — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/SKILL.md

---

## 2. Atoms

### Definitions

**Core Definition:** An atom is the smallest unit of state in Jotai — an independent, isolated piece of state with a value and, optionally, a read and/or write function.

**Technical Definition:** An atom is created with `atom()`. A **primitive atom** (`atom(initialValue)`) is readable and writable; its write function is equivalent to `useState`'s setter. A **derived atom** (`atom(readFunction)`) is read-only and computes its value from other atoms via `get`. A **write-only atom** (`atom(null, writeFunction)`) has no readable value and encapsulates actions. A **read-write atom** (`atom(readFunction, writeFunction)`) both derives a value and accepts writes. Atoms are defined at module level and are referenced by identity.

**Beginner-Friendly Explanation:** An atom is a piece of state. You create it with `atom()`. A primitive atom holds a value; a derived atom computes a value from other atoms; a write-only atom performs actions.

### Purposes

- To define small, independent pieces of state.
- To compose state bottom-up from atoms.
- To enable fine-grained reactivity through atom-level dependency tracking.
- To encapsulate actions in write-only atoms.
- To provide granular read/write access with read-write atoms.
- To enable code splitting and modular state.

### Syntax Rules and Structure

**Primitive atom:**

```ts
const countAtom = atom(0);
const userAtom = atom<User | null>(null);
```

**Read-only derived atom:**

```ts
const doubledAtom = atom((get) => get(countAtom) * 2);
```

**Write-only atom:**

```ts
const incrementAtom = atom(null, (get, set) => {
  set(countAtom, get(countAtom) + 1);
});
```

**Read-write atom:**

```ts
const nameAtom = atom(
  (get) => get(userAtom)?.name ?? '',
  (get, set, newName: string) => {
    set(userAtom, { ...get(userAtom), name: newName });
  }
);
```

**Syntax Rules:**

1. Primitive atoms are always writable; their write is equivalent to `setState`.
2. Derived atoms are writable only if a write function is provided.
3. `get` reads other atoms; `set` writes to other atoms.
4. Define atoms at module level; never inside components.
5. Keep atoms small; one atom should hold one piece of state.

**Constraints and Limitations:**

- Primitive atoms can hold any type, but objects should be updated immutably.
- Derived atoms are cached until their dependencies change.
- Write-only atoms cannot be read.

### Annotated Code Examples

```ts
import { atom } from 'jotai';

// Primitive
export const countAtom = atom(0);

// Read-only derived
export const doubledAtom = atom((get) => get(countAtom) * 2);

// Write-only action
export const incrementAtom = atom(null, (get, set) => {
  set(countAtom, get(countAtom) + 1);
});

// Read-write lens
export const userNameAtom = atom(
  (get) => get(userAtom)?.name ?? '',
  (get, set, name: string) => {
    set(userAtom, { ...get(userAtom), name });
  }
);
```

**Expected Output:** `countAtom` holds a number; `doubledAtom` recomputes when `countAtom` changes; `incrementAtom` updates `countAtom`; `userNameAtom` reads and writes the user's name.

**Why this output:** Each atom has a specific role: primitive (state), derived (computation), write-only (action), read-write (lens).

### Real-World Cases

- **Primitive:** Counters, booleans, strings.
- **Derived:** Totals, filtered lists, formatted values.
- **Write-only:** Reset actions, multi-atom updates.
- **Read-write:** Object property lenses.

### References

- Jotai — Core: atom — https://jotai.org/docs/core/atom
- Jotai — Tutorial: Read-only atoms — https://tutorial.jotai.org
- Jotai — Tutorial: Write-only atoms — https://tutorial.jotai.org
- Jotai — Tutorial: Read-write atoms — https://tutorial.jotai.org

---

## 3. Derived Atoms

### Definitions

**Core Definition:** A derived atom is an atom whose value is computed from other atoms, updating automatically when its dependencies change.

**Technical Definition:** A derived atom is created with `atom((get) => expression)`. The `get` function reads other atoms; Jotai tracks which atoms are read and recomputes the derived atom when any dependency changes. Derived atoms are cached: the computation runs only when dependencies change, not on every render. Derived atoms can be chained (a derived atom can depend on another derived atom). Read-write derived atoms also accept a write function, enabling lens-like property access.

**Beginner-Friendly Explanation:** A derived atom is a formula. It reads other atoms and computes a value. When any atom it reads changes, the formula recalculates automatically.

### Purposes

- To compute derived values without storing redundant state.
- To chain computations in a composable, declarative way.
- To avoid manual memoization and dependency arrays.
- To keep a single source of truth for each fact.
- To provide granular read/write access with lens atoms.

### Syntax Rules and Structure

**Read-only derived atom:**

```ts
const fullNameAtom = atom((get) => `${get(firstNameAtom)} ${get(lastNameAtom)}`);
```

**Chained derived atoms:**

```ts
const subtotalAtom = atom((get) => get(priceAtom) * get(quantityAtom));
const taxAtom = atom((get) => get(subtotalAtom) * 0.1);
const totalAtom = atom((get) => get(subtotalAtom) + get(taxAtom));
```

**Read-write derived atom (lens):**

```ts
const nameAtom = atom(
  (get) => get(userAtom).name,
  (get, set, name: string) => {
    set(userAtom, { ...get(userAtom), name });
  }
);
```

**Syntax Rules:**

1. Derived atoms must be pure; do not perform side effects inside `get`.
2. Dependencies are tracked automatically; no dependency arrays are needed.
3. Derived atoms are cached until their dependencies change.
4. Chaining derived atoms is the idiomatic way to compose computations.
5. Write functions in read-write atoms should update the parent atom immutably.

**Constraints and Limitations:**

- Derived atoms cannot be written to unless a write function is provided.
- Excessive derivation chains can be harder to debug; keep them focused.
- Derived atoms do not replace `useMemo` for component-local computations.

### Annotated Code Examples

```ts
import { atom, useAtomValue } from 'jotai';

const priceAtom = atom(10);
const quantityAtom = atom(2);
const subtotalAtom = atom((get) => get(priceAtom) * get(quantityAtom));
const taxAtom = atom((get) => get(subtotalAtom) * 0.1);
const totalAtom = atom((get) => get(subtotalAtom) + get(taxAtom));

function Total() {
  const total = useAtomValue(totalAtom);
  return <p>${total.toFixed(2)}</p>;
}
```

**Expected Output:** The total starts at `$22.00` and updates when `priceAtom` or `quantityAtom` changes.

**Why this output:** `totalAtom` depends on `subtotalAtom` and `taxAtom`; both depend on `priceAtom` and `quantityAtom`. Jotai tracks the chain and recomputes only the affected atoms.

### Real-World Cases

- **Cart totals:** Subtotal, tax, and total.
- **Filtered lists:** Deriving a filtered list from a source list and a filter atom.
- **Formatted values:** Currency, dates, and names.
- **Validation state:** Deriving `isValid` from multiple field atoms.

### References

- Jotai — Tutorial: Read-only atoms — https://tutorial.jotai.org
- Jotai — Guide: Derive — https://jotai.org/docs/guides/derive
- Feature-Sliced Design — Jotai: The Power of Minimalist State Design — https://feature-sliced.design/blog/jotai-state-management

---

## 4. Async Atoms

### Definitions

**Core Definition:** An async atom is an atom whose read function returns a Promise, integrating asynchronous data with React Suspense.

**Technical Definition:** An async atom is created with `atom(async (get) => { ... })`. When read, it returns a Promise. Jotai suspends the component until the Promise resolves, so the consumer must be wrapped in a `<Suspense>` boundary. Async atoms can be made non-suspending with `loadable(atom)` (which returns a discriminated union of `loading`, `hasData`, and `hasError` states) or `unwrap(atom, fallback)` (which returns a fallback until the Promise resolves). Async write atoms (`atom(null, async (get, set, arg) => { ... })`) do **not** trigger Suspense; they are used for asynchronous actions.

**Beginner-Friendly Explanation:** An async atom fetches data and suspends the component until the data is ready. You wrap the component in `<Suspense>` to show a fallback. If you do not want Suspense, use `loadable` or `unwrap`.

### Purposes

- To integrate asynchronous data fetching with React Suspense.
- To compose async data with other atoms.
- To avoid manual loading state management with Suspense.
- To provide non-Suspense alternatives with `loadable` and `unwrap`.
- To handle async actions without triggering Suspense.

### Syntax Rules and Structure

**Async read atom (Suspense):**

```ts
const userAtom = atom(async (get) => {
  const id = get(userIdAtom);
  const res = await fetch(`/api/users/${id}`);
  return res.json();
});
```

**Non-Suspense with `loadable`:**

```ts
import { loadable } from 'jotai/utils';
const loadableUserAtom = loadable(userAtom);
// { state: 'loading' } | { state: 'hasData', data } | { state: 'hasError', error }
```

**Non-Suspense with `unwrap`:**

```ts
import { unwrap } from 'jotai/utils';
const unwrappedAtom = unwrap(userAtom, (prev) => prev ?? 'Loading...');
```

**Async write atom:**

```ts
const saveUserAtom = atom(null, async (get, set, user: User) => {
  await fetch('/api/users', { method: 'POST', body: JSON.stringify(user) });
  set(userAtom, user);
});
```

**Syntax Rules:**

1. Async read atoms suspend by default; wrap consumers in `<Suspense>`.
2. Use `loadable` for a discriminated union of states without Suspense.
3. Use `unwrap` for a fallback value without Suspense.
4. Async write atoms do **not** trigger Suspense.
5. Use `AbortController` for cancellable fetches inside async atoms.

**Constraints and Limitations:**

- Async atoms require a Suspense boundary; missing boundaries crash the app.
- `loadable` returns a discriminated union; always check `state` first.
- Async atoms are not automatically cancellable; use `AbortController`.

### Annotated Code Examples

```tsx
import { atom, useAtomValue } from 'jotai';
import { loadable } from 'jotai/utils';
import { Suspense } from 'react';

const userIdAtom = atom(1);
const userAtom = atom(async (get) => {
  const id = get(userIdAtom);
  const res = await fetch(`/api/users/${id}`);
  return res.json() as Promise<{ name: string }>;
});

const loadableUserAtom = loadable(userAtom);

function UserNameWithSuspense() {
  const user = useAtomValue(userAtom);
  return <span>{user.name}</span>;
}

function UserNameWithLoadable() {
  const user = useAtomValue(loadableUserAtom);
  if (user.state === 'loading') return <span>Loading…</span>;
  if (user.state === 'hasError') return <span>Error</span>;
  return <span>{user.data.name}</span>;
}

function App() {
  return (
    <Suspense fallback={<span>Loading…</span>}>
      <UserNameWithSuspense />
    </Suspense>
  );
}
```

**Expected Output:** `UserNameWithSuspense` suspends until the user is fetched; `UserNameWithLoadable` renders its own loading state without Suspense.

**Why this output:** Async atoms suspend by default; `loadable` provides a non-Suspense discriminated union.

### Real-World Cases

- **API data:** Fetching users, posts, and products.
- **Async actions:** Saving data and updating atoms.
- **Lazy loading:** Loading data on demand.

### References

- Jotai — Async Read Atoms — https://tutorial.jotai.org
- Jotai — Async Write Atoms — https://tutorial.jotai.org
- Jotai — Async utilities: loadable/unwrap — https://jotai.org/docs/utilities/async
- agents-inc/skills — Jotai Async Patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/examples/async.md

---

## 5. Atom Families

### Definitions

**Core Definition:** An atom family is a function that creates parameterised atoms on demand, caching each atom by its parameter so the same parameter returns the same atom.

**Technical Definition:** `atomFamily(initializeAtom, areEqual?)` returns a function `(param) => Atom`. When called with a parameter, it returns the cached atom for that parameter or creates a new one with `initializeAtom(param)`. The optional `areEqual` function determines whether two parameters are considered equal (default is `Object.is`). The `atomFamily` utility from `jotai/utils` is **deprecated**; the `jotai-family` package is the recommended replacement for new code, offering the same API plus additional utilities. Atom families hold every atom they create until told otherwise; for cleanup, use `remove()` or `setShouldRemove()`.

**Beginner-Friendly Explanation:** An atom family creates atoms dynamically based on a parameter. Calling `todoFamily('foo')` returns an atom for `'foo'`; calling it again returns the same atom. This is useful when you have many similar atoms keyed by an ID.

### Purposes

- To create parameterised atoms for collections of similar state.
- To cache atoms by parameter so the same parameter returns the same atom.
- To avoid creating a new atom every render for each item.
- To support per-item state (e.g., a todo's editing state).
- To enable per-item re-render isolation.

### Syntax Rules and Structure

**General Syntax (`jotai-family`):**

```ts
import { atom } from 'jotai';
import { atomFamily } from 'jotai-family';

const todoFamily = atomFamily((id: string) => atom({ id, text: '', done: false }));

const todoAtom = todoFamily('todo-1'); // Returns the cached atom for 'todo-1'
```

**With custom equality:**

```ts
import deepEqual from 'fast-deep-equal';
const itemFamily = atomFamily((params: { id: string }) => atom(params), deepEqual);
```

**Syntax Rules:**

1. Use the `jotai-family` package for new code; `atomFamily` from `jotai/utils` is deprecated.
2. The `initializeAtom` function receives the parameter and returns an atom.
3. The `areEqual` function determines parameter equality (default `Object.is`).
4. Atom families hold every atom they create until told otherwise; use `remove()` or `setShouldRemove()` for cleanup.
5. Define the family at module level, never inside components.

**Constraints and Limitations:**

- Deprecated `atomFamily` will be removed in a future version.
- Families can leak memory if atoms are never removed.
- Parameters must be stable; new object parameters on every call create new atoms.

### Annotated Code Examples

```ts
import { atom } from 'jotai';
import { atomFamily } from 'jotai-family';

const todoFamily = atomFamily((id: string) =>
  atom({ id, text: '', done: false })
);

const todo1 = todoFamily('todo-1');
const todo1Again = todoFamily('todo-1'); // Same atom
const todo2 = todoFamily('todo-2'); // Different atom
```

**Expected Output:** `todo1` and `todo1Again` are the same atom; `todo2` is a different atom.

**Why this output:** `atomFamily` caches atoms by parameter. The same parameter returns the cached atom.

### Real-World Cases

- **Todo lists:** Per-todo editing state.
- **Dynamic forms:** Per-field atoms.
- **Chat messages:** Per-message state.
- **Data grids:** Per-row state.

### References

- Jotai — Family utility — https://jotai.org/docs/utilities/family
- jotai-family — GitHub — https://github.com/jotaijs/jotai-family
- agents-inc/skills — Jotai Best Practices (jotai-family) — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/SKILL.md

---

## 6. Jotai vs Recoil

### Definitions

**Core Definition:** Jotai and Recoil are both atomic state management libraries for React, but Jotai is smaller, unopinionated, and actively maintained, while Recoil is archived and no longer maintained.

**Technical Definition:** Both libraries use an atomic model: small atoms of state, derived selectors/atoms, and families for parameterised atoms. Recoil provides selectors, `selectorFamily`, snapshots, and `useRecoilCallback`. Jotai provides a more primitive, unopinionated API without built-in caching strategies, plus a rich utility/extension ecosystem. Recoil has been archived and does not work with React 19; Jotai is actively maintained and React 19-compatible. Bundle sizes: Jotai is about 12 KB min+gzip including the library, while Recoil is about 25 KB. A compatibility package (`jotai-recoil-compat`) allows Recoil code to run on Jotai with no changes.

**Beginner-Friendly Explanation:** Recoil is no longer maintained and does not work with React 19. Jotai is the modern, maintained alternative with a similar atomic model. It is smaller, has a simpler API, and has a large ecosystem of utilities and extensions.

### Purposes

- To choose the maintained atomic state library.
- To migrate from Recoil to Jotai with minimal changes.
- To understand the API differences between the two.
- To leverage Jotai's larger utility ecosystem.

### Comparison Table

| Dimension | Jotai | Recoil |
|-----------|-------|--------|
| **Maintenance** | Active | Archived, unmaintained |
| **React 19** | Supported | Not supported |
| **Bundle size** | ~12 KB min+gzip | ~25 KB |
| **API style** | Primitive, unopinionated | Selectors, `selectorFamily`, snapshots |
| **Families** | `jotai-family` package | `atomFamily`, `selectorFamily` |
| **Extensions** | Query, XState, Immer, tRPC, URQL, Effect, etc. | Limited |
| **Migration** | `jotai-recoil-compat` (drop-in) | — |

**Syntax Rules:**

1. Use Jotai for new atomic state; Recoil is archived.
2. Use `jotai-recoil-compat` as a drop-in replacement for Recoil code.
3. Jotai's `atomFamily` (via `jotai-family`) covers both Recoil's `atomFamily` and `selectorFamily` use cases.
4. Jotai does not provide snapshots; use stores and providers for isolation.
5. Jotai's API is more primitive; caching strategies are explicit rather than built-in.

**Constraints and Limitations:**

- Jotai's families hold atoms until removed; Recoil's families have different caching semantics.
- Recoil's `snapshot` API has no direct Jotai equivalent.
- Migration may require API adjustments for selectors and callbacks.

### Annotated Code Examples

**Jotai (modern):**

```ts
import { atom } from 'jotai';
const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);
```

**Recoil (archived):**

```js
import { atom, selector } from 'recoil';
const countState = atom({ key: 'count', default: 0 });
const doubledSelector = selector({ key: 'doubled', get: ({ get }) => get(countState) * 2 });
```

**Expected Output:** Both create a count atom and a doubled selector. Jotai's API is more primitive; Recoil requires explicit keys and selector config.

**Why this output:** Jotai uses a minimal `atom` function; Recoil requires configuration objects with keys.

### Real-World Cases

- **New projects:** Jotai.
- **Migrating from Recoil:** `jotai-recoil-compat`.
- **React 19 apps:** Jotai (Recoil is incompatible).

### References

- jotai-recoil-compat — npm — https://www.npmjs.com/package/jotai-recoil-compat
- Feature-Sliced Design — Recoil vs Jotai — https://feature-sliced.design/blog/jotai-state-management
- IEEE Software — Web Application State Management: A Review — https://www.computer.org

---

## 7. Jotai Best Practices

### Definitions

**Core Definition:** Jotai best practices are the guidelines for defining atoms, composing derived state, handling async data, and organising stores to build maintainable and performant applications.

**Technical Definition:** Best practices centre on **module-level atom definition** (never inside components), **small, focused atoms** (one atom per state concern), **derived atoms for computed state** (no stored redundancy), **write-only atoms for actions** (encapsulating multi-atom updates), **Suspense boundaries for async atoms** (or `loadable`/`unwrap`), **`jotai-family` for parameterised atoms** (not the deprecated `atomFamily`), **`useAtomValue`/`useSetAtom` for reads and writes** (to avoid accidental subscriptions), **Provider/store isolation for tests and multi-store apps**, and **`atomWithStorage` with `getOnInit` for persistence** (to avoid flicker).

**Beginner-Friendly Explanation:** Good Jotai usage means defining atoms outside components, keeping atoms small, using derived atoms for computed values, using write atoms for actions, wrapping async atoms in Suspense, and using `jotai-family` instead of the deprecated `atomFamily`.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Define atoms at module level** | Never inside components; new atoms every render break state |
| **Keep atoms small** | One atom per state concern; compose with derived atoms |
| **Use derived atoms for computed state** | Automatic dependency tracking; no stored redundancy |
| **Use write-only atoms for actions** | Encapsulate multi-atom updates and post-async state changes |
| **Wrap async atom consumers in Suspense** | Async atoms suspend by default |
| **Use `loadable`/`unwrap` for non-Suspense** | Discriminated union or fallback value |
| **Use `jotai-family` for parameterised atoms** | `atomFamily` from `jotai/utils` is deprecated |
| **Use `useAtomValue` for reads** | Avoids accidental subscriptions |
| **Use `useSetAtom` for writes** | Components that only write do not re-render on reads |
| **Use Provider/store for isolation** | Tests and multi-store architectures |
| **Use `atomWithStorage` with `getOnInit`** | Avoids flicker; beware SSR hydration |
| **Use `splitAtom` with `keyExtractor`** | Per-item re-render isolation for lists |

### Annotated Code Examples

**✅ Good Practices:**

```ts
// Module-level atoms
export const countAtom = atom(0);
export const doubledAtom = atom((get) => get(countAtom) * 2);

// Write-only action atom
export const resetAtom = atom(null, (get, set) => {
  set(countAtom, 0);
  set(itemsAtom, []);
});

// Async atom with Suspense
export const userAtom = atom(async (get) => {
  const id = get(userIdAtom);
  return (await fetch(`/api/users/${id}`)).json();
});

// Non-Suspense alternative
export const loadableUserAtom = loadable(userAtom);

// Parameterised atoms with jotai-family
const todoFamily = atomFamily((id: string) => atom({ id, done: false }));
```

```tsx
function Counter() {
  const count = useAtomValue(countAtom); // Read only
  const setCount = useSetAtom(countAtom); // Write only
  return <button onClick={() => setCount((c) => c + 1)}>{count}</button>;
}
```

**❌ Bad Practices:**

```ts
// ❌ Atom created inside component
function Bad() {
  const atom = atom(0); // New atom every render!
}

// ❌ Deprecated atomFamily
import { atomFamily } from 'jotai/utils'; // Use jotai-family

// ❌ Async atom without Suspense
function BadAsync() {
  const user = useAtomValue(userAtom); // Crashes without Suspense
}

// ❌ Storing derived state
const [total, setTotal] = useState(0); // Derive with a derived atom instead
```

**Expected Output:** The good practices produce stable atoms, fine-grained reactivity, and safe async handling. The bad practices cause broken state, deprecation warnings, Suspense crashes, and redundant state.

**Why this output:** Atoms inside components lose state on every render; deprecated `atomFamily` will be removed; async atoms require Suspense; stored derived state can go stale.

### Real-World Cases

- **All Jotai applications:** These practices apply universally.
- **Large apps:** Module-level atoms and `jotai-family` for collections.
- **SSR apps:** Provider/store isolation and `atomWithStorage` with `getOnInit`.

### References

- agents-inc/skills — Jotai Best Practices — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/SKILL.md
- Jotai — Documentation — https://jotai.org/docs
- Jotai — Async utilities — https://jotai.org/docs/utilities/async
- Jotai — Family utility — https://jotai.org/docs/utilities/family

---

## References

- Jotai — Official Documentation — https://jotai.org/docs
- Jotai — Tutorial — https://tutorial.jotai.org
- Jotai — Async Read Atoms — https://tutorial.jotai.org
- Jotai — Async Write Atoms — https://tutorial.jotai.org
- Jotai — Read-only atoms — https://tutorial.jotai.org
- Jotai — Write-only atoms — https://tutorial.jotai.org
- Jotai — Read-write atoms — https://tutorial.jotai.org
- Jotai — Family utility — https://jotai.org/docs/utilities/family
- Jotai — Async utilities (loadable/unwrap) — https://jotai.org/docs/utilities/async
- Jotai — Derive guide — https://jotai.org/docs/guides/derive
- Jotai — Core: atom — https://jotai.org/docs/core/atom
- jotai-family — GitHub — https://github.com/jotaijs/jotai-family
- jotai-recoil-compat — npm — https://www.npmjs.com/package/jotai-recoil-compat
- agents-inc/skills — Jotai Quick Guide — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/SKILL.md
- agents-inc/skills — Jotai Core Patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/examples/core.md
- agents-inc/skills — Jotai Async Patterns — https://github.com/agents-inc/skills/blob/main/src/skills/web-state-jotai/examples/async.md
- Feature-Sliced Design — Jotai: The Power of Minimalist State Design — https://feature-sliced.design/blog/jotai-state-management
- IEEE Software — Web Application State Management: A Review of Leading React Frameworks — https://www.computer.org