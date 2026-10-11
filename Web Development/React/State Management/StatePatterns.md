# React State Patterns — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React state patterns are reusable architectural solutions for managing, sharing, and updating state in React applications.

**Technical Definition:** State patterns in React are established structural approaches for deciding where state should live, how it should be shaped, how it should be updated, and how it should be shared between components. These patterns address the fundamental challenge that state placement is the single largest determinant of application complexity — the same feature can be elegant or tortuous depending on whether state lives in the right component and flows in the right direction. React provides built-in primitives (`useState`, `useReducer`, Context) that implement these patterns, while external libraries (Immer, XState) extend them for specialised needs. The core principle is: state should be minimised, colocated, derived where possible, and lifted only when genuinely shared.

**Beginner-Friendly Explanation:** State patterns are like recipes for deciding where to put your data and how to change it. Should this piece of information live inside one component, or should it be shared with others? Should you store it, or just calculate it when needed? These patterns help you answer those questions so your app stays clean and bug-free.

### Key Characteristics

- **Colocation by default:** State should live as close as possible to the components that use it.
- **Lift only when necessary:** State moves up the tree only when multiple distant components genuinely need the same data.
- **Derive, don't duplicate:** Values that can be computed from existing state or props should never be stored separately.
- **Immutable updates:** State must be replaced, not mutated, so React can detect changes.
- **Normalise complex data:** Nested or duplicated data should be flattened into ID-indexed structures.
- **Model with machines:** Complex, multi-step UI flows benefit from explicit state machines.

### Prerequisites

- React function components, `useState`, and `useReducer`.
- JavaScript objects, arrays, spread operator, and destructuring.
- Understanding of props and one-way data flow.
- Familiarity with React Context.

### Related Programming Areas

- **State Fundamentals** — the raw mechanics of `useState` and `useReducer`.
- **Component Patterns** — container/presentational, compound components.
- **External State Libraries** — Redux, Zustand, Jotai for global state.
- **Server State** — TanStack Query, SWR for async data.

---

## 1. Lifting State Up

### Definitions

**Core Definition:** Lifting state up is the pattern of moving shared state from child components to their closest common ancestor so multiple siblings can access it.

**Technical Definition:** When two or more components need to reflect the same changing data, the state is removed from each child and moved to their nearest common parent. The parent then passes the state down via props and passes callback functions down so children can request changes. This keeps a single source of truth and ensures components stay synchronised.

**Beginner-Friendly Explanation:** When two components need to share information, you move that information up to their parent. The parent holds the data and gives it to both children.

### Purposes

- To enable multiple components to share the same state.
- To maintain a single source of truth for shared data.
- To keep sibling components synchronised.
- To enable one component to influence another's rendering.

### Syntax Rules and Structure

**General Syntax:**

```jsx
function Parent() {
  const [shared, setShared] = useState(initialValue);
  return (
    <>
      <ChildA value={shared} onChange={setShared} />
      <ChildB value={shared} />
    </>
  );
}
```

**Component Breakdown:**

| Component | Description |
|-----------|-------------|
| `Parent` | Nearest common ancestor that owns the state. |
| `ChildA` | Receives state and a setter callback. |
| `ChildB` | Receives state for display. |

**Syntax Rules:**

1. State is declared in the nearest common ancestor.
2. The state value and its setter are passed down as props.
3. Children must not maintain their own copy of the lifted state.

**Constraints and Limitations:**

- Over-lifting causes unnecessary re-renders across the tree.
- Prop drilling becomes painful if the tree is deep.

### Annotated Code Examples

```jsx
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      />
      <Panel
        title="Contact"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      />
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section>
      <h3>{title}</h3>
      {isActive ? <p>{children}</p> : <button onClick={onShow}>Show</button>}
    </section>
  );
}
```

**Expected Output:** Clicking "Show" on one panel opens it and closes the other.

**Why this output:** `activeIndex` lives in `Accordion`, the nearest common parent. Both `Panel` instances receive `isActive` and `onShow`, so they stay synchronised.

### Real-World Cases

- **Accordions and tabs:** Only one panel open at a time.
- **Master-detail views:** Selecting a list item shows details in a sibling pane.
- **Forms with dependent fields:** Changing one field updates another.

### References

- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- CoreUI — How to lift state up in React — https://coreui.io

---

## 2. Colocating State

### Definitions

**Core Definition:** Colocating state is the practice of keeping state as close as possible to the components that actually use it.

