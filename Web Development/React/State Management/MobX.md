# MobX — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** MobX is a battle-tested, scalable state-management library that makes state management simple and scalable by transparently applying functional reactive programming (TFRP).

**Technical Definition:** MobX is a reactive state management library that builds a runtime dependency graph of your application state. It distinguishes four core concepts: **Observables** (trackable state fields), **Computeds** (derived values cached from observables), **Actions** (methods that modify state), and **Reactions** (side effects that run when observables change). When a computation (a computed getter or an `observer` component render) runs, MobX records which observables were read. Those reads become dependencies; when any dependency changes, MobX marks the computation as stale and re-runs it only if needed. This is **transparent reactivity** — you do not wire subscriptions manually; you just read state, and MobX does the tracking. In React, the `observer` higher-order component from `mobx-react-lite` records the observables read during render and re-renders the component on those alone, applying `React.memo` automatically. MobX is best suited for **rich domain models** with interconnected state and many derived values, where fine-grained tracking beats manual subscription. Its strength is transparent dependency tracking that updates only what actually changed; its scaling risk is **implicit coupling and store sprawl** without architectural boundaries.

**Beginner-Friendly Explanation:** MobX is a state manager built on the idea of **observables**. You mark your data as observable, and MobX watches it automatically. When the data changes, MobX knows exactly which computed values and which components depend on that data, and it updates only those. You do not write subscriptions, selectors, or dependency arrays — you just read the data, and MobX figures out the rest. It feels like magic at first: you change a property, and the right parts of the UI update. The trade-off is that this magic is invisible, so large codebases need clear boundaries to avoid a tangled web of implicit dependencies. MobX shines when your domain logic is the hard part — editors, complex forms, interconnected models — and is less ideal as a simple global dictionary.

### Key Characteristics

- **Transparent reactivity:** MobX records which observables are read inside computed values and `observer` renders; no manual subscriptions.
- **Fine-grained updates:** Only the minimal set of computeds and components that depend on a changed observable re-run.
- **Four core concepts:** `observable`, `computed`, `action`, and `reaction` are enough to master MobX.
- **`makeAutoObservable`:** One call infers every annotation — properties become observables, getters become computeds, methods become actions, generators become flows.
- **`observer` HOC:** Wraps React components, records observable reads, and re-renders only on those reads (applies `React.memo` automatically).
- **Strict mode (`enforceActions`):** Forbids mutating observed observables outside an action, enforcing a clear mutation boundary.
- **Async via `flow`/`runInAction`:** Code after `await` is outside the action scope; wrap post-await assignments or use `flow` generators.
- **Reaction disposers:** Every `autorun`, `reaction`, and `when` returns a disposer that must be called to avoid leaks.
- **Architecture-sensitive:** Without boundaries, MobX drifts into store sprawl and implicit coupling; Feature-Sliced Design and ownership rules mitigate this.

### Prerequisites

- React function components and Hooks (`useState`, `useEffect`).
- JavaScript classes, getters/setters, and closures.
- Understanding of reference equality and re-render triggers.
- TypeScript (recommended; MobX ships with typings out of the box).
- Basic knowledge of asynchronous JavaScript (`async`/`await`, generators).

### Related Programming Areas

- **State Management** — Redux, Zustand, Jotai, Valtio.
- **Reactive Programming** — signals, observables, dependency graphs.
- **React Integration** — `mobx-react-lite` (`observer`, `useLocalObservable`).
- **Feature-Sliced Design** — architectural boundaries for reactive stores.
- **Server State** — TanStack Query, SWR (MobX is for client/domain state).

---

## 1. MobX

### Definitions

**Core Definition:** MobX is a reactive state-management library that builds a dependency graph from observables to computeds and reactions, updating only what actually changed.

**Technical Definition:** MobX's core API exports `makeObservable`, `makeAutoObservable`, `observable`, `computed`, `action`, `flow`, `runInAction`, `reaction`, `autorun`, `when`, and `toJS`. A store class calls `makeAutoObservable(this)` in its constructor to annotate properties, getters, and methods. `observer` from `mobx-react-lite` makes React components reactive. MobX distinguishes two kinds of derivations: **computed values** (pure functions of observable state, cached and recomputed only when dependencies change) and **reactions** (side effects that run automatically when state changes). Actions are the only sanctioned way to modify state under strict mode.

