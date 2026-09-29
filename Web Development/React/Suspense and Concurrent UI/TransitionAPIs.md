# Transition APIs: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Transition APIs are React's set of Hooks and functions—`useTransition`, `startTransition`, and `useOptimistic`—that allow developers to mark certain state updates as non-urgent, enabling React to keep the UI responsive by prioritising urgent updates and deferring or interrupting non-urgent ones.

**Technical Definition:** Transition APIs are part of React 18's concurrent rendering feature set. They provide a mechanism to categorise state updates into two priority levels: **urgent** (e.g., typing, clicking, hovering) and **transition** (e.g., rendering search results, switching tabs, filtering lists). Updates wrapped in `startTransition` or executed via the `startTransition` function returned by `useTransition` are assigned `TransitionPriority` in React's internal Scheduler. This allows React to interrupt transition renders if an urgent update occurs, discard stale transition renders, and keep the existing UI visible until the new UI is ready. `useTransition` additionally provides an `isPending` boolean for tracking whether a transition is in progress. React 19 introduced `useOptimistic`, which combines transitions with optimistic state to display predicted results immediately while a background update processes.

**Beginner-Friendly Explanation:** When you type in a search box, you want the text to appear instantly—that's urgent. But updating a list of 10,000 search results? That can wait a moment. Transition APIs let you tell React which updates are urgent and which can be deferred. While the deferred update is happening in the background, the old UI stays visible and interactive, and you can show a subtle loading indicator. This keeps your app feeling fast and responsive, even when it's doing heavy work.

### Key Characteristics

- **Priority-Based Updates:** Urgent updates are processed synchronously; transition updates are processed in the background and can be interrupted.
- **Non-Blocking:** Transition updates do not block the main thread or prevent user interactions.
- **Interruptible:** If a new transition starts, React discards the in-progress transition render and restarts with the latest state.
- **Stale UI Preservation:** The existing UI remains visible and interactive during a transition; it is not replaced by a fallback unless no previous UI exists.
- **Pending State Tracking:** `useTransition` provides an `isPending` flag to indicate whether a transition is in progress.
- **Batching:** Multiple state updates inside a single `startTransition` call are batched together.
- **Composability:** Transitions can be nested, and `useOptimistic` builds on top of transitions for optimistic UI.

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useEffect`, `useMemo`).
- Familiarity with React's rendering lifecycle (render phase vs commit phase).
- Basic understanding of concurrent rendering and `createRoot`.
- Knowledge of asynchronous JavaScript and Promises (for optimistic updates).
- Awareness of the difference between urgent and non-urgent updates.

### Related Programming Areas

- **Concurrent Rendering:** The underlying architecture that enables transitions.
- **Suspense:** Integrates with transitions for deferred rendering.
- **Optimistic UI:** Instant feedback patterns for mutations.
- **State Management:** Coordinating local and global state updates.
- **Web Performance:** Interaction to Next Paint (INP), responsiveness.
- **User Experience:** Perceived performance, loading states, and feedback.

### Core Concepts / Features

1. The `useTransition` Hook
2. The `startTransition` Function
3. Optimistic UI Updates
4. Stale UI Handling

---

## Core Concept 1: The `useTransition` Hook

### Definitions

**Core Definition:** `useTransition` is a React Hook that returns a `startTransition` function and an `isPending` flag, allowing components to mark state updates as non-urgent and track the loading state of those updates.

**Technical Definition:** `useTransition()` is a Hook that must be called at the top level of a component. It returns a tuple `[isPending, startTransition]`. The `startTransition` function accepts a synchronous callback containing one or more state updates. These updates are assigned `TransitionPriority` by React's Scheduler, making them interruptible by urgent updates. The `isPending` boolean is `true` while any transition initiated by this Hook is in progress and `false` otherwise. React guarantees that `isPending` transitions to `true` before the transition render begins, preventing visual tearing between the urgent UI and the pending indicator.

**Beginner-Friendly Explanation:** Imagine you have a search box and a results list. You want the search box to update instantly as you type, but the results list can take a moment. `useTransition` gives you two things: a function to wrap the results update (telling React "this is not urgent") and a flag that tells you whether the results update is still in progress. You can use that flag to show a spinner or dim the results while they're updating.

### Purposes

- To separate urgent UI updates from non-urgent ones.
- To keep the UI responsive during heavy renders.
- To track the loading state of a transition via `isPending`.
- To prevent stale UI from blocking user interactions.
- To show subtle loading indicators without unmounting the current UI.
- To coordinate multiple state updates into a single transition.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useTransition } from 'react';

function MyComponent() {
  const [isPending, startTransition] = useTransition();

  function handleClick() {
    startTransition(() => {
      // Non-urgent state updates go here
      setState(newValue);
    });
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>Update</button>
    </div>
  );
}
```