**Technical Definition:** State colocation means placing state in the lowest component in the tree that needs it, rather than lifting it prematurely. State should be colocated with the components that consume it. State lifted too high forces intermediate components to forward props they do not use, and every parent in the chain re-renders when the state changes, even though only the leaf component reads it. Moving state to the component that consumes it eliminates the forwarding chain and confines re-renders.

**Beginner-Friendly Explanation:** Keep state in the component that needs it. Do not put it at the top of the app just because that feels organised.

### Purposes

- To minimise the number of components that re-render when state changes.
- To avoid unnecessary prop forwarding through intermediate components.
- To make components easier to reason about and test in isolation.
- To reduce the surface area for bugs.

### Syntax Rules and Structure

**✅ Colocated (Correct):**

```jsx
function Dashboard() {
  return (
    <div>
      <Widget1 />  {/* Each widget owns its state */}
      <Widget2 />
    </div>
  );
}

function Widget1() {
  const [data, setData] = useState(null);
  // Only Widget1 re-renders when data changes
}
```

**❌ Over-Lifted (Incorrect):**

```jsx
function Dashboard() {
  const [widget1Data, setWidget1Data] = useState(null);
  const [widget2Data, setWidget2Data] = useState(null);
  // Every widget re-renders when any state changes
}
```

**Syntax Rules:**

1. Declare state in the component that uses it.
2. Lift only when multiple components genuinely need the same data.
3. Use `React.memo` to contain re-renders when state must be lifted.

### Annotated Code Examples

```jsx
function ProductPage() {
  return (
    <div>
      <ProductDetails />
      <ProductReviews />  {/* Reviews manage their own sort state */}
    </div>
  );
}

function ProductReviews() {
  const [sortOrder, setSortOrder] = useState('newest');
  // Sort order is local to reviews — no other component needs it
}
```

**Expected Output:** Changing the review sort order does not re-render `ProductDetails`.

**Why this output:** `sortOrder` is colocated with `ProductReviews`. It is not lifted to `ProductPage`, so only `ProductReviews` re-renders.

### Real-World Cases

- **Dashboard widgets:** Each widget manages its own loading and data state.
- **Independent forms:** Each form section manages its own fields.
- **Sidebar toggles:** The sidebar owns its open/closed state.

### References

- Steve Kinney — Colocation of State — https://stevekinney.com
- tessl-labs — Frontend State Management — https://tessl.io

---

## 3. Derived State

### Definitions

**Core Definition:** Derived state is data that can be computed from existing state or props, and therefore should not be stored as separate state.

**Technical Definition:** Derived state is any value that can be calculated from existing state during render. Storing derived state redundantly leads to synchronisation bugs, extra memory usage, and unnecessary re-renders. React recommends computing derived values during render. If a value can be calculated from the component's props or its existing state variables during rendering, it should not be placed in state.

**Beginner-Friendly Explanation:** If you can figure something out from what you already know, do not write it down separately. Just calculate it when you need it.

### Purposes

- To avoid synchronisation bugs from redundant state.
- To reduce the number of state variables and re-renders.
- To keep a single source of truth for data.
- To simplify component logic.

### Syntax Rules and Structure

**❌ Redundant (Anti-Pattern):**

```jsx
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);  // Must be kept in sync manually
```

**✅ Derived (Correct):**

```jsx
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

**Syntax Rules:**

1. Compute derived values during render, not in state.
2. Use `useMemo` only for expensive computations.
3. Never store a value that can be calculated.

### Annotated Code Examples

```jsx
function ShoppingCart() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple', price: 1.5, quantity: 2 },
    { id: 2, name: 'Bread', price: 3.0, quantity: 1 },
  ]);

  // Derived during render — always correct
  const total = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  const itemCount = items.reduce((sum, i) => sum + i.quantity, 0);

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

**Expected Output:** Items: 3, Total: $6.00.

**Why this output:** `total` and `itemCount` are computed from `items` on every render. They can never go out of sync because they are not stored separately.

### Real-World Cases

- **Cart totals:** Summing item prices.
- **Filtered lists:** Filtering todos by status.
- **Validation results:** Checking if a form is valid.
- **Formatted dates:** Converting timestamps to display strings.

### References

- React — Choosing the State Structure — https://18.react.dev/learn/choosing-the-state-structure
- tessl-labs — Derived State — https://tessl.io

---

## 4. Computed State

### Definitions

**Core Definition:** Computed state is a synonym for derived state — values calculated from existing state rather than stored independently.

**Technical Definition:** Computed state refers to values produced by pure functions of existing state and props. The term emphasises the act of computation: rather than storing a value and updating it whenever dependencies change, you compute it fresh on each render. The recommended tool for expensive computations is `useMemo`, which caches the result between renders.