**Beginner-Friendly Explanation:** MobX is the reactive state manager. You create a store class, call `makeAutoObservable(this)`, and MobX watches every property. Components wrapped in `observer` re-render only when the properties they actually read change.

### Purposes

- To provide transparent, fine-grained reactivity without manual subscriptions.
- To model rich domain logic with interconnected state and derived values.
- To eliminate constant memoization and manual dependency arrays.
- To enforce a clear mutation boundary through actions.
- To scale from a small store to a large domain model with architectural discipline.
- To integrate async data through `flow` and `runInAction`.

### Syntax Rules and Structure

**General Syntax (store class):**

```ts
import { makeAutoObservable, flow } from 'mobx';

class TodoStore {
  todos: Todo[] = [];

  constructor() {
    makeAutoObservable(this);
  }

  get activeTodos(): Todo[] {
    return this.todos.filter((t) => !t.done);
  }

  addTodo(title: string): void {
    this.todos.push({ id: crypto.randomUUID(), title, done: false });
  }

  *fetchTodos(): Generator {
    const res = yield fetch('/api/todos');
    this.todos = yield res.json();
  }
}

export const todoStore = new TodoStore();
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `makeAutoObservable(this)` | Infers annotations: properties → observable, getters → computed, methods → action, generators → flow |
| `makeObservable(this, {...})` | Explicit annotations; required for inheritance |
| `observable` | Defines a trackable field |
| `computed` | Marks a getter that derives and caches output |
| `action` | Marks a method that modifies state |
| `flow` | Async generator action that carries state changes across `await` |
| `observer` | HOC that makes a React component reactive |

**Syntax Rules:**

1. Call `makeAutoObservable(this)` in the constructor of every store class; without it, nothing is observable and nothing re-renders.
2. Use `makeObservable` with explicit annotations wherever inheritance is involved; `makeAutoObservable` throws on subclasses.
3. Wrap every component that reads an observable in `observer()` from `mobx-react-lite`.
4. Carry state changes across `await` in `runInAction()`, or write the whole operation as a `flow` generator.
5. Call the disposer every reaction returns.

**Constraints and Limitations:**

- `makeAutoObservable` cannot be used with inheritance; base and subclasses must use `makeObservable`.
- Methods that derive information and take arguments cannot be annotated as computed (their output is not memoized).
- MobX is not a server-state library; an observable holds a copy that nothing refreshes automatically.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Basic Store with `makeAutoObservable`

```ts
import { makeAutoObservable } from 'mobx';

interface Todo {
  id: string;
  title: string;
  done: boolean;
}

class TodoStore {
  todos: Todo[] = [];
  filter: 'all' | 'active' | 'done' = 'all';

  constructor() {
    makeAutoObservable(this);
  }

  get filteredTodos(): Todo[] {
    if (this.filter === 'all') return this.todos;
    return this.todos.filter((t) => (this.filter === 'done' ? t.done : !t.done));
  }

  addTodo(title: string): void {
    this.todos.push({ id: crypto.randomUUID(), title, done: false });
  }

  toggleTodo(id: string): void {
    const todo = this.todos.find((t) => t.id === id);
    if (todo) todo.done = !todo.done;
  }
}

export const todoStore = new TodoStore();
```

**Expected Output:** The store exposes `todos`, `filter`, a computed `filteredTodos`, and actions `addTodo` and `toggleTodo`.

**Why this output:** `makeAutoObservable` infers all annotations: `todos` and `filter` become observables, `filteredTodos` becomes a computed getter, and the methods become actions.

#### Example 2: React Component with `observer`

```tsx
import { observer } from 'mobx-react-lite';
import { todoStore } from './todo-store';

