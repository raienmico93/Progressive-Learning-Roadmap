# React State Management Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** State management patterns are the reusable architectural solutions for deciding where state lives, how it changes, how it is shared, and how it survives over time.

**Technical Definition:** State management patterns are the structural conventions that govern four questions: **ownership** (which component or store is the authority for a fact), **mutation** (how state changes — directly, via reducers, via events, via machines), **distribution** (how changes propagate — lifting, colocation, context, observers, pub/sub), and **durability** (how state is persisted, hydrated, and migrated across versions). The patterns in this cheat sheet operate at different layers: **placement patterns** (lifting, colocation) decide where state lives; **mutation patterns** (reducers, state machines, event sourcing) decide how it changes; **distribution patterns** (observer, pub/sub, CQRS) decide how changes reach consumers; and **durability patterns** (persistence, hydration, migration) decide how state survives reloads and releases. The overarching principle is that **each pattern solves a specific problem, and applying it outside that problem creates new ones** — a reducer does not fix prop drilling, and persistence does not fix stale server data.

**Beginner-Friendly Explanation:** State management patterns are the recipes for organising your app's data. Some patterns tell you **where** to put state (close to where it is used, or lifted up). Some tell you **how** to change it (through a reducer, through a state machine, through events). Some tell you **how other parts of the app find out** when it changes (observers, pub/sub). And some tell you how to **keep it around** after a reload (persistence, hydration, migration). You do not need all of them in every app — you pick the patterns that match the problems you actually have.

### Key Characteristics

- **Layered:** Placement, mutation, distribution, and durability patterns solve different problems and compose.
- **Problem-specific:** Each pattern has a narrow best-fit scenario; misapplication creates new complexity.
- **Composable:** Lifting + reducers + context + persistence is a legitimate stack; so is colocation + state machines + pub/sub.
- **Trade-off driven:** Every pattern trades simplicity for capability (or vice versa).
- **Durability is explicit:** Persistence, hydration, and migration are not automatic; they require versioning and validation.
- **Backend-inspired patterns are not free:** Event sourcing and CQRS bring strong guarantees but also significant operational overhead.

### Prerequisites

- React function components, `useState`, `useReducer`, and `useContext`.
- JavaScript Promises, `async`/`await`, and event handling.
- Basic understanding of state categories (local, shared, server, URL, form).
- Familiarity with `localStorage`, `sessionStorage`, and JSON serialisation.
- Basic knowledge of event-driven architecture and backend patterns (for event sourcing and CQRS).

### Related Programming Areas

- **State Fundamentals** — `useState`, `useReducer`, derived state.
- **State Management Libraries** — Redux, Zustand, Jotai, XState, TanStack Query.
- **Architecture Patterns** — event-driven architecture, CQRS, event sourcing.
- **Performance** — re-render control through selectors and subscriptions.

---

## 1. Lifting State

### Definitions

**Core Definition:** Lifting state is the practice of moving state from a child component up to the closest common ancestor that needs it, then passing it down as props to the components that read it.

**Technical Definition:** Lifting state is React's canonical solution to the "two siblings need the same data" problem. When two or more components need to reflect the same changing data, the state is removed from each child and moved to their nearest common ancestor. The parent passes the state value and the setter (or a derived callback) down as props. This keeps a **single source of truth** and keeps siblings synchronised. The trade-off is **prop drilling** when the common ancestor is distant, and **re-render propagation** when the lifted state changes.

**Beginner-Friendly Explanation:** If two components need to share information, you move that information to their parent. The parent holds the data and gives it to both children.

### Purposes

- To synchronise sibling components that must agree on a value.
- To establish a single source of truth for shared state.
- To avoid duplicating state across components.
- To enable one component to influence another's rendering.
- To keep state ownership explicit and traceable.

### Syntax Rules and Structure

**General Syntax:**

```tsx
function Parent() {
  const [value, setValue] = useState(initialValue);
  return (
    <>
      <ChildA value={value} onChange={setValue} />
      <ChildB value={value} />
    </>
  );
}
```

**Component Breakdown:**

| Element | Description |
|---------|-------------|
| `Parent` | Nearest common ancestor; owns the state |
| `value` | The shared state value |
| `setValue` | The setter or a callback to request changes |
| `ChildA` | A consumer that reads and changes the value |
| `ChildB` | A consumer that only reads the value |

**Syntax Rules:**

1. Lift to the **nearest** common ancestor, not the root.
2. Pass the setter or a domain-specific callback (e.g., `onAddItem`) rather than exposing the raw setter when the child should not know the state shape.
3. Never keep a duplicate copy of the lifted state in a child.
4. Derive values in the parent or children rather than storing duplicates.

**Constraints and Limitations:**

- Prop drilling when the ancestor is far from the consumers.
- Every intermediate component re-renders when the state changes.
- Over-lifting centralises state unnecessarily and couples unrelated components.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Lifting for Sibling Synchronisation

```tsx
function TemperatureCalculator() {
  const [celsius, setCelsius] = useState(0);

  return (
    <div>
      <TemperatureInput
        label="Celsius"
        value={celsius}
        onChange={setCelsius}
      />
      <TemperatureInput
        label="Fahrenheit"
        value={celsius * 9 / 5 + 32}
        onChange={(f) => setCelsius((f - 32) * 5 / 9)}
      />
    </div>
  );
}

function TemperatureInput({ label, value, onChange }) {
  return (
    <label>
      {label}:
      <input
        type="number"
        value={Math.round(value * 10) / 10}
        onChange={(e) => onChange(Number(e.target.value))}
      />
    </label>
  );
}
```

**Expected Output:** Typing 100 in Celsius displays 212 in Fahrenheit, and vice versa.

**Why this output:** `celsius` lives in the parent, so both inputs read from the same source of truth. Changing either input updates the shared value, and both re-render in sync.

#### Example 2: Lifting Only When Needed

```tsx
// ❌ Over-lifted: modal state lives at the app root
function App() {
  const [isModalOpen, setModalOpen] = useState(false);
  return (
    <Layout isModalOpen={isModalOpen} onOpenModal={() => setModalOpen(true)}>
      <Page onOpenModal={() => setModalOpen(true)} />
    </Layout>
  );
}

// ✅ Colocated: modal state lives with the trigger
function Page() {
  const [isModalOpen, setModalOpen] = useState(false);
  return (
    <>
      <button onClick={() => setModalOpen(true)}>Open</button>
      {isModalOpen && <Modal onClose={() => setModalOpen(false)} />}
    </>
  );
}
```

**Expected Output:** Both work, but the second version keeps `Layout` out of the re-render path and removes prop drilling.

**Why this output:** The modal is only used inside `Page`, so lifting it to `App` created unnecessary coupling. Lifting is a tool for **sharing**, not a default.