**Beginner-Friendly Explanation:** Computed state is just another name for derived state. You compute it when you need it instead of storing it.

### Purposes

- To avoid stale data from manually synchronised state.
- To reduce state complexity.
- To leverage `useMemo` for performance when computation is expensive.

### Syntax Rules and Structure

```jsx
const expensiveResult = useMemo(
  () => computeExpensiveValue(a, b),
  [a, b]
);
```

**Syntax Rules:**

1. Compute during render for simple values.
2. Use `useMemo` for expensive computations.
3. Include all dependencies in the dependency array.

### Annotated Code Examples

```jsx
function SearchResults({ items, query }) {
  // Computed during render — derived from props
  const filtered = useMemo(
    () => items.filter((item) =>
      item.name.toLowerCase().includes(query.toLowerCase())
    ),
    [items, query]
  );

  return (
    <ul>
      {filtered.map((item) => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

**Expected Output:** A filtered list of items matching the query.

**Why this output:** `filtered` is computed from `items` and `query`. `useMemo` caches the result until either dependency changes.

### Real-World Cases

- **Search filtering:** Filtering large datasets.
- **Sorting:** Sorting table rows by column.
- **Aggregation:** Computing averages, sums, and counts.

### References

- React — Choosing the State Structure — https://18.react.dev/learn/choosing-the-state-structure

---

## 5. State Machines

### Definitions

**Core Definition:** A state machine is a model of computation where a system can be in exactly one of a finite number of states, and transitions between states are triggered by events.

**Technical Definition:** A finite state machine (FSM) defines a set of possible states, a set of events, and a transition function that maps (state, event) pairs to next states. In React, state machines provide a structured way to manage complex UI logic by making invalid states impossible. XState is the leading library for implementing state machines and statecharts in React. Instead of letting the state change freely, FSM allows you to define a set of possible states and describe how the app data can move from one well-defined state to another.

**Beginner-Friendly Explanation:** A state machine is a way to define exactly what states your UI can be in and how it moves between them. For example, a form can be "idle", "submitting", "success", or "error" — and nothing else.

### Purposes

- To eliminate invalid state combinations.
- To make complex UI logic explicit and visual.
- To simplify debugging by making transitions traceable.
- To model multi-step workflows and authentication flows.

### Syntax Rules and Structure

**General Syntax (XState v5):**

```jsx
import { createMachine } from 'xstate';

const machine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});
```

**Syntax Rules:**

1. Define `initial` state.
2. Define all possible `states`.
3. Define transitions via `on` events.
4. Use `context` for extended state.

**Constraints and Limitations:**

- Requires learning a new mental model.
- May be overkill for simple components.

### Annotated Code Examples

```jsx
import { createMachine, interpret } from 'xstate';

