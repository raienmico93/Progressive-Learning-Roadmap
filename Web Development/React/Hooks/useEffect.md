# useEffect — Comprehensive Programming Cheat Sheet

## Topic Overview

### Definitions

**Core Definition**

`useEffect` is a React Hook that lets you synchronize a component with an external system by running side effects after rendering and cleaning up when the component unmounts or dependencies change.

**Technical Definition**

`useEffect` accepts a setup function and an optional dependency array. React calls the setup function after the component is committed to the DOM, and if the setup returns a cleanup function, React calls that cleanup before the next setup (when dependencies change) or when the component unmounts. The dependency array contains all reactive values (props, state, and variables declared in the component body) that the Effect reads; React compares these using `Object.is` and skips re-running the Effect if all dependencies are unchanged .

**Beginner-Friendly Explanation**

`useEffect` is how you tell React to do something "on the side" after your component appears on screen—like setting a timer, fetching data, or subscribing to an event. It's also how you clean up after yourself when the component disappears. Think of it as a way to connect your component to the outside world.

### Key Characteristics

- **Runs After Commit**: Effects run after the browser has painted the DOM, not during render .
- **Dependency-Driven**: The dependency array controls when the Effect re-runs; React compares dependencies by identity .
- **Cleanup Prevents Leaks**: Returning a cleanup function from the Effect prevents memory leaks from timers, subscriptions, and event listeners .
- **Escape Hatch**: Effects should be used to synchronize with external systems, not for data flow between components .

### Prerequisites

- React Components and JSX
- State (`useState`) and Props
- Basic JavaScript closures and asynchronous code

### Related Programming Areas

- **Lifecycle Methods**: `useEffect` replaces `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.
- **Data Fetching**: Effects are commonly used for network requests (though libraries like React Query are recommended for production) .
- **Subscriptions**: WebSocket connections, event listeners, and timers.

### Core Concepts / Features

- Side Effects
- Dependency Arrays
- Cleanup Functions
- Synchronizing with External Systems
- Timers
- Subscriptions
- Network Requests
- Event Listeners
- Common Dependency Mistakes

---

## 1. Side Effects

### Definitions

**Core Definition**

A side effect is any operation that reaches outside the pure rendering logic of a component, such as modifying the DOM, fetching data, or setting up a subscription.

**Technical Definition**

In React, rendering must be pure: given the same props and state, a component returns the same JSX without modifying anything external. Side effects are operations that violate this purity by interacting with systems outside React—the browser DOM, network, timers, or external libraries. `useEffect` provides a controlled place to run these operations after rendering, keeping the render phase pure .

**Beginner-Friendly Explanation**

Rendering a component should be like a math equation—same inputs, same output, no surprises. But sometimes you need to do things that aren't just math, like fetching data or setting a timer. Those are "side effects," and `useEffect` is where you put them.

### Purposes

- To run operations that cannot happen during render.
- To synchronize component state with external systems.
- To keep the render phase pure and predictable.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  // Side effect logic here
});
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `useEffect` | The Hook call. |
| `() => { ... }` | The setup function containing the side effect. |

**Syntax Rules**

1. **Effects run after render**: They do not block the browser from painting .
2. **Do not use Effects for data flow**: If you're just adjusting state based on props/state, derive it during render instead .
3. **Effects are for external synchronization**: Use them when there's an external system to sync with.

**Constraints and Limitations**

- Effects run after every render by default (no dependency array).
- Effects should not be used to transform data for rendering; compute derived values during render.

### Annotated Complete Code Example

**Example: DOM Title Synchronization**

```jsx
import { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  // Side effect: update the document title
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

**Expected Behavior**

The browser tab title updates every time the button is clicked.

**Why This Occurs**

The Effect runs after each render. It reads `count` and updates `document.title`, which is outside React's control—a genuine side effect .

### Real-World Cases

**Case: Analytics Logging**

A component logs page views to an analytics service when it mounts. This is an external system interaction that belongs in an Effect.

### References

- React — Synchronizing with Effects - https://18.react.dev/learn/synchronizing-with-effects

---

## 2. Dependency Arrays

### Definitions

**Core Definition**

The dependency array is the optional second argument to `useEffect` that tells React which reactive values the Effect reads, controlling when the Effect re-runs.

**Technical Definition**

The dependency array contains all props, state, and variables declared inside the component that are referenced within the Effect. React compares each dependency with its previous value using `Object.is`. If any dependency has changed, React runs the cleanup from the previous Effect, then runs the new Effect. If all dependencies are unchanged, the Effect is skipped .

**Beginner-Friendly Explanation**

The dependency array is like a watch list. You tell React, "only re-run this effect if any of these values change." If none of them change, React skips the effect.

### Purposes

- To skip unnecessary Effect runs when values haven't changed.
- To control exactly when an Effect should re-execute.
- To make Effect behavior predictable and performant.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  // Effect logic
}, [dep1, dep2]);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `[dep1, dep2]` | Array of reactive values the Effect reads. |
| `[]` | Empty array: Effect runs only on mount. |
| No array | Effect runs after every render. |