### Real-World Cases

- **Master-detail views:** Selecting an item in a list shows details in a sibling pane.
- **Accordions and tabs:** Only one section open at a time.
- **Forms with dependent fields:** Changing one field updates another.
- **Filters and results:** A filter panel and a results list sharing filter state.

### References

- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state

---

## 2. Colocating State

### Definitions

**Core Definition:** Colocating state is the practice of keeping each piece of state as close as possible to the components that actually use it.

**Technical Definition:** State colocation places state in the **lowest** component in the tree that needs it, rather than lifting it prematurely. State lifted too high forces intermediate components to forward props they do not use, and every parent in the chain re-renders when the state changes, even though only the leaf reads it. Colocation confines re-renders to the smallest subtree and eliminates the forwarding chain. The rule is: keep state as low as it can live, and lift it only when another component genuinely needs to read or change it.

**Beginner-Friendly Explanation:** Put state where it is used. If only one small part of the UI needs a piece of state, that state should live in that small part, not at the top of the app.

### Purposes

- To minimise the number of components that re-render when state changes.
- To avoid prop forwarding through intermediate components.
- To make components easier to reason about and test in isolation.
- To reduce the surface area for bugs.
- To keep state ownership visible and local.

### Syntax Rules and Structure

**Colocation decision:**

| State usage | Placement |
|-------------|-----------|
| One component uses it | Inside that component |
| A small subtree uses it | In the subtree's root |
| Multiple distant components use it | Lift to their nearest common ancestor, or use Context |
| App-wide, low-frequency | Context or a store |
| App-wide, high-frequency | Store with selectors |

**Syntax Rules:**

1. Start with state in the component that uses it.
2. Lift only when a sibling or cousin also needs it.
3. Use Context only when prop drilling becomes unwieldy, not as a default.
4. Move state down when you notice intermediate components forwarding props they do not use.

**Constraints and Limitations:**

- Colocation is not a rule against lifting; it is a **default** that lifting overrides when sharing is real.
- State that must survive route changes cannot be colocated in a route component.

### Annotated Code Examples

```tsx
// ❌ State too high: every keystroke re-renders the whole page
function ProductPage() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <div>
      <Header />
      <SearchBar query={searchQuery} onQueryChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
      <Footer />
    </div>
  );
}

// ✅ Colocated: only the search subtree re-renders
function ProductPage() {
  return (
    <div>
      <Header />
      <ProductSearch />
      <Footer />
    </div>
  );
}

function ProductSearch() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <>
      <SearchBar query={searchQuery} onQueryChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
    </>
  );
}
```

**Expected Output:** In the colocated version, typing in the search bar does not re-render `Header` or `Footer`.

**Why this output:** `searchQuery` is confined to `ProductSearch`. `Header` and `Footer` are outside its subtree, so they are not re-rendered.

### Real-World Cases

- **Search bars:** Keeping query state inside the search component.
- **Modals:** Keeping open/closed state inside the modal trigger.
- **Tabs:** Keeping the active tab inside the tab group.
- **Widgets:** Each dashboard widget owning its own loading and filter state.

### References

- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- pproenca/dot-skills — State Colocation with Consumers — https://github.com/pproenca/dot-skills

---

## 3. State Machines

### Definitions

**Core Definition:** A state machine is a model of computation with a finite set of named states, a set of events, and deterministic transitions between states.

**Technical Definition:** A finite state machine (FSM) consists of a finite set of **states** `S`, a finite set of **events** `E`, a **transition function** `δ: S × E → S`, and an initial state `s₀`. The machine is always in exactly one state, and every transition is explicit. Events without a transition are ignored. This makes **impossible states impossible**: there is no way to be simultaneously `loading` and `success`, because those are distinct states rather than independent booleans. XState is the dominant React implementation, supporting hierarchy, parallelism, and the actor model. State machines are appropriate for **complex, finite, event-driven logic** (auth flows, wizards, media players) and overkill for trivial toggles.

**Beginner-Friendly Explanation:** A state machine is a map of all the modes your component can be in, and all the events that move it between them. It can only be in one mode at a time, and every move is explicit.

### Purposes

- To eliminate impossible states and invalid transitions.
- To make every valid state and path explicit and visible.
- To make complex UI logic testable without rendering.
- To provide a shared vocabulary for designers, developers, and QA.
- To handle async workflows (loading, success, failure, timeout) deterministically.

### Syntax Rules and Structure

**General Syntax (XState v5):**

```ts
import { createMachine, createActor } from 'xstate';

const machine = createMachine({
  id: 'fetch',
  initial: 'idle',
  context: { data: null, error: null },
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      on: {
        RESOLVE: 'success',
        REJECT: 'failure',
      },
    },
    success: {},
    failure: { on: { RETRY: 'loading' } },
  },
});
```

**Syntax Rules:**

1. Every machine has exactly one `initial` state.
2. Each state declares the events it accepts via `on`.
3. Events without a handler are ignored, not errors.
4. Quantitative data lives in `context`; qualitative modes live in states.
5. Update context only through `assign` actions.

**Constraints and Limitations:**

- Over-modelling simple toggles adds ceremony.
- The machine models logic, not rendering; the UI consumes snapshots.
- Hierarchy and parallelism require statecharts, not flat FSMs.

### Annotated Code Examples

```tsx
import { useMachine } from '@xstate/react';
import { createMachine } from 'xstate';

const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  context: { data: null as string[] | null, error: null as string | null },
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      invoke: {
        src: async () => {
          const res = await fetch('/api/items');
          if (!res.ok) throw new Error(`HTTP ${res.status}`);
          return res.json();
        },
        onDone: {
          target: 'success',
          actions: ({ context, event }) => {
            context.data = event.output;
          },
        },
        onError: {
          target: 'failure',
          actions: ({ context, event }) => {
            context.error = event.error.message;
          },
        },
      },
    },
    success: {},
    failure: { on: { RETRY: 'loading' } },
  },
});

function FetchButton() {
  const [snapshot, send] = useMachine(fetchMachine);
  return (
    <button
      onClick={() => send({ type: snapshot.matches('failure') ? 'RETRY' : 'FETCH' })}
      disabled={snapshot.matches('loading')}
    >
      {snapshot.matches('loading') ? 'Loading…' : 'Fetch'}
    </button>
  );
}
```

**Expected Output:** Clicking the button transitions `idle → loading → success` (or `failure`). Retry transitions `failure → loading`.

**Why this output:** The machine explicitly models each mode. The button only needs to know which event to send based on the current state; the machine enforces that only valid transitions occur.

### Real-World Cases