const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  context: { data: null, error: null },
  states: {
    idle: { on: { FETCH: 'loading' } },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: { target: 'success', actions: 'setData' },
        onError: { target: 'failure', actions: 'setError' },
      },
    },
    success: { on: { FETCH: 'loading' } },
    failure: { on: { RETRY: 'loading' } },
  },
});
```

**Expected Output:** The machine transitions through idle → loading → success/failure.

**Why this output:** The FSM explicitly defines every valid state and transition. It is impossible to be in `loading` and `success` simultaneously.

### Real-World Cases

- **Authentication flows:** Login, logout, token refresh.
- **Checkout wizards:** Step 1, step 2, step 3, confirmation.
- **Data fetching:** Idle, loading, success, error.
- **Product tours:** Multi-step onboarding flows.

### References

- XState — Official Documentation — https://xstate.js.org
- DHTMLX — Using XState in React — https://dhtmlx.com

---

## 6. State Charts

### Definitions

**Core Definition:** A statechart is an extension of state machines that adds hierarchy, concurrency, and communication between machines.

**Technical Definition:** Statecharts are a visual formalism that extends finite state machines with nested (hierarchical) states, parallel (concurrent) regions, and guards. XState supports statecharts natively, allowing complex application logic to be organised into visual structures. When a single state machine is not enough, XState introduces actors — multiple machines that can run in parallel and communicate with each other.

**Beginner-Friendly Explanation:** Statecharts are like state machines but bigger. They let you nest states inside other states and run multiple machines at the same time.

### Purposes

- To model complex, hierarchical UI logic.
- To run independent state machines in parallel.
- To visualise application behaviour.
- To manage communication between separate machines.

### Syntax Rules and Structure

```jsx
const machine = createMachine({
  type: 'parallel',
  states: {
    editor: { /* editor state machine */ },
    preview: { /* preview state machine */ },
  },
});
```

**Syntax Rules:**

1. Use `type: 'parallel'` for concurrent regions.
2. Nest states for hierarchical behaviour.
3. Use actors for communication between machines.

### Annotated Code Examples

```jsx
const appMachine = createMachine({
  type: 'parallel',
  states: {
    auth: {
      initial: 'loggedOut',
      states: {
        loggedOut: { on: { LOGIN: 'loggedIn' } },
        loggedIn: { on: { LOGOUT: 'loggedOut' } },
      },
    },
    theme: {
      initial: 'light',
      states: {
        light: { on: { TOGGLE: 'dark' } },
        dark: { on: { TOGGLE: 'light' } },
      },
    },
  },
});
```

**Expected Output:** Auth and theme states change independently.

**Why this output:** The `parallel` type runs both regions concurrently. Changing the theme does not affect auth state.

### Real-World Cases

- **Complex dashboards:** Multiple independent widgets.
- **Multi-user collaboration:** Presence, editing, and chat as parallel machines.
- **Game state:** Player, enemies, and environment as separate machines.

### References

- XState — Statecharts — https://xstate.js.org
- DHTMLX — Statecharts in XState — https://dhtmlx.com

---

## 7. Finite State Machines

### Definitions

**Core Definition:** A finite state machine (FSM) is a mathematical model of computation with a finite number of states and deterministic transitions.

**Technical Definition:** An FSM is defined by a set of states, a set of input events, a transition function, and an initial state. In React, FSMs are used to model UI behaviour where the component can be in exactly one state at a time. The crucial point is that transitions are explicit.

**Beginner-Friendly Explanation:** A finite state machine is a way to say "this component can only be in one of these states, and it can only change in these specific ways."

### Purposes

- To make invalid states impossible.
- To make state transitions explicit and auditable.
- To simplify testing by enumerating all states and transitions.

### Syntax Rules and Structure

```jsx
const machine = createMachine({
  initial: 'idle',
  states: {
    idle: { on: { START: 'running' } },
    running: { on: { STOP: 'idle' } },
  },
});
```

### Annotated Code Examples

```jsx
// Traffic light FSM
const trafficLight = createMachine({
  initial: 'green',
  states: {
    green: { on: { TIMER: 'yellow' } },
    yellow: { on: { TIMER: 'red' } },
    red: { on: { TIMER: 'green' } },
  },
});
```

**Expected Output:** The light cycles green → yellow → red → green.

**Why this output:** The FSM defines exactly three states and the transitions between them. It is impossible to jump from green to red.

### Real-World Cases

- **Traffic lights:** Simple cyclic FSM.
- **Form validation:** Invalid → valid → submitting → success.
- **Media players:** Playing, paused, stopped.

### References

- XState — Finite State Machines — https://xstate.js.org
- Egghead — Handle HTTP request state with XState — https://egghead.io

---

## 8. XState

### Definitions

**Core Definition:** XState is a JavaScript library for creating, interpreting, and executing state machines and statecharts.

**Technical Definition:** XState is a comprehensive tool for handling non-trivial state management and orchestration tasks in web apps written in pure JavaScript or popular front-end frameworks such as React. It provides `createMachine` for defining machines, `interpret` for running them, and `@xstate/react` for React hooks and utilities. The Stately platform provides a visual editor and inspection tools.

**Beginner-Friendly Explanation:** XState is a library that helps you build state machines in JavaScript and React.

### Purposes

- To implement state machines and statecharts in React.
- To make complex state logic visual and testable.
- To eliminate entire categories of bugs from complex flows.

### Syntax Rules and Structure

```jsx
import { createMachine, interpret } from 'xstate';
import { useMachine } from '@xstate/react';

const [state, send] = useMachine(machine);
```

**Syntax Rules:**

1. Define the machine with `createMachine`.
2. Use `useMachine` in React components.
3. Send events with `send({ type: 'EVENT' })`.

### Annotated Code Examples

```jsx
import { useMachine } from '@xstate/react';
import { createMachine } from 'xstate';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});