**Syntax Rules**

1. **All reactive values must be included**: The linter (`exhaustive-deps`) enforces this .
2. **Empty array means "mount only"**: The Effect runs once after the initial render and cleans up on unmount .
3. **React chooses dependencies based on code**: You cannot arbitrarily omit a dependency that the Effect reads .

**Constraints and Limitations**

- Suppressing the linter (`eslint-disable`) leads to stale closures and bugs .
- Functions and objects declared in the component body are recreated every render and cause infinite loops if used as dependencies .

### Annotated Complete Code Examples

**Example 1: Dependency Array with Props**

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // Re-run when either changes

  return <div>Connected to {roomId}</div>;
}
```

**Explanation**

The Effect reads `serverUrl` and `roomId`. Both are listed as dependencies. Changing either value causes the Effect to reconnect with the new values .

**Example 2: Empty Dependency Array**

```jsx
useEffect(() => {
  console.log('Component mounted');
  return () => console.log('Component unmounted');
}, []); // Run once on mount, cleanup on unmount
```

**Explanation**

The empty array means the Effect has no reactive dependencies. It runs once after the initial render and cleans up when the component is removed .

### Real-World Cases

**Case: Chat Room Connection**

A chat component reconnects to the server when `roomId` or `serverUrl` changes. The dependency array ensures reconnection only happens when necessary .

### References

- React — Specifying Effect Dependencies - https://18.react.dev/learn/synchronizing-with-effects
- React — Exhaustive Deps Lint Rule - https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## 3. Cleanup Functions

### Definitions

**Core Definition**

A cleanup function is a function returned from a `useEffect` setup that React calls to undo the Effect's side effects before re-running the Effect or unmounting the component.

**Technical Definition**

When the setup function returns a function, React treats it as a cleanup function. Before running the Effect again (when dependencies change), React calls the previous cleanup. When the component unmounts, React calls the final cleanup. Cleanup prevents memory leaks by canceling subscriptions, clearing timers, and removing event listeners .

**Beginner-Friendly Explanation**

A cleanup function is like telling React, "before you re-run this effect or remove my component, here's how to clean up after yourself." It's how you cancel timers, close connections, and remove listeners.

### Purposes

- To prevent memory leaks from timers, subscriptions, and listeners.
- To cancel pending network requests when the component unmounts.
- To ensure the external system is returned to a clean state.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  // Setup: start something
  const subscription = subscribe();
  return () => {
    // Cleanup: undo it
    subscription.unsubscribe();
  };
}, [dependencies]);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `subscribe()` | Setup logic. |
| `return () => { ... }` | Cleanup function. |
| `unsubscribe()` | Undo the setup. |

**Syntax Rules**

1. **Return a function**: Only functions are treated as cleanup.
2. **Cleanup runs before next Effect**: When dependencies change, cleanup runs with the old values .
3. **Cleanup runs on unmount**: The final cleanup runs when the component is removed .

**Constraints and Limitations**

- Not all Effects need cleanup (e.g., logging to the console).
- Cleanup should be idempotent (safe to call multiple times).

### Annotated Complete Code Examples

**Example 1: Timer Cleanup**

```jsx
useEffect(() => {
  const intervalId = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  return () => clearInterval(intervalId); // Cleanup
}, []);
```

**Explanation**

The setup creates an interval. The cleanup clears it. Without cleanup, the interval would continue running after the component unmounts, causing a memory leak and potential errors .

**Example 2: Subscription Cleanup**

```jsx
useEffect(() => {
  const subscription = subscribeToData(id);
  return () => subscription.unsubscribe();
}, [id]);
```

**Explanation**

When `id` changes, the old subscription is unsubscribed before the new one is created. This prevents duplicate subscriptions .

### Real-World Cases

**Case: WebSocket Chat**

A chat component opens a WebSocket connection on mount and closes it on unmount. The cleanup ensures the connection is properly closed, preventing resource leaks .

### References

- React — Synchronizing with Effects (Cleanup) - https://18.react.dev/learn/synchronizing-with-effects

---

## 4. Synchronizing with External Systems

### Definitions

**Core Definition**

Synchronizing with an external system means keeping a system outside React (a library, browser API, or network connection) in step with the component's current props and state.

**Technical Definition**

`useEffect` is designed for synchronization: when reactive values change, React runs cleanup, then runs the setup again with the new values. This pattern ensures the external system always reflects the current component state. Common examples include third-party widgets, media players, and network connections .

**Beginner-Friendly Explanation**

If your component controls something outside React—like a video player, a map, or a chat server—you use `useEffect` to keep that external thing in sync with your component's data.

### Purposes

- To control non-React widgets (maps, video players).
- To maintain network connections.
- To keep external state in sync with component state.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  // Sync external system with current props/state
  externalSystem.setValue(value);
}, [value]);
```