- **Authentication flows:** `loggedOut` → `loggingIn` → `loggedIn` / `error`.
- **Checkout wizards:** `cart` → `shipping` → `payment` → `review` → `confirmed`.
- **Media players:** `playing` / `paused` / `stopped` / `buffering`.
- **Data fetching:** `idle` / `loading` / `success` / `failure` / `timeout`.

### References

- XState — Official Documentation — https://stately.ai/docs/xstate
- Stately — State Machines and Statecharts — https://stately.ai/docs/state-machines-and-statecharts
- David Harel — Statecharts: A Visual Formalism for Complex Systems — https://www.sciencedirect.com/science/article/pii/0167642387900359

---

## 4. Event-Driven State

### Definitions

**Core Definition:** Event-driven state is a model in which state changes are triggered by discrete events (messages) rather than by direct method calls or property assignments.

**Technical Definition:** In an event-driven model, components or services **emit events** (plain objects describing what happened), and **event handlers** or **reducers** decide how state changes. The emitter does not know who handles the event or what the state will become. This decouples the **cause** (a user action, a WebSocket message, a timer) from the **effect** (the state transition). React's `dispatch` + reducer pattern is the simplest event-driven model; `EventTarget`, `CustomEvent`, and `BroadcastChannel` provide browser-native event buses; pub/sub libraries provide in-process event buses. Event-driven state is the foundation of reducers, state machines, and event sourcing.

**Beginner-Friendly Explanation:** Event-driven state means state changes happen because **something happened** — a click, a message, a timer — and you describe that something as an event. The code that changes state listens for events and reacts.

### Purposes

- To decouple event producers from state transformers.
- To make every state change traceable to a discrete event.
- To enable multiple independent consumers to react to the same event.
- To support cross-component, cross-tab, or cross-iframe communication.
- To provide a uniform model for user actions, network messages, and timers.

### Syntax Rules and Structure

**Reducer-style events:**

```ts
type Action =
  | { type: 'add'; text: string }
  | { type: 'toggle'; id: string }
  | { type: 'remove'; id: string };
```

**Browser events:**

```ts
const bus = new EventTarget();
bus.dispatchEvent(new CustomEvent('cart:add', { detail: { id: 'p1' } }));
bus.addEventListener('cart:add', (e) => { /* ... */ });
```

**Cross-tab events:**

```ts
const channel = new BroadcastChannel('app');
channel.postMessage({ type: 'cart:add', id: 'p1' });
channel.onmessage = (e) => { /* ... */ };
```

**Syntax Rules:**

1. Events are plain objects with a `type` (and optional payload).
2. Events are immutable descriptions of what happened; they do not describe the next state.
3. Handlers decide how to translate events into state changes.
4. Use `CustomEvent` with `detail` for browser-native events.
5. Clean up listeners on unmount; use `BroadcastChannel` for cross-tab communication.

**Constraints and Limitations:**

- Event buses can become implicit coupling if handlers are scattered.
- Events are not ordered guarantees unless the bus provides them.
- Overusing events for direct parent-child communication obscures data flow.

### Annotated Code Examples

```tsx
import { useEffect, useReducer } from 'react';

type Event = { type: 'item:added'; id: string; name: string };

function reducer(state: CartItem[], event: Event): CartItem[] {
  switch (event.type) {
    case 'item:added':
      return [...state, { id: event.id, name: event.name }];
    default:
      return state;
  }
}

function Cart() {
  const [items, dispatch] = useReducer(reducer, []);

  useEffect(() => {
    const bus = new EventTarget();
    const handler = (e: Event) => {
      // Normalise CustomEvent to a plain event object
      dispatch((e as CustomEvent<Event>).detail);
    };
    bus.addEventListener('item:added', handler as EventListener);
    return () => bus.removeEventListener('item:added', handler as EventListener);
  }, []);

  return <ul>{items.map((i) => <li key={i.id}>{i.name}</li>)}</ul>;
}
```

**Expected Output:** Dispatching an `item:added` event on the bus appends an item to the cart.

**Why this output:** The event is a plain description of what happened. The reducer translates it into a state change. The cart does not know who emitted the event.

### Real-World Cases

- **Shopping carts:** `item:added`, `item:removed`, `cart:cleared`.
- **Notifications:** `notification:received`.
- **WebSocket messages:** Each message is an event.
- **Cross-tab sync:** Updating the cart in one tab and reflecting it in another.

### References

- MDN — EventTarget — https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
- MDN — BroadcastChannel — https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer

---

## 5. Reducer Pattern

### Definitions

**Core Definition:** The reducer pattern centralizes all state transitions in a single pure function that takes the current state and an action and returns the next state.

**Technical Definition:** A reducer is a pure function `(state, action) => newState` that computes the next state without mutating the previous state and without side effects. React's `useReducer` hook and Redux both implement this pattern. The reducer enforces a single mutation boundary: every state transition is a named action, and the reducer is the only place state changes. This makes state transitions testable in isolation (call the reducer directly) and auditable (every action is a discrete record). Reducers compose: a root reducer delegates to slice reducers.

**Beginner-Friendly Explanation:** A reducer is the single function that decides how state changes. You give it the current state and an action, and it returns the new state. Because it is the only place state changes, it is easy to test and debug.

### Purposes

- To centralize state transition logic in one auditable place.
- To make state changes predictable and testable in isolation.
- To eliminate scattered setters and inconsistent update paths.
- To enable reducer composition for large state trees.
- To provide a foundation for time-travel debugging and event sourcing.

### Syntax Rules and Structure

**General Syntax:**

```ts
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'decrement':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}
```

**Syntax Rules:**

1. Reducers must be pure: no side effects, no external reads, no mutation.
2. Always return a new state object for handled actions.
3. Always return the current state for unhandled actions.
4. Actions must be plain serializable objects with a `type`.
5. Use `useReducer` for component-level reducers; Redux Toolkit for app-level.

**Constraints and Limitations:**

- Reducers cannot perform async work; side effects belong in middleware or thunks.
- Deeply nested state requires careful immutable updates.
- Very large reducers become hard to read; split by feature.

### Annotated Code Examples

```tsx
import { useReducer } from 'react';

type State = { count: number; step: number };
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; step: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.step };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
  return (
    <div>
      <p>Count: {state.count} (step {state.step})</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'setStep', step: 5 })}>Step 5</button>
    </div>
  );
}
```

**Expected Output:** Clicking `+` increments by the current step; setting step to 5 makes subsequent increments jump by 5.

**Why this output:** All transitions are centralized in `reducer`. The component only dispatches named actions; the reducer computes the next state immutably.

### Real-World Cases

- **Forms:** Complex multi-field state with validation.
- **Todos:** Add, toggle, remove, filter.
- **Undo/redo:** Record actions and replay or reverse them.
- **Games:** Move history, scores, and turn state.

### References

- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- React — useReducer — https://react.dev/reference/react/useReducer
- Redux — Reducers — https://redux.js.org/tutorials/fundamentals/part-3-state-actions-reducers