function Toggle() {
  const [state, send] = useMachine(toggleMachine);
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.value === 'inactive' ? 'OFF' : 'ON'}
    </button>
  );
}
```

**Expected Output:** A button that toggles between ON and OFF.

**Why this output:** `useMachine` runs the machine and provides the current state and a `send` function. Dispatching `TOGGLE` transitions between states.

### Real-World Cases

- **Product tours:** Multi-step onboarding with branching.
- **Checkout flows:** Complex e-commerce checkout.
- **Authentication:** Login, MFA, token refresh.

### References

- XState — Official Website — https://xstate.js.org
- Stately — Visual Editor — https://stately.ai

---

## 9. State Reducers

### Definitions

**Core Definition:** A state reducer is a pure function that takes the current state and an action, and returns the next state.

**Technical Definition:** State reducers are the foundation of `useReducer` and Redux. They centralise all state transition logic in one place, making state changes predictable and testable. A reducer must be pure: given the same state and action, it must always return the same next state. Reducers can be composed — a root reducer delegates to slice reducers.

**Beginner-Friendly Explanation:** A reducer is a function that decides how state changes. You give it the current state and an action, and it tells you the new state.

### Purposes

- To centralise state transition logic.
- To make state changes predictable and testable.
- To enable reducer composition for large applications.

### Syntax Rules and Structure

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
}
```

### Annotated Code Examples

```jsx
function todosReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return [...state, { id: action.id, text: action.text, done: false }];
    case 'TOGGLE':
      return state.map((t) =>
        t.id === action.id ? { ...t, done: !t.done } : t
      );
    case 'DELETE':
      return state.filter((t) => t.id !== action.id);
    default:
      return state;
  }
}
```

**Expected Output:** The reducer correctly handles add, toggle, and delete actions.

**Why this output:** Each action type maps to a pure transformation. The spread operator and `map`/`filter` create new arrays without mutating the original.

### Real-World Cases

- **Todo lists:** Add, toggle, delete.
- **Shopping carts:** Add item, remove item, update quantity.
- **Forms:** Update field, reset, submit.

### References

- React — useReducer — https://react.dev/reference/react/useReducer
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer

---

## 10. State Normalisation

### Definitions

**Core Definition:** State normalisation is the practice of flattening nested or duplicated state into ID-indexed collections to eliminate redundancy.

