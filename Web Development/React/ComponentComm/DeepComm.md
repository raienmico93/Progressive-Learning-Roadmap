# Deep Component Communication: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Deep component communication refers to the architectural patterns and techniques for sharing data, state, and behaviour between React components that are separated by many levels in the component tree—where direct parent-child prop passing becomes impractical or impossible.

**Technical Definition:** Deep component communication addresses the limitations of React's unidirectional prop-passing model when the "nearest common ancestor" of components that need to share data is far removed from those components. It encompasses four primary strategies: (1) the Context API, which allows a parent to "teleport" data to any descendant without threading props through intermediate components; (2) external state-management libraries (Redux, Zustand, XState), which hold state outside the React tree and expose it to any component via subscriptions; (3) event-based pub/sub patterns, which enable non-hierarchical, decoupled communication through custom event buses; and (4) architectural boundaries, which determine where state should live relative to domain, feature, or page-level scopes to prevent over-coupling and maintainability degradation.

**Beginner-Friendly Explanation:** In a small React app, passing data from parent to child is easy—you just use props. But in a large app, you often have a component at the top of the tree that needs to share data with a component buried ten levels deep. Passing props through every single level in between is tedious and messy (this is called "prop drilling"). Deep component communication is about better ways to handle this: using a shared "channel" (Context), a global "bulletin board" (state management library), or a "messaging system" (pub/sub events) so that components can talk to each other without going through every intermediate layer.

### Key Characteristics

- **Prop Drilling Avoidance:** Context and external stores eliminate the need to pass props through intermediate components that do not use them.
- **Implicit vs. Explicit Data Flow:** Context and stores introduce implicit data flow (consumers "pull" data from a provider), whereas prop passing is explicit. This improves ergonomics but can make data flow harder to trace.
- **Re-render Propagation:** Context updates cause all consumers to re-render; external stores with selectors allow fine-grained subscriptions, re-rendering only components that read the changed slice.
- **Decoupling:** Event-based patterns allow publishers and subscribers to communicate without holding direct references to each other, enabling truly decoupled architectures.
- **Architectural Scalability:** Feature-based boundaries (domain, feature, page) prevent state from becoming a globally entangled "god object" as the application grows.

### Prerequisites

- Solid understanding of React components, props, and the `useState` Hook.
- Familiarity with the component tree hierarchy and prop drilling.
- Working knowledge of `useContext` and the Context API.
- Basic understanding of external state management concepts (stores, reducers, selectors).
- Awareness of architectural patterns (feature-based, domain-driven design).

### Related Programming Areas

- **State Management:** Coordinating state across disparate branches of the component tree.
- **Event-Driven Architecture:** Pub/sub, event emitters, and message buses.
- **Dependency Injection:** Providing dependencies (services, configs) to deeply nested components.
- **Feature-Sliced Design:** Organising code by domain/feature with clear public APIs and boundaries.
- **Performance Optimisation:** Avoiding unnecessary re-renders caused by context or store updates.

### Core Concepts / Features

1. Context API (Providing Data to Deeply Nested Trees)
2. State-Management Libraries (Redux, Zustand, XState)
3. Event-Based Patterns (Pub/Sub and Custom Events)
4. Architectural Boundaries (Domain, Feature, and Page-Level State)

---

## Core Concept 1: Context API (Providing Data to Deeply Nested Trees)

### Definitions

**Core Definition:** The React Context API is a built-in feature that allows a parent component to make data available to any component in the tree below it—no matter how deep—without passing it explicitly through props.

**Technical Definition:** Context provides a way to pass data through the component tree without having to pass props down manually at every level. It is designed to share data that can be considered "global" for a tree of React components, such as the current authenticated user, theme, or preferred language. Context is created with `React.createContext()`, provided to a subtree via a `<Context.Provider value={...}>` component, and consumed by any descendant via the `useContext(Context)` Hook. When the provider's `value` changes, all consumers of that context re-render, regardless of whether they are direct children or deeply nested descendants. Context is an alternative to prop drilling: instead of passing a prop through every intermediate component, the provider "teleports" the value to the components that need it. However, Context is primarily a *dependency injection* mechanism, not a complete state management solution; for frequently changing state, selector-based external stores are often more performant.

**Beginner-Friendly Explanation:** Imagine a family group chat. Instead of each sibling asking the parent to relay a message, the parent posts the message in the group chat, and all siblings can read it directly. The parent doesn't have to pass the message through each sibling individually—any sibling in the group can see it. That's Context: a shared channel that any component in the tree can tap into, no matter how deep it is.

### Purposes

- To avoid prop drilling—passing props through many intermediate components that do not use them.
- To share data that is "global" to a subtree, such as theme, locale, or authenticated user.
- To provide a single source of truth for shared data across deeply nested components.
- To reduce the boilerplate of manually threading props through the component tree.
- To enable feature-scoped sharing without introducing an external state management library.