---

## 6. Observer Pattern

### Definitions

**Core Definition:** The observer pattern defines a one-to-many dependency where a subject notifies all registered observers automatically when its state changes.

**Technical Definition:** The observer pattern consists of a **subject** (the state holder) and one or more **observers** (subscribers). The subject maintains a list of observers and calls their update function on change. In React, the observer pattern underlies subscription-based state libraries (MobX's `observer`, Zustand's `subscribe`, Valtio's `subscribe`, TanStack Query's observers). It is the **mechanism** behind reactivity: the state graph notifies the exact computations and components that depend on a changed value, without requiring the consumer to poll or re-read the entire state. The pattern is about **push notification**, not about the shape of state.

**Beginner-Friendly Explanation:** The observer pattern is a subscription: the state says "tell me when you change," and the state calls you when it does. React state libraries use this internally to update only the components that care.

### Purposes

- To notify subscribers automatically when state changes.
- To decouple the state holder from its consumers.
- To enable fine-grained reactivity (only affected observers run).
- To support multiple independent consumers of the same state.
- To provide the mechanism for libraries like MobX, Zustand, and Valtio.

### Syntax Rules and Structure

**Minimal observer implementation:**

```ts
function createObservable<T>(initial: T) {
  let value = initial;
  const observers = new Set<(value: T) => void>();

  return {
    get: () => value,
    set: (next: T) => {
      value = next;
      observers.forEach((fn) => fn(value));
    },
    subscribe: (fn: (value: T) => void) => {
      observers.add(fn);
      return () => observers.delete(fn);
    },
  };
}
```

**React consumption:**

```tsx
function useObservable<T>(observable: ReturnType<typeof createObservable<T>>) {
  const [value, setValue] = useState(observable.get());
  useEffect(() => observable.subscribe(setValue), [observable]);
  return value;
}
```

**Syntax Rules:**

1. The subject maintains a collection of observers.
2. `subscribe` returns an unsubscribe function.
3. Observers are called on every change (or on a selected slice).
4. Always clean up subscriptions on unmount to avoid leaks.
5. In React 18+, `useSyncExternalStore` is the correct way to subscribe to external stores.

**Constraints and Limitations:**

- Without selectors, every observer re-renders on every change.
- Subscriptions without cleanup cause memory leaks.
- Observer order is not guaranteed unless the subject provides it.
- The observer pattern does not define mutation semantics; it only defines notification.

### Annotated Code Examples

```tsx
import { useSyncExternalStore } from 'react';

function createStore<T>(initial: T) {
  let state = initial;
  const listeners = new Set<() => void>();

  return {
    getSnapshot: () => state,
    setState: (next: T) => {
      state = next;
      listeners.forEach((l) => l());
    },
    subscribe: (listener: () => void) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const store = createStore({ count: 0, name: 'Ada' });

function Count() {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <span>{snapshot.count}</span>;
}

function Name() {
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <span>{snapshot.name}</span>;
}
```

**Expected Output:** Changing `count` re-renders both `Count` and `Name`, because both subscribe to the whole store.

**Why this output:** `useSyncExternalStore` subscribes to the whole snapshot. To restrict re-renders to specific slices, the store must expose selector-aware subscriptions (as Zustand and TanStack Query do internally).

### Real-World Cases

- **State libraries:** MobX, Zustand, Valtio, TanStack Query all use observers internally.
- **External stores:** Custom stores bridged with `useSyncExternalStore`.
- **Event emitters:** Node.js-style emitters in browser code.
- **Real-time data:** WebSocket subscribers.

### References

- MDN — Observer pattern (conceptual) — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
- React — useSyncExternalStore — https://react.dev/reference/react/useSyncExternalStore
- MobX — Reactions — https://mobx.js.org/reactions.html

---

## 7. Pub/Sub

### Definitions

**Core Definition:** Publish/subscribe is a messaging pattern where senders (publishers) emit messages to named channels, and receivers (subscribers) listen to channels without knowing who published.

**Technical Definition:** Pub/sub decouples publishers from subscribers through a **broker** or **event bus**. Publishers emit messages to a topic/channel; subscribers register handlers for topics. The broker routes messages to matching subscribers. In React, pub/sub appears as browser-level event buses (`EventTarget`, `BroadcastChannel`), in-process event emitters (`mitt`, `eventemitter3`), and cross-process systems (Redis Pub/Sub, WebSocket fan-out). Pub/sub is a **topology** (who receives what), while the observer pattern is a **relationship** (one subject, many observers). Pub/sub is often used for notifications, analytics, and cross-cutting communication — but it can obscure data flow if used for core state.

**Beginner-Friendly Explanation:** Pub/sub is a message board. Publishers post messages to a topic; subscribers read messages on topics they care about. Neither knows the other exists.

### Purposes

- To decouple event producers from event consumers.
- To allow multiple independent consumers to react to the same message.
- To support cross-component, cross-tab, or cross-service communication.
- To broadcast notifications and analytics events.
- To enable loose coupling in event-driven architectures.

### Syntax Rules and Structure

**In-process pub/sub (mitt):**

```ts
import mitt from 'mitt';

type Events = {
  'cart:add': { id: string };
  'cart:remove': { id: string };
};

const emitter = mitt<Events>();

emitter.on('cart:add', ({ id }) => console.log('added', id));
emitter.emit('cart:add', { id: 'p1' });
```

**Browser pub/sub (BroadcastChannel):**

```ts
const channel = new BroadcastChannel('app');
channel.postMessage({ topic: 'cart:add', payload: { id: 'p1' } });
channel.onmessage = (e) => { /* handle */ };
```

**Syntax Rules:**

1. Define a topic/event map for type safety.
2. Publishers emit to topics; subscribers listen to topics.
3. Always unsubscribe (`off`) on unmount.
4. Use `BroadcastChannel` for cross-tab; use `mitt` for in-process.
5. Keep payloads small and serializable.

**Constraints and Limitations:**

- Pub/sub makes data flow implicit; tracing a state change becomes harder.
- No delivery guarantees unless the broker provides them.
- Ordering is not guaranteed in distributed pub/sub.
- Overuse for direct parent-child communication obscures React's data flow.

### Annotated Code Examples

```tsx
import { useEffect } from 'react';
import mitt from 'mitt';

type Events = { 'notification:show': { text: string } };
const bus = mitt<Events>();

function useNotifications() {
  const [items, setItems] = useState<string[]>([]);
  useEffect(() => {
    const handler = ({ text }: Events['notification:show']) =>
      setItems((prev) => [...prev, text]);
    bus.on('notification:show', handler);
    return () => bus.off('notification:show', handler);
  }, []);
  return items;
}

function Notifier() {
  return <button onClick={() => bus.emit('notification:show', { text: 'Saved!' })}>Save</button>;
}
```