**Component Breakdown:**
- `useTransition()`: Returns a tuple `[isPending, startTransition]`.
- `isPending`: A boolean indicating whether a transition is in progress.
- `startTransition(callback)`: A function that accepts a synchronous callback containing state updates.
- `setState(newValue)`: State updates inside the callback are marked as non-urgent.

**Syntax Rules:**
- `useTransition` must be called at the top level of a component or custom Hook. It cannot be called inside loops, conditions, or nested functions.
- The callback passed to `startTransition` must be synchronous. It cannot be an `async` function.
- State updates inside the callback are batched and marked as non-urgent.
- `isPending` is `true` from the moment `startTransition` is called until the transition render is committed.
- If a new transition starts while another is in progress, the previous transition is discarded, and `isPending` remains `true`.
- Multiple `useTransition` calls in the same component are independent; each has its own `isPending` flag.

**Constraints and Limitations:**
- Transitions cannot be used to control text inputs. React warns if you call `startTransition` for an update that controls an input's value.
- `isPending` does not indicate *which* transition is pending if multiple transitions are active; it only indicates whether *any* transition from this Hook is pending.
- Transitions do not work with `flushSync`.
- If a transition is interrupted too many times, React may eventually commit the current state to avoid starvation.
- `useTransition` requires `createRoot` (React 18+); it does not work with the legacy `ReactDOM.render`.

### Annotated Code Examples

**Example 1: Search Input with `useTransition`**

```jsx
import React, { useState, useTransition } from 'react';

// A component that renders a large, filterable list
function SearchResults({ query }) {
  // Generate a large list and filter it based on the query
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`)
    .filter(item => item.toLowerCase().includes(query.toLowerCase()));

  return (
    <ul>
      {items.slice(0, 50).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}

function SearchApp() {
  // Urgent state: the input value
  const [input, setInput] = useState('');
  // Non-urgent state: the query used for filtering
  const [query, setQuery] = useState('');
  // Transition Hook: tracks pending state and provides startTransition
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    // Urgent update: keep the input responsive
    setInput(e.target.value);

    // Non-urgent update: defer the query update
    startTransition(() => {
      setQuery(e.target.value);
    });
  }

  return (
    <div>
      <input
        value={input}
        onChange={handleChange}
        placeholder="Type to search..."
      />
      {/* Show a loading indicator while the transition is pending */}
      {isPending && <p>Updating results...</p>}
      {/* The results list renders with the deferred query */}
      <SearchResults query={query} />
    </div>
  );
}

export default SearchApp;
```

**Expected Output:** As the user types, the input field updates instantly with every keystroke. The search results list updates a moment later, and "Updating results..." appears briefly while the transition is in progress. If the user types quickly, intermediate search results are skipped, and only the final query's results are rendered.

**Why This Output Occurs:** The `setInput` call is urgent, so React processes it immediately, keeping the input responsive. The `setQuery` call inside `startTransition` is non-urgent, so React can interrupt it if the user types another character. When the user types quickly, React discards in-progress renders of `SearchResults` and restarts with the latest query. The `isPending` flag from `useTransition` indicates when the transition is in progress, allowing the UI to show "Updating results..." without unmounting the existing results list.

**Example 2: Tab Switching with `useTransition` and Pending Indicator**

```jsx
import React, { useState, useTransition } from 'react';

// A component that simulates a heavy render
function SlowTabContent({ tab }) {
  const start = performance.now();
  // Artificial delay: 100ms per render
  while (performance.now() - start < 100) {}

  return (
    <div style={{ padding: '20px', border: '1px solid #ccc' }}>
      <h2>Tab {tab}</h2>
      <p>This is the content for tab {tab}.</p>
    </div>
  );
}

function TabContainer() {
  const [tab, setTab] = useState('A');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <div>
      <div>
        <button onClick={() => selectTab('A')} disabled={tab === 'A'}>
          Tab A
        </button>
        <button onClick={() => selectTab('B')} disabled={tab === 'B'}>
          Tab B
        </button>
        <button onClick={() => selectTab('C')} disabled={tab === 'C'}>
          Tab C
        </button>
      </div>

      {/* Dim the content while the transition is pending */}
      <div style={{ opacity: isPending ? 0.5 : 1 }}>
        <SlowTabContent tab={tab} />
      </div>

      {isPending && <p>Switching tab...</p>}
    </div>
  );
}