const TodoList = observer(() => {
  return (
    <ul>
      {todoStore.filteredTodos.map((todo) => (
        <li key={todo.id} onClick={() => todoStore.toggleTodo(todo.id)}>
          {todo.title}
        </li>
      ))}
    </ul>
  );
});
```

**Expected Output:** The list re-renders only when `filteredTodos` or the todos it depends on change; toggling a todo updates the list without re-rendering unrelated components.

**Why this output:** `observer` records the observables read during render (`filteredTodos`, which depends on `todos` and `filter`). It re-renders only when those reads change.

### Real-World Cases

- **Editors and complex forms:** Rich domain models with many derived values.
- **Dashboards:** Interconnected state with computed totals, filters, and aggregates.
- **Collaborative apps:** Domain logic that is the hard part, not the UI.

### References

- MobX — API Reference — https://mobx.js.org/api.html
- MobX — Creating Observable State — https://mobx.js.org/observable-state.html
- agents-inc/skills — MobX Quick Guide — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md
- Feature-Sliced Design — The Case for a Reactive MobX Architecture — https://feature-sliced.design/blog/mobx-reactive-programming
- khasky/state-management-playbook — MobX in Depth — https://github.com/khasky/state-management-playbook

---

## 2. Observables

### Definitions

**Core Definition:** An observable is a trackable field that stores state; MobX records reads of observables inside computations and reactions.

**Technical Definition:** Properties, entire objects, arrays, Maps, and Sets can all be made observable. `makeObservable(target, annotations)` marks existing object properties as observable; `makeAutoObservable(target)` infers annotations automatically. The `observable(source)` function clones a plain object, array, Map, or Set and makes it observable recursively (by default). Observable arrays have additional methods: `clear()`, `replace(newItems)`, and `remove(value)`. Observable Maps are useful when you want to react to additions and removals of specific entries. Under the hood, MobX replaces object properties with getter/setter pairs that track reads and trigger reactions.

**Beginner-Friendly Explanation:** An observable is a piece of state that MobX watches. When you read it inside a computed or an `observer` component, MobX remembers that read. When the observable changes, MobX knows exactly what to update.

### Purposes

- To define the reactive source of truth for the dependency graph.
- To track property reads automatically without manual subscriptions.
- To support objects, arrays, Maps, and Sets as observable collections.
- To enable fine-grained updates by tracking individual property reads, not whole-store changes.
- To provide collection methods (`clear`, `replace`, `remove`) for observable arrays.

### Syntax Rules and Structure

**Class store:**

```ts
class Store {
  count = 0;
  items: Item[] = [];

  constructor() {
    makeAutoObservable(this);
  }
}
```

**Observable object:**

```ts
import { observable } from 'mobx';

const todosById = observable({
  'TODO-123': { title: 'Learn MobX', done: false },
});
todosById['TODO-456'] = { title: 'Build an app', done: true };
```

**Observable array:**

```ts
const tags = observable(['high prio', 'medium prio']);
tags.push('low prio');
```

**Syntax Rules:**

1. Call `makeAutoObservable(this)` or `makeObservable(this, {...})` in the constructor.
2. Use `observable(source)` for standalone objects and collections.
3. Observable arrays support `clear()`, `replace()`, and `remove(value)`.
4. Use `{ deep: false }` to make an array or Map shallowly observable.
5. Use `toJS()` to convert observable structures back to plain JavaScript recursively.

**Constraints and Limitations:**

- Making an object observable recursively (the default) can be expensive for large graphs; use `{ deep: false }` for shallow observability.
- Adding observable properties after instantiation is possible with `extendObservable`, but the addition itself is not observable.
- Observable state works best for facts (domain data, UI session facts, request status), not for derived values that should be computed.

### Annotated Code Examples

```ts
import { makeAutoObservable, observable, toJS } from 'mobx';

class CartStore {
  items: { id: string; price: number; qty: number }[] = [];
  discountCode: string | null = null;

  constructor() {
    makeAutoObservable(this);
  }

  get total(): number {
    return this.items.reduce((sum, i) => sum + i.price * i.qty, 0);
  }

  addItem(item: { id: string; price: number }): void {
    this.items.push({ ...item, qty: 1 });
  }
}