**Expected Output:** Clicking “Save” emits a `notification:show` event; the notification list appends “Saved!”.

**Why this output:** The publisher (`Notifier`) does not know about the subscriber (`useNotifications`). The bus routes the message by topic.

### Real-World Cases

- **Notifications:** Toasts, banners, and alerts.
- **Analytics:** Tracking events across components.
- **Cross-tab sync:** Updating state across browser tabs.
- **Micro-frontends:** Communicating between independent apps.

### References

- mitt — GitHub — https://github.com/developit/mitt
- MDN — BroadcastChannel — https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel
- Redux — Middleware (pub/sub for actions) — https://redux.js.org/understanding/history-and-design/middleware

---

## 8. Event Sourcing

### Definitions

**Core Definition:** Event sourcing is a pattern where the state of an application is derived by replaying an ordered log of events, rather than by storing the current state directly.

**Technical Definition:** In event sourcing, **events are the source of truth**, not the current state. Every state change is recorded as an immutable event in an append-only log. The current state is computed by folding the event log from the beginning (or from a snapshot). Events are named in the past tense (`OrderPlaced`, `ItemAdded`) and contain all data needed to reconstruct the change. Event sourcing enables auditability, time travel, and projection rebuilding, at the cost of complexity: event schema evolution, snapshots for performance, idempotency, and eventual consistency.

**Beginner-Friendly Explanation:** Event sourcing means storing **what happened** instead of **what is**. Instead of saving "the cart has 3 items," you save the events "added apple, added banana, added cherry" and calculate the cart when you need it.

### Purposes

- To provide a complete, immutable audit log of everything that happened.
- To enable time travel and state reconstruction at any point.
- To rebuild projections when business logic changes.
- To decouple write and read models (often paired with CQRS).
- To support event-driven integrations.

### Syntax Rules and Structure

**Event log:**

```ts
type DomainEvent =
  | { type: 'ItemAdded'; id: string; price: number }
  | { type: 'ItemRemoved'; id: string };

const eventLog: DomainEvent[] = [
  { type: 'ItemAdded', id: 'apple', price: 1 },
  { type: 'ItemAdded', id: 'banana', price: 2 },
];
```

**Fold to compute state:**

```ts
function fold(events: DomainEvent[]): { items: string[]; total: number } {
  return events.reduce(
    (state, event) => {
      switch (event.type) {
        case 'ItemAdded':
          return { items: [...state.items, event.id], total: state.total + event.price };
        case 'ItemRemoved':
          return { ...state, items: state.items.filter((i) => i !== event.id) };
        default:
          return state;
      }
    },
    { items: [], total: 0 }
  );
}
```

**Syntax Rules:**

1. Events are immutable, past-tense, and self-describing.
2. The event log is append-only.
3. State is derived by folding the log.
4. Snapshots are used to avoid replaying the entire log.
5. Events must be versioned and evolvable.

**Constraints and Limitations:**

- Complexity is high: event versioning, snapshots, and idempotency.
- Reads are more expensive unless projections are maintained.
- Eventual consistency complicates UI state.
- Overkill for CRUD applications with simple state.

### Annotated Code Examples

```ts
type CartEvent =
  | { type: 'ItemAdded'; id: string; price: number }
  | { type: 'ItemRemoved'; id: string };

function apply(state: { items: string[]; total: number }, event: CartEvent) {
  switch (event.type) {
    case 'ItemAdded':
      return { items: [...state.items, event.id], total: state.total + event.price };
    case 'ItemRemoved':
      return { ...state, items: state.items.filter((i) => i !== event.id) };
  }
}

const events: CartEvent[] = [
  { type: 'ItemAdded', id: 'apple', price: 1 },
  { type: 'ItemAdded', id: 'banana', price: 2 },
  { type: 'ItemRemoved', id: 'apple' },
];

const cart = events.reduce(apply, { items: [], total: 0 });
// { items: ['banana'], total: 3 }
```

**Expected Output:** The final cart contains `banana` and a total of 3, reconstructed from the event log.

**Why this output:** State is a fold over events. Removing `apple` does not subtract its price in this simplified example (the removal event would normally carry the price or the fold would look up the original event).

### Real-World Cases

- **Financial ledgers:** Every transaction is an event.
- **E-commerce orders:** Placed, paid, shipped, delivered.
- **Collaborative editors:** Every edit is an event.
- **Audit-critical systems:** Healthcare, compliance, and finance.

### References

- Martin Fowler — Event Sourcing — https://martinfowler.com/eaaDev/EventSourcing.html
- Greg Young — Event Sourcing (video) — https://www.youtube.com/watch?v=8JKjvY4etTY
- Microsoft — Event Sourcing pattern — https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

---

## 9. CQRS

### Definitions

**Core Definition:** Command Query Responsibility Segregation (CQRS) separates the write model (commands) from the read model (queries), often using different data stores and models for each.

**Technical Definition:** CQRS splits operations into **commands** (mutations that return no data) and **queries** (reads that never mutate). The write side handles commands and produces events or updates the write store; the read side is optimized for queries and may be a separate projection. CQRS is often paired with event sourcing: commands append events; projections fold events into read-optimized views. On the frontend, CQRS manifests as separate **write stores** (mutations, optimistic updates) and **read stores** (query caches), or as separate mutation and query endpoints. The pattern improves scalability and read/write optimisation at the cost of eventual consistency and complexity.

**Beginner-Friendly Explanation:** CQRS means using different models for reading and writing. Writes go through commands; reads go through queries. Often the read side is a separate, optimized copy of the data.

### Purposes

- To optimise reads and writes independently.
- To separate mutation complexity from query performance.
- To enable event-driven architectures and projections.
- To support high-read, low-write workloads (and vice versa).
- To allow eventual consistency where strong consistency is not required.

### Syntax Rules and Structure

**Command (write):**

```ts
type Command =
  | { type: 'AddItem'; id: string; price: number }
  | { type: 'RemoveItem'; id: string };

function executeCommand(command: Command) {
  // Validate, append event, publish
}
```

**Query (read):**

```ts
function getCartTotal(cartId: string): Promise<number> {
  // Read from the optimized read model
}
```

**Frontend separation:**

```ts
// Write store (commands)
const useCartCommands = create((set) => ({
  addItem: (item) => set((s) => ({ items: [...s.items, item] })),
}));

// Read store (queries / projections)
const useCartQuery = () => useQuery({ queryKey: ['cart'], queryFn: fetchCart });
```

**Syntax Rules:**

1. Commands mutate state and return no data (or only an acknowledgement).
2. Queries read state and never mutate.
3. The read model may lag behind the write model (eventual consistency).
4. Commands and queries use separate models, not necessarily separate databases.
5. In the frontend, keep mutation state and query cache separate; invalidate query caches after commands.