### Syntax Rules and Structure

**Step 1: Create the Context**
```jsx
import { createContext } from 'react';

const ThemeContext = createContext('light'); // Default value
```

**Component Breakdown:**
- `createContext(defaultValue)`: Creates a context object with an optional default value.
- The default value is used only when a component reads the context without a matching provider above it.

**Step 2: Provide the Context**
```jsx
function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

**Component Breakdown:**
- `<ThemeContext.Provider value={...}>`: Wraps the subtree that should have access to the context.
- `value={{ theme, setTheme }}`: The data and setter are passed as the context value.
- All descendants of `<Toolbar />` can read this value.

**Step 3: Consume the Context**
```jsx
import { useContext } from 'react';

function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button
      style={{ background: theme === 'dark' ? '#333' : '#fff' }}
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      Toggle Theme
    </button>
  );
}
```

**Component Breakdown:**
- `useContext(ThemeContext)`: Reads the nearest provider's value.
- `{ theme, setTheme }`: Destructures the context value.
- The button reads `theme` and calls `setTheme` to request changes.

**Syntax Rules:**
- Create the context with `createContext()`. The default value is optional but recommended for testing and fallback.
- Wrap the subtree that needs access in `<Context.Provider value={...}>`.
- Consume the context with `useContext(Context)` in any descendant component.
- When the provider's `value` changes, all consumers re-render. To avoid unnecessary re-renders, memoise the value object with `useMemo` or split the context into separate state and dispatch contexts.
- Context is best for data that is truly "global" to a subtree (theme, locale, auth). Avoid using it for state that changes frequently and is only needed by a few components.
- For state + dispatch patterns, use `useReducer` with Context: put the reducer's `state` and `dispatch` into two separate contexts to prevent dispatch-only consumers from re-rendering when state changes.

**Constraints and Limitations:**
- All consumers of a context re-render when the provider's `value` changes, even if they only read a small part of the value.
- Context makes component reuse more difficult because consumers are coupled to the context's shape.
- Context is not a complete state management solution; it does not provide selectors, middleware, or time-travel debugging.
- For frequently changing state (e.g., form inputs, counters), Context can cause performance issues; external stores with selectors are often better.
- Context should be used sparingly; for simple prop drilling, component composition (passing JSX as children) is often a simpler solution.

### Annotated Code Example: Theme Context for Deeply Nested Components

```jsx
import { createContext, useContext, useState } from 'react';

// Step 1: Create the context
const ThemeContext = createContext('light');

// Deeply nested component that consumes the context
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      style={{
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333',
      }}
    >
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

// Intermediate component that does NOT use the context
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

// Parent component: provides the context
export default function ThemeApp() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <h1>Theme Switcher</h1>
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

**Expected Output:** A heading "Theme Switcher" and a button labelled "Switch to dark mode". Clicking the button changes the theme to dark, updating the button's background to dark and text to white, and changing the label to "Switch to light mode".

**Why This Output Occurs:** The `theme` state lives in `ThemeApp`, which provides it via `ThemeContext.Provider`. `ThemedButton` is a deeply nested descendant (rendered inside `Toolbar`) and consumes the context with `useContext(ThemeContext)`. `Toolbar` is an intermediate component that does not use the context at all—it simply renders its children. When the button calls `setTheme('dark')`, the provider's value changes, and `ThemedButton` re-renders with the new theme. The context "teleports" the theme data directly to the component that needs it, bypassing `Toolbar` entirely.

### Real-World Cases

- **Theme switching:** A theme context provides the current theme and a toggle function to all components in the app.
- **User authentication:** An auth context provides the current user and login/logout functions to the entire app.
- **Locale/internationalisation:** A locale context provides the current language and translation function to all components.
- **Feature flags:** A feature-flag context provides enabled/disabled flags to components throughout the app.
- **Shopping cart:** A cart context provides cart items and add/remove functions to product pages and the cart sidebar.

### References

- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – useContext: https://react.dev/reference/react/useContext
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- React Legacy Documentation – Context: https://legacy.reactjs.org/docs/context.html

---

## Core Concept 2: State-Management Libraries (Redux, Zustand, XState)

### Definitions

**Core Definition:** State-management libraries are external tools that hold application state outside of the React component tree, allowing any component—regardless of its position in the hierarchy—to subscribe to and update shared state directly.