const cart = new CartStore();
cart.addItem({ id: 'p1', price: 10 });
console.log(toJS(cart.items)); // [{ id: 'p1', price: 10, qty: 1 }]
```

**Expected Output:** `cart.items` is observable; `toJS` returns a plain array. `total` is a computed getter that updates when items change.

**Why this output:** `makeAutoObservable` makes `items` and `discountCode` observable, `total` computed, and `addItem` an action.

### Real-World Cases

- **Domain entities:** Users, products, orders.
- **UI session state:** Selected ID, open/closed flags, form values.
- **Request status:** Idle, loading, success, error.
- **Collections:** Lists, maps of entities by ID.

### References

- MobX — Creating Observable State — https://mobx.js.org/observable-state.html
- MobX — API Reference: observable — https://mobx.js.org/api.html
- Feature-Sliced Design — Observable state: what should be observable — https://feature-sliced.design/blog/mobx-reactive-programming

---

## 3. Actions

### Definitions

**Core Definition:** An action is any piece of code that modifies state; MobX marks actions as the sanctioned boundary for mutations.

**Technical Definition:** `makeAutoObservable` marks all methods as actions automatically. `makeObservable` allows explicit `action` annotations. Actions batch mutations: reactions and computeds run only after the outermost action completes, so intermediate states are never observed. Under strict mode (`enforceActions`), MobX forbids mutating observed observables outside an action, enforcing a clear boundary. `runInAction` creates an anonymous action for cases where an action wrapper is not available (e.g., after `await`). `flow` is an async generator action that carries state changes across `await` boundaries.

**Beginner-Friendly Explanation:** Actions are the only functions allowed to change observable state. They batch changes so the rest of the app sees one update, not a cascade of intermediate states.

### Purposes

- To provide a clear, auditable boundary for state mutations.
- To batch multiple mutations into a single reaction cycle.
- To enforce strict mode so mutations cannot happen accidentally.
- To carry state changes across asynchronous operations with `flow` or `runInAction`.
- To prevent intermediate states from being observed by reactions and components.

### Syntax Rules and Structure

**Action method (auto-annotated):**

```ts
class Store {
  count = 0;
  constructor() { makeAutoObservable(this); }
  increment(): void { this.count += 1; } // auto-action
}
```

**Explicit action:**

```ts
import { makeObservable, observable, action } from 'mobx';

class Store {
  count = 0;
  constructor() {
    makeObservable(this, { count: observable, increment: action });
  }
  increment(): void { this.count += 1; }
}
```

**`runInAction` (after `await`):**

```ts
async fetchCount(): Promise<void> {
  const res = await fetch('/api/count');
  const data = await res.json();
  runInAction(() => { this.count = data.count; });
}
```

**`flow` generator:**

```ts
*fetchCount(): Generator {
  const res = yield fetch('/api/count');
  const data = yield res.json();
  this.count = data.count; // Inside flow, this is still an action
}
```

**Syntax Rules:**

1. Use `makeAutoObservable(this)` to annotate all methods as actions automatically.
2. Use `makeObservable` with explicit `action` annotations when inheritance is involved.
3. Wrap post-`await` state assignments in `runInAction`.
4. Use `flow` generators for multi-step async actions; `flow` preserves the action scope across `yield`.
5. Enable strict mode (`enforceActions`) to catch mutations outside actions.

**Constraints and Limitations:**

- Under strict mode, mutating an observed observable outside an action throws.
- Code after `await` is outside the original action scope; it must be re-wrapped.
- Actions should not return Promises directly; use `flow` for async actions.

### Annotated Code Examples

```ts
import { makeAutoObservable, runInAction } from 'mobx';

class UserStore {
  user: User | null = null;
  loading = false;

  constructor() { makeAutoObservable(this); }

  async fetchUser(id: string): Promise<void> {
    this.loading = true; // Inside the action scope
    const res = await fetch(`/api/users/${id}`);
    const user = await res.json();
    runInAction(() => {
      this.user = user; // Post-await: must be wrapped
      this.loading = false;
    });
  }
}
```

**Expected Output:** `fetchUser` sets `loading` inside the action, fetches, and assigns `user` and `loading` inside `runInAction`.

**Why this output:** The code after `await` runs in a later tick, outside the action that started it; `runInAction` re-enters action scope for the post-await assignments.

### Real-World Cases

- **Form submissions:** Batching multiple field updates.
- **Async data fetching:** Wrapping post-await assignments.
- **Multi-step operations:** `flow` generators for sequential async actions.
- **Event handlers:** Auto-annotated methods as actions.

### References

- MobX — Actions — https://mobx.js.org/actions.html
- MobX — Configuration: enforceActions — https://mobx.js.org/configuration.html
- agents-inc/skills — MobX Critical Requirements — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md
- MobX pitfalls — strict mode and runInAction — https://errors.standardbeagle.com

---

## 4. Computed Values

### Definitions

**Core Definition:** A computed value is a derived value that is computed from the current observable state using a pure function and cached until its dependencies change.

**Technical Definition:** Computed values are created by annotating JavaScript getters with `computed`. `makeAutoObservable` annotates all getters as computed automatically; `makeObservable` requires explicit `computed` annotations. A computed value is evaluated lazily and cached: MobX tracks which observables it reads, and recomputes only when one of those dependencies changes. Computeds can depend on other computeds, forming a derivation chain. Methods that derive information and take arguments cannot be annotated as computed; their output is not memoized. Use `MobX-utils computedFn` for memoized parameterised derivations.

**Beginner-Friendly Explanation:** A computed value is a formula. It reads observables and caches the result. When an observable it reads changes, the formula recalculates automatically. If you read it again without changing any dependency, MobX returns the cached result.

### Purposes

- To derive values from state without storing redundant data.
- To cache expensive derivations and recompute only when dependencies change.
- To compose derivation chains with automatic dependency tracking.
- To keep a single source of truth for each fact.
- To avoid using reactions for computed data (a common mistake).

### Syntax Rules and Structure

**Computed getter (auto-annotated):**

```ts
class Store {
  items: Item[] = [];
  constructor() { makeAutoObservable(this); }
  get total(): number {
    return this.items.reduce((sum, i) => sum + i.price, 0);
  }
}
```

**Explicit computed:**

```ts
import { makeObservable, observable, computed } from 'mobx';