**Constraints and Limitations:**

- Eventual consistency requires UI handling (loading, stale data, reconciliation).
- Two models mean two places to maintain and keep in sync.
- Overkill for simple CRUD applications with low complexity.

### Annotated Code Examples

```tsx
// Command side
function useAddToCart() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (item: Item) => api.addToCart(item),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['cart'] }),
  });
}

// Query side
function CartTotal() {
  const { data } = useQuery({ queryKey: ['cart'], queryFn: fetchCart });
  return <span>{data?.total ?? 0}</span>;
}
```

**Expected Output:** Adding an item executes the command and invalidates the read cache; the query refetches and displays the new total.

**Why this output:** The write side (mutation) and read side (query) are separate models. The command invalidates the query cache to reconcile the read model.

### Real-World Cases

- **E-commerce:** Write-optimised order commands; read-optimised product catalogs.
- **Banking:** Commands for transfers; queries for balances.
- **Collaborative apps:** Commands for edits; queries for projections.
- **Analytics dashboards:** Separate read models for reporting.

### References

- Martin Fowler — CQRS — https://martinfowler.com/bliki/CQRS.html
- Microsoft — CQRS pattern — https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs
- Greg Young — CQRS Documents — https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf

---

## 10. State Normalization

### Definitions

**Core Definition:** State normalization is the practice of flattening nested state into ID-indexed collections so each entity is stored exactly once.

**Technical Definition:** Normalization eliminates duplication by storing entities in **tables** keyed by unique IDs (`{ byId: { '1': {...} }, allIds: ['1'] }`) and referencing them by ID rather than embedding copies. This mirrors database normalization: one authoritative copy per entity, with relationships expressed as IDs. React libraries provide adapters (`createEntityAdapter` in Redux Toolkit, `normalizr` for transformations) to manage the `byId`/`allIds` shape and generate selectors. Normalization simplifies updates (edit one entity, not every copy) and prevents synchronisation bugs.

**Beginner-Friendly Explanation:** Normalization means storing each item once and referring to it by ID, instead of copying it into every list that mentions it.

### Purposes

- To eliminate duplicated entities and synchronisation bugs.
- To simplify updates (edit one entity, not every copy).
- To enable efficient lookup by ID.
- To support relationships between entities via IDs.
- To make selectors and joins straightforward.

### Syntax Rules and Structure

**Normalized shape:**

```ts
{
  users: {
    byId: { '1': { id: '1', name: 'Ada' } },
    allIds: ['1'],
  },
  posts: {
    byId: { 'p1': { id: 'p1', title: 'Hello', authorId: '1' } },
    allIds: ['p1'],
  },
}
```

**Redux Toolkit entity adapter:**

```ts
const usersAdapter = createEntityAdapter<User>();
const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    usersUpdated: usersAdapter.updateOne,
  },
});
```

**Syntax Rules:**

1. Store each entity once, keyed by ID.
2. Reference related entities by ID, not embedded objects.
3. Use adapters (`createEntityAdapter`, `normalizr`) to manage the shape.
4. Use selectors to join entities for the UI.
5. Normalize at the API boundary, not in components.

**Constraints and Limitations:**

- Joins must be computed in selectors (extra work at read time).
- The normalized shape is less convenient to inspect than nested data.
- Not every nested structure needs normalization; only entities with identity and reuse.

### Annotated Code Examples

```ts
const usersAdapter = createEntityAdapter<User>();

const usersSlice = createSlice({
  name: 'users',
  initialState: usersAdapter.getInitialState(),
  reducers: {
    userAdded: usersAdapter.addOne,
    userUpdated: usersAdapter.updateOne,
  },
});

export const { selectAll: selectAllUsers, selectById: selectUserById } =
  usersAdapter.getSelectors((state) => state.users);
```

**Expected Output:** `selectAllUsers` returns all users; `selectUserById(state, '1')` returns one user; updates via `userUpdated` touch only one entity.

**Why this output:** The adapter maintains `byId` and `allIds`, and generates memoized selectors.

### Real-World Cases

- **Social feeds:** Posts, users, comments, and likes.
- **E-commerce:** Products, categories, carts, and orders.
- **Project management:** Projects, tasks, users, and tags.
- **Chat:** Conversations, messages, and participants.

### References

- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Essentials — Normalizing State — https://redux.js.org/tutorials/essentials/part-6-performance-normalization
- normalizr — GitHub — https://github.com/paularmstrong/normalizr

---

## 11. State Persistence

### Definitions

**Core Definition:** State persistence is the practice of saving state to durable storage (browser storage or a server) so it survives reloads and sessions.

**Technical Definition:** Persistence writes state to a storage backend: `localStorage` (permanent, ~5 MB), `sessionStorage` (tab-scoped), IndexedDB (browser-managed, large), cookies (small, sent to server), or a server-side store. On the frontend, persistence is typically scoped to **client state that should survive reloads** (theme, preferences, drafts) and **server cache that should be rehydrated** (query cache). Persistence must handle serialization, versioning, and exclusion of sensitive data. Libraries provide middleware (`zustand/persist`, Redux Persist, TanStack Query persistence plugins).

**Beginner-Friendly Explanation:** Persistence means saving state so it is still there after a reload. You save to the browser's storage or to the server, and load it back when the app starts.

### Purposes

- To preserve user preferences and settings across sessions.
- To save form drafts and wizard progress.
- To rehydrate server caches for faster startup.
- To support offline-first experiences.
- To keep non-sensitive session data across reloads.

### Syntax Rules and Structure

**Manual persistence:**

```ts
useEffect(() => {
  const saved = localStorage.getItem('settings');
  if (saved) setSettings(JSON.parse(saved));
}, []);

useEffect(() => {
  localStorage.setItem('settings', JSON.stringify(settings));
}, [settings]);
```

**Zustand persist:**

```ts
const useSettingsStore = create(
  persist(
    (set) => ({ theme: 'light', setTheme: (theme) => set({ theme }) }),
    { name: 'settings', partialize: (s) => ({ theme: s.theme }) }
  )
);
```

**TanStack Query persistence:**

```ts
import { persistQueryClient } from '@tanstack/query-persist-client-core';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

persistQueryClient({
  queryClient,
  persister: createSyncStoragePersister({ storage: localStorage }),
  maxAge: 1000 * 60 * 60 * 24,
});
```

**Syntax Rules:**

1. Read storage in an effect or lazy initializer, never during server render.
2. Persist only non-sensitive, serializable state.
3. Use `partialize` to exclude actions, transient data, and secrets.
4. Version the storage key and provide migration.
5. Handle parse errors gracefully (corrupt or outdated data).