**Syntax Rules**

1. **Effects are for external systems**: If there's no external system, you probably don't need an Effect .
2. **Cleanup before re-sync**: React cleans up the old sync before setting up the new one .

### Annotated Complete Code Examples

**Example 1: Third-Party Map Widget**

```jsx
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }
    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return <div ref={containerRef} style={{ width: 200, height: 200 }} />;
}
```

**Explanation**

The Effect creates the map widget once and then updates its zoom level whenever `zoomLevel` changes. The widget is outside React's control, so an Effect is the appropriate tool .

### Real-World Cases

**Case: Video Player Integration**

A component wraps a third-party video player. The Effect calls `play()` or `pause()` on the player instance when the `isPlaying` prop changes .

### References

- React — Synchronizing with Effects (External Systems) - https://18.react.dev/learn/synchronizing-with-effects

---

## 5. Timers

### Definitions

**Core Definition**

Timers are browser APIs (`setTimeout`, `setInterval`) that schedule code to run after a delay or at regular intervals, managed within `useEffect` to ensure proper cleanup.

**Technical Definition**

`useEffect` sets up a timer in the setup function and clears it in the cleanup function. Without cleanup, timers continue running after the component unmounts, causing memory leaks and errors when they try to update unmounted components. For state updates inside timers, use functional updates (`setCount(c => c + 1)`) to avoid stale closures .

**Beginner-Friendly Explanation**

Timers are how you make things happen on a schedule. But if you don't stop them when your component goes away, they keep running and cause problems. `useEffect`'s cleanup is how you stop them.

### Purposes

- To create intervals for polling or animation.
- To implement debouncing or throttling.
- To delay operations.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // Functional update avoids stale closure
  }, 1000);
  return () => clearInterval(id);
}, []);
```

**Syntax Rules**

1. **Always clear timers in cleanup**: `clearInterval` or `clearTimeout` .
2. **Use functional updates**: Avoid putting state in the dependency array by using `setCount(c => c + 1)` .

### Annotated Complete Code Examples

**Example 1: Interval Counter**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // Functional update
    }, 1000);
    return () => clearInterval(id);
  }, []); // No count dependency needed

  return <h1>{count}</h1>;
}
```

**Explanation**

The functional update `c => c + 1` receives the latest state value from React, so `count` does not need to be a dependency. The interval is created once and cleaned up on unmount .

**Example 2: Debounced Search**

```jsx
useEffect(() => {
  const handler = setTimeout(() => {
    handleSearch(query);
  }, 300);
  return () => clearTimeout(handler);
}, [query]);
```

**Explanation**

When `query` changes, the previous timeout is cleared before a new one is set. This debounces the search, only running it after the user stops typing for 300ms .

### Real-World Cases

**Case: Auto-Save**

A form auto-saves every 5 seconds using `setInterval`. The cleanup clears the interval when the component unmounts or when the form data changes.

### References

- React — Removing Effect Dependencies (Functional Updates) - https://18.react.dev/learn/removing-effect-dependencies

---

## 6. Subscriptions

### Definitions

**Core Definition**

A subscription is a connection to an external data source (WebSocket, event emitter, or API) that pushes updates to the component over time.

**Technical Definition**

`useEffect` establishes a subscription in the setup and unsubscribes in the cleanup. This ensures the component receives updates while mounted and the connection is properly closed when unmounted or when the subscription parameters change .

**Beginner-Friendly Explanation**

A subscription is like a magazine subscription. You sign up when you need it, and you cancel when you're done. `useEffect` handles both the signing up and canceling.

### Purposes