class Store {
  items: Item[] = [];
  constructor() {
    makeObservable(this, { items: observable, total: computed });
  }
  get total(): number {
    return this.items.reduce((sum, i) => sum + i.price, 0);
  }
}
```

**Syntax Rules:**

1. Annotate getters as computed; `makeAutoObservable` does this automatically.
2. Computeds must be pure; do not modify state inside a computed.
3. Computeds can read other computeds, forming chains.
4. Parameterised derivations cannot be computed; use `computedFn` from `mobx-utils`.
5. Do not use reactions to compute derived values; use computeds.

**Constraints and Limitations:**

- Computeds are cached; if a dependency changes but the computed is not read, it is not recomputed until read.
- Computeds cannot have side effects.
- Methods that derive information and take arguments are tracked but not memoized.

### Annotated Code Examples

```ts
import { makeAutoObservable } from 'mobx';

class CartStore {
  items: { price: number; qty: number }[] = [];
  taxRate = 0.1;

  constructor() { makeAutoObservable(this); }

  get subtotal(): number {
    return this.items.reduce((sum, i) => sum + i.price * i.qty, 0);
  }

  get tax(): number {
    return this.subtotal * this.taxRate;
  }

  get total(): number {
    return this.subtotal + this.tax;
  }
}
```

**Expected Output:** `subtotal`, `tax`, and `total` are computed getters that update automatically when `items` or `taxRate` change.

**Why this output:** Each getter is annotated as computed by `makeAutoObservable`. The derivation chain `items → subtotal → tax → total` is tracked automatically.

### Real-World Cases

- **Cart totals:** Subtotal, tax, and total.
- **Filtered lists:** Filtering by an observable filter.
- **Formatted values:** Currency, dates, and names.
- **Validation state:** Deriving `isValid` from multiple fields.
- **Dashboard aggregates:** Sums, averages, and counts.

### References

- MobX — Deriving Information with Computeds — https://mobx.js.org/computeds.html
- MobX — API Reference: computed — https://mobx.js.org/api.html
- Feature-Sliced Design — Computed is the workhorse — https://feature-sliced.design/blog/mobx-reactive-programming
- khasky/state-management-playbook — Computed is the workhorse — https://github.com/khasky/state-management-playbook

---

## 5. Reactions

### Definitions

**Core Definition:** A reaction is a side effect that runs automatically when the observable state it reads changes.

**Technical Definition:** MobX distinguishes two kinds of derivations: computed values (pure derivations) and reactions (side effects). Reactions are created with `autorun`, `reaction`, or `when`. `autorun` runs immediately and tracks all observable reads inside the function, re-running on any change. `reaction` separates the tracked expression from the effect, so the effect runs only when the tracked expression's result changes. `when` runs an effect once when a predicate becomes true. Every reaction returns a **disposer** function that must be called to stop it and release its closures. Reactions are for side effects (logging, persistence, network sync), not for computing derived values.

**Beginner-Friendly Explanation:** Reactions are side effects that happen when state changes. `autorun` runs now and on every change; `reaction` watches one expression and runs an effect when it changes; `when` runs once when a condition becomes true. Always call the disposer.

### Purposes

- To run side effects (logging, persistence, analytics) when state changes.
- To synchronise observable state with external systems.
- To run an effect once when a condition is met (`when`).
- To separate the tracked expression from the effect (`reaction`).
- To bridge MobX state to non-React code.

### Syntax Rules and Structure

**`autorun`:**

```ts
const disposer = autorun(() => {
  console.log('count:', store.count);
});
// Later: disposer();
```

**`reaction`:**

```ts
const disposer = reaction(
  () => store.count,
  (count, previousCount) => {
    console.log(`count changed: ${previousCount} → ${count}`);
  }
);
```

**`when`:**

```ts
const disposer = when(
  () => store.isReady,
  () => { console.log('ready!'); }
);
// Or promise form:
await when(() => store.isReady);
```

**Syntax Rules:**

1. `autorun` runs immediately and on every change of any observable read.
2. `reaction` tracks the first function and runs the effect only when its result changes.
3. `when` runs the effect once when the predicate becomes true; returns a disposer or Promise.
4. Every reaction returns a disposer; call it in cleanup functions.
5. Use reactions for side effects, not for computing derived values.

**Constraints and Limitations:**

- Reactions keep their closures alive until disposed; missing disposers cause memory leaks.
- `autorun` can re-run more often than expected if it reads many observables.
- Reactions do not participate in React's Suspense; use them outside render.

### Annotated Code Examples

```ts
import { autorun, reaction, when } from 'mobx';