export default TabContainer;
```

**Expected Output:** Clicking a tab button dims the current tab content (opacity 0.5) and shows "Switching tab...". After ~100ms, the new tab content appears and the dimming is removed. The buttons remain clickable throughout the transition.

**Why This Output Occurs:** The `selectTab` function wraps the `setTab` update in `startTransition`, marking it as non-urgent. React keeps the current tab visible while preparing the new tab's render in the background. The `isPending` flag drives the opacity change and the "Switching tab..." message. If the user clicks another tab before the first transition completes, React discards the in-progress render and starts a new transition for the latest tab.

### Real-World Cases

- **Search autocomplete:** Keeping the input responsive while filtering a large dataset.
- **Tab navigation:** Switching between heavy tabs without blocking the UI.
- **Filtering large lists:** Applying filters without freezing the interface.
- **Route transitions:** Preparing the next page while keeping the current one interactive.
- **Theme switching:** Applying a new theme without blocking user interactions.
- **Data visualisation updates:** Re-rendering complex charts without locking the main thread.

---

## Core Concept 2: The `startTransition` Function

### Definitions

**Core Definition:** `startTransition` is a standalone function exported by React that marks state updates as non-urgent, allowing it to be used outside of functional components, such as in utility modules, custom routers, or state management libraries.

**Technical Definition:** `startTransition(scope)` is a top-level React API that accepts a synchronous callback (`scope`) containing one or more state updates. Unlike the `startTransition` function returned by `useTransition`, the standalone `startTransition` does not provide an `isPending` flag because it is not associated with a component. It is designed for use in non-component contexts where a transition is needed but no pending indicator is required. React assigns `TransitionPriority` to the updates inside the callback. The function is stable and can be imported and used anywhere in the application, including outside React components.

**Beginner-Friendly Explanation:** Sometimes you need to trigger a non-urgent update from outside a React component—like in a router, a state management store, or a utility function. The `useTransition` Hook only works inside components, so React provides a standalone `startTransition` function that you can import and use anywhere. It does the same thing—marks updates as non-urgent—but without the `isPending` flag.

### Purposes

- To mark state updates as non-urgent from outside React components.
- To integrate transitions into routing libraries and state managers.
- To defer updates in utility functions and custom event handlers.
- To provide a consistent API for non-urgent updates across the application.
- To enable transitions in contexts where Hooks cannot be used.
- To coordinate transitions in global state management.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { startTransition } from 'react';

// Outside a component (e.g., in a router or store)
function navigateTo(path) {
  startTransition(() => {
    // State updates marked as non-urgent
    setRoute(path);
  });
}
```

**Component Breakdown:**
- `import { startTransition } from 'react'`: Imports the standalone function.
- `startTransition(() => { ... })`: Wraps state updates in a transition.
- `setRoute(path)`: The state update inside the callback is marked as non-urgent.
- Unlike `useTransition`, there is no `isPending` flag.

**Usage in a Custom Router:**
```jsx
import { startTransition } from 'react';

function createRouter() {
  let listeners = [];
  let route = '/';

  function navigate(path) {
    startTransition(() => {
      route = path;
      listeners.forEach(listener => listener(route));
    });
  }

  function subscribe(listener) {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  }

  function getRoute() {
    return route;
  }

  return { navigate, subscribe, getRoute };
}

export const router = createRouter();
```

**Component Breakdown:**
- `createRouter()`: Creates a router instance with `navigate`, `subscribe`, and `getRoute` methods.
- `navigate(path)`: Wraps the route update in `startTransition`, marking it as non-urgent.
- `listeners.forEach(...)`: Notifies all subscribers of the route change.
- Because `startTransition` is standalone, it can be used in this non-component context.

**Usage in a State Management Store:**
```jsx
import { startTransition } from 'react';

function createStore(initialState) {
  let state = initialState;
  let listeners = [];

  function setState(updater, options = {}) {
    const nextState = typeof updater === 'function' ? updater(state) : updater;

    if (options.transition) {
      startTransition(() => {
        state = nextState;
        listeners.forEach(listener => listener(state));
      });
    } else {
      state = nextState;
      listeners.forEach(listener => listener(state));
    }
  }

  function subscribe(listener) {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  }

  function getState() {
    return state;
  }

  return { setState, subscribe, getState };
}
```

**Component Breakdown:**
- `setState(updater, options)`: Accepts an updater function and options.
- `options.transition`: If `true`, the update is wrapped in `startTransition`.
- `startTransition(() => { ... })`: Marks the update as non-urgent.
- This pattern is used by state management libraries like Zustand and Jotai to support transitions.

**Syntax Rules:**
- `startTransition` is a stable function that can be imported and used anywhere.
- The callback must be synchronous; it cannot be an `async` function.
- State updates inside the callback are batched and marked as non-urgent.
- There is no `isPending` flag with the standalone `startTransition`; use `useTransition` if you need a pending indicator.
- `startTransition` can be called from event handlers, utility functions, routers, and stores.
- If `startTransition` is called outside of a React event handler, React may not batch the updates.