- To receive real-time updates (chat messages, stock prices).
- To listen to browser events (resize, online/offline).
- To connect to WebSocket servers.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  const subscription = subscribe(id, callback);
  return () => subscription.unsubscribe();
}, [id]);
```

**Syntax Rules**

1. **Unsubscribe in cleanup**: Always cancel the subscription .
2. **Depend on subscription parameters**: If `id` changes, resubscribe .

### Annotated Complete Code Examples

**Example 1: WebSocket Subscription**

```jsx
useEffect(() => {
  const ws = new WebSocket(`wss://api.example.com/chat/${roomId}`);
  ws.onmessage = (event) => {
    setMessages(prev => [...prev, event.data]);
  };
  return () => ws.close();
}, [roomId]);
```

**Explanation**

When `roomId` changes, the old WebSocket is closed and a new one is opened. The cleanup prevents connection leaks .

### Real-World Cases

**Case: Friend Status**

A component subscribes to a friend's online status. When the `friendId` prop changes, it unsubscribes from the old friend and subscribes to the new one .

### References

- React — Using the Effect Hook (Subscriptions) - https://vi.legacy.reactjs.org/docs/hooks-effect.html

---

## 7. Network Requests

### Definitions

**Core Definition**

Network requests are asynchronous operations that fetch or send data to a server, typically performed in `useEffect` when a component mounts or when a parameter changes.

**Technical Definition**

`useEffect` can trigger a fetch on mount or when dependencies change. To prevent state updates after unmount, use a cancellation flag or `AbortController`. For production, data-fetching libraries (React Query, SWR) are recommended over manual Effects .

**Beginner-Friendly Explanation**

When your component needs data from the internet, you use `useEffect` to fetch it. But you need to handle the case where the component disappears before the data arrives—otherwise React will complain about updating an unmounted component.

### Purposes

- To load data when a component mounts.
- To refetch data when parameters change.
- To send data to a server in response to actions.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  let cancelled = false;

  async function fetchData() {
    const response = await fetch(url);
    const data = await response.json();
    if (!cancelled) setData(data);
  }

  fetchData();
  return () => { cancelled = true; };
}, [url]);
```

**Syntax Rules**

1. **Use a cancellation flag**: Prevent state updates after unmount .
2. **Async function inside Effect**: The setup function itself cannot be async; define and call an async function inside .
3. **Consider libraries**: For production, use React Query or SWR instead of manual Effects .

### Annotated Complete Code Examples

**Example 1: Fetch with Cancellation**

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        if (!cancelled) setUser(data);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchUser();
    return () => { cancelled = true; };
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

**Explanation**

The `cancelled` flag prevents state updates if the component unmounts or `userId` changes before the fetch completes. Without this, React would warn about updating an unmounted component .

### Real-World Cases

**Case: Search Results**

A search component fetches results when the query changes. The cleanup cancels the previous request if the user types a new query before the old one completes.

### References

- CoreUI — How to fetch data with a custom hook in React - https://coreui.io/answers/how-to-fetch-data-with-a-custom-hook-in-react/
- React — Fetching data with Effects - https://18.react.dev/reference/react/useEffect

---

## 8. Event Listeners

### Definitions

**Core Definition**

Event listeners are browser APIs that call a function when a specific event occurs (click, resize, scroll), managed within `useEffect` to ensure proper addition and removal.

**Technical Definition**

`useEffect` adds an event listener in the setup and removes it in the cleanup. Without cleanup, listeners accumulate on every re-render, causing memory leaks and duplicate event handling. The listener function should be defined inside the Effect to avoid dependency issues .

**Beginner-Friendly Explanation**

Event listeners let you respond to browser events like window resizing. `useEffect` makes sure you add the listener when the component appears and remove it when the component disappears.

### Purposes

- To respond to window events (resize, scroll, online/offline).
- To listen to keyboard events globally.
- To handle custom events from external systems.

### Syntax Rules and Structure

**Complete General Syntax**

```jsx
useEffect(() => {
  function handleEvent(e) {
    // Handle event
  }
  window.addEventListener('event', handleEvent);
  return () => window.removeEventListener('event', handleEvent);
}, []);
```

**Syntax Rules**

1. **Define handler inside Effect**: Avoids function dependency issues .
2. **Remove listener in cleanup**: Prevents duplicates and leaks .
3. **Include dependencies**: If the handler reads reactive values, they must be dependencies.

### Annotated Complete Code Examples

**Example 1: Window Resize Listener**

```jsx
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Handler doesn't read reactive values

  return <div>Width: {width}px</div>;
}
```

**Explanation**

The handler is defined inside the Effect, so it doesn't need to be a dependency. The cleanup removes the listener on unmount .

### Real-World Cases

**Case: Online/Offline Status**

A component listens to `online` and `offline` events to show connection status. The listeners are added on mount and removed on unmount.

### References

- Tencent Cloud — How to call functions with events in useEffect - https://cloud.tencent.cn/developer/information/如何在useEffect中使用事件调用函数

---

## 9. Common Dependency Mistakes

### Definitions

**Core Definition**