const store = new CartStore();

// autorun: runs immediately and on every change
const disposeAutorun = autorun(() => {
  console.log('cart total:', store.total);
});

// reaction: effect only when the tracked value changes
const disposeReaction = reaction(
  () => store.items.length,
  (length, prevLength) => {
    console.log(`items: ${prevLength} → ${length}`);
  }
);

// when: run once when the condition is true
const disposeWhen = when(
  () => store.items.length > 0,
  () => console.log('first item added')
);

// Cleanup
disposeAutorun();
disposeReaction();
disposeWhen();
```

**Expected Output:** The autorun logs the total on every change; the reaction logs item-count changes; the `when` logs once when the first item is added.

**Why this output:** Each reaction tracks its specific reads and runs according to its type. The disposers stop them and release their closures.

### Real-World Cases

- **Persistence:** Saving state to `localStorage` on change.
- **Analytics:** Logging events when state changes.
- **External sync:** Pushing state to a WebSocket or server.
- **One-time effects:** Showing a toast when a condition is met.

### References

- MobX — Running Side Effects with Reactions — https://mobx.js.org/reactions.html
- MobX — API Reference: autorun, reaction, when — https://mobx.js.org/api.html
- agents-inc/skills — Reactions: the disposer requirement — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md
- khasky/state-management-playbook — Reactions, sparingly — https://github.com/khasky/state-management-playbook

---

## 6. MobX React (`mobx-react-lite`)

### Definitions

**Core Definition:** `mobx-react-lite` is the React binding for MobX that provides the `observer` higher-order component and `useLocalObservable` hook for function components.

**Technical Definition:** `observer(Component)` wraps a React function component, records the observables read during render, and re-renders the component only when those observables change. It applies `React.memo` automatically, so observables that are accessible but not read do not cause re-renders. `useLocalObservable(initializer, annotations?)` creates a local observable store for a component, useful for component-scoped observable state. The older `useObserver`, `useLocalStore`, and `useAsObservableSource` APIs are removed in MobX 7; use `observer` and `useLocalObservable` instead. `mobx-react` is a thin wrapper around `mobx-react-lite` that also supports class components and the `@observer` decorator.

**Beginner-Friendly Explanation:** `observer` makes a component reactive: it watches the observables the component reads and re-renders only when those change. `useLocalObservable` creates a small observable store just for that component.

### Purposes

- To make React components re-render only when the observables they read change.
- To eliminate manual `useMemo`, `useCallback`, and dependency arrays for observable state.
- To provide component-scoped observable state with `useLocalObservable`.
- To integrate MobX with function components (and class components via `mobx-react`).
- To support deep observable reads (e.g., `todos[0].author.displayName`) without extra wiring.

### Syntax Rules and Structure

**`observer` HOC:**

```tsx
import { observer } from 'mobx-react-lite';

const TimerView = observer(({ timer }) => (
  <span>Seconds passed: {timer.secondsPassed}</span>
));
```

**`useLocalObservable`:**

```tsx
import { useLocalObservable, observer } from 'mobx-react-lite';