**Constraints and Limitations:**
- The standalone `startTransition` does not provide an `isPending` flag.
- It cannot be used to control text inputs (React warns about this).
- It does not work with `flushSync`.
- Transitions are only interruptible if React is using concurrent rendering (`createRoot`).
- If called outside of a React event handler, updates may not be batched, leading to multiple renders.
- In React 18, `startTransition` outside of components may not work as expected with some state management libraries; check library documentation for support.

### Annotated Code Examples

**Example 1: Custom Router with `startTransition`**

```jsx
import React, { useState, useEffect, startTransition } from 'react';

// A simple router store
const routerStore = {
  route: '/',
  listeners: new Set(),
  navigate(path) {
    startTransition(() => {
      this.route = path;
      this.listeners.forEach(listener => listener(path));
    });
  },
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },
  getRoute() {
    return this.route;
  },
};

// Hook to use the router in components
function useRoute() {
  const [route, setRoute] = useState(routerStore.getRoute());

  useEffect(() => {
    const unsubscribe = routerStore.subscribe(setRoute);
    return unsubscribe;
  }, []);

  return route;
}

// A heavy page component
function HeavyPage({ route }) {
  const start = performance.now();
  while (performance.now() - start < 100) {}

  return <div>Content for route: {route}</div>;
}

function App() {
  const route = useRoute();

  return (
    <div>
      <nav>
        <button onClick={() => routerStore.navigate('/home')}>Home</button>
        <button onClick={() => routerStore.navigate('/about')}>About</button>
        <button onClick={() => routerStore.navigate('/contact')}>Contact</button>
      </nav>
      <HeavyPage route={route} />
    </div>
  );
}

export default App;
```

**Expected Output:** Clicking a navigation button updates the route. The current page content remains visible and interactive while the new page renders in the background. Once the new page is ready, it replaces the old content.

**Why This Output Occurs:** The `routerStore.navigate` method wraps the route update in the standalone `startTransition` function. This marks the update as non-urgent, allowing React to interrupt it if the user clicks another button. The `useRoute` Hook subscribes to the store and updates the component when the route changes. The `HeavyPage` component simulates a heavy render, demonstrating that the UI remains responsive during the transition.

**Example 2: State Store with Optional Transitions**

```jsx
import React, { useState, useEffect, startTransition } from 'react';

// A minimal state store with transition support
function createStore(initialState) {
  let state = initialState;
  const listeners = new Set();

  return {
    getState: () => state,
    setState: (updater, { transition = false } = {}) => {
      const nextState =
        typeof updater === 'function' ? updater(state) : updater;

      const notify = () => {
        state = nextState;
        listeners.forEach(listener => listener(state));
      };

      if (transition) {
        startTransition(notify);
      } else {
        notify();
      }
    },
    subscribe: (listener) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    },
  };
}

const counterStore = createStore({ count: 0 });

function useCounter() {
  const [state, setState] = useState(counterStore.getState());

  useEffect(() => {
    return counterStore.subscribe(setState);
  }, []);

  return state;
}

function CounterDisplay() {
  const { count } = useCounter();

  // Simulate a heavy render
  const start = performance.now();
  while (performance.now() - start < 50) {}

  return <p>Count: {count}</p>;
}

function App() {
  return (
    <div>
      <button
        onClick={() =>
          counterStore.setState(
            (s) => ({ count: s.count + 1 }),
            { transition: true }
          )
        }
      >
        Increment (Transition)
      </button>
      <button
        onClick={() =>
          counterStore.setState((s) => ({ count: s.count + 1 }))
        }
      >
        Increment (Urgent)
      </button>
      <CounterDisplay />
    </div>
  );
}

export default App;
```

**Expected Output:** Clicking "Increment (Transition)" updates the counter without blocking the UI, even though `CounterDisplay` has a heavy render. Clicking "Increment (Urgent)" updates the counter synchronously, which may block the UI briefly.

**Why This Output Occurs:** The store's `setState` method accepts a `transition` option. When `true`, it wraps the notification in `startTransition`, marking the update as non-urgent. The `CounterDisplay` component subscribes to the store and re-renders when the state changes. Because the transition update is non-urgent, React can interrupt it if the user interacts with the UI.

### Real-World Cases

- **Routers:** Integrating transitions into React Router, TanStack Router, or custom routers.
- **State management:** Adding transition support to Zustand, Jotai, or Redux stores.
- **Utility functions:** Deferring updates in debounce/throttle utilities.
- **Analytics:** Sending non-urgent updates to analytics services.
- **Web Workers:** Coordinating state updates from worker threads.
- **Third-party libraries:** Providing a transition-aware API for non-React code.

---

## Core Concept 3: Optimistic UI Updates

### Definitions

**Core Definition:** Optimistic UI updates are a pattern where the UI is updated immediately with a predicted result before the server confirms the operation, using React's `useOptimistic` Hook combined with transitions.