**Technical Definition:** State-management libraries decouple state from the component hierarchy by maintaining a global store that lives outside React. Components subscribe to slices of this store via hooks (e.g., `useSelector` in Redux, `useStore` in Zustand, `useActor` in XState) and re-render only when their selected slice changes. This eliminates the prop-drilling and re-render issues associated with lifted state and Context. The three dominant paradigms are: (1) **Flux/Centralised (Redux Toolkit)** — a single global store with dispatched actions and pure reducers, offering predictable state transitions and time-travel debugging; (2) **Hook-based/Minimal (Zustand)** — a simple external store consumed via hooks, with no providers, actions, or reducers required; and (3) **State Machine/Actor (XState)** — state machines and statecharts that model complex, deterministic workflows with explicit states and transitions. Redux requires your app to be wrapped in context providers, while Zustand does not. XState uses event-driven programming and the actor model to handle complex logic in predictable, visual ways.

**Beginner-Friendly Explanation:** Imagine a public bulletin board in a town square. Anyone can pin a message to it, and anyone can read from it. Components don't need to go through their parent to communicate—they just read from and write to the bulletin board. That's an external state store: a shared place for data that lives outside the family (component tree). Redux is like a formal government office with strict procedures; Zustand is like a casual community whiteboard; XState is like a flowchart that only allows certain actions at certain times.

### Purposes

- To synchronise components without requiring a common parent to mediate.
- To avoid prop drilling and the re-render performance issues of Context.
- To provide a single source of truth for app-wide state.
- To enable fine-grained subscriptions so components only re-render when their specific data changes.
- To support complex state management patterns such as middleware, devtools, persistence, and time-travel debugging.
- To model complex workflows with explicit states and transitions (XState).

### Syntax Rules and Structure

**Pattern 1: Zustand (Minimal Hook-Based Store)**

```javascript
import { create } from 'zustand';

// Create the store (lives outside React)
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// Deeply nested component: reads and writes
function CounterButton() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  return <button onClick={increment}>Count: {count}</button>;
}

// Another component: reads the same state
function CountDisplay() {
  const count = useStore((state) => state.count);
  return <p>Count is: {count}</p>;
}
```

**Component Breakdown:**
- `create((set) => ({ ... }))`: Creates a Zustand store with state and actions.
- `useStore((state) => state.count)`: Selects the `count` slice; only re-renders when `count` changes.
- No providers are needed; the store is a hook that can be used anywhere.

**Pattern 2: Redux Toolkit (Centralised Store with Slices)**

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
  },
});

const { increment, decrement } = counterSlice.actions;
const store = configureStore({ reducer: { counter: counterSlice.reducer } });

function CounterButton() {
  const dispatch = useDispatch();
  return <button onClick={() => dispatch(increment())}>Increment</button>;
}

function CountDisplay() {
  const count = useSelector((state) => state.counter.value);
  return <p>Count: {count}</p>;
}

function App() {
  return (
    <Provider store={store}>
      <CounterButton />
      <CountDisplay />
    </Provider>
  );
}
```

**Component Breakdown:**
- `configureStore`: Creates the Redux store.
- `<Provider store={store}>`: Makes the store available to all components.
- `useDispatch()`: Returns the dispatch function for actions.
- `useSelector((state) => state.counter.value)`: Selects the `value` slice.

**Pattern 3: XState (State Machine / Actor Model)**

```javascript
import { createMachine } from 'xstate';
import { useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
});

function ToggleButton() {
  const [state, send] = useMachine(toggleMachine);
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.value === 'inactive' ? 'Off' : 'On'}
    </button>
  );
}
```

**Component Breakdown:**
- `createMachine({ ... })`: Defines the state machine with states and transitions.
- `useMachine(toggleMachine)`: Creates and starts the actor, returning the current state and `send` function.
- `send({ type: 'TOGGLE' })`: Sends an event to the machine, triggering a transition.

**Syntax Rules:**
- **Zustand:** Create a store with `create()`. Use selector functions to subscribe to specific slices. No provider is needed.
- **Redux Toolkit:** Create slices with `createSlice()`. Configure the store with `configureStore()`. Wrap the app in `<Provider store={store}>`. Use `useSelector` to read and `useDispatch` to write. Do not mutate state directly; use Immer's draft state in reducers.
- **XState:** Define machines with `createMachine()`. Use `useMachine` or `useActor` to run them in components. Send events with `send()`. Machines model explicit states and transitions, preventing invalid states.
- Choose the library based on complexity: Zustand for simple to medium apps, Redux Toolkit for large apps with complex state, XState for complex workflows with clear states and transitions.
- External stores are best for *client* state (UI flags, user preferences, cart contents). For *server* state (API data), use TanStack Query or SWR.

**Constraints and Limitations:**
- External stores add dependencies and bundle size. Redux is the heaviest, while Zustand and XState are lightweight.
- Redux requires significant boilerplate (slices, actions, reducers) compared to Zustand.
- Zustand does not provide the strict predictability and time-travel debugging of Redux out of the box.
- XState has a steeper learning curve and is overkill for simple state.
- External stores are overkill for simple sibling communication that can be handled with lifted state or Context.

### Annotated Code Example: Zustand Store for Deep Component Communication

```javascript
import { create } from 'zustand';