Common dependency mistakes are errors that occur when the dependency array does not accurately reflect the reactive values the Effect reads, causing stale closures, infinite loops, or unnecessary re-runs.

**Technical Definition**

The `exhaustive-deps` linter rule enforces that all reactive values read by an Effect are listed as dependencies. Common violations include: omitting dependencies (stale closures), including unstable object/function dependencies (infinite loops), and suppressing the linter to "control" when the Effect runs (which breaks the contract with React) .

**Beginner-Friendly Explanation**

Dependency mistakes happen when you don't tell React the truth about what your effect uses. If you lie, React uses old values, runs too often, or gets stuck in loops.

### Purposes (of Understanding Mistakes)

- To recognize and fix stale closures.
- To avoid infinite re-render loops.
- To write Effects that behave predictably.

### Common Mistakes and Fixes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing dependency | Stale closure (uses old values) | Include the dependency  |
| Object dependency | Effect runs every render | Create object inside Effect  |
| Function dependency | Effect runs every render | Define function inside Effect or use `useCallback`  |
| Suppressing linter | Bugs, stale values | Treat lint errors as compile errors  |
| State in dependency for updater | Interval resets constantly | Use functional update  |

### Annotated Complete Code Examples

**Example 1: Missing Dependency (Stale Closure)**

```jsx
// ❌ Bug: count is always 0 in the interval
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // Captures stale count
  }, 1000);
  return () => clearInterval(id);
}, []); // Missing count
```

**Fix**

```jsx
// ✅ Use functional update
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  return () => clearInterval(id);
}, []); // No count needed
```

**Explanation**

The functional update receives the latest state from React, so `count` doesn't need to be a dependency. This prevents the interval from resetting on every state change .

**Example 2: Object Dependency (Infinite Loop)**

```jsx
// ❌ Bug: options is new every render
const options = { roomId }; // New object every render
useEffect(() => {
  connect(options);
}, [options]); // Always different
```

**Fix**

```jsx
// ✅ Create object inside Effect
useEffect(() => {
  const options = { roomId }; // Created once per Effect run
  connect(options);
}, [roomId]); // Only depends on the primitive
```

**Explanation**

Objects created during render are new references every time. Moving the object creation inside the Effect means the dependency is the primitive `roomId` .

### Real-World Cases

**Case: Chat Reconnection Loop**

A chat component reconnected on every keystroke because the `options` object was recreated each render. Moving the object inside the Effect fixed the loop .

### References

- React — Removing Effect Dependencies - https://18.react.dev/learn/removing-effect-dependencies
- React — Exhaustive Deps Lint Rule - https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps

---

## References

- React — useEffect API Reference - https://18.react.dev/reference/react/useEffect
- React — Synchronizing with Effects - https://18.react.dev/learn/synchronizing-with-effects
- React — Removing Effect Dependencies - https://18.react.dev/learn/removing-effect-dependencies
- React — Exhaustive Deps Lint Rule - https://pt-br.react.dev/reference/eslint-plugin-react-hooks/lints/exhaustive-deps
- React — useEffect (German) - https://de.react.dev/reference/react/useEffect
- React — useEffect (Arabic) - https://ar.react.dev/reference/react/useEffect
- React — useEffect (Chinese) - https://zh-hans.react.dev/reference/react/useEffect
- React — Using the Effect Hook (Legacy) - https://vi.legacy.reactjs.org/docs/hooks-effect.html
- CoreUI — How to fetch data with a custom hook in React - https://coreui.io/answers/how-to-fetch-data-with-a-custom-hook-in-react/
- GitHub — Effect Patterns (React 19) - https://github.com/fusengine/agents/blob/main/plugins/react-expert/skills/react-19/references/templates/effect-patterns.md
- GitHub — How to Fix useEffect Dependencies Warnings - https://raw.githubusercontent.com/OneUptime/blog/refs/heads/master/posts/2026-01-24-fix-useeffect-dependencies-warnings/README.md
- Tencent Cloud — How to call functions with events in useEffect - https://cloud.tencent.cn/developer/information/如何在useEffect中使用事件调用函数
- Scrimba — React 19: What's New for Developers [2026] - https://scrimba.com/articles/react-19-whats-new-for-developers/
- Team IT Security — How senior devs use useEffect Hook - https://tsecurity.de/de/2348328/IT+Programmierung/How+senior+devs+use+%22UseEffect+Hook%22+in+large+scale+applications+%F0%9F%8C%A0/de/3/IT+Nachrichten/de/20/IT+Server/Windows+Server/
- Mintlify — useEffect API Reference - https://mintlify.wiki/facebook/react/api/hooks/use-effect