**Technical Definition:** `useOptimistic(state, updateFn)` is a React Hook introduced in React 19 that returns an optimistic version of the state and a function to update it. When called, the optimistic update is applied immediately, and the UI re-renders with the predicted state. When the underlying async operation (e.g., a form submission or data mutation) completes, the optimistic state is discarded and replaced with the actual result. The optimistic update is temporary and automatically reverts if the operation fails or the component re-renders without the async operation completing. `useOptimistic` must be used inside a transition (e.g., within `startTransition` or an async action) for the optimistic state to persist during the async operation.

**Beginner-Friendly Explanation:** Imagine you're using a "like" button on a social media post. Instead of waiting for the server to confirm your like, the button instantly turns blue and the count goes up. That's an optimistic update—you're predicting that the server will accept your request. If the server fails, the button reverts to its original state. `useOptimistic` is React's way of making this pattern easy and automatic.

### Purposes

- To provide instant feedback for user actions that involve server communication.
- To improve perceived performance by eliminating waiting time.
- To handle mutations (likes, comments, form submissions) without blocking the UI.
- To automatically revert optimistic state if the operation fails.
- To combine the responsiveness of transitions with optimistic predictions.
- To simplify the implementation of optimistic UI patterns.

### Syntax Rules and Structure

**General Syntax:**
```jsx
import { useOptimistic, startTransition } from 'react';

function MyComponent() {
  const [state, setState] = useState(initialState);

  // useOptimistic returns the optimistic state and an updater function
  const [optimisticState, addOptimistic] = useOptimistic(
    state,
    (currentState, optimisticValue) => {
      // Return the new optimistic state
      return [...currentState, optimisticValue];
    }
  );

  async function handleSubmit(newItem) {
    // Apply the optimistic update immediately
    addOptimistic(newItem);

    // Perform the actual async operation
    await saveToServer(newItem);

    // Update the real state (optimistic state reverts automatically)
    setState(prev => [...prev, newItem]);
  }

  return (
    <div>
      {optimisticState.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}
```

**Component Breakdown:**
- `useOptimistic(state, updateFn)`: Returns `[optimisticState, addOptimistic]`.
- `state`: The real state that will be updated after the async operation.
- `updateFn(currentState, optimisticValue)`: A function that returns the new optimistic state based on the current state and the optimistic value.
- `optimisticState`: The state to render, which includes optimistic updates.
- `addOptimistic(value)`: A function to apply an optimistic update.
- When `state` changes (after the async operation), `optimisticState` reverts to the real state.

**Optimistic Update with Form Submission:**
```jsx
import { useOptimistic, useState, useRef } from 'react';

function MessageForm() {
  const [messages, setMessages] = useState([]);
  const formRef = useRef(null);

  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages, newMessage) => [
      ...currentMessages,
      { text: newMessage, sending: true },
    ]
  );

  async function formAction(formData) {
    const message = formData.get('message');
    addOptimisticMessage(message);
    formRef.current.reset();

    // Simulate sending to server
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Update real state
    setMessages(prev => [...prev, { text: message, sending: false }]);
  }

  return (
    <div>
      <form ref={formRef} action={formAction}>
        <input name="message" type="text" required />
        <button type="submit">Send</button>
      </form>
      <ul>
        {optimisticMessages.map((msg, index) => (
          <li key={index} style={{ opacity: msg.sending ? 0.5 : 1 }}>
            {msg.text} {msg.sending && '(sending...)'}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Component Breakdown:**
- `useOptimistic(messages, ...)`: Creates an optimistic version of the messages list.
- `addOptimisticMessage(message)`: Adds a message optimistically with `sending: true`.
- `formAction(formData)`: The async action that performs the real update.
- `setMessages(...)`: Updates the real state, which reverts the optimistic state.
- The optimistic message appears immediately with reduced opacity and a "(sending...)" label.

**Syntax Rules:**
- `useOptimistic` must be called at the top level of a component or custom Hook.
- The `updateFn` must be a pure function that returns the new optimistic state.
- `addOptimistic` can only be called inside a transition (e.g., during a form action or inside `startTransition`).
- The optimistic state reverts automatically when the real state updates or when the component re-renders without an active transition.
- `useOptimistic` can be used with any state type, not just arrays.
- Multiple `useOptimistic` calls can be used in the same component.

**Constraints and Limitations:**
- `useOptimistic` does not persist across page reloads; it is temporary.
- If the async operation fails, the optimistic state reverts, but you must handle the error separately (e.g., with an Error Boundary or try/catch).
- `addOptimistic` must be called inside a transition; calling it outside a transition logs a warning.
- The optimistic update is discarded when the component re-renders without an active transition.
- `useOptimistic` is a React 19 feature; it is not available in React 18.
- Optimistic updates cannot be used for operations that require server-generated data (e.g., IDs, timestamps) unless you provide temporary placeholders.

### Annotated Code Examples

**Example 1: Optimistic Like Button**

```jsx
import React, { useState, useOptimistic, startTransition } from 'react';