**Technical Definition:** Data normalisation in React state management eliminates data redundancy and ensures each data entity has only one "authoritative source" (single source of truth). The core principles are: entity separation (different types of data entities stored in separate "tables") and ID indexing (each "table" is keyed by the entity's unique ID). This is analogous to database normalisation and makes state updates simpler and more predictable.

**Beginner-Friendly Explanation:** Normalisation means storing your data in flat lists indexed by ID, instead of nesting it inside other objects. It is like organising a database.

### Purposes

- To eliminate data duplication and synchronisation bugs.
- To make state updates simpler and more predictable.
- To avoid deeply nested state that is hard to update.
- To improve performance by reducing unnecessary re-renders.

### Syntax Rules and Structure

**❌ Nested (Anti-Pattern):**

```jsx
const state = {
  posts: [
    {
      id: 1,
      title: 'Post 1',
      author: { id: 1, name: 'Alice' },
      comments: [
        { id: 1, text: 'Nice!', author: { id: 2, name: 'Bob' } },
      ],
    },
  ],
};
```

**✅ Normalised (Correct):**

```jsx
const state = {
  posts: { byId: { 1: { id: 1, title: 'Post 1', authorId: 1, commentIds: [1] } }, allIds: [1] },
  users: { byId: { 1: { id: 1, name: 'Alice' }, 2: { id: 2, name: 'Bob' } }, allIds: [1, 2] },
  comments: { byId: { 1: { id: 1, text: 'Nice!', authorId: 2 } }, allIds: [1] },
};
```

### Annotated Code Examples

```jsx
function postsReducer(state, action) {
  switch (action.type) {
    case 'ADD_COMMENT': {
      const { postId, comment } = action.payload;
      return {
        ...state,
        comments: {
          ...state.comments,
          byId: { ...state.comments.byId, [comment.id]: comment },
          allIds: [...state.comments.allIds, comment.id],
        },
        posts: {
          ...state.posts,
          byId: {
            ...state.posts.byId,
            [postId]: {
              ...state.posts.byId[postId],
              commentIds: [...state.posts.byId[postId].commentIds, comment.id],
            },
          },
        },
      };
    }
    default:
      return state;
  }
}
```

**Expected Output:** A comment is added to both the comments table and the post's commentIds.

**Why this output:** Normalisation keeps comments and posts separate but linked by IDs. No data is duplicated.

### Real-World Cases

- **Social media feeds:** Posts, users, comments, likes.
- **E-commerce:** Products, categories, carts, orders.
- **Project management:** Projects, tasks, users, tags.

### References

- React — Choosing the State Structure — https://18.react.dev/learn/choosing-the-state-structure
- GitHub — 5 React Normalization Rules — https://github.com/khaledM-salem/5-React-Normalization-Rules

---

## 11. State Immutability

### Definitions

**Core Definition:** State immutability is the principle that state values must never be modified directly; new values must be created instead.

**Technical Definition:** Although objects in React state are technically mutable, they should be treated as immutable. When updating an object or array in state, a new object or array must be created (typically using the spread operator) and passed to the setter. Mutating state directly does not trigger a re-render because React compares references, not contents. Treating state as immutable enables React's rendering optimisations and prevents a whole class of bugs.

**Beginner-Friendly Explanation:** You cannot change state directly. Instead, you make a copy with the changes you want and give that copy to React.

### Purposes

- To ensure React detects changes and re-renders correctly.
- To enable safe comparison of previous and next state.
- To prevent bugs caused by shared references.
- To enable React's performance optimisations.

### Syntax Rules and Structure

```jsx
// ✅ Objects
setUser({ ...user, name: 'New' });

// ✅ Arrays
setItems([...items, newItem]);
setItems(items.filter((i) => i.id !== id));
```

### Annotated Code Examples

```jsx
function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={(e) => {
        // ✅ New object — React detects the change
        setPosition({ x: e.clientX, y: e.clientY });
      }}
    >
      <div style={{ transform: `translate(${position.x}px, ${position.y}px)` }} />
    </div>
  );
}
```

**Expected Output:** The dot follows the pointer.

**Why this output:** `setPosition` receives a new object each time, so React detects the change and re-renders.

### Real-World Cases

- **Todo lists:** Adding, removing, and updating items.
- **Forms:** Updating nested form fields.
- **Game state:** Moving pieces on a board.

### References

- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state

---

## 12. Immutable.js

### Definitions

**Core Definition:** Immutable.js is a JavaScript library that provides persistent immutable data structures, such as `Map`, `List`, and `Set`.

**Technical Definition:** Immutable.js provides data structures that cannot be changed once created. Operations like `set`, `push`, and `update` return new immutable structures rather than mutating the original. Immutable.js uses structural sharing to make these operations efficient. However, converting between Immutable.js objects and plain JavaScript objects adds overhead, and the library has fallen out of favour compared to Immer.

**Beginner-Friendly Explanation:** Immutable.js gives you special data structures that you cannot change. Instead, you create new versions with your changes.

### Purposes

- To enforce immutability at the data structure level.
- To enable efficient structural sharing.
- To prevent accidental mutations.

### Syntax Rules and Structure

```jsx
import { Map, List } from 'immutable';

const map = Map({ name: 'Alice' });
const newMap = map.set('name', 'Bob'); // New map, original unchanged
```

### Annotated Code Examples

```jsx
import { List } from 'immutable';

const list = List([1, 2, 3]);
const newList = list.push(4); // New list: [1, 2, 3, 4]
// list is still [1, 2, 3]
```

**Expected Output:** `list` remains unchanged; `newList` contains the new value.

**Why this output:** Immutable.js data structures return new instances from update operations.

### Real-World Cases

- **Complex nested data:** Deeply nested state that is hard to update immutably.
- **Performance-critical apps:** Structural sharing reduces copying.

### References

- Immutable.js — Official Documentation — https://immutable-js.com

---

## 13. Immer

### Definitions

**Core Definition:** Immer is a library that lets you write mutable-looking code to produce immutable state updates.

**Technical Definition:** Immer provides a `produce` function that takes a base state and a "recipe" function. Inside the recipe, you can mutate a draft object as if it were mutable. Immer records the changes and produces a new immutable state with structural sharing. Immer also provides `useImmer` and `useImmerReducer` hooks for React. Immer copies exactly the levels that changed, and it detects "no-op" state changes, returning the original state if nothing actually changed.

**Beginner-Friendly Explanation:** Immer lets you write code that looks like it mutates state, but it actually creates a new immutable copy behind the scenes.

### Purposes

- To simplify immutable updates for deeply nested state.
- To avoid verbose spreading of nested objects.
- To reduce boilerplate in reducers.

### Syntax Rules and Structure

```jsx
import { produce } from 'immer';

const nextState = produce(currentState, (draft) => {
  draft.user.address.city = 'Paris';
});
```

**Syntax Rules:**

1. `produce` takes the current state and a recipe function.
2. Mutate the `draft` inside the recipe.
3. Immer returns a new immutable state.

### Annotated Code Examples

```jsx
import { useImmer } from 'use-immer';

function UserProfile() {
  const [user, updateUser] = useImmer({
    name: 'Alice',
    address: { city: 'London', country: 'UK' },
  });

  function moveToParis() {
    updateUser((draft) => {
      draft.address.city = 'Paris';  // Looks mutable, but is immutable
    });
  }

  return (
    <div>
      <p>{user.name} lives in {user.address.city}</p>
      <button onClick={moveToParis}>Move</button>
    </div>
  );
}
```

**Expected Output:** The city updates to Paris.

**Why this output:** `useImmer` wraps `useState` with Immer's `produce`. The draft mutation is converted into an immutable update.

### Real-World Cases

- **Complex forms:** Updating deeply nested form state.
- **Redux reducers:** Simplifying reducer logic.
- **Nested data structures:** Editing deeply nested objects.

### References

- Immer — React & Immer — https://immerjs.github.io/immer/example-setstate
- Redux — Immutable Update Patterns — https://redux.js.org/usage/structuring-reducers/immutable-update-patterns

---

## 14. Structural Sharing

### Definitions

**Core Definition:** Structural sharing is a technique where new data structures share unchanged parts with the original, avoiding unnecessary copying.

**Technical Definition:** Structural sharing is an optimisation used by immutable data libraries (Immer, Immutable.js, TanStack Query) where only the parts of a data structure that change are copied, while unchanged parts retain their original references. This makes immutable updates efficient and enables React to detect changes by reference comparison. TanStack Query uses structural sharing to ensure that as many references as possible are kept intact between re-renders.

**Beginner-Friendly Explanation:** Structural sharing means when you change one part of a data structure, the unchanged parts are reused instead of copied.

### Purposes

- To make immutable updates efficient.
- To enable React's reference-based change detection.
- To reduce memory usage.

### Syntax Rules and Structure

**Automatic with Immer:**

```jsx
const nextState = produce(state, (draft) => {
  draft.a = 2; // Only `a` is copied; `b` and `c` are shared
});
```

**TanStack Query:**

```jsx
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  structuralSharing: true, // Default
});
```

### Annotated Code Examples

```jsx
// With Immer, structural sharing happens automatically
const state = { a: 1, b: { c: 2, d: 3 }, e: 4 };
const next = produce(state, (draft) => {
  draft.a = 10;
});

console.log(next.b === state.b); // true — b is shared!
console.log(next.a); // 10
console.log(state.a); // 1 — original unchanged
```

**Expected Output:** `next.b` is the same reference as `state.b`.

**Why this output:** Immer copies only the top-level object and the `a` property. The `b` and `e` properties are shared by reference.

### Real-World Cases

- **Large lists:** Updating one item shares the rest.
- **TanStack Query:** Preserving object identity between fetches.
- **Redux:** Efficient state updates.

### References

- Immer — Performance — https://immerjs.github.io/immer/performance
- TanStack Query — Render Optimizations — https://tanstack.com/query

---

## 15. State Update Patterns

### Definitions

**Core Definition:** State update patterns are the established techniques for correctly and efficiently updating React state.

**Technical Definition:** State update patterns encompass the rules and idioms for updating primitives, objects, arrays, and nested structures immutably. Key patterns include: using functional updates when the new state depends on the previous state, spreading objects and arrays to create new references, using `map`/`filter` for array transformations, and using `useReducer` for complex multi-field transitions. State updates in React must be immutable — never modify the existing state directly.

**Beginner-Friendly Explanation:** These are the correct ways to change state without causing bugs.

### Purposes

- To update state correctly without triggering stale closure bugs.
- To create new references so React detects changes.
- To handle complex state transitions with reducers.

### Syntax Rules and Structure

| Update Type | Pattern |
|-------------|---------|
| Primitive | `setCount(count + 1)` |
| Functional | `setCount(c => c + 1)` |
| Object | `setUser({ ...user, name: 'New' })` |
| Array add | `setItems([...items, newItem])` |
| Array remove | `setItems(items.filter(i => i.id !== id))` |
| Array update | `setItems(items.map(i => i.id === id ? {...i, done: true} : i))` |
| Nested object | `setUser({ ...user, address: { ...user.address, city: 'New' } })` |

### Annotated Code Examples

```jsx
function Form() {
  const [user, setUser] = useState({ name: '', address: { city: '' } });

  // Update nested field
  function updateCity(city) {
    setUser((prev) => ({
      ...prev,
      address: { ...prev.address, city },
    }));
  }

  // Add to array
  const [tags, setTags] = useState([]);
  function addTag(tag) {
    setTags((prev) => [...prev, tag]);
  }
}
```

**Expected Output:** Nested fields and arrays update correctly.

**Why this output:** The spread operator creates new objects at each level that changes, so React detects the new reference.

### Real-World Cases

- **All React applications:** These patterns apply universally.
- **Code reviews:** Checking for mutation and stale closures.

### References

- Strapi — Master React useState — https://strapi.io
- CoreUI — How to update state in React — https://coreui.io

---

## 16. State Management Best Practices

### Definitions

**Core Definition:** State management best practices are guidelines for choosing where state should live, how it should be shaped, and how it should be updated.

**Technical Definition:** Best practices encompass the principles of state structure (group related state, avoid contradictions, avoid redundant state, avoid duplication, avoid deeply nested state), state placement (colocate by default, lift only when necessary), and state updates (immutable updates, functional updates, reducers for complex logic). Start with React's built-in tools — `useState`, `useReducer`, and Context — and add external libraries only when the application genuinely outgrows them.

**Beginner-Friendly Explanation:** These are the rules of thumb that experienced React developers follow to keep their state clean and their apps fast.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Colocate by default** | Keep state as close as possible to where it is used. |
| **Lift only when necessary** | Lift state only when multiple distant components need it. |
| **Derive, don't store** | If you can compute it, compute it. Do not store it. |
| **Avoid redundant state** | Do not store values that can be calculated. |
| **Avoid contradictory state** | Use a single status field instead of multiple booleans. |
| **Group related state** | If values always change together, merge them. |
| **Avoid deep nesting** | Flat state is easier to update. |
| **Normalise complex data** | Store entities by ID in flat tables. |
| **Use functional updates** | When new state depends on previous state. |
| **Keep reducers pure** | No side effects, no mutations. |
| **Start simple** | Use React's built-in tools; add libraries only when needed. |

### Annotated Code Examples

**✅ Good State Structure:**

```jsx
// Single status field prevents contradictions
const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'success' | 'error'

// Derived during render
const isLoading = status === 'loading';
const isError = status === 'error';
```

**❌ Bad State Structure:**

```jsx
// Contradictory state — impossible combinations possible
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
```

**Expected Output:** The good structure makes invalid states impossible.

**Why this output:** A single `status` field can only be one value at a time. Multiple booleans allow `isLoading: true` and `isSuccess: true` simultaneously.

### Real-World Cases

- **All React applications:** Best practices apply universally.
- **Team projects:** Consistency reduces integration bugs.
- **Scalable applications:** Best practices keep code maintainable as features grow.

### References

- React — Choosing the State Structure — https://18.react.dev/learn/choosing-the-state-structure
- RT Camp — Choosing the Right React State Management Strategy — https://rtcamp.com
- tessl-labs — Frontend State Management — https://tessl.io
- Steve Kinney — Lifting State Intelligently — https://stevekinney.com

---

## References

- React — Choosing the State Structure — https://18.react.dev/learn/choosing-the-state-structure
- React — Sharing State Between Components — https://react.dev/learn/sharing-state-between-components
- React — Updating Objects in State — https://react.dev/learn/updating-objects-in-state
- React — Updating Arrays in State — https://react.dev/learn/updating-arrays-in-state
- React — Extracting State Logic into a Reducer — https://react.dev/learn/extracting-state-logic-into-a-reducer
- React — useReducer — https://react.dev/reference/react/useReducer
- Steve Kinney — Colocation of State — https://stevekinney.com
- Steve Kinney — Lifting State Intelligently — https://stevekinney.com
- XState — Official Documentation — https://xstate.js.org
- Stately — Visual Editor — https://stately.ai
- Immer — React & Immer — https://immerjs.github.io/immer/example-setstate
- Immer — Performance — https://immerjs.github.io/immer/performance
- TanStack Query — Render Optimizations — https://tanstack.com/query
- Redux — Immutable Update Patterns — https://redux.js.org/usage/structuring-reducers/immutable-update-patterns
- RT Camp — Choosing the Right React State Management Strategy — https://rtcamp.com
- tessl-labs — Frontend State Management — https://tessl.io
- GitHub — 5 React Normalization Rules — https://github.com/khaledM-salem/5-React-Normalization-Rules
- DHTMLX — Using XState in React — https://dhtmlx.com
- CoreUI — How to lift state up in React — https://coreui.io
- CoreUI — How to update state in React — https://coreui.io
- Strapi — Master React useState — https://strapi.io