const Counter = observer(() => {
  const store = useLocalObservable(() => ({
    count: 0,
    get doubled() { return this.count * 2; },
    increment() { this.count += 1; },
  }));
  return <button onClick={store.increment}>{store.doubled}</button>;
});
```

**Syntax Rules:**

1. Wrap every component that reads an observable in `observer()` from `mobx-react-lite`.
2. Use `useLocalObservable` for component-scoped observable state; the initializer returns the observable object.
3. `observer` applies `React.memo` automatically; no extra memoization is needed.
4. `observer` works with function components; use `mobx-react` for class components.
5. Use `<Observer>` inside class component render if needed (legacy).

**Constraints and Limitations:**

- An unwrapped component renders once with the initial value and then stops; `observer` is mandatory for reactivity.
- `observer` tracks reads during render; observables read in effects or callbacks are not tracked by the component.
- `mobx-react-lite` requires React 18 or later.
- `useObserver` and `useLocalStore` are removed in MobX 7.

### Annotated Code Examples

```tsx
import { observer, useLocalObservable } from 'mobx-react-lite';

const TodoApp = observer(() => {
  const store = useLocalObservable(() => ({
    todos: [] as { id: string; title: string; done: boolean }[],
    get remaining() {
      return this.todos.filter((t) => !t.done).length;
    },
    add(title: string) {
      this.todos.push({ id: crypto.randomUUID(), title, done: false });
    },
    toggle(id: string) {
      const t = this.todos.find((t) => t.id === id);
      if (t) t.done = !t.done;
    },
  }));

  return (
    <div>
      <p>{store.remaining} remaining</p>
      <button onClick={() => store.add('New todo')}>Add</button>
      <ul>
        {store.todos.map((t) => (
          <li key={t.id} onClick={() => store.toggle(t.id)}>
            {t.title}
          </li>
        ))}
      </ul>
    </div>
  );
});
```

**Expected Output:** The component re-renders only when `remaining` or the `todos` list changes; toggling a todo updates the count and list.

**Why this output:** `useLocalObservable` creates an observable store scoped to the component. `observer` records the reads (`remaining`, `todos`) during render and re-renders on those changes.

### Real-World Cases

- **Feature components:** Components reading domain stores.
- **Local UI state:** Component-scoped observable state with `useLocalObservable`.
- **Lists:** Observable arrays rendered with `observer` for fine-grained updates.

### References

- MobX — React Integration — https://mobx.js.org/react-integration.html
- mobx-react-lite — npm — https://www.npmjs.com/package/mobx-react-lite
- MobX — Migrating to MobX 7 — https://mobx.js.org/migrating-to-mobx-7.html
- agents-inc/skills — observer requirement — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md

---

## 7. MobX Best Practices

### Definitions

**Core Definition:** MobX best practices are the guidelines for structuring stores, using observables, actions, computeds, reactions, and React integration to build maintainable, scalable applications.

**Technical Definition:** Best practices combine **`makeAutoObservable` discipline** (call it in every store constructor), **inheritance handling** (use `makeObservable` with explicit annotations for base and subclasses), **`observer` everywhere** (wrap every component reading observables), **async safety** (`runInAction` or `flow` across `await`), **disposer discipline** (call every reaction's disposer), **computed-first thinking** (derive values, do not store them), **reaction restraint** (reactions for side effects, not derived data), and **architectural boundaries** (Feature-Sliced Design, ownership, public APIs) to prevent store sprawl and implicit coupling.

**Beginner-Friendly Explanation:** Good MobX usage means calling `makeAutoObservable` in every store, wrapping every reading component in `observer`, wrapping post-await assignments in `runInAction`, calling reaction disposers, computing derived values instead of storing them, and using reactions only for side effects. And in large codebases, it means clear store boundaries.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Call `makeAutoObservable(this)` in every store** | Without it, nothing is observable and nothing re-renders |
| **Use `makeObservable` for inheritance** | `makeAutoObservable` throws on subclasses |
| **Wrap every reading component in `observer`** | Unwrapped components render once and stop |
| **Wrap post-await assignments in `runInAction`** | Code after `await` is outside the action scope |
| **Use `flow` for async generators** | Carries the action scope across `yield` |
| **Call every reaction disposer** | Prevent memory leaks and lingering closures |
| **Compute, don't store derived state** | Use computeds instead of reactions for derived values |
| **Use reactions only for side effects** | Logging, persistence, external sync |
| **Avoid the global “effects.ts” anti-pattern** | Scattered reactions referencing everything |
| **Enforce architectural boundaries** | Feature-Sliced Design, ownership, public APIs |
| **Keep server state out of MobX** | Use TanStack Query or SWR for server data |
| **Use `useLocalObservable` for component state** | Scoped observable state without global stores |

### Annotated Code Examples

**✅ Good Practices:**

```ts
import { makeAutoObservable, runInAction } from 'mobx';