**Constraints and Limitations:**

- `localStorage` is synchronous, ~5 MB, and XSS-accessible.
- Persisted data can become stale if the schema changes.
- Persisting server state can conflict with cache freshness policies.

### Real-World Cases

- **Theme and preferences:** Persisted user settings.
- **Draft forms:** Saving form progress.
- **Wizard state:** Resuming multi-step flows.
- **Query cache:** Faster startup with rehydrated server data.

### References

- MDN — Window.localStorage — https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- Zustand — Persist middleware — https://zustand.docs.pmnd.rs/middlewares/persist
- TanStack Query — Persistence — https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient

---

## 12. State Hydration

### Definitions

**Core Definition:** State hydration is the process of restoring persisted or server-rendered state into the client store at startup.

**Technical Definition:** Hydration has two related meanings. In **SSR hydration**, the server sends a serialized state snapshot alongside the HTML, and the client initializes its store with that snapshot before rendering, so the first client render matches the server and no data is re-fetched. In **persistence hydration**, the client reads stored state from `localStorage` or IndexedDB and initializes the store with it. Hydration must be **deterministic and side-effect-free** in SSR, and must handle **mismatches** between server and client state (e.g., persisted preferences that differ from server defaults) without producing hydration warnings. Libraries provide dehydration/rehydration APIs (`persistQueryClient`, Zustand's `onRehydrateStorage`, Redux Persist's `PersistGate`, Apollo's `cache.extract`/`cache.restore`).

**Beginner-Friendly Explanation:** Hydration is loading saved state back into the app when it starts. On the server-rendered page, it means giving the browser the same data the server used so the first render matches.

### Purposes

- To make server-rendered and client-rendered output match (avoid hydration mismatches).
- To restore persisted preferences and drafts.
- To avoid re-fetching data the server already fetched.
- To provide a seamless "resume where you left off" experience.
- To seed the cache for fast first render.

### Syntax Rules and Structure

**SSR dehydration/rehydration (TanStack Query):**

```ts
// Server
const dehydratedState = dehydrate(queryClient);

// Client
const queryClient = new QueryClient();
hydrate(queryClient, dehydratedState);
```

**Zustand rehydration callback:**

```ts
persist(creator, {
  name: 'settings',
  onRehydrateStorage: () => (state, error) => {
    if (error) console.error('hydration failed', error);
  },
});
```

**Manual hydration guard:**

```tsx
function useHydrated() {
  const [hydrated, setHydrated] = useState(false);
  useEffect(() => setHydrated(true), []);
  return hydrated;
}
```

**Syntax Rules:**

1. Dehydrate on the server after all queries load; rehydrate on the client before render.
2. Use the library's hydration APIs rather than reading storage directly in render.
3. Handle loading states during asynchronous hydration.
4. Avoid rendering persisted-only UI until hydration completes (use a `hydrated` flag).
5. Keep the dehydrated payload small; strip large or sensitive data.

**Constraints and Limitations:**

- Asynchronous hydration can cause a flash of default state.
- Server and client state can diverge (e.g., persisted theme); handle mismatches explicitly.
- Hydration payloads increase HTML size; strip large data.

### Real-World Cases

- **SSR apps:** Dehydrating and rehydrating query caches.
- **Persisted preferences:** Rehydrating theme and locale.
- **Draft forms:** Restoring form drafts after reload.
- **Offline-first apps:** Rehydrating IndexedDB state at startup.

### References

- TanStack Query — SSR and Hydration — https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- Zustand — Persist: hydration — https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot

---

## 13. State Migration

### Definitions

**Core Definition:** State migration is the process of transforming persisted state from an older schema version into the current schema version when the application loads.

**Technical Definition:** Persisted state outlives the code that wrote it. When the shape of state changes (fields added, renamed, removed, or restructured), old persisted data must be **migrated** to the current shape before it is hydrated. Migrations are versioned: the store records a `version` alongside the persisted state, and on load compares the stored version to the current version. If they differ, a `migrate` function transforms the old state. Migrations must be **idempotent**, **deterministic**, and **defensive** (validate the input, provide defaults, and discard unrecoverable data). Libraries provide migration hooks: Zustand's `version` + `migrate`, Redux Persist's `migrate` (with `createMigrate`), and TanStack Query's `persister` with `buster`/`maxAge`.

**Beginner-Friendly Explanation:** Migration means upgrading saved data when your app changes. If you saved an old format, the app converts it to the new format before using it.

### Purposes

- To keep persisted state compatible with the current schema.
- To avoid crashes from missing or renamed fields.
- To discard unrecoverable or corrupted data safely.
- To evolve state shape without losing user data.
- To support long-lived apps across many releases.

### Syntax Rules and Structure

**Zustand version + migrate:**

```ts
persist(creator, {
  name: 'settings',
  version: 2,
  migrate: (persisted, version) => {
    if (version === 0) {
      return { ...persisted, newField: 'default' };
    }
    if (version === 1) {
      return { ...persisted, renamedField: persisted.oldField };
    }
    return persisted as Settings;
  },
});
```

**Redux Persist `createMigrate`:**

```ts
import { createMigrate } from 'redux-persist';

const migrations = {
  1: (state) => ({ ...state, newField: 'default' }),
  2: (state) => ({ ...state, renamed: state.old }),
};

persistReducer(
  { key: 'root', version: 2, migrate: createMigrate(migrations) },
  rootReducer
);
```

**Syntax Rules:**

1. Store a `version` with the persisted state.
2. Provide a `migrate` function that handles each version step.
3. Validate the input; discard or reset unrecoverable data.
4. Make migrations deterministic and idempotent.
5. Test migrations with fixtures of old state.

**Constraints and Limitations:**

- Migrations must be written before shipping the schema change.
- Chained migrations can become complex; keep steps small and documented.
- Migrating large state can slow startup; consider background migration.

### Annotated Code Examples

```ts
const useSettings = create(
  persist(
    (set) => ({
      theme: 'light' as 'light' | 'dark',
      locale: 'en',
      setTheme: (theme: 'light' | 'dark') => set({ theme }),
    }),
    {
      name: 'settings',
      version: 2,
      migrate: (persisted: any, version: number) => {
        if (version < 2) {
          return { theme: persisted.theme ?? 'light', locale: persisted.locale ?? 'en' };
        }
        return persisted;
      },
    }
  )
);
```

**Expected Output:** A persisted state with version 0 or 1 is upgraded to version 2 with defaults for any missing fields.

**Why this output:** `migrate` receives the persisted state and its version, and returns the current shape. Missing fields are filled with defaults.

### Real-World Cases

- **Long-lived apps:** Evolving settings shape across releases.
- **Feature rollouts:** Adding new persisted fields with defaults.
- **Renamed fields:** Migrating `oldField` to `newField`.
- **Corrupted storage:** Resetting to defaults when data is invalid.

### References

- Zustand — Persist: migrations — https://zustand.docs.pmnd.rs/integrations/persisting-store-data#migrations
- Redux Persist — Migrations — https://github.com/persist-org/redux-persist#state-reconciler-and-migrations
- TanStack Query — Persistence — https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient

---

## 14. State Management Best Practices

### Definitions

**Core Definition:** State management best practices are the guidelines for choosing patterns, placing state, handling mutations, and managing durability to build maintainable applications.

**Technical Definition:** Best practices combine **classification before tooling** (identify owner, lifetime, and scope), **colocation by default** (lift only when sharing is real), **single source of truth per fact** (no duplicated state), **derived not stored** (compute totals, filters, and validity), **reducers for complex transitions** (centralise multi-field logic), **state machines for finite event-driven flows** (auth, wizards), **normalization for entity graphs** (avoid duplication), **versioned persistence with migration** (never hydrate unvalidated state), **hydration discipline** (deterministic, mismatch-safe), and **server state in dedicated libraries** (never duplicated into client stores).

**Beginner-Friendly Explanation:** Good state management means choosing the right pattern for each problem, putting state where it belongs, never storing what you can calculate, and versioning anything you save.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Classify before choosing** | Name owner, lifetime, and scope in one sentence |
| **Colocate by default** | Lift only when a second consumer genuinely needs it |
| **One source of truth per fact** | Two owners will eventually disagree |
| **Derive, don't store** | Compute totals, filters, and validity during render |
| **Use reducers for complex transitions** | Centralise multi-field state logic |
| **Use state machines for finite flows** | Eliminate impossible states in auth and wizards |
| **Use event-driven state for decoupling** | Events for cross-component and cross-tab communication |
| **Normalize entity graphs** | Store each entity once, reference by ID |
| **Persist selectively** | Use `partialize`; exclude secrets and transient data |
| **Version persisted state** | Provide `version` and `migrate` |
| **Hydrate deterministically** | Use library hydration APIs; handle mismatches |
| **Keep server state in query libraries** | Do not duplicate server data into client stores |
| **Test reducers and machines in isolation** | Pure functions are the easiest to test |
| **Document the chosen patterns** | Make the architecture reviewable |

### Annotated Code Examples

**✅ Good Practices:**

```ts
// Colocated UI state
function Search() {
  const [query, setQuery] = useState('');
  // ...
}

// Reducer for complex transitions
function reducer(state: FormState, action: FormAction): FormState { /* ... */ }

// Normalized entities
const usersAdapter = createEntityAdapter<User>();

// Versioned persistence
persist(creator, {
  name: 'settings',
  version: 2,
  migrate: (persisted, version) => { /* upgrade */ },
  partialize: (s) => ({ theme: s.theme }),
});

// Server state in TanStack Query
const { data } = useQuery({ queryKey: ['user', id], queryFn: fetchUser });
```

**❌ Bad Practices:**

```ts
// ❌ Storing derived state
const [total, setTotal] = useState(0);

// ❌ Duplicating server data in a client store
const useStore = create((set) => ({ user: null, setUser: (u) => set({ user: u }) }));

// ❌ Persisting unversioned state
localStorage.setItem('state', JSON.stringify(state));

// ❌ Persisting secrets
persist(creator, { name: 'auth' }); // includes tokens

// ❌ Over-lifting trivial UI state
const [isModalOpen, setModalOpen] = useState(false); // at app root
```

**Expected Output:** The good practices produce predictable, maintainable, performant state. The bad practices cause stale derived data, duplicated server state, incompatible persisted data, security risks, and unnecessary re-renders.

**Why this output:** Stored derived state drifts; duplicated server state breaks caching; unversioned persistence crashes on schema changes; persisted secrets are an XSS risk; over-lifting spreads re-renders.

### Real-World Cases

- **All React applications:** These practices apply universally.
- **SSR apps:** Hydration discipline and versioned persistence.
- **Large apps:** Reducers, state machines, and normalization.
- **Event-driven apps:** Pub/sub for cross-cutting communication.

### References

- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- Redux — Style Guide — https://redux.js.org/style-guide/
- Zustand — Persist — https://zustand.docs.pmnd.rs/middlewares/persist
- TanStack Query — SSR — https://tanstack.com/query/latest/docs/framework/react/guides/ssr

---

## References

- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- React — useReducer — https://react.dev/reference/react/useReducer
- React — useSyncExternalStore — https://react.dev/reference/react/useSyncExternalStore
- React — Choosing the State Structure — https://react.dev/learn/choosing-the-state-structure
- React — Preserving and Resetting State — https://react.dev/learn/preserving-and-resetting-state
- Steve Kinney — Colocation of State — https://stevekinney.com/courses/react-performance/colocation-of-state
- XState — Official Documentation — https://stately.ai/docs/xstate
- Stately — State Machines and Statecharts — https://stately.ai/docs/state-machines-and-statecharts
- David Harel — Statecharts: A Visual Formalism for Complex Systems — https://www.sciencedirect.com/science/article/pii/0167642387900359
- MDN — EventTarget — https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
- MDN — BroadcastChannel — https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel
- MDN — Window.localStorage — https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- Martin Fowler — Event Sourcing — https://martinfowler.com/eaaDev/EventSourcing.html
- Martin Fowler — CQRS — https://martinfowler.com/bliki/CQRS.html
- Microsoft — Event Sourcing pattern — https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing
- Microsoft — CQRS pattern — https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs
- Redux Toolkit — createEntityAdapter — https://redux-toolkit.js.org/api/createEntityAdapter
- Redux Essentials — Normalizing State — https://redux.js.org/tutorials/essentials/part-6-performance-normalization
- Redux — Style Guide — https://redux.js.org/style-guide/
- Zustand — Persist middleware — https://zustand.docs.pmnd.rs/middlewares/persist
- Zustand — Persist: migrations — https://zustand.docs.pmnd.rs/integrations/persisting-store-data#migrations
- Redux Persist — GitHub — https://github.com/persist-org/redux-persist
- TanStack Query — SSR and Hydration — https://tanstack.com/query/latest/docs/framework/react/guides/ssr
- TanStack Query — Persistence — https://tanstack.com/query/latest/docs/framework/react/plugins/persistQueryClient
- React — hydrateRoot — https://react.dev/reference/react-dom/client/hydrateRoot
- mitt — GitHub — https://github.com/developit/mitt
- normalizr — GitHub — https://github.com/paularmstrong/normalizr
- pproenca/dot-skills — State Colocation with Consumers — https://github.com/pproenca/dot-skills
- Frontend Engineering — Categories of State — https://github.com/ualiyou/frontend-engineering