function LikeButton({ postId, initialLikes, initialLiked }) {
  // Real state from the server
  const [likes, setLikes] = useState(initialLikes);
  const [liked, setLiked] = useState(initialLiked);

  // Optimistic state
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likes,
    (currentLikes, delta) => currentLikes + delta
  );

  const [optimisticLiked, setOptimisticLiked] = useOptimistic(
    liked,
    (_, newLiked) => newLiked
  );

  function handleLike() {
    const newLiked = !optimisticLiked;
    const delta = newLiked ? 1 : -1;

    startTransition(async () => {
      // Apply optimistic updates
      addOptimisticLike(delta);
      setOptimisticLiked(newLiked);

      try {
        // Simulate server request
        await new Promise(resolve => setTimeout(resolve, 500));

        // Update real state
        setLikes(prev => prev + delta);
        setLiked(newLiked);
      } catch (error) {
        // Optimistic state reverts automatically
        console.error('Failed to update like:', error);
      }
    });
  }

  return (
    <button onClick={handleLike}>
      {optimisticLiked ? '❤️' : '🤍'} {optimisticLikes}
    </button>
  );
}

export default function App() {
  return <LikeButton postId={1} initialLikes={42} initialLiked={false} />;
}
```

**Expected Output:** The button displays "🤍 42" initially. Clicking it instantly changes to "❤️ 43" (optimistic update). After 500ms (when the simulated server request completes), the button remains "❤️ 43" because the real state has been updated. If the request were to fail, the button would revert to "🤍 42".

**Why This Output Occurs:** The `handleLike` function wraps the optimistic updates in `startTransition`. The `addOptimisticLike` and `setOptimisticLiked` calls update the optimistic state immediately, causing the UI to show the predicted result. After the async operation completes, `setLikes` and `setLiked` update the real state, which causes the optimistic state to revert to the real state (which matches the optimistic prediction). If the operation fails, the real state is not updated, and the optimistic state reverts to the original values.

**Example 2: Optimistic Todo List**

```jsx
import React, { useState, useOptimistic, useRef } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build a project', completed: false },
  ]);
  const formRef = useRef(null);

  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => [
      ...currentTodos,
      { id: Date.now(), text: newTodo, completed: false, pending: true },
    ]
  );

  async function addTodo(formData) {
    const text = formData.get('todo');
    if (!text.trim()) return;

    // Apply optimistic update
    addOptimisticTodo(text);
    formRef.current.reset();

    // Simulate server request
    await new Promise(resolve => setTimeout(resolve, 800));

    // Update real state
    setTodos(prev => [
      ...prev,
      { id: Date.now(), text, completed: false },
    ]);
  }

  return (
    <div>
      <h1>Todo List</h1>
      <form ref={formRef} action={addTodo}>
        <input name="todo" type="text" placeholder="Add a todo..." />
        <button type="submit">Add</button>
      </form>
      <ul>
        {optimisticTodos.map(todo => (
          <li
            key={todo.id}
            style={{ opacity: todo.pending ? 0.5 : 1 }}
          >
            {todo.text} {todo.pending && '(saving...)'}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default TodoApp;
```

**Expected Output:** The list displays "Learn React" and "Build a project". When the user types a new todo and submits the form, the new todo appears instantly with reduced opacity and "(saving...)" next to it. After 800ms, the "(saving...)" label disappears and the todo becomes fully opaque.

**Why This Output Occurs:** The `addTodo` function is a form action. React automatically wraps form actions in a transition. The `addOptimisticTodo(text)` call adds the new todo optimistically with `pending: true`. The UI renders the optimistic todo with reduced opacity. After the simulated server request completes, `setTodos` updates the real state, and the optimistic state reverts to the real state, which includes the new todo without the `pending` flag.

### Real-World Cases

- **Social media likes:** Instantly updating the like count and heart icon.
- **Todo lists:** Adding, editing, and deleting todos with instant feedback.
- **Chat applications:** Sending messages that appear instantly while being delivered.
- **E-commerce carts:** Adding items to the cart with instant feedback.
- **Form submissions:** Showing the submitted data immediately while the server processes it.
- **Comment systems:** Posting comments that appear instantly.

---

## Core Concept 4: Stale UI Handling

### Definitions

**Core Definition:** Stale UI handling is the practice of keeping the existing UI visible and interactive during a transition, rather than replacing it with a blank skeleton or loading state, until the new UI is ready.

**Technical Definition:** In concurrent rendering, when a transition is in progress, React keeps the current (stale) UI on screen. The stale UI is not unmounted or replaced by a fallback unless no previous UI exists. React prepares the new UI in the background (work-in-progress Fiber tree) and commits it only when it is complete. If a higher-priority update occurs, React interrupts the transition, keeps the stale UI, and processes the urgent update. This behaviour is distinct from Suspense, where a fallback is shown if no content is available. With transitions, the stale UI remains interactive, and `isPending` can be used to show a subtle loading indicator without tearing down the layout.

**Beginner-Friendly Explanation:** Imagine you're browsing a photo gallery and you click a filter. Instead of the entire gallery disappearing and showing a blank loading screen, the current photos stay visible (maybe slightly dimmed) while React prepares the filtered results in the background. When the new results are ready, they replace the old ones. This feels much smoother than a blank screen because you never lose context.

### Purposes

- To prevent jarring layout shifts during transitions.
- To maintain user context while new content is being prepared.
- To keep the UI interactive during heavy renders.
- To provide subtle loading feedback without unmounting content.
- To improve perceived performance and user experience.
- To avoid the "flash of empty content" that occurs with skeleton-only loading states.

### Syntax Rules and Structure

**General Syntax:**
```jsxfunction MyComponent() {
  const [isPending, startTransition] = useTransition();
  const [data, setData] = useState(initialData);

  function handleUpdate(newData) {
    startTransition(() => {
      setData(newData);
    });
  }

  return (
    <div>
      {/* Stale UI remains visible; dim it while pending */}
      <div style={{ opacity: isPending ? 0.7 : 1 }}>
        <ExpensiveComponent data={data} />
      </div>
      {isPending && <p>Updating...</p>}
    </div>
  );
}
```

**Component Breakdown:**
- `isPending`: `true` while the transition is in progress.
- `style={{ opacity: isPending ? 0.7 : 1 }}`: Dims the stale UI without unmounting it.
- `ExpensiveComponent`: Remains mounted and interactive during the transition.
- The stale UI is replaced only when the transition commits.

**Stale UI vs Suspense Fallback:**
```jsx
// ❌ Suspense fallback: unmounts the old UI and shows a skeleton
<Suspense fallback={<Skeleton />}>
  <HeavyContent />
</Suspense>

// ✅ Transition: keeps the old UI visible and dims it
function TransitionWrapper() {
  const [isPending, startTransition] = useTransition();
  return (
    <div style={{ opacity: isPending ? 0.7 : 1 }}>
      <HeavyContent />
    </div>
  );
}
```

**Component Breakdown:**
- Suspense fallback: When content suspends, React unmounts the old content and shows the fallback. This causes a layout shift.
- Transition: When a transition is in progress, React keeps the old content mounted and visible. The `isPending` flag can be used for subtle visual feedback.

**Syntax Rules:**
- Use transitions (not Suspense) when you want to keep the existing UI visible during an update.
- Use the `isPending` flag to apply subtle visual feedback (dimming, blurring, loading indicator).
- Do not unmount the stale UI during a transition; React handles this automatically.
- If the transition is interrupted, the stale UI remains visible until a new render is committed.
- Combine transitions with `useDeferredValue` for a simpler API when you don't need `isPending`.

**Constraints and Limitations:**
- Stale UI handling only applies to transitions, not to Suspense boundaries.
- The stale UI is not updated during the transition; it shows the previous state until the transition commits.
- If the transition takes a long time, the stale UI may become outdated; use `isPending` to inform the user.
- In React 18, transitions initiated outside of `startTransition` (e.g., urgent updates) do not preserve stale UI; they may show a fallback.
- Stale UI handling does not apply to the initial mount; there is no previous UI to preserve.

### Annotated Code Examples

**Example 1: Stale UI vs Suspense Fallback Comparison**

```jsx
import React, { useState, useTransition, Suspense, lazy } from 'react';

const HeavyList = lazy(() => import('./HeavyList'));

// ❌ Suspense approach: unmounts old UI, shows skeleton
function SuspenseApproach({ filter }) {
  return (
    <Suspense fallback={<p>Loading list...</p>}>
      <HeavyList filter={filter} />
    </Suspense>
  );
}

// ✅ Transition approach: keeps old UI visible, dims it
function TransitionApproach({ filter }) {
  const [isPending, startTransition] = useTransition();
  const [currentFilter, setCurrentFilter] = useState(filter);

  function handleFilterChange(newFilter) {
    startTransition(() => {
      setCurrentFilter(newFilter);
    });
  }

  return (
    <div>
      <button onClick={() => handleFilterChange('A')}>Filter A</button>
      <button onClick={() => handleFilterChange('B')}>Filter B</button>
      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        <HeavyList filter={currentFilter} />
      </div>
      {isPending && <p>Updating list...</p>}
    </div>
  );
}
```

**Expected Output:** With the Suspense approach, clicking a filter button replaces the entire list with "Loading list..." while the new list loads. With the Transition approach, the old list remains visible (dimmed to 60% opacity) while the new list is prepared, and "Updating list..." appears. The old list is replaced only when the new list is ready.

**Why This Output Occurs:** In the Suspense approach, `HeavyList` suspends when the filter changes, causing React to unmount the old list and show the fallback. In the Transition approach, the filter change is wrapped in `startTransition`, so React keeps the old list mounted and visible while preparing the new list in the background. The `isPending` flag drives the opacity change and the loading message. This demonstrates the core difference: Suspense replaces, transitions preserve.

**Example 2: Search Results with Stale UI Preservation**

```jsx
import React, { useState, useTransition } from 'react';

function SearchResults({ query }) {
  // Simulate a heavy filter operation
  const items = Array.from({ length: 5000 }, (_, i) => `Result ${i + 1}`)
    .filter(item => item.toLowerCase().includes(query.toLowerCase()));

  return (
    <ul>
      {items.slice(0, 30).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}

function SearchApp() {
  const [input, setInput] = useState('');
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    setInput(e.target.value);
    startTransition(() => {
      setQuery(e.target.value);
    });
  }

  return (
    <div>
      <input
        value={input}
        onChange={handleChange}
        placeholder="Search..."
      />
      {isPending && <p>Updating results...</p>}
      {/* Stale results remain visible and dimmed */}
      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        <SearchResults query={query} />
      </div>
    </div>
  );
}

export default SearchApp;
```

**Expected Output:** As the user types, the input updates instantly. The existing search results remain visible (dimmed) while new results are computed in the background. When the new results are ready, they replace the old ones and the dimming is removed.

**Why This Output Occurs:** The `setQuery` update is wrapped in `startTransition`, so React keeps the old `SearchResults` mounted and visible while computing the new results. The `isPending` flag drives the opacity change, providing subtle feedback without unmounting the list. This avoids the jarring "blank screen" effect that would occur if the list were replaced by a skeleton on every keystroke.

### Real-World Cases

- **Search results:** Keeping previous results visible while new ones load.
- **Tab switching:** Keeping the current tab visible while the next tab renders.
- **Data filtering:** Keeping the filtered list visible while applying new filters.
- **Theme switching:** Keeping the current theme visible while the new theme loads.
- **Route transitions:** Keeping the current page visible while the next page loads.
- **Dashboard updates:** Keeping current chart data visible while new data loads.

---

## References

- React Official Documentation – `useTransition`: https://react.dev/reference/react/useTransition
- React Official Documentation – `startTransition`: https://react.dev/reference/react/startTransition
- React Official Documentation – `useOptimistic`: https://react.dev/reference/react/useOptimistic
- React Official Documentation – `useDeferredValue`: https://react.dev/reference/react/useDeferredValue
- React Official Documentation – Managing State with Transitions: https://react.dev/learn/managing-state
- React Official Documentation – `<Suspense>`: https://react.dev/reference/react/Suspense
- React 18 Release Notes – Transitions: https://react.dev/blog/2022/03/29/react-v18#new-feature-transitions
- React 19 Release Notes – `useOptimistic`: https://react.dev/blog/2024/12/05/react-19
- React Working Group – `startTransition` and `useTransition`: https://github.com/reactwg/react-18/discussions/129
- React Working Group – `useDeferredValue`: https://github.com/reactwg/react-18/discussions/129
- React Working Group – Concurrent React: https://github.com/reactwg/react-18/discussions/64
- React Working Group – New Suspense SSR Architecture: https://github.com/reactwg/react-18/discussions/37
- TanStack Query – Suspense Guide: https://tanstack.com/query/v5/docs/framework/react/guides/suspense
- TanStack Query – Optimistic Updates: https://tanstack.com/query/v5/docs/framework/react/guides/optimistic-updates
- Next.js – `useOptimistic` Guide: https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- Epic React – How React Suspense Works Under the Hood: https://www.epicreact.dev/how-react-suspense-works-under-the-hood-throwing-promises-and-declarative-loading-states
- Steve Kinney – `useTransition` and `startTransition`: https://stevekinney.com/courses/react-performance/use-transition
- Steve Kinney – Optimistic Updates: https://stevekinney.com/courses/react-performance/optimistic-updates
- Syncfusion – React 19 `useOptimistic` Hook: https://www.syncfusion.com/blogs/post/react-19-useoptimistic-hook
- LogRocket – A Guide to React 18's `useTransition`: https://blog.logrocket.com/react-18-usetransition/
- FreeCodeCamp – The Modern React Data Fetching Handbook: https://www.freecodecamp.org/news/the-modern-react-data-fetching-handbook-suspense-use-and-errorboundary-explained/
- MDN Web Docs – Promise: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- MDN Web Docs – Scheduler API: https://developer.mozilla.org/en-US/docs/Web/API/Prioritized_Task_Scheduling_API