class TodoStore {
  todos: Todo[] = [];
  loading = false;

  constructor() {
    makeAutoObservable(this); // Required
  }

  get remaining(): number {
    return this.todos.filter((t) => !t.done).length; // Computed
  }

  async fetchTodos(): Promise<void> {
    this.loading = true;
    const res = await fetch('/api/todos');
    const todos = await res.json();
    runInAction(() => {           // Post-await inside action
      this.todos = todos;
      this.loading = false;
    });
  }
}
```

```tsx
import { observer } from 'mobx-react-lite';

const TodoCount = observer(() => (
  <span>{todoStore.remaining} remaining</span> // Observer wraps reads
));
```

**❌ Bad Practices:**

```ts
// ❌ Missing makeAutoObservable
class BrokenStore {
  count = 0;
  increment() { this.count += 1; } // Nothing re-renders
}

// ❌ Post-await without runInAction
async badFetch() {
  const res = await fetch('/api');
  this.data = await res.json(); // Strict mode error
}

// ❌ Reaction for derived data
reaction(() => store.items, () => { store.total = ... }); // Use computed

// ❌ Unwrapped component reading observables
function Bad() {
  return <span>{store.count}</span>; // Renders once, never updates
}

// ❌ Missing disposer
autorun(() => console.log(store.count)); // Leaks
```

**Expected Output:** The good practices produce reactive, strictly bounded, scalable stores. The bad practices cause non-reactivity, strict-mode errors, redundant reactions, frozen components, and memory leaks.

**Why this output:** `makeAutoObservable` is mandatory for reactivity; `runInAction` is required after `await`; reactions do not replace computeds; `observer` is required for component reactivity; disposers prevent leaks.

### Real-World Cases

- **All MobX applications:** These practices apply universally.
- **Large codebases:** Feature-Sliced Design boundaries prevent store sprawl.
- **React 18/19:** `observer` and `useLocalObservable` are the supported APIs.
- **Async-heavy apps:** `flow` and `runInAction` for safe async mutations.

### References

- agents-inc/skills — MobX Critical Requirements — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md
- Feature-Sliced Design — The Case for a Reactive MobX Architecture — https://feature-sliced.design/blog/mobx-reactive-programming
- khasky/state-management-playbook — MobX Pitfalls — https://github.com/khasky/state-management-playbook
- MobX — Common Pitfalls & Best Practices — https://mobx.js.org/best-practices.html
- MobX — Configuration — https://mobx.js.org/configuration.html

---

## References

- MobX — Official Documentation — https://mobx.js.org/
- MobX — API Reference — https://mobx.js.org/api.html
- MobX — Creating Observable State — https://mobx.js.org/observable-state.html
- MobX — Deriving Information with Computeds — https://mobx.js.org/computeds.html
- MobX — Actions — https://mobx.js.org/actions.html
- MobX — Running Side Effects with Reactions — https://mobx.js.org/reactions.html
- MobX — React Integration — https://mobx.js.org/react-integration.html
- MobX — Migrating to MobX 7 — https://mobx.js.org/migrating-to-mobx-7.html
- MobX — Configuration: enforceActions — https://mobx.js.org/configuration.html
- mobx-react-lite — npm — https://www.npmjs.com/package/mobx-react-lite
- agents-inc/skills — MobX Quick Guide — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/SKILL.md
- agents-inc/skills — MobX Core Examples — https://github.com/agents-inc/skills/blob/main/dist/plugins/web-state-mobx/skills/web-state-mobx/examples/core.md
- Feature-Sliced Design — The Case for a Reactive MobX Architecture — https://feature-sliced.design/blog/mobx-reactive-programming
- khasky/state-management-playbook — Practical guide to client state architecture — https://github.com/khasky/state-management-playbook
- MobX pitfalls — strict mode and runInAction — https://errors.standardbeagle.com
- IEEE Software — Web Application State Management: A Review of Leading React Frameworks — https://www.computer.org
- Zenodo — Measuring the Cost of State: A Controlled Benchmark of Seven React State Management Libraries — https://zenodo.org