// Create the store — lives outside React
const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text, done: false }],
  })),
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ),
  })),
}));

// Deeply nested component: Add todo form
function AddTodo() {
  const addTodo = useTodoStore((state) => state.addTodo);
  const [text, setText] = useState('');

  function handleSubmit(e) {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text.trim());
      setText('');
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

// Another deeply nested component: Todo list
function TodoList() {
  const todos = useTodoStore((state) => state.todos);
  const toggleTodo = useTodoStore((state) => state.toggleTodo);

  return (
    <ul>
      {todos.map(todo => (
        <li
          key={todo.id}
          onClick={() => toggleTodo(todo.id)}
          style={{ textDecoration: todo.done ? 'line-through' : 'none' }}
        >
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Intermediate components that don't use the store
function Sidebar() {
  return <div><AddTodo /></div>;
}

function MainContent() {
  return <div><TodoList /></div>;
}

// Parent: renders both branches
export default function TodoApp() {
  return (
    <div>
      <h1>Todo List</h1>
      <Sidebar />
      <MainContent />
    </div>
  );
}
```

**Expected Output:** A form with an input and "Add" button, and an empty todo list. Typing a task and clicking "Add" adds it to the list. Clicking a task toggles its done state (strikethrough).

**Why This Output Occurs:** The `useTodoStore` Zustand store holds all todo state outside React. `AddTodo` and `TodoList` are in separate branches of the component tree, separated by intermediate components (`Sidebar`, `MainContent`) that know nothing about the store. Both components subscribe to the store directly—`AddTodo` selects the `addTodo` action, and `TodoList` selects the `todos` array and the `toggleTodo` action. When `addTodo` updates the store, `TodoList` re-renders with the new todo. The components communicate directly through the store—no parent mediation, no prop drilling, and no Context providers are needed.

### Real-World Cases

- **E-commerce carts:** The cart state is stored in a global store; product pages add items, and the cart sidebar reads them.
- **User authentication:** Auth state (user, token, login status) is stored globally; navbar and protected routes read it.
- **Notification systems:** Notification state is stored globally; a notification bell and a notification panel both read it.
- **Dashboard filters:** Filter state is stored globally; multiple chart widgets read the same filters and update together.
- **Multi-step workflows:** XState machines model the workflow with explicit states (idle, loading, success, error) and transitions, preventing invalid states.

### References

- Redux – Style Guide: https://redux.js.org/style-guide/
- Redux Toolkit – Quick Start: https://redux.js.org/tutorials/quick-start
- Zustand – Comparison: https://zustand.docs.pmnd.rs/learn/getting-started/comparison
- Zustand – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- XState – Core Concepts: https://stately.ai/docs/xstate
- XState – Use a machine in React: https://stately.ai/docs/xstate-react
- DEV Community – Redux Toolkit vs Zustand vs Jotai: https://dev.to/zeeshanali0704/frontend-system-design-redux-toolkit-vs-zustand-vs-jotai-1npn

---

## Core Concept 3: Event-Based Patterns (Pub/Sub and Custom Events)

### Definitions

**Core Definition:** Event-based patterns are communication mechanisms where components publish named events to a central event bus or target, and other components subscribe to those events—enabling non-hierarchical, decoupled communication between components that have no direct parent-child relationship.

**Technical Definition:** Event-based patterns implement the publish-subscribe (pub/sub) model, where publishers emit named events with optional payloads, and subscribers register callbacks for specific event names. In React, this is typically implemented using the browser's `EventTarget` and `CustomEvent` APIs, or a lightweight event emitter library. `EventTarget` is the foundation of the browser's event system—it provides `addEventListener`, `removeEventListener`, and `dispatchEvent` methods. `CustomEvent` allows defining custom event types beyond built-in types like "click" or "keydown", with any kind of data passed through the `detail` property. The browser's event system shares similarities with both the Observer and Pub-Sub patterns: when a listener is attached directly to a DOM element, it behaves like the Observer pattern; when `EventTarget` is used as a standalone event system, it behaves more like the Pub-Sub pattern, where publishers and subscribers are loosely coupled and communicate through named events. The key advantage of event-based patterns is decoupling: publishers and subscribers do not hold direct references to each other, making them ideal for cross-cutting concerns like notifications, analytics, or logging.

**Beginner-Friendly Explanation:** Imagine a town crier. Anyone in the town can shout a message ("The market is open!"), and anyone who is listening for that message will hear it and react. The person shouting doesn't need to know who is listening, and the listeners don't need to know who shouted. That's a pub/sub system: a way for components to broadcast events without knowing or caring who receives them. In React, you can use the browser's built-in `EventTarget` and `CustomEvent` to create this kind of system.

### Purposes

- To enable communication between components that have no direct parent-child relationship.
- To decouple publishers from subscribers—the publisher does not need to know which components are listening.
- To implement cross-cutting concerns such as notifications, analytics tracking, or logging.
- To avoid the re-render overhead of Context or the boilerplate of state-management libraries for simple event notifications.
- To allow dynamic subscription and unsubscription at runtime.

### Syntax Rules and Structure

**Pattern 1: Custom Event System with EventTarget and CustomEvent**

```javascript
// Create a standalone event target (the event bus)
const eventBus = new EventTarget();

// Publisher: dispatch a custom event with data
function publishEvent(eventName, data) {
  const event = new CustomEvent(eventName, { detail: data });
  eventBus.dispatchEvent(event);
}

// Subscriber: listen for the event
function subscribeToEvent(eventName, callback) {
  eventBus.addEventListener(eventName, callback);
  // Return an unsubscribe function
  return () => eventBus.removeEventListener(eventName, callback);
}
```

**Component Breakdown:**
- `new EventTarget()`: Creates a standalone event bus (not attached to any DOM element).
- `new CustomEvent(eventName, { detail: data })`: Creates a custom event with a payload.
- `eventBus.dispatchEvent(event)`: Publishes the event.
- `eventBus.addEventListener(eventName, callback)`: Subscribes to the event.
- `return () => eventBus.removeEventListener(eventName, callback)`: Cleanup function to unsubscribe.

**Pattern 2: React Hook for Subscribing to Events**

```jsx
import { useEffect } from 'react';

function useEvent(eventName, handler) {
  useEffect(() => {
    const unsubscribe = subscribeToEvent(eventName, handler);
    return unsubscribe; // Cleanup on unmount
  }, [eventName, handler]);
}
```

**Component Breakdown:**
- `useEffect`: Subscribes when the component mounts and unsubscribes on unmount.
- `subscribeToEvent`: Returns an unsubscribe function used as the cleanup.
- `[eventName, handler]`: Re-subscribes if the event name or handler changes.

**Syntax Rules:**
- Create a single, shared `EventTarget` instance (the event bus) in a module that both publishers and subscribers can import.
- Use `CustomEvent` to define named events with payloads via the `detail` property.
- Always return a cleanup function from `useEffect` that removes the event listener.
- Name events descriptively (e.g., `'todo:added'`, `'user:login'`) to avoid collisions.
- Use a custom hook (`useEvent`) to encapsulate subscription logic and cleanup.
- Prefer event-based patterns for notifications and cross-cutting concerns, not for primary application state.

**Constraints and Limitations:**
- Event-based patterns introduce implicit coupling: it is difficult to trace which components respond to an event without searching the codebase.
- Events are fire-and-forget; there is no built-in way to know whether a subscriber received the event or to retrieve a response.
- Event ordering is not guaranteed across multiple subscribers.
- Type safety is limited; event payloads are typed as `any` unless you create a typed event map.
- Overusing pub/sub can lead to "event spaghetti" where the application's behaviour becomes difficult to reason about.

### Annotated Code Example: Notification System with Custom Events

```jsx
// eventBus.js — shared event bus module
const eventBus = new EventTarget();

export function emitNotification(message, type = 'info') {
  const event = new CustomEvent('notification', {
    detail: { message, type, timestamp: Date.now() },
  });
  eventBus.dispatchEvent(event);
}

export function onNotification(handler) {
  eventBus.addEventListener('notification', handler);
  return () => eventBus.removeEventListener('notification', handler);
}
```

```jsx
// NotificationDisplay.jsx — subscribes to notifications
import { useState, useEffect } from 'react';
import { onNotification } from './eventBus';

function NotificationDisplay() {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const unsubscribe = onNotification((event) => {
      const { message, type, timestamp } = event.detail;
      setNotifications(prev => [...prev, { message, type, id: timestamp }]);
    });

    return unsubscribe; // ✅ Cleanup on unmount
  }, []);

  return (
    <div className="notifications">
      {notifications.map(n => (
        <div key={n.id} className={`notification ${n.type}`}>
          {n.message}
        </div>
      ))}
    </div>
  );
}
```

```jsx
// AnyComponent.jsx — publishes a notification from anywhere
import { emitNotification } from './eventBus';

function SaveButton() {
  function handleSave() {
    // ... save logic ...
    emitNotification('Changes saved successfully!', 'success');
  }

  return <button onClick={handleSave}>Save</button>;
}

// App: components are in completely different branches
export default function App() {
  return (
    <div>
      <header>
        <NotificationDisplay /> {/* Subscriber */}
      </header>
      <main>
        <SaveButton /> {/* Publisher */}
      </main>
    </div>
  );
}
```

**Expected Output:** A "Save" button and an empty notification area. Clicking "Save" triggers a success notification that appears in the notification area (e.g., "Changes saved successfully!" with a green background).

**Why This Output Occurs:** The `eventBus` module creates a standalone `EventTarget` shared by all components. `SaveButton` calls `emitNotification()`, which dispatches a `'notification'` CustomEvent with the message and type. `NotificationDisplay` subscribes to the `'notification'` event via `onNotification()` inside a `useEffect`. When the event fires, the subscriber's callback receives the event, extracts the `detail`, and adds it to the notifications state. The publisher and subscriber are in completely different branches of the component tree and have no direct relationship—they communicate purely through the shared event bus.

### Real-World Cases

- **Notification systems:** A toast notification system where any component can emit a notification and a dedicated notification container displays them.
- **Analytics tracking:** Components emit `'page:view'` or `'button:click'` events that an analytics subscriber logs.
- **Undo/redo systems:** Commands emit events that an undo manager subscribes to and records.
- **Real-time collaboration:** WebSocket messages are dispatched as custom events, and multiple UI components subscribe to update their state.
- **Plugin architectures:** Third-party plugins subscribe to application events without modifying the core code.

### References

- MDN Web Docs – EventTarget: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
- MDN Web Docs – CustomEvent: https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent
- tsecurity.de – EventTarget - CustomEvent | Components Communication in React: https://tsecurity.de/de/2761056/IT+Programmierung/EventTarget+-+CustomEvent+|+components+communication+in+React+-+part+Three/
- CSS-Tricks – Understanding Event Emitters: https://css-tricks.com/understanding-event-emitters/
- David Walsh – Pub/Sub JavaScript Object: https://davidwalsh.name/pubsub-javascript

---

## Core Concept 4: Architectural Boundaries (Domain, Feature, and Page-Level State)

### Definitions

**Core Definition:** Architectural boundaries are the organisational principles that determine where state should live relative to domain, feature, or page-level scopes, preventing state from becoming globally entangled and ensuring that components only depend on the state they genuinely need.

**Technical Definition:** Architectural boundaries in React applications refer to the deliberate partitioning of state and logic into scopes that align with the application's domain model, feature modules, or page-level concerns. Rather than placing all state in a single global store (which leads to over-coupling and maintainability issues), state is colocated with the feature or domain that owns it. Feature-Sliced Design (FSD) is one such methodology: it organises code into layers (app, pages, widgets, features, entities, shared) and enforces clear boundaries between slices, with each slice exposing a public API and keeping its internal state management private. The principle is that a feature with both local store state and server data needs an explicit bridge in a custom hook or named action, and view-scoped local feature stores are preferred over global stores for page-instance state. State-management strategy typically distinguishes between server state (TanStack Query for API data), client UI state (Zustand for UI flags), form state (React Hook Form), and URL state (React Router).

**Beginner-Friendly Explanation:** Imagine a large office building. If every employee stored all their files in one central filing cabinet, the cabinet would become a chaotic mess, and it would be impossible to know who owns what. Instead, each department has its own filing cabinet, and within each department, each team has its own drawer. Architectural boundaries are like these departmental filing cabinets: state is organised by domain or feature, not dumped into one global pile. This keeps things organised, prevents departments from accidentally interfering with each other, and makes it clear who owns what.

### Purposes

- To prevent state from becoming a globally entangled "god object" that is difficult to understand and maintain.
- To ensure that state is colocated with the domain or feature that owns it, improving cohesion.
- To establish clear public APIs between features, preventing implicit coupling.
- To reduce the blast radius of changes—modifying one feature's state does not affect unrelated features.
- To enable teams to work independently on different features without stepping on each other's toes.
- To distinguish between different types of state (server, client, form, URL) and assign each to the appropriate management tool.

### Syntax Rules and Structure

**Pattern 1: Feature-Sliced Design Layered Structure**

```
src/
├── app/                 # App-wide setup (providers, routing, global styles)
├── pages/               # Page-level compositions (route targets)
│   └── dashboard/
│       ├── ui/          # Page components
│       └── model/       # Page-level state (if any)
├── widgets/             # Large self-contained UI blocks
│   └── user-profile/
│       ├── ui/
│       └── model/
├── features/            # User interactions and business logic
│   └── add-to-cart/
│       ├── ui/
│       ├── model/       # Feature-scoped state
│       └── api/         # Feature-specific API calls
├── entities/            # Business entities (User, Product, Order)
│   └── product/
│       ├── ui/
│       ├── model/       # Entity state (normalised data)
│       └── api/
└── shared/              # Reusable utilities, UI kit, config
    ├── ui/
    ├── lib/
    └── api/
```

**Component Breakdown:**
- **app:** Global providers, routing, theme, and app-wide state.
- **pages:** Route-level components that compose widgets and features.
- **widgets:** Self-contained UI blocks that combine features and entities.
- **features:** User interactions (e.g., "add to cart", "login") with their own state.
- **entities:** Business entities (User, Product) with normalised state.
- **shared:** Reusable, domain-agnostic code.

**Pattern 2: State Colocation Decision Tree**

```
1. Is the state used by only one component?
   → YES: Keep it as local useState in that component.
   → NO: Continue.

2. Is the state used by a few closely related components in the same feature?
   → YES: Lift it to the closest common ancestor within the feature.
   → NO: Continue.

3. Is the state needed across multiple features or app-wide?
   → YES: Consider Context (for rarely changing data) or an external store.
   → NO: Continue.

4. Is the state server data (from an API)?
   → YES: Use TanStack Query or SWR (server state).
   → NO: Continue.

5. Is the state form data?
   → YES: Use React Hook Form or Formik (form state).
   → NO: Use URL state (React Router) or client state (Zustand).
```

**Syntax Rules:**
- Place state as close to the components that use it as possible. Only lift it when sharing is required.
- Distinguish between server state (API data), client state (UI flags, preferences), form state, and URL state. Use the appropriate tool for each.
- In Feature-Sliced Design, each slice (feature, entity) should expose a public API (index file) and keep its internal state private.
- A feature with both local store state and React Query data needs an explicit bridge in a custom hook or named action.
- Prefer view-scoped local feature stores over global stores for page-instance state.
- Use Context for rarely changing, global values (theme, locale, auth), not for frequently changing state.

**Constraints and Limitations:**
- Feature-Sliced Design introduces architectural overhead and a learning curve; it is best suited for medium-to-large applications.
- Over-architecting early can slow down development; start simple and refactor toward boundaries as the app grows.
- Determining the "right" boundary is subjective and depends on the domain; there is no one-size-fits-all answer.
- Moving state between scopes (e.g., from local to feature to global) requires refactoring; plan for this evolution.

### Annotated Code Example: Feature-Sliced State with Clear Boundaries

```jsx
// features/cart/model/cartStore.js — feature-scoped Zustand store
import { create } from 'zustand';

export const useCartStore = create((set) => ({
  items: [],
  addItem: (product) => set((state) => {
    const existing = state.items.find(i => i.id === product.id);
    if (existing) {
      return { items: state.items.map(i =>
        i.id === product.id ? { ...i, qty: i.qty + 1 } : i
      )};
    }
    return { items: [...state.items, { ...product, qty: 1 }] };
  }),
  removeItem: (id) => set((state) => ({
    items: state.items.filter(i => i.id !== id),
  })),
}));
```

```jsx
// features/cart/ui/AddToCartButton.jsx — uses the feature store
import { useCartStore } from '../model/cartStore';

export function AddToCartButton({ product }) {
  const addItem = useCartStore((state) => state.addItem);

  return (
    <button onClick={() => addItem(product)}>
      Add to Cart
    </button>
  );
}
```

```jsx
// widgets/cart-summary/ui/CartSummary.jsx — uses the same feature store
import { useCartStore } from '../../../features/cart/model/cartStore';

export function CartSummary() {
  const items = useCartStore((state) => state.items);
  const total = items.reduce((sum, item) => sum + item.price * item.qty, 0);

  return (
    <div>
      <h3>Cart ({items.length} items)</h3>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

```jsx
// pages/product-page/ui/ProductPage.jsx — page composes feature widgets
import { AddToCartButton } from '../../../features/cart/ui/AddToCartButton';
import { CartSummary } from '../../../widgets/cart-summary/ui/CartSummary';

export function ProductPage({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
      <AddToCartButton product={product} />
      <CartSummary />
    </div>
  );
}
```

**Expected Output:** A product page showing the product name, price, an "Add to Cart" button, and a cart summary. Clicking "Add to Cart" adds the product to the cart, updating the item count and total in the cart summary. Clicking again increments the quantity.

**Why This Output Occurs:** The `cartStore` is a feature-scoped Zustand store that lives in the `features/cart` slice. Both `AddToCartButton` (in the feature's `ui` layer) and `CartSummary` (in the `widgets` layer) import and use the same store. The `ProductPage` (in the `pages` layer) composes these components but does not manage cart state itself. The cart state is colocated with the cart feature—it is not in a global app store. If another feature needed cart data, it would import the `cartStore` from the cart feature's public API. This keeps the cart state scoped to its feature, with clear boundaries and a public API.

### Real-World Cases

- **E-commerce:** A `cart` feature owns the cart store; a `checkout` feature reads from it but does not modify it.
- **User authentication:** An `auth` feature owns the auth store; a `navbar` widget reads the current user from it.
- **Dashboard analytics:** A `filters` feature owns the filter state; multiple `chart` widgets read and update it.
- **Multi-step forms:** A `checkout` feature owns the form state across multiple steps; each step component reads and writes its portion.
- **Notification system:** A `notifications` feature owns the notification store; a `notification-bell` widget reads the unread count.

### References

- Feature-Sliced Design – Scaling Frontend Architecture: https://feature-sliced.design/
- Feature-Sliced Design – Layers and Slices: https://feature-sliced.design/docs/reference/layers
- React-Vite-Boilerplate – Architecture Overview: https://github.com/MhamedEl-shahawy/React-Vite-Boilerplate/blob/main/docs/ARCHITECTURE_OVERVIEW.md
- Redux – Code Structure: https://redux.js.org/usage/structuring-reducers/structuring-reducers
- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- React Hook Form – Documentation: https://react-hook-form.com/

---

## Comparison and Decision Guidance

| Pattern | Best For | Data Flow | Re-render Granularity | Provider Required | Boilerplate | Decoupling Level |
|---|---|---|---|---|---|---|
| **Context API** | Feature-scoped sharing (theme, auth, locale) | Implicit: provider → consumers | All consumers re-render | Yes | Medium | Medium |
| **Zustand** | Medium apps needing simple global state | Hook-based: store → hook | Selector-based (fine-grained) | No | Low | High |
| **Redux Toolkit** | Large apps with complex, predictable state | Centralised: dispatch → reducer → store | Selector-based (fine-grained) | Yes | High | High |
| **XState** | Complex workflows with explicit states | Event-driven: send → machine → state | Actor-based (fine-grained) | Yes (optional) | High | High |
| **Pub/Sub Events** | Notifications, analytics, cross-cutting concerns | Event bus: emit → listeners | N/A (subscribers update their own state) | No | Low | Very High |
| **Feature-Sliced Design** | Medium-to-large apps needing clear boundaries | Layered: feature → shared | Feature-scoped | N/A (architectural pattern) | Medium | High |

**Decision Guidance:**
- **Start with Context.** For most deep component communication, Context is sufficient. It is built-in, well-documented, and requires no dependencies.
- **Use Zustand when Context re-renders too often.** Zustand's selector-based subscriptions prevent unnecessary re-renders, and it requires no providers.
- **Use Redux Toolkit when you need predictability and devtools.** For large teams and complex state, Redux's strict unidirectional flow, middleware, and time-travel debugging are invaluable.
- **Use XState when your logic is a state machine.** If your feature has clear states and transitions (idle → loading → success/error), XState prevents invalid states and makes the logic visual and testable.
- **Use pub/sub for notifications and cross-cutting concerns.** Events are ideal for fire-and-forget notifications (toasts, analytics) where the publisher does not need a response.
- **Use Feature-Sliced Design to organise your codebase.** As the app grows, FSD provides a clear structure for where state should live and how features should communicate.

---

## References

- React Official Documentation – Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
- React Official Documentation – useContext: https://react.dev/reference/react/useContext
- React Official Documentation – Scaling Up with Reducer and Context: https://react.dev/learn/scaling-up-with-reducer-and-context
- React Legacy Documentation – Context: https://legacy.reactjs.org/docs/context.html
- Redux – Style Guide: https://redux.js.org/style-guide/
- Redux Toolkit – Quick Start: https://redux.js.org/tutorials/quick-start
- Redux – Code Structure: https://redux.js.org/usage/structuring-reducers/structuring-reducers
- Zustand – Comparison: https://zustand.docs.pmnd.rs/learn/getting-started/comparison
- Zustand – React Hooks: https://zustand.docs.pmnd.rs/reference/hooks/use-store
- XState – Core Concepts: https://stately.ai/docs/xstate
- XState – Use a machine in React: https://stately.ai/docs/xstate-react
- Feature-Sliced Design – Scaling Frontend Architecture: https://feature-sliced.design/
- Feature-Sliced Design – Layers and Slices: https://feature-sliced.design/docs/reference/layers
- MDN Web Docs – EventTarget: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget
- MDN Web Docs – CustomEvent: https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent
- tsecurity.de – EventTarget - CustomEvent | Components Communication in React: https://tsecurity.de/de/2761056/IT+Programmierung/EventTarget+-+CustomEvent+|+components+communication+in+React+-+part+Three/
- CSS-Tricks – Understanding Event Emitters: https://css-tricks.com/understanding-event-emitters/
- David Walsh – Pub/Sub JavaScript Object: https://davidwalsh.name/pubsub-javascript
- DEV Community – Redux Toolkit vs Zustand vs Jotai: https://dev.to/zeeshanali0704/frontend-system-design-redux-toolkit-vs-zustand-vs-jotai-1npn
- React-Vite-Boilerplate – Architecture Overview: https://github.com/MhamedEl-shahawy/React-Vite-Boilerplate/blob/main/docs/ARCHITECTURE_OVERVIEW.md
- TanStack Query – Overview: https://tanstack.com/query/latest/docs/framework/react/overview
- React Hook Form – Documentation: https://react-hook